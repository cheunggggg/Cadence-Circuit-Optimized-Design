# 4-Bit Carry Lookahead Adder(VLSI Design)

Transistor-level design and simulation of a 4-bit **Carry Lookahead Adder (CLA)** in Cadence Virtuoso. Carry outputs C1–C4 are computed in parallel using optimized NAND-NAND logic with sized CMOS devices targeting minimum Energy-Delay Product (EDP).

**Team:** Selena Yu, Brandon Cheung — UCLA ECE 115C, SPRING 2025

---

## Performance Summary

### Simulated

| Parameter | VDD = 1.1V | VDD = 0.9V (Optimum) |
|-----------|-----------|----------------------|
| C0→C4 Delay | 71.74 ps | 107.05 ps |
| Power @ 100 MHz | 51.39 µW | 32.46 µW |
| Energy | 0.514 pJ | 0.325 pJ |
| EDP | 3.69 × 10⁻²³ J·s | 3.47 × 10⁻²³ J·s |

### Hand-Calculated

| Parameter | VDD = 1.1V | VDD = 0.9V |
|-----------|-----------|------------|
| C0→C4 Delay | 69.56 ps | 93.667 ps |
| Power @ 100 MHz | — | 34.07 µW |
| Energy | — | 0.341 pJ |
| EDP | — | 3.19 × 10⁻²³ J·s |

**Chosen optimum VDD: 0.9 V** — selected by minimizing EDP across a hand-calculated sweep from 0.7 V to 1.1 V.

---

## Architecture

C1, C2, C3, and C4 are computed in parallel using a two-stage NAND-NAND implementation of the standard CLA boolean equations. Each carry output has its own dedicated path:

- **C1** — Inverter + NAND2
- **C2** — Inverter + NAND2 + NAND3
- **C3** — Inverter + NAND2 + NAND3 + NAND4
- **C4** — Inverter + NAND2 + NAND3 + NAND4 + NAND5 *(critical/longest path)*

The external load on each output is **20 fF**.

---

## Device Sizing

Widths were calculated by optimizing the logical effort fan-out (`fopt = ⁿ√(GBH)`) per path. G^(1/2) = 2.6, B = 1. The C4 path was sized first with `fopt = 5`; remaining paths were shifted to ensure C4 holds the worst-case delay while minimizing total power.

| Path | fopt |
|------|------|
| C1 | 7 |
| C2 | 7 |
| C3 | 3.4 |
| C4 | 5 |

### Device Widths (µm)

<details>
<summary>C1 Path</summary>

| Gate | NMOS | PMOS |
|------|------|------|
| Inverter | 0.35 | 0.52 |
| NAND2 | 0.69 | 0.52 |
| NAND2 (end) | 3.46 | 2.60 |

</details>

<details>
<summary>C2 Path</summary>

| Gate | NMOS | PMOS |
|------|------|------|
| Inverter | 0.45 | 0.67 |
| NAND2 | 0.89 | 0.67 |
| NAND3 | 1.72 | 0.86 |
| NAND3 (end) | 6.67 | 3.34 |

</details>

<details>
<summary>C3 Path</summary>

| Gate | NMOS | PMOS |
|------|------|------|
| Inverter | 2.30 | 3.45 |
| NAND2 | 4.60 | 3.45 |
| NAND3 | 8.87 | 4.43 |
| NAND4 | 14.41 | 5.40 |
| NAND4 (end) | 22.34 | 8.38 |

</details>

<details>
<summary>C4 Path (critical)</summary>

| Gate | NMOS | PMOS |
|------|------|------|
| Inverter | 3.88 | 1.26 |
| NAND2 | 2.53 | 1.89 |
| NAND3 | 4.87 | 2.43 |
| NAND4 | 7.93 | 2.97 |
| NAND5 | 11.74 | 3.52 |
| NAND5 (end) | 22.55 | 6.75 |

</details>

Gates labeled "(end)" are the final stage driving the 20 fF external load capacitance.

---

## Simulation & Verification

Simulations were run in **Cadence Virtuoso Maestro** with a parametric sweep of all inputs (C0, G0–G3, P0–P3) over logic 0/1 combinations. Approximately 30 input combinations were verified against a hand-derived truth table — all outputs matched.

Delay was measured as the average of rise-time and fall-time delays at the 50% VDD crossing. The transient response confirmed that C4 is consistently the last output to cross the 50% threshold, validating it as the critical path.

---

## Repository Structure

```
cadence files/
├── final/          ← Full CLA schematic and final Maestro simulation
│   ├── schematic/  ← Cadence .oa schematic (all four carry paths)
│   └── maestro/    ← Simulation setup, parametric sweep, results
├── fanout/         ← Fanout / logical effort exploration testbench
│   ├── schematic/
│   └── maestro/
└── power/          ← Power characterization testbench
    ├── schematic/
    └── maestro/
```

All schematic and simulation files are in Cadence OpenAccess (`.oa`) format, intended for use with **Cadence Virtuoso**.

---

## Area Estimates

Estimated from stick diagrams using λ = 25 nm:

| Circuit | Estimated Area |
|---------|---------------|
| C1 | 1.84 × 10⁻¹² m² |
| C2 | 3.36 × 10⁻¹² m² |
| C3 | 5.44 × 10⁻¹² m² |
| C4 | 8.16 × 10⁻¹² m² |
| **Total** | **1.88 × 10⁻¹¹ m²** |
