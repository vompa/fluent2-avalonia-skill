# Dialogs, drawers, messages, toasts and progress

Read this file when you build or review confirmation dialogs, side panels (drawers), popovers and info buttons, message bars (info bars), toasts, progress bars, spinners or skeleton loading in an Avalonia app. It also tells you which surface to use for which kind of message. Project briefs win over Fluent defaults; record deviations.

## Contents

1. [Choosing the surface](#1-choosing-the-surface)
2. [Dialog](#2-dialog)
3. [Drawer (side panel)](#3-drawer-side-panel)
4. [Popover and info label](#4-popover-and-info-label)
5. [Message bar](#5-message-bar)
6. [Toast](#6-toast)
7. [Progress, spinner, skeleton](#7-progress-spinner-skeleton)
8. [Checklists](#8-checklists)
9. [Sources](#9-sources)

## 1. Choosing the surface

Use the least intrusive surface that still does the job (Derived from the component guidance below):

| Situation | Surface |
| --- | --- |
| Input is invalid at one place | Field error at the field. Never a dialog. |
| A persistent, page-level condition the user must know about or fix | Message bar |
| A short confirmation of the user's own, completed action | Toast |
| A decision before data loss or a destructive action | Alert dialog |
| Work in a side context next to the content | Drawer (inline) |
| Optional, short explanation next to a control | Popover or tooltip |
| A long-running operation | Progress bar; spinner for short blocking waits |

A toast disappears, so it must not carry information the user cannot find elsewhere. Errors and warnings therefore belong in a message bar, not in a toast. ([Toast][f2-toast], [Message bar][f2-messagebar])

## 2. Dialog

Size and surface (Fluent default, web source): padding 24, gap 8 between parts, radius 8, shadow 64, 1 px transparent border, max width 600, title 20/26 Semibold, backdrop `rgba(0,0,0,0.4)`, at most three footer actions. WinUI content dialog: width 320 to 548, height 184 to 756, padding 24, 18, 24, 24. A brief may choose its own width and radius (for example 420 wide, radius 12, backdrop 0.45); that stays within the WinUI range and is fine when recorded. ([Dialog source][gh-dialogsurface], [Dialog constants][gh-dialogconst], [WinUI dialogs][wl-dialogs])

Kinds and dismissal:

- Modal: closes by outside click, Esc or footer action; without a Cancel button show a close "x". Non-modal: close "x" at top right, Esc, footer (not outside click). Alert: only the footer buttons close it; use it solely for data loss or destructive actions. ([Dialog][f2-dialog])
- Never nest dialogs; one dialog per window (WinUI); at least one operable element; header and footer stay fixed while the content scrolls. ([Dialog][f2-dialog], [WinUI dialogs][wl-dialogs])
- Esc equals the close button and yields "no action". Do not use a dialog for field validation. ([WinUI dialogs][wl-dialogs])

Focus:

- On open, focus the first operable element and trap focus inside; on close, return focus to the element that opened it. The WinUI default button gets brand styling, answers Enter, and receives focus when nothing else can. ([Dialog][f2-dialog], [WinUI dialogs][wl-dialogs])
- Initial focus never lands on a destructive control. For a destructive confirmation, initial focus and Enter go to "Cancel" (Derived); for a neutral confirmation the primary button is fine. ([WinUI keyboard][wl-keyboard])
- After closing any transient surface (menu, popover, list) focus must not be lost. In Avalonia the focused element disappearing leaves no focus owner, and key events then miss your view; restore focus deliberately (see `avalonia-mapping.md`). ([Accessibility][f2-a11y])

Content:

- Title states the main point, sentence case, no final punctuation except a question mark, never just "Error". Text puts the consequence first and does not repeat the title. Buttons are the concrete answer to the title in one or two words. ([Dialog][f2-dialog], [WinUI dialogs][wl-dialogs])
- Example: title "Delete this appointment permanently?", text "After this it is no longer in the calendar.", buttons "Delete" (destructive, first in WinUI order) and "Cancel" (safe, last). Not "Yes", "No" or "OK". Derived.
- Button order in a destructive dialog follows WinUI (action left, cancel right); see `buttons-and-selection.md`.

## 3. Drawer (side panel)

- Inline (next to content, no dim) or overlay (modal, dimmed). Structure: header (title, close, quick actions), body, footer (primary button at the start). Header and footer stay fixed. Four widths: small (default), medium, large, full. Warn before data loss on close; keep multi-step flows to two or three steps; show an error as a message bar at the top of the drawer; navigation drawers sit on the left, notification drawers on the right. ([Drawer][f2-drawer])
- An inline side panel needs no overlay shadow; a left or right border is enough. ([Elevation][f2-elev])
- Closing a panel or tab that has unsaved changes should ask "Discard changes?" with "Discard" and "Keep editing" (Derived). ([Drawer][f2-drawer])

## 4. Popover and info label

- Popover: a small surface at its trigger. Never for essential information. Do not nest popovers; Esc closes; trap focus when interactive; limit size and scroll in one direction; a "learn more" link goes at the bottom. ([Popover][f2-popover])
- Info label: label plus an info button (small, 20 x 20, at least 2 px away from other controls). Enter opens, Esc closes. More than two or three per area signals a design problem. Content is one or two short sentences with at most one link having concrete text. ([Info label][f2-infolabel])

## 5. Message bar

Sizes (Fluent default, source): min height 36, left padding 12, 1 px border, radius 4, icon 20 with 8 gap; multi-line adds top padding 10 and puts actions below; close "x". WinUI InfoBar has icon, title, text, action and close. A denser brief may use radius 8 and a 16 px icon. ([Message bar source][gh-messagebar], [WinUI InfoBar][wl-infobar])

Kinds and duties:

- Info (action optional), Success, Warning (does not block; needs a button or link), Error (blocks; needs a button or link). Every bar is dismissible. An unresolved warning or error returns in the next session. The bar never blinks. To make screen readers notice a changed message, close and reopen it. ([Message bar][f2-messagebar], [WinUI InfoBar][wl-infobar])
- Placement: on a page, under the command bar inside the content, not above the navigation; in a container at the start or under the title; in tabs directly under the tab strip; in a card under the header; in a form at the top; in a drawer under the header. On a failed submit Fluent gives the bar focus while the fields additionally mark their errors. A form-level rule may instead focus the first invalid field; choose one and record it. ([Message bar][f2-messagebar])
- Stacking: sort errors, warnings, successes, info; collapse extras behind a count; "Dismiss all" becomes "Dismiss" when one remains; show a one-line form when space is short. A project may cap the visible bars (for example two, then "n more"). ([Message bar][f2-messagebar])
- Text: optional short concrete title in sentence case (not "Warning"); body one or two short sentences, at most two lines of about 100 characters; success without the word "success" or "successfully" and without congratulation; link text starts with a verb, never a URL, never "click here"; the button is usually a verb, with a noun if ambiguous. ([Message bar][f2-messagebar])
- Assistive technology: errors and warnings use an assertive live region, info and success a polite one. Use assertive sparingly. In Avalonia that is `AutomationProperties.LiveSetting`. ([Message bar][f2-messagebar])

## 6. Toast

- Surface (Fluent default, source): padding 12, radius 4, shadow 8, title 14/20 Semibold, body 14/20 Regular, subtitle 12, icon 16 with 8 gap, action link 12 from the text. Without an action it disappears after 7 s and pauses under the mouse; a toast without an action takes no keyboard focus; a close "x" only if the information can be found again; at most 4 at once with 16 px between; a fixed corner; never for required actions or critical information. A brief may choose another duration (for example 5 s, 8 s with "Undo"), position and radius, and one toast at a time; record that. ([Toast][f2-toast], [Toast source][gh-toast])
- Text: short title; confirmations as past participle with the verb last ("File saved"); progress in progressive form with the noun; combine similar events into one toast; no period; body at most about 60 characters; button one or two words, at most 36 characters. ([Toast][f2-toast])
- Progress in a toast: "X minutes remaining", "X percent complete", "X of Y MB"; determinate duration uses a bar with percent, otherwise a spinner, never both. ([Toast][f2-toast])
- Provide "Undo" in the toast for reversible changes. Derived.

## 7. Progress, spinner, skeleton

- Progress bar: only for operations longer than a second. Determinate is preferred; indeterminate only for short waits; static when paused. One bar for all steps and it never moves backward. Optional label, status text in progressive form with a trailing ellipsis preceded by a space ("Checking records ..."); avoid time estimates; an error replaces the status text with the problem plus the next step. Thickness 2 px (large 4), radius 4, 0.3 s change; cancel via a close button at the top right. WinUI: a ring blocks, a bar does not; ring at least 20 x 20. ([Progress bar][f2-progressbar], [Progress source][gh-progress], [WinUI progress][wl-progress])
- Spinner: from 1 second; centered over the loading area; blocks nothing else; a label from 3 seconds; label concrete, reuses the verb of the triggering action, at most three words, ending with an ellipsis preceded by a space; avoid vague words such as "Loading". A brief may allow only a text "Loading ..." instead of a spinner; that already follows the label rules. ([Spinner][f2-spinner])
- Skeleton: for loads over about a second when the structure is known; wave (default) or pulse; start all queries at once; do not skeleton fixed elements; mark the region busy and do not lose focus after loading. ([Skeleton][f2-skeleton])

## 8. Checklists

### Dialog

- [ ] Width within 320 to 548 (or the project value); padding and radius from tokens; screenshot light and dark.
- [ ] Title states the point; buttons are concrete answers; no "OK", "Yes", "No" (grep resource strings).
- [ ] Initial focus is on Cancel for a destructive dialog; Esc cancels; Enter follows the focused button (headless test).
- [ ] Focus returns to the invoker after close (test).
- [ ] Never opens from a dialog; at most one open (test).
- [ ] Field errors never appear in a dialog (review).

### Message bar and toast

- [ ] Errors and warnings are message bars with an action; toasts carry nothing the user cannot find again.
- [ ] Message text length within the limits (unit test on resource strings).
- [ ] Live region setting: assertive for error/warning, polite for info/success (test the property).
- [ ] The toast timer pauses under the mouse and is the project value; reduced motion removes the slide.
- [ ] Unresolved warnings and errors return after restart (test, if the product keeps this rule).

### Progress

- [ ] Determinate bar for known duration; it never goes backward (test with a fake progress source).
- [ ] Spinner or progress appears only after about 1 second (test with a delayed task).
- [ ] Status text uses progressive form and an ellipsis with a space before it.
- [ ] Errors give the problem and the next step, and a way to retry.

## 9. Sources

[f2-toast]: https://fluent2.microsoft.design/components/web/react/core/toast/usage
[f2-messagebar]: https://fluent2.microsoft.design/components/web/react/core/messagebar/usage
[f2-dialog]: https://fluent2.microsoft.design/components/web/react/core/dialog/usage
[f2-drawer]: https://fluent2.microsoft.design/components/web/react/core/drawer/usage
[f2-popover]: https://fluent2.microsoft.design/components/web/react/core/popover/usage
[f2-infolabel]: https://fluent2.microsoft.design/components/web/react/core/infolabel/usage
[f2-progressbar]: https://fluent2.microsoft.design/components/web/react/core/progressbar/usage
[f2-spinner]: https://fluent2.microsoft.design/components/web/react/core/spinner/usage
[f2-skeleton]: https://fluent2.microsoft.design/components/web/react/core/skeleton/usage
[f2-a11y]: https://fluent2.microsoft.design/accessibility
[f2-elev]: https://fluent2.microsoft.design/elevation
[wl-dialogs]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/dialogs-and-flyouts/dialogs
[wl-keyboard]: https://learn.microsoft.com/en-us/windows/apps/develop/input/keyboard-interactions
[wl-infobar]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/infobar
[wl-progress]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/progress-controls
[gh-dialogsurface]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-dialog/library/src/components/DialogSurface/useDialogSurfaceStyles.styles.ts
[gh-dialogconst]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-dialog/library/src/contexts/constants.ts
[gh-messagebar]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-message-bar/library/src/components/MessageBar/useMessageBarStyles.styles.ts
[gh-toast]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-toast/library/src/components/Toast/useToastStyles.styles.ts
[gh-progress]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-progress/library/src/components/ProgressBar/useProgressBarStyles.styles.ts
