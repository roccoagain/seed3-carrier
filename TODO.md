# TODO

Schematic review findings, 2026-09-11.

## Fix before fab

- [ ] **Protect the 5 V input.** J3 is an unkeyed 2-pin header with no reverse-polarity or overvoltage protection. Reversed, U1, U2, the Seed and C14/C15/C18 see −5 V. If the bench supply is set too high, U1 (TLV9062) and U2 (TPA6111A2) are rated to only 5.5 V; the Seed tolerates up to 17 V.
  - Replace J3 with a keyed connector (barrel jack or JST).
  - Add P-FET reverse protection. Do not use a series Schottky: the Seed's VIN minimum is 5 V, so a diode drop leaves it underpowered.
  - Add a polyfuse and a ~5.6 V TVS or crowbar.
- [ ] **Stop the input plug-in pop.** The J1 tip side of C3 has no DC path to ground, while its other side sits at VREF (2.5 V) through RV1. Plugging in pulls the floating node to 0 V and sends a 2.5 V step into U1A. Add 1–2.2 MΩ from J1 tip to GND.

## Should fix

- [ ] **Add a power pin to J4 (DISPLAY_UART).** It has only GND/TX/RX. Add 3V3 or 5V unless the display will always be powered separately.
- [ ] **Add bulk capacitance at J3.** C9 (10 µF 1206 ceramic) derates to about 5 µF at 5 V. Ceramic-only input capacitance with long bench leads can ring on connection; add a ~100 µF electrolytic.
- [ ] **Ground the unplugged input.** Wire J1's TN switch pad to GND so the input is grounded when nothing is plugged in.
- [ ] **Restore an AUDIO_IN test point.** RV1 level calibration relies on measuring AUDIO_IN (target ≤ ~3 Vpp, below the Seed's 3.6 Vpp limit), but no test point exists on that net.
- [ ] **Check mono-plug behavior on J5.** A TS plug shorts R to sleeve, so U2 VOUT2 drives about 4.7 Ω. The TPA6111A2 has short-circuit and thermal protection, but bench-check heating.

## Housekeeping

- [ ] **Clear the two ERC warnings.** The `VIN` label sits on +5V and the `DGND` label sits on GND. Rename or delete the labels.
- [ ] **Verify the Seed3 pinout.** A1 uses the Daisy Seed Rev4 symbol and footprint. Confirm it matches the Seed3, especially VIN (pin 39) and the audio pins (16–19).
