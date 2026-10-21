# Accessibility: keyboard, focus, screen readers, contrast, motion

Read this file when you design or review keyboard navigation, focus order and focus restoration, shortcuts, region switching with F6, screen-reader support (UI Automation), contrast, high contrast, text scaling or reduced motion in an Avalonia app. Accessibility applies from the first sketch; it is a build requirement, not a late audit. Avalonia-specific hooks are in `avalonia-mapping.md`.

## Contents

1. [Standard and targets](#1-standard-and-targets)
2. [Keyboard as the primary model](#2-keyboard-as-the-primary-model)
3. [Focus order and restoration](#3-focus-order-and-restoration)
4. [Regions and F6](#4-regions-and-f6)
5. [Shortcuts and access keys](#5-shortcuts-and-access-keys)
6. [Screen readers and UI Automation](#6-screen-readers-and-ui-automation)
7. [Contrast, color and text scale](#7-contrast-color-and-text-scale)
8. [High contrast](#8-high-contrast)
9. [Reduced motion](#9-reduced-motion)
10. [Verification](#10-verification)
11. [Checklist](#11-checklist)
12. [Sources](#12-sources)

## 1. Standard and targets

- Fluent components meet or exceed WCAG 2.1 AA. WCAG 2.2 AA is stricter and a sound target for a new product. ([Accessibility][f2-a11y])
- Text contrast 4.5:1; large text (18.5 px bold or 24 px regular and above) 3:1; non-text elements, icons and field borders 3:1. Disabled controls are exempt. ([Accessibility][f2-a11y])
- A lighter third text tone must meet these ratios too: measure it rather than trusting its token name, and restrict it to large text or icons if it falls short. Because "secondary" colors are the usual place where contrast quietly fails. Derived.
- Reflow for web: usable at 400 percent zoom down to 320 px width. For a desktop app, define and document a minimum window size and support text scaling up to 200 percent without clipping. ([Accessibility][f2-a11y])

## 2. Keyboard as the primary model

- Everything operable with a pointer is operable with the keyboard. Tab order equals visual order; only operable elements are tab stops. Inside groups (radio buttons, lists, button strips) use arrow keys for inner navigation, so the group is one tab stop. ([Keyboard accessibility][wl-kbdaccess], [Keyboard interactions][wl-keyboard])
- Key meanings (WinUI): Tab and Shift+Tab move between operable elements; Space activates the focused element (not while typing text); Enter activates buttons and links, opens choice controls and confirms them; in lists Enter does what Space does unless an extra action exists; Esc cancels only transient surfaces and never page navigation; Home and End go to first and last; Page Up and Down move by page; text input outranks all other keys. ([Keyboard interactions][wl-keyboard])
- Initial focus goes to the first or most common element, never to a destructive action. ([Keyboard accessibility][wl-kbdaccess])
- Field-level behavior (select on Tab, Esc ladder, commit on Enter) is in `inputs.md`.

## 3. Focus order and restoration

- Logical reading order, headings in a sensible hierarchy, a Z-shaped focus path (left to right, top to bottom). ([Accessibility][f2-a11y])
- Focus must not get lost when a temporary element closes: return it to the invoker after closing a menu, list, popover or dialog. ([Accessibility][f2-a11y], [Dialog][f2-dialog])
- Avalonia pitfall (observed): when the focused control disappears (for example a panel hides itself via its close button), nothing owns keyboard focus and key events never reach your view. Listen at the top level during tunneling, and when no element has focus, restore it and handle the key yourself. Reproduce it in a headless test: close the panel with a click, then press an arrow key.
- Write the focus order as a numbered list in the pull request for every new screen. Derived from Fluent's advice to specify focus order and screen-reader notes in the design. ([Accessibility][f2-a11y])

## 4. Regions and F6

- F6 moves between large regions (navigation, header, content, side panel, bottom tabs) and Shift+F6 moves back. Windows does not provide this for custom shells; build it yourself, and give each region an automation name so users hear where they land. ([Keyboard accessibility][wl-kbdaccess])
- Check the F6 key against any global shortcut you already use before binding it. Derived.

## 5. Shortcuts and access keys

- Accelerators (Ctrl+letter) for frequent commands, access keys (Alt+letter) with visible key hints. Show the shortcut in the tooltip and expose it as `AutomationProperties.AcceleratorKey`. WinUI tooltips show accelerators by default. ([Keyboard accessibility][wl-kbdaccess], [WinUI tooltips][wl-tooltips])
- Every shortcut gets a tooltip or menu text, and every tooltip that names a shortcut matches a real key binding (cross-check in code). Derived.

## 6. Screen readers and UI Automation

- Every element needs an accessible name. Controls report role and content through automation peers; choose the correct text element type, because a wrong role confuses assistive tech. Use more than one cue (icon plus color, sound plus blink). ([Accessibility overview][wl-a11y])
- Fields: name, role, value, required, help, error. Range controls report minimum, maximum, value and unit. Radio buttons announce position ("2 of 4"). ([Spin button][f2-spin], [WinUI radio buttons][wl-radio])
- Images: informative ones have short alt text; decorative ones are hidden. Icon buttons have a name. ([Image][f2-image], [Iconography][f2-icons])
- Messages: errors and warnings are assertive live regions, info and success polite. ([Message bar][f2-messagebar])
- Do not carry meaning by bold, italic or color alone. ([Text][f2-text])
- Write text that is short, descriptive and self-contained, so each sentence works when read alone. ([Accessibility][f2-a11y])

## 7. Contrast, color and text scale

- Color is never the only carrier of meaning; calendar or status states also carry text or an icon. Red-green color blindness affects roughly 8 percent of men and 0.5 percent of women. ([Color][f2-color])
- Contrast checks cover every state (rest, hover, pressed, focus, disabled where relevant) and every theme. Fluent's own normal field border fails 3:1 (about 1.53:1) and relies on a darker bottom edge; do the same measurement for your borders (see `foundations.md`). ([Input source][gh-input])
- Typography minimums: 12 px Regular, 14 px Semibold. Text scaling to 200 percent without clipping. ([Windows typography][wl-typography], [Accessibility][f2-a11y])

## 8. High contrast

- Windows contrast themes replace your colors. Hard-coded colors prevent that, so use theme resources everywhere and test the OS contrast themes with every view. WinUI documents this in its accessibility overview. Whether and how Avalonia maps system colors in your version is (derived, not verified); verify before promising support. ([Accessibility overview][wl-a11y])
- Add the high-contrast variant as a third theme next to light and dark when the product requires it; plan time for it, because it exposes every hard-coded color. ([Tokens][f2-tokens])
- Keep a visible transparent outline on focus rings and borders (Fluent adds a 2 px transparent outline) so the system theme can paint it. ([Button source][gh-button])

## 9. Reduced motion

- Provide a "no motion" switch honoring the OS setting where available and an in-app setting otherwise. When on: transitions go to zero (Fluent source: 0.01 ms), no flashing, no sudden jumps, motion only on the focused element, and no information carried by animation alone. ([Motion][f2-motion], [Button source][gh-button])
- Include every transition in the switch: opacity fades, press scales, slides, the input underline. Test it by turning the switch on and asserting that no transition has a non-zero duration. Derived.

## 10. Verification

- Run Narrator (or another screen reader) over every new screen: name, role, value, required state and error are announced, and the focus order matches your list. Record the result in the pull request. Add "screen reader checked" to the acceptance list, next to "contrast measured". ([Accessibility overview][wl-a11y])
- Use Accessibility Insights for Windows or the UI Automation inspector for a structural check of names and roles. (Tooling suggestion, Derived.)
- Headless Avalonia tests cover Tab order, key handling and focus restoration; see `avalonia-mapping.md`.

## 11. Checklist

- [ ] Every interactive element has an accessible name (grep `AutomationProperties.Name` or labeled-by; inspector output).
- [ ] Tab order matches reading order; numbered list in the PR (headless test walks Tab and asserts the sequence).
- [ ] Arrow keys navigate inside groups; each group is one tab stop.
- [ ] Focus returns to the invoker after every transient surface closes (test per surface).
- [ ] Focus is never lost when the focused control hides (test: close a panel, press an arrow key).
- [ ] F6 and Shift+F6 cycle the regions; each region has a name (test).
- [ ] Shortcuts appear in tooltips and `AcceleratorKey`; no tooltip names a shortcut that does not exist.
- [ ] Contrast measured for text (4.5:1), large text and non-text (3:1) in all themes and states.
- [ ] No meaning is carried by color or emphasis alone (review plus grayscale screenshot).
- [ ] Reduced-motion switch zeroes all transitions (test).
- [ ] High-contrast themes reviewed on every view, or the limitation documented.
- [ ] Narrator pass recorded for each new screen.

## 12. Sources

[f2-a11y]: https://fluent2.microsoft.design/accessibility
[f2-color]: https://fluent2.microsoft.design/color
[f2-motion]: https://fluent2.microsoft.design/motion
[f2-tokens]: https://fluent2.microsoft.design/design-tokens
[f2-icons]: https://fluent2.microsoft.design/iconography
[f2-dialog]: https://fluent2.microsoft.design/components/web/react/core/dialog/usage
[f2-spin]: https://fluent2.microsoft.design/components/web/react/core/spin/usage
[f2-image]: https://fluent2.microsoft.design/components/web/react/core/image/usage
[f2-messagebar]: https://fluent2.microsoft.design/components/web/react/core/messagebar/usage
[f2-text]: https://fluent2.microsoft.design/components/web/react/core/text/usage
[wl-a11y]: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/accessibility-overview
[wl-kbdaccess]: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/keyboard-accessibility
[wl-keyboard]: https://learn.microsoft.com/en-us/windows/apps/develop/input/keyboard-interactions
[wl-tooltips]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/tooltips
[wl-radio]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/radio-button
[wl-typography]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/typography
[gh-button]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-button/library/src/components/Button/useButtonStyles.styles.ts
[gh-input]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-input/library/src/components/Input/useInputStyles.styles.ts
