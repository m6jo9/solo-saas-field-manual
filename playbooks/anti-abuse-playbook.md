# Anti-abuse playbook

This is a “do the next right thing” checklist when you notice abuse.

## 0) Don’t panic: confirm the shape
- What is being hit? (signup, login, search, uploads, public pages)
- Is it spiking now or slowly rising?
- Is it concentrated on a few IPs or widely distributed?
- What is the cost driver? (DB, egress, CPU, third-party APIs)

If you can’t answer those, start with logs + a simple request counter.

## 1) Stop the bleeding (cheap controls)
Pick the bluntest tool that works:
- raise rate limits for the abused endpoints
- enable “challenge” on suspicious flows
- disable new signups temporarily
- turn off a risky public feature
- add a temporary allowlist for known-good traffic (careful: can backfire)

### Reason codes matter
When you block/challenge, log a reason code.
If you don’t, you’ll “fix” it and learn nothing.

## 2) Protect the expensive parts
- cache hot reads
- cap pagination (hard maximum, no “page=999999”)
- add per-account quotas for expensive actions
- introduce cooldowns (per action)

## 3) Clean up and prevent re-entry
- ban/disable abusive accounts
- invalidate sessions/tokens for compromised users
- rotate any credentials you suspect were exposed
- add automated tests around the boundary you just patched

## 4) Post-incident notes (10 minutes)
Write down:
- what broke first
- what you changed
- what you’ll automate next time

If you’re solo, the “postmortem” can be a markdown file.
The point is to avoid repeating the same incident.
