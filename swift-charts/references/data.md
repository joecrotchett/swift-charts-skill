# Data for charts

Shaping app data so it can be plotted.

## Start from the question

Decide what question the chart answers, then give the data one property per thing the chart must show. A bar chart of counts per category needs a category and a count. Splitting those bars by a second dimension needs a third property and one row per combination (category, sub-category, amount), not one row per category with several amount columns.

## Plottable values

### `PlottableValue` - iOS 16.0+

**Use for:** every piece of data handed to a mark or a `by:` modifier. Written with the shorthand `.value("Label", value)`.

The type of the value decides how the chart treats it:

| Value type | Treated as | Result on an axis |
|---|---|---|
| Number (`Int`, `Double`, `Decimal`, other integer and floating-point types) | Quantitative | A continuous number scale |
| `Date` | Temporal | A continuous date scale with date labels |
| `String` | Categorical | One band per distinct value, in data order |

So the same `BarMark(x:y:)` is a vertical bar chart when `x` is a string and `y` a number, and horizontal when they are swapped: the bar grows along whichever axis holds the quantity. Swapping them is the quick fix when category labels are crowded along the bottom.

**Bars of different widths** (a range per period, when the periods differ in length) make a long period look bigger than its total: say beside the chart that the height is the total, or plot a rate per unit of time instead.

**A range as a value.** `.value("Time", start ..< end)` plots an interval in one parameter. A bar given a range of dates on x spans that period less the gap any bar leaves at its default width, about a third of the period, so pass `width: .inset(1)` for bars that should touch, an alternative to `unit:` when the periods are not calendar units, such as half hours:

<!-- snippet: Axes/AxisRecipes.swift#range-value -->
```swift
Chart(readings) { reading in
    BarMark(
        // One bar per half hour: the x value is a range of dates.
        x: .value("Time", reading.time ..< reading.time.addingTimeInterval(1800)),
        y: .value("Occupancy", reading.percent)
    )
}
```

The label is the first argument: a string, a `LocalizedStringKey`, a `LocalizedStringResource` or a `Text`.

### `Plottable` - iOS 16.0+

**Use for:** plotting a type of your own directly, without converting it at every `.value(...)`. The numeric types, `Date` and `String` already conform.

An enum with `String` raw values needs only the conformance. It plots as categories, and its raw values are used as localized string keys wherever the chart shows them, on an axis or in a legend:

<!-- snippet: Gallery/GalleryData.swift#plottable-enum -->
```swift
enum OrderStatus: String, Plottable {
    case queued = "Queued"
    case brewing = "Brewing"
    case served = "Served"
}
```

<!-- snippet: Data/PlottableSnippets.swift#enum-chart -->
```swift
struct StatusChart: View {
    let counts: [StatusCount]

    var body: some View {
        Chart(counts) { item in
            BarMark(
                x: .value("Status", item.status),
                y: .value("Orders", item.count)
            )
        }
    }
}
```

For any other type, say which primitive value (a number, a `Date` or a `String`) stands for it, and how to get back from one. The initializer is failable for primitive values with no counterpart:

<!-- snippet: Data/PlottableSnippets.swift#custom-plottable -->
```swift
struct StoreCode: Plottable, Hashable {
    let code: Int

    var primitivePlottable: String { "Store \(code)" }

    init(code: Int) { self.code = code }

    init?(primitivePlottable: String) {
        guard let code = Int(primitivePlottable.dropFirst("Store ".count)) else { return nil }
        self.code = code
    }
}
```

**Prefer an enum over strings** for a fixed set of categories: the compiler checks the cases and the names can be localized.

**Pitfalls:** an identifier stored as an `Int` (a year, a product number) plots as a quantity, giving a continuous number axis instead of one band per value. Convert it to a `String` to get categories.

## Dates and time units

`.value("Day", date, unit: .day)` plots a date as the whole calendar unit that contains it. Use it whenever a date stands for a period, which is nearly always the case for bars: one bar per day, per month.

- With a unit, a bar occupies its day or month, and an axis label can be centred under it.
- Without a unit, the date is a single instant. A bar keeps its default width, a few points, centred on that instant: with a few periods (a week of days, six months) it is a sliver with empty space between, and with many it can look right but is shifted half a period. The first and last bars hang half outside the plot and are drawn over the axis.

Use the same unit on every mark that shares the axis, including a `RuleMark` used as a selection indicator, so they align.

**A unit is cut in a calendar, and by default it is the device's.** When the dates were recorded in another time zone (daily totals stored as midnight UTC are the usual case), a device west of that zone puts every mark on the previous day, and one east of it can do the reverse. Pass the data's calendar to the value, `.value("Day", date, unit: .day, calendar: dataCalendar)`, and to the axis, which otherwise still places its marks and writes its labels in the device's calendar and time zone. (To keep the default axis untouched instead, move each date to the start of the same day in the device's calendar before charting.) Text written about such a date needs the same care: on iOS, `Text(date, format: style)` formats in the environment's time zone and ignores one set on the style, so a day stored as midnight UTC is written as the day before anywhere west of Greenwich. Format to a string first, `Text(date.formatted(style))`, or set `\.timeZone` in the environment. With the calendar passed to both:

