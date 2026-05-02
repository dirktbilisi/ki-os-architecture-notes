# Architecture — Four Layers

The system that holds up over time has four layers. Each layer answers a different question. Mixing them — which is the natural drift — produces the chaos that kills personal AI workflows.

## The four layers

```
┌─────────────────────────────────────────────────────────┐
│                      WORKFLOWS                          │
│  Time-bound, repeatable cycles (start / sync / shutdown)│
├─────────────────────────────────────────────────────────┤
│                       SKILLS                            │
│  Discrete capabilities triggered by command or context  │
├─────────────────────────────────────────────────────────┤
│                       MEMORY                            │
│  Hot / episodic / semantic — what the model recalls     │
├─────────────────────────────────────────────────────────┤
│                        STATE                            │
│  Truth at a point in time — kernel state, session, goals│
└─────────────────────────────────────────────────────────┘
```

Each layer is **lower than** the layer above. State changes most often. Workflows change least.

## Layer 1 — State

The thinnest layer. State is what is true *right now*:

- Current session ID and date
- Active goals (today, this week, this month)
- Open actions
- Background jobs in flight
- Last checkpoint timestamp

State files are small (≤2KB each), JSON or YAML, structured. They are the **truth-source** that the rest of the system reads from.

### Key principle: state files have one writer

Each state file is owned by exactly one process or workflow. Multiple writers create race conditions, contradictions, and the slow rotting of "which file is the real truth".

In practice this means: a session-runtime tool writes session state, a goals tool writes goals state, and so on. The model **reads** all of them but **writes** to only one at a time, mediated by a tool.

### What goes in state vs. memory

A common confusion. State is "what is true now". Memory is "what was true at points in the past, persisted for recall".

Example:
- State: "Active goals today: goal-01, goal-02"
- Memory: "Last week's session focused on launching the new editor — see episodic record 2026-04-27"

If you put memory-content in state files, state files balloon. If you put state-content in memory, you can't query "what is active right now" without scanning the entire history.

## Layer 2 — Memory

The reservoir. Memory is everything-that-was-and-may-be-relevant-again.

Three sub-layers:

### Hot memory
- "Active context" — the verdichtung from the last session, the next steps
- "Runtime snapshot" — small, frequently updated, always loaded at session start
- Maximum a few KB total

### Episodic memory
- Session summaries: each session writes one paragraph at shutdown
- Recent episodes (last 30 days) in detail
- Older episodes archived as references but not loaded by default

### Semantic memory
- Stable facts about the operator (role, preferences, expertise)
- Project-specific knowledge that has solidified into "always true" status
- Loaded selectively when the topic is relevant

The naming convention — hot / episodic / semantic — borrows loosely from cognitive science but is operational, not theoretical. Hot is what you're using right now. Episodic is what happened. Semantic is what's stable.

## Layer 3 — Skills

Discrete, named capabilities. Triggered by slash command (`/research`, `/textcheck`, `/doku`) or by phrase pattern matching.

Skills sit *between* memory and workflows. They consume memory (read context) and produce output that may persist back into memory or files.

A skill is well-defined when:
- It has a clear input pattern
- It has a defined output path
- It has explicit boundaries (what it does NOT do)
- It can be invoked twice without re-reading its own documentation

If any of these is missing, it's not a skill — it's a recurring chat prompt with a name.

(See companion repo [claude-code-skills-templates](https://github.com/dirktbilisi/claude-code-skills-templates) for skill conventions in detail.)

## Layer 4 — Workflows

Time-bound cycles. Three core ones:

- **Start** — beginning of session: load state, hot memory, recent episodes; render dashboard; identify the next concrete step
- **Sync** — mid-session checkpoint: save what's been done so the next session can resume cleanly
- **Shutdown** — end of session: write the verdichtung, update goals, archive episode

Workflows are not skills. They are larger orchestrations that *use* skills, read state, update memory, and follow a defined sequence.

A workflow has phases. Each phase has a clear input → output transformation. Workflows can be paused and resumed; skills cannot.

## Why this layering matters

Without layering, every interaction with the system is a fresh negotiation about what's true and what to do. With layering, each layer answers its own question:

- "What is true right now?" → State
- "What was true at points in the past?" → Memory
- "What discrete things can I do?" → Skills
- "What is the rhythm of using this?" → Workflows

When you violate the layering — putting workflows in skills, putting state in memory, mixing layers in the same file — the system gets harder to reason about. Within weeks, you can't tell where to look for a piece of information. Within months, you start avoiding the system because using it feels like archaeology.

## What I tried that didn't work

### Single big "context" file
First attempt: one big `context.md` file with everything — goals, recent work, project status, preferences. Within a month it was 40 pages and the model kept skipping the important sections.

**Lesson:** specialization beats consolidation. Many small files in defined paths beats one big mega-file.

### No workflow layer
Second attempt: skills only, no workflow layer. Each session started from scratch with whatever context I happened to load.

**Lesson:** without start/sync/shutdown discipline, sessions drift. The workflow layer is what makes a session feel coherent rather than a collection of unrelated chats.

### State and memory in the same file
Third attempt: a "current state" file that included historical context. It got rewritten on every session and lost old information.

**Lesson:** state and memory must be physically separate. State is overwritten; memory accumulates. Mixing them means one of these properties wins, and you lose the other.

## What I would change if starting over

Two things:

1. **Add a meta-layer for tool documentation.** When the system has 25+ skills, knowing which skill exists and what it does becomes its own knowledge problem. A skill registry that the model can query is worth the setup cost.

2. **Build the synchronization workflows before the skills.** Start / sync / shutdown should be the first thing built, not an afterthought added when the system has gotten messy. They define the rhythm everything else fits into.
