# Puzzle `d4e7a2f9.json` — Context-Dependent Shape Mutation (HARD)

## Overview

A **13×13** grid is divided into **4 rooms** by a gray (color 5) cross-shaped wall at row 6 and column 6. Each room contains 1–3 small colored shapes. The transformation is governed by **5 interacting rules** that chain together — understanding any single rule in isolation is insufficient.

```
 Room 0 (top-left)  │  Room 1 (top-right)
     rows 0–5       │      rows 0–5
     cols 0–5       │      cols 7–12
 ────────────────────┼────────────────────
 Room 2 (bottom-left)│  Room 3 (bottom-right)
     rows 7–12      │      rows 7–12
     cols 0–5       │      cols 7–12
```

---

## The 5 Rules (Layered & Interacting)

### Rule 1 — Room Detection + Shape Census
- The gray cross-wall (color 5) at row 6 and col 6 splits the grid into 4 quadrant rooms.
- Identify each **connected component** (4-connected, same non-zero non-wall color) as a distinct shape.
- Assign each shape to its room based on which quadrant its cells occupy.

### Rule 2 — Stability Classification
- Count distinct shapes per room.
- **≥ 2 shapes** in a room → ALL shapes there are **STABLE**
- **Exactly 1 shape** in a room → that shape is **UNSTABLE**

### Rule 3 — Dominant-Color Mirror (Stable Rooms Only)
This is the first tricky rule that makes stable rooms NOT trivially unchanged:
- In each stable room, find the **dominant color** = the color occupying the **most cells** (count pixels, not shapes).
- Shapes whose color **equals** the dominant color → **remain unchanged**.
- Shapes whose color **differs** from the dominant color (minority) → get **horizontally mirrored** (reflected left↔right) within the room's column bounds.
  - Mirror formula: `new_col = room_col_min + (room_col_max − old_col)`
- The minority shape keeps its original color; only its position changes.

> **Why this is hard:** A solver initially thinks "stable = no change" but then notices minority shapes silently flip position. The cell-counting for dominance vs just counting shapes is a subtle trap.

### Rule 4 — Unstable Transformation
For the single shape alone in its room:
- **If touching the wall** (any cell 4-adjacent to a gray pixel):
  - Recolor to **yellow (4)** — NO rotation. Wall blocks movement.
- **If NOT touching the wall:**
  - **Rotate 90° clockwise** around the shape's center of mass AND recolor to **yellow (4)**.

### Rule 5 — Gravity Sink (Global Post-Pass)
After all above transformations:
- Every shape that is now **color 4 (yellow)** — i.e., every unstable-transformed shape — **sinks downward** within its room as far as possible.
- It drops until hitting the room's bottom edge (row before the wall or grid boundary) or landing on top of another non-empty cell.
- Stable shapes (original or mirrored) do **NOT** sink. Only yellow shapes are affected.

> **Why this is hard:** Gravity is the final twist that makes outputs look "wrong" until you realize yellow shapes have dropped. A solver who figured out rules 1–4 will still get the wrong answer until they notice the vertical displacement pattern.

---

## Rule Interaction Chain

```
Input Grid
    │
    ▼
[R1] Detect rooms + shapes
    │
    ▼
[R2] Classify: stable (≥2) vs unstable (1)
    │
    ├─── Stable rooms ──► [R3] Count cells per color
    │                          │
    │                     Dominant stays, minority mirrors L↔R
    │
    ├─── Unstable rooms ─► [R4] Check wall adjacency
    │                          │
    │                     Touching wall? → recolor 4 only
    │                     Not touching?  → rotate 90° CW + recolor 4
    │
    ▼
[R5] All yellow (4) shapes sink to room bottom (gravity)
    │
    ▼
Output Grid
```

---

## Training Example Walkthroughs

### Example 1 — Introduces stable mirror + unstable rotate + gravity

