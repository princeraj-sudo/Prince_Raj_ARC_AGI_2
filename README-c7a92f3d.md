# Puzzle c7a92f3d

## Overview

A 22x22 grid divided into 4 quadrants by a maroon (9) plus-shaped divider at row 11 and column 11. Each quadrant contains a large pixel-font letter (7 rows x 5 cols) filled with colored cells. The rest of the grid outside the letter shapes is black (0).

There are 4 letters in the top half (rows 0-10) and 4 letters in the bottom half (rows 12-21), placed at fixed column positions. Each example uses a different set of 8 letters.

Letter positions (column start offsets): 0, 6, 12, 17

- Top half letters: rows 2-8
- Bottom half letters: rows 13-19

Each quadrant has a dominant fill color for its letters:

| Quadrant | Position | Letter Color | Corner Tag Cell | Corner Tag Value |
|----------|----------|-------------|-----------------|-----------------|
| Q1 (top-left) | rows 0-10, cols 0-10 | blue (1) | (0, 10) | 1 |
| Q2 (top-right) | rows 0-10, cols 12-21 | green (3) | (0, 21) | 3 |
| Q3 (bottom-left) | rows 12-21, cols 0-10 | orange (7) | (12, 10) | 7 |
| Q4 (bottom-right) | rows 12-21, cols 12-21 | red (2) | (12, 21) | 2 |

The four corner tag cells are always present and unchanged between input and output.

25 gray (5) cells are scattered inside the letter shapes (never adjacent to each other). Two yellow (4) triplets (3 consecutive cells) are placed inside the shapes. Two special cells exist at (10,10) and (12,12) that are not gray.


## Rules

There are 4 rules that transform the input into the output. They work together in a specific order.

### Rule 1: Alphabetical Letter Rearrangement

The top half and bottom half each contain 4 letters in some shuffled order. In the output, each half has its 4 letters rearranged into alphabetical order (A-Z, left to right).

The internal content of each letter (colors, gray cells, yellow cells) moves with the letter to its new position. The letter shape and its pixel contents are preserved; only the position changes.

Example (Train 0):

    Top input:  N L O P  -->  Top output:  L N O P
    Bot input:  R I A G  -->  Bot output:  A G I R

Example (Train 1):

    Top input:  T E D B  -->  Top output:  B D E T
    Bot input:  K A H C  -->  Bot output:  A C H K

Example (Train 2):

    Top input:  P L O F  -->  Top output:  F L O P
    Bot input:  S R I K  -->  Bot output:  I K R S

Test 0:

    Top input:  W A E V  -->  Top output:  A E V W
    Bot input:  D U T S  -->  Bot output:  D S T U

### Rule 2: Gray Cell Replacement

After the letters have been rearranged, every gray (5) cell is replaced by the highest color value among its cardinal neighbors (up, down, left, right).

- Only valid neighbors are counted: skip black (0), gray (5), and maroon (9).
- The neighbor with the numerically largest color value wins.
- The gray cell becomes that max color.
- No two gray cells are ever adjacent to each other.
- Every gray cell has at least one valid colored neighbor.

Gray distribution across quadrants:

| Quadrant | Gray Count |
|----------|-----------|
| Q1 | 9 |
| Q2 | 4 |
| Q3 | 5 |
| Q4 | 7 |
| Total | 25 |

All 25 gray cells become non-gray in the output. Zero gray cells remain.

### Rule 3: Special Pair Comparison and Next-Color Shift

Cells (10,10) and (12,12) are a special pair. They are not gray. They contain regular colors. Their transformation:

1. Compare the color values at (10,10) and (12,12).
2. The cell with the higher value gets replaced with next_color(value).
3. The cell with the lower value stays unchanged.
4. If equal, both get next_color(value).

The next_color cycle (skipping 0 and 5):

    1 -> 2 -> 3 -> 4 -> 6 -> 7 -> 8 -> 9 -> 1 (wraps)

| Example | (10,10) | (12,12) | Result |
|---------|---------|---------|--------|
| Train 0 | 8 (azure) | 2 (red) | 8 -> 9, 2 stays |
| Train 1 | 6 (magenta) | 3 (green) | 6 -> 7, 3 stays |
| Train 2 | 7 (orange) | 2 (red) | 7 -> 8, 2 stays |
| Test 0 | 8 (azure) | 4 (yellow) | 8 -> 9, 4 stays |

### Rule 4: Yellow Triplet Flood to Azure

After Rules 1-3 have been applied, scan the entire grid for groups of 3 or more consecutive yellow (4) cells in a horizontal or vertical line. Every such cell is replaced with azure (8).

This includes:

- Yellow triplets that were already present in the input (and moved with their letter in Rule 1).
- New yellow triplets that formed because a gray cell was replaced by yellow (4) via Rule 2, completing a run of 3 adjacent yellows.

The second case is the chain reaction: Rule 2 can create yellow cells that trigger Rule 4. This interaction is the hardest part of the puzzle to notice.

In every example, at least one gray cell is engineered so that its replacement by Rule 2 produces a yellow value that completes a new triplet caught by Rule 4.


## Rule Application Order

    Input Grid
        |
        v
    Rule 1: Rearrange letters alphabetically (top and bottom halves separately)
        |
        v
    Rule 2: Replace gray cells with max cardinal neighbor
        |
        v
    Rule 3: Special pair at (10,10) vs (12,12), bigger gets next_color
        |
        v
    Rule 4: Convert 3+ consecutive yellow cells (h/v) to azure
        |
        v
    Output Grid


