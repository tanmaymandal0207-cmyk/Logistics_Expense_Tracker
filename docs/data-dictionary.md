# Data dictionary

**Type:** `In` = typed input (blue font) · `Calc` = formula (black font, do not overwrite).
Tables start at row 6 (header), with data from row 7. Rows 1–4 hold header KPIs.

---

## `AUBURN` (sheet AUBURN LOAD CLEAN)

One row per load moved for Auburn, by own truck (`BN`) or a hired lorry (`3rd Party`).

| Col | Field | Type | Definition / formula |
|---|---|---|---|
| A | S.NO. | Calc | `=IF($B7="","",ROW()-6)` |
| B | LOADING DATE | In | Date the lorry was loaded |
| C | MONTH | Calc | `=IF($B7="","",EOMONTH($B7,-1)+1)` (1st of month, the reporting key) |
| D | PROVIDER | In | `BN` (own fleet) / `3rd Party` |
| E | LORRY NO. | In | Vehicle registration |
| F | MOBILE NO. | In | Driver / lorry contact |
| G | FROM | In | Loading point |
| H | POINT GR NO. | In | Pickup point + GR (goods receipt) reference and box count |
| I | TO GR NO. | In | Delivery point + GR reference |
| J | LORRY MAKE | In | Body size, e.g. `32FEET SXL`, `24FEET`, `FLIGHT` |
| K | LORRY RATE (₹) | In | Cost paid to the lorry |
| L | BILLING RATE (₹) | In | Amount billed to Auburn |
| M | NET SAVING (₹) | Calc | `=IF(OR($K7="",$L7=""),"",$L7-$K7)` |
| N | MARGIN % | Calc | `=IF(OR($L7="",$L7=0,$M7=""),"",$M7/$L7)` |
| O | ADVANCE PAID (₹) | In | Advance paid to the lorry |
| P | BALANCE TO PAY (₹) | Calc | `=IF(OR($K7="",$O7=""),"",$K7-$O7)` |
| Q | BILL STATUS | In | `DONE` / `PENDING` |
| R | INVOICE NO. | In | Bill number raised to Auburn |
| S | PAYMENT STATUS | In | `RECEIPT` / `PENDING` |
| T | REACH DATE | In | Arrival at unloading point |
| U | UNLOAD DATE | In | Unloading start |
| V | UNLOAD END | In | Unloading end (only when logged as a range) |
| W | TRANSIT DAYS | Calc | `=IF(OR($B7="",$U7=""),"",$U7-$B7)` |
| X | CURRENT STATUS | In | `Closed` / `Running` |
| Y | BROKER ADDRESS | In | Transporter / broker (`BN` for own fleet) |
| Z | REMARKS | In | Extra charges, halting, overload, etc. |
| AA | REACH AS LOGGED | In (audit) | Original text of reach date (grouped) |
| AB | UNLOAD AS LOGGED | In (audit) | Original text of unload date(s) (grouped) |

**Header KPIs:** Loads, Period, Avg transit days, Lorry rate paid, Billed to Auburn, Net saving, Margin %, Advance paid, Balance to pay, Own-fleet loads, 3rd-party loads, No advance logged.

## `MARKET` (sheet MARKET LOAD CLEAN)

Own trucks hired out to outside customers. There is no lorry cost on these rows.

| Col | Field | Type | Definition / formula |
|---|---|---|---|
| A | S.NO. | Calc | `=IF($B7="","",ROW()-6)` |
| B | LOAD DATE | In | |
| C | MONTH | Calc | `=IF($B7="","",EOMONTH($B7,-1)+1)` |
| D | LORRY NO. | In | Own truck |
| E | CUSTOMER NAME | In | |
| F | LOAD FROM | In | |
| G | DELIVERY POINT | In | |
| H | RATE (₹) | In | Freight charged |
| I | ADVANCE RECEIVED (₹) | In | |
| J | BALANCE DUE (₹) | Calc | `=IF(OR($H7="",$I7=""),"",$H7-$I7)` |
| K | ADVANCE RECEIVED DATE | In | |
| L | DAYS TO RECEIVE | Calc | `=IF(OR($B7="",$K7=""),"",$K7-$B7)` |
| M | POD STATUS | In | `DONE` or blank |
| N | POD REF | In | Reference split out of the old POD field |
| O | REMARKS | In | `ALL FREIGHT CLEAR NO DUES` / `BALANCE PENDING` / … (conditional formatting highlights each of the three cases) |
| P | POD AS LOGGED | In (audit) | Original POD text |

**Header KPIs:** Loads, Period, Customers, Revenue, Received, Balance due, Avg load value, Avg days to receive, POD not done.

## `KMRUN` (sheet KM RUN CLEAN)

One row per truck per month.

| Col | Field | Type | Definition / formula |
|---|---|---|---|
| A | MONTH | In | 1st of month |
| B | TRUCK NO. | In | Text (keeps leading zeros) |
| C | MAKE | In | |
| D | CAPACITY | In | `32 Feet` / `24 Feet` |
| E | DRIVER | In | |
| F | ACTUAL KM | In | |
| G | TARGET KM | In | Default 15,000 (cell `D3`) |
| H | VARIANCE (KM) | Calc | `=IF(OR($F7="",$G7=""),"",$F7-$G7)` |
| I | UTILISATION % | Calc | `=IF(OR($F7="",$G7="",$G7=0),"",$F7/$G7)` (red → yellow → green scale: 40% / 75% / 100%) |
| J | INCENTIVE (₹) | Calc | `=IF($H7="","",MAX(0,$H7)*$D$2)`: ₹ per km above target, rate in `D2` |

**Header KPIs:** Total actual km, Total target km, Fleet utilisation, Truck-months at target, Total incentive earned.

## MONTHLY SALE CLEAN

| Col | Field | Formula (row 4) |
|---|---|---|
| A | MONTH | `=DATE(2024,5,1)`, then the next month on each row |
| B | AUBURN LOADS | `=COUNTIFS(AUBURN[MONTH],$A4)` |
| C | AUBURN BILLED (₹) | `=SUMIFS(AUBURN[BILLING RATE (₹)],AUBURN[MONTH],$A4)` |
| D | AUBURN LORRY COST (₹) | `=SUMIFS(AUBURN[LORRY RATE (₹)],AUBURN[MONTH],$A4)` |
| E | AUBURN MARGIN (₹) | `=$C4-$D4` |
| F | AUBURN MARGIN % | `=IFERROR($E4/$C4,"")` |
| G | MARKET LOADS | `=COUNTIFS(MARKET[MONTH],$A4)` |
| H | MARKET SALE (₹) | `=SUMIFS(MARKET[RATE (₹)],MARKET[MONTH],$A4)` |
| I | TOTAL SALE (₹) | `=$C4+$H4` |

Row 60 holds the `TOTAL`, which must equal the `AUBURN` and `MARKET` header KPIs.
