# Contributing

Thanks for considering a contribution. This repo is built on **real production use over months**, not theoretical architecture. The most valuable additions follow that same standard.

## High-value contributions

- **New anti-patterns** with the standard structure: what you tried, what happened, why it failed, what you do instead
- **Failure-mode reports** from your own AI-workflow experiments (with enough specificity to be useful)
- **Pattern refinements** with proof of why the change works better
- **Migration stories** — how you moved from ad-hoc to layered AI-OS, what broke, what worked
- **Adjacent tooling notes** — scripts, hooks, shell aliases that operationalize the discipline

## Lower-value contributions (please skip)

- Theoretical architecture proposals without months of personal use
- "Best practice" lists without failure-mode evidence
- Tool recommendations without integration patterns
- Generic productivity tips

## How to contribute

### Small fixes (typos, clarifications)
Open a PR directly.

### New anti-pattern
1. Use the structure of existing anti-patterns in `anti-patterns.md`:
   - What I did
   - What happened
   - Why it failed
   - What I do instead
2. Submit PR adding it to `anti-patterns.md`

### New layer / pattern document
Open an issue first to discuss whether it fits the four-layer structure or needs its own file.

## Style guide

- **Concrete over abstract.** Specific failure modes beat general principles
- **Time-tested.** If you've used a pattern less than a month, it's not yet ready for this repo
- **Failure-honest.** Anti-patterns are the highest-value section. Be willing to share what didn't work

## License

Contributions are released under [CC BY-SA 4.0](./LICENSE) — attribution required, share-alike applies.
