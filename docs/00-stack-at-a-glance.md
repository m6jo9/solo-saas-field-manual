# Stack at a glance

This manual assumes a pretty common “solo SaaS” setup:

- A web app (Next.js / Remix / Rails / whatever)
- A CDN/WAF in front (cheap filtering at the edge)
- A real database with proper access control (row-level security helps a lot if you use it correctly)
- Object storage for user uploads (with signed URLs)
- A managed auth provider or auth library you *actually* understand

None of these are requirements. The patterns are.

## Guiding principles

### 1) Middleware is a bouncer, not a judge
Use edge/middleware to block obvious junk:
- abusive IPs / ASNs (careful: false positives)
- missing/invalid origin
- obviously automated traffic (no cookies, no JS, suspicious UA)
- rate-limit spikes

But don’t let middleware become your “security layer”.
Real enforcement should happen where you have context: server handlers, DB policies, signed artifacts.

### 2) You can't outsmart “free”
If you have *any* free capability (including “free trial”), expect automation.
The goal is not “no abuse”. The goal is:
- cheap to detect
- cheap to throttle
- cheap to recover from

### 3) Make the happy path fast, not the unsafe path easy
Users will tolerate a small amount of friction **if** it’s predictable.
Bots will not. That’s leverage.

### 4) Favor mechanisms you can run alone
If you’re solo, you want:
- fewer moving parts
- fewer dashboards
- fewer “it’s probably fine” assumptions

A good “solo” system is one you can debug from logs, not vibes.
