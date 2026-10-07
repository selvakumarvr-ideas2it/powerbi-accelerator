# Centene Payment Integrity – Dataset Check & Theme

## Theme (from centene.com live CSS, 2026-10-06)
File: `Centene_Theme.json` (Power BI → View → Themes → Browse for themes)

| Role | Hex | Source on centene.com |
|---|---|---|
| Primary (Centene blue) | `#00598C` | `--brand-color-global`, most-used brand colour |
| Dark navy (titles) | `#003356` | headings / footer |
| Teal (secondary) | `#00A0A4` | `--brand-color-secondary-global` |
| Body grey | `#58595B` | body text |
| Accents | `#F58220` orange, `#5C2F92` purple, `#CB187D` magenta, `#2C8641` green, `#AAC932` lime, `#262768` indigo | brand palette variables |
| Page background | `#F4F4F4` | section backgrounds |
| Fonts | Roboto (body), Roboto Slab (headings) | `font-family` declarations; Segoe UI fallback if Roboto isn't installed |

Suggested semantic use: Pre-pay savings = `#00598C`, Post-pay savings = `#00A0A4`, Good = `#2C8641`, Bad = `#A94442`.

## Dataset profile
- 12 CSVs: 1 fact (`fact_pi_measures`, 248 rows, monthly grain, Jan-2023 → Dec-2025, 36 months) + 11 dims.
- No duplicate PKs, no nulls in measures, no negatives.
- Reconciles: `pre_pay + post_pay = total_savings` on all rows; paid ≤ billed; processed ≤ pre-edit allowed ≤ submitted on all rows.

### Control totals (use in the validation sheet)
| Column | Sum |
|---|---|
| total_billed_amount | 6,332,770,753.40 |
| total_paid_amount | 4,391,703,679.79 |
| pre_pay_savings_amount | 69,744,660.79 |
| post_pay_savings_amount | 87,020,242.45 |
| total_savings_amount | 156,764,903.24 |
| total_claims_submitted_amount | 3,130,776 |
| total_claims_pre_edit_allowed_amount | 2,848,948 |
| total_claims_processed_amount | 2,715,219 |
| true_positive_rate_pct (avg) | 15,933.70 / 248 = 64.25 |

## Issues to fix before modelling
1. **File names swapped**: `dim_employer_group.csv` actually holds the PI Service data (10 rows) and `dim_pi_service.csv` holds the Employer Group data. Rename/swap in Power Query.
2. **Date key mismatch**: fact `date_key` is `YYYYMMDD` (e.g. 20250401), but `dim_date.date_key` is a surrogate 1…2557. Join on a derived key (`YEAR*10000+MONTH*100+DAY` from `full_date`) or convert fact key to a date.
3. **Employer group orphans**: 124 fact rows (50%) have `employer_group_key = null`; keys 2–6 (50 rows) have no matching dim row (dim has only key 1). Add an "Unknown / Not Assigned" member.
4. **Business-rule conflicts**: employer group 1 belongs to client 2 (HealthFirst, ASO), but 9 rows pair it with client 1; HealthFirst rows are split ASO 55 / FI 57. Document as a data-quality finding.
5. **`total_claims_*_amount` are whole numbers** (602–24,878) – they behave like claim counts, not dollars. Confirm with the brief; label accordingly.
6. **`dim_claim_status` has no FK in the fact** – it cannot be related; leave it out of the model or note it as unused.
7. **SCD2 rows handled correctly**: retired Analytic7 v1 (key 7), expired provider (key 7) and Legacy client (key 3) never appear in the fact.
8. `true_positive_rate_pct` is a rate – aggregate with AVERAGE (or a weighted average), never SUM.
