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

The entire application lives in a single monolithic file: [index.html](index.html) (~1,700 lines of HTML + embedded CSS + vanilla JS). A mirrored copy exists at [SND_Challenge/index.html](SND_Challenge/index.html) — **treat `index.html` as source of truth** and keep both in sync when changes apply to both.

External dependency: **p5.js v1.9.3** via CDN (no local install).  
Backend: **Cloudflare Workers** at `WORKER_URL = 'https://nash-api.ogaldamez.workers.dev'`  
- `GET /data` — returns visualization dataset (keywords, months, dots)  
- `POST /query` — RAG semantic search (`{ query: string }` → `{ answer, sources }`)

## Code Organization (inside index.html)

The JS is organized into sequential sections:

1. **Constants & config** — `WORKER_URL`, `PALETTE`, `KEYWORD_COLORS`, `MES_ES`, `STORY_BEATS`, `PAUSE_KW`/`PAUSE_TIME`
2. **Data loading** (`loadData`, `normalizeOldFormat`) — fetches `/data`, converts raw array to structured object
3. **Data processing** (`processData`) — enriches normalized data, builds `SECS`/`SECS_TIME` section structures, matches story beats to dots
4. **Viz initialization** (`initViz`) — main controller managing two navigation modes (keyword `kw` vs temporal `time`) and two view modes (month scatter vs day-stack), section rendering, IntersectionObserver scrollytelling, and all filter/highlight logic
5. **p5.js sketch** — renders dots with lerp animation, axes, tooltips, hover/click-to-modal, decorative particles
6. **Query/RAG panel** — POST to `/query`, displays answer + collapsible sources, handles loading/error states

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
| Salud | `#5f2a91` | medicamentos, imss bienestar, vacunas, insumos médicos, personal de salud |

> Las keys en `KEYWORD_COLORS` deben estar en **minúsculas** porque así las envía Qdrant. Verificar con el botón "⚙ Colores" si se agregan nuevas keywords.

The app is **dark-only** — the light theme and `#theme-toggle` button have been removed. Background is pure black (`#000000`).

## Code Style

- **Single-file structure** — preserve unless explicitly asked to refactor
- **Naming conventions** — uppercase constants, grouped DOM refs, Spanish UI text
- **CSS** — reuse existing custom properties (`var(--...)`), no hardcoded colors
- **Comments** — concise, only where logic is non-obvious
- **Edits** — small and surgical to minimize regressions in this large file
- **Dependencies** — do not add new external libraries; do not change `WORKER_URL` or API payload format unless explicitly asked
