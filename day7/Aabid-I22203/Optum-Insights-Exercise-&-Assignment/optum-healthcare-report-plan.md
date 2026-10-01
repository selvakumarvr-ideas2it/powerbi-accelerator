# Optum Insight Healthcare Report: Implementation Plan

**Created:** 2026-09-30
**Project:** `day-6/Optum-Healthcare-Insights.pbip`
**Data:** `day-6/data/*.csv` (4 dimension tables, 2 fact tables)
**Branding source:** `day-6/optum-insights-branding-research.md`
**Tooling:**
- **powerbi-authoring-local** MCP builds the semantic model: TMDL tables, relationships, measures, hierarchies.
- **powerbi-report-mcp** builds the PBIR report: theme, pages, visuals, slicers, navigation, drillthrough and bookmarks.

---

## 1. Data profile (verified 2026-09-30)

| Table | Rows | Grain / key | Notes |
|---|---|---|---|
| DimDate | 730 | DateKey (yyyymmdd), 2025-01-01 → 2026-12-31 | Year, MonthNo, Month, Quarter |
| DimMember | 300 | MemberKey | Age 18–89, Gender F/M, PlanType (Commercial / Medicaid / Medicare Advantage), State (8) |
| DimPayer | 4 | PayerKey | Alpha, Beta, Gamma, Delta. LOB: Commercial / Medicare Advantage / Medicaid |
| DimProvider | 60 | ProviderKey | 6 specialties. NetworkStatus In/Out of Network |
| FactClaims | 1,800 | ClaimID (1 row = 1 claim) | Billed $13.46M, Allowed $10.78M, Paid $8.94M. ClaimCount = MemberMonths = 1 on every row |
| FactRevenue | 700 | DateKey × PayerKey | Revenue (premium) $17.81M |

- **Referential integrity:** 0 orphan keys in either fact table, and no duplicate ClaimIDs.
- **Data quirk:** it's synthetic data. 734 claims have Allowed > Billed and 758 have Paid > Allowed. Ratio KPIs are calculated in aggregate, and the plan flags this as a data caveat rather than "fixing" it.

## 2. Semantic model (star schema)

```
DimDate ─┬─< FactClaims >─┬─ DimMember
         │                ├─ DimProvider
         │                └─ DimPayer ─┐
         └─< FactRevenue >─────────────┘
```
All relationships are many-to-one and single-direction, keyed on integer surrogate keys. DimDate is marked as a date table on `Date`.

**Column work**
- DateKey / *Key columns become `int64` and are hidden. Metric columns get `summarizeBy none` once measures exist.
- `DimDate[Month]` sorts by `MonthNo`. Add `DimDate[Year-Month]` (text, sorted by `YearMonthNo`) for a continuous trend axis.
- `DimMember[Age Band]` calculated column: 18–34 / 35–49 / 50–64 / 65+.
- `DimMember[Gender]` gets display values Female / Male.

**Hierarchies (drill down)**
| Table | Hierarchy | Levels |
|---|---|---|
| DimDate | Calendar | Year → Quarter → Month |
| DimProvider | Provider Hierarchy | Specialty → ProviderName |
| DimMember | Geography | PlanType → State |

**Measures** (display folders in a dedicated `_Measures` table)
| Folder | Measure | DAX (summary) | Format |
|---|---|---|---|
| Claims | Total Claims | SUM(ClaimCount) | #,0 |
| Claims | Billed Amount | SUM(BilledAmount) | $#,0 |
| Claims | Allowed Amount | SUM(AllowedAmount) | $#,0 |
| Claims | Paid Amount | SUM(PaidAmount) | $#,0 |
| Claims | Avg Paid per Claim | Paid / Claims | $#,0 |
| Claims | Allowed to Billed % | Allowed / Billed | 0.0% |
| Claims | Paid to Allowed % | Paid / Allowed | 0.0% |
| Members | Unique Members | DISTINCTCOUNT(FactClaims[MemberKey]) | #,0 |
| Members | Member Months | SUM(MemberMonths) | #,0 |
| Members | Paid PMPM | Paid / Member Months | $#,0.00 |
| Members | Claims per 1K Members | Claims / Members × 1000 | #,0 |
| Members | Avg Member Age | AVERAGEX over members with claims | 0.0 |
| Financial | Revenue | SUM(RevenueAmount) | $#,0 |
| Financial | Medical Loss Ratio | Paid / Revenue | 0.0% |
| Financial | Operating Margin | Revenue − Paid | $#,0 |
| Financial | Margin % | Margin / Revenue | 0.0% |
| Time Intel | Revenue PY, Paid PY, Claims PY | SAMEPERIODLASTYEAR | as base |
| Time Intel | Revenue YoY %, Paid YoY %, Claims YoY % | DIVIDE(CY − PY, PY) | +0.0%;-0.0% |
| Network | Out of Network Paid | CALCULATE(Paid, NetworkStatus = "Out of Network") | $#,0 |
| Network | OON Paid % | OON Paid / Paid | 0.0% |
| Network | Provider Count | DISTINCTCOUNT(FactClaims[ProviderKey]) | #,0 |
| UI | Report Title (Drillthrough) | "Provider: " & SELECTEDVALUE(ProviderName) | text |
| UI | MLR Color | returns a hex color: ≥ 85% red, ≥ 75% amber, else green | text (for conditional formatting) |