| Room | Shapes | Count | Status | Dominant | Action |
|------|--------|-------|--------|----------|--------|
| 0 TL | Blue bar (1) 3 cells, Red dot (2) 1 cell | 2 | STABLE | Blue (3>1) | Blue stays. Red mirrors: (3,4)→(3,1) |
| 1 TR | Green L (3) 3 cells | 1 | UNSTABLE | — | Not touching wall → rotate 90° CW + yellow, then sink to rows 4–5 |
| 2 BL | Azure bar (8) 3 cells | 1 | UNSTABLE | — | Not touching wall → rotate 90° CW (horiz→vert) + yellow, then sink to rows 10–12 |
| 3 BR | Green dot (3) 1 cell, Orange bar (7) 3 cells | 2 | STABLE | Orange (3>1) | Orange stays. Green mirrors: (8,8)→(8,11) |

**Key observations for solver:**
- Room 0: the red dot moved — it wasn't just "stable = unchanged"
- Room 2: the azure bar changed orientation AND shifted to the bottom — two operations chained

### Example 2 — Wall-touching unstable + gravity on wall shapes

| Room | Shapes | Count | Status | Dominant | Action |
|------|--------|-------|--------|----------|--------|
| 0 TL | Red L (2) 3 cells, touches wall | 1 | UNSTABLE | — | Touches wall → recolor 4 only, then sink to rows 4–5 |
| 1 TR | Blue vert bar (1) 3 cells | 1 | UNSTABLE | — | Not touching wall → rotate (vert→horiz) + yellow, then sink to row 5 |
| 2 BL | Magenta bar (6) 2 cells, Green L (3) 3 cells | 2 | STABLE | Green (3>2) | Green stays. Magenta mirrors: (8,1-2)→(8,3-4) |
| 3 BR | Red bar (2) 2 cells, Azure bar (8) 3 cells | 2 | STABLE | Azure (3>2) | Azure stays. Red mirrors: (7,8-9)→(7,10-11) |

**Key observations:**
- Room 0: shape touches wall, so it only recolors (no rotation) — BUT it still sinks!
- Room 3: red bar flipped to other side of room despite being near the wall (because stable shapes use mirror, not wall-block)

### Example 3 — Gravity sinking visually prominent

| Room | Shapes | Count | Status | Dominant | Action |
|------|--------|-------|--------|----------|--------|
| 0 TL | Azure L (8) 3 cells, Orange bar (7) 2 cells | 2 | STABLE | Azure (3>2) | Azure stays. Orange mirrors: (4,4-5)→(4,0-1) |
| 1 TR | Blue bar (1) 2 cells, touches wall | 1 | UNSTABLE | — | Touches wall → recolor 4 only, then sink to row 5 |
| 2 BL | Maroon L (9) 3 cells | 1 | UNSTABLE | — | Not touching wall → rotate 90° CW + yellow, then sink to rows 11–12 |
| 3 BR | Magenta bar (6) 2 cells, Red L (2) 3 cells | 2 | STABLE | Red (3>2) | Red stays. Magenta mirrors: (8,10-11)→(8,8-9) |

**Key observations:**
- Room 1: blue bar sinks from row 1 all the way to row 5 — very visible gravity
- Room 2: maroon L first rotates, THEN sinks — both operations visible in one shape

### Example 4 — All rule types present simultaneously

| Room | Shapes | Count | Status | Dominant | Action |
|------|--------|-------|--------|----------|--------|
| 0 TL | Green vbar (3) 2 cells, touches wall | 1 | UNSTABLE | — | Recolor 4 only, already at room bottom → no sink |
| 1 TR | Blue L (1) 3 cells, Azure bar (8) 2 cells | 2 | STABLE | Blue (3>2) | Blue stays. Azure mirrors: (4,10-11)→(4,8-9) |
| 2 BL | Orange L (7) 3 cells, Maroon dot (9) 1 cell | 2 | STABLE | Orange (3>1) | Orange stays. Maroon mirrors: (11,1)→(11,4) |
| 3 BR | Magenta L (6) 3 cells | 1 | UNSTABLE | — | Not touching wall → rotate + yellow + sink to rows 11–12 |

**Key observations:**
- Room 0: wall-touching unstable shape at bottom of room — gravity has no effect (already at floor)
- Room 2: tiny maroon dot mirrors across the room — easy to miss if not counting cells

---

## Test Example — Novel Shapes

