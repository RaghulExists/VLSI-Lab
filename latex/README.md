# VLSI Lab — Report Diagrams (LaTeX / Overleaf)

Textbook-style **schematic**, **stick diagram** and **layout** for the three analog
experiments, as ready-to-compile LaTeX (`circuitikz` + `tikz`). Every file was
compiled and visually checked.

## Folder structure

```
latex/
├── 01_inverter/
│   ├── inverter_schematic.tex
│   ├── inverter_stick_diagram.tex
│   └── inverter_layout.tex
├── 02_nand2/
│   ├── nand2_schematic.tex
│   ├── nand2_stick_diagram.tex
│   └── nand2_layout.tex
├── 03_common_source_amplifier/
│   ├── csa_schematic.tex
│   └── csa_layout.tex          (stick diagram intentionally omitted — see note below)
└── all_diagrams_combined.tex   (optional: one document that prints all 8 figures)
```

## How to use in Overleaf

Each `.tex` in the numbered folders is a **standalone** document: compiling it gives a
single tightly-cropped figure (ideal for dropping a PNG/PDF into a Word or LaTeX report).

1. New Overleaf Project → upload the file you want (or paste it into `main.tex`).
2. Menu → **Compiler → pdfLaTeX** → **Recompile**.
3. Download the PDF, or in Overleaf use the PDF → "Download" / screenshot for a report.

**To embed a figure inside your own LaTeX report** instead of compiling standalone:
copy the `tikzpicture` (or `circuitikz`) block into your document and make sure the
preamble has:

```latex
\usepackage{circuitikz}
\usepackage{tikz}
\usetikzlibrary{calc}
```

**To get all figures at once:** compile `all_diagrams_combined.tex` (an `article`
document, also pdfLaTeX) — it produces one multi-page PDF with all 8 figures.

## Colour convention (Mead–Conway)

| Layer | Colour |
|-------|--------|
| Metal1 | blue |
| Polysilicon | red |
| n-diffusion (active) | green |
| p-diffusion (active) | orange |
| Contact / via | black square |
| n-well | dashed box |
| Select / implant | dotted box |

A legend is drawn on each stick diagram and layout.

## Note on the CS amplifier "stick diagram"

Stick diagrams are a convention for **digital** gates (inverter, NAND, NOR). An analog
amplifier is normally documented with a **schematic + transistor-level layout** (both
provided). A separate stick diagram for the common-source amplifier is therefore not
standard and is omitted on purpose; the layout already shows the diffusion / poly / metal
"sticks" if you need to refer to them.

## Device parameters used (gpdk180, from the lab manual)

| Circuit | Device | W | L |
|---------|--------|---|---|
| Inverter / NAND | PMOS | 40 µm | 180 n |
| Inverter / NAND | NMOS | 20 µm | 180 n |
| CS amplifier | NMOS (driver) | 6 µm | 180 n |
| CS amplifier | PMOS (mirror) | 8.85 µm | 180 n |

(The drawings are schematic/topological; add these W/L values as labels in your report
if your examiner wants them shown on the transistors — ask and I can annotate them.)
