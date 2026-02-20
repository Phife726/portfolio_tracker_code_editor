# DetailScreen2 Redesign — Option A: Hero Banner + Stat Row

**Date:** 2026-02-20
**Audience:** Executives / Portfolio Managers skimming for Red/Amber projects
**Goal:** Instant RAG status recognition, key metrics without scrolling, clean mobile layout

---

## Layout (640 × 1136 px phone)

| Section | Y | H | Description |
|---------|---|---|-------------|
| Header bar | 0 | 88 | Blue bar, back arrow, title, quick-status icon |
| RAG Hero Banner | 88 | 100 | Full-width colored block (Red/Amber/Green) + project title |
| Stage / Cost Row | 188 | 80 | Two stat tiles on gray background |
| KPI Section | 268 | 132 | Metric name + Target vs Actual tiles |
| Org Row | 400 | 60 | Dept · BU · Site compact |
| Timeline Row | 460 | 60 | Start → Target dates |
| Blocker Section | 520 | 160 | Conditional red tint if blocker populated |
| Details Section | 680 | 180 | Additional Details text |
| Footer / Actions | 860 | 88 | Last Updated + Delete/Edit icons |

Total content height: 948px (fits in 1048px below header)

---

## Data Binding

All fields reference `BrowseGallery1.Selected.FieldName` (no form control):

| Field | Power Fx expression |
|-------|---------------------|
| Title | `BrowseGallery1.Selected.Title` |
| RAG Status | `BrowseGallery1.Selected.'RAG Status'.Value` |
| Stage | `BrowseGallery1.Selected.Stage.Value` |
| Cost Est. | `Text(BrowseGallery1.Selected.'Cost Est. (USD)', "$#,##0")` |
| KPI Metric | `BrowseGallery1.Selected.'KPI Metric'` |
| KPI Target | `Text(BrowseGallery1.Selected.'KPI Target Value', "#,##0")` |
| KPI Actual | `Text(BrowseGallery1.Selected.'KPI Actual Value', "#,##0")` |
| Department | `BrowseGallery1.Selected.Department` |
| BU | `BrowseGallery1.Selected.BU` |
| Site(s) | `BrowseGallery1.Selected.'Site(s)'` |
| Start Date | `Text(BrowseGallery1.Selected.'Start Date', "mmm d, yyyy")` |
| Target Date | `Text(BrowseGallery1.Selected.'Target Date', "mmm d, yyyy")` |
| Current Blocker | `BrowseGallery1.Selected.'Current Blocker'` |
| Additional Details | `BrowseGallery1.Selected.'Additional Details'` |
| Modified | `Text(BrowseGallery1.Selected.Modified, "mmm d, yyyy")` |

---

## Visual Decisions

- **RAG banner color:** `If(rag="Red", RGBA(196,43,28,1), rag="Amber", RGBA(210,130,0,1), RGBA(0,145,65,1))`
- **RAG badge text:** `"● CRITICAL"` / `"● AT RISK"` / `"● ON TRACK"`
- **Blocker background:** red tint (`RGBA(255,240,240,1)`) when blocker text populated, gray otherwise
- **Delete icon:** red (`RGBA(196,43,28,1)`) to signal destructive action
- **Edit icon:** dark blue (`RGBA(0,18,107,1)`)
- **Section headers:** all-caps, 10-11pt, gray (`RGBA(100,100,100,1)`)

---

## Implementation

- Edit `Controls/64.json`: remove `DetailForm1_1` (uid=69), add ~32 new controls (UIDs 1000–1032)
- Update existing: `IconDelete1_1` and `full_edit_mode_icon` Y positions and colors
- Repack `.msapp` using explicit file list
