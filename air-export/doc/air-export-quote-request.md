# Air Export Quote Request (RFQ) — Design & UX Documentation

> Mockup: [`../mockup/air-export-quote-request.html`](../mockup/air-export-quote-request.html)
> Shared theme: [`../theme/theme.css`](../theme/theme.css)
> Design language parent: [`../../fcl-export-booking.html`](../../fcl-export-booking.html)
> Token source of truth: `src/styles.scss :root` · system reference: [`../../../02-design-system.md`](../../../02-design-system.md)

A conversion-focused, single-purpose page that lets a US exporter request an **air freight
export quote** in three short steps, collecting only the minimum a forwarder needs to price the
move. It is a static prototype (HTML + Tailwind CDN + Lucide), visually identical to the FCL
Export Booking mockup — same tokens, primitives, layout shell, and interaction patterns.

---

## 1. Theme consistency — reuse, not reinvention

The page invents **no new design language**. Everything visual comes from the primitives already
shipped in `fcl-export-booking.html`, lifted verbatim into a single shared stylesheet
(`../theme/theme.css`) and `<link>`-ed in. The FCL page inlined those same rules in a `<style>`
block; extracting them once means both mockups stay in lock-step and future mockups in this folder
can link the same file.

Reused as-is:

| Concern | Source |
|---|---|
| Brand / semantic colors | `:root` tokens (`--brand-primary #2A3E8A`, `--brand-secondary #78B947`, `--ok/--info/--warn/--err`, slate neutrals) — mirror of `src/styles.scss` |
| Cards & section slabs | `.card`, `.card--tight`, `.section-accent`, `.section-head` + icon chip |
| Buttons | `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.btn-soft-primary`, `.btn-sm` |
| Status / mode | `.badge--active/--atrisk/--delivered`, `.chip--mode`, `.pill / .pill--on` |
| Wizard tracker | `.progress-rail`, `.pr-step`, `.pr-dot`, `.pr-line` |
| Form controls | `.field-label`, `.input`, `.select`, `.input-group`, `.input-addon`, `.unit-select`, `.cc-select` |
| Choice controls | `.radio-card`, `.switch` + `.toggle-row`, `.stepper` |
| Summary | `.sum-row`, sticky `.sticky-summary`, `.flash` highlight |
| Icons | Lucide via `data-lucide`, re-rendered after every DOM update (matches the app's `ngAfterViewChecked` convention) |

Layout uses Tailwind utilities (grid/flex/spacing); **identity** stays in the SCSS primitives —
exactly the rule the design system mandates ("Layout = Tailwind, identity = SCSS primitives").

---

## 2. UX approach

### Four steps, mirroring the FCL Export wizard 1:1
The page uses the **same 4-step wizard as FCL Export** so the two flows feel identical and a user
who has booked FCL already knows the rhythm. Each air step is the direct analogue of its FCL
counterpart:

| Step | FCL Export | Air Export |
|---|---|---|
| 1 | Service & Cargo | **Service & Cargo** — service type + commodity & terms |
| 2 | Route | **Route** — origin/destination airports + schedule |
| 3 | Containers & Customs | **Pieces & Services** — pieces/weight (chargeable-weight calc) + customs & service add-ons |
| 4 | Parties & Review | **Parties & Review** — shipper, consignee & notify-party cards, notes, contact details, final recap |

> Earlier this flow was condensed to 3 steps (cargo folded into step 1, services into step 2), but
> that overloaded the first two steps. Splitting cargo dimensioning and services into their own
> step — exactly where FCL puts "Containers & Customs" — keeps each step short and the cross-mode
> experience consistent.

A user can jump directly to any step via the clickable tracker.

### Progressive disclosure
Fields appear only when relevant, so the default form stays short:

- **Service type** (step 1) reshapes the Route step — "Door" legs reveal pickup-ZIP /
  delivery-address fields and auto-tick the matching *Pickup / Delivery required* service toggles
  in step 3 (single source of truth, no conflicting answers).
- **Dangerous goods & Temperature-controlled** use the **same toggle-switch capture as FCL
  Export** (the established, more polished pattern): two `toggle-row` switches in the Commodity
  card — DG (flame icon) and Temperature-controlled (snowflake icon). Switching one on highlights
  the row and reveals only its relevant fields — UN number + IATA DG class for DG; set temperature
  + cold-chain method for temperature-controlled. The *Shipment type* select still auto-suggests
  the matching toggle (choosing *Perishable* turns on temperature control; *Dangerous goods* turns
  on DG), exactly mirroring FCL's reefer auto-suggest.
- **Notify party** follows FCL's pattern — a "Same as consignee" toggle (on by default) that
  reveals notify company/country only when switched off.
- **Dimensions** are explicitly optional and visually de-emphasized — but if supplied they upgrade
  the quote from gross-weight-only to true chargeable weight.

### Cognitive-load reducers
- **Smart defaults & examples** — a pre-filled sample piece line, realistic placeholders
  (`Electronic components`, `UN3480`, `PO-2026-0042`), Incoterms ordered by air-export frequency
  (FCA / EXW / CPT first), customs + AES on by default.
- **Required fields** are marked with a red `*`; everything else is genuinely optional.
- **Steppers and pill toggles** replace free-text where a constrained choice is clearer
  (pieces, stackable yes/no, weight unit).
- **One persistent summary** (sticky on desktop, stacks below on mobile) keeps the running
  total visible without scrolling back.

---

## 3. Logistics workflow assumptions

The form models a **US-outbound air export RFQ** handed to a forwarder's pricing desk.

- **Direction is fixed** (US → overseas), matching the FCL export scope and the existing
  `air-export-quote*` modules in the app. Origin lists US IATA gateways (JFK/LAX/ORD/ATL/MIA/DFW/
  SFO/SEA); destination lists common overseas airports.
- **Service scope** is expressed as the four standard freight legs — Airport→Airport,
  Door→Airport, Airport→Door, Door→Door — the air analogue of the FCL Port/Door matrix.
- **Chargeable weight is the pricing unit.** Air freight bills on the *greater* of actual gross
  weight and volumetric weight. The mockup computes this live using the **IATA factor**
  (volume cm³ ÷ 6000 = volumetric kg, i.e. 1 m³ ≈ 167 kg) and rounds up to the next 0.5 kg per
  airline convention. This is the single most important number on the page and is styled as the
  brand-tinted key metric.
- **AES / EEI filing** is surfaced as a required-by-default service with an `AES / EEI` badge —
  US exports above the de-minimis threshold legally require Electronic Export Information.
- **DGR / perishable handling** follows IATA categories (DG class incl. Class 9 lithium batteries;
  perishable temperature + max transit) rather than the IMDG/reefer terms used for ocean.

### Required-fields rationale (the minimum to quote air freight)

| Field | Why a forwarder needs it |
|---|---|
| Service type | Determines which legs (and surcharges) to price |
| Cargo description + Shipment type | Commodity acceptance, DG/perishable surcharges, screening |
| Incoterms | Who pays which leg; where liability transfers |
| Pieces + gross weight | Base rate driver |
| Dimensions *(optional)* | Promotes to chargeable weight; many lanes are volume-limited |
| Origin / destination airport | Lane, carrier, transit, rate sheet |
| Cargo ready date | Capacity / space booking and validity window |
| Shipper + consignee + contact | To return the quote and run compliance screening |
| Notify party | Defaults to "same as consignee"; only expanded if a third party must be notified |

Declared value, HS codes, MAWB/HAWB, and full address books are intentionally **deferred to
booking** — they are not needed to *price* a shipment and would hurt conversion here. The
shipper / consignee / notify-party cards mirror FCL Export's parties layout, and contact details
live in their own card (under Additional Notes) exactly as in FCL — so a user moving between
shipment modes sees an identical parties-and-contact step.

---

## 4. Responsive strategy

Mobile-first, validated at 360 px → 1440 px+:

- **Shell**: `max-w-7xl` centered container; a 12-column grid splits 8 (form) / 4 (summary) on
  `lg`, and collapses to a single column below — the summary stacks **below** the form on mobile so
  the primary task leads.
- **Sticky summary** only sticks at `lg+` (`@media (min-width:1024px)`); on small screens it scrolls
  inline, avoiding a viewport-eating fixed panel.
- **Field grids** step down responsively: service cards `1 → 2 → 4` columns
  (`sm:grid-cols-2 xl:grid-cols-4`); paired inputs `1 → 2`; piece-line internals use a 12-col grid
  on `sm` and a 2-col stack on mobile.
- **Step tracker** sits in an `overflow-x-auto` card so it never wraps awkwardly on narrow screens.
- Touch targets (steppers, switches, pills, radio-cards) are ≥ 36 px tall.

---

## 5. Accessibility considerations

- **Semantic HTML** — real `<button>`, `<select>`, `<input>`, `<textarea>`, `<label>`; the segmented
  control uses `role="tablist"` / `aria-selected`.
- **Labels & names** — every control has a visible `.field-label` or an `aria-label` (piece-line
  L/W/H inputs, weight unit, country code).
- **Status is never color-alone** — badges/chips pair color with icon + text (design-system rule 5).
- **Visible focus** — global `:focus-visible` ring driven by `--focus-ring`; switches show a focus
  ring on `:focus-visible`.
- **Required fields** marked with a `*` that is also part of the label text, not color-only.
- **Keyboard** — the entire wizard (tracker, nav, toggles, steppers) is operable without a mouse.
- **Contrast** — ink `#0f172a` on white and brand `#2A3E8A` on `--brand-50` meet WCAG AA.

Known prototype gaps (acceptable for a mockup, listed for the production build): radio-cards should
become a keyboard-navigable `radiogroup`; live regions should announce the chargeable-weight
recalculation; client-side validation messaging is not yet wired.

---

## 6. Future enhancements

- **Live rate preview** — replace "On request" with an indicative band once a rate engine exists.
- **Airport autocomplete** — type-ahead IATA search instead of curated `<select>`s.
- **Saved address book / parties** — pull shipper/consignee from prior shipments.
- **Document upload** — packing list / commercial invoice / MSDS (for DG) at RFQ time.
- **Unit memory** — remember kg vs lbs and cm vs in per user.
- **Quick-quote mode** — the segmented toggle is stubbed; wire it to a one-screen variant
  (route + chargeable weight only).
- **Multi-currency & Incoterm-aware fields** — show insurance value only when the Incoterm implies
  seller-borne risk.
- **Validity countdown** — quotes expire; show the window on the results page.

---

## 7. Recommended backend / API integrations

This is a docs mockup; the production screen lives in the Angular app
(`src/app/air-export-quote/`, ng-zorro-antd) and would bind to:

| Capability | Integration |
|---|---|
| Airport / port reference | UN/LOCODE + IATA dataset (reuse the dashboard origin/destination filter source) |
| Rate / lane lookup | Forwarder TMS or carrier rate API (e.g. WebCargo / cargo.one) for indicative bands |
| Chargeable-weight calc | Server-confirm the client estimate (gross vs volumetric ÷ 6000) at submit |
| Compliance screening | Denied-party / OFAC screening on shipper + consignee |
| Export filing | AES/EEI (ACE AESDirect) hand-off for the customs service |
| DG validation | IATA DGR list validation for UN number ↔ class consistency |
| Quote persistence | Existing `air-export-quote` create endpoint → returns a quote ref for `quote-results.html` |
| Notifications | Email/SMS quote-ready (the "4 business hours" SLA) |

### Suggested submit payload (shape)

```jsonc
{
  "mode": "AIR", "direction": "EXPORT",
  "service": "A2A",                       // A2A | D2A | A2D | D2D
  "incoterm": "FCA",
  "commodity": { "description": "Electronic components", "type": "GENERAL", "stackable": true },
  "dangerousGoods":  { "enabled": false, "unNumber": null, "iataClass": null },
  "temperatureControlled": { "enabled": false, "setTempC": null, "coldChainMethod": null },
  "route": { "originAirport": "JFK", "destinationAirport": "LHR", "cargoReadyDate": "2026-05-31" },
  "pieces": [ { "qty": 3, "lengthCm": 120, "widthCm": 80, "heightCm": 90, "grossKgPerPiece": 85 } ],
  "computed": { "grossKg": 255, "volumeM3": 2.59, "chargeableKg": 432 },
  "services": ["CUSTOMS_AES"],
  "shipper":   { "company": "Acme Exports LLC", "einTaxId": "12-3456789", "country": "US" },
  "consignee": { "company": "Overseas buyer Ltd.", "contact": null, "country": "GB" },
  "notifyParty": { "sameAsConsignee": true, "company": null, "country": null },
  "contact":   { "person": "Jordan Lee", "email": "jordan@acmeexports.com", "phone": "+1...", "referencePo": "PO-2026-0042" },
  "notes": ""
}
```

---

## 8. File map

```
docs/ui-ux/mockups/air-export/
├── theme/
│   └── theme.css                       # shared primitives (extracted from fcl-export-booking.html)
├── mockup/
│   └── air-export-quote-request.html   # this page — links ../theme/theme.css
└── doc/
    └── air-export-quote-request.md     # this document
```
