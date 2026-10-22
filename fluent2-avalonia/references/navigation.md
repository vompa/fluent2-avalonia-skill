# Navigation: tabs, navigation pane, menus, toolbar, tooltips

Read this file when you build or review tab strips, a navigation pane or rail, breadcrumbs, menus and context menus, command bars or toolbars with overflow, tooltips and keyboard shortcut lists in an Avalonia app. If the project brief defines these components, its values win; Fluent supplies behavior and fallback numbers.

## Contents

1. [Tabs](#1-tabs)
2. [Navigation pane](#2-navigation-pane)
3. [Breadcrumb](#3-breadcrumb)
4. [Menu and context menu](#4-menu-and-context-menu)
5. [Toolbar and overflow](#5-toolbar-and-overflow)
6. [Tooltip](#6-tooltip)
7. [Shortcut list](#7-shortcut-list)
8. [Checklists](#8-checklists)
9. [Sources](#9-sources)

## 1. Tabs

Appearance:

- Fluent default (web source): the selected tab shows a 2 px underline in the brand color (3 px for vertical tabs); hover shows a gray underline on unselected tabs; focus a 2 px ring. Heights small, medium, large are about 32, 44 and 52 (derived, not verified: computed from padding plus the 20 px line height). Padding small 6/6, medium 12/10, large 16/10, text 14/20. ([Tab source][gh-tab])
- A segmented strip (tinted track, selected tab as a raised surface with accent text) is a legitimate compact alternative. Record it as a deviation. ([Tablist][f2-tablist])

Behavior:

- Exactly one tab is active, already on first render. The horizontal strip neither wraps nor scrolls: when not everything fits, put the rest into an overflow menu. Tabs use text, text plus icon, or icon only, consistently; icon-only tabs need an accessible name and a tooltip. Labels are short and parallel. The active tab is exposed as current to assistive technology. ([Tablist][f2-tablist])
- Keyboard (WinUI tab view): Left/Right move focus, not selection; Space or Enter selects; arrows do not wrap; Tab leaves the strip; Ctrl+Tab and Ctrl+Shift+Tab select the next or previous tab (with wrap); Ctrl+F4 closes the focused tab; Ctrl+1 to Ctrl+8 select the nth tab and Ctrl+9 the last. Fluent's tablist guidance differs in places (some implementations select on arrow); pick one model per project, and test it. ([WinUI tab view][wl-tabview], [Tablist][f2-tablist])
- One tab stop for the strip: Tab lands on the active tab and the next Tab leaves the strip; arrows move inside. In Avalonia use the roving tab stop recipe in `avalonia-mapping.md`, section 8.
- How many icon-only tabs fit (Derived): count 32 px per tab plus 2 px between tabs, so n tabs need 34 n - 2 px (a 240 px strip holds 7). Show the overflow chip only when not all tabs fit, and reserve its width only then, so a strip that fits shows no chip. The selected tab and the most recently used tab are always among the visible ones: when the overflow takes the last tabs, swap a hidden selected or last-used tab with the last visible one, because a user who cannot see where they are loses the point of the strip. Tabs with text labels need more width each, so the number that fits is smaller and must be measured, not copied from the icon-only count (mechanism: `avalonia-mapping.md`, section 8, Toolbar overflow panel).
- Closeable tabs in a side panel: Ctrl+W as a close shortcut is Derived; ask before closing a tab with unsaved changes (see `dialogs-and-messages.md`).

## 2. Navigation pane

- A list of links to top-level areas, collapsible, at most two levels. Fluent default width 260; inline from 641 px, overlay below. Show the selection. Categories can be an accordion with the chevron in a fixed place. A secondary action (at most one) appears on hover and must be reachable without hover. Tooltips for collapsed items go to the side, not above. Name the landmark "Navigation". ([Nav][f2-nav])
- WinUI NavigationView: expanded from 1008 px window width, compact rail from 641 px, menu button only up to 640 px; header area 52 px. ([NavigationView][wl-nav])
- A narrow icon rail (for example 88 px wide with 12 px labels) is a valid desktop choice. Keep label text at or above the 12 px minimum and put tooltips to the right of the item. Derived. ([Windows typography][wl-typography])
- Switching between main areas: use a short cross-fade rather than sliding large regions. ([Motion][f2-motion])

## 3. Breadcrumb

- Use only when the product has real hierarchy of at least two levels. Does not wrap; overflow goes into a menu starting from the second item (the first and last stay visible); long item names are shortened after about 30 characters with a tooltip up to about 80; mark the last item as current; label the navigation landmark. ([Breadcrumb][f2-breadcrumb])
- If the app has no hierarchical navigation, do not add one.

## 4. Menu and context menu

- Purpose: a list of actions hidden behind a trigger. Entry kinds: action, checkbox, radio. Put frequent actions first and risky ones last, separated; use group headings and dividers; avoid submenus. Shortcut text is secondary content and must trigger the same action. ([Menu][f2-menu], [WinUI menus][wl-menus])
- Surface (Fluent default, source): width 138 to 300, padding 4, gap between entries 2, entry min height 32 with padding 6, radius 4 (WinUI overlays 8), shadow 16. WinUI entry padding 11, 9, 11, 10 and a 16 x 16 icon, shown only for familiar commands. ([Menu source][gh-menu], [WinUI menus][wl-menus], [Geometry][wl-geometry])
- Keyboard: open with Enter, Space or Alt+Down (Alt+Down: derived, not verified) at the trigger; in a popup the arrows move between entries and wrap at the ends (popups only), Esc closes and returns focus to the trigger; in a split menu Right enters the second part and Left closes a submenu. A context menu also opens from the keyboard (Menu key or Shift+F10, a Windows convention; derived, not verified against the cited pages). ([WinUI menus][wl-menus], [WinUI keyboard][wl-keyboard])
- Which entry has focus on open (observed, Avalonia 12.1.4): a menu opened from the keyboard already has focus on its first entry, so Down moves to the second entry and Up as the first key moves to the last. The textbook sequence "Down selects the first entry" describes a menu in which nothing is focused yet. In a test, open the menu from the keyboard, record which entry is focused, then assert the arrow results from that state; do not assert the textbook sequence without looking.
- Entries without an icon (observed, Avalonia 12.1.4): Avalonia does not reserve the icon column by itself, so an entry without an icon starts further left than its neighbors and the texts do not line up. Give those entries an empty 16 x 16 `Border` as their `Icon`; the texts then stay flush. The entry text must never be wider than the menu surface: shorten it with an ellipsis (`TextTrimming="CharacterEllipsis"`) or raise the menu width, and test it with the longest label of the app, in every language (measure the entry's text against the surface width and assert that it fits).
- Never put the main action of a split button into its own menu again. ([Button][f2-button])
- Do not use a menu for a single action. Use a button.

## 5. Toolbar and overflow

- A toolbar gives quick access to frequent actions. It never wraps. When space runs out, the last items move into an overflow menu ("..." or "More") that shows text labels for icon-only items. Group related items with a divider or spacing; keep destructive actions apart; name each toolbar if there are several; icon buttons have a tooltip plus an accessible name. ([Toolbar][f2-toolbar])
- Prefer an overflow menu over horizontal scrolling of a header row. Because everything stays reachable by keyboard and nothing hides off-screen. Derived from the guidance above; a project brief that chose scrolling must record why.
- In Avalonia, drive overflow from the available width: a panel measures its children, keeps those that fit and moves the rest into the menu, last item first (sketch in `avalonia-mapping.md`, section 8). Keep the overflow button in the tab order. Test at the minimum window width and at a wide width, where the items must return.
- Items that do not fit stay in the tree, so hide them completely: `IsEnabled` false, `Opacity` 0 and a 0 x 0 size. With less, the invisible items stay in the tab order and clog it (observed, Avalonia 12.1.4).
- The menu shows a text label for every hidden item, including icon-only ones. Rebuild its entries after layout, not during measure.

## 6. Tooltip

- Content: supplementary, short, plain text, never essential. Shown on hover and on keyboard focus. Gap to the target 4 px; delay 250 ms in and out, but 0 ms when another tooltip is already open; Esc hides it (WinUI also Ctrl). Fluent default surface: padding 4/11/6/11, radius 4, 12/16 text, max width 240. ([Tooltip][f2-tooltip], [Tooltip source][gh-tooltip], [Tooltip logic][gh-tooltipbase], [WinUI tooltips][wl-tooltips])
- Text rules: for an unlabeled control a short noun phrase; for an active control what it enables; for a disabled control when it becomes available; do not repeat visible text; no interactive controls inside; name the keyboard shortcut. Icon buttons that run an action may read better as verb plus object ("Move appointment"); that deviates from Fluent's noun-phrase advice and is a reasonable project choice. ([Tooltip][f2-tooltip], [WinUI tooltips][wl-tooltips])
- A tooltip is a logical child of its target (observed, Avalonia 12.1.4): descendant selectors of the target, such as `Border.chip TextBlock`, also match the text inside the tooltip. Seen: a tooltip text painted in the target's text color on the dark tooltip surface, contrast 1.0. Write selectors for text inside a target with the child combinator (`Border.chip > TextBlock`), or give the tooltip's own text a style that sets the foreground explicitly. Test: open every real tooltip of the page, not only the one on a fresh default button, read the foreground and surface colors of the tooltip text and assert 4.5:1, in light and dark. The test rule is Derived from the observation.
- Recommended Avalonia treatment (Derived): radius 8 (overlay), one shadow token, 12/16 text, `ToolTip.ShowDelay` 250, no tooltip on text that is already fully visible, one tooltip on every icon-only button. Set delay and placement once for the app with the `ToolTip.TipProperty.Changed` pattern in `avalonia-mapping.md`, section 6; `OverrideDefaultValue` on the `ToolTip` properties throws. In headless tests a tooltip appears in the window overlay instead of a popup, so shadow and edge behavior are not verifiable there (`avalonia-mapping.md`, section 10).

## 7. Shortcut list

A shortcut list (help page, popover or sheet that shows the keys of the app) is a reference that is scanned, not read. Derived from Fluent's keyboard and content guidance; Microsoft does not specify the layout.

- Group by area ("Navigation", "Editing", "Calendar"), not alphabetically, and in the order the areas appear on screen. Show each group title at the section-heading step of the project's type ramp (Fluent default: 14/20 Semibold in compact popovers, 16/22 in sheets), above its rows. Because users look for the area first and the key second.
- Two columns per row: keys on the left in a column of fixed width, the description on the right. Share the key column width across all groups, also when each group is its own `ItemsControl` (`Grid.IsSharedSizeScope` and `SharedSizeGroup`, `avalonia-mapping.md`, section 4, Layout notes), so the descriptions start at the same x everywhere.
- Description: at most two lines, ideally one. Shorten the text (verb plus object, "New appointment"); do not rely on wrapping, and set `MaxLines` 2 with an ellipsis as a guard so one long text cannot make the rows uneven. Test with the longest description of the app.
- Key chips: one chip per key, all the same height (and a minimum width, so "N" and "Ctrl" look like the same kind of object), text at least 12 px, written the way the keycap reads ("Ctrl", "Shift", "Esc"). Put a "+" between the keys of a chord and "then" between the steps of a sequence. Chips do not carry meaning by color.
- Hint at the top, one line: "Letter keys work only when no input field has focus." Single-character shortcuts are a hazard for speech input and typing (WCAG 2.1.4, Character Key Shortcuts), so they must not fire while a text field has focus; the hint tells users why a key does nothing while they type. Shortcuts with a modifier work in fields; keep them in the same list.
- Link "Show all shortcuts" from the list to the complete list in the help when the list shows only the common ones. A list that is complete needs no link.
- Accessible: each row reads as one name, keys first and then the action ("Ctrl+N, New appointment"); the single chips are decorative (`AutomationProperties.AccessibilityView="Raw"`, check the value in your version). The list opens from a menu entry or a button with a tooltip, closes on Esc, and focus returns to the invoker (`accessibility.md`, section 3).
- Every key shown matches a real key binding (cross-check in code, `accessibility.md`, section 5); generate the list from the binding table where the app has one, so the two cannot drift apart.

## 8. Checklists

- [ ] Exactly one tab is active at first render (headless test).
- [ ] Left/Right move focus; Enter or Space selects; Ctrl+Tab cycles; no wrap with arrows (headless keyboard test).
- [ ] The strip is one tab stop, on the active tab (headless test: Tab in, Tab out).
- [ ] A tab strip that does not fit shows an overflow menu; it never wraps or scrolls (test at minimum width).
- [ ] Icon-only tabs and rail items have names and tooltips (grep `AutomationProperties.Name`).
- [ ] Navigation items expose the selected state and a landmark name.
- [ ] Menu entries 32 px or the project value; Esc closes; focus returns to the trigger (headless test).
- [ ] Menu opened from the keyboard: the focused entry after open is recorded, and Down and Up behave as described in section 4 (headless keyboard test).
- [ ] Dangerous entries are last and separated; shortcut text equals the real shortcut (cross-check key bindings).
- [ ] Toolbar overflow keeps every command reachable by keyboard (tab through at minimum width); hidden items are not tab stops.
- [ ] Icon-only tab strip: the number of tabs that fit follows 34 n - 2 px; the overflow chip appears only when something is hidden; the selected and the last-used tab are visible at every width (test at three widths).
- [ ] Menu entries without an icon use an empty 16 x 16 icon so the texts line up; the longest label fits the menu surface (test).
- [ ] Every real tooltip of the page (opened, not assumed) has text contrast of at least 4.5:1 in light and dark; selectors for text inside a target use the child combinator `>` (test).
- [ ] Shortcut list: grouped by area, fixed shared key column, descriptions at most two lines, chips of equal height, letter-key hint at the top, link to the full list, every key matches a real binding (screenshot plus cross-check).
- [ ] Tooltips: delay 250 ms; Esc hides; none on fully visible text; disabled controls say when they become available.

## 9. Sources

[f2-tablist]: https://fluent2.microsoft.design/components/web/react/core/tablist/usage
[f2-nav]: https://fluent2.microsoft.design/components/web/react/core/nav/usage
[f2-motion]: https://fluent2.microsoft.design/motion
[f2-breadcrumb]: https://fluent2.microsoft.design/components/web/react/core/breadcrumb/usage
[f2-menu]: https://fluent2.microsoft.design/components/web/react/core/menu/usage
[f2-button]: https://fluent2.microsoft.design/components/web/react/core/button/usage
[f2-toolbar]: https://fluent2.microsoft.design/components/web/react/core/toolbar/usage
[f2-tooltip]: https://fluent2.microsoft.design/components/web/react/core/tooltip/usage
[wl-tabview]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/tab-view
[wl-nav]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/navigationview
[wl-typography]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/typography
[wl-menus]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/menus
[wl-geometry]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/geometry
[wl-keyboard]: https://learn.microsoft.com/en-us/windows/apps/develop/input/keyboard-interactions
[wl-tooltips]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/tooltips
[gh-tab]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-tabs/library/src/components/Tab/useTabStyles.styles.ts
[gh-menu]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-menu/library/src/components/MenuItem/useMenuItemStyles.styles.ts
[gh-tooltip]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-tooltip/library/src/components/Tooltip/useTooltipStyles.styles.ts
[gh-tooltipbase]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-tooltip/library/src/components/Tooltip/useTooltipBase.tsx
