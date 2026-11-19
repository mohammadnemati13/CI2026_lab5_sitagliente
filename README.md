# CI2026_lab5 – Wumpus

Guide an agent through a dark 4x4 cave with a classifier system: grab the gold and climb out
without falling into a pit or meeting the wumpus. Everything is in `lab5.ipynb`.

## Formulation

- **Cave**: squares are `(row, column)`, from 1 to 4, with the rows counted from the bottom. The agent starts at
  (1,1), facing east.
- **Percepts**: a set of strings, never the position: `stench` (wumpus next to it), `breeze` (pit next to it),
  `glitter` (gold here), `bump` (wall), `scream` (wumpus killed), `at_start`.
- **Actions**: `fwd`, `turn_l`, `turn_r`, `grab`, `shoot`, `climb`.
- **Score**: +1000 for climbing out with the gold, −1000 for dying, −1 for every action, −10 for the arrow.

## The agent

Every step: it writes the message window, the rules choose an action, the cave executes it and answers with
the new percepts, it updates the memory.

- **Memory**: where the agent believes it is and where it is facing, the visited
  squares, the squares proved free of pits and of the wumpus, and the way back to (1,1).
- **Deduction**: in a square without breeze no neighbour has a pit, in a square without stench no neighbour has
  the wumpus. A square is safe when it has no pit and no wumpus, and it has not been visited yet.
- **Message window**: the percepts plus short messages seen from the agent's point of view (`safe_ahead`,
  `safe_left`, `way_back_right`, `home_here`, `has_gold`, `aim`, ...).
- **Rules**: 19 classifiers, each a pair of sets (condition, action). A rule matches when its condition is a subset
  of the message window. Among the matching rules the highest tier wins, ties are broken at random.

| Tier | Condition → action | Meaning |
|---|---|---|
| 50 | `{glitter} → {grab}` | the gold is here |
| 45 | `{aim} → {shoot}` | the start smells: shoot east |
| 40 | `{has_gold, home_here} → {climb}` | home with the gold: leave |
| 30 | `{has_gold, way_back_…}` | follow the way back home |
| 20 | `{safe_ahead} → {fwd}` | a safe square ahead: go |
| 10 | `{safe_left}`, `{safe_right}`, `{safe_behind}` | turn towards a safe square |
| 5 | `{way_back_…}` | nothing safe here: go back one square |
| 5 | `{home_here} → {climb}` | home with nothing safe left: give up |

## Results

| Cave | Won | Died | Gave up | Score |
|---|---|---|---|---|
| example cave (wumpus (3,1), gold (3,2), pits (1,3), (3,3), (4,4)) | yes, in 23 actions | no | no | 977 |
| 2000 random caves | 733 | 0 | 1267 | mean 353.2 |

- Random caves: every square but the start has a pit with probability 0.2, the wumpus and the gold are on two
  different free squares.
- The agent never dies: it only steps into squares proved safe, or back along the way it came.
- When nothing safe is left it goes home and climbs out without the gold. In some caves the gold cannot be reached
  at all, in others it cannot be reached without taking a risk.


## Credits

- AI assistance: All ideas, design decisions, and core implementations are my own. AI was used to refine the code, improve readability, and enhance the clarity and structure of this README.
