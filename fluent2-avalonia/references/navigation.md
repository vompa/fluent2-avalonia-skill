# Navigation: tabs, navigation pane, menus, toolbar, tooltips

Read this file when you build or review tab strips, a navigation pane or rail, breadcrumbs, menus and context menus, command bars or toolbars with overflow, and tooltips in an Avalonia app. If the project brief defines these components, its values win; Fluent supplies behavior and fallback numbers.

## Contents

1. [Tabs](#1-tabs)
2. [Navigation pane](#2-navigation-pane)
3. [Breadcrumb](#3-breadcrumb)
4. [Menu and context menu](#4-menu-and-context-menu)
5. [Toolbar and overflow](#5-toolbar-and-overflow)
6. [Tooltip](#6-tooltip)
7. [Checklists](#7-checklists)
8. [Sources](#8-sources)

## 1. Tabs

Appearance:

- Fluent default (web source): the selected tab shows a 2 px underline in the brand color (3 px for vertical tabs); hover shows a gray underline on unselected tabs; focus a 2 px ring. Heights small, medium, large are about 32, 44 and 52 (derived, not verified: computed from padding plus the 20 px line height). Padding small 6/6, medium 12/10, large 16/10, text 14/20. ([Tab source][gh-tab])
- A segmented strip (tinted track, selected tab as a raised surface with accent text) is a legitimate compact alternative. Record it as a deviation. ([Tablist][f2-tablist])

Behavior:

- Exactly one tab is active, already on first render. The horizontal strip neither wraps nor scrolls: when not everything fits, put the rest into an overflow menu. Tabs use text, text plus icon, or icon only, consistently; icon-only tabs need an accessible name and a tooltip. Labels are short and parallel. The active tab is exposed as current to assistive technology. ([Tablist][f2-tablist])
- Keyboard (WinUI tab view): Left/Right move focus, not selection; Space or Enter selects; arrows do not wrap; Tab leaves the strip; Ctrl+Tab and Ctrl+Shift+Tab select the next or previous tab (with wrap); Ctrl+F4 closes the focused tab; Ctrl+1 to Ctrl+8 select the nth tab and Ctrl+9 the last. Fluent's tablist guidance differs in places (some implementations select on arrow); pick one model per project, and test it. ([WinUI tab view][wl-tabview], [Tablist][f2-tablist])
- One tab stop for the strip: Tab lands on the active tab and the next Tab leaves the strip; arrows move inside. In Avalonia use the roving tab stop recipe in `avalonia-mapping.md`, section 8.
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
- Recommended Avalonia treatment (Derived): radius 8 (overlay), one shadow token, 12/16 text, `ToolTip.ShowDelay` 250, no tooltip on text that is already fully visible, one tooltip on every icon-only button. Set delay and placement once for the app with the `ToolTip.TipProperty.Changed` pattern in `avalonia-mapping.md`, section 6; `OverrideDefaultValue` on the `ToolTip` properties throws. In headless tests a tooltip appears in the window overlay instead of a popup, so shadow and edge behavior are not verifiable there (`avalonia-mapping.md`, section 10).

## 7. Checklists

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
- [ ] Tooltips: delay 250 ms; Esc hides; none on fully visible text; disabled controls say when they become available.

## 8. Sources

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
