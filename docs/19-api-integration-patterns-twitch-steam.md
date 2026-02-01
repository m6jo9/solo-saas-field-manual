# API integration patterns (Twitch + Steam)

Integrations rot when they leak across your app.
Keep them contained and predictable.

## Structure

- `lib/clients/*` → API clients (auth, fetch, normalize)
- `app/api/*` route handlers → thin glue only
- `lib/cache/*` → small TTL caches (token + rate limit safety)

Rules:
- route handlers should not “know” OAuth details
- clients should return **normalized** objects (not raw vendor shapes)

## Twitch pattern

### client_credentials token cache (with expiresAt)

Keep an in-memory cache (and optionally a shared cache):
- `accessToken`
- `expiresAt` (epoch ms)
- refresh a little early (skew)

```ts
// lib/clients/twitch/token.ts
let cached: { token: string; expiresAt: number } | null = null;

export async function getAppToken(): Promise<string> {
  const now = Date.now();
  if (cached && cached.expiresAt - 30_000 > now) return cached.token;

  // POST https://id.twitch.tv/oauth2/token (client_credentials)
  // Use server-only env vars: TWITCH_CLIENT_ID / TWITCH_CLIENT_SECRET
  const { access_token, expires_in } = await fetchTokenSomehow();

  cached = { token: access_token, expiresAt: now + expires_in * 1000 };
  return cached.token;
}
```

### Parallel fetch (users + streams)

Fetch in parallel, then combine.

```ts
// lib/clients/twitch/profile.ts
export async function getChannelSummary(login: string) {
  const token = await getAppToken();

  const [user, stream] = await Promise.all([
    fetchUserByLogin(token, login),
    fetchStreamByLogin(token, login),
  ]);

  return normalizeChannel(user, stream);
}
```

### Normalized return object

Return a stable shape your UI can rely on:

```ts
type ChannelSummary = {
  id: string;
  login: string;
  displayName: string;
  avatarUrl?: string;
  isLive: boolean;
  title?: string;
  viewerCount?: number;
};
```

## Steam pattern

Steam input is messy.
Accept it, normalize it, then proceed.

### Accept messy user input

Inputs you’ll see:
- `steamid64` (17 digits)
- vanity name
- full profile URLs (multiple formats)

Normalize first:

```ts
// lib/clients/steam/normalize.ts
export function normalizeSteamInput(input: string): { vanity?: string; steamid64?: string } {
  const s = input.trim();

  const id64 = s.match(/\b\d{17}\b/)?.[0];
  if (id64) return { steamid64: id64 };

  const vanity = s
    .replace(/^https?:\/\//, "")
    .replace(/\/+$/, "")
    .split("/")
    .filter(Boolean)
    .pop();

  return vanity ? { vanity } : {};
}
```

### ResolveVanityURL before other calls

If you don’t have `steamid64`, resolve it first, then use `steamid64` for everything else.

Flow:
1) normalize input
2) if vanity → `ResolveVanityURL`
3) then call everything else using `steamid64`

### Parallelize API calls (conceptually)

Once you have `steamid64`, fetch in parallel:
- player summaries
- owned games
- recently played
- badges / level

Then normalize into a single object your UI expects.

## Rate limit safety

Default posture: assume vendor APIs will throttle you.

Use:
- small TTL caches (token, profile summaries)
- request coalescing (dedupe concurrent requests by key)
- background refresh (refresh cache after serving stale-ish data)
- ISR/revalidate for public pages (if applicable)
- per-user throttles for “search” style endpoints

Avoid:
- calling vendors on every page load for the same user
- chaining calls serially when parallel is safe

## Never expose secrets

- Keep secrets in **server-only** env vars.
- Never return access tokens to the client.
- Don’t log tokens, auth headers, or full vendor responses.
- Strip/normalize before returning anything to UI.
- If you must debug, log **ids + timing**, not payloads.

## Checklist

- [ ] API clients live under `lib/clients/*`
- [ ] Route handlers are thin (validate input → call client → return)
- [ ] Twitch token cached with `expiresAt` + early refresh skew
- [ ] Twitch fetches parallelize where safe; returns normalized object
- [ ] Steam input is normalized (id64/vanity/URL supported)
- [ ] Vanity is resolved before other Steam calls
- [ ] Vendor calls are cached/throttled; no per-request vendor spam
- [ ] No secrets or tokens ever leave the server
