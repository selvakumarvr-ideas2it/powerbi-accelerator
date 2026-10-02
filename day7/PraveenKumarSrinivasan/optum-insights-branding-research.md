# Optum Insights: Brand and Color Research for a Power BI Theme

**Researched:** 2026-09-30
**Purpose:** Reference for building a custom Power BI report theme (JSON) that matches Optum / Optum Insight branding.

---

## 1. Company context (market research summary)

| Item | Detail |
|---|---|
| Parent | UnitedHealth Group (UHG). Four reportable segments: UnitedHealthcare, Optum Health, **Optum Insight**, Optum Rx |
| Optum Insight's role | Data, analytics, technology and consulting for payers, care providers, employers, governments and life-sciences companies. Its focus areas are clinical insight, lower care costs, better care quality and revenue-cycle / payment integrity |
| 2026 change | On 1 Jan 2026, **Optum Financial (including Optum Bank)** moved from Optum Health into Optum Insight |
| Scale | Q1 2026 revenue: **$5.1B** (vs $5.0B in Q1 2025) |
| Brand architecture | Optum Insight has **no separate visual identity**. It uses the master **Optum** brand (orange wordmark). "Optum Insight" appears as a descriptor next to the Optum wordmark |
| Brand evolution | Optum rebranded in **2021–2022** (Brand New covered it on 31 May 2022). The older "O" symbol was dropped, and a bold **title-case "Optum" wordmark in solid orange** replaced it. The palette also moved from the legacy orange (#E87722) to a brighter orange (#FF612B) set against warm off-white |
| Brand personality | Warm, modern, helpful, optimistic. The brand calls its palette "bright and cheerful". The orange sets Optum apart from UHG's corporate blue |

> **Brand-evolution warning:** many older PDFs and templates still show the **legacy (pre-2022) palette**: Bright Orange #E87722, Bold Orange #C25608, Marigold #F2B411, and the Frutiger font. **Do not use these for new reports.** The current palette is below.

---

## 2. Current Optum color palette (verified from live optum.com CSS)

These hex values were **taken directly from the design tokens (`--dmp-color-*` CSS custom properties) in the production optum.com / business.optum.com stylesheets** (`clientlib-optum`, `clientlib-base`) on 2026-09-30. Where they exist, the token names from Optum's own design system are kept.

### 2.1 Core brand colors

| Role | Name | HEX | RGB | Source token / note |
|---|---|---|---|---|
| **Primary brand** | Optum Orange | **#FF612B** | 255, 97, 43 | `--dmp-color-theme-brand-50`; DPL `$color-orange`; Pantone **165 C**; CMYK 0/68/96/0 |
| Brand background | Warm White (cream) | **#FAF8F2** | 250, 248, 242 | `--dmp-color-theme-primary-5`, `--dmp-color-header-bar` |
| Surface | White | **#FFFFFF** | 255, 255, 255 | `--dmp-color--background-primary` |
| Primary text | Charcoal | **#3D3C38** | 61, 60, 56 | `--dmp-color--text-primary`. The **most-used color on the site** (121 uses) |
| Headings | Black | **#000000** | 0, 0, 0 | `--dmp-heading-color` |

### 2.2 Accent / theme colors

| Name | HEX | Token | Use |
|---|---|---|---|
| Light Peach | **#FFD1AB** | `--dmp-color-theme-accent-1` | Soft highlight fills, backgrounds |
| Peach | **#F9A667** | `--dmp-color-theme-accent-2` | Secondary warm accent |
| Sky Blue | **#D9F6FA** | `--dmp-color-theme-accent-3` / `primary-10`; DPL `$color-sky-blue` | Cool contrast panels, callouts |
| Sand / Stone | **#EDE8E0** | `--dmp-color-theme-accent-4` | Neutral warm panel background |

### 2.3 "Vibrant" interaction oranges (accessible oranges for text and buttons)

| Name | HEX | Token | Note |
|---|---|---|---|
| Orange Tint 1 | #FFF8F6 | `vibrant-interaction-1` | Hover background |
| Orange Tint 2 | #FBDED6 | `vibrant-interaction-2` | Active / selected background |
| Vibrant Orange | **#D74120** | `vibrant-interaction-3` | Button background (4.5:1 on white, so AA text is OK) |
| Vibrant Orange Hover | #C93919 | `vibrant-interaction-4` | Hover |
| Vibrant Orange Dark | **#B83C20** | `vibrant-interaction-5` | Active (5.7:1 on white) |

### 2.4 Warm neutral scale (Optum grays have a warm, brown undertone, not blue-gray)

| Token | HEX | Token | HEX |
|---|---|---|---|
| neutral-1 | #FFFFFF | neutral-60 | #84827A |
| neutral-5 | #FAFAFA | neutral-70 | **#73716A** |
| neutral-10 | #F3F3F3 | neutral-80 | #504F4A |
| neutral-20 | #E7E6E4 | neutral-90 | **#3D3C38** |
| neutral-30 | #CECDCA | neutral-100 | #232220 |
| neutral-50 | #989790 | neutral-110 | #000000 |

Also seen: article borders #E4E1DA, muted text #78766F, and a legacy text gray #4B4D4F (Optum Sans modules).

### 2.5 Semantic / status colors

| State | Foreground | Background |
|---|---|---|
| Success | **#066605** | #E0F3DF |
| Info | **#095F87** | #DFF3F9 |
| Warning | **#FD602B** (orange) | #FFE8CD |
| Danger / Error | **#D71515** | #FCF0F0 |
| Links | #095F87, hover #05344A | |
| Focus ring | #0C55B8 | |
| Rating stars | #FAAF00 | |

### 2.6 Blues present on the site (UHG family, secondary)

| HEX | Where it's used |
|---|---|
| **#002677** | UHG / UnitedHealthcare corporate navy. Used as `--color-secondary` in provider modules and focus rings (71 uses) |
| #0C55B8 | Focus outlines, interactive blue |
| #00396C, #001E60, #003D7F | Dark navies in legacy / UHC components |

These blues are **not the Optum brand color**, but they show up alongside it on Optum pages. They make a useful **cool counterpoint** in a chart palette, and they tie Optum Insight reports to the UHG family.

---

## 3. Typography

| Context | Font | Notes |
|---|---|---|
| Optum web (current) | **Enterprise Sans** (Regular, Medium, Bold, XBold, Condensed) | Primary family (`--dmp-font-family-primary`). Headings use **Enterprise Sans Bold** |
| Editorial accent | **Enterprise Serif Text** (Regular, Medium, Semibold) | Used for pull quotes and editorial headlines |
| Optum products / apps | **Optum Sans** | Accessible product font (Optum DPL) |
| Fallback chain (official) | `Helvetica, Arial, sans-serif` | From the CSS font stacks |
| Legacy (pre-2022) | Frutiger | Don't use |

Web conventions: heading `line-height: 1`, H3 = 1.875rem (30px), body text 16–18px with 24px line height, and headings in black while body text is charcoal #3D3C38.

**Power BI implication:** Enterprise Sans and Optum Sans are proprietary and **not available in Power BI Service**. Viewers without the font installed will see a fallback. Recommended:
- **Segoe UI** (body/labels) and **Segoe UI Semibold / Bold** (titles, KPI values). Power BI renders these natively, and they are closest in feel to Enterprise Sans.
- Or **Arial**, which is in Optum's own official fallback chain. Use it if strict brand fidelity matters more than on-screen polish.

---

## 4. Accessibility: measured WCAG contrast ratios

| Color | vs White | vs Cream #FAF8F2 | Verdict for Power BI |
|---|---|---|---|
| Optum Orange #FF612B | 3.00 | 2.83 | OK for **bars, lines, large KPI numbers and accents**. **Not** for small text (fails AA 4.5) |
| Vibrant Orange #D74120 | 4.50 | 4.24 | Use when orange **text** is needed on white |
| Vibrant Orange Dark #B83C20 | 5.67 | 5.34 | Safe orange text anywhere |
| Charcoal #3D3C38 | 11.04 | 10.40 | Default text: excellent |
| Warm Gray 70 #73716A | 4.88 | 4.60 | Secondary text, axis labels: passes AA |
| Warm Gray 50 #989790 | 2.93 | 2.76 | Gridlines and borders only, never text |
| Navy #002677 | 13.60 | 12.81 | Excellent. Strong second series |
| Info Blue #095F87 | 7.00 | 6.59 | Excellent |
| Peach #F9A667 | 1.97 | 1.85 | Fills only, needs a darker neighbor. Charcoal text on it = 5.61 ✓ |
| Light Peach #FFD1AB | 1.40 | 1.32 | Background / highlight only |
| Sky Blue #D9F6FA | 1.13 | 1.07 | Panel background only (charcoal text on it = 9.73 ✓) |

Brand rule (Optum guideline): on **dark** backgrounds use **white** text only. On **light** backgrounds use dark gray / charcoal text only.

---

## 5. Recommended Power BI theme mapping (derived from the findings)

> Everything in Section 2 is **sourced**. The ordering and pairing below are **my recommendations** for charts, since Optum publishes no public data-viz palette.

### 5.1 `dataColors` (series order)
Warm and cool colors alternate so neighboring series stay distinguishable, including for color-vision deficiency (orange/blue is the safest pairing).

| # | HEX | Name |
|---|---|---|
| 1 | #FF612B | Optum Orange |
| 2 | #002677 | Navy |
| 3 | #F9A667 | Peach |
| 4 | #095F87 | Info Blue (teal-blue) |
| 5 | #73716A | Warm Gray |
| 6 | #B83C20 | Burnt / Vibrant Orange Dark |
| 7 | #0C55B8 | Interactive Blue |
| 8 | #FFD1AB | Light Peach |
| 9 | #3D3C38 | Charcoal |
| 10 | #989790 | Light Warm Gray |

### 5.2 Structural colors
| Theme property | Value |
|---|---|
| `background` | #FFFFFF |
| `secondaryBackground` / page wallpaper | #FAF8F2 (warm white). Visuals sit on white cards on a cream canvas, which matches optum.com |
| `foreground` (text) | #3D3C38 |
| `foregroundNeutralSecondary` (labels) | #73716A |
| `foregroundNeutralTertiary` | #989790 |
| `backgroundLight` | #F3F3F3 |
| `backgroundNeutral` / borders | #CECDCA |
| gridlines | #E7E6E4 |
| `tableAccent` | #FF612B |
| `hyperlink` | #095F87 |
| `visitedHyperlink` | #05344A |
| title text | #000000 |

### 5.3 Sentiment / KPI
| Property | Value |
|---|---|
| `good` | #066605 |
| `neutral` | #FAAF00 (or #73716A for a quieter neutral) |
| `bad` | #D71515 |
| `maximum` / `center` / `minimum` (diverging) | #B83C20 / #FAF8F2 / #095F87 |

### 5.4 Sequential ramp (single-hue orange, light → dark)
`#FFF8F6 → #FBDED6 → #FFD1AB → #F9A667 → #FF612B → #D74120 → #B83C20`
(Every step is an Optum token.)

### 5.5 Text classes
| Class | Font | Size | Color |
|---|---|---|---|
| title | Segoe UI Semibold | 14 | #000000 |
| header | Segoe UI Semibold | 12 | #3D3C38 |
| label | Segoe UI | 10 | #3D3C38 |
| callout (KPI value) | Segoe UI Bold | 28 | #3D3C38 (or #FF612B for the hero KPI; it's large text, so 3:1 is acceptable) |

### 5.6 Styling cues from optum.com that are worth copying
- Warm off-white page background with white cards, and **no heavy borders**. Separate with whitespace or a subtle #E7E6E4 line.
- Use orange **sparingly as a highlight** (one hero series or KPI, an accent bar under titles). Don't flood the page with it.
- Primary buttons are **charcoal (#3D3C38)**, not orange. Orange is reserved for brand moments and "vibrant" CTAs (#D74120).
- Rounded corners (pills, cards). A 4–8px visual border radius fits.
- Grays should always be the warm neutral scale. Avoid cool blue-grays such as #64748B.

---

## 6. Draft theme JSON (starting point)

```json
{
  "name": "Optum Insight",
  "dataColors": ["#FF612B","#002677","#F9A667","#095F87","#73716A","#B83C20","#0C55B8","#FFD1AB","#3D3C38","#989790"],
  "background": "#FFFFFF",
  "secondaryBackground": "#FAF8F2",
  "foreground": "#3D3C38",
  "foregroundNeutralSecondary": "#73716A",
  "foregroundNeutralTertiary": "#989790",
  "backgroundLight": "#F3F3F3",
  "backgroundNeutral": "#CECDCA",
  "tableAccent": "#FF612B",
  "hyperlink": "#095F87",
  "visitedHyperlink": "#05344A",
  "good": "#066605",
  "neutral": "#FAAF00",
  "bad": "#D71515",
  "maximum": "#B83C20",
  "center": "#FAF8F2",
  "minimum": "#095F87",
  "textClasses": {
    "title":   { "fontFace": "Segoe UI Semibold", "fontSize": 14, "color": "#000000" },
    "header":  { "fontFace": "Segoe UI Semibold", "fontSize": 12, "color": "#3D3C38" },
    "label":   { "fontFace": "Segoe UI", "fontSize": 10, "color": "#3D3C38" },
    "callout": { "fontFace": "Segoe UI Bold", "fontSize": 28, "color": "#3D3C38" }
  },
  "visualStyles": {
    "page": { "*": { "background": [{ "color": { "solid": { "color": "#FAF8F2" } }, "transparency": 0 }] } },
    "*": { "*": {
      "background": [{ "show": true, "color": { "solid": { "color": "#FFFFFF" } }, "transparency": 0 }],
      "border": [{ "show": false }],
      "visualHeader": [{ "foreground": { "solid": { "color": "#73716A" } } }]
    } }
  }
}
```

---

## 7. Sources

- Live CSS design tokens: https://www.optum.com/ and https://business.optum.com/en/ (`/etc.clientlibs/dmp/clientlibs/clientlib-optum…min.css`, `clientlib-base.css`), extracted 2026-09-30
- Optum Orange #FF612B / Pantone 165 C: https://www.brandcolorcode.com/optum
- Optum DPL color tokens (`$color-orange` #FF612B, `$color-sky-blue` #D9F6FA): https://dpl.myoptum.com/app/elements/color.html (not reachable from this network; values confirmed via search index)
- Optum Sans typography: https://dpl.myoptum.com/web/design-basics/typography/
- Legacy palette (#E87722 / #C25608 / #F2B411, Frutiger): https://logotyp.us/logo/optum/ and https://1000logos.net/optum-logo/
- Rebrand coverage (31 May 2022): https://www.underconsideration.com/brandnew/archives/new_logo_for_optum.php
- Segments, 2026 realignment and Q1 2026 revenue: https://www.sec.gov/Archives/edgar/data/0000731766/000073176626000121/earningsrelease1q26press.htm and https://www.sec.gov/Archives/edgar/data/0000731766/000073176626000127/unh-20260331.htm

**Caveats:** the official brand portal (brand.optum.com) and the Optum DPL sites require internal access or were blocked from this network, so the palette comes from the **production website CSS**. That is authoritative for what Optum actually ships, but it is not the formal brand book. Optum publishes no public data-viz palette, so Section 5 is a recommendation built from sourced brand tokens. If you have access to Optum's internal brand center, check it for an official chart palette.
