# Small multiples

![Three short bar charts stacked vertically, titled Tea, Mocha and Latte, each on the same 0 to 700 scale, with one time axis under the last.](../../assets/gallery/small-multiples.png)

## Visual signature

Several small charts of the same type stacked or laid out in a grid, one per category or measure, lined up on a shared axis. Each has its own short title; often only the last shows the shared axis labels. Apple's Activity screen shows Move, Exercise and Stand this way. Not this form if the series share one plot area (`multi-series-line.md`, `stacked-bar.md`).

## Composition

**Verdict: escape hatch.** Swift Charts has no faceting. Each small chart is its own `Chart`, arranged with ordinary SwiftUI layout. Minimum iOS 16.0.

- One `Chart` per group in a `VStack` or grid, built by a `ForEach`.
- The same scale domain on every chart where values should be comparable across them, worked out from all of them. Without it each chart scales to its own data and equal heights mean different values.
- The shared axis's labels on the last chart only, and the data's calendar on the values and the axis (`../data.md`).
- **The plot's height set, not the chart's** (`chartPlotStyle { $0.frame(height:) }`, `../plot-area.md`). A chart's frame includes its axis labels: with `.frame(height: 60)` on each chart, the one showing the months measured a 42-point plot against 60 for the others, so its bars read 70% as tall on the same scale (iOS).

<!-- snippet: Design/HIGPatterns.swift#small-multiples -->
```swift
struct SmallMultiples: View {
    let trends: [DrinkTrend]
    let calendar: Calendar // the calendar the dates were made in
    @Environment(\.locale) private var locale

    private var drinks: [String] {
        var seen: [String] = []
        for trend in trends where !seen.contains(trend.drink) { seen.append(trend.drink) }
        return seen
    }

    /// One top for every chart, from all of them: the largest value rounded up to 100.
    private var top: Int {
        max(100, ((trends.map(\.cups).max() ?? 0) + 99) / 100 * 100)
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            ForEach(drinks, id: \.self) { drink in
                Text(drink).font(.caption.bold())
                Chart(trends.filter { $0.drink == drink }) { item in
                    BarMark(
                        x: .value("Month", item.month, unit: .month, calendar: calendar),
                        y: .value("Cups", item.cups)
                    )
                }
                // The same y scale in every chart, so heights compare across them.
                .chartYScale(domain: 0 ... top)
                .chartYAxis {
                    AxisMarks(values: [0, top])
                }
                // Grid lines on every chart; the months under the last chart only.
                .chartXAxis {
                    AxisMarks(values: .stride(by: .month, calendar: calendar)) { _ in
                        AxisGridLine()
                        if drink == drinks.last {
                            AxisValueLabel(
                                format: Date.FormatStyle(locale: locale, calendar: calendar, timeZone: calendar.timeZone).month(.narrow),
                                centered: true
                            )
                        }
                    }
                }
                // The plot's height, not the chart's: a chart frame includes the month labels,
                // so the last chart's plot would be shorter and its bars would read smaller.
                .chartPlotStyle { $0.frame(height: 60) }
            }
        }
    }
}
```

## Variations

**Different measures with different units:** leave each chart its own y scale, and give each its own colour and a value label, as the Activity charts do.

## Accessibility

Each chart is its own set of elements with its own descriptor. Make each title a header, label one element per bar, and say in each summary which scale the chart is drawn on. Whether VoiceOver relates the charts to one another has not been heard.

## Pitfalls

- Use one chart with a colour encoding instead when the series are few and should be compared directly.
