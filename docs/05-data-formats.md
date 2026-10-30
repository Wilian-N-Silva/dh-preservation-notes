# 05 · Data formats

The client reads its master data from two kinds of tables inside the resource
container: **JSON tables** and **binary `.tbl` tables**. Both describe rows of the
same logical tables; which form a given table uses depends on the build.

All offsets below are byte offsets inside one record. Integers are **32-bit
little-endian signed** unless stated otherwise. Values shown in examples are synthetic.

## Binary `.tbl` tables

### Header (32 bytes)

| Offset | Type | Meaning | Confidence |
|---|---|---|---|
| 0x00 | u32 | Magic `0x31071220` | CONFIRMED |
| 0x04 | u32 | Format/profile number (`67` in the analysed build) | CONFIRMED |
| 0x08 | u32 | Record count | CONFIRMED |
| 0x0C–0x1F | u32 × 5 | Zero in every analysed table | CONFIRMED |

Then `count` records of a **fixed stride** follow back to back; file size =
`32 + count × stride`. IDs are unique per table. Strings are fixed-size,
NUL-terminated UTF-8 buffers (unused bytes zero).

### `quest` (stride 0x45C)

A quest is one node of the world map (a story chapter node, an event, a dungeon).

| Offset | Field | Notes |
|---|---|---|
| 0x000 | `id` | |
| 0x004 | `manage_id` | The quest's **representative area** (see quest_area) |
| 0x008 | `name` | string, 0x3FE bytes |
| 0x408 | `type` | 1 = normal quest, 4 = special event (others exist, see [09](09-calendar-and-events.md)) |
| 0x40C | `background_image_id` | World-map island art: `00_01_MAP_%02d` (falls back to `_01` if missing) |
| 0x410 | `pattern_id` | |
| 0x414 | `event_skit_id` | Story scene played for events (`pro_%04d` archive); 0 = none |
| 0x418 | `event_skit_count` | Number of scene parts |
| 0x41C | `banner_id` | |
| 0x420 | `ranking_id` | -1 = no ranking |
| 0x424 | `max_energy` | |
| 0x428 | `world_map_page` | 0 = event page; 1..9 = story map pages |
| 0x42C | `start_date` | string, 24 bytes, `YYYY-MM-DD HH:MM:SS` |
| 0x444 | `end_date` | string, 24 bytes |

### `quest_area` (stride 0x1038)

A quest area is one playable stage (a battle sequence) inside a quest.

| Offset | Field | Notes |
|---|---|---|
| 0x000 | `id` | |
| 0x004 | `quest_id` | Parent quest |
| 0x008 | `manage_id` | |
| 0x00C | `name` | string, 0x3FE |
| 0x40A | `caption` | string, 0x802 |
| 0xC0C | `background_image_id` | Battle background, e.g. 203xx → `02_bg_xx` |
| 0xC10 | `round` | Number of battle rounds (waves) |
| 0xC14 | `use_ap` | Energy cost |
| 0xC1C | `pop_up_table_flag` | u8 |
| 0xC20 | `scenario_id` | Also the **difficulty-group key** (see [09](09-calendar-and-events.md)) |
| 0xC24 | `scenario_name` | string, 0x3FE |
| 0x1022 | `scenario_flg` | u8 |
| 0x1024 | `before_quest_area_id` | Unlock prerequisite (0 = none) |
| 0x1028 | `end_production_flag` | |
| 0x102C | `max_items` | |
| 0x1030 | `max_digimon` | |
| 0x1034 | `max_drops` | |

### Encounter tables

`enemy_tableset` (stride 32): which enemy group appears in which round of an area.

| Offset | Field |
|---|---|
| 0x00 | `id` |
| 0x04 | `quest_area_id` |
| 0x08 | `round` (a specific round, or **-1 = every round without its own row**) |
| 0x0C | `enemy_group_id` |
| 0x10 | `rank_min` |
| 0x14 | `rank_max` |
| 0x18 | `pop_up_min` (enemies spawned, minimum) |
| 0x1C | `pop_up_max` |

`enemy_encount` (stride 32): members of an enemy group.

