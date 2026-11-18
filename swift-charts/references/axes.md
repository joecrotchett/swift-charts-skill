# Axes

An axis explains a position scale with grid lines, ticks and labels. Axes are generated from the data, and the default is a good place to start: change an axis only where it would communicate better. The range an axis covers belongs to its scale (`scales.md`); this file covers which values the axis marks and how each is drawn.

## What an axis is made of

| Level | Type | Says |
|---|---|---|
| The axis | `chartXAxis { }`, `chartYAxis { }` | What this axis contains: one or more `AxisMarks` |
| A set of positions | `AxisMarks` | Which values to mark, on which edge, in which preset style |
| What is drawn at each position | `AxisGridLine`, `AxisTick`, `AxisValueLabel` | The line across the plot, the short mark at its edge, the text |

Customise at the shallowest level that does the job: a parameter of `AxisMarks` before a content closure, one `AxisMarks` before several.

## Axes

### `chartXAxis` - iOS 16.0+

**Use for:** hiding or replacing the x axis. `chartYAxis` is the same for y.

- `chartXAxis(.hidden)` removes the axis: labels, ticks and grid lines. On horizontal bars the lines between the rows belong to the y axis and go with it. The argument is a SwiftUI `Visibility`, so a toggle passes `.hidden` or `.automatic` to one modifier rather than adding and removing it.
- `chartXAxis { AxisMarks(...) }` replaces the default axis with your own.

The closure can hold several `AxisMarks`, and `if`, `if`-`else`, `switch` and `if #available`. Put a condition here, not in a conditional view modifier around the chart, which gives SwiftUI two different charts.

### `chartXAxisLabel` - iOS 16.0+

**Use for:** a title for the axis, naming what it measures and its unit: `.chartXAxisLabel("Energy density (watts per m²)")`. `chartYAxisLabel` is the same for y. It is separate from the labels on the axis marks.

By default the x title sits below the axis at the leading edge and the y title above the axis at the trailing edge. `position:` (an `AnnotationPosition`) and `alignment:` move it, and a closure form takes any view. A y title at `.leading` or `.trailing` turns to run along the plot's side; `alignment:` places a title against the plot area's edges. A title is never wrapped or shortened: a long one runs past the chart's frame at large text sizes, even in a closure with `lineLimit`, so keep titles short or use a shorter one at large sizes.

<!-- snippet: Dashboard/DashboardPatterns.swift#axis-titles -->
```swift
struct TitledAxes: View {
    let beans: [Bean]

    var body: some View {
        Chart(beans) { bean in
            PointMark(x: .value("Acidity", bean.acidity), y: .value("Body", bean.body))
        }
        .chartXAxisLabel("Acidity (pH points below neutral)")
        .chartYAxisLabel("Body (score out of 10)")
    }
}
```

Stating the unit here keeps the labels on the axis marks short (`design.md`, *Axes*).

### `AxisMarks` - iOS 16.0+

**Use for:** saying which values an axis marks and how. Every parameter is optional, and each one left out keeps its default. They are listed in the order the initializer takes them.

| Parameter | Chooses | Type |
|---|---|---|
| `format:` | How default labels are formatted | A `FormatStyle` |
| `preset:` | Where labels sit relative to their grid lines | `AxisMarkPreset` |
| `position:` | The edge of the chart the marks sit on | `AxisMarkPosition` |
| `values:` | The values that get a mark | `AxisMarkValues`, or an array of values |
| `stroke:` | The stroke of default grid lines and ticks | `StrokeStyle` |
| A trailing closure | What is drawn at each value, replacing the defaults | Axis marks, below |

`AxisMarks()` with no arguments recreates the default axis, so start from it and add one parameter at a time. A default category axis draws grid lines and labels but no ticks; a number or date axis draws all three. Without a closure, `AxisMarks` draws the default grid line, tick and label at each value. That is enough to change the count, the format or the side:

<!-- snippet: Axes/AxisRecipes.swift#format -->
```swift
func percentAxis() -> some View {
    chartYAxis {
        AxisMarks(format: Decimal.FormatStyle.Percent.percent.scale(1), values: [0, 50, 100])
    }
}
```

The `values:` there assume a scale from 0 to 100. Leave them out when the scale can pass either end, as normalized stacks of negative values do, or nothing is labelled beyond them.

With a closure, only what the closure lists is drawn. Use two `AxisMarks` to put different parts at different values, such as grid lines at every quarter but labels at every half:

<!-- snippet: Axes/AxisRecipes.swift#compose -->
```swift
func percentAxisWithMinorGridLines() -> some View {
    chartYAxis {
        AxisMarks(values: [0, 50, 100]) {
            AxisValueLabel(format: Decimal.FormatStyle.Percent.percent.scale(1))
        }
        AxisMarks(values: [0, 25, 50, 75, 100]) {
            AxisGridLine()
        }
    }
}
```

The closure can take the `AxisValue` it is drawing, to decide per value what to show (see `AxisValue`).

**Pitfalls:**

