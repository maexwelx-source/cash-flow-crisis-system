# Cash Flow Crisis System

A 24-month SaaS cash flow model.

The idea came from a real problem: most financial models are built by finance for finance. They're static, locked, and by the time you get the numbers, the decision window is already closed. This model is built differently - you flip a switch, the numbers update, and you immediately see whether the runway survives.

---

## What it does

The model runs a 24-month cash flow forecast for a SaaS business with built-in crisis scenarios. You set your baseline assumptions once, then use the Decisions sheet to simulate anti-crisis measures - cut marketing, freeze hiring, raise prices, inject emergency funding - and watch how each decision affects burn rate, runway, and closing balance across all 24 months.

It's not a reporting tool. It's a decision support tool.

---

## How it's structured

**Assumptions** - all editable inputs in one place. Revenue drivers, cost structure, macro parameters, working capital, and crisis trigger thresholds. Blue-bordered cells are the only things you need to touch.

**Decisions** - 8 anti-crisis levers with on/off switches:

- Cut Marketing Spend - reduces marketing by the effect value you set
- Cut Headcount - reduces payroll by the same logic
- Freeze Hiring - stops payroll from growing due to new hires
- Raise Prices - increases average check in the new MRR formula
- Reduce Churn - subtracts from the churn rate starting month 2
- Kill Infra 20% - cuts infrastructure costs
- Collect AR Faster - shifts cash collection earlier by reducing AR delay
- Emergency Fundraise - one-time cash injection in the month you specify

**Cash Flow Model** - the engine. Fully formula-driven across all 24 months. MRR accumulates properly (new clients add to the recurring base, churn applies from month 2). AR lag is modelled proportionally. Tax is calculated on accrual-basis profit. Every formula references either Assumptions or Decisions - nothing is hardcoded.

**Export Data** - a flat table that feeds directly into Power BI. 240 rows, 8 columns including a dynamic Scenario Name that shows which decisions are active.

---

## The financial logic

MRR compounds month over month:

```
MRR[m] = (MRR[m-1] + NewMRR[m-1]) x (1 - effective_churn)
```

Cash inflow applies an AR lag:

```
Cash[m] = frac x Gross[m-1] + (1 - frac) x Gross[m]
where frac = MIN(1, AR_delay_days / 30)
```

Breakeven is calculated pre-tax to avoid circular logic:

```
Breakeven = (Total Outflow - Variable Costs - Tax) / (1 - Variable Cost %)
```

Runway shows months of cash left at current burn, with a hard floor at zero and an Exhausted flag when balance goes negative.

---

## Power BI connection

Connect via Get Data - Excel Workbook - Export Data sheet only. Set Value to Decimal, Month Num to Whole Number, Date Full to Date. Sort the Date column by Month Num in Data view so the axis renders chronologically.

DAX measures to create:

```dax
Total Inflow = CALCULATE(SUM(Export_Data[Value]), Export_Data[Sub-category] = "Total Inflow")

Net Cash Flow = CALCULATE(SUM(Export_Data[Value]), Export_Data[Sub-category] = "Net Cash Flow")

Closing Balance = CALCULATE(SUM(Export_Data[Value]), Export_Data[Sub-category] = "Closing Balance")

Burn Rate = CALCULATE(SUM(Export_Data[Value]), Export_Data[Sub-category] = "Burn Rate")

Avg Burn Rate =
AVERAGEX(
    FILTER(
        VALUES(Export_Data[Date]),
        CALCULATE(SUM(Export_Data[Value]), Export_Data[Sub-category] = "Net Cash Flow") < 0
    ),
    CALCULATE(SUM(Export_Data[Value]), Export_Data[Sub-category] = "Burn Rate")
)
```

Recommended visuals: Line and Clustered Column for NCF plus Balance on dual axis, Stacked Column for outflow breakdown filtered to Category = Outflow, Bar Chart for Burn Rate with conditional red/grey coloring, Cards for KPIs, Slicer on Scenario Name for scenario comparison.

---

## What I'd do differently next time

The AR lag formula works well for delays up to 30 days but flattens out after that - 45 days and 90 days produce the same result. A proper multi-month waterfall would fix this but adds significant complexity for what's meant to be an operational model.

Tax is approximated on an accrual basis rather than actual cash payment timing. Good enough for liquidity planning, not good enough for tax compliance.

Inventory Turnover is in the Assumptions sheet but not connected to any formula. It's there as a placeholder for product companies who'd want to model working capital tied up in stock.

---

## Stack

Excel - Power BI (DAX) - HTML( Dashboard) -  for model generation and validation

---




