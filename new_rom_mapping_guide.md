# How to Map Tables for a New Undocumented NC Miata ROM

This guide describes a systematic procedure for identifying the generation and predicting table addresses for a new ROM that is not yet in the metadata directory. It is based on structural analysis of all 103 documented MX-5 NC ROMs.

---

## Overview

All NC Miata ROMs share the same SH7058 processor and the same fundamental table structure. What varies between ROMs is:

1. **Generation** — determines table dimensions and inter-table spacing
2. **Firmware revision** — shifts the entire base calibration block by a uniform offset
3. **Speeps patch presence** — [Flex], Launch Control, Flat Shift, MAF Emulation (SD/Alpha-N), and Patch-category tables are aftermarket additions at fixed addresses, not factory

The key insight: **within a generation, all 8 factory Spark Target base tables (and by extension, all factory calibration tables) move as a rigid block.** If you can locate one table, you can compute the offset delta from a known ROM and predict all others.

---

## Step 1: Determine the Generation

You need three discriminators, checked in order:

### Discriminator A: Spark Target Table Dimensions

Locate the "Spark Target | High Fuel Demand, Low-Det" table (or any of the 8 base spark target tables). Check its element count:

| Element Count | Grid Size | Generation |
|---------------|-----------|------------|
| 210           | 15 × 14   | **NC1.0**  |
| 225           | 15 × 15   | **NC2.0 or NC3.0** (proceed to B) |

If you don't have the spark target table yet, the Y-axis element count alone is sufficient: 14 rows = NC1, 15 rows = NC2/NC3.

### Discriminator B: DBW Brake Override Presence

Search for any table with `category="DBW - Brake Override"`. This is a binary test with zero false positives or negatives across all 103 known ROMs.

| DBW Brake Override | Generation |
|--------------------|------------|
| Absent             | **NC2.0 (Early)** |
| Present            | **NC2.0 (Late) or NC3.0** (proceed to C) |

### Discriminator C: Base Table Address Range

Check the address of "Spark Target | High Fuel Demand, Low-Det":

| Address Range         | Generation |
|-----------------------|------------|
| 0xD1280 – 0xD17F8    | **NC2.0 (Late)** |
| 0xD37FC – 0xDBD80    | **NC3.0** |

The gap between the highest NC2-late address (0xD17F8) and the lowest NC3 address (0xD37FC) is over 8KB — there is no ambiguity zone.

**Alternative C discriminator using ROM ID prefixes:**
- Prefixes `LFL`, `LFM`, `LFGJ`, `LFGK`(ED+), `LFGM`(EE+), `LFGN`(EE+), `LFGP` → NC3.0
- Prefixes `L862`, `LF9H`(EE), `LF9K`(EE), `LF9R`, `LF9S`, `LF9T`, `LFF`, `LFGK`(EC), `LFGM`(EC), `LFGN`(EC) → NC2.0 (Late)

Note: The suffix letter matters — LF9HED is NC2-early while LF9HEE is NC2-late.

### Decision Tree Summary

```
New ROM
  │
  ├─ Elements = 210 (15×14) ──────────────────── NC1.0
  │
  └─ Elements = 225 (15×15)
       │
       ├─ No DBW Brake Override ──────────────── NC2.0 (Early)
       │
       └─ Has DBW Brake Override
            │
            ├─ HFD Low-Det addr < 0xD2000 ───── NC2.0 (Late)
            │
            └─ HFD Low-Det addr ≥ 0xD2000 ───── NC3.0
```

---

## Step 2: Select a Reference ROM

Once you know the generation, pick the **closest known sub-group** as your reference. The reference ROM should ideally share the same ROM ID prefix as your new ROM.

### Reference Sub-Groups

#### NC1.0 Reference Points

| Sub-Group | HFD Low-Det Addr | Representative ROM | Best for prefixes |
|-----------|-------------------|--------------------|-------------------|
| A1 | 0xCF330 | L831ED  | L831(ED), LFG2(EE), LFG3(EE), LFG7(ED), LFH9(EC), LFJ1(EC) |
| A2 | 0xCF3A4 | LFG1EK  | LFG1, LFG2(EJ+), LFG3(EK+), LFN, LFZZ, LSYA |
| A3 | 0xCF3B4 | L831EE  | L831(EE), LFG2(EF), LFG3(EF), LFH9(ED) |
| A4 | 0xCF3D4 | L831EF  | L831(EF+), LF6J, LFG2(EG), LFG3(EG), LFG7(EG), LFG8, LFG9, LFH9(EE+), LFJ1(EE+), LFJ5 |
| A5 | 0xCF42C | LF4WEG  | LF4W, LF4X, LF5, LF8H |

