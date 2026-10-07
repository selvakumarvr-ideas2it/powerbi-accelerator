# KPI Dictionary - Centene Payment Integrity

Every measure lives in the `_Measures` table, grouped into display folders. Format strings are set on the measure, so every visual formats them the same way.


## Savings

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Total Savings** | Pre-pay + post-pay savings identified by PI services. | `SUM(fact_pi_measures[total_savings_amount])` | `\$#,0;(\$#,0);\$#,0` |
| **Pre-Pay Savings** | Savings avoided before payment (claim edits, prepay review). | `SUM(fact_pi_measures[pre_pay_savings_amount])` | `\$#,0;(\$#,0);\$#,0` |
| **Post-Pay Savings** | Overpayments recovered after payment. | `SUM(fact_pi_measures[post_pay_savings_amount])` | `\$#,0;(\$#,0);\$#,0` |
| **Pre-Pay Share %** | Share of total savings captured before payment (cost avoidance is cheaper than recovery). | `DIVIDE([Pre-Pay Savings], [Total Savings])` | `0.0%` |
| **Savings Rate %** | Total savings as a share of billed dollars. | `DIVIDE([Total Savings], [Total Billed])` | `0.00%` |
| **Savings per Processed Claim** | Total savings divided by claims processed. | `DIVIDE([Total Savings], [Claims Processed])` | `\$#,0.00;(\$#,0.00);\$#,0.00` |
| **Savings YTD** | Calendar year-to-date savings. | `TOTALYTD([Total Savings], dim_date[full_date])` | `\$#,0;(\$#,0);\$#,0` |

## Claims & Payments

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Total Billed** | Provider billed amount. | `SUM(fact_pi_measures[total_billed_amount])` | `\$#,0;(\$#,0);\$#,0` |
| **Total Paid** | Amount paid on the claims. | `SUM(fact_pi_measures[total_paid_amount])` | `\$#,0;(\$#,0);\$#,0` |
| **Paid to Billed %** | Paid / billed. | `DIVIDE([Total Paid], [Total Billed])` | `0.0%` |
| **Claims Submitted** | Claims submitted (source column is named *_amount but holds whole-number claim counts). | `SUM(fact_pi_measures[total_claims_submitted_amount])` | `#,0` |
| **Claims Pre-Edit Allowed** | Claims that passed pre-payment edits. | `SUM(fact_pi_measures[total_claims_pre_edit_allowed_amount])` | `#,0` |
| **Claims Processed** | Claims processed through adjudication. | `SUM(fact_pi_measures[total_claims_processed_amount])` | `#,0` |
| **Pre-Edit Reduction %** | Share of submitted claims stopped by pre-payment edits. | `1 - DIVIDE([Claims Pre-Edit Allowed], [Claims Submitted])` | `0.0%` |
| **Processing Yield %** | Processed / submitted claims. | `DIVIDE([Claims Processed], [Claims Submitted])` | `0.0%` |

## Accuracy

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Avg True Positive Rate %** | Simple average of the row-level true positive rate (rates are averaged, never summed). | `DIVIDE(AVERAGE(fact_pi_measures[true_positive_rate_pct]), 100)` | `0.0%` |
| **Savings-Weighted TPR %** | True positive rate weighted by savings dollars - big findings count more. | `DIVIDE( SUMX(fact_pi_measures, fact_pi_measures[true_positive_rate_pct] * fact_pi_measures[total_savings_amount]), [Total Savings] ) / 100` | `0.0%` |

