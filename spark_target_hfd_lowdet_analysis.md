# Spark Target | High Fuel Demand, Low-Det — ROM Structure Analysis

## Scope

Analysis of the 3D table **"Spark Target | High Fuel Demand, Low-Det"** across 103 MX-5 (NC Miata) ROMs in the `metadata/` directory. L3R3EE is excluded (Mazda6, no spark tables).

All tables in the "Spark Target - Base" category are **factory Mazda calibration**. Tables prefixed with `[Flex]`, along with Spark Target Blend, Flat Shift, Launch Control, MAF Emulation (SD/Alpha-N), and anything in a `Patch` category, are **Speeps aftermarket patches** injected at fixed ROM locations.

---

## Two Generations: NC1 vs NC2/NC3

The 103 ROMs split cleanly into two structural families based on table dimensions:

| Generation | Grid Size | Elements | Y-Axis (RPM rows) | ROM Count |
|------------|-----------|----------|--------------------|-----------|
| **NC1**    | 15 x 14   | 210      | 14                 | 48        |
| **NC2/NC3**| 15 x 15   | 225      | 15                 | 55        |

NC2 added an extra RPM row. No ROM has any other element count — the split is binary with zero exceptions.

### Axis Offsets (constant within each generation, zero outliers)

| Generation | X-Axis offset from table | Y-Axis offset from table |
|------------|--------------------------|--------------------------|
| NC1        | -60 bytes                | -116 bytes               |
| NC2/NC3    | -60 bytes                | -120 bytes               |

The Y-axis difference (-116 vs -120) reflects the extra RPM row: 15 × 4 = 60 vs 14 × 4 = 56 bytes of axis data, plus alignment.

---

## Factory Base Table Layout (8 tables, all 103 ROMs)

Table ordering by address is **identical across all 103 ROMs**:

| Position | Table Name                                        | Elements |
|----------|---------------------------------------------------|----------|
| 0        | Spark Target \| High Fuel Demand, High-Det, IMRC  | 120/128* |
| 1        | Spark Target \| High Fuel Demand, High-Det         | 210/225* |
| 2        | Spark Target \| Low Fuel Demand, High-Det, IMRC   | 120/128* |
| 3        | Spark Target \| Low Fuel Demand, High-Det           | 210/225* |
| 4        | Spark Target \| High Fuel Demand, Low-Det, IMRC   | 120/128* |
| 5        | **Spark Target \| High Fuel Demand, Low-Det**       | **210/225*** |
| 6        | Spark Target \| Low Fuel Demand, Low-Det, IMRC    | 120/128* |
| 7        | Spark Target \| Low Fuel Demand, Low-Det            | 210/225* |

*NC1/NC2 respectively

### Inter-Table Spacing (relative to HFD Low-Det = 0)

| Table                          | NC1 Offset | NC2/NC3 Offset |
|--------------------------------|------------|----------------|
| HFD, High-Det, IMRC           | -3652      | -3784          |
| HFD, High-Det                 | -3056      | -3184          |
| LFD, High-Det, IMRC           | -2124      | -2192          |
| LFD, High-Det                 | -1528      | -1592          |
| HFD, Low-Det, IMRC            | -596       | -600           |
| **HFD, Low-Det**              | **0**      | **0**          |
| LFD, Low-Det, IMRC            | +932       | +992           |
| LFD, Low-Det                  | +1528      | +1592          |

These offsets are **rock-solid within each generation** — verified across all 103 ROMs with zero exceptions. The entire 8-table block moves as a unit.

---

## Sub-Groups Within Each Generation

### NC1 — 5 Sub-Groups (48 ROMs, no DBW)

| Sub-Group | HFD Low-Det Address | Delta from A1 | Count | ROM IDs |
|-----------|---------------------|---------------|-------|---------|
| A1 | 0xCF330 | +0      | 6  | L831ED, LFG2EE, LFG3EE, LFG7ED, LFH9EC, LFJ1EC |
| A2 | 0xCF3A4 | +0x74   | 15 | LFG1EK, LFG1ER, LFG2EJ, LFG2EL, LFG2EM, LFG2EP, LFG3EK, LFG3EN, LFN3ER, LFN4EK, LFN4EP, LFN5EN, LFZZE0, LFZZEA, LSYAEA |
| A3 | 0xCF3B4 | +0x84   | 4  | L831EE, LFG2EF, LFG3EF, LFH9ED |
| A4 | 0xCF3D4 | +0xA4   | 14 | L831EF, L831EG, LF6JE0, LFG2EG, LFG3EG, LFG7EG, LFG8EF, LFG8EG, LFG9EH, LFH9EE, LFH9EF, LFJ1EE, LFJ1EF, LFJ5EG |
| A5 | 0xCF42C | +0xFC   | 9  | LF4WEG, LF4XED, LF4XEE, LF4XEG, LF5AEG, LF5BEG, LF5CEG, LF5DEG, LF8HED |

All NC1 ROMs have `has_dbw_bos = false`. The deltas (+0x74, +0x84, +0xA4, +0xFC) are not regular intervals — each represents a distinct firmware revision where code/data preceding the spark tables grew by different amounts.

### NC2/NC3 — 12 Sub-Groups (55 ROMs)

#### NC2 Early — No DBW (17 ROMs)

| Sub-Group | Address  | Delta from B1a | Count | ROM IDs |
|-----------|----------|----------------|-------|---------|
| B1a | 0xD0B60 | +0      | 5  | L843EC, LF9HED, LF9JED, LF9KEC, LFAKEB |
| B1b | 0xD0C34 | +0xD4   | 6  | LFDJEA, LFDKEB, LFDLEC, LFDMEA, LFDNEB, LFDPEC |
| B1c | 0xD0E4C | +0x2EC  | 6  | LF9VEB, LF9WEB, LF9XEB, LFAAEB, LFABEB, LFACEB |

