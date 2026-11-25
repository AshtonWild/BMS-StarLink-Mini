# Test procedure

This takes a newly assembled board from first inspection to a working pack. Stages 1–4 use a bench supply and resistors instead of cells, so a fault cannot cause a fire. Only move to real cells (stage 5) after every earlier stage passes.

Expected values come from the BM3451 datasheet and this board's component values. They have not been confirmed on a built board, so treat a small difference as normal and a large one as a fault to find.

## Safety

- Work on a non-flammable surface with nothing metal loose on the bench.
- Wear eye protection when cells are connected.
- Keep a way to disconnect the pack quickly (an inline connector on a cell wire).
- Never leave the first charge or first discharge unattended.
- Stop if anything gets hot, smells, or a reading is far from the expected value.

## Equipment

| Item | Needed for |
|---|---|
| Multimeter with diode mode | All stages |
| Bench supply, 0–25 V, adjustable current limit | Stages 2–4 |
| 5 × 100 Ω resistors, 1% or better, 1 W or more | Cell simulator |
| 1 kΩ resistor, 1 W | Test load |
| 2.0 kΩ resistor | Temperature test |
| Second bench supply, or the IP2368 module | Charger tests |
| Electronic load or power resistors (optional) | Load and overcurrent tests |

## Thresholds for your chip

Check which BM3451 version is fitted and use its column.

| | TNDC-T28A | UNDC-T28A | TJDC-T28A (obsolete) |
|---|---|---|---|
| Overcharge cut-off (per cell) | 4.250 V | 4.235 V | 4.250 V |
| Overcharge release | 4.190 V | 4.175 V | 4.190 V |
| Low-voltage cut-off | 2.80 V | 2.80 V | 2.50 V |
| Low-voltage release | 3.00 V | 3.00 V | 2.70 V |
| Balancing starts | 4.190 V | 4.180 V | 4.190 V |

Tolerances: overcharge ±0.025 V, low-voltage ±0.08 V, balancing ±0.05 V.

## Where to probe

There are no test points. Use these pads on the bottom side.

| Signal | Probe here | Reference |
|---|---|---|
| B− (battery negative) | BT−1 pad | n/a |
| Cell taps | BT+1 … BT+5 pads | BT−1 |
| Discharge gate (DO) | Q7 gate, or R25 | BT−1 |
| Charge gate (CO) | Q6 gate (pin 4) | PACK− |
| Output | PACK+ and PACK− pads | n/a |

---

## Stage 1: inspection, no power

1. **Look over the board.** Check for solder bridges on U1, missing parts, and tombstoned resistors.
2. **U1 orientation.** Pin 1 dot matches the board marking.
3. **D1 orientation.** Multimeter in diode mode:

   | Red probe | Black probe | Pass |
   |---|---|---|
   | PACK− | PACK+ | 0.5–0.7 V |
   | PACK+ | PACK− | open |

   If it fails, D1 is backwards. **Do not continue.**
4. **Fuse.** Resistance from BT+5 to PACK+ is under 1 Ω.
5. **No shorts between cell taps.** Measure resistance between each neighbouring pair: BT−1 to BT+1, BT+1 to BT+2, BT+2 to BT+3, BT+3 to BT+4, BT+4 to BT+5. Each must be well above 1 kΩ. A reading near 47 Ω means a balance transistor is shorted or bridged.
6. **Sense resistor.** Resistance from BT−1 to Q7's source pins is under 1 Ω (R23 is 5 mΩ; most meters show 0.0–0.2 Ω).
7. **Thermistor.** Resistance across TH1 is about 10 kΩ at room temperature (about 12 kΩ at 20 °C, 8 kΩ at 30 °C).

## Stage 2: power up on the cell simulator

Build the simulator: five 100 Ω resistors in series. The six points along the chain stand in for the cell taps.

```
Supply +  ──┬── BT+5
           100 Ω
            ├── BT+4
           100 Ω
            ├── BT+3
           100 Ω
            ├── BT+2
           100 Ω
            ├── BT+1
           100 Ω
Supply −  ──┴── BT−1
```

1. Set the supply to **18.5 V** (3.7 V per cell) with a **100 mA** current limit. Leave it off.
2. Connect **BT−1 first**, then BT+1 up to BT+5.
3. Turn the supply on. It should draw about 37 mA (the resistor chain). More than about 45 mA means a fault; switch off and recheck stage 1.
4. Measure each cell tap. Each step is 3.7 V ±0.1 V.
5. Connect the 1 kΩ test load across PACK+ and PACK−.

| Measurement | Pass |
|---|---|
| PACK+ to PACK− | about 18.5 V (output is on) |
| Q7 gate to BT−1 | about 10–12 V |
| Q6 gate to PACK− | about 10–12 V |

**If the output is off at first power-up:** some protection chips start in the protected state. Remove the test load, wait a few seconds, and refit it. If it is still off, briefly apply a charger (stage 4, step 3) to wake it.

## Stage 3: protection tests on the simulator

Keep the simulator and the 1 kΩ load connected. Each protection has a delay of about 1 second.

### 3.1 Low-voltage cut-off

1. Lower the supply slowly from 18.5 V. Watch the PACK voltage.
2. The output should switch off when the per-cell voltage reaches the low-voltage cut-off: about **14.0 V** total for TNDC/UNDC, about **12.5 V** for TJDC.
3. Pass: PACK+ to PACK− drops to near 0 V and the Q7 gate reads 0 V.

Reset: remove the test load, raise the supply back to 18.5 V, wait 5 seconds, and refit the load. The output should be back. If it is not, the board needs a charger to wake up (stage 4, step 3). That is expected behaviour with R24 = 10 MΩ.

