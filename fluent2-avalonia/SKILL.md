---
name: fluent2-avalonia
description: Fluent 2 design rules, component behavior and review checklists for Avalonia desktop apps (AXAML, Styles.axaml, ControlThemes, custom controls, ViewModels). Use this skill whenever a task touches Avalonia UI - building, styling or reviewing text, number, money, time, date, search, combo box and tag-picker fields, buttons, checkboxes, switches, tabs, menus, tooltips, dialogs, side panels, message bars, toasts and progress; keyboard handling (Tab, Enter, Esc, arrows, F6), focus and states (hover, pressed, disabled, error); spacing, typography, color and contrast, motion and reduced motion; high contrast and screen readers (AutomationProperties); UI copy and error messages; AI suggestion panels, drafts and agent actions with correct AI labeling; and headless UI tests of these. Trigger even if the user never says Fluent or design system. Not for web/React, WPF-only or MAUI work.
---

# Fluent 2 for Avalonia

This skill turns Microsoft's Fluent 2 guidance (design system, WinUI documentation, Fluent UI source) into rules, numbers, checklists and test patterns for Avalonia desktop apps. It is an independent, paraphrased summary with a source link on each rule. It does not replace the project's own design brief. Examples target Avalonia 12; [avalonia-12-notes.md](references/avalonia-12-notes.md) lists the Avalonia 11 names, behavior changes and observed pitfalls.

## The one rule that comes first

If the project has a design brief, a token file or an established style, its values win over every Fluent value in this skill. Fluent supplies the behavior and the fallback numbers. When the project deliberately differs from Fluent, record the deviation with its reason in the project (template in [foundations.md](references/foundations.md), section 11), so later edits do not undo it and reviewers can tell "differs on purpose" from "differs by mistake".

Values carry a tag: **Fluent default** (from Fluent guidance or source; starting point only), **Derived** (a recommendation built on Fluent principles; Microsoft does not state it) and **(derived, not verified)** (derived and unconfirmed; check before relying on it). Parser rules for time, date, number and money fields are Derived; Fluent defines the principles, not the masks. Behavior seen in a real project carries **observed** and the Avalonia version.

## How to work

1. **Find the project's own truth first.** Look for a design brief (`design.md`, `DESIGN.md`, `docs/design*`), a token file, `App.axaml`, `Styles.axaml`, `Colors.axaml` or equivalents, and existing controls. Note heights, radii, colors, font and any recorded deviations. Extend what exists; do not start a second style system.
2. **Identify the component or topic and open the matching reference file** (table below). Read only what the task needs.
3. **Implement or fix** using the project's tokens. Use Fluent defaults only where the project is silent, and say which origin a number has when you report it.
4. **Walk the checklist** at the end of the reference section. Every checklist item names how to verify it: a test, a screenshot or a search in the code. Do the check, do not just tick the box.
5. **Test the behavior**, not only the look: keyboard paths, focus, parsers, states. Patterns for headless Avalonia tests are in [avalonia-mapping.md](references/avalonia-mapping.md). State what you did not verify.
6. **Report briefly:** what changed, which rule it follows, which checks ran, which were not possible, and any new deviation to record.

When reviewing existing UI, report findings as a table with columns Finding, Rule (reference file and section), Fix, and mark each as "differs from Fluent, recorded" (fine), "differs from Fluent, not recorded" (decide and record) or "violates a rule" (fix).

## Where to look