#### NC2.0 (Early) Reference Points

| Sub-Group | HFD Low-Det Addr | Representative ROM | Best for prefixes |
|-----------|-------------------|--------------------|-------------------|
| B1a | 0xD0B60 | L843EC  | L843, LF9H(ED), LF9J, LF9K(EC), LFAK |
| B1b | 0xD0C34 | LFDJEA  | LFDx |
| B1c | 0xD0E4C | LF9VEB  | LF9V–X, LFAA–AC |

#### NC2.0 (Late) Reference Points

| Sub-Group | HFD Low-Det Addr | Representative ROM | Best for prefixes |
|-----------|-------------------|--------------------|-------------------|
| B2a | 0xD1280 | LF9HEE  | LF9H(EE), LF9K(EE) |
| B2b | 0xD15B4 | LF9RED  | LF9R, LF9S, LF9T |
| B2c | 0xD1614 | LFGKEC  | LFGK(EC), LFGM(EC), LFGN(EC) |
| B2d | 0xD17F8 | L862ED  | L862, LFF |

#### NC3.0 Reference Points

| Sub-Group | HFD Low-Det Addr | Representative ROM | Best for prefixes |
|-----------|-------------------|--------------------|-------------------|
| B3a | 0xD37FC | LFLGEC  | LFLG, LFMS |
| B3b | 0xD3B30 | LFLVEB  | LFLV, LFLW |
| B3c | 0xD9614 | LFGJED  | LFGJ, LFGK(ED+), LFGL, LFGM(EE+), LFGN(EE+), LFGP |
| B3d | 0xDBD04 | LFLMEA  | LFLM, LFLN, LFLP, LFLR, LFLX, LFMA |
| B3e | 0xDBD80 | LFLEEC  | LFLE |

---

## Step 3: Locate the Anchor Table in the New ROM

You need to find the address of **one known table** in the new ROM binary. The best anchor is "Spark Target | High Fuel Demand, Low-Det" because:

- It exists in all 103 known ROMs
- It's a 3D table with a distinctive data pattern (spark advance values in float format)
- Its dimensions are generation-specific (210 or 225 elements)

### How to Find It in a Raw Binary

1. **Byte-scan for the Y-axis data.** The Y-axis immediately precedes the X-axis, which immediately precedes the table data. The Y-axis contains RPM breakpoints as big-endian floats (e.g., 500.0, 1000.0, 1500.0, ... 7000.0 or 7500.0).

2. **Search for the RPM float sequence.** Common RPM breakpoints stored as IEEE 754 big-endian floats:
   - 500.0 = `0x43FA0000`
   - 1000.0 = `0x447A0000`
   - 1500.0 = `0x44BB8000`
   - 2000.0 = `0x44FA0000`

3. **Verify by checking the X-axis.** The X-axis should contain LOAD breakpoints as big-endian floats, typically values like 4.2, 5.6, 7.0, ... up to ~22.0.

4. **The table data starts 60 bytes after the X-axis start** (X-axis offset = -60 from table).

### Expected Address Ranges by Generation

| Generation | Expected Address Range |
|------------|----------------------|
| NC1.0      | 0xCF300 – 0xCF500   |
| NC2.0 (Early) | 0xD0B00 – 0xD0F00 |
| NC2.0 (Late)  | 0xD1200 – 0xD1900 |
| NC3.0      | 0xD3700 – 0xDBE00   |

---

## Step 4: Compute the Offset Delta

Once you have the new ROM's anchor table address:

```
delta = new_rom_anchor_address - reference_rom_anchor_address
```

Example: New ROM's HFD Low-Det is at 0xD0BE0, reference B1a (L843EC) is at 0xD0B60:
```
delta = 0xD0BE0 - 0xD0B60 = +0x80 (128 bytes)
```

### Validate the Delta

The delta should be **the same for all factory base tables** within the same category block. Verify by checking 2-3 other tables:

- Check "Spark Target | High Fuel Demand, High-Det" — should be at `reference_addr + delta`
- Check a table from a different category (e.g., "Fuel Target OL") — should also show the same delta if it's in the same calibration block

**WARNING:** Different category blocks may have different deltas. The spark tables, fuel tables, and other calibration blocks may shift independently. Always verify the delta for each major category block separately.

---

## Step 5: Predict All Table Addresses

Apply the delta to every table address from the reference ROM's metadata XML:

```
new_address = reference_address + delta
```

### Inter-Table Offsets (Constant Within Generation)

