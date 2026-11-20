# Composing chart content

How a chart is put together: the framework's model, what it does for you, the `Chart` initializers, and how marks combine into series, layers, stacks and groups. Read it first for any chart.

## The model

A chart is built from four kinds of building block: **marks** (the shapes that represent data), **scales** (how data values become positions, colours and sizes), **axes** and **legends** (which explain the scales). You write the marks. The framework generates scales, axes and a legend that fit the data, and you override them with chart modifiers only where the design differs from the default.

A mark represents data through its **mark properties**: x and y, and the properties with a `by:` encoding, foreground style, symbol, symbol size and line style. Plotting a data value with a property (`x: .value(...)`, `foregroundStyle(by: .value(...))`) creates the scale for it. There is no prebuilt component per chart type. Changing the form is usually changing the mark: `BarMark` to `LineMark` to `PointMark`, with everything else left alone.

The plot area, where the marks are drawn, can be customised like the axes and the legend (`axes.md`, `legend.md`, `plot-area.md`).

## What Swift Charts handles for you

A chart made with the framework's own marks, scales and axes gets these without any code:

- Light and Dark Mode, and Increase Contrast.
- Dynamic Type for its labels, and a layout that adapts to the space and the device.
- Numbers and dates on its default axes formatted for the locale.
- Right-to-left layout: the x axis runs from the right, the value axis and its title move to the left, the legend moves to the right, and leading and trailing annotation positions swap. It follows `\.layoutDirection`, not the locale: an Arabic locale alone changes digits and month names and mirrors nothing.
- A starting point for VoiceOver: an element per position, and for most charts an Audio Graph. It is not enough on its own (`accessibility.md`).
- Animation when its data changes inside a SwiftUI animation.
- The same 2D code on every Apple platform.

Do not rebuild any of these by hand, and do not defeat them: hard-coded colours that only work in one appearance, fixed font sizes on labels, hand-formatted number strings, and custom drawing in place of marks each give one up.

**The locale.** An axis given an explicit `format:` uses that style's own locale, the device's, and ignores a `\.locale` set on the view (a preview, a test, one view shown in another language); give the style the environment's locale there (`data.md`, the calendar example). The same goes for the words the app writes about the chart: read `\.locale` in the view and pass it to whatever builds the headline, the element labels and values, and the chart descriptor, so a number is written one way everywhere.

**Animating a change.** A chart animates like any SwiftUI view. When a control switches the values shown for the same categories, animate it:

<!-- snippet: Sessions/SessionPatterns.swift#animated-switch -->
```swift
/// The same categories with different values: animate the change.
struct StoreSwitcher: View {
    let stores: [StoreSales]
    @State private var selected = 0

    var body: some View {
        VStack {
            Picker("Store", selection: $selected.animation(.easeInOut)) {
                ForEach(stores.indices, id: \.self) { index in
                    Text(stores[index].store).tag(index)
                }
            }
            .pickerStyle(.segmented)
            Chart(stores[selected].days) { day in
                BarMark(x: .value("Day", day.day, unit: .weekday), y: .value("Cups", day.cups))
            }
        }
    }
}
```

When the switch changes what the marks are (days to months), do not animate one into the other: bars for days and bars for months are different things. Remove the animation for that change with `.transaction { $0.animation = nil }` on the container.

## Chart content and builders

### `Chart` - iOS 16.0+

**Use for:** the view that displays a chart. Its content is marks.

Three initializers. Choose by what the chart's content is made of:

| Initializer | Use when |
|---|---|
| `Chart(data) { element in ... }` | One collection, one mark (or group of marks) per element, and the elements are `Identifiable`. |
| `Chart(data, id: \.keyPath) { element in ... }` | The same, when the elements are not `Identifiable`. The key path must be `Hashable` and unique per element; when no one property is unique and the element is `Hashable`, use `\.self`. |
| `Chart { ... }` | Anything else: several collections, a collection of series each holding its own points, or individual marks such as a threshold line alongside data. |

The first two wrap the data in an implicit `ForEach`. `Chart { }` takes any mix of individual marks and `ForEach` constructs. Several series do not by themselves rule out `Chart(data)`: one flat collection with a row per series and position, and a `by:` encoding to tell the series apart, is still one mark per element. It is nested data (a list of series, each with its points) that needs `Chart { }` and a `ForEach` inside a `ForEach`.

Examples: `gallery/bar.md` (data initializer), `gallery/multi-series-line.md` (content initializer).

**Pitfalls:** reaching for `Chart(data)` and then needing a second series or a `RuleMark` beside it. A rule written inside that closure is created once per element: the copies draw as one line, heavier than intended when its colour is translucent (rendered: four copies of a 30% red rule drew solid red), and each copy is an accessibility element: four drinks gave four identical rule elements, one when the rule was written once (read on iOS). Start with `Chart { ForEach(data) { ... } }` when the design has anything other than one mark per element.

### `ChartContent` - iOS 16.0+

**Use for:** the protocol every mark conforms to, and the type to write when a piece of a chart is worth naming or reusing.

Anything that goes in a chart is chart content: marks, vectorized plots, `ForEach`, `Plot`. Content can be listed individually, generated with `ForEach`, or mixed. All of it shares the modifiers in `mark-modifiers.md`.

