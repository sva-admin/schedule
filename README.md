<p align="center"><a href="https://loop.sv-academy.org"><img src="https://raw.githubusercontent.com/sva-admin/claude-skills/main/assets/sv-academy-512.png" width="96" alt="Silicon Valley Academy"/></a></p>

# schedule

Claude wakes up at 7am, does the chore, and proves it did.

> **Where this came from**
>
> Scheduling is built into Claude Code itself. This repo is not that feature: it is an original Silicon Valley Academy how-to skill that teaches Claude to use it well. Anthropic's own documentation is the reference.

## Learn it free, step by step

[Morning briefing on autopilot](https://loop.sv-academy.org/articles/morning-briefing-on-autopilot)

## Try it

Paste this into Claude Code:

```
Install the skill from https://github.com/sva-admin/schedule, then set up a 7am weekday routine that reads my calendar and unread inbox and writes a briefing to a dated note.
```

## The problem it solves

Setting a cron is the easy part. The routine still dies, and here is how.

It dies because the prompt said "the file we were working on", and at 7am there is no we. It dies because someone typed today's date into it and now it rewrites the same stale file forever. It dies because a connector went unauthorized in March and every run since has returned a confident summary of work that never happened.

That last one is the expensive failure. A broken run still produces paragraphs. Paragraphs look like success. Nobody checks the folder for six weeks.

## What the skill does

- **Talks you out of it when a schedule is the wrong shape.** Rhythms belong on cron. Triggers belong on a watcher. Irreversible actions do not run unattended.
- **Picks the right surface.** A persistent task that survives restarts, or a session-only cron that dies with the window. Those are different things and the difference is where routines go missing.
- **Writes the prompt for a stranger.** Absolute paths, named connectors, the date computed at run time, the output format written down instead of remembered.
- **Puts the clock on an odd minute.** Everyone asks for nine, so everyone lands on `0 9`. Yours goes at `57 8` and never queues.
- **Makes it fail loud.** The routine reads its own output back off disk and starts its report with a failure marker if the write did not land. No silent success.
- **Dry runs it before arming it.** The exact prompt, executed once by hand. A routine that has never worked once will not start working on a Tuesday.

## What it is opinionated about

- One job per routine. Bundle four chores and you get four ways to fail and zero ways to tell which one did.
- No hardcoded dates, ever.
- Never guess a task id. List first, read the stored prompt, then edit.
- Pause before you delete. Disabling keeps the prompt and the history.
- `lastRunAt` older than the interval means broken, no matter how good the last output looked.

## License

MIT. See [LICENSE](LICENSE).

## Credit

Scheduled tasks and cron execution are a built-in Claude Code feature by [Anthropic](https://www.anthropic.com). The reference is their documentation at [docs.claude.com/en/docs/claude-code](https://docs.claude.com/en/docs/claude-code). This repository contains no Anthropic code or text: it is an independent Silicon Valley Academy skill that teaches Claude how to drive that feature without producing a routine that rots.

---

Part of the [SV Academy Claude skills collection](https://github.com/sva-admin/claude-skills) · [sv-academy.org](https://sv-academy.org)

SV Academy is the education brand of [Silicon Valley Innovation](https://svinnovation.com), a Bangkok software and AI company that can build this for your business.
