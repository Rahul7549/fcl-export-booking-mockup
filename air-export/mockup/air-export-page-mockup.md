# Air Export Request / Booking — Page Mockup

> **Service:** Air Export &nbsp;|&nbsp; **Type:** Request / Booking creation &nbsp;|&nbsp; **Status:** Mockup v1
> **Design system:** [02-design-system.md](../../../02-design-system.md) &nbsp;|&nbsp; **Quote spec:** [03-quote-creation-spec.md](../../../03-quote-creation-spec.md)
> **Last updated:** 2026-05-29

A single, responsive, enterprise-grade page for creating an Air Export request and converting
it into a booking. The design is grounded **entirely** in the existing
[Design System](../../../02-design-system.md) — indigo brand, slate neutrals, Lucide icons,
Tailwind layout + SCSS primitives. No new color, radius, or shadow language is introduced.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Design principles](#2-design-principles)
3. [Personas & jobs-to-be-done](#3-personas--jobs-to-be-done)
4. [Page anatomy](#4-page-anatomy)
5. [Full-page wireframe (desktop)](#5-full-page-wireframe-desktop)
6. [Section-by-section breakdown](#6-section-by-section-breakdown)
   - 6.1 [Page header & progress stepper](#61-page-header--progress-stepper)
   - 6.2 [Shipment Information](#62-shipment-information)
   - 6.3 [Route Information](#63-route-information)
   - 6.4 [Cargo Information](#64-cargo-information)
   - 6.5 [Shipper Information](#65-shipper-information)
   - 6.6 [Consignee Information](#66-consignee-information)
   - 6.7 [Special Requirements](#67-special-requirements)
   - 6.8 [Document Upload](#68-document-upload)
   - 6.9 [AI Assistant (Cargo Copilot)](#69-ai-assistant-cargo-copilot)
   - 6.10 [Summary sidebar & submit bar](#610-summary-sidebar--submit-bar)
7. [Component → design-system mapping](#7-component--design-system-mapping)
8. [Interaction & states](#8-interaction--states)
9. [Responsive behavior](#9-responsive-behavior)
10. [Accessibility](#10-accessibility)
11. [Design decisions & rationale](#11-design-decisions--rationale)

---

## 1. Overview

The **Air Export Request / Booking** page is the primary data-entry surface where a freight
forwarder, operations agent, or customer captures everything needed to (a) price an air export
shipment and (b) hand a clean, validated job to operations. It replaces the classic dense,
single-scroll ERP form with a **guided, card-based, progressively-disclosed** layout that keeps
cognitive load low while collecting a large amount of structured data.

A request moves through a clear lifecycle:

```
Draft ──▶ Quote requested ──▶ Quoted ──▶ Booked ──▶ Handed to Ops (job created)
```

The page supports two entry modes that share the same form:

| Mode | Goal | Who | What's mandatory |
|---|---|---|---|
| **Request a rate** | Get a quotation | Customer, Forwarder | Route + cargo + commodity (lightweight) |
| **Create booking** | Commit a shipment | Forwarder, Ops | Full shipper/consignee + documents + special handling |

The single page adapts the required fields to the chosen mode rather than forcing two separate
screens — see [§8](#8-interaction--states).

---

## 2. Design principles

1. **Progressive disclosure over a wall of fields.** Each domain is a `.card` section that can be
   collapsed once complete. Advanced/conditional fields (DG details, temperature range, insurance
   value) appear only when their trigger is on.
2. **One screen, one source of truth.** A persistent **summary sidebar** mirrors the running
   chargeable weight, route, and estimated price so the user never loses the big picture.
3. **The system does the math.** Chargeable weight, volumetric weight, and total volume are
   computed live from dimensions — never hand-keyed (see [§6.4](#64-cargo-information)).
4. **Tokens only.** Every surface, control, and status uses design-system `var(--…)` tokens and
   SCSS primitives (`.card`, `.btn`, `.pill`, `.badge`, `.kv`). No bespoke hex.
5. **Status = color + icon + text.** Validation, completeness, and AI signals are never color-alone.
6. **AI assists, never blocks.** The Cargo Copilot suggests, validates, and pre-fills, but the user
   stays in control; nothing auto-submits.
7. **Keyboard- and ops-efficient.** Tab order follows reading order, every control has a label and a
   visible focus ring, and power users can save a draft at any time.

---

## 3. Personas & jobs-to-be-done

| Persona | Primary job on this page | What "good" looks like |
|---|---|---|
| **Freight Forwarder** | Build a request, generate a quote, submit a booking | Fast entry, smart defaults, instant chargeable-weight & price feedback |
| **Operations Team** | Validate the request, create the operational job | Complete, conflict-free data; DG/temperature flags surfaced early |
| **Customer (Shipper)** | Submit requirements, upload docs, request a rate | Guidance, plain language, no jargon walls, clear progress |

Each persona sees the **same** page; visibility of internal-only fields (e.g. cost notes, ops
remarks) is governed by role — see Functional Spec §User Actions.

---

## 4. Page anatomy

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  APP TOP BAR (existing global nav)                                             │
├──────────────────────────────────────────────────────────────────────────────┤
│  PAGE HEADER  ── title · breadcrumb · mode toggle · status badge · actions     │
│  PROGRESS STEPPER  ── Shipment ▸ Route ▸ Cargo ▸ Parties ▸ Requirements ▸ Docs │
├───────────────────────────────────────────────┬──────────────────────────────┤
│  MAIN COLUMN  (form, max-w-3xl of the grid)    │  SIDEBAR  (sticky, summary)   │
│                                                │                              │
│  [ Shipment Information      card ]            │  [ Cargo Copilot       card ] │
│  [ Route Information         card ]            │  [ Shipment summary    card ] │
│  [ Cargo Information         card ]            │  [ Estimated price     card ] │
│  [ Shipper Information       card ]            │  [ Completeness        card ] │
│  [ Consignee Information     card ]            │                              │
│  [ Special Requirements      card ]            │  (sidebar is sticky on ≥lg)   │
│  [ Document Upload           card ]            │                              │
│                                                │                              │
├───────────────────────────────────────────────┴──────────────────────────────┤
│  STICKY SUBMIT BAR  ── Save draft · Request quote · Create booking · errors    │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Grid:** `mx-auto max-w-7xl px-4`. On `≥lg` the body is a 12-column grid: **8 cols** form / **4
cols** sticky sidebar (`grid-cols-12 gap-6`, sidebar `lg:col-span-4 lg:sticky lg:top-20`). Below
`lg`, the sidebar collapses under the form and the Copilot + summary become a single sticky
bottom sheet trigger.

---

## 5. Full-page wireframe (desktop)

```
┌────────────────────────────────────────────────────────────────────────────────────────────┐
│ Dashboard › Requests › New                                       ●Draft   [Save draft]  [⋯]  │
│                                                                                              │
│  ✈  Air Export Request                                  ┌─ Mode ───────────────────────┐    │
│      Create a request, get a quote, submit a booking    │ ( Request a rate )( Booking ) │    │
│                                                         └──────────────────────────────┘    │
│                                                                                              │
│  ①Shipment ─── ②Route ─── ③Cargo ─── ④Parties ─── ⑤Requirements ─── ⑥Documents  [Review ⑦] │
│  ●━━━━━━━━━━━━━●━━━━━━━━━━━○─────────────○────────────────○────────────────○                  │
├──────────────────────────────────────────────────────────┬───────────────────────────────────┤
│ MAIN (8 cols)                                             │ SIDEBAR (4 cols, sticky)          │
│                                                          │                                   │
│ ┌── ① Shipment Information ───────────────────[▾]──┐     │ ┌─ ✦ Cargo Copilot ────────────┐  │
│ │ Shipment Type*   [ Air Export        ▾]          │     │ │ ✓ Route looks valid          │  │
│ │ Service Type*    (A→A)(D→D)(D→A)(A→D)            │     │ │ ⚠ Add HS code for faster      │  │
│ │ Movement Type*   [ Loose / ULD       ▾]          │     │ │   customs clearance          │  │
│ │ Incoterm*        [ FCA               ▾]          │     │ │ 💡 Suggest: Express for       │  │
│ │ Commodity Type*  [ General cargo     ▾]          │     │ │   pharma at <72h transit     │  │
│ │ Cargo Description [__________________________]   │     │ │ [Apply all]   [Dismiss]      │  │
│ └──────────────────────────────────────────────────┘     │ └──────────────────────────────┘  │
│                                                          │                                   │
│ ┌── ② Route Information ──────────────────────[▾]──┐     │ ┌─ Shipment summary ───────────┐  │
│ │ Origin airport*  [JFK New York ✈   ▾]            │     │ │ Route   JFK → FRA            │  │
│ │ Dest. airport*   [FRA Frankfurt ✈  ▾]   ✈─────▶  │     │ │ Service Door → Door          │  │
│ │ ┌ Pickup (Door) ─────┐ ┌ Delivery (Door) ─────┐  │     │ │ Pieces  12                   │  │
│ │ │ Address  [______]  │ │ Address  [______]    │  │     │ │ Gross   840 kg              │  │
│ │ └────────────────────┘ └──────────────────────┘  │     │ │ Volume  4.10 m³             │  │
│ └──────────────────────────────────────────────────┘     │ │ Chargeable  840 kg ⓘ        │  │
│                                                          │ └──────────────────────────────┘  │
│ ┌── ③ Cargo Information ──────────────────────[▾]──┐     │                                   │
│ │ [ Pieces table: L×W×H · qty · wt · type ]        │     │ ┌─ Estimated price ────────────┐  │
│ │  + Add line     ▸ live totals strip below        │     │ │  $ 3,240  – $ 3,910          │  │
│ │ Pieces 12 · Gross 840kg · Vol 4.10m³ · Chg 840kg │     │ │  indicative · all-in air     │  │
│ └──────────────────────────────────────────────────┘     │ │  [See breakdown]             │  │
│                                                          │ └──────────────────────────────┘  │
│ ┌── ④ Shipper ──────────┐ ┌── ④ Consignee ───────┐      │                                   │
│ │ Company  [_________]  │ │ Company  [_________]  │      │ ┌─ Completeness ───────────────┐  │
│ │ Contact  [_________]  │ │ Contact  [_________]  │      │ │ ▓▓▓▓▓▓▓░░░ 72%               │  │
│ │ Email    [_________]  │ │ Email    [_________]  │      │ │ 3 required fields left        │  │
│ │ Phone    [_________]  │ │ Phone    [_________]  │      │ │ • Consignee email             │  │
│ │ Address  [_________]  │ │ Address  [_________]  │      │ │ • MSDS (DG on)                │  │
│ └───────────────────────┘ └───────────────────────┘      │ │ • Incoterm                    │  │
│                                                          │ └──────────────────────────────┘  │
│ ┌── ⑥ Special Requirements ───────────────────────┐     │                                   │
│ │ ☐ Dangerous goods  ☐ Temp-controlled            │     │                                   │
│ │ ☐ Express  ☐ Oversized  ☐ Insurance  ☐ Stackable │     │                                   │
│ │ ▸ (conditional panels expand below when toggled)  │     │                                   │
│ └──────────────────────────────────────────────────┘     │                                   │
│                                                          │                                   │
│ ┌── ⑦ Document Upload ────────────────────────────┐     │                                   │
│ │ [ Commercial invoice ][ Packing list ][ C/O ]    │     │                                   │
│ │ [ MSDS ][ + Additional ]   drag & drop zone      │     │                                   │
│ └──────────────────────────────────────────────────┘     │                                   │
├──────────────────────────────────────────────────────────┴───────────────────────────────────┤
│  ● 3 fields need attention        [ Save draft ]   [ Request quote ]   [ Create booking ▸ ]   │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Section-by-section breakdown

Each section is a `.card` with a `.section-head` (Lucide icon chip + `.section-title`) and a
collapse chevron. Section state is shown as a small `.badge` in the header: `Incomplete` (slate),
`Needs attention` (`--warn`), or `Complete` (`--ok`). Field labels use the **eyebrow** idiom
(uppercase, tracked, `--muted`). Required fields carry a `*` and an accessible `aria-required`.

Legend for field tables: **R** = required, **C** = conditional, **A** = auto-computed, **AI** =
Copilot can suggest/pre-fill.

### 6.1 Page header & progress stepper

```
Dashboard › Requests › New                              ●Draft   [Save draft]   [⋯ More]
✈  Air Export Request                       ┌ Mode ┐ ( Request a rate )( Create booking )
   Create a request, get a quote, book it   └──────┘
①Shipment ── ②Route ── ③Cargo ── ④Parties ── ⑤Requirements ── ⑥Documents ── ⑦Review
```

- **Breadcrumb** — text links, `--muted`, last crumb `--ink`.
- **Status badge** — `.badge--*` reflecting lifecycle (Draft / Quote requested / Quoted / Booked).
- **Mode toggle** — `.pill` group, single-select; switching reflows required-field rules.
- **Stepper** — built on the existing `.progress-rail` / `.pr-step` primitive. Steps are clickable
  anchors that scroll-spy to the matching card. State per step: `done` (✓, `--ok`), `progress`
  (filled, brand), `pending` (hollow, slate). The stepper is **navigation + status**, not a wizard
  gate — the user may fill sections in any order (single-page form).
- **`[⋯ More]`** — overflow menu (`.menu-panel`): Duplicate request, Import from previous shipment,
  Discard draft.

### 6.2 Shipment Information

The "what kind of shipment" foundation. Driving choices here (Service Type, Commodity) cascade to
later sections.

```
┌── ① Shipment Information ─────────────────────────────────── ●Complete  [▾] ┐
│ SHIPMENT TYPE *      [ Air Export                                       ▾]    │
│ SERVICE TYPE *       ( Airport→Airport )( Door→Door )( Door→Airport )( A→D )  │
│ MOVEMENT TYPE *      [ Loose / Build-up (ULD)                           ▾]    │
│ INCOTERM *           [ FCA — Free Carrier                               ▾]    │
│ COMMODITY TYPE *     [ General cargo                                    ▾]    │
│ CARGO DESCRIPTION    [ e.g. 12 cartons electronic components, palletised ]   │
└──────────────────────────────────────────────────────────────────────────── ┘
```

| Field | Type | Flags | Notes / options |
|---|---|---|---|
| Shipment Type | Select | R | Fixed to **Air Export** here; select kept for parity with other flows |
| Service Type | Radio-card / segmented | R, AI | Airport→Airport, Door→Door, Door→Airport, Airport→Door. Drives which **Route** address fields are required ([§6.3](#63-route-information)) |
| Movement Type | Select | R | Loose, Palletised, ULD (build-up), Back-to-back |
| Incoterm | Select | R, AI | Air-relevant: EXW, FCA, CPT, CIP, DAP, DDP, DPU. Copilot warns on sea-only terms (FOB/CFR/CIF) |
| Commodity Type | Select | R, AI | General cargo, Perishables, Pharma/Healthcare, Dangerous goods, Live animals, Valuables, Electronics. Selecting DG/Pharma/Perishables auto-suggests the matching **Special Requirement** toggle |
| Cargo Description | Text (multiline) | AI | Free text; Copilot can draft from commodity + pieces and flags vague descriptions ("samples", "gifts") that delay customs |

**Cascades:** `Door→*` enables pickup address; `*→Door` enables delivery address; Commodity =
Dangerous goods turns on the DG panel and makes **MSDS** a required document.

### 6.3 Route Information

```
┌── ② Route Information ─────────────────────────────────────── ●Complete  [▾] ┐
│ ORIGIN AIRPORT *        [ JFK · New York JFK Intl ✈            ▾]             │
│ DESTINATION AIRPORT *   [ FRA · Frankfurt am Main ✈            ▾]   JFK ✈▶ FRA│
│ ┌ Pickup (Door)  shown for Door→* ─────┐ ┌ Delivery (Door) for *→Door ─────┐  │
│ │ ADDRESS *   [ 21 Hudson Yards…    ]  │ │ ADDRESS *   [ Hanauer Landstr…]  │  │
│ │ CITY / ZIP  [ New York ][ 10001 ]    │ │ CITY / ZIP  [ Frankfurt ][60314] │  │
│ │ READY DATE  [ 2026-06-03 ]           │ │ NOTES       [ dock 4, 08–16h ]   │  │
│ └──────────────────────────────────────┘ └──────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Field | Type | Flags | Notes |
|---|---|---|---|
| Origin airport | Async search-select | R, AI | IATA 3-letter + city; typeahead on code or city. Shows airport ✈ chip |
| Destination airport | Async search-select | R, AI | Same; Copilot validates pair is serviceable & not identical |
| Pickup address | Address group | C (Door→*) | Street, city, ZIP, country, **ready date** |
| Delivery address | Address group | C (*→Door) | Street, city, ZIP, country, delivery notes |

The **route ribbon** (`JFK ✈▶ FRA`) is a `.chip--mode` echoing the selected airports; it also
appears in the sidebar summary. Conditional address cards animate in/out (height + fade) so the
section never shows irrelevant fields.

### 6.4 Cargo Information

The most computation-heavy section. A **pieces table** captures dimensions per line; totals and
**chargeable weight** are derived live — never typed.

```
┌── ③ Cargo Information ─────────────────────────────────────── ●Complete  [▾] ┐
│  PACKAGES                                                                     │
│  ┌────┬───────┬────────┬────────┬────────┬───────────┬──────────┬─────────┐  │
│  │ #  │ Qty   │ L (cm) │ W (cm) │ H (cm) │ Wt/pc(kg) │ Pkg type │  Vol m³ │  │
│  ├────┼───────┼────────┼────────┼────────┼───────────┼──────────┼─────────┤  │
│  │ 1  │ [10]  │ [120]  │ [80]   │ [100]  │ [ 60 ]    │ [Pallet▾]│  0.96·10│  │
│  │ 2  │ [ 2]  │ [ 60]  │ [40]   │ [ 50]  │ [ 30 ]    │ [Carton▾]│  0.12·2 │  │
│  │ +  Add line                                                              │  │
│  └────┴───────┴────────┴────────┴────────┴───────────┴──────────┴─────────┘  │
│                                                                              │
│  ┌ Live totals ───────────────────────────────────────────────────────────┐ │
│  │ Pieces 12 │ Gross 840 kg │ Volume 4.10 m³ │ Volumetric 683 kg │         │ │
│  │ CHARGEABLE WEIGHT  840 kg  ⓘ max(gross, volumetric · IATA 1:6000)        │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Field | Type | Flags | Notes |
|---|---|---|---|
| Qty (per line) | Number | R | Pieces per identical line |
| L / W / H | Number (cm) | R | Per-piece dimensions; unit toggle cm/in at section level |
| Weight / piece | Number (kg) | R | Per-piece gross weight; unit toggle kg/lb |
| Package type | Select | — | Carton, Pallet, Crate, Drum, Bundle, Loose, ULD |
| Volume (line) | — | A | `L·W·H·qty / 1,000,000` m³ |
| Total pieces | — | A | Σ qty |
| Gross weight | — | A | Σ (wt/pc · qty) |
| Total volume | — | A | Σ line volume |
| Volumetric weight | — | A | `volume_cm³ / 6000` (IATA 167 kg/m³) |
| **Chargeable weight** | — | A | `max(gross, volumetric)`; the figure that prices the shipment |

The **chargeable weight** value is emphasized (display weight, `--ink`) with an info `ⓘ` tooltip
explaining the IATA 1:6000 rule. A unit toggle (cm/in, kg/lb) lives in the section header and
converts displayed values without changing stored SI values.

### 6.5 Shipper Information

```
┌── ④ Shipper Information ──────────────────────────── ●Needs attention  [▾] ┐
│ COMPANY *        [ Acme Components LLC                       ]  ◾ from CRM ▾ │
│ CONTACT PERSON   [ Dana Wright                              ]               │
│ EMAIL *          [ dana@acme.com                            ]               │
│ PHONE            [ +1 212 555 0148                          ]               │
│ ADDRESS          [ 21 Hudson Yards, New York, NY 10001, US  ]               │
│ ☐ Same as pickup address                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

| Field | Type | Flags | Notes |
|---|---|---|---|
| Company | Search-select / text | R, AI | Typeahead against saved contacts/CRM; free text allowed |
| Contact person | Text | — | |
| Email | Email | R | Validated format; used for quote/booking notifications |
| Phone | Tel | — | International format hint |
| Address | Text (multiline) | — | "Same as pickup" prefill toggle |

Selecting a saved company pre-fills the remaining fields (Copilot "from CRM" affordance).

### 6.6 Consignee Information

Mirrors Shipper exactly (same field set, validations, and CRM prefill), with a **"Same as
delivery address"** convenience toggle. Shipper and Consignee render **side-by-side** on `≥md`
(2-col) and **stacked** below — see [§9](#9-responsive-behavior).

| Field | Type | Flags | Notes |
|---|---|---|---|
| Company | Search-select / text | R, AI | CRM typeahead |
| Contact person | Text | — | |
| Email | Email | R | |
| Phone | Tel | — | |
| Address | Text (multiline) | — | "Same as delivery" prefill |

### 6.7 Special Requirements

A row of `.pill` toggles. Each ON toggle expands a **conditional panel** (progressive disclosure)
that collects only the data that requirement needs. Off = panel hidden and its fields excluded
from validation.

```
┌── ⑤ Special Requirements ──────────────────────────────────────────[▾] ┐
│ ( ⚠ Dangerous goods ) ( ❄ Temp-controlled ) ( ⚡ Express )               │
│ ( ⬚ Oversized )       ( 🛡 Insurance )        ( ▦ Stackable )            │
│                                                                          │
│ ▸ Dangerous goods  (expanded)                                            │
│   UN NUMBER* [UN1263]  CLASS* [3 ▾]  PACKING GRP [II ▾]  PROPER NAME* […]│
│   ⚠ MSDS document becomes required →  jump to Documents                  │
│                                                                          │
│ ▸ Temp-controlled (expanded)                                             │
│   RANGE* [ +2°C ]–[ +8°C ]   PACKAGING [ Active / Passive ▾ ]            │
└──────────────────────────────────────────────────────────────────────────┘
```

| Requirement | Conditional fields | Flags | Downstream effect |
|---|---|---|---|
| Dangerous goods | UN number, hazard class, packing group, proper shipping name | R when on | **MSDS** document required; IATA DGR note; Copilot risk flag |
| Temperature controlled | Min/max °C, active/passive packaging | R when on | Pharma/cold-chain handling code; suggests Express |
| Express | (none) | — | Re-prices for fastest routing; SLA note in summary |
| Oversized | Oversize reason / max single-piece dims | — | Triggers ULD/freighter check by Copilot |
| Insurance required | Insured value + currency | R when on | Adds insurance line to estimate |
| Stackable | (toggle only — yes/no) | — | Affects volumetric / build-up assumptions |

### 6.8 Document Upload

A document **checklist** drives uploads, so the user always knows what's expected. Each row is a
dropzone tile showing required/optional state, accepted types, and upload progress.

```
┌── ⑥ Document Upload ────────────────────────────────────────────────[▾] ┐
│ ┌ Commercial invoice * ─┐ ┌ Packing list * ──────┐ ┌ Certificate of origin┐ │
│ │ ✓ invoice_889.pdf     │ │ ⬆ drag & drop / browse│ │ ⬆ optional           │ │
│ │   1.2 MB · [view][✕]  │ │   PDF, JPG, PNG ≤10MB │ │                      │ │
│ └───────────────────────┘ └───────────────────────┘ └──────────────────────┘ │
│ ┌ MSDS  (required: DG on)┐ ┌ + Additional attachments ──────────────────────┐ │
│ │ ⚠ missing             │ │  drag files here or browse                       │ │
│ └───────────────────────┘ └──────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────────────┘
```

| Document | State | Notes |
|---|---|---|
| Commercial invoice | Required (booking) | PDF/JPG/PNG ≤ 10 MB |
| Packing list | Required (booking) | |
| Certificate of origin | Optional | Required for some lanes — Copilot flags by destination |
| MSDS | Conditional | Required when Dangerous goods is on |
| Additional attachments | Optional | Multi-file; e.g. export licence, fumigation cert |

Tiles show four states: **empty** (dropzone), **uploading** (`.skeleton` + progress), **done**
(✓ filename, view/remove), **error** (`--err`, retry). Drag-over highlights the tile with the
brand focus ring.

### 6.9 AI Assistant (Cargo Copilot)

A premium, **always-visible** sidebar card (top of the sidebar on `≥lg`; a bottom-sheet on
mobile). It reacts live to form state and groups its output into three intents, each a row with an
icon, plain-language message, and a one-tap action.

```
┌─ ✦ Cargo Copilot ───────────────────────────────────┐
│  Powered by your shipment data · live                │
│                                                      │
│  VALIDATE                                            │
│  ✓ JFK → FRA is a serviceable air lane              │
│  ✗ Consignee email missing            [Fix]         │
│  ⚠ Description "samples" may delay customs [Improve] │
│                                                      │
│  COMPLETE                                            │
│  💡 HS code not set — add for faster clearance [Add] │
│  💡 Door→Door selected but no pickup date     [Set]  │
│                                                      │
│  SUGGEST                                             │
│  ✦ Pharma + 2–8°C → enable Temp-controlled   [Apply]│
│  ✦ Transit-critical? Express saves ~36h      [Apply]│
│  ⚠ Risk: DG class 3 needs MSDS + DGR check          │
│                                                      │
│  [ Apply all safe suggestions ]      [ Dismiss ]    │
└──────────────────────────────────────────────────────┘
```

**Capabilities (mapped to the brief):**

| Intent | Behavior | Example |
|---|---|---|
| Validate shipment info | Cross-checks fields for correctness & consistency | Identical origin/dest; sea-only Incoterm on air; invalid email |
| Detect missing data | Surfaces required + high-value optional gaps | Missing HS code, pickup date, consignee email |
| Suggest services | Recommends Service Type / Special Requirements | Pharma → Temp-controlled; tight transit → Express |
| Recommend shipment options | Routing / packaging hints | Oversized → freighter/ULD; consolidate cartons to a pallet |
| Improve data quality | Rewrites vague descriptions, normalizes addresses | "gifts" → itemized commercial description |
| Highlight risks | DG, embargo, customs, lane-restriction flags | DG class 3 documentation; restricted commodity for lane |

**Design notes:** the card uses `data-accent="indigo"` for the brand top bar; each row is
color + icon + text (✓ `--ok`, ✗ `--err`, ⚠ `--warn`, 💡/✦ brand). Actions are `.btn-sm
.btn-soft-primary`. "Apply all" only applies **non-destructive** suggestions; service changes that
affect price always re-render the estimate and show a toast. The Copilot **never** submits or
overwrites a user-entered value without an explicit Apply.

### 6.10 Summary sidebar & submit bar

**Sidebar cards (below the Copilot, `≥lg` sticky):**

- **Shipment summary** — `.kv` list: Route, Service, Pieces, Gross, Volume, **Chargeable weight**.
  Mirrors live form state; the single place to confirm the "shape" of the shipment.
- **Estimated price** — indicative all-in range (`$3,240 – $3,910`) with a `[See breakdown]` link
  opening a modal (air freight + fuel/security + pickup/delivery + insurance). Clearly labeled
  *indicative* until a firm quote is returned.
- **Completeness** — a progress meter (`.progress-rail` style) + a checklist of remaining required
  fields, each a deep link that scrolls to and focuses the field.

**Sticky submit bar (bottom):**

```
● 3 fields need attention        [ Save draft ]   [ Request quote ]   [ Create booking ▸ ]
```

- **Save draft** — `.btn-secondary`; always enabled; persists partial state.
- **Request quote** — `.btn-soft-primary`; enabled once rate-mode minimum is met.
- **Create booking** — `.btn-primary`; enabled only when booking validation passes; otherwise
  disabled with the error count as the reason (click scrolls to first error).

---

## 7. Component → design-system mapping

Every region maps to an existing primitive from [§3 of the design system](../../../02-design-system.md#3-component-primitives-catalog).
No new primitives are required; two **form** primitives (field wrapper, dropzone) are the only
additions and are exactly the gaps called out in design-system §6.

| Region | Primitive(s) | Tokens |
|---|---|---|
| Section container | `.card`, `.card[data-accent="indigo"]` | `--r-card`, `--shadow-card`, `--border` |
| Section header | `.section-head` + `.ico`, `.section-title` | brand icon chip |
| Field label | Eyebrow type idiom | `text-[11px] uppercase tracking-wide font-bold text-slate-500` |
| Text / number / email input | **`.field` + `.input`** (new `_forms.scss`, design-system §6) | `--r-control`, `--border`, `--focus-ring` |
| Select / airport search | wrap `app-custom-dropdown` / `app-normal-dropdown` | `.dropdown-list` / `.dropdown-item` |
| Service Type / Incoterm picker | segmented **radio-card** group (design-system §6) | brand active fill `--brand-50` |
| Special-requirement toggles | `.pill` / `.pill--on` | brand on-state |
| Section status / lifecycle | `.badge--*`, `.chip-status--*` | semantic `--ok/--warn/--err/--info` |
| Stepper | `.progress-rail` / `.pr-step` | `done/progress/pending` states |
| Pieces table totals | `.kv` + `.chip-strong` | dense values |
| Copilot card | `.card[data-accent="indigo"]` + `.btn-soft-primary` | brand accent |
| Summary / completeness | `.kv`, `.progress-rail`, `.badge` | |
| Buttons | `.btn-primary`, `.btn-secondary`, `.btn-soft-primary`, `.btn-ghost`, `.btn-sm` | |
| Upload tiles | **dropzone** (new) on `.card--tight`; `.skeleton` while uploading | `--err` error state |
| Toasts / modal (breakdown) | toast service + modal shell (design-system §6) | `--shadow-pop` |

---

## 8. Interaction & states

**Mode switching** — toggling *Request a rate* ⇄ *Create booking* recomputes required fields with
an animated update of section badges; no data is lost. Rate mode dims (not hides) booking-only
sections (full parties, documents) and labels them *Optional for a quote*.

**Validation timing** — validate on **blur** for format errors; validate **required** on submit
attempt; the Copilot validates **live** (debounced) but only *advises*. First invalid field on a
failed submit receives focus and a scroll-into-view.

**Field states** — default, hover, focus (`--focus-ring`), filled, **error** (`--err` border +
helper text + icon), **disabled** (conditional fields when their toggle is off), **success** tick
on async-validated fields (airport, email).

**Section states** — Incomplete (slate badge), Needs attention (`--warn`, ≥1 error), Complete
(`--ok`). Completing a section collapses it to a one-line summary the user can re-expand.

**Auto-save** — drafts save on a debounce and on navigation away; a subtle "Saved ✓ HH:MM" appears
near the status badge. Save uses the `.badge` micro-state, never a blocking spinner.

**Empty / loading / error** — async selects show `.skeleton`; the price card shows a shimmer while
recalculating; network failure on submit shows a non-blocking error toast with retry and keeps the
form intact.

**Destructive** — Discard draft and Remove document require confirmation (modal shell); both are
reversible via undo toast where feasible.

---

## 9. Responsive behavior

A **single** responsive layout (no separate mobile/tablet mockups), driven by the design-system
breakpoints (`sm 640 · md 768 · lg 1024 · xl 1280 · 2xl 1536`).

| Range | Layout |
|---|---|
| **`< sm` (mobile)** | Single column. Cards full-width, `px-4`. Shipper/Consignee **stack**. Pieces table becomes **stacked cards** per line (label : value), not a horizontal table. Sidebar content (Copilot, summary, completeness) moves into a **bottom sheet** opened by a sticky "✦ Copilot · 72%" pill. Submit bar is a sticky footer; primary action full-width, secondary in an overflow. Stepper becomes a compact horizontal scroll-spy chip strip. |
| **`sm–md`** | Still single column for form; the in-section 2-col groups (address pickup/delivery, shipper/consignee) **stay stacked** until `md`. |
| **`md` (tablet)** | Form remains one main column but **paired cards go 2-up** (Shipper ∣ Consignee, Pickup ∣ Delivery). Pieces table shows as a real scrollable table. Sidebar still below the form (not yet side-by-side). |
| **`lg` (laptop)** | The signature layout: **8-col form / 4-col sticky sidebar**. Copilot + summary + completeness become a persistent rail (`lg:sticky lg:top-20`). Submit bar spans the content width. |
| **`xl / 2xl` (desktop)** | Same 8/4 split inside `max-w-7xl`; extra width goes to generous gutters and roomier pieces table. Content never exceeds `max-w-7xl` (centered). |

**Rules applied:** container `mx-auto max-w-7xl`; section rhythm `mt-6`; card padding
`1–1.25rem`; touch targets ≥ 44px on coarse pointers; the pieces table never forces horizontal
page scroll — it virtualizes/stacks instead.

---

## 10. Accessibility

Accessibility is part of "done" (design-system §4.9). Targets **WCAG 2.1 AA**.

- **Semantics** — the form is a real `<form>`; each card is a `<section>` with an `<h2>` and
  `aria-labelledby`; the stepper is a `<nav aria-label="Request steps">` with `aria-current="step"`.
- **Labels** — every control has a visible `<label for>`; eyebrow labels are real labels, not
  decorative text. Required fields use `aria-required` and the `*` is announced.
- **Errors** — error text is tied via `aria-describedby`; the on-submit summary is an
  `aria-live="assertive"` region that lists each error as an in-page link.
- **Focus** — visible `:focus-visible` ring using `--focus-ring`; logical tab order matches reading
  order; conditional panels move focus to their first field when expanded.
- **Status not color-alone** — every badge/Copilot row pairs color with an icon and text.
- **Contrast** — body text `--ink` on `--panel` and `--muted` on white both meet AA; brand
  `--brand-primary` on white for actions meets AA for UI/large text (verify any small white-on-brand
  text per theme-compliance doc).
- **Copilot** — suggestions are reachable by keyboard; "Apply" actions are buttons with descriptive
  accessible names ("Apply suggestion: enable Temperature controlled").
- **Live regions** — chargeable weight, completeness %, and price updates are announced politely
  (`aria-live="polite"`) so screen-reader users hear recalculations.
- **Motion** — collapse/expand and route animations respect `prefers-reduced-motion`.

---

## 11. Design decisions & rationale

| Decision | UI/UX rationale | Freight rationale |
|---|---|---|
| **Single page + sticky sidebar** (not a multi-step wizard) | Power users (forwarders/ops) hate gated wizards; they jump around and bulk-edit | Ops need the whole picture to validate consistency (route ⇄ Incoterm ⇄ DG) |
| **Card sections with collapse** | Progressive disclosure tames a 40+ field form; reduces cognitive load | Each card maps to a real operational data domain (route, cargo, parties, compliance, docs) |
| **One Request/Booking page, mode toggle** | Avoids duplicate screens & divergent logic | A quote and a booking share 90% of the same shipment data; only commitment differs |
| **Auto-computed chargeable weight** | Removes the #1 manual error; instant feedback | Chargeable weight (IATA 1:6000) is what actually prices air freight; hand-keying it is error-prone |
| **Pieces table → stacked cards on mobile** | Horizontal tables are unusable on phones | Field/warehouse users often enter dims on mobile at the dock |
| **Conditional panels for special requirements** | Show only relevant fields; less clutter | DG/temp/insurance each carry distinct mandatory data & documents |
| **Document checklist (not a generic uploader)** | Users know exactly what's expected and what's missing | Air export compliance hinges on the right docs (invoice, packing list, MSDS, C/O) |
| **Always-on Cargo Copilot, advisory only** | Premium, intelligent feel without taking control | Surfaces compliance/lane risks early — the costliest mistakes in air export |
| **Tokens & primitives only** | Guaranteed visual consistency with the rest of the app | — |
| **Status = color + icon + text** | Accessibility + matches established app pattern | DG/temperature flags must be unmistakable |

---

*This mockup is the visual & structural source of truth. See the
[Functional Specification](../doc/air-export-functional-specification.md) for behavior, validation,
and workflow, and the [Theme Compliance](../theme/air-export-theme-compliance.md) document for
token-level design-system conformance.*
