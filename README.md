# OMR Answer Sheet Generator

A LaTeX-based generator for customizable Optical Mark Recognition (OMR) answer sheets.

## Files

- `omrSheet.tex` — Main LaTeX document. Edit the configuration section at the top of the file to customize the answer sheet.
- `omrSheet.pdf` — Compiled output (generated, not committed).
- `.gitignore` — Excludes build artifacts.

## Compile

```bash
pdflatex omrSheet.tex
```

## Customization

Edit the configuration variables at the top of `omrSheet.tex`:

| Variable | Description |
|---|---|
| `\nquestions` | Total number of questions |
| `\ncols` | Number of columns |
| `\options` | Answer options (e.g., `{A,B,C,D}`) |
| `\bubblegap` | Horizontal gap between bubbles |
| `\ysep` | Vertical gap between question rows |
| `\circlesize` | Diameter of each bubble |
| `\headerheight` | Height of the colored header bar |
| `boxfill`, `boxborder`, `headerfill`, etc. | Colors |

## Template Structure

The `omrSheet.tex` template consists of three sections:

1. **User Configuration** — Edit the `\def` values to set questions, columns, spacing, bubble size, box style, header dimensions, and colors.
2. **Auto-Computed Values** — The template auto-calculates row counts, offsets, and column boundaries based on the configuration.
3. **Column Boxes & Headers** — TikZ draws each column as a rounded rectangle with a colored header bar, accent line, and label (e.g., `Q1 – Q5`).
4. **Questions & Bubbles** — A loop draws question numbers and circular answer option bubbles inside each column, with drop shadows for readability.

See [`doc/omrSheet.md`](doc/omrSheet.md) for a detailed reference of every macro and drawing command.
