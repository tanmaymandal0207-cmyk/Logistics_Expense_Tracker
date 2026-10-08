# Formula reference

Key formulas behind the KPIs and the dashboard, copied from the workbook.

---

## Year filter

**Dropdown:** `BN DASHBOARD!K3`. Data validation is a list over the named range `YearList`:

```excel
YearList = OFFSET('BN DASH DATA'!$X$4, 0, 0, SUMPRODUCT(--('BN DASH DATA'!$X$4:$X$12<>"")), 1)
```

**Year list** (`BN DASH DATA!X4:X12`). It grows with the data, so no maintenance is needed:

```excel
X4  = "All years"
X5  = IFERROR(YEAR(MIN(AUBURN[LOADING DATE])),"")
X6  = IF(OR($X5="",$X5>=YEAR(MAX(AUBURN[LOADING DATE]))),"",$X5+1)    ' filled down to X12
```

**Date window** (`BN DASHBOARD!C61:C62`):

```excel
FROM = IF($K$3="All years", DATE(1990,1,1),  DATE($K$3,1,1))
TO   = IF($K$3="All years", DATE(2100,12,31), DATE($K$3,12,31))
```

## KPI tiles

```excel
AUBURN BILLED = SUMIFS(AUBURN[BILLING RATE (₹)], AUBURN[LOADING DATE], ">="&$C$61, AUBURN[LOADING DATE], "<="&$C$62)
LORRY COST    = SUMIFS(AUBURN[LORRY RATE (₹)],   AUBURN[LOADING DATE], ">="&$C$61, AUBURN[LOADING DATE], "<="&$C$62)
TOTAL SALE    = AUBURN BILLED + SUMIFS(MARKET[RATE (₹)], MARKET[LOAD DATE], ">="&$C$61, MARKET[LOAD DATE], "<="&$C$62)
MARGIN %      = IFERROR((AUBURN BILLED − LORRY COST) / AUBURN BILLED, "")
LOADS         = COUNTIFS(AUBURN[LOADING DATE], ">="&$C$61, …) + COUNTIFS(MARKET[LOAD DATE], ">="&$C$61, …)
FLEET USE     = IF(Σ target = 0, "n/a", Σ actual km / Σ target km)
```

Sub-captions use `TEXT()`, e.g. `=TEXT($D$6,"₹#,##0")&" from Auburn"`.

## Per-truck utilisation

```excel
MAKE       = IFERROR(INDEX(KMRUN[MAKE], MATCH($B11, KMRUN[TRUCK NO.], 0)), "")
ACTUAL KM  = SUMIFS(KMRUN[ACTUAL KM], KMRUN[TRUCK NO.], $B11, KMRUN[MONTH], ">="&$C$61, KMRUN[MONTH], "<="&$C$62)
UTIL %     = IFERROR($F11/$G11, "")
```

`TRUCK NO.` is text in both places. A numeric truck number would make `MATCH` fail for the leading-zero truck.

## Chart series (`BN DASH DATA`)

**Monthly base** (`A4:F40`): one row per month, built with `SUMIFS` on `AUBURN[MONTH]` / `MARKET[MONTH]`.

**Calendar-month series** (`Z5:AG16`), Jan–Dec for the selected year, or each month summed across all years:

```excel
AUBURN (AA5) = SUMPRODUCT((MONTH($A$4:$A$40)=$Y5) * ($A$4:$A$40>=FROM) * ($A$4:$A$40<=TO) * $B$4:$B$40)
MARGIN (AG5) = IF($AA5=0, NA(), ($AA5-$AD5)/$AA5)
```

`NA()` leaves a gap in a line chart where a zero would draw a false drop to 0%.

**Category labels:**

```excel
Z5 = IF(K3="All years", TEXT(DATE(2000,$Y5,1),"mmm"), TEXT(DATE(K3,$Y5,1),"mmm-yy"))
```

## Register KPIs worth noting

```excel
' loads with a lorry rate but no advance logged
NO ADVANCE LOGGED = COUNTIF(AUBURN[LORRY RATE (₹)],">0") - SUMPRODUCT((AUBURN[LORRY RATE (₹)]>0)*(AUBURN[ADVANCE PAID (₹)]<>""))

' distinct customers (wrap in ROUND: floating-point gives 24.000000000000007)
CUSTOMERS = SUMPRODUCT((MARKET[CUSTOMER NAME]<>"")/COUNTIF(MARKET[CUSTOMER NAME],MARKET[CUSTOMER NAME]&""))

' period label
PERIOD = TEXT(MIN(AUBURN[LOADING DATE]),"mmm yyyy")&" to "&TEXT(MAX(AUBURN[LOADING DATE]),"mmm yyyy")
```

## Conditional formatting

| Range | Rule |
|---|---|
| Dashboard `H11:H20` | Data bar on utilisation |
| Dashboard `H11:H21` | Red text below 70% |
| `MARKET[REMARKS]` | Three rules: *freight clear*, *balance pending*, anything else |
| `KMRUN[UTILISATION %]` | 3-colour scale, red 40% → yellow 75% → green 100% |
