# DAX Library

Extracted from `Model.bim`.

## Total Income
- Folder: `BASE MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
SUM(FactSalesHeader[NetAmount])
```

## Total Expenses
- Folder: `BASE MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
SUM(FactExpense[ExpenseAmount])
```

## Net Profit
- Folder: `BASE MEASURES`
- Format: `"Rs"#,0.###############;-"Rs"#,0.###############;"Rs"#,0.###############`

```DAX
[Total Income] - [Total Expenses]
```

## Profit Margin %
- Folder: `BASE MEASURES`
- Format: `#,0.0%;-#,0.0%;#,0.0%`

```DAX
DIVIDE([Net Profit], [Total Income], 0)
```

## Payments Received
- Folder: `BASE MEASURES`
- Format: `"Rs"#,0.###############;-"Rs"#,0.###############;"Rs"#,0.###############`

```DAX
CALCULATE(
    SUM(FactPayment[PaymentAmount]),
    FactPayment[PaymentStatus] = "Received"
)
```

## Payments Pending
- Folder: `BASE MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
VAR Pending = CALCULATE(SUM(FactPayment[PaymentAmount]), FactPayment[PaymentStatus] = "Pending")
RETURN Pending
```

## Collection Rate %
- Folder: `BASE MEASURES`
- Format: `0.00%;-0.00%;0.00%`

```DAX
DIVIDE([Payments Received], [Total Income], 0)
```

## Expense % of Total
- Folder: `BREAKDOWN MEASURES`
- Format: `0.00%;-0.00%;0.00%`

```DAX
DIVIDE(
    [Total Expenses],
    CALCULATE([Total Expenses], REMOVEFILTERS(DimExpCategory)),
    0
)
```

## Expense % of Revenue
- Folder: `BREAKDOWN MEASURES`
- Format: `0.00%;-0.00%;0.00%`

```DAX
DIVIDE([Total Expenses], [Total Income], 0)
```

## Payment Method % of Total
- Folder: `BREAKDOWN MEASURES`

```DAX
DIVIDE(
    [Total Income],
    CALCULATE([Total Income], REMOVEFILTERS(DimPaymentMethod)),
    0
)
```

## Payment Fees Total
- Folder: `BREAKDOWN MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
SUM(FactPayment[FeeAmount])
```

## Payment Fees % of Revenue
- Folder: `BREAKDOWN MEASURES`
- Format: `0.00%;-0.00%;0.00%`

```DAX
DIVIDE([Payment Fees Total], [Total Income], 0)
```

## Cash Flow
- Folder: `BASE MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
[Total Income] - [Total Expenses]
```

## MoM Revenue
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactSalesHeader[OrderDate])           -- Last sale date
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1                 -- Start of current month
VAR END_CURR = MAXDATE                                  -- End of current month
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1                 -- Start of previous month
VAR END_PREV = EOMONTH(MAXDATE, -1)                    -- End of previous month

-- Current Month Revenue
VAR CURR_MONTH =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Revenue
VAR PREV_MONTH =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

-- Absolute change
VAR _change = CURR_MONTH - PREV_MONTH

-- Percent change
VAR _pctchange = DIVIDE(_change, PREV_MONTH, 0)

-- Formatted output
VAR _Condition =
    IF(
        ISBLANK(PREV_MONTH),
        "No previous month data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "+$#,###;-$#,###") & " MoM)",
            UNICHAR(129031) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "+$#,###;-$#,###") & " MoM)"
        )
    )

RETURN
    _Condition
```

## YoY Revenue
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactSalesHeader[OrderDate])          -- Last actual sale date
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD (from Jan 1 to last sale date)
VAR CY_YTD =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(CURR_YEAR, 1, 1),
            LASTSALE
        )
    )

-- Previous Year YTD (same period last year)
VAR PY_YTD =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(PREV_YEAR, 1, 1),
            DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE))
        )
    )

-- Absolute change YoY
VAR _change = CY_YTD - PY_YTD

-- Percent change YoY
VAR _pctchange = DIVIDE(_change, PY_YTD, 0)

-- Formatted text for arrows, % and absolute change
VAR _Condition =
    IF(
        ISBLANK(PY_YTD),
        "No previous year data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "+$#,###;-$#,###") & " YoY)",
            UNICHAR(129031) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "+$#,###;-$#,###") & " YoY)"
        )
    )

RETURN
    _Condition
```

