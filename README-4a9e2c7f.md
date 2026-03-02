# Shape Color Sort Puzzle

A custom ARC-AGI puzzle built on circular alphabetical color sorting within outline loop shapes on a 22x22 grid.


## Overview

You are given a 22x22 grid containing 4 distinct outline shapes: a diamond, a crown, an upward arrow, and a star (plus shape). Each shape is a single-pixel-wide continuous loop (like a ring or chhalla). Each shape's cells are filled with random colors, and exactly one cell per shape is GRAY (5). Your job is to figure out the output by sorting each shape's colors in circular alphabetical order starting from the gray cell.


## Color Palette

  Color       Code    Used
  blue        1       yes
  red         2       yes
  green       3       yes
  yellow      4       yes
  gray        5       yes (anchor, exactly 1 per shape)
  magenta     6       yes
  orange      7       yes
  azure       8       NO (never used)
  brown       9       yes
  black       0       background only


## The 4 Shapes

Each grid contains exactly these four outline shapes placed in different quadrants:

  Shape       Description                                             Size
  Diamond     Diamond/rhombus outline, staircase edges                9x9, 32 cells
  Crown       3 peaks connected by a bar with a wide base             6x12, 44 cells
  Arrow       Upward-pointing arrow with triangular head and shaft    8x7, 26 cells
  Star        Plus/cross shape with 4 arms                           7x9, 28 cells

Each shape is a single-pixel-wide loop. The last cell connects back to the first cell, forming a closed ring. You can walk along the shape like a snake following one continuous path.


## The Rule

There is one rule applied to each shape independently.

### Normal Alphabetical Order of Colors

  blue, brown, gray, green, magenta, orange, red, yellow

### Circular Alphabetical Order (starting from gray)

Take the normal alphabetical order and cut it at gray. Start from gray and continue forward, wrapping around:

  gray, green, magenta, orange, red, yellow, blue, brown, (back to gray)

Think of all 8 colors arranged in a circle in alphabetical order. Put your finger on gray and go clockwise. That is the circular order.

### How to Apply

For each shape:

1. Find the gray cell in the shape. That is your starting point.
2. Count all the colors in the shape. For example: 1 gray, 1 green, 2 magenta, 6 orange, 8 red, 8 blue, 6 brown.
3. Sort them in circular alphabetical order: gray first, then green, then magenta, then orange, then red, then yellow, then blue, then brown. The count of each color stays the same as input. No new color is added, no color is removed. Only positions change.
4. Stand at the gray cell. Start walking clockwise along the loop.
5. Fill each cell with the next sorted color as you walk.
6. When you complete the full loop and arrive back behind gray, you are done.

Background (black/0) cells never change. Only the shape cells get rearranged.


## Worked Example (Diamond from Train 0)

### Input

  Row 1:  . . . B O . . . .
  Row 2:  . . B M O O . . .
  Row 3:  . O W . . R M . .
  Row 4:  B g . . . . W B .
  Row 5:  G O . . . . . W W
  Row 6:  . O B . . . . R B
  Row 7:  . . R R . . R R .
  Row 8:  . . . W W R R . .
  Row 9:  . . . . B B . . .

  g = gray, B = blue, R = red, G = green, M = magenta, O = orange, W = brown

### Step 1: Find Gray

Gray is at Row 4, Col 1. That is the starting point.

### Step 2: Count Colors

  gray    x 1
  green   x 1
  magenta x 2
  orange  x 6
  red     x 8
  blue    x 8
  brown   x 6
  Total   = 32 cells

### Step 3: Sort in Circular Order

  g, G, M, M, O, O, O, O, O, O, R, R, R, R, R, R, R, R, B, B, B, B, B, B, B, B, W, W, W, W, W, W

Same counts as input, just arranged in circular alphabetical order.

