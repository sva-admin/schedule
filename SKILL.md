---
name: schedule
description: Turn a recurring chore into a durable Claude routine that runs on a cron schedule and proves it ran. Picks the right surface (persistent scheduled task vs session-only cron), writes a prompt that survives having zero memory of this conversation, dry runs it, then arms it. Trigger on "schedule this", "every morning", "run this daily", "every Monday", "set up a routine", "automate this weekly", "make this recurring", "remind me every", "cron job for Claude", "wake up and do X", or when the user describes work they will otherwise redo by hand on a fixed rhythm.
---

# schedule

Claude Code can already run work on a clock. What it will not do for you is decide what deserves a clock, write a prompt that still makes sense to a cold session at 7am, or make the routine shout when it breaks. That is this skill.

The failure mode you are designing against is not a crash. It is a routine that quietly stops producing output while everyone assumes it is fine.

## Inputs

- The chore, in the user's own words
- The rhythm it should run on
- Where the output has to land: a file path, a note, a message, the chat

If the user gave you the chore but not the destination, ask. A routine with no defined output has no way to be verified, and an unverifiable routine is the one that dies silently.

## Step 1: decide whether it belongs on a clock at all

Schedule it only when all three hold:

1. **It repeats on a rhythm, not on a trigger.** "Every weekday at 7am" is a schedule. "Whenever a PR opens" is not: that is a hook or a watcher, and cron will do it badly.
2. **The output is worth reading even when nothing changed.** A briefing that says "quiet day" still earns its slot. A refresh that only matters when something moved is a poll, and polls belong in a watcher that streams changes instead of a cron that guesses.
3. **A late run is harmless.** If firing a day late causes damage (posting publicly, moving money, deleting records), it does not run unattended. Schedule the draft, let a human send it.

If the request fails any of these, say so plainly and offer the version that actually fits. Talking someone out of a bad routine is a better outcome than building it, because a routine nobody trusts trains them to ignore the ones that work.

## Step 2: pick the surface

Two different mechanisms answer to the word "schedule". Choosing wrong is the most common mistake here.

**Persistent scheduled task (the default).** Stored on disk as its own `SKILL.md` under `~/.claude/scheduled-tasks/<task-id>/`, armed with a cron expression or a one-time timestamp. It survives restarts. It runs while the app is open, and if the app was closed when the task came due it runs on next launch. Every real routine, the morning briefing, the weekly review, the monthly cleanup, goes here.

**Session cron.** In-memory, tied to the session you are sitting in. It disappears when the session ends and recurring jobs expire after seven days. Use it for exactly one thing: "for the next while, in this session, check X every few minutes." Never use it for anything the user expects to still exist tomorrow.

Say which one you picked and why, in one line. Users conflate the two and then wonder where their routine went.

## Step 3: write the prompt for a stranger

Each run starts cold. It has no memory of this conversation, no idea which file you were both looking at, no sense of what "the usual format" means. Write the prompt as if handing it to a competent colleague who has never met you.

The self containment checklist:

- **Absolute paths only.** Never "the file we were just editing", never a relative path that assumes a working directory.
- **Name every tool and connector explicitly.** "Use the Calendar connector to list today's events", not "check my calendar".
- **Compute the date at run time.** Write "use today's date in YYYY-MM-DD" and never bake in an actual date. A hardcoded date is a routine that produces the same stale file forever.
- **Put the output format in the prompt.** Sections, frontmatter, length, tone. If it lives only in this conversation, it is gone by the first run.
- **Carry over the preferences the user expressed here.** They said keep it short because they read it on their phone: that sentence goes in the prompt verbatim.
- **One job per routine.** A routine that briefs you and cleans your inbox and refreshes a cache fails in three ways and tells you about none of them. Split it.

## Step 4: pick a minute that is not on the hour

Everyone who asks for "around nine" gets `0 9 * * *`, so every routine on the planet lands at the same instant and queues behind each other. When the user's time is approximate, nudge it: `57 8 * * *` or `4 9 * * *`. Hourly becomes `7 * * * *`, not `0 * * * *`.

