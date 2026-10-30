# dh-preservation-notes

Research notes on the client–server protocol and game rules of **Digimon Heroes!**
(Bandai Namco, mobile card game, 2013–2017, servers shut down in 2017), written
for preservation and for anyone who wants to understand how the game worked or
design their own card game inspired by it.

**This repository contains documentation only.** No game files, no assets, no
images, no binaries, no master data dumps, no decryption tools or keys, no
network captures and no server code. Examples are synthetic (invented values).
Nothing here lets anyone play the game; it describes how it worked.

## Contents

| Doc | What it covers |
|---|---|
| [01 Architecture](docs/01-architecture.md) | What the client computes, what the server decides, how data reaches the client |
| [02 Protocol](docs/02-protocol.md) | HTTP envelope, result codes, response conventions |
| [03 Endpoints](docs/03-endpoints.md) | API catalog (apiID, script/action, key fields) |
| [04 Boot, tutorial and world map](docs/04-boot-tutorial-worldmap.md) | Login → tutorial steps → first world map, and the traps on the way |
| [05 Data formats](docs/05-data-formats.md) | Master table layouts (`.tbl` records and JSON tables), dates, IDs |
| [06 Battle and cards](docs/06-battle-and-cards.md) | Card queue and chains, affinities, encounters, enemy ranks |
| [07 Progression formulas](docs/07-progression-formulas.md) | Rewards, card EXP and levels, stat growth, Digifuse, capacity, energy |
| [08 Card distribution (eggs)](docs/08-card-distribution.md) | How cards are distributed: drops, first-clear rewards, eggs/gasha |
| [09 Calendar and events](docs/09-calendar-and-events.md) | Date windows, the int32 date limit, event quests and difficulty groups |
| [10 Example tables](docs/10-example-tables.md) | Synthetic example rows for every table described here |
| [Glossary](docs/glossary.md) | Terms and labels used in these notes |

## How to read the confidence labels

Every non-trivial claim is tagged:

- **CONFIRMED**: verified directly (native client code reading and/or observed behaviour of the original client).
- **INFERRED**: strongly supported but not directly proven (for example, a field meaning deduced from usage).
- **UNKNOWN**: open question; do not treat as fact.
- **RECONSTRUCTED**: a rule chosen to make a working reconstruction when the historical value is lost
  (for example, egg star chances). It is plausible, documented and *not* historical truth.

Version notes: the client analysed is the global Android build **1.0.52** (app_ver 2.220).
Some table layouts were read from an iOS 1.0.13 build of the same game; where Android/iOS
equivalence is not proven, the text says so.

## Scope and policy

- Preservation and research. Not affiliated with or endorsed by Bandai Namco or the Digimon
  rights holders. "Digimon" and related names are trademarks of their respective owners.
- No support, no Discord, no requests. Issues may be closed without reply; the notes are
  maintained when time allows. Corrections backed by evidence are welcome.
- No real-money features are documented beyond describing that they existed.
- If you build something from these notes, please make it your own game: original art,
  original names and your own data. Do not redistribute the original client or its files.
