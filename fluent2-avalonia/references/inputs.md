# Inputs: fields, states, keyboard behavior, parsers

Read this file when you build, style or review any input: text, multi-line text, search, number, money, time, date, combo box, list pick, multi-select or tag picker. It holds the field anatomy, the state model, a behavior matrix for Tab, click, typing, arrows, Enter, Esc and blur, parser examples and a checklist per field type.

Fluent 2 specifies the anatomy, states, labels, help and error text, and the behavior of spin buttons and the WinUI number, time and date boxes. It does not specify masks or parsers for times, dates, numbers and money. Those parts are marked **Derived** below: a recommended pattern built from Fluent's principles, not a Microsoft rule. If the project brief defines field widths, heights or behavior, the brief wins.

## Contents

1. [Field anatomy](#1-field-anatomy)
2. [States](#2-states)
3. [Behavior matrix](#3-behavior-matrix)
4. [The commit pipeline](#4-the-commit-pipeline)
5. [Single-line text](#5-single-line-text)
6. [Multi-line text](#6-multi-line-text)
7. [Search field](#7-search-field)
8. [Number field](#8-number-field)
9. [Money field](#9-money-field)
10. [Time field](#10-time-field)
11. [Date field](#11-date-field)
12. [Choice fields: select, combo box, list](#12-choice-fields-select-combo-box-list)
13. [Multi-select and tag picker](#13-multi-select-and-tag-picker)
14. [Checklists](#14-checklists)
15. [Sources](#15-sources)

## 1. Field anatomy

| Part | Rule | Source |
| --- | --- | --- |
| Field | A field is label + input + optional info button + help text + validation message. Build one reusable container so name, help and error are bound to the input, not repeated as loose text blocks per screen. Derived: in Avalonia this is a templated control or a `UserControl`. | [Field][f2-field] |
| Label | Above the input by default, for quick scanning; to the left only when space is tight and fields are alike. A short noun phrase, not a sentence or an instruction, sentence case, no trailing colon, no end punctuation. Wrap long labels; do not truncate them. Regular 14/20 (Fluent default). | [Label][f2-label], [Label source][gh-label], [WinUI labels][wl-labels] |
| Required | Mark required fields with an asterisk next to the label and expose "required" to screen readers. If every field is required, one general note before the form is enough; a two-field sign-in needs none. The save button may stay disabled until required fields are filled. Fluent colors the asterisk red; a project may keep it neutral to avoid clashing with the error color (recorded deviation). | [Field][f2-field], [Label source][gh-label], [WinUI forms][wl-forms] |
| Help text | Under the field, explains values or format. 12/16 in the third text tone, 2 px below the field (Fluent default). Fluent writes fragments without a final period and full sentences with one; stay consistent per project. | [Field][f2-field], [Field source][gh-field] |
| Error text | Short and helpful; shows the way forward. Icon 12 px, danger color, 2 px below the field (Fluent default). Fluent's Field shows help and validation separately. A compact form may let the error replace the help line to avoid layout jumps (Derived); either way keep the error visible and never only in a tooltip. | [Field][f2-field], [Responsible AI][f2-rai] |
| Placeholder | A brief hint or a format example, no final period. Never a replacement for the label: it disappears on focus, which burdens people with cognitive impairments, and it must also be exposed as an accessible name or hint. Fluent default placeholder color `#707070`; measure the effective contrast (section 2). | [Input][f2-input], [Input source][gh-input] |
| Width | Roughly the expected input length. WinUI: slightly wider than the longest expected value; if that is very wide, split into two fields. Set `MaxLength` and tell the user when the limit is reached. Document the limit per field (for example name 80, phone 30, email 254). | [Input][f2-input], [WinUI text box][wl-textbox] |
| Size (web source) | Heights 24, 32, 40 (small, medium, large), radius 4, horizontal padding 12 (medium), icons 16, 20, 24, text 14/20. WinUI minimum height 32, minimum width 64. A desktop brief may use 34 or another height; keep one height for all controls. | [Input source][gh-input], [WinUI defaults][winui-generic] |

## 2. States

Fluent distinguishes seven states, each driven by its own tokens. The first five appear in most briefs; add selected and error.

| State | Fluent behavior | Note |
| --- | --- | --- |
| Rest | Surface 1, 1 px border, bottom edge darker (accessible). | See `foundations.md` section 5 on contrast. |
| Hover | Border and bottom edge one step darker; no shadow. | A disabled control does not react. Fluent shows a "not allowed" cursor; a project may keep the default cursor. |
| Pressed | Bottom edge darker still, only when an already-focused field is clicked again. | |
| Keyboard focus | Input: 2 px brand underline that grows in 200 ms and leaves in 50 ms. Button: border plus an inner 1 px ring in the focus color and a transparent 2 px outline for high contrast. Card and table row: 2 px outline. Link: underline. | A project may use a uniform focus ring; for inputs consider adding the underline as well, because focus must also be visible after a mouse click. |
| Disabled | Dedicated disabled colors, exempt from contrast rules. Label, help text and existing error text stay readable; only the input looks disabled. The tooltip should say why and, if possible, how to enable it. | A uniform 0.5 opacity is a valid shortcut; do not dim the label and help text with it. |
| Selected | Tab text and underline in brand color; tag or chip filled with brand background 2 and brand border; row with brand background 2. | |
| Error | Border in the danger color except on focus; message under the field. Fields also support warning (dark orange) and success (green). | A warning such as "a record with this name exists" fits a field better than a blocking error. |

Sources: [Button source][gh-button], [Input source][gh-input], [Field source][gh-field], [Link source][gh-link], [Button][f2-button], [Field][f2-field], [Label][f2-label].

Placeholder contrast: measure the effective contrast, not the token. The 4.95:1 of `#707070` on white holds for the full-strength color. A template can lower the `Opacity` of the placeholder element: the Avalonia 12 Fluent TextBox template halves it, and the placeholder measured about 2.4:1 in the light theme (observed, Avalonia 12.1.4). Compute the ratio from the foreground blended over the field surface by the element's `Opacity`, per theme and per state, and fail the field when it is under 4.5:1. A style cannot change the template's value (`avalonia-mapping.md`, section 3); set it in `TemplateApplied` or choose a darker foreground. Derived.

Decision point, error border on focus: Fluent draws the danger border except while the field has focus, where the border turns neutral and the brand underline takes over; the message under the field stays. A project may keep the danger border while the field is focused, so a user who is correcting the value still sees which field is wrong. Both are valid; record the choice (`foundations.md`, section 11) and cover the focused-with-error state in a screenshot and a test. Derived.

## 3. Behavior matrix

This matrix is the heart of consistent keyboard behavior. Rows are events, columns are field types. Text input always has priority over other keys: Space types a space in a field, it does not "press" anything. ([WinUI keyboard][wl-keyboard])

| Event | Text | Number / money | Time | Date | Choice |
| --- | --- | --- | --- | --- | --- |
| Focus by Tab | Caret at end, nothing selected (edit) | Select all (replace) | Select all | Select all | Focus rectangle |
| Focus by click | Caret at click point | Caret at click point | Caret at click point | Caret; click on the calendar icon opens the picker | Opens the list |
| Typing | Any character | Digits, decimal and group separator, minus only if negative is allowed | Digits, colon, dot, comma | Digits, dot, slash, hyphen | Jumps to the matching item |
| Up / Down | Nothing | Step | Minute step | Day | Moves highlight (open) |
| Page Up / Down | Nothing | Large step | Hour | Month | Page of items |
| Home / End | Line start / end | Minimum / maximum, if set | As number | Nothing | First / last item |
| Enter | Runs the default button (single line) | Commits the value, then the default button | As number | As number | Selects, closes |
| Esc | Restores the last committed value if changed, otherwise closes the form | As text | As text | As text | Closes the list, no change |
| Blur | Checks length and required | Parses, formats, writes display format | Parses, writes `HH:mm` | Parses, writes display format | Commits selection |
| Error | Danger border, message and icon at the field | As text | As text | As text | As text |
| Save with errors | Focus moves to the first invalid field | As text | As text | As text | As text |
| After first error | Re-validate while typing; error clears as soon as the value is valid | As text | As text | As text | - |

All rows are **Derived** from WinUI text-box, number-box and keyboard guidance and from Fluent's spin-button logic ([WinUI text box][wl-textbox], [WinUI number box][wl-numberbox], [WinUI keyboard][wl-keyboard], [Spin button source][gh-spinbuttonlogic]), except where a later section cites a Fluent rule directly. Two sources disagree on where focus goes after a failed submit: Fluent's message bar guidance gives the bar focus, a form-level rule often prefers the first invalid field. Pick one per project and write it down. ([Message bar][f2-messagebar])

The Esc ladder inside a form, from innermost to outermost (Derived): first Esc closes an open list or popup; second Esc restores a changed field to its last committed value; the next Esc closes the form. Esc cancels only transient surfaces, never page navigation. ([WinUI keyboard][wl-keyboard])

## 4. The commit pipeline

All typed fields (number, money, time, date) share one pipeline. Build it once and reuse it. (Derived.)

1. Filter at typing time: reject characters that can never be valid in this field (a letter in a number field) and strip them on paste.
2. Hold the raw text while the user types; do not reformat mid-typing, and do not recompute dependent fields.
3. Commit on Enter or blur: parse the raw text with a tolerant parser (see the tables below) into a typed value (`decimal`, `TimeOnly`, `DateOnly`, `int`) or an error.
4. On success, write the display format back to the text (for example `930` becomes `09:30`) and raise the value to the ViewModel. Dependent fields (end time after start time) recompute only now.
5. On failure, keep the text, mark the field invalid and show a message that names the problem and the valid range. Do not overwrite or clear the input silently; WinUI's "overwrite invalid input" mode swallows the user's text, so do not enable it for important data. ([WinUI number box][wl-numberbox])
   No stale value after a failed commit: the ViewModel must not silently keep the previous valid value while the field shows the rejected text, or a Save button writes a value the user cannot see. Pick one of two patterns by binding style (Derived):
   - String binding (the ViewModel holds the text): publish the raw text with no parsed value, so the ViewModel sees "no value" instead of the old one.
   - Typed binding (`TimeOnly? Value`, `decimal? Amount`): add a read-only `IsValid` flag on the control, bound back to the ViewModel (`Mode=OneWayToSource`), and make every Save-style command report `CanExecute` false while a bound field is invalid. A successful commit and Esc reset the flag.
   Test: type garbage, commit, assert Save is disabled (or the value is empty); type a valid value, commit, assert it is enabled again.
6. Esc restores the last committed value and selects it. ([Spin button source][gh-spinbuttonlogic])
7. An empty field means "no value", which is not zero. Required fields report the empty state as an error; optional ones commit "no value".

Because separating raw text from the parsed value lets a user type freely, makes tests simple (parser unit tests with a table of inputs) and keeps formatting a display concern.

## 5. Single-line text

- Name, address, email, phone, short note. Enter in a single-line field commits the field and triggers the form's default button. Not in multi-line fields. Derived. ([WinUI dialogs][wl-dialogs])
- Tab entry places the caret at the end with nothing selected, because editing is the common case; fast fields (time, number, search) select all, because replacing is. WinUI recommends selecting all when replacement is the main case. Avalonia 12 already selects all on Tab into a single-line `TextBox` (observed, Avalonia 12.1.4), which is the opposite of what text fields need; set the caret to the end with a behavior (`avalonia-mapping.md`, section 6) and test it first. ([WinUI text box][wl-textbox])
- Click always places the caret and selects nothing. Double-click selects a word or a segment, triple-click the line, Ctrl+A everything. Derived.
- Clear (x) button: WinUI shows it in focused, single-line, editable fields that contain text. Use it for search fields; skip it in ordinary form fields where an accidental click could erase required data (Derived). ([WinUI text box][wl-textbox], [Searchbox][f2-searchbox])
- Paste: WinUI's `MaxLength` does not limit pasted text. Intercept paste, remove line breaks and leading or trailing spaces (including non-breaking), truncate to the limit and show a warning such as "Shortened to 80 characters." (Derived.) ([WinUI text box][wl-textbox])
- Read-only fields look editable and confuse users: use them only as a temporary state; keep the text selectable and copyable; if the text is never editable, show a plain text block. ([WinUI text box][wl-textbox])
- Spell check: WinUI turns it on by default; turn it off for names, phone numbers and codes, keep it on for notes and free text. Whether your Avalonia version supports a spell-check switch is not verified here. ([WinUI text controls][wl-textcontrols])
- Input scope: on touch devices an input type (phone, email, number) selects the on-screen keyboard but does not validate. Check whether Avalonia exposes an equivalent for your target; validation stays in the ViewModel. (derived, not verified) ([WinUI text box][wl-textbox])

## 6. Multi-line text

- Fixed height, not growing as the user types; vertical scroll bar on overflow; `resize` only if you want it. WinUI says to set the height and not let the box grow. The source shows min heights 40, 52 and 64 and max heights 200, 260 and 320 for small, medium and large (assignment to sizes is (derived, not verified)). ([Textarea][f2-textarea], [Textarea source][gh-textarea], [WinUI text box][wl-textbox])
- Counter: for limited text show a running count below ("120 characters left"); warn near the limit (for example from 90 percent). Derived. ([WinUI text box][wl-textbox])
- Enter inserts a line break, Tab leaves the field, Ctrl+Enter runs the default button. Derived.
- Plain text only unless rich text is truly required. ([WinUI text box][wl-textbox])

## 7. Search field

- Magnifier icon at the start, clear button only when there is content, optional filter icon; same rest-to-focus transition as the input. On a narrow window collapse to the magnifier and expand on click. ([Searchbox][f2-searchbox])
- On focus show recent searches; while typing, suggestions replace them; most relevant first; related topics after hits. With no result show one line such as "No results" plus the way to create or broaden. ([Searchbox][f2-searchbox], [WinUI auto-suggest][wl-autosuggest])
- Keys: Down moves into the list, Up/Down choose, Enter picks, Esc closes the list, a second Esc clears the field, a third closes the form (Derived). Space in the list does not pick, because text input has priority. ([WinUI keyboard][wl-keyboard], [WinUI auto-suggest][wl-autosuggest])
- Enter or the magnifier submits the query when no suggestion is highlighted. ([WinUI auto-suggest][wl-autosuggest])

## 8. Number field

Fluent's spin button changes a value in small steps and still allows typing. Under about three possible values use a plain input; for imprecise values use a slider. ([Spin button][f2-spin])

- Layout (source): height 24 or 32; two stacked step buttons in a 24 px wide column (each 16 high), left padding 10, radius 4, underline as for inputs. WinUI's number box has optional inline arrows or a compact popup shown on focus. Show arrows only for small ranges (up to roughly 20 steps); omit them otherwise. ([Spin button source][gh-spinbutton], [WinUI number box][wl-numberbox])
- Typing: allow digits, the decimal separator, the group separator, and minus only when the minimum is below zero. Reject everything else at typing time and strip it on paste. Derived. ([WinUI number box][wl-numberbox])
- Keys: Up/Down step (mouse wheel only while focused); Page Up/Down large step; Home/End jump to minimum/maximum if set; holding a key repeats after 300 ms, then accelerates (Fluent source). Shift+Up/Down for the smallest step is Derived. ([Spin button source][gh-spinbuttonlogic], [WinUI number box][wl-numberbox])
- Limits: at minimum or maximum the step button is disabled but the input stays active so the target remains large enough. Clamp on step; do not wrap around. ([Spin button][f2-spin])
- Commit: on Enter and blur the value is accepted and rounded to the step precision; formatting by the user is replaced by the configured format without an error. Out-of-range or non-numeric input is an error that names the valid range, for example "The value is between 0 and 120." (Derived; do not use silent overwrite.)
- Expressions: WinUI can evaluate `+ - * / ^`. Leave that off wherever numbers must be exact or auditable; formulas invite surprises. Derived. ([WinUI number box][wl-numberbox])
- Unit: Fluent puts the unit in the placeholder ("0 cm"). A project may put it in the label ("Duration (min)"), which stays visible while typing. Either is fine; do not do both. ([Spin button][f2-spin])
- Display: tabular digits; value left-aligned in forms (same edge as the label) and right-aligned in table cells. Derived.
- Accessibility: expose range, value and unit through an automation peer (range value pattern, `HelpText` such as "Minutes, 0 to 120"); name the step buttons "Increase" and "Decrease" and take them out of the tab order (mouse only). Derived. ([Spin button][f2-spin])

## 9. Money field

Fluent has no money component. WinUI's number box with a decimal formatter covers currency and fraction digits. ([WinUI number box][wl-numberbox])

- Build it on the number field: no step arrows, fixed fraction digits (usually two), the currency symbol in the label or a suffix, a width suited to the largest amount. Derived.
- Use `decimal` for amounts, never floating point. Do not round silently: more fraction digits than allowed is an error ("At most two decimal places.") because quiet rounding changes the user's number and can break audit requirements. Derived.
- Parsing follows the culture. Example for a locale with decimal comma and group dot: `12` is 12.00, `12,5` is 12.50, `1.234,56` is 1234.56, `1234` is 1234.00 (no automatic decimal shift), `12,505` is an error. Numpad convenience: a single dot followed by one or two digits and no comma reads as a decimal (`12.5` is 12.50). Strip spaces, currency signs and non-breaking spaces on paste. (Derived.)
- Keys: Up/Down plus or minus 1.00, Shift+Up/Down 0.10, Page Up/Down 10.00, Home/End only with limits, wheel only while focused. Derived.
- Display on commit: rewrite in the display format of the locale (example locale: `1.234,50`). Negative amounts in the danger color with a minus sign, not parentheses. Empty is not 0.00; empty means "not given". Derived.

## 10. Time field

WinUI's time picker shows a time and opens a selection overlay; it supports a 24-hour clock, a minute increment and an unset state. Fluent web has no time picker. ([WinUI time picker][wl-timepicker])

Recommended default for fast keyboard entry (Derived): a text field in `HH:mm`, tabular digits, width about 100 px, 24-hour clock, with step equal to the calendar grid (5, 10 or 15 minutes). A picker popup is optional. Decide between tolerant parsing with normalization on blur (recommended: fast typists, easy paste and correction) and a fixed mask `__:__` (harder to edit and paste).

Parser table (24-hour example locale):

| Input | Result |
| --- | --- |
| `9`, `09` | 09:00 |
| `930`, `0930` | 09:30 |
| `9:30`, `9.30`, `9,30` | 09:30 |
| `9:5` | 09:05 |
| `24`, `9:60` | Range error: "The time is between 00:00 and 23:59." |
| `abc` | Format error: "Enter a time such as 09:30." |

A field that explicitly allows "until midnight" may accept 24:00. For 12-hour locales extend the parser with am/pm suffixes and keep the same display rule.

- Allowed characters: digits, colon, dot, comma; at most five characters. Tab selects all; click places the caret; double-click selects hour or minute.
- Keys: Up/Down plus or minus the step, Shift+Up/Down one minute, Page Up/Down one hour, Home/End only with limits (for example opening hours). No wrap past midnight: clamp. Wheel only while focused.
- Commit: on Enter or blur write `HH:mm`. Dependent fields (end after start) recalculate only after commit, never during typing. End before or equal to start is an error on the field: "The end must be after the start." A zero-length range is rarely meant; a field that allows it says so.
- Accessibility: name "Start, time"; the peer reports a time value; a help text states the format. ([WinUI accessibility][wl-a11y])

## 11. Date field

Pick by use. WinUI's date picker (day, month, year) suits known dates such as a birthday where calendar context is irrelevant. The calendar date picker suits dates with context (an appointment, a departure): a calendar overlays the page, shows weekdays, can show load, can be cleared by selecting the chosen day again, and respects min and max dates. ([WinUI date picker][wl-datepicker], [WinUI calendar date picker][wl-calendardatepicker], [WinUI date and time][wl-datetime])

Recommended default (Derived): one date field with two variants, with and without the calendar button. Display format per locale (for example `dd.MM.yyyy`), width about 160 px, calendar icon 16 px at the end. Week starts on the locale's first day, today highlighted.

Parser table (day-first example locale, assuming the current year is 2026):

| Input | Result |
| --- | --- |
| `21102026` | 21.10.2026 |
| `211026`, `21.10.26` | 21.10.2026 |
| `21.10.`, `21.10` | 21.10.2026 (current year) |
| `1.2.26` | 01.02.2026 |
| `30.02.2026` | Error: "30.02.2026 does not exist." |

A two-digit year resolves to the century closest to today; for a birthday it must not land in the future. A date without a year is only valid when the field says so (for example a recurring birthday). A month-first locale needs its own parser order; parameterize it by culture.

- Keys in the field: Up/Down plus or minus one day, Shift+Up/Down one week, Page Up/Down one month, Ctrl+Page one year, wheel only while focused; with no value, Up sets today.
- Opening the calendar: click on the icon. Alt+Down is a common Windows habit for opening choice fields (derived, not verified for date pickers).
- Inside the calendar popup: arrows move by day and week, Page Up/Down by month, Ctrl+Page by year, Home/End to week start/end, Enter selects and closes, Esc closes without change and returns focus to the field. Blocked days (closed, unavailable) are dimmed and explain why in a tooltip. The popup overlays and pushes nothing aside. ([WinUI calendar date picker][wl-calendardatepicker], [WinUI keyboard][wl-keyboard])
- Out-of-range input is an error that states the range ("The date is between 22.10.2026 and 21.10.2027."), not a silent clamp.
- Date and time together: two fields side by side (for example 160 + 100 wide, 12 apart) bound to one value; changing the date keeps the time. ([WinUI date and time][wl-datetime])

## 12. Choice fields: select, combo box, list

- Pick the component by the number of options. Fluent: a native-looking Select for at least four options; Dropdown for custom list styling and multi-select; Combobox for choosing or typing with filter, long lists and free input. WinUI: fewer than five options are better as radio buttons or checkboxes; a long list of rich items is a list view; selection that is secondary and has a default should not dominate the layout. A project rule such as "2 to 4 radio buttons, up to about 12 in a list, beyond that type-ahead" is a valid concrete choice. ([Select][f2-select], [Dropdown][f2-dropdown], [Combobox][f2-combobox], [WinUI combo box][wl-combobox])
- Open and pick: Enter or Alt+Down opens (Alt+Down: observed in the Avalonia 12 ComboBox), Up/Down highlight, Enter picks, Esc closes without change, letters jump to a matching item (also when closed). In WinUI, arrows in an open list do not change the selection until Enter or click; Esc restores the initial value. Fluent says Space picks the highlighted item; apply that only when the user is not filtering, because text input has priority. ([Combobox][f2-combobox], [WinUI combo box][wl-combobox], [WinUI keyboard][wl-keyboard])
- Avalonia 12 ComboBox (observed, Avalonia 12.1.4): the shipped control already opens with Enter and Alt+Down, moves the highlight with the arrows without selecting, closes with Esc without change and jumps to an item by letter, so test these before you write a behavior. With keyboard focus it fills itself with the accent color (template part `HighlightBackground`), which clashes with the neutral field look and lowers text contrast. A style cannot undo it, because the template sets the value itself (`avalonia-mapping.md`, section 3): override the theme resource the part reads, or set the part's `Background` in `TemplateApplied`, and check the result with the template probe (section 4 there) and a focus screenshot.
- Editable: submit on Enter or blur when no item matches; if invalid, show a field error and keep the text, rather than quietly resetting. Valid free input may optionally be added to the list. Derived. ([WinUI combo box][wl-combobox])
- List content: items phrased in parallel (all nouns or all verbs), no shared first word, "None" where meaningful, sentence case. One line per item, logical order: most frequent first, names alphabetical, numbers numeric, dates chronological. Long lists scroll. ([Dropdown][f2-dropdown], [WinUI combo box][wl-combobox])
- Menu and list surface (Fluent default, source): min width 138, max width 300, padding 4, item min height 32, radius 4 (WinUI overlays 8), shadow 16. A list item height of 40 matches WinUI list rows; keep to one project value. Show about eight items before scrolling; the list is as wide as the field (Derived). ([Menu source][gh-menu], [Geometry][wl-geometry])

## 13. Multi-select and tag picker

- Dropdown or combo box multi-select: a checkbox per item, the list stays open until Esc or an outside click, and the choice is shown elsewhere (tags in the field or a count). ([Combobox][f2-combobox], [Dropdown][f2-dropdown])
- Tag picker: typing filters, Up/Down navigate, Enter picks the highlighted or first match, Backspace removes the last chosen tag, optional "remove all", tags wrap. ([Tag picker][f2-tagpicker])
- Tags: plain or interactive, removable; spacing 8 / 6 / 4 px (medium / small / extra small); overflow shown as "+n". ([Tag][f2-tag])
- Derived guidance: use chips for a handful of quick options (up to about six) and a tag picker above that, so the screen does not become a chip cloud. State the keyboard behavior in a short help text.

## 14. Checklists

### Any field

- [ ] Label above the input, sentence case, no colon; verified in a screenshot of the form.
- [ ] The label, help text and error are exposed through automation (`AutomationProperties.Name`, `HelpText`, `LabeledBy`) (test or Accessibility Insights).
- [ ] Required marker plus `IsRequiredForForm`; a "required" test case exists.
- [ ] Border contrast 3:1 or a darker bottom edge (script output).
- [ ] Placeholder contrast measured with the element's `Opacity` applied, 4.5:1 per theme (script output).
- [ ] The error-on-focus choice (neutral like Fluent, or danger border kept) is recorded and drawn (screenshot of a focused field with an error).
- [ ] All five or seven states drawn: rest, hover, focus, disabled, error (and pressed, selected where relevant); screenshot per state in light and dark.
- [ ] Disabled fields keep label and help readable; an explanation is available when the reason is not obvious.
- [ ] A `MaxLength` or documented limit exists where relevant; paste handling tested.
- [ ] Focus after a failed submit goes to one defined place (first invalid field or the message bar), covered by a test.

### Text and search

- [ ] Tab entry leaves the caret at the end (text) or selects all (fast fields); headless test checks `SelectionStart` and `SelectionLength`.
- [ ] Enter triggers the default button in single-line fields and inserts a line break in multi-line fields (test).
- [ ] Esc restores the last committed value, a second Esc closes the form (test).
- [ ] The clear button appears only in search fields and only with content.
- [ ] Search shows recent items on focus, one "no results" line and the Esc ladder (test).

### Number, money

- [ ] Parser unit tests cover every row of the tables above plus an invalid and an empty case.
- [ ] After a failed commit Save is disabled or the value is empty, never the old value (test, section 4).
- [ ] Money uses `decimal` and rejects extra fraction digits (grep the diff for `double` and `float`).
- [ ] Up/Down, Page, Home/End, wheel-only-when-focused, clamp without wrap: headless keyboard test.
- [ ] Step buttons are disabled at the limits while the input remains active.
- [ ] Peer reports range, value and unit.

### Time, date

- [ ] Parser tables are unit tests; impossible inputs produce a range error that names the valid range, unreadable inputs a format error with an example.
- [ ] End equal to start is an error unless the field says otherwise (test).
- [ ] Dependent fields recompute only after commit (test that typing does not change the end field).
- [ ] Calendar popup: arrows, Page keys, Enter selects, Esc closes without change, focus returns to the field (headless test).
- [ ] No silent clamp to min or max.

### Choice

- [ ] Component matches the number of options (see section 12).
- [ ] Esc closes without change; Enter picks; type-ahead works (test).
- [ ] The list is as wide as the field and scrolls beyond about eight items.
- [ ] Multi-select above six values uses a tag picker, and Backspace removes the last tag (test).

## 15. Sources

[f2-field]: https://fluent2.microsoft.design/components/web/react/core/field/usage
[f2-label]: https://fluent2.microsoft.design/components/web/react/core/label/usage
[f2-input]: https://fluent2.microsoft.design/components/web/react/core/input/usage
[f2-button]: https://fluent2.microsoft.design/components/web/react/core/button/usage
[f2-rai]: https://fluent2.microsoft.design/responsible-AI/
[f2-searchbox]: https://fluent2.microsoft.design/components/web/react/core/searchbox/usage
[f2-spin]: https://fluent2.microsoft.design/components/web/react/core/spin/usage
[f2-textarea]: https://fluent2.microsoft.design/components/web/react/core/textarea/usage
[f2-select]: https://fluent2.microsoft.design/components/web/react/core/select/usage
[f2-dropdown]: https://fluent2.microsoft.design/components/web/react/core/dropdown/usage
[f2-combobox]: https://fluent2.microsoft.design/components/web/react/core/combobox/usage
[f2-tagpicker]: https://fluent2.microsoft.design/components/web/react/core/tagpicker/usage
[f2-tag]: https://fluent2.microsoft.design/components/web/react/core/tag/usage
[f2-messagebar]: https://fluent2.microsoft.design/components/web/react/core/messagebar/usage
[wl-textbox]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/text-box
[wl-numberbox]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/number-box
[wl-timepicker]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/time-picker
[wl-datepicker]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/date-picker
[wl-calendardatepicker]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/calendar-date-picker
[wl-datetime]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/date-and-time
[wl-combobox]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/combo-box
[wl-autosuggest]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/auto-suggest-box
[wl-textcontrols]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/text-controls
[wl-labels]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/labels
[wl-forms]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/forms
[wl-dialogs]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/dialogs-and-flyouts/dialogs
[wl-keyboard]: https://learn.microsoft.com/en-us/windows/apps/develop/input/keyboard-interactions
[wl-a11y]: https://learn.microsoft.com/en-us/windows/apps/design/accessibility/accessibility-overview
[wl-geometry]: https://learn.microsoft.com/en-us/windows/apps/design/signature-experiences/geometry
[gh-button]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-button/library/src/components/Button/useButtonStyles.styles.ts
[gh-input]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-input/library/src/components/Input/useInputStyles.styles.ts
[gh-field]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-field/library/src/components/Field/useFieldStyles.styles.ts
[gh-label]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-label/library/src/components/Label/useLabelStyles.styles.ts
[gh-link]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-link/library/src/components/Link/useLinkStyles.styles.ts
[gh-textarea]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-textarea/library/src/components/Textarea/useTextareaStyles.styles.ts
[gh-spinbutton]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-spinbutton/library/src/components/SpinButton/useSpinButtonStyles.styles.ts
[gh-spinbuttonlogic]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-spinbutton/library/src/components/SpinButton/useSpinButton.tsx
[gh-menu]: https://github.com/microsoft/fluentui/blob/master/packages/react-components/react-menu/library/src/components/MenuItem/useMenuItemStyles.styles.ts
[winui-generic]: https://github.com/microsoft/microsoft-ui-xaml/blob/main/dxaml/xcp/dxaml/themes/generic.xaml
