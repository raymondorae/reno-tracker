# Reno Tracker — Changelog & Issue Log

## How to use this file

- **Bugs** are prefixed with `BUG-XXX`
- **Features** are prefixed with `FEAT-XXX`
- Status: `open` | `in-progress` | `done` | `wont-fix`
- Priority: `critical` | `high` | `medium` | `low`

---

## Changelog

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

| ID | Priority | Status | Description | Steps to Reproduce | Date Logged |
|----|----------|--------|-------------|---------------------|-------------|
| BUG-003 | high | done | JSON parse failed on spec/tender AI responses | Upload spec or tender PDF, parse via AI — response contains markdown code fences or preamble text that breaks JSON.parse | 2026-03-10 |
| BUG-004 | high | done | JSON parse fails on large AI responses despite valid-looking start | max_tokens too low (4096) causes truncation with component arrays. extractJSON had no repair logic for truncated JSON. | 2026-03-10 |
| BUG-005 | medium | done | Extract Items button misleadingly showed estimates | Merge modal showed £0 values for extract-only items; upload zone desc implied estimates; AI user message was identical for both modes | 2026-03-10 |

## Open Feature Requests

| ID | Priority | Status | Description | Notes | Date Logged |
|----|----------|--------|-------------|-------|-------------|
| FEAT-001 | medium | open | Data export/import as JSON | Allow backup and transfer between devices | 2026-03-10 |
| FEAT-002 | medium | open | Cross-device sync | Replace localStorage with cloud storage or simple backend | 2026-03-10 |
| FEAT-003 | low | open | Print-friendly / PDF export view | Clean layout for sharing with builder or partner | 2026-03-10 |
| FEAT-004 | medium | done | Add/edit/delete spec items in UI | Part 1: add via modal. Part 2: edit (pre-filled modal) + delete (with confirmation) on each row | 2026-03-10 |
| FEAT-005 | medium | done | Add/edit/delete tenders in UI | Add/delete tenders, rename builder, edit per-item pricing/included/notes inline | 2026-03-10 |
| FEAT-006 | low | open | Add/edit/delete scenarios in UI | Can currently edit items within scenarios but not create new ones | 2026-03-10 |
| FEAT-007 | medium | open | Custom milestone editing | Allow user to change milestone names, dates, and percentages | 2026-03-10 |
| FEAT-008 | low | open | Photo/receipt attachment per item | Link images to owner-supplied items or milestone payments | 2026-03-10 |
| FEAT-009 | high | open | Mobile responsive polish | Test and fix layout on small screens | 2026-03-10 |
| FEAT-010 | high | done | Add new owner-supplied items manually via UI | Must be able to add items not in the spec doc (e.g. discovered during build) | 2026-03-10 |
| FEAT-011 | critical | done | Upload zones on Tender Comparison tab | Two areas: one for spec document upload, one for builder tender uploads. Accepts PDF/images. | 2026-03-10 |
| FEAT-012 | critical | done | AI parsing of spec document | Parses spec, extracts items with location-based estimates, shows merge confirmation modal | 2026-03-10 |
| FEAT-013 | critical | done | AI parsing of builder tenders | Maps tender items to spec, extracts pricing, flags deviations/omissions | 2026-03-10 |
| FEAT-014 | high | done | Multiple options per owner-supplied item | Each item (e.g. "kitchen worktop") should support 2-3 supplier/price options. User selects which option is active — only the active one feeds into budget. Example: Option A (Howdens laminate £2,400), Option B (quartz £4,100), Option C (dekton £6,800). | 2026-03-10 |
| FEAT-015 | high | done | Auto-populate owner-supplied items from spec | AI extracts owner-supplied items with low/mid/high estimates during spec parse | 2026-03-10 |
| FEAT-016 | high | done | Partial spec merge with confirmation | Review modal to select/overwrite/skip AI-parsed items instead of full replace | 2026-03-10 |
| FEAT-017 | high | done | Location-based cost estimates | Spec items get eLow/eMid/eHigh tuned to user's area, dashboard shows tier breakdown | 2026-03-10 |
| FEAT-018 | medium | done | Flexible quote selection with tiers | Owner-supplied options tagged Budget/Mid/Premium, AI Estimate tender auto-generated | 2026-03-10 |
| FEAT-019 | high | done | Multi-project support | Create, switch, delete, rename projects. Per-project data isolation. Auto-migration from old format. | 2026-03-10 |
| FEAT-020 | high | done | Component breakdown for spec items and tenders | Part 1: data model + AI parsing + merge modal label. Part 2: expandable sub-rows in comparison matrix with tender component comparison. | 2026-03-10 |
| FEAT-021 | high | done | Two-mode spec parsing, selective estimation & manual entry | Part 1: two parse buttons, manual status, conditional AI Estimate tender. Part 2: per-item, per-section, and bulk estimation with progress. Part 3: inline manual estimate editing, clear button. | 2026-03-10 |

