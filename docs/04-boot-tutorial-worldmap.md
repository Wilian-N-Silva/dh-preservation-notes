# 04 · Boot, tutorial and the first world map

The tutorial is a **state machine** driven by `tutorial_step`, which the server returns in
the login response and the client advances with `tutorial_step` / `tutorial_complete`
calls. Replies shown here are *reconstructed* (they make the original client behave
correctly); they are not recovered historical server responses.

## Boot

1. **Resource check** (`get_image_list`, API 50): the server returns `file_list[]`
   (`file_name`, `mode`, `md5`, `lang`) and the download server inside `result`, and
   **`server_time`** at the root of the JSON document (a sibling of `result`, not inside it).
   The qualified success status is `S20000`. Files whose MD5 differs are downloaded.
2. **Login** (API 0): the response carries the whole account (`cards`, `decks`, `item`,
   `quest` progress, `card_book`, balances, AP, capacity), **`tutorial_step`** and
   **`serverTS`**.
3. **Home** (`common_home`, API 88) when entering the main screen.

## Tutorial steps (as qualified)

| Step | What happens | Confidence |
|---|---|---|
| 0 → 10 | Login returns step 0; the client moves to 10 and enters the **guided battle** (unless local resume state diverts it) | CONFIRMED (static) |
| 20 / 30 | After the guided battle, the completion widget registers step 20 **or** 30 depending on configuration; 20 is not seen in every run | CONFIRMED (static), conditional |
| 30 | `tutorial_step = 30` (API 35), then `tutorial_complete` (API 34) carrying the **team** the client built; acknowledging API 34 does not grant cards by itself | CONFIRMED (dynamic) |
| 40 | After the API 34 ack, `tutorial_step = 40`; its ack triggers a **new login** (API 0). This return to login is expected | CONFIRMED (dynamic) |
| 40 (login) | The login at step 40 must return the **same owned roster and decks** | CONFIRMED |
| gift | With a roster of **≤ 5 cards** the client shows the tutorial **gift** and calls the egg purchase API (25) for it; a larger roster skips the gift. Do not add cards just to skip it | CONFIRMED |
| 50 | The gift reply must keep the existing roster and add the gift **once**; then `tutorial_step = 50` | CONFIRMED (dynamic) |
| 60 | The ack of step 50 leads to `tutorial_step = 60`; its ack triggers a login and the transition to the **main widget** | CONFIRMED |
| 60 (login) | The login restores step 60 with the same ownership and decks, **without** granting the gift again; the next request is the card list (API 16). Step 60 is still tutorial continuation, not the end of the tutorial | CONFIRMED (dynamic) |

A minimal acknowledgement for API 34/35 is `{"result_code": "", "result": {}}` (the client
accepts an empty status for these).

## Getting from step 60 to the world map

This is where most reconstructions get stuck. The following configuration (real UTC clock)
is qualified to reach the map widget (`uiWorldMapWidget`); not every point is proven to be
strictly necessary in every setup:

1. **`common_home` (API 88) succeeds.** Every sub-parser of its result must succeed before the
   client proceeds; a successful login alone is not proof of map entry (CONFIRMED).
2. **The resource contains coherent `quest` and `quest_area` rows** for the first chapter.
   One required quest lookup dereferences the result **without a null check**: a missing or
   placeholder quest crashes or loops instead of showing an error (CONFIRMED). Serving coherent
   rows avoids the need to patch the client.
3. **The quests are active now**: `start <= now < end` against the **server clock**
   (CONFIRMED). If the first chapter has no active quest, the client can fall back repeatedly
   instead of drawing a usable map (INFERRED).
4. **The server clock is synchronized everywhere**: `result.serverTS` in login and API 88 and
   the root `server_time` in API 50. API 50 updates the time base unconditionally, so a later
   response with a missing/empty value can undo an earlier sync (CONFIRMED).
5. **Dates are int32-safe UTC strings** (see [09](09-calendar-and-events.md)). With a real
   "now", any quest whose original end date is in the past is hidden: adapt the windows of the
   copy you serve (for example end `2037-12-31 00:00:00`).

## Traps that send you back to the title screen or freeze

- Sending `serverTS` / `server_time` in one response and omitting it in another.
- Returning a **partial** `cards[]` or `item[]` (the client replaces its inventory).
- Inconsistent account timestamps between calls (community report: a changing creation time
  sends the client back to the title screen; not verified in these notes).
- A shop/item call with an unexpected `context` value (the client waits for a branch that
  never comes).
- Missing master rows that a screen joins to (quests, areas, enemy groups, drop tables): many
  joins have no fallback.
- Dates after 2038 or empty end dates.
