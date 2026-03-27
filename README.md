# OpenPAVE Generate-PDF Skill

Generate branded PDF proposals and documents from structured JSON content. Two professional themes powered by Puppeteer, with C&R branding built in.

## Themes

### Dark Theme (PAVE Cobalt)
- Cobalt dark background (`#0a0f1e`), lime-green accents (`#c8ff00`)
- Fixed-page layout with explicit page divs (A4, 297mm)
- Embedded C&R + PAVE logos on cover and page headers
- Client logo required on cover
- Best for: PAVE proposals, client-facing sales documents

### Light Theme (C&R Professional)
- White background with C&R brand colors: **Blue #2459BB**, **Light Blue #6A9EF5**, **Neon Yellow #E4FE54**
- Outfit + Space Mono fonts (matching CnR Credentials deck)
- Flowing layout with Puppeteer's native `@page` margins
- Repeating header with dual logos (C&R + client) and blue bottom border
- Footer with "Commercial in Confidence" center text
- Best for: SOWs, proposals, product vision docs, specifications

## What's New in v2.2.0

- **Renamed** from `pdf` to `generate-pdf` for clarity
- **Client logo is mandatory** -- Skill refuses to generate without a client logo. Exits with a clear error explaining how to provide one via `--logo2`, `--client-logo`, or `logos.logo2` in JSON.
- **White logo detection** -- Light theme detects white/light logos on transparent backgrounds (invisible on white pages) and rejects them with a helpful error asking for a dark-colored version.
- **Sandbox-safe logo loading** -- `getBundledCnrLogo()` checks file existence directly (sandbox `fs.existsSync` returns false for directories).
- **Base64 fallback chain** -- `cat|base64` -> `base64 -i` -> `fs.readFileSync` for maximum sandbox compatibility.
- **Embedded header/footer** -- Light theme embeds header/footer in page HTML instead of using Puppeteer's `displayHeaderFooter` (which silently drops data URI images).

## What's New in v2.0.0

- **C&R Branding Colors** -- Light theme redesigned with brand palette: Blue #2459BB, Light Blue #6A9EF5, White, Neon Yellow accent #E4FE54
- **Bundled C&R Logo** -- `assets/cnr-logo-black.png` and `cnr-logo-white.png` ship with the skill. Auto-detected when no `--logo1` / `--cnr-logo` is provided.
- **Professional Header** -- Dual logo header (C&R | Client) with blue bottom border and document title
- **Commercial in Confidence Footer** -- Three-column footer: copyright, "Commercial in Confidence", page numbers
- **Outfit Font** -- Matching CnR Credentials deck typography

## Requirements

- Node.js >= 16
- Puppeteer (`npm install -g puppeteer` or available via npx)
- Python 3 + Pillow (for white logo detection on light theme)

## Installation

```bash
pave install generate-pdf
```

## Usage

```bash
# Dark theme
pave run generate-pdf dark sample -o content.json
pave run generate-pdf dark generate -i content.json -o proposal.pdf --client-logo client.png --open

# Light theme (C&R branded)
pave run generate-pdf light sample -o content.json
pave run generate-pdf light generate -i content.json -o doc.pdf --logo2 client-logo.png --open
pave run generate-pdf light generate -i content.json -o doc.pdf --logo2 towngas.png --accent "#2459BB" --open

# HTML preview (no Puppeteer needed)
pave run generate-pdf light preview -i content.json --logo2 client.png
pave run generate-pdf dark preview -i content.json --client-logo client.png
```

### Logo Options

| Theme | C&R Logo | Client Logo |
|-------|----------|-------------|
| Dark  | `--cnr-logo` (auto: bundled white) | `--client-logo` (mandatory) |
| Light | `--logo1` (auto: bundled black) | `--logo2` or `--client-logo` (mandatory) |

The C&R logo is bundled in `assets/` and auto-detected. Client logos must be provided by the user -- the skill will not generate without one.

**Important:** For the light theme, provide a dark-colored logo. White/light logos on transparent backgrounds will be detected and rejected (invisible on white pages).

## Content JSON Schema

### Dark Theme

```json
{
  "entity": "C&R Wise AI Limited",
  "logos": {
    "cnr": null,
    "pave": "/path/to/pave-logo.png",
    "client": "/path/to/client-logo.png"
  },
  "cover": {
    "title": "<span class='accent'>PAVE</span> AI Enablement Proposal",
    "subtitle": "Prepared for Acme Corp",
    "clientName": "Acme Corp",
    "preparedBy": "C&R WISE AI LIMITED",
    "date": "MARCH 2026",
    "version": "1.0",
    "badge": "Confidential",
    "headerLabel": "Acme Corp - PAVE AI Enablement"
  },
  "pages": [
    {
      "blocks": [
        { "type": "section", "label": "Section 01" },
        { "type": "h1", "text": "Executive Summary" },
        { "type": "p", "text": "Supports **bold**, *italic*, and `code`." }
      ]
    }
  ]
}
```

### Light Theme

```json
{
  "accent": "#2459BB",
  "logos": {
    "logo1": null,
    "logo2": "/path/to/client-logo.png"
  },
  "header": {
    "title": "Project Name",
    "subtitle": "Scope of Work"
  },
  "footer": {
    "left": "(c) 2026 C&R Wise AI Limited",
    "center": "Commercial in Confidence"
  },
  "blocks": [
    { "type": "title", "text": "Project Name" },
    { "type": "subtitle", "text": "Scope of Work" },
    { "type": "h1", "text": "1. Executive Summary" },
    { "type": "p", "text": "Body text with **bold** and *italic*." },
    { "type": "table", "columns": ["A", "B"], "rows": [{ "cells": ["1", "2"] }] },
    { "type": "callout", "variant": "info", "lines": ["Note text"] },
    { "type": "page-break" }
  ]
}
```

## Block Types

| Type | Dark | Light | Description |
|------|------|-------|-------------|
| `section` | Yes | - | Section label with accent line |
| `title` | - | Yes | Document title (centered) |
| `subtitle` | - | Yes | Document subtitle (centered) |
| `h1`-`h4` | Yes | Yes | Headings |
| `p` | Yes | Yes | Paragraph (inline markdown) |
| `hr` | Yes | Yes | Horizontal rule |
| `table` | Yes | Yes | Table with columns, rows, widths |
| `ul` / `ol` | Yes | Yes | Lists (light supports nesting) |
| `blockquote` | Yes | Yes | Styled quote block |
| `callout` | Yes | Yes | Callout box (dark: accent; light: info/warning/success/accent) |
| `kv` | Yes | Yes | Key-value pairs |
| `two-col` | Yes | Yes | Two-column layout with sub-blocks |
| `code` | - | Yes | Code block |
| `tree` | - | Yes | File tree (preformatted) |
| `page-break` | - | Yes | Force page break |

## Bundled Assets

```
assets/
  cnr-logo-black.png   # C&R logo for light backgrounds
  cnr-logo-white.png   # C&R logo for dark backgrounds
```

## License

MIT - C&R Wise AI Limited
