# Anti-Patterns

What didn't work. Things I tried, lived with for a while, then stopped doing — with the reasons attached.

This file is the most useful one in the repo, in the same way that "things to not do" guides usually are. Most architectural advice describes what to do; the harder knowledge is what to avoid.

---

## Anti-pattern 1 — One big context file

**What I did:** Created a single `CONTEXT.md` (~5KB initially) and instructed the model to load it at session start. It contained current goals, recent work, project status, brand voice, preferences.

**What happened:** Within 2 weeks the file was 25KB. Model started skipping sections, mixing details from different topics, and producing inconsistent output depending on which part of CONTEXT.md it had "in mind".

**Why it failed:** When everything is in one file, the model can't selectively load. It either loads all of it (token-expensive, attention-diluting) or skims it (and then misses the load-bearing parts).

**What I do instead:** Many small files in defined paths. Index file (MEMORY.md) with one-line pointers. Selective loading via tier system. (See [load-order.md](./load-order.md).)

---

## Anti-pattern 2 — No workflow layer

**What I did:** Built a collection of skills (research, doku, textcheck) and assumed they would compose naturally. No /start, no /sync, no /shutdown.

**What happened:** Each session began with manual context-rebuilding. "What was I doing yesterday?" — no answer. "Are we still on track for the launch?" — no idea. After a few weeks, I started avoiding the system because using it felt like archaeology.

**Why it failed:** Skills are atomic tools. Without a containing rhythm, they don't produce a coherent workflow — they produce a series of unrelated chats.

**What I do instead:** Three-cycle discipline (start / sync / shutdown). The skills sit *inside* the workflow, not parallel to it. (See [workflow-cycles.md](./workflow-cycles.md).)

---

## Anti-pattern 3 — Mixing state and memory in the same file

**What I did:** A single `current-state.md` file that contained both "what's true now" (active goals, current session) and "what was true recently" (last 5 episodes, recent decisions).

**What happened:** Every session-shutdown rewrote the file. Recent history got lost because the rewrite focused on current state. Within a month, "recent episodes" was always empty.

**Why it failed:** State and memory have opposite update patterns. State gets overwritten on changes. Memory accumulates over time. When the same file serves both, one of these properties wins, and the other is lost.

**What I do instead:** Physical separation. State files (small, JSON, single-writer, overwritten on update). Memory files (Markdown, append-only or selectively-edited, persistent).

---

## Anti-pattern 4 — Loading Tier 3 by default

**What I did:** "Just in case", I loaded all project files at session start. Reasoning: model should have full context.

**What happened:** Token costs went 30x higher than necessary. Model attention diluted across irrelevant files. Important Tier 0 / Tier 1 details got ignored in favor of older Tier 3 content.

**Why it failed:** Pre-emptive loading violates the principle that context should *earn* its place in the session. Loading everything is the same as loading nothing in terms of attention.

**What I do instead:** Tier 0 + Tier 1 by default. Tier 2/3 on demand. (See [load-order.md](./load-order.md).)

---

## Anti-pattern 5 — Fake verdichtungs

**What I did:** When tired or rushed at session-end, wrote shutdown verdichtungs like "worked on the editor today" or "made progress on multiple items".

**What happened:** Six weeks later, those verdichtungs were useless. I couldn't remember what was actually done. Episodic memory had data but no information.

**Why it failed:** A verdichtung that doesn't answer "what would I remember about this in 6 months" provides no value. Worse, it occupies the slot where a real summary would have lived — so the gap isn't even visible.

**What I do instead:** Either write a real verdichtung (3-5 sentences, specific) or skip the shutdown entirely. The skipped shutdown is a visible gap; the fake verdichtung is invisible decay.

---

## Anti-pattern 6 — Ad-hoc skill creation

**What I did:** Whenever a workflow felt repeatable, I created a new skill. Within a few months, I had 30+ skills, many overlapping.

**What happened:** Skills with unclear boundaries produced inconsistent output. Two research skills with slightly different focus areas confused even me. Some skills got used once and forgotten — but stayed in the registry, cluttering it.

**Why it failed:** A skill needs explicit conventions (naming, output paths, brand-voice handling, conflict resolution). Without conventions, skill collections rot.

