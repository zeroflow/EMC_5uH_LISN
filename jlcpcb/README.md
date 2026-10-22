# Ordering the 5µH LISN at JLCPCB

This folder makes the V1 R0 design orderable at JLCPCB (PCB + SMD assembly)
without touching any of the original design files. Everything here is
generated from the unmodified `5uH_LISN.kicad_sch`, `5uH_LISN.kicad_pcb` and
`Gerber_IPC356_BOM_camset/`.

| File | Use |
|---|---|
| `parts.csv` | Single source: every reference, who fits it (`jlc`, `hand`, `dnp`), LCSC number, MPN, original MPN, notes |
| `generate.py` | Builds `out/` from the KiCad files and `parts.csv` (Python 3, standard library only) |
| `out/gerbers.zip` | Upstream Gerber and drill files, zipped for upload |
| `out/jlc_bom.csv` | BOM for JLCPCB assembly (`jlc` parts only) |
| `out/jlc_cpl.csv` | Placement file for those parts |
| `out/hand_bom.csv` | Parts you solder yourself |
| `addons.csv` | Accessories for measuring with the LISN (Mouser BOM import format), not part of the board |
| `out/order_bom.csv` | The same parts in the column names of Mouser's BOM import: `Quantity 1` = 1 LISN, `Quantity 2` = 10 LISNs |

`generate.py` refuses to write anything unless every reference in the
schematic and on the PCB appears exactly once in `parts.csv` and the values
match the schematic. `generate.py --check` reports stale files in `out/`.

## Who fits what

| `assembly` | Parts | Why |
|---|---|---|
| `jlc` (bottom side) | all resistors and capacitors, BAV99, GDT | jellybean SMD parts, 18 LCSC numbers |
| `hand` | 10 × Würth 744314110, 2 × Coilcraft 1008PS-105KLC, 2 × SMA, 6 × banana jack, case | not stocked at LCSC, or through-hole; the inductors are already excluded from the position file in the upstream layout |
| `dnp` | H1–H4 | mounting holes, no part |

All SMD parts sit on the **bottom** side, the SMA and banana jacks on the top.

## Ordering

1. **PCB:** upload `out/gerbers.zip`. 2 layers, 112.4 × 60.5 mm, 1.6 mm FR-4,
   1 oz copper. Any surface finish; the exposed copper along the bottom edge
   is the ground contact to the case.
2. **Assembly:** enable PCB Assembly, choose **Bottom side**, upload
   `out/jlc_bom.csv` and `out/jlc_cpl.csv`.
3. **Check the placement preview.** Only D1–D4 (BAV99) are orientation
   critical: pins 1 and 2 are both GND, so pin 3 (the common node) must sit on
   the single pad. Their `rot_offset` of 270° was taken from the JLCPCB preview
   on 2026-10-22. All other JLC parts are unpolarised two-terminal parts. If a
   rotation is wrong, put the correction into `rot_offset` in `parts.csv` and
   regenerate.
4. **Hand parts:** import `out/order_bom.csv` with Mouser's BOM tool. The
   column names follow Mouser's export format, so the mapping should be
   detected automatically. Other distributors: map the columns by hand.
   The Coilcraft 1008PS-105KRC (L6/L12) was not in stock at Mouser on
   2026-10-22 (lead time 9 weeks); plan the order around it. Coilcraft
   samples are free on request.

## Accessories (`addons.csv`)

Not part of the board, but needed or useful for a measurement:

- **50 Ω SMA termination (required):** each LISN line is only 50 Ω/5 µH when
  its RF port is terminated. Terminate the port that is not connected to the
  analyzer.
- **Extra 10 dB SMA attenuator (optional):** in addition to the on-board
  10 dB attenuator and limiter, for the first power-ups.
- **SMA cable, banana plugs, DC cable:** LISN to analyzer and LISN to DUT.
  Keep the DUT cable identical for all compared measurements.

Part numbers were taken from distributor listings on 2026-10-22 and not
checked for stock.

## Case

The PCB **replaces the lid** of a Hammond 1590B: it is screwed onto the box
with the four lid screws (H1–H4), components facing inside, jacks outside.
The 1590BBK named in the schematic is powder coated, which insulates the rim.
Either remove the coating where the PCB touches it or use the unpainted 1590B.

## Deviations from the original BOM

LCSC stock checked on 2026-10-22. Stock changes; check the BOM matching step at
JLCPCB before ordering.

- **0.1 % thin-film resistors → 1 % thick film.** None of the original 0.1 %
  parts were in stock. 1 % changes the attenuation by well below 0.1 dB.
- **R7/R8, R27/R28: 0.25 W instead of 0.4 W** (R17/R18 are 0.5 W). The
  "1 W / 30 dBm" rating of the attenuator in the upstream README has not been
  re-checked with these parts.
- **R3/R4: 0.5 W 5 % instead of 1 W 1 %.** Damping resistor of the 22 µF
  supply capacitor, no DC current.
- **C1/C2:** standard 22 µF 100 V X7R 2220 instead of the stacked TDK CKG57N.
- **C15/C16:** Murata instead of TDK, same rating.

## Regenerating

```bash
python3 jlcpcb/generate.py          # writes out/
python3 jlcpcb/generate.py --check  # exit 1 if out/ is stale
```

Placement coordinates are in the Gerber coordinate system (PCB file origin,
Y up), rotation as stored in the PCB file plus `rot_offset`.
