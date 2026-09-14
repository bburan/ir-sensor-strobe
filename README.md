# Strobed IR Emitter / Sensor Board

A small four‑channel infrared emitter/detector board. Four IR LEDs (OP140) are
driven as a single strobed array by an external control signal; four
phototransistors (OP550) act as independent detectors, each presenting an analog
output to a downstream acquisition system. The board is all through‑hole and is
intended to be hand‑soldered.

![Board overview](IR%20sensor%20strobe.png)

---

## 1. Overview / Theory of Operation

The board contains two functional blocks that share a common +5 V / GND rail.

### 1.1 Strobed IR emitter (one channel, driving four LEDs)

```
 +5V ──┬─[68Ω R2]─▷|─ D1 ─┐
       ├─[68Ω R3]─▷|─ D2 ─┤
       ├─[68Ω R4]─▷|─ D3 ─┤   common cathode
       └─[68Ω R5]─▷|─ D4 ─┴────────────┐
                                        │ (collector)
   CTL ──┬─[1kΩ R1]── B  Q1 (NPN, TO‑92)  ◄
         │                E ── GND
       [10kΩ R10]
         │
        GND
```

All four IR emitters (D1–D4, OP140) sit between the +5 V rail and a common node,
each with its own 68 Ω series resistor (R2–R5) setting the forward current to
roughly

```
I_LED ≈ (5 V − V_f) / 68 Ω ≈ (5 − 1.6) / 68 ≈ 50 mA per LED
```

The common cathode node is pulled to ground by low‑side NPN switch **Q1**. Q1 is
driven from the **CTL** input through the 1 kΩ base resistor **R1**
(I_B ≈ (V_CTL − 0.7 V)/1 kΩ, ≈ 2.6 mA at 3.3 V logic, ≈ 4.3 mA at 5 V). When CTL
is asserted high, Q1 saturates and all four LEDs illuminate together; when CTL is
low the array is dark. This lets a host pulse ("strobe") the emitters
synchronously — typically to reject ambient light and/or reduce average power by
sampling the detectors only while the emitters are on.

The 10 kΩ resistor **R10** pulls the CTL input to ground so the array stays dark
whenever CTL is undriven or J1 is unplugged; it draws negligible current when the
host drives CTL high and does not appreciably reduce Q1's base drive.

Peak collector current in Q1 is the sum of the four branches (**≈ 200 mA**), so
Q1 must be rated accordingly (see BOM notes).

### 1.2 Photodetectors (four independent channels)

```
 +5V ──[10kΩ]──┬──────► Vout (to J2)
               │ C
              (Q_photo, OP550)
               │ E
              GND
```

Each phototransistor (Q2–Q5, OP550) is wired in a common‑emitter / collector‑load
configuration: collector pulled to +5 V through a 10 kΩ load resistor (R6–R9),
emitter to ground. The output is taken at the collector:

* **Dark / no IR:** phototransistor off → output pulled high (≈ +5 V).
* **Illuminated:** phototransistor conducts → output pulled toward ground.

The four collector nodes are brought out on **J2 (Vout)** for reading by an ADC,
comparator, or logic input on the host. The four channels are electrically
independent; only the emitter array is strobed in common.

### 1.3 Notes and caveats

* **Supply bypassing is on‑board.** A bulk + ceramic pair (**C1 = 10 µF**,
  **C2 = 0.1 µF**) sits across the +5 V/GND entry at J4 to absorb the ~200 mA
  switching transient of the emitter array. No external decoupling is required,
  though additional bulk upstream never hurts on a long supply lead.
* **CTL has an on‑board pulldown.** The 10 kΩ resistor **R10** holds CTL low when
  it is undriven or unplugged, so the array defaults to dark. Any 3.3–5 V logic
  high will still switch Q1.
* **No reverse‑polarity or over‑current protection.** Observe supply polarity at
  J4 — note that C1 is polarized and a reversed supply will stress it along with
  Q1 and the phototransistors.

---

## 2. Bill of Materials (what to order)

Quantities are per board. All parts are through‑hole.