Use `:00` or `:30` only when the user named that exact minute and clearly meant it, because it lines up with a standup or a market open. Nobody notices four minutes. The queue does.

Cron is five fields, minute hour day-of-month month day-of-week, and it is read in the user's local time. No timezone math.

```
57 8 * * *     most mornings, just before nine
4 9 * * 1-5    weekdays, just after nine
33 17 * * 5    Friday evening wrap-up
0 7 1 * *      the 1st of the month at 7am
7 * * * *      hourly, off the hour
```

For a genuine one-time run ("remind me at 3pm tomorrow"), use the one-time timestamp field, not a cron expression. Cron has no concept of firing once, and a pinned cron will come back to haunt the same date next month.

## Step 5: make it fail loud

This is the part everyone skips and the reason routines rot. A run that errors halfway through still returns text, and text that reads like a summary looks like success. Unless the routine checks its own work, nobody finds out for weeks.

Every routine that produces an artifact ends with a verification block. Put this in the prompt itself, not in your head:

- After writing, read the file back from disk at the exact path.
- If it is missing or empty, retry the write once.
- If it still fails, begin the entire output with a loud failure marker and the literal tool error. Do not summarize the work as though it happened.
- If it verified, end the output with the full path that was written.

For routines whose output is a message rather than a file, verify the equivalent: the draft exists, the row landed, the count went up. Verify the effect, not the intention.

## Step 6: dry run before you arm it

Run the prompt once, right now, by hand. Not a paraphrase: the exact text you are about to store. Then check three things:

- The artifact actually exists where the prompt claimed it would.
- Nothing in the run depended on context that only exists in this conversation.
- Every connector the prompt names was actually reachable.

A routine that has never completed once will not start working at 7am on a Tuesday.

## Step 7: arm it, then tell the truth about how it runs

Create the task with a kebab-case id that reads like what it does (`morning-briefing`, `weekly-content-review`), a one-line description, the verified prompt, and the schedule.

Then tell the user, in plain terms:

- It runs while the app is open. If the app was closed when it came due, it runs on next launch instead of being skipped.
- Recurring runs get a few minutes of deliberate dispatch delay. One-time runs fire on time.
- The prompt now lives on disk at `~/.claude/scheduled-tasks/<task-id>/SKILL.md` and can be read or edited there.

## Managing what is already there

- **List first, always.** Never guess a task id. Listing gives you the id, the schedule, the next run, the last run, and the path to its prompt.
- **Read the prompt before editing it.** Open the `SKILL.md` at the listed path. Editing a routine you have not read is how the verification block gets deleted.
- **Update in place.** Change the schedule, the prompt, or the description without recreating the task, so its history stays intact.
- **Pause instead of deleting.** Disabling stops the automatic runs and keeps everything else. Delete only when the routine is genuinely dead.
- **`lastRunAt` is the health check.** A routine whose last run is older than its interval is broken, no matter how good its output looked the last time anyone read it.

## Done looks like

- One routine, one job, with a name that says what it does.
- A prompt a stranger could execute with no other context.
- A schedule on an odd minute, in local time, with one-time runs using a timestamp rather than a cron.
- A verification step that reads the artifact back and shouts on failure.
- A completed manual dry run before it was ever armed.
- The user told, out loud, when it fires, where the output lands, and how to pause it.

## Anti-patterns

- Hardcoding today's date into a daily routine.
- "Continue what we discussed" or any other pointer to a conversation the run cannot see.
- Bundling four chores into one routine so a single failure takes out all four.
- Scheduling something irreversible and unattended.
- Using session cron for a routine the user expects to still exist next week.
- Shipping without a dry run, then discovering at 7am that a connector was never connected.
- Treating a returned summary as proof the file was written.

## Reference

Claude Code's scheduling behavior, cron semantics, and task storage layout are documented by Anthropic at [docs.claude.com/en/docs/claude-code](https://docs.claude.com/en/docs/claude-code). When the runtime details in this skill and the official docs disagree, the docs win.
