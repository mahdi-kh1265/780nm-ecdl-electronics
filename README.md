# 780nm-ecdl-electronics

Control, interface, piezo-actuation, and power electronics for POSM's 780 nm Littrow external-cavity diode laser.

This repository contains the custom electronics surrounding POSM's 780 nm ECDL used for rubidium spectroscopy, modulation-transfer spectroscopy (MTS), and laser-frequency stabilization.

The system is divided into three hardware projects:

1. **`ecdl-laser-bboard`** — laser/TEC controller interface, slow current bias, fast current modulation, monitoring, and Red Pitaya/FPGA connectivity.
2. **`780nm-ecdl-grating`** — high-voltage piezoelectric grating actuator driver for ECDL tuning, scanning, and slow lock correction.
3. **`780nm-grating-psu`** — dedicated low-voltage and high-voltage power supply for the grating driver. **Currently in development.**

---

# ECDL Hardware Context

The current system is designed around a 780 nm Littrow ECDL for the rubidium D2 region.

The broader laser assembly includes:

* 780 nm laser diode
* temperature-controlled TO-can mount
* diffraction grating in Littrow configuration
* piezoelectric grating actuator
* optical isolator
* free-space output optics
* fiber launch into the spectroscopy system
* MTS-based atomic frequency reference

The electronics in this repository do **not** contain the complete spectroscopy lock. MTS RF/demodulation electronics and FPGA lock logic are maintained separately.

---

# 1. `ecdl-laser-bboard`

## Laser / TEC Interface and Current-Control Baseboard

`ecdl-laser-bboard` is the central electrical interface between the laser assembly, commercial laser/TEC controller, Red Pitaya/FPGA control system, and monitoring hardware.

It is intentionally an **interface and control-conditioning board**, not a replacement precision laser-current driver.

The commercial LD/TEC controller remains responsible for the actual regulated laser current and TEC control.

# 2. `780nm-ecdl-grating`

## High-Voltage ECDL Grating / PZT Driver

The second major PCB drives the piezoelectric actuator mounted behind the Littrow grating.

Moving the piezo changes the diffraction-grating angle and external-cavity length, which changes the ECDL frequency.

The board is intended for:

* laser-frequency scanning
* finding rubidium spectroscopy features
* setting the PZT operating point
* lock acquisition
* slow frequency correction
* recentering the fast current actuator during long locks

---

# 3. `780nm-grating-psu`

## Dedicated Grating-Driver Power Supply

> **Current repository status: this PCB is still in development.**

The intended role of this board is to generate all rails required by `780nm-ecdl-grating` from a low-voltage system input.

Primary design goals include:

* low-noise analog rails
* controlled startup
* current limiting and fault protection
* sufficient +155 V power for dynamic piezo charging
* proper high-voltage spacing
* low coupling from the switching converters into the analog control circuitry

The committed KiCad project should be treated as a placeholder until the PSU design is frozen and reviewed.

# 4. Repository Structure

```text
780nm-ecdl-electronics/
│
├── ecdl-laser-bboard/
│   ├── ecdl-laser-bboard.kicad_sch
│   ├── sch2.kicad_sch
│   ├── sch3.kicad_sch
│   ├── ecdl-laser-bboard.kicad_pcb
│   ├── ecdl-laser-bboard.kicad_pro
│   └── libraries/
│
├── 780nm-ecdl-grating/
│   ├── 780nm-ecdl-grating.kicad_sch
│   ├── 780nm-ecdl-grating.kicad_pcb
│   ├── 780nm-ecdl-grating.kicad_pro
│   └── libraries/
│
├── 780nm-grating-psu/
│   ├── 780nm-grating-psu.kicad_sch
│   ├── 780nm-grating-psu.kicad_pcb
│   └── 780nm-grating-psu.kicad_pro
│
├── LICENSE
└── README.md
```

The current projects use **KiCad 10** format.

---

# 5. Major Components

| Function                      | Component                      | PCB                  |
| ----------------------------- | ------------------------------ | -------------------- |
| High-voltage PZT amplifier    | TI OPA462                      | `780nm-ecdl-grating` |
| Precision analog buffering    | TI OPA192                      | `780nm-ecdl-grating` |
| Precision reference           | ADR4550                        | `780nm-ecdl-grating` |
| Slow laser-current bias DAC   | TI DAC80501                    | `ecdl-laser-bboard`  |
| Precision analog conditioning | TI OPA4192                     | `ecdl-laser-bboard`  |
| True-zero negative supply     | TI LM7705                      | `ecdl-laser-bboard`  |
| Input protection / eFuse      | TPS25947 family                | `ecdl-laser-bboard`  |
| Low-noise power conversion    | LT862x family                  | `ecdl-laser-bboard`  |
| ECDL mount interface          | DB9 / DB15                     | `ecdl-laser-bboard`  |
| FPGA/control interface        | Red Pitaya header / analog I/O | `ecdl-laser-bboard`  |