## Grid Layout

    col 0          col 10  col 11  col 12          col 21
     |                |       |       |                |
     L1 (5 cols)  L2 (5 cols) | L3 (5 cols)  L4 (5 cols)
                              |
    row 0   [corner Q1]      9  [                corner Q2]
    row 1                     9
    row 2   +--- letter ---+  9  +--- letter ---+--- letter ---+
    row 3   | shapes rows  |  9  | shapes rows  | shapes rows  |
    ...     | 2 through 8  |  9  | 2 through 8  | 2 through 8  |
    row 8   +--------------+  9  +--------------+--------------+
    row 9                     9
    row 10            (10,10) 9
    row 11  9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9
    row 12  [corner Q3]      9  (12,12)          [corner Q4]
    row 13  +--- letter ---+  9  +--- letter ---+--- letter ---+
    ...     | shapes rows  |  9  | shapes rows  | shapes rows  |
    row 19  +--------------+  9  +--------------+--------------+
    row 20                    9
    row 21                    9

    Corner tags: (0,10)=1  (0,21)=3  (12,10)=7  (12,21)=2
    Special pair: (10,10) and (12,12)
    Letters are inside shapes only; everything outside is black.


## Training and Test Examples

| # | Top Letters (Input) | Top Letters (Output) | Bot Letters (Input) | Bot Letters (Output) |
|---|--------------------|--------------------|--------------------|--------------------|
| Train 0 | N L O P | L N O P | R I A G | A G I R |
| Train 1 | T E D B | B D E T | K A H C | A C H K |
| Train 2 | P L O F | F L O P | S R I K | I K R S |
| Test 0 | W A E V | A E V W | D U T S | D S T U |

| # | Gray | Special Pair | Yellow Triplet Cells (Input) | Yellow Triplets (Output) |
|---|------|-------------|-----------------------------|-----------------------|
| Train 0 | 25 to 0 | (10,10)=8 to 9, (12,12)=2 to 2 | 6 | 0 |
| Train 1 | 25 to 0 | (10,10)=6 to 7, (12,12)=3 to 3 | 6 | 0 |
| Train 2 | 25 to 0 | (10,10)=7 to 8, (12,12)=2 to 2 | 6 | 0 |
| Test 0 | 25 to 0 | (10,10)=8 to 9, (12,12)=4 to 4 | 6 | 0 |


## Why This Puzzle Is Difficult

1. Large grid (22x22) with 484 cells. Small changes are hard to spot visually.
2. Four distinct rules must be discovered and applied in the correct sequence.
3. Letter rearrangement is the most visible transformation but requires recognizing the pixel-font letter shapes and noticing that their order changed to alphabetical.
4. Gray cells are scattered inside the letter shapes and blend in. The solver must figure out the max-cardinal-neighbor replacement rule.
5. The special pair at (10,10) and (12,12) is subtle. Only 2 cells out of 484 follow a different rule (comparison + next-color shift). Easy to overlook.
6. The yellow triplet to azure conversion seems simple when triplets already exist in the input. But the chain reaction (gray replaced by yellow, forming a new triplet, which then becomes azure) requires understanding that Rule 4 runs after Rule 2.
7. Each training example uses completely different letters, so the solver cannot memorize positions. They must understand the alphabetical sorting principle.
8. The letter shapes carry their internal content (colors, grays, yellows) when they move. This means the solver cannot just redraw the shapes; they must track cell-level content.
9. The corner tag cells at (0,10), (0,21), (12,10), (12,21) are a subtle clue about quadrant identity but do not change between input and output.
10. The maroon divider must be identified as structural, not as a participant in color rules.


## Categories Used

| Category | How It Appears |
|----------|---------------|
| Object Detection | Identifying letter shapes, divider, corner tags, gray cells, yellow triplets, special pair |
| Spatial Reasoning | Letter positions, alphabetical reordering, content moving with letters |
| Topology | Cardinal neighbor analysis for gray cell replacement |
| Color Mapping | Gray to max neighbor (Rule 2), next_color cycle (Rule 3), yellow to azure (Rule 4) |
| Logic and Conditional | Special pair comparison (if bigger then shift, else keep), triplet threshold |
| Pattern Recognition | Recognizing pixel-font letters, detecting alphabetical sorting pattern |
| Sequential Reasoning | Rules must be applied in order: sort, then gray replace, then special pair, then yellow flood |
| Counting | Gray distribution per quadrant, consecutive yellow cell counting |


## Estimated Solve Time: 15 to 25 minutes

1. Notice the maroon divider creating two halves: about 1 minute
2. Recognize the pixel-font letter shapes in each half: about 2 minutes
3. Notice letters are rearranged in alphabetical order in the output: about 3 minutes
4. Spot that gray cells disappear and figure out max-neighbor replacement: about 4 minutes
5. Notice the special pair at (10,10) and (12,12) behaves differently: about 3 minutes
6. Decode the comparison and next-color shift for the special pair: about 3 minutes
7. Notice yellow triplets become azure in the output: about 2 minutes
8. Realize gray replacement can create new yellow triplets that also become azure: about 2 minutes
9. Verify rule ordering across all training examples: about 2 minutes


## File Info

| Aspect | Details |
|--------|---------|
| Puzzle ID | c7a92f3d.json |
| Grid Size | 22x22 (all examples) |
| Total Cells | 484 per grid |
| Train Examples | 3 |
| Test Examples | 1 |
| Rules | 4 (interacting, ordered) |
| Gray Cells per Example | 25 (9+4+5+7 across quadrants) |
| Special Pair | (10,10) and (12,12) |
| Yellow Triplets per Example | 2 direct + 1 emergent (minimum) |
| Gray in Output | 0 (all replaced) |
| Yellow Triplets in Output | 0 (all converted to azure) |
| Letters per Example | 8 unique (4 top, 4 bottom), different across examples |
| Validation Status | All tests pass |