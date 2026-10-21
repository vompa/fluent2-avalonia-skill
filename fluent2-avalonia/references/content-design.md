# Content design: UI text, errors, formats

Read this file when you write or review any user-facing text in an Avalonia app: button labels, titles, messages, error text, empty states, tooltips, placeholders, help text, and the format of numbers, dates, times and amounts.

Language note: the examples here are in English. The project's own language governs all real strings. Apply the rules, not the wording: a German, French or Japanese interface keeps its own capitalization, punctuation, politeness form and number format. If the project has a style guide or a term list, it wins over this file.

## Contents

1. [Stance and tone](#1-stance-and-tone)
2. [Mechanics](#2-mechanics)
3. [Words to avoid](#3-words-to-avoid)
4. [Button labels](#4-button-labels)
5. [Error messages](#5-error-messages)
6. [Numbers, dates, times, amounts](#6-numbers-dates-times-amounts)
7. [State texts](#7-state-texts)
8. [Checklist](#8-checklist)
9. [Sources](#9-sources)

## 1. Stance and tone

- Treat language as design material, like icons. Before writing, answer three questions: who reads this, what do they want to do next, and how much time do they have. Because text that fits the moment is shorter and calmer. ([Content design][f2-content])
- Content should simplify tasks, highlight decisions and prioritize next steps. ([Content design][f2-content])
- Tone matches intent: factual in navigation, headers, buttons and table heads; calm and friendly on dashboards, empty states and help; never jokey; no calls for enthusiasm. In conflicts and errors stay neutral and without blame. Because users in a hurry read tone as attitude. ([Content design][f2-content])
- Prefer common words, an appropriate amount of information and a clear recommended action; short sentences and fragments are fine. ([Content design][f2-content])
- Address: Fluent writes in the second person and uses first person only when it speaks for the reader. Choose the project's form once (for example formal "you" in languages that distinguish) and keep it. Never let the software speak as "I" or "we" with feelings (see `responsible-ai.md`). ([Content design][f2-content])
- Tense and voice: mostly present tense, active voice unless the action matters more than the actor. After the user's own action report the result ("Appointment moved to Thu 14:30"). ([Content design][f2-content])

## 2. Mechanics

- Sentence case for all UI text, including titles; only the first word and proper nouns are capitalized. Follow the language's own rules (for example capitalized nouns). No all-caps for emphasis. ([Content design][f2-content], [Typography][f2-typo])
- Punctuation: question marks always; periods only after full sentences; no periods in headings, buttons, labels, list items; avoid exclamation marks except for genuinely happy moments; no colons after labels. Where the project writes help text as full sentences with a period, keep that rule consistently. ([Content design][f2-content])
- Keep a term list: pick one word per concept and use it everywhere ("customer", not "client" in one place and "guest" in another). Keep a glossary next to the manual. ([Content design][f2-content])
- Simplify jargon so a broad audience understands; keep terms consistent across product and docs. ([Content design][f2-content])
- Alt text for icons and images is part of the content. Write it as short as a label. ([Content design][f2-content], [Iconography][f2-icons])

## 3. Words to avoid

- Directional words ("above", "left", "below"): they assume sight and translate badly. Refer to the card or group by name ("under Contact"). ([Content design][f2-content])
- "Click here" and bare "Learn more": link text must make sense alone. ([Link][f2-link])
- Raw technical details in the main message; show them in an expandable detail or a log. ([Message bar][f2-messagebar])
- Terms like "Error" as a title; "invalid input" without saying what is valid.
- Search the code and the help content for these words when reviewing (grep for "above", "below", "left", "right", "here").

## 4. Button labels

Short rules; the full set is in `buttons-and-selection.md`.

| Purpose | Pattern | Example |
| --- | --- | --- |
| Create | "New" + object; in a menu: button "New", entries name the object | "New customer"; menu entries "Customer", "Note" |
| Add existing | "Add" + type | "Add product" |
| Confirm a dialog | Concrete answer to the title | "Delete" and "Cancel" for "Delete this file?" |
| Multi-step | "Next", "Back" | |
| Finish | "Done" (confirmation), "Finish" (selection applied), "Close" (informational), "Got it" (important notice), "Hide" (permanent) | |
| Discard | "Cancel" discards; "Close" changes nothing; no "x" next to "Close" | |
| Save | "Save" plus a toast or bar as feedback; ideally autosave | |
| Toast or bar action | One or two words, at most 36 characters; links start with a verb | "Undo", "Try again", "Check connection" |

([Button][f2-button], [Dialog][f2-dialog], [Toast][f2-toast], [Message bar][f2-messagebar])

## 5. Error messages

Fluent asks for three things: name what failed, say why, and offer two ways forward. An error without a next step leaves people stuck. Technical detail goes into a collapsible section or a log. ([Responsible AI][f2-rai], [Message bar][f2-messagebar])

| Part | Rule | Example (a report could not be saved) |
| --- | --- | --- |
| Title (what) | Concrete, sentence case, not "Error" | "Report not saved" |
| Text (why) | One sentence, no blame, no jargon | "The connection to the server was lost." |
| Way 1 (button) | One verb | "Try again" |
| Way 2 (link) | Verb plus object | "Check connection" |
| Detail | Collapsible or logged | "Show details" |

- Field errors state the problem and the way out: "The postal code has four digits." Never "Invalid input" without saying what is valid. ([Field][f2-field])
- A dialog is only for a decision; field problems stay at the field. ([WinUI dialogs][wl-dialogs])
- For failures of an AI feature always offer a way that does not need the AI ("Write the reply yourself"). ([Responsible AI][f2-rai])
- Availability messages say why something is unavailable and what to do: "AI replies are paused because no API key is set. Add one under Settings > AI." not "AI unavailable". ([Responsible AI][f2-rai])

## 6. Numbers, dates, times, amounts

Fluent defers to the Microsoft Writing Style Guide here, which was not read for this skill. The rules below are Derived from common practice and from the project-neutral needs of a desktop UI; the project's locale and style guide govern.

| Quantity | Rule | Example (example locale, day-first, 24-hour clock, decimal comma) |
| --- | --- | --- |
| Date in lists and headers | Abbreviated weekday plus day and month | "Wed 21.10." |
| Date in inputs and receipts | Full numeric date in the locale format | "21.10.2026" |
| Time | 24-hour `HH:mm` if the locale uses it; no "o'clock" in fields | "09:30" |
| Duration | Number plus abbreviated unit | "60 min" |
| Amount | Locale separators, two decimals; currency in the column header or after a non-breaking space | "1.234,50 EUR" |
| Percent | Number, non-breaking space where the locale uses it, percent sign | "38 %" |
| Numbers in text | In UI always digits, not words | "3 appointments to rebook" |

Always format through the culture (`CultureInfo`), never hand-built strings, and use non-breaking spaces between a number and its unit so the line does not break inside it. Use the same format in input, display and exports.

## 7. State texts

| State | Rule | Example |
| --- | --- | --- |
| Loading | Progressive form, at most three words, ends with an ellipsis preceded by a space, concrete (repeats the action word) | "Loading ...", "Searching customers ..." |
| Empty | One sentence: what is missing and how to create it, with a button | "No entries yet." plus "Add entry" |
| No result | One line | "No results." |
| Tooltip, disabled control | When it becomes available | "Available once a name is entered." |
| Tooltip, icon only | Fluent: short noun phrase, sentence case, no final punctuation. Action icons often read better as verb plus object. | "Move appointment" |
| Placeholder | An example or format hint, no period; never the label | "e.g. 09:30" |
| Help text | A sentence or fragment, one line | "The name as on the ID." |

([Spinner][f2-spinner], [WinUI auto-suggest][wl-autosuggest], [Tooltip][f2-tooltip], [Input][f2-input], [Field][f2-field])

## 8. Checklist

- [ ] Every string follows sentence case and the punctuation rules; a unit test or grep over resource strings finds no all-caps or trailing colon labels.
- [ ] One term per concept; the glossary is current (review against the term list).
- [ ] No direction words or "click here" in strings or help (grep).
- [ ] Every error has what, why and at least one way forward; AI errors have a no-AI path (review of error resources).
- [ ] Buttons are verbs; no "OK", "Yes", "No" in dialogs (grep).
- [ ] Dates, times, amounts use the culture formatters; no manual string formatting (grep for `ToString("` on dates and decimals in view code).
- [ ] Loading, empty, no-result and disabled-tooltip texts exist for every list and every non-obvious disabled control (review).
- [ ] The text stays readable at 200 percent scale and in the longest language (screenshot).

## 9. Sources

[f2-content]: https://fluent2.microsoft.design/content-design
[f2-typo]: https://fluent2.microsoft.design/typography
[f2-icons]: https://fluent2.microsoft.design/iconography
[f2-link]: https://fluent2.microsoft.design/components/web/react/core/link/usage
[f2-messagebar]: https://fluent2.microsoft.design/components/web/react/core/messagebar/usage
[f2-button]: https://fluent2.microsoft.design/components/web/react/core/button/usage
[f2-dialog]: https://fluent2.microsoft.design/components/web/react/core/dialog/usage
[f2-toast]: https://fluent2.microsoft.design/components/web/react/core/toast/usage
[f2-field]: https://fluent2.microsoft.design/components/web/react/core/field/usage
[f2-rai]: https://fluent2.microsoft.design/responsible-AI/
[f2-spinner]: https://fluent2.microsoft.design/components/web/react/core/spinner/usage
[f2-tooltip]: https://fluent2.microsoft.design/components/web/react/core/tooltip/usage
[f2-input]: https://fluent2.microsoft.design/components/web/react/core/input/usage
[wl-dialogs]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/dialogs-and-flyouts/dialogs
[wl-autosuggest]: https://learn.microsoft.com/en-us/windows/apps/develop/ui/controls/auto-suggest-box
