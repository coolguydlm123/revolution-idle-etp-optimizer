# Revolution Idle ETP Optimizer

A single-page tool for the Epsilon Tree in Revolution Idle. Pick a node, enter your total ETP and your Pent / Rank / Tier, and it finds the best split of levels along the path from that node back to ε. It also prints a checkpoint table with the best split at every N ETP. A Spark rarity section shows the drop chance of every rarity for a given Spark Luck. You can set an A node ratio (with one-click presets) to spread levels across several A nodes, even on different branches. You can enter your whole tree, and for any odd A node (A1, A3 … A15) give a target multiplier to get the cheapest build and the ETP it needs. Node levels can be rounded to a step (e.g. 10) for tidy builds. A Singularity tab works out how long atoms take to reach the next Singularity.

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
3. Multiplied by these bonuses (all multiply each other):
   - Pent milestone 10: `1 + 0.01 × Pent` (from Pent 10).
   - Rank 500: `1 + 0.01 × floor(Rank / 100)` (from Rank 500).
   - Tier 200: `1 + 0.01 × floor(Tier / 50)` (from Tier 200).
   - Refine Tree Node 197: ×1.25, A14 only.
   - Refine Tree Node 198: ×1.25, A16 only.

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

The A14 chain (B7, A14) was checked against real in-game values and matched to three digits at Pent 13 and Pent 16, and again at Pent 25 + Rank 503 + Refine 197 (C4 47 / B7 200 / A14 712: C4 ×1.617, B7 ×3.717, A14 efficiency 2,659, Spark Luck ×10.537).

## Singularity tab (read from the game's code)

- **Atoms gain** = (Sing mult^1.02)^(1 + 0.005 × singularities), raised to the power of relic #69. Past each atom threshold a penalty raises the gain to 1 / divisor. Thresholds: 1e10, 1e50, 1.4e154, 1.6e256, 1.8e308, 1e500, 1e1000, 1e5000, 1e10000, 1e25000. Refine nodes 126, 127, 128, 156, 157, Pent milestone 11, Singularity tree nodes 3.1 / 3.2 / 6.2 / 8, Blue spark and A2 reduce them.
- **Relic #69** (Outdated "Nice" Gadget): level n → n+1 costs gold e((1.4e15 + 1e13·n) × soft caps), with soft caps from levels 100, 250, 400, 500, 1000, 4000 and 10000. Effect = 1 + 0.005 × level × relic boosts × max(1, 1 + (sacrifice level − 250) / 500). The tool takes level and sacrifice as automatic, so only gold is needed.
- **Speed** = Local Speed base × shop multiplier × Time Flux (1–60×). Times shown are real time.
- Sing mult is kept fixed, but in the game it grows during a run and with every Singularize, so long multi-singularity times come out too long.

## Limits
- Assumes every node is unlocked (no level 5 requirement).
- Efficiency bonuses other than the four above are not modelled.
- The optimum comes from a numeric search, so levels can be off by a few.
- No `maxLevel` is known; use the max level box if you have one.

## Spark Luck and spark rarity chances

From `ParticlesData.UpdateSparkRarityValues`. Only Spark Luck matters (Spark Mastery does not). At luck 1 the chances are F 25, E 25, D 20, C 15, C+ 10, B 4, B+ 1 (%). Between the breakpoints 1, 2, 3, 4, 5, 7, 10, 15, 20, 30, 40, 50, 60, 70, 80, 90, 100 each rarity's chance changes linearly, and the total always stays 100%.

| Luck | F | E | D | C | C+ | B | B+ | A | A+ | S | SS | SSS | 4S | ★ |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 25 | 25 | 20 | 15 | 10 | 4 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 2 | 10 | 20 | 30 | 20 | 12 | 5.5 | 2 | 0.5 | 0 | 0 | 0 | 0 | 0 | 0 |
| 3 | 5 | 10 | 25 | 25 | 16 | 9 | 4.5 | 3 | 1.6 | 0.8 | 0.1 | 0 | 0 | 0 |
| 4 | 0 | 2 | 18 | 23 | 22 | 14 | 8 | 6 | 4 | 2.4 | 0.5 | 0.1 | 0 | 0 |
| 5 | 0 | 0 | 10 | 15 | 20 | 21 | 14 | 9 | 6 | 3 | 1.5 | 0.4 | 0.1 | 0 |
| 7 | 0 | 0 | 0 | 5 | 15 | 30 | 20 | 12 | 8 | 5 | 3 | 1.5 | 0.5 | 0 |
| 10 | 0 | 0 | 0 | 0 | 0 | 25 | 30 | 20 | 12 | 6 | 4 | 2 | 1 | 0 |
| 15 | 0 | 0 | 0 | 0 | 0 | 0 | 25 | 35 | 16 | 10 | 7 | 4 | 2.8 | 0.2 |
| 20 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 20 | 40 | 16 | 12 | 6 | 5 | 1 |
| 30 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 20 | 40 | 18 | 12 | 8 | 2 |
| 40 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 30 | 30 | 24 | 12 | 4 |
| 50 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 10 | 25 | 35 | 20 | 10 |
| 60 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 20 | 42 | 26 | 12 |
| 70 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 5 | 40 | 40 | 15 |
| 80 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 30 | 50 | 20 |
| 90 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 70 | 30 |
| 100+ | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 100 |

Checked against the game at luck 1.581 and 10.537 (all shown chances match). A14's effect never reaches ×100, so luck 100 is out of reach.
