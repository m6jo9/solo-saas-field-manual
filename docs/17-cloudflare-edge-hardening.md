# Cloudflare edge hardening (CDN + WAF + origin protection)

Cloudflare should be your **public edge**.
Your app should assume *the internet is hostile* and *the origin is private*.

Use Cloudflare for:
- CDN + caching
- basic bot/WAF controls
- rate limiting (often paid)
- origin IP hiding + firewalling

## Minimal setup (Free plan)

Do this first:
- **Proxy** your DNS records (orange cloud) for the app domain.
- TLS mode: **Full (strict)** (use a real cert on origin).
- Enable **Always Use HTTPS**.
- Enable **Auto Minify** (HTML/CSS/JS) only if it doesn’t break your app.
- Enable **Brotli** (or equivalent compression toggle).
- Security: turn on basic bot protections if available on Free (e.g., “bot fight” style toggle).

Keep it boring:
- One canonical hostname.
- One origin (or a small, explicit set).

## Cache rules (cache static, bypass API/auth)

Goal:
- cache what’s safe + cheap to cache
- never cache anything user-specific or auth-related

Cache **static**:
- `/_next/static/*`
- `/assets/*`
- `/images/*` (if not user-specific)
- `/*.css`, `/*.js`, `/*.ico`, `/*.png`, `/*.jpg`, `/*.svg`, `/*.woff2`

Bypass cache for **dynamic**:
- `/api/*`
- `/auth/*`
- `/login`, `/signup`, `/logout`
- `/account/*`, `/billing/*`, `/admin/*`
- anything that sets/reads cookies or uses `Authorization`

Practical rule of thumb:
- If the response depends on a cookie, header, or user id → **do not cache at the edge**.

## Rate limiting (note: Pro may be needed)

Edge rate limiting is a force multiplier.
Many Cloudflare rate limit features require **Pro (~$20/mo)** or higher.

Start with intent-based limits:
- `/login` + `/api/auth/*`
- `/signup`
- `/reset-password` (or equivalent)
- expensive endpoints (search, uploads, webhook receivers)

Prefer actions:
- **Managed Challenge** first (less user pain than hard blocks)
- **Block** for obvious abuse patterns

### Suggested limits (starting point)

| Endpoint | Keying | Limit | Window | Action |
|---|---|---:|---:|---|
| Login | IP + email/username | 10 | 1 min | Managed Challenge |
| Signup | IP | 3 | 1 hour | Block |
| Password reset | IP + email | 5 | 1 hour | Managed Challenge |

Notes:
- If you can key by **email**/**username**, do it (bots rotate IPs).
- Add a small server-side delay after repeated failures (even 250–750ms helps).

## Turnstile (optional)

Only add a challenge when you have a signal.
Good triggers:
- repeated auth failures
- burst signup attempts
- suspicious countries/ASNs (if you use geo rules)
- “cheap to create, expensive to serve” endpoints

Don’t gate *everything*.
Gate:
- signup
- login after N failed attempts
- password reset
- contact/feedback forms (spam magnets)

## Origin protection (hide origin IP)

You want:
- Cloudflare → origin
- everyone else → **no route**

Conceptually:
- keep the origin IP **out of DNS** (only Cloudflare is public)
- firewall the origin to **only allow Cloudflare** IP ranges (by source IP)
- if you must SSH, restrict it to your IP (or a VPN), not the whole internet

Practical patterns:
- bind your app to `127.0.0.1` and proxy through Nginx locally
- allow inbound 80/443 only if it’s from Cloudflare (at the server firewall)
- log + alert on any direct-origin hits (they should be zero)

## Emergency mode (“panic button”)

Have a plan for “we’re getting hammered”:
- Enable “under attack” style mode / higher security level.
- Tighten rate limits on auth + expensive endpoints.
- Temporarily **disable signup** (feature flag) and serve a simple message.
- Cache more aggressively for public pages (if safe).
- Block obvious abusive geos/ASNs *only if you can tolerate collateral*.
- If needed, route traffic to a static maintenance page.

Don’t improvise while on fire.
Write the steps down now.

## Launch checklist

- [ ] DNS is proxied through Cloudflare (no direct origin record exposed)
- [ ] TLS is Full (strict); HTTP redirects to HTTPS
- [ ] Static routes cached; API/auth routes bypass cache
- [ ] Auth endpoints have edge rate limits (and in-app limits)
- [ ] Turnstile is enabled only where it pays for itself
- [ ] Origin firewall only allows Cloudflare source IPs (conceptually enforced)
- [ ] Direct-to-origin traffic is impossible (or alarms immediately)
- [ ] “Panic button” steps documented + tested