| Ref(s)          | Qty | Value / Part          | Package / Footprint                              | Notes |
|-----------------|-----|-----------------------|--------------------------------------------------|-------|
| R1              | 1   | 1 kΩ                  | Axial, ¼ W, 12.7 mm (0.5") pitch (DIN0411)       | Q1 base resistor |
| R2, R3, R4, R5  | 4   | 68 Ω                  | Axial, ¼ W, 12.7 mm pitch                        | LED current limit (~50 mA each) |
| R6, R7, R8, R9  | 4   | 10 kΩ                 | Axial, ¼ W, 12.7 mm pitch                        | Phototransistor collector load |
| R10             | 1   | 10 kΩ                 | Axial, ¼ W, 12.7 mm pitch                        | CTL pulldown |
| C1              | 1   | 10 µF                 | Radial electrolytic, 2.5 mm pitch (polarized)    | Bulk supply bypass at J4; observe polarity |
| C2              | 1   | 0.1 µF                | Ceramic disc, 5.0 mm pitch                       | HF supply bypass at J4 |
| D1, D2, D3, D4  | 4   | OP140 emitter         | 2.54 mm PinHeader 1x02                           | Connects to socket for emitter |
| Q2, Q3, Q4, Q5  | 4   | OP550 phototransistor | 2.54 mm PinHeader 1x02                           | Connects to socket for phototransistor |
| Q1              | 1   | NPN transistor        | TO‑92 (inline)                                   | ≥300 mA I_C, e.g. 2N2222A / PN2222A / 2N4401 |
| J1              | 1   | 1‑pin header          | 2.54 mm PinHeader 1×01                           | Connects to socket for control input |
| J2              | 1   | 4‑pin header          | 2.54 mm PinHeader 1×04                           | Connects to sockets for detector outputs |
| J3              | 1   | 5‑pin header          | 2.54 mm PinHeader 1×05                           | Connects to sockets for control/detector output |
| J4              | 1   | 2‑pin header          | 2.54 mm PinHeader 1×02                           | Power in: pin 1 = +5 V, pin 2 = GND |

**Emitter/detector wavelength.** OP140 (emitter) and OP550 (detector) are the
parts called out in the design. Confirm the detector's spectral response overlaps
the emitter's output for your specific parts before ordering in quantity, and
consult the current datasheets for lead configuration.


---

## 3. Assembly

The board is single‑sided‑population, through‑hole, hand‑solder friendly. Nothing
requires reflow.

### 3.1 Soldering order

Populate lowest‑profile parts first so the board sits flat while you work:

1. **Resistors R1–R10** (axial). Bend leads to the 12.7 mm pitch, seat flat,
   solder, and trim. Resistors are non‑polar. Double‑check value groups:
   * R1 = 1 kΩ (base)
   * R2, R3, R4, R5 = 68 Ω (LED series)
   * R6, R7, R8, R9 = 10 kΩ (detector load)
   * R10 = 10 kΩ (CTL pulldown)
2. **Ceramic cap C2** (0.1 µF, non‑polar) and **transistor Q1** (TO‑92). Seat Q1
   with a few mm of lead so the case is not stressed; solder and trim.
3. **Headers J1, J2, J3, J4**. Solder one pin, reflow while pressing the header
   flush and square, then solder the rest. Pin 1 of each connector is the
   **square/rectangular pad** on the board.
4. **Electrolytic cap C1** (10 µF, **polarized**). Match the `+` lead to the `+`
   pad / longer‑lead marking; the silkscreen band indicates the negative side.
5. **Emitters D1–D4 and detectors Q2–Q5** — see §3.2 / §3.3 for orientation.

Clean flux as appropriate and inspect for bridges, especially at the connector
rows.

### 3.2 Polarity — the emitters and detectors are polarized

Each of the eight emitter/detector positions is a **2‑pad footprint marked
`+ ⏚`** on the silkscreen (`+` = the driven/collector pad, `⏚` = the
ground/cathode side):

* **IR LEDs (D1–D4, OP140):** the `+` pad is the **anode** (toward the 68 Ω
  resistor / +5 V). The `⏚` pad is the **cathode** (toward Q1). Install with the
  correct anode/cathode orientation — a reversed LED will not emit.
* **Phototransistors (Q2–Q5, OP550):** the `+` pad is the **collector** (toward
  the 10 kΩ load and Vout). The `⏚` pad is the **emitter** (ground). A reversed
  phototransistor will not conduct correctly.

Confirm the anode/cathode and collector/emitter leads on your specific part's
datasheet before soldering.

### 3.3 Mounting the optoelectronics (remote)

Each emitter and detector lands on a **1×02, 2.54 mm through‑hole footprint**. The
optoelectronics are meant to be mounted **remotely**, not on the board: fit 2‑pin
headers in these positions and connect the actual OP140 / OP550 devices on flying
leads / a small harness. This lets the emitters and detectors be positioned on a
fixture (e.g. aimed across a gap or at a target) rather than fixed to the PCB.

Maintain the `+ ⏚` polarity described in §3.2 out at the remote device.

---

## 4. PCB Fabrication (what to order)

Generate Gerbers/drill from `IR sensor strobe.kicad_pcb` (KiCad 8:
*File → Fabrication Outputs → Gerbers* and *Drill Files*), or order directly from
the KiCad project at any standard fab. Specifications:

| Parameter        | Value                                  |
|------------------|----------------------------------------|
| Layers           | 2 (F.Cu / B.Cu)                        |
| Board size       | ≈ 37.5 mm × 43.5 mm                     |
| Thickness        | 1.6 mm                                  |
| Material         | FR‑4                                    |
| Finish           | HASL or ENIG (either is fine — all THT) |
| Min feature      | Default KiCad clearances; no fine‑pitch parts, so any vendor's economy process is adequate |
| Silkscreen       | Both sides used (title/attribution on back) |

There are no controlled‑impedance, high‑voltage, or thermal requirements.

---

## 5. Connector Pinout & Usage

| Connector | Pins | Signal | Direction | Description |
|-----------|------|--------|-----------|-------------|
| **J4** (Power) | 1 | +5 V | in | Board supply. |
|                | 2 | GND  | in | Supply return. |
| **J1** (CTL)   | 1 | CTL  | in | Strobe control. Logic high → all IR LEDs on. |
| **J2** (Vout)  | 1–4 | Vout[1..4] | out | The four phototransistor collector outputs. High ≈ dark, low ≈ illuminated. Read with ADC / comparator / logic input. |
| **J3** (GND)   | 1–5 | GND | — | Ground bus; all five pins common to GND. Use for detector/return grounds in the sensor harness. |

### Typical connection

1. Supply **+5 V / GND** to **J4** (on‑board bypass caps C1/C2 handle local
   decoupling — see §1.3).
2. Drive **J1 (CTL)** from a host GPIO/timer. Assert high while sampling to strobe
   the emitters; hold low to save power / go dark.
3. Read the four **J2 (Vout)** channels on the host, referenced to a **J3** ground.

---

## 6. Bring‑up / Test

1. **Unpowered:** verify J4 polarity. A continuity/resistance check from +5 V to
   GND will briefly show the bypass caps (C1/C2) charging before settling to a
   high resistance — it should **not** read a hard short. Confirm C1's polarity.
2. **Power, CTL low:** apply +5 V. Each Vout on J2 should read ≈ +5 V (detectors
   dark, pulled up through 10 kΩ). Quiescent current is small (just the detector
   loads).
3. **CTL high:** assert CTL. Supply current should rise by ~200 mA (the emitter
   array). Confirm IR emission with a phone camera (IR appears as a faint
   pale/purple glow on many cameras) or an IR‑sensitive card.
4. **Detector response:** with CTL high, illuminate a detector (reflect the
   emitter off a nearby surface, or point an IR source at it). Its Vout should
   drop toward ground and recover when the light is removed. Repeat per channel.
5. **Strobe:** pulse CTL and confirm Vout tracks the emitter on/off phases as
   expected for your sampling scheme.