## Closed / Done

| ID | Description | Resolution | Date Closed |
|----|-------------|------------|-------------|
| BUG-001 | Owner Supplied tab has no way to input amounts or add items | Fixed — add items UI and editable fields added in v1.1.0 | 2026-03-10 |
| BUG-002 | React error #310 — hooks called after conditional return | Fixed — all hooks moved before early return, reverted broken implementation and re-implemented cleanly | 2026-03-10 |
| FEAT-010 | Add new owner-supplied items manually via UI | Add item form on Owner Supplied tab | 2026-03-10 |
| FEAT-011 | Upload zones on Tender Comparison tab | Drag-and-drop upload for spec doc + builder tenders with file preview | 2026-03-10 |
| FEAT-014 | Multiple options per owner-supplied item | Each item supports multiple supplier/price options with active selection toggle | 2026-03-10 |
| FEAT-012 | AI parsing of spec document | Client-side Anthropic API parses spec with location-based estimates, merge confirmation modal | 2026-03-10 |
| FEAT-013 | AI parsing of builder tenders | Maps tender items to spec IDs, extracts pricing, flags deviations/omissions | 2026-03-10 |
| FEAT-015 | Auto-populate owner-supplied items from spec | AI extracts owner-supplied items with low/mid/high tier estimates | 2026-03-10 |
| FEAT-016 | Partial spec merge with confirmation | Review modal to add/overwrite/skip parsed items, no destructive full-replace | 2026-03-10 |
| FEAT-017 | Location-based cost estimates | eLow/eMid/eHigh per spec item tuned to user's area, dashboard tier breakdown | 2026-03-10 |
| FEAT-018 | Flexible quote selection with tiers | Owner-supplied options tagged Budget/Mid/Premium, AI Estimate tender auto-generated | 2026-03-10 |
| FEAT-004 | Add/edit/delete spec items in UI | Part 1: add via modal. Part 2: edit + delete with confirmation on each comparison matrix row | 2026-03-10 |
| FEAT-005 | Add/edit/delete tenders in UI | Add/delete tenders, rename builder inline, edit per-item pricing/included/notes inline in matrix | 2026-03-10 |
| FEAT-019 | Multi-project support | Create, switch, delete, rename projects. Per-project localStorage. Auto-migration. Empty state handling. | 2026-03-10 |
| BUG-003 | JSON parse failed on spec/tender AI responses | Robust extractJSON helper: strips code fences, trims preamble/postamble, regex fallback for outermost JSON, console logging on failure. Strengthened system prompts to explicitly forbid markdown formatting. | 2026-03-10 |
| FEAT-020 | Component breakdown for spec items and tenders | Part 1: data model + AI parsing + merge modal label. Part 2: expandable sub-rows in comparison matrix with chevron toggle, tender component parallel display, components total row. | 2026-03-10 |
| BUG-004 | JSON parse fails on large AI responses | Increased max_tokens 4096→16384, added repairTruncatedJSON helper (unclosed string handling, bracket closing), stop_reason truncation detection, clear console logging. | 2026-03-10 |
| FEAT-021 | Two-mode spec parsing, selective estimation & manual entry | Part 1: extract-only vs with-estimates modes. Part 2: per-item ✦ button, per-section Est, bulk Get All Estimates with progress. Part 3: inline manual estimate editing (click Range column), clear estimates button. | 2026-03-10 |
| BUG-005 | Extract Items button misleadingly showed estimates | Merge modal shows "No estimates" for extract-only items. Upload zone desc neutralised. AI user message differentiated. Hint text under buttons. | 2026-03-10 |

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

- When real spec and tender data is loaded, update version to 1.1.0
- Any breaking change to the localStorage data model should increment to 2.0.0 and include migration logic
- FEAT-011/012/013 are the highest priority — they transform the app from a manual tracker to an AI-powered tool
