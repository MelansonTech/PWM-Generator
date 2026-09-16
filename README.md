# PWM Generator

A low-cost, adjustable PWM signal generator built around an LM393 dual comparator.
One half of the comparator runs a sawtooth oscillator, the other slices it against an
adjustable threshold, and a discrete push-pull follower drives the output to 100 mA.

No microcontroller, no firmware — two trimmers and a jumper.

Complete CircuitStudio design: schematic, PCB, and a released manufacturing package
(Gerbers, NC drill, pick & place, BOM, STEP).

---

## Specifications

| | |
|---|---|
| Supply voltage | 5 – 15 V DC |
| Output current | up to 100 mA |
| Output swing | GND to the input rail, less the input diode drop and one V<sub>BE</sub> |
| Duty cycle | 0 – 100 %, set by trimmer R2 |
| Frequency | 15 Hz – 300 kHz across three jumper-selected ranges |
| Soft start | duty ramps up from zero on power-up (C6) |
| Shutdown | active-high SD input forces 0 % duty |
| Board | 25.0 × 28.0 mm (0.98 × 1.10 in), 2-layer |
| Assembly | SMD on the bottom side only, plus 4 through-hole parts |
| Placements | 40 (19 unique parts, C18 not populated) |

## Frequency ranges

Range is selected by jumpering one pair on **J2**, which switches the oscillator's
timing capacitor:

| J2 jumper | Timing cap | Range |
|---|---|---|
| 1 – 2 | C4, 2.2 nF | 55 kHz – 300 kHz |
| 3 – 4 | C5, 100 nF | 1.5 kHz – 10 kHz |
| 5 – 6 | C7, 10 µF | 15 Hz – 120 Hz |

Within a range, **R9** sets the frequency. Exactly one pair should be jumpered at a time.

## Connections

**J1** — 1×4, 0.1 in pitch:

| Pin | Net | Description |
|---|---|---|
| 1 | `PWM_OUT` | PWM output, up to 100 mA |
| 2 | `Vin` | 5 – 15 V DC supply |
| 3 | `GND` | Ground |
| 4 | `SD` | Shutdown, active high — forces 0 % duty. Leave open or tie to GND for normal operation |

## Adjustments

| Part | Function |
|---|---|
| R9 | Frequency, within the range selected by J2 |
| R2 | Duty cycle |
| C6 | Soft-start time constant — increase for a slower ramp, decrease for a faster one |

Both are 10 kΩ top-adjust trimmers (CT94EW103).

## How it works

- **Input** — Vin enters through D1 (MBR0560 Schottky) for reverse-polarity protection.
  U1, an HT7550-1 SOT-89 LDO, derives the +5 V rail that the comparator and timing
  network run from. The output stage runs from the input rail, not the 5 V rail, so the
  PWM output swings as high as the supply allows.
- **Sawtooth oscillator** — One half of U2 (LM393) charges the selected timing capacitor
  through R9 and discharges it at a fixed reference, producing a 0 – 2.5 V sawtooth.
  J2 picks the capacitor and therefore the range; R9 sets the charge current and
  therefore the frequency.
- **PWM comparator** — The other half of U2 compares the sawtooth against a DC control
  voltage from R2. Where the control voltage sits within the sawtooth's 0 – 2.5 V span
  is the duty cycle.
- **Soft start** — C6 sits on the control node and charges through R5 at power-up, so
  the control voltage ramps from 0 V and the duty cycle ramps with it.
- **Shutdown** — Driving SD high turns on Q5, which pulls the control node to ground and
  holds the output at 0 % duty.
- **Output stage** — Q3 (NPN, SMBT2222A) and Q4 (PNP, MMBT2907A) form a complementary
  emitter follower. R6 (10 Ω) is in series with the output and R14 (1 kΩ) pulls it down.

Key parts: **U2** LM393DT · **U1** HT7550-1 · **Q1–Q3, Q5** SMBT2222A · **Q4** MMBT2907A-7-F · **D1** MBR0560

## Repository layout

```
Melanson Tech - PWM Generator.PrjPcb      CircuitStudio project
Melanson Tech - PWM Generator.SchDoc      Schematic
Melanson Tech - PWM Generator.CSPcbDoc    PCB layout
PWM Generator Rev 1.OutJob                Output job - regenerates everything below

Default Configuration/Outputs/
  PWM Generator Rev 1.PDF                 Schematic and PCB drawings
  Gerber/                                 RS-274X, 2 layers, bottom-side assembly
  NC Drill/                               Excellon drill files and drill report
  Pick Place/                             Placement data, .csv and .txt
  BOM/                                    Bill of materials with Digi-Key part numbers
  ExportSTEP/                             3D model of the assembled board
```

## Opening the design

Open `Melanson Tech - PWM Generator.PrjPcb` in Altium CircuitStudio (the outputs here
were generated with 1.5.2) or in Altium Designer, which reads the same formats.

The project's library search path points at a local folder, so schematic symbols and
footprints may not resolve on another machine. The design files themselves carry
everything needed to view, plot, and fabricate the board — the libraries are only
required to edit components or re-run an ECO.

## Ordering boards

The manufacturing package under `Default Configuration/Outputs/` was generated on
2024-04-06 and is what you send to a fab house:

- **Gerbers** — RS-274X, metric, 2 layers (`.GTL` top copper, `.GBL` bottom copper),
  with `.Outline` as the board outline
- **Drill** — `.DRL` (Excellon) plus `.DRR`; 40 plated holes, 4 tools, smallest 0.3 mm
- **Assembly** — pick & place plus the `.GBP` bottom paste layer. Every SMD part is on
  the bottom side, so this is a single-sided reflow. J1, J2 and the two trimmers are
  through-hole.
- **BOM** — `BOM_PartType-Melanson Tech - PWM Generator.xls`, with Digi-Key part numbers
  for most line items. C18 is marked **DNP** — do not populate.

If you change the design, re-run `PWM Generator Rev 1.OutJob` rather than editing these
files by hand.

## License

**CERN Open Hardware Licence Version 2 — Strongly Reciprocal (`CERN-OHL-S-2.0`).**

Copyright © 2024–2026 MelansonTech (Shawn Melanson).

This source describes Open Hardware and is licensed under CERN-OHL-S v2. You may redistribute
and modify it under the terms of that licence. This source is distributed *WITHOUT ANY EXPRESS
OR IMPLIED WARRANTY, INCLUDING OF MERCHANTABILITY, SATISFACTORY QUALITY AND FITNESS FOR A
PARTICULAR PURPOSE.* Please see the licence for applicable conditions.

In short: you may build, modify and sell this design, but if you distribute a modified version —
or a product made from one — you must publish your modified source under the same licence.

The full licence text is in [`LICENSE`](LICENSE), or at <https://cern.ch/cern-ohl>.
