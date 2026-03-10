# Reno Tracker — Changelog & Issue Log

## How to use this file

- **Bugs** are prefixed with `BUG-XXX`
- **Features** are prefixed with `FEAT-XXX`
- Status: `open` | `in-progress` | `done` | `wont-fix`
- Priority: `critical` | `high` | `medium` | `low`

---

## Changelog

### v1.1.0 — 2026-03-10 (AI Document Parsing)

- **FEAT-011:** Upload zones on Tender Comparison tab — drag-and-drop or click-to-browse for spec document and builder tenders (PDF/image)
- **FEAT-012:** AI parsing of spec documents — uploads spec to Anthropic API (claude-sonnet-4-20250514), extracts all line items, splits into builder-scope specItems and owner-supplied fixtures, auto-populates both sections
- **FEAT-013:** AI parsing of builder tenders — uploads each tender, maps items to spec, extracts pricing, flags deviations and omissions, auto-populates comparison matrix
- **FEAT-015:** Owner-supplied items auto-populated from spec document parsing (extracted by AI alongside spec items)
- API key management — secure input modal, stored in localStorage, shown in header with status indicator
- Upload zone component with drag-and-drop, file preview, parse button, loading spinner, success/error states
- Dynamic category detection — comparison matrix adapts to categories found in parsed spec (not hardcoded)
- Graceful empty states — dashboard and comparison tab handle zero tenders/spec items cleanly
- Spinner component for AI loading states

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
| BUG-001 | high | open | Owner Supplied tab has no way to input amounts for items we need to supply — only shows pre-loaded sample data with no add/edit capability | Go to Owner Supplied tab, try to add a new item | 2026-03-10 |

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
| FEAT-010 | high | open | Add new owner-supplied items manually via UI | Must be able to add items not in the spec doc (e.g. discovered during build) | 2026-03-10 |
| FEAT-011 | critical | done | Upload zones on Tender Comparison tab | Two areas: one for spec document upload, one for builder tender uploads. Accepts PDF/images. | 2026-03-10 |
| FEAT-012 | critical | done | AI parsing of spec document | On upload, use Anthropic API to read the spec doc and auto-populate specItems + ownerSupplied items. Should split builder-scope work from owner-supplied fixtures/fittings automatically. | 2026-03-10 |
| FEAT-013 | critical | done | AI parsing of builder tenders | On upload, use Anthropic API to read each tender, map items to spec, extract pricing, flag deviations from spec, flag omissions. Auto-populate the tender comparison matrix. | 2026-03-10 |
| FEAT-014 | high | open | Multiple options per owner-supplied item | Each item (e.g. "kitchen worktop") should support 2-3 supplier/price options. User selects which option is active — only the active one feeds into budget. Example: Option A (Howdens laminate £2,400), Option B (quartz £4,100), Option C (dekton £6,800). | 2026-03-10 |
| FEAT-015 | high | done | Auto-populate owner-supplied items from spec | When spec doc is parsed by AI, owner-supplied items should be automatically extracted and populated with descriptions and estimated budgets. | 2026-03-10 |

## Closed / Done

| ID | Description | Resolution | Date Closed |
|----|-------------|------------|-------------|
| FEAT-011 | Upload zones on Tender Comparison tab | Drag-and-drop upload for spec doc + builder tenders with file preview, parse button, loading/success/error states | 2026-03-10 |
| FEAT-012 | AI parsing of spec document | Client-side Anthropic API call parses spec into specItems + ownerSupplied items with structured prompts | 2026-03-10 |
| FEAT-013 | AI parsing of builder tenders | Client-side Anthropic API call maps tender items to spec, extracts pricing, flags deviations/omissions | 2026-03-10 |
| FEAT-015 | Auto-populate owner-supplied items from spec | Included in FEAT-012 — AI spec parsing extracts owner-supplied items alongside builder-scope items | 2026-03-10 |

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
