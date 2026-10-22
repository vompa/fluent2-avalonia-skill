# Avalonia mapping: how Fluent concepts become AXAML, styles, behaviors and tests

Read this file when you turn a Fluent rule from the other reference files into Avalonia code: choosing selectors for states, wiring tokens, writing transitions, adding keyboard behavior, exposing automation, restoring focus, and testing with headless input. Fluent itself says little about Avalonia; everything here is practical guidance, so treat it as **Derived**. Examples target Avalonia 12; section 11 lists the Avalonia 11 names. Property, event and template-part names change between versions and themes, so confirm each one against the control theme and the version your project uses before copying it. Where a statement is an observed behavior rather than documented, it says **observed** and names the version; every observed item here comes from one Avalonia 12.1.4 desktop project on Windows and was not re-checked on Avalonia 11.

## Contents

1. [Start from what the project already has](#1-start-from-what-the-project-already-has)
2. [Tokens as resources](#2-tokens-as-resources)
3. [States as selectors](#3-states-as-selectors)
   - [Precedence](#precedence)
4. [Templates and the field underline](#4-templates-and-the-field-underline)
   - [Required template parts](#required-template-parts)
5. [Transitions and reduced motion](#5-transitions-and-reduced-motion)
6. [Behaviors as attached properties](#6-behaviors-as-attached-properties)
   - [Tooltip defaults](#tooltip-defaults)
7. [Typed fields: parser, control, ViewModel](#7-typed-fields-parser-control-viewmodel)
8. [Keyboard, focus and navigation](#8-keyboard-focus-and-navigation)
   - [Roving tab stop](#roving-tab-stop)
   - [Toolbar overflow panel](#toolbar-overflow-panel)
9. [Automation](#9-automation)
10. [Headless tests](#10-headless-tests)
    - [What headless does not show](#what-headless-does-not-show)
11. [Avalonia 11 to 12 names](#11-avalonia-11-to-12-names)
12. [Observed pitfalls](#12-observed-pitfalls)
13. [Checklist](#13-checklist)
14. [Sources](#14-sources)

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
- A style cannot override a value that the control template sets itself (observed, Avalonia 12.1.4). Seen with the placeholder `Opacity` of 0.5, the accent fill of the `HighlightBackground` part and the radius and surface of `PopupBorder`: the style is accepted and does nothing, with no warning. Because values set in the template rank above style setters. Check with a probe test (apply the style, read the value from the template part), then either set a local value in `TemplateApplied` (section 4) or override the theme resource the template reads.
- A state selector on a template part needs the `/template/` form, for example `TextBox:pointerover /template/ Border#PART_BorderElement`. The part names come from the theme's template; look them up in the theme source of the version you use.
- A hover style that reuses the same brush as the rest state is invisible. When you change the rest state, re-check every state that shares the token.
- Fluent's disabled state is exempt from contrast rules, but keep label and help text readable: apply the disabled look to the input element, not to the field container.

### Precedence

Three rules decide which value a property ends up with (observed, Avalonia 12.1.4; the ranking is derived from these observations, not taken from the Avalonia documentation):

1. A value that the control template sets itself beats every style setter. A style on the control cannot change it; change it where the template reads it (a theme resource, a control property that the template binds to, or `TemplateApplied`).
2. Among style setters, a rule with a pseudo-class (`:checked`, `:unchecked`, `:pointerover`, `:pressed`) beats the base rule of the same control. The theme ships pseudo-class rules for the states it styles, so your base rule loses against them in exactly those states.
3. A local value (an attribute in XAML, or a value set in code) beats both. State and class styles then have no effect on that property.

Consequences seen in practice:

- CheckBox: a base style on the box part (`Border#NormalRectangle`) left the border at the theme's value while unchecked. The same setter under `CheckBox:unchecked /template/ Border#NormalRectangle` took effect. Write the state you want to change; do not rely on the base rule.
- Dialog layout: `Orientation` or `HorizontalAlignment` written as attributes on an element could not be changed by a class style (compact layout, narrow window). Move such values from the XAML element into a style, so another style can override them.
- MenuItem: the entry radius hangs on `MenuItem.CornerRadius` through a `TemplateBinding`, so it can only be set on the `MenuItem` itself (`<Style Selector="MenuItem"><Setter Property="CornerRadius" .../></Style>`); a setter on the template border does nothing.

When a property does not change, find which of the three sets it (probe test, section 4), then change it at that level.

## 4. Templates and the field underline

Fluent's input has a darker bottom edge and, on focus, a 2 px brand underline that grows from the center. In Avalonia that means a field with two extra elements: a bottom-edge border and a focus underline whose horizontal scale goes from 0 to 1. Sketch of the template route, for when you own the whole template:

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

Getting the shipped template (observed, Avalonia 12.1.4):

- The AXAML source of the Fluent templates is inside the NuGet package only in compiled form, so you cannot copy it offline. A complete replacement template risks losing functions the shipped one has (clear and reveal buttons, selection, context menu, undo, automation). Prefer to keep the shipped template and add to it; the template source on GitHub ([Avalonia source][avalonia-src]) is the readable reference.
- Print the visual tree of the real control with a headless probe test, then read the part names from the output:

```csharp
[AvaloniaFact]
public void Probe_text_box_template()
{
    var box = new TextBox { Text = "x" };
    var window = new Window { Content = box };
    window.Show();
    box.ApplyTemplate();   // the template exists only after this

    foreach (var visual in box.GetVisualDescendants())   // using Avalonia.VisualTree;
        Console.WriteLine($"{visual.GetType().Name} '{(visual as Control)?.Name}' opacity {(visual as Visual)?.Opacity}");
}
```

- Insert the missing parts (bottom edge, focus underline) from an attached property or a control subclass in `TemplateApplied` (`e.NameScope.Find<Panel>("<name from the probe>")`, then `Children.Add(...)`). Put the look and the motion in style classes (`Border.focus-underline` with its `Transitions`), and switch a state class such as `active` from code on focus changes. Because a reduced-motion override can then reach the transitions by class (`foundations.md`, section 8), and a selector that crosses into code-inserted elements through `/template/` is not guaranteed (derived, not verified).

Template parts you will meet in the Avalonia 12 Fluent templates (observed, Avalonia 12.1.4; which control carries which part is derived, not verified, so confirm with the probe):

| Part | Typical owner | What it is |
| --- | --- | --- |
| `PART_BorderElement` | TextBox | Border that draws background, border and radius. Style states through it. |
| `Background` | ComboBox and others | Named element that paints the control surface. |
| `HighlightBackground` | ComboBox | Accent-colored fill shown with keyboard focus; sets its own values (section 3). |
| `PopupBorder` | ComboBox, flyouts | Surface, border and radius of the popup. Sets its own values (section 3). |
| `PART_Placeholder` | TextBox | Placeholder text; the template sets its `Opacity`. |
| `PART_FloatingPlaceholder` | TextBox | Placeholder variant that floats above the text. |
| `PART_TextPresenter` | TextBox | Text, caret and selection. |
| `PART_ScrollViewer` | TextBox (multi-line) | Scrolling around the presenter. |

### Required template parts

A replacement template must contain the parts that the control looks up by name. Names in the Avalonia 12 Fluent theme:

| Control | Part or element | Role | Status |
| --- | --- | --- | --- |
| ToggleSwitch | `PART_SwitchKnob`, `PART_MovingKnobs` | The knob and the element that moves it. A custom template without them does not compile: error AVLN2205. | Observed (12.1.4) |
| CheckBox | `Border#NormalRectangle` | The box that draws border and fill. Style its states through it (section 3, Precedence). | Name observed; whether the template requires it: derived, not verified |
| RadioButton | `Ellipse#OuterEllipse` | The outer ring of the circle. | Name observed; whether the template requires it: derived, not verified |
| MenuItem | `Border#PART_LayoutRoot` | Surface of one entry; reach it with `MenuItem /template/ Border#PART_LayoutRoot`. Its radius is not set here (section 3, Precedence). | Name observed |
| Menu flyout presenter | `Border#LayoutRoot` | Surface of the open menu: background, border, radius, shadow. Reach it through the template of the presenter. | Name observed; the presenter type name: probe it |

Context menus, combo box popups and other controls have their own templates; run the probe from this section to list their names instead of assuming that they match. Copy the names from the probe output of your version, because they are theme details and can change.

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
- Keep the override styles available apart from the OS query, for example as a static `Overrides()` method that the OS hook and the test both apply (pattern in `foundations.md`, section 8). A test that can only run when the OS setting is on will never run.
- Test: with the switch on, walk the visual tree and assert that no element has a non-empty `Transitions` collection with a non-zero duration (section 10).

## 6. Behaviors as attached properties

Put keyboard and focus behavior that applies to many fields into attached properties or small control subclasses, so XAML opts in with one attribute and one test pins the behavior.

Avalonia 12 already selects everything when Tab moves into a single-line `TextBox` (observed, Avalonia 12.1.4). That is right for fast fields (time, date, number, money, search) and wrong for text fields, where editing is the common case and the caret belongs at the end (`inputs.md`, section 5). So a behavior needs two modes, and the headless test comes first because it shows which one a field needs in your version.

```csharp
public enum TabFocusMode { Default, CaretAtEnd, SelectAll }

public static class TabFocus
{
    public static readonly AttachedProperty<TabFocusMode> ModeProperty =
        AvaloniaProperty.RegisterAttached<TextBox, TabFocusMode>("Mode", typeof(TabFocus));

    public static void SetMode(TextBox box, TabFocusMode value) => box.SetValue(ModeProperty, value);
    public static TabFocusMode GetMode(TextBox box) => box.GetValue(ModeProperty);

    static TabFocus() =>
        ModeProperty.Changed.AddClassHandler<TextBox>(OnModeChanged);

    private static void OnModeChanged(TextBox box, AvaloniaPropertyChangedEventArgs e)
    {
        box.GotFocus -= OnGotFocus;
        if (e.NewValue is TabFocusMode.CaretAtEnd or TabFocusMode.SelectAll) box.GotFocus += OnGotFocus;
    }

    // FocusChangedEventArgs is the Avalonia 12 type (section 11 has the older name)
    private static void OnGotFocus(object? sender, FocusChangedEventArgs e)
    {
        if (sender is not TextBox box || e.NavigationMethod != NavigationMethod.Tab) return;
        // Post, because the control applies its own selection while it handles the focus change
        // (derived, not verified); the test shows whether the post is needed.
        Dispatcher.UIThread.Post(() =>
        {
            if (GetMode(box) == TabFocusMode.SelectAll) box.SelectAll();
            else box.CaretIndex = box.Text?.Length ?? 0;   // caret at end, nothing selected
        });
    }
}
```

Use it as `local:TabFocus.Mode="SelectAll"` on time, date, number, money and search fields and `local:TabFocus.Mode="CaretAtEnd"` on name, address and note fields. A text field after a button needs the test "Tab in, `SelectionStart` equals `SelectionEnd` equals the text length"; a fast field needs the test "Tab in, the selection covers the whole text". Write both tests first and let them decide whether the behavior is needed at all.

Other behaviors worth centralizing: Esc restores the last committed value (and consumes the key so the next Esc closes the form); Enter commits and runs the default button; paste sanitizing; wheel changes the value only while focused; arrow-step repeat with a 300 ms delay then acceleration. For one tab stop per group see [Roving tab stop](#roving-tab-stop) in section 8.

### Tooltip defaults

`ToolTip.ShowDelay` and `ToolTip.Placement` are attached properties, and you want one default for the whole app (250 ms, a fixed placement). Calling `OverrideDefaultValue` on them throws "Metadata already set", because `Control` has already set their metadata (observed, Avalonia 12.1.4). The pattern that works: subscribe once to `ToolTip.TipProperty.Changed` and set the values at style priority, so a value set locally on one control still wins.

```csharp
public static class ToolTipDefaults
{
    // Call once at startup, for example from App.Initialize.
    public static void Register() =>
        ToolTip.TipProperty.Changed.AddClassHandler<Control>((control, e) =>
        {
            if (e.NewValue is null) return;   // no tip, nothing to configure
            control.SetValue(ToolTip.ShowDelayProperty, 250, BindingPriority.Style);
            control.SetValue(ToolTip.PlacementProperty, PlacementMode.Top, BindingPriority.Style);
        });
}
```

Test: set `ToolTip.Tip` on a control and assert `ToolTip.GetShowDelay(control)` is 250; set `ToolTip.SetShowDelay(control, 0)` locally afterwards and assert that it stays 0. The values are the project's tokens, not constants (`navigation.md`, section 6).

## 7. Typed fields: parser, control, ViewModel

Follow the pipeline in `inputs.md` section 4:

1. A pure parser class per type (`TimeInputParser`, `DateInputParser`, `MoneyInputParser`) with a method such as `bool TryParse(string text, CultureInfo culture, out T value, out string? error)`. No UI types inside. The error string comes from a resource, not a literal.
2. A templated control (`TimeField`, `NumberField`, ...) that owns raw text, calls the parser on Enter or `LostFocus`, writes the display format back and exposes a typed styled property (`TimeOnly? Value`, `decimal? Amount`).
3. The ViewModel binds only to the typed value and to a validation state; it never sees raw text.
4. Parser unit tests drive a table of inputs and expected results straight from the tables in `inputs.md`, plus invalid and empty inputs.

Money uses `decimal`; the display and parse paths take a `CultureInfo`; no `double` anywhere in the money path. Dependent fields recompute on the committed value change, not on text change.

## 8. Keyboard, focus and navigation

- Group as one tab stop: `KeyboardNavigation.TabNavigation="Once"` on the group container with arrow handling inside (radio groups, button strips). `Cycle` wraps inside popups such as menus; `Continue` is the default. When the one tab stop must be the selected item (tab strip, radio group), use the recipe under [Roving tab stop](#roving-tab-stop).
- Region switching with F6: keep an ordered list of region roots, intercept `KeyDown` for F6 and Shift+F6 on the window, move focus to the first focusable descendant of the next region. Give each root `AutomationProperties.Name`.
- Restoring focus after a transient surface closes: store the invoker before opening, then call `invoker.Focus()` after close (`Dispatcher.UIThread.Post` when closing from inside a handler). When the focused element hides itself, no element owns focus and keys never reach your view (observed); listen with `AddHandler(InputElement.KeyDownEvent, handler, RoutingStrategies.Tunnel)` on the window, and if `TopLevel.GetTopLevel(this)?.FocusManager?.GetFocusedElement()` is null, restore focus and handle the key. (`TopLevel.GetTopLevel` is the Avalonia 12 way to reach the root; section 11 has the older name.)
- Menus and flyouts: when a click handler closes a flyout, its items can lose their `DataContext`, so command parameters arrive empty (observed). Run the command first and close the flyout afterwards with `Dispatcher.UIThread.Post`.
- Esc and parent views with tunnel handlers: when a parent view handles keys in the tunnel phase (to close a panel on Esc, to move a selection with arrows), its handler runs before the focused field, so the field never gets its own Esc (restore the value) and the panel closes with the edit lost (observed, Avalonia 12.1.4). Pattern: the field publishes an attached property such as `HasPendingChange` (true while the text differs from the last committed value or while a popup is open), and the parent handler asks the focused element before it acts:

```csharp
private void OnTunnelKeyDown(object? sender, KeyEventArgs e)
{
    var focused = TopLevel.GetTopLevel(this)?.FocusManager?.GetFocusedElement();
    if (e.Key == Key.Escape && focused is Control field && PendingChange.GetHasPendingChange(field))
        return;   // the field restores its value first; the next Esc reaches this handler
    // ... close the panel, move the selection
}
```

  Keep the field-side rule from `inputs.md` (section 3): it clears `HasPendingChange` when it restores or commits and marks the key handled.
- `Button.IsDefault` is a window-wide hotkey for Enter (observed, Avalonia 12.1.4): it fires for every Enter that no control handled, anywhere in the window, and when several visible buttons are default, all of them fire. Inline cards and side panels with their own default button make Enter in one place run the action of another. Set `IsDefault` on one button per visible form or panel only (bind it to the "panel is open and active" state, or set it when the form opens and clear it when it closes). A multi-line field handles Enter itself (`AcceptsReturn`), so it does not trigger the default button. Avalonia has no built-in Ctrl+Enter; add it per field, for example `<TextBox.KeyBindings><KeyBinding Gesture="Ctrl+Enter" Command="{Binding SaveCommand}"/></TextBox.KeyBindings>` (derived, not verified for your version).
- Dragging: releasing pointer capture inside `PointerReleased` raises `PointerCaptureLost` at once and cancels the operation; finish the drop first, then release (observed).
- Destructive dialogs: put `IsDefault` on the safe button (Cancel), not on the destructive one, and `IsCancel` on Cancel; the dialog's initial focus goes to the same button (`dialogs-and-messages.md`, section 2).

### Roving tab stop

A tab strip or radio group is one tab stop: Tab lands on the selected item, the next Tab leaves the group, arrows move inside. Avalonia has no ready-made pattern for this. Recipe used in an Avalonia 12.1.4 project (observed):

1. A style takes every item that is not selected out of the tab order. The item stays focusable from code, because `IsTabStop` only affects Tab.
2. An attached behavior on the container handles the arrow keys, moves focus to the neighbor and, for a radio group, selects it too. Make one behavior per model (for example `TabStripBehavior`: arrows move focus, Enter or Space selects; `RadioGroupBehavior`: arrows move and select), or one behavior with a mode.

```xml
<!-- Adapt the item type: ToggleButton, RadioButton, or TabItem (pseudo-class :selected). -->
<Style Selector="StackPanel.roving.has-selection > ToggleButton:not(:checked)">
  <Setter Property="IsTabStop" Value="False"/>
</Style>
```

```csharp
public static class RovingFocus
{
    public static readonly AttachedProperty<bool> IsEnabledProperty =
        AvaloniaProperty.RegisterAttached<Panel, bool>("IsEnabled", typeof(RovingFocus));

    public static void SetIsEnabled(Panel panel, bool value) => panel.SetValue(IsEnabledProperty, value);
    public static bool GetIsEnabled(Panel panel) => panel.GetValue(IsEnabledProperty);

    static RovingFocus() => IsEnabledProperty.Changed.AddClassHandler<Panel>((panel, e) =>
    {
        panel.RemoveHandler(InputElement.KeyDownEvent, OnKeyDown);
        if (e.NewValue is true) panel.AddHandler(InputElement.KeyDownEvent, OnKeyDown);
    });

    private static void OnKeyDown(object? sender, KeyEventArgs e)
    {
        if (sender is not Panel panel) return;
        var step = e.Key switch { Key.Left or Key.Up => -1, Key.Right or Key.Down => 1, _ => 0 };
        if (step == 0) return;

        var items = panel.Children.OfType<InputElement>().Where(c => c.IsVisible && c.IsEffectivelyEnabled).ToList();
        var current = items.IndexOf((TopLevel.GetTopLevel(panel)?.FocusManager?.GetFocusedElement() as InputElement)!);
        var next = current + step;
        if (current < 0 || next < 0 || next >= items.Count) return;   // no wrap at the ends

        items[next].Focus(NavigationMethod.Directional);
        if (items[next] is RadioButton radio) radio.IsChecked = true;   // selection follows focus (radio group only)
        e.Handled = true;
    }
}
```

The style needs the class `has-selection` on the container while one item is selected (set it from the behavior or bind it to the ViewModel); with no selection every item stays a tab stop, so the group can still be entered (derived, not verified). Check member names such as `Focus(NavigationMethod, ...)` against your version.

Tests: Tab into the group lands on the selected item; the next Tab leaves the group; Right and Left move focus (and selection, for radios); the last item does not wrap; `Focus()` on an item that is not a tab stop still works.

### Toolbar overflow panel

A toolbar that never wraps needs a panel that measures its children and moves those that do not fit into a menu (`navigation.md`, section 5). Sketch (Derived; not verified in full):

```csharp
public sealed class OverflowPanel : Panel
{
    // Children: the items first, the overflow ("More") button last.
    private readonly List<Control> _hidden = new();
    public IReadOnlyList<Control> HiddenItems => _hidden;
    public event EventHandler? HiddenItemsChanged;

    protected override Size MeasureOverride(Size available)
    {
        var more = Children[^1];
        var items = Children.Take(Children.Count - 1).ToList();
        foreach (var child in Children) child.Measure(Size.Infinity);   // natural sizes

        var fits = items.Sum(i => i.DesiredSize.Width) <= available.Width;
        var budget = fits ? available.Width : available.Width - more.DesiredSize.Width;

        var hidden = new List<Control>();
        double used = 0;
        foreach (var item in items)
        {
            used += item.DesiredSize.Width;
            if (used > budget) hidden.Add(item);
        }

        foreach (var item in items) item.Classes.Set("overflowed", hidden.Contains(item));
        more.Classes.Set("overflowed", hidden.Count == 0);
        if (!hidden.SequenceEqual(_hidden))
        {
            _hidden.Clear();
            _hidden.AddRange(hidden);
            // Fill the menu after layout, never inside measure.
            Dispatcher.UIThread.Post(() => HiddenItemsChanged?.Invoke(this, EventArgs.Empty));
        }

        var visibleWidth = items.Except(hidden).Sum(i => i.DesiredSize.Width) + (hidden.Count > 0 ? more.DesiredSize.Width : 0);
        return new Size(Math.Min(visibleWidth, available.Width), Children.Max(c => c.DesiredSize.Height));
    }

    protected override Size ArrangeOverride(Size finalSize)
    {
        double x = 0;
        foreach (var child in Children)
        {
            if (child.Classes.Contains("overflowed")) { child.Arrange(new Rect(x, 0, 0, 0)); continue; }   // 0 x 0
            child.Arrange(new Rect(x, 0, child.DesiredSize.Width, finalSize.Height));
            x += child.DesiredSize.Width;
        }
        return finalSize;
    }
}
```

```xml
<Style Selector="local|OverflowPanel > Control.overflowed">
  <Setter Property="IsEnabled" Value="False"/>
  <Setter Property="Opacity" Value="0"/>
</Style>
```

A hidden child needs all three: `IsEnabled` false, `Opacity` 0 and a 0 x 0 arrange. With less, the invisible items stay in the tab order and clog it (observed, Avalonia 12.1.4). Changing `IsVisible` from inside measure would invalidate the layout again (derived, not verified), which is why the sketch uses a class. When `HiddenItemsChanged` fires, rebuild the menu entries from `HiddenItems` with their text labels (icon-only items need a text in the menu) and commands. Tests: at the minimum window width and at a wide width, press Tab through the toolbar and compare the focus sequence with the visible items plus the "More" button; open the menu and check that every hidden command is in it.

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

For custom controls override `OnCreateAutomationPeer` and return a peer that reports the right role and, for range controls, the range-value pattern (minimum, maximum, value). A `TextBox` peer already implements `IValueProvider`, whose `Value` is the text and clashes by name with `IRangeValueProvider.Value`, which is a number. Derive from `TextBoxAutomationPeer` and implement the range interface explicitly, so both patterns stay available (Derived; let the compiler list the members of the interface in your version):

```csharp
public sealed class NumberFieldAutomationPeer : TextBoxAutomationPeer, IRangeValueProvider
{
    private readonly NumberField _field;

    public NumberFieldAutomationPeer(NumberField owner) : base(owner) => _field = owner;

    // Explicit, because IValueProvider.Value (string) is inherited from the base peer.
    double IRangeValueProvider.Value => (double)(_field.Value ?? _field.Minimum);
    double IRangeValueProvider.Minimum => (double)_field.Minimum;
    double IRangeValueProvider.Maximum => (double)_field.Maximum;
    double IRangeValueProvider.SmallChange => (double)_field.Step;
    double IRangeValueProvider.LargeChange => (double)_field.LargeStep;
    bool IRangeValueProvider.IsReadOnly => _field.IsReadOnly;
    void IRangeValueProvider.SetValue(double value) => _field.Value = (decimal)value;
}

// in NumberField
protected override AutomationPeer OnCreateAutomationPeer() => new NumberFieldAutomationPeer(this);
```

The conversion to `double` is for the automation surface only; the money path stays `decimal`. Keep the step buttons of a number field out of the tab order (`Focusable="False"` or `IsTabStop="False"`) and name them "Increase" and "Decrease". Fluent and WinUI sources for the rules: see `accessibility.md`.

## 10. Headless tests

Use the headless platform with the Skia renderer when you need frames. Two ways to run tests on it, depending on the test framework:

- xUnit v2 or NUnit: the `Avalonia.Headless.XUnit` or `Avalonia.Headless.NUnit` package with `[AvaloniaFact]` or `[AvaloniaTest]` (below).
- xUnit v3: the attribute packages do not fit; start one `HeadlessUnitTestSession` for the test assembly and run each test body through `Session.Dispatch` (observed, Avalonia 12.1.4). Share one session, because the process hosts one Avalonia application.

```csharp
public static class Headless
{
    public static readonly HeadlessUnitTestSession Session =
        HeadlessUnitTestSession.StartNew(typeof(TestAppBuilder));
}

public class TimeFieldTests
{
    [Fact]
    public Task Tab_into_time_field_selects_all() =>
        Headless.Session.Dispatch(() =>
        {
            // build the window, press keys, assert, exactly as in the [AvaloniaFact] example below
        }, CancellationToken.None);
}
```

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
    TabFocus.SetMode(field, TabFocusMode.SelectAll);
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

Saving many PNG frames in one run can crash the test host with an access violation (0xC0000005) inside Skia (observed, Avalonia 12.1.4, Windows). Keep image tests in their own, smaller runs (one theme or one view group per run, selected by trait or class filter) and keep the standard run free of bulk image saves.

Run only one test process per working copy at a time and stop local servers first: suites that start databases, simulators or other apps in parallel can turn headless runs red at random (observed).

### What headless does not show

- Tooltips: in a headless test a tooltip appears in the window's overlay layer instead of a popup window (observed, Avalonia 12.1.4). You can assert that it opens, its content, the delay and its placement relative to the target; shadow, border and behavior at a screen edge cannot be verified. Look at those in the running app and say so in the report. Other popups (menus, flyouts, combo lists) may behave the same way (derived, not verified).
- A `MinWidth` on the window clamps the width a test asks for (observed): a responsive test at 600 px inside a window with `MinWidth` 800 measures 800 px. Lift the minimum inside the test (`window.MinWidth = 0`) or measure the content directly (`content.Measure(new Size(600, 800))`).

## 11. Avalonia 11 to 12 names

Avalonia 12 renames several APIs and template parts that this skill's patterns use. Observed in Avalonia 12.1.4; the right-hand column is the name to use there, the left-hand column is what Avalonia 11 code and older examples contain. Docs: [Avalonia documentation][avalonia-docs].

| Avalonia 11 | Avalonia 12 | Where it shows up |
| --- | --- | --- |
| `GotFocusEventArgs` | `FocusChangedEventArgs` | Handlers for `GotFocus`, `LostFocus`; section 6 |
| `OnGotFocus(GotFocusEventArgs)`, `OnLostFocus(RoutedEventArgs)` overrides | `OnGotFocus(FocusChangedEventArgs)`, `OnLostFocus(FocusChangedEventArgs)` | Custom field controls |
| `Visual.GetVisualRoot()` | `TopLevel.GetTopLevel(visual)` | Focus manager access; section 8 |
| `Watermark` (and its foreground) | `PlaceholderText` with `PlaceholderForeground` | Placeholder styling and tests |
| Watermark-named template parts | `PART_Placeholder`, `PART_FloatingPlaceholder` | Styles that reach into the TextBox template; section 4 |

How to find the right name in your version instead of guessing: build and read the compiler error (it names the missing member or type), or list the members by reflection (`typeof(TextBox).GetProperties()`, `typeof(TextBox).Assembly.GetTypes()` filtered by name), or run the template probe from section 4.

## 12. Observed pitfalls

- Local values beat style setters and classes; set state colors only in styles.
- Template-set values beat styles too (placeholder `Opacity`, `HighlightBackground`, `PopupBorder`); see section 3. Probe, then set a local value in `TemplateApplied` or override the theme resource.
- A hover selector that reuses the rest brush hides hover.
- `FontFamily="Name, Fallback"` only finds installed fonts; an embedded font needs its resource family name (for example `fonts:Inter#Inter` in one project's setup); check how your theme registers it.
- Headless image tests with enter or exit animations need at least 200 ms before the frame capture, otherwise the image shows a half fade.
- Stub-based UI tests that match a request by path plus query break when you add a query parameter; update every stub that wires that route.
- Pseudo-class rules beat base rules; a base style on a template part can lose against the theme's `:unchecked` or `:pointerover` rule. Write the state selector (section 3, Precedence).
- `MenuItem` radius is a `TemplateBinding` to `MenuItem.CornerRadius`; set it on the `MenuItem`, not on the template border (section 3).
- `OverrideDefaultValue` on `ToolTip.ShowDelay` or `ToolTip.Placement` throws "Metadata already set"; use the `ToolTip.TipProperty.Changed` pattern (section 6).
- A custom `ToggleSwitch` template without `PART_SwitchKnob` and `PART_MovingKnobs` fails with AVLN2205 (section 4).
- Overflowed toolbar items that are only transparent still take Tab stops; disable them and arrange them at 0 x 0 (section 8).
- A window `MinWidth` clamps headless test widths (section 10).

## 13. Checklist

- [ ] No hex colors or magic numbers in view XAML; all values come from resources (grep).
- [ ] Theme-dependent resources use `DynamicResource` (grep for `StaticResource` on brushes).
- [ ] State styling uses selectors from section 3, with no local brush values on elements (grep for `Background="` in views).
- [ ] The field template has the bottom edge and focus underline (screenshot of rest and focus, both themes).
- [ ] Transitions use the token durations and curves; reduced-motion test passes.
- [ ] Behaviors are attached properties or control subclasses with headless tests.
- [ ] Every style that targets a template part was checked with a probe test; values the template sets itself are overridden in `TemplateApplied` or through the theme resource (section 3).
- [ ] A property that does not change was traced to its source (template, pseudo-class rule, local value) and changed there (section 3, Precedence).
- [ ] A custom template contains the required parts of its control (section 4, Required template parts; the compiler or a probe test confirms it).
- [ ] Names that changed between Avalonia 11 and 12 match the project's version (section 11).
- [ ] Parsers are pure classes with table tests; no `double` in the money path.
- [ ] Groups are one tab stop and the stop is the selected item; arrows do not wrap; F6 cycles named regions (headless test; section 8).
- [ ] An overflow panel hides items completely (disabled, transparent, 0 x 0) and Tab visits only visible items (headless test at minimum and wide width).
- [ ] Tooltip delay and placement come from one app-wide default, not from `OverrideDefaultValue` (section 6).
- [ ] Automation names, help, required and live-region settings present on custom controls (test or inspector).
- [ ] Every selector and property used in this file was checked against the project's Avalonia version.

## 14. Sources

Avalonia documentation and source are the authority for selectors, properties and the headless API: [Avalonia documentation][avalonia-docs] and [Avalonia source][avalonia-src]. The Fluent and WinUI rules these patterns implement are cited in the other reference files.

[avalonia-docs]: https://docs.avaloniaui.net/
[avalonia-src]: https://github.com/AvaloniaUI/Avalonia
