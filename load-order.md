# Load Order — Tier-based Context Loading

How much context to load at session start, in what order. The single most important decision in long-running AI workflows.

## The core principle

Don't load the whole vault. Don't load randomly. Load in **tiers**, smallest first, with each tier earning its place.

```
Tier 0 (always loaded)
   ↓
Tier 1 (loaded by default)
   ↓
Tier 2 (loaded for specific contexts)
   ↓
Tier 3 (loaded only on explicit request)
```

The point of tiering is **token economics + cognitive economics**. Loading too much costs tokens (real money) AND dilutes attention (the model has more to ignore).

## Tier 0 — Runtime anchors

Always loaded at session start. These are tiny files (≤1KB each typically) that establish the system's current state.

Typical contents:
- Session ID, current date, last workflow
- Active goals (today, this week)
- Open actions
- Last 3-5 events
- Kernel state (active / paused / shutdown)
- Health indicators

**Total Tier 0 size: ~5-10KB.**

The model needs this to know *what time it is* in the system. Without Tier 0, every session starts as if it's the first session.

### Files in this tier

```
state/session/current.json
state/kernel.json
state/orchestration.json
state/goals.json
state/events.json
state/actions.json
```

## Tier 1 — Compact system context

Loaded by default at session start. Slightly larger (~10-30KB total), gives the model the operational context for "today".

Typical contents:
- Active context (last session's verdichtung, next steps)
- Runtime snapshot
- Daily/weekly/monthly goals (full text, not just IDs)
- Active projects index

### Files in this tier

```
memory/hot/active-context.md
memory/hot/runtime-snapshot.md
goals/daily-goals.json
goals/weekly-goals.json
goals/monthly-goals.json
projects/active-projects.json
```

After loading Tier 0 + Tier 1, the model has a complete picture of: what's true, what's going on, what's next.

## Tier 2 — Persistent profile context

Only loaded when relevant. These files describe the operator's identity, voice, brand — things that don't change session-to-session but matter for specific kinds of work.

### Files in this tier

```
identity/operator-model.md
brand/brand-voice.md
brand/brand-visual.md
context/personal.md
context/business.md
skills/registry.json
```

**When to load:**
- Brand voice files: when generating external-facing text
- Identity files: when the topic is about the operator themselves
- Skill registry: when triaging what skill applies

**When NOT to load:**
- Routine technical work
- Pure operational tasks (sync, shutdown)
- Code-heavy sessions

## Tier 3 — Operational detail

Selectively loaded for specific tasks. The full archive — daily logs, individual project files, framework references, decision logs.

### Files in this tier

```
memory/episodic/recent-episodes.json (last 30 days)
memory/semantic/semantic-memory.md
memory/cold/archive-index.json
projects/<specific-project>/...
daily/<specific-date>.md
decisions/<specific-decision>.md
resources/frameworks/<specific-framework>.md
```

**When to load:**
- Working on a specific project: load that project's files
- Following up on a past decision: load that decision file
- Investigating a question: load the relevant framework

**Not loaded by default. Ever.** The cost of loading Tier 3 by default is enormous — both in tokens and in noise.

## How tiering plays out in practice

### Session start (typical)
- Tier 0 + Tier 1 loaded automatically by the start workflow
- Total context loaded: ~25KB
- Model has full operational picture

### When user asks a project-specific question
- Tier 2 / Tier 3 files for that project loaded selectively
- Loaded incremental: file by file, not in batches
- Model now has project context on top of operational picture

### When user wants to write external content
- Tier 2 brand-voice file loaded
- Model now has voice constraints on top of operational picture

### Anti-pattern: always load Tier 2-3
The temptation is to load everything "just in case". Result:
- Token costs explode
- Model attention dilutes
- Important Tier 0 / Tier 1 details get ignored in favor of older, less-relevant Tier 3 content

## Rules for an efficient session

1. **Never wholesale-scan project / daily / resource directories.** Use indexes, not file listings.
2. **At session start, load only condensed indices.** Full-text loading happens on demand.
3. **Don't treat any single file as the only source of truth** for current state — orchestration of multiple state files gives a more reliable picture.
4. **Daily logs are readable history, not session anchors.** The session anchor is `current.json`, not yesterday's daily log.
5. **When new insights emerge, ask three questions before persisting:**
   - Is this relevant only to the current session? → don't persist
   - Will the next session need this? → persist to Tier 1 (hot memory)
   - Is this a stable semantic fact? → persist to Tier 2/3 (semantic memory)

## Date discipline

A subtle but important rule: **the session date is set once at session start and does not change.**

If a session opens at 23:50 and runs past midnight, the session date stays at the original date. Otherwise: a `/sync` at 00:15 might write to the wrong file, and a `/shutdown` at 00:30 might land in tomorrow's daily log instead of today's.

Implementation: read date from `state/session/current.json`, never from system clock during the session.

## The math behind tiering

For a system with ~3MB total content:

- Loading everything at session start: ~750k tokens, ~$2.25/session at current Claude rates
- Loading Tier 0 + Tier 1 only: ~6k tokens, ~$0.02/session
- Loading Tier 0 + Tier 1 + occasional Tier 2/3: ~25k tokens average, ~$0.07/session

Difference: ~30x cheaper per session, with no loss of working capability — because Tier 2 and 3 are loaded *when needed*, not preemptively.

Over a year of daily sessions: $700+ in token savings, plus the cognitive benefit of focused context.
