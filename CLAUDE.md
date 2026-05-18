# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running Locally

No build step or package manager. Serve with any static file server:

```bash
python -m http.server 8000
# Open http://localhost:8000
```

After changes, manually verify: data loading/error states, plot interactions (filters, tabs, scrollytelling), query panel request/response, and mobile layout breakpoints.

## Architecture

The entire application lives in a single monolithic file: [index.html](index.html) (~3,100 lines of HTML + embedded CSS + vanilla JS).

External dependency: **p5.js v1.9.3** via CDN (no local install).  
Backend: **Cloudflare Workers** at `WORKER_URL = 'https://nash-api.cvelazquezm.workers.dev'`  
- `GET /data` — returns visualization dataset (keywords, months, dots)  
- `POST /query` — RAG semantic search (`{ query: string }` → `{ answer, sources }`)

## Page Structure

The page has two distinct zones separated by a tab bar:

### 1. Narrative / Scrollytelling zone (before the tab bar)
A sequence of full-viewport sections, each with a canvas background animating colored dots:

- **`#welcome`** (300vh) — sticky panel with 3 sub-panels (`#wp-0`, `#wp-1`, `#wp-2`) driven by scroll position
- **6 analysis sections** (`#analisis-desaparecidos`, `#analisis-politica-exterior`, `#analisis-actores-politicos`, `#analisis-seguridad-publica`, `#analisis-economia`, `#analisis-salud`) — each 200vh with a sticky canvas. Each follows the same pattern:
  - `#xx-sticky` wrapper (sticky, 100vh) containing canvas + vg overlay + body
  - 2 panels (`.xx-panel`) switched by a scroll IIFE
  - Sankey diagram embedded as `<iframe src="Sankeys/sankey_*.html">` in the first panel
  - A second IIFE drives the canvas animation (dots colored by category, highlights the section's category)
- **`#conclusiones`** — standard fade-in section with canvas showing all dots at uniform opacity
- **`#dot-note`** — fixed label (bottom-left), visible while scrolling past wp-1 and through the analysis sections, hidden once the tab bar is reached

### 2. Interactive explorer zone (after the tab bar)
Two tabs:
- **Explora los datos** — p5.js scatter plot (`#chart-area`) with toolbar filters, keyword legend, and scroll-driven chapter sections
- **Haz una consulta** — RAG query input + answer display

## Code Organization (inside index.html)

Sequential `<script>` blocks follow the HTML sections:

1. **`WORKER_URL` + `SUBJECT_COLORS`** — declared in `<head>`, available to all subsequent scripts
2. **Per-section IIFEs** — each analysis section has 2 scripts immediately after its `</section>`: one scroll-panel switcher and one canvas animation
3. **`KEYWORD_COLORS`** + welcome canvas IIFE — declared after the last analysis section
4. **Main app scripts** (after tab bar): p5.js sketch, `loadData`/`processData`/`initViz`, tab switcher, RAG panel

`window._appData` is set once by the welcome canvas fetch and reused by all analysis section canvases via `(window._appData || fetch(...))`.

## Key Data Structures

**Dot** (each data point):
```js
{ id, ki, mi, dateObj, rx, ry, score, subject, frase, fecha, storyIdx,
  cx, cy,       // current animated position
  tx, ty,       // target position (recomputed on mode/filter change)
  alpha, targetAlpha }
```

**Section** (`SECS` / `SECS_TIME` arrays drive scrollytelling):
```js
{ type: 'kw'|'mo'|'pause'|'beat', kw, mo, tag, title, body, ... }
```

## Color System

Dot color is determined by the **keyword** (`KW[d.ki]`) via `KEYWORD_COLORS`. `keywordRgb(kw)` returns the RGB object used by the p5 renderer; keywords not in the map fall back to `#888888`. The legend also uses `KEYWORD_COLORS` for dot and label colors. `PALETTE` / `CL(i)` is kept for chapter heading accents in the scrollytelling sections.

Keyword → category → color mapping:
| Categoría | Color | Keywords |
|---|---|---|
| Desapariciones | `#ffffff` | desaparecidos, madres buscadoras, personas desaparecidas, comisión nacional de búsqueda, fosas |
| Seguridad pública | `#f34648` | homicidios, violencia, narcotráfico, huachicol, crimen organizado, fentanilo |
| Política exterior | `#4874e0` | estados unidos, trump, migrantes, aranceles, soberanía |
| Actores políticos | `#f4b723` | lópez obrador, morena, pan, pri, oposición, cuarta transformación, corrupción |
| Economía | `#00ab8c` | inflación, salario mínimo, inversiones |
| Salud | `#b06df5` | medicamentos, imss bienestar, vacunas, insumos médicos, personal de salud |

> Las keys en `KEYWORD_COLORS` deben estar en **minúsculas** porque así las envía Qdrant.

The app is **dark-only**. Background is pure black (`#000000`).

## Typography

- **Kameron 600** — titles (`#welcome-h1`, `#xx-title`), labels, toolbar elements, legend items (`.li`, `.lt`), tags (`.wp0-tag`)
- **Noto Sans 300** — body text (`.scroll-text`, `#welcome-p`, `#wp0-note`, analysis `#xx-text p`)
- Both loaded via Google Fonts `@import` at the top of `<style>`

## Sankeys

[Sankeys/](Sankeys/) contains 6 standalone HTML files (one per keyword category). Each has `background: transparent` and is embedded via `<iframe>` inside the first panel of its corresponding analysis section.

## Worker (Cloudflare Workers)

Source lives in [worker/src/index.js](worker/src/index.js), deployed via Wrangler (`worker/wrangler.toml`).

**Endpoints:**
- `GET /data` — scrolls all points from Qdrant, builds and returns the visualization dataset (1-hour in-memory cache)
- `POST /query` — RAG pipeline: embed query with `gemini-embedding-001` → vector search in Qdrant (`SDN_Challenge_Desaparecidos` collection, top 15) → generate answer with `gemini-2.5-flash`
- `GET /debug` — returns keyword/subject counts and field-validation stats (useful after re-ingestion)

**Required Wrangler secrets:** `QDRANT_URL`, `QDRANT_API_KEY`, `GEMINI_API_KEY`

```bash
cd worker
npx wrangler deploy
npx wrangler secret put QDRANT_URL
```

## Code Style

- **Single-file structure** — preserve unless explicitly asked to refactor
- **Naming conventions** — uppercase constants, grouped DOM refs, Spanish UI text
- **CSS** — reuse existing custom properties (`var(--...)`), no hardcoded colors
- **Edits** — small and surgical to minimize regressions in this large file
- **Dependencies** — do not add new external libraries; do not change `WORKER_URL` or API payload format unless explicitly asked