### Step 4: Walk the Loop from Gray, Fill Sorted Colors

  Step  1: (4,1) START   fill GRAY
  Step  2: (3,1) upar    fill green
  Step  3: (3,2) right   fill magenta
  Step  4: (2,2) upar    fill magenta
  Step  5: (2,3) right   fill orange
  Step  6: (1,3) upar    fill orange
  Step  7: (1,4) right   fill orange
  Step  8: (2,4) neeche  fill orange
  Step  9: (2,5) right   fill orange
  Step 10: (3,5) neeche  fill orange
  Step 11: (3,6) right   fill red
  Step 12: (4,6) neeche  fill red
  Step 13: (4,7) right   fill red
  Step 14: (5,7) neeche  fill red
  Step 15: (5,8) right   fill red
  Step 16: (6,8) neeche  fill red
  Step 17: (6,7) left    fill red
  Step 18: (7,7) neeche  fill red
  Step 19: (7,6) left    fill blue
  Step 20: (8,6) neeche  fill blue
  Step 21: (8,5) left    fill blue
  Step 22: (9,5) neeche  fill blue
  Step 23: (9,4) left    fill blue
  Step 24: (8,4) upar    fill blue
  Step 25: (8,3) left    fill blue
  Step 26: (7,3) upar    fill blue
  Step 27: (7,2) left    fill brown
  Step 28: (6,2) upar    fill brown
  Step 29: (6,1) left    fill brown
  Step 30: (5,1) upar    fill brown
  Step 31: (5,0) left    fill brown
  Step 32: (4,0) upar    fill brown    DONE, full loop complete

### Output

  Row 1:  . . . O O . . . .
  Row 2:  . . M O O O . . .
  Row 3:  . G M . . O R . .
  Row 4:  W g . . . . R R .
  Row 5:  W W . . . . . R R
  Row 6:  . W W . . . . R R
  Row 7:  . . W B . . B R .
  Row 8:  . . . B B B B . .
  Row 9:  . . . . B B . . .

Notice: gray stays in its place. Walking clockwise from gray, you see the colors flowing in sorted order: g, G, MM, OOOOOO, RRRRRRRR, BBBBBBBB, WWWWWW. Same colors as input, just rearranged along the loop.


## Why Circular and Not Normal Alphabetical?

Normal alphabetical order is: blue, brown, gray, green, magenta, orange, red, yellow.

But gray is the anchor/starting point. So we cut the circle at gray:

  normal:   ... blue, brown, GRAY, green, magenta, orange, red, yellow, blue, brown ...
                               ^
                          cut here

  result:   GRAY, green, magenta, orange, red, yellow, blue, brown

Blue and brown come last because they are alphabetically before gray. Since we start from gray and go forward, they wrap around to the end.

Same as a clock: if you start counting from 6, you get 6,7,8,9,10,11,12,1,2,3,4,5. The numbers 1-5 are not gone, they just come at the end because you started from 6.


## Key Points

1. Each shape is an independent loop. Solve each shape separately.
2. Gray cell is the anchor. Find it in each shape.
3. Colors are not added or removed. Only rearranged.
4. The count of each color in output is the same as input.
5. Walk clockwise along the loop starting from gray.
6. Azure/cyan (8) is never used anywhere.
7. Background (0) is never changed.


## How to Play

1. Open data/training/shape_color_sort.json in the ARC testing interface or any compatible viewer.
2. Study the 3 training pairs (input to output). Each has 4 shapes with the same rule.
3. Solve the test input by applying the rule to each shape.
4. Build your answer in the grid editor and submit.


## File Locations

  ARC-AGI-2/
    data/
      training/
        shape_color_sort.json           the puzzle file
      evaluation/
        shape_color_sort.json           copy for evaluation
      prince-training/
        shape_color_sort.json           local copy
        generate_shape_color_sort.py    generator script
        README-shape_color_sort.md      this file


## Regenerating the Puzzle

  cd ARC-AGI-2/data/prince-training
  python3 generate_shape_color_sort.py

This will regenerate the JSON files and print visual grid representations for inspection.