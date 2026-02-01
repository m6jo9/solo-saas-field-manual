# DNS + networking basics (for app builders who don’t want to become network engineers)

You don’t need to know BGP. You do need to avoid the common footguns.

## Minimum set of things to get right
- Use a CDN/WAF in front of your origin if your app is public.
- Keep your origin locked down (only allow traffic from the CDN if possible).
- Put admin surfaces behind auth and ideally behind additional restrictions (IP allowlist, VPN, etc).

## DNS records you’ll touch
- `A` / `AAAA` — point a name at an IP
- `CNAME` — alias one name to another
- `TXT` — verification records (email, domain ownership, etc)

### TTL
TTL is “how long caches remember this”.
Short TTLs make changes propagate faster but increase DNS query load.
For most SaaS apps:
- 300–900s TTL is fine for active records
- longer TTLs for stable stuff

## Certificates
Use managed TLS. Don’t hand-roll cert renewal unless you enjoy surprise outages.

## If you only remember one thing
Make it possible to move your app.
Vendor lock-in often arrives as “we forgot how DNS works”.
