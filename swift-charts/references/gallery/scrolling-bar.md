# Scrolling bar chart

No image: offscreen, a scrolling chart draws its axis and grid but no bars (`../interaction.md`).

## Visual signature

A bar chart of a long time range that shows only a window of it. Signs in a static image: bars cut off at the leading or trailing edge, far more history implied than fits, and a date range above the chart (for example "Mar 3 – Apr 2") that describes the visible window, with a total for that window.

## Composition

Minimum iOS 17.0 for the four scrolling modifiers.

- `chartScrollableAxes(.horizontal)` turns scrolling on.
- `chartXVisibleDomain(length:)` sets how much of the axis is visible. For a date axis the length is in seconds.
- `chartScrollTargetBehavior(.valueAligned(matching:majorAlignment:))` makes scrolling come to rest on a boundary: here on a midnight, preferring the first of a month.
- `chartScrollPosition(x:)` binds the leading edge of the window, so the surrounding view can show the visible range and its total.

<!-- snippet: Gallery/ScrollingBarChart.swift#chart -->
```swift
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct ScrollingBarChartRecipe: View {
    let days: [DailySales]
    /// The leading edge of the visible window. Read it to summarise what is on screen.
    @Binding var scrollPosition: Date

    /// The calendar the dates were recorded in: days are cut in it, not in the device's (`data.md`).
    var calendar = Calendar(identifier: .gregorian)
    @Environment(\.locale) private var locale

    private let visibleSeconds = 3600 * 24 * 30

    var body: some View {
        Chart {
            ForEach(days) { item in
                BarMark(
                    x: .value("Day", item.day, unit: .day, calendar: calendar),
                    y: .value("Cups", item.cups)
                )
            }
            .foregroundStyle(.blue)
        }
        .chartScrollableAxes(.horizontal)
        .chartXVisibleDomain(length: visibleSeconds)
        .chartScrollTargetBehavior(
            .valueAligned(
                matching: DateComponents(hour: 0),
                majorAlignment: .matching(DateComponents(day: 1))
            )
        )
        .chartScrollPosition(x: $scrollPosition)
        .chartXAxis {
            AxisMarks(
                format: Date.FormatStyle(locale: locale, calendar: calendar, timeZone: calendar.timeZone).month().day(),
                values: .stride(by: .day, count: 7, calendar: calendar)
            )
        }
    }
}
```

## Variations

- **Start at the latest data.** Initialise the bound position to the last date minus the visible length, so the chart opens on the most recent window.
- **Headline for the window.** Compute the end of the window as `scrollPosition + visible length` and sum the data inside it for the text above the chart.

## Pitfalls

- The scroll position is the leading edge of the window, not its centre.
- When a control switches the chart between different data sets (30 days of daily bars, 12 months of monthly bars), disable the animation for that change; the bars are different things and should not morph into each other.
- A scrollable chart cannot be captured with `ImageRenderer`: it comes out with no bars. To export an image, render a non-scrolling version of the chart. Its own chart descriptor is ignored too; `../interaction.md` (*Scrolling*) gives the route that works.
- On an iOS 16 deployment target there is no framework scrolling: see `../availability/README.md`.
