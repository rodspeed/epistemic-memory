# Epistemic Memory Framework — Design Document

A design for AI memory that knows what it doesn't know. This framework treats every belief about the user as a hypothesis rather than a fact — with explicit confidence, decay, and contradiction tracking.

## The Problem

Most AI memory systems work like a flattering portrait. They accumulate observations about the user, treat each one as equally true, and never forget or doubt anything. The result is a caricature: a static, internally consistent model that diverges from the actual person over time.

The failure modes are specific:

- **Confidence without evidence** — "you're a deep thinker" carries the same weight as "you use dark mode," despite one being observable and the other being interpretive.
- **No decay** — a preference stated six months ago is treated as current gospel.
- **Coherence bias** — contradictions get smoothed over. The system builds a neat narrative that misses the mess of actual human behavior.
- **No self-audit** — the system never asks "am I wrong about this?"

The founding insight: "leave the door ajar for surprises — humans are messy, with contradictions wrapped within contradictions." The framework is designed to hold that messiness without collapsing it into false clarity.

## The Core Design

Ten components, each solving a specific failure mode.

### 1. Per-Belief Metadata

Every belief carries: confidence score (0.0–1.0), first observed date, last confirmed date, last challenged date, and permanence class. No belief exists as bare text — the metadata is the point.

### 2. Confidence That Must Be Earned

Confidence reflects evidence quality, not how insightful a belief sounds. Observable behaviors ("uses bullet points in every message") score higher than interpretive models ("values efficiency over thoroughness"). Hard cap at 0.90 for anything interpretive — you never fully know another person's inner state.

### 3. Permanence Classes

Separates "how confident am I" from "how stable is this trait." Four classes:

- **Stable** (decade timescale) — heritage, core values, cognitive style
- **Durable** (year timescale) — work patterns, tool preferences, creative habits
- **Situational** (month/week timescale) — job feelings, current project priorities, mood patterns
- **Unknown** — default for new beliefs until enough data exists to classify

A belief can be high-confidence and situational (you're certain about something that will change), or low-confidence and stable (you're uncertain about something that won't).

### 4. Dormancy Decay

When the system goes unused, confidence attenuates. Exponential decay with permanence-dependent half-lives: stable ~2 years, durable ~5 months, situational ~6 weeks. Same math as forgetting curves — because the problem is the same. A system that hasn't seen you in three months should be less sure about your situational preferences.

### 5. Tensions Log

A running record of contradictions and surprises. The default status for any entry is "unresolved." This is deliberate — it resists the urge to wrap every contradiction in a resolution narrative. Some tensions are real and persistent. The user who craves deep work but keeps choosing interrupt-driven roles isn't confused; they're human.

### 6. Self-Report vs. Behavior Rule

When the user says "I'm a morning person" but every session starts at 11pm, both data points go on the record. Neither automatically wins. Stated self-perception and observed behavior are different evidence streams. The divergence itself is often the most interesting signal.

### 7. Silent Consistency Counts

The system doesn't wait for the user to narrate themselves. If someone consistently asks for concise output across 20 sessions without ever saying "I prefer brevity," that pattern confirms a belief about communication preferences. Behavioral evidence accumulates without requiring metacognitive statements from the user.

### 8. Session Counter

Every 10 sessions, the system triggers a periodic review: present the 3 highest and 3 lowest confidence beliefs for the user to validate, challenge, or ignore. This is the load-bearing enforcement mechanism — without it, the review process is aspirational rather than real. The counter turns "we should periodically check" into "we check on session 10, 20, 30."

### 9. Maintenance Cost Bound

Only update beliefs relevant to the current session. Background sweeps handle everything else. This prevents the framework from becoming a tax on every interaction. If you're discussing a screenplay, don't re-evaluate beliefs about work preferences.

### 10. The Epistemology Evolves

The framework for interpreting beliefs should itself improve over time. New permanence classes can emerge. Decay rates can be tuned based on observed accuracy. The rules for weighting evidence can change. Treating the epistemology as fixed would be ironic.

## What the Adversarial Review Found

The framework went through a structured adversarial review before deployment. A fresh-context agent was asked to assume the design was wrong and find specific failure modes. This is what was wrong in v1:

- **Untrackable field** — the original design included an "observation count" per belief. In practice, there's no reliable way to count observations in a conversation-based system. What counts as one observation? The field was dropped entirely rather than left as fiction.
- **Speculative psychometrics mixed with observables** — v1 stored "estimated IQ range" and "likely MBTI" alongside observable traits like "prefers bullet points." These are fundamentally different kinds of claims. The fix: split them into separate categories with different confidence rules. Speculative psychometric estimates are capped lower and flagged as interpretive.
- **Coherence bias in the tensions log** — the original tensions log template included a "likely resolution" field. This subtly encouraged the system to resolve tensions prematurely. The field was removed. Unresolved is the healthy default.
- **No enforcement mechanism** — the periodic review was described but had no trigger. Adding the session counter (component 8) gave it teeth. This was the most important fix — a review process that depends on the AI remembering to do it will not happen reliably.

## Benefits Observed

What changes when you have epistemic memory vs. flat memory:

- **Graceful aging** — after a gap, the system returns with appropriately reduced confidence instead of stale certainty. It says "last time we talked, you were frustrated with X — is that still the case?" instead of assuming.
- **Contradiction tolerance** — tensions stay on the record instead of being silently resolved. This matches how people actually experience themselves.
- **Calibrated trust** — the user can see *why* the system believes something and how confident it is. This makes corrections targeted rather than wholesale ("your confidence on that is too high" vs. "you don't know me").
- **Self-correcting over time** — decay, periodic reviews, and the tensions log create pressure toward accuracy rather than accumulation.
- **Resistance to sycophancy** — when confidence must be earned through evidence, the system can't build a flattering model from thin air. Observable behavior outranks interpretive narrative.

## Limitations and Open Questions

- **Bootstrap problem** — the framework is most useful once you have 20+ beliefs, but the early sessions are when you most need good epistemics. Cold-start heuristics are still rough.
- **Decay rates are guesses** — the half-lives (2yr/5mo/6wk) are reasonable but not empirically validated. They need tuning per user.
- **Session counter assumes regular use** — if someone uses the system twice a year, a 10-session review cycle means reviews every 5 years. The counter may need a time-based fallback.
- **Tensions log can grow unbounded** — no mechanism yet for archiving resolved tensions or pruning stale ones.
- **Single-user design** — this was built for one user working with one AI. Multi-user systems would need tenant isolation and different decay models.
- **No ground truth** — there's no external measure of whether the belief model is accurate. The periodic review is the closest thing, and it depends on the user's own self-knowledge.

## How to Implement

Starting points for building persistent AI memory with this framework:

1. **Start with the metadata, not the beliefs.** Add confidence scores and dates to whatever memory format you already use. This alone changes how you think about stored information.
2. **Pick your permanence classes.** The four listed here (stable/durable/situational/unknown) work for personal profiles. Domain-specific systems might need different categories.
3. **Implement decay before anything else.** It's the cheapest component and provides the most immediate value. Stale certainty is the most common failure mode in AI memory.
4. **Add the tensions log early.** Even if you don't have the session counter or periodic reviews yet, recording contradictions changes the system's posture from "I know you" to "I'm learning you."
5. **Build the session counter.** Whatever review process you design, give it a mechanical trigger. Aspirational maintenance doesn't happen.
6. **Run an adversarial review.** Have someone (or a separate AI session) try to break your framework. Assume the design is wrong and look for specific failure modes. Every framework has them.

The framework is portable — nothing here depends on a specific AI provider or memory format.
