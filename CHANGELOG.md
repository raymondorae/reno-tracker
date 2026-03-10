# Reno Tracker — Changelog & Issue Log

## How to use this file

- **Bugs** are prefixed with `BUG-XXX`
- **Features** are prefixed with `FEAT-XXX`
- Status: `open` | `in-progress` | `done` | `wont-fix`
- Priority: `critical` | `high` | `medium` | `low`

---

## Changelog

### v2.0.0 — 2026-03-10 (Supabase Auth & Cloud Storage)

- **FEAT-025:** User authentication and cloud persistence — Supabase Auth (shared with Homegrown app on `ddokpzadckyyzolopkai.supabase.co`), cloud storage in `reno_projects` table with RLS, localStorage cache/offline fallback, automatic migration of existing localStorage data on first login.
- **Breaking change:** New data layer (localStorage → Supabase + localStorage cache). AuthGate wraps App — login required.
- Supabase JS v2 CDN added.
- AuthGate component: login/signup screen with zinc/amber theme, tab toggle, email/password auth, error/success messages.
- Cloud sync: 2-second debounced saves to Supabase after every localStorage write. Sync status indicator (saved/saving/error/offline) in Settings modal.
- Data loading: Supabase-first with localStorage fallback. On first login with existing localStorage data, all projects migrated to Supabase with user notification.
- Project operations (create/delete/rename/switch) sync to Supabase.
- Settings modal: cloud sync status section with coloured indicator dot, sign-out button.
- API key (`reno-tracker-ak`) remains per-device in localStorage, not synced to cloud.
- Online/offline detection updates sync status automatically.

### FEAT-025 — done (Supabase Auth & Cloud Storage)

- **AuthGate component:** Separate wrapper that owns auth state. Renders login/signup screen when no session, mounts App with `session` and `onSignOut` props when authenticated. App unmounts/remounts on auth change for clean data loading.
- **Auth flow:** `sb.auth.getSession()` on mount, `sb.auth.onAuthStateChange()` for ongoing updates. Sign-up shows "Check your email" confirmation message. Sign-out clears session and returns to login.
- **Cloud persistence:** `reno_projects` table with `UNIQUE(user_id, project_id)` constraint for upsert-by-client-ID. Row Level Security ensures users only access their own data. `name` column denormalised from JSONB for listing.
- **Debounced save:** `supabaseSave()` upserts to Supabase, wrapped by `debouncedCloudSave()` with 2-second timeout. `save()` calls localStorage immediately then `debouncedCloudSave()`. Cleanup effect cancels pending saves on unmount (sign-out).
- **Loading flow:** Fetch from Supabase first. If rows exist → load most recent, cache all to localStorage. If no rows → check localStorage for migration → upsert all to Supabase. If Supabase errors → fall back to localStorage.
- **Project operations:** `switchProject` fetches from Supabase first (fallback localStorage). `createProject` fires-and-forgets upsert. `deleteProject` fires-and-forgets delete. `renameProject` updates name column for non-active projects.
- **Settings UI:** Sync status dot (green=saved, amber-pulsing=saving, red=error, grey=offline). User email display. Red-tinted Sign Out button.

### v1.7.1 — 2026-03-10 (Excel & JSON Export)

