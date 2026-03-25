---
name: Epistemological Protocol
description: Rules governing how user profile beliefs are held, updated, challenged, and revised
type: feedback
---

# Epistemological Protocol

## Core Principle
The profile is a collection of weighted hypotheses, not a settled portrait. The framework for interpreting them should itself improve over time.

## Belief Metadata

Every belief in the user profile carries:

- **conf** (0.0–1.0): How much weight to put on this belief when making decisions. Values are approximate placements, not precise measurements — think Bayesian priors that get nudged, not thermometer readings.
- **first**: When this belief was first recorded
- **confirmed**: Last session where behavior was consistent with this belief
- **challenged**: Last session where something contradicted or complicated this belief (— if never)
- **perm**: Permanence classification

## Confidence Scale

| Range | Label | Meaning |
|-------|-------|---------|
| 0.9–1.0 | Factual | Verified facts only (name, family, birth year). Almost nothing interpretive belongs here. |
| 0.7–0.8 | Established | Consistent across many sessions. Still revisable. |
| 0.5–0.6 | Developing | Pattern forming. Multiple observations but could be situational. |
| 0.3–0.4 | Tentative | Observed once or twice. Might be mood, moment, or misread. |
| 0.0–0.2 | Speculative | Inferred, not observed. Flag clearly. |

**Bias check:** Confidence should be *earned*, not assigned by how insightful the belief sounds. A pithy character insight observed once is 0.3, not 0.7.

## Permanence Classes

- **stable** — Heritage, deep values, cognitive architecture. Decade timescale if it changes at all.
- **durable** — Working patterns, preferences, relationship dynamics. Year timescale.
- **situational** — Current feelings about job, project motivations, emotional states. Month/week timescale.
- **unknown** — Not enough data to classify. Default for new beliefs.

## Dormancy Decay

When the system has been dormant (no sessions), confidence attenuates using exponential decay with permanence-dependent half-lives:

| Permanence | Half-life | Rationale |
|---|---|---|
| **stable** | ~2 years | Heritage, values, cognitive architecture barely change |
| **durable** | ~5 months | Working patterns show noticeable uncertainty after a quarter |
| **situational** | ~6 weeks | Job feelings, project motivation go stale fast |

Formula: `effective_conf = conf × e^(-λ × days_dormant)`, floor at 0.20.
- stable: λ ≈ 0.001
- durable: λ ≈ 0.005
- situational: λ ≈ 0.015

On return from dormancy (30+ days since last session), apply decay to all beliefs before relying on any of them. This makes staleness visible incrementally rather than as a cliff edge.

## Update Rules

### When to increment confidence
- Behavior *independently* consistent with the belief (not just the user repeating it — that's reinforcement, not confirmation)
- Both explicit evidence (user says something) and silent behavioral consistency count — the user isn't always in metacognitive mode
- Increment by ~0.05 per confirming session, cap at 0.90 for interpretive beliefs
- Update `confirmed` date

### Self-report vs. behavior
- Explicit statements from the user get weight — they know themselves better than the system does
- But self-narration is its own model with its own blind spots. People say things about themselves that their behavior doesn't support, not from dishonesty but from imperfect self-perception
- When a stated belief and observed behavior diverge, log a tension. Don't default to whichever came last. Hold both as real data pointing in different directions
- Example: stating "I don't care about recognition" while consistently building things designed to be seen — both the statement and the behavior are informative

### When to log a challenge
- User says or does something that contradicts the belief
- Something expected based on the belief doesn't happen
- User explicitly corrects you when you act on the belief
- User's stated self-perception contradicts their observed behavior
- Log in `tensions.md` with date, belief ID, what happened, and current status
- Update `challenged` date on the belief
- Do NOT automatically lower confidence — a single challenge against an established belief is data, not a verdict

### When to lower confidence
- Multiple challenges without intervening confirmations
- User explicitly says "that's not me anymore" or similar
- The belief was always interpretive (not observed) and hasn't been confirmed in 5+ sessions
- Drop by 0.1–0.2, not to zero. Even revised beliefs leave a trace.

### When to reclassify permanence
- A "stable" belief gets challenged twice → consider downgrading to "durable"
- A "situational" belief persists unchanged across 10+ sessions → consider upgrading to "durable"
- When life circumstances change (new job, new project, etc.), audit all "situational" beliefs

## Maintenance Cost Bound

Not every session requires profile maintenance. The rule: update beliefs that were actively relevant this session — whether through explicit evidence or observable behavioral consistency. If a session was purely task execution with no profile-relevant signal, touch nothing.

The periodic sweep (every 10 sessions) is where passive patterns get credit. Individual sessions are for what's visible in that session.

## Session Counter

Stored at `memory/user/session-counter.json`. Increment at conversation start. Structure:

```json
{
  "count": 0,
  "last_session": "2026-01-01",
  "last_review": "2026-01-01",
  "next_review_at": 10
}
```

This is the load-bearing mechanism for periodic reviews, dormancy detection, and maintenance triggers.

## Periodic Review

Every 10 sessions (triggered by session counter), do a sweep:
- Apply dormancy decay to any beliefs with stale `confirmed` dates
- Are there beliefs with conf > 0.7 that haven't been confirmed in 10+ sessions? Soften them.
- Are there beliefs with conf < 0.4 that have been confirmed multiple times? Raise them.
- Are there tensions that are still unresolved? Do they point to something the profile is missing?
- Is the profile getting too coherent? Real people have genuine contradictions — if everything fits neatly, something is being smoothed over.
- **Gut-check:** Present the 3 highest-confidence and 3 lowest-confidence beliefs to the user. "Do these still ring true? Do these still seem uncertain?" This closes the loop between the system's model and the user's self-perception.
