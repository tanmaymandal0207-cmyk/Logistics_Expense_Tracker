# Restructuring notes

How each original sheet maps to the clean layer, what was wrong with it, and how the result was validated.
Counts refer to the full dataset. Example values are from the anonymised sample.

---

## 1. Auburn Load → AUBURN LOAD CLEAN (table `AUBURN`)

**Original:** table `Table1`, 23 columns, 1,264 rows of which 150 were blank. 1,114 real loads.

| Original column | Issue found | Clean column(s) | Treatment |
|---|---|---|---|
| `S.NO.` | Hand-typed. Contained a stray `S` and didn't run 1…n | `S.NO.` | `=IF($B7="","",ROW()-6)` |
| `Month` | 29 labels: `May`, `JUNE `, `JUNE,26`, `OCTOBER,25`, plus one real date | `MONTH` | `=EOMONTH(date,-1)+1`, the first day of the month |
| `LOADING DATE ` | 5 values stored as text (`10.05.2024`, `26-12-204`, `01-03-2025`…) | `LOADING DATE` | Converted to dates. ⚠ `26-12-204` became 2025-12-26 and should be 2024-12-26 (see exceptions) |
| `Provider` | 8 spellings | `PROVIDER` | `BN` / `3rd Party` |
| `LORRY NO.` | Same vehicle in several formats | `LORRY NO.` | Kept as logged (open item: vehicle master) |
| `MOBILE NO.` | 2 cells typed as `-` | `MOBILE NO.` | Kept |
| `FROM `, `POINT GR NO.`, `TO GR NO.` | Trailing spaces, mixed separators | same | Trimmed |
| `LORRY MAKE` | 29 raw spellings | `LORRY MAKE` | Trimmed, down to 21. Master mapping still open |
| ` LORRY RATE ` | 1 text value (`-`) | `LORRY RATE (₹)` | Numeric only |
| `Billing Rate Charges` | 1 text value | `BILLING RATE (₹)` | Numeric only |
| `Net Saving` | Mix of formulas and 16 typed numbers | `NET SAVING (₹)`, `MARGIN %` | One formula per column |
| `Adance Paid (INR)` | 1 text value, 44 blanks | `ADVANCE PAID (₹)` | Numeric. Blanks counted in KPI *No advance logged* |
| `Balance To pay` | Mix of formulas and 32 typed numbers | `BALANCE TO PAY (₹)` | `=LORRY − ADVANCE` |
| `BILL STATUS `, `Payment Status` | Trailing spaces | `BILL STATUS`, `PAYMENT STATUS` | Trimmed |
| `Invoice No.` | Text and numbers mixed (`01`, `295`) | `INVOICE NO.` | Numeric (7 text values remain) |
| `REACH UNLOADING POINT ` | 1,039 text values (`dd-mm-yyyy`, `SAME DAY`, `LOADING`) | `REACH DATE` + `REACH AS LOGGED` | Parsed to a date. Original text kept |
| `UNLOADING DATE ` | 1,040 text values, some holding two dates (`15/05/2024./18/05/2024`) | `UNLOAD DATE`, `UNLOAD END` + `UNLOAD AS LOGGED` | Split into start/end. Original text kept |
| — | — | `TRANSIT DAYS` | New: `=UNLOAD DATE − LOADING DATE` |
| `Current Status` | `Closed`, `closed`, `closed `, `-` | `CURRENT STATUS` | `Closed` / `Running` |
| `BROAKER ADRESS` | Typo in header, 48 raw values, near-duplicates | `BROKER ADDRESS` | Trimmed (45). Party master still open |
| `REMARKS ` | Free text | `REMARKS` | Kept |

The `AS LOGGED` columns (AA:AB) are grouped and collapsed. They exist only so that every parsed date can be traced back to what was typed.

## 2. Market Load → MARKET LOAD CLEAN (table `MARKET`)

| Original | Issue | Clean |
|---|---|---|
| 41 columns (`A:AO`): 28 empty, one holding a stray note | Sheet bloat | 16 columns |
| `RATE` | One `45K` text value on a row with no load date | Row excluded. Numeric `RATE (₹)` |
| `ADVANCE` / `BALANCE` | `-` typed in 125 balance cells | `ADVANCE RECEIVED (₹)` input and `BALANCE DUE (₹)` formula |
| `ADVANCE RECEIVED DATE` | 101 text values (`-`, typo `26-08-205`) | Real dates. Unparseable values left blank |
| — | — | `DAYS TO RECEIVE` = received date − load date |
| `POD STATUS ` | `DONE`, `DONE `, `DONE,<ref>`, `BILL NO.<n>` | `POD STATUS` (`DONE`) + `POD REF` + `POD AS LOGGED` |
| `CUSTOMER NAME ` | 25 raw values, 24 after trimming | Trimmed |

## 3. Monthly KM Run → KM RUN CLEAN (table `KMRUN`)

| Original | Issue | Clean |
|---|---|---|
| 694 rows, 65 used | Bloat | 60 truck-months (10 trucks × Mar–Aug 2026) |
| `Truck No` | 54 stored as numbers, 11 as text (leading-zero truck) | Text throughout |
| `Year` + `Month` as two text fields | Can't filter by date | `MONTH` as a real date |
| `Veriation`, `Incentive` (`=IF(J2<0,0,J2*1)`, rate hard-coded) | Typo; rate buried in formula | `VARIANCE (KM)`, `UTILISATION %`, `INCENTIVE (₹)` using the rate cell `D2` |

## 4. Monthly Sale → MONTHLY SALE CLEAN

The original was typed by hand. Checked against the load register:

- **9 of 28 months** had an Auburn figure different from Σ billing in the register (largest gap 9.6% of the month).
- **3 months** had a total ≠ Auburn + Market.

The clean version contains no typed numbers. Each month is `SUMIFS`/`COUNTIFS` over `AUBURN[MONTH]` and `MARKET[MONTH]`, pre-built from May 2024 to Dec 2028, with a `TOTAL` row.

## 5. Report (original pivot)

The pivot on `Auburn Load` is kept as-is because it shows clearly why the cleaning was needed: the same month, provider or status splits into several groups (`JUNE` / `JUNE ` / `JUNE,26`, `BN` / `BN `, `Closed` / `closed `). Its grand total still ties to the clean register.

---

## Validation performed

1. **Row alignment:** original row *n* ↔ clean row *n+5* for all 1,114 loads (lorry no. compared row by row).
2. **Totals tie:** Σ `BILLING RATE` and Σ `LORRY RATE` agree across `AUBURN`, `MONTHLY SALE CLEAN` (TOTAL row) and the `Report` pivot grand total.
3. **Row arithmetic:** `NET SAVING = BILLING − LORRY` and `BALANCE = LORRY − ADVANCE` on every populated row.
4. **Counts:** BN 684 + 3rd Party 430 = 1,114. Market 171.
5. **Independent recalculation:** the workbook was recalculated in a second engine (LibreOffice) and all 11,435 formula results were compared with Excel's cached values. The only difference is one cell in the *original* `Auburn Load` sheet (Excel treats `"" − n` as 0, LibreOffice as `#VALUE!`).

## Open exceptions

See the table in the [README](../README.md#open-exceptions-flagged-not-silently-fixed). They were left visible on purpose: fixing them needs source documents (trip sheets, bills), not a formula.