<!-- snippet: Data/CalendarSnippets.swift#data-calendar -->
```swift
/// Daily totals that were counted in UTC, drawn correctly on a device in any time zone.
struct DailyTotalsInDataTimeZone: View {
    let days: [DailySales]
    /// The calendar the days were counted in, with its time zone set.
    let dataCalendar: Calendar
    @Environment(\.locale) private var locale

    var body: some View {
        Chart(days) { day in
            // Without `calendar:` the day is cut in the device's calendar, and midnight UTC
            // falls on the previous day anywhere west of Greenwich.
            BarMark(x: .value("Day", day.day, unit: .day, calendar: dataCalendar), y: .value("Cups", day.cups))
        }
        .chartXAxis {
            // The axis has to be told too, or its marks and labels follow the device.
            AxisMarks(values: .stride(by: .day, count: 7, calendar: dataCalendar)) {
                AxisGridLine()
                AxisTick()
                // An explicit format carries its own locale, so hand it the view's.
                AxisValueLabel(format: Date.FormatStyle(locale: locale, timeZone: dataCalendar.timeZone).month().day())
            }
        }
    }
}
```

Write accessibility labels and chart descriptor dates with the same time zone, so what is spoken names the day that is drawn.

## Categories and ordering

Categories appear in the order their values first occur in the data. To rank bars, sort the data.

## Binning and aggregation

A chart draws the data it is given; it does not count or sum. To show how values are distributed, or totals per period, group the data into bins first and plot one mark per bin. The bin types work out the boundaries; the grouping is ordinary Swift.

### `NumberBins` - iOS 16.0+

**Use for:** dividing a range of numbers into bins, for a histogram.

| Initializer | Bins |
|---|---|
| `NumberBins(data: values)` | A count chosen from the data |
| `NumberBins(data: values, desiredCount: 8)` | About that many, with round boundaries. Asking for 8 can give 10 |
| `NumberBins(range: 0 ... 100, count: 5)` | Exactly that many equal bins |
| `NumberBins(size: 10, range: 0 ... 100)` | Equal bins of a given width |
| `NumberBins(thresholds: [0, 18, 35, 65, 120])` | Bins between the boundaries you list; N boundaries make N - 1 bins |

It is a collection of `ChartBinRange`. `bins.index(for: value)` says which bin a value belongs to, and `bins.thresholds` lists the boundaries.

<!-- snippet: Data/BinSnippets.swift#histogram -->
```swift
struct HistogramRecipe: View {
    let beans: [Bean]

    /// One entry per bin: the range it covers and how many beans fall in it.
    private var counts: [(range: ChartBinRange<Double>, count: Int)] {
        let bins = NumberBins(data: beans.map(\.acidity), desiredCount: 8)
        let grouped = Dictionary(grouping: beans) { bins.index(for: $0.acidity) }
        return bins.indices.map { index in
            (range: bins[index], count: grouped[index]?.count ?? 0)
        }
    }

    var body: some View {
        Chart(counts, id: \.range.lowerBound) { bin in
            BarMark(
                x: .value("Acidity", bin.range),
                y: .value("Beans", bin.count),
                // Neighbouring bars one hairline apart; the default leaves wide gaps.
                width: .inset(1)
            )
        }
        .chartXScale(domain: .automatic(includesZero: false))
    }
}
```

Recipe: `gallery/histogram.md`.

**Binning on two axes** gives a two-dimensional histogram: bin each property, group by the pair of indices, and plot a `RectangleMark` per cell coloured by its count. Create a cell for every combination, with a count of zero where there is no data, or the grid has holes. Recipe: `gallery/heat-map.md`.

### `DateBins` - iOS 16.0+

**Use for:** the same for dates: totals per week, per fortnight, per six hours.

- `DateBins(unit: .weekOfYear, range: first ... last)`, with `by:` for multiples of the unit.
- `DateBins(timeInterval: 6 * 3600, range:)` for a fixed length of time.
- `DateBins(data:desiredCount:)`, `DateBins(range:desiredCount:)` and `DateBins(thresholds:)` as for numbers.

<!-- snippet: Data/BinSnippets.swift#date-bins -->
```swift
func weeklyTotals(of days: [DailySales], calendar: Calendar) -> [(week: ChartBinRange<Date>, cups: Int)] {
    guard let first = days.first?.day, let last = days.last?.day else { return [] }
    let bins = DateBins(unit: .weekOfYear, range: first ... last, calendar: calendar)
    let grouped = Dictionary(grouping: days) { bins.index(for: $0.day) }
    return bins.indices.map { index in
        (week: bins[index], cups: grouped[index]?.reduce(0) { $0 + $1.cups } ?? 0)
    }
}
```

For whole calendar units with one row per unit already in the data, `unit:` on the value is simpler (see *Dates and time units*).

### `ChartBinRange` - iOS 16.0+

**Use for:** one bin, with a `lowerBound` and an `upperBound`. Pass it straight to `.value("Label", bin)` and the bar spans the bin. Every bin excludes its upper bound except the last, which includes it, so a value on a boundary is counted once.

**Pitfalls:**

- Plotting a bin with `BarMark(xStart:xEnd:y:)`. That form treats y as a position, and draws a dash at the count instead of a bar up to it. Pass the bin as the x value.
- Leaving the default bar width. Histogram bars should touch or nearly touch: `width: .inset(1)` or `.ratio(1)`.
- Binning by hand with rounding. `index(for:)` handles the boundaries.
