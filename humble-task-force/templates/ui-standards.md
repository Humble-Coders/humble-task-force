# UI ticket standards — the visual & interaction Definition of Done

Every UI-facing ticket (a screen, a component, a theme, or any visual change) must satisfy these.
`/draft-ticket` pastes the relevant subset into the ticket as a **`## 🖼️ UI standards`** section and folds
the load-bearing ones into **Acceptance Criteria**. When embedding: adapt the platform names to the project
(Compose for Android, SwiftUI for iOS, Compose-Desktop / your web framework), and drop any line that
genuinely doesn't apply — but the defaults below apply to almost every UI ticket.

## Design fidelity
- [ ] **Match the provided design exactly.** If a design is attached (screenshot / Figma / mockup), it is
      the source of truth — reproduce spacing, sizing, color, type, hierarchy, and every state faithfully.
      Do **not** approximate or silently "improve" it. *(Only when the ticket explicitly says "no design —
      build it yourself" do you design against the brand kit / design system.)*
- [ ] **The design is usually provided by the PM and is NOT attached to the issue.** At `/start-ticket`,
      **ask the PM for the design assets before building** — then build against those + the project's design
      system / brand kit.
- [ ] Use the **design-system tokens** (colors, type scale, spacing, radius, elevation) and
      **reuse/extend the shared components** — no one-off hardcoded colors/sizes, no duplicated component code.

## Theming
- [ ] Support **both light and dark themes.** Every color comes from a theme token defined in *both* modes;
      nothing hardcoded that breaks in the other. Verify the screen in light **and** dark.

## Native components
- [ ] Prefer **native platform components** wherever they achieve the design — top / nav bar, side bar /
      navigation drawer, tab bar, lists, dialogs, pickers, switches, menus, bottom sheets, etc. (Material on
      Android, HIG on iOS, native desktop/web equivalents). Don't hand-roll what the platform already gives you.
- [ ] Where the design **cannot** be achieved with a native component, **tell the PM, explain the trade-off,
      and proceed** with the closest native approach or a minimal custom component — don't silently diverge.

## Layout, insets & responsiveness
- [ ] **Edge-to-edge, strictly.** The app draws to all screen edges — no default letterbox bars or
      system-colored chrome eating the layout. Backgrounds / gradients extend *under* the system bars.
- [ ] **Respect safe areas for content.** No content or interactive element sits under the **status bar,
      notch / display cutout, home indicator**, or — on Android — the **bottom gesture / navigation bar**
      (back / home). Apply the correct insets so content is never obscured or hard to tap; only decorative
      background bleeds underneath.
- [ ] **Responsive** across the supported device sizes (small phone → large phone → tablet / foldable where
      relevant) and **both orientations**. Use adaptive / constraint layouts, not fixed pixel positions; cap
      content width and center on large screens. **Desktop: resizable window with a sensible minimum size and
      a layout that reflows** — no clipping or fixed-size assumptions.
- [ ] **Correct text truncation** — overflowing text **ellipsizes (`…`)** cleanly on the intended number of
      lines instead of clipping, overlapping, or pushing the layout; wrap only where multi-line is intended.

## Input & keyboard
- [ ] **Right keyboard per field** — email keyboard for email, **numeric / number-pad for amounts,
      quantities, phone, OTP, PIN**, secure entry for passwords; set sensible autocapitalize / autocorrect /
      autofill hints (e.g. off for email & password, on for names).
- [ ] **Keyboard action buttons** — show the correct IME action (**Next** moves focus to the next field;
      **Done / Go / Search** submits or dismisses) and wire it up. Where the platform's keyboard lacks a
      return key (e.g. number pads), provide an on-keyboard **Done** via an accessory bar / toolbar.
- [ ] **Keep the focused field visible** — scroll / inset so the keyboard never covers the active field or
      its error message. Dismiss the keyboard on tap-outside / scroll where appropriate.

## States & feedback
- [ ] Define **loading, empty, error, and disabled** states for every screen that fetches or submits data.
      Disable controls and show progress during async work; surface errors inline (design-consistent), never
      as raw text dumps or silent failures.
- [ ] Consistent **press / hover / focus feedback** and transitions from the design system; keep motion
      subtle and **respect reduce-motion**.
- [ ] **Preserve UI state** across rotation / configuration change / process death and desktop resize —
      don't lose form input, scroll position, or selection.

## Accessibility & content
- [ ] Accessible labels / content descriptions on all interactive and informative elements; logical focus order.
- [ ] Respect **dynamic type / font scaling** — layouts don't break at the largest supported font size.
- [ ] Minimum touch-target size (~**48dp** Android / **44pt** iOS); sufficient color **contrast** (aim WCAG AA).
- [ ] **No hardcoded user-facing strings** — everything goes through the project's string / i18n resources.

## Architecture & verification
- [ ] Follow the project's UI architecture (e.g. `/kmp-arch`: native UI per platform, **no business logic in
      the UI**, ViewModels / state drive rendering). No secrets in the UI or committed.
- [ ] **Verify on the range:** smallest + largest supported device (or window size), **light + dark**, and
      the largest font scale — plus every state the design specifies.
