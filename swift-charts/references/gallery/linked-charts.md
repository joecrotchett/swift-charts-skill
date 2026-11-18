# Linked charts

![Two bar charts titled Harbour and Station, one above the other, with a grey band behind the same day in both.](../../assets/gallery/linked-charts.png)

## Visual signature

Two or more charts on one screen, usually sharing an axis, where the same period or item is highlighted in each: a shaded band behind the same bar, or the same point enlarged. Touching one chart moves the highlight in all of them, and nearby figures or other charts may narrow to the highlighted item. Typical of dashboards. Without the shared highlight it is `small-multiples.md`.

## Composition

- Each chart binds `chartXSelection` to the same value. Minimum iOS 17.0.
- A `RectangleMark` band, written once as a `ChartContent` type, drawn in every chart while a selection exists.
- The same x domain on every chart, covering all their days, so a day falls at the same place even when one chart's records start later; and the same y domain where heights should compare, which also keeps the plots the same width.

<!-- snippet: Dashboard/DashboardPatterns.swift#linked -->
```swift
/// Two charts that share one selection: touching either highlights the same day in both.
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct LinkedCharts: View {
    let stores: [StoreSales]
    @State var selectedDay: Date?
    /// The calendar the dates were recorded in (`data.md`).
    var calendar = Calendar(identifier: .gregorian)

    /// Every chart's days, from the first to the end of the last, so a day falls at the same place in each
    /// even when one store's records start later.
    private var sharedDays: ClosedRange<Date> {
        let days = stores.flatMap(\.days).map(\.day)
        let first = days.min() ?? .now, last = days.max() ?? .now
        return first ... (calendar.date(byAdding: .day, value: 1, to: last) ?? last)
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 16) {
            ForEach(stores) { store in
                Text(store.store).font(.headline)
                Chart {
                    if let selectedDay {
                        SelectedDayBand(day: selectedDay)
                    }
                    ForEach(store.days) { day in
                        BarMark(
                            x: .value("Day", day.day, unit: .day),
                            y: .value("Cups", day.cups)
                        )
                    }
                }
                // The same domains and the same binding on every chart. The shared y domain also gives every
                // chart the same axis labels, so their plots are the same width and the days line up.
                .chartXScale(domain: sharedDays)
                .chartYScale(domain: 0 ... 200)
                .chartXSelection(value: $selectedDay)
            }
        }
    }
}
```

<!-- snippet: Dashboard/DashboardPatterns.swift#selection-band -->
```swift
/// A band behind the marks for the selected day, as a reusable piece of chart content.
struct SelectedDayBand: ChartContent {
    let day: Date

    var body: some ChartContent {
        RectangleMark(
            x: .value("Selected day", day, unit: .day),
            width: .ratio(1)
        )
        .foregroundStyle(.gray.opacity(0.25))
        // Decoration: the bars already describe the day.
        .accessibilityHidden(true)
    }
}
```

## Variations

**Filtering other views to the selection.** Derive the selected period from the bound value in the model and filter the data other charts show. Fix those charts' colour and size scale domains so nothing changes colour as the subset changes (`../scales.md`).

**A chart as the control.** A small chart of categories whose bars switch what the other charts are coloured by, standing in for a legend (`../design.md`).

## Pitfalls

- **Text that promises the band "in both" charts** is wrong for a day only one of them recorded: the other has no bar there and draws none.
- The bound value is the raw position, not a data point. The band's `unit:` places it on the right day for drawing; snap the value yourself before using it to look anything up (`../interaction.md`).
- On an iOS 16 target the selection modifier has to be gated. Replace `chartXSelection` with the gated modifier under *Custom gestures* in `../interaction.md`, which falls back to an overlay gesture writing the same binding; the band and everything else here are iOS 16.0.
