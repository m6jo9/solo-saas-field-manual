# Reference snippets (copy-friendly)

These are intentionally small and “pattern level”.
Adapt them to your stack. Don’t copy-paste blindly.

## Decision reason codes

Keep them stable. Don’t encode user PII.

```txt
OK
DENY_ORIGIN_MISMATCH
DENY_RATE_LIMIT
DENY_COOLDOWN_ACTIVE
DENY_AUTH_REQUIRED
DENY_ROLE_REQUIRED
DENY_MIME_NOT_ALLOWED
DENY_SIZE_EXCEEDED
CHALLENGE_CAPTCHA
CHALLENGE_STEPUP_AUTH
```

## Audit log shape (pseudo schema)

You want an append-only record of sensitive actions.
Name it whatever you want — the important bit is the fields.

```sql
-- pseudo-SQL
create table audit_events (
  id            bigserial primary key,
  ts            timestamptz not null default now(),
  actor_id      text,          -- user/service id (avoid PII)
  action        text not null,  -- e.g. "UPLOAD_CREATE"
  target        text,           -- e.g. "file:<id>"
  decision      text not null,  -- OK / DENY_* / CHALLENGE_*
  reason_code   text,
  request_id    text,
  ip_hash       text,           -- hash, not raw IP
  meta          jsonb
);
```

## Signed upload intent (conceptual)

- Server validates auth + quota + MIME + size.
- Server returns a short-lived signed URL/policy.
- Client uploads directly to object storage.
- Client reports completion (object key + checksum).
- Server writes metadata row and triggers processing.

If you need to proxy uploads through your server for control, do it intentionally.
It costs more. It can be worth it.