These offsets from "HFD Low-Det" are guaranteed constant — use them to cross-validate:

| Table (relative to HFD Low-Det = 0) | NC1.0 | NC2.0/NC3.0 |
|--------------------------------------|-------|-------------|
| HFD, High-Det, IMRC                 | -3652 | -3784       |
| HFD, High-Det                       | -3056 | -3184       |
| LFD, High-Det, IMRC                 | -2124 | -2192       |
| LFD, High-Det                       | -1528 | -1592       |
| HFD, Low-Det, IMRC                  | -596  | -600        |
| **HFD, Low-Det**                    | **0** | **0**       |
| LFD, Low-Det, IMRC                  | +932  | +992        |
| LFD, Low-Det                        | +1528 | +1592       |

---

## Step 6: Handle Speeps Patch Tables

Speeps patch tables ([Flex], Launch Control, Flat Shift, MAF Emulation, Patch-category items) are **not predicted by offset deltas**. They are injected at fixed absolute addresses:

| Family   | Flex Spark Anchor | Flex Blend |
|----------|-------------------|------------|
| NC1.0    | 0xEE2EC           | 0xF0D00    |
| NC2.0+   | 0xEE3EC           | 0xF1080    |

If the new ROM has Speeps patches, these addresses should be used as-is. If it doesn't have patches yet, these addresses indicate where patches will be placed by Speeps' tooling.

---

## Step 7: Validate the Full Map

After predicting all addresses, validate by:

1. **Read the predicted address** in the binary — does the data make sense for that table type?
2. **Check axis values** — RPM and LOAD breakpoints should be physically reasonable
3. **Cross-check inter-table offsets** — verify the constant offsets from the table above
4. **Compare scaling** — scaling IDs are just decimal representations of addresses, so `new_scaling = decimal(new_address)`

---

## Quick Reference: Generation Identification Cheat Sheet

| Property | NC1.0 | NC2.0 (Early) | NC2.0 (Late) | NC3.0 |
|----------|-------|----------------|---------------|-------|
| Grid size | 15×14 | 15×15 | 15×15 | 15×15 |
| Elements | 210 | 225 | 225 | 225 |
| DBW BOS | No | No | **Yes** | **Yes** |
| HFD Low-Det range | 0xCF3xx | 0xD0Bxx–0xD0Exx | 0xD12xx–0xD17xx | 0xD37xx–0xDBDxx |
| Y-axis rows | 14 | 15 | 15 | 15 |
| ROM count | 48 | 17 | 12 | 26 |
| ROM prefixes | L831, LF4-8, LFG1-9, LFH9, LFJ, LFN, LFZZ | L843, LF9H-K(early), LF9V-X, LFA, LFD | L862, LF9(late), LFF, LFGx(EC) | LFGx(ED+), LFL, LFM |

---

## Worked Example: Mapping a Hypothetical New ROM "LFG2EN"

1. **Prefix analysis**: LFG2 prefix appears in NC1.0 across sub-groups A1–A4. The suffix "EN" is between "EM" (A2, addr 0xCF3A4) and "EP" (A2, addr 0xCF3A4).

2. **Hypothesis**: LFG2EN is likely NC1.0, sub-group A2.

3. **Verify**: Open the binary, search for the RPM float sequence near 0xCF3xx. If found at 0xCF3A4 — exact match to A2, delta = 0. If found at a nearby address, compute delta.

4. **Suppose** HFD Low-Det is at 0xCF3C0 (delta = +0x1C from A2 reference 0xCF3A4).

5. **Predict** all other table addresses by adding +0x1C to every address in the LFG1EK (A2 reference) metadata XML.

6. **Cross-validate**: Check that HFD Low-Det IMRC is at 0xCF3C0 - 596 = 0xCF174. Read that address — should contain 120 float elements of spark advance data.

---

## Known Edge Cases

- **LFLPEB**: Missing checksummodule value (all other ROMs have `21053000`). May need special handling for checksum correction.
- **LFLEEC**: Singleton sub-group, +124 bytes offset from the nearest NC3 group (B3d). Likely a minor firmware revision.
- **Trailing space**: The XML table name `"Spark Target | Low Fuel Demand, Low-Det "` has a trailing space in all 103 XMLs. Strip whitespace when doing name matching.
- **L3R3EE**: Mazda6 ROM, not MX-5. Has no spark target tables. Do not use as a reference.
- **Cross-generation prefixes**: LF9HED (NC2-early) vs LF9HEE (NC2-late), LFGKEC (NC2-late) vs LFGKED (NC3). The suffix letter indicates the revision — always check, don't rely on prefix alone.
