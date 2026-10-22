# Avalonia 12 notes: renamed APIs, changed behavior, observed pitfalls

Read this file when code from Avalonia 11 examples fails to compile or behaves differently, when an attached property, a style selector or a test filter does not work as written, or before you trust a pitfall list from memory. Everything here was seen in one Avalonia 12.1.4 desktop project on Windows (Fluent theme) and was not re-checked on Avalonia 11 or on other platforms or versions; confirm each item against the version your project uses. The patterns that depend on these items are in `avalonia-mapping.md`.

## Contents

1. [Avalonia 11 to 12 names](#1-avalonia-11-to-12-names)
2. [Changed behavior and API limits](#2-changed-behavior-and-api-limits)
3. [Observed pitfalls](#3-observed-pitfalls)
4. [Sources](#4-sources)

## 1. Avalonia 11 to 12 names

Avalonia 12 renames several APIs and template parts that this skill's patterns use. Observed in Avalonia 12.1.4; the right-hand column is the name to use there, the left-hand column is what Avalonia 11 code and older examples contain. Docs: [Avalonia documentation][avalonia-docs].

| Avalonia 11 | Avalonia 12 | Where it shows up |
| --- | --- | --- |
| `GotFocusEventArgs` | `FocusChangedEventArgs` | Handlers for `GotFocus`, `LostFocus`; `avalonia-mapping.md`, section 6 |
| `OnGotFocus(GotFocusEventArgs)`, `OnLostFocus(RoutedEventArgs)` overrides | `OnGotFocus(FocusChangedEventArgs)`, `OnLostFocus(FocusChangedEventArgs)` | Custom field controls |
| `Visual.GetVisualRoot()` | `TopLevel.GetTopLevel(visual)` | Focus manager access; `avalonia-mapping.md`, section 8 |
| `Watermark` (and its foreground) | `PlaceholderText` with `PlaceholderForeground` | Placeholder styling and tests |
| Watermark-named template parts | `PART_Placeholder`, `PART_FloatingPlaceholder` | Styles that reach into the TextBox template; `avalonia-mapping.md`, section 4 |

How to find the right name in your version instead of guessing: build and read the compiler error (it names the missing member or type), or list the members by reflection (`typeof(TextBox).GetProperties()`, `typeof(TextBox).Assembly.GetTypes()` filtered by name), or run the template probe from `avalonia-mapping.md`, section 4.

## 2. Changed behavior and API limits

Observed in Avalonia 12.1.4. Each row names the pattern that fails and what to do instead.

| Pattern | What happens in Avalonia 12 | Do instead |
| --- | --- | --- |
| `AvaloniaProperty.RegisterAttached` with a static class as owner (`typeof(MyBehavior)` where `MyBehavior` is `static`) | Not accepted; the owner must be `Control` or a non-static class. | Declare the behavior class without `static` (for example `public abstract class MyBehavior` holding only static members) and register with `RegisterAttached<MyBehavior, TextBox, TValue>("Name")`. If an analyzer asks for `static`, suppress it for that type. |
| `Selectors.Is<T>()` with `T` derived from `Animatable` | The type argument must be a `StyledElement` or derived. | Use `Is<Control>()` or `OfType<Control>()`. |
| Style setter with a compiled binding (`{Binding Subject}`) | Does not compile or resolve without the data type. | Put `x:DataType` on the `Style` element: `<Style Selector="ListBoxItem" x:DataType="vm:Row">`. |
| Partial `OnXChanged(old, new)` methods of a generated property | The parameters take the nullability of the property. | Declare `old` and `new` nullable when the property is nullable, or the build reports nullability warnings (errors when warnings are errors). |
| `dotnet test --filter-method` and `--filter-class` (Microsoft.Testing.Platform runner) | Needs fully qualified names; a class filter and a method filter given together combine with AND, so a method of another class matches nothing ("Total: 0"). Several filters of the same kind combine with OR. | Pass the full name `Namespace.Class.Method`, or run per class; check the reported test count, not only the exit code. Do not use wildcards (`*`) in the filter: restricted shells and permission layers can refuse or expand them, and a partial name matches nothing. |

## 3. Observed pitfalls

- Local values beat style setters and classes; set state colors only in styles.
- Template-set values beat styles too (placeholder `Opacity`, `HighlightBackground`, `PopupBorder`); see `avalonia-mapping.md`, section 3. Probe, then set a local value in `TemplateApplied` or override the theme resource.
- A hover selector that reuses the rest brush hides hover.
- `FontFamily="Name, Fallback"` only finds installed fonts; an embedded font needs its resource family name (for example `fonts:Inter#Inter` in one project's setup); check how your theme registers it.
- Headless image tests with enter or exit animations need at least 200 ms before the frame capture, otherwise the image shows a half fade.
- Stub-based UI tests that match a request by path plus query break when you add a query parameter; update every stub that wires that route.
- Pseudo-class rules beat base rules; a base style on a template part can lose against the theme's `:unchecked` or `:pointerover` rule. Write the state selector (`avalonia-mapping.md`, section 3, Precedence).
- `MenuItem` radius is a `TemplateBinding` to `MenuItem.CornerRadius`; set it on the `MenuItem`, not on the template border (`avalonia-mapping.md`, section 3).
- `OverrideDefaultValue` on `ToolTip.ShowDelay` or `ToolTip.Placement` throws "Metadata already set"; use the `ToolTip.TipProperty.Changed` pattern (`avalonia-mapping.md`, section 6).
- A custom `ToggleSwitch` template without `PART_SwitchKnob` and `PART_MovingKnobs` fails with AVLN2205 (`avalonia-mapping.md`, section 4).
- Overflowed toolbar items that are only transparent still take Tab stops; disable them and arrange them at 0 x 0 (`avalonia-mapping.md`, section 8).
- A tooltip is a logical child of its target: a descendant selector such as `Border.chip TextBlock` also colors the tooltip text (seen: contrast 1.0 on the dark tooltip surface). Use `>` and test every real tooltip (`navigation.md`, section 6; `avalonia-mapping.md`, section 3).
- A menu entry without an icon starts left of its neighbors: the icon column is not reserved. An empty 16 x 16 `Border` as `Icon` keeps the texts flush (`navigation.md`, section 4).
- Two `*` columns stay unequal under infinite width; give the frame a width. `SharedSizeGroup` needs `Grid.IsSharedSizeScope` on a common ancestor. `DesiredSize` includes the margin (`avalonia-mapping.md`, section 4, Layout notes).
- In headless tests a flyout is opened with `ShowAt` and its content is found from the window, not from the owner; `RaiseEvent(Button.ClickEvent)` does not run the command (`avalonia-mapping.md`, section 10, Flyouts, keys and commands).
- A window `MinWidth` clamps headless test widths (`avalonia-mapping.md`, section 10).
- A reduced-motion class on the root window with a type-only style loses against setters under classes and pseudo-classes, and styles added to an open window leave template transitions behind; use an activator style on the `Application` (`avalonia-mapping.md`, section 5).
- `ITransition` does not expose duration or property; read the concrete type (`avalonia-mapping.md`, section 10, Reading transitions).
- Automation names are derived from content and can be a type name or empty with no sign in source or screenshot; scan them in a test (`accessibility.md`, section 7).
- Tab follows source order: a bottom-docked `DockPanel` child written before the content takes focus first, and `KeyboardNavigation.TabIndex` did not change that (`avalonia-mapping.md`, section 8, Tab order walk).
- A view that handles keys itself must be the explicit F6 focus target, otherwise arrows reach another handler (`avalonia-mapping.md`, section 8, F6 regions).
- A scan for `#` finds neither named colors nor shadow colors in `BoxShadow` strings (`avalonia-mapping.md`, section 2).
- A control with `protected override Type StyleKeyOverride => typeof(ToggleButton)` is not matched by a style on its own type (`c|AccordionHeader`); target the base type plus a class (`ToggleButton.accordion`) and write a test that proves the style applies (`avalonia-mapping.md`, section 3, Precedence).
- A `RenderTransform` set by a style is a `TransformOperations`, not a `RotateTransform`: a cast gives `null` or fails. Read the matrix instead (`avalonia-mapping.md`, section 5, Transforms from styles).
- A custom header that must announce expanded or collapsed (accordion) needs its own `ToggleButtonAutomationPeer` that implements `IExpandCollapseProvider`; the stock peer reports only the toggle state (`avalonia-mapping.md`, section 9, Expanded state on a toggle header).
- A theme `MinHeight` of 34 on buttons turns `Width="28" Height="28"` into a 28 x 34 element; small icon buttons need `MinHeight="28"` as well (`avalonia-mapping.md`, section 4, Layout notes).
- Cards inside a scrolling panel lie below the visible edge in headless tests, so clicks and frames miss them; call `BringIntoView()` and wait one frame first (`avalonia-mapping.md`, section 10, Scrolling containers).
- A counter or other property-dependent behavior of a composite field worked when built in code and never ran when created from XAML, because XAML assigns properties one by one in markup order and one input (`AcceptsReturn`) was not yet set when the content arrived. Test with instances created from XAML and recompute in `OnPropertyChanged` (`avalonia-mapping.md`, section 10, Instances built from XAML; `inputs.md`, section 6).

## 4. Sources

Avalonia documentation and source are the authority for names, selectors, properties and the headless API: [Avalonia documentation][avalonia-docs] and [Avalonia source][avalonia-src].

[avalonia-docs]: https://docs.avaloniaui.net/
[avalonia-src]: https://github.com/AvaloniaUI/Avalonia
