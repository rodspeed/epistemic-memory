# Epistemic Memory

**A protocol for AI assistants to hold beliefs as weighted hypotheses — not settled facts.**

Most AI memory systems accumulate observations about you, treat each one as equally true, and never forget or doubt anything. "You're a deep thinker" carries the same weight as "you use dark mode." A preference stated six months ago is treated as current gospel. Contradictions get smoothed over. The result is a flattering caricature that diverges from the actual person over time.

Epistemic Memory fixes this by treating every stored belief as a hypothesis — with explicit confidence, decay, and contradiction tracking.

## What This Is

- **A protocol**, not a library. No code, no dependencies, no build step.
- **Templates** for storing user beliefs with per-belief metadata (confidence, permanence, dates).
- **A skill** (`/mirror`) for Claude Code that surfaces what the AI thinks it knows — as a prose portrait, an audit dashboard, or a structured interview.
- **Drop-in adoption.** Copy the files into your Claude Code project and start using it.

## The Core Idea

Every belief about the user carries metadata:

```
## Builder Philosophy
`conf:0.75 | first:2026-03-10 | confirmed:2026-03-25 | challenged:— | perm:durable`

Builds for herself first, monetizes if it generalizes.
```

| Field | What it means |
|-------|---------------|
| `conf` | Confidence (0.0–1.0). Must be *earned* through evidence, not assigned by how insightful it sounds. |
| `first` | When this belief was first recorded. |
| `confirmed` | Last session where behavior was consistent with this belief. |
| `challenged` | Last session where something contradicted it. `—` if never. |
| `perm` | How stable this trait is: `stable` (decade), `durable` (year), `situational` (month), `unknown`. |

Confidence and permanence are independent. You can be certain about something that will change (high conf, situational), or uncertain about something deep (low conf, stable).

## 10 Components

1. **Per-belief metadata** — no belief exists as bare text
2. **Confidence that must be earned** — observable behavior outranks interpretive narrative
3. **Permanence classes** — separates "how sure" from "how stable"
4. **Dormancy decay** — confidence attenuates when the system goes unused
5. **Tensions log** — contradictions stay on record instead of being resolved away
6. **Self-report vs. behavior rule** — when stated and observed diverge, hold both
7. **Silent consistency** — behavioral patterns count as evidence without explicit narration
8. **Session counter** — mechanical trigger for periodic reviews (every 10 sessions)
9. **Maintenance cost bound** — only update beliefs relevant to the current session
10. **Evolving epistemology** — the framework itself improves over time

Full rationale, adversarial review findings, and limitations: [docs/design.md](docs/design.md)

## Quick Start

1. Copy `memory/` into your Claude Code project memory directory
2. Copy `.claude/skills/mirror/` into your project's `.claude/skills/`
3. Add the snippet from [`CLAUDE.md.example`](CLAUDE.md.example) to your project's `CLAUDE.md`
4. Start a conversation — beliefs accumulate naturally as you work together
5. Run `/mirror` to see what the AI thinks it knows about you

## The Mirror Skill

Four modes for inspecting and refining the model:

| Command | What it does |
|---------|-------------|
| `/mirror` | Prose portrait — "here's who I think you are" |
| `/mirror audit` | Epistemological dashboard — confidence distributions, decay, drift |
| `/mirror gut-check` | Quick validation of highest and lowest confidence beliefs |
| `/mirror interview` | Structured conversation that probes blind spots and tests stale beliefs |

## Why Not Just Use MEMORY.md?

Claude Code's built-in memory stores facts: "prefers dark mode," "uses Python," "name is Alex." Epistemic Memory stores hypotheses: beliefs held at different weights, with mechanisms to weaken stale ones, flag contradictions, and periodically ask "am I still right about this?" The difference is between a filing cabinet and a living model.

## FAQ

**Is this only for Claude Code?**
The protocol is general — any AI system with persistent memory can adopt the belief format and update rules. The Mirror skill is Claude Code specific, but the memory structure works anywhere.

**How many beliefs before this is useful?**
5–10 is enough. The value shows up as soon as you have beliefs at different confidence levels and the system starts distinguishing what it knows well from what it's guessing.

**What if I don't want the Mirror skill?**
The protocol stands alone. Add metadata to your existing memory files and follow the update rules. The skill is optional tooling on top.

**How is this different from RAG or vector memory?**
RAG retrieves relevant context. This is about the *quality* of what's stored — whether the system knows what it knows, what it's guessing, and what's gone stale. They're complementary, not competing.

## Built From Use

This is a framework and starter kit, extracted from a personal system used in daily Claude Code sessions. The repo ships clean — session counter at zero, example profile fictional — so you build your own from scratch. It's not a theoretical proposal; it was stress-tested through adversarial review before release. See [docs/design.md](docs/design.md) for the full story, including what broke in v1 and how it was fixed.

## License

[MIT](LICENSE)