#### NC2 Late — DBW Added (12 ROMs)

| Sub-Group | Address  | Delta from B1a | Count | ROM IDs |
|-----------|----------|----------------|-------|---------|
| B2a | 0xD1280 | +0x720  | 2  | LF9HEE, LF9KEE |
| B2b | 0xD15B4 | +0xA54  | 3  | LF9RED, LF9SED, LF9TEE |
| B2c | 0xD1614 | +0xAB4  | 3  | LFGKEC, LFGMEC, LFGNEC |
| B2d | 0xD17F8 | +0xC98  | 4  | L862ED, LFFCED, LFFDED, LFFEEE |

#### NC3 — DBW, Large Address Shift (26 ROMs)

| Sub-Group | Address  | Delta from B1a | Count | ROM IDs |
|-----------|----------|----------------|-------|---------|
| B3a | 0xD37FC | +0x2C9C  | 2  | LFLGEC, LFMSEA |
| B3b | 0xD3B30 | +0x2FD0  | 2  | LFLVEB, LFLWEB |
| B3c | 0xD9614 | +0x8AB4  | 6  | LFGJED, LFGKED, LFGLEF, LFGMEE, LFGNEE, LFGPEF |
| B3d | 0xDBD04 | +0xB1A4  | 15 | LFLMEA, LFLMEC, LFLMED, LFLNEC, LFLPE0, LFLPEA, LFLPEB, LFLPEC, LFLPED, LFLREA, LFLXE0, LFLXEC, LFMAE0, LFMAEB, LFMAEC |
| B3e | 0xDBD80 | +0xB220  | 1  | LFLEEC |

---

## Key Structural Boundaries

### NC1 → NC2 Boundary
- Table grid changes from 15x14 to 15x15 (extra RPM row)
- Base address jumps from ~0xCFxxx to ~0xD0xxx (~6KB shift)
- Inter-table spacing changes (e.g., IMRC→non-IMRC: 596→600 bytes)

### NC2-early → NC2-late Boundary (DBW Insertion)
- DBW Brake Override logic added to firmware
- Base tables shift ~+0x720 to +0xC98 (1,824–3,224 bytes) from the DBW code insertion
- `has_dbw_bos` flips from `false` to `true` — **perfect binary classifier, zero exceptions**
- Same ROM ID prefixes appear on both sides (LF9H: ED=pre-DBW, EE=post-DBW; LF9K: EC=pre-DBW, EE=post-DBW)

### NC2 → NC3 Boundary
- Same table dimensions (15x15), same inter-table spacing
- Large base address jump — the LFGK/LFGM/LFGN families show an **exact 0x8000 (32,768 byte) shift**:
  - LFGKEC (NC2-late): 0xD1614 → LFGKED (NC3): 0xD9614
  - LFGMEC (NC2-late): 0xD1614 → LFGMEE (NC3): 0xD9614
  - LFGNEC (NC2-late): 0xD1614 → LFGNEE (NC3): 0xD9614
- Both NC2-late and NC3 have DBW — DBW alone doesn't distinguish them

---

## Speeps Patch Tables (not factory)

All `[Flex]` spark target tables, Spark Target Blend, and other patch-category tables are injected by Speeps at **fixed absolute addresses** that do not shift between firmware revisions:

| Family   | Flex Anchor Address | Notes |
|----------|---------------------|-------|
| NC1      | 0xEE2EC             | Identical across all 48 ROMs |
| NC2/NC3  | 0xEE3EC             | Identical across all 55 ROMs |

The Flex block ordering differs from the base block: Flex groups by fuel demand first (all HFD, then all LFD), while the base block interleaves HFD/LFD within each det level.

---

## ROM ID Prefix Patterns

| Generation | Prefixes |
|------------|----------|
| NC1 | L831, LF4W, LF4X, LF5A–D, LF6J, LF8H, LFG1–3, LFG7–9, LFH9, LFJ1, LFJ5, LFN3–5, LFZZ, LSYA |
| NC2 early | L843, LF9H(ED), LF9J, LF9K(EC), LF9V–X, LFAA–AC, LFAK, LFDx |
| NC2 late | L862, LF9H(EE), LF9K(EE), LF9R–T, LFFx, LFGK/M/N(EC) |
| NC3 | LFGJ–GP(ED/EE/EF), LFLx, LFMx |

Cross-generation prefixes (same base, different suffix = different generation):
- **LF9H**: ED = NC2-early, EE = NC2-late
- **LF9K**: EC = NC2-early, EE = NC2-late
- **LFGK/LFGM/LFGN**: EC = NC2-late, ED/EE = NC3

---

## Anomalies

- **LFLPEB**: Only ROM with `checksummodule = None` (all others: `21053000`). Structurally identical to its B3d siblings otherwise. May need special handling for checksum correction.
- **LFLEEC**: Singleton sub-group at 0xDBD80, +0x7C (124 bytes) above the 15-ROM B3d group. Minor firmware revision; structurally identical otherwise.
- **Trailing space**: The XML name `"Spark Target | Low Fuel Demand, Low-Det "` has a trailing space in all 103 XMLs. Not causing issues in current analysis but will break exact string matching if not stripped.

---

## Scaling IDs

Scaling IDs are simply the **decimal representation of the address** — they carry no independent information and change whenever addresses change.
