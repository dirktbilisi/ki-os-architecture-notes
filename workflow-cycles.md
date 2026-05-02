# Workflow Cycles — Start, Sync, Shutdown

Time-discipline matters more in LLM-augmented workflows than in conventional ones. Here is why, and the three cycles that handle it.

## Why time-discipline matters

LLMs have no inherent sense of "the current session". Every interaction is, technically, fresh. The session-feeling is something you construct through:

- Loading the right context at start
- Saving state at well-defined points
- Closing cleanly so the next session can resume

Without these, every session devolves into a fresh negotiation about what's true. With them, sessions feel coherent — they pick up where the previous one ended, they know what's been done, they know what's next.

## The three cycles

```
SESSION START
   ↓
   /start  →  load tier-0 + tier-1 context, render dashboard, identify next step
   ↓
[work happens]
   ↓
   /sync  →  checkpoint: save what's been done, update hot memory, no session-end
   ↓
[more work happens, possibly more /sync checkpoints]
   ↓
   /shutdown  →  write verdichtung, update goals, archive episode, close session
   ↓
SESSION END
```

These three are the operational backbone. Everything else (skills, edits, generations) happens between /start and /shutdown.

## /start — opening the session

### What it does

1. Read system clock for current date
2. Check `state/session/current.json` — is there an active session?
   - If yes (`status = active`): treat as resume, keep existing session-id and date
   - If no: create new session-id, set new date, status active
3. Load Tier 0 (state files) and Tier 1 (hot memory + active goals)
4. Build a compact dashboard:
   - Date and session status
   - Goal traffic-lights (month / week / day)
   - Top 3 for today
   - Critical blockers
   - Active project focus
5. Update `current.json` with start timestamp and last_workflow="start"
6. Surface the next concrete action

### What it does NOT do

- Does not load Tier 2 / Tier 3 unless asked
- Does not regenerate goals (just reads them)
- Does not assume what the user wants to do next — it surfaces options

### Hard rules

- Date is set ONCE at session start. No "what time is it" calls during the session — date comes from `current.json`.
- If goals files are empty or missing, name the gap explicitly. Don't hallucinate goal content.

### Why this matters

Without /start, a session begins with the model knowing nothing. The first 5 minutes get spent re-establishing context that should have been ambient.

## /sync — mid-session checkpoint

### What it does

1. Read current session-id from `state/session/current.json`
2. Take a one-paragraph snapshot of what's been done since last sync (or session start)
3. Append to `state/session/checkpoints.json` with timestamp and summary
4. Update `memory/hot/runtime-snapshot.md` with the summary

### When to use

- Automatic (cron-triggered): every 10 minutes during active sessions
- Manual: after completing a major piece of work
- Before any anticipated context switch (lunch, meeting, day-end if not shutting down)

### What it does NOT do

- Does not end the session
- Does not write daily logs
- Does not modify goals
- Does not promote anything to episodic memory (that's shutdown's job)

### Why this matters

If the system crashes mid-session, /sync ensures recovery is possible. Without /sync, an unexpected context-loss means starting over from the last shutdown — which could be hours ago.

Also: /sync is a mental hygiene practice. The act of summarizing what's been done in one paragraph forces clarity that doesn't happen otherwise.

## /shutdown — closing the session

### What it does

1. Read current session state
2. Generate verdichtung (3-5 sentence summary of the session)
3. Write to `memory/episodic/session-summaries.md` with date, session-id, summary, next-step
4. Update `memory/hot/active-context.md` to reflect the verdichtung
5. Update goals (status changes if any goals were completed)
6. Append daily-log entry if today's daily log exists (or create it)
7. Set session status to "shutdown" in `current.json`
8. Render closing summary to user: "Session closed. Next start will resume from: ..."

### What it does NOT do

- Does not delete or archive anything (cold-archive is a separate weekly process)
- Does not generate reports beyond the verdichtung
- Does not assume tomorrow's session — only sets up the resume hook

### Hard rules

- Date for shutdown comes from `current.json`, not system clock. Even if shutdown happens at 02:00 the next morning, it goes into the original session's date.
- The verdichtung is written every shutdown, even short ones. No exceptions.
- If the user explicitly skips shutdown (e.g. closes laptop without /shutdown), the next /start treats the previous session as still-active and resumes — but the missing verdichtung is a gap that won't be filled.

### Why this matters

Without /shutdown, sessions don't actually end — they just trail off. The next /start has nothing to anchor to. Episodic memory has gaps. After a few weeks of skipped shutdowns, the system loses its memory of what happened.

The 60 seconds spent on shutdown is the smallest, highest-leverage discipline in the whole system.

## Optional: /weekly-review

A larger cycle that runs once a week, separately from individual sessions:

1. Read all session-summaries from the last 7 days
2. Look for recurring themes (questions, blockers, patterns)
3. Identify candidates for promotion: episodic facts that should become semantic
4. Identify candidates for demotion: stale semantic facts that should be archived
5. Update goals for next week
6. Render a weekly summary

The weekly review is the maintenance window for memory layers. Without it, semantic memory accumulates contradictions and episodic memory becomes harder to navigate.

## Anti-patterns

### Anti-pattern 1: skipping /shutdown
"It's late, I'll just close the laptop." Result: tomorrow morning's session starts with no verdichtung from yesterday. Loss compounds: by week's end, half the sessions have no proper handoff.

### Anti-pattern 2: running /start multiple times in one session
The system should detect this (`status = active`) and treat it as a resume. But sometimes operators force a fresh /start, which can create date inconsistencies.

### Anti-pattern 3: writing fake verdichtungs
"Worked on stuff today." This is worse than no verdichtung — it occupies the slot of a real summary without providing the value. Either write a real verdichtung or skip the shutdown (and accept the gap).

### Anti-pattern 4: treating /sync as /shutdown
/sync is a checkpoint, not an ending. It doesn't write to episodic memory. A session that uses /sync but never /shutdown has lots of intermediate snapshots but no episode record.

### Anti-pattern 5: shutting down when paused, not when done
If the user is going to lunch, /sync is the right call, not /shutdown. /shutdown should mark a real end of work.

## Why these three cycles are non-negotiable

You can build an LLM-augmented workflow without start/sync/shutdown discipline. It will work for a few weeks. It will start to feel ad-hoc by month two. By month four, you'll abandon it.

The cycles are the spine of the system. Skills, memory, state — all of these depend on the cycles to function. Without the cycles, those layers exist but aren't used coherently.

Time-discipline is the difference between a tool you build and a system you live with.
