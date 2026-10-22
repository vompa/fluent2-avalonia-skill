# Avalonia mapping: how Fluent concepts become AXAML, styles, behaviors and tests

Read this file when you turn a Fluent rule from the other reference files into Avalonia code: choosing selectors for states, wiring tokens, writing transitions, adding keyboard behavior, exposing automation, restoring focus, and testing with headless input. Fluent itself says little about Avalonia; everything here is practical guidance, so treat it as **Derived**. Examples target Avalonia 12; [avalonia-12-notes.md](avalonia-12-notes.md) lists the Avalonia 11 names, changed behavior and observed pitfalls. Property, event and template-part names change between versions and themes, so confirm each one against the control theme and the version your project uses before copying it. Where a statement is an observed behavior rather than documented, it says **observed** and names the version; every observed item here comes from one Avalonia 12.1.4 desktop project on Windows and was not re-checked on Avalonia 11.

## Contents

1. [Start from what the project already has](#1-start-from-what-the-project-already-has)
2. [Tokens as resources](#2-tokens-as-resources)
3. [States as selectors](#3-states-as-selectors)
   - [Precedence](#precedence)
4. [Templates and the field underline](#4-templates-and-the-field-underline)
   - [Required template parts](#required-template-parts)
   - [Layout notes](#layout-notes)
5. [Transitions and reduced motion](#5-transitions-and-reduced-motion)
   - [Transforms from styles](#transforms-from-styles)
6. [Behaviors as attached properties](#6-behaviors-as-attached-properties)
   - [Tooltip defaults](#tooltip-defaults)
7. [Typed fields: parser, control, ViewModel](#7-typed-fields-parser-control-viewmodel)
8. [Keyboard, focus and navigation](#8-keyboard-focus-and-navigation)
   - [Roving tab stop](#roving-tab-stop)
   - [Toolbar overflow panel](#toolbar-overflow-panel)
   - [F6 regions](#f6-regions)
   - [Tab order walk](#tab-order-walk)
9. [Automation](#9-automation)
   - [Expanded state on a toggle header](#expanded-state-on-a-toggle-header)
   - [Finding and fixing accessible names](#finding-and-fixing-accessible-names)
10. [Headless tests](#10-headless-tests)
    - [Reading transitions](#reading-transitions)
    - [What headless does not show](#what-headless-does-not-show)
    - [Flyouts, keys and commands](#flyouts-keys-and-commands)
    - [Scrolling containers](#scrolling-containers)
    - [Instances built from XAML](#instances-built-from-xaml)
11. [Checklist](#11-checklist)
12. [Sources](#12-sources)

Avalonia 11 to 12 names, changed behavior and a list of observed pitfalls live in [avalonia-12-notes.md](avalonia-12-notes.md).

## 1. Start from what the project already has

- Find the theme setup first: `App.axaml` (`FluentTheme` or a custom theme), the project's `Styles.axaml` and `Colors.axaml` (names vary), a design brief and a token file. Extend those; do not add a parallel style system. Because two sources of truth drift apart.
- Avalonia's built-in Fluent theme follows Fluent-style visuals but is not a promise of Fluent 2 conformance in every detail (heights, field underline, focus drawing). Compare it against `inputs.md` and `foundations.md` and override only what the brief requires.
- Prefer a style or a `ControlTheme` override over a new control class, and a new control class (templated, with a small ViewModel-friendly API) over duplicating XAML in each view. Because behavior such as parsing and Esc handling belongs in one place.

## 2. Tokens as resources

- Put every color, radius, thickness, height and duration in a resource dictionary with semantic names (for example `SurfaceBrush`, `FieldBorderBrush`, `AccentBrush`, `ControlHeight`, `ControlRadius`). Per-theme values go into `ThemeDictionaries` under the keys `Light` and `Dark`.
- Reference theme-dependent values with `DynamicResource`. `StaticResource` binds once and does not follow a theme switch. Because a runtime switch between light, dark and a future high-contrast variant must repaint everything.
- No color literals in view XAML: no hex, no named colors (`White`, `Black`), no colors inside shadow strings. A scan for `#` alone misses the last two (observed, Avalonia 12.1.4). Run all three in CI over the view folders, excluding the token file, and add the code-behind check:

```bash
# hex colors (3 to 8 digits, with alpha), also inside BoxShadow strings
grep -rnE "#[0-9A-Fa-f]{3,8}\b" --include=*.axaml Views/
# named colors in color-bearing attributes (extend the list with the names your code base uses)
grep -rnE "(Background|Foreground|Fill|Stroke|BorderBrush|Color)=\"(White|Black|Red|Green|Blue|Gray|Grey|Yellow|Orange)\"" --include=*.axaml Views/
# shadows written inline instead of through a resource
grep -rnE "BoxShadow=\"[^{]" --include=*.axaml Views/
# colors built in code
grep -rnE "Brushes\.|Colors\.|Color\.Parse|Color\.FromRgb" --include=*.cs Views/
```

  Adjust the folder names; allow `Transparent` where it is meant. Because every literal that gets through is a color that high contrast and the dark theme cannot repaint.

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

Shadows are colors too. Declare them as `BoxShadows` resources per theme (a dark theme needs a stronger shadow alpha than a light one) and reference them with `DynamicResource` (observed, Avalonia 12.1.4):

```xml
<ResourceDictionary.ThemeDictionaries>
  <ResourceDictionary x:Key="Light">
    <BoxShadows x:Key="PopupShadow">0 8 16 0 #29000000</BoxShadows>
  </ResourceDictionary>
  <ResourceDictionary x:Key="Dark">
    <BoxShadows x:Key="PopupShadow">0 8 16 0 #52000000</BoxShadows>
  </ResourceDictionary>
</ResourceDictionary.ThemeDictionaries>
<!-- in a view: <Border BoxShadow="{DynamicResource PopupShadow}"/> -->
```

The values are placeholders; use the project's shadow tokens (`foundations.md`, section 7).

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
- Tooltip text: a tooltip is a logical child of its target, so a descendant selector of the target (`Border.chip TextBlock`) also styles the text inside the tooltip (observed, Avalonia 12.1.4). Seen: tooltip text in the target's text color on the dark tooltip surface, contrast 1.0. Use the child combinator for text that belongs to the target (`Border.chip > TextBlock`) and test every real tooltip (`navigation.md`, section 6).
- MenuItem: the entry radius hangs on `MenuItem.CornerRadius` through a `TemplateBinding`, so it can only be set on the `MenuItem` itself (`<Style Selector="MenuItem"><Setter Property="CornerRadius" .../></Style>`); a setter on the template border does nothing.
- Style key: a control that overrides `StyleKeyOverride` (for example `protected override Type StyleKeyOverride => typeof(ToggleButton);` so that it reuses the theme of `ToggleButton`) is not hit by a style on its own type, `c|AccordionHeader` (observed, Avalonia 12.1.4). Select the base type plus a class instead, and add the class in the control's constructor so no view can forget it. Because type selectors compare against the style key, not the CLR type (derived, not verified).

```xml
<!-- wrong: matches nothing, no warning -->
<Style Selector="c|AccordionHeader"><Setter Property="MinHeight" Value="44"/></Style>
<!-- right: base type of the style key plus the class that the control adds to itself -->
<Style Selector="ToggleButton.accordion"><Setter Property="MinHeight" Value="44"/></Style>
```

  Test "the style applies": create the control in a window with the app styles and assert a value that only this style sets. Write the test first and watch it fail with the wrong selector; pick a property whose theme value differs, or it passes for the wrong reason.

```csharp
[AvaloniaFact]
public void Accordion_header_style_applies()
{
    var header = new AccordionHeader();            // StyleKeyOverride => typeof(ToggleButton); adds the class "accordion"
    var window = new Window { Content = header };
    window.Show();
    Assert.Equal(44, header.MinHeight);            // set only by the accordion style
}
```

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

- Insert the missing parts (bottom edge, focus underline) from an attached property or a control subclass in `TemplateApplied` (`e.NameScope.Find<Panel>("<name from the probe>")`, then `Children.Add(...)`). Put the look and the motion in style classes (`Border.focus-underline` with its `Transitions`), and switch a state class such as `active` from code on focus changes. Because the motion then lives in styles, where the reduced-motion override (section 5) reaches it, and a selector that crosses into code-inserted elements through `/template/` is not guaranteed (derived, not verified).

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

### Layout notes

Layout findings from an Avalonia 12.1.4 project (observed); the fixes are Derived, so check each against the real layout.

- Two `*` columns are not equal under infinite width. When a `Grid` is measured without a width limit (inside a popup or flyout that sizes to its content, a horizontal `ScrollViewer`, a `StackPanel`), each star column takes the width of its content, so two columns of unequal content stay unequal. Give the surrounding frame a width: stretch it (`HorizontalAlignment="Stretch"`) with a `MaxWidth`, or a fixed `Width`, so the grid is measured with a finite width. Check with the `ActualWidth` of both columns' content.
- One fixed column across several lists: put `SharedSizeGroup="Keys"` on the column definition in each row template and `Grid.IsSharedSizeScope="True"` on a common ancestor of all the `ItemsControl`s. The column then takes the width of the widest cell of every list in the scope, so keys, labels or times line up across groups. Without the scope, every row sizes on its own. A column with a shared size group must be `Auto`-sized or fixed.
- A theme style with `MinHeight` 34 on `Button` turns `Width="28" Height="28"` into a 28 x 34 element, because the minimum beats the explicit height (observed, Avalonia 12.1.4). Small icon buttons need `MinHeight="28"` (and `MinWidth` where the theme sets one) next to the size, ideally in the style of the icon-button class. Measure the target: after layout, assert `Bounds.Width` and `Bounds.Height` of every small button against the design size, because the screenshot of a 28 x 34 button looks plausible.
- `DesiredSize` includes the element's `Margin`. When you re-measure row heights in code or compare them with a design value in a test, subtract the vertical margin, or the result is too large by the margin (derived from the layout contract; confirm with one row).

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
- A class on the root window plus a style that only selects the control type and clears `Transitions` does not work in general (observed, Avalonia 12.1.4). A setter under a class or pseudo-class (`.toast`, `:pressed`) ranks above a style that selects only the type, so the override loses exactly where the app's own motion is set (section 3, Precedence). Because the override has to outrank every style that sets `Transitions`. The pattern that worked has three parts:
  1. An activator style: the selector carries a class condition that never matches anything, which lifts the style to the rank of class rules and still hits every control.

```csharp
public static class ReducedMotion
{
    // Applied by the OS hook and by the test alike; works without the OS setting.
    public static Styles Overrides() => new()
    {
        new Style(x => x.Is<Control>().Not(y => y.Class("never-set")))
        {
            Setters = { new Setter(Animatable.TransitionsProperty, new Transitions()) },
        },
    };
}
// OS hook or setting: Application.Current!.Styles.Add(ReducedMotion.Overrides());
```

  2. Attach the overrides to the `Application` before the first window exists (for example in `OnFrameworkInitializationCompleted`, before the main window is created). Styles added to a window that is already open left a 75 ms transition of the Fluent template in place (observed).
  3. In the test, ignore template parts that are not visible. Avalonia sets a transition on a part of the `ToggleSwitch` template in code, and no style reaches it; the part is not drawn, so it does not matter to the user.
- Test: apply `ReducedMotion.Overrides()` to the application, open every page, and assert that no visible element has a transition with a duration above zero. `ITransition` does not expose its duration, so read it as shown in [Reading transitions](#reading-transitions) (section 10). A test that can only run when the OS setting is on will never run; this is why the override is a plain method.

### Transforms from styles

A `RenderTransform` set by a style setter (`rotate(90deg)`, `scale(0.97)`) is a `TransformOperations`, not a `RotateTransform` or `ScaleTransform` (observed, Avalonia 12.1.4). A test that casts to `RotateTransform` gets `null` (or an exception) and fails for the wrong reason. Read the resulting matrix, which every transform type exposes through `Value` (check the member in your version):

```csharp
static double RotationDegrees(Visual element)
{
    var m = element.RenderTransform!.Value;                  // Matrix of the transform operations
    return Math.Atan2(m.M12, m.M11) * 180 / Math.PI;
}
// Assert.Equal(90, RotationDegrees(chevron), precision: 3);
```

With a `TransformOperationsTransition` on the element the value is intermediate while the animation runs; read it after the transition or with the reduced-motion override applied (section 5, Reduced motion), so the test sees the end state. Scale reads from `m.M11` and `m.M22`.

## 6. Behaviors as attached properties

Put keyboard and focus behavior that applies to many fields into attached properties or small control subclasses, so XAML opts in with one attribute and one test pins the behavior.

Avalonia 12 already selects everything when Tab moves into a single-line `TextBox` (observed, Avalonia 12.1.4). That is right for fast fields (time, date, number, money, search) and wrong for text fields, where editing is the common case and the caret belongs at the end (`inputs.md`, section 5). So a behavior needs two modes, and the headless test comes first because it shows which one a field needs in your version.

```csharp
public enum TabFocusMode { Default, CaretAtEnd, SelectAll }

// The owner of an attached property must not be a static class in Avalonia 12 (observed;
// avalonia-12-notes.md, section 2). If an analyzer asks for "static", suppress it for this type.
public abstract class TabFocus
{
    public static readonly AttachedProperty<TabFocusMode> ModeProperty =
        AvaloniaProperty.RegisterAttached<TabFocus, TextBox, TabFocusMode>("Mode");

    public static void SetMode(TextBox box, TabFocusMode value) => box.SetValue(ModeProperty, value);
    public static TabFocusMode GetMode(TextBox box) => box.GetValue(ModeProperty);

    static TabFocus() =>
        ModeProperty.Changed.AddClassHandler<TextBox>(OnModeChanged);

    private static void OnModeChanged(TextBox box, AvaloniaPropertyChangedEventArgs e)
    {
        box.GotFocus -= OnGotFocus;
        if (e.NewValue is TabFocusMode.CaretAtEnd or TabFocusMode.SelectAll) box.GotFocus += OnGotFocus;
    }

    // FocusChangedEventArgs is the Avalonia 12 type (avalonia-12-notes.md, section 1, has the older name)
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
- Region switching with F6: mark regions with a named attached property, order them as they appear on screen, intercept `KeyDown` for F6 and Shift+F6 on the window and move focus to each region's focus target; recipe under [F6 regions](#f6-regions).
- Tab order is source order: see [Tab order walk](#tab-order-walk) for the `DockPanel` trap and a test that prints the real order.
- Restoring focus after a transient surface closes: store the invoker before opening, then call `invoker.Focus()` after close (`Dispatcher.UIThread.Post` when closing from inside a handler). When the focused element hides itself, no element owns focus and keys never reach your view (observed); listen with `AddHandler(InputElement.KeyDownEvent, handler, RoutingStrategies.Tunnel)` on the window, and if `TopLevel.GetTopLevel(this)?.FocusManager?.GetFocusedElement()` is null, restore focus and handle the key. (`TopLevel.GetTopLevel` is the Avalonia 12 way to reach the root; [avalonia-12-notes.md](avalonia-12-notes.md), section 1, has the older name.)
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
public abstract class RovingFocus   // not static: the owner of an attached property cannot be (avalonia-12-notes.md, section 2)
{
    public static readonly AttachedProperty<bool> IsEnabledProperty =
        AvaloniaProperty.RegisterAttached<RovingFocus, Panel, bool>("IsEnabled");

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

### F6 regions

Windows gives custom shells no F6 handling, so build it (`accessibility.md`, section 4, has the rules). Recipe from an Avalonia 12.1.4 project (observed): an attached property names a region, one handler on the window cycles the regions, and each region may name its own focus target.

```csharp
public abstract class Region   // not static: attached-property owner (avalonia-12-notes.md, section 2)
{
    public static readonly AttachedProperty<string?> NameProperty =
        AvaloniaProperty.RegisterAttached<Region, Control, string?>("Name");
    public static readonly AttachedProperty<Control?> FocusTargetProperty =
        AvaloniaProperty.RegisterAttached<Region, Control, Control?>("FocusTarget");

    public static void SetName(Control c, string? value) => c.SetValue(NameProperty, value);
    public static string? GetName(Control c) => c.GetValue(NameProperty);
    public static void SetFocusTarget(Control c, Control? value) => c.SetValue(FocusTargetProperty, value);
    public static Control? GetFocusTarget(Control c) => c.GetValue(FocusTargetProperty);

    // The region name is also the automation name, so a screen reader announces where focus lands.
    static Region() => NameProperty.Changed.AddClassHandler<Control>(
        (c, e) => AutomationProperties.SetName(c, e.NewValue as string));

    // Call from a tunnel KeyDown handler on the window; pageRoot is the current page or the window content.
    public static void HandleF6(Control pageRoot, KeyEventArgs e)
    {
        if (e.Key != Key.F6) return;

        // Screen order (top to bottom, then left to right), not tree order: the two differ with docked children.
        var regions = pageRoot.GetSelfAndVisualDescendants().OfType<Control>()
            .Where(c => GetName(c) is not null && c.IsEffectivelyVisible)
            .OrderBy(c => c.TranslatePoint(default, pageRoot)?.Y ?? 0)
            .ThenBy(c => c.TranslatePoint(default, pageRoot)?.X ?? 0)
            .ToList();
        if (regions.Count == 0) regions.Add(pageRoot);   // fallback: the whole page is one region

        var focused = TopLevel.GetTopLevel(pageRoot)?.FocusManager?.GetFocusedElement() as Visual;
        var current = regions.FindIndex(r => focused is not null && (r == focused || r.IsVisualAncestorOf(focused)));
        var step = e.KeyModifiers.HasFlag(KeyModifiers.Shift) ? -1 : 1;
        var from = current >= 0 ? current : (step > 0 ? -1 : 0);
        var next = regions[(from + step + regions.Count) % regions.Count];

        // Explicit target first (the view itself when it handles keys), else the first focusable descendant.
        var target = GetFocusTarget(next)
            ?? next.GetVisualDescendants().OfType<InputElement>()
                   .FirstOrDefault(i => i.Focusable && i.IsEffectivelyEnabled && i.IsEffectivelyVisible)
            ?? next;
        target.Focus(NavigationMethod.Tab);
        e.Handled = true;
    }
}
```

Rules in the code, each for a reason:

- Order by screen position, because tree order follows the XAML and a bottom-docked bar written first would otherwise be the first region.
- Fallback to the whole page, so F6 still moves focus into a page that declares no regions instead of doing nothing.
- An explicit focus target per region. A view that handles keys itself (a calendar, a canvas, a list with custom arrows) sets itself as the target and is `Focusable`, because focus on its first button would send arrows to the wrong handler.
- Hook it once per window: `window.AddHandler(InputElement.KeyDownEvent, (_, e) => Region.HandleF6(CurrentPage, e), RoutingStrategies.Tunnel)`, and check that no global key binding already uses F6.

Test with real keys: press F6 repeatedly and compare the names of the regions that hold focus with the expected order, including wrap-around; press Shift+F6 and compare the reverse.

```csharp
window.KeyPressQwerty(PhysicalKey.F6, RawInputModifiers.None);
var focused = window.FocusManager!.GetFocusedElement() as Visual;
var region = focused!.GetSelfAndVisualAncestors().OfType<Control>().First(c => Region.GetName(c) is not null);
Assert.Equal("Navigation", Region.GetName(region));
// Shift+F6: window.KeyPressQwerty(PhysicalKey.F6, RawInputModifiers.Shift);
```

### Tab order walk

Tab follows source (tree) order, not the layout you see (observed, Avalonia 12.1.4). Seen case: a `DockPanel` child docked at the bottom that was written before the content in the source got focus first; `KeyboardNavigation.TabIndex` changed nothing. The fix that worked: a `Grid` with rows, so source order equals reading order. Derived rule: write the children in reading order and avoid docking as a layout trick for anything that takes focus.

To get the numbered focus list the pull request needs (`accessibility.md`, section 3), let a headless test print it: press Tab, read the automation name of the focused element, and stop when an element repeats.

```csharp
static List<string> TabOrder(Window window, int max = 80)
{
    var seen = new List<Control>();
    var names = new List<string>();
    for (var i = 0; i < max; i++)
    {
        window.KeyPressQwerty(PhysicalKey.Tab, RawInputModifiers.None);
        if (window.FocusManager?.GetFocusedElement() is not Control focused || seen.Contains(focused)) break;   // wrapped around
        seen.Add(focused);
        names.Add(ControlAutomationPeer.CreatePeerForElement(focused).GetName());
    }
    return names;
}
// Assert.Equal(new[] { "Search", "Filter", "Add", "Row 1" }, TabOrder(window));
```

Use the peer name (not the control type) so the list is what a screen-reader user hears and a missing name shows up at once (section 9, Finding and fixing accessible names). A group that is a roving tab stop appears once. Run it per screen and per layout state (side panel open and closed).

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
| Expanded or collapsed state of a header | A custom peer that implements `IExpandCollapseProvider` (below) |

Avalonia derives accessible names from content, and the result is often wrong without any sign in the source or in a screenshot (observed, Avalonia 12.1.4; rules in `accessibility.md`, section 7). `IsRequiredForForm` is read from the attached property, not from the peer: use `AutomationProperties.GetIsRequiredForForm(field)` in tests.

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

### Expanded state on a toggle header

"The header is a button with an expanded state" (`data-display.md`, section 6) does not come with a `ToggleButton`: its stock peer reports the toggle state, not expanded or collapsed, so a screen reader hears "checked" instead of "expanded" (observed, Avalonia 12.1.4). Give the header its own peer that adds `IExpandCollapseProvider`, with the state taken from `IsChecked`. Sketch (derived, not verified; let the compiler list the members of the interface in your version, for example `ShowsMenu`):

```csharp
public sealed class AccordionHeader : ToggleButton
{
    public AccordionHeader() => Classes.Add("accordion");
    protected override Type StyleKeyOverride => typeof(ToggleButton);
    protected override AutomationPeer OnCreateAutomationPeer() => new AccordionHeaderAutomationPeer(this);
}

public sealed class AccordionHeaderAutomationPeer : ToggleButtonAutomationPeer, IExpandCollapseProvider
{
    private readonly ToggleButton _header;

    public AccordionHeaderAutomationPeer(ToggleButton owner) : base(owner) => _header = owner;

    public ExpandCollapseState ExpandCollapseState =>
        _header.IsChecked == true ? ExpandCollapseState.Expanded : ExpandCollapseState.Collapsed;
    public bool ShowsMenu => false;
    public void Expand() => _header.IsChecked = true;
    public void Collapse() => _header.IsChecked = false;
}
```

Keep the toggle provider of the base peer; both patterns stay available. Raising a property-changed event when `IsChecked` changes, so a running screen reader announces the new state, is (derived, not verified); check it with Narrator. Test: get the peer with `ControlAutomationPeer.CreatePeerForElement(header)`, check that it implements `IExpandCollapseProvider` (or ask for the provider with the lookup your version offers, `GetProvider<T>()` or `GetPattern`), assert `Collapsed` at rest, call `Expand()`, assert `Expanded` and `IsChecked`, call `Collapse()` and assert the reverse.

### Finding and fixing accessible names

Scan every page for elements whose name is empty or looks like a type name, with the same peer a screen reader reads (`using Avalonia.Automation.Peers;`). Run it for all pages in one test and fail with the whole list, so one run shows every problem.

```csharp
static IEnumerable<string> NameProblems(Visual page) =>
    page.GetSelfAndVisualDescendants().OfType<Control>()
        .Where(c => c.IsEffectivelyVisible && c.Focusable)
        .Select(c => (Control: c, Name: ControlAutomationPeer.CreatePeerForElement(c).GetName()))
        .Where(x => string.IsNullOrWhiteSpace(x.Name) || LooksLikeTypeName(x.Name, x.Control))
        .Select(x => $"{x.Control.GetType().Name} '{x.Control.Name}' has name '{x.Name}'");

static bool LooksLikeTypeName(string name, Control control) =>
    name == control.GetType().FullName                       // "Avalonia.Controls.StackPanel" on a button with non-text content
    || name == control.DataContext?.GetType().ToString()     // list rows without a name show the model type
    || (name.Contains('.') && !name.Contains(' '));          // heuristic for other qualified names

// [Fact] per page:  Assert.True(!NameProblems(window).Any(), string.Join(Environment.NewLine, NameProblems(window)));
```

Template parts of composite controls can add noise; fix real findings first and keep an explicit, reviewed ignore list for the rest. Fixes that worked:

1. Name from the first text of the content, for buttons whose content is a panel with an icon and a label. An attached property that sets the name once the content exists; an explicit `AutomationProperties.Name` always wins (derived, not verified beyond the observed case):

```csharp
public abstract class AccessibleName   // not static: attached-property owner (avalonia-12-notes.md, section 2)
{
    public static readonly AttachedProperty<bool> FromContentProperty =
        AvaloniaProperty.RegisterAttached<AccessibleName, Control, bool>("FromContent");
    public static void SetFromContent(Control c, bool value) => c.SetValue(FromContentProperty, value);
    public static bool GetFromContent(Control c) => c.GetValue(FromContentProperty);

    static AccessibleName() => FromContentProperty.Changed.AddClassHandler<Control>((control, e) =>
    {
        control.AttachedToVisualTree -= OnAttached;
        if (e.NewValue is true) control.AttachedToVisualTree += OnAttached;
    });

    private static void OnAttached(object? sender, VisualTreeAttachmentEventArgs e)
    {
        if (sender is not Control control || !string.IsNullOrEmpty(AutomationProperties.GetName(control))) return;
        Dispatcher.UIThread.Post(() =>   // after the content is realized
        {
            var text = control.GetVisualDescendants().OfType<TextBlock>()
                .Select(t => t.Text).FirstOrDefault(t => !string.IsNullOrWhiteSpace(t));
            if (text is not null) AutomationProperties.SetName(control, text);
        });
    }
}
```

2. List rows: a style setter on the row container takes the name from the row's ViewModel. The compiled binding in a setter needs `x:DataType` on the `Style` (`avalonia-12-notes.md`, section 2):

```xml
<Style Selector="ListBox.messages ListBoxItem" x:DataType="vm:MessageRow">
  <Setter Property="AutomationProperties.Name" Value="{Binding Subject}"/>
</Style>
```

3. Help and icon-only buttons: name the topic ("Help on delivery date"), not only "Help", because a screen reader's element list shows ten identical "Help" entries. Use the same text as the tooltip, from one localizable resource:

```xml
<Button Classes="help" AutomationProperties.Name="{Binding HelpName}" ToolTip.Tip="{Binding HelpName}"/>
```

4. Unlabeled fields: `AutomationProperties.LabeledBy` pointing at the visible label, or a `Name` where no label exists.

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
| Tab-order walk | Press Tab repeatedly and compare the sequence of focused element names with the numbered focus list (helper in section 8, Tab order walk). |
| F6 regions | Press F6 and Shift+F6 repeatedly; assert the region names in order, the wrap-around and the fallback page (section 8, F6 regions). |
| Name scan | Open every page; assert that no focusable element has an empty or type-like automation name (section 9, Finding and fixing accessible names). |
| Esc ladder | Edit a field, press Esc, field shows the old value; Esc again raises the close command. |
| Enter and default button | Enter in a single-line field invokes the default command; not in a multi-line field. |
| Parser table | `[Theory]` with the rows of the parser tables, plus invalid and empty input. |
| Error replaces help | Setting an error hides the help text and shows the message with its icon. |
| Focus restore | Open a popup or dialog from a button, close it, assert the button has focus; close a panel with a mouse click, press an arrow key, assert the view handled it. |
| Reduced motion | Switch on, open every page, assert that no visible element has a non-zero transition (helper in Reading transitions below; override pattern in section 5). |
| Tooltip contrast | Open every real tooltip of the page (set the tip, show it, find its text in the overlay), resolve the text and surface colors and assert 4.5:1 in light and dark; a descendant selector of the target can change the text color (section 3, Precedence). |
| Contrast | Resolve brushes from both theme variants (`Application.Current.RequestedThemeVariant`), compute WCAG ratios, assert 4.5 and 3.0 thresholds per state. |
| Screenshots | `window.CaptureRenderedFrame()` after a short wait for animations (at least 200 ms for fades; advance the render timer or delay), once per state per theme. |
| Style applies | Create the control in a window with the app styles and assert a value that only the new style sets (section 3, Precedence, Style key). |
| Scroll container | `BringIntoView()` on the card, wait one frame, then click or capture (below, Scrolling containers). |
| XAML-built field | Load a mini view that declares the field in markup and assert the behavior, in more than one attribute order (below, Instances built from XAML). |

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

### Reading transitions

`ITransition` exposes neither the duration nor the animated property in Avalonia 12 (observed, Avalonia 12.1.4), so a test cannot read them through the interface. Read them from the concrete type, by reflection (works for every transition class) or typed where the app uses only a few classes:

```csharp
static IEnumerable<(string Property, TimeSpan Duration)> ReadTransitions(Animatable element)
{
    foreach (var transition in element.Transitions ?? new Transitions())
    {
        var type = transition.GetType();
        var duration = type.GetProperty("Duration")?.GetValue(transition) as TimeSpan? ?? TimeSpan.Zero;
        var property = type.GetProperty("Property")?.GetValue(transition) as AvaloniaProperty;
        yield return (property?.Name ?? type.Name, duration);
    }
}
// Typed alternative (derived, not verified): if (transition is Transition<double> t) use t.Duration and t.Property.

static IEnumerable<string> MotionLeaks(Visual root) =>
    root.GetSelfAndVisualDescendants().OfType<Animatable>()
        .Where(e => e is not Visual v || v.IsEffectivelyVisible)   // invisible template parts are not motion the user sees
        .SelectMany(e => ReadTransitions(e).Where(t => t.Duration > TimeSpan.Zero)
            .Select(t => $"{e.GetType().Name}: {t.Property} {t.Duration.TotalMilliseconds} ms"));

// per page, with ReducedMotion.Overrides() applied to the Application:  Assert.Empty(MotionLeaks(window));
```

The reflection read is also a quick way to check the other direction: with the switch off, the same walk should find the token durations (100 ms, 200 ms) and not zero.

### What headless does not show

- Tooltips: in a headless test a tooltip appears in the window's overlay layer instead of a popup window (observed, Avalonia 12.1.4). You can assert that it opens, its content, the delay and its placement relative to the target; shadow, border and behavior at a screen edge cannot be verified. Look at those in the running app and say so in the report. Flyouts are observed to behave the same way (next subsection); menus and combo lists may (derived, not verified).
- A `MinWidth` on the window clamps the width a test asks for (observed): a responsive test at 600 px inside a window with `MinWidth` 800 measures 800 px. Lift the minimum inside the test (`window.MinWidth = 0`) or measure the content directly (`content.Measure(new Size(600, 800))`).

### Flyouts, keys and commands

Observed in Avalonia 12.1.4 headless tests; the way to reach the content is Derived from that, so confirm it in your version.

- A flyout's content is not a child of the control that owns it, because the flyout lives in a popup. Open it from the test with `flyout.ShowAt(control)` and look for the content from the window: `window.GetVisualDescendants()` reaches it, and a search from the owning control does not. Search by name, type or automation name, and assert that the search found exactly the expected element.
- Send keys to the element that should handle them: `control.RaiseEvent(new KeyEventArgs { RoutedEvent = InputElement.KeyDownEvent, Key = Key.Down })`. Window-level helpers such as `KeyPressQwerty` go to the focused element of the window, which is not inside a popup unless focus was moved there.
- `button.RaiseEvent(new RoutedEventArgs(Button.ClickEvent))` raises the event but does not run the button's `Command`. To test the command, execute it directly (`button.Command!.Execute(button.CommandParameter)`), or click with the headless mouse (`MouseDown` and `MouseUp` at the center of the button) so the control runs its own click path.

### Scrolling containers

Cards, rows and panels inside a scrolling container are laid out at their full position, so one below the visible edge is clipped: a headless click at its coordinates hits nothing, and a captured frame shows an empty area (observed, Avalonia 12.1.4). Bring the element into view first, then wait one frame so layout and render catch up, and only then click or capture:

```csharp
card.BringIntoView();
Dispatcher.UIThread.RunJobs();                     // run the pending layout (call names: check your version)
await Task.Delay(50);                              // one frame; or advance the render timer of the headless platform
var center = card.TranslatePoint(new Point(card.Bounds.Width / 2, card.Bounds.Height / 2), window)!.Value;
window.MouseDown(center, MouseButton.Left);        // take the point after BringIntoView, not before
```

Compute the click point after `BringIntoView`, because the scroll offset changed it. For a screenshot of the whole list, give the window a height that fits the content instead of scrolling, or capture one frame per scroll position.

### Instances built from XAML

A composite field can pass every test with instances built in code and still not work in the app (observed, Avalonia 12.1.4). Seen: a character counter next to a multi-line field never ran from XAML. The XAML loader assigns the properties one after another in markup order; the `Text` arrived while `AcceptsReturn` was still `false`, the counter logic read that state once and never again. In code the test set the properties in a different order and was green.

- Test behavior of composite fields (counters, derived text, enabled states, anything that reads two properties) with instances created from XAML, not only with `new` and property assignments. Because the assignment order and timing of the loader is part of the behavior.
- Use a mini view: a small `UserControl` in the test project, declared in AXAML like production code, that holds the field with the attributes the app uses. Compiled XAML takes the same path as the app. A runtime-loaded string (`AvaloniaRuntimeXamlLoader.Load`) is a shortcut whose assignment order is (derived, not verified) the same.
- Repeat the test with the attributes in a second order (`AcceptsReturn` before `Text`, and after): the field must behave the same either way.
- Fix in the control, not in the markup order. Recompute dependent state in `OnPropertyChanged` for every property it reads, or wire it in `OnApplyTemplate` or on attach (`OnLoaded`) after all properties exist:

```csharp
protected override void OnPropertyChanged(AvaloniaPropertyChangedEventArgs change)
{
    base.OnPropertyChanged(change);
    if (change.Property == TextProperty || change.Property == AcceptsReturnProperty || change.Property == MaxLengthProperty)
        UpdateCounter();   // reads Text, AcceptsReturn and MaxLength, so any of them can trigger it
}
```

```xml
<!-- test project: CounterFieldHost.axaml -->
<UserControl xmlns="https://github.com/avaloniaui" xmlns:local="using:Tests.Controls">
  <local:CounterTextBox x:Name="Field" Text="Hello" AcceptsReturn="True" MaxLength="200"/>
</UserControl>
```

## 11. Checklist

- [ ] No hex colors, named colors, inline `BoxShadow` values or magic numbers in view XAML; all values come from resources (the scans in section 2); shadows are `BoxShadows` resources per theme.
- [ ] Theme-dependent resources use `DynamicResource` (grep for `StaticResource` on brushes).
- [ ] State styling uses selectors from section 3, with no local brush values on elements (grep for `Background="` in views).
- [ ] The field template has the bottom edge and focus underline (screenshot of rest and focus, both themes).
- [ ] Transitions use the token durations and curves; the reduced-motion override is an activator style on the `Application` and the test finds no visible non-zero transition on any page (section 5).
- [ ] Behaviors are attached properties or control subclasses with headless tests.
- [ ] Every style that targets a template part was checked with a probe test; values the template sets itself are overridden in `TemplateApplied` or through the theme resource (section 3).
- [ ] A property that does not change was traced to its source (template, pseudo-class rule, local value) and changed there (section 3, Precedence).
- [ ] A custom template contains the required parts of its control (section 4, Required template parts; the compiler or a probe test confirms it).
- [ ] Names and behavior that changed between Avalonia 11 and 12 match the project's version, and no attached-property owner is a static class ([avalonia-12-notes.md](avalonia-12-notes.md)).
- [ ] Parsers are pure classes with table tests; no `double` in the money path.
- [ ] Groups are one tab stop and the stop is the selected item; arrows do not wrap (headless test; section 8).
- [ ] F6 and Shift+F6 cycle regions in screen order, each with a name and a focus target; a page without regions is one region (test; section 8, F6 regions).
- [ ] The Tab walk of each screen matches the numbered focus list, and nothing docked out of source order takes focus first (test; section 8, Tab order walk).
- [ ] An overflow panel hides items completely (disabled, transparent, 0 x 0) and Tab visits only visible items (headless test at minimum and wide width).
- [ ] Selectors for text inside a target use the child combinator `>`, so they do not reach the tooltip text; every real tooltip passes the contrast test (section 3, Precedence; section 10).
- [ ] Equal-width star columns have a finite width from their frame; columns shared across lists use `SharedSizeGroup` inside a `Grid.IsSharedSizeScope`; row heights taken from `DesiredSize` exclude the margin (section 4, Layout notes).
- [ ] Flyout tests open with `ShowAt`, search from the window, send keys with `RaiseEvent` on the handling element and run commands directly or with the headless mouse (section 10).
- [ ] Tooltip delay and placement come from one app-wide default, not from `OverrideDefaultValue` (section 6).
- [ ] Automation names, help, required and live-region settings present on custom controls (test or inspector).
- [ ] The name scan finds no empty or type-like names on any page; list rows, icon buttons and help buttons have names that identify their target (section 9).
- [ ] A control that overrides `StyleKeyOverride` is styled through its base type plus a class, and a test proves that the style applies (section 3, Precedence).
- [ ] Tests of style-set transforms read the matrix (`RenderTransform.Value`), not a cast to `RotateTransform` (section 5, Transforms from styles).
- [ ] An expandable header has a peer with `IExpandCollapseProvider`; a test gets the peer and checks the state and `Expand()` and `Collapse()` (section 9).
- [ ] Target size measured: small icon buttons have `MinHeight` and `MinWidth` next to `Width` and `Height`, and a layout test compares `Bounds` with the design size (section 4, Layout notes).
- [ ] Click and image tests on cards inside a scrolling panel call `BringIntoView()` and wait one frame first (section 10, Scrolling containers).
- [ ] Composite fields are also tested with instances created from XAML, in more than one attribute order (section 10, Instances built from XAML).
- [ ] Every selector and property used in this file was checked against the project's Avalonia version.

## 12. Sources

Avalonia documentation and source are the authority for selectors, properties and the headless API: [Avalonia documentation][avalonia-docs] and [Avalonia source][avalonia-src]. The Fluent and WinUI rules these patterns implement are cited in the other reference files.

[avalonia-docs]: https://docs.avaloniaui.net/
[avalonia-src]: https://github.com/AvaloniaUI/Avalonia