**What I do instead:** Four conventions for every skill (see [claude-code-skills-templates](https://github.com/dirktbilisi/claude-code-skills-templates)). Quarterly review of skill registry. Explicit SKIP documentation for skills considered but not built.

---

## Anti-pattern 7 — Brand voice as adjectives

**What I did:** Brand voice file said: "professional yet approachable, confident yet humble, informative yet engaging".

**What happened:** Every text the model produced sounded like every other consultant on the internet. The voice was vague enough that the model averaged it to generic.

**Why it failed:** Adjectives without contrast give the model nothing to push against. "Confident" describes 80% of brand voices; the differentiation lives in *what's specifically excluded*.

**What I do instead:** Brand voice as constraint blocks: positive markers + negative markers + forbidden vocabulary + structural rules + one example. (See [brand-voice-prompting](https://github.com/dirktbilisi/brand-voice-prompting).)

---

## Anti-pattern 8 — Manual orchestration of every step

**What I did:** At session start, manually loaded files, manually checked state, manually decided what to do next.

**What happened:** Every session started with 5-10 minutes of overhead before any real work. The friction made me skip sessions or shorten them.

**Why it failed:** Orchestration logic should be in tools (workflows, scripts) — not in the operator's head. If you're the orchestrator, you become the bottleneck.

**What I do instead:** /start workflow runs the orchestration. Tool reads state, loads tier-1, renders dashboard, identifies next step. Operator decides what to *do*, not what to *load*.

---

## Anti-pattern 9 — No date discipline

**What I did:** Used "today's date" liberally during sessions. Sometimes pulled it from system clock, sometimes from the user message, sometimes hard-coded.

**What happened:** Sessions spanning midnight wrote files in inconsistent dates. Daily logs had gaps. Sync checkpoints landed in the wrong day's record.

**Why it failed:** "Today" is ambiguous in long-running sessions. Without a single source-of-truth date for the session, every workflow that uses the date can produce a slightly different answer.

**What I do instead:** Session date set ONCE at /start, stored in `state/session/current.json`. All workflows read from there. System clock is consulted only at /start to set the date. After that, the session date is frozen.

---

## Anti-pattern 10 — Treating LLMs like search engines

**What I did:** "Find me X in my notes." Hoped the LLM would search the vault and surface the answer.

**What happened:** LLMs are pattern matchers, not search engines. They confidently produced plausible-but-wrong answers when the actual content wasn't in their loaded context.

**Why it failed:** LLMs work with what's in context. If the answer isn't loaded, they don't know they don't know — they generate.

**What I do instead:** Use actual search tools (grep, ripgrep) for "find X in files". Use LLMs for "synthesize, summarize, transform what's already loaded". Don't conflate the two.

---

## Anti-pattern 11 — Optimizing too early

**What I did:** Designed elaborate state machines and memory hierarchies before knowing how I'd actually use the system.

**What happened:** Most of the elaboration didn't survive contact with daily use. I rebuilt half of it within 3 months.

**Why it failed:** Architecture should be load-bearing for actual use cases, not for imagined ones. Building for hypothetical complexity creates real complexity that may serve nothing.

**What I do instead:** Start with the smallest viable system. Add structure when actual friction emerges, not in anticipation of friction. Most "premature complexity" gets removed within 6 months — the things that survive are the things that solved real problems.

---

## Anti-pattern 12 — No archive discipline

**What I did:** Never archived old content. Reasoning: it's all searchable, why archive.

**What happened:** Session-start context loading included references to projects from 8 months ago that were no longer relevant. Memory layers accumulated stale facts. Quarterly cleanup got harder each quarter.

**Why it failed:** Information has a half-life. Treating year-old context as equally relevant as last week's context produces noise that drowns out signal.

**What I do instead:** Quarterly archive sweep. Anything not touched in 90 days gets moved to cold storage (still readable, but not loaded by default). The decision "is this still load-bearing" gets made deliberately, not by default-of-keeping-everything.

---

## The meta-pattern

Most of these anti-patterns share a common root: **mixing things that should be separated**.

- State and memory in the same file (anti-pattern 3)
- Hot and semantic memory in one place (anti-pattern 1)
- Tier 0/1/2/3 loaded together (anti-pattern 4)
- Workflow logic in the operator's head (anti-pattern 8)
- Pattern matching used as search (anti-pattern 10)

The discipline that fixes most of these is the same discipline: **separate concerns explicitly, even when they feel related**.

It costs effort up front. It pays back in months of usability.
