# Footprint selections

Footprints are assigned in `seed3-carrier.kicad_sch` from KiCad's installed standard libraries. These choices favor hand soldering. PCB placement and routing are still separate work.

| Components | Selection |
| --- | --- |
| R1–R16 | 1206 (3.2 × 1.6 mm), enlarged HandSolder pads |
| C1–C3, C5, C8–C11, C16, C17 | 1206 ceramic, enlarged HandSolder pads |
| C12/C13, 4.7 µF | Murata GRM32ER71H475KA88L, 50 V X7R, 1210 with enlarged HandSolder pads |
| RV1 | Bourns 3296W-1-105LF, 1 MΩ linear, through-hole multiturn input trimmer |
| C7, 47 µF | 1210 (3.2 × 2.5 mm) ceramic, enlarged HandSolder pads |
| U1, TLV9062IDR | Standard SOIC-8, 3.9 × 4.9 mm body, 1.27 mm pitch |
| U2, TPA6111A2DR | SOIC-8, 3.9 × 4.9 mm body, 1.27 mm pitch |
| C14, C15 | 470 µF, 10 V polarized radial electrolytic; 8 mm body diameter, 3.5 mm lead pitch |
| C18 | 47 µF, 10 V polarized radial electrolytic; 5 mm body diameter, 2 mm lead pitch |
| J5 | Same Sky SJ1-3535NG, through-hole right-angle 3.5 mm TRS jack |
| A1 | Existing Electrosmith Daisy Seed module footprint |
| J1, J2 | Neutrik NRJ4HF, board-mounted right-angle 6.35 mm mono switched jacks |
| J3, 5V_IN | Through-hole 1×2 header, 2.54 mm pitch |
| J4, DISPLAY_UART | Through-hole 1×3 header, 2.54 mm pitch |
| RV2, RV3 | Alps RK09K1130A8G, 10 kΩ linear (1B), vertical, 15 mm shaft; Alps RK09K single vertical footprint |
| TP2, TP4–TP8 | Exposed 2 mm diameter test pads |

## Part selection and assembly

- Match the exact jack and potentiometer models above when ordering. These footprints are not universal. J1/J2 signal pads are S (sleeve) and T (tip); their unused switch pads SN and TN remain unconnected; potentiometer pin 2 is the wiper. Potentiometer MP pads are mechanical anchors and have no assigned circuit net.
- Jack openings face parallel to the PCB; potentiometer shafts project perpendicular to it. Establish their positions and enclosure clearances before routing.
- Select C0G/NP0 for C1 (100 pF) and C2 (1 nF). Use suitable X7R/X5R ceramics for the larger capacitances. These ceramic footprints are nonpolar, including C7. C14/C15/C18 are polarized electrolytics; observe the schematic polarity and select parts matching the specified body diameter and lead pitch.
- Capacitor manufacturer part numbers other than C12/C13 remain to be selected. Check voltage rating and effective capacitance under DC bias, particularly C7 (47 µF, 1210) and C9 (10 µF, 1206). A nominal capacitance/package assignment alone does not establish effective capacitance in the circuit.
- Header cable pin order follows the schematic, not an assumed display or power cable standard.

## Manufacturer references

