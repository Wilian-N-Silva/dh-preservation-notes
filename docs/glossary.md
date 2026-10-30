# Glossary

| Term | Meaning |
|---|---|
| **AP / energy** | Stamina spent to start a quest area (`use_ap`); regenerates over time |
| **Bits** | Soft currency earned in battles (`gold` in the protocol) |
| **Digimoney** | Premium currency (`coin` in the protocol) |
| **Friend points (FP)** | Currency from helpers/friends, used for the normal egg |
| **Card** | A Digimon card; `card_id` is the master row, `user_card_id` an owned copy |
| **Card book / album** | Collection record of every card ever obtained |
| **Capacity** | Maximum number of owned cards (`max_digimon_count`) |
| **Chain / Ex-Chain** | Bonus attacks triggered by picking attack cards of the same kind |
| **Digifuse / union** | Feeding cards to another card to give it EXP |
| **Evolution / evolution_union** | Turning a card into another with materials and Bits |
| **Egg / gasha** | Random card draw from a server-defined pool |
| **Helper / friend card** | Another player's leader card that joins a battle |
| **Leader skill** | Passive skill of the team's leader |
| **Level limit / limit break** | Maximum level of a card; limit breaks raise the owned card's limit |
| **Master (table)** | Static game data downloaded by the client (quests, cards, enemies…) |
| **Quest / quest area** | A world-map node / one playable stage inside it |
| **Rank (enemy)** | 1..10, rolled per enemy by the client; scales stats and rewards |
| **Round** | One wave of enemies inside a quest area |
| **serverTS / server_time** | Server clock sent to the client; "now" for every date window |
| **Tutorial step** | Server-stored tutorial progress (0, 10, 20/30, 40, 50, 60…) |

Confidence labels (CONFIRMED / INFERRED / UNKNOWN / RECONSTRUCTED) are explained in the
[README](../README.md).