## 3. Theme (from branding research §5/§6)

- **Custom theme "Optum Insight":** data colors `#FF612B, #002677, #F9A667, #095F87, #73716A, #B83C20, #0C55B8, #FFD1AB, #3D3C38, #989790`.
- **Canvas and surfaces:** cream page canvas #FAF8F2, white cards, no borders, 6px rounded corners.
- **Text:** charcoal #3D3C38 for text, black for titles, Segoe UI family.
- **Sentiment:** good #066605, neutral #FAAF00, bad #D71515. Diverging scale #B83C20 / #FAF8F2 / #095F87.
- **Banner:** white bar, title in charcoal Segoe UI Bold, "optum insight" brand mark in Optum Orange (large text, so 3:1 is OK), and a 4px Optum Orange accent line along the bottom of the banner. Orange stays a highlight color.
- **Buttons / navigator:** charcoal fill and white text for the selected page, white fill and charcoal text otherwise (mirrors optum.com primary/secondary buttons).

## 4. Common page chrome (every visible page)

| Element | Position (1280×720 canvas) |
|---|---|
| Banner shape (white) + brand mark + page title | 0,0 · 1280×52 |
| Orange accent line | 0,48 · 1280×4 |
| **Navigation:** page navigator (all visible pages) | right side of banner, 690,8 · 575×36 |
| **Slicer row (synced across pages):** Year · Quarter · Payer · Plan Type (+ page-specific slicer) | y = 57, height 60 |

Slicers use sync groups (`Year`, `Quarter`, `Payer`, `PlanType`), so a selection follows the user across pages.

## 5. Pages and the 3-30-300 rule

The **3-second** zone is the top KPI row (the answer). The **30-second** zone is the middle row: trend and breakdown charts that explain the answer. The **300-second** zone is the bottom: detail table or matrix, drill down, decomposition and drillthrough.

### Page 1: Executive Overview
**Job:** "Is the book of business profitable, and is medical cost under control?"
| Zone | Visuals |
|---|---|
| 3 s | KPI cards: **Medical Loss Ratio (hero)** · Revenue · Paid Amount · Operating Margin · Unique Members |
| 30 s | Revenue vs Paid Amount combo chart on the **Calendar hierarchy (drill down Year → Quarter → Month)** · MLR by Payer bar |
| 300 s | Matrix Payer × Year (Revenue, Paid, MLR, Margin %) with the Trend / Table **bookmark toggle** |
| Bookmarks | "Exec – Trend View", "Exec – Table View" (show/hide the combo chart or matrix), "Exec – Reset" (clear slicers) |

### Page 2: Claims Cost Analysis
**Job:** "Where are our claim dollars going and how much is discounted?"
| Zone | Visuals |
|---|---|
| 3 s | KPI cards: Paid Amount · Total Claims · Avg Paid per Claim · Allowed to Billed % · Paid YoY % |
| 30 s | Billed / Allowed / Paid by Specialty (clustered bar) · Paid Amount trend (line, **Calendar drill down**) |
| 300 s | Provider-hierarchy matrix (**drill down Specialty → Provider**, Claims / Billed / Allowed / Paid / Avg Paid). Right-click a provider → **Drill through to Provider Detail** |

### Page 3: Provider Network
**Job:** "Which providers drive cost and how much leaks out of network?"
| Zone | Visuals |
|---|---|
| 3 s | KPI cards: Provider Count · Out of Network Paid · OON Paid % · Avg Paid per Claim |
| 30 s | Paid by Specialty split by Network Status (stacked bar) · In vs Out of Network donut |
| 300 s | Top providers table (Provider, Specialty, Network, Claims, Paid, Avg Paid) as the **drillthrough source** · scatter of Claims vs Avg Paid by provider |
| Extra slicer | Network Status |

