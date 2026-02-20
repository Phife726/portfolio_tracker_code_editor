# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Microsoft Power Apps Canvas application** (`.msapp` format) for enterprise project portfolio management at Americhem. The `.msapp` file is a ZIP archive containing YAML screen definitions, JSON control/reference files, and app metadata.

There is no traditional build system, package manager, or test runner. All development, testing, and deployment occurs through **Power Apps Studio** (the web-based maker portal at make.powerapps.com).

## Working with the .msapp File

The `Americhem_Portfolio_Tracker_v1.msapp` file can be unzipped to inspect or modify its contents directly. The internal structure:

- `Src/*.pa.yaml` — Screen and component logic in Power Fx YAML format
- `Controls/*.json` — Control definitions with properties
- `References/DataSources.json` — SharePoint connection and full list schema
- `References/Themes.json` — Americhem brand theme (primary blue: `#00509E` / RGB(56,96,178))
- `Properties.json` — App-level metadata (portrait phone layout, 640×1136)
- `AppCheckerResult.sarif` — Static analysis output from Power Apps AppChecker

To deploy changes: re-zip the contents back into `.msapp` format and import into Power Apps Studio, or edit directly in the studio.

## Architecture

**Pattern**: Master-detail with wizard-style navigation

**Data backend**: SharePoint Online list "Project Portfolio Tracker" at `https://americhemo365.sharepoint.com/sites/PMO`

**Screen flow**:
```
HomeScreen → BrowseScreen1 (list/search/sort)
           → EditScreen1 (new project wizard, ~107 KB)
BrowseScreen1 → DetailScreen1 → DetailScreen2 (read-only detail views)
             → StatusUpdateScreen_2 (RAG status / update)
             → scr_Intake (data intake form, ~99 KB)
```

**Global variables** (initialized in `App.pa.yaml` OnStart):
- `gblUser` — current user object
- `gblThemeColor` — Americhem brand blue (`#00509E`)
- `locWizardStep` — tracks step in multi-step wizard (starts at 1)

## Key Data Model Fields

The SharePoint list tracks projects with fields including: Title, Department, BU, SBG, Site(s), Executive Sponsor, Project Stakeholders, Category, Pillar, Priority, Stage, RAG Status, Start/Target Date, KPI Metric/Target/Actual, Cost Est. (USD), Estimation Confidence (default: Medium +/- 25%), Mandatory (bool), Recurring (bool), Current Blocker, Additional Details.

## Editing Source Files

When editing `.pa.yaml` files directly, use Power Fx syntax. Key conventions observed in this app:
- Navigation uses `Navigate(ScreenName)`
- Form resets use `ResetForm(FormName)` and `NewForm(FormName)`
- Data refresh uses `Refresh(DataSourceName)`
- Controls reference the SharePoint data source as `'Project Portfolio Tracker'`