- Read [foundations.md](references/foundations.md) when the task involves spacing, grid, type ramp, colors, contrast, shadows, motion durations and curves, radius, tokens, themes, recording deviations.
- Read [inputs.md](references/inputs.md) when the task involves any input: text, multi-line, search, number, money, time, date, combo box, list pick, tag picker; field anatomy; states; Tab, click, Esc, Enter and arrow behavior; parsers.
- Read [buttons-and-selection.md](references/buttons-and-selection.md) when the task involves buttons, split buttons, links, checkbox, radio, switch, slider; choosing a selection control.
- Read [navigation.md](references/navigation.md) when the task involves tabs (also icon-only), navigation pane, breadcrumb, menus and context menus, toolbar and overflow, tooltips, shortcut lists.
- Read [dialogs-and-messages.md](references/dialogs-and-messages.md) when the task involves dialogs, drawers, popovers, message bars, toasts, progress, spinners, skeletons; choosing the right surface.
- Read [data-display.md](references/data-display.md) when the task involves lists, tables, cards, avatars, badges, accordions, dividers, images.
- Read [content-design.md](references/content-design.md) when the task involves labels, titles, buttons text, error text, empty states, tooltips text, number and date formats.
- Read [accessibility.md](references/accessibility.md) when the task involves keyboard model, focus order and restoration, tab order, F6 regions, shortcuts, screen readers and accessible names (name scan), contrast, high contrast, reduced motion, Narrator checks.
- Read [responsible-ai.md](references/responsible-ai.md) when the task involves AI output, drafts, assistants, agents, disclosure, consent, feedback, review scoring.
- Read [avalonia-mapping.md](references/avalonia-mapping.md) when the task involves writing AXAML, selectors for states and value precedence (also style keys of derived controls), resources and themes, template parts (required parts, probing), transitions and the reduced-motion override, layout notes (star columns, shared sizes, margins, minimum heights), transforms from styles, attached behaviors (roving tab stop, toolbar overflow panel, tooltip defaults, F6 regions), automation properties, expand and collapse peers and name scans, focus code, tab order walks, headless tests (also flyouts, keys and commands, scrolling containers, instances built from XAML) and their limits.
- Read [avalonia-12-notes.md](references/avalonia-12-notes.md) when code from Avalonia 11 examples does not compile or behave, an attached property, a selector or a test filter fails in Avalonia 12, or you want the list of observed pitfalls.

Most UI tasks need two or three of these. A new input control, for example, needs [inputs.md](references/inputs.md), [accessibility.md](references/accessibility.md) and [avalonia-mapping.md](references/avalonia-mapping.md).

## Rules at a glance

Foundations (details in [foundations.md](references/foundations.md))

- Use a 4 px grid with 2, 6 and 10 as nudges. A single grid keeps rhythm predictable.
- Type: nothing under 12 px Regular or 14 px Semibold; sentence case; no bold or italic as the only meaning carrier. Small text is unreadable for some users.
- Contrast: 4.5:1 for text, 3:1 for large text, icons and field borders; measure per state and theme. Fluent's own field border is about 1.5:1 and relies on a darker bottom edge, so do not assume a border token passes.
- Color comes from semantic resources, never hex, named colors or shadow colors in views. Hard-coded colors break dark and high-contrast themes.
- Motion explains state changes: hover and color about 100 ms, enter about 200 ms, exit shorter; a reduced-motion setting zeroes everything, including fades and press scales, and a test proves it on every page.

Inputs (details in [inputs.md](references/inputs.md))

- Every input sits in a field with label above, optional help, and error under it; the error names the problem and the way out. Placeholders are examples, never labels, because they vanish on focus.
- Tab into a text field puts the caret at the end; Tab into time, date, number, money and search fields selects all, because those are replaced, not edited. Avalonia 12 selects all by default, so text fields need a behavior.
- Typing is filtered, commit happens on Enter or blur: parse, validate, write the display format back. Never overwrite or clear invalid input silently, and never reformat while typing. After a failed commit the ViewModel must not keep the old value unnoticed.
- Esc ladder: close the open list, then restore the changed field, then close the form. Enter commits the field and runs the default button in single-line fields only.
- Money uses `decimal`, rejects extra fraction digits instead of rounding, and an empty field is not zero.
- Dependent fields (end after start) recompute after commit, not during typing.
- Test composite fields (counter, derived state) also with instances created from XAML, because markup assigns properties one by one and logic that reads two of them can miss the order.

Controls (details in [buttons-and-selection.md](references/buttons-and-selection.md), [navigation.md](references/navigation.md), [dialogs-and-messages.md](references/dialogs-and-messages.md), [data-display.md](references/data-display.md))

