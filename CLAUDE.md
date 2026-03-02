# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Microsoft Power Apps Canvas application** (`.msapp` format) for enterprise project portfolio management at Americhem. The `.msapp` file is a ZIP archive containing YAML screen definitions, JSON control/reference files, and app metadata.

There is no traditional build system, package manager, or test runner. All development, testing, and deployment occurs through **Power Apps Studio** (the web-based maker portal at make.powerapps.com).

## Working with the .msapp File

Extract the app to `/tmp/msapp_extracted/` for editing:
```bash
mkdir -p /tmp/msapp_extracted && cd /tmp/msapp_extracted
unzip -o /workspaces/portfolio_tracker_code_editor/Americhem_Portfolio_Tracker_v1.msapp
```

Internal structure:
- `Src/*.pa.yaml` — Screen and component logic in Power Fx YAML format
- `Controls/*.json` — Control definitions with properties (**authoritative for import**)
- `References/DataSources.json` — SharePoint connection and full list schema
- `References/Themes.json` — Americhem brand theme (primary blue: `#00509E` / RGB(56,96,178))
- `Properties.json` — App-level metadata (portrait phone layout, 640×1136)
- `AppCheckerResult.sarif` — Static analysis output from Power Apps AppChecker

### CRITICAL: Controls JSON is Authoritative

MSAppStructureVersion 2.4.0 reads `Controls/*.json` on import — **editing only `Src/*.pa.yaml` has NO effect**. Always edit BOTH the Controls JSON and the corresponding YAML when making changes.

### Repack Command

Always use explicit file paths. Never use `zip -r output.msapp .` (creates duplicates, never removes deleted files).

```bash
rm /workspaces/portfolio_tracker_code_editor/Americhem_Portfolio_Tracker_v1.msapp
cd /tmp/msapp_extracted && zip /workspaces/portfolio_tracker_code_editor/Americhem_Portfolio_Tracker_v1.msapp \
  .gitignore Header.json Properties.json AppCheckerResult.sarif \
  References/AuxDataMap.json References/Themes.json References/ModernThemes.json \
  References/DataSources.json References/Templates.json References/Resources.json \
  Controls/1.json Controls/4.json Controls/64.json \
  Controls/129.json Controls/382.json Controls/388.json \
  Src/App.pa.yaml Src/HomeScreen.pa.yaml Src/BrowseScreen1.pa.yaml \
  Src/DetailScreen2.pa.yaml Src/EditScreen1.pa.yaml \
  Src/StatusUpdateScreen_2.pa.yaml Src/_EditorState.pa.yaml
```

Verify with `unzip -l` after building — confirm no duplicates or stale entries.

## Architecture

**Pattern**: Master-detail with wizard-style navigation
**Layout**: Portrait phone, 640×1136 px
**Data backend**: SharePoint Online list "Project Portfolio Tracker" at `https://americhemo365.sharepoint.com/sites/PMO`

### Screen Flow

```
HomeScreen → BrowseScreen1 (list/search/sort)
           → EditScreen1 (new project wizard)
BrowseScreen1 → DetailScreen2 (read-only detail view)
             → StatusUpdateScreen_2 (quick RAG status update)
             → EditScreen1 (edit existing project)
```

### Screen → Controls JSON Mapping

| Controls file | Screen |
|---|---|
| `Controls/1.json` | App |
| `Controls/4.json` | BrowseScreen1 |
| `Controls/64.json` | DetailScreen2 |
| `Controls/129.json` | EditScreen1 |
| `Controls/382.json` | HomeScreen |
| `Controls/388.json` | StatusUpdateScreen_2 |

### Global Variables (initialized in App.pa.yaml OnStart)

- `gblUser` — current user object (`User()`)
- `gblThemeColor` — Americhem brand blue (`ColorValue("#00509E")`)
- `locWizardStep` — wizard step counter (starts at 1)

## Key Data Model Fields

Choice fields use `.Value` accessor (e.g., `Stage.Value`, `'RAG Status'.Value`).

| Display Name | Internal SP field | Type |
|---|---|---|
| Title | `Title` | string |
| Department | `field_1` | Choice |
| Support Function(s) | `field_2` | Multi-choice |
| BU | `field_3` | Choice |
| SBG | `field_4` | Multi-choice |
| Site(s) | `field_5` | Multi-choice |
| Pillar | `field_8` | Choice |
| Category | `field_7` | Choice |
| Priority | `field_9` | Choice |
| Stage | `field_10` | Choice |
| RAG Status | `RAG_x0020_Status` | Choice (default: Green) |
| Start Date | `field_11` | date |
| Target Date | `field_12` | date |
| KPI Metric | `KPI_x0020_Metric` | Choice |
| KPI Target Value | `KPI_x0020_Target_x0020_Value` | number |
| KPI Actual Value | `KPI_x0020_Actual_x0020_Value` | number |
| Spend Type | `field_15` | Choice |
| Cost Est. (USD) | `field_16` | number |
| Estimation Confidence | `Estimation_x0020_Confidence` | Choice (default: Medium +/- 25%) |
| Current Blocker | `Current_x0020_Blocker` | string |
| Additional Details | `field_17` | string |
| Mandatory | `Mandatory` | boolean (default: false) |
| Recurring | `Recurring_2` | boolean (default: false) |

## Power Fx Conventions

**Navigation**: `Navigate(ScreenName)` or `Navigate(EditScreen1, ScreenTransition.None)`
**New form**: `NewForm(EditForm1); Navigate(EditScreen1, ScreenTransition.None)`
**Form reset**: `ResetForm(FormName)`
**Data refresh**: `Refresh([@'Project Portfolio Tracker'])`
**Data source reference**: `[@'Project Portfolio Tracker']` (with `@` disambiguation prefix)

**DetailScreen2 data binding** — reads directly from gallery selection (no form control):
```
BrowseGallery1.Selected.Title
BrowseGallery1.Selected.'RAG Status'.Value
BrowseGallery1.Selected.Stage.Value
Text(BrowseGallery1.Selected.'Cost Est. (USD)', "$#,##0")
Text(BrowseGallery1.Selected.'Start Date', "mmm d, yyyy")
```

**Permission check pattern**:
```
DataSourceInfo([@'Project Portfolio Tracker'], DataSourceInfo.CreatePermission)
```

**RAG color values**:
- Red/Critical: `RGBA(196, 43, 28, 1)`
- Amber/At Risk: `RGBA(210, 130, 0, 1)`
- Green/On Track: `RGBA(0, 145, 65, 1)`
- Brand blue: `RGBA(56, 96, 178, 1)`
- Dark navy: `RGBA(0, 18, 107, 1)`

**RAG conditional color**:
```
If(rag="Red", RGBA(196,43,28,1), rag="Amber", RGBA(210,130,0,1), RGBA(0,145,65,1))
```

**BrowseScreen1 local variable**: `SortDescending1` (toggled by sort icon `OnSelect`)

**Active project filter** (excludes Cancelled/Completed):
```
Filter([@'Project Portfolio Tracker'], Stage.Value <> "Cancelled", Stage.Value <> "Completed")
```
