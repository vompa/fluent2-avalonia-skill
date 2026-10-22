# Buttons, links and selection controls

Read this file when you style or review buttons, split buttons, toggle buttons, links, checkboxes, radio buttons, switches or sliders in an Avalonia app: which type to use, how many primaries, size and spacing defaults, labels, keyboard behavior and states. Values tagged Fluent default give way to the project's brief or tokens; see `foundations.md` section 1.

## Contents

1. [Buttons](#1-buttons)
2. [Links](#2-links)
3. [Checkbox](#3-checkbox)
4. [Radio buttons](#4-radio-buttons)
5. [Switch](#5-switch)
6. [Slider](#6-slider)
7. [Which selection control](#7-which-selection-control)
8. [Checklists](#8-checklists)
9. [Sources](#9-sources)

## 1. Buttons

Types (Fluent): standard (secondary: surface plus border), primary (brand filled, transparent border), outline, subtle (no surface), transparent; shapes rounded, square, circular; variants split, menu button (arrow), compound (title plus description), toggle (on/off). Map your project's names onto these once, for example "ghost" to subtle. ([Button][f2-button], [Button source][gh-button])

Hierarchy and placement:

- One primary button per layout. If more than two buttons share the same importance, make all of them neutral. Because two primaries tell the user nothing about what is expected. ([Button][f2-button])
- Many secondary actions: use outline, subtle or transparent variants. Place the primary at the top or left (right in right-to-left layouts). ([Button][f2-button])
- WinUI: show one or two buttons at once; for more actions prefer a selection control plus one command button. ([WinUI buttons][wl-buttons])
- Dialog order (WinUI): the primary action ("do it") on the left, a second "do it" in the middle, the safe action (close, cancel) at the far right. With several decision buttons the order is affirm, negative, cancel. A lone button sits at the right in a dialog, at the left on a page, and at the right inside a container such as a toast, flyout or list row. A project may keep its own order in form footers and still follow WinUI in confirmation dialogs; record it. ([WinUI buttons][wl-buttons], [WinUI dialogs][wl-dialogs])

Sizes (Fluent default, web source): medium padding 5/12, min width 96, text 14/20 Semibold, border 1, radius 4, icon 20 with 6 spacing; small padding 3/8, min width 64, 12/16 Regular; large padding 8/16, 16/22 Semibold, icon 24; icon-only squares 24, 32, 40. WinUI: min width 120 for short text, at most about 26 characters of text, padding 8, 4, 8, 5. A dense desktop brief may use a single 32 or 34 px height and 16 px icons; if you do, keep one height for buttons and fields and note it. ([Button source][gh-button], [WinUI buttons][wl-buttons])

Labels:

- Use a verb in the active voice, sentence case, no final punctuation. Creating: "New" plus the object ("New item"); in a menu the button says "New" and the entries name the object. Adding existing things: "Add" plus the type. Multi-step: "Next" and "Back". ([Button][f2-button])
- Finishing: "Done" for plain confirmation, "Finish" when a selection is applied by the button, "Close" for informational notices and in error messages (not "OK"), "Got it" for important notices, "Hide" for permanent dismissal. "Cancel" discards changes; "Close" changes nothing; do not pair a close "x" with a "Close" button. ([Button][f2-button])
- In dialogs the button text answers the title question concretely: "Delete" and "Cancel" for "Delete this file?", not "Yes" and "No". ([Button][f2-button], [Dialog][f2-dialog])
- Saving: ideally automatic; otherwise "Save" with feedback by toast or message bar. ([Button][f2-button])
- Keep text short; if a label exceeds about 26 characters, stack the buttons or wrap. ([WinUI buttons][wl-buttons])

States and motion:

- States: rest, hover (one step stronger), pressed (two steps), focus, disabled; Fluent animates only color, border and text for 100 ms with the easy-ease curve, no scale. A press scale (0.97 in 100 ms) is an optional brand touch; remove it when animations are off. ([Button source][gh-button])
- Disabled buttons explain themselves: a tooltip says why and, if possible, when it becomes available ("Available once a name is entered"). Keep the click target inert, not hidden. ([Button][f2-button], [Tooltip][f2-tooltip])
- Toggle button: rest is off, selected is on; show the state by text or an icon as well as color (for example a check mark at the start). ([Button][f2-button])
- Split button: the main action is the label, further actions are in the menu and do not repeat the main one. Inner corners are 0 where the halves meet. On touch both halves open the menu (WinUI). ([Button][f2-button], [WinUI buttons][wl-buttons])
- Keyboard: Enter and Space invoke; Tab order follows reading order; show shortcuts in the tooltip. ([WinUI keyboard][wl-keyboard], [WinUI tooltips][wl-tooltips])
- Contrast: text 4.5:1 and icons 3:1 in every state; the border has no requirement; disabled is exempt. Check text on a gradient primary at both ends. ([Button][f2-button])

## 2. Links

- Types: standard (brand color, most accessible), subtle, inverted, on-brand. Underline on hover, focus and pressed; keep at least two visual cues that it is interactive (color plus underline). ([Link][f2-link], [Link source][gh-link])
- Link text works without context: never "click here", no bare "Learn more" without a subject, no raw URL. Announce when a link opens a new window. ([Link][f2-link])
- Use a link for navigation, a button for an action. Because assistive technology announces them differently. ([Link][f2-link])
- Avalonia: a button styled as a link (`Button.link`) keeps keyboard and automation behavior of a button; make sure its automation role matches what it does. Derived.

## 3. Checkbox

- Size and parts (Fluent default): box 16 (large 20), radius 2, border in the accessible neutral border color (`#616161`), checked fills with brand and a white check, mixed shows a dash, hover darkens the border, disabled dims border and mark. The click target is the box plus 8 px each side; make the whole row clickable including the label, at the project's control height. Label to the right, 4 px gap, 14/20 (or the project's 8 px). In Avalonia the box is `Border#NormalRectangle`; set its border in a `:unchecked` style, because a base style loses against the theme's state rule (`avalonia-mapping.md`, sections 3 and 4). ([Checkbox source][gh-checkbox])
- A checkbox does not apply a change immediately; it records a choice that is submitted. Use it for single yes/no or for a group of at most about seven options. ([Checkbox][f2-checkbox], [WinUI checkboxes][wl-checkbox])
- Parent checkbox with child items: the mixed state is set by code only, never by the user directly; children are indented (WinUI example 24). Text at most two lines. Do not place two groups side by side; give each group a group label. ([Checkbox][f2-checkbox], [WinUI checkboxes][wl-checkbox])
- Phrase the label as the statement the check makes true, and avoid negations: "Notify guests by email", not "Do not send email". Because a ticked negative is a double negative. ([Checkbox][f2-checkbox])
- A "select all" box over a list uses the mixed state for partial selection. Derived.

## 4. Radio buttons

- Circle 16; exactly one option; preselect the most sensible one and place it first; vertical layout preferred; about five options at most (WinUI: more than eight belongs in a combo box); offer "None" where it makes sense. ([Radio group][f2-radiogroup], [Radio source][gh-radio], [WinUI radio buttons][wl-radio])
- Labels are short, wrap rather than truncate, sentence case, no final punctuation; the group heading has no colon. Longer explanations go into a subtitle (full sentences) or an info label. ([Radio group][f2-radiogroup])
- Keyboard (WinUI): Tab lands on the selected option, or on the first if none is selected, and focus alone does not select; arrows move focus and select ("selection follows focus"); Ctrl+Arrow moves focus without selecting and Space selects; no wrap at the ends. Screen readers announce name, role, state and position ("2 of 4"). ([WinUI radio buttons][wl-radio], [WinUI keyboard][wl-keyboard])
- Avalonia: make the group one tab stop on the selected option, with arrow navigation that moves and selects and does not wrap; recipe in `avalonia-mapping.md`, section 8 (roving tab stop). The outer ring of the circle is `Ellipse#OuterEllipse` in the Avalonia 12 Fluent template (observed). Derived.

## 5. Switch

- Track 40 x 20, thumb 18 with 2 px gap (small 32 x 16), 1 px border, pill radius, 200 ms easy-ease transition; on state brand fill and white thumb (Fluent default). ([Switch source][gh-switch])
- A switch acts immediately. For settings that need a Save or Apply step, use a checkbox. Because users expect a switch to take effect when flipped. ([Switch][f2-switch], [WinUI toggles][wl-toggles])
- Label: one or two words, a noun (or a verb in its positive state), never "turn on/off", never a question. Place it above or beside the switch, close to it. Extra explanation goes into a tooltip. WinUI: replace the default On/Off text only when needed, with one word. ([Switch][f2-switch], [WinUI toggles][wl-toggles])
- Avalonia: a custom `ToggleSwitch` template needs the parts `PART_SwitchKnob` and `PART_MovingKnobs`, otherwise it does not compile (`avalonia-mapping.md`, section 4).
- Switch versus checkbox: a switch is an action ("online booking active"), a checkbox is a state to be submitted. ([WinUI toggles][wl-toggles])

## 6. Slider

- For approximate or relative values, not for exact input: money, durations and quantities use a field. Needs a label (can be visually hidden), min and max named, value shown in a tooltip, at least four distinct values (otherwise radio buttons). Keep the thumb size constant. ([Slider][f2-slider], [WinUI slider][wl-slider])
- Skip it entirely when the product has no relative values.

## 7. Which selection control

| Situation | Control |
| --- | --- |
| One yes/no choice that is submitted | Checkbox |
| A setting that applies instantly | Switch |
| One of 2 to about 5 options | Radio buttons |
| One of up to about 8 to 12 options | Combo box or select (see `inputs.md`) |
| One of very many | Combo box with type-ahead filter |
| Several of up to about 7 | Checkboxes |
| Several of many | Tag picker or multi-select list (see `inputs.md`) |
| Approximate value on a range | Slider |
| Exact number | Number or money field |

Counts differ between sources (Fluent radio about 5, WinUI combo box from 9, WinUI radio over 8); a project may fix its own thresholds, and then they win. ([Select][f2-select], [Radio group][f2-radiogroup], [WinUI combo box][wl-combobox])

## 8. Checklists

### Button

- [ ] Exactly one primary per view or footer (search the XAML for `Classes="primary"` per view).
- [ ] Label is a verb phrase, sentence case, no trailing period; no "OK", "Yes" or "No" in dialogs (grep resource strings).
- [ ] Every state is drawn and reviewed in light and dark (screenshots): rest, hover, pressed, focus, disabled.
- [ ] Disabled buttons that are not self-explanatory have a tooltip with the reason (test: `ToolTip.Tip` not empty).
- [ ] Icon-only buttons have `AutomationProperties.Name` and a tooltip.
- [ ] Press scale and color transitions are zero when reduced motion is on (test).
- [ ] Shortcuts appear in tooltips and in `AutomationProperties.AcceleratorKey`.

### Checkbox, radio, switch

- [ ] The whole row, label included, is clickable (headless click on the label toggles).
- [ ] Border contrast is 3:1 in rest state (script).
- [ ] Labels are positive statements; "select all" shows the mixed state for partial selection (test).
- [ ] Radio group is a single tab stop on the selected option; arrows move and select; no wrap (headless keyboard test).
- [ ] Switches exist only on views without a save button (review).
- [ ] The switch transition is 200 ms and disabled by reduced motion.

## 9. Sources

[f2-button]: https://fluent2.microsoft.design/components/web/react/core/button/usage
[f2-link]: https://fluent2.microsoft.design/components/web/react/core/link/usage
[f2-dialog]: https://fluent2.microsoft.design/components/web/react/core/dialog/usage
[f2-tooltip]: https://fluent2.microsoft.design/components/web/react/core/tooltip/usage
[f2-checkbox]: https://fluent2.microsoft.design/components/web/react/core/checkbox/usage
[f2-radiogroup]: https://fluent2.microsoft.design/components/web/react/core/radiogroup/usage
[f2-switch]: https://fluent2.microsoft.design/components/web/react/core/switch/usage
[f2-slider]: https://fluent2.microsoft.design/components/web/react/core/slider/usage
[f2-select]: https://fluent2.microsoft.design/components/web/react/core/select/usage
[wl-buttons]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/buttons
[wl-dialogs]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/dialogs-and-flyouts/dialogs
[wl-keyboard]: https://learn.microsoft.com/en-us/windows/apps/develop/input/keyboard-interactions
[wl-tooltips]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/tooltips
[wl-checkbox]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/checkbox
[wl-radio]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/radio-button
[wl-toggles]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/toggles
[wl-slider]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/slider
[wl-combobox]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/combo-box
[gh-button]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-button/library/src/components/Button/useButtonStyles.styles.ts
[gh-link]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-link/library/src/components/Link/useLinkStyles.styles.ts
[gh-checkbox]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-checkbox/library/src/components/Checkbox/useCheckboxStyles.styles.ts
[gh-radio]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-radio/library/src/components/Radio/useRadioStyles.styles.ts
[gh-switch]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-switch/library/src/components/Switch/useSwitchStyles.styles.ts
