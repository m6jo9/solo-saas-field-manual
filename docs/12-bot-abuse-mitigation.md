# Bot + abuse mitigation (without turning your app into a jail)

You’re not fighting “hackers”. You’re fighting **economics**.

Bots win when:
- your endpoints are cheap to call
- your checks are expensive
- your system is easy to reset (new account, new IP, repeat)

Your job is to change that math.

## Layer 1: make “first contact” annoying
Do the cheap stuff first:
- basic rate limiting on signup/login/search/upload
- block obviously invalid origins/hosts
- add small artificial delays on repeated failures (server-side)
- require email verification for anything that costs you money

If you’re solo: a single middleware gate + a small in-app rate limiter is fine.

## Layer 2: challenge only when suspicious
CAPTCHAs are a tax on real users.
Use them like a seatbelt: you want them there, but not always engaged.

Good triggers:
- many attempts from a new session
- impossible traffic patterns (signup → immediately hammer expensive endpoints)
- high-risk actions (bulk upload, public posting, link creation)

Bad triggers:
- “every signup”
- “every page load”
- “everyone from country X”

## Layer 3: make abuse expensive to scale

### Add friction that bots hate
- per-account cooldowns (e.g., “can do X once per Y minutes”)
- per-resource quotas (uploads, posts, invites)
- proof of work *if you’re comfortable with the trade-offs*
- require a “warmup” period before powerful actions

### Use signed artifacts
For anything that can be replayed, use short-lived signed tokens:
- upload intents
- download URLs
- action tokens (e.g., confirm destructive actions)

This prevents “scrape once, replay forever”.

## Instrumentation that actually helps
You want to know:
- where abuse enters (which endpoints / flows)
- what it costs (CPU, egress, storage, support time)
- which mitigations worked (drop rate, false positives)

Track reason codes for blocks/challenges.
If you can’t measure it, you’ll just keep adding friction and hope.

## When you should *not* fight the bots
Sometimes the right answer is:
- remove the public feature
- add an account requirement
- add a delay queue
- throttle hard and accept some user pain

This is not moral. It’s math.
