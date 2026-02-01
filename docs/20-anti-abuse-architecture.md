# Anti-abuse architecture (practical, not academic)

A public SaaS is an economy:
- attackers spend time/money to get value (spam, scraping, resale, harassment)
- you spend time/money to stop them
- users spend patience when you add friction

The goal is to push abuse costs up without pushing user costs too high.

## Layers that work

### 1) Front door: cheap filtering
CDN/WAF + middleware:
- allowlist your public origins
- block obvious junk
- coarse rate limits
- early “challenge” for suspicious flows

Think of this as the bouncer.

### 2) App layer: intent-aware controls
Your server knows context:
- which account
- which action
- how expensive it is
- whether it’s new/suspicious

Do:
- per-action cooldowns (post/create/invite)
- per-account quotas (uploads/egress)
- progressive friction (challenge only after suspicious signals)

### 3) Data layer: hard enforcement
This is the lock.
If a request bypasses middleware, the DB should still prevent cross-tenant access.

If you can’t enforce at the DB:
- enforce in a single data-access layer
- and write tests that prove it

### 4) Signed artifacts for replay resistance
Short-lived tokens/URLs for:
- uploads/downloads
- sensitive actions
- one-time operations

These reduce “capture and replay” abuse.

## Kill switches (solo-friendly)
Have at least one:
- global rate limit multiplier
- maintenance mode
- disable new signups
- disable a risky feature

It’s better to ship a blunt kill switch than to build a perfect classifier you can’t maintain.

## Trade-offs you should be honest about
- CAPTCHAs reduce bots and also reduce conversions.
- IP blocks reduce abuse and also block legitimate shared networks.
- Strict quotas reduce costs and also frustrate power users.

Write down your default posture.
When you change it during an incident, you’ll know what “normal” was.
