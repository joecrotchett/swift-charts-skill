# Accessibility

Making a chart usable with VoiceOver, Audio Graphs and other assistive technologies. Every chart in an app has to be accessible, however people perceive it. A chart is not an image: one label is rarely enough. The rule everything here follows from: for each thing a chart shows visually, decide how it is represented non-visually.

## What Swift Charts provides automatically

The full account, with the wording for each kind of chart, is in `accessibility-defaults.md`. What matters for every chart:

- **One accessibility element per position, not per mark.** The position is along the axis that holds the categories or the independent value: x for a vertical chart, y for horizontal bars. A single bar reads "Latte, 50". Marks sharing a position are summarised: a stack as its total, grouped bars as an average, several lines as a span. On a number or date axis, more than a few marks are merged into stretches of the axis, each with a summary.
- **Axes, legend, annotation views and views in a `chartBackground` are not elements.** Views in a `chartOverlay` are.
- **A default Audio Graph** for most charts, titled "Chart", with no summary, the `.value` labels as axis titles, and each point described as its x value and a bare number: no units, and on a date axis no date.
- **The words come from the code.** `.value` labels are spoken as axis names, and the label of a category or series is made plural inside summaries ("3 Sizes", "2 Stores"). Write every label as words a listener will understand, and a category's or series' label in the singular.

That is a starting point. The framework knows the values and their labels, not the chart's purpose, its units, or how a date should be said. The rest of this file is what the app adds.

## Decide what an element is

How much of the chart one accessibility element should cover depends on what the chart is for:

| The chart's purpose | Elements | Example |
|---|---|---|
| Exact values per item | One per position, the default for categories and for a few dates | A step count for each day |
| The overall shape, not individual readings | One per meaningful portion, each summarising it; automatic on a number or date axis | Elevation over a route, described a stretch at a time |
| A small preview that opens a fuller chart | One for the whole chart, with a succinct description | A sparkline in a button |

A chart on a continuous axis with many marks is already in the middle case: the framework merges it into portions itself. Supply your own portions when its wording will not do, which is usually because it has no units (*One element per `Plot`*).

In every case, leave the axes out of it. The framework does not make axis or tick labels into elements, and they should not become separate stops: people get values from the marks and the Audio Graph.

For the last case, collapse the chart into one element. This also removes its Audio Graph, which is right for a preview and wrong for a chart people study:

<!-- snippet: Accessibility/AccessibilitySnippets.swift#single-label -->
```swift
struct CompactChartButton: View {
    let days: [DailySales]
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            Chart(days) { day in
                BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
            }
            .chartXAxis(.hidden)
            .chartYAxis(.hidden)
            .frame(width: 120, height: 44)
        }
        // One succinct description for the whole thing; the detail screen has the full chart.
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Sales, last 30 days")
        .accessibilityValue("\(days.map(\.cups).reduce(0, +)) cups in total")
    }
}
```

## Labels and values

**Put a mark's label and value on a `Plot` around it.** A labelled `Plot` (iOS 16.0+) is one accessibility element with that label, whatever else shares its position:

<!-- snippet: Accessibility/AccessibilitySnippets.swift#mark-labels -->
```swift
struct AccessibleBars: View {
    let days: [DailySales]

    var body: some View {
        Chart(days) { day in
            // The label goes on a Plot around the mark: that makes this bar one element,
            // however many bars there are.
            Plot {
                BarMark(
                    x: .value("Day", day.day, unit: .day),
                    y: .value("Cups", day.cups)
                )
            }
            // Context first, in an unambiguous format; then the value with its unit spelled out.
            .accessibilityLabel(day.day.formatted(.dateTime.weekday(.wide).month(.wide).day()))
            .accessibilityValue("\(day.cups) cups")
        }
    }
}
```

Four modifiers apply to a `Plot`, and to marks:

| Modifier | Does |
|---|---|
| `accessibilityLabel(_:)` | Says what the thing is |
| `accessibilityValue(_:)` | Says the value it shows |
| `accessibilityHidden(_:)` | Hides it from assistive technologies |
| `accessibilityIdentifier(_:)` | An identifier, for UI tests |

Label and value accept a string, a `Text`, a `LocalizedStringKey` or a `LocalizedStringResource`, so they can be localized like any other text.

**On a bare mark a label is unreliable.** It is read when the mark is alone in its element. On stacked or grouped bars, several series, a heat map, or an axis dense enough to be merged it is ignored and the automatic summary is read; on the marks of a custom `ChartContent` it can land on each inner mark separately. The key-path forms on vectorized plots are not read at all (`vectorized-plots.md`). A labelled `Plot` around a function `LinePlot` is one element with that label and value. `accessibilityHidden` works on a bare mark everywhere.

**A rule across the plot is the exception:** it is always an element of its own, so label it directly.

**Label every date.** Unlabelled, a mark at a date is announced as a range of date-times ("Mar 1, 2026 at 12 AM to Mar 2, 2026 at 12 AM"). A label replaces it: "Sunday, March 1".

