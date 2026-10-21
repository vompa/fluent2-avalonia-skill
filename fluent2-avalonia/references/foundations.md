# Foundations: layout, type, color, elevation, motion, shape, tokens

Read this file when you set up or review a theme, a Styles file, spacing, typography, colors, shadows, animations or corner radii for an Avalonia app. Component behavior lives in the other reference files; this file holds the values they build on.

## Contents

1. [How to read the values](#1-how-to-read-the-values)
2. [Design principles](#2-design-principles)
3. [Layout and spacing](#3-layout-and-spacing)
4. [Typography](#4-typography)
5. [Color and contrast](#5-color-and-contrast)
6. [Icons](#6-icons)
7. [Elevation and material](#7-elevation-and-material)
8. [Motion](#8-motion)
9. [Shape: radius and stroke](#9-shape-radius-and-stroke)
10. [Tokens and themes](#10-tokens-and-themes)
11. [Recording deviations](#11-recording-deviations)
12. [Checklist](#12-checklist)
13. [Sources](#13-sources)

## 1. How to read the values

Every number in this skill carries one of three origins. Keep them apart, because they bind differently.

| Tag | Meaning | Binding |
| --- | --- | --- |
| **Fluent default** | Value taken from Fluent 2 guidance, the Fluent UI React source or WinUI documentation | Starting point only |
| **Derived** | Recommendation built on Fluent principles, but not stated by Microsoft | Judgment call, adjust freely |
| **(derived, not verified)** | Derived and not confirmed against a source or a running app | Check before relying on it |

Rule zero: if the project has a design brief, a token file or an established style, its values win over every Fluent default here. Fluent is the fallback and the source of behavior; it does not replace the brief. Where the project differs from Fluent on purpose, write the deviation down once with the reason (section 11) so later changes do not undo it by accident.

Web values (Fluent UI React) and Windows values (WinUI) differ in places, for example 32 px web fields versus WinUI controls with a different template. The tables below name the platform. For an Avalonia desktop app on Windows, prefer WinUI values for type and geometry and Fluent web values where WinUI is silent, and state which one you chose.

## 2. Design principles

- Natural on each platform: use Windows habits (shortcuts, focus drawing, text input behavior) and build bespoke patterns only where the product needs them. Because users carry platform expectations into every app. ([Principles][f2-principles])
- Built for focus: show as few simultaneous elements as the task allows. Because visual noise costs concentration; a form with fewer visible fields is faster to scan. ([Principles][f2-principles])
- One for all: design for different abilities from the first sketch, not as a late fix. Keyboard use is a first-class path equal to the mouse. ([Principles][f2-principles])
- Brand identity versus behavior: Fluent's signature look (color, sound, illustration) belongs to Microsoft products. Adopt the behavior and keep your own brand look. Because a product that copies the look without being a Microsoft product confuses ownership. ([Principles][f2-principles])
- Prefer the platform control and style it over rebuilding it. Because a built-in control already carries keyboard, automation and text-input behavior you would otherwise have to re-create.

## 3. Layout and spacing

- Build on a 4 px grid. Fluent default steps: 0, 2, 4, 6, 8, 10, 12, 16, 20, 24, 28, 32, 36, 40, 48, 52, 56. The values 2, 6 and 10 are nudges that compensate for icon padding. Because a single grid keeps rhythm predictable; the nudges exist so icons and text still align. ([Layout][f2-layout], [Spacing source][gh-spacing])
- If the project brief allows only the multiples of 4, still permit 2, 6 and 10 for padding around icons and inside fields when the brief itself uses them. Because a brief that lists 4/8/12 but ships 6 and 10 contradicts itself. Derived.
- Measure spacing from the element's bounding box, not from the glyph. Because icons carry internal padding. ([Layout][f2-layout])
- In WinUI all margins and padding are multiples of 4 effective pixels. Column gutter Fluent default: 12 below 640 px window width, 24 above. ([Margin and padding][wl-margins])
- NavigationView reference values (Fluent default, WinUI): content margin 12 in minimal mode and 24 otherwise; header area 52 high; expanded pane from 1008 px width, compact pane 641 to 1007 px, menu button only up to 640 px. Use them as a model for window-size adaptation, not as a mandate. ([NavigationView][wl-nav])
- Web breakpoints for reference: 320, 480, 640, 1024, 1366, 1920. Techniques: reposition, resize, reflow, show/hide, restructure. A desktop-only app needs few breakpoints; define a minimum window size instead and document it. ([Layout][f2-layout])
- Proximity groups, equal spacing marks equal rank, and whitespace replaces divider lines. Align text left and center only icons and short captions under icons. Because grouping by distance is quieter than grouping by lines. ([Layout][f2-layout])
- Touch target Fluent default: 44 x 44 (iOS, web), 48 x 48 (Android). A mouse-and-keyboard desktop app may go lower, but state the minimum (for example 32 or 34 high) and test touch-capable screens separately. Because the target size is a product decision tied to input devices. ([Layout][f2-layout])
- Forms: label above the field, left aligned. Fluent's WinUI example uses 80 px tall fields with 24 between fields and 48 between groups. With compact fields scale the gaps down but keep the group-to-field ratio above about 1.5 to 1. Because the ratio, not the number, is what lets the eye see groups. The behavior at 125 % and 150 % display scaling is not verified here; test it. ([Forms][wl-forms])
- Text line length: 50 to 60 characters, never under 20 or over 60 in running text. Limit help text columns accordingly. ([Typography][wl-typography])

## 4. Typography

Fluent default type ramp for Windows (size/line height in px):

| Style | Size/line | Weight |
| --- | --- | --- |
| Caption | 12/16 | Regular |
| Body | 14/20 | Regular |
| Body strong | 14/20 | Semibold |
| Body large | 18/24 | Regular |
| Subtitle | 20/28 | Semibold |
| Title | 28/36 | Semibold |
| Title large | 40/52 | Semibold |
| Display | 68/92 | Semibold |

([Windows typography][wl-typography])

Fluent default ramp for web: Caption 2 10/14, Caption 1 12/16, Body 1 14/20, Subtitle 2 16/22, Subtitle 1 20/26, Title 3 24/32, Title 2 28/36, Title 1 32/40, Large title 40/52, Display 68/92. Weights: Regular 400, Medium 500, Semibold 600, Bold 700. ([Typography][f2-typo], [Font tokens][gh-fonts])

Rules:

- Use the system font where possible: Segoe UI Variable on Windows, Segoe UI on web. If the brand uses another typeface (for example to match a web UI), choose it deliberately, embed it, and keep the system font as fallback. Because a missing font must degrade to something legible, not to a random default. ([Typography][f2-typo])
- Smallest text: 12 px Regular, 14 px Semibold. Smaller text becomes unreadable in some scripts and at some scaling factors. ([Windows typography][wl-typography])
- Windows ramp uses Regular and Semibold. Medium (500) is a web weight; use it only if the typeface carries it well and the project brief names it. Avoid bold and italic: use Semibold for emphasis, because italics reduce readability for people with dyslexia. ([Windows typography][wl-typography])
- Never convey meaning by bold or italic alone. Because screen readers do not announce emphasis. ([Text][f2-text])
- Sentence case for every UI string including titles; never all-caps for emphasis. Follow the language's own capitalization rules (for example nouns capitalized in German) rather than English title case. Because all-caps text is harder to read and scales badly. ([Typography][f2-typo], [Content design][f2-content])
- Truncating: wrap by default. Use an ellipsis only when the container is clearly bounded or a "show more" affordance follows, and then expose the full text in a tooltip. Do not truncate tags or badges. A project brief may be stricter ("wrap, truncate only table cells and cards with a tooltip"); follow it. ([Windows typography][wl-typography])
- Alignment: left. Do not justify text in UI. ([Windows typography][wl-typography])
- Use tabular (equal-width) digits for numbers that change or line up in columns. In Avalonia that is `FontFeatures` with `+tnum`, if the typeface supports it. Derived.

## 5. Color and contrast

- Think in three palettes: neutral (surfaces, text, layout), shared (accents for avatars, calendars, badges; adjusted saturation in dark mode) and brand. Global tokens hold raw values; alias tokens give meaning ("text on brand"). Components should reference aliases only. Because changing a theme then touches one layer. ([Color][f2-color], [Tokens][f2-tokens])
- Interaction ladder: rest is lightest, hover one step darker, pressed two steps darker, selected darkest; focus does not change color, it thickens or adds an outline. WinUI inverts this (controls get lighter on interaction). Pick one direction project-wide and keep web and desktop consistent if both exist. Because a mixed direction reads as an inconsistency. ([Color][f2-color])
- Status colors only for status: red danger, yellow or orange caution, green positive. Always pair with text or an icon. Warnings are not red when red means error in the same product. Because color alone fails color-blind users. ([Color][f2-color])
- Contrast targets (WCAG): 4.5:1 for text, 3:1 for large text (18.5 px bold or 24 px regular and up) and for non-text elements such as icons and field borders. Fluent states conformance to WCAG 2.1 AA; WCAG 2.2 AA is stricter and a good target. Disabled controls are exempt. ([Accessibility][f2-a11y])
- Field borders: Fluent's normal field border (`#d1d1d1`) has only about 1.53:1 against white (computed with the WCAG formula). Fluent closes the gap with a darker bottom edge (`#616161`, about 6.19:1). If your field border is lighter than 3:1, copy that approach: darken the bottom edge or the whole border. Do not assume a "strong line" token reaches 3:1; measure it. ([Input source][gh-input], [Light alias tokens][gh-lightcolor])
- Placeholder text color Fluent default `#707070` on white, about 4.95:1. ([Input source][gh-input])
- Fluent neutral values (light, Fluent default): text `#242424`, `#424242`, `#616161`, `#707070`; surfaces `#ffffff`, `#fafafa`, `#f5f5f5`, `#f0f0f0`, `#ebebeb`, `#e6e6e6`; borders `#d1d1d1`, `#e0e0e0`, `#f0f0f0`; subtle hover `#f5f5f5`, pressed `#e0e0e0`; overlay `rgba(0,0,0,0.4)`. Use them as a reference scale, not as a palette to paste over a brand. ([Grey tokens][gh-colors], [Light alias tokens][gh-lightcolor])
- Mapping technique: when you describe a Fluent component, map its neutral/brand/status roles to your tokens in one table (surface 1 to your base surface, border 1 to your strong line, brand to your accent, status to your semantic colors) and keep that table next to the theme. Derived.
- Check contrast per state and per theme (light, dark, high contrast), with a script, and keep the numbers in the pull request. Derived.

## 6. Icons

- Regular (outline) icons for orientation; filled icons to mark a selected state. A project with a single stroke set may show selection by surface and text instead; that is a valid deviation. ([Iconography][f2-icons])
- Icons of 12 px are informational only, never interactive. Larger touch-oriented screens need larger icons. ([Iconography][f2-icons])
- Fluent default sizes in source: icon inside a button 20 (small 20, large 24); in an input 20 (small 16, large 24); in a message bar 20; in a WinUI menu item 16. Dense desktop UIs often use 16; that is acceptable if recorded. ([Button source][gh-button], [Input source][gh-input], [Menu source][gh-menu])
- Use literal metaphors named for their shape; put modifiers (plus, check) filled at the bottom right; use one color, inheriting the parent's text color. ([Iconography][f2-icons])
- Every icon-only control needs an accessible name and a tooltip. Icons read differently across cultures, so do not rely on them alone for critical actions. ([Iconography][f2-icons])

## 7. Elevation and material

- Fluent default shadow depths (blur radius): 2, 4, 8, 16, 28, 64; each is a sharp key shadow plus a soft ambient one with one light direction. On Windows, outline strokes replace the key shadow. ([Elevation][f2-elev])
- Typical use (Fluent default): card 2 or 4, tooltip 8, toast 8, menu and listbox 16, drawer 28, dialog 64. Overlaying surfaces carry the higher values. An inline (non-overlaying) side panel needs no shadow, a border is enough. Because shadow communicates "floats above content". ([Elevation][f2-elev], [Toast source][gh-toast])
- Provide a small set of shadow tokens: card, popup (menu, list, tooltip) and dialog. Hover should not add a shadow in a flat design. Derived.
- Materials: Solid is the common one; Smoke dims everything behind a modal (always translucent black, not theme dependent); Acrylic suits short-lived, light-dismiss surfaces; Mica suits the window background. Avalonia transparency support varies by Windows version and compositor; treat Acrylic and Mica as optional and test them. ([Material][f2-material])

## 8. Motion

- Principles: functional, natural, consistent, engaging; bigger elements and longer distances need more time; nothing may feel sluggish or abrupt. Animate to explain a state change, not to decorate. ([Motion][f2-motion])
- Fluent default duration tokens: ultra-fast 50 ms, faster 100, fast 150, normal 200, gentle 250, slow 300, slower 400, ultra-slow 500. WinUI: faster 83 ms, fast 167 ms, normal 250 ms. ([Durations][gh-durations], [Timing and easing][wl-timing])
- Fluent default curves (cubic-bezier): easy ease `0.33, 0, 0.67, 1` for hover and color changes; decelerate `0, 0, 0, 1` for things entering; accelerate `1, 0, 1, 1` for things leaving. WinUI calls them "fast out, slow in" (entering) and "slow out, fast in" (leaving). Because entering elements should arrive quickly and settle, leaving ones should start slowly and go. ([Curves][gh-curves], [Timing and easing][wl-timing])
- Button press: Fluent animates only color, border and text (100 ms) and does not scale. A press scale (for example 0.97) is a legitimate brand choice if recorded; drop it when animations are off. ([Button source][gh-button])
- Input underline: the 2 px brand underline grows from the center in 200 ms (scale 0 to 1, decelerate) on focus and disappears in 50 ms (accelerate) on blur. Nothing else on an input animates. ([Input source][gh-input])
- Transitions that fit: show/hide for menus and dialogs, elevation changes, top-level swaps as a quick cross-fade instead of sliding large elements, container transforms. Skeletons may pulse or wave. ([Motion][f2-motion], [Skeleton][f2-skeleton])
- Reduced motion: honor a "no motion" setting everywhere: durations to about 0, no flashing, no sudden jumps, motion only on the focused element, no information carried by animation alone. Fluent's source drops transitions to 0.01 ms. Include every transition in the switch, including opacity fades and press scales. ([Motion][f2-motion], [Button source][gh-button])
- Do not animate actions the user repeats many times per minute with the keyboard. Derived.

## 9. Shape: radius and stroke

- Fluent default radius tokens: none 0, small 2, medium 4, large 6, XL 8, 2XL 12, 3XL 16, 4XL 24, 5XL 32, 6XL 40, circle 10000. Default controls 4, controls under 32 px high 2, large components 8 or 12. ([Radius tokens][gh-radius], [Shapes][f2-shapes])
- WinUI on Windows 11: 8 for windows, flyouts, dialogs, menus and teaching tips (overlays); 4 for controls (button, checkbox, combo box, text box, list view) and bars (progress, scroll, slider); tooltips 4 as an exception because of their size; 0 where edges meet (two halves of a split button, a flyout touching its trigger). Because radius signals hierarchy: overlays rounder than the controls inside. ([Geometry][wl-geometry])
- A brand may round more softly (for example 8 for controls, 12 for cards). That is fine when recorded as a deliberate deviation. Derived.
- Stroke tokens: thin 1, thick 2, thicker 3, thickest 4 px; round line caps; match stroke to size (small spinner 2, large 4). Focus rings are 2 px. ([Stroke tokens][gh-stroke], [Shapes][f2-shapes])
- Shapes by role: rectangle for buttons, fields and cards; circle for people; pill for slider tracks, switch tracks, tags and badges. ([Shapes][f2-shapes])

## 10. Tokens and themes

- Two layers: global (raw hex, type, radius, stroke, animation) and alias (meaning, such as "text on brand"). Themes: light, dark, high contrast and brand color. ([Tokens][f2-tokens])
- Apply fonts, base colors and theme once at the root (the equivalent of the Fluent provider) and override locally only on purpose. ([Fluent provider][f2-fluentprovider])
- Keep one source of truth for tokens (a design brief plus a token file); do not maintain a parallel Figma kit by hand unless you also keep it in sync. The Windows UI kit lives in Figma, linked from the Fluent 2 Windows page. ([Get started: design][f2-start-design], [Windows][f2-windows])
- In XAML never hard-code hex values in views. Hard-coded colors are the reason high contrast and dark mode fail. See `avalonia-mapping.md`.
- High contrast is not in many design briefs. Treat it as a theme to add: all colors reachable through resources, tested with the OS contrast themes. Whether your Avalonia version maps system colors automatically is (derived, not verified). ([Accessibility overview][wl-a11y])

## 11. Recording deviations

When the project deviates from a Fluent default, write one line in the project's design document:

```text
Deviation: control height 34 instead of Fluent 32.
Reason: larger target for fast desktop use; one height for all controls.
Applies to: buttons, text fields, combo boxes.
Decided: <who>, <date>.
```

Because a deviation without a reason looks like a bug to the next reviewer, and an unrecorded Fluent default looks like an omission. A review should report "differs from Fluent, recorded" as fine and "differs from Fluent, not recorded" as a finding.

## 12. Checklist

- [ ] Spacing values are on the grid or are the documented nudges 2, 6, 10 (search the styles for off-grid numbers).
- [ ] Type sizes follow the ramp; nothing below 12 px; no all-caps strings (search for `TextCaseConverter`, uppercase transforms and upper-case literals).
- [ ] Text contrast is 4.5:1 or better in every theme and state; borders and icons 3:1 (script output attached).
- [ ] Field border contrast is measured, and the bottom edge or border is darkened if the border is under 3:1.
- [ ] Colors come from resources (`DynamicResource`), with no hex literals in views (grep for `#` in `.axaml` view files).
- [ ] Shadow tokens exist for card, popup and dialog; inline panels have none.
- [ ] Every transition uses a token duration and curve; a "reduce motion" switch sets all of them to zero (test).
- [ ] Radius values follow the project tokens; deviations from Fluent are recorded.
- [ ] Every deliberate difference from Fluent has a recorded reason (section 11).

## 13. Sources

[f2-principles]: https://fluent2.microsoft.design/design-principles
[f2-layout]: https://fluent2.microsoft.design/layout
[f2-typo]: https://fluent2.microsoft.design/typography
[f2-color]: https://fluent2.microsoft.design/color
[f2-icons]: https://fluent2.microsoft.design/iconography
[f2-elev]: https://fluent2.microsoft.design/elevation
[f2-motion]: https://fluent2.microsoft.design/motion
[f2-shapes]: https://fluent2.microsoft.design/shapes
[f2-material]: https://fluent2.microsoft.design/material
[f2-a11y]: https://fluent2.microsoft.design/accessibility
[f2-tokens]: https://fluent2.microsoft.design/design-tokens
[f2-content]: https://fluent2.microsoft.design/content-design
[f2-start-design]: https://fluent2.microsoft.design/get-started/design
[f2-windows]: https://fluent2.microsoft.design/components/windows
[f2-fluentprovider]: https://fluent2.microsoft.design/components/web/react/core/fluentprovider/usage
[f2-text]: https://fluent2.microsoft.design/components/web/react/core/text/usage
[f2-skeleton]: https://fluent2.microsoft.design/components/web/react/core/skeleton/usage
[wl-margins]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/alignment-margin-padding
[wl-nav]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/navigationview
[wl-forms]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/forms
[wl-typography]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/typography
[wl-geometry]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/geometry
[wl-timing]: https://learn.microsoft.com/en-us/windows/apps/design/motion/timing-and-easing
[wl-a11y]: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/accessibility-overview
[gh-durations]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/durations.ts
[gh-curves]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/curves.ts
[gh-radius]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/borderRadius.ts
[gh-stroke]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/strokeWidths.ts
[gh-spacing]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/spacings.ts
[gh-fonts]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/fonts.ts
[gh-colors]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/global/colors.ts
[gh-lightcolor]: https://github.com/microsoft/fluentui/blob/master/packages/tokens/src/alias/lightColor.ts
[gh-button]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-button/library/src/components/Button/useButtonStyles.styles.ts
[gh-input]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-input/library/src/components/Input/useInputStyles.styles.ts
[gh-menu]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-menu/library/src/components/MenuItem/useMenuItemStyles.styles.ts
[gh-toast]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-toast/library/src/components/Toast/useToastStyles.styles.ts
