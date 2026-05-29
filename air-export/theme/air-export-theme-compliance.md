# Air Export Request / Booking — Theme Compliance

> **Service:** Air Export &nbsp;|&nbsp; **Type:** Design-system conformance &nbsp;|&nbsp; **Status:** v1
> **Companion docs:** [Page Mockup](../mockup/air-export-page-mockup.md) · [Functional Spec](../doc/air-export-functional-specification.md) · [Design System](../../../02-design-system.md)
> **Last updated:** 2026-05-29

This document certifies that the Air Export page design consumes **only** the existing
[Design System](../../../02-design-system.md) tokens and primitives — no new color, radius,
shadow, or typographic language is introduced. The only additions are the **form primitives**
(`field`, `input`, `select`, radio-card, switch, dropzone) that the design system itself flags as
the missing piece for data-entry screens (design-system §6).

---

## Table of Contents

1. [Compliance summary](#1-compliance-summary)
2. [Color tokens](#2-color-tokens)
3. [Typography](#3-typography)
4. [Spacing, radius & elevation](#4-spacing-radius--elevation)
5. [Component mapping](#5-component-mapping)
6. [Status semantics](#6-status-semantics)
7. [Responsive compliance](#7-responsive-compliance)
8. [Accessibility & WCAG](#8-accessibility--wcag)
9. [New primitives required](#9-new-primitives-required)
10. [Compliance checklist](#10-compliance-checklist)

---

## 1. Compliance summary

| Area | Verdict | Notes |
|---|---|---|
| Color | ✅ Compliant | Only `:root` brand/semantic tokens + slate ramp |
| Typography | ✅ Compliant | Type scale per design-system §1.4; Roboto/Poppins (loaded in `index.html`) |
| Spacing / radius / shadow | ✅ Compliant | 4px scale; `--r-card/--r-control/--r-pill`; shadow scale |
| Components | ✅ Compliant | Reuses `.card/.btn/.pill/.badge/.kv/.timeline/.progress-rail` |
| Forms | ⚠ Additive | Uses the **planned** `_forms.scss` primitives (design-system §6) — token-based, no new language |
| Icons | ✅ Compliant | Lucide via `data-lucide` + `lucide.createIcons()` |
| Accessibility | ✅ Targets AA | Per design-system §4.9 |
| Responsive | ✅ Compliant | Design-system breakpoints + `max-w-7xl` container |

---

## 2. Color tokens

All colors resolve to canonical `:root` tokens (design-system §1.1–1.2). **No hex literals** are
introduced in this design.

| Usage on the page | Token | Tailwind alias (per design-system §2) |
|---|---|---|
| Primary actions (Create booking, active step, links) | `--brand-primary` `#2A3E8A` | `bg-brand` / `text-brand` |
| Active pill / KPI / selected radio-card fill | `--brand-50` `#EEF2FA` | `bg-brand-50` |
| Badge active fill | `--brand-100` `#E9EDF7` | — |
| Secondary action (Save draft, confirm) | `--brand-secondary` `#78B947` | `text-brandgreen` |
| Success / complete / on-time | `--ok` `#10b981` | `text-ok` |
| Info / in-progress / in-transit | `--info` `#3b82f6` | `text-info` |
| Warning / needs-attention / at-risk | `--warn` `#f59e0b` | `text-warn` |
| Error / invalid / DG alert | `--err` `#ef4444` | `text-err` |
| Primary text | `--ink` `#0f172a` | `text-ink` |
| Labels / secondary text / eyebrow | `--muted` `#64748b` | `text-muted` |
| Hairline borders, dividers, input borders | `--border` `#e2e8f0` | `border` |
| Card / input surfaces | `--panel` `#ffffff` | `bg-panel` |
| Hover rows, muted fills | `--surface-subtle` `#f8fafc` | — |
| Zebra / chip background | `--surface-2` `#f1f5f9` | — |
| Keyboard focus outline | `--focus-ring` `rgba(42,62,138,.35)` | — |

**Rules honored:** tokens only; no component re-declares `--brand…`; semantic colors used per their
defined meaning (no decorative reuse of `--err`/`--ok`).

---

## 3. Typography

Type scale follows design-system §1.4. Fonts loaded in `src/index.html`: **Roboto** for body/UI and
**Poppins** for headings/display (the type scale of sizes/weights below is canonical regardless of
family).

| Page element | Role (design-system §1.4) | Spec |
|---|---|---|
| Page title "Air Export Request" | Page/section heading | 1.0–1.125rem / 700–800 / `--ink` (`text-lg font-bold`) |
| Section card titles | Card title | 0.95rem / 700 / `--ink` (`.section-title`) |
| Field labels (eyebrow) | **Eyebrow label** | 0.70rem / 700–800 / `.06–.08em` / uppercase / `--muted` |
| Chargeable weight value | Display / KPI value | 1.875rem / 700 / `--ink` (`text-3xl font-bold`) |
| Estimated price range | Display / KPI value | 1.875rem / 700 / `--ink` |
| Input text, body copy | Body | 0.875rem / 400–600 / `--ink` (`text-sm`) |
| Summary `.kv` values | Value (dense) | 0.85rem / 600 / `--ink` |
| Helper / meta / "Saved ✓" | Caption / meta | 0.75rem / `--muted` (`text-xs`) |

The eyebrow label idiom is used consistently for every field and section label — the signature of
these screens (design-system §1.4 note, §4.8).

---

## 4. Spacing, radius & elevation

Per design-system §1.3 (Tailwind 4px scale).

| Property | Token / value | Applied to |
|---|---|---|
| Section vertical rhythm | `mt-6` (24px) | Gap between cards |
| Card padding | `1.25rem` / `1rem` tight | `.card` / `.card--tight` upload tiles |
| Intra-group gap | `gap-3` / `gap-4` | Field grids, pill groups |
| Grid gutter | `gap-6` | 8/4 form-sidebar grid |
| Card radius | `--r-card` `1rem` | All section cards, sidebar cards, upload tiles |
| Control radius | `--r-control` `.75rem` | Inputs, selects, primary buttons |
| Small control radius | `--r-control-sm` `.625rem` | `.btn-sm`, chips-as-buttons |
| Pill radius | `--r-pill` `9999px` | Pills, badges, chips, icon circles |
| Resting elevation | `--shadow-card` | Cards |
| Hover elevation | `--shadow-card-hover` | Card hover lift |
| Popover elevation | `--shadow-pop` | Dropdowns, price-breakdown modal, toasts |

Container: `mx-auto max-w-7xl px-4` (design-system §4.7). Content never exceeds `max-w-7xl`.

---

## 5. Component mapping

Every region maps to an existing primitive (design-system §3). See also Mockup §7.

| Page region | Design-system primitive | Status |
|---|---|---|
| Section containers | `.card`, `.card[data-accent]`, `.card--tight` | Reuse |
| Section headers | `.section-head` + `.ico` + `.section-title` | Reuse |
| Stepper | `.progress-rail` / `.pr-step` (done/progress/pending) | Reuse |
| Completeness meter | `.progress-rail` style | Reuse |
| Status badges (section + lifecycle) | `.badge--active/--delivered/--delayed/--atrisk` | Reuse |
| Route ribbon, totals chips | `.chip--mode/--status/--eta`, `.chip-strong` | Reuse |
| Special-requirement toggles | `.pill` / `.pill--on` | Reuse |
| Summary / pieces totals | `.kv` (`dt`/`dd`), `.kv-icon` | Reuse |
| Buttons | `.btn-primary/.btn-secondary/.btn-soft-primary/.btn-ghost/.btn-sm` | Reuse |
| Copilot card | `.card[data-accent="indigo"]` + `.btn-soft-primary` | Reuse |
| Overflow / mode menus | `.menu-item` / `.menu-panel` | Reuse |
| Loading (async selects, price) | `.skeleton` shimmer | Reuse |
| Airport / company search | `app-custom-dropdown` / `.dropdown-list` / `.dropdown-item` | Reuse |
| Text/number/email inputs | `.field` + `.input` | **New (planned §6)** |
| Service Type / Incoterm picker | radio-card / segmented group | **New (planned §6)** |
| DG / temperature / insurance switches | `.switch` | **New (planned §6)** |
| Document tiles | dropzone on `.card--tight` | **New (planned §6)** |

---

## 6. Status semantics

Locked to the design-system status mapping (§3 "Status semantics"). Status is always **color +
icon + text**, never color alone (design-system §4.5).

| Meaning on this page | Color | Lucide icon | Class |
|---|---|---|---|
| Section complete / valid field / serviceable lane | `--ok` | `circle-check` | `.badge--delivered`, `.chip-status--delivered` |
| In progress / quote requested / in-transit | `--info` / brand | `loader-circle`, `plane` | `.badge--active`, `.chip-status--transit` |
| Needs attention / missing field / customs risk | `--warn` | `clock-alert`, `alert-triangle` | `.badge--atrisk` |
| Error / invalid Incoterm / DG alert / upload failed | `--err` | `alert-triangle` | `.badge--delayed`, `.chip-status--delayed` |
| Pending / disabled conditional field | slate | `flag` | `.pending` |

The Copilot reuses these exact semantics for its Validate/Complete/Suggest rows (✓ `--ok`,
✗ `--err`, ⚠ `--warn`, 💡/✦ brand).

---

## 7. Responsive compliance

Single responsive design using design-system breakpoints (§1.3 / §4.7): `sm 640 · md 768 ·
lg 1024 · xl 1280 · 2xl 1536`. Full behavior in Mockup §9.

| Breakpoint | Compliance behavior |
|---|---|
| `< sm` | Single column; pieces table → stacked cards; sidebar → bottom sheet; touch targets ≥ 44px |
| `sm–md` | Paired groups stay stacked; form single column |
| `md` | Paired cards 2-up (Shipper∣Consignee, Pickup∣Delivery); table scrollable |
| `lg` | Signature 8-col form / 4-col sticky sidebar (`lg:sticky lg:top-20`) |
| `xl/2xl` | Same split within `max-w-7xl`; extra width to gutters |

No horizontal page scroll at any width; container centered with `mx-auto max-w-7xl`.

---

## 8. Accessibility & WCAG

Targets **WCAG 2.1 AA** per design-system §4.9. Full treatment in Mockup §10.

| Requirement | Implementation | WCAG |
|---|---|---|
| Text contrast | `--ink`/`--muted` on `--panel` meet AA; verify small white-on-`--brand` text | 1.4.3 |
| Non-text contrast | Input borders `--border`, focus ring `--focus-ring` ≥ 3:1 | 1.4.11 |
| Status not color-alone | Color + icon + text everywhere | 1.4.1 |
| Labels & instructions | `<label for>` on every control; eyebrow labels are real labels | 3.3.2 |
| Error identification | Inline + `aria-describedby` + `aria-live` summary | 3.3.1 |
| Required fields | `aria-required`, announced `*` | 3.3.2 |
| Keyboard operable | All controls incl. Copilot reachable; logical tab order | 2.1.1 |
| Focus visible | `:focus-visible` ring via `--focus-ring` | 2.4.7 |
| Headings & landmarks | `<form>`, `<section>`+`<h2>`, `<nav aria-label>` stepper, `aria-current` | 1.3.1 / 2.4.6 |
| Live updates | Chargeable weight, completeness, price → `aria-live="polite"` | 4.1.3 |
| Reduced motion | Collapse/route animations respect `prefers-reduced-motion` | 2.3.3 |
| Touch target size | ≥ 44px on coarse pointers | 2.5.5 (AAA, adopted) |

**Recommendations:** run an automated contrast pass on the price-card and any white-on-brand badge
text; verify the radio-card group exposes a proper `radiogroup` role; ensure the bottom-sheet
Copilot traps and restores focus correctly on mobile.

---

## 9. New primitives required

These are **not** new design language — they are the form primitives the design system explicitly
defers to `_forms.scss` (design-system §6). Each is built from existing tokens:

| Primitive | Built from | Tokens |
|---|---|---|
| `.field` (label + hint + control + error) | layout wrapper | spacing scale, `--muted`, `--err` |
| `.input` (text/number/email) | bordered control | `--r-control`, `--border`, `--focus-ring`, `--panel` |
| `.select` | wraps `app-custom-dropdown`/`app-normal-dropdown` | `.dropdown-list`/`.dropdown-item` |
| radio-card / segmented group | `.pill`-derived | `--brand-50` active fill, `--r-control` |
| `.switch` (toggle) | new control | `--brand-primary` on, `--border` off |
| dropzone tile | `.card--tight` + states | `--focus-ring` drag-over, `--err` error, `.skeleton` upload |

Implementing these here advances the shared `_forms.scss` consolidation (design-system §5–6),
benefiting every future data-entry screen.

---

## 10. Compliance checklist

- [x] No hardcoded hex for brand/semantic colors — `var(--…)` / Tailwind aliases only.
- [x] No component re-declares `--brand…` tokens locally.
- [x] Radius limited to `--r-card` / `--r-control(-sm)` / `--r-pill`.
- [x] Elevation limited to the shadow scale (`card` / `card-hover` / `pop`).
- [x] Type scale per §1.4; eyebrow labels for field/section labels.
- [x] Reuses display primitives (`.card/.badge/.pill/.kv/.progress-rail/.chip`).
- [x] Status = color + icon + text (semantic mapping locked).
- [x] Icons via Lucide `data-lucide` + `lucide.createIcons()`.
- [x] Container `mx-auto max-w-7xl`; section rhythm `mt-6`; card padding `1–1.25rem`.
- [x] Responsive per design-system breakpoints; no horizontal scroll.
- [x] Accessibility targets WCAG 2.1 AA (real controls, labels, focus ring, live regions).
- [⚠] Form primitives use the planned `_forms.scss` (token-based; advances design-system §5–6).

**Overall:** ✅ **Compliant** with the existing design system; the only additions are the
token-based form primitives the design system itself prescribes.

---

*See the [Page Mockup](../mockup/air-export-page-mockup.md) and
[Functional Specification](../doc/air-export-functional-specification.md) for the full design.*