- **FEAT-001:** Data export and backup — Excel (.xlsx) multi-tab export with Dashboard, Tenders, Milestones, Owner Supplied, and Scenarios tabs. JSON backup export and import for device transfer. All accessible from Settings modal.
- SheetJS (xlsx) CDN added for client-side Excel generation.
- Excel export includes currency formatting (£#,##0) and column width hints across all tabs.
- JSON import validates structure, patches missing fields, and replaces project data with confirmation.

### FEAT-001 — done (Excel & JSON Export)

- **Export to Excel:** Multi-tab .xlsx workbook generated client-side via SheetJS. Tab 1 (Dashboard): project summary, budget overview, tier estimates. Tab 2 (Tenders): full comparison matrix grouped by category with all tender prices. Tab 3 (Milestones): payment schedule with amounts and status. Tab 4 (Owner Supplied): all items with all options (max-options column groups). Tab 5 (Scenarios): each scenario with included/excluded items and budget status.
- **Export JSON:** Full project data exported as formatted JSON for backup and device transfer.
- **Import JSON:** File picker in Settings modal. Validates that the file contains a `specItems` array. Warns user that import replaces all data. Patches missing `specDocuments`, `tenderDocuments`, and `location` fields for safety.
- **UI:** "Data Export & Backup" section in Settings modal with three buttons: "Export to Excel" (amber), "Export JSON" (zinc), "Import JSON" (zinc + hidden file input).
- **Filename:** Sanitised project name + date, e.g. `My-Renovation-2026-03-10.xlsx`.

### BUG-008 — done (Budget input not editable)

- **Root cause:** v1.7.0 placed budget/contingency inputs in the small tier card header, but users naturally click the big prominent "TOTAL BUDGET £0" summary card which was a static `<div>`, not an input.
- **Fix:** Moved budget and contingency editing into the Total Budget summary card itself. Budget shows as an inline `<input>` styled to match the card's big bold text (transparent bg, bottom border that highlights amber on focus). Contingency is an inline input in the subtitle. Removed the redundant small inputs from the tier card header.

### v1.7.0 — 2026-03-10 (Budget Editing, Production Builds, Contingency Editing)

- **FEAT-024:** Editable budget and contingency — inline number inputs on the Dashboard (alongside area) let users set their total budget (£) and contingency percentage. Values persist to localStorage. Defaults preserved for existing users (£250k / 15%).
- **Production React builds:** Switched CDN from `react.development.min.js` to `react.production.min.js` (and ReactDOM equivalent) for faster runtime and no dev-mode warnings.
- **DEBUG-gated console output:** All `console.log`, `console.warn`, and `console.error` calls gated behind a `DEBUG` flag (default `false`). Set `DEBUG=true` in source to re-enable diagnostic logging.

### FEAT-024 — done (Budget & Contingency Editing)

- Editable total budget field (number input with £ prefix) on Dashboard, persists via existing `save()` to localStorage.
- Editable contingency percentage field (0–100, clamped) on Dashboard, persists the same way.
- Both inputs sit alongside the existing Area input in the "Estimated Cost by Tier" card header.
- No data model changes — `totalBudget` and `contingencyPct` already existed in the data model. This adds the missing UI.
- Existing users keep their current values (demo: £250k/15%, new projects: £0/10%).

### BUG-006 — done (Collapsible categories not working)

- **Root cause:** `CATS.map()` returned nested arrays with `null` entries for empty categories inside `<tbody>`. React's reconciliation can fail to properly re-render conditionally shown `<tr>` elements when the array structure is `[[<tr/>, <tr/>], null, [<tr/>], ...]`.
- **Fix:** Changed `CATS.map()` to `CATS.flatMap()` and `return null` to `return []` — React now receives a clean flat array of `<tr>` elements, ensuring collapse/expand state changes trigger correct re-renders.

### FEAT-023 — done (Spec validation / audit)

- "Audit Extraction" button in the spec document upload card — enabled only when spec items AND spec documents both exist.
- Sends uploaded spec document(s) and current extracted items JSON to Anthropic API with an audit-specific prompt.
- AI checks for: missing items, double-counted items, misallocated categories, and quantity mismatches.
- Results displayed in a modal with colour-coded sections: red for missing/qty issues, amber for double-counted/misallocated.
- Missing items: "+ Add" button auto-creates the spec item with suggested category, status "manual", cascades to tenders/scenarios.
- Quantity mismatches: "Update" button corrects the qty to match the spec value.
- Buttons change to "Added" / "Updated" (green) after actioning, using reactive data checks.
- Clean state: green summary card with checkmark when no issues found.
- Summary assessment shown at the top of the modal.

### FEAT-022 — done (Collapsible category groups in tender comparison matrix)

- Spec items in the comparison matrix are now grouped by category with collapsible header rows.
- Category header row shows: collapse/expand chevron, category name (bold, uppercase), item count, estimate range subtotal, and per-tender subtotals.
- All categories start collapsed by default for a cleaner overview of 70+ items.
- "Expand All" / "Collapse All" toggle link above the matrix.
- Three-level hierarchy: Category (collapsible) → Item (with existing component expand) → Components.
- Chevron rotates smoothly on toggle (reuses existing `.chv` transition).
- Grand totals row preserved at bottom.

### BUG-005 — done (Extract Items button misleadingly showed estimates)

- **Root cause:** Merge confirmation modal always displayed `£0 – £0 – £0` for extract-only items because `fmt(null)` formats as `£0`. Upload zone desc said "Estimates tuned for..." even though primary action is extract-only. User message sent to AI was identical for both modes.
- **Fix 1:** Merge modal now shows "No estimates — use ✦ buttons after import" instead of `£0` values when eLow/eMid/eHigh are all null.
- **Fix 2:** Upload zone desc changed from "Estimates tuned for [area]" to neutral "PDF or images — extract items, then estimate separately".
- **Fix 3:** AI user message now differs per mode — extract-only says "Do NOT estimate costs", estimates mode says "with cost estimates for [location]".
- **Fix 4:** Added hint text under each parse button: "Fast — items only, no cost estimates" and "Slower — includes eLow/eMid/eHigh + components".

### v1.6.0 — 2026-03-10 (Decoupled Spec Parsing & Selective Estimation)

- **FEAT-021:** Two-mode spec parsing, selective AI estimation, and manual estimate entry
- Part 1: Two parse buttons — "Extract Items from Spec" (fast, no estimates) and "Extract with AI Estimates" (full pricing). New "manual" status (blue outlined dot) for items without estimates. Conditional AI Estimate tender creation.
- Part 2: Per-item ✦ button, per-section "✦ Est." at category headers, and bulk "✦ Get All Estimates" with progress indicator. Validates API key and location. Inline spinners and error display.
- Part 3: Manual estimate entry — click Range column to edit eLow/eMid/eHigh inline. Typing estimates changes status to "confirmed" (green dot). "Clr" button per item resets estimates to null and status to "manual". Enter/Escape/Done to save or cancel.

### FEAT-021 — done (Two-Mode Spec Parsing & Selective Estimation)

- **FEAT-021 (Part 1):** Split spec parsing into two modes — "Extract Items Only" (fast, no estimates, no components) and "Extract with AI Estimates" (existing full behaviour with location-based pricing). Two parse buttons: primary "Extract Items from Spec" and secondary "Extract with AI Estimates". New "manual" status (blue outlined dot) for items parsed without estimates. Extract-only mode uses a simpler AI prompt for faster, more reliable results. `applyMerge` only creates/updates the AI Estimate tender when items have estimates. Manual items show as excluded in AI Estimate tender. Status legend updated to include manual indicator.
- **FEAT-021 (Part 2):** Selective AI estimation — get cost estimates per item, per category section, or in bulk. Per-item: ✦ button on manual items sends a small focused API call for that one item (eLow/eMid/eHigh + components breakdown). Per-section: "✦ Est." button on category headers estimates all manual items in that category in one API call. Bulk: "✦ Get All Estimates" button above the comparison matrix iterates through categories one at a time with progress indicator ("Estimating Electrical (3/7 sections)..."). All modes validate API key and location before calling, show inline loading spinners (pulsing ⟳), display inline errors per item on failure, and auto-update the AI Estimate tender on success. Uses functional `setData` to safely compose concurrent state updates.
- **FEAT-021 (Part 3):** Manual estimate entry — click the Range column in the comparison matrix to edit eLow/eMid/eHigh values inline with number inputs. Enter or Done button saves; Escape cancels. Typing estimates on a "manual" item changes status to "confirmed" (green dot). Typing estimates on an "estimated" item also changes to "confirmed". "Clr" button in the actions column resets eLow/eMid/eHigh to null, components to [], and status back to "manual". `updateEstimate()` and `clearEstimate()` functions added.

### BUG-004 — done (JSON Parse Fails on Large AI Responses)

- **Root cause:** `max_tokens` was set to 4096 — component breakdown arrays made responses much larger, causing truncation mid-JSON.
- **Fix 1:** Increased `max_tokens` from 4096 to 16384 in the Anthropic API call (shared by both spec and tender parsing).
- **Fix 2:** Added `repairTruncatedJSON()` helper — detects unclosed strings, cleans trailing partial tokens (commas, colons), and closes open brackets/braces in correct order using a stack. Integrated into `extractJSON` as a fallback after initial parse fails.
- **Fix 3:** `aiCall` now checks API response `stop_reason` — if `"max_tokens"`, logs console warning ("AI response was truncated") and returns `{text, truncated}` flag. Spec parsing shows truncation warning in the UI status message. Tender parsing logs to console.
- **Fix 4:** `extractJSON` now logs clearly whether parse succeeded on first try, after repair, or via regex fallback.
- **No duplicate API calls found** — only one fetch per parse operation, no React StrictMode double-render. A 401 error appearing before a successful call is likely a stale key or browser extension, not a code issue.

### FEAT-020 — done (Component Breakdown)

- **FEAT-020 (Part 1):** Component breakdown data model and AI parsing — specItems now support an optional `components` array where each component has `{description, qty, unitCost, subtotal}`. AI spec parsing prompt instructs the model to break each item into components whose subtotals approximate the eMid value. Tender parsing prompt now requests `tenderComponents` per item showing what the builder actually quoted. Merge confirmation modal shows "(X components)" label next to items that have breakdown detail. Backwards compatible — existing items without components work unchanged.
- **FEAT-020 (Part 2):** Expandable component breakdown in tender comparison matrix — chevron toggle (▶/▼) on spec items that have components, click to expand/collapse sub-rows showing individual component detail (description, qty, unit cost, subtotal). Tender columns show `tenderComponents` in parallel for side-by-side comparison. Components total row at bottom of each expanded section. Multiple items can be expanded simultaneously. Styled with muted background, indentation, amber left border accent, and smooth chevron rotation transition.

### v1.5.0 — 2026-03-10 (Multi-Project Support)

- **FEAT-019:** Multi-project support — create, switch, delete, and rename independent renovation projects
- **Data layer:** New localStorage keys: `reno-tracker-projects` (project metadata array), `reno-tracker-active-project` (active ID), `reno-tracker-proj-{id}` (per-project data). Auto-migration from old `reno-tracker-v2` single-key format.
- **Project switcher:** Dropdown in header lists all projects with name, created date, and "Demo" badge. Active project highlighted with checkmark. Chevron rotates on open. Transparent overlay for outside-click closing.
- **Create projects:** "New Project" button opens modal with name input and "Start with demo data" checkbox. Blank projects start with empty arrays for all entities.
- **Delete projects:** ✕ button on non-demo projects with confirmation dialog. Cannot delete the demo project or the last remaining project. Auto-switches to another project after deletion.
- **Rename projects:** Pencil icon next to project name in header enables inline rename. Saves on Enter/blur, updates both metadata array and project data.
- **Empty states:** All tabs render gracefully for blank projects — Dashboard shows £0 with safe division-by-zero guards, other tabs show "No items yet" messages.
- **Reset behaviour:** Reset button now respects project type — demo projects reset to demo data, blank projects reset to empty data.
- **API key:** Stored globally (`reno-tracker-ak`), shared across all projects. Location is per-project.

### v1.4.0 — 2026-03-10 (Add/Edit/Delete Spec Items & Tenders)

- **FEAT-004:** Full CRUD for spec items — add via modal (category, item, unit, qty, notes, estimate range), edit via pre-filled modal, delete with confirmation that cascades to all tenders and scenarios
- **FEAT-005:** Full CRUD for tenders — add via modal (builder name, auto-creates empty pricing for all spec items), delete with confirmation (auto-selects another tender, warns about AI Estimate regeneration), rename builder name inline (double-click column header), edit per-item pricing/included/notes inline in the comparison matrix (click any cell to edit)

### FEAT-004 — done

- **FEAT-004 (Part 1):** Add new spec items via UI — "Add Item" button on the Tender Comparison tab opens a modal with fields for category (dropdown), item name, unit, quantity, spec notes, and eLow/eMid/eHigh estimates. New items get a unique ID, status "confirmed" (green dot), appear in the comparison matrix with empty tender pricing, and are auto-included in all scenarios. Persists to localStorage.
- **FEAT-004 (Part 2):** Edit and delete spec items via UI — Each row in the comparison matrix has "Edit" and "Del" buttons. Edit opens the same modal pre-filled with the item's current values; saving updates the specItems array in place. Delete shows a confirmation dialog ("Delete [item name]? This will also remove it from all tenders and scenarios.") and on confirm removes the item from specItems, all tender items, and all scenario itemsIncluded lists. All changes persist to localStorage.

### v1.3.0 — 2026-03-10 (Partial Spec Merge, Location Estimates, Flexible Quotes)

- **FEAT-016:** Partial spec merge with confirmation — AI-parsed items shown in a review modal where user selects which to add, overwrite, or skip. No more destructive full-replace on re-parse.
- **FEAT-017:** Location-based cost estimates — each spec item gets eLow/eMid/eHigh estimates tuned to the user's area (e.g. Surrey). Dashboard shows Budget / Mid-Range / Premium cost tiers.
- **FEAT-018:** Flexible quote selection — owner-supplied options tagged with tier (Budget/Mid/Premium). AI auto-generates an "AI Estimate" tender from mid-range values.
- Status indicators on all items — sample (grey), estimated (amber pulsing), confirmed (green) dots throughout the UI
- Settings modal combining API key and location in one place
- Data migration for old ownerSupplied format (flat budget → options array)
- AI Estimate tender auto-created/updated when spec is parsed
- Scenarios auto-expanded to include newly parsed items

### v1.1.0 — 2026-03-10 (Upload Zones, Multi-Option Owner Supplied, Add Items UI)

- **FEAT-011:** Upload zones on Tender Comparison tab — drag-and-drop or click-to-browse for spec document and builder tenders (PDF/image)
- **FEAT-014:** Multiple options per owner-supplied item — each item supports supplier/price options with active selection toggle, only active option feeds into budget
- **FEAT-010:** Add new owner-supplied items manually via UI — users can add items not in the spec doc
- **BUG-001:** Fixed — Owner Supplied tab now supports adding and editing items
- **BUG-002:** Fixed — React error #310 caused by hooks called after conditional return statement

### v1.0.0 — 2026-03-10 (Initial release)

- Dashboard with budget overview and utilisation bar
- Spec-to-tender comparison matrix with deviation and omission flags
- Milestone payment tracker with status toggles
- Owner-supplied fixtures & fittings tracker with ordered/delivered states
- Scenario modelling with per-item toggles and budget impact
- Sample data for 18 spec items across 12 categories
- Three sample tenders with realistic pricing and deviations
- Nine milestone payment stages (deposit through retention)
- Nine owner-supplied fixture items
- Four pre-built scenarios
- localStorage persistence
- Vercel static deployment

---

## Open Bugs

None. All bugs resolved.

## Open Feature Requests

| ID | Priority | Status | Description | Notes | Date Logged |
|----|----------|--------|-------------|-------|-------------|
| FEAT-009 | high | open | Mobile responsive polish | Test and fix layout on small screens | 2026-03-10 |
| FEAT-027 | high | open | Minimise AI token costs | Batching, caching, Haiku for extraction, reducing prompt size | 2026-03-10 |
| FEAT-007 | medium | open | Custom milestone editing | Allow user to change milestone names, dates, and percentages | 2026-03-10 |
| FEAT-006 | medium | open | Add/edit/delete scenarios in UI | Can currently edit items within scenarios but not create new ones | 2026-03-10 |
| FEAT-026 | medium | open | Link to Homegrown app | Shared home screen showing both Renovation Tracker and Homegrown app options | 2026-03-10 |
| FEAT-028 | medium | open | Google Docs and .docx upload support | Client-side text extraction from .docx files and Google Docs links | 2026-03-10 |
| FEAT-029 | medium | open | Paywall for AI features | Stripe integration, token tracking per user, usage limits on free tier | 2026-03-10 |
| FEAT-003 | low | open | Print-friendly / PDF export view | Clean layout for sharing with builder or partner | 2026-03-10 |
| FEAT-008 | low | open | Photo/receipt attachment per item | Link images to owner-supplied items or milestone payments | 2026-03-10 |

## Closed / Done

| ID | Description | Resolution | Date Closed |
|----|-------------|------------|-------------|
| BUG-001 | Owner Supplied tab has no way to input amounts or add items | Fixed — add items UI and editable fields added in v1.1.0 | 2026-03-10 |
| BUG-002 | React error #310 — hooks called after conditional return | Fixed — all hooks moved before early return, reverted broken implementation and re-implemented cleanly | 2026-03-10 |
| BUG-003 | JSON parse failed on spec/tender AI responses | Robust extractJSON helper: strips code fences, trims preamble/postamble, regex fallback for outermost JSON, console logging on failure | 2026-03-10 |
| BUG-004 | JSON parse fails on large AI responses | Increased max_tokens 4096→16384, added repairTruncatedJSON helper, stop_reason truncation detection | 2026-03-10 |
| BUG-005 | Extract Items button misleadingly showed estimates | Merge modal shows "No estimates" for extract-only items. Upload zone desc neutralised. AI user message differentiated | 2026-03-10 |
| BUG-006 | Collapsible categories not working | Changed CATS.map() to CATS.flatMap() — gives React a flat array for proper reconciliation | 2026-03-10 |
| BUG-008 | Budget input not editable | Moved budget/contingency editing into the Total Budget summary card | 2026-03-10 |
| FEAT-001 | Excel & JSON export/import | Multi-tab Excel (.xlsx) export with 5 tabs, JSON backup/restore — all in Settings modal (v1.7.1) | 2026-03-10 |
| FEAT-002 | Cross-device sync | Superseded by FEAT-025 (Supabase cloud storage) | 2026-03-10 |
| FEAT-004 | Add/edit/delete spec items in UI | Add via modal, edit + delete with confirmation on each comparison matrix row (v1.4.0) | 2026-03-10 |
| FEAT-005 | Add/edit/delete tenders in UI | Add/delete tenders, rename builder inline, edit per-item pricing/included/notes (v1.4.0) | 2026-03-10 |
| FEAT-010 | Add new owner-supplied items manually via UI | Add item form on Owner Supplied tab (v1.1.0) | 2026-03-10 |
| FEAT-011 | Upload zones on Tender Comparison tab | Drag-and-drop upload for spec doc + builder tenders with file preview (v1.1.0) | 2026-03-10 |
| FEAT-012 | AI parsing of spec document | Client-side Anthropic API parses spec with location-based estimates, merge confirmation modal (v1.3.0) | 2026-03-10 |
| FEAT-013 | AI parsing of builder tenders | Maps tender items to spec IDs, extracts pricing, flags deviations/omissions (v1.3.0) | 2026-03-10 |
| FEAT-014 | Multiple options per owner-supplied item | Each item supports multiple supplier/price options with active selection toggle (v1.1.0) | 2026-03-10 |
| FEAT-015 | Auto-populate owner-supplied items from spec | AI extracts owner-supplied items with low/mid/high tier estimates (v1.3.0) | 2026-03-10 |
| FEAT-016 | Partial spec merge with confirmation | Review modal to add/overwrite/skip parsed items, no destructive full-replace (v1.3.0) | 2026-03-10 |
| FEAT-017 | Location-based cost estimates | eLow/eMid/eHigh per spec item tuned to user's area, dashboard tier breakdown (v1.3.0) | 2026-03-10 |
| FEAT-018 | Flexible quote selection with tiers | Owner-supplied options tagged Budget/Mid/Premium, AI Estimate tender auto-generated (v1.3.0) | 2026-03-10 |
| FEAT-019 | Multi-project support | Create, switch, delete, rename projects. Per-project localStorage. Auto-migration (v1.5.0) | 2026-03-10 |
| FEAT-020 | Component breakdown for spec items and tenders | Data model + AI parsing + expandable sub-rows in comparison matrix (v1.5.0) | 2026-03-10 |
| FEAT-021 | Two-mode spec parsing, selective estimation & manual entry | Extract-only vs with-estimates, per-item/section/bulk estimation, inline manual editing (v1.6.0) | 2026-03-10 |
| FEAT-022 | Collapsible category groups in tender comparison matrix | Category headers with chevron toggle, expand/collapse all, subtotals per category (v1.6.0) | 2026-03-10 |
| FEAT-023 | Spec validation / audit view | AI audit of extraction against spec documents, add missing items, fix qty mismatches (v1.6.0) | 2026-03-10 |
| FEAT-024 | Budget & contingency editing | Inline editable budget (£) and contingency (%) inputs on Dashboard (v1.7.0) | 2026-03-10 |
| FEAT-025 | User authentication and cloud storage | Supabase Auth (shared with Homegrown app), cloud persistence, localStorage cache/offline fallback, debounced sync (v2.0.0) | 2026-03-10 |

---

## Implementation Notes

### AI Document Parsing (FEAT-011, 012, 013)

The app uses the Anthropic API client-side via fetch to `api.anthropic.com/v1/messages` using `claude-sonnet-4-20250514`. Documents are uploaded as base64 (PDF or image), sent to the API with a structured prompt requesting JSON output, and the response is parsed into the app's data model.

**Spec doc parsing prompt should:**
- Extract every line item with category, description, quantity, unit, and notes
- Separate builder-scope items (specItems) from owner-supplied items (ownerSupplied)
- Return structured JSON matching the app's data model

**Tender parsing prompt should:**
- Map each quoted line item to the corresponding specItem by ID
- Extract price, included/excluded status, and any notes or deviations
- Flag where the tender description differs from the spec
- Flag where spec items are missing entirely from the tender
- Return structured JSON matching the app's tender data model

### Owner Supplied Multi-Option (FEAT-014)

Data model change:
```json
{
  "id": "os1",
  "item": "Kitchen worktop",
  "specNotes": "From spec: 3m run, 600mm deep",
  "options": [
    { "id": "opt1", "supplier": "Howdens", "description": "Laminate, oak effect", "price": 2400, "active": true },
    { "id": "opt2", "supplier": "Benchmarx", "description": "Quartz, white", "price": 4100, "active": false },
    { "id": "opt3", "supplier": "Bespoke", "description": "Dekton, Trilium", "price": 6800, "active": false }
  ],
  "ordered": false,
  "delivered": false
}
```

Budget calculations should use the `active` option's price. Scenario modelling should also respect the active selection.

---

## Notes

- Supabase project ID: `ddokpzadckyyzolopkai` (shared with Homegrown app)
- Supabase anon key is embedded in index.html — safe to expose (RLS protects data)
- API key (`reno-tracker-ak`) is per-device in localStorage, never synced to cloud
- Any new tables should use RLS and reference `auth.users(id)` for the same shared auth