### Page 4: Member Population
**Job:** "Who are our members and what does it cost to care for them?"
| Zone | Visuals |
|---|---|
| 3 s | KPI cards: Unique Members · Paid PMPM · Claims per 1K Members · Avg Member Age |
| 30 s | Paid by Age Band × Gender (clustered column) · Members by Plan Type (donut) |
| 300 s | Paid by **Geography hierarchy (drill down Plan Type → State)** bar · **decomposition tree** of Paid by Plan Type / State / Age Band / Specialty |

### Page 5: Provider Detail (drillthrough, hidden)
**Job:** "Everything about one provider."
- **Drillthrough field:** `DimProvider[ProviderName]`. Keep all filters on. The page is hidden from navigation.
- **Visuals:** dynamic title (`Report Title (Drillthrough)`), a **Back** button, KPI cards (Claims, Paid, Avg Paid per Claim, Unique Members), a monthly Paid trend, Paid by Payer, and a claim-level detail table (ClaimID, Date, Member, Payer, Billed, Allowed, Paid).

## 6. Interactivity checklist

| Requirement | Implementation |
|---|---|
| Navigation | `pageNavigator` in every banner (hidden pages excluded automatically) + Back button on drillthrough |
| Slicers | Year, Quarter, Payer, Plan Type on every page (dropdown, multi-select, synced); Network Status on the Provider page |
| Drill down | Calendar (Year → Quarter → Month), Provider (Specialty → Provider), Geography (Plan Type → State) hierarchies bound to chart axes / matrix rows |
| Drill through | Provider Detail page with a `DimProvider[ProviderName]` drillthrough filter |
| Bookmarks | Trend View / Table View toggle + Reset on Executive Overview, driven by bookmark buttons |
| Tooltips | Enhanced tooltips enabled report-wide |

## 7. Build steps

1. Scaffold the PBIP: `Optum-Healthcare-Insights.pbip` plus `.SemanticModel` (TMDL, CSV partitions with absolute paths) and `.Report` (PBIR v4).
2. **powerbi-authoring-local:** `ConnectFolder` on the model, then:
   - create the `_Measures` table, calculated columns, sort-by columns and hierarchies;
   - create relationships and mark the date table;
   - create all measures;
   - save back to TMDL.
3. **powerbi-report-mcp:** `pbir_set_report`, then:
   - apply the Optum theme and report settings;
   - create the 4 pages plus the drillthrough page;
   - on each page add the chrome, slicers, KPIs and visuals;
   - add the drillthrough and Back button;
   - add bookmarks and bookmark buttons;
   - hide the filter pane.
4. Post-build: sync slicer groups, bookmark visibility states, `pbir_validate_wireframe` (report scope) and `pbir_model_usage`.
5. **Validation in Power BI Desktop (manual):** open the `.pbip` and refresh. Check the KPI totals against §1 (Paid $8.94M, Revenue $17.81M, MLR ≈ 50.2%). Test drill down, drillthrough and bookmarks, then save.

## 8. Known limitations / follow-ups
- Enterprise Sans / Optum Sans are proprietary, so the report uses Segoe UI (per the research).
- Bookmarks are written as PBIR JSON (visual visibility and page). Re-capture them in Desktop (Update bookmark) if you change the layout.
- PMPM uses the provided `MemberMonths` column. It is 1 per claim in this synthetic dataset, so PMPM equals average paid per claim. With a real eligibility table, compute member months from enrollment instead.

---

## 9. Build status (2026-09-30): ✅ built, pending first open in Power BI Desktop

| Item | Result |
|---|---|
| Semantic model | 7 tables, 6 relationships, 28 measures, 3 hierarchies, 5 calculated columns. Built via powerbi-authoring-local and exported to TMDL |
| Report | 4 visible pages + 1 hidden drillthrough page, 84 visuals, custom theme `Optum_Insight*.json` |
| Schema validation | All 96 PBIR JSON files validate against Microsoft's published schemas (page 2.1.0, visualContainer 2.7.0, bookmark 2.0.0, bookmarksMetadata 1.0.0) |
| Binding check | `pbir_model_usage`: every visual field resolves. Unused spares: Paid to Allowed %, Revenue PY/YoY %, Claims PY/YoY % |

