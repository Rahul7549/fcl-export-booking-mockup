# FCL Export Booking — Mockups (V1 & V2)

Two viewable, self-contained mockups of the **FCL export booking / quote request** screen.
Both use the **same** brand tokens (`#2A3E8A`), primitives (`.card`/`.btn`/`.pill`/`.badge`/
`.chip`/switches/steppers), spacing, typography, and Lucide icons — so either can drop into the
product. They differ in **layout paradigm and UX flow**, not in branding.

| File | Direction | Concept |
|---|---|---|
| `fcl-export-booking.html` | **V1** | **Guided wizard** — 4 gated steps, horizontal top step-tracker, single sticky text summary (8/4 split). |
| `fcl-export-booking-v2.html` | **V2** | **Shipment Builder** — single free-scroll canvas, sticky left section-navigator + completeness ring, sticky right *visual* Shipment Preview (3-zone cockpit). |
| `quote-results.html` | **Results** | **Quote comparison marketplace** — the page shown *after* "Request Quote": summary header, sticky filter sidebar, sortable carrier cards with expandable pricing breakdown + route timeline, compare bar/modal. |

> Open either file directly in a browser. No build step. Both carry the same field rules:
> US-origin export semantics, AES/EEI, pickup-ZIP-only door origin, KG/LBS container unit,
> phone country-code selector, HS code removed, declared value commented out.

---

## V2 concept in one line
Turn a step-by-step **form** into a **shipment you watch assemble** — a route that draws itself,
containers that stack in a live preview, and a completeness ring that fills as you go.

---

## What's different in V2

### 1. Alternative layout structure — 3-zone cockpit vs 2-zone wizard
- **Left rail (sticky):** combined **navigator + progress**. A circular **completeness ring**, four section links with per-section done-checks, and a primary action. Replaces V1's horizontal top tracker.
- **Center (single canvas):** all four sections on one scroll — no hard step gating. Led by a **visual route builder**.
- **Right (sticky):** a **Shipment Preview** that *visualizes* the shipment (route line + sailing ship, container-stack blocks, live TEU/weight/cost drivers) — richer than V1's text-only summary.

### 2. Updated UX flow
- **Free navigation** instead of Next/Back gating: power users (forwarders, repeat shippers) jump to any section via the rail or scrollspy. Fewer clicks to edit something three steps back.
- **Scroll = progress.** A scrollspy keeps the rail/section in sync; the ring shows overall %.
- **Section-level collapse:** any completed section collapses to keep the canvas short — progressive disclosure without losing the single-page benefit.

### 3. Enhanced section organization (4 smarter groups, route-first)
V1 ordered by form convenience (Service → Route → Cargo → Services/Contact). V2 reorders to the
**logistics mental model**:
1. **Route & service** (merged) — the visual route builder leads, because "from/to + how far" is the first thing a freight person thinks.
2. **Cargo & containers** — commodity + handling + the container mix together.
3. **Customs & services** — export compliance (AES/EEI) and value-adds.
4. **Parties & contact**.

### 4. New interactions / micro-interactions
- **Sailing-ship animation** along the route connector (respects `prefers-reduced-motion`).
- **Live route endpoints** — preview POL/POD update as you change the selects.
- **Container-stack visualization** — colored cubes accumulate per container (capped with “+N”), animating in (`pop`).
- **Completeness ring** animates toward 100%; section numbers flip to green checks when complete.
- **Auto-save “Draft saved · Ns ago”** pill that ages over time.
- **Flash-on-change** summary values; scrollspy highlight slides between rail items.

### 5. Modern card / form variations
- **Section cards** use a **left accent bar + numbered badge + collapsible header** (V1 used a top accent + fixed step panels).
- **Route builder** is a distinct gradient panel with origin/destination nodes flanking the animated ocean leg.
- **Service types** become compact **chips** inside the builder (V1 used large radio-cards) — denser, faster to switch, and visually tied to the route.
- **Service toggles** are presented as a tidy **2-column tile grid**.

### 6. Sticky summary / progress improvements
- V1: one sticky text summary + a top tracker.
- V2: **two complementary sticky aides** — the **progress rail** (where am I / what's left) on the left and the **visual preview** (what am I building) on the right. The progress signal is always visible as a ring, not just dots.

### 7. Better mobile responsiveness strategy
- **Single-canvas scales down naturally** (it's already one column).
- The left rail collapses into a **sticky horizontal scrollspy chip bar** under the header.
- The preview moves below the form on tablet, and key numbers live in a **sticky bottom action bar** (weight + “Request quote”) on mobile — thumb-reachable, always present.
- Three tiers: `< lg` single column + bottom bar · `lg` form + preview side-by-side · `xl` full 3-zone cockpit.

---

## V1 vs V2 — comparison notes

| Dimension | V1 — Guided Wizard | V2 — Shipment Builder |
|---|---|---|
| Mental model | Step-by-step form | Assemble a shipment on one canvas |
| Navigation | Linear (Next/Back), tracker | Free (scroll / rail / scrollspy) |
| Progress signal | Horizontal dots | Completeness **ring** + per-section checks |
| Summary | Sticky **text** rail | Sticky **visual** preview (route + container stack) |
| Best for | First-timers, lots of guidance, controlled validation gates | Repeat/expert users, speed, editing freely |
| Cognitive load | One step at a time (lower per screen) | Whole picture visible (higher context) |
| Visual “wow” | Clean, conventional | Premium, distinctive, logistics-flavored |
| Form length feel | Short steps | Long page, mitigated by collapse + rail |
| Implementation effort | Lower | Higher (scrollspy, ring, viz) |

---

## Why a client may prefer V2
1. **Looks premium and distinctive** — the route visualization and container stack make it feel like purpose-built freight software, not a generic form. Strong demo appeal.
2. **Faster for repeat users** — no step gating; jump anywhere, edit anything, see everything. Logistics teams quote many shipments a day.
3. **Stronger “what am I building” feedback** — the live preview + completeness ring reduce uncertainty and abandonment.
4. **Logistics-native information architecture** — route-first ordering matches how freight professionals think.
5. **Scales elegantly to mobile** — a single canvas + sticky bottom action is simpler and more robust than reflowing a multi-step wizard.
6. **Enterprise cockpit feel** — two sticky aides (navigate + preview) read as a serious operations tool.

### When V1 might still win
- Audiences who want **maximum hand-holding** and **hard validation gates** (e.g., occasional/first-time exporters), or where **implementation speed** matters most. V1 is the safer, more conventional choice; V2 is the more ambitious, modern one.

> Recommendation for the pitch: show **V1 as the dependable standard** and **V2 as the premium,
> differentiated direction** — and note both share one design system, so picking V2 costs design
> consistency nothing.
