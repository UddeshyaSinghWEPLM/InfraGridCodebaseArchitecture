# InfraGrid Branding & Customization Design Doc

> **Status**: Living document — work proceeds component by component / page by page, each increment logged in §5 as it ships.
> **Baseline**: This app was forked from `PLMInfrastructureRegisterCodebaseArchitecture` (see `SESSION_HANDOFF.md` and `upstream` git remote). Its existing design language is documented in full in `DESIGN.md` at this repo's root — that file is **not edited** by this effort, so the diff against `upstream` stays small and mergeable. This doc only records what's *changing* and why.
> **Scope so far**: logo unchanged; colors defined but intentionally unmapped; typography decided but not yet wired into code; border radius being changed incrementally, one surface at a time, not as a single global sweep.

---

## 1. Logo

Unchanged. Same asset (`src/Assets/weplm.jpeg`), same usage.

## 2. Color Palette

Three brand colors are reserved. **Do not apply these anywhere yet** — which color goes on which element will be decided per component/page as that work comes up, not as a batch now.

| Role | Hex |
|---|---|
| Primary | `#0d2086` |
| Secondary | `#1059fa` |
| Tertiary | `#ccd8fa` |

**Audit note (why this matters later):** the current codebase already hardcodes a near-identical navy — `#0C2086` (majority) and `#0D2086` (minority, exact match to the new Primary) — directly in ~27 files, ~90 occurrences (buttons, selected states, focus rings, chat bubbles, table selection borders, etc.), with no central token for it. When Primary's usage is eventually decided, most of this is likely a find-and-centralize job (into one token, consistent hex) rather than an actual hue change, since the new Primary already matches the app's existing accent almost exactly.

## 3. Typography

The existing app uses a **tri-font system** (documented in `DESIGN.md` §2): `--font-sans` (Inter/Plus Jakarta Sans — body & most UI), `.font-serif-headline` (Playfair Display — the page-title font used on nearly every screen), `--font-mono` (JetBrains Mono — data cells, identifiers). All three are CSS variables set once in `src/index.css`.

**Decided:**
- `--font-sans` → **Metropolis** (provided in `public/Fonts/AvenirFont/Metropolis/`, 18 weight/style `.otf` files). Replaces Inter/Plus Jakarta Sans as the default body/UI font. Chosen because it's purpose-built as a free Avenir substitute, and as a pure workhorse sans (no stylistic alternates) it's the safest universal choice for body text and controls.
- `.font-serif-headline` → **Montserrat Alternates**, heavy weight (SemiBold/Bold), provided in `public/Fonts/AvenirFont/Montserrat/Montserrat-Full-Version/Desktop Fonts/Montserrat Alternates/`. Replaces Playfair Display as the page-title/heading font. Its alternate letterforms give headlines a distinct, logotype-like character versus body text — preserving the serif-vs-sans *contrast* the current design relies on, just with a new sans-vs-sans (workhorse-vs-display) pairing instead.
- `--font-mono` → **unchanged** (JetBrains Mono). None of the provided font files are monospace, so swapping this is not possible without breaking data-table column alignment. Not an open decision — a hard constraint.
- Plain **Montserrat** (also provided, full family + Alternates) is currently unused — available if a third typographic role is ever needed.

**Not yet done:** no `@font-face` declarations or `src/index.css` variable changes have been made. This section records the decision; implementation is a separate future step.

## 4. Border Radius

**No global sweep.** Early in this process a blanket "radius → 0" was considered and explicitly rejected in favor of going incrementally, one surface at a time, because:
- `rounded-full` (37 usages app-wide: avatars, the theme toggle, segmented controls, status dots) must stay circular — a global override would square off things that are functionally circles/pills, not "rounded corners."
- A handful of hardcoded raw-CSS radius values exist outside Tailwind's utility classes entirely (e.g. the data table's sticky header, Sonner toast styling in `index.css`) and need individual handling regardless of any token-level approach.

**Working convention:** express a target radius via Tailwind's existing token scale where it lands exactly on one (avoids introducing arbitrary `rounded-[Npx]` values where a named class already matches). This app runs Tailwind CSS v4.3, whose default `@theme` radius scale is:

