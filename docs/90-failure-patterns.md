# Failure patterns (a.k.a. “things that break in small SaaS”)

No war stories with identifying details here — just common patterns I’ve seen repeatedly.

## 1) “We shipped a public endpoint and it got scraped”
Usually this is search, profiles, or some “explore” page.
Symptoms:
- traffic ramps fast
- cache hit rate looks weird
- egress spikes
- your DB cries

Fixes that actually help:
- require auth for high-volume views
- add pagination + hard caps
- add cheap caching + request collapsing
- rate-limit by resource, not just IP

## 2) “Admin was ‘hidden’, not protected”
An admin page isn’t safe because it isn’t linked.
If it exists, it will be found.

Fix:
- server-side role checks
- data-layer enforcement
- separate admin surface (even separate subdomain) if you can

## 3) “Uploads became a money pit”
Public uploads attract:
- large files
- repeated retries
- content you don’t want to host

Fix:
- signed uploads
- strict size/MIME limits
- per-account quotas
- short-lived download links

## 4) “Auth edge cases caused user-facing chaos”
Common culprits:
- callback URL mismatches
- cookies not set in certain environments
- cached SSR pages treated as public

Fix:
- deterministic redirect rules
- automated tests for auth boundaries
- disable caching on user-specific pages unless you know what you’re doing

## 5) “One dependency update took prod down”
It’s not the update. It’s the lack of rollback discipline.

Fix:
- pin versions
- ship small changes
- have a known-good redeploy path
