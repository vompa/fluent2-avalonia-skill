# Avalonia mapping: how Fluent concepts become AXAML, styles, behaviors and tests

Read this file when you turn a Fluent rule from the other reference files into Avalonia code: choosing selectors for states, wiring tokens, writing transitions, adding keyboard behavior, exposing automation, restoring focus, and testing with headless input. Fluent itself says little about Avalonia; everything here is practical guidance, so treat it as **Derived** and sketched against Avalonia 11 conventions. Property and pseudo-class names change between versions and themes, so confirm each one against the control theme and the version your project uses before copying it. Where a statement is an observed behavior rather than documented, it says so.

## Contents

1. [Start from what the project already has](#1-start-from-what-the-project-already-has)
2. [Tokens as resources](#2-tokens-as-resources)
3. [States as selectors](#3-states-as-selectors)
4. [Templates and the field underline](#4-templates-and-the-field-underline)
5. [Transitions and reduced motion](#5-transitions-and-reduced-motion)
6. [Behaviors as attached properties](#6-behaviors-as-attached-properties)
7. [Typed fields: parser, control, ViewModel](#7-typed-fields-parser-control-viewmodel)
8. [Keyboard, focus and navigation](#8-keyboard-focus-and-navigation)
9. [Automation](#9-automation)
10. [Headless tests](#10-headless-tests)
11. [Observed pitfalls](#11-observed-pitfalls)
12. [Checklist](#12-checklist)
13. [Sources](#13-sources)

## 1. Start from what the project already has

- Find the theme setup first: `App.axaml` (`FluentTheme` or a custom theme), the project's `Styles.axaml` and `Colors.axaml` (names vary), a design brief and a token file. Extend those; do not add a parallel style system. Because two sources of truth drift apart.
- Avalonia's built-in Fluent theme follows Fluent-style visuals but is not a promise of Fluent 2 conformance in every detail (heights, field underline, focus drawing). Compare it against `inputs.md` and `foundations.md` and override only what the brief requires.
- Prefer a style or a `ControlTheme` override over a new control class, and a new control class (templated, with a small ViewModel-friendly API) over duplicating XAML in each view. Because behavior such as parsing and Esc handling belongs in one place.

## 2. Tokens as resources

- Put every color, radius, thickness, height and duration in a resource dictionary with semantic names (for example `SurfaceBrush`, `FieldBorderBrush`, `AccentBrush`, `ControlHeight`, `ControlRadius`). Per-theme values go into `ThemeDictionaries` under the keys `Light` and `Dark`.
- Reference theme-dependent values with `DynamicResource`. `StaticResource` binds once and does not follow a theme switch. Because a runtime switch between light, dark and a future high-contrast variant must repaint everything.
- No hex literals in view XAML. Search for them in CI (`grep -rn "#[0-9A-Fa-f]\{6\}" --include=*.axaml` in view folders, excluding the token file).

```xml
<ResourceDictionary xmlns="https://github.com/avaloniaui">
  <ResourceDictionary.ThemeDictionaries>
    <ResourceDictionary x:Key="Light">
      <SolidColorBrush x:Key="SurfaceBrush" Color="#FFFFFF"/>
      <SolidColorBrush x:Key="FieldBottomEdgeBrush" Color="#616161"/>
    </ResourceDictionary>
    <ResourceDictionary x:Key="Dark">
      <SolidColorBrush x:Key="SurfaceBrush" Color="#292929"/>
      <SolidColorBrush x:Key="FieldBottomEdgeBrush" Color="#ADADAD"/>
    </ResourceDictionary>
  </ResourceDictionary.ThemeDictionaries>
  <CornerRadius x:Key="ControlRadius">8</CornerRadius>
  <x:Double x:Key="ControlHeight">34</x:Double>
</ResourceDictionary>
```

The values are placeholders; use the project's tokens. A third theme variant for high contrast can be declared as a custom `ThemeVariant` (check the API in your version); whether Avalonia maps OS contrast colors automatically is (derived, not verified).

## 3. States as selectors

| Fluent state | Avalonia selector | Note |
| --- | --- | --- |
| Hover | `:pointerover` | Not set on touch. |
| Pressed | `:pressed` | Buttons and toggle buttons. For text boxes, a pressed look is rarely needed. |
| Keyboard focus | `:focus-visible` | Keyboard-driven focus only; mouse focus uses `:focus`. Use `:focus` when the focus must also show after a click (inputs). |
| Focus inside a composite | `:focus-within` | Container-level highlight. |
| Disabled | `:disabled` | Avoid dimming labels and help text through the container. |
| Selected, checked | `:selected`, `:checked`, `:indeterminate` | Per control type. |
| Error | `:error` | Set by data validation (`INotifyDataErrorInfo`, `DataValidationErrors`). For ViewModel-driven or non-blocking states add your own classes: `Classes.invalid`, `Classes.warning`, `Classes.success` bound to ViewModel flags. |
| Read-only | `TextBox[IsReadOnly=True]` | An attribute selector works independent of pseudo-class support. |

Rules that save debugging time:

- State colors live in styles, never as local values on the element. Local values beat style setters and state selectors, so a hand-set `Background` makes hover and disabled look broken. Derived from Avalonia's value precedence.
- A state selector on a template part needs the `/template/` form, for example `TextBox:pointerover /template/ Border#PART_BorderElement`. The part names come from the theme's template; look them up in the theme source of the version you use.
- A hover style that reuses the same brush as the rest state is invisible. When you change the rest state, re-check every state that shares the token.
- Fluent's disabled state is exempt from contrast rules, but keep label and help text readable: apply the disabled look to the input element, not to the field container.

## 4. Templates and the field underline

Fluent's input has a darker bottom edge and, on focus, a 2 px brand underline that grows from the center. In Avalonia that means a `ControlTheme` for the field with two extra elements: a bottom-edge border and a focus underline whose `ScaleTransform` goes from 0 to 1 horizontally. Sketch only; copy the structure of your theme's `TextBox` template and add the two parts.

```xml
<!-- Inside the TextBox ControlTheme template, below the content presenter -->
<Border x:Name="PART_BottomEdge" Height="1" VerticalAlignment="Bottom"
        Background="{DynamicResource FieldBottomEdgeBrush}"/>
<Border x:Name="PART_FocusUnderline" Height="2" VerticalAlignment="Bottom"
        Background="{DynamicResource AccentBrush}" Opacity="0"
        RenderTransformOrigin="50%,50%" RenderTransform="scaleX(0)">
  <Border.Transitions>
    <Transitions>
      <TransformOperationsTransition Property="RenderTransform" Duration="0:0:0.2"/>
      <DoubleTransition Property="Opacity" Duration="0:0:0.05"/>
    </Transitions>
  </Border.Transitions>
</Border>
```

```xml
<Style Selector="^:focus /template/ Border#PART_FocusUnderline">
  <Setter Property="Opacity" Value="1"/>
  <Setter Property="RenderTransform" Value="scaleX(1)"/>
</Style>
```

The exit transition is shorter than the enter transition in Fluent (50 ms against 200 ms); you can mirror that with a separate transition set per state. Use `:focus` rather than `:focus-visible` so the underline also shows after a mouse click, because the field must always show where typing goes.

Focus rings for buttons and list rows: the control's `FocusAdorner` draws the ring. An adorner drawn outside the element's bounds is clipped by the adorner layer; turn clipping off with `AdornerLayer.IsClipEnabled="False"` on the element (observed). Calling `Focus(NavigationMethod.Tab)` shows the ring only if the element did not already have focus; to test it, focus another element first (observed).

## 5. Transitions and reduced motion

```xml
<Style Selector="Button">
  <Setter Property="Transitions">
    <Transitions>
      <BrushTransition Property="Background" Duration="0:0:0.1" Easing="{StaticResource EaseHover}"/>
      <BrushTransition Property="BorderBrush" Duration="0:0:0.1" Easing="{StaticResource EaseHover}"/>
    </Transitions>
  </Setter>
</Style>
<Style Selector="Button:pressed">
  <Setter Property="RenderTransform" Value="scale(0.97)"/>
</Style>
```

Define the Fluent curves once as resources (check the class and property names in your version; Avalonia exposes a spline easing):

```xml
<SplineEasing x:Key="EaseHover"   X1="0.33" Y1="0" X2="0.67" Y2="1"/>  <!-- easy ease -->
<SplineEasing x:Key="EaseEnter"   X1="0"    Y1="0" X2="0"    Y2="1"/>  <!-- decelerate, things entering -->
<SplineEasing x:Key="EaseExit"    X1="1"    Y1="0" X2="1"    Y2="1"/>  <!-- accelerate, things leaving -->
```

Durations from `foundations.md`: hover and color 100 ms, enter 200 ms, exit 50 to 120 ms. Animate only color, opacity and transform; avoid animating layout properties, because layout animation causes repeated measure passes.

Reduced motion:

- Provide one app-level switch (OS preference where your platform API exposes it, otherwise a setting) and make it zero every transition, including press scales, fades and the input underline.
- A practical route: add a class such as `no-motion` on the root window and a style that clears `Transitions` for the controls and their template parts (`/template/` selectors), or build transitions from a few shared style classes that the switch removes. Verify with the test below instead of trusting the selectors.
- Test: with the switch on, walk the visual tree and assert that no element has a non-empty `Transitions` collection with a non-zero duration (section 10).

## 6. Behaviors as attached properties

Put keyboard and focus behavior that applies to many fields into attached properties or small control subclasses, so XAML opts in with one attribute and one test pins the behavior.

```csharp
public static class SelectAllOnTab
{
    public static readonly AttachedProperty<bool> IsEnabledProperty =
        AvaloniaProperty.RegisterAttached<TextBox, bool>("IsEnabled", typeof(SelectAllOnTab));

    public static void SetIsEnabled(TextBox box, bool value) => box.SetValue(IsEnabledProperty, value);
    public static bool GetIsEnabled(TextBox box) => box.GetValue(IsEnabledProperty);

    static SelectAllOnTab() =>
        IsEnabledProperty.Changed.AddClassHandler<TextBox>(OnIsEnabledChanged);

    private static void OnIsEnabledChanged(TextBox box, AvaloniaPropertyChangedEventArgs e)
    {
        box.GotFocus -= OnGotFocus;
        if (e.NewValue is true) box.GotFocus += OnGotFocus;
    }

    private static void OnGotFocus(object? sender, GotFocusEventArgs e)
    {
        if (sender is TextBox box && e.NavigationMethod == NavigationMethod.Tab) box.SelectAll();
    }
}
```

Use it as `local:SelectAllOnTab.IsEnabled="True"` on time, date, number, money and search fields. What your Avalonia version selects on Tab by default is something to check, not assume: write the headless test first and let it decide whether the behavior is needed.

Other behaviors worth centralizing: Esc restores the last committed value (and consumes the key so the next Esc closes the form); Enter commits and runs the default button; paste sanitizing; wheel changes the value only while focused; arrow-step repeat with a 300 ms delay then acceleration.

## 7. Typed fields: parser, control, ViewModel

Follow the pipeline in `inputs.md` section 4:

1. A pure parser class per type (`TimeInputParser`, `DateInputParser`, `MoneyInputParser`) with a method such as `bool TryParse(string text, CultureInfo culture, out T value, out string? error)`. No UI types inside. The error string comes from a resource, not a literal.
2. A templated control (`TimeField`, `NumberField`, ...) that owns raw text, calls the parser on Enter or `LostFocus`, writes the display format back and exposes a typed styled property (`TimeOnly? Value`, `decimal? Amount`).
3. The ViewModel binds only to the typed value and to a validation state; it never sees raw text.
4. Parser unit tests drive a table of inputs and expected results straight from the tables in `inputs.md`, plus invalid and empty inputs.

Money uses `decimal`; the display and parse paths take a `CultureInfo`; no `double` anywhere in the money path. Dependent fields recompute on the committed value change, not on text change.

## 8. Keyboard, focus and navigation

- Group as one tab stop: `KeyboardNavigation.TabNavigation="Once"` on the group container with arrow handling inside (radio groups, button strips). `Cycle` wraps inside popups such as menus; `Continue` is the default.
- Region switching with F6: keep an ordered list of region roots, intercept `KeyDown` for F6 and Shift+F6 on the window, move focus to the first focusable descendant of the next region. Give each root `AutomationProperties.Name`.
- Restoring focus after a transient surface closes: store the invoker before opening, then call `invoker.Focus()` after close (`Dispatcher.UIThread.Post` when closing from inside a handler). When the focused element hides itself, no element owns focus and keys never reach your view (observed); listen with `AddHandler(InputElement.KeyDownEvent, handler, RoutingStrategies.Tunnel)` on the window, and if `TopLevel.GetTopLevel(this)?.FocusManager?.GetFocusedElement()` is null, restore focus and handle the key.
- Menus and flyouts: when a click handler closes a flyout, its items can lose their `DataContext`, so command parameters arrive empty (observed). Run the command first and close the flyout afterwards with `Dispatcher.UIThread.Post`.
- Dragging: releasing pointer capture inside `PointerReleased` raises `PointerCaptureLost` at once and cancels the operation; finish the drop first, then release (observed).

## 9. Automation

| Need | Property |
| --- | --- |
| Accessible name of any element | `AutomationProperties.Name` |
| Name from a visible label | `AutomationProperties.LabeledBy` |
| Hint, format, range text | `AutomationProperties.HelpText` |
| Required field | `AutomationProperties.IsRequiredForForm` |
| Shortcut | `AutomationProperties.AcceleratorKey` (and tooltip) |
| Access key | `AutomationProperties.AccessKey` |
| Live region for message bars | `AutomationProperties.LiveSetting` (`Assertive` for errors and warnings, `Polite` otherwise) |
| Hide decorative elements | `AutomationProperties.AccessibilityView="Raw"` (check the value name in your version) |

For custom controls override `OnCreateAutomationPeer` and return a peer that reports the right role and, for range controls, the range-value pattern (minimum, maximum, value). Keep the step buttons of a number field out of the tab order (`Focusable="False"` or `IsTabStop="False"`) and name them "Increase" and "Decrease". Fluent and WinUI sources for the rules: see `accessibility.md`.

## 10. Headless tests

Use `Avalonia.Headless.XUnit` (or the NUnit package) with the Skia renderer when you need frames.

```csharp
[assembly: AvaloniaTestApplication(typeof(TestAppBuilder))]

public class TestAppBuilder
{
    public static AppBuilder BuildAvaloniaApp() =>
        AppBuilder.Configure<App>()
            .UseSkia()
            .UseHeadless(new AvaloniaHeadlessPlatformOptions { UseHeadlessDrawing = false });
}
```

```csharp
[AvaloniaFact]
public void Tab_into_time_field_selects_all()
{
    var field = new TextBox { Text = "09:30" };
    SelectAllOnTab.SetIsEnabled(field, true);
    var window = new Window { Content = new StackPanel { Children = { new Button(), field } } };
    window.Show();

    window.KeyPressQwerty(PhysicalKey.Tab, RawInputModifiers.None);   // focus moves from the button

    Assert.Equal(0, field.SelectionStart);
    Assert.Equal(field.Text!.Length, field.SelectionEnd - field.SelectionStart);
}
```

Helpers on the window: `KeyPressQwerty`, `KeyReleaseQwerty`, `KeyTextInput`, `MouseDown`, `MouseMove`, `MouseUp`, `MouseWheel`. Use them to drive keyboard and mouse paths, not only ViewModel methods.

Patterns worth having:

| Pattern | What it asserts |
| --- | --- |
| Tab-order walk | Press Tab repeatedly and compare the sequence of focused element names with the numbered focus list. |
| Esc ladder | Edit a field, press Esc, field shows the old value; Esc again raises the close command. |
| Enter and default button | Enter in a single-line field invokes the default command; not in a multi-line field. |
| Parser table | `[Theory]` with the rows of the parser tables, plus invalid and empty input. |
| Error replaces help | Setting an error hides the help text and shows the message with its icon. |
| Focus restore | Open a popup or dialog from a button, close it, assert the button has focus; close a panel with a mouse click, press an arrow key, assert the view handled it. |
| Reduced motion | Switch on, walk the visual tree, assert no non-zero transition (use a helper that reads `Transitions`). |
| Contrast | Resolve brushes from both theme variants (`Application.Current.RequestedThemeVariant`), compute WCAG ratios, assert 4.5 and 3.0 thresholds per state. |
| Screenshots | `window.CaptureRenderedFrame()` after a short wait for animations (at least 200 ms for fades; advance the render timer or delay), once per state per theme. |

A small contrast helper:

```csharp
static double Luminance(Color c)
{
    static double Lin(byte v) { var s = v / 255.0; return s <= 0.03928 ? s / 12.92 : Math.Pow((s + 0.055) / 1.055, 2.4); }
    return 0.2126 * Lin(c.R) + 0.7152 * Lin(c.G) + 0.0722 * Lin(c.B);
}
static double Contrast(Color a, Color b)
{
    var (hi, lo) = (Math.Max(Luminance(a), Luminance(b)), Math.Min(Luminance(a), Luminance(b)));
    return (hi + 0.05) / (lo + 0.05);
}
```

Run only one test process per working copy at a time and stop local servers first: suites that start databases, simulators or other apps in parallel can turn headless runs red at random (observed).

## 11. Observed pitfalls

- Local values beat style setters and classes; set state colors only in styles.
- A hover selector that reuses the rest brush hides hover.
- `FontFamily="Name, Fallback"` only finds installed fonts; an embedded font needs its resource family name (for example `fonts:Inter#Inter` in one project's setup); check how your theme registers it.
- Headless image tests with enter or exit animations need at least 200 ms before the frame capture, otherwise the image shows a half fade.
- Stub-based UI tests that match a request by path plus query break when you add a query parameter; update every stub that wires that route.

## 12. Checklist

- [ ] No hex colors or magic numbers in view XAML; all values come from resources (grep).
- [ ] Theme-dependent resources use `DynamicResource` (grep for `StaticResource` on brushes).
- [ ] State styling uses selectors from section 3, with no local brush values on elements (grep for `Background="` in views).
- [ ] The field template has the bottom edge and focus underline (screenshot of rest and focus, both themes).
- [ ] Transitions use the token durations and curves; reduced-motion test passes.
- [ ] Behaviors are attached properties or control subclasses with headless tests.
- [ ] Parsers are pure classes with table tests; no `double` in the money path.
- [ ] Groups are one tab stop; F6 cycles named regions (headless test).
- [ ] Automation names, help, required and live-region settings present on custom controls (test or inspector).
- [ ] Every selector and property used in this file was checked against the project's Avalonia version.

## 13. Sources

Avalonia documentation and source are the authority for selectors, properties and the headless API: [Avalonia documentation][avalonia-docs] and [Avalonia source][avalonia-src]. The Fluent and WinUI rules these patterns implement are cited in the other reference files.

[avalonia-docs]: https://docs.avaloniaui.net/
[avalonia-src]: https://github.com/AvaloniaUI/Avalonia
