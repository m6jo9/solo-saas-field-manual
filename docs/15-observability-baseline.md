# Observability baseline (the solo-friendly version)

You don’t need a NASA mission control wall.
You need enough signal to answer:
- “is it broken?”
- “who is impacted?”
- “what changed?”
- “what do we do next?”

## The minimum set of signals

### 1) Structured logs
Log with fields, not poetry.
At minimum include:
- request id / trace id
- user id (or anonymous session id)
- route name (not raw URL if it contains identifiers)
- status code
- latency
- reason codes for blocks/challenges

### 2) Error tracking
You want stack traces grouped and searchable.
Capture:
- deployment/version
- environment (prod/staging)
- user/session id
- breadcrumbs for key actions

### 3) Basic metrics
You can get far with:
- requests per minute (overall + key endpoints)
- error rate (4xx vs 5xx)
- latency p50/p95
- queue depth (if you have background work)

### 4) Uptime checks (with realism)
Check a couple critical paths:
- homepage / health endpoint
- login
- a “read” path behind auth

Don’t make uptime your only signal. It’s a smoke alarm, not a diagnosis.

## Alerts (don’t page yourself into burnout)
Start with:
- sustained 5xx rate
- error tracker spike
- latency spike
- runaway costs (egress, storage, compute)

Avoid alerting on every 404. Public apps get random noise.

## What tends to break first
- auth callbacks / session handling
- uploads
- background jobs silently failing
- “temporary” debug code accidentally shipped

Observability is not vanity. It’s how you stay calm under pressure.