- [TI TLV9062 datasheet and D/SOIC package drawing](https://www.ti.com/lit/ds/symlink/tlv9062.pdf)
- [Neutrik NRJ4HF](https://www.neutrik.com/en/product/nrj4hf)
- [DigiKey NRJ4HF, part 6473-NRJ4HF-ND](https://www.digikey.com/en/products/detail/neutrik-americas-inc/NRJ4HF/29427414)
- [Alps RK09K/RK09D series and RK09K1130A8G specifications](https://tech.alpsalpine.com/e/products/category/potentiometers/sub/02/series/rk09k/)

## Headphone output

U2 uses a project-local TPA6111A2DR symbol in `seed3-carrier.kicad_sym`, based on KiCad's LM4810 symbol geometry with the TPA6111A2 SOIC pinout checked against TI's datasheet. The footprint remains a standard KiCad SOIC-8 footprint.

The headphone branch takes `HP_SOURCE` directly from U1B's output. Two independent inverting channels feed J5 tip (left) and ring (right); sleeve connects to ground. Both channels receive the same mono signal. R9–R12 are 20 kΩ, giving a gain of -1 per channel and 10 kΩ effective inverting-node resistance. C12/C13 are now 4.7 µF, giving a nominal 1.7 Hz input corner. R13/R14 permanently connect the output capacitors to ground through 100 Ω; R15/R16 add 4.7 Ω in series with the jack contacts. The nominal output corner is 12.6 Hz with 32 Ω headphones and 19.8 Hz with 16 Ω headphones, including the bleeder and series resistance. Midband headphone attenuation from the series resistor is about 1.2 dB at 32 Ω and 2.2 dB at 16 Ω. Output capacitor positive terminals face U2. C16 is the 1 µF bias bypass; C17/C18 provide local 100 nF/47 µF supply decoupling. Shutdown pin 5 is grounded for always-on operation.

Place decoupling close to U2, keep feedback paths short, and route headphone return currents away from the guitar-input and bias circuitry. The +5 V supply must accommodate the added headphone load. Existing firmware volume control remains responsible for level; start bench testing at low volume. J5 uses its normally closed tip contact to mute J2 automatically on headphone insertion. See the switching description below.

### New part references

- [TI TPA6111A2 datasheet](https://www.ti.com/lit/ds/symlink/tpa6111a2.pdf)
- [DigiKey TPA6111A2DR, cut tape 296-10846-1-ND](https://www.digikey.com/en/products/detail/texas-instruments/TPA6111A2DR/382220)
- [DigiKey SJ1-3535NG, CP1-3535NG-ND](https://www.digikey.com/en/products/detail/same-sky-formerly-cui-devices/SJ1-3535NG/738699)

## Automatic line-output mute

J5 uses `Connector_Audio:AudioJack3_SwitchTR` and `Connector_Audio:Jack_3.5mm_CUI_SJ1-3535NG_Horizontal`. Manufacturer pins 1/2/3/4/5 correspond to KiCad pads S/T/R/TN/RN. With no plug inserted, T connects to TN (and R to RN). Inserting a TRS plug opens those normal contacts.

The line path is U2 channel 1 -> C14 -> R15 (4.7 Ω) -> J5 T/TN -> LINE_SEND -> R3 (100 Ω) -> J2 tip. R4 (100 kΩ) remains from J2 tip to ground. When headphones are inserted, the T/TN contact opens, removing the audio drive from J2 while R4 provides its ground reference. The headphone channels stay connected to T and R. RN is unused and explicitly marked no-connect; the two amplifier outputs are never joined.

C6 is removed because C14 already blocks DC. The line output now depends on U2 and has inverted polarity relative to the former U1B line output, with approximately the same unity-gain magnitude into a high-impedance load. This mutes the signal at the ¼-inch jack; it does not shut down an external power amplifier. Plug transients and residual feedthrough require bench checks.

[Jack switching diagram and pinout](https://www.sameskydevices.com/product/resource/sj1-353xng.pdf)

## Input level adjustment

RV1 is a Bourns 3296W-1-105LF 1 MΩ board trimmer between C3 and R1, referenced to VREF. Pin 3 is the incoming signal, pin 1 is VREF, and pin 2 is the wiper; clockwise rotation increases the level. R5 is increased to 10 MΩ to retain a DC reference if wiper contact is interrupted without significantly loading the trimmer. Nominal input resistance is about 0.91 MΩ at full level and approaches 1 MΩ as the trimmer is reduced. C4 is removed because its 100 pF capacitance would roll off guitar treble against the trimmer's source resistance; C1 remains at the jack for RF filtering.

This is a manual input attenuator, not an automatic limiter. For a passive guitar, use full clockwise if the hardest playing stays below clipping. For an active guitar or pedals, turn down first and calibrate with the hottest intended signal. Target no more than approximately 3 V peak-to-peak at TP2 (AUDIO_IN), allowing margin below the Seed's documented 3.6 V peak-to-peak range. The maximum setting does not protect against an arbitrarily hot input. Firmware output volume cannot correct input clipping. RV1 should remain accessible for adjustment when changing sources; it is an internal setup trimmer, not a front-panel performance control.

[Bourns trimmer datasheet](https://www.bourns.com/docs/product-datasheets/3296.pdf) · [DigiKey RV1](https://www.digikey.com/en/products/detail/bourns-inc/3296W-1-105LF/1088047) · [Seed input limits](https://daisy.nyc3.cdn.digitaloceanspaces.com/products/seed/Daisy_Seed_datasheet.pdf)

## Output charging and startup

R13/R14 provide permanent discharge paths even with J2/J5 unplugged. Unloaded output-capacitor time constants are now 47 ms instead of approximately 47 seconds on the left and an undefined floating right output. About five time constants (235 ms nominal) discharge an already charged output-side node to below 1% when the amplifier side is stationary. This is not a guaranteed power-on settling time: the internal bias also ramps.

With C12/C13 increased to 4.7 µF and C16 retained at 1 µF, the nominal time constants follow TI's Equation 8: bias 230 ms >= input 94 ms >= output 47 ms. Any headphone or line load only reduces the output time constant. Use 1% resistors and ±20% or better C14/C15. Select/verify effective C12/C13 capacitance within 3.3–6.2 µF and C16 at least 0.68 µF at operating bias and temperature; these bounds preserve the relationship even at resistor/output-capacitor tolerance corners using TI's typical 230 kΩ internal bias resistance. C12/C13's selected 50 V X7R part has low steady-state DC voltage across it because both sides sit near midrail. Verify capacitance derating when substituting parts. Internal bias-resistance production limits are not specified, so timing calculations are design checks, not a guaranteed pop specification.

R15/R16 keep the amplifier load above 16 Ω for headphones rated 16 Ω or higher despite the bleeders: at nominal 16 Ω, (16 + 4.7) || 100 = 17.15 Ω. Each 100 Ω bleeder dissipates at most about 63 mW for a 2.5 V excursion; use at least 0.25 W 1206 resistors. The added loading reduces available headphone level somewhat. The jack still disconnects J2 when headphones are inserted.

This uses passive charging and the TPA6111A2's internal pop-reduction behavior; there is no timed output relay or firmware startup mute. Bench-check startup, power-down, and insertion/removal into 16/32/64 Ω dummy loads and a high-impedance line load before testing with headphones on ears. Check peak transient voltage and final DC on both jack channels. Mechanical contact noise and shutdown behavior are not eliminated by the RC calculation.

[TI startup guidance, Section 10.2.2](https://www.ti.com/lit/ds/symlink/tpa6111a2.pdf) · [C12/C13 manufacturer specifications](https://search.murata.co.jp/Ceramy/image/img/A01X/G101/ENG/GRM32ER71H475KA88-01A.pdf)

## Verification

All 49 physical components have assigned footprints in the installed library, and all symbol pins have matching pads. Connectivity checks cover the new input trimmer and both output discharge/isolation networks; unrelated connections are preserved. Netlist checks cover both open and closed jack-switch states. KiCad schematic ERC reports 0 violations. The rendered schematic was inspected. PCB layout, enclosure fit, power-on transients, output DC, and audio performance under headphone load still require hardware validation.