**Fixes applied on top of the MCP output (tool defects worth knowing):**
- `pbir_add_bookmark` wrote `bookmarks/{id}/bookmark.json` with a non-existent `bookmarks/2.0.0` schema. It was rewritten to the PBIR format `bookmarks/{id}.bookmark.json` plus `bookmarks.json {items:[...]}`.
- Bookmark action buttons were written with `bookmarkDisplayName`. Changed to the `bookmark` property.
- The drillthrough page had an invalid `isAllFilter` key. It was replaced with `pageBinding` (type Drillthrough) and `howCreated: "Drillthrough"`.
- Slicers got `syncGroup` (Year, Quarter, PayerName, PlanType …) so selections persist across pages.

**Reference values to check in Desktop (all filters cleared, computed from the CSVs):**
| KPI | Expected |
|---|---|
| Revenue | $17,805,366 |
| Paid Amount | $8,940,886 |
| Medical Loss Ratio | 50.2% (2025: 46.1%, 2026: 55.0%) |
| Operating Margin | $8,864,480 |
| Total Claims / Unique Members | 1,800 / 300 |
| Avg Paid per Claim | $4,967 |
| Allowed to Billed % | 80.1% |
| Out of Network Paid / OON % | $1,243,556 / 13.9% |
| Active Providers | 60 |
| Avg Member Age | 53.1 |

**Key object IDs**
| Object | ID |
|---|---|
| Executive Overview page | 5dc554754350ac99d8ab |
| Claims Cost Analysis page | 399d60727b92a8defd19 |
| Provider Network page | afcbd8fca51d4eb14929 |
| Member Population page | da178205ffa92ac0f81a |
| Provider Detail page (drillthrough) | c7f40d1e745fb2e9565d |
| Exec chart / matrix (bookmark toggle) | bf38707f3af00d6025fd / 5c2298a5230bdeb36b4a |
| Bookmarks: Chart / Table / Reset | 58c3c2f7ef190c77ca44 / 2702c5220642c52ba864 / dc3f0d78f519f609fb60 |

**First-open checklist (Desktop)**
1. Open `Optum-Healthcare-Insights.pbip`, then run Refresh. The CSV paths are absolute (`...\day-6\data\`), so update them if you move the folder.
2. Check the KPI values against the table above.
3. Test the Chart / Table / Reset buttons (Ctrl+click in Desktop). If Reset doesn't clear the slicers: clear them manually, then Bookmarks pane → *Exec - Reset Filters* → Update.
4. Right-click a provider (Claims matrix or Provider Scorecard) → Drill through → Provider Detail. The Back button should return you to the source page.
5. Use the drill arrows on the combo chart, trend line, Specialty bar and Plan Type bar.
6. Save (Ctrl+S) so Desktop normalises the files.

## 10. First open in Desktop (2026-09-30)
- Opened in Power BI Desktop and ran a full refresh. Row counts: DimDate 730, FactClaims 1,800, FactRevenue 700.
- The DAX check matches every reference value in §9 (Revenue $17,805,366.39, Paid $8,940,886.48, MLR 50.2%, OON 13.9%, Avg Age 53.1).
- **Fix:** Paid / Revenue / Claims YoY % showed +103.8% with no year selected (2025+2026 vs 2025). They now fall back to the **latest year in context** when several years are selected. Results: all years = 2026 vs 2025 (Paid +3.8%, Revenue −13.1%, Claims +6.7%); 2025 alone = blank.
- Measure changes live in the open Desktop session. **Save (Ctrl+S) in Desktop** to write them to TMDL.

## 11. Post-review fixes (2026-09-30), confirmed working in Desktop
| Issue reported | Cause | Fix |
|---|---|---|
| Only the first KPI card per page styled correctly | `pbir_add_visual` writes inline 8pt fonts on labels, titles, slicers and axes, which override the theme | Stripped the inline defaults from all visuals, so the theme now applies |
| Card values rendered in a serif font | Theme `fontFamily` used bare names ("Segoe UI Bold") | Changed to full Power BI font stacks |
| Navigation bar too small | Navigator was 34px inside a 52px banner | Banner 60px, navigator 48px with 11pt labels |
| Slicers clipped | Slicer row was 52px | Slicers 60px. Rows rebalanced: KPI y130 h90, middle y225 h235, bottom y465 h249 |
| Chart / Table / Reset / Back buttons did nothing | Actions were written as `objects.action` (tool), then as `objects.visualLink` | Moved to `visualContainerObjects.visualLink` {show, type, bookmark} |
| Button labels missing | `show` was inside a selector entry | `show` goes in a selector-less entry, and state properties in default/hover entries |
| Auto grey subtitles under chart titles | Theme default | Theme `subTitle` show: false. Buttons and navigator get padding 0 |
| Open question | Navigator lists hidden "Provider Detail" in edit mode | Check in Reading view. If needed: Format → Pages → Show hidden pages Off |
