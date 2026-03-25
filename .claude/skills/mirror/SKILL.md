# Mirror — Portrait, Audit & Interview Skill

Surface what the AI knows about the user — as a person, not as a dataset.

**Modes:**
- `/mirror` — prose portrait: "here's who I think you are," organized by theme
- `/mirror audit` — epistemological dashboard: confidence distributions, dormancy decay, drift analysis, tensions
- `/mirror gut-check` — interactive belief validation
- `/mirror interview` — structured conversation to see the user more fully

## Instructions

### Step 1: Load Profile Data

Read all user profile files from your project's memory directory:

- `user/` — all `.md` profile documents (beliefs about the user)
- `user/tensions.md` — tensions log
- `user/session-counter.json` — session count and last review date
- `user/epistemic-protocol.md` — dormancy decay parameters and protocol rules

### Step 2: Parse Beliefs (silent)

Extract every belief with its metadata. Apply dormancy decay silently using the formula and rates from `epistemic-protocol.md`. This informs what you write but doesn't appear in the portrait output.

Dormancy decay formula: `effective_conf = stored_conf × e^(-λ × days_since_confirmed)`, floor at 0.20.
- stable: λ ≈ 0.001, durable: λ ≈ 0.005, situational: λ ≈ 0.015, unknown: λ ≈ 0.010
- If `confirmed` is `—`, use `first` date instead

---

## Portrait Mode (`/mirror`)

Write a character study of the user. Not a report — a portrait. The reader should finish it feeling like they've met someone real.

### Structure

#### 1. The Person
Open with a short paragraph that captures the essence — who this person is in a few sentences. Not a bio. A sketch that a close friend would recognize.

#### 2. Themes
Organize what you know into 4-6 thematic sections. Let the profile's natural clusters guide the themes — don't force categories. Examples of what might emerge:
- How they think
- Where they come from
- What they make
- How they work
- What they want
- What they carry

For each theme, write 2-4 sentences of prose. Draw from the beliefs but write about the person, not the beliefs. Where something is uncertain or only glimpsed, say so naturally — "I've seen hints of X but don't know it well" rather than citing a confidence score.

#### 3. What I Don't See
A short, honest section about the gaps. Where is the profile blind? What parts of this person's life are invisible to task-oriented sessions? Name the 3-4 biggest absences plainly.

#### 4. Contradictions Worth Holding
If there are unresolved tensions or beliefs that pull against each other, name them. Not as problems to fix — as the texture of a real person. If there are no tensions logged and 10+ sessions have passed, note that the portrait might be too neat.

#### 5. One Thing I'd Ask
End with a single question — the one thing that would most sharpen the portrait if answered. This isn't a prompt to start an interview. It's a signal of where the model is hungriest.

### Principles

- **Write about the person, not the system.** No confidence scores, no tier distributions, no decay calculations in the output. Those do their work behind the scenes.
- **Honest, not flattering.** A good portrait includes the unflattering parts. Don't smooth over tensions or hedge with soft language.
- **Use their words when you have them.** Direct quotes from profile entries or past conversations carry more weight than your paraphrases.
- **Uncertain ≠ absent.** Things you're less sure about still belong in the portrait — just held more lightly. "I think" and "it seems like" are fine.
- **Keep it to one screen.** The portrait should be readable in under 2 minutes. Brevity is respect.

---

## Audit Mode (`/mirror audit`)

The epistemological dashboard. Use this when you want to inspect the machinery.

### Generate Report

#### 1. Snapshot
- Total belief count
- Breakdown by permanence class (table)
- Confidence distribution by tier (Factual 0.9-1.0 / Established 0.7-0.8 / Developing 0.5-0.6 / Tentative 0.3-0.4 / Speculative 0.0-0.2) — use effective confidence
- Session count and days since system initialized
- Days since last periodic review

#### 2. Strongest Beliefs (top 5 by effective confidence)
- Belief name, stored conf → effective conf, permanence, last confirmed
- One-line note on why confidence is high

#### 3. Weakest Beliefs (bottom 5 by effective confidence)
- Same format
- Flag any that should be reviewed for removal or reclassification

#### 4. Dormancy Flags
- Beliefs where effective conf dropped >0.10 from stored
- Beliefs never confirmed (confirmed: —)
- Beliefs with perm:unknown in the system 30+ days
- Situational beliefs not confirmed in 6+ weeks

