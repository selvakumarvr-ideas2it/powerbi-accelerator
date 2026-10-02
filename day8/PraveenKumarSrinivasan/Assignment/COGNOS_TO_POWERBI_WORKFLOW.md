# Cognos Analytics to Power BI Conversion Workflow

**Date:** 2026-09-30  
**Project:** Cognos Sales Report → Power BI Interactive Dashboard  
**Reference:** cog-bi.pbip

---

## Table of Contents

1. [Overview](#overview)
2. [Project Architecture](#project-architecture)
3. [Phase 1: Initial Setup](#phase-1-initial-setup)
4. [Phase 2: Semantic Model Development](#phase-2-semantic-model-development)
5. [Phase 3: Report Design](#phase-3-report-design)
6. [Phase 4: Field Binding & Data Connection](#phase-4-field-binding--data-connection)
7. [Phase 5: Visual Alignment & Formatting](#phase-5-visual-alignment--formatting)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Best Practices](#best-practices)
10. [Tools & MCP Resources](#tools--mcp-resources)

---

## Overview

### Objective
Convert a Cognos Analytics report (complex_sales_report.xml) into a fully functional Power BI project (.pbip) with:
- Complete semantic model in TMDL format
- Synthetic data generation (96+ records)
- 3-30-300 visual design principle implementation
- Professional KPI cards with proper data binding
- Consistent visual alignment and styling

### Key Deliverables
- ✅ 3 report pages (Revenue Overview, Product Analysis, Trend)
- ✅ 12 KPI cards with proper data binding
- ✅ 8 analytical charts/tables
- ✅ Complete semantic model with 3 tables and 6 measures
- ✅ Professional color scheme and formatting

---

## Project Architecture

### Directory Structure
```
cog-bi.pbip/
├── cog-bi.SemanticModel/
│   └── definition/
│       ├── database.tmdl
│       ├── model.tmdl
│       ├── relationships.tmdl
│       └── tables/
│           ├── FactSales.tmdl
│           ├── DimProduct.tmdl
│           └── DimDate.tmdl
└── cog-bi.Report/
    └── definition/
        ├── report.json
        └── pages/
            ├── e601a55fefa0934dbe1b/ (Revenue Overview)
            ├── 5bb1921c4ab7a5aaf530/ (Product Analysis)
            └── 77c6b3692230bce4c73f/ (Trend)
```

### Semantic Model Schema

#### **FactSales Table** (96 records)
| Column | Type | Summarization | Purpose |
|--------|------|----------------|---------|
| OrderYear | String | None | Year dimension |
| OrderMonth | String | None | Month dimension (1-12) |
| ProductKey | String | None | Product foreign key |
| Region | String | None | Geographic dimension |
| Revenue | Double | Sum | Sales amount |
| Cost | Double | Sum | Product cost |
| Quantity | Double | Sum | Units sold |
| Price | Double | Average | Unit price |
| Discount | Double | Sum | Promotional discount |

**Measures:**
- `Total Revenue` = SUM(Revenue)
- `Total Cost` = SUM(Cost)
- `Total Quantity` = SUM(Quantity)
- `Gross Profit` = SUM(Revenue) - SUM(Cost)
- `Net Revenue` = SUM(Revenue) - SUM(Discount)
- `Avg Unit Price` = AVERAGE(Price)

#### **DimProduct Table** (8 records)
| Column | Type |
|--------|------|
| ProductKey | String |
| ProductName | String |
| ProductLine | String |
| Category | String |

#### **DimDate Table** (12 records)
| Column | Type |
|--------|------|
| DateKey | String |
| OrderYear | String |
| OrderMonth | String |
| MonthName | String |
| Quarter | String |

### Relationships
- `FactSales.ProductKey` → `DimProduct.ProductKey` (Many-to-One)
- `FactSales.OrderMonth` → `DimDate.OrderMonth` (Many-to-One)

---

## Phase 1: Initial Setup

### Step 1: Create Project Structure
```powershell
# Create PBIP project directory
New-Item -ItemType Directory -Path "cog-bi.pbip"
cd cog-bi.pbip

# Create semantic model folder
New-Item -ItemType Directory -Path "cog-bi.SemanticModel/definition/tables"

# Create report folder structure
New-Item -ItemType Directory -Path "cog-bi.Report/definition/pages"
```

### Step 2: Initialize Project Configuration
Create `cog-bi.pbip/cog-bi.pbip` file:
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/pbip/project/2.0.0/schema.json",
  "version": "1.0",
  "semanticModel": "cog-bi.SemanticModel",
  "report": "cog-bi.Report"
}
```

### Step 3: Extract Data from Cognos Report
- Parse complex_sales_report.xml to identify:
  - Data fields and dimensions
  - Aggregation functions
  - Report structure and layout
  - Styling and formatting rules

---

## Phase 2: Semantic Model Development

### Step 1: Create Database Configuration
File: `cog-bi.SemanticModel/definition/database.tmdl`
```tmdl
database SalesModel
  compatibilityLevel: 1606
  model "SalesModel" = {
    tables: [FactSales, DimProduct, DimDate]
  };
```

### Step 2: Create Model Metadata
File: `cog-bi.SemanticModel/definition/model.tmdl`
```tmdl
model "SalesModel"
  culture: en-US
  defaultMeasureDefinition: measure 'Total Revenue'
  sourceQueryCulture: en-US
```

### Step 3: Create Tables with Synthetic Data

**Key Considerations:**
- Use `Table.FromRecords()` M expression for data generation
- Ensure data consistency across relationships
- Generate sufficient records (96+) for meaningful analysis
- Cover all relevant time periods and product categories

#### Example: FactSales Data Generation
```m
let
  data = Table.FromRecords({
    [OrderYear=2023, OrderMonth=1, ProductKey=1, Region="North", Revenue=15000, Cost=7500, Quantity=150, Price=100, Discount=750],
    [OrderYear=2023, OrderMonth=1, ProductKey=2, Region="South", Revenue=12000, Cost=6000, Quantity=120, Price=100, Discount=600],
    ...
  })
in
  data
```

### Step 4: Define Measures
All measures must use DAX with proper formatting:
```tmdl
measure 'Total Revenue' = SUM(FactSales[Revenue])
  formatString: $#,0.00
  
measure 'Gross Profit' = SUM(FactSales[Revenue]) - SUM(FactSales[Cost])
  formatString: $#,0.00
  
measure 'Avg Unit Price' = AVERAGE(FactSales[Price])
  formatString: $#,0.00
```

### Step 5: Configure Data Types and Summarization
```tmdl
column Revenue
  dataType: double
  summarizeBy: sum
  formatString: $#,0.00
  
column Price
  dataType: double
  summarizeBy: average
  formatString: $#,0.00
```

### Step 6: Create Relationships
File: `cog-bi.SemanticModel/definition/relationships.tmdl`
```tmdl
relationship: many FactSales[ProductKey] to one DimProduct[ProductKey]

relationship: many FactSales[OrderMonth] to one DimDate[OrderMonth]
```

---

## Phase 3: Report Design

### Step 1: Create Report Pages
File: `cog-bi.Report/definition/report.json`

For each page, create:
- Page configuration (name, displayName, dimensions)
- Background styling (dark navy #1a2332)
- Visual containers

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.1.0/schema.json",
  "name": "e601a55fefa0934dbe1b",
  "displayName": "Revenue Overview",
  "displayOption": "FitToPage",
  "height": 720,
  "width": 1280,
  "objects": {
    "background": [{
      "properties": {
        "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#1a2332'" } } } } }
      }
    }]
  }
}
```

### Step 2: Design KPI Cards
**3-30-300 Rule Implementation:**
- 3 seconds: KPI cards show key metrics at a glance
- 30 seconds: Charts provide detailed breakdowns
- 300 seconds: Tables enable deep-dive analysis

**KPI Card Specifications:**
- Width: 280px, Height: 100px
- Background: White (#FFFFFF)
- Border: 1px colored, radius 6px
- Drop shadow: Enabled with matching color
- Category labels: Disabled
- Font size: 16px for values

### Step 3: Create Visual Layout
**Card Distribution (Row 1):**
```
Total Span: x=15 to x=1235 (1220 pixels)
Card 1: x=15   (left section start)
Card 2: x=328  (left section, position 2)
Card 3: x=642  (right section, position 1)
Card 4: x=955  (right section end, ending at x=1235)

Gap spacing: 33 pixels between cards
```

**Visual Sections (Rows 2-3):**
- Left section: x=15, width=605 (48% of space)
- Right section: x=625, width=610 (48% of space)

### Step 4: Implement Professional Styling
```json
{
  "visualContainerObjects": {
    "title": [{
      "properties": {
        "text": { "expr": { "Literal": { "Value": "'Card Title'" } } }
      }
    }],
    "background": [{
      "properties": {
        "show": { "expr": { "Literal": { "Value": "true" } } },
        "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#FFFFFF'" } } } } }
      }
    }],
    "border": [{
      "properties": {
        "show": { "expr": { "Literal": { "Value": "true" } } },
        "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#4472C4'" } } } } },
        "radius": { "expr": { "Literal": { "Value": "6D" } } },
        "width": { "expr": { "Literal": { "Value": "1D" } } }
      }
    }],
    "dropShadow": [{
      "properties": {
        "show": { "expr": { "Literal": { "Value": "true" } } },
        "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#4472C4'" } } } } },
        "angle": { "expr": { "Literal": { "Value": "180D" } } }
      }
    }]
  }
}
```

---

## Phase 4: Field Binding & Data Connection

### Step 1: Understanding Field Types

**Measure Fields:**
Use `type: "measure"` for DAX-defined calculations:
```json
{
  "bucket": "Values",
  "fields": [{"field": "FactSales[Total Revenue]", "type": "measure"}]
}
```

**Column Fields:**
Use `type: "column"` for raw columns:
```json
{
  "bucket": "Values",
  "fields": [{"field": "FactSales[Revenue]", "type": "column"}]
}
```

**Aggregation Fields:**
Use `type: "aggregation"` with aggregation function:
```json
{
  "bucket": "Values",
  "fields": [{
    "field": "FactSales[Revenue]",
    "type": "aggregation",
    "aggregation": "Avg"
  }]
}
```

### Step 2: Bind KPI Cards Using pbir_bulk_bind

**Tool:** `mcp__powerbi-report-mcp__pbir_bulk_bind`

```json
{
  "pageId": "e601a55fefa0934dbe1b",
  "updates": [
    {
      "visualId": "3f6b5f76c5350c8b18e8",
      "bindings": [{
        "bucket": "Values",
        "fields": [{"field": "FactSales[Avg Unit Price]", "type": "measure"}]
      }]
    },
    {
      "visualId": "4ff7a494421fdd4cc237",
      "bindings": [{
        "bucket": "Values",
        "fields": [{"field": "FactSales[Gross Profit]", "type": "measure"}]
      }]
    }
  ]
}
```

### Step 3: Handle Column Aggregations
For fields without predefined measures, use aggregations:

**Average Revenue/Product:**
```json
{
  "field": "FactSales[Revenue]",
  "type": "aggregation",
  "aggregation": "Avg"
}
```

**Product Count:**
```json
{
  "field": "DimProduct[ProductKey]",
  "type": "aggregation",
  "aggregation": "Count"
}
```

### Step 4: Validate Bindings
- Verify all fields exist in semantic model
- Check field types match (column vs. measure)
- Ensure aggregation functions are appropriate
- Use validation tools to catch errors before opening in Desktop

---

## Phase 5: Visual Alignment & Formatting

### Step 1: List All Visuals
**Tool:** `mcp__powerbi-report-mcp__pbir_list_visuals`

```json
{
  "pageId": "e601a55fefa0934dbe1b",
  "visualType": "card",
  "limit": 100
}
```

### Step 2: Position KPI Cards
**Tool:** `mcp__powerbi-report-mcp__pbir_move_visual`

**Alignment Formula:**
```
Total available width: 1220px (x=15 to x=1235)
Card width: 280px × 4 = 1120px
Available gaps: 1220 - 1120 = 100px
Gap size: 100px ÷ 3 gaps ≈ 33.33px

Positions:
- Card 1: x = 15
- Card 2: x = 15 + 280 + 33 = 328
- Card 3: x = 328 + 280 + 34 = 642
- Card 4: x = 642 + 280 + 33 = 955
```

### Step 3: Apply Consistent Formatting
Use `pbir_format_visual` for:
- Font sizes and colors
- Border styles and colors
- Background colors
- Drop shadow effects

### Step 4: Reload Report
**Tool:** `mcp__powerbi-report-mcp__pbir_reload_report`

```
confirm: true
```

---

## Troubleshooting Guide

### Issue 1: Cards Not Displaying Values

**Root Cause:** Incorrect field type or missing aggregation

**Solution:**
1. Verify field exists in semantic model
2. Check if field is a measure or column
3. For columns, add appropriate aggregation
4. Use `pbir_bulk_bind` to rebind fields

**Example Fix:**
```json
// WRONG - using column without aggregation
{"field": "FactSales[Revenue]", "type": "column"}

// CORRECT - using measure
{"field": "FactSales[Total Revenue]", "type": "measure"}

// CORRECT - using column with aggregation
{"field": "FactSales[Revenue]", "type": "aggregation", "aggregation": "Avg"}
```

### Issue 2: Alignment Inconsistencies

**Root Cause:** Different x-positions for cards on different pages

**Solution:**
1. Analyze row 2 visual positions (left and right bounds)
2. Calculate equal spacing for 4 cards
3. Apply same positions to all pages
4. Verify left card aligns with left visual
5. Verify right card aligns with right visual

### Issue 3: Malformed JSON in Visual Files

**Root Cause:** Incomplete bracket structures, missing commas, or improperly formatted objects

**Symptoms:**
- "Unexpected character encountered while parsing value"
- "Invalid JSON primitive"
- Visual doesn't load

**Prevention:**
1. Always validate JSON before saving
2. Use proper escaping for quotes
3. Ensure all arrays and objects are properly closed
4. Use UTF-8 without BOM encoding

**Fix:**
```powershell
# Validate JSON
$json | ConvertFrom-Json -ErrorAction Stop

# Save with proper encoding
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllText($filePath, $content, $utf8NoBom)
```

### Issue 4: Empty Filter Names

**Root Cause:** Filter ID generation failed

**Solution:**
```json
"filterConfig": {
  "filters": [{
    "name": "fc541b73e3248137ebbd",  // 16-20 char hex string
    "field": {...},
    "type": "Advanced"
  }]
}
```

### Issue 5: Category Labels Still Visible

**Root Cause:** categoryLabels object not present or show property incorrect

**Solution:**
```json
"objects": {
  "categoryLabels": [{
    "properties": {
      "show": {
        "expr": {
          "Literal": {"Value": "false"}
        }
      }
    }
  }]
}
```

---

## Best Practices

### Data Modeling
1. **Use Measures for Calculations**
   - Define all aggregations as DAX measures
   - Improves performance and reusability
   - Ensures consistency across visuals

2. **Proper Column Summarization**
   - Set `summarizeBy: none` for dimensions (keys, dates, categories)
   - Set `summarizeBy: sum` for numeric facts
   - Set `summarizeBy: average` for rates and ratios

3. **Format Strings**
   - Currency: `$#,0.00`
   - Quantity: `#,0`
   - Percentage: `0.0%`
   - Always match format to data type

### Report Design
1. **3-30-300 Principle**
   - Row 1: KPI cards (3 seconds)
   - Rows 2-3: Charts and tables (30 seconds + 300 seconds)

2. **Color Scheme**
   - Use corporate colors: Gold (#D4A574), Blue (#4472C4), Green (#70AD47), Orange (#C55A11)
   - Dark background (#1a2332) for contrast
   - White cards (#FFFFFF) for readability

3. **Consistent Spacing**
   - Equal gaps between visuals
   - Align all visuals to grid
   - Maintain consistent margins

### File Management
1. **Encoding**
   - Always use UTF-8 without BOM
   - Prevents encoding errors when opening in Desktop

2. **Validation**
   - Validate JSON before saving
   - Test field bindings against semantic model
   - Verify visual positions before reload

3. **Backup**
   - Keep original Cognos report for reference
   - Version control TMDL and JSON files
   - Document custom measures and calculations

---

## Tools & MCP Resources

### MCP Servers Used

#### powerbi-authoring-local
**Purpose:** Manage semantic model structure
**Key Operations:**
- `database_operations` - TMDL export/import
- `table_operations` - Create and configure tables
- `measure_operations` - Define DAX measures
- `relationship_operations` - Create relationships

#### powerbi-report-mcp
**Purpose:** Design and format reports
**Key Operations:**
- `pbir_list_visuals` - List all visuals on a page
- `pbir_add_visual` - Create new visuals
- `pbir_bulk_bind` - Rebind multiple visuals
- `pbir_move_visual` - Reposition visuals
- `pbir_format_visual` - Apply formatting
- `pbir_reload_report` - Refresh in Desktop

### Command-Line Tools

**PowerShell:**
```powershell
# Validate JSON
$content | ConvertFrom-Json

# Encode to UTF-8 without BOM
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllText($path, $content, $utf8NoBom)

# Generate random IDs
[char[]](48..57 + 97..102) | Sort-Object {Get-Random} | Select-Object -First 16
```

**M Language (Power Query):**
```m
// Standard table generation pattern
let
  data = Table.FromRecords({
    [Column1=value1, Column2=value2],
    [Column1=value1, Column2=value2]
  })
in
  data
```

### File Format References

**TMDL (Tabular Model Definition Language):**
- Located in: `SemanticModel/definition/`
- Files: database.tmdl, model.tmdl, relationships.tmdl, tables/*.tmdl
- Format: Declarative model definition

**JSON (Report Definition):**
- Located in: `Report/definition/pages/*/`
- Schema: Microsoft Power BI JSON schemas v2.1.0+
- Must be valid JSON (no comments, proper escaping)

---

## Summary Checklist

### Semantic Model ✅
- [ ] Database configuration (database.tmdl)
- [ ] Model metadata (model.tmdl)
- [ ] All tables created with data
- [ ] Columns configured with data types
- [ ] All measures defined with DAX
- [ ] Relationships established
- [ ] Format strings applied

### Report Pages ✅
- [ ] 3 pages created with proper names
- [ ] Dark background applied
- [ ] 12 KPI cards added
- [ ] Cards positioned and sized
- [ ] 8 analytical charts/tables added
- [ ] All visuals aligned properly

### Data Binding ✅
- [ ] All 12 cards bound to correct fields
- [ ] Field types verified (measure vs. column)
- [ ] Aggregations applied where needed
- [ ] Filter configurations set correctly
- [ ] Category labels disabled

### Formatting ✅
- [ ] Card styling applied (borders, shadows, colors)
- [ ] Consistent spacing between visuals
- [ ] Alignment with visual grid
- [ ] Professional color scheme
- [ ] All files encoded UTF-8 without BOM

### Validation ✅
- [ ] All JSON valid and parseable
- [ ] All fields exist in semantic model
- [ ] Report opens without errors
- [ ] All visuals display values
- [ ] All alignments verified

---

## References

### Microsoft Documentation
- [Power BI PBIP Format](https://learn.microsoft.com/power-bi/developer/projects/projects-overview)
- [TMDL Reference](https://learn.microsoft.com/analysis-services/tmdl/tmdl-overview)
- [DAX Function Reference](https://learn.microsoft.com/dax/dax-function-reference)
- [Power Query M Reference](https://learn.microsoft.com/powerquery-m/)

### Related Workflows
- See adjacent conversion projects for similar report types
- Reference dashboard styling in templates/
- Use sample TMDL files as templates for new tables

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-09-30 | Initial documentation |

---

**Last Updated:** 2026-09-30  
**Maintained By:** Analytics Team  
**Status:** ✅ Complete & Validated