**Label or hide every rule.** A target or average line is an element with a bare number and no name. Give it a label, and a value with its unit. Hide a rule or band that only marks the current selection: it reads as a position with no value.

<!-- snippet: Accessibility/AccessibilitySnippets.swift#rule-label -->
```swift
struct BarsWithLabelledTarget: View {
    let days: [DailySales]
    let target: Int

    var body: some View {
        Chart {
            ForEach(days) { day in
                Plot {
                    BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
                }
                .accessibilityLabel(day.day.formatted(.dateTime.weekday(.wide).month(.wide).day()))
                .accessibilityValue("\(day.cups) cups")
            }
            // A rule across the plot is an element of its own, so it is labelled directly.
            RuleMark(y: .value("Target", target))
                .foregroundStyle(.red)
                .accessibilityLabel("Target")
                .accessibilityValue("\(target) cups")
                // The visible label; annotation views are not read, so the rule carries the words too.
                .annotation(position: .top, alignment: .leading) {
                    Text("Target: \(target, format: .number)").font(.caption).foregroundStyle(.red)
                }
        }
    }
}
```

**Writing them:**

- **Context, then detail.** Putting the date or name first lets people scan for the item they want and tells them where they are when they jump around the chart: "June 1, 36 pancakes". A bare value is rarely enough. Say what it belongs to (the date, the place, the series), then the value. Do not repeat what the Audio Graph or the chart's overview already gives, such as the axis name.
- **Actual values, not judgements.** "Rose from 110 to 440" lets people draw their own conclusion; "rose rapidly" is yours. Avoid words like rapidly, gradually and almost.
- **No ambiguous formats or abbreviations.** "June 6", not "6/6". "60 minutes" or "60 meters", not "60m".
- **What it represents, not what it looks like.** Name the series ("Harbour store"), not its colour.
- **Refer to the axes in the same order everywhere** in the app, for instance always x first.
- **Let the grammar agree with the number:** `Text("^[\(count) cup](inflect: true)")` reads "1 cup" and "2 cups".

**Hide what adds nothing.** `accessibilityHidden(true)` removes a mark from the elements and from the automatic Audio Graph alike. Use it on decoration, such as a gauge's track, and on any layer that repeats another's data: an area under a line makes every value count twice, and points drawn over a line become a second series. Hide the extra layer, or put points on the line with `symbol`, which adds no marks.

## One element per `Plot`

`Plot` (iOS 16.0+) groups chart content, and a labelled plot is one accessibility element whose frame covers the marks inside it. That makes the plot the unit for deciding what an element is.

**One mark, one element.** Each segment of a stacked bar:

<!-- snippet: Accessibility/AccessibilitySnippets.swift#plot-per-segment -->
```swift
struct StackedBarsWithSegmentElements: View {
    let orders: [DrinkOrder]

    var body: some View {
        Chart(orders) { order in
            // Without the Plot these labels are not read, and each drink is one element
            // ("Latte, total 50, 3 Sizes"). With it, each segment is its own.
            Plot {
                BarMark(x: .value("Drink", order.drink), y: .value("Cups", order.cups))
                    .foregroundStyle(by: .value("Size", order.size))
            }
            .accessibilityLabel("\(order.drink), \(order.size)")
            .accessibilityValue("\(order.cups) cups")
        }
    }
}
```

The same works for each cell of a heat map, each bar of a timeline or a population pyramid, each point of a scatter plot, and each point of every line in a multi-series chart.

**Several marks, one element.** Put the marks that make up one thing in one plot. For a portion of a dense line, the plot holds that stretch's marks; the line is still drawn as one:

<!-- snippet: Accessibility/AccessibilitySnippets.swift#plot-per-portion -->
```swift
struct ElevationProfile: View {
    let samples: [ElevationSample]
    /// How many samples one accessibility element describes.
    var samplesPerPortion = 50

    private var portions: [[ElevationSample]] {
        stride(from: 0, to: samples.count, by: samplesPerPortion).map {
            Array(samples[$0 ..< min($0 + samplesPerPortion, samples.count)])
        }
    }

    var body: some View {
        Chart {
            ForEach(portions, id: \.first?.id) { portion in
                // One Plot per stretch of the route: one element, lying over that stretch.
                // The line is still drawn as one line.
                Plot {
                    ForEach(portion) { sample in
                        LineMark(x: .value("Distance", sample.kilometre), y: .value("Elevation", sample.metres))
                    }
                }
                .accessibilityLabel(
                    "Kilometre \(portion.first?.kilometre.formatted() ?? "") to \(portion.last?.kilometre.formatted() ?? "")")
                .accessibilityValue(
                    "Starts at \(portion.first?.metres ?? 0) metres, ends at \(portion.last?.metres ?? 0) metres, lowest \(portion.map(\.metres).min() ?? 0), highest \(portion.map(\.metres).max() ?? 0)")
            }
        }
    }
}
```

