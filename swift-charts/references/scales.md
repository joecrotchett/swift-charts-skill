# Scales

A scale maps data values to something visual: a position on an axis, a colour, a symbol, a size. Every scale is generated from the data; configure one only where the design differs from the default. A position scale is explained by an axis (`axes.md`); the others are explained by the legend (`legend.md`).

## Position scales

A position scale has three parts, each set by a parameter of the same modifier:

| Part | Is | Parameter | Type |
|---|---|---|---|
| Domain | The data values the scale covers | `domain:` | `ScaleDomain` |
| Range | The stretch of the plot area those values map onto | `range:` | `PositionScaleRange`, in practice `.plotDimension(...)`, a `PlotDimensionScaleRange` |
| Type | The function between them | `type:` | `ScaleType` |

### `chartYScale` - iOS 16.0+

**Use for:** overriding the y scale. `chartXScale` is the same for x. Pass only the parts that differ from the default: `chartYScale(domain:)`, `chartYScale(range:)`, `chartYScale(type:)`, or any combination.

### `ScaleDomain` - iOS 16.0+

**Use for:** the `domain:` of a scale.

**An exact domain** when the possible values are known and people should see the data against all of them, such as a percentage that can reach 100 whatever today's data is:

<!-- snippet: Scales/ScaleSnippets.swift#exact -->
```swift
func percentScale() -> some View {
    chartYScale(domain: 0 ... 100)
}
```

A closed range (`0 ... 100`) or a two-element array (`[0, 100]`) both work for numbers and dates. For a category scale, an array of the categories sets their order and can include categories the data lacks:

<!-- snippet: Scales/ScaleSnippets.swift#category-domain -->
```swift
func drinksInMenuOrder() -> some View {
    chartXScale(domain: ["Tea", "Latte", "Mocha", "Cocoa"])
}
```

**An adjusted automatic domain** when the data should still decide the extent:

- Number scales include zero by default, for every mark type, so that changing the mark never changes the scale. Bars need the zero baseline. Points and lines often do not: turn it off when zero is not meaningful and the data sits far from it.

<!-- snippet: Scales/ScaleSnippets.swift#nozero -->
```swift
func fittedToData() -> some View {
    chartXScale(domain: .automatic(includesZero: false))
        .chartYScale(domain: .automatic(includesZero: false))
}
```

- `reversed: true` flips the direction, for a rank axis with 1 at the top:

<!-- snippet: Scales/ScaleSnippets.swift#reversed -->
```swift
func rankedFromTop() -> some View {
    chartYScale(domain: .automatic(reversed: true))
}
```

- `modifyInferredDomain`, the trailing closure of `.automatic(dataType:)`, edits the inferred domain: sort the categories, or make sure a value such as a target is always on the scale.

<!-- snippet: Scales/ScaleSnippets.swift#modify -->
```swift
func sortedCategories() -> some View {
    chartXScale(domain: .automatic(dataType: String.self) { categories in
        categories.sort()
    })
}

func alwaysShowing(_ target: Double) -> some View {
    chartYScale(domain: .automatic(dataType: Double.self) { values in
        values.append(target)
    })
}
```

**Pitfalls:**

- Turning off the zero baseline on a bar chart. Bar length is the value; a bar that does not start at zero misstates it.
- Using `modifyInferredDomain` to force an exact domain. Its result can still be extended by axis mark values and reversed; pass the exact domain directly.
- Narrowing a domain and expecting marks outside it to disappear. They are drawn beyond the plot area; clip it with `chartPlotStyle` (`plot-area.md`).

### `PlotDimensionScaleRange` - iOS 16.0+

**Use for:** the `range:` of a position scale. Written `.plotDimension`, it is the full width or height of the plot area, optionally padded in points:

- `.plotDimension(padding: 20)` keeps 20 points free at both ends.
- `.plotDimension(startPadding: 0, endPadding: 8)` pads each end separately. `chartYScale(range: .plotDimension(endPadding: 8))` keeps the highest points and their symbols off the top edge.

<!-- snippet: Scales/ScaleSnippets.swift#padding -->
```swift
func paddedPlot() -> some View {
    chartXScale(range: .plotDimension(padding: 20))
}
```

### `ScaleType` - iOS 16.0+

**Use for:** the `type:` of a scale, when the automatic choice (linear for numbers, date for dates, category for strings) is wrong for the data.

| Type | Use when | Since |
|---|---|---|
| `.linear` | The default for numbers | iOS 16.0 |
| `.log` | Values span several orders of magnitude and ratios matter. Values must be positive | iOS 16.0 |
| `.squareRoot`, `.power(exponent:)` | Large values should be compressed less severely than a log does | iOS 16.4 |
| `.symmetricLog`, `.symmetricLog(slopeAtZero:)` | Log-like compression for data that includes zero or negative values | iOS 16.4 |
| `.date`, `.category` | Forcing a date or a discrete scale | iOS 16.0 |

