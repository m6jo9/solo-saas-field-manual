# App hardening (Next.js-flavored, works elsewhere)

You can do a lot without turning your app into a fortress.

## The minimum viable hardening (do this first)

### Enforce a single public origin
Most abuse is “drive-by” automation. Make requests prove they came through your intended origin.

- Validate `Origin` and/or `Host` for state-changing requests.
- Don’t rely on CORS alone. CORS is a browser control, not a server control.

If you support multiple domains, make the allowlist explicit and version-controlled.

### Rate limit by *intent*, not just IP
IP-only rate limits are easy to dodge (VPNs, residential proxies).
A better approach is layered keys:
- IP as a coarse signal
- user/session identifier when present
- device/browser fingerprint *only if you understand the privacy trade-off*
- resource-specific keys (e.g., “per email”, “per username”, “per invite code”)

If you’re solo: start with IP + user/session.

### Put state changes behind a CSRF strategy you can explain
If you have cookies + sessions, you need CSRF protection.
If you have token auth in headers, CSRF is usually not the primary risk.

Pick one approach and make it consistent:
- SameSite cookies + CSRF token for unsafe methods
- OR strict auth headers and no ambient cookies for APIs

### Fail closed on admin surfaces
This is where small apps get wrecked:
- admin endpoints accidentally exposed
- “temporary” bypasses left in
- “internal” flags controlled by client input

If something is admin-only, enforce it server-side and in the database.
Don’t just hide the button.

## Edge/middleware controls (the cheap bouncer)
Good middleware rules:
- are fast
- are deterministic
- have clear logs/reason codes
- can be rolled back quickly

Bad middleware rules:
- try to do full authentication
- depend on external services on every request
- block real users during spikes

### Add reason codes
When you block or challenge, emit a short reason code.
Future-you will thank present-you when a customer asks “why can’t I sign up?”.

Example codes:
- `DENY_ORIGIN_MISMATCH`
- `DENY_RATE_LIMIT`
- `CHALLENGE_CAPTCHA`
- `CHALLENGE_STEPUP_AUTH`

## “I shipped and then…” (common failures)
- I shipped “public search” and got scraped into the ground.
- I shipped signup and got botted before my first tweet finished loading.
- I had a single admin route that wasn’t locked down “because nobody knows it exists”.

These aren’t exotic. Assume they’ll happen and make the safe option the default.
