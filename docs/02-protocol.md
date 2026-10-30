# 02 · Protocol

## Transport

- Plain **HTTP/1.1** (no TLS in the analysed build) to the game's API host.
- Each call is a `POST` to `/<script>.php` with a form-encoded body
  (`application/x-www-form-urlencoded`) containing four fields:

| Field | Meaning |
|---|---|
| `action` | Action name (also repeated in the URL query `?action=`) |
| `params` | A **JSON** object (the actual request, see below) |
| `token` | A per-request authentication value. It exists; how it is computed is intentionally not documented here |
| `tsec` | A timestamp-like value sent with the token |

## The `params` envelope

Every request JSON carries a common envelope plus call-specific fields:

```json
{
  "url": "http://<api-host>/questAPI.php?action=quest_start",
  "protocol": 51,
  "msg_id": -1,
  "user_id": 1000001,
  "lang": "1",
  "sys": 2,
  "apiID": 39,
  "ver": "105211368",
  "app_ver": "2.220",
  "...call specific fields...": "..."
}
```

- `apiID` identifies the call (see [03](03-endpoints.md)); the client matches the response to
  the request by it.
- `sys` = platform (2 = Android in the analysed build), `lang` = language index.
- `protocol` / `ver` / `app_ver` are fixed for a given client build.

## Responses

Responses are JSON:

```json
{ "result_code": "s90001", "result": { "...": "..." } }
```

- `result_code` is a short status string. Codes starting with lowercase `s` or uppercase `S`
  followed by digits are success codes per API family (for example `s70000` login,
  `s90001` quest start, `s90002` quest end, `s50000` egg purchase, `S50000` egg info list).
  Some readers accept an **empty** string; the historical code for those APIs is unknown.
- Error and maintenance codes exist (the client shows dialogs or returns to title), but they
  are not catalogued here.
- `result` contains the payload. Many APIs return **complete** collections (for example the full
  `cards[]` list after a purchase or fusion): the client **replaces** its local copy with what it
  receives, so a partial list means the other cards disappear on the client.
- Several responses also carry `serverTS` / `server_time`: the server's current time, used as
  "now" for every date window (see [09](09-calendar-and-events.md)).

## Practical rules learned the hard way

1. **Field types matter.** The native JSON readers read numbers as numbers and strings as
   strings; a number sent as a string (or the reverse) is read as the default value.
2. **Absent keys fall back to defaults** (0, -1, empty). A missing `serverTS` makes "now" = 1970.
3. **Order is irrelevant, keys are not.** Readers look up keys by name.
4. **Replace semantics**: send full lists where the client expects full lists.
5. The client also sends analytics/SDK traffic to third-party hosts at startup; the game works
   without them answering.
