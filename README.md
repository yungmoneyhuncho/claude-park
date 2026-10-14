# park

Stop paying full price to come back to a long Claude Code session.

## Why this exists

I work one session per project, often for 8+ hours straight. One night I left two of those sessions open overnight. In the morning I typed one message into each. Those two messages used up my entire 5-hour limit. 100%, nothing left, before I'd done any work.

Here's why. Claude caches your session's history for about an hour. Walk away longer than that (overnight, lunch, a meeting, your laptop dies) and your next message re-reads the entire history at full price. On a 1M-context model like Opus or Sonnet, one "morning, continue" can cost more than a whole day of normal prompts.

## What park does

Works in the Claude desktop app and the CLI:

- **Warns at 50 min idle**: "cache expires in ~10 min, reply now or let park save it".
- **Auto-parks at 56 min**: compacts the session *while the cache is still warm* (the cheapest moment a compaction can ever run) into a full handoff, and saves it to a state folder in your project.
- **Saves on close**: closing the app or terminal writes a snapshot (local files, no model call).
- **Picks up automatically**: open a new session in that project and type anything. park attaches the parked state to your first message. You don't have to go back into five old sessions and tell each one to wrap up.
- **Guards cold returns**: if you do come back to an expired, unparked session, park holds your first message once and tells you what it would cost. Send it again to go ahead anyway.
- **Tracks savings**: a toast on every resume, `/park stats` for the lifetime total.

## The state folder

Every project gets a plain-markdown record of its sessions (gitignored automatically):

```
.claude/park/
  CONTEXT.md        how to read this folder
  INDEX.md          every session, newest first (generated)
  sessions/<date>_<id>/
    CONTEXT.md      goal, compacted summary, where it stopped, next step
    00_summary.md   the full compaction handoff
    01_asks.md      every user message
    02_replies.md   recent assistant replies
    03_files.md     files edited and read
    04_commands.md  shell commands run
```

Small files first, detail behind them, so an agent reads a few hundred tokens and only goes deeper when it needs to. Every session is told the folder exists, so one session can read what another did without you repeating it. The layout follows Jake Van Clief's Interpretable Context Methodology: the folder structure is the state.

## What you'll see

Reopen a session you left overnight and type anything:

```
Prompt dropped by a hook: Cache expired (idle 540 min). Sending here re-sends ~640k tokens at full price.
Cheapest: start a NEW session in this project; park loads this session's state into it automatically.
Or send your message again to continue here anyway.
```

Nothing was sent, so that cost 0 tokens. Open a new session in the same project, type "continue", and it starts with the parked state already loaded.

## Tip: one feature per session

The best fix is smaller sessions. Ask Claude to tell you when a session starts drifting into a second feature, and open a new session in the same project for it. park's state folder lets those sessions read each other, so splitting up work doesn't lose context.

## Commands

- `/park`: compact and save now (do it before you walk away).
- `/park stats`: resumes, tokens not re-sent, snapshots.

## Install

```bash
claude plugin marketplace add yungmoneyhuncho/claude-park
```

```bash
claude plugin install park@park
```

Then restart Claude Code or the desktop app.

## Safety

- park only auto-loads snapshots written on your own machine. A `.claude/park` folder that ships inside a cloned repo is ignored.
- Loaded snapshots are handed to Claude as notes about past work, not as instructions.
- park refuses to write if `.claude` or `.claude/park` is a symlink.
- The state folder is gitignored automatically.

## Honest limits

- Auto-park needs the app running. If the machine crashes or loses power, there's nothing left to compact; the latest per-turn snapshot is what a new session picks up.
- The 60 min cache window is the long-context default. If your plan or overage drops it to 5 min, the timers fire too late; run `/park` yourself before you leave.
- Compaction is not free. It's just cheapest while the cache is warm. Skipping it and coming back cold costs more.
- Savings are estimates: the parked session's context size minus the size of the handoff that replaced it.
- Timings live at the top of `hooks/register.ts`.

MIT