- One primary button per layout; labels are verbs; confirmation buttons answer the dialog title ("Delete", "Cancel"), never "OK" or "Yes".
- Disabled controls that are not self-explanatory say why in a tooltip. A tooltip is a logical child of its target, so selectors for text in the target use `>`; open every real tooltip in a test and check its contrast.
- Radio groups are one tab stop with arrows that move and select. Switches act instantly; a form with a Save button uses checkboxes.
- A control that overrides `StyleKeyOverride` is styled through its base type plus a class, not its own type; a test proves the style applies. An expandable header needs a peer that reports expanded or collapsed.
- Tab strips and toolbars never wrap; the overflow goes into a menu that the keyboard reaches.
- Initial focus in a dialog is never on a destructive action (a destructive question puts focus and Enter on Cancel; Esc cancels); focus returns to the invoker on close.
- Field problems stay at the field, persistent conditions go into a message bar, short confirmations into a toast, decisions before data loss into an alert dialog.

Content (details in [content-design.md](references/content-design.md))

- Say what failed, why, and offer two ways forward; no technical detail in the main text. Avoid direction words ("above") and "click here". One term per concept.
- Format numbers, dates and amounts through the culture, with the project's language rules, not English habits.

Accessibility (details in [accessibility.md](references/accessibility.md))

- The keyboard is a first-class path: tab order equals reading order (Tab follows source order, so check it with a test), groups are one tab stop, F6 switches regions, shortcuts appear in tooltips.
- Every element has an accessible name, and Avalonia's derived names are often a type name or empty, so scan them in a test; fields expose required, help and error; messages use live regions (assertive for errors).
- Verify with a screen reader pass and record it; do not rely on code inspection alone.

AI features (details in [responsible-ai.md](references/responsible-ai.md))

- Every AI output carries a visible AI mark and the approved disclaimer; drafts are never sent without a review step; side effects can be declined and undone; each AI function has its own off switch; failures offer a way without AI; anything that prepares or proposes without being asked is an agent.

## Fluent default values at a glance

These are fallbacks. The project's tokens win. Web values come from Fluent UI React source, Windows values from WinUI documentation.

| Item | Fluent default |
| --- | --- |
| Grid | 4 px; nudges 2, 6, 10 |
| Field and button height | 32 (small 24, large 40); WinUI minimum 32 |
| Control radius | 4; WinUI overlays 8; cards 4 to 6 |
| Stroke | 1 px border; 2 px focus underline or ring |
| Body text | 14/20 Regular; caption 12/16; subtitle 20/28 Semibold |
| Hover or color transition | 100 ms, curve `0.33, 0, 0.67, 1` |
| Enter or exit curves | `0, 0, 0, 1` and `1, 0, 1, 1` |
| Input underline | grows 200 ms on focus, leaves 50 ms |
| Tooltip | delay 250 ms, max width 240, gap 4, 12/16 text |
| Menu item | min height 32, width 138 to 300, shadow 16 |
| Dialog | padding 24, width up to 600 (WinUI 320 to 548), shadow 64 |
| Toast | 7 s without action, at most 4 stacked, 16 px apart, shadow 8 |
| Message bar | min height 36, radius 4, icon 20 |
| Progress bar | 2 px (large 4); show only for operations over 1 s |
| Contrast | text 4.5:1, large text and non-text 3:1 |

## Limits and honesty

- Fluent 2 pages were read through summaries; numbers are most reliable where source code or WinUI docs state the same value. The reference files say where a value is derived or unverified.
- Not covered or not verified: visual comparison against real Fluent components, Avalonia support for spell check, on-screen keyboard input scope, Mica or Acrylic, high-contrast color mapping, behavior at 125 and 150 percent display scaling, the Microsoft Writing Style Guide, WinUI TeachingTip, command bar and calendar view, Segoe Fluent Icons.
- Examples target Avalonia 12; the Avalonia 11 names are listed in [avalonia-12-notes.md](references/avalonia-12-notes.md). Items marked **observed** were seen in one Avalonia 12.1.4 desktop project on Windows (Fluent theme) and were not re-checked on Avalonia 11 or on other platforms.
- Avalonia selectors, property names, template parts and the headless API vary by version. Confirm each against the project's version before relying on it, with a compiler error, reflection or a template probe test.
- Fluent is a Microsoft design system; this skill is an independent summary and is not affiliated with or endorsed by Microsoft.