- A closure with only `AxisValueLabel` loses the grid lines and ticks. List every part the design shows.
- The values of an `AxisMarks` are added to the scale's automatic domain, whether they are an array, a stride or an automatic count. `AxisMarks(values: [0, 50, 100])` on data that peaks at 40 stretches the axis to 100; on data that peaks at 164, `.stride(by: 25)` ends the axis at 175 and `.stride(by: 50)` at 200. Usually that is wanted; set the scale's domain when it is not.

### `AxisMarkValues` - iOS 16.0+

**Use for:** the `values:` of `AxisMarks` when not listing them in an array.

| Value | Marks |
|---|---|
| `.automatic(desiredCount: 3)` | About that many values. The framework may use a different number to keep the values round: asking for 4 on a 0 to 100 axis gives 5 |
| `.automatic(minimumStride: 5, desiredCount: 4)` | As above, with a minimum stride |
| `.stride(by: 25)` | Every multiple of a number |
| `.stride(by: .hour, count: 3)` | Every third hour; any `Calendar.Component` (`.day`, `.month`, ...) |
| `[0, 50, 100]` | Exactly these values. Works for numbers, dates and category strings |

Each also takes `roundLowerBound:` and `roundUpperBound:`.

### `AxisMarkPosition` - iOS 16.0+

**Use for:** the edge an axis sits on: `.leading` or `.trailing` for a y axis, `.top` or `.bottom` for an x axis. The default puts a y axis on the trailing edge and an x axis at the bottom.

<!-- snippet: Axes/AxisRecipes.swift#leading -->
```swift
struct LeadingValueAxis: View {
    let sales: [DrinkSales]

    var body: some View {
        Chart(sales) { item in
            BarMark(x: .value("Drink", item.drink), y: .value("Cups", item.cups))
        }
        .chartYAxis {
            AxisMarks(position: .leading)
        }
    }
}
```

### `AxisMarkPreset` - iOS 16.0+

**Use for:** where labels sit relative to the plot area and their grid lines.

| Preset | Labels | Looks like |
|---|---|---|
| `.aligned` | Outside the plot area, centred on their grid line | The default number y axis |
| `.extended` | Outside the plot area, beside a grid line that is extended out to them | The default number and category x axis; a leading value axis whose labels should line up with the rest of the interface |
| `.inset` | Inside the plot area, above their grid line | The default for category names on a y axis, drawn above each row. On a trailing y axis the names are drawn past the plot's trailing edge, outside the chart's frame; on a category x axis the bars are narrowed to make room and cover the names |
| `.automatic` | Picks one of the above from the axis and the data | |

Use a preset to move category names out of the plot on a horizontal bar chart, timeline or heat map, where the default `.inset` labels take space from the marks, and on a dot plot or grid of symbols, where marks at the leading edge cover them:

<!-- snippet: Axes/AxisRecipes.swift#category-rows -->
```swift
struct RowLabelsOutside: View {
    let shares: [DrinkSales]

    var body: some View {
        Chart(shares) { share in
            BarMark(x: .value("Cups", share.cups), y: .value("Drink", share.drink))
        }
        .chartYAxis {
            AxisMarks(preset: .aligned, position: .leading)
        }
    }
}
```

**Pitfalls:** `.inset` on a number axis draws the labels over the marks.

## Axis marks

The three things drawn at each value, used inside an `AxisMarks` closure. Each takes `foregroundStyle(_:)`, `font(_:)` and `offset(x:y:)`.

### `AxisGridLine` - iOS 16.0+

**Use for:** the line across the plot at a value. `AxisGridLine(stroke: StrokeStyle(lineWidth: 0.5))` sets width and dash; `centered:` moves it to the middle of a band or unit. Colour it with `foregroundStyle`.

### `AxisTick` - iOS 16.0+

**Use for:** the short mark at the edge of the plot. `AxisTick(length: 8)` takes a length in points and `stroke:` as for grid lines. A longer tick does not move its label, so it runs into the text; on a y axis give the label `horizontalSpacing:` a few points more than the tick's length (12 for an 8-point tick). `length:` also accepts `.label` and `.longestLabel`, which run the tick out to the label, each with an `extendPastBy:` form.

### `AxisValueLabel` - iOS 16.0+

**Use for:** the text at a value.

| Form | Shows |
|---|---|
| `AxisValueLabel()` | The value, formatted automatically |
| `AxisValueLabel(format: .dateTime.weekday(.narrow))` | The value through a `FormatStyle` |
| `AxisValueLabel("Goal")` | Fixed text |
| `AxisValueLabel { VStack { ... } }` | Any view, such as two lines of text |

The automatic format switches to scientific notation ("8.0E6", "2.0E7") once an axis reaches several million. Give such an axis a format. Compact notation ("20M") also saves width in a narrow chart:

<!-- snippet: Design/HIGPatterns.swift#compact-labels -->
```swift
func shortValueLabels() -> some View {
    chartYAxis {
        AxisMarks(format: IntegerFormatStyle<Int>.number.notation(.compactName))
    }
}
```

