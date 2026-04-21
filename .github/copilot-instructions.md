# Project Guidelines

## Architecture
- This workspace is a static, single-page web app implemented in HTML, CSS, and vanilla JavaScript.
- The main app lives in `index.html` and a mirrored copy exists in `SND_Challenge/index.html`.
- The app has two primary UI flows:
  - Exploratory visualization (p5.js scatter/sections/scrollytelling).
  - Query panel (RAG-style API requests).
- Backend data is fetched from `WORKER_URL` in the page script (`/data` and `/query` endpoints).

## Build and Test
- No package manager, build step, or automated test suite is configured.
- Run locally with any static file server (for example: `python -m http.server 8000`) and open the page in a browser.
- After changes, manually verify:
  - Initial data loading and loading/error states.
  - Plot and section interactions (filters, tabs, and scroll-driven behavior).
  - Query panel request/response behavior.
  - Theme toggle and mobile layout breakpoints.

## Code Style
- Preserve the existing single-file structure unless a task explicitly asks for refactoring.
- Keep existing naming and sectioning patterns (uppercase constants, grouped DOM refs, descriptive Spanish UI text).
- Reuse existing CSS custom properties and avoid introducing hardcoded colors when a variable already exists.
- Keep comments concise and only where logic is non-obvious.

## Conventions
- Treat `index.html` as the source of truth; when a change is meant for both copies, keep `SND_Challenge/index.html` in sync.
- Keep external dependencies minimal; current visualization dependency is p5.js via CDN.
- Do not change API base URL or payload/response handling format unless explicitly requested.
- Prefer small, surgical edits to avoid regressions in this large monolithic file.
