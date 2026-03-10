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
| Tender Comparison | Live (v3) | Upload zones, AI spec parsing with merge confirmation, AI tender parsing, comparison matrix with estimate ranges, status indicators |
| Milestones | Live | Payment schedule by contract %, status toggles (pending/partial/paid/overdue) |
| Owner Supplied | Live (v3) | Multi-option with tier tags (Budget/Mid/Premium), add real quotes, add new items, ordered/delivered toggles |
| AI Integration | Live | Spec parsing with location-based estimates, tender parsing, partial merge with confirmation modal, AI Estimate tender auto-generation |
| Scenarios | Live | What-if modelling, toggle items per scenario, budget impact calculation |

---

## Data Model

All state is held in a single JSON object saved to localStorage. Key entities:

- **specItems** — Line items from the spec document (id, category, item, unit, qty, specNotes, status, eLow, eMid, eHigh)
- **tenders** — Builder submissions with status (sample/estimated/confirmed), per-item pricing, included flag, and notes
- **milestones** — Payment stages with % of contract, due date, status, paid amount
- **ownerSupplied** — Fixtures/fittings with options[] array (each option has supplier, description, price, active, tier), ordered/delivered toggles
- **scenarios** — Named sets of included/excluded spec items and owner items for what-if modelling
- **specDocuments** — Uploaded spec document files (base64 data URLs)
- **tenderDocuments** — Uploaded tender files with builderName label
- **location** — User's area for location-based cost estimates (e.g. "Surrey")

---

## AI Integration (Live)

The app uses the Anthropic API (client-side fetch to `api.anthropic.com/v1/messages` using `claude-sonnet-4-20250514` with `anthropic-dangerous-direct-browser-access` header) to:

1. **Parse spec documents** — User uploads PDF/images. AI extracts all line items with location-based eLow/eMid/eHigh estimates, splits into builder-scope and owner-supplied. Results shown in a merge confirmation modal where user selects which items to add/overwrite/skip.
2. **Parse tender responses** — User uploads each builder's tender. AI maps items to spec IDs, extracts pricing, flags deviations and omissions.
3. **Auto-generate AI Estimate tender** — After spec parse, an "AI Estimate" tender is created from mid-range values so the user can compare against real quotes.
4. **Location-aware pricing** — Estimates adjusted for local labour and material costs based on user's configured area.

This runs entirely client-side. No backend needed. User provides their own Anthropic API key via the Settings modal (stored in localStorage under `reno-tracker-ak`).

---

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| Single HTML file | Zero build complexity, instant Vercel deploy, easy to maintain |
| CDN dependencies | No node_modules, no version drift in lockfiles |
| localStorage | Free, zero-infra, works offline. Trade-off: no cross-device sync |
| Sample data pre-loaded | User can explore immediately, replace with real data later |
| GBP currency | UK-based user |
| Client-side Anthropic API | No backend to manage, runs in browser. User supplies API key via Settings modal. |
| Partial merge on parse | AI parse shows confirmation modal — user picks items to add/overwrite/skip instead of destructive replace |
| Location-based estimates | AI cost estimates tuned to user's area for more accurate budgeting |
| Tier-based owner options | Budget/Mid/Premium tiers on owner-supplied items for quick cost comparison |

---

## Current State

- **Version:** 1.3.0
- **Last updated:** 2026-03-10
- **Sample data:** 18 spec items (with eLow/eMid/eHigh estimates), 3 sample tenders, 9 milestones, 9 owner-supplied items (with Budget/Mid/Premium tiers), 4 scenarios
- **AI features:** Spec parsing with merge confirmation, tender parsing, location-based estimates, AI Estimate tender auto-generation
- **Owner supplied:** Multi-option with tier tags, add real quotes, add new items UI

---

## Next Steps (Priority Order)

1. ~~FEAT-011: Upload zones~~ — Done (v1.1.0)
2. ~~FEAT-014: Owner Supplied v2 — multi-option~~ — Done (v1.1.0)
3. ~~FEAT-010: Add new owner-supplied items~~ — Done (v1.1.0)
4. FEAT-012: AI parsing of uploaded spec doc → auto-populate specItems and ownerSupplied
5. FEAT-013: AI parsing of uploaded tenders → auto-populate tender comparison matrix with flags
6. FEAT-015: Owner Supplied — auto-populate from spec doc via AI
7. FEAT-004/005: Add/edit/delete spec items and tenders in UI
8. FEAT-009: Mobile responsive polish
