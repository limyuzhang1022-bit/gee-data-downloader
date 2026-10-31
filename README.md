# GEE Data Downloader

An Agent Skill for Codex and Claude Code that selects geospatial products for a requested study period, recognizes Google Earth Engine asset paths, and generates reproducible GEE JavaScript export code with product metadata, official URLs, and literature citations.

## Features

- Selects products from the study area, date range, temporal scale, variable, and spatial resolution.
- Verifies GEE dataset IDs, actual temporal coverage, bands, scale factors, units, and native resolution.
- Recognizes Cloud Assets and Legacy Assets paths from GEE Assets screenshots.
- Generates monthly, annual, and static GeoTIFF export code.
- Applies physically appropriate monthly sums, monthly means, categorical handling, and quality control.
- Provides product pages, data DOIs, method papers, and formatted citations.
- Supports selection workflows for MODIS, CHIRPS, ERA5-Land, GLEAM, CLCD, OpenLandMap, HWSD, and related products.

## Compatibility

| Runtime | Personal installation path | Direct invocation |
|---|---|---|
| Codex | `~/.codex/skills/gee-data-downloader/` | `$gee-data-downloader` |
| Claude Code | `~/.claude/skills/gee-data-downloader/` | `/gee-data-downloader` |

The shared `SKILL.md` and `references/` files follow the Agent Skills structure supported by both runtimes. `agents/openai.yaml` supplies Codex interface metadata, while Claude Code loads the core skill and references directly.

## Installation

### Codex

```bash
git clone https://github.com/limyuzhang1022-bit/gee-data-downloader.git "$HOME/.codex/skills/gee-data-downloader"
```

Invoke the skill explicitly or use a matching request:

```text
$gee-data-downloader Download monthly CHIRPS precipitation for the selected study area from 2002-01 through 2025-12, with one TIFF per month.
```

### Claude Code

Personal installation:

```bash
git clone https://github.com/limyuzhang1022-bit/gee-data-downloader.git "$HOME/.claude/skills/gee-data-downloader"
```

Project installation:

```bash
git clone https://github.com/limyuzhang1022-bit/gee-data-downloader.git ".claude/skills/gee-data-downloader"
```

Direct invocation example:

```text
/gee-data-downloader Download monthly ERA5-Land runoff for the selected study area from 2002-01 through 2025-12 and convert it to mm/month.
```

Claude Code can also load the skill automatically from its `description`. See the [official Claude Code Skills documentation](https://code.claude.com/docs/en/skills) for installation and invocation behavior.

## Example Study-Area Asset

```text
projects/underground-water-488604/assets/xj
```

Provide the intended study-area asset path when requesting code, or attach a GEE Assets screenshot for path recognition.

## Output

Each request produces:

1. A product-selection decision and temporal-coverage assessment.
2. The study-area asset ID and asset type.
3. Complete JavaScript for the GEE Code Editor.
4. Dataset ID, provider, variable, units, scale factor, resolution, aggregation, and quality-control information.
5. Official product URLs, data DOIs, and method papers.
6. Export task count, Google Drive folder, file naming, and required placeholders.

## Repository Structure

```text
gee-data-downloader/
|-- SKILL.md
|-- README.md
|-- agents/
|   `-- openai.yaml
`-- references/
    |-- dataset-guide.md
    `-- gee-code-patterns.md
```

## Data and Citation Policy

The skill prioritizes the [Google Earth Engine Data Catalog](https://developers.google.com/earth-engine/datasets), product-provider websites, data DOIs, and original method papers. It verifies the actual last available observation and reports both native resolution and export-grid scale.
