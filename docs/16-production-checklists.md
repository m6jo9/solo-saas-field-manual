# Production checklists

This is the stuff I wish I had written down earlier.

## Pre-ship (public launch)
If you do nothing else, do these:

### Security + access
- [ ] Admin surfaces require server-side authorization (not just hidden UI)
- [ ] Privileged credentials never run in the browser
- [ ] CSRF strategy is consistent for state-changing requests
- [ ] Uploads are signed + size-limited + MIME-checked
- [ ] Basic rate limits on: signup, login, password reset, search, uploads, public posting
- [ ] Secrets live in a secret manager or locked-down env vars (no “checked-in” configs)

### Reliability
- [ ] You have a rollback plan (even if it’s “redeploy previous commit”)
- [ ] You can redeploy from scratch (docs + scripts, not tribal memory)
- [ ] Background work has retries + dead-letter behavior (or is intentionally synchronous)

### Observability
- [ ] Error tracking is wired up in production
- [ ] Logs include request ids + user/session ids
- [ ] You can answer: “what changed?” (version tagging)

### Costs + abuse
- [ ] Quotas exist for the expensive stuff (uploads, egress, heavy queries)
- [ ] You can ban/disable accounts quickly
- [ ] You can rate-limit without code changes (config switch)

## Weekly “keep it healthy”
- [ ] Review error tracker top issues
- [ ] Review top blocked/challenged reason codes (false positives?)
- [ ] Rotate/expire old tokens if your stack supports it
- [ ] Check storage growth + egress trends
- [ ] Confirm backups exist and you can restore one

## Incident readiness (5 minutes)
- [ ] Print or pin **playbooks/incident-quickstart.md**
- [ ] Put your “who to contact” list somewhere accessible
- [ ] Document the “kill switches” (rate limit up, feature flags off, maintenance mode)

If you’re solo, the goal is not perfection.
The goal is: you can recover fast without guessing.
