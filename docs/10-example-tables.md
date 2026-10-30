# 10 · Example tables (synthetic)

All values below are **invented** for illustration. They show shapes and how tables join,
not the game's real data.

## A story quest with one area

`quest`

| id | manage_id | type | background_image_id | world_map_page | name | start_date | end_date |
|---|---|---|---|---|---|---|---|
| 1001 | 5001 | 1 | 2 | 1 | Example Plains | 2014-01-01 00:00:00 | 2037-12-31 00:00:00 |

`quest_area`

| id | quest_id | round | use_ap | scenario_id | before_quest_area_id | background_image_id | name |
|---|---|---|---|---|---|---|---|
| 5001 | 1001 | 3 | 5 | 0 | 0 | 20301 | The First Path |
| 5002 | 1001 | 4 | 6 | 0 | 5001 | 20302 | The Second Path |

## Encounters for area 5001

`enemy_tableset` (round -1 = default for rounds without their own row)

| id | quest_area_id | round | enemy_group_id | rank_min | rank_max | pop_up_min | pop_up_max |
|---|---|---|---|---|---|---|---|
| 1 | 5001 | -1 | 10 | 1 | 2 | 1 | 2 |
| 2 | 5001 | 3 | 11 | 3 | 3 | 1 | 1 |

`enemy_encount`

| id | enemy_group_id | enemy_id | pop_up_rate | correction | boss_flag | drop_item_id | item_drop_rate |
|---|---|---|---|---|---|---|---|
| 1 | 10 | 700 | 60 | 0 | 0 | 0 | -1 |
| 2 | 10 | 701 | 40 | 0 | 0 | 0 | -1 |
| 3 | 11 | 702 | 100 | 0 | 1 | 0 | -1 |

`enemy_param`

| id | enemy_name | rank_type | hp | turn_num | attack | defence | attribute | exp | bit | image_id | drop_card_id | card_drop_rate | drop_table_id | item_drop_rate |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 700 | Slimeling | 1 | 40 | 2 | 8 | 2 | 0 | 30 | 20 | 9700 | 300 | 10 | 5 | 20 |
| 701 | Rockling | 1 | 60 | 3 | 10 | 4 | 1 | 35 | 25 | 9701 | 301 | 10 | 5 | 20 |
| 702 | Big Rock | 2 | 300 | 3 | 25 | 8 | 1 | 200 | 120 | 9702 | 302 | 50 | 6 | 50 |

`enemy_rankset` (base 10000)

| id | rank_type | rank | hp_inc_rate | attack_inc_rate | exp_inc_rate | bit_inc_rate | card_drop_inc_rate | drop_item_inc_rate |
|---|---|---|---|---|---|---|---|---|
| 1 | 1 | 1 | 10000 | 10000 | 10000 | 10000 | 10000 | 10000 |
| 2 | 1 | 2 | 12000 | 11000 | 13000 | 12000 | 11000 | 11000 |

Worked example: enemy 700 at rank 2 → EXP = trunc(30 × 1.3) = 39, Bits = trunc(20 × 1.2) = 24.

`enemy_dropset`

| id | rank1_item_id | rank2_item_id | rank3_item_id | … | rank10_item_id |
|---|---|---|---|---|---|
| 5 | 8 | 8 | 9 | … | 12 |

`quest_reward` (first clear)

| id | quest_area_id | type | object_id | count |
|---|---|---|---|---|
| 1 | 5002 | 1 | 310 | 1 |

## Cards

`card` (selected fields)

| card_id | name | type | rarity | cost | leader_capacity | level_limit | hp | atk | spd | def | sup | attribute[5] | image_id |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 300 | Slimeling | 0 | 0 | 3 | 1 | 20 | 30 | 12 | 10 | 8 | 6 | 0,-1,-1,-1,-1 | 1300 |
| 310 | Brightwing | 2 | 4 | 9 | 3 | 45 | 70 | 40 | 35 | 30 | 25 | 3,1,-1,-1,-1 | 1310 |

`card_exp` (cumulative `total_exp`)

| level | exp | total_exp |
|---|---|---|
| 1 | 100 | 0 |
| 2 | 150 | 100 |
| 3 | 210 | 250 |

`card_levelup_param` (per growth `type`, `route`, `zone`)

| id | type | route | zone | hp | attack | speed | defense | support |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 0 | 3 | 2 | 1 | 1 | 1 |
| 2 | 0 | 0 | 1 | 4 | 3 | 2 | 2 | 1 |
| 3 | 0 | 0 | 2 | 5 | 4 | 2 | 3 | 2 |

Worked example (card 300, level_limit 20 → q = 6): going from level 1 to level 8 adds zone 0
for levels 2–6 (5 times) and zone 1 for levels 7–8 (2 times): HP 30 + 5×3 + 2×4 = 53, plus the
rarity bonus for rarity 0 (= 0).

## An owned card (inventory row, as the server sends it)

```json
{ "user_card_id": 900001, "card_id": 300, "level": 8, "exp": 1200, "level_limit": -1,
  "hp": 53, "attack": 28, "speed": 19, "defense": 17, "support": 13, "lock_flg": 0 }
```

`level_limit: -1` means "use the card master's limit".
