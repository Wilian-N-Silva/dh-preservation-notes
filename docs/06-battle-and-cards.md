# 06 · Battle and cards

The battle runs entirely on the client. This page describes the mechanics that are
qualified; open parts are marked.

## Team and shared HP

- The player fields a team of up to **five** own Digimon plus one **helper** (another
  player's leader card), six "icons" in battle. One own card is the leader, whose
  **leader skill** applies to the team.
- The team has **one shared HP pool**: the sum of the HP of the team members (plus slot/rune
  bonuses). Enemy attacks reduce this pool (CONFIRMED).
- Battle HP is an integer: damage is truncated, there is a minimum of 1 damage, HP never
  goes below 0, and displayed values are capped at 9999 (CONFIRMED).

## The card queue (attack cards)

Attacks are made with **attack cards**, not by tapping a Digimon directly.

- The board shows a queue of **six attack cards**. Each card has a **kind** (colour/type,
  six kinds) and a **number** 1..9; one special kind always has number 0 (a wildcard-like
  card) (CONFIRMED).
- On each turn the player picks **three** cards one after another. When a card is picked,
  the queue shifts and a **new random card** is generated at the tail. It is a generator,
  not a pre-shuffled deck (CONFIRMED). Tutorial steps and some skills override the generator.
- Which team members attack depends on the kinds picked and the members' **affinities**.

## Chains

- Three picks of the same kind form a **Chain** (an automatic bonus attack sequence).
- An **Ex-Chain** is a stronger chain variant with its own counter (CONFIRMED that both exist
  and are counted separately; exact multipliers UNKNOWN for this game).
- Number sequences (for example 3-4-5) also matter in the original design (INFERRED from
  in-game help; exact rules UNKNOWN in these notes).

## Affinities (attributes)

- Each card has five affinity slots (card master `+0x68..+0x78`). Each slot holds 0..4, or -1
  for none (CONFIRMED storage).
- The mapping of numbers to names (for example Dragon / Knight / Nature / Dark …) is
  INFERRED from matching known cards; do not hardcode it as fact.
- Same name does not mean same affinity: different variants of the same species can differ.

## Skills

- **Main skill**: has a cooldown counted in turns; when ready, the player activates it
  (hold to preview, press to activate). After use the counter resets (CONFIRMED).
- **Leader skill**: passive, from the leader card.
- Skill effects are dispatched by a skill **type** (dozens of types exist). Two mechanics
  verified on the original client (numbers below are synthetic):
  - *Instant attack* (one type): damage to each target = `ATK × power / 100`
    (illustration: ATK 40, power 150 → 60).
  - *Defence* (another type): reduces each incoming hit by a fixed amount for N enemy turns.
- Coefficients for all other skills are UNKNOWN in these notes.

## Enemies

- Each enemy has HP, attack, defence, an attack interval in turns (`turn_num`), an attribute
  and an image. Its **rank** (1..10) scales HP/ATK/DEF and rewards via `enemy_rankset`
  (see [05](05-data-formats.md) and [07](07-progression-formulas.md)).
- An area has `round` waves; each wave spawns `pop_up_min..pop_up_max` enemies from a group.
- Bosses are flagged in the encounter table (`boss_flag`).

## Incoming damage and defence

- Damage received is reduced by the team's defence through a **step table** (ordered
  thresholds, with a fallback to the last step). The table's historical coefficients are
  UNKNOWN (CONFIRMED structure).
- Temporary effects (buffs/debuffs) are stored as percentage fields with turn counters.

## End of battle

- Victory → results screen (EXP, Bits, drops, new cards) → the client sends **quest end**
  with the totals (see [03](03-endpoints.md), [07](07-progression-formulas.md)).
- Defeat → offer to **continue** (paid with an item/currency) or give up. Giving up sends a
  give-up request; progress in that area is lost.
