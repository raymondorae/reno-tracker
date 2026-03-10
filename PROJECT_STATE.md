# Reno Tracker — Project State

*Last updated: 10 March 2026*

---

## Project Overview

**App:** Renovation Budget & Project Tracker
**Type:** Static single-page web app (no build step)
**Stack:** HTML, React 18 (CDN, production builds), Tailwind CSS (CDN), Babel standalone, Supabase JS v2 (CDN)
**Hosting:** Vercel (static deployment via GitHub auto-deploy)
**Repo:** github.com/raymondorae/reno-tracker
**Auth:** Supabase Auth (shared with Homegrown app on `ddokpzadckyyzolopkai.supabase.co`)
**Data persistence:** Supabase `reno_projects` table (primary) + browser localStorage (cache/offline fallback)
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
| Dashboard | Live | Budget overview with editable budget/contingency/area, utilisation bar, tender selector, issue summary, milestone overview |
| Tender Comparison | Live (v5) | Upload zones, AI spec parsing with merge confirmation, AI tender parsing, comparison matrix with estimate ranges and inline estimate editing, status indicators, inline cell editing, add/delete tenders, rename builders, clear estimates |
| Milestones | Live | Payment schedule by contract %, status toggles (pending/partial/paid/overdue) |
| Owner Supplied | Live (v3) | Multi-option with tier tags (Budget/Mid/Premium), add real quotes, add new items, ordered/delivered toggles |
| AI Integration | Live (v2) | Two-mode spec parsing (extract-only or with AI estimates), selective per-item/section/bulk estimation, manual estimate entry, tender parsing, partial merge with confirmation modal, AI Estimate tender auto-generation |
| Multi-Project | Live | Create, switch, delete, rename projects. Per-project data isolation. Auto-migration from old format. |
| Data Export | Live | Excel (.xlsx) multi-tab export, JSON backup/restore via Settings modal |
| Scenarios | Live | What-if modelling, toggle items per scenario, budget impact calculation |
| Auth & Cloud Sync | Live | Supabase auth (login/signup), cloud persistence with 2s debounced save, localStorage cache, offline fallback, automatic migration |

---

## Data Model

### Supabase (primary — cloud)

- **`reno_projects`** table — One row per user per project:
  - `id` (UUID PK), `user_id` (FK → auth.users), `project_id` (TEXT, e.g. `"proj-demo"`), `name` (TEXT), `data` (JSONB — full project data), `created_at`, `updated_at`
  - `UNIQUE(user_id, project_id)` — enables upsert by client-side ID
  - Row Level Security: users can only CRUD their own rows

### localStorage (cache / offline fallback)

- **`reno-tracker-projects`** — Array of project metadata: `{id, name, createdAt, isDemo}`
- **`reno-tracker-active-project`** — ID of the currently active project
- **`reno-tracker-proj-{id}`** — Per-project JSON data object containing:
  - **specItems** — Line items from the spec document (id, category, item, unit, qty, specNotes, status, eLow, eMid, eHigh, components[])
  - **tenders** — Builder submissions with status (sample/estimated/confirmed), per-item pricing, included flag, and notes
  - **milestones** — Payment stages with % of contract, due date, status, paid amount
  - **ownerSupplied** — Fixtures/fittings with options[] array (each option has supplier, description, price, active, tier), ordered/delivered toggles
  - **scenarios** — Named sets of included/excluded spec items and owner items for what-if modelling
  - **specDocuments** — Uploaded spec document files (base64 data URLs)
  - **tenderDocuments** — Uploaded tender files with builderName label
  - **location** — User's area for location-based cost estimates (per-project)
- **`reno-tracker-ak`** — Anthropic API key (per-device, not per-account, persists across sign-out/sign-in)

### Data flow

1. On login: fetch all projects from Supabase → cache to localStorage. If Supabase is empty but localStorage has data → migrate to Supabase.
2. On save: write to localStorage immediately → debounced (2s) upsert to Supabase.
3. On switch project: fetch from Supabase first → fall back to localStorage if offline.
4. On create/delete/rename: localStorage immediate + fire-and-forget Supabase sync.

---

## AI Integration (Live)

