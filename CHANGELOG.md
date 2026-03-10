# Reno Tracker — Changelog & Issue Log

## How to use this file

- **Bugs** are prefixed with `BUG-XXX`
- **Features** are prefixed with `FEAT-XXX`
- Status: `open` | `in-progress` | `done` | `wont-fix`
- Priority: `critical` | `high` | `medium` | `low`

---

## Changelog

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
| — | — | — | — | — | — |

## Open Feature Requests

| ID | Priority | Status | Description | Notes | Date Logged |
|----|----------|--------|-------------|-------|-------------|
| FEAT-001 | medium | open | Data export/import as JSON | Allow backup and transfer between devices | 2026-03-10 |
| FEAT-002 | medium | open | Cross-device sync | Replace localStorage with cloud storage or simple backend | 2026-03-10 |
| FEAT-003 | low | open | Print-friendly / PDF export view | Clean layout for sharing with builder or partner | 2026-03-10 |
| FEAT-004 | medium | open | Add/edit/delete spec items in UI | Currently requires code change to modify spec items | 2026-03-10 |
| FEAT-005 | medium | open | Add/edit/delete tenders in UI | Currently requires code change to add new tenders | 2026-03-10 |
| FEAT-006 | low | open | Add/edit/delete scenarios in UI | Can currently edit items within scenarios but not create new ones | 2026-03-10 |
| FEAT-007 | medium | open | Custom milestone editing | Allow user to change milestone names, dates, and percentages | 2026-03-10 |
| FEAT-008 | low | open | Photo/receipt attachment per item | Link images to owner-supplied items or milestone payments | 2026-03-10 |
| FEAT-009 | high | open | Mobile responsive polish | Test and fix layout on small screens | 2026-03-10 |
| FEAT-010 | high | done | Add new owner-supplied items manually via UI | Must be able to add items not in the spec doc (e.g. discovered during build) | 2026-03-10 |
| FEAT-011 | critical | done | Upload zones on Tender Comparison tab | Two areas: one for spec document upload, one for builder tender uploads. Accepts PDF/images. | 2026-03-10 |
| FEAT-012 | critical | open | AI parsing of spec document | On upload, use Anthropic API to read the spec doc and auto-populate specItems + ownerSupplied items. Should split builder-scope work from owner-supplied fixtures/fittings automatically. | 2026-03-10 |
| FEAT-013 | critical | open | AI parsing of builder tenders | On upload, use Anthropic API to read each tender, map items to spec, extract pricing, flag deviations from spec, flag omissions. Auto-populate the tender comparison matrix. | 2026-03-10 |
| FEAT-014 | high | done | Multiple options per owner-supplied item | Each item (e.g. "kitchen worktop") should support 2-3 supplier/price options. User selects which option is active — only the active one feeds into budget. Example: Option A (Howdens laminate £2,400), Option B (quartz £4,100), Option C (dekton £6,800). | 2026-03-10 |
| FEAT-015 | high | open | Auto-populate owner-supplied items from spec | When spec doc is parsed by AI, owner-supplied items should be automatically extracted and populated with descriptions and estimated budgets. | 2026-03-10 |

## Closed / Done

| ID | Description | Resolution | Date Closed |
|----|-------------|------------|-------------|
| BUG-001 | Owner Supplied tab has no way to input amounts or add items | Fixed — add items UI and editable fields added in v1.1.0 | 2026-03-10 |
| BUG-002 | React error #310 — hooks called after conditional return | Fixed — all hooks moved before early return, reverted broken implementation and re-implemented cleanly | 2026-03-10 |
| FEAT-010 | Add new owner-supplied items manually via UI | Add item form on Owner Supplied tab | 2026-03-10 |
| FEAT-011 | Upload zones on Tender Comparison tab | Drag-and-drop upload for spec doc + builder tenders with file preview | 2026-03-10 |
| FEAT-014 | Multiple options per owner-supplied item | Each item supports multiple supplier/price options with active selection toggle | 2026-03-10 |

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
