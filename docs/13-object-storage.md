# Object storage (R2/S3/etc.) without giving attackers free bandwidth

Uploads are where you quietly go broke:
- egress bills
- storage churn
- malware-ish payloads
- users hotlinking your bucket as a CDN

## The safe default: signed upload + signed download
Flow:
1. Client asks your server for an upload intent.
2. Server checks auth + quotas + content policy.
3. Server returns a short-lived signed URL (or signed POST policy).
4. Client uploads directly to object storage.
5. Client tells your server “upload completed” (with the object key / checksum).
6. Server records metadata + runs async processing if needed.

Downloads:
- either proxy through your app (more control, more cost)
- or issue signed URLs with short TTLs

## Content policy
At minimum:
- allowlist MIME types (don’t trust file extensions)
- max size per file + per account per day
- virus/malware scanning if you have UGC at scale (even “small” scale can get you flagged)

If you’re solo: start with MIME allowlist + size limits + quotas. Add scanning later.

## Naming and access
- Do not use user-provided filenames as object keys.
- Use random IDs (or stable hashes) and store the original name separately.
- Make buckets private by default.
- If you need “public” assets, split them into a separate bucket/prefix.

## “I’ve seen this fail when…”
- public buckets got scraped and rehosted elsewhere
- signed URLs had long TTLs, effectively becoming permanent public links
- upload endpoints accepted arbitrary content and became a malware drop

You don’t need paranoia. You need a few solid guardrails.
