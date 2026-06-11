# Ihsan Design System

A structural design system for building **navigable, auth-gated, mobile-first web apps**.

This document describes *how an app is put together* — its shell, navigation, side menu, authentication flow, mobile behavior, recurring components, and accessibility. It speaks in the structural vocabulary of [Material Design](https://m3.material.io/) (app bar, navigation drawer, elevation, breakpoints, FABs) but is **implementation-agnostic**: every pattern maps to Material UI, but can be built just as well in plain CSS, Tailwind, or any component library.

## Philosophy

Most apps re-solve the same structural problems: where does navigation live, how does the side menu behave on a phone, how do you gate features behind sign-in without trapping the user, what does the page skeleton look like. This system answers those questions once, opinionatedly, so each app can spend its effort on what makes it different.

It is **not a brand**. It prescribes no color palette and no typeface — bring your own. It prescribes *structure*: the layout, the flows, the behaviors, the interaction rules.

## How to use this document

- **Building a new app?** Adopt Sections 2–6 wholesale. They define the shell, navigation, auth, and responsive behavior. Reach for Section 7 when you need a specific piece, and treat Section 8 as a hard checklist before shipping.
- **Bring your own visual identity.** Pick your own colors, fonts, and logo. This system governs *how things are arranged and behave*, not how they're painted.
- **The rules are prescriptive on purpose.** "Always" and "Never" are deliberate. Deviate when your app genuinely needs to, but make it a conscious decision, not an accident.
- **Material UI is the reference implementation,** not a requirement. Each component lists its MUI mapping and a plain-CSS/Tailwind note.

## Table of contents

1. [Principles](#1-principles)
2. [App shell layout](#2-app-shell-layout)
3. [Navigation](#3-navigation)
4. [Navigation drawer (side menu)](#4-navigation-drawer-side-menu)
5. [Auth flow](#5-auth-flow)
6. [Mobile-friendliness](#6-mobile-friendliness)
7. [Component patterns](#7-component-patterns)
8. [Accessibility & interaction](#8-accessibility--interaction)

---

## 1. Principles

Six rules that everything else in this document derives from. When a detailed decision is unclear, return to these.

1. **Primary navigation lives in the drawer, not the top bar.** The app bar carries the menu trigger, identity, and a few contextual actions — nothing more. Destinations live in the side menu. This keeps the top bar uncluttered and identical across every screen size.

2. **Public-first; gate on action.** Render as much as possible to signed-out users. Ask for sign-in at the moment a protected action is taken — not with an upfront wall and not with a silent redirect.

3. **Mobile is the default; desktop is the enhancement.** Design every screen for a phone first. Larger viewports *add* affordances (a permanent drawer, inline search), they don't change the fundamental structure.

4. **Every overlay traps focus and closes predictably.** Drawers, dialogs, and sheets trap keyboard focus, lock body scroll, and close on both `Escape` and a scrim tap. No exceptions.

5. **One source of truth for auth.** A single context provides `{ user, loading, signIn, signOut }` to the whole tree. Components read auth state; they never fetch it themselves.

6. **Structure is shared; identity is not.** Layout, flows, spacing rhythm, and behavior come from this system. Color, type, and voice come from the app.

---

## 2. App shell layout

Every app is built from the same three-part skeleton.

```
┌─────────────────────────────────────────┐
│  App Bar   [≡]  Brand        [search][•] │  ← top, fixed/sticky
├──────────┬──────────────────────────────┤
│          │                              │
│  Nav     │   Content region             │
│  Drawer  │   (page, max-width container)│
│  (left)  │                              │
│          │                              │
└──────────┴──────────────────────────────┘
```

- **App Bar** — a fixed/sticky bar across the top. Holds the drawer trigger, brand, and contextual actions. See [§3](#3-navigation).
- **Navigation Drawer** — the left side menu. Holds the account block and primary navigation. See [§4](#4-navigation-drawer-side-menu).
- **Content region** — the page. Constrained to a readable max width and centered on large viewports.

### Responsive rule

The drawer's behavior is governed by **one breakpoint: `md` (900px)**.

| Viewport | Drawer behavior |
| --- | --- |
| `< md` (phones, small tablets) | **Temporary** — slides in over the content with a scrim. Closed by default. |
| `≥ md` (desktops) | **Permanent** for apps with **4+ primary destinations**; otherwise stay **temporary** and triggered by the menu button. |

> A permanent desktop drawer is an enhancement, not a requirement. Small apps (1–3 destinations) are cleaner with a temporary drawer at every size. Don't add a permanent rail just because there's room.

### Dimensions

| Element | Value |
| --- | --- |
| App bar height | **56px** below `md`, **64px** at `md`+ |
| Drawer width | **280–288px** |
| Content max width | **~840–960px** for reading/forms; full-bleed for dashboards |
| Content horizontal padding | **16px** mobile, **24px** desktop |

### Z-index layering

Layer from back to front. Reuse one named scale so overlays never fight. These are Material UI's default `theme.zIndex` tokens:

```
content              : 0
mobile stepper       : 1000
fab / speed dial     : 1050
app bar              : 1100
navigation drawer    : 1200
modal (dialog/sheet) : 1300   (the modal scrim shares this stacking context, just beneath its surface)
snackbar / toast     : 1400
tooltip              : 1500
```

> To keep the **app bar above a permanent drawer**, raise it explicitly — in Material UI, set `zIndex: theme.zIndex.drawer + 1` on the app bar, since the default app bar token (1100) sits *below* the drawer's 1200. Reuse this scale rather than inventing your own.

### Composition

Wrap the shell once, at the root, around the routed content:

```jsx
<AuthProvider>
  <AppShell>            {/* renders AppBar + Drawer + scrim */}
    <main>
      <Routes />        {/* the routed page renders here */}
    </main>
  </AppShell>
</AuthProvider>
```

The shell owns the drawer's open/closed state and passes a toggle to the app bar. Pages never render their own app bar or drawer — they render only into the content region.

---

## 3. Navigation

### App bar anatomy

```
[≡]   Brand / Title  ························  [search] [notifications•] [avatar]
 │         │                                       │          │           │
 menu   identity                              contextual actions (right-aligned)
```

- **Left: menu trigger.** A hamburger (`≡`) button that toggles the drawer. Always present below `md`. At `md`+ with a permanent drawer, it may be hidden.
- **Center/left: brand or page title.** App name on the home surface; the current page's title on inner surfaces. Keep it to one line; truncate with ellipsis.
- **Right: contextual actions.** Zero to three icon buttons for the most important global actions — typically search, notifications, and/or the account avatar. More than three belongs in the drawer or an overflow menu.

**Rules**

- **Never** put primary navigation links in the app bar. Destinations live in the drawer ([Principle 1](#1-principles)).
- Keep the app bar **identical in structure across breakpoints**. On mobile, collapse search into an icon that opens a [full-screen search overlay](#6-mobile-friendliness); don't restructure the bar.
- Badge actionable counts (unread notifications, cart items) on their icon. See [Nav List Item](#nav-list-item) and [§8](#8-accessibility--interaction) for badge a11y.

### Drawer navigation order

The drawer is the navigation surface. Its contents follow a fixed vertical order:

```
┌────────────────────────────┐
│  Profile block             │  ← account identity or sign-in CTA
├────────────────────────────┤
│  ▸ Primary destination 1   │  ← the core nav list
│  ▸ Primary destination 2   │
│  ▸ Primary destination 3   │
├────────────────────────────┤  ← divider
│  ▸ Utility / secondary     │  ← settings, help, external links
│  ▸ ...                     │
│                            │
│  (flexible space)          │
├────────────────────────────┤
│  ⎋ Sign out                │  ← pinned to the bottom
└────────────────────────────┘
```

1. **Profile block** at the top — avatar + name + email, or a sign-in CTA when signed out. See [§4](#4-navigation-drawer-side-menu).
2. **Primary navigation list** — the app's core destinations.
3. **Divider**, then **secondary/utility links** — settings, help, external links.
4. **Sign out** pinned to the bottom (signed-in only), separated from navigation so it's never tapped by accident.

### Active state & behaviors

- **Highlight the active route.** Use a filled/tinted background or a left accent border on the matching nav item. Match it against the current path with exact matching for the home route and prefix matching for sections.
- **Auto-close on navigate (mobile).** Tapping a destination on a temporary drawer navigates *and* closes the drawer. A permanent drawer stays open.
- **Badge nav items** that have pending counts (e.g. a notifications destination), mirroring any badge shown in the app bar.

---

## 4. Navigation drawer (side menu)

The drawer is the most behavior-rich surface in the app. Get it right and the app feels native on a phone.

### Variants — and when to use each

| Variant | Description | Use when |
| --- | --- | --- |
| **Temporary** | Slides over content with a scrim; dismissed by scrim tap, `Escape`, swipe, or navigating. | The default. Always use below `md`. |
| **Permanent** | Always visible, docked to the left; content is offset to make room. No scrim. | `md`+ **and** the app has 4+ primary destinations. |
| **Persistent** | Docked but dismissible; content reflows when it opens/closes. | Rare. Only when desktop users need to reclaim horizontal space (e.g. a wide canvas/editor). |

Default to **responsive**: temporary below `md`, switching to permanent at `md`+ if the app qualifies.

### Dimensions & structure

- **Width:** 280–288px.
- **Full height**, anchored left.
- **Contents** follow the [drawer navigation order](#drawer-navigation-order) from §3.

### Behaviors (temporary variant)

A temporary drawer must support **all** of these:

- **Swipe to open** from the left screen edge. Use a small edge hit-area (~20px) and require a minimum horizontal travel (~50px) before committing, so vertical scrolls aren't hijacked.
- **Swipe to close** — a left-swipe on the open drawer dismisses it.
- **Scrim tap to close** — tapping the darkened backdrop dismisses it.
- **`Escape` to close.**
- **Auto-close on navigate.**
- **Keep mounted** (`keepMounted`) so open/close is instant and swipe feels responsive; the drawer's content stays in the DOM, hidden.
- **Body scroll lock** while open ([§8](#8-accessibility--interaction)).

> On iOS, disable the backdrop transition and the swipe-to-discover hint to avoid janky native-feeling conflicts. (MUI's `SwipeableDrawer` exposes `disableBackdropTransition` and `disableDiscovery` for exactly this.)

### Profile block

The first thing in the drawer. Two states:

**Signed in:**
```
┌────────────────────────────┐
│  (avatar)  Display Name     │
│            email@example.com│
└────────────────────────────┘
```
- **Avatar:** the user's photo if available; otherwise **initials** on a colored background as a deterministic fallback (derive the color from the user id/name so it's stable).
- **Name** (primary) and **email** (secondary, truncated).

**Signed out:**
```
┌────────────────────────────┐
│   [ Sign in ]              │
│   Sign in to sync & save   │
└────────────────────────────┘
```
- A clear **sign-in CTA** plus one line on why it's worth it. Tapping it runs the [auth flow](#5-auth-flow).

### Implementation notes

- **MUI:** `Drawer` (or `SwipeableDrawer` for the temporary variant), `List` / `ListItemButton` / `ListItemIcon` / `ListItemText`, `Avatar`, `Divider`.
- **Plain CSS / Tailwind:** a fixed-position `<nav>` translated off-canvas (`translateX(-100%)` → `translateX(0)`), a sibling scrim `<div>`, and JS for the swipe/focus/scroll-lock behaviors above. The behaviors are not optional just because the styling is hand-rolled.

---

## 5. Auth flow

A single, predictable authentication model: one context, public-first access, and gating at the point of action.

### The auth context

Expose auth through **one provider at the root** and a `useAuth()` hook. Nothing else fetches or stores auth state.

```ts
type User = {
  uid: string;
  email: string | null;
  displayName: string | null;
  photoURL: string | null;
};

type AuthContext = {
  user: User | null;     // null when signed out
  loading: boolean;      // true until the initial session check resolves
  signIn: () => Promise<void>;
  signOut: () => Promise<void>;
};
```

```jsx
const { user, loading, signIn, signOut } = useAuth();
```

**Rules**

- **Restore the session on mount.** On load, the provider checks for an existing session and sets `loading: true` until it resolves. Gate any auth-dependent UI on `!loading` to prevent a **flash** of signed-out (or signed-in) content.
- **OAuth is the default sign-in method.** Google sign-in via popup/redirect covers the common case with no password handling. Offer additional methods only when the product needs them.
- **`user` shape is fixed** (above). Map whatever your auth backend returns into this shape at the provider boundary so the rest of the app is backend-agnostic.

### Public-first, gate-on-action

Do **not** wall the app behind a login screen, and do **not** silently redirect signed-out users away from protected routes. Instead:

1. **Render public content** to everyone.
2. When a user takes a **protected action** (saving, bookmarking, opening a members-only route), intercept it with an **[Auth Gate](#auth-gate)** — an inline prompt explaining the value and offering sign-in.
3. **Resume the intended action after login.** Capture the pending action/destination, run the auth flow, and on success **complete the original action automatically** so the user isn't dumped back at square one.

```jsx
function handleSave(item) {
  if (!user) {
    showAuthGate({ resumeWith: () => save(item) }); // run save() after sign-in
    return;
  }
  save(item);
}
```

### Protected routes

- **Client-side gating is the baseline.** A protected route renders an [Auth Gate](#auth-gate) in place of its content when `!user && !loading`. The user stays where they are; the URL doesn't change.
- **Reach for server-side gating** (middleware / server-checked sessions) when content must *never* reach the client unauthenticated — paid content, private data, compliance requirements. Client-side gating controls UX; server-side gating controls access. Use server-side when access is the concern.

### Login screen

When you need a dedicated sign-in surface (a `/login` route or the Auth Gate's expanded form):

- **Centered card** on a plain background.
- **Brand mark + one-line value proposition.**
- **One primary action**: the OAuth button.
- **Inline error region** for failures (popup closed, network error) — never a raw alert.

### Sign out

`signOut()` clears the auth state, resets any identified analytics/session, and returns the user to a sensible public surface. It does not need a confirmation dialog.

### Implementation notes

- **MUI:** `Dialog` or inline `Card` for the Auth Gate and login screen; `Button` for the OAuth action; `Alert` for the error region.
- **Backends:** the pattern is agnostic. Firebase Auth, Auth0, Clerk, or a custom OAuth/JWT backend all fit — map their user object into the `User` shape and implement `signIn`/`signOut` against their SDK.

---

## 6. Mobile-friendliness

Mobile is the default ([Principle 3](#1-principles)). These rules make "mobile-first" concrete.

### Breakpoints

Use the Material scale. One breakpoint — `md` — carries most of the structural weight.

| Name | Min width | Typical role |
| --- | --- | --- |
| `xs` | 0 | Phones (base styles target here) |
| `sm` | 600 | Large phones / small tablets |
| **`md`** | **900** | **The drawer breakpoint** — temporary below, permanent-eligible above |
| `lg` | 1200 | Desktops; widen/center content |
| `xl` | 1536 | Large desktops; cap content width |

**Write base styles for `xs` and layer enhancements upward.** Never write desktop styles and patch them back down for mobile.

### Touch & input

- **Touch targets ≥ 48×48px** with **≥ 8px** between adjacent targets. This applies to nav items, icon buttons, and FABs.
- **Inputs are ≥ 16px font-size** on mobile to prevent iOS auto-zoom.

### Mobile-specific patterns

- **Search → full-screen overlay.** Below `md`, the app-bar search icon opens a full-screen search surface (input + results) rather than cramming an input into the bar.
- **FAB for the primary action.** Promote the single most important creating/adding action to a [Floating Action Button](#floating-action-button) fixed to the bottom-right. Add bottom padding to scrollable content so the FAB never covers the last item.
- **Bottom sheets over side drawers** for contextual detail on mobile (e.g. item details, filters). A bottom sheet is reachable by thumb; a right-side drawer is not.

### Viewport & safe areas

- Ship the standard viewport meta:
  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
  ```
- Respect device **safe-area insets** (notches, home indicators) for fixed top/bottom elements:
  ```css
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  ```

---

## 7. Component patterns

A catalogue of the recurring pieces. Each entry gives **purpose · anatomy · rules · MUI mapping · plain-CSS/Tailwind note**. Build these once per app and reuse them everywhere.

### App Bar

- **Purpose:** top-level identity and global actions; hosts the drawer trigger.
- **Anatomy:** `[menu] · brand/title · spacer · [actions]`.
- **Rules:** fixed/sticky; one line; no primary nav links; ≤ 3 right-side actions; structurally identical across breakpoints.
- **MUI:** `AppBar` + `Toolbar` + `IconButton` + `Typography`.
- **Plain/Tailwind:** `position: sticky; top: 0` header with flex layout; reserve content top-padding equal to the bar height.

### Navigation Drawer

- **Purpose:** the navigation surface; account identity + destinations.
- **Anatomy:** [profile block → primary list → divider → utility list → sign out](#drawer-navigation-order).
- **Rules:** 280–288px; temporary below `md`; all [drawer behaviors](#behaviors-temporary-variant); auto-close on navigate.
- **MUI:** `SwipeableDrawer` / `Drawer` + `List`.
- **Plain/Tailwind:** off-canvas `<nav>` + scrim; hand-wire swipe, focus trap, scroll lock.

### Profile Block

- **Purpose:** show who's signed in (or prompt sign-in) at the top of the drawer.
- **Anatomy:** avatar + name + email **or** sign-in CTA.
- **Rules:** initials fallback when no photo; truncate long email; CTA states the value of signing in.
- **MUI:** `Avatar` + `Typography` + `Button`.
- **Plain/Tailwind:** rounded image / initials circle; flex column for text.

### Nav List Item

- **Purpose:** one destination in the drawer.
- **Anatomy:** `icon · label · [badge]`.
- **Rules:** ≥ 48px tall; active state via tint/left-border; badge numeric counts; whole row is the tap target.
- **MUI:** `ListItemButton` + `ListItemIcon` + `ListItemText` + `Badge`.
- **Plain/Tailwind:** flex row, `min-height: 48px`; `aria-current="page"` on the active item.

### Floating Action Button

- **Purpose:** the single most important create/add action on a surface.
- **Anatomy:** circular button, icon, bottom-right, fixed.
- **Rules:** **at most one per screen**; never for destructive or secondary actions; keep clear of the bottom safe-area; pad scroll content beneath it.
- **MUI:** `Fab`.
- **Plain/Tailwind:** `position: fixed; bottom/right` circular button with elevation shadow.

### Login Screen

- **Purpose:** dedicated sign-in surface.
- **Anatomy:** centered card · brand · value line · OAuth button · error region.
- **Rules:** one primary action; inline (not alert) errors; no dead-ends (always a way back to public content).
- **MUI:** `Card` + `Button` + `Alert`.
- **Plain/Tailwind:** centered max-width card; flex column.

### Auth Gate

- **Purpose:** intercept a protected action with a sign-in prompt instead of a wall or a redirect.
- **Anatomy:** icon · title · short value list · OAuth button · dismiss.
- **Rules:** appears **in place** (inline panel or dialog), not a route change; **captures and resumes** the intended action after sign-in; dismissible back to public content.
- **MUI:** `Dialog` or inline `Card` + `Button`.
- **Plain/Tailwind:** modal or inline panel reusing the [overlay rules](#8-accessibility--interaction).

### Loading State

- **Purpose:** communicate work in progress without layout shift.
- **Anatomy:** skeleton placeholders (preferred) or a centered spinner.
- **Rules:** prefer **skeletons** that match final layout for content; use a spinner only for indeterminate full-surface waits; reserve the final dimensions to avoid shift; gate auth-dependent UI on `loading`.
- **MUI:** `Skeleton` / `CircularProgress`.
- **Plain/Tailwind:** pulsing placeholder blocks; spinner via CSS animation.

### Empty State

- **Purpose:** turn "nothing here" into a next step.
- **Anatomy:** icon/illustration · one-line explanation · primary action.
- **Rules:** always offer the action that fills the emptiness (often the same as the FAB); never show a bare blank region.
- **MUI:** `Box` + `Typography` + `Button`.
- **Plain/Tailwind:** centered column.

### Error State

- **Purpose:** explain a failure and offer recovery.
- **Anatomy:** icon · plain-language message · retry action.
- **Rules:** never expose raw error text/stack traces; always give a retry or a way out; scope the error to the failed region, not the whole app, when possible.
- **MUI:** `Alert` (inline) / error `Box` (full-surface) + retry `Button`.
- **Plain/Tailwind:** bordered alert panel; full-surface centered column for fatal errors.

---

## 8. Accessibility & interaction

Treat this section as a **release checklist** for any overlay or interactive surface. Several of these are easy to forget and obvious when missing.

### Overlays (drawer, dialog, bottom sheet)

- **Focus trap.** While open, `Tab`/`Shift+Tab` cycle **within** the overlay and never reach the page behind it.
- **Initial focus** moves into the overlay on open (the first actionable element or a labeled close button).
- **Return focus** to the triggering element on close.
- **`Escape` closes** the overlay.
- **Scrim tap closes** the overlay (for temporary drawers and modal dialogs).
- **Body scroll lock** while open — the page behind must not scroll.
- **`aria-modal="true"`** and an appropriate `role` (`dialog`) on modal surfaces.

### Gestures

- **Edge-swipe to open** the drawer: ~20px edge hit-area, ~50px minimum travel before committing so vertical scrolling isn't stolen.
- **Swipe to close** on the open drawer.
- Gestures are **additive** — every gesture has a non-gesture equivalent (the menu button, the scrim, the close button). Never make a gesture the *only* way to do something.

### ARIA & semantics

- **Menu trigger:** `aria-label="Open menu"`, `aria-expanded={open}`, `aria-controls="<drawer id>"`.
- **Active nav item:** `aria-current="page"`.
- **Badges:** pair the visual count with an accessible label (e.g. `aria-label="Notifications, 3 unread"`); a bare colored dot needs a text equivalent.
- **Icon-only buttons** always carry an `aria-label`.
- **Landmarks:** one `<header>` (app bar), one `<nav>` (drawer), one `<main>` (content).

### Keyboard

- Every interactive element is reachable and operable by keyboard.
- **Visible focus indicators** — never remove focus outlines without replacing them with an equally clear style.
- Logical tab order following visual order.

### Motion & preferences

- **Honor `prefers-reduced-motion`:** reduce or remove drawer slides, transitions, and non-essential animation when the user asks for it.
- **Honor `prefers-color-scheme`** if the app supports light/dark — follow the system preference by default and let the user override.

---

*This is a structural design system. It tells you how to assemble an app, not how to brand it. Bring your own colors, type, and voice — and keep the structure consistent.*
