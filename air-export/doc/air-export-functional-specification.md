# Air Export Request / Booking — Functional Specification

> **Service:** Air Export &nbsp;|&nbsp; **Type:** Request / Booking creation &nbsp;|&nbsp; **Status:** Spec v1
> **Companion docs:** [Page Mockup](../mockup/air-export-page-mockup.md) · [Theme Compliance](../theme/air-export-theme-compliance.md) · [Design System](../../../02-design-system.md)
> **Last updated:** 2026-05-29

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Business value](#2-business-value)
3. [Scope & out of scope](#3-scope--out-of-scope)
4. [Roles & permissions](#4-roles--permissions)
5. [Workflow (end-to-end)](#5-workflow-end-to-end)
6. [Data model](#6-data-model)
7. [Validation rules](#7-validation-rules)
8. [Derived calculations](#8-derived-calculations)
9. [User actions & outcomes](#9-user-actions--outcomes)
10. [AI Assistant (Cargo Copilot) behavior](#10-ai-assistant-cargo-copilot-behavior)
11. [Error handling](#11-error-handling)
12. [States & lifecycle](#12-states--lifecycle)
13. [Notifications](#13-notifications)
14. [Acceptance criteria](#14-acceptance-criteria)
15. [Future enhancements](#15-future-enhancements)

---

## 1. Purpose

The Air Export Request / Booking page exists to **capture, validate, and submit** all information
required to price and execute an air export shipment, in one guided surface. It is the on-ramp for
two outcomes that share the same data:

1. **A quotation** — an indicative or firm price for an air export move.
2. **A booking** — a committed shipment that operations can turn into an operational job.

It replaces fragmented, email/spreadsheet-driven intake and dense ERP forms with a structured,
assisted experience that produces **clean, complete, compliant** shipment records.

---

## 2. Business value

| Value | How the page delivers it |
|---|---|
| **Faster intake** | Smart defaults, CRM prefill, auto-computed weights, and one page instead of many reduce time-to-submit. |
| **Higher data quality** | Field validation + the Cargo Copilot catch errors (bad Incoterm, missing email, vague descriptions) before they reach operations. |
| **Fewer compliance failures** | DG/temperature/document rules are enforced at entry; the costliest air-export mistakes (missing MSDS, wrong commodity) are flagged early. |
| **Better conversion** | A frictionless "Request a rate" path turns more enquiries into quotes; a clean quote→booking transition reduces drop-off. |
| **Operational handoff** | A validated request becomes an operational job with no rekeying, shrinking ops effort and error. |
| **Scalability** | One responsive page serves customers, forwarders, and ops across devices, lowering training and support cost. |

---

## 3. Scope & out of scope

**In scope:** shipment/route/cargo/party capture, special requirements, document upload,
chargeable-weight calculation, indicative pricing display, draft autosave, quote request, booking
submission, Cargo Copilot validation/suggestions, role-based field visibility.

**Out of scope (handled elsewhere / future):** carrier rate negotiation engine, AWB issuance,
operational job execution, invoicing/accounting, track-and-trace, customs filing integration.
These are downstream of this page and referenced where relevant.

---

## 4. Roles & permissions

| Role | Create | Submit booking | See internal fields | Notes |
|---|---|---|---|---|
| **Customer (Shipper)** | ✓ (own) | Request only | ✗ | Cannot see cost notes/margins; can request a rate and submit a booking request pending forwarder confirmation |
| **Freight Forwarder** | ✓ | ✓ | ✓ (cost/margin) | Full access; can quote and book on behalf of customers |
| **Operations** | ✓ | ✓ | ✓ + ops remarks | Validates and creates the operational job |
| **Admin** | ✓ | ✓ | ✓ | Plus configuration (commodity lists, doc rules) |

Internal-only fields (cost notes, margin, ops remarks) are rendered conditionally by role and are
never sent to customer-facing views or notifications.

---

## 5. Workflow (end-to-end)

```
        ┌────────────┐      ┌───────────────┐      ┌──────────┐      ┌──────────┐      ┌───────────────┐
 START ▶│  Draft     │ ───▶ │ Quote         │ ───▶ │ Quoted   │ ───▶ │ Booked   │ ───▶ │ Job created   │ ▶ OPS
        │ (autosave) │      │ requested     │      │ (price)  │      │          │      │ (handoff)     │
        └────────────┘      └───────────────┘      └──────────┘      └──────────┘      └───────────────┘
              │                    │                     │                 │
       edit any section    validation passes      accept / revise   docs complete +
                           (rate minimum)         the quote         booking validation
```

1. **Create / open draft.** User lands on the page in `Draft`. Mode defaults to *Request a rate*.
   Autosave runs on debounce.
2. **Capture shipment data.** User fills the cards in any order; the stepper and completeness meter
   track progress. Copilot advises live.
3. **Request a quote.** When rate-mode minimum is met, *Request quote* enables. Submitting
   validates, transitions to `Quote requested`, and triggers pricing.
4. **Quoted.** An indicative/firm price returns; the estimate card and status update to `Quoted`.
   User can revise inputs (re-prices) or proceed.
5. **Create booking.** User switches to *Create booking*, completes parties + required documents.
   Booking validation must pass. Submitting transitions to `Booked`.
6. **Handoff to operations.** A booking generates an operational job; ops validates and executes.
   Any post-submit change requires an amendment (see [§12](#12-states--lifecycle)).

At every step the user may **Save draft**, **Duplicate**, or **Discard** (with confirmation).

---

## 6. Data model

Logical entities captured by the page (field-level types/flags are in the
[Mockup §6](../mockup/air-export-page-mockup.md#6-section-by-section-breakdown)).

```
AirExportRequest
├─ meta            { id, status, mode(rate|booking), createdBy, role, createdAt, updatedAt }
├─ shipment        { shipmentType, serviceType, movementType, incoterm, commodityType, description, hsCode? }
├─ route           { originAirport(IATA), destAirport(IATA),
│                    pickup?{ address, city, zip, country, readyDate },
│                    delivery?{ address, city, zip, country, notes } }
├─ cargo
│   ├─ pieces[]    { qty, lengthCm, widthCm, heightCm, weightKgPerPc, packageType }
│   └─ totals      { pieces, grossKg, volumeM3, volumetricKg, chargeableKg }   // derived
├─ shipper         { company, contact, email, phone, address }
├─ consignee       { company, contact, email, phone, address }
├─ special
│   ├─ dangerousGoods?    { unNumber, hazardClass, packingGroup, properName }
│   ├─ temperature?       { minC, maxC, packaging }
│   ├─ insurance?         { value, currency }
│   ├─ express            bool
│   ├─ oversized?         { reason }
│   └─ stackable          bool
├─ documents[]     { type, fileName, size, mime, status(uploading|done|error) }
└─ estimate        { low, high, currency, indicative:bool, breakdown[] }       // returned by pricing
```

Units are stored in **SI** (cm, kg, m³); display unit toggles (cm/in, kg/lb) are presentation-only.

---

## 7. Validation rules

### 7.1 Field-level

| Field | Rule |
|---|---|
| Shipment Type | Required; must be *Air Export*. |
| Service Type | Required; one of A→A, D→D, D→A, A→D. |
| Movement Type | Required. |
| Incoterm | Required; must be an air-applicable term (EXW, FCA, CPT, CIP, DAP, DDP, DPU). Sea-only terms (FOB, CFR, CIF) → **error** with explanation. |
| Commodity Type | Required. |
| Cargo Description | Optional but Copilot warns if blank or vague ("samples", "gifts", "parts"). |
| Origin / Destination airport | Required; valid IATA code from lookup; **must differ**; pair must be serviceable. |
| Pickup address block | Required **iff** Service Type starts with *Door*. Ready date ≥ today. |
| Delivery address block | Required **iff** Service Type ends with *Door*. |
| Pieces — qty / L / W / H / weight | Required per line; numeric > 0; sane upper bounds (e.g. single piece ≤ 7,000 kg / dims ≤ 320 cm warn for ULD/freighter). At least **one** piece line required. |
| Package type | Required per line. |
| Shipper.company / .email | Company required; email required + RFC-valid format. |
| Consignee.company / .email | Company required; email required + RFC-valid format. |
| Phone | Optional; if present, valid international format. |
| DG fields (when DG on) | UN number (UN + 4 digits), hazard class (1–9), packing group (I/II/III where applicable), proper shipping name — all required. |
| Temperature (when on) | minC < maxC; both required; range plausibility check. |
| Insurance (when on) | Insured value > 0 + currency required. |
| Documents | Commercial invoice & packing list required **for booking**; MSDS required **iff** DG on; file ≤ 10 MB; mime ∈ {pdf, jpg, png}. |

### 7.2 Cross-field / business

- **Mode minimums:**
  - *Request a rate* requires: Service Type, Commodity, Origin, Destination, ≥1 valid piece line.
  - *Create booking* additionally requires: Incoterm, full Shipper (company+email), full Consignee
    (company+email), required documents, and all active special-requirement panels valid.
- **Commodity ⇄ Special Requirement consistency:** Commodity = Dangerous goods **requires** the DG
  panel on (and vice-versa). Commodity = Perishables/Pharma suggests Temperature.
- **Service Type ⇄ Route:** address requirements derive from Door/Airport endpoints (§7.1).
- **Chargeable weight** must be computable (all piece dims/weights present) before pricing.

### 7.3 Timing

- **On blur:** format validations (email, phone, IATA, numeric).
- **On submit:** required + cross-field validations; focus first error.
- **Live (debounced, advisory):** Copilot validation — never blocks, only advises.

---

## 8. Derived calculations

All computed live and read-only:

| Output | Formula |
|---|---|
| Line volume (m³) | `L_cm · W_cm · H_cm · qty / 1,000,000` |
| Total pieces | `Σ qty` |
| Gross weight (kg) | `Σ (weightKgPerPc · qty)` |
| Total volume (m³) | `Σ line volume` |
| **Volumetric weight (kg)** | `Σ (L_cm · W_cm · H_cm · qty) / 6000` — IATA standard (≈167 kg/m³) |
| **Chargeable weight (kg)** | `max(grossWeight, volumetricWeight)` |

The chargeable weight drives the indicative price. Unit toggles convert display only (1 in = 2.54
cm; 1 lb = 0.453592 kg); stored SI values are authoritative.

---

## 9. User actions & outcomes

| Action | Precondition | Outcome |
|---|---|---|
| **Save draft** | Always | Persists partial record; status stays/becomes `Draft`; "Saved ✓ HH:MM". |
| **Switch mode** | Always | Recomputes required fields; preserves all entered data; updates section badges. |
| **Add / remove piece line** | ≥1 line must remain | Recomputes totals & chargeable weight live. |
| **Toggle special requirement** | Always | Expands/collapses its panel; includes/excludes its fields from validation; may update required documents (DG→MSDS). |
| **Upload document** | File ≤10MB, allowed mime | Tile → uploading → done; updates completeness & doc checklist. |
| **Apply Copilot suggestion** | Suggestion present | Applies the specific non-destructive change; re-prices if relevant; toast confirms; undoable. |
| **Request quote** | Rate-mode minimum met | Validates → `Quote requested` → pricing → `Quoted`; estimate populated. |
| **Create booking** | Booking validation passes | `Booked`; generates operational job; confirmation + notifications. |
| **Duplicate request** | Existing record | New `Draft` prefilled from source. |
| **Discard draft** | Draft exists | Confirmation modal; soft-delete with undo toast. |

Disabled actions always state the reason (e.g. *Create booking* disabled → "3 fields need
attention", click scrolls to first error).

---

## 10. AI Assistant (Cargo Copilot) behavior

The Copilot is **advisory and non-blocking**. It observes form state (debounced) and emits items in
three intents — **Validate, Complete, Suggest** — each with an icon, plain-language message, and an
optional one-tap action. Behavior detail:

| Capability | Trigger | Action offered | Guardrail |
|---|---|---|---|
| Validate info | Inconsistent/invalid fields | *Fix* (focus field) / *Improve* (rewrite) | Never edits without Apply |
| Detect missing data | Required/high-value gaps | *Add* / *Set* (focus field) | Lists, does not invent values |
| Suggest services | Commodity/transit patterns | *Apply* (toggle service) | Re-prices + toast on price-affecting changes |
| Recommend options | Cargo profile (oversized, etc.) | *Apply* / informational | Routing hints are advisory |
| Improve data quality | Vague description, raw address | *Improve* (suggested text shown first) | User confirms before replace |
| Highlight risks | DG, embargo, lane/customs | Informational + link to fix | Risks cannot be auto-dismissed silently |

**"Apply all safe suggestions"** applies only non-destructive items (focus/fill of empty fields,
enabling clearly-implied services); it never overwrites user-entered values or submits the form.
All applies are individually undoable. If the assistant is unavailable, the page remains fully
functional (graceful degradation — Copilot card shows an unobtrusive "assistant offline" note).

---

## 11. Error handling

| Scenario | Handling |
|---|---|
| **Field format invalid** | Inline `--err` border + helper text + icon on blur; `aria-describedby`. |
| **Required missing on submit** | `aria-live` error summary at top + sticky-bar count; focus first error; each summary item links to its field. |
| **Cross-field conflict** (e.g. DG commodity but DG panel off) | Both related fields flagged; Copilot offers a one-tap reconcile. |
| **Sea-only Incoterm chosen** | Hard validation error with explanation; suggests nearest air-applicable term. |
| **Identical origin/destination** | Inline error on destination. |
| **Upload too large / wrong type** | Tile → error state with reason + retry; other uploads unaffected. |
| **Pricing failed/timeout** | Estimate card shows error + retry; form stays editable; status not advanced. |
| **Submit network failure** | Non-blocking toast with retry; no data loss; record stays in prior state. |
| **Autosave failure** | "Couldn't save — retrying" indicator; local buffer retained; retry on reconnect. |
| **Concurrent edit** (two users) | Optimistic lock; on conflict, prompt to review/merge; never silently overwrite. |
| **Permission denied** (role) | Action hidden or disabled with tooltip; internal fields not rendered. |

Principle: **never lose user input**; errors are recoverable, explained in plain language, and
paired with a clear next step.

---

## 12. States & lifecycle

| Status | Meaning | Allowed transitions |
|---|---|---|
| **Draft** | Being created; autosaved | → Quote requested, → Booked (if all booking rules met), Discarded |
| **Quote requested** | Submitted for pricing | → Quoted, → Draft (revise) |
| **Quoted** | Price returned | → Booked, → Draft (revise → re-quote) |
| **Booked** | Committed shipment | → Job created, → Amendment requested |
| **Job created** | Handed to operations | → Amendment requested (controlled) |
| **Amendment requested** | Change after booking | → Booked (re-approved) |
| **Discarded** | Soft-deleted draft | (restorable via undo window) |

Revising inputs that affect price after `Quoted` invalidates the firm quote and returns to an
indicative state until re-quoted. Changes after `Booked` follow a controlled amendment flow (audit
trail, re-validation).

---

## 13. Notifications

| Event | Recipient | Channel |
|---|---|---|
| Quote requested | Forwarder/Ops queue | In-app + email |
| Quote ready | Requester (customer/forwarder) | In-app + email |
| Booking created | Shipper, Consignee contact, Ops | Email (with summary) |
| Document missing/rejected | Requester | In-app |
| Amendment requested | Ops + requester | In-app + email |

Notification content uses only role-appropriate fields (no internal cost/margin to customers).

---

## 14. Acceptance criteria

- [ ] User can create, autosave, and resume a Draft without data loss.
- [ ] Mode toggle correctly switches required-field sets without losing input.
- [ ] Chargeable weight = `max(gross, volumetric@1:6000)` updates live as pieces change.
- [ ] *Request quote* enables only when rate-minimum is met and produces an estimate.
- [ ] *Create booking* enables only when full booking validation passes; disabled state explains why.
- [ ] DG commodity forces DG panel + MSDS document; missing MSDS blocks booking.
- [ ] Service Type drives pickup/delivery address requirements correctly.
- [ ] All validations fire at the specified timing (blur/submit/live).
- [ ] Copilot is advisory only; no auto-submit; every Apply is undoable.
- [ ] Role-based visibility hides internal fields from customers.
- [ ] Page is fully usable by keyboard and meets WCAG 2.1 AA (see theme-compliance doc).
- [ ] Layout adapts per the responsive rules with no horizontal page scroll on mobile.

---

## 15. Future enhancements

- **AWB / HAWB pre-generation** and carrier eAWB integration from booking data.
- **Live carrier rate shopping** replacing the indicative estimate with bookable carrier options.
- **Automated HS-code & commodity classification** from the description (Copilot → tariff lookup).
- **OCR document intake** — extract invoice/packing-list fields to pre-fill cargo & parties.
- **Restricted-party & embargo screening** at submit (denied-party lists).
- **Lane intelligence** — historical transit times, capacity alerts, and CO₂ estimates per route.
- **Templates & repeat shipments** — save shipment profiles for one-click re-booking.
- **Multi-piece scan entry** — mobile barcode/dimensioner capture at the dock.
- **Quote comparison & approval workflow** with margin controls for forwarders.
- **Track-and-trace handoff** linking the booking to milestone tracking post-job-creation.

---

*See the [Page Mockup](../mockup/air-export-page-mockup.md) for layout and the
[Theme Compliance](../theme/air-export-theme-compliance.md) document for design-system token
conformance.*
