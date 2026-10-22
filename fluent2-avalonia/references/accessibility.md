# Accessibility: keyboard, focus, screen readers, contrast, motion

Read this file when you design or review keyboard navigation, focus order and focus restoration, shortcuts, region switching with F6, screen-reader support (UI Automation), contrast, high contrast, text scaling or reduced motion in an Avalonia app. Accessibility applies from the first sketch; it is a build requirement, not a late audit. Avalonia-specific hooks are in `avalonia-mapping.md`.

## Contents

1. [Standard and targets](#1-standard-and-targets)
2. [Keyboard as the primary model](#2-keyboard-as-the-primary-model)
3. [Focus order and restoration](#3-focus-order-and-restoration)
4. [Regions and F6](#4-regions-and-f6)
5. [Shortcuts and access keys](#5-shortcuts-and-access-keys)
6. [Screen readers and UI Automation](#6-screen-readers-and-ui-automation)
7. [Accessible names: scan and fix](#7-accessible-names-scan-and-fix)
8. [Contrast, color and text scale](#8-contrast-color-and-text-scale)
9. [High contrast](#9-high-contrast)
10. [Reduced motion](#10-reduced-motion)
11. [Verification](#11-verification)
12. [Checklist](#12-checklist)
13. [Sources](#13-sources)

## 1. Standard and targets

- Fluent components meet or exceed WCAG 2.1 AA. WCAG 2.2 AA is stricter and a sound target for a new product. ([Accessibility][f2-a11y])
- Text contrast 4.5:1; large text (18.5 px bold or 24 px regular and above) 3:1; non-text elements, icons and field borders 3:1. Disabled controls are exempt. ([Accessibility][f2-a11y])
- A lighter third text tone must meet these ratios too: measure it, with the element's `Opacity` applied, rather than trusting its token name, and restrict it to large text or icons if it falls short. Because "secondary" colors are the usual place where contrast quietly fails. Derived.
- Reflow for web: usable at 400 percent zoom down to 320 px width. For a desktop app, define and document a minimum window size and support text scaling up to 200 percent without clipping. ([Accessibility][f2-a11y])

## 2. Keyboard as the primary model

- Everything operable with a pointer is operable with the keyboard. Tab order equals visual order; only operable elements are tab stops. Inside groups (radio buttons, lists, button strips) use arrow keys for inner navigation, so the group is one tab stop. ([Keyboard accessibility][wl-kbdaccess], [Keyboard interactions][wl-keyboard])
- Key meanings (WinUI): Tab and Shift+Tab move between operable elements; Space activates the focused element (not while typing text); Enter activates buttons and links, opens choice controls and confirms them; in lists Enter does what Space does unless an extra action exists; Esc cancels only transient surfaces and never page navigation; Home and End go to first and last; Page Up and Down move by page; text input outranks all other keys. ([Keyboard interactions][wl-keyboard])
- Initial focus goes to the first or most common element, never to a destructive action. ([Keyboard accessibility][wl-kbdaccess])
- Esc and parents that handle keys in the tunnel phase: a parent view that handles Esc (or Enter, arrows) while the event tunnels down runs before the focused field, so the field never restores its value and the panel closes with the edit lost (observed, Avalonia 12.1.4). Let the parent ask first whether the focused field has a pending change (an attached property such as `HasPendingChange`, true while the text differs from the committed value or a popup is open) and leave the key to the field while it is true. Pattern and code: `avalonia-mapping.md`, section 8.
- Field-level behavior (select on Tab, Esc ladder, commit on Enter) is in `inputs.md`.

## 3. Focus order and restoration

- Logical reading order, headings in a sensible hierarchy, a Z-shaped focus path (left to right, top to bottom). ([Accessibility][f2-a11y])
- Focus must not get lost when a temporary element closes: return it to the invoker after closing a menu, list, popover or dialog. ([Accessibility][f2-a11y], [Dialog][f2-dialog])
- Avalonia pitfall (observed): when the focused control disappears (for example a panel hides itself via its close button), nothing owns keyboard focus and key events never reach your view. Listen at the top level during tunneling, and when no element has focus, restore it and handle the key yourself. Reproduce it in a headless test: close the panel with a click, then press an arrow key.
- Tab follows the order of the elements in the source, not the order on screen (observed, Avalonia 12.1.4). A `DockPanel` child docked at the bottom but written before the content gets focus first, and `KeyboardNavigation.TabIndex` did not change it. Lay the screen out in reading order: a `Grid` with rows, or children written in the order users read them. Because a layout trick that looks right on screen sends keyboard and screen-reader users to the footer first.
- Write the focus order as a numbered list in the pull request for every new screen. Derived from Fluent's advice to specify focus order and screen-reader notes in the design. ([Accessibility][f2-a11y]) Generate the list instead of typing it: a headless test presses Tab, reads the automation name of the focused element at each step and stops when an element repeats; the printed list is the one in the pull request, and the same test pins it afterwards (helper `TabOrder` in `avalonia-mapping.md`, section 8). Derived.

## 4. Regions and F6

- F6 moves between large regions (navigation, header, content, side panel, bottom tabs) and Shift+F6 moves back. Windows does not provide this for custom shells; build it yourself, and give each region an automation name so users hear where they land. ([Keyboard accessibility][wl-kbdaccess])
- Check the F6 key against any global shortcut you already use before binding it. Derived.
- Recipe for nested views (observed, Avalonia 12.1.4; code in `avalonia-mapping.md`, section 8, F6 regions):
  - An attached property marks each region and carries its name; the same text becomes the automation name, so the screen reader announces where focus lands. Nested views mark their own regions the same way.
  - Order the regions as they appear on screen (top to bottom, then left to right), not in tree order, because docked or re-ordered children make the two differ.
  - A page that declares no region counts as one region, so F6 still moves focus into it instead of doing nothing.
  - Each region has an explicit focus target. When the view handles keys itself (calendar, canvas, custom list), the view is the target and must be focusable, because focus on its first button would send the arrow keys to another handler. Without a target, use the first focusable descendant.
  - Test with real keys: press F6 repeatedly and compare the region names with the expected order, including wrap-around; Shift+F6 runs it in reverse. Derived.

## 5. Shortcuts and access keys

- Accelerators (Ctrl+letter) for frequent commands, access keys (Alt+letter) with visible key hints. Show the shortcut in the tooltip and expose it as `AutomationProperties.AcceleratorKey`. WinUI tooltips show accelerators by default. ([Keyboard accessibility][wl-kbdaccess], [WinUI tooltips][wl-tooltips])
- Every shortcut gets a tooltip or menu text, and every tooltip that names a shortcut matches a real key binding (cross-check in code). Derived.
- Single-character shortcuts (letter keys without a modifier) act only while no input field has focus; otherwise they break typing and speech input (WCAG 2.1.4, Character Key Shortcuts). Say so in the app's shortcut list, whose layout and rules are in `navigation.md`, section 7. Derived.

## 6. Screen readers and UI Automation

- Every element needs an accessible name. Controls report role and content through automation peers; choose the correct text element type, because a wrong role confuses assistive tech. Use more than one cue (icon plus color, sound plus blink). ([Accessibility overview][wl-a11y])
- Avalonia fills in names from content, and the result is often wrong without any visible sign; scan for it (section 7).
- Fields: name, role, value, required, help, error. Range controls report minimum, maximum, value and unit. Radio buttons announce position ("2 of 4"). ([Spin button][f2-spin], [WinUI radio buttons][wl-radio])
- Images: informative ones have short alt text; decorative ones are hidden. Icon buttons have a name. ([Image][f2-image], [Iconography][f2-icons])
- Messages: errors and warnings are assertive live regions, info and success polite. ([Message bar][f2-messagebar])
- Do not carry meaning by bold, italic or color alone. ([Text][f2-text])
- Write text that is short, descriptive and self-contained, so each sentence works when read alone. ([Accessibility][f2-a11y])

## 7. Accessible names: scan and fix

Avalonia derives an accessible name from the content when no name is set, and the result cannot be seen in the source, in a screenshot or in the running window (observed, Avalonia 12.1.4):

- A button whose content is not plain text (a panel with an icon and a label) is announced with the content's type name, for example "Avalonia.Controls.StackPanel".
- A list row without a name is announced with the type name of its model.
- A field without a label or name has no name at all.

So treat names as something to test, not to review by eye.

- Scan every page in a headless test: create the automation peer of each focusable, visible element, read its name, and fail on an empty name or a name that looks like a type name; print all findings at once, with the page, the element type and its name. Code: `avalonia-mapping.md`, section 9, Finding and fixing accessible names. The scan reads the same peer a screen reader reads, which is why it finds what a code review cannot. Derived.
- Required fields: read `AutomationProperties.GetIsRequiredForForm(field)` in tests. The peer does not report the value (observed), so a test through the peer passes or fails for the wrong reason.
- Buttons with non-text content: derive the name from the first text inside the content (an attached property or a helper), and set an explicit name where there is no text, because an icon-only button has nothing to derive from. An explicit name always wins.
- List rows: a style setter on the row container binds the name to the row's ViewModel (the setter needs `x:DataType` on the `Style`; `avalonia-12-notes.md`, section 2), so every row announces its content and not the model type.
- Help and info buttons: name the topic, "Help on delivery date", not only "Help". A screen reader's element list shows the names without context, and ten entries called "Help" cannot be told apart. Use the same text for the tooltip, from one localizable resource.
- Fields without a visible label: `AutomationProperties.LabeledBy` to the visible label where there is one, a name otherwise.
- Names are UI text: they follow `content-design.md` (sentence case, no technical terms, one term per concept) and come from resources, not literals.

## 8. Contrast, color and text scale

- Color is never the only carrier of meaning; calendar or status states also carry text or an icon. Red-green color blindness affects roughly 8 percent of men and 0.5 percent of women. ([Color][f2-color])
- Contrast checks cover every state (rest, hover, pressed, focus, disabled where relevant) and every theme. Fluent's own normal field border fails 3:1 (about 1.53:1) and relies on a darker bottom edge; do the same measurement for your borders (see `foundations.md`). ([Input source][gh-input])
- Typography minimums: 12 px Regular, 14 px Semibold. Text scaling to 200 percent without clipping. ([Windows typography][wl-typography], [Accessibility][f2-a11y])

## 9. High contrast

- Windows contrast themes replace your colors. Hard-coded colors prevent that, so use theme resources everywhere and test the OS contrast themes with every view. WinUI documents this in its accessibility overview. Whether and how Avalonia maps system colors in your version is (derived, not verified); verify before promising support. ([Accessibility overview][wl-a11y])
- Add the high-contrast variant as a third theme next to light and dark when the product requires it; plan time for it, because it exposes every hard-coded color. ([Tokens][f2-tokens])
- Keep a visible transparent outline on focus rings and borders (Fluent adds a 2 px transparent outline) so the system theme can paint it. ([Button source][gh-button])

## 10. Reduced motion

- Provide a "no motion" switch honoring the OS setting where available and an in-app setting otherwise. When on: transitions go to zero (Fluent source: 0.01 ms), no flashing, no sudden jumps, motion only on the focused element, and no information carried by animation alone. ([Motion][f2-motion], [Button source][gh-button])
- Include every transition in the switch: opacity fades, press scales, slides, the input underline. Test it by turning the switch on and asserting that no transition has a non-zero duration. Derived.

## 11. Verification

- Run Narrator (or another screen reader) over every new screen: name, role, value, required state and error are announced, and the focus order matches your list. Record the result in the pull request. Add "screen reader checked" to the acceptance list, next to "contrast measured". ([Accessibility overview][wl-a11y])
- Use Accessibility Insights for Windows or the UI Automation inspector for a structural check of names and roles. (Tooling suggestion, Derived.)
- Headless Avalonia tests cover Tab order, F6 regions, key handling, focus restoration and the name scan (section 7); see `avalonia-mapping.md`. They do not replace the Narrator pass.

## 12. Checklist

- [ ] Every interactive element has an accessible name (grep `AutomationProperties.Name` or labeled-by; inspector output).
- [ ] The name scan over all pages finds no empty and no type-like names; list rows, icon buttons and help buttons name their target ("Help on <topic>"); required state is read with `AutomationProperties.GetIsRequiredForForm` (test).
- [ ] Tab order matches reading order; numbered list in the PR, generated by a headless Tab walk that reads peer names and pinned by a test; no bottom-docked element is written before the content.
- [ ] Arrow keys navigate inside groups; each group is one tab stop.
- [ ] Focus returns to the invoker after every transient surface closes (test per surface).
- [ ] Focus is never lost when the focused control hides (test: close a panel, press an arrow key).
- [ ] Esc in an edited field restores the value before an enclosing panel closes, also when the panel handles keys in the tunnel phase (test: edit, press Esc, assert old value and panel still open; press Esc again, assert closed).
- [ ] F6 and Shift+F6 cycle the regions in screen order; each region has a name and a focus target (the view itself where it handles keys); a page without regions is one region (test with real keys).
- [ ] Shortcuts appear in tooltips and `AcceleratorKey`; no tooltip names a shortcut that does not exist.
- [ ] Letter-key shortcuts do not fire while a text field has focus (headless test: type the letter in a field), and the shortcut list says so (`navigation.md`, section 7).
- [ ] Contrast measured for text (4.5:1), large text and non-text (3:1) in all themes and states.
- [ ] No meaning is carried by color or emphasis alone (review plus grayscale screenshot).
- [ ] Reduced-motion switch zeroes all transitions (test).
- [ ] High-contrast themes reviewed on every view, or the limitation documented.
- [ ] Narrator pass recorded for each new screen.

## 13. Sources

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
