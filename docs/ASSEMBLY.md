# Assembly guide

Read the warning in the [README](../README.md) first. Do every step in [TESTING.md](TESTING.md) before connecting real cells.

## 1. Parts

The BOM in `production/` lists every assembled part with its LCSC number. Make these changes to it.

### Required substitutions

| Part | BOM says | Fit instead | Notes |
|---|---|---|---|
| U1 | BM3451TJDC-T28A (C5343512) | BM3451TNDC-T28A or BM3451UNDC-T28A (JLCPCB C94597) | Same pads and orientation |
| D1 | SMCJ24CA (C3040015) | SMCJ24A, one-way | Turned 180° from the board marking, see section 3 |
| TH1 | 0603 NTC on the board | Leaded 10 kΩ NTC (B about 3950) on two wires | Tape it to a middle cell |

Do **not** use BM3451 versions HEDC or BHDC (wrong voltage for Li-ion) or any `-T20B` version (different chip).

### Optional improvements

| Part | BOM says | Better | Effect |
|---|---|---|---|
| R2 | 10 MΩ | 5.1 MΩ | Charge FET turns off about twice as fast |
| C7, C8, C9 | 100 nF ±10% | 100 nF ±5% | More accurate protection delays |
| C10 | 22 nF ±10% | 22 nF ±5% | Same |

### Not on the BOM

- 5 × Li-ion 26650 cells, matched (same model, same age, within 0.05 V of each other)
- IP2368 charger/converter module
- 2.5 mm² (about 13–14 AWG) silicone wire for the cells and pack output
- A second 10 kΩ NTC for the IP2368's temperature input
- Neutral-cure silicone or epoxy for strain relief
- Fish paper or Kapton tape if the cells sit against the board

Where to buy the replacement parts is listed in [DESIGN_REVIEW.txt](DESIGN_REVIEW.txt), section 6.

## 2. Board layout

- **Bottom side:** all the electronics, plus the PACK+ and PACK− wire pads.
- **Top side:** ten cell wire pads (BT−1 … BT+5) and TH1.

The board links the cells in series for you. Each cell gets its own pair of pads:

| Cell | Negative wire to | Positive wire to |
|---|---|---|
| 1 (lowest) | BT−1 | BT+1 |
| 2 | BT−2 | BT+2 |
| 3 | BT−3 | BT+3 |
| 4 | BT−4 | BT+4 |
| 5 (highest) | BT−5 | BT+5 |

BT+1 and BT−2 are the same copper, as are BT+2/BT−3, BT+3/BT−4 and BT+4/BT−5.

## 3. Fitting D1 (safety-critical)

The board was drawn for a two-way TVS, so its cathode mark is on the wrong side for the one-way SMCJ24A.

- The **cathode** (the end with the band) goes on the D1 pad connected to **PACK+**. D1 sits beside the PACK− wire pad; its PACK+ pad is the one **farther** from that wire pad. Confirm with a continuity check to the PACK+ wire pad.
- The other pad connects to PACK−.
- Fitted backwards, D1 shorts the pack as soon as the output turns on.

Check it with a multimeter in diode mode **before any cell is connected**:

| Red probe | Black probe | Reading |
|---|---|---|
| PACK− | PACK+ | about 0.5–0.7 V |
| PACK+ | PACK− | open (OL) |

If the readings are the other way round, D1 is backwards. Remove it and refit it.

## 4. Soldering order

1. Fit or check U1 and D1 (sections 1 and 3).
2. Solder the leaded thermistor's two wires to the TH1 pads. It has no polarity.
3. Run the bare-board tests in [TESTING.md](TESTING.md), stages 1–3, using a bench supply. **No cells yet.**
4. Solder the PACK+ and PACK− wires.
5. Solder the cell wires to the board **before** connecting them to the cells, or solder them with the cells insulated and one end at a time. Never let a loose cell wire touch the board.
6. Connect the cells in this order: **BT−1 first**, then BT+1, and work upward one cell at a time to BT+5. The BM3451 must have its ground (BT−1) connected before any other tap.
7. After each cell, measure its voltage at the board pads. It must be positive and match the cell. A reversed cell will destroy U1.
8. Glue each wire to the board a few millimetres back from its pad.
9. Tape the thermistor to the side of a middle cell and cover it with foam or tape so it reads the cell, not the air.
10. If the cells rest on the board, put fish paper or Kapton between them and the board.

## 5. Connecting the IP2368 module

- PACK+ → module battery positive, PACK− → module battery negative.
- Set the module for a 5-cell Li-ion pack (21.0 V full charge) as its documentation describes.
- Wire the second thermistor from the module's NTC input to the cells. This is what stops charging in freezing temperatures; the BMS itself cannot.
- Keep the charge current at or below the cells' rated charge current and well under the board's 10 A charge limit.

## 6. Mounting

Four M3 holes with copper keep-out rings. Use nylon or insulated standoffs, and make sure no metal hardware touches the wire pads.