The test deliberately uses shapes **never seen in training** to ensure the solver has truly learned the rules, not memorized shape-specific patterns.

| Room | Shape Type | Description | Count | Status | Dominant | Expected Action |
|------|-----------|-------------|-------|--------|----------|-----------------|
| 0 TL | **T-shape** (9) 4 cells — `┬` pointing down | `9 9 9` / `. 9 .` | 1 | UNSTABLE | — | Not touching wall → rotate 90° CW (T rotates to point left) + yellow + sink to rows 3–5 |
| 1 TR | **2×2 block** (8) 4 cells + **Z-zigzag** (3) 3 cells | Block: `8 8`/`8 8`; Zig: `. 3`/`3 3` | 2 | STABLE | Block/8 (4>3) | Block stays. Zigzag mirrors L↔R: (4,10),(5,9-10)→(4,9),(5,9-10) |
| 2 BL | **Plus/cross** (1) 5 cells + **Staircase** (6) 3 cells | Plus: `.1.`/`111`/`.1.`; Stair: `6.`/`66` | 2 | STABLE | Plus/1 (5>3) | Plus stays. Staircase mirrors: (8,4),(9,4-5)→(8,1),(9,0-1) |
| 3 BR | **S-zigzag** (7) 4 cells — `77..`/`..77` | Zigzag spanning 2 rows | 1 | UNSTABLE | — | Touches wall (top row adj to row 6) → recolor 4 only + sink to rows 11–12 |

### Test Shape Catalog (all new vs training)

```
T-SHAPE (Room 0)     2×2 BLOCK (Room 1)    PLUS/CROSS (Room 2)    S-ZIGZAG (Room 3)
  9 9 9                 8 8                    . 1 .                  7 7 . .
  . 9 .                 8 8                    1 1 1                  . . 7 7
                                               . 1 .
                      Z-ZIGZAG (Room 1)      STAIRCASE (Room 2)
                        . 3                    6 .
                        3 3                    6 6
```

No L-shapes, no single bars, no isolated dots — every test shape is visually and structurally distinct from training shapes.

---

## Categories Used (8 total)

| # | Category | How Used |
|---|----------|----------|
| 1 | **Object Detection** | Identify connected shapes + room boundaries |
| 2 | **Pattern Recognition** | Cross-wall room structure, shape types |
| 3 | **Counting** | Cell count per color for dominance; shape count for stability |
| 4 | **Logic & Conditional Rules** | Stable vs unstable branching, wall override, dominant vs minority |
| 5 | **Geometric Transformation** | 90° CW rotation for unstable free shapes |
| 6 | **Color Mapping** | All unstable shapes → yellow (4), regardless of original color |
| 7 | **Gravity / Physics** | Yellow shapes sink to room floor post-transformation |
| 8 | **Topology** | Connected components, wall adjacency detection |

---

## Why This Puzzle Takes 10+ Minutes

1. **False "stability" comfort** — A solver first thinks stable rooms are unchanged, then realizes minority shapes mirror. This requires re-examining all examples.
2. **Cell counting, not shape counting** — Dominance is by pixel area, not by number of shapes. A room with two shapes of the same size creates ambiguity.
3. **Chained operations** — Unstable shapes undergo up to 3 operations (rotate → recolor → sink). Each step must be understood in sequence.
4. **Gravity as a hidden final layer** — Without noticing gravity, computed outputs are close but vertically wrong. This is the "aha moment."
5. **Wall adjacency interacts differently per context** — Wall-touching matters for unstable shapes (blocks rotation) but is irrelevant for stable shapes (mirror happens regardless). This asymmetry is confusing.

---

## Quality Checklist

- ✅ Grid size is square (13×13) and consistent across all 5 grids
- ✅ No math or arithmetic on pixel values — only visual/spatial logic
- ✅ 5 interacting rules (exceeds minimum of 3)
- ✅ Shapes are meaningful: L-shapes, bars, T-shapes, dots
- ✅ Pattern is novel — room-context-dependent mutation with gravity
- ✅ JSON is valid ARC-AGI format
- ✅ All examples verified programmatically via `generate_d4e7a2f9.py`
- ✅ Explanation matches every training + test example