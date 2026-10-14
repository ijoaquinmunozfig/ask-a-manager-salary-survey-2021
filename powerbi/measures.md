# Power BI measures

All measures use the median instead of the average, because a few very high salaries would distort the average.

The table is named `ask_a_manager_clean`. Measures live in a separate table (`Measures Table (DAX)`) so they are easy to find.

## Core measures (all pages)

```dax
TotalResponses =
    COUNTROWS(
        ask_a_manager_clean
    )
```
Number of respondents in the current filter context.

```dax
MedianSalaryUSD =
    MEDIAN(
        ask_a_manager_clean[salary_usd]
    )
```
Median annual base salary in USD.

```dax
MedianTotalCompensationUSD =
    MEDIAN(
        ask_a_manager_clean[total_comp_usd]
    )
```
Median of salary plus additional compensation (bonuses, overtime). Missing additional compensation counts as 0.

```dax
%WithExtraCompensation = 
    DIVIDE(
        CALCULATE(
            COUNTROWS(ask_a_manager_clean),
            ask_a_manager_clean[additional_comp_usd] > 0
            ),
            COUNTROWS(ask_a_manager_clean)
    )
```
Share of respondents who report additional compensation above 0.

```dax
MedianSalary(n>=30) = 
    IF(
        [Responses] >= 30,
        [MedianSalaryUSD]
    )
```
Hides medians backed by fewer than 30 responses. Used in the equity charts.

## Cover

```dax
TotalCountries =
    CALCULATE(
        DISTINCTCOUNT(ask_a_manager_clean[country]),
        ask_a_manager_clean[country] <> "Other"
    )
```
Number of countries represented, excluding the grouped `Other` category.

## Role & Industry

```dax
TopPaidJobFamily = 
    VAR tbl =
        ADDCOLUMNS(
            FILTER(
                VALUES(ask_a_manager_clean[job_family]),
                CALCULATE(
                    COUNTROWS(ask_a_manager_clean)
                ) >= 30
            ),
            "@med",
            CALCULATE(
                MEDIAN(ask_a_manager_clean[salary_usd])
            )
        )
    RETURN
        MAXX(
            TOPN(1, tbl, [@med], DESC),
            ask_a_manager_clean[job_family])
```
Job family with the highest median salary, among families with at least 30 responses.

## Career

These measures rely on `exp_overall_rank` (1 = "1 year or less" ... 8 = "41 years or more").

```dax
MedianSalary(0-4yrs) = 
    CALCULATE(
        [MedianSalaryUSD],
        ask_a_manager_clean[exp_overall_rank] <= 2
        )
```

```dax
Mediansalary(21+yrs) = 
    CALCULATE(
        [MedianSalaryUSD],
        ask_a_manager_clean[exp_overall_rank] >= 6
        )
```
Median salary at the start of a career versus after 21 or more years.

## Equity

```dax
MedianSalaryWomen = 
    CALCULATE(
        [MedianSalary(30+Responses)],
        ask_a_manager_clean[gender] = "Woman"
        )
```

```dax
MedianSalaryMen = 
    CALCULATE(
        [MedianSalary(30+Responses)],
        ask_a_manager_clean[gender] = "Man"
        )
```

```dax
GenderGap% = 
    DIVIDE(
        [MedianSalaryWomen] - [MedianSalaryMen],
        [MedianSalaryMen]
        )
```
Unadjusted gap: a negative value means women's median is lower. It does not control for role, seniority or experience.

## Geography

```dax
MedianSalaryUSD_US = 
    CALCULATE(
        [MedianSalaryUSD],
        'ask_a_manager_clean'[country] = "United States"
    )
```

```dax
TopPaidState = 
    VAR tbl = ADDCOLUMNS(
        FILTER(
            FILTER(VALUES(ask_a_manager_clean[state]),
            NOT ask_a_manager_clean[state] IN {"Non-US", "Unknown", "Other"}),
            CALCULATE(
                COUNTROWS(ask_a_manager_clean)) >= 30),
        "@med",
        CALCULATE(
            MEDIAN(ask_a_manager_clean[salary_usd])
            )
        )
    RETURN
        MAXX(
            TOPN(1, tbl, [@med], DESC),
            ask_a_manager_clean[state]
            )
```
US state with the highest median salary, excluding non-US, unknown and grouped states, and states with fewer than 30 responses.

## Calculated columns (salary distribution chart)

```dax
Salary bin sort = 
    MIN(
        FLOOR(
            ask_a_manager_clean[salary_usd],
            10000
            ),
            200000
    )
```
Lower bound of each 10,000 USD range. Everything above 200,000 is grouped into one bin.

```dax
Salary range = 
    IF(
        ask_a_manager_clean[Salary bin sort] >= 200000,
        "200k+",
        FORMAT(ask_a_manager_clean[Salary bin sort] / 1000, "0")
        &
        "-"
        &
        FORMAT((ask_a_manager_clean[Salary bin sort] + 10000) / 1000, "0") & "k"
    )
```
Label shown on the axis (for example `70-80k`). Sorted by `Salary bin sort`.

## Model setup

| Column | Sort by column |
|---|---|
| `age` | `age_rank` |
| `exp_overall` | `exp_overall_rank` |
| `exp_field` | `exp_field_rank` |
| `education` | `education_rank` |
| `Salary range` | `Salary bin sort` |

- The `_rank` columns and `Salary bin sort` are hidden from the report view.
- `state` has the data category State or Province, so the map recognizes it.
- Formats: medians as currency with 0 decimals, `%WithExtraComp` and `GenderGap %` as percentages.
- The slicers (industry, country, job family) are synced across the analysis pages.
