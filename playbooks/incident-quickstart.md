# Incident quickstart (solo operator edition)

Print this. Or pin it somewhere you’ll actually see.

## 1) Triage: is it user-visible?
- Are requests failing? (5xx rate, error tracker spike)
- Is the site slow? (p95 latency)
- Are signups/logins broken?
- Are uploads broken?

If yes: announce degraded status somewhere (even a simple banner).

## 2) Freeze risky changes
- Stop deploys unless they are fixes.
- If you just deployed: consider rolling back first.
  Rolling back is often faster than debugging.

## 3) Find the blast radius
- Which endpoints are failing?
- Which users are impacted?
- Is it one region/provider or everything?

## 4) Apply a safe mitigation
Choose one:
- enable maintenance mode
- disable new signups
- raise rate limits / enable challenges
- scale up temporarily (only if you trust the code path)

## 5) Confirm recovery
- error rate down?
- latency back to normal?
- core flows working (login + a read + a write)?

## 6) Leave breadcrumbs
Write down:
- start time (approx)
- symptoms
- actions taken
- what fixed it (or what you believe fixed it)

Future-you will forget details. Notes are cheap.