## MoM Revenue Color
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactSalesHeader[OrderDate])           -- Last sale date
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1                 -- Start of current month
VAR END_CURR = MAXDATE                                  -- End of current month
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1                 -- Start of previous month
VAR END_PREV = EOMONTH(MAXDATE, -1)                    -- End of previous month

-- Current Month Revenue
VAR CURR_MONTH =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Revenue
VAR PREV_MONTH =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

-- MoM percent change
VAR _pctchange = DIVIDE(CURR_MONTH - PREV_MONTH, PREV_MONTH, 0)

-- Assign color
RETURN
SWITCH(
    TRUE(),
    ISBLANK(PREV_MONTH), "#808080",      -- Gray if no previous month data
    _pctchange > 0, "#008000",           -- Green if positive growth
    _pctchange < 0, "#FF0000",           -- Red if decline
    "#000000"                             -- Black if zero change
)
```

## CF Bars
- Folder: `TIMESERIES`

```DAX
VAR _table =
    SUMMARIZE(
        ALLSELECTED('DimDate'),
        'DimDate'[MonthShort],
        "IncomeValue", [Total Income]
    )
VAR _maxval = MAXX(_table, [IncomeValue])
VAR _minval = MINX(_table, [IncomeValue])
VAR _currval = [Total Income]

RETURN
SWITCH(
    TRUE(),
    _currval = _maxval, "Green",
    _currval = _minval, "Red",
    "Light Grey"
)
```

## CF Bars Expense
- Folder: `TIMESERIES`

```DAX
VAR _table =
    SUMMARIZE(
        ALLSELECTED('DimDate'),
        'DimDate'[MonthShort],
        "ExpenseValue", [Total Expenses]
    )
VAR _maxval = MAXX(_table, [ExpenseValue])
VAR _minval = MINX(_table, [ExpenseValue])
VAR _currval = [Total Expenses]   -- Use Expenses, not Income

RETURN
SWITCH(
    TRUE(),
    _currval = _maxval, "Red",         -- Max Expense → Red
    _currval = _minval, "Green",       -- Min Expense → Green
    "Light Grey"
)
```

## MoM Expenses
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactExpense[ExpenseDate])           -- Last expense date
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1               -- Start of current month
VAR END_CURR = MAXDATE                                -- End of current month
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1               -- Start of previous month
VAR END_PREV = EOMONTH(MAXDATE, -1)                  -- End of previous month

-- Current Month Expenses
VAR CURR_MONTH =
    CALCULATE(
        SUM(FactExpense[ExpenseAmount]),
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Expenses
VAR PREV_MONTH =
    CALCULATE(
        SUM(FactExpense[ExpenseAmount]),
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

-- Absolute change
VAR _change = CURR_MONTH - PREV_MONTH

-- Percent change
VAR _pctchange = DIVIDE(_change, PREV_MONTH, 0)

-- Formatted output
VAR _Condition =
    IF(
        ISBLANK(PREV_MONTH),
        "No previous month data",
        IF(
            _pctchange > 0,
            UNICHAR(129031) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "-$#,###;+$#,###") & " MoM)",  -- Up arrow for increase in Expense
            UNICHAR(129029) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "-$#,###;+$#,###") & " MoM)"   -- Down arrow for decrease
        )
    )

RETURN
    _Condition
```

## YoY Expenses
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactExpense[ExpenseDate])          -- Last actual expense date
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD Expenses (Jan 1 to last expense date)
VAR CY_YTD =
    CALCULATE(
        SUM(FactExpense[ExpenseAmount]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(CURR_YEAR, 1, 1),
            LASTSALE
        )
    )

-- Previous Year YTD Expenses (same period last year)
VAR PY_YTD =
    CALCULATE(
        SUM(FactExpense[ExpenseAmount]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(PREV_YEAR, 1, 1),
            DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE))
        )
    )

-- Absolute change YoY
VAR _change = CY_YTD - PY_YTD

-- Percent change YoY
VAR _pctchange = DIVIDE(_change, PY_YTD, 0)

-- Formatted output with arrows
VAR _Condition =
    IF(
        ISBLANK(PY_YTD),
        "No previous year data",
        IF(
            _pctchange > 0,
            UNICHAR(129031) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "-$#,###;+$#,###") & " YoY)",   -- Up arrow for decrease = good
            UNICHAR(129029) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "-$#,###;+$#,###") & " YoY)"    -- Down arrow for increase = bad
        )
    )