The app uses the Anthropic API (client-side fetch to `api.anthropic.com/v1/messages` using `claude-sonnet-4-20250514` with `anthropic-dangerous-direct-browser-access` header) to:

1. **Parse spec documents (two modes)** — "Extract Items Only" (fast, no estimates) or "Extract with AI Estimates" (full location-based pricing with component breakdown). Results shown in merge confirmation modal.
2. **Selective estimation** — Per-item ✦ button, per-section "✦ Est." at category headers, and bulk "✦ Get All Estimates" with progress. Manual estimate entry via clickable Range column. Clear estimates button per item.
3. **Parse tender responses** — User uploads each builder's tender. AI maps items to spec IDs, extracts pricing, flags deviations and omissions.
4. **Auto-generate AI Estimate tender** — After spec parse or estimation, an "AI Estimate" tender is created/updated from mid-range values so the user can compare against real quotes.
5. **Location-aware pricing** — Estimates adjusted for local labour and material costs based on user's configured area.

This runs entirely client-side. No backend needed. User provides their own Anthropic API key via the Settings modal (stored in localStorage under `reno-tracker-ak`).

---

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| Single HTML file | Zero build complexity, instant Vercel deploy, easy to maintain |
| CDN dependencies | No node_modules, no version drift in lockfiles |
| Supabase + localStorage | Cloud persistence for cross-device sync, localStorage as cache and offline fallback. Shared Supabase project with Homegrown app for single sign-on. |
| Sample data pre-loaded | User can explore immediately, replace with real data later |
| GBP currency | UK-based user |
| Client-side Anthropic API | No backend to manage, runs in browser. User supplies API key via Settings modal (global, shared across projects). |
| Multi-project support | Each project has independent data in its own localStorage key. API key is global; location is per-project. |
| Partial merge on parse | AI parse shows confirmation modal — user picks items to add/overwrite/skip instead of destructive replace |
| Location-based estimates | AI cost estimates tuned to user's area for more accurate budgeting |
| Tier-based owner options | Budget/Mid/Premium tiers on owner-supplied items for quick cost comparison |
| Decoupled estimation | Spec extraction and cost estimation are independent steps — user can extract items fast, then selectively estimate per-item, per-section, or in bulk |

---

## Current State

- **Version:** 2.0.0
- **Last updated:** 2026-03-10
- **Auth:** Supabase login/signup (shared with Homegrown app). AuthGate component wraps App — no session = login screen.
- **Cloud sync:** All project data persisted to Supabase `reno_projects` table with 2s debounced saves. localStorage used as cache and offline fallback. Existing localStorage data auto-migrated on first login. Sync status indicator in Settings.
- **Multi-project:** Create, switch, delete, rename independent projects. Auto-migration from old single-key format. Demo project included by default. Blank projects start empty with graceful empty states across all tabs.
- **Sample data:** 18 spec items (with eLow/eMid/eHigh estimates), 3 sample tenders, 9 milestones, 9 owner-supplied items (with Budget/Mid/Premium tiers), 4 scenarios
- **AI features:** Two-mode spec parsing (extract-only or with estimates), selective per-item/section/bulk AI estimation with progress, manual estimate entry via inline editing, tender parsing, location-based estimates, AI Estimate tender auto-generation
- **Owner supplied:** Multi-option with tier tags, add real quotes, add new items UI
- **Spec items:** Full CRUD — add, edit, delete via UI with cascade to tenders and scenarios. Inline estimate editing (click Range column). Clear estimates button.
- **Tenders:** Full CRUD — add, delete, rename builder, edit per-item pricing/included/notes inline in comparison matrix

---

## Next Steps (Priority Order)

1. FEAT-009: Mobile responsive polish
2. FEAT-027: Minimise AI token costs
3. FEAT-007: Custom milestone editing
4. FEAT-006: Add/edit/delete scenarios in UI
5. FEAT-026: Link to Homegrown app (shared home screen)
6. FEAT-028: Google Docs and .docx upload support
7. FEAT-029: Paywall for AI features
8. FEAT-003: Print-friendly / PDF export view
9. FEAT-008: Photo/receipt attachment per item
