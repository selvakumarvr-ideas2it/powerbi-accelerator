# 10-Minute Customer Demo + 5-Minute Q&A

**Audience:** Centene Payment Integrity leadership  **Goal:** show where PI saves money, prove that the numbers are right, and show how to investigate a provider.

| Time | Section | What to show / say |
|---|---|---|
| 0:00–1:00 | **Context** | "Payment Integrity stops or recovers incorrect claim payments. This report covers 36 months (2023–2025) of PI results across 2 clients, 4 lines of business and 10 PI services." Show the model diagram: one fact table, ten dimensions. |
| 1:00–1:30 | **Data trust** | "Before building, I profiled the extract and found four issues: swapped file names, a date key that could not join, and 174 rows with missing or unknown employer groups. Each one is fixed in Power Query and covered by a test." |
| 1:30–4:00 | **Page 1 – Executive Overview** | **$156.8M saved**, 2.48% of billed dollars, 64% accuracy. 2024 dipped 24%, and 2025 recovered **+23%**. Turn on drill mode on the trend chart and click **2024** to see which quarter dropped. Pre-pay (blue) vs post-pay (teal): 44% of savings come before payment. Donut: Medicare is the largest LOB at $47.3M (30%), ahead of Commercial at $39.1M (25%). |
| 4:00–6:30 | **Page 2 – Service & Analytic** | Matrix: expand **Prepay → Prepayment Claim Editing**, then the analytics under it. Coordination of Benefits is the #1 service ($26.5M) and DRG Validation the smallest ($4.7M). Funnel: 9% of submitted claims are stopped by pre-pay edits. Analytic scorecard: compare simple TPR with savings-weighted TPR. |
| 6:30–8:30 | **Page 3 – Drill-through** | Back on page 1, right-click **Riverside Medical Group** → Drill through → Provider Detail. "$29.8M saved, and it is #2 by only $20k." Show the line vs prior year, then the claim-line table, including the "Not Assigned" and "Unmapped" employer groups. Click **Back**. |
| 8:30–9:30 | **Slicers** | Choose Client = HealthFirst Employer Trust and Funding = ASO. All three pages update, because the slicers are synced. |
| 9:30–10:00 | **Close** | "Every primary KPI is reconciled: 24 tests against values calculated straight from the CSVs, 24 pass. Next steps: fix the employer-group and funding-type keys at the source." |

## Q&A preparation

| Likely question | Answer |
|---|---|
| *How do you know $156.8M is right?* | I calculated it independently from the CSV files, then queried the Power BI model with DAX. They match to the cent (test T02). The yearly totals add up to the same number (T12–T14). Pre-pay + post-pay − total = $0 (T05). |
| *Why is TPR averaged, not summed?* | It is a rate. Summing 248 percentages gives 15,934%, which means nothing. I offer a simple average and a savings-weighted average. |
| *Explain the rank measure.* | `ISINSCOPE` keeps the total row blank. `RANKX` over `ALLSELECTED(dim_provider)` ranks against all providers the slicers allow. I use the whole table because the leaderboard also shows type and state columns. |
| *Why does the YoY card show +23.3% with no year selected?* | That card uses *Latest Year YoY %*: it finds the latest year with data (2025) and compares it with 2024. A plain `SAMEPERIODLASTYEAR` with no year selected would compare overlapping windows. |
| *What are "Unmapped Group (key n)" rows?* | 50 fact rows point to employer groups that are missing from the dimension. Rather than lose those dollars, Power Query adds placeholder members. They are created dynamically, so if the source adds more orphans they still appear. |
| *Why is dim_claim_status missing?* | The fact table has no claim-status key, so it cannot be related. I reported this as a data gap. |
| *Why is Sunrise Rehabilitation listed once with "LLC"?* | The provider dimension is SCD2. The old record expired in March 2024 and has no fact rows, so all savings sit on the current LLC record. |
| *Can this go to the Power BI Service?* | Yes. Publish from Desktop and switch the `DataFolder` parameter to a SharePoint or gateway path. The PBIP files are already in Git. |