Make your own by conforming a struct and returning marks from `body`, exactly as a SwiftUI view returns views. Use it to give a recurring element one definition:

<!-- snippet: Marks/CustomChartContent.swift#custom -->
```swift
/// A reusable piece of chart content: a labelled target line.
struct TargetLine: ChartContent {
    let title: String
    let value: Int

    var body: some ChartContent {
        RuleMark(y: .value(title, value))
            .foregroundStyle(.red)
            .accessibilityLabel(title)
            .annotation(position: .top, alignment: .trailing) {
                Text(title).font(.caption).foregroundStyle(.red)
            }
    }
}
```

A custom type takes the same modifiers as a built-in mark, and they reach the marks inside it, with one exception worth designing for: **a style set inside the type wins over the same style applied from outside.** `TargetLine(...).foregroundStyle(.green)` stays red, a `foregroundStyle(by:)` from outside no longer colours the marks either (its legend can remain, describing colours that are not drawn), and a `lineStyle` set inside is kept the same way. A style from outside does not reach the views in an annotation inside the type; style those where they are written. Set inside only what must never change, and leave the rest to the caller. To make a custom mark as general as the framework's, give it `PlottableValue` properties instead of concrete types (`gallery/candlestick.md`).

`AnyChartContent` erases the type when different kinds of content must be stored or returned as one type.

### `ChartContentBuilder` - iOS 16.0+

**Use for:** the result builder behind a chart's content closure. Mark a property or function `@ChartContentBuilder` to factor part of a chart out of `body` while keeping the same syntax, including conditionals and `if #available`:

<!-- snippet: Marks/CustomChartContent.swift#builder -->
```swift
@ChartContentBuilder
private var salesLine: some ChartContent {
    ForEach(days) { item in
        LineMark(x: .value("Day", item.day), y: .value("Cups", item.cups))
    }
}

var body: some View {
    Chart {
        salesLine
        TargetLine(title: "Target", value: 400)
    }
}
```

**Prefer a `ChartContent` struct** when the piece takes parameters or is used in more than one chart; a builder property when it only tidies one chart's body.

### Chart content supports ordinary control flow

Inside a chart's content you can use `ForEach`, `if` and `if let` with `else if` and `else`, `switch`, `if #available`, and local `let` constants. Use `if let` to add marks only while some state exists, such as a selection indicator (`gallery/selection-line.md`), and `if` for optional overlays such as an average line (`gallery/range-bar.md`).

## Layering marks

Several marks in one chart share its scales and axes, so they line up automatically. Compose a chart form that no single mark provides by layering: a faint range bar, a tick on top of it, and a rule across everything (`gallery/range-bar.md`).

Marks that should align on a band must use the same plottable value on that axis, including the same `unit:` for dates, and the same width.

To put one mark behind the others regardless of where it is written, use `zIndex` (`mark-modifiers.md`).

## Series

A series is a set of marks that belong together, such as one line. There are two ways to say which series a mark belongs to:

- **The `series:` parameter** on `LineMark` and `AreaMark`. Use it when the series should be told apart but styled by hand, with a fixed colour each.
- **A `by:` style encoding**: `foregroundStyle(by:)`, `symbol(by:)` or `lineStyle(by:)`. Use it when the series name should also drive colour, symbol or dash pattern and appear in the legend.

Marks with the same `series:` value always form one line; without `series:`, marks that share the same values for those encodings do. With neither, every `LineMark` in the chart joins into a single line, and a stray diagonal runs from the end of one series to the start of the next. See `gallery/multi-series-line.md`.

## Applying a modifier to a group

`ForEach` is itself chart content, so a mark modifier applied to a `ForEach` applies to every mark it produces. Put shared styling there instead of repeating it on each mark: one `foregroundStyle(by:)` on the inner `ForEach` of a nested pair styles a whole series.

### `Plot` - iOS 16.0+

**Use for:** grouping several marks into one piece of chart content, so a modifier or a single annotation applies to the group as a whole.

**Prefer over** `ForEach` when the marks are written out individually, not generated from a collection.

A `Plot` groups without separating: `LineMark`s in different plots still join into one line when they are one series, and a modifier applied outside the plots, such as `foregroundStyle(by:)` on the enclosing `ForEach`, still reaches the marks inside them. Each plot is drawn after the ones before it, so a bar in a later plot can cover the line's segment from an earlier one. On iOS `.zIndex(1)` on the `LineMark` (iOS 17) brings the whole line in front; in a macOS render it did not, so check there.

An annotation on a `Plot` is positioned relative to the whole group, and its `AnnotationContext.targetSize` is the group's size. That is how one label spans two rules in a range selection (`gallery/selection-line.md`).

## Stacking and grouping

Bars and areas that share a position stack on top of each other, in the order their elements appear in the data, whether or not they are coloured. `foregroundStyle(by:)` does not cause the stacking; it colours the pieces so they can be seen. Uncoloured, several rows for one category draw as a single bar of their total divided by hairlines: if a bar is taller than any one value, aggregate the data or encode what distinguishes the rows. `MarkStackingMethod` (`mark-configuration.md`) changes how they stack; `position(by:)` (`mark-modifiers.md`) places bars side by side instead. See `gallery/stacked-bar.md`, `gallery/grouped-bar.md` and `gallery/area.md`.
