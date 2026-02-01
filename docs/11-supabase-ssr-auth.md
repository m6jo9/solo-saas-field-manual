# Auth with SSR / server-rendered apps (Supabase-ish patterns)

This is intentionally vendor-agnostic in spirit.

SSR auth has one job: **make it hard to accidentally treat “unauthenticated” as “authorized”**.

## Rules that keep you out of trouble

### 1) Treat the client as hostile
Even if you wrote the client.
Even if it’s “just a dashboard”.
Never trust claims coming from the browser.

The browser can:
- forge requests
- replay tokens
- call endpoints you “didn’t link to”

### 2) Put authorization checks close to the data
If your database supports row-level policies, use them.
If it doesn’t, enforce authorization in your data-access layer.

The goal: no handler should be able to “forget” to filter by owner/tenant.

### 3) Separate “public” and “privileged” server clients
Have an explicit boundary:
- a client that acts like the user (scoped)
- a privileged server client (service role / admin credentials) used sparingly

If you don’t have this boundary, you will eventually leak privilege through a “quick fix”.

### 4) Explicit redirect rules
SSR auth bugs often show up as:
- redirect loops
- “sometimes logged out” state
- caching a page as if it were public

Make redirects deterministic:
- unauthenticated → login
- authenticated but insufficient role → 403 page (don’t bounce endlessly)

## Session storage and cookies
If you use cookies for session storage:
- mark cookies `HttpOnly` where possible
- use `Secure` in production
- set `SameSite=Lax` as a baseline (or `Strict` if you can)
- be careful with cross-site embeds; they change the rules

## What I’ve seen fail in small teams
- Using an admin credential in a request path that sometimes runs in the browser.
- “Temporary” bypasses for support/debug that become permanent.
- Relying on UI hiding instead of server-side enforcement.

If you only do one thing: write a tiny automated test that proves a non-owner can’t read/update another user’s rows.
It’s boring. It saves you.
