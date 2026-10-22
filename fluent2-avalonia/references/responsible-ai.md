# Responsible AI in the interface: disclosure, control, agents, review rubric

Read this file when a view shows or triggers AI output: suggestion panels, drafted replies, assistants, automatic actions, AI-filled fields, background agents. It covers the five Fluent principles, rules for agents, visual and content patterns, and a scoring rubric you can use as a review list. The rules are about interface behavior, not about model quality.

Language note: example strings are English; use the project's language. Disclaimer wording should be approved by whoever owns product content, and kept in one central place so it is not retyped per view. If the project has its own AI policy, it wins; where it is silent, use this file.

## Contents

1. [Five principles and what they mean in a view](#1-five-principles-and-what-they-mean-in-a-view)
2. [Agents](#2-agents)
3. [Visual and content patterns](#3-visual-and-content-patterns)
4. [Review rubric](#4-review-rubric)
5. [Checklist](#5-checklist)
6. [Sources](#6-sources)

## 1. Five principles and what they mean in a view

| Principle | Fluent idea (paraphrased) | Interface rule (Derived unless cited) |
| --- | --- | --- |
| Transparency | Make AI presence, reasoning and limits visible in interaction, image and text; mark and cite; explain scope and data use precisely; give specific availability messages for failures; keep a non-emotional, non-conscious, non-social tone; use "I" only functionally; do not present agents as team members; separate human and AI in names, pronouns and imagery; no flattery. | Every AI output carries a small persistent "AI" pill plus the approved disclaimer line. A drafted reply shows "AI draft, not sent yet". The assistant has no human first name and an icon, not a face. AI text is factual: no "I'm happy to", no feelings, no first names. Show sources (a manual topic, a record) as links under the answer. ([Responsible AI][f2-rai], [Agent principles][ms-agent-resp]) |
| Realistic expectations | Communicate capabilities and limits at first use (side panel explainer, tooltips at entry points, input placeholder, welcome card); for agents announce activation, triggers, access rights and powers; describe latency specifically. | The input placeholder names the scope ("Ask a question about using the app", not "Ask me anything"). Waiting text is concrete ("Searching the manual ..."), not "One moment please". First open shows a one-time card "What the assistant can and cannot do" with a "Got it" button. ([Responsible AI][f2-rai]) |
| Avoid over-reliance | Always show the approved disclaimer; never present output as fully reliable; make sources and accuracy easy to check; distinguish AI from non-AI content everywhere; add friction that prompts critical thinking; frame output as possibly incomplete. | Never send AI drafts automatically; require a visible review step before sending. Mark AI-created records (a booking, a classification) with the AI badge until a person confirms them. Numbers that carry legal or financial weight are never computed by the model; show them with their source ("from the price list"). ([Responsible AI][f2-rai]) |
| User control | Let users influence results; make effects clear; name actions clearly and consistently; announce activation explicitly (never silent); make consent conscious and specific; keep global AI settings findable. | A settings page with one switch per AI function and one sentence on its purpose. Every AI action with side effects shows target and consequence before running and can be declined. "Undo" for anything the AI changed. Switchable off at any time. ([Responsible AI][f2-rai]) |
| Collect feedback | Ask for explicit feedback on harm, explain why; use telemetry for implicit signals; categorize harm; keep labels neutral and descriptive. | Under each AI output two neutral buttons, "Helpful" and "Not helpful". On "Not helpful" offer categories: inaccurate, incomplete, inappropriate, harmful, biased, other. A sentence about purpose, and only claims that are true ("no customer data is stored" only if it is). Tooltips are neutral ("This answer was helpful"), never "Great!" or "Sorry". ([Responsible AI][f2-rai]) |

## 2. Agents

Agents act with more autonomy, so they need all of the above plus the following. ([Responsible AI][f2-rai], [Agent UX][ms-agent-ux], [Agent principles][ms-agent-resp])

- What counts as an agent: any function that prepares or proposes something without the user starting it. A function that reads incoming requests and prepares appointments, sorts a mailbox on its own or fills a form in the background is an agent, even if the product calls it a "suggestion" or "assistant". Such a function needs the agent criteria below and in the rubric: what the user can expect before the first interaction (trigger, access, powers), a protocol of what it did, and confirm or decline for every result. Because users judge the risk by what happens without them, not by what the feature is called. Derived.

- Disclose activation and autonomy: what triggers the agent, what it may read, what it may do. Example text: "Reads requests, proposes free times, books only after confirmation" or "Books on its own within these rules". Let the autonomy level be set per location or scope. ([Responsible AI][f2-rai])
- Let people approve or reject an agent action before it runs, explain how to correct, refine or undo, and never present an agent action as final. Show agent-made bookings as records with the AI badge and "Confirm" and "Decline" buttons; offer "Undo" in a toast and on the record; avoid wording like "booked for good" before a person confirms. ([Responsible AI][f2-rai], [Agent UX][ms-agent-ux])
- Keep background activity inspectable and controllable: an activity log (who or what, when, with a link to the record) and a status line that says what the agent is doing now. ([Agent UX][ms-agent-ux])
- Accept uncertainty: show confidence and reasoning when it matters, admit uncertainty in plain, factual words ("Not sure - please check"), and add deliberate friction for consequential decisions (human confirmation). Show a "read with uncertainty" marker where an AI match is unsure, and require confirmation. Payments, refunds, cancellations and other legal or financial actions always get a human. ([Agent UX][ms-agent-ux], [Agent principles][ms-agent-resp])
- Connect rather than replace: the agent supports people and says so; when a person should take over, offer the path to a human. ([Agent UX][ms-agent-ux])
- Built for purpose: personality only with a benefit; neutral tone for transactional work; no avatar with a face; keep the neutral mode as the only mode unless users need choices. ([Agent principles][ms-agent-resp])
- Resist bias: language, suggestions and imagery can skew expectations; human names and friendly faces create an illusion of competence. Keep drafts gender-neutral when gender is unknown, make no assumptions about age, origin or health, and sample drafts before production use. ([Agent principles][ms-agent-resp])

## 3. Visual and content patterns

| Pattern | Rule |
| --- | --- |
| Entry-point indicator | A consistent mark where AI is active, before interaction: a small pill "AI" (12 px, soft accent background, accent text, pill radius) before every AI-driven spot: drafts, assistant, AI-made records, "what to do" hints. The assistant's launcher shows the same mark. |
| Reasoning panel | Under suggestions an expandable "Why this suggestion?" (accordion) with two or three plain sentences: source (price list, schedule), rule applied ("30 minute break kept"), uncertainty. ([Accordion][f2-accordion]) |
| Memory indicator | Only if the system remembers things: "Remembered for this customer: preferred staff member" with a "Forget" button. Not needed if nothing is stored separately; verify that before leaving it out. |
| Presence and disclaimer | AI presence at every active entry point; the approved disclaimer on every AI output. In the staff UI too, for example a line under a draft: "Created by AI, please review." |
| Availability message | Say why a feature is unavailable and how to fix it. |
| Error message | What failed, why, two ways forward, one of them without AI. See `content-design.md`. |
| Feedback prompt | After AI output, with categories; neutral tooltips. |

([Responsible AI][f2-rai])

Feedback without a backend. A project that has no server for feedback can still collect it honestly (Derived):

- Store each feedback locally, one JSON line per entry (time, AI function, rating, category, optional comment), in a file in the app's data folder. Decide whether the AI output text goes into the entry, and say so in the purpose sentence, because output text can contain personal data.

```csharp
var entry = new FeedbackEntry(DateTimeOffset.UtcNow, Function: "reply-draft", Rating: "not-helpful", Category: "inaccurate");
File.AppendAllText(feedbackPath, JsonSerializer.Serialize(entry) + Environment.NewLine);
```

- Write the purpose sentence so that it states exactly what happens, for example "Your rating is saved in a file on this computer. It is not sent anywhere." Do not write "helps us improve" while nothing leaves the device. Reword the sentence the day a server exists.
- Name the missing part as its own task in the report: the transport to a server and the evaluation of the entries are open work, not part of this feature. Score the "Feedback" criterion by what exists (local collection with categories), and list the rest as open.
- Test: a rating appends exactly one line, a second rating appends a second line, and the file is valid JSON per line.

## 4. Review rubric

Score each criterion from 0 to 3; criteria that do not apply may be marked not applicable, and the maximum shrinks accordingly. Grades: A at 90 percent or more (ready to ship); B at 80 to 90 percent (every criterion at least 2, an improvement plan for each 0, re-review); C at 75 to 80 percent (fix all 0s, re-score); below 75 percent or two or more zeros: fail. Automatic fail if "Avoid over-reliance" scores 0 or 1, or if "Realistic expectations" scores 0 or 1 for agents. ([Responsible AI][f2-rai])

| Principle | Criterion | Three points mean (paraphrased) | Review question |
| --- | --- | --- | --- |
| Transparency | Before interaction | Every AI area is easy to recognize; AI is clearly distinct from people; agent memory is understandable | Does every AI spot carry the pill and an icon instead of a face? |
| Transparency | During interaction | Fitting failure messages with next steps, no dead ends, approved wording; limits are mentioned | Does every AI failure offer two ways, one without AI? |
| Transparency | Tone | Emotional and first-person language consistently avoided | Scan AI text for "I", "sorry", "happy to", first names |
| Transparency | Understanding and verifiability | Reasoning is easy to check for all output types; agent actions are traceable | Is there a "Why this suggestion?" and an activity log? |
| Expectations | Before interaction | All elements approved, sources shown, autonomy fully described | Does onboarding say what the AI may and may not do? |
| Expectations | After interaction | Realistic for the work context, autonomy fully described | Does the result say what still needs checking ("not sent yet")? |
| Over-reliance (auto-fail at 0 or 1) | - | Approved disclaimer on all output, all sources linked, agents easy to steer | Disclaimer everywhere, sources linked, approval before sending? |
| Control | - | Stop, change, edit, confirm, delete, forget, reset at any time; sensible approval; settings easy to reach | Undo, decline and per-function off switches present? |
| Feedback | - | Comprehensive mechanism with categories (harmful, inaccurate, incomplete, biased, inappropriate) | Both buttons with categories, in context? |

Mark every criterion you did not actually test as "not scored", and say so. The table is a template; scoring needs a real review of the running feature.

First-use card. The criterion "Expectations, before interaction" stays in the rubric: a one-time card that says what the AI can and cannot do. The card may be missing when the brief for the feature does not ask for it. Then score the criterion one point lower than the rest of the evidence would give (at most 2) and report the card as open, so the gap is visible instead of silently accepted. A missing card never turns into a pass by omission. Derived.

## 5. Checklist

- [ ] Every AI output shows the pill and the disclaimer line (search the views for AI output bindings without the pill).
- [ ] AI drafts are never sent without a visible review step (test: no code path sends without a user command).
- [ ] The first-use explainer exists and is dismissible once (test persistence); if the brief did not ask for it, the criterion is scored lower and reported as open.
- [ ] Every function that prepares or proposes something on its own is treated as an agent: expectation text, activity log, confirm and decline (review of the feature list).
- [ ] Without a feedback backend: entries are stored locally as JSON lines and the purpose sentence says exactly that; server and evaluation are listed as open (test and review).
- [ ] Each AI function has its own switch and a one-sentence purpose (settings review).
- [ ] Every AI side effect shows target and consequence first, and has Decline and Undo (test).
- [ ] The "Why this suggestion?" panel exists where suggestions appear (screenshot).
- [ ] Failure messages follow what, why, two ways, with a non-AI path (review of error resources).
- [ ] Feedback buttons with categories exist after each output (screenshot); the purpose sentence is true (confirm with the data owner).
- [ ] AI-created records carry the AI mark until a person confirms them (test).
- [ ] Legal or financial numbers are never produced by the model (grep for model output feeding calculation code; review).
- [ ] Rubric scored, with untested criteria marked as such.

## 6. Sources

[f2-rai]: https://fluent2.microsoft.design/responsible-AI/
[f2-accordion]: https://fluent2.microsoft.design/components/web/react/core/accordion/usage
[ms-agent-ux]: https://microsoft.design/articles/ux-design-for-agents/
[ms-agent-resp]: https://microsoft.design/articles/principles-for-responsible-agent-design/