| Token | Value |
|---|---|
| `rounded-xs` | 2px |
| `rounded-sm` | **4px** |
| `rounded-md` | 6px |
| `rounded-lg` | 8px |
| `rounded-xl` | 12px |
| `rounded-2xl` | 16px |
| `rounded-3xl` | 24px |
| `rounded-full` | 9999px (always exempt — see above) |

No `@theme` override exists in `src/index.css`, so these are the live values. When a future instruction gives a target in px, check this table first before reaching for an arbitrary value.

## 5. Progress Log

| Date | Surface | Change | Notes |
|---|---|---|---|
| 2026-11-05 | Top Navbar (header strip) | All corner radii → `rounded-sm` (4px) | Scope: logo image frame (`NavigationController.tsx`), `SegmentedControlSharedComponent` (container, buttons, active pill — previously `rounded-xl`/`rounded-lg`/`rounded-md` mixed), `ProfileSettingsSharedComponent`'s collapsed trigger button. Circular elements left untouched (`rounded-full` avatar-initials circle). **Not included**: the profile dropdown panel's own internal content (cards, menu items) — that's a separate surface, not "the navbar" strip itself. **Side effect to be aware of**: `SegmentedControlSharedComponent` is shared — this same radius now also applies everywhere else it's used (`ViewEditModeToggleSharedComponent`'s Mode toggle on Environment Overview/Resources, `ControllBlockSharedComponent`, `AddCustomColorModalController`), not just the navbar. |
| 2026-11-05 | App-wide light mode canvas | `--color-canvas` (`src/index.css`) `#ffffff` → `#f9f9f9` | Light mode only — `.dark`'s `--color-canvas: #000000` untouched. Scoped precisely: this variable only drives `html`/`body`'s background, not individual card surfaces (`CardSharedComponent` etc. hardcode their own `bg-white`), so this doesn't wash out every white surface in the app — just the page canvas behind them. |
| 2026-11-05 | Top Navbar background | No code change needed | The navbar's header bar already hardcodes `bg-white` (`sm:bg-white/90` + blur on larger screens) independently of `--color-canvas` — so once the body became `#f9f9f9`, the navbar was already "a bit lighter than the background" for free, satisfying the request with zero edits. |
| 2026-11-05 | Infrastructure Register, Environment Overview & Subscriptions — data table card | Card wrapper + header bar corner radii → 4px | Applied at `DataTableContainerSharedComponent`'s own `CardSharedComponent` usage (`!rounded-sm` override — `CardSharedComponent` itself, used app-wide for every card, was deliberately left untouched) plus the hardcoded `.data-table-sticky-header-container` corner radius in `index.css` (`0.75rem` → `0.25rem`, needed so the navy header bar's corners don't look clipped against the card's new smaller radius). Explicitly includes **all three** pages that share this component, per instruction — not scoped to just the two named pages. |
| 2026-11-05 | Infrastructure Register & Environment Overview — Search box and Columns dropdown | Corner radii → 4px | `ExpandableSearchSharedComponent` and `ColumnVisibilityDropdownSharedComponent` edited directly (not override-at-call-site), since both are used **only** on these two pages — no other page's search/columns control is affected. |
| 2026-11-05 | Uptime Request page (list + detail) | Cards, search bars, buttons, and dropdowns → 4px | Full sweep of every non-circular corner radius across `EnvironmentUptimeRequestListFeedStaticComponent`, `EnvironmentUptimeRequestCardStaticComponent`, `EnvironmentUptimeRequestFormStaticComponent`, and `EnvironmentUptimeRequestDetailScreenController` (list + detail both included). Shared components involved (`CardSharedComponent`, `ButtonSharedComponent`, `PrimaryActionButtonSharedComponent`, `CustomSelectSharedComponent`, `CustomDateTimePickerSharedComponent`, `InputSharedComponent`) were **not edited directly** — overridden per call site instead (`className`/`triggerClassName`/`dropdownClassName="!rounded-sm"`), since all of them are used on other pages too (Resources, Settings, etc.) that weren't part of this request. |
| 2026-11-05 | Feature Request page | Cards, search bars, buttons, and dropdowns → 4px | Same full-sweep, same-overrides-not-shared-components approach as the Uptime Request page above, applied to `FeatureRequestListFeedStaticComponent`, `FeatureRequestCardStaticComponent`, and `FeatureRequestFormStaticComponent`. Feature Requests has no separate detail page. |

---

**Next**: awaiting the next component/page to target.
