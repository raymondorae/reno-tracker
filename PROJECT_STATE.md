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
| Tender Comparison | Live (v1) | Spec-to-tender matrix, per-item pricing, deviation/omission flags, totals row |
| Tender Comparison | Planned (v2) | Upload zones for spec doc + builder tenders, AI-powered parsing into structured data |
| Milestones | Live | Payment schedule by contract %, status toggles (pending/partial/paid/overdue) |
| Owner Supplied | Live (v1) | Fixtures & fittings budget vs actual, ordered/delivered toggles |
| Owner Supplied | Planned (v2) | AI auto-populate from spec, multiple options per item with selection, add new items via UI |
| Scenarios | Live | What-if modelling, toggle items per scenario, budget impact calculation |

---

## Data Model

All state is held in a single JSON object saved to localStorage. Key entities:

- **specItems** — Line items from the spec document (id, category, item, unit, qty, specNotes)
- **tenders** — Builder submissions, each containing per-item pricing, included flag, and notes
- **milestones** — Payment stages with % of contract, due date, status, paid amount
- **ownerSupplied** — Fixtures/fittings the owner sources (budget, actual, ordered, delivered)
- **scenarios** — Named sets of included/excluded spec items and owner items for what-if modelling

### Planned data model changes (v2)

**ownerSupplied.options[]** — Each owner-supplied item gains an `options` array. Each option contains: supplier, description, price, and an `active` boolean. Only the active option feeds into budget calculations.

**specDocument** — Raw uploaded spec doc (stored as base64), used as the source of truth for AI parsing.

**tenderDocuments[]** — Raw uploaded tender responses from builders, each parsed by AI into structured tender data.

---

## AI Integration

The app uses the Anthropic API (client-side fetch to `api.anthropic.com/v1/messages` using `claude-sonnet-4-20250514`) to:

1. **Parse spec documents** — User uploads a PDF/image of the spec. AI extracts all line items, splits them into builder-scope specItems and owner-supplied items, and auto-populates both sections.
2. **Parse tender responses** — User uploads each builder's tender PDF/image. AI reads it, maps each quoted item to the corresponding spec line item, identifies pricing, flags deviations from spec, and flags omissions.
3. **Cross-reference** — AI compares each tender against the spec document and populates the comparison matrix with flags.

This runs entirely client-side. No backend needed. User provides their own Anthropic API key via an in-app settings area.

---

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| Single HTML file | Zero build complexity, instant Vercel deploy, easy to maintain |
| CDN dependencies | No node_modules, no version drift in lockfiles |
| localStorage | Free, zero-infra, works offline. Trade-off: no cross-device sync |
| Sample data pre-loaded | User can explore immediately, replace with real data later |
| GBP currency | UK-based user |
| Client-side Anthropic API | No backend to manage, runs in browser. User supplies API key. |

---

## Current State

- **Version:** 1.0.0
- **Last updated:** 2026-03-10
- **Sample data:** 18 spec items, 3 tenders, 9 milestones, 9 owner-supplied items, 4 scenarios

---

## Next Steps (Priority Order)

1. FEAT-011: Add document upload zones to Tender Comparison tab (spec doc area + builder tender area)
2. FEAT-012: AI parsing of uploaded spec doc → auto-populate specItems and ownerSupplied
3. FEAT-013: AI parsing of uploaded tenders → auto-populate tender comparison matrix with flags
4. FEAT-014: Owner Supplied v2 — multiple options per item with active selection toggle
5. FEAT-015: Owner Supplied — auto-populate from spec doc via AI
6. FEAT-010: Add new owner-supplied items manually via UI
7. FEAT-004/005: Add/edit/delete spec items and tenders in UI
8. FEAT-009: Mobile responsive polish
