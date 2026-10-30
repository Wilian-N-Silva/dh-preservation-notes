# 01 · Architecture

## The split between client and server

Digimon Heroes! is a **thin-server, fat-client** game. The native library of the
client (a C++ engine exposed to Android through JNI) runs the whole battle and
most game math locally. The server is mostly an **authority for the account**:
inventory, currencies, progress, and the master data the client downloads.

| Concern | Who does it | Confidence |
|---|---|---|
| Battle simulation (card queue, chains, damage, enemy turns) | Client | CONFIRMED |
| Enemy encounter generation and enemy **rank** roll | Client (random per enemy) | CONFIRMED |
| EXP / Bits / drops earned in a battle | Client computes and **reports** them; server accepts or rejects | CONFIRMED |
| Digifuse (fusion) preview: EXP gain and cost | Client computes; server returns the new card state | CONFIRMED |
| Stat growth per level | Client can recompute; server sends stored stats | CONFIRMED (client helper) |
| Inventory, currencies, capacity, team | Server (client mirrors) | CONFIRMED |
| Egg (gasha) results | Server | CONFIRMED |
| Which quests/areas exist and when | Master data downloaded from the server | CONFIRMED |
| Server time ("now") used for date windows | Server (in login/home responses) | CONFIRMED |

Consequence for a reconstruction: the server cannot uniquely reproduce the actual battle from
the IDs and totals alone; it can **check** what the client reports (enemies fought, EXP, Bits,
drops). The client never sends
a random seed; it sends the enemy IDs it generated and the totals it computed. How the
original server checked them is UNKNOWN; a workable reconstruction is described in
[07](07-progression-formulas.md) §1 (RECONSTRUCTED).

## How game data reaches the client

1. At startup the client asks the server for a **file list** (resource check). Each
   entry carries a file name and an MD5. Files whose MD5 differs from the local copy
   are downloaded.
2. The main data file is a single **resource container** (one per language). Inside it
   are the **master tables**: JSON tables and binary `.tbl` tables. The container is
   protected; these notes deliberately do **not** describe how to open it.
3. Images that are not inside the APK (card art, enemy sprites) are downloaded on demand
   from a resource path, by file name (for example `card_%04d.png`, `enemy_%d.png`), using
   the card's `image_id`, not its card ID.
4. The client parses the masters into in-memory tables and joins them by ID
   (see [05](05-data-formats.md)).

Important: some UI paths **do not fall back** gracefully when a master row they join
to is missing. A missing row can freeze or crash the client instead of showing an
error (see the event notes in [09](09-calendar-and-events.md)).

## Namespaces that look alike but are different

- **card_id** (species/variant row in the card master) ≠ **image_id** (art file number)
  ≠ **user_card_id** (one owned copy in an inventory).
- **quest** (a node on the world map) ≠ **quest area** (one playable stage inside it).
- **item id** (consumables, tickets, capacitors) ≠ **card id**.

Mixing them up is the most common source of wrong conclusions.