## Time Intelligence

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Total Savings PY** | Savings for the same period last year. | `CALCULATE([Total Savings], SAMEPERIODLASTYEAR(dim_date[full_date]))` | `\$#,0;(\$#,0);\$#,0` |
| **Savings YoY %** | Change vs same period last year (blank for 2023 - no prior year). | `VAR Curr = [Total Savings] VAR Prev = [Total Savings PY] RETURN IF(NOT ISBLANK(Prev), DIVIDE(Curr - Prev, Prev))` | `+0.0%;-0.0%;0.0%` |
| **Latest Year YoY %** | Latest year in the current selection vs the year before. Used on KPI cards. | `VAR LastYear = YEAR(MAX(fact_pi_measures[period_date])) VAR Curr = CALCULATE([Total Savings], dim_date[year] = LastYear) VAR Prev = CALCULATE([Total Savings], dim_date[year] = LastYear - 1) RETURN IF(NOT ISBLANK(Prev), DIVIDE(Curr - Prev, Prev))` | `+0.0%;-0.0%;0.0%` |

## Provider

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Provider Savings Rank** | Rank of the provider by total savings within the current selection. | `IF( ISINSCOPE(dim_provider[provider_name]) && NOT ISBLANK([Total Savings]), RANKX(ALLSELECTED(dim_provider), [Total Savings], , DESC, Dense) )` | `0` |
| **Providers with Savings** | Distinct providers that have fact rows. | `DISTINCTCOUNT(fact_pi_measures[provider_key])` | `#,0` |

## Data Quality

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Fact Row Count** | Rows in the fact table. | `COUNTROWS(fact_pi_measures)` | `#,0` |
| **Rows Without Employer Group** | Fact rows whose employer_group_key was null in the source. | `CALCULATE([Fact Row Count], dim_employer_group[mapping_status] = "Not Assigned (null key)")` | `#,0` |
| **Rows With Orphan Employer Group** | Fact rows pointing to employer group keys missing from the dimension. | `CALCULATE([Fact Row Count], dim_employer_group[mapping_status] = "Orphan key (missing in dimension)")` | `#,0` |
| **Rows With Employer Group Client Conflict** | Fact rows whose employer group belongs to a different client (business-rule conflict). Must be 0; currently 9. | `COUNTROWS(FILTER(fact_pi_measures, VAR egClient = RELATED(dim_employer_group[eg_client_key]) RETURN NOT ISBLANK(egClient) && egClient <> fact_pi_measures[client_key]))` | `#,0` |
| **Savings Reconciliation Gap** | Total savings minus (pre-pay + post-pay). Must be 0. | `[Total Savings] - ([Pre-Pay Savings] + [Post-Pay Savings])` | `\$#,0.00;(\$#,0.00);\$#,0.00` |

## Client

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Active Clients** | Clients with PI activity in the current selection. | `DISTINCTCOUNT(fact_pi_measures[client_key])` | `#,0` |
| **Client Savings Share %** | Client share of total savings in the current selection. | `DIVIDE([Total Savings], CALCULATE([Total Savings], ALLSELECTED(dim_client)))` | `0.0%` |

## Labels

| Measure | Business definition | DAX | Format |
|---|---|---|---|
| **Provider Page Title** | Dynamic header for the drill-through page. | `"Provider Detail  \|  " & SELECTEDVALUE(dim_provider[provider_name], "All providers - right-click a provider and choose Drill through")` | `text` |
| **Data Range Label** | Footer text showing the period covered by the current selection. | `"Data: " & FORMAT(MIN(fact_pi_measures[period_date]), "mmm yyyy") & " - " & FORMAT(MAX(fact_pi_measures[period_date]), "mmm yyyy")` | `text` |

## Primary KPIs and their reconciled values

| KPI | Value (all periods) | Validation test |
|---|---|---|
| Total Savings | $156,764,903.24 | T02 - PASS |
| Pre-Pay Savings | $69,744,660.79 | T03 - PASS |
| Post-Pay Savings | $87,020,242.45 | T04 - PASS |
| Savings Rate % | 2.48% | T08 - PASS |
| Avg True Positive Rate % | 64.25% | T11 - PASS |
| Latest Year YoY % (2025 vs 2024) | 23.32% | T16 - PASS |
| Total Billed | $6,332,770,753.40 | T06 - PASS |
| Claims Submitted | 3,130,776 | T09 - PASS |
