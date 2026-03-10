# Reno Tracker — Project State

*Last updated: 10 March 2026*

---

## Project Overview

**App:** Renovation Budget & Project Tracker
**Type:** Static single-page web app (no build step)
**Stack:** HTML, React 18 (CDN), Tailwind CSS (CDN), Babel standalone
**Hosting:** Vercel (static deployment via GitHub auto-deploy)
**Repo:** github.com/raymondorae/reno-tracker
**Data persistence:** Browser localStorage (key: `reno-tracker-v2`)
**Currency:** GBP (UK-based user)

---

## Architecture

```
reno-tracker/
├── index.html          ← Entire app (React components + data + styles)
├── vercel.json          ← Vercel static config (no build command)
├── README.md            ← Deployment instructions
├── CLAUDE_CODE_PROMPT.md ← Session prompt for Claude Code
├── PROJECT_STATE.md     ← This file
└── CHANGELOG.md         ← Bug & feature log
```

Single-file architecture — all React components, default data, and styling live inside `index.html`. No bundler, no node_modules, no build step. React and Tailwind are loaded from CDN at runtime.

---

## Modules

| Module | Status | Description |
|--------|--------|-------------|
| Dashboard | Live | Budget overview, utilisation bar, tender selector, issue summary, milestone overview |
| Tender Comparison | Live (v3) | Multi-file upload zones for spec doc + builder tenders (persistent base64 storage), AI-powered parsing, spec-to-tender matrix, deviation/omission flags |
| Milestones | Live | Payment schedule by contract %, status toggles (pending/partial/paid/overdue) |
| Owner Supplied | Live (v1) | Fixtures & fittings budget vs actual, ordered/delivered toggles |
| Owner Supplied | Planned (v2) | Multiple options per item with selection, add new items via UI |
| AI Integration | Live | API key management, spec doc parsing, tender parsing via Anthropic API (claude-sonnet-4-20250514) |
| Scenarios | Live | What-if modelling, toggle items per scenario, budget impact calculation |

---

## Data Model

All state is held in a single JSON object saved to localStorage. Key entities:

- **specItems** — Line items from the spec document (id, category, item, unit, qty, specNotes)
- **tenders** — Builder submissions, each containing per-item pricing, included flag, and notes
- **milestones** — Payment stages with % of contract, due date, status, paid amount
- **ownerSupplied** — Fixtures/fittings the owner sources (budget, actual, ordered, delivered)
- **scenarios** — Named sets of included/excluded spec items and owner items for what-if modelling

- **specDocuments** — Uploaded spec document files stored as base64 (id, filename, dateUploaded, base64, mediaType)
- **tenderDocuments** — Uploaded tender document files stored as base64 (id, builderName, filename, dateUploaded, base64, mediaType), grouped by builder name for display and AI parsing

### Planned data model changes (v2)

**ownerSupplied.options[]** — Each owner-supplied item gains an `options` array. Each option contains: supplier, description, price, and an `active` boolean. Only the active option feeds into budget calculations.

---

## AI Integration (Live)

The app uses the Anthropic API (client-side fetch to `api.anthropic.com/v1/messages` using `claude-sonnet-4-20250514` with `anthropic-dangerous-direct-browser-access` header for CORS) to:

1. **Parse spec documents** — User uploads a PDF/image of the spec. AI extracts all line items, splits them into builder-scope specItems and owner-supplied items, and auto-populates both sections. Categories are dynamically discovered from the document.
2. **Parse tender responses** — User uploads each builder's tender PDF/image. AI reads it, maps each quoted item to the corresponding spec line item by ID, identifies pricing, flags deviations from spec, and flags omissions. Multiple tenders can be uploaded and parsed.
3. **Cross-reference** — The comparison matrix auto-generates deviation and omission flags from the structured tender data.

This runs entirely client-side. No backend needed. User provides their own Anthropic API key via an in-app modal (stored in localStorage under `reno-tracker-api-key`).

**Key implementation details:**
- Documents are stored as base64 in localStorage within the main `reno-tracker-v2` data object (`specDocuments[]` and `tenderDocuments[]`)
- Multiple files can be uploaded per section (spec or tender), sent together in a single API call
- Tender documents are labelled with a builder name and grouped by builder for display and parsing
- `docToContentBlock` helper converts stored docs to API content blocks (PDF → `document` type, images → `image` type)
- AI returns structured JSON which is extracted via fence/brace detection
- Spec parsing clears existing tenders (since IDs change) and creates a single "Full spec" scenario
- Tender parsing makes one API call per builder group, adds new tenders without removing existing ones
- The `extractJSON` helper handles both fenced code blocks and raw JSON in responses

---

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| Single HTML file | Zero build complexity, instant Vercel deploy, easy to maintain |
| CDN dependencies | No node_modules, no version drift in lockfiles |
| localStorage | Free, zero-infra, works offline. Trade-off: no cross-device sync |
| Sample data pre-loaded | User can explore immediately, replace with real data later |
| GBP currency | UK-based user |
| Client-side Anthropic API | No backend to manage, runs in browser. User supplies API key. Uses `anthropic-dangerous-direct-browser-access` header for CORS. |
| API key in localStorage | Separate from app data (`reno-tracker-api-key`), never included in data export/reset |
| Spec parse clears tenders | When a new spec is parsed, existing tenders are cleared since spec item IDs change |
| Documents stored as base64 in localStorage | Persists across sessions, no server needed. Trade-off: increases localStorage usage |
| Tender docs grouped by builder | One API call per builder group enables accurate per-builder tender parsing |

---

## Current State

- **Version:** 1.2.0
- **Last updated:** 2026-03-10
- **Sample data:** 18 spec items, 3 tenders, 9 milestones, 9 owner-supplied items, 4 scenarios
- **AI features:** Spec doc parsing, tender parsing, auto-population of specItems + ownerSupplied + tenders
- **Document storage:** Spec docs and tender docs persisted as base64 in localStorage, multi-file support, grouped by builder for tenders

---

## Next Steps (Priority Order)

1. ~~FEAT-011: Upload zones~~ — Done (v1.1.0)
2. ~~FEAT-012: AI spec parsing~~ — Done (v1.1.0)
3. ~~FEAT-013: AI tender parsing~~ — Done (v1.1.0)
4. ~~FEAT-015: Owner-supplied from spec~~ — Done (v1.1.0)
5. FEAT-014: Owner Supplied v2 — multiple options per item with active selection toggle
6. FEAT-010: Add new owner-supplied items manually via UI
7. FEAT-004/005: Add/edit/delete spec items and tenders in UI
8. FEAT-009: Mobile responsive polish
