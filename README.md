# Two-Stage Miller-Compensated Op-Amp

A two-stage CMOS operational amplifier in a TSMC 018 (0.18µm) process: a PMOS-input differential first stage with an NMOS current-mirror load, followed by a common-source second stage, with Miller compensation (including a nulling resistor) across the two stages.

## Topology
- **First stage:** PMOS differential pair `M1`, `M2` (l=1µ, w=10µ), tail current source `I1` = 8µA
- **First-stage load:** NMOS current mirror `M4`, `M5` (l=1µ, w=6µ)
- **Bias mirror:** `M6`, `M7` (l=1.5µ, w=6µ) generating the bias for the second stage
- **Second stage:** PMOS pull-up `M3` (l=1.5µ, w=162.5µ) and NMOS pull-down `M8` (l=1.5µ, w=35.5µ) — both large-width devices for gain and drive
- **Compensation:** Miller capacitor `C1` = 2p in series with nulling resistor `Rz` = 11k (moves the RHP zero), plus a separate load/output capacitor `C2` = 2p

## Tests Conducted
Split across two schematics — one for frequency-domain characterization, one for time-domain.

### AC Analysis (`2-stage_Opamp.asc`)
| Test | Command | Result |
|---|---|---|
| CMRR | `V3` on Vin2 = DC 0.8V, AC 1; `.ac dec 100 1 1G` | Common-mode gain ≈ −2.6dB → **CMRR = 78 − (−2.6) ≈ 80.6dB** |
| PSRR | `V1` on Vdd = DC 2.5V, AC 1; `.ac dec 100 1 1G` | Supply gain ≈ −5.19dB → **PSRR = 78 − (−5.19) ≈ 83.2dB** |

### Transient Analysis (`2-stage_Opamp_TRAN.asc`)
| Test | Command | What it measures |
|---|---|---|
| Slew rate | `.tran 0 10u 0 1n` with `.meas` directives timing V(vout) between 0.88V and 1.52V on both rising and falling edges | Rising and falling slew rate (`SR_rise`, `SR_fall`), computed as ΔV / Δt between the two threshold crossings |
| Output voltage swing | Unity-gain follower, `.dc V3 0 2.6 1m` | Output tracks input until ≈2.48V — **swing ≈ 0.7mV to 2.479V** on a 2.5V supply (upper-side headroom loss ≈21mV) |
| Input-referred offset voltage | Unity-gain configuration (`R1` = 1Ω feedback), `.op`, V_os = V(in1,DC) − V(out,DC) | V_os = 0 in simulation (no device mismatch modeled — this measures the ideal/simulated offset, not a real-silicon expectation) |

## Repository Structure
```
Two-Stage-Miller-Compensated-Op-Amp/
├── README.md
│
├── Schematics/
│   ├── 2-stage_Opamp.asc               # AC analysis: CMRR, PSRR
│   └── 2-stage_Opamp_TRAN.asc          # Transient analysis: slew rate, swing, offset
│
├── AC Tests/
│   ├── cmrr/                           # screenshot 
│   └── psrr/
│
└── Transient Tests/
    ├── slew_rate/
    ├── output_voltage_swing/
    └── input_offset_voltage/
```
> Note: This design uses `tsmc018.lib` (TSMC 0.18µm PDK), included via `.include tsmc018.lib` in all three schematics. Since this is a licensed foundry PDK, not committing the library file itself to the public repo.
