# 03 · Endpoints

A **qualified subset** of the client's API (Android 1.0.52): calls whose request and the
response fields the client reads were verified. It is not an exhaustive list of every API
the client knows. All requests use the envelope in [02](02-protocol.md).

"ACCEPTED" in the status column means: the client behaves correctly with that value. It is
not proof of the historical server's code. Where the historical code is unknown, it is marked so.

| apiID | Script / action | Request fields (beyond the envelope) | Success status | Main response fields read by the client | Confidence |
|---|---|---|---|---|---|
| 0 | `homeAPI.php?action=login` | `android_uiid`, `store_name` | `s70000` | `cards`, `decks`, `item`, `quest` (progress), `card_book`, `tutorial_step`, balances, level/EXP, AP, capacity, `serverTS` | CONFIRMED |
| 5 | `giftAPI.php?action=get_all` | `gift_box_ids[]` | empty ACCEPTED; historical UNKNOWN | `gift_box_list[]`, full `cards[]`/`item[]`, `coin`, `gold`, `friend_point`, `max_digimon_count`, `action_point`, `time_to_max_action_point` | CONFIRMED |
| 8 | `friendAPI.php?action=apply` | `request_id`, `apply_context` | `s40000` | acknowledgement only | CONFIRMED |
| 11 | `cardAPI.php?action=deck_edit` | `decks`, `use_deck_no`, `pvp_use_deck_no`, `pvp_def_use_deck_no` | `s10000` | deck fields | CONFIRMED |
| 12 | `cardAPI.php?action=sell` | `sell_list[]` | empty ACCEPTED; historical UNKNOWN | `gold`, full `cards[]` | CONFIRMED |
| 14 | `cardAPI.php?action=union` (Digifuse) | `base_user_card_id`, `sacrifice_cards[{user_card_id}]` (1–5) | omitted ACCEPTED; historical UNKNOWN | `base_card`, `great_success_flg`, `gold`, full `cards[]`, `decks` | CONFIRMED |
| 15 | `cardAPI.php?action=evolution_union` | `base_user_card_id`, `item_id`, `item_count`, `evolution_id`, `sacrifice_cards{user_card_id1..5}` | historical UNKNOWN | `evolution_card`, full `cards[]`/`item[]`, `decks`, `gold` | INFERRED (reader qualified) |
| 16 | `cardAPI.php?action=list` | — | empty ACCEPTED | `cards[]` | CONFIRMED |
| 24 | `gashaAPI.php?action=list` | `future` | empty ACCEPTED | `gasha_info[]` (categories, images, `gasha[]` with IDs, currency, cost, count, bonus, display dates) | CONFIRMED |
| 25 | `gashaAPI.php?action=buy` | `gasha_id`, `multiple`, `offer_id`, `free` | `s50000` | `new_cards[]`, full `cards[]`, `coin`, `gasha_point`; optional `user_item[]` (replaces items) | CONFIRMED |
| 27 | `gashaAPI.php?action=list_all_active_digimons` | — | `S50000` | `gasha_info[{gasha_id, digimon_list[{weight, card_id}]}]` (the egg *Info* list) | CONFIRMED |
| 28 | `shopAPI.php?action=list` | — | empty ACCEPTED | `shop_product[]` (id, name, item, quantity, price, image, dates…) | CONFIRMED |
| 29 | `shopAPI.php?action=buy` | `shop_product_id`, `context`, `free_tickets` | `s120000` | `item_id`, `count`, `coin`, AP fields, `max_digimon_count`, team limits, `item[]` | CONFIRMED |
| 30 | `shopAPI.php?action=multi_buy` | UNKNOWN | UNKNOWN | UNKNOWN (bundle purchase route) | identified only |
| 33 | `registAPI.php?action=regist_name` | `name` | `s110000` | acknowledgement | CONFIRMED |
| 34 | `registAPI.php?action=tutorial_complete` | `decks[]` | empty ACCEPTED | acknowledgement | CONFIRMED |
| 35 | `registAPI.php?action=tutorial_step` | `tutorial_step` | empty ACCEPTED | acknowledgement | CONFIRMED |
| 36 | `itemAPI.php?action=use` | `item_id`, `count`, `context` | `s80000` | AP fields, `max_digimon_count`, `item[]` | CONFIRMED |
| 39 | `questAPI.php?action=quest_start` | `quest_id`, `quest_area_id`, `friend_user_id`, `use_deck_no`, autobattle flags, `enemy[]` (generated enemy IDs) | `s90001` | `decks`, `friend_card`, AP fields, `continue_price` | CONFIRMED |
| 40 | `questAPI.php?action=quest_end` | quest/area/friend/deck IDs, `enemy[]`, `card[]` (dropped), `exp`, `gold`, `friend_point`, `missions_progress`, `fight_turns`, elapsed time, flags | `s90002` | `quest` (progress), new `cards[]`, `item[]`, `quest_reward`, level/EXP/gold, AP, `card_book` | CONFIRMED |
| 44 | `questAPI.php?action=give_up` | `quest_id`, `quest_area_id`, `continue_count`, `enemy[]`, `card[]`, `give_up_context` | `s90006` | balances, item count, AP fields | CONFIRMED |
| 46 | `friendAPI.php?action=search2` | `invite_id`, `friends`, `fb`, `all`, `current_team`, `exclude[]` | empty ACCEPTED | `user[]` (helper candidates with `leader_card`) | CONFIRMED |
| 50 | `commonAPI.php?action=get_image_list` | not established | `S20000` (qualified branch) | `file_list[{file_name, mode, md5, lang}]`, download server; `server_time` at the document root | CONFIRMED (response) |
| 65 | `homeAPI.php?action=show_coins` | — | `S70001` | login-shaped account result (balances) | CONFIRMED |
| 66 | `friendAPI.php?action=get_friends` | — | `S40000` | an empty result is accepted; populated list UNKNOWN | CONFIRMED (bounded) |
| 88 | `commonHomeAPI.php?action=common_home` | `send_login_bonus` | sub-status object (e.g. daily login `s70004`) | `friendpoints_list[]`, optional `quest`, `gift_box_list`, `daily_login`, `serverTS` | CONFIRMED |
| — | `pingAPI.php?action=ping` | `user_bucket`, `o_type`, `aux`/`value` | none needed | — | CONFIRMED (request) |

Partially mapped (native readers exist, wire contract not qualified): API 2 (login bonus),
API 3 (gift list, reads `result.gift_box_list.gift_box_list`). API 4 is a native stub.

## Notes per call

- **quest_start (39)**: the client sends the enemies it generated. The response decides the
  helper ("friend") card and the continue price.
- **quest_end (40)**: the client sends totals; the server validates them (see
  [07](07-progression-formulas.md) §1) and returns the authoritative progress and new cards.
  `quest` is the list of area progress entries (`quest_id`, `quest_area_id`, `step`).
- **Full-list responses** (`cards[]`, `item[]`): always send the complete inventory.
- **Shop contexts** (`context` in 29/36): the same purchase behaves differently depending on
  the menu it came from (for example an "upgrade" context applies the item immediately; an
  "item" context only stores it). An unexpected context can leave the client waiting.