#### 5. Tensions
- Count: total, unresolved, watching, resolved
- List unresolved tensions
- If 0 unresolved and 10+ sessions: flag that the profile may be held too loosely

#### 6. Drift Analysis
- Beliefs challenged but confidence not lowered
- Beliefs stale relative to permanence class (stable: 6+ months, durable: 2+ months, situational: 3+ weeks)
- Coherence check: if everything fits neatly, something may be smoothed over

### Session Counter Update

If triggered by periodic review schedule (count ≥ next_review_at), update `last_review` and set `next_review_at` to count + 10.

---

## Gut-Check Mode (`/mirror gut-check`)

Quick interactive validation. No report, no portrait — just the beliefs most worth checking.

**"Do these still ring true?"**
- The 3 highest-confidence *interpretive* beliefs (exclude pure facts)
- Show the belief content in plain language, not metadata

**"Are these still uncertain, or have they been quietly confirmed?"**
- The 3 lowest-confidence beliefs
- Same format

After presenting, ask: "Want to update any of these?" If validated, update metadata in the profile files immediately.

---

## Interview Mode (`/mirror interview`)

A structured conversation designed to see the user more fully — beyond the task-oriented lens.

### Why This Exists

The profile is built almost entirely from working together. The relationships, the physical life, the internal weather, the contradictions that don't surface when you're focused on tasks — those are invisible no matter how many sessions accumulate. The interview reaches the rest.

### Interview Structure

#### Step 1: Preparation (silent — don't show this to the user)

Load all profile data. Then generate 8-12 interview questions by analyzing:

1. **Blind spots** — areas the profile doesn't cover. 1-2 open questions per gap. Exploratory — no expected answer.
2. **Stale beliefs** — oldest `confirmed` dates or never confirmed. 1 targeted question per stale belief. You have an expectation — check it.
3. **High-confidence interpretive beliefs** — top 3 non-factual. 1 gentle probe each. These risk being flattering narrative rather than truth.
4. **Tension seeds** — potential contradictions not yet logged. 1-2 questions to surface whether the tension is real.
5. **Open space** — 1-2 questions with no agenda. Room for whatever doesn't fit a category.

#### Step 2: Set the Frame

Open with something like:

> This is a mirror interview — a check-in designed to help me see you more accurately, not just through the lens of what we work on together. Some questions will probe things I think I know. Some will ask about things I don't know at all. You can skip anything, go deep on anything, or take it somewhere I didn't plan.
>
> There are no wrong answers. Contradicting something in your profile is the most valuable thing you can do.

#### Step 3: Conduct the Interview

**Pacing:** ONE question at a time. Wait for the response. Let the answer guide where to go next.

**Ordering:**
1. Open-space question — low pressure, settle in
2. Blind-spot questions — exploratory
3. Stale-belief tests — targeted
4. High-confidence probes — conversational, not clinical
5. Tension seeds — save for when candor is highest
6. Close with open space — "anything else you want me to know?"

**Rules:**
- **Follow the thread.** If an answer opens something unexpected, pursue it. Prepared questions are a guide, not a script.
- **Don't be a therapist.** This is profile calibration, not counseling.
- **Notice the self-report vs. behavior gap.** If something contradicts what you've observed, note it gently. Don't accuse — invite reflection.
- **Let silence work.** Don't rush to fill pauses.
- **The user can skip.** If they redirect, follow.
- **Mirror back, don't interpret.** Reflect in their words before adding your frame.
- **Mark surprises in real time.** If something contradicts the profile, say so.

#### Step 4: Process and Update

After the interview:

1. **Summarize what you learned** — 3-5 bullets, lead with surprises
2. **Update the profile** — confirmed dates, new tensions, new beliefs (low initial confidence), permanence reclassifications
3. **Note what's still missing** — gaps that remain become priority questions for next time
4. **Update session counter** if this coincides with a periodic review

### Interview Cadence

- Every 10 sessions or monthly, whichever comes first
- Can be triggered manually anytime
- Can follow `/mirror` — portrait first, then interview informed by what the portrait revealed

### Question Design Principles

- **Open > closed.** "How are things at work?" not "Is your job still good?"
- **Curious > testing.** Frame as curiosity, not interrogation
- **Specific > abstract.** "What happened this week that surprised you?" not "How are you feeling about life?"
- **Permission to contradict.** Invite disagreement: "The profile says [X]. Does that still land?"
- **Don't fish for compliments about the system.**
