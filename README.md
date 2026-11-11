# PRG1 — Sundrop Farm

A terminal farming game in a single Python file, written for a first-year programming assignment. The player begins with $20, a 5x5 farm in Albatross Town and a $100 debt, and has twenty in-game days to clear it. Seeds are bought from a shop, planted on the grid and harvested for cash, with every action drawn against a daily energy allowance. Reaching $100 by the end of day 20 wins the game and records the resulting profit on a local scoreboard.

## Contents

| File | Description |
| --- | --- |
| `sundrop_farm.py` | Complete game, single file, standard library only |

## Running

```bash
python sundrop_farm.py
```

The source uses f-strings and no third-party imports, so Python 3.6 or later is required. `savegame.txt` is created on the first save and `scoreboard.txt` on the first win, both in the working directory; the high-score screen fails if the latter does not yet exist.

## Gameplay

The program opens on a main menu offering a new game, the high-score table or loading a save. A new game starts in town, the hub: 1 enters the seed shop, 2 enters the farm, 3 ends the day, 9 saves and exits, 0 exits without saving. The shop stocks three seed types, and the bag holds ten seeds in total across all types. On the farm the player moves one plot at a time and acts on the current plot; planting consumes one seed from the bag, and harvesting a mature crop sells it immediately at its sale value. Ending the day restores energy to 10, advances the date, reduces every planted crop's remaining growth by one and returns the player to the farmhouse.

| Key | Action | Energy |
| --- | --- | --- |
| `W` `A` `S` `D` | Move one plot, if the move stays inside the grid | 1 |
| `P` | Plant a seed on the current empty plot | 1 |
| `H` | Harvest the current plot, once growth has reached 0 | 1 |
| `R` | Return to town | 0 |

## Crops and economy

| Crop | Seed price | Days to harvest | Sale value | Profit per plot |
| --- | --- | --- | --- | --- |
| Lettuce | $2 | 2 | $3 | $1 |
| Potato | $3 | 3 | $6 | $3 |
| Cauliflower | $5 | 6 | $14 | $9 |

Per plot per day that is $0.50 for lettuce, $1.00 for potato and $1.50 for cauliflower, so the slow crop returns three times the rate of the fast one. The cost is a plot locked for six days and five times the capital committed, which is decisive while the balance sits near zero.

## Implementation

- The farm is a list of five rows of five cells; each cell is a three-slot list of `[crop code, player marker, days remaining]`. The farmhouse is the literal `['HSE', 'X', None]` at `farm[2][2]`, leaving 24 plantable plots. Position is not stored separately: `get_position` scans for the `'X'` marker each turn, and movement rewrites the marker in the old and new cells.
- Crop statistics sit in the `seeds` dictionary keyed by three-letter code, with `seed_list` fixing the shop's display order. The inventory is `game_vars['bag']`, a code-to-count map; the ten-seed cap is enforced by summing its values against the requested quantity before a purchase is priced.
- `end_day` advances `day`, resets `energy` and decrements each planted plot's counter. Counters are held as fixed-width strings (`f'{n:^3}'`) so the grid prints in alignment, and are cast back to `int` whenever they are read or decremented.
- `save_game` writes four `key: value` lines followed by a `Farm:` header and one tab-separated row per grid row, using `repr` of each cell. `load_game` reverses this by splitting on those delimiters.
- A win appends `name,profit` to `scoreboard.txt`. `show_high_scores` parses each line into a tuple, sorts descending through a key function and prints the top five.
- Menus are functions that call one another (`main_loop` into `in_town`, then `in_shop` or `in_farm`), so navigation is mutual recursion rather than a loop. Numeric input is read with `int(input())` inside `try`/`except ValueError`, then range-checked.

## Constraints and limitations

- Loading does not restore the farm. `load_game` appends rows inside the per-cell loop and reassigns a local `farm` that is never returned to the caller, so only the day, energy, money and bag values survive.
- Non-numeric input is caught and re-prompted, but the calling function then falls through to compare an unassigned `choice`, ending in a traceback.
- Planting is not gated on energy, so it succeeds at 0 energy and drives the counter negative.
- Navigation by recursion means the call stack grows for the length of a session, and a long game can approach the interpreter's recursion limit.

## Key takeaways

- Separating per-crop constants from the logic that consumes them makes the economy table a single editable dictionary rather than scattered literals.
- A parallel nested list is a workable grid representation at this scale, at the cost of string-typed counters and a scan to locate the player.
- Text serialisation by hand shows both the appeal of a readable save file and the fragility of parsing it back without a defined format.
- Layering menus as mutually recursive functions keeps control flow legible while quietly moving a loop's cost onto the call stack.
