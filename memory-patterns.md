# Memory Patterns

Three layers of memory — hot, episodic, semantic. What each is for, what goes where, what happens when the layers blur.

## Hot memory

**Purpose:** what the model needs in the next ~5 minutes of conversation.

**Contents:**
- `active-context.md` — the verdichtung from the last session, plus the one or two next steps
- `runtime-snapshot.md` — current kernel state, session ID, last workflow, last update timestamp

**Size:** ≤2KB total. Both files combined.

**Update frequency:** every session shutdown (active-context.md) and every sync checkpoint (runtime-snapshot.md).

**When loaded:** at every session start, automatically.

**Format:** Markdown, terse. Bullet points. No prose paragraphs.

### Anti-pattern in hot memory: storage drift

The temptation is to add "just one more thing" to active-context.md — a recent decision, a useful fact, a pending question. Within weeks, hot memory is 30KB and the actual hot content is buried.

**Discipline:** active-context.md is **summary** not **archive**. If a fact deserves to live longer than 24-48 hours, it goes to episodic or semantic memory, not into hot.

## Episodic memory

**Purpose:** what happened, in chronological sequence, recallable for context.

**Contents:**
- `session-summaries.md` — one paragraph per session, append-only. Includes: session ID, date, summary, next step.
- `recent-episodes.json` — structured version of the last ~30 days of sessions
- Daily logs in `daily/YYYY-MM-DD.md` — full text of significant work

**Size:** grows linearly. Recent (30 days) loaded selectively; older archived.

**Update frequency:** at every session shutdown.

**When loaded:** on demand. Not loaded by default at session start.

### Pattern: the verdichtung

Each session ends with a ~3-5 sentence verdichtung (compaction). Not a transcript. Not a feature list. The verdichtung answers: "If I read this in 6 months, what would I remember about this session?"

Examples of good verdichtungs:
- "Built the editor's snooze feature plus three smaller fixes. Discovered a race condition in the polling code — fixed it but the underlying bug deserves a refactor next sprint. User signed off on the new keyboard shortcuts."
- "Three-hour design conversation with [stakeholder]. Outcome: we abandon the multi-tenant model in favor of single-tenant per customer. This changes the whole infrastructure plan — see project file. Next: rewrite the technical doc."

Examples of bad verdichtungs:
- "Worked on the editor today. Made progress." (no information)
- "Did 14 things across the codebase: changed file A, refactored function B, fixed bug in C, added test D, ..." (transcript, not summary)

The verdichtung is the working unit of episodic memory. If it's bad, the memory is useless even though it's stored.

### Anti-pattern: not writing verdichtungs

The temptation is to skip the shutdown step "just this once". After a few weeks of skipped verdichtungs, episodic memory has gaps and the system feels less reliable. Every session needs a verdichtung — even short ones.

**Discipline:** if a session is too short for a verdichtung (e.g. quick lookup), it doesn't need a session-shutdown — but if you ran a workflow, you write the verdichtung.

## Semantic memory

**Purpose:** stable facts that don't change session-to-session.

**Contents:**
- Operator profile (role, expertise, preferences)
- Stable project facts (what's already decided, what's already built)
- Domain knowledge that has solidified
- Anti-knowledge (things explicitly NOT to do, with reasons)

**Format:** Markdown, organized topically. Each file is one topic.

**Size:** grows slowly. Updates are deliberate.

**When loaded:** selectively. Brand voice when generating content; identity when topic is personal; project semantics when working on that project.

### Pattern: pointer files

Semantic memory grows over time. To prevent it becoming an archaeological dig:

- `MEMORY.md` (or equivalent index file) — short pointers to where specific knowledge lives
- Maximum ~150 chars per pointer
- Format: `- [Topic title](file.md) — one-line hook`

The index is loaded by default. The actual files are loaded on demand, based on the index.

### Anti-pattern: writing tutorials in semantic memory

The temptation is to write long explanations in semantic memory files — turning them into mini-tutorials for future you.

**Discipline:** semantic memory captures **facts and decisions**, not explanations. The fact: "We use JWT tokens for auth, not sessions, decided 2026-03-10 because of multi-device support." Not: a 2000-word essay on JWT tokens.

If you find yourself writing tutorials in semantic memory, the right place for them is probably a project README or a docs/ directory — not memory.

## Cross-layer rules

### Rule 1 — Each fact has one home

A piece of information belongs to exactly one layer. If "the editor uses JWT auth" lives in both hot memory and semantic memory, eventually they will diverge (one gets updated, the other doesn't). Same fact in two places = at least one of them is wrong.

### Rule 2 — Promotion and demotion

Information moves between layers over time:

- Hot → episodic: at every shutdown, hot memory's content gets summarized into the episodic record
- Episodic → semantic: if a pattern recurs across sessions, it earns a place in semantic memory
- Semantic → archive: if a stable fact becomes outdated, it's archived (not deleted) with a date stamp

The promotion happens at workflow boundaries (shutdown, weekly review), not opportunistically.

### Rule 3 — Cold storage is one-way

Once a fact is cold-archived, it doesn't get loaded automatically. If you need to bring it back, you read it explicitly and consider whether to re-promote it.

This sounds bureaucratic. It is. But without it, "old but maybe still relevant" information slowly accumulates in working memory and dilutes the new stuff.

## Memory failure modes

### Failure 1: hot memory becomes a junk drawer
Symptom: active-context.md is 50 lines long, half of it irrelevant to the current session.
Fix: at every shutdown, ruthlessly cut hot memory back to the next-step + verdichtung essentials.

### Failure 2: episodic memory has gaps
Symptom: trying to remember what happened two weeks ago, no record exists for several days.
Fix: every session that ran a workflow gets a verdichtung. No exceptions for "small" sessions.

### Failure 3: semantic memory contradicts itself
Symptom: two semantic files claim contradictory things about the same topic.
Fix: weekly review checks for contradictions. Contradiction = decide which is current, archive the other.

### Failure 4: nothing in memory matches what's actually true
Symptom: model insists on facts that are no longer current. State file says X, semantic file says Y, they disagree.
Fix: the rule is "trust state over memory". Memory is a snapshot of past truth, state is current truth. When they conflict, state wins, memory gets updated.

## Why the layers are worth the discipline

Without layered memory, every session starts from zero. With layered memory:

- Hot memory means each session starts knowing "what we were just doing"
- Episodic memory means questions like "when did we decide X?" have answers
- Semantic memory means stable facts don't need re-discovery every time

The discipline cost is real — maybe 10 minutes per session at shutdown. The payoff is a system that compounds in usefulness over months instead of degrading into a chat history nobody can navigate.
