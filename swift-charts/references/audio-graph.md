# Audio Graphs and chart descriptors

Giving a chart's Audio Graph a title, a summary, named series, units and dates, and giving one to a chart that has none. What the Audio Graph gets with no code is listed in `accessibility-defaults.md`.

Every chart delivered gets one (the protocol, `AXChartDescriptorRepresentable`, is SwiftUI's: a file that declares one imports SwiftUI as well as Accessibility): the automatic Audio Graph has the title "Chart", no summary, no units and no dates, and some charts have none at all. A descriptor replaces everything the Audio Graph is given: the title and a summary, which VoiceOver speaks so people learn the chart's purpose and main features before exploring it, the axes, the series, and how each value is said, units and dates included. Conform a type to `AXChartDescriptorRepresentable` and attach it with `accessibilityChartDescriptor(_:)`. Both come from SwiftUI and the Accessibility framework, not from Swift Charts, and exist on every OS that Swift Charts does.

<!-- snippet: Accessibility/AccessibilitySnippets.swift#descriptor -->
```swift
/// A count with its unit as a plain string that agrees in number: "1 cup", "36 cups".
/// A descriptor takes strings, not `Text`, so the agreement is resolved here. Pass the view's
/// locale so the number is written the way the chart beside it writes numbers.
func spokenCups(_ count: Int, locale: Locale = .current) -> String {
    String(AttributedString(localized: "^[\(count) cup](inflect: true)", locale: locale).characters)
}

struct SalesChartDescriptor: AXChartDescriptorRepresentable {
    let days: [DailySales]
    let calendar: Calendar // the calendar the dates were made in
    /// The value axis as drawn. Set it on the chart with `chartYScale(domain:)` and pass the same range here:
    /// an automatic axis rounds its ends, and the descriptor cannot read them.
    let cupsAxis: ClosedRange<Double>

    func makeChartDescriptor() -> AXChartDescriptor {
        let cups = days.map { Double($0.cups) }
        let dayName = Date.FormatStyle(timeZone: calendar.timeZone).month(.wide).day()
        let xAxis = AXCategoricalDataAxisDescriptor(
            title: "Day",
            categoryOrder: days.map { $0.day.formatted(dayName) }
        )
        let yAxis = AXNumericDataAxisDescriptor(
            title: "Cups sold",
            range: cupsAxis,
            gridlinePositions: []
        ) { value in
            // `Int` traps on a value that is not finite; describe only a finite one.
            value.isFinite ? spokenCups(Int(value.rounded())) : ""
        }
        let series = AXDataSeriesDescriptor(
            name: "Harbour store",
            isContinuous: false,
            dataPoints: days.map { day in
                AXDataPoint(x: day.day.formatted(dayName), y: Double(day.cups))
            }
        )
        return AXChartDescriptor(
            title: "Daily sales at the Harbour store",
            // The main message in actual values: no "rose" or "fell", which would be a judgement.
            summary: "\(days.count) days. First day \(spokenCups(days.first?.cups ?? 0)), last day \(spokenCups(days.last?.cups ?? 0)), highest \(spokenCups(Int(cups.max() ?? 0))).",
            xAxis: xAxis,
            yAxis: yAxis,
            additionalAxes: [],
            series: [series]
        )
    }
}

struct DescribedSalesChart: View {
    let days: [DailySales]
    let calendar: Calendar

    /// Rounded up to the next 50 cups (at least 50, so empty data still has an axis); the drawn axis and the descriptor share it.
    private var cupsAxis: ClosedRange<Double> {
        0 ... max(50, (Double(days.map(\.cups).max() ?? 0) / 50).rounded(.up) * 50)
    }

    var body: some View {
        AccessibleBars(days: days, calendar: calendar)
            .chartYScale(domain: cupsAxis)
            .accessibilityChartDescriptor(SalesChartDescriptor(days: days, calendar: calendar, cupsAxis: cupsAxis))
    }
}
```

That example has a categorical x axis and separate bars. Choosing each part:

- **Dates:** a categorical axis of written-out dates, as above. A date axis has no descriptor type of its own.
- **`isContinuous`:** `true` when the chart draws one connected shape over an ordered x such as time or distance (a line, an area, or bars standing in for one, such as an elevation profile); `false` for separate totals, a bar per day or per category included, and for a line or area drawn over unordered categories. The Audio Graph draws its own picture of the chart from the descriptor, a line or bars accordingly.
- **Several numbers at one position** (a range's low and high, a box plot's five numbers) are one named series per number, over the same categories.
- **Every series named**, one `AXDataSeriesDescriptor` per series the listener should be able to play on its own. In a stacked chart each point carries its piece's own value, not its height in the stack; the framework's own Audio Graph does the same.
- **The summary in actual values.** The system adds its own remarks on trend beneath it.
- **The y axis's range matches the drawn scale:** from zero when the chart's value axis includes zero, from the lowest value when it does not. When you set the axis's values yourself and they reach past the data, they widen the scale (a log axis labelled at 100 and 1,000,000 runs from 100 to 1,000,000): take the range from those values. An automatic axis's rounded ends cannot be read without a `ChartProxy`, and the framework's own Audio Graph uses them (0 to 300 for data up to 219): set the value axis's domain yourself with `chartYScale(domain:)` and give the descriptor that same range.
- **A scrolling chart ignores its descriptor** (`chartScrollableAxes`): attach it inside an `accessibilityRepresentation` instead (`interaction.md`, *Scrolling*).
- **A descriptor that changes with state needs a new chart identity.** The Audio Graph keeps the descriptor it read for the chart's identity, so a control that changes only the range or summary leaves the old one (read on iOS). Give the chart `.id` built from every such state (`accessibility.md`, *Pitfalls*).
- **A third dimension** goes in `additionalAxes: [AXNumericDataAxisDescriptor(...)]`, each point carrying its value as `AXDataPoint(x:y:additionalValues: [.number(z)])` (read on iOS as an extra axis; `additionalValues` cannot be read back in Swift, so test the axes and counts).
- **Rules** such as a target or average are not series; state them in the summary.
- **Categories go on the descriptor's x axis whichever way the chart is drawn.** The y axis of a descriptor is always numeric, so a horizontal bar chart is described with its categories on x.
- **Values that agree in number.** The value description and the summary are plain strings, so `Text`'s inflection is not available to them directly; resolve it into a string, as `spokenCups` does above, so a value of one is not read as "1 cups". Use that one form for every count the app speaks or writes, and pass it the locale: the number is then written in that locale, while an untranslated English noun still agrees in English ("1.234 cups" under a German locale); a hand-written `count == 1 ? ... : ...` is a second way to get the same thing wrong.
- **`updateChartDescriptor(_:)` needs no implementation** unless the descriptor must change after it is made; the protocol supplies an empty one.
- **In a module whose default isolation is the main actor,** mark the descriptor type `nonisolated`. Its value-description closure is called by the accessibility system, and nothing here establishes that the call arrives on the main thread.

A descriptor is an ordinary value, so test it: its title, summary, axis titles, category order, series names and point counts, and what the value description returns for a given number. A data point's own number cannot be read back in Swift (`yValue` has no accessible number), so assert on the summary and the axis range instead.

A number on both axes takes `AXNumericDataAxisDescriptor` twice and data points made of two numbers:

<!-- snippet: Accessibility/AccessibilitySnippets.swift#descriptor-numeric -->
```swift
/// A descriptor for a line over a number axis: both axes numeric, each point a pair of numbers.
struct ElevationDescriptor: AXChartDescriptorRepresentable {
    let samples: [ElevationSample]

    func makeChartDescriptor() -> AXChartDescriptor {
        let heights = samples.map { Double($0.metres) }
        let lowest = heights.min() ?? 0
        let highest = heights.max() ?? 0
        return AXChartDescriptor(
            title: "Elevation along the route",
            summary: "\(Int(samples.last?.kilometre ?? 0)) kilometres. Lowest \(Int(lowest)) metres, highest \(Int(highest)) metres.",
            xAxis: AXNumericDataAxisDescriptor(
                title: "Distance",
                range: 0 ... (samples.last?.kilometre ?? 0),
                gridlinePositions: []
            ) { "\($0.formatted(.number.precision(.fractionLength(1)))) kilometres" },
            yAxis: AXNumericDataAxisDescriptor(
                title: "Elevation", range: lowest ... highest, gridlinePositions: []
            ) { $0.isFinite ? "\(Int($0.rounded())) metres" : "" },
            additionalAxes: [],
            series: [AXDataSeriesDescriptor(
                name: "Elevation",
                isContinuous: true,
                dataPoints: samples.map { AXDataPoint(x: $0.kilometre, y: Double($0.metres)) }
            )]
        )
    }
}
```

With thousands of points, describe what the chart shows, not every point: a sample, or the binned values the fallback chart draws.

A single value, such as a gauge, is one category with one point:

<!-- snippet: Accessibility/AccessibilitySnippets.swift#descriptor-single -->
```swift
/// A descriptor for a chart of one value, such as a gauge: one category, one point.
struct ProgressDescriptor: AXChartDescriptorRepresentable {
    let title: String
    /// From 0 to 1.
    let progress: Double

    func makeChartDescriptor() -> AXChartDescriptor {
        let percent = (progress * 100).rounded()
        return AXChartDescriptor(
            title: title,
            summary: "\(Int(percent)) percent.",
            xAxis: AXCategoricalDataAxisDescriptor(title: "Measure", categoryOrder: [title]),
            yAxis: AXNumericDataAxisDescriptor(title: "Percent", range: 0 ... 100, gridlinePositions: []) {
                $0.isFinite ? "\(Int($0.rounded())) percent" : ""
            },
            additionalAxes: [],
            series: [AXDataSeriesDescriptor(
                name: title, isContinuous: false, dataPoints: [AXDataPoint(x: title, y: percent)])]
        )
    }
}
```

A grid of two categories, such as a heat map, has no second category axis to give: describe it as one series per row, each over the column categories.

**Some charts get no automatic Audio Graph at all**: those whose marks do not have a value on each axis. That covers pies, donuts and gauges, heat maps and calendar grids, timelines, a single stacked bar, a one-axis strip of points, and candlesticks. A descriptor gives them one; for a single stacked bar, a pie or a donut, its value range runs from 0 to the total, of which each part is a share.

## Pitfalls

- Relying on the default Audio Graph for a chart of dates or of values with a unit.
- A pie, heat map, timeline or other chart with no automatic Audio Graph shipped without a descriptor.
- `isContinuous` that does not match the data, or a series with no name.
- A summary that judges ("rose sharply") instead of giving values.