### 3.2 Overcharge cut-off

All five simulated cells rise together, so move through 20.5–21.3 V without stopping. If some cells sit in the balancing range while others are below it, the balance resistors load the chain and the readings jump around. That is harmless here, but confusing.

1. From 18.5 V, step the supply straight to **21.6 V** (4.32 V per cell).
2. After about 1 second the charge gate should switch off.
3. Pass: Q6 gate to PACK− reads about 0 V. The PACK voltage with the 1 kΩ load stays up but drops by about 0.5 V, because discharge still flows through Q6's internal diode.
4. Step the supply back to 18.5 V. Pass: Q6 gate returns to about 10–12 V.

### 3.3 Over-temperature

1. With the supply at 18.5 V and the output on, hold a **2.0 kΩ** resistor across the TH1 pads. This simulates a very hot cell.
2. Pass: the output switches off and both gates read about 0 V.
3. Remove the resistor. Pass: the output returns.

### 3.4 Sense resistor check (optional)

If you have a supply that can deliver 1 A: pass exactly 1.00 A from BT−1 through R23 to Q7's source and measure the voltage directly across R23's pads. Pass: 5.0 mV ±0.3 mV. This confirms the overcurrent trip points without needing a 20 A load.

## Stage 4: charger behaviour on the simulator

1. Trigger the low-voltage cut-off as in 3.1 and leave the supply at the low voltage. Remove the test load.
2. Raise the supply to 18.5 V.
3. **Wake-up test.** Connect a second current-limited supply (set to 21 V, 100 mA) or the IP2368 module to PACK+ and PACK−, as a charger would be.
4. Pass: the output comes back within a few seconds and stays on after the charger is removed.

If the IP2368 module will not start because it sees no battery voltage, note it. The pack will then need a different way to wake after a deep discharge (for example a bench supply), or R24 changed to 2 MΩ. See [DESIGN_REVIEW.txt](DESIGN_REVIEW.txt).

**Disconnect the simulator before going further.**

## Stage 5: first connection of real cells

1. Measure every cell. All five must be within **0.05 V** of each other and between 3.0 V and 4.1 V. Charge or discharge individual cells to match them first if not.
2. Connect the cells as described in [ASSEMBLY.md](ASSEMBLY.md), **BT−1 first**, checking each cell's voltage at the pads as you go.
3. With all five connected, measure:

   | Measurement | Pass |
   |---|---|
   | Each cell at its pads | Same as measured off the board |
   | BT−1 to BT+5 | Sum of the five cells |
   | PACK+ to PACK− (1 kΩ load) | Same as BT−1 to BT+5 |

4. Leave it for 10 minutes with no load. Nothing should be warm.
5. Measure the idle current if you can (meter in series with BT−1): well under 1 mA.

## Stage 6: first charge

1. Connect the IP2368 module, or a bench supply set to **21.0 V** with the current limit at about 1 A.
2. Watch the cell voltages. They should rise together.
3. Near the end of charge, cells above the balancing voltage switch on their balance resistors. Check by measuring across the 47 Ω resistors (R13, R5, R8, R14, R17 for cells 1–5): about 4.2 V means that cell is balancing, 0 V means it is not. A balancing resistor runs warm (about 0.4 W).
4. Pass: charging ends with every cell between 4.15 V and 4.22 V and no cell above the overcharge cut-off.
5. If one cell hits the cut-off early, charging stops and restarts as the balancer bleeds it down. That is normal for an unbalanced pack and may take several cycles to settle. A cell that is always far ahead is a weak cell; replace it.

## Stage 7: load test

1. Connect the real load, or an electronic load set to about 7 A.
2. Run for 30 minutes. Check temperatures with a finger or thermometer: Q6, Q7, R23, F1 and the wire pads should stay below about 60 °C.
3. **Plug-in test.** Disconnect and reconnect the Starlink or IP2368 several times with the pack live. Pass: the output stays on. If it trips, unplug the load for a few seconds to reset it and note the result.
4. **Full discharge (supervised).** Run the pack down under load until the BMS switches off. Pass: it cuts off when the lowest cell reaches the low-voltage cut-off, and no cell is below about 2.5 V afterwards.
5. **Wake-up.** Plug USB-C power into the IP2368. Pass: charging starts and the output returns.

## Stage 8: overcurrent (optional, experienced builders only)

This needs a load that can draw more than 23 A at 21 V (about 500 W) and suitable wiring. Do not use a dead short as a test.

1. Raise the load current slowly from 15 A.
2. Pass: the output switches off between about 17 A and 23 A, after about 0.2 seconds.
3. Disconnect the load to reset.

## Results record

| Test | Expected | Measured | Pass |
|---|---|---|---|
| 1.3 D1 diode check | 0.5–0.7 V / open | | |
| 1.5 Tap-to-tap resistance | > 1 kΩ | | |
| 2.3 Simulator current | about 37 mA | | |
| 2.5 Output on, gates | 18.5 V, 10–12 V | | |
| 3.1 Low-voltage cut-off (total) | 14.0 V (TNDC/UNDC) | | |
| 3.2 Overcharge cut-off | Q6 gate 0 V at 21.6 V | | |
| 3.3 Over-temperature | Output off with 2.0 kΩ | | |
| 4 Charger wake-up | Output returns | | |
| 5.5 Idle current | < 1 mA | | |
| 6 End-of-charge cell voltages | 4.15–4.22 V | | |
| 7.2 Temperatures at 7 A | < 60 °C | | |
| 7.4 Discharge cut-off, lowest cell | 2.8 V (TNDC/UNDC) | | |
| 7.5 Wake-up from IP2368 | Charging starts | | |
