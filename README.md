# Solo SaaS Field Manual (Public)

A small handbook for people shipping (and operating) a solo/very‑small SaaS **in the real world**.

If your app is public-facing, has UGC, offers a free tier, or is even mildly interesting — it will get:
- scraped
- brute-forced
- spammed
- “creative” signup automation
- accidental traffic spikes that look like a DDoS at 2am

This repo is about being *boringly* production-ready with a minimum viable stack.

## What this is
- Practical docs, checklists, and playbooks.
- Patterns you can implement in any stack (examples lean web-app flavored).
- A maintainer voice: trade-offs, gotchas, “here’s what usually breaks”.

## What this is not
- A framework.
- A perfect reference architecture.
- A list of tools you must buy.

## How to use it
1. Read **docs/00-stack-at-a-glance.md** to map this to your stack.
2. Do **docs/16-production-checklists.md** before you ship anything public.
3. Print **playbooks/incident-quickstart.md** (seriously).
4. Copy templates and make them yours. Delete what you won’t maintain.

## Principles (the ones that survive contact with reality)
- **Boring defaults win.** If you can’t explain it half-asleep, it’s too clever.
- **Middleware is a bouncer, not a judge.** Keep obvious junk out. Do real enforcement deeper.
- **Every public feature is an abuse feature.** Treat “nice UX” and “cheap abuse” as conflicting requirements.
- **Your future self is on-call.** Make choices that reduce pager fatigue.

## Repo map
- **docs/** — the field guide
- **playbooks/** — what to do when things go sideways
- **templates/** — threat model + runbook scaffolding
- **infra-samples/** — copyable deployment skeletons with placeholders
- **.env.example** — configuration guidance (no secrets)

## Contributing
If you have scars and you can explain them as repeatable patterns, PRs welcome.
Keep it practical. No vendor marketing. No “just add AI”.

See **CONTRIBUTING.md** for the vibe.
