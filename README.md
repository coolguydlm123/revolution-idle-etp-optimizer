# Revolution Idle ETP Optimizer

A single-page tool for the Epsilon Tree in Revolution Idle. Pick a node, enter your total ETP and your Pent, and it finds the best split of levels along the path from that node back to ε. It also prints a checkpoint table with the best split at every N ETP.

Open `index.html` in any browser. No install and no internet connection needed.

## How the tree works (read from the game's code)

**ETP cost per level** is flat and depends only on the ring:

| Node | ETP per level |
|---|---|
| ε (center) | 10 |
| D1, D2 | 7 |
| C1 to C4 | 4 |
| B1 to B8 | 2 |
| A1 to A16 | 1 |

**Efficiency** of a node:
1. Level goes through soft caps: above 100 it becomes `100·(L/100)^0.75`, above 1,000 `1000·(x/1000)^0.6`, above 1e6 `1e6·(x/1e6)^0.5`, above 1e10 a flatter log curve.
2. Multiplied by the effect of the node one step toward ε, if that node has at least one level.
3. Multiplied by the Pent milestone 10 bonus, `1 + 0.01 × Pent` (from 10 Pent).

**Effect** is a function of efficiency `e`. Ids are heap-ordered (parent of id `i` is `(i-1)>>1`).

| Nodes | Effect |
|---|---|
| ε, D, C, B | `x = 1 + 0.01·e`, then soft caps at 2 (^0.75), 10 (^0.5), 1,000 (^0.3), 1e6 (^0.2), 1e10 (^0.1) |
| A1, A3, A5, A7, A9, A11, A13, A15 | `1 + 0.05·e` |
| A2 | `1 + 0.01·e^0.75`, caps at 1.5, 2, 3 |
| A4 | `1 + 0.01·e^0.8` |
| A6 | `1 + 0.005·e^0.6` |
| A8 | `1 + 0.01·e^0.65` |
| A10 | `1 + 0.05·e^0.55`, caps at 1.5, 2, 2.5 |
| A12 | `1 + 0.01·e^0.92`, caps at 2, 2.5 |
| A14 (Spark Luck) | efficiency caps at 100 (^0.84), 400, 900, 1600, then `100·(1+0.01x)/(100+0.01x)`, ceiling ×100 |
| A16 | `1 + 0.01·e^0.4`, caps at 1.1, 1.2, 1.5 |

The A14 chain (B7, A14) was checked against real in-game values and matched to three digits at Pent 13 and Pent 16.

## Limits
- Assumes every node is unlocked (no level 5 requirement).
- Other efficiency bonuses that exist in the game's code are not modelled.
- The optimum comes from a numeric search, so levels can be off by a few.
- No `maxLevel` is known; use the max level box if you have one.