`AxisMarks(format:)` takes any `FormatStyle` for the axis's value type, dates included: `AxisMarks(format: .dateTime.day(), values: .stride(by: .day, count: 7))`.

Parameters that matter:

- `centered:` puts the label in the middle of its band or unit instead of at its start. It defaults to true for categories and false for numbers and dates. Set it to true on a date axis whose marks are plotted with a matching `unit:` and labelled at every unit, so each label sits under its bar. It centres the label between two neighbouring axis marks, not within the unit: with a label every third day the label sits under the middle of the three, so leave it false whenever the stride is longer than the unit.
- `orientation:` is `.horizontal`, `.vertical` (reading top to bottom) or `.verticalReversed`.
- `collisionResolution:` decides what happens when labels do not fit on the default axis. An `AxisValueLabel` listed in an `AxisMarks` closure does not truncate, even with `.truncate` set: long names overlap, so turn them vertical or use fewer. The options: `.truncate` shortens them (with an ellipsis while there is room for one; on a narrow band only a letter or two is left), `.greedy` keeps the first label and then each next one that clears the last one drawn, `.disabled` draws them all even if they overlap, and the last can run past the plot's trailing edge. Vertical labels take their height from the plot, so the bars lose height to the longest name; they also log a harmless runtime message about `anchor:` values. `.greedy(priority:minimumSpacing:)` lets some labels win over others.
- `anchor:`, `multiLabelAlignment:`, `horizontalSpacing:` and `verticalSpacing:` adjust placement.
- A number or date label on a horizontal axis sits after its grid line, so the label of a grid line at the plot's trailing edge has no room: it is left out, or runs past the chart's frame, and the axis ends on an unlabelled line. `AxisMarks(preset: .aligned)` centres each label on its line and so labels both ends; give the scale's range a little padding so the end labels stay inside the chart's frame.

For date axes, format with a `Date.FormatStyle`: `.dateTime.month(.narrow)` for month initials, `.dateTime.weekday(.narrow)` for day initials, `.dateTime.month().day()` for "Mar 4".

<!-- snippet: Axes/AxisRecipes.swift#centered -->
```swift
struct WeekdayAxis: View {
    let days: [DailySales]

    var body: some View {
        Chart(days) { day in
            BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .day)) { _ in
                AxisGridLine()
                AxisValueLabel(format: .dateTime.weekday(.narrow), centered: true)
            }
        }
    }
}
```

**Pitfalls:**

- Category labels that do not fit are truncated to a few letters and an ellipsis, not rotated. Turn them with `orientation: .vertical`, shorten the names, or switch to horizontal bars.
- Centred labels on marks plotted without a unit do not line up with the marks.

### `AxisValue` - iOS 16.0+

**Use for:** deciding what to draw from the value being marked. It is the parameter of the `AxisMarks` closure.

- `value.as(Date.self)` returns the value as a type, or `nil` if it is not one. Unwrap it with `if let`.
- `value.index` is the position of this value along the axis and `value.count` the number of values, for treating the first or last differently.

The worked example in `gallery/custom-axes.md` uses both: an axis that marks every third hour, shows AM or PM only where it changes, puts the date under the first label and at each midnight, and draws the midnight grid line solid.

**Pitfalls:**

- A test such as `hour == 0` only matches if the stride lands on that value. With a stride of three hours the data must start at an hour divisible by three.
- A label gets only the space between two marks. Two-line labels every three hours are truncated at phone width; use a longer stride there.

## Styling axis marks and whole axes

Style individual parts with the modifiers on axis marks:

<!-- snippet: Axes/AxisRecipes.swift#styled -->
```swift
struct StyledAxis: View {
    let sales: [DrinkSales]

    var body: some View {
        Chart(sales) { item in
            BarMark(x: .value("Drink", item.drink), y: .value("Cups", item.cups))
        }
        .chartXAxis {
            AxisMarks { _ in
                AxisValueLabel(orientation: .vertical)
                    .font(.caption.bold())
                    .foregroundStyle(Color.primary)
            }
        }
        .chartYAxis {
            AxisMarks { _ in
                AxisGridLine(stroke: StrokeStyle(lineWidth: 1, dash: [2, 4]))
                    .foregroundStyle(.orange)
                AxisTick(length: 8)
                // A longer tick does not move the label: space it past the tick.
                AxisValueLabel(horizontalSpacing: 12)
            }
        }
    }
}
```

`AnyAxisMark` and `AnyAxisContent` are type erasers, for a function that returns different axis marks or axis content from different branches. Axis content that both axes share goes in an `@AxisContentBuilder` property returning `some AxisContent`, as `@ChartContentBuilder` does for marks; one property serves a category axis and a number axis.

From iOS 17.0, `chartXAxisStyle` and `chartYAxisStyle` receive the whole axis as a view (`ChartAxisContent`). That view includes the axis's grid lines, so a background applied to it covers the plot area.

Worked example with images: `gallery/custom-axes.md`.

## Pitfalls

- Customising an axis that was fine. Start from the default and change one thing at a time.