<!-- snippet: Scales/ScaleSnippets.swift#log -->
```swift
struct LogScaleChart: View {
    let stores: [DrinkSales]

    var body: some View {
        Chart(stores) { store in
            PointMark(x: .value("Followers", store.cups), y: .value("Store", store.drink))
        }
        .chartXScale(type: .log)
    }
}
```

On a linear scale the three smaller stores in that chart sit on top of each other at the left edge; on the log scale each power of ten gets equal space.

The 16.4 types need a gate on an iOS 16.0 target (`availability/README.md`, *Gate a chart modifier*).

## Colour and symbol scales

A `by:` encoding on a mark (`mark-modifiers.md`) creates one of these scales.

### `chartForegroundStyleScale` - iOS 16.0+

**Use for:** choosing which colour each encoded value gets.

Four forms:

- A dictionary literal from value to style: `.chartForegroundStyleScale(["Small": .yellow, "Large": .brown])`. The written order becomes the legend order.
- A closure from value to style, when the mapping lives elsewhere: `.chartForegroundStyleScale { colorFor($0) }`. It must return a style for every value in the data: give it a non-optional fallback, and do not force-unwrap a dictionary lookup.
- A `Gradient` as the range, when the encoded value is a number: `.chartForegroundStyleScale(range: Gradient(colors: [.yellow, .red]))`. Example: `gallery/heat-map.md`.
- A `domain:` alone, listing every value the scale can have, in order: `.chartForegroundStyleScale(domain: ["Small", "Medium", "Large"])`. Colours stay automatic, but each value keeps its colour whichever values the data currently contains. Reordering this domain reassigns the automatic colours.
- A `domain:` with a `mapping:` closure, to set the legend's order and each value's colour separately: `.chartForegroundStyleScale(domain: sizes, mapping: { colour(for: $0) })`. Reordering the domain then moves only the legend; the stacking still follows the data.

**Fix the domain whenever the data can be filtered, or several charts show the same categories.** Colours are assigned in the order values appear, so without a fixed domain a filter that removes the first category hands its colour to the next one, and two charts of different subsets disagree about what blue means.

<!-- snippet: Dashboard/DashboardPatterns.swift#stable-colours -->
```swift
/// The colour of each size stays the same whichever sizes the filtered data happens to contain.
struct StableColourBars: View {
    let orders: [DrinkOrder]
    static let sizes = ["Small", "Medium", "Large"]

    var body: some View {
        Chart(orders) { order in
            BarMark(x: .value("Drink", order.drink), y: .value("Cups", order.cups))
                .foregroundStyle(by: .value("Size", order.size))
        }
        .chartForegroundStyleScale(domain: Self.sizes)
    }
}
```

Applied to a container, the modifier reaches every chart inside it: put one scale on the stack that holds two charts of the same categories and both use the same colours.

Example: `gallery/stacked-bar.md`.

**Pitfalls:**

- The automatic palette (blue, green, orange, purple, red, cyan, yellow, in data order) repeats after seven, and carries no meaning. When values have a colour people expect, set the scale, or the chart will mislabel them.
- Legend order follows the scale; stacking order follows the data. Keep them the same.
- A fixed domain keeps every listed value in the legend, including values the current data lacks.

### `chartSymbolScale` - iOS 16.0+

**Use for:** choosing the shape for each encoded value: `.chartSymbolScale(["Harbour": Circle().strokeBorder(lineWidth: 3), ...])`. Shapes are `ChartSymbolShape`s (`mark-configuration.md`). `chartSymbolScale(domain:)` fixes which value gets which automatic shape, as for colours.

Example: `gallery/multi-series-line.md`.

### `chartLineStyleScale` - iOS 16.0+

**Use for:** the dash pattern `lineStyle(by:)` gives each value. `chartLineStyleScale(domain:)` fixes which value gets which automatic pattern, so a series keeps its dashes when another is filtered out, as a fixed colour domain keeps its colour:

<!-- snippet: Scales/ScaleSnippets.swift#line-style-scale -->
```swift
/// Each store keeps its dash pattern whichever stores are drawn, as a fixed colour domain keeps colours.
struct StableLineStyles: View {
    let stores: [StoreSales]
    /// Every store the chart can show, in a fixed order.
    let allStores: [String]

    var body: some View {
        Chart(stores) { store in
            ForEach(store.days) { day in
                LineMark(x: .value("Day", day.day), y: .value("Cups", day.cups))
            }
            .foregroundStyle(by: .value("Store", store.store))
            .lineStyle(by: .value("Store", store.store))
        }
        .chartForegroundStyleScale(domain: allStores)
        .chartLineStyleScale(domain: allStores)
    }
}
```

### `chartSymbolSizeScale` - iOS 16.0+

**Use for:** the range of point sizes that `symbolSize(by:)` maps to: `.chartSymbolSizeScale(range: 20 ... 200)`, in square points of area. Set it whenever sizes are encoded; the default range is too large for most charts. Example in `mark-modifiers.md`.

Add a `domain:` as well, `.chartSymbolSizeScale(domain: 0 ... 1500, range: 2 ... 100)`, when the data can be filtered. A fixed domain makes a size mean the same value in every view.
