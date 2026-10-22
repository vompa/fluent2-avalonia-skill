# fluent2-avalonia

Based on Microsoft's **[Fluent 2 design system](https://fluent2.microsoft.design/)**, the design language behind Windows, Microsoft 365 and Teams.

<p align="center">
  <img src="docs/images/fluent2-teams-chat.webp" width="620" alt="Fluent 2 in practice: a Microsoft Teams chat list on a phone, surrounded by soft 3D shapes, emoji reactions and a meeting card, illustrating Fluent 2's layered materials, rounded shapes and expressive color.">
</p>
<p align="center"><sub>Fluent 2 in practice: layered surfaces, rounded shapes, expressive color and clear hierarchy, here in Microsoft Teams. Image © Microsoft, from fluent2.microsoft.design.</sub></p>

<p align="center">
  <img src="docs/images/fluent2-design-site.webp" width="860" alt="The Fluent 2 design system website: left navigation with Get started, Design language (color, elevation, iconography, layout, material, motion, shapes, typography), Guidelines (accessibility, content design, design tokens) and Web React components; a second window shows the Typography page.">
</p>
<p align="center"><sub>The Fluent 2 website at <a href="https://fluent2.microsoft.design/">fluent2.microsoft.design</a>: design language, guidelines and components per platform. This skill condenses those pages into rules and checklists for Avalonia. Image © Microsoft.</sub></p>

The two images above are © Microsoft Corporation and are shown for reference only; they are not covered by this repository's MIT license.

A skill for Claude Code and Claude.ai that teaches Claude to build and review Avalonia desktop UIs the way Microsoft's Fluent 2 design system describes them: component behavior, states, keyboard handling, accessibility, content design, motion, and responsible AI patterns, with checklists you can verify and patterns for headless Avalonia tests.

It is aimed at developers who build Avalonia (AXAML) apps with Claude and want consistent, accessible, keyboard-friendly interfaces without re-deriving the rules each time. Typical triggers: "build a time field", "review my TextBox style", "add an AI suggestion panel", "why can't I tab through this form".

Tested in practice across three implementation rounds on an Avalonia 12 desktop app; Avalonia 11 names are listed alongside.

The skill never replaces your own design brief. If a project has a brief or tokens, those values win; Fluent values are the fallback, and deliberate deviations are recorded with a reason.

## Install

Copy the folder `fluent2-avalonia/` into one of these places:

- `.claude/skills/` inside your project (shared with the team through the repository), or
- `~/.claude/skills/` (available in all your projects).

```bash
cp -r fluent2-avalonia ~/.claude/skills/
```

For Claude.ai, zip the folder as a `.skill` package (the archive root contains the `fluent2-avalonia/` folder) and upload it in the skills settings. Packaging details follow Anthropic's skill documentation.

## Structure

```text
fluent2-avalonia/
  SKILL.md                    entry point: when it applies, how to work, rules at a glance
  references/
    foundations.md            layout, type, color, elevation, motion, shape, tokens, deviations
    inputs.md                 field anatomy, states, behavior matrix, parsers for time/date/number/money
    buttons-and-selection.md  buttons, links, checkbox, radio, switch, slider
    navigation.md             tabs, navigation pane, menus, toolbar overflow, tooltips
    dialogs-and-messages.md   dialogs, drawers, message bars, toasts, progress
    data-display.md           lists, tables, cards, avatars, badges, accordions
    content-design.md         UI text, error messages, formats
    accessibility.md          keyboard, focus, F6 regions, screen readers, contrast, motion
    responsible-ai.md         AI disclosure, control, agents, review rubric
    avalonia-mapping.md       Fluent concepts as selectors, resources, behaviors, headless tests
    avalonia-12-notes.md      Avalonia 11 to 12 names, changed behavior, observed pitfalls
  evals/evals.json            three realistic test prompts
scripts/validate.py           structure and link checks (standard library only)
docs/images/                  the two Fluent 2 reference images shown above (© Microsoft)
```

Claude reads `SKILL.md` first and opens a reference file only when the task needs it.

## Check

```bash
python scripts/validate.py
```

The script checks the frontmatter, the `SKILL.md` length, that referenced files exist, that local Markdown links and anchors resolve, and that `evals/evals.json` is well formed. It makes no network requests. The same check runs in GitHub Actions on every push.

## Sources and relationship to Microsoft

The content is an independent, paraphrased summary built from public sources: the [Fluent 2 design system](https://fluent2.microsoft.design), Microsoft Learn documentation for WinUI 3 and Windows design, and the open source [Fluent UI](https://github.com/microsoft/fluentui) repository. Each rule links to its source in the reference file where it appears.

This project is not affiliated with or endorsed by Microsoft. Fluent, Segoe, Windows and WinUI are trademarks of Microsoft Corporation. Microsoft texts are paraphrased, not copied. Values marked "Derived" are recommendations of this skill, not Microsoft rules, and values marked "derived, not verified" were not checked against a running app or a source.

## License

MIT, see [LICENSE](LICENSE).

## Deutsch

Dieser Skill bringt Claude bei, Avalonia-Desktopoberflächen nach dem Designsystem Fluent 2 von Microsoft zu bauen und zu prüfen: Verhalten von Komponenten, Zustände, Tastaturbedienung, Barrierefreiheit, Texte, Bewegung und verantwortungsvolle KI-Muster, jeweils mit prüfbaren Checklisten und Mustern für Headless-Tests.

Installation: den Ordner `fluent2-avalonia/` nach `.claude/skills/` im Projekt oder nach `~/.claude/skills/` kopieren. Prüfung: `python scripts/validate.py`.

Hat ein Projekt einen eigenen Design-Brief oder eigene Tokens, gelten diese Werte; Fluent ist nur die Rückfallebene, und bewusste Abweichungen werden mit Begründung notiert. Die Inhalte sind sinngemäß zusammengefasst, nicht aus den Microsoft-Texten kopiert; das Projekt ist unabhängig und steht in keiner Verbindung zu Microsoft. Grundlage ist das Designsystem [Fluent 2](https://fluent2.microsoft.design/) von Microsoft; die beiden Bilder oben (Teams-Chat als Beispiel für Fluent 2 in der Praxis, Ansicht der Fluent-2-Website) stammen von dort und sind © Microsoft, nicht Teil der MIT-Lizenz. Sprache der Skill-Dateien: Englisch. Lizenz: MIT.
