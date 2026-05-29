# Air Export — Mockups & Specification

This folder contains the UI/UX design package for the **Air Export Request / Booking** flow.

## Structure

```
air-export/
├── mockup/
│   └── air-export-page-mockup.md              # Visual mockup, wireframes, component breakdown
├── doc/
│   └── air-export-functional-specification.md # Purpose, workflow, validation, actions, errors
├── theme/
│   └── air-export-theme-compliance.md         # Design-system token & accessibility conformance
└── README.md
```

## Read in this order

1. [Page Mockup](./mockup/air-export-page-mockup.md) — what it looks like and how it's laid out.
2. [Functional Specification](./doc/air-export-functional-specification.md) — how it behaves.
3. [Theme Compliance](./theme/air-export-theme-compliance.md) — how it conforms to the design system.

## Related docs

- [Design System](../../02-design-system.md) — single source of truth for tokens & primitives
- [Quote Creation Spec](../../03-quote-creation-spec.md)
- [Mockups index](../README.md)

## Conventions

- All designs follow the [Design System](../../02-design-system.md) tokens and primitives — indigo
  brand, slate neutrals, Lucide icons, Tailwind layout + SCSS primitives.
- ASCII wireframes show layout; component tables map regions to design-system primitives.
- A single responsive design is documented inline (no separate device mockups).