Refer to the KiCad schematics rather than this table for current component values and installed options.

---

# 6. Recommended Bring-Up Sequence

The complete ECDL should **not** be the first load connected during PCB bring-up.

Each board should be tested independently first.

## `ecdl-laser-bboard`

1. Inspect for assembly faults and shorts.
2. Verify resistance from each power rail to ground.
3. Apply current-limited 12 V with the laser and TEC disconnected.
4. Verify eFuse behavior.
5. Verify all 5 V and logic rails.
6. Verify the LM7705 negative rail.
7. Verify DAC80501 communication over SPI.
8. Sweep the slow-bias DAC and measure `V_BIAS`.
9. Apply a known voltage to `RP_IN`.
10. Verify each fast-loop sensitivity setting.
11. Verify the complete summed `EXT_LD_SET` transfer function.
12. Verify `RP_EN` / `LD_SHD` behavior.
13. Inject known monitor voltages and verify each buffer.
14. Verify DB9/DB15 continuity pin-by-pin.
15. Connect the commercial LD/TEC controller without a laser.
16. Only after the interface is characterized should the real ECDL mount be connected.

## `780nm-ecdl-grating`

1. Bring up the low-voltage rails first.
2. Verify the precision reference.
3. Verify RP input buffering and filtering.
4. Verify the analog summer at low voltage.
5. Verify OPA462 enable/status behavior.
6. Apply the +155 V rail with a current limit and no piezo attached.
7. Command several static output voltages.
8. Verify `PIEZO_OUT` with a properly rated measurement setup.
9. Verify the monitor-divider calibration.
10. Test initially with a dummy capacitive load.
11. Apply triangle/ramp waveforms.
12. Verify stability, overshoot, heating, and slew behavior.
13. Connect the real grating piezo only after the electrical tests pass.

---

# 7. Characterization Data to Add

Once assembled, this repository should contain measured performance rather than only schematic intent.

Recommended measurements include:

## Laser baseboard

* 12 V input current
* rail noise
* DAC transfer function
* DAC INL/DNL where useful
* fast-input transfer function
* each fast sensitivity setting
* complete `EXT_LD_SET` transfer function
* monitor-channel gain and offset
* monitor noise
* shutdown latency
* temperature drift
* crosstalk between fast and slow controls

## Grating controller

* output voltage range
* command-to-PZT gain
* center-voltage adjustment range
* monitor-divider calibration
* output noise spectral density
* output ripple
* PZT step response
* triangle-wave fidelity
* capacitive-load stability
* maximum practical slew rate
* OPA462 temperature
* large-signal bandwidth

## Closed-loop ECDL

Eventually:

* PZT tuning coefficient in MHz/V
* current tuning coefficient in MHz/mA
* scan range
* mode-hop-free tuning range
* fast-loop bandwidth
* slow-loop/recenter behavior
* frequency-noise spectrum
* long-term lock stability
* actuator saturation statistics

A future structure could be:

```text
measurements/
├── laser-bboard/
│   ├── fast_path/
│   ├── slow_bias/
│   ├── monitors/
│   └── power/
│
├── grating/
│   ├── transfer_function/
│   ├── noise/
│   ├── pzt_response/
│   └── hv_monitor/
│
└── locked_ecdl/
    ├── tuning_coefficients/
    ├── spectra/
    ├── lock_performance/
    └── drift/
```

---

# 8. Safety

This repository contains circuits capable of damaging:

* laser diodes
* TECs
* piezoelectric actuators
* ADC/FPGA inputs
* test equipment
* and the operator if high-voltage nodes are handled improperly.

In particular:

### Laser diodes

Laser diodes can be destroyed by brief overcurrent, ESD, reverse voltage, connector mistakes, or uncontrolled startup transients.

### PZT driver

The grating board uses approximately **+155 V**.

Treat the high-voltage section as hazardous laboratory electronics even though available current is limited.

Do not probe `PIEZO_OUT` with equipment or accessories that are not appropriately voltage-rated.

### Connector pinouts

Always verify:

* DB9 pinout
* DB15 pinout
* laser polarity
* monitor-photodiode polarity
* TEC polarity
* thermistor connections
* controller common/reference

before connecting the actual laser mount.

---

# License

Released under the license included in [`LICENSE`](LICENSE).

---

## POSM

**Photonics and Optical Sciences Club at Minnesota**
University of Minnesota

[POSM website](https://posmphotonics.org/)
