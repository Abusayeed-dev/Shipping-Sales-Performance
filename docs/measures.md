# DAX measures — Shipping Sales Performance


---

## 1. Total Revenue

```dax
Total Revenue =
SUM(vessels[Annual_Contract_Value_USD])
```

Sum of Annual Contract Value across all vessels. Used as the headline KPI on View 1.

---

## 2. Budget Total

```dax
Budget Total =
SUM(budget[Budget_USD])
```

Monthly budget aggregate, sliceable by Product / Year / Month via the `Date` table relationship.

---

## 3. Actual Total

```dax
Actual Total =
SUM(budget[Actual_USD])
```

Actual booked revenue (mirror of `Budget Total`).

---

## 4. Variance %

```dax
Variance % =
DIVIDE(
    [Actual Total] - [Budget Total],
    [Budget Total],
    0
)
```

Dynamic variance — recalculates when the user filters by month or product.

> Format: percentage (1 decimal place) → **Measure tools → Format → Percentage → 1 decimal**.

---

## 5. CRM Health Score

```dax
CRM Health Score =
AVERAGE(vessels[CRM_Completeness_Score])
```

Average completeness of CRM records (0–1 scale). Conditional-format the KPI card red below 0.75.

> Format: percentage (0 decimals) — values are stored as 0.57, 0.9, etc.

---

## 6. Win Rate

```dax
Win Rate =
DIVIDE(
    COUNTROWS(FILTER(vessels, vessels[Sales_Stage] = "Won")),
    COUNTROWS(vessels),
    0
)
```

Won opportunities ÷ total opportunities. With current data this returns ~22.5%.

> Format: percentage (1 decimal).

---

## 7. Renewal Rate

```dax
Renewal Rate =
DIVIDE(
    COUNTROWS(FILTER(vessels, vessels[Renewal_Status] = "Renewed")),
    COUNTROWS(FILTER(vessels, NOT(ISBLANK(vessels[Renewal_Date])))),
    0
)
```

Renewed contracts ÷ contracts up for renewal. Note the change from the original plan: `<>""` doesn't work for date columns once typed as Date — `ISBLANK` is the correct check.

> Format: percentage (1 decimal). Current data: 1 Renewed / 45 with renewal dates ≈ 2.2%. Realistic for early-cycle data.

---

## 8. Upsell Gap Count

```dax
Upsell Gap Count =
COUNTROWS(
    FILTER(
        vessels,
        vessels[NavStation] = 1 && vessels[NavFleet] = 0
    )
)
```

Vessels that own NavStation but not NavFleet — the core upsell motion. With current data this returns **112**.

> Format: whole number with thousands separator.

---



### At-Risk Renewal Value

```dax
At-Risk Renewal Value =
CALCULATE(
    SUM(vessels[Annual_Contract_Value_USD]),
    vessels[Subscription_Type] = "Flat Fee",
    vessels[Margin_Flag] = "At-Risk",
    vessels[Days_Until_Renewal] <= 365
)
```

Total ACV at risk in the next quarter. The headline number for View 3.

### Active Pipeline Count

```dax
Active Pipeline Count =
COUNTROWS(
    FILTER(
        vessels,
        NOT(vessels[Sales_Stage] IN { "Won", "Lost" })
    )
)
```

Open opportunities count for the View 1 KPI card.

### NavFleet Penetration

```dax
NavFleet Penetration =
DIVIDE(
    SUM(vessels[NavFleet]),
    COUNTROWS(vessels),
    0
)
```

Percentage of fleet that owns NavFleet. Flip the column name to compute penetration for any product.

---