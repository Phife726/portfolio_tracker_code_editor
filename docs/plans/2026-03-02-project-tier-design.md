# Project Tier Feature Design

**Date:** 2026-03-02
**Status:** Approved

## Overview

Add Project Tier (an existing required SharePoint Choice field) to three screens:

1. **EditScreen1** — new form field (2nd position, after Title)
2. **BrowseScreen1** — new filter toggle (defaults to Tier 1 only)
3. **DetailScreen2** — subtle badge in the hero banner for quick reference

## SharePoint Field

- Display name: `Project Tier`
- Internal name: `Project_x0020_Tier`
- Power Fx reference: `'Project Tier'` (with `.Value` accessor for filtering/display)
- Type: Single-select Choice (required)
- Known values: `"Tier 1: Strategic Portfolio"`, `"Tier 2: ..."`

---

## Section 1 — EditScreen1: Project Tier as 2nd Form Field

### Control

A new `TypedDataCard` named `ProjectTier_DataCard1` inserted as the second child of `EditForm1`, immediately after `Title_DataCard2`.

### Pattern

Follows the existing `ClassicComboBoxEdit` pattern used by `Department_DataCard2`:

| Property | Value |
|---|---|
| Variant | `ClassicComboBoxEdit` |
| DataField | `"Project_x0020_Tier"` |
| Default | `=ThisItem.'Project Tier'` |
| Required | `=true` |
| Update | `=DataCardValuePT.Selected` |

Child controls mirror `Department_DataCard2`:
- `DataCardKeyPT` — label (`Text: =Parent.DisplayName`, Height=48, Y=10)
- `DataCardValuePT` — `Classic/ComboBox`, `Items: =Choices([@'Project Portfolio Tracker'].'Project Tier')`, `SelectMultiple: false`
- `ErrorMessagePT` — error label
- `StarVisiblePT` — required star

### Files Changed

- `Src/EditScreen1.pa.yaml`
- `Controls/65.json`

---

## Section 2 — BrowseScreen1: Tier 1 Filter Toggle

### New Control

`chk_TierFilter` toggle placed on a second filter row below `chk_MyProjects`.

| Property | Value |
|---|---|
| Control | `Classic/Toggle@2.1.0` |
| Default | `=true` (Tier 1 only on load) |
| TrueText | `"Tier 1 Only"` |
| FalseText | `"Show All Tiers"` |
| TrueFill | `=RGBA(56, 96, 178, 1)` |
| X | `=20` |
| Y | `=130` |
| Width | `=274` |
| Height | `=31` |

### Layout Adjustments

| Control | Property | Before | After |
|---|---|---|---|
| `TextSearchBox1` | `Y` | `=153` | `=168` |
| `BrowseGallery1` | `Y` | `=221` | `=239` |
| `BrowseGallery1` | `Height` | `=914` | `=896` |

### Gallery Filter Change

Add a 5th condition to `BrowseGallery1.Items`:

```
// CONDITION 5: Tier Filter
(!chk_TierFilter.Value || 'Project Tier'.Value = "Tier 1: Strategic Portfolio")
```

Full updated Items formula:

```
=SortByColumns(
    Filter(
        [@'Project Portfolio Tracker'],

        // CONDITION 1: Search Text
        StartsWith(Title, TextSearchBox1.Text),

        // CONDITION 2: My Projects Logic
        (
            !chk_MyProjects.Value ||
            'Executive Sponsor'.Email = User().Email ||
            User().Email in 'Project Stakeholders'.Email
        ),

        // CONDITION 3: Department Filter
        (
            IsBlank(drp_Department.Selected.Value) ||
            Department.Value = drp_Department.Selected.Value
        ),

        // CONDITION 4: Status Exclusion
        Stage.Value <> "Cancelled",
        Stage.Value <> "Completed",

        // CONDITION 5: Tier Filter
        (
            !chk_TierFilter.Value ||
            'Project Tier'.Value = "Tier 1: Strategic Portfolio"
        )
    ),

    // SORT LOGIC
    "Title",
    If(SortDescending1, SortOrder.Descending, SortOrder.Ascending)
)
```

### Files Changed

- `Src/BrowseScreen1.pa.yaml`
- `Controls/4.json`

---

## Section 3 — DetailScreen2: Subtle Tier Badge in Hero Banner

### New Control

`LblTierBadge` — a small right-aligned label placed on the same row as the RAG badge (`LblRAGBadge`). The RAG badge is left-aligned and short, so there is no visual conflict.

| Property | Value |
|---|---|
| Control | `Label@2.5.1` |
| Y | `=98` |
| X | `=Parent.Width - 136 - 16` (right-aligned with 16px margin) |
| Width | `=136` |
| Height | `=24` |
| Text | `=If(IsBlank(BrowseGallery1.Selected.'Project Tier'.Value), "", Left(BrowseGallery1.Selected.'Project Tier'.Value, 6))` → displays `"Tier 1"` or `"Tier 2"` |
| Align | `=Align.Right` |
| Size | `=11` |
| FontWeight | `=FontWeight.Semibold` |
| Color | `=RGBA(255, 255, 255, 0.75)` (75% opacity white — subtle on the RAG banner) |
| Font | `=Font.'Open Sans'` |
| BorderColor | `=RGBA(0,0,0,0)` |

No existing controls need repositioning.

### Files Changed

- `Src/DetailScreen2.pa.yaml`
- `Controls/24.json`

---

## Repack

After all edits, rebuild using the standard explicit-path zip command (24 files, no new files added):

```bash
rm /workspaces/portfolio_tracker_code_editor/Americhem_Portfolio_Tracker_v1.msapp
cd /tmp/msapp_extracted && zip /workspaces/portfolio_tracker_code_editor/Americhem_Portfolio_Tracker_v1.msapp \
  .gitignore Header.json Properties.json AppCheckerResult.sarif \
  References/AuxDataMap.json References/Themes.json References/ModernThemes.json \
  References/DataSources.json References/Templates.json References/Resources.json \
  Controls/1.json Controls/4.json Controls/24.json \
  Controls/65.json Controls/197.json Controls/217.json \
  Resources/PublishInfo.json \
  Src/App.pa.yaml Src/HomeScreen.pa.yaml Src/BrowseScreen1.pa.yaml \
  Src/DetailScreen2.pa.yaml Src/EditScreen1.pa.yaml \
  Src/StatusUpdateScreen_2.pa.yaml Src/_EditorState.pa.yaml
```

Verify with `unzip -l` — confirm 24 files, no duplicates.
