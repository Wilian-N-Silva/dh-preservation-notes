# 08 · Card distribution

Cards (Digimon) reached the player through four channels.

## 1. Battle drops

Each defeated enemy yields **at most one** drop: either **its own card**
(`enemy_param.drop_card_id`) **or** an **item** chosen by its rolled rank
(`enemy_dropset.rank{R}_item_id`), never both from the same enemy. Drop thresholds scale with the rank
(see [07](07-progression-formulas.md) §1). Higher-ranked enemies are worth more EXP/Bits
and drop more often. Dropped cards arrive at **level 1** with the base stats from the card
master. CONFIRMED.

## 2. First-clear rewards

`quest_reward` rows give a fixed reward the **first** time an area is cleared: type 1 = card,
type 2 = item, with an object ID and count. Repeat clears do not pay it again. Event areas
use the same table. CONFIRMED structure; how the original server delivered it (immediately in
the quest-end response or via the gift box) is not fully qualified (the client can show it on
the results screen).

## 3. Eggs (gasha / "Digi-Eggs")

Eggs were the main way to obtain rare cards.

### What is known (CONFIRMED / documented)

- The client's egg table is only a **storefront**: name, text, dates, currency and costs. The
  **pool** (which cards) and the **odds** came from the server and changed over time ("rotations").
- The in-game *Info* screen of an egg listed the possible Digimon ordered by stars, with **no
  percentages**.
- Star rating = card rarity index + 1 (1★ … 10★, where 10★ is the "LR" tier).
- Egg types that existed: Normal (friend points), Premium (premium currency "Digimoney"),
  ticket eggs (Rare, SP, Green SP…), limited/medal/chip eggs and **Box Gasha** eggs.
- Prices observed in gameplay footage (one period, may have changed):

| Egg | Stars | Packs |
|---|---|---|
| Premium (Digimoney) | 5★–10★ | 1 = 300; 5 = 1500 + 1 bonus; 10 = 3000 + 3 bonus |
| Normal (friend points) | 1★–8★ | 1 = 200 FP; 5 = 1000 FP + 1 bonus |
| Green SP ticket | 1★–9★ | 1 ticket = 1 egg; 5 tickets = 5 + 1 bonus |

- Community reports (official forum, 2017, paraphrased): a player described eggs as boxes that
  are eventually exhausted after a few thousand draws (not verified for every egg type); rare
  cards appeared mainly through rotations and events; one player's 28 premium eggs gave 11 cards
  at SP tier or above and no LR.

### What is lost

The exact pool of each egg in each period, the box sizes, and the per-star odds. They were
server-side and no complete record is known.

### A documented reconstruction (RECONSTRUCTED)

One reasonable rule, used in a private restoration and stated here as a *design*, not history:

1. **Pool** = every collectible card whose star rating is inside the egg's star range.
2. **Draw** = pick a star with fixed chances, then a card uniformly within that star.
3. **Star chances** chosen to roughly match the community sample (SP+ ≈ 9 %, LR ≈ 0.4 %):

| Egg | Star chances (%) |
|---|---|
| Premium 5★–10★ | 5★ 35 · 6★ 25 · 7★ 18 · 8★ 12.6 · 9★ 9 · 10★ 0.4 |
| Normal 1★–8★ | 1★ 30 · 2★ 22 · 3★ 17 · 4★ 12 · 5★ 9 · 6★ 6 · 7★ 3 · 8★ 1 |
| Green SP 1★–9★ | 1★ 20 · 2★ 16 · 3★ 14 · 4★ 13 · 5★ 12 · 6★ 10 · 7★ 8 · 8★ 5 · 9★ 2 |

4. Optional: down-weight (×0.25) cards whose evolution line climbs 5+ stars above them, echoing a
   forum remark that cards with very strong evolution lines had low drop rates.

If you design your own game: keep the pool server-side, publish your odds (players and many
app stores expect it), and prefer boxes or pity counters over unbounded randomness.

## 4. Fusion, evolution and gifts

- **Digifuse** consumes cards to give EXP to another (see [07](07-progression-formulas.md) §5).
- **Evolution** turns a card into another using materials and Bits (§6).
- **Gifts** (login bonus, events, compensation) arrive in a gift box and are collected
  explicitly; a gift can be a card, an item or currency.