RETURN
    _Condition
```

## MoM Cash Flow
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactSalesHeader[OrderDate])           -- Last sale date (also used for Cash Flow)
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1                 -- Start of current month
VAR END_CURR = MAXDATE                                  -- End of current month
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1                 -- Start of previous month
VAR END_PREV = EOMONTH(MAXDATE, -1)                    -- End of previous month

-- Current Month Cash Flow
VAR CURR_MONTH =
    CALCULATE(
        [Total Income] - [Total Expenses],
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Cash Flow
VAR PREV_MONTH =
    CALCULATE(
        [Total Income] - [Total Expenses],
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

-- Absolute change
VAR _change = CURR_MONTH - PREV_MONTH

-- Percent change
VAR _pctchange = DIVIDE(_change, PREV_MONTH, 0)

-- Formatted output
VAR _Condition =
    IF(
        ISBLANK(PREV_MONTH),
        "No previous month data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "+$#,###;-$#,###") & " MoM)",  -- Positive CF growth
            UNICHAR(129031) & FORMAT(_pctchange, "0.0%") & " (" & FORMAT(_change, "+$#,###;-$#,###") & " MoM)"   -- Negative CF growth
        )
    )

RETURN
    _Condition
```

## YoY Cash Flow
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactSalesHeader[OrderDate])         -- Last actual date
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD Cash Flow
VAR CY_YTD =
    CALCULATE(
        [Total Income] - [Total Expenses],
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(CURR_YEAR, 1, 1),
            LASTSALE
        )
    )

-- Previous Year YTD Cash Flow
VAR PY_YTD =
    CALCULATE(
        [Total Income] - [Total Expenses],
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(PREV_YEAR, 1, 1),
            DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE))
        )
    )

-- Absolute change
VAR _change = CY_YTD - PY_YTD

-- Percent change
VAR _pctchange = DIVIDE(_change, PY_YTD, 0)

-- Formatted text with arrows
VAR _Condition =
    IF(
        ISBLANK(PY_YTD),
        "No previous year data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & " " & FORMAT(_pctchange, "0.0%") &
            " (" & FORMAT(_change, "+$#,###;-$#,###") & " YoY)",
            UNICHAR(129031) & " " & FORMAT(_pctchange, "0.0%") &
            " (" & FORMAT(_change, "+$#,###;-$#,###") & " YoY)"
        )
    )

RETURN
    _Condition
```

## YoY Revenue Color
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactSalesHeader[OrderDate])
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD
VAR CY_YTD =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(CURR_YEAR, 1, 1),
            LASTSALE
        )
    )

-- Previous Year YTD
VAR PY_YTD =
    CALCULATE(
        SUM(FactSalesHeader[NetAmount]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(PREV_YEAR, 1, 1),
            DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE))
        )
    )

-- Percent Change
VAR _pctchange = DIVIDE(CY_YTD - PY_YTD, PY_YTD, 0)

-- Assign color based on value
RETURN
SWITCH(
    TRUE(),
    ISBLANK(PY_YTD), "#808080",         -- Gray if no previous data
    _pctchange > 0, "#008000",           -- Green if growth
    _pctchange < 0, "#FF0000",           -- Red if negative
    "#000000"                             -- Black if zero change
)
```

## YoY Expense Color
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactExpense[ExpenseDate])
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD
VAR CY_YTD =
    CALCULATE(
        SUM(FactExpense[ExpenseDate]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(CURR_YEAR, 1, 1),
            LASTSALE
        )
    )

-- Previous Year YTD
VAR PY_YTD =
    CALCULATE(
        SUM(FactExpense[ExpenseDate]),
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(PREV_YEAR, 1, 1),
            DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE))
        )
    )

-- Percent Change
VAR _pctchange = DIVIDE(CY_YTD - PY_YTD, PY_YTD, 0)

