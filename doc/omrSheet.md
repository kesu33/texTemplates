# OMR Sheet — Detailed Reference

> **TODO:** 
1. Add a header section to `omrSheet.tex` containing exam name and student information fields (e.g., name, roll number, class, date).


This document describes every macro and drawing block inside [`omrSheet.tex`](omrSheet.tex).

---

## 1. Preamble

```latex
\documentclass[a4paper]{article}
\usepackage[top=1cm, bottom=1cm, inner=1cm, outer=1cm]{geometry}
\usepackage{tikz}
\usetikzlibrary{math}
\usepackage{xcolor}
```

| Package | Purpose |
|---|---|
| `geometry` | Sets 1 cm margins on all sides. |
| `tikz` | All drawing is done in a `tikzpicture`. |
| `tikz` `math` library | Enables `\pgfmathsetmacro` and `\pgfmathtruncatemacro`. |
| `xcolor` | Named colors (`RGB`) for boxes, headers, and bubbles. |

---

## 2. User Configuration

All user-facing knobs are plain `\def` macros placed at the top of the picture.

### Question Layout

| Macro | Default | Description |
|---|---|---|
| `\nquestions` | `20` | Total number of question rows. |
| `\ncols` | `4` | Columns; rows are auto-computed as `ceil(\nquestions/\ncols)`. |
| `\options` | `{A,B,C,D}` | Answer options; add or remove letters to change choices. |

### Spacing

| Macro | Default | Description |
|---|---|---|
| `\numgap` | `0.4` | Horizontal space from question number to first bubble (cm). |
| `\bubblegap` | `0.72` | Horizontal space between each bubble (cm). |
| `\xsep` | `4.5` | Horizontal distance between column centers (cm). |
| `\ysep` | `1.35` | Vertical distance between question rows (cm). |

### Bubble Appearance

| Macro | Default | Description |
|---|---|---|
| `\circlesize` | `5.5mm` | Diameter of each answer bubble. |
| `\bubblethick` | `1.2pt` | Border stroke width for bubbles. |

### Column Box Appearance

| Macro | Default | Description |
|---|---|---|
| `\boxpadx` | `0.2` | Horizontal padding inside the column box (cm). |
| `\boxpady` | `0.5` | Bottom vertical padding inside the column box (cm). |
| `\boxcorner` | `10pt` | Rounded-corner radius. |
| `\boxlinewidth` | `1pt` | Border stroke width of the column box. |

### Header Bar

| Macro | Default | Description |
|---|---|---|
| `\headerheight` | `0.7` | Height of the colored header bar (cm). |
| `\headertoppad` | `0` | Padding above the header bar (cm). |
| `\headerbtmpad` | `0.5` | Gap below the header bar before first row (cm). |

### Colors

| Name | Usage |
|---|---|
| `boxfill` | Column box background. |
| `boxborder` | Column box border. |
| `headerfill` | Header bar background. |
| `headertext` | Header label text color. |
| `qnumcolor` | Question number text color. |
| `bubblefill` | Bubble interior. |
| `bubbleborder` | Bubble border. |
| `bubbletext` | Bubble letter color. |

---

## 3. Auto-Computed Values

Calculated from the user configuration; do not edit directly.

```latex
\pgfmathsetmacro{\nrows}{ceil(\nquestions/\ncols)}
\pgfmathtruncatemacro{\nrowsint}{\nrows}
\pgfmathsetmacro{\firstrowoffset}{\headertoppad + \headerheight + \headerbtmpad}
```

- `\nrows` / `\nrowsint` — Rows per column.
- `\firstrowoffset` — Total vertical space from the box top edge to the center of the first question row. This combines header padding, the header bar itself, and the gap below it.

---

## 4. Column Boxes & Headers

This block runs once per column using `\foreach \col in {0,...,\ncols-1}`.

### Per-Column Indices

| Macro | Computation | Description |
|---|---|---|
| `\colstart` | `\col * \nrowsint` | Zero-based index of the first question in this column. |
| `\colend` | `min((\col+1)*\nrowsint, \nquestions)-1` | Zero-based index of the last question. |
| `\nrowsthiscol` | `\colend - \colstart + 1` | How many rows this column actually has. |
| `\qs` | `\colstart + 1` | First question number for the header label. |
| `\qe` | `\colend + 1` | Last question number for the header label. |

### Coordinates

| Macro | Description |
|---|---|
| `\bxl` | Left edge of the column box. |
| `\bxr` | Right edge of the column box. |
| `\byt` | Top edge of the column box (fixed at `y = 0`). |
| `\byb` | Bottom edge, computed from `\firstrowoffset`, `\nrowsthiscol`, and `\boxpady`. |
| `\bxmid` | Horizontal center of the column box. |
| `\hbart` | Top of the header bar inside the box. |
| `\hbarb` | Bottom of the header bar inside the box. |
| `\hbarmid` | Vertical center of the header bar (used for label placement). |

### Drawing Steps (per column)

1. **Drop shadow** — A black 12 % rectangle offset `(0.07, -0.07)` with rounded corners.
2. **Main box** — A filled rectangle with `boxfill`, `boxborder`, and rounded corners.
3. **Header fill** — A filled rectangle clipped to the box’s rounded corners, painted `headerfill`.
4. **Accent line** — A thin line (`boxborder!80`, 0.6 pt) at the bottom of the header bar.
5. **Header label** — A bold sans-serif node centered horizontally and vertically: e.g. `Q1 -- Q5`.

---

## 5. Questions & Bubbles

This block runs once per question using `\foreach \i in {0,...,\nquestions-1}`.

### Per-Question Indices

| Macro | Computation | Description |
|---|---|---|
| `\row` | `mod(\i, \nrowsint)` | Row index within the column. |
| `\col` | `int(\i / \nrowsint)` | Column index. |
| `\qnum` | `\i + 1` | One-based question number. |

### Coordinates

| Macro | Computation | Description |
|---|---|---|
| `\x` | `\col * \xsep` | Column center x-coordinate. |
| `\y` | `-\firstrowoffset - \row * \ysep` | y-coordinate of the question row center. |

### Drawing Steps (per question)

1. **Question number** — Left-aligned text at `(\x, \y)`. Single-digit numbers are padded with a leading zero (`01.`, `02.`, …).
2. **Answer bubbles** — A loop `\foreach \opt [count=\k from 1] in \options` draws:
   - A small shadow circle (`black!10`, offset `+0.04 / -0.04`).
   - A main circle (`minimum size=\circlesize`, `draw=bubbleborder`, `fill=bubblefill`) containing the option letter.
   - Bubble x-coordinate: `\x + \numgap + \k * \bubblegap`.

---

## 6. Design Notes

- Everything fits in a single `tikzpicture`; no external pages or boxes are used.
- Changing `\ncols` does **not** require manual row rebalancing: `\nrowsint` is auto-calculated.
- If `\nquestions` is not evenly divisible by `\ncols`, the last column is shorter; the bottom edge of each box adjusts independently via `\nrowsthiscol`.
- The header label format `Q\qs\ --\ Q\qe` renders a text range (e.g., `Q1 -- Q5`). Modify the node text to change the label.