| Offset | Field |
|---|---|
| 0x00 | `id` |
| 0x04 | `enemy_group_id` |
| 0x08 | `enemy_id` (→ enemy_param) |
| 0x0C | `pop_up_rate` (appearance weight) |
| 0x10 | `correction` (weight correction) |
| 0x14 | `boss_flag` (0 normal, 1 boss, 2 rank taken from battle state) |
| 0x18 | `drop_item_id` |
| 0x1C | `item_drop_rate` (-1 = use the enemy's drop table) |

`enemy_param` (stride 168): one enemy.

| Offset | Field |
|---|---|
| 0x00 | `id` |
| 0x04 | `enemy_name` (string, 0x62) |
| 0x68 | `rank_type` (→ enemy_rankset) |
| 0x6C | `size` |
| 0x70 | `hp` |
| 0x74 | `turn_num` (attack interval in turns) |
| 0x78 | `attack` |
| 0x7C | `defence` |
| 0x80 | `attribute` |
| 0x84 | `exp` (base) |
| 0x88 | `bit` (base Bits) |
| 0x8C | `image_id` (sprite) |
| 0x90 | `drop_card_id` |
| 0x94 | `card_drop_rate` |
| 0x98 | `drop_table_id` (→ enemy_dropset) |
| 0x9C | `item_drop_rate` |
| 0xA0 | `drop_orbe_id` |
| 0xA4 | `orbe_drop_rate` |

`enemy_rankset` (stride 44): multipliers per rank type and rank (1..10).
Rates use a **base of 10000** (10000 = ×1.0).

| Offset | Field |
|---|---|
| 0x00 | `id` |
| 0x04 | `rank_type` |
| 0x08 | `rank` |
| 0x0C | `hp_inc_rate` |
| 0x10 | `turn_inc_count` |
| 0x14 | `attack_inc_rate` |
| 0x18 | `defence_inc_rate` |
| 0x1C | `exp_inc_rate` |
| 0x20 | `bit_inc_rate` |
| 0x24 | `card_drop_inc_rate` |
| 0x28 | `drop_item_inc_rate` |

`enemy_dropset` (stride 44): `id` at 0x00, then `rank1_item_id` … `rank10_item_id`
at 0x04 … 0x28 (the item dropped depends on the enemy's rank).

`quest_reward` (stride 24): first-clear rewards of an area.

| Offset | Field |
|---|---|
| 0x00 | `id` |
| 0x04 | `quest_area_id` |
| 0x08 | `type` (1 = card, 2 = item) |
| 0x0C | `object_id` |
| 0x10 | `count` |
| 0x14 | `delete_flg` (u8) |

### `card` (card master, stride 0x990 = 2448)

| Offset | Field | Confidence |
|---|---|---|
| 0x004 | `card_id` | CONFIRMED |
| 0x020 | `type` (growth family) | CONFIRMED |
| 0x024 | `rarity` (index 0..9; stars = rarity + 1) | CONFIRMED |
| 0x028 | `cost` | CONFIRMED |
| 0x02C | `leader_capacity` | CONFIRMED |
| 0x030 | `level_limit` (max level) | CONFIRMED |
| 0x034–0x044 | HP, ATK, SPD, DEF, SUP at level 1 | CONFIRMED |
| 0x068–0x078 | five `attribute` (affinity) slots; values > 4 → -1 (none) | CONFIRMED |
| 0x080 | `image_id` | CONFIRMED |
| 0x0CC | `name` (string) | CONFIRMED |
| 0x98C | `exceed_limit_growth_type` (growth family above the level limit) | CONFIRMED |

Other card-related tables (JSON in the analysed build): `card_exp`
(`id, level, exp, total_exp`), `card_levelup_param`
(`id, type, route, zone, hp, attack, speed, defense, support`), skill tables,
evolution recipes, `card_exceed_limit` (limit-break materials).

## JSON tables

JSON masters have the shape `{"table": [ {row}, {row}, ... ]}`. Each row is an
object keyed by field name. The reader looks fields up **by key**; a missing key gets
the reader's default (usually `0`, sometimes `-1`), which is a frequent source of
silent misbehaviour.

Notable: in `quest_area.json` the key `scenario_id` is stored at the in-memory
offset the difficulty grouping uses, so it is not just a label (CONFIRMED).

## Dates

- Format: `YYYY-MM-DD HH:MM:SS` (UTC is the safe convention).
- The client converts dates to **32-bit signed Unix seconds**. Anything after
  2038-01-19 overflows. Use `2037-12-31 00:00:00` as the "forever" end date.
  A date such as `2099-...` breaks the window check (CONFIRMED, observed).
- A quest or area is active when `start ≤ now < end`.
- `now` is the **server time** sent by the server. If the server never sends a time,
  `now` is 0 (1970), and every window that starts later is closed (CONFIRMED).
