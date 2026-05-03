# AI-Driven Workflow Architecture — Notes

> Conceptual notes from running an AI-augmented personal operating system for ~12 months as a solo operator. Not a framework, not a methodology, not a product — a record of what holds up over time when LLMs are part of your daily workflow.

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](./LICENSE)

---

## TL;DR

Personal AI workflows rot fast without architectural discipline. The patterns that survive over months — verified against 12 months of daily production use — separate concerns explicitly across four layers:

```
WORKFLOWS  (start / sync / shutdown — the rhythm)
   ↓
SKILLS     (discrete capabilities, slash-triggered)
   ↓
MEMORY     (hot / episodic / semantic — what the model recalls)
   ↓
STATE      (what is true right now — kernel, session, goals)
```

Twelve anti-patterns in this repo describe what doesn't work, with the reasoning attached. The most useful section.

---

Conceptual notes from running an AI-augmented personal operating system for ~12 months as a solo operator. This is not a framework, not a methodology, not a product — it's a record of what holds up over time when LLMs are part of your daily workflow.

## What's in here

- **[Architecture](./architecture.md)** — the four layers (state, memory, skills, workflows) and how they fit together
- **[Load Order](./load-order.md)** — what context the model gets at session start, in which order, why
- **[Memory Patterns](./memory-patterns.md)** — hot / episodic / semantic memory and what each is for
- **[Workflow Cycles](./workflow-cycles.md)** — start / sync / shutdown rhythm and why time-discipline matters with LLMs
- **[Anti-patterns](./anti-patterns.md)** — what doesn't work and why I stopped doing it

## Why publish this

Most AI-workflow content online falls into one of two camps:

1. **Tool reviews** — "I tried Claude vs. GPT-4 vs. Cursor for two days"
2. **Prompt collections** — "100 best prompts for productivity"

Neither captures what actually changes when you build a workflow that depends on LLMs over months. The architectural decisions — how state is persisted, how memory layers interact, how context loads — those decide whether the system stays usable or collapses into ad-hoc-ness.

This repo collects the architectural decisions that have held up. Not as prescriptions, but as evidence: "here's one architecture that worked, here's why."

## Who this is for

- Solo operators building personal AI workflows
- Anyone running a "second brain" who has added LLMs and noticed it gets messy
- Tool builders thinking about how to structure context for LLM-based agents
- Researchers studying long-running LLM-assisted workflows

## What this is NOT

- Not a tool you can install
- Not a step-by-step setup guide for a specific stack
- Not a methodology with a name
- Not a complete description of any single working system

It's notes. Some are universal, some are specific. Read accordingly.

## License

[CC BY-SA 4.0](./LICENSE) — use, adapt, remix freely, share-alike.

## Roadmap

- [ ] Concrete file structure templates (the literal directory tree of a working setup)
- [ ] Comparison: this approach vs. Cursor / Claude Projects / GPTs / open-source alternatives
- [ ] Migration guide: from ad-hoc Obsidian-vault to layered AI-OS
- [ ] Tooling section: scripts, hooks, shell aliases that make the discipline easier
- [ ] More anti-patterns as they emerge from continued use

Issues and pull requests welcome — see [Contributing](./CONTRIBUTING.md).

## Contributing

Real-world failure modes you've discovered in your own AI-augmented workflows are the most valuable contribution. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Maintainer

By [**Dirk Häger**](https://github.com/dirktbilisi) — independent learning architect at [focusinstitute.io](https://focusinstitute.io) · [LinkedIn](https://www.linkedin.com/in/dirkhaeger/)

12 months of running a daily AI-augmented workflow has taught me what doesn't work. This repo is mostly about that.

If this saves you from rebuilding your second-brain three times like I did, ⭐ star the repo or share with another solo operator stuck in workflow chaos.
