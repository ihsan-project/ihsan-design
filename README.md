# Ihsan Design System

A structural design system for building **navigable, auth-gated, mobile-first web apps** — expressed in [Material Design](https://m3.material.io/)'s structural vocabulary but **implementation-agnostic** (works with Material UI, plain CSS, Tailwind, or any component library).

It governs **how an app is assembled and behaves** — its shell, navigation, side menu, auth flow, mobile behavior, recurring components, and accessibility. It is **not a brand**: it prescribes no colors and no fonts. Bring your own visual identity; keep the structure consistent.

## Using this with claude.ai/design

Point the design-system field at this repository's URL. Claude will read [`DESIGN_SYSTEM.md`](./DESIGN_SYSTEM.md) — the single, self-contained spec — and apply these structural patterns to whatever you build.

## What's inside

The full spec lives in **[`DESIGN_SYSTEM.md`](./DESIGN_SYSTEM.md)**:

1. **Principles** — the six rules everything derives from.
2. **App shell layout** — the App Bar + Navigation Drawer + content skeleton, and the one breakpoint (`md`/900px) that drives it.
3. **Navigation** — app bar anatomy and the fixed drawer ordering (profile → primary nav → utilities → sign out).
4. **Navigation drawer (side menu)** — variants, dimensions, and the full set of mobile behaviors (swipe, scrim, ESC, auto-close).
5. **Auth flow** — one auth context, **public-first / gate-on-action**, and resuming the intended action after sign-in.
6. **Mobile-friendliness** — breakpoints, touch targets, full-screen search, FABs, safe areas.
7. **Component patterns** — App Bar, Drawer, Profile Block, Nav Item, FAB, Login Screen, Auth Gate, and Loading / Empty / Error states.
8. **Accessibility & interaction** — focus traps, ESC/scrim, ARIA, keyboard, and reduced-motion as a release checklist.

## Core ideas at a glance

- **Primary navigation lives in the drawer, not the top bar.**
- **Public-first; gate on action** — no login walls, no silent redirects.
- **Mobile is the default; desktop is the enhancement.**
- **Every overlay traps focus and closes on ESC + scrim.**
- **One auth context** as the single source of truth.
- **Structure is shared; identity is not.**

## License

[MIT](./LICENSE) — free to use, copy, modify, and share.
