# Data display: lists, tables, cards, people, badges, accordions

Read this file when you build or review lists, tables, cards, avatars and personas, badges and pills, accordions, dividers, images or the less common components (tree, carousel, rating) in an Avalonia app. Project briefs win over the Fluent defaults quoted here.

## Contents

1. [List](#1-list)
2. [Table](#2-table)
3. [Card](#3-card)
4. [Avatar and persona](#4-avatar-and-persona)
5. [Badge and pill](#5-badge-and-pill)
6. [Accordion](#6-accordion)
7. [Divider and image](#7-divider-and-image)
8. [Components to skip unless the product needs them](#8-components-to-skip-unless-the-product-needs-them)
9. [Checklists](#9-checklists)
10. [Sources](#10-sources)

## 1. List

- Use for similar, independent items. Patterns: single action, several actions, selection as the main action, selection as a side action (checkbox). A click on the row selects; a click on an action runs only that action. No sub-levels (use a tree), no nested components. Role: listbox when selectable, list otherwise. ([List][f2-list])
- Keyboard: Enter and Space invoke; Right moves into secondary actions, Left or Esc goes back; with selection as a side action Enter runs the main action and Space toggles selection; Home and End jump to first and last; Page Up and Down move by page (the last two are Derived from WinUI list conventions). ([List][f2-list], [WinUI keyboard][wl-keyboard])
- Items are phrased in parallel, similar in length, with no empty or disabled filler entries. WinUI list item min height 40. ([List][f2-list], [WinUI defaults][winui-generic])
- Selected row: brand background 2 and, if you like, a 3 px marker; keep it consistent with the project's tokens.

## 2. Table

- Row height (Fluent default): 44 medium, 34 small, 24 extra small; cell padding 0/8; bottom border 1 px in the second neutral border; selected row brand background 2; focus 2 px outline. A brief may set its own minimum row height (for example 36) and cell padding 12; that sits between small and medium and is fine when recorded. ([Table source][gh-table])
- Numbers right-aligned under a right-aligned header, tabular digits; text left-aligned. Derived.
- Truncation: tables and cards may truncate with an ellipsis if the full text is available in a tooltip. ([Windows typography][wl-typography])

## 3. Card

- A container for one object; parts: preview, header, footer, all optional; vertical or horizontal; Fluent default padding 8, 12, 16 with radius 2, 4, 6; shadow 4; transparent border. A clickable card needs exactly one clear primary action; otherwise make it selectable. Focus modes: off, no-tab, tab-exit, tab-only. ([Card][f2-card], [Card source][gh-card])
- A brief may use radius 12, padding 16 or 20, a visible border and a different shadow. Record it.
- Keyboard for clickable cards: Tab reaches the card, Enter runs the primary action, secondary actions are reached with Right (list rule). Derived.

## 4. Avatar and persona

- Avatar: circle for a person, square for a group; Fluent default medium size 32 (a 30 px initials circle is close; the 4 px grid favors 32). Presence badge and activity ring are available. A presence badge at 32 px or less is hard to see: also state the status as text or a tooltip. Avatar groups (spread, stack, pie) show an overflow avatar with a count from five on. ([Avatar][f2-avatar], [Avatar group][f2-avatargroup])
- Persona: avatar plus presence plus up to four text lines, one word or short phrase per line. ([Persona][f2-persona])

## 5. Badge and pill

- Height 20 (small 16, large 24, extra large 32; tiny 6 and extra small 10 are dots), minimum width equal to the height, circular radius, 12 px text. One or two words, sentence case, never truncated, sizes not mixed, placed close to the thing described. Icon-only badges need an accessible label. Warning or danger colors come with a way to resolve. No focus unless it has a tooltip. ([Badge][f2-badge], [Badge source][gh-badge])
- The 12 px minimum applies to badge text; a brief with 11.5 px should move to 12. ([Windows typography][wl-typography])

## 6. Accordion

- Groups collapsible sections, all closed by default; opening one closes the previous (several may stay open when that does not overwhelm); chevron at the start or end, consistent; the header is a button with an expanded state; header height at least 44 (small 32), padding 0/12/0/10; header text short, sentence case, no period. Content that is required for the task does not belong inside. ([Accordion][f2-accordion], [Accordion source][gh-accordion])
- Use it for rarely needed form fields ("More details") and open it automatically when it contains filled values. Enter and Space toggle it. A header height of 34 matching the project control height is a legitimate deviation. Derived.
- In Avalonia 12 a `ToggleButton` header does not report expanded or collapsed on its own (observed, Avalonia 12.1.4). Give it a custom automation peer that implements `IExpandCollapseProvider`, with the state from `IsChecked` and `Expand()` and `Collapse()` setting it; sketch and test in `avalonia-mapping.md`, section 9. If the header is a control that overrides `StyleKeyOverride`, style it through the base type plus a class (`ToggleButton.accordion`), not its own type (`avalonia-mapping.md`, section 3, Precedence).

## 7. Divider and image

- Divider: block-wide or inset; keep at least 12 px above and below (8 with an icon); not directly under headings; a pure line is hidden from assistive technology; text inside stays short. Prefer whitespace and group titles to dividers. ([Divider][f2-divider], [Divider source][gh-divider])
- Image: alt text is one or two short sentences that do not repeat the surrounding text; decorative images have none (presentation role); use shadows sparingly. In Avalonia set `AutomationProperties.Name` on informative images and hide decorative ones from automation. ([Image][f2-image])

## 8. Components to skip unless the product needs them

- Tree: for nested data only; indent 24 (medium) or 12 (small), item min height 32 (small 24), at most one or two quick actions, multi-select with a mixed parent state. ([Tree][f2-tree], [Tree source][gh-tree])
- Carousel: auto-play must be pausable; full-screen below 600 px; reflow down to 320 px. ([Carousel][f2-carousel])
- Rating: stars with 3:1 contrast, 16 px up to 20, interactive ones extra large. ([Rating][f2-rating])
- Breadcrumb: only with real hierarchy; see `navigation.md`.

## 9. Checklists

- [ ] Row and item heights meet the project minimum (headless layout test reads `Bounds.Height`).
- [ ] A list has one clear selection model and Home/End/Page keys work (headless test).
- [ ] Numeric columns are right-aligned with tabular digits (screenshot).
- [ ] Status is never carried by an avatar badge alone; text or tooltip exists.
- [ ] Badge text is 12 px or larger and not truncated (screenshot at 200 percent text scale).
- [ ] An accordion header is a button with an expanded state in automation (test: get the peer, check `IExpandCollapseProvider`, `Expand()` and `Collapse()`); it opens by itself when it holds values.
- [ ] Informative images have an accessible name; decorative ones are hidden from automation.
- [ ] A clickable card has exactly one primary action.

## 10. Sources

[f2-list]: https://fluent2.microsoft.design/components/web/react/core/list/usage
[f2-card]: https://fluent2.microsoft.design/components/web/react/core/card/usage
[f2-avatar]: https://fluent2.microsoft.design/components/web/react/core/avatar/usage
[f2-avatargroup]: https://fluent2.microsoft.design/components/web/react/core/avatargroup/usage
[f2-persona]: https://fluent2.microsoft.design/components/web/react/core/persona/usage
[f2-badge]: https://fluent2.microsoft.design/components/web/react/core/badge/usage
[f2-accordion]: https://fluent2.microsoft.design/components/web/react/core/accordion/usage
[f2-divider]: https://fluent2.microsoft.design/components/web/react/core/divider/usage
[f2-image]: https://fluent2.microsoft.design/components/web/react/core/image/usage
[f2-tree]: https://fluent2.microsoft.design/components/web/react/core/tree/usage
[f2-carousel]: https://fluent2.microsoft.design/components/web/react/core/carousel/usage
[f2-rating]: https://fluent2.microsoft.design/components/web/react/core/rating/usage
[wl-keyboard]: https://learn.microsoft.com/en-us/windows/apps/develop/input/keyboard-interactions
[wl-typography]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/typography
[winui-generic]: https://github.com/microsoft/microsoft-ui-xaml/blob/main/dxaml/xcp/dxaml/themes/generic.xaml
[gh-table]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-table/library/src/components/TableRow/useTableRowStyles.styles.ts
[gh-card]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-card/library/src/components/Card/useCardStyles.styles.ts
[gh-badge]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-badge/library/src/components/Badge/useBadgeStyles.styles.ts
[gh-accordion]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-accordion/library/src/components/AccordionHeader/useAccordionHeaderStyles.styles.ts
[gh-divider]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-divider/library/src/components/Divider/useDividerStyles.styles.ts
[gh-tree]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-tree/library/src/components/TreeItemLayout/useTreeItemLayoutStyles.styles.ts
