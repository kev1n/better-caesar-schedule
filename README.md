# better-caesar-schedule

Live config for the [Better CAESAR](https://github.com/kev1n/better-caesar)
extension. The extension fetches `bucket-schedule.json` from GitHub's raw
CDN (`raw.githubusercontent.com`, served with `access-control-allow-origin: *`
and a short cache TTL), so editing this file and pushing reflects in
clients within minutes.

## Edit

```sh
git pull
$EDITOR bucket-schedule.json
git commit -am "schedule: <what changed>"
git push
```

The extension client-caches for 30 min and fails open: if this file
becomes unreachable for longer than the cache window, the extension
behaves as if no server existed (no kill, no banner, all buckets open).

## Schema

```json
{
  "releases": [
    "2026-06-01T17:00:00Z",
    "2026-06-08T17:00:00Z",
    "2026-06-15T17:00:00Z"
  ],
  "kill": null,
  "banner": {
    "id": "2026-08-15-maint",
    "message": "Heads up: maintenance Fri 9pm. [Status](https://example.com/status)."
  }
}
```

- **`releases`** — three ISO-8601 timestamps, ordered:
  - `[0]` Class of 2027 and earlier
  - `[1]` Class of 2028
  - `[2]` Class of 2029 and later (open release)
  Each bucket unlocks once `Date.now()` is past its timestamp.

- **`kill`** — set to `{ "id": "...", "message": "..." }` to disable the
  extension entirely; `null` to clear. Kill *behavior* always applies —
  `id` controls only the toast UI dismissal.

- **`banner`** — set to `{ "id": "...", "message": "..." }` to show a
  passive amber strip at the top of CAESAR / paper.nu; `null` to clear.

`id` is the dismissal key. Once a user clicks the close button, that id
is cached in their browser and won't re-show. Edit `message` while
keeping the same `id` to fix typos without re-pestering everyone; bump
`id` (e.g. `"maint-2026-08-15-v2"`) to force a re-show.

`message` supports inline `[text](url)` links with `http://` or
`https://` URLs. Anything else renders as plain text.

## Why this exists in a separate repo

The extension needs a stable, CORS-enabled URL it can fetch
`bucket-schedule.json` from. GitHub's raw CDN gives us that for free —
no server, no Docker, no auth, no maintenance — at the cost of the file
being publicly readable. That's fine: the URL ends up in the extension
bundle anyway, and there are no secrets in the schedule.
