# Release checklist

Small, boring releases beat heroic ones.

## Before you ship
- [ ] Feature flags/killswitches ready for risky features
- [ ] Migrations reviewed (and reversible if possible)
- [ ] Smoke test plan written (2–5 minutes)
- [ ] Rollback plan exists (even if it’s “redeploy previous build”)

## Ship
- [ ] Deploy during a time you can watch it
- [ ] Watch error tracker for 10–15 minutes
- [ ] Watch p95 latency + 5xx rate
- [ ] Confirm login works (auth is a frequent casualty)

## After
- [ ] Tag the release (commit hash / version)
- [ ] Note any manual steps you performed
- [ ] If you hotfixed something, write down why