-- Assign color based on value
RETURN
SWITCH(
    TRUE(),
    ISBLANK(PY_YTD), "#808080",         -- Gray if no previous data
    _pctchange > 0, "#008000",           -- Green if growth
    _pctchange < 0, "#FF0000",           -- Red if negative
    "#000000"                             -- Black if zero change
)
```

## YoY Cash Flow Color
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactSalesHeader[OrderDate])
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD Cash Flow
VAR CY_YTD =
    CALCULATE(
        [Total Income] - [Total Expenses],
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(CURR_YEAR, 1, 1),
            LASTSALE
        )
    )

-- Previous Year YTD Cash Flow
VAR PY_YTD =
    CALCULATE(
        [Total Income] - [Total Expenses],
        DATESBETWEEN(
            'DimDate'[Date],
            DATE(PREV_YEAR, 1, 1),
            DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE))
        )
    )

-- Percent change
VAR _pctchange = DIVIDE(CY_YTD - PY_YTD, PY_YTD, 0)

-- Assign color
RETURN
SWITCH(
    TRUE(),
    ISBLANK(PY_YTD), "#808080",     -- Gray if no previous data
    _pctchange > 0, "#008000",      -- Green if positive cash flow growth
    _pctchange < 0, "#FF0000",      -- Red if decline
    "#000000"                        -- Black if no change
)
```

## MoM Profit Margin %
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactSalesHeader[OrderDate])           -- Last sale date
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1                 -- Start of current month
VAR END_CURR = MAXDATE                                  -- End of current month
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1                 -- Start of previous month
VAR END_PREV = EOMONTH(MAXDATE, -1)                    -- End of previous month