For a mark built from several, such as a box plot's whisker, box and median or a candlestick's wick and body, put the plot inside the custom `ChartContent` type so that a label given to it from outside describes the whole (`gallery/candlestick.md`). Where the marks are written inline, as in `gallery/box-plot.md`, the plot goes around them in the chart.

Labels on plots change the elements only. The automatic Audio Graph is whatever it was, with its bare values, or absent; that is the chart descriptor's job.

A labelled plot per mark turns the automatic merging off: 400 labelled plots are 400 stops. Label per mark only when the individual values matter, and per portion otherwise. As a guide: one series of up to about a month of days reads well a day at a time; several series over the same dates, or a longer run, are better as a portion per series (a week, a month), since a listener otherwise walks every point of one series before reaching the next.

### When the elements should not follow the marks

- **A simpler chart for assistive technologies:** `accessibilityRepresentation` (SwiftUI, iOS 15.0+) gives VoiceOver another view in place of the drawn chart, elements and Audio Graph both. Use it when the drawn chart is too layered to describe mark by mark, such as stacked segments that are better heard as one total per category:

<!-- snippet: Community/CommunityPatterns.swift#representation -->
```swift
struct StackedBarsWithSimplerRepresentation: View {
    let orders: [DrinkOrder]

    private var totals: [(drink: String, cups: Int, detail: String)] {
        Dictionary(grouping: orders, by: \.drink).map { drink, rows in
            (drink, rows.map(\.cups).reduce(0, +),
             rows.map { "\($0.size) \($0.cups)" }.formatted(.list(type: .and)))
        }
        .sorted { $0.drink < $1.drink }
    }

    var body: some View {
        Chart(orders) { order in
            BarMark(x: .value("Drink", order.drink), y: .value("Cups", order.cups))
                .foregroundStyle(by: .value("Size", order.size))
        }
        // Assistive technologies get this chart in place of the drawn one: one bar per drink,
        // so each label and value is read, and the Audio Graph plays the totals.
        .accessibilityRepresentation {
            Chart(totals, id: \.drink) { total in
                BarMark(x: .value("Drink", total.drink), y: .value("Cups", total.cups))
                    .accessibilityLabel(total.drink)
                    .accessibilityValue("\(total.cups) cups: \(total.detail)")
            }
        }
    }
}
```

- **Elements with traits or a layout of their own:** `accessibilityChildren` (SwiftUI, iOS 15.0+) swaps a view's elements for those of views you supply, which are never drawn. Apply it to the plot area inside `chartPlotStyle`, so the views are laid out in the plot's frame; applied to the chart they fill the whole chart, axes included, and a chart with a colour encoding loses its automatic Audio Graph. The calendar recipe uses it to mark the first day of each month as a heading (`accessibilityAddTraits(.isHeader)`), which lets VoiceOver users jump between months (`gallery/calendar.md`).

## The Audio Graph

**Every chart gets a chart descriptor.** The automatic Audio Graph is titled "Chart", has no summary, gives bare numbers with no units, drops the dates of a date axis, and does not exist at all for pies, heat maps, timelines and other charts without a value on each axis. Writing the descriptor is in `audio-graph.md`.

## Descriptions everyone reads

A visible title and a one-line summary of the chart's message help every reader and are what a VoiceOver user meets first (`design.md`, *Descriptions*). They do not replace the labels or the descriptor.

## Beyond VoiceOver

Keyboard, Switch Control and Voice Control navigation, colour and contrast, announcing changes, and Dynamic Type are in `accessibility-beyond-voiceover.md`. They apply to every chart too.

## Testing a chart's accessibility

- **Read the elements.** Accessibility Inspector (Xcode, *Open Developer Tool*) shows the label and value of each element in a running app without VoiceOver. Compare them with what the chart is meant to say; this is where an ignored label or a merged axis shows up.
- **Listen.** Turn VoiceOver on, move through the chart, and open its Audio Graph from the rotor.

## Pitfalls

- **Labels that change while the data does not.** The chart keeps the accessibility elements it made for each data element's id. When a control changes only what the labels say (a toggle that relabels the same bars), the ids stay the same and VoiceOver keeps reading the old words, though the marks redraw. Give the chart a new identity when that state changes, `.id(state)` on the `Chart`. Read on iOS with the elements dumped before and after.

- Treating the chart as an image with one label, when its purpose is the individual values.
- Labelling every point of a dense chart whose purpose is the overall shape.
- Labels on bare marks that share a position. They are not read; put them on a `Plot`.
- A label that is only a number, a subjective word, or the name of a colour.
- A target line announced as a bare number, or a selection indicator left as an extra stop.
- A value written only in an annotation or a `chartBackground` view. Neither is read.
- Decoration, or a second layer of the same data, left visible to assistive technologies.
- Relying on animation alone to signal that the chart changed.
