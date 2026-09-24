# Carbonique field notes

A single-page reference for the eddy covariance towers of the Carbonique project
(carbon dynamics in southern Quebec wetlands). It gathers every field visit, the
known data caveats and a description of each site and its instruments in one
self-contained HTML file.

**Live page:** https://davidulicio.github.io/carbonique_fieldnotes/

The page has no dependencies: open `index.html` in any browser, online or offline.
It has a light and a dark mode and works on a phone.

## What the page shows

The page has two views, switched with the tabs under the header.

### Maintenance log

- Summary tiles: number of field notes, routine visits, major work, faults and
  data caveats.
- A visit timeline per tower. Hover a marker to see the visit, click it to jump to
  the note.
- Filters (Everything, Routine, Major work, Faults, Data caveats only) and a
  search box that covers notes and caveats.
- One section per tower with every field note. Serial numbers, firmware versions
  and part sizes are kept exactly as recorded.
- Data caveats for the people processing the flux data: what happened, the
  affected period, how to handle it, and whether the field notes confirm it.
  Some come with diagnostic plots (click to enlarge).
- Recommended TraceAnalysis ini files that carry the fixes for those caveats.

### Sites & towers

- **All sites at a glance:** AmeriFlux ID, EC (sonic) height, air temperature and
  RH heights, the TA/RH level closest to the EC height (the one to use for
  gap-filling), north offset, canopy height and start date.
- **One card per site:** description, photos, a mast diagram with the height of
  every sensor (canopy shaded when it is a fixed value), site information, one box
  per instrument with its model, variables and height or depth, and the
  processing settings.
- **Setup common to all sites:** variable naming, time base, data acquisition,
  SmartFlux real-time processing and where the files live.

Direct links: `#sites` opens the Sites & towers view and `#site-UQAM_3` jumps to
one site, for example
https://davidulicio.github.io/carbonique_fieldnotes/#site-UQAM_3

## The towers

| Code | Site | AmeriFlux ID | Location |
|---|---|---|---|
| UQAM_1 | Lac Saint-Pierre Disturbed Marsh | CA-CQ1 | Baie-du-Febvre |
| UQAM_2 | Natural Lac-à-la-Tortue Open bog | CA-CQ2 | Notre-Dame-du-Mont-Carmel |
| UQAM_3 | Carbonique Lac-à-la-Tortue Forested bog | CA-CQ3 | Notre-Dame-du-Mont-Carmel |
| UQAM_4 | Saint-Remi Disturbed Forested Peatland | CA-CQ4 | Saint-Rémi |
| UQAM_5 | Saint-François du Lac Natural Marsh | CA-CQ5 | Saint-François-du-Lac |
| MCGILL_1 | Lac Saint-Pierre Restored Marsh | CA-CQ9 | Baie-du-Febvre |

## Where the content comes from

| Source | Used for |
|---|---|
| EcoFlux Inventory field notes (grandwazoo.ddns.net/ecoflux, login required) | Every visit in the maintenance log |
| Data-pipeline notes (Word document: inconsistencies, issues and sensor changes to tackle on the data pipeline) | Data caveats and their plots |
| *Info on Carbonique EC sites* (Word document) | The whole Sites & towers view, including the photos |
| SmartFlux metadata (the `.metadata` file inside each `.ghg`) | Sonic heights, north offsets, sensor separations and canopy heights written in the Word document |

## Visit types

Each field note is classified as one of three types:

| Type | Meaning |
|---|---|
| Routine | Scheduled upkeep: cleaning, data download, USB swaps, washer fluid and methanol refills, guy wires, drone and vegetation surveys. |
| Major work | Planned significant work: installation, commissioning, upgrades, seasonal preparation, structural changes, tower certification, new sensors. |
| Fault | Any visit driven by a fault or an unplanned condition. A visit that mixes routine tasks with a fault counts as a fault, and its description starts with `PROBLEM:`. |

## How the page is updated

The page is rebuilt about once a month with Claude, using the `ecoflux-report`
skill:

1. Log in to the EcoFlux Inventory in Chrome, then ask Claude to refresh the
   EcoFlux report.
2. Claude starts from the `index.html` in this repository, reads only the new or
   changed field notes, classifies them and rebuilds the page. The caveats, their
   plots and the site information are carried forward from the current page.
3. If there is a new edition of the data-pipeline notes or of *Info on Carbonique
   EC sites*, attach it to the request and it is folded in.
4. Upload the new `index.html` to this repository. GitHub Pages publishes it
   within a few minutes.

### Updating the site information

Edit *Info on Carbonique EC sites* in Word and give it to Claude at the next
refresh. The Sites & towers view is built from the document's structure, so keep
it:

- **Heading 1:** one per site, written as
  `UQAM_1 Lac Saint-Pierre, Quebec (46.1645N, 72.6851W)`. A Heading 1 without
  coordinates (e.g. *Setup common to all sites*) becomes a network-wide section.
- **Normal text right under it:** the site description.
- **Pictures:** anywhere under the site heading, they become the site photos.
- **Heading 2:** the sections (*Site info*, *Instruments info*, *Processing
  settings*, or any new one).
- **Heading 3:** one instrument.
- **Bullets:** `Label: value`.

The mast diagram and the at-a-glance table read these labels, so keep them as
they are: `Sonic height`, the `TA_1_1_1, RH_1_1_1 (...)` lines, `Height` (wind
monitor, net radiometer, snow depth sensor), `Intake heights` (profile system),
`North offset`, `AmeriFlux ID`, `Station active since` and `Canopy height set in
SmartFlux`. A height written as text ("To be checked") is listed as not recorded
yet.

## Conventions

- **Variable names** follow the AmeriFlux `VAR_H_V_R` convention (horizontal
  position, vertical level, replicate). For air sensors level 1 is the highest, so
  `TA_1_1_1` is always the upper HygroVUE10. For soil, `_1_1_1` to `_1_4_1` are the
  profile from shallow to deep and `_2_1_1` to `_5_1_1` are horizontal replicas.
- **Time:** all loggers and SmartFlux units record in EST (UTC-5) all year, with
  no daylight saving time.
- **Heights** are above the ground unless the site card says otherwise.

## Using the data in the page

Everything the page displays is embedded in `index.html` as JavaScript constants,
so it can be pulled out without the EcoFlux login:

| Constant | Content |
|---|---|
| `NOTES` | Classified field notes: `id`, site `s`, date `d`, author `w`, type `t`, title `ti`, description `b` |
| `PIPE` | Data caveats: site, edition, affected period, description, how to handle it, cross-checks |
| `PLOTS` | Caveat plots as base64 images |
| `SITEINFO` | The parsed *Info on Carbonique EC sites* document, photos included |

For example, in Python:

```python
import json, re
html = open("index.html", encoding="utf-8").read()
notes = json.loads(re.search(r"const NOTES = (\[.*?\]);\s*const PIPE", html, re.S).group(1))
sites = json.loads(re.search(r"const SITEINFO = (\{.*?\});\n", html, re.S).group(1))
faults = [n for n in notes if n["s"] == "UQAM_3" and n["t"] == "issue"]
```

## Repository contents

| File | Description |
|---|---|
| `index.html` | The page, fully self-contained (styles, scripts, data, plots and photos) |
| `README.md` | This file |

## Maintainer

David Trejo, Research Support Professional, UQAM (Carbonique project).
