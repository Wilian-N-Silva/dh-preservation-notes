# 07 · Progression formulas

All formulas below come from reading the native client (Android 1.0.52) and, where
noted, from observed behaviour. Integer division truncates toward zero.
`f32(x)` means the value is computed in **32-bit float** (single precision), then
truncated to an integer; the float rounding matters for exact matches.

## 1. Battle rewards per enemy (EXP, Bits, drops)

Each enemy in an area has a **rank** (1..10), rolled by the client per enemy from
the area's `[rank_min, rank_max]` (uniform, CONFIRMED). The rank selects one
`enemy_rankset` row (by the enemy's `rank_type` and the rolled rank). Rates are on
a base of 10000.

```text
EXP   = trunc( f32( enemy.exp * f32(rankset.exp_inc_rate / 10000) * M ) )
Bits  = trunc( f32( enemy.bit * f32(rankset.bit_inc_rate / 10000) * M ) )

card_threshold = trunc( f32( f32(enemy.card_drop_rate / 100) * f32(rankset.card_drop_inc_rate / 10000) * M ) )
item_threshold = trunc( f32( f32(enemy.item_drop_rate / 100) * f32(rankset.drop_item_inc_rate / 10000) * M ) )

card dropped  -> enemy.drop_card_id                   (if drop_card_id > 0 and the roll passes card_threshold)
item dropped  -> enemy_dropset[enemy.drop_table_id].rank{R}_item_id   (R = rolled rank)
```

- `M` is an event modifier (1.0 when no event bonus table applies). CONFIRMED for M = 1;
  the event bonus tables are UNKNOWN.
- When an encounter row has `item_drop_rate = -1`, the enemy's drop table is used;
  otherwise the encounter overrides it (the override path is not fully qualified).
- `boss_flag = 2` takes the rank from battle state instead of the roll (UNKNOWN details).
- HP/ATK/DEF scaling by `hp_inc_rate` etc. uses the same base-10000 convention (INFERRED).

The client reports to the server, at quest end: the list of enemy IDs fought (in round
order), the total EXP, total Bits, and the dropped card/item IDs. The server only sees
IDs and totals, never the ranks. A faithful server accepts the report when **some**
assignment of allowed ranks (the same rank for an enemy's EXP, Bits and drops)
reproduces the totals exactly.

## 2. Encounters (waves)

For an area with `round = N`, for each round `r` in 1..N the client uses the
`enemy_tableset` row with `round == r`, or the row with `round == -1` if there is none.
That row names an enemy group and how many enemies spawn (`pop_up_min..pop_up_max`);
members are picked from the group by `pop_up_rate` with `correction` applied (CONFIRMED
structure; exact weighting INFERRED). The request that starts a quest carries the
**flat** list of generated enemy IDs, with no round separators.

## 3. Card EXP and level

`card_exp` rows: `level, exp (to next), total_exp (cumulative)`.

```text
level(EXP) = max { L : total_exp[L] <= EXP }, capped by the card's effective level limit
```

The effective level limit is the owned card's own `level_limit` unless it is -1, in which
case the card master's `level_limit` is used (limit breaks raise the owned value).

## 4. Stat growth per level

Stats grow by **integer additions** in three zones of the level range (CONFIRMED):

```text
M = card.level_limit ; q = trunc(M / 3)
zone(i) = 0 if i <= q ; 1 if q < i <= 2q ; 2 if i > 2q

S = level-1 base stats from the card master
family = card_levelup_param rows for (card.type, route)  (fallback route 0)
for i = 2 .. min(L, M):   S += family[zone(i)]           (HP, ATK, SPD, DEF, SUP)
S += rarity_bonus[card.rarity]   (added once to all five stats)
if L > M:  same loop with card.exceed_limit_growth_type for levels M+1 .. L
```

`rarity_bonus` by rarity index 0..9: `0,0,0,1,1,2,2,2,2,2` (CONFIRMED values in the client).
No interpolation, no floats. A missing zone row aborts the computation.

## 5. Digifuse (fusion)

Feeding 1..5 material cards to a receiver card (CONFIRMED, native preview):

```text
cost (Bits) = 100 × receiver.level × number_of_materials

base   = Σ_materials (100 × material.leader_capacity + 100 × material.level)
weight = 10 + Σ_materials W[material.rarity]      W = [1,2,3,4,5,6,6,6,6,6]
same   = number of materials whose attribute equals the receiver's attribute
EXP_gain = min(2^31-1, base × weight × T[same] / 100)     T = [10,15,16,17,18,19]

new EXP = receiver.exp + EXP_gain ; new level = level(new EXP) (section 3)
stats  : grow incrementally from the old level to the new level (section 4)
```

Client gates: the receiver must be below its effective level limit; Bits ≥ cost.
The server reply also carries a `great_success_flg` (a "great success" bonus existed;
its probability and effect are UNKNOWN).

Observed check: a preview of 2520 EXP for a given material set matched this formula
exactly on the original client.

## 6. Evolution (Digivolve)

An evolution recipe names the source card, the target card, required materials
(by attribute and minimum level, plus items) and a Bits price; the preview price uses
the real levels involved. The result is a new card row at level 1 of the target card
(CONFIRMED structure; full recipe table semantics INFERRED).

## 7. Inventory capacity

- The account has a card capacity. Observed values: 7 during the tutorial, 12 after,
  then +5 per "Hard Drive" item, up to 250 (the item caption states the cap). CONFIRMED
  for the observed steps.
- Exceeding capacity blocks battle start and gasha; the client offers a "not enough
  space" menu (sell / upgrade).

## 8. Energy (AP)

- Starting a quest area costs `quest_area.use_ap`.
- AP regenerates 1 point per fixed interval; **180 seconds** is the documented player-facing
  rule (community FAQ), not a verified server constant (INFERRED).
- The client shows "seconds until full", not a timestamp.
- Whether giving up refunds AP is UNKNOWN; a reconstruction can choose no refund.

## 9. What is still unknown

Damage coefficients for every skill, chain multipliers, the great-success rate, the
exact egg probabilities (see [08](08-card-distribution.md)), player-level EXP curve beyond the
first levels, and event reward modifiers.