-- Current Month Profit Margin
VAR CURR_PM =
    CALCULATE(
        DIVIDE([Net Profit], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Profit Margin
VAR PREV_PM =
    CALCULATE(
        DIVIDE([Net Profit], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

-- Absolute change
VAR _change = CURR_PM - PREV_PM

-- Percent change
VAR _pctchange = DIVIDE(_change, PREV_PM, 0)

-- Formatted output with arrows
VAR _Condition =
    IF(
        ISBLANK(PREV_PM),
        "No previous month data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & " " & FORMAT(CURR_PM, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " MoM)",
            UNICHAR(129031) & " " & FORMAT(CURR_PM, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " MoM)"
        )
    )

RETURN
    _Condition
```

## YoY Profit Margin %
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactSalesHeader[OrderDate])
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD Profit Margin
VAR CY_PM =
    CALCULATE(
        DIVIDE([Net Profit], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], DATE(CURR_YEAR, 1, 1), LASTSALE)
    )

-- Previous Year YTD Profit Margin
VAR PY_PM =
    CALCULATE(
        DIVIDE([Net Profit], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], DATE(PREV_YEAR, 1, 1), DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE)))
    )

-- Absolute change
VAR _change = CY_PM - PY_PM

-- Percent change
VAR _pctchange = DIVIDE(_change, PY_PM, 0)

-- Formatted output with arrows
VAR _Condition =
    IF(
        ISBLANK(PY_PM),
        "No previous year data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & " " & FORMAT(CY_PM, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " YoY)",
            UNICHAR(129031) & " " & FORMAT(CY_PM, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " YoY)"
        )
    )

RETURN
    _Condition
```

## MoM Collection Rate %
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactPayment[PaymentDate])           -- Last payment date
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1               -- Start of current month
VAR END_CURR = MAXDATE                                -- End of current month
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1               -- Start of previous month
VAR END_PREV = EOMONTH(MAXDATE, -1)                  -- End of previous month

-- Current Month Collection Rate
VAR CURR_RATE =
    CALCULATE(
        DIVIDE([Payments Received], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Collection Rate
VAR PREV_RATE =
    CALCULATE(
        DIVIDE([Payments Received], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

-- Absolute change
VAR _change = CURR_RATE - PREV_RATE

-- Percent change
VAR _pctchange = DIVIDE(_change, PREV_RATE, 0)

-- Formatted output with arrows
VAR _Condition =
    IF(
        ISBLANK(PREV_RATE),
        "No previous month data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & " " & FORMAT(CURR_RATE, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " MoM)",
            UNICHAR(129031) & " " & FORMAT(CURR_RATE, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " MoM)"
        )
    )

RETURN
    _Condition
```

## YoY Collection Rate %
- Folder: `TIMESERIES`

```DAX
VAR LASTSALE = MAX(FactPayment[PaymentDate])        -- Last payment date
VAR CURR_YEAR = YEAR(LASTSALE)
VAR PREV_YEAR = CURR_YEAR - 1

-- Current Year YTD Collection Rate
VAR CY_RATE =
    CALCULATE(
        DIVIDE([Payments Received], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], DATE(CURR_YEAR, 1, 1), LASTSALE)
    )

-- Previous Year YTD Collection Rate
VAR PY_RATE =
    CALCULATE(
        DIVIDE([Payments Received], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], DATE(PREV_YEAR, 1, 1), DATE(PREV_YEAR, MONTH(LASTSALE), DAY(LASTSALE)))
    )

-- Absolute change
VAR _change = CY_RATE - PY_RATE

-- Percent change
VAR _pctchange = DIVIDE(_change, PY_RATE, 0)

-- Formatted output with arrows
VAR _Condition =
    IF(
        ISBLANK(PY_RATE),
        "No previous year data",
        IF(
            _pctchange > 0,
            UNICHAR(129029) & " " & FORMAT(CY_RATE, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " YoY)",
            UNICHAR(129031) & " " & FORMAT(CY_RATE, "0.0%") &
            " (" & FORMAT(_pctchange, "0.0%") & " YoY)"
        )
    )

RETURN
    _Condition
```

## MoM Collection Rate Color
- Folder: `TIMESERIES`

```DAX
VAR MAXDATE = MAX(FactPayment[PaymentDate])
VAR ST_CURR = EOMONTH(MAXDATE, -1) + 1
VAR END_CURR = MAXDATE
VAR ST_PREV = EOMONTH(MAXDATE, -2) + 1
VAR END_PREV = EOMONTH(MAXDATE, -1)

-- Current Month Collection Rate
VAR CURR =
    CALCULATE(
        DIVIDE([Payments Received], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous Month Collection Rate
VAR PREV =
    CALCULATE(
        DIVIDE([Payments Received], [Total Income], 0),
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

VAR _pctchange = DIVIDE(CURR - PREV, PREV, 0)

RETURN
SWITCH(
    TRUE(),
    ISBLANK(PREV), "#808080",       -- No previous data
    _pctchange > 0, "#008000",      -- Improved
    _pctchange < 0, "#FF0000",      -- Declined
    "#000000"                       -- No change
)
```

## Color YTD Collection Rate
- Folder: `TIMESERIES`

```DAX
VAR MaxDate =
    MAX ( FactPayment[PaymentDate] )
VAR CY =
    YEAR ( MaxDate )
VAR PY =
    CY - 1
VAR CURR =
    CALCULATE (
        DIVIDE ( [Payments Received], [Total Income], 0 ),
        DATESBETWEEN ( 'DimDate'[Date], DATE ( CY, 1, 1 ), MaxDate )
    )
VAR PREV =
    CALCULATE (
        DIVIDE ( [Payments Received], [Total Income], 0 ),
        DATESBETWEEN (
            'DimDate'[Date],
            DATE ( PY, 1, 1 ),
            DATE ( PY, MONTH ( MaxDate ), DAY ( MaxDate ) )
        )
    )
VAR _pctchange =
    DIVIDE ( CURR - PREV, PREV, 0 )
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( PREV ), "#808080",
        _pctchange > 0, "#008000",
        _pctchange < 0, "#FF0000",
        "#000000"
    )
```

## MoM Expense Color
- Folder: `TIMESERIES`

```DAX
VAR LASTEXP = MAX(FactExpense[ExpenseDate])

-- Current Month Start & End
VAR CM_START = DATE(YEAR(LASTEXP), MONTH(LASTEXP), 1)
VAR CM_END   = LASTEXP

-- Previous Month Start & End
VAR PM_START = DATE(YEAR(LASTEXP), MONTH(LASTEXP) - 1, 1)
VAR PM_END   = EOMONTH(LASTEXP, -1)

-- Current Month Expense
VAR CM =
    CALCULATE(
        SUM(FactExpense[ExpenseAmount]),
        DATESBETWEEN('DimDate'[Date], CM_START, CM_END)
    )

-- Previous Month Expense
VAR PM =
    CALCULATE(
        SUM(FactExpense[ExpenseAmount]),
        DATESBETWEEN('DimDate'[Date], PM_START, PM_END)
    )

-- Percent Change
VAR _pctchange = DIVIDE(CM - PM, PM, 0)

RETURN
SWITCH(
    TRUE(),
    ISBLANK(PM), "#808080",     -- Gray if no previous month
    _pctchange > 0, "#008000",  -- Green if increased
    _pctchange < 0, "#FF0000",  -- Red if decreased
    "#000000"                   -- Black if no change
)
```

## Previous Year Revenue
- Folder: `TIMESERIES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
VAR LASTSALE =
    MAX(FactSalesHeader[OrderDate])

VAR CURR_YEAR =
    YEAR(LASTSALE)

VAR PREV_YEAR =
    CURR_YEAR - 1

VAR PREV_START =
    DATE(PREV_YEAR, 1, 1)

VAR PREV_END =
    DATE(PREV_YEAR, 12, 31)

RETURN
CALCULATE(
    SUM(FactSalesHeader[NetAmount]),
    DATESBETWEEN(
        'DimDate'[Date],
        PREV_START,
        PREV_END
    )
)
```

## Waterfall - Expected
- Folder: `BREAKDOWN MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
[Total Income]
```

## Waterfall - Received
- Folder: `BREAKDOWN MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
[Payments Received]
```

## Waterfall - Pending
- Folder: `BREAKDOWN MEASURES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
-[Payments Pending]
```

## Payments Failed
- Folder: `TIMESERIES`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
VAR Failed = CALCULATE(SUM(FactPayment[PaymentAmount]), FactPayment[PaymentStatus] = "Failed")
RETURN
Failed
```

## MoM Cash Flow Color
- Folder: `TIMESERIES`

```DAX
VAR MaxDate =
    MAX('DimDate'[Date])

-- Current month boundaries
VAR ST_CURR = EOMONTH(MaxDate, -1) + 1
VAR END_CURR = MaxDate

-- Previous month boundaries
VAR ST_PREV = EOMONTH(MaxDate, -2) + 1
VAR END_PREV = EOMONTH(MaxDate, -1)

-- Current month CF
VAR CF_CURR =
    CALCULATE(
        [Cash Flow],
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous month CF
VAR CF_PREV =
    CALCULATE(
        [Cash Flow],
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

VAR _pctchange = DIVIDE(CF_CURR - CF_PREV, CF_PREV, 0)

RETURN
SWITCH(
    TRUE(),
    ISBLANK(CF_PREV), "#808080",     -- No previous month
    _pctchange > 0, "#008000",       -- Green (increase)
    _pctchange < 0, "#FF0000",       -- Red (decline)
    "#000000"                        -- Black (no change)
)
```

## MoM Profit Color
- Folder: `TIMESERIES`

```DAX
VAR MaxDate =
    MAX('DimDate'[Date])

-- Current month boundaries
VAR ST_CURR = EOMONTH(MaxDate, -1) + 1
VAR END_CURR = MaxDate

-- Previous month boundaries
VAR ST_PREV = EOMONTH(MaxDate, -2) + 1
VAR END_PREV = EOMONTH(MaxDate, -1)

-- Current month profit
VAR PROF_CURR =
    CALCULATE(
        [Net Profit],
        DATESBETWEEN('DimDate'[Date], ST_CURR, END_CURR)
    )

-- Previous month profit
VAR PROF_PREV =
    CALCULATE(
        [Net Profit],
        DATESBETWEEN('DimDate'[Date], ST_PREV, END_PREV)
    )

VAR _pctchange = DIVIDE(PROF_CURR - PROF_PREV, PROF_PREV, 0)

RETURN
SWITCH(
    TRUE(),
    ISBLANK(PROF_PREV), "#808080",   -- No previous month
    _pctchange > 0, "#008000",       -- Increase = green
    _pctchange < 0, "#FF0000",       -- Decrease = red
    "#000000"                        -- No change
)
```

## Most Expensive Payment Method
- Folder: `BREAKDOWN MEASURES`

```DAX
VAR MaxFeeMethod =
    TOPN(
        1,
        SUMMARIZE(
            FactPayment,
            DimPaymentMethod[PaymentMethod],
            "TotalFees", [Payment Fees Total]
        ),
        [TotalFees],
        DESC
    )
VAR MethodName =
    MAXX(MaxFeeMethod, DimPaymentMethod[PaymentMethod])
RETURN
    "Most Expensive Payment Method is: " & MethodName
```

## Payment Details - Avg Days Outstanding
- Folder: `DRILL-THROUGH`
- Format: `#,0`

```DAX
VAR FilteredPayments = 
    FILTER(
        FactPayment,
        FactPayment[PaymentStatus] IN {"Pending", "Failed"}
    )
RETURN
    AVERAGEX(
        FilteredPayments,
        DATEDIFF(FactPayment[PaymentDate], TODAY(), DAY)
    )
```

## Payment Details - Total Amount
- Folder: `DRILL-THROUGH`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
CALCULATE(
    SUM(FactPayment[PaymentAmount])
)
```

## Payment Details - Oldest Payment Days
- Folder: `DRILL-THROUGH`
- Format: `0`

```DAX
VAR FilteredPayments = 
    FILTER(
        FactPayment,
        FactPayment[PaymentStatus] IN {"Pending", "Failed"}
    )
RETURN
    MAXX(
        FilteredPayments,
        DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY)
    )
```

## Payment Details - Transaction Count
- Folder: `DRILL-THROUGH`
- Format: `0`

```DAX
COUNTROWS(FactPayment)
```

## Payment Aging - 0-15 Days
- Folder: `DRILL-THROUGH`
- Format: `"Rs"#,0.###############;-"Rs"#,0.###############;"Rs"#,0.###############`

```DAX
VAR Today = MAX(FactPayment[PaymentDate])
RETURN
    CALCULATE(
        SUM(FactPayment[PaymentAmount]),
        FactPayment[PaymentStatus] IN {"Pending", "Failed"},
        FILTER(
            FactPayment,
            DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY) <= 15
        )
    )
```

## Payment Aging - 16-30 Days
- Folder: `DRILL-THROUGH`
- Format: `"Rs"#,0.###############;-"Rs"#,0.###############;"Rs"#,0.###############`

```DAX
VAR Today = MAX(FactPayment[PaymentDate])
RETURN
    CALCULATE(
        SUM(FactPayment[PaymentAmount]),
        FactPayment[PaymentStatus] IN {"Pending", "Failed"},
        FILTER(
            FactPayment,
            DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY) > 15 &&
            DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY) <= 30
        )
    )
```

## Payment Aging - 31-60 Days
- Folder: `DRILL-THROUGH`
- Format: `"Rs"#,0.###############;-"Rs"#,0.###############;"Rs"#,0.###############`

```DAX
VAR Today = MAX(FactPayment[PaymentDate])
RETURN
    CALCULATE(
        SUM(FactPayment[PaymentAmount]),
        FactPayment[PaymentStatus] IN {"Pending", "Failed"},
        FILTER(
            FactPayment,
            DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY) > 30 &&
            DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY) <= 60
        )
    )
```

## Payment Aging - 60+ Days
- Folder: `DRILL-THROUGH`
- Format: `"Rs"#,0.###############;-"Rs"#,0.###############;"Rs"#,0.###############`

```DAX
VAR Today = MAX(FactPayment[PaymentDate])
RETURN
    CALCULATE(
        SUM(FactPayment[PaymentAmount]),
        FactPayment[PaymentStatus] IN {"Pending", "Failed"},
        FILTER(
            FactPayment,
            DATEDIFF(FactPayment[PaymentDate], MAX(FactPayment[PaymentDate]), DAY) > 60
        )
    )
```

## Payment Aging - Amount
- Folder: `DRILL-THROUGH`
- Format: `\$#,0.###############;(\$#,0.###############);\$#,0.###############`

```DAX
VAR SelectedBucket = SELECTEDVALUE('Payment Aging Categories'[AgingBucket])
RETURN
    SWITCH(
        SelectedBucket,
        "0-15 Days", [Payment Aging - 0-15 Days],
        "16-30 Days", [Payment Aging - 16-30 Days],
        "31-60 Days", [Payment Aging - 31-60 Days],
        "60+ Days", [Payment Aging - 60+ Days],
        BLANK()
    )
```

## Location Color
- Folder: `Format`

```DAX
VAR _table =
    SUMMARIZE(
        ALLSELECTED('DimLocation'),
        'DimLocation'[Location],
        "RevenueValue", [Total Income]
    )
VAR _maxval = MAXX(_table, [RevenueValue])
VAR _minval = MINX(_table, [RevenueValue])
VAR _currval = [Total income]

RETURN
SWITCH(
    TRUE(),
    _currval = _maxval, "Green",   -- Green (Best)
    _currval = _minval, "Red",   -- Red (Worst)
    "Light Grey"                        -- Grey (Others)
)
```
