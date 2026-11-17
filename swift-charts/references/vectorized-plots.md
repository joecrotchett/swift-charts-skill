# Vectorized plots

Plotting a whole collection with one piece of chart content, and plotting mathematical functions. All of it is iOS 18.0+ (macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0).

## Marks and plots

A mark draws one data element, so a chart of marks is a `ForEach` that creates one mark per element. A plot takes the collection itself and draws all of it. Every mark has a plot that draws the same thing:

| Mark | Plot | From a collection | From a function |
|---|---|---|---|
| `AreaMark` | `AreaPlot` | Yes | Yes |
| `LineMark` | `LinePlot` | Yes | Yes, including parametric |
| `PointMark` | `PointPlot` | Yes | |
| `RectangleMark` | `RectanglePlot` | Yes | |
| `RuleMark` | `RulePlot` | Yes | |
| `BarMark` | `BarPlot` | Yes | |
| `SectorMark` | `SectorPlot` | Yes | |

Each plot's initializers mirror its mark's (`marks.md`), with the collection as the first argument and key paths in place of values.

**Use a plot** for a larger data set whose points are all styled the same way, from the same properties, and to draw a function. A plot treats the collection as one thing and can process it in parallel.

**Use marks** when there are fewer points and they need individual treatment: different mark types or modifiers per element, or layering between elements with `zIndex`. Marks are also the only choice on an iOS 17 or earlier target.

Plots and marks mix freely in one chart: a `RuleMark` threshold over a `BarPlot`, or a `LinePlot` trend line over a `PointPlot`.

## Plotting a collection

### `PlottableProjection` - iOS 18.0+

**Use for:** telling a plot which property of each element to use. It is what `.value(_:_:)` produces inside a plot, where the second argument is a key path instead of a value: `.value("Acidity", \.acidity)`.

| Form | Meaning |
|---|---|
| `.value("Cups", \.cups)` | One property |
| `.value("Time", \.start, \.end)` | An interval between two properties, in a single parameter |
| `.value("Day", \.day, unit: .day)` | A date property with a calendar unit, as for marks (`data.md`) |
| `.value("Series", "Harbour")` | The same constant for every element |

The collection must be a `RandomAccessCollection`, such as an array.

<!-- snippet: Plots/VectorizedPlots.swift#collection -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct BeanPointPlot: View {
    let beans: [Bean]

    var body: some View {
        Chart {
            PointPlot(
                beans,
                x: .value("Acidity", \.acidity),
                y: .value("Body", \.body)
            )
            // Key-path modifiers first...
            .foregroundStyle(by: .value("Origin", \.origin))
            .symbolSize(by: .value("Sweetness", \.sweetness))
            // ...then modifiers that take a plain value.
            .opacity(0.7)
        }
        .chartSymbolSizeScale(range: 10 ... 120)
    }
}
```

### `VectorizedChartContent` - iOS 18.0+

The protocol every collection plot conforms to, and where the key-path modifiers live. They read each element's styling from the element:

| Modifier | Takes |
|---|---|
| `foregroundStyle(by:)`, `symbol(by:)`, `symbolSize(by:)`, `lineStyle(by:)`, `position(by:axis:span:)` | A `PlottableProjection`: `.value("Origin", \.origin)` |
| `foregroundStyle(_:)` | A key path to a `ShapeStyle` property |
| `opacity(_:)` | A key path to a `CGFloat` from 0 to 1 |
| `lineStyle(_:)` | A key path to a `StrokeStyle` |
| `symbolSize(_:)` | A key path to a `CGFloat` area in square points, or to a `CGSize` |
| `accessibilityLabel(_:)`, `accessibilityValue(_:)`, `accessibilityHidden(_:)`, `accessibilityIdentifier(_:)` | A key path to the text, `Bool` or `String`. The label and value are not read by VoiceOver, on a few points or many: a plot's elements keep their automatic wording, so describe a plot through its chart descriptor (`audio-graph.md`) |

A plot is also `ChartContent`, so every modifier in `mark-modifiers.md` works on it with a plain value, applied to the whole plot: `cornerRadius(4)`, `opacity(0.7)`, `interpolationMethod(.catmullRom)`.

**Apply key-path modifiers first, plain-value modifiers after.** A plain-value modifier returns ordinary chart content, and the key-path modifiers no longer exist on the result.

**Pitfalls:**

- A key-path modifier after a plain-value one does not compile, and the error does not say why: "instance method 'foregroundStyle(by:)' requires that 'KeyPath<Root, Value>' conform to 'Plottable'". Move the key-path modifier up.
- A colour stored on each element is `foregroundStyle(\.colour)`, with no `by:`. `foregroundStyle(by:)` takes a `.value(...)` and makes a scale and a legend.
- `symbolSize(by:)` needs `chartSymbolSizeScale(range:)`, as with marks (`mark-modifiers.md`).

### `MarkDimensions` - iOS 18.0+

**Use for:** the `width:` and `height:` of `BarPlot` and `RectanglePlot` and the radii of `SectorPlot`. It is the plot counterpart of `MarkDimension` (`mark-configuration.md`) with the same cases, and each case also accepts a key path so the size can come from the element: `.fixed(\.barWidth)`, `.ratio(\.share)`.

### Intervals and bars

An interval takes a single parameter with two key paths:

<!-- snippet: Plots/VectorizedPlots.swift#intervals -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct ShiftRectanglePlot: View {
    let shifts: [Shift]

    var body: some View {
        Chart {
            RectanglePlot(
                shifts,
                x: .value("Time", \.start, \.end),
                y: .value("Barista", \.barista)
            )
            .foregroundStyle(by: .value("Barista", \.barista))
            .cornerRadius(4)
        }
    }
}
```

`BarPlot` stacks, and takes `stacking:`, exactly as `BarMark` does:

<!-- snippet: Plots/VectorizedPlots.swift#bars -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct OrdersBarPlot: View {
    let orders: [DrinkOrder]

    var body: some View {
        Chart {
            BarPlot(
                orders,
                x: .value("Drink", \.drink),
                y: .value("Cups", \.cups)
            )
            .foregroundStyle(by: .value("Size", \.size))
        }
    }
}
```

### Supporting an older OS

Gate the plot inside the chart and give earlier releases the equivalent marks:

<!-- snippet: Plots/VectorizedPlots.swift#fallback -->
```swift
struct DailySalesLine: View {
    let days: [DailySales]

    var body: some View {
        Chart {
            if #available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *) {
                LinePlot(days, x: .value("Day", \.day), y: .value("Cups", \.cups))
            } else {
                ForEach(days) { day in
                    LineMark(x: .value("Day", day.day), y: .value("Cups", day.cups))
                }
            }
        }
    }
}
```

## Keeping a large plot fast

- **Key paths to stored properties.** A stored property is read at a constant memory offset; a computed one runs its getter for every point. Compute derived values (a projected coordinate, a ratio) once in the element's initializer and store them. Keep any other computation out of rendering too.
- **Group the collection by style.** When colour comes from a category, order the data by that category, so the style alternates as few times as possible along the collection.
- **State what is already known.** If the distinct styles are few and known, or the overall bounds of the chart are, set the scale domains explicitly: a range such as `0 ... 500` or the list of categories, not `.automatic`, which still has the framework work the domain out.
- **Small symbols** for a dense scatter: `symbolSize(4)`.
- **Animate by changing what a key path points at.** Switching the property plotted on x inside `withAnimation`, or through a binding with `.animation(...)`, moves every point of a `PointPlot` to its new place.
- **Fix the colour and size scale domains** when a selection filters the data, so nothing changes colour or size as the subset changes (`scales.md`).

**Below iOS 18 there are no plots,** and nothing makes a very large number of individual marks fast. Draw fewer marks: bin or aggregate the data first (`data.md`). A dense scatter becomes a two-dimensional histogram, one rectangle per cell coloured by count (`gallery/heat-map.md`), which shows the same density with a few hundred marks. Since that is a different chart form, gate the whole chart, not a mark inside it (`availability/README.md`, *Gate a whole chart*).

## Plotting functions

### `LinePlot` - iOS 18.0+

**Use for:** graphing a function, besides plotting a collection as a line.

- `LinePlot(x:y:domain:function:)` graphs y = f(x). `x` and `y` are the labels for the two values; the closure takes x and returns y.
- `LinePlot(x:y:t:domain:function:)` graphs a parametric curve: the closure takes `t` and returns `(x:, y:)`. Its `domain` is required and must be finite.

The labels are not decoration: VoiceOver's description of the chart names the axes with them ("The x axis is capacity density. The y axis is probability").

The curve is drawn across the chart's x scale. Set that scale with `chartXScale(domain:)`, and normally `chartYScale(domain:)` too, so the chart shows the part of the function that matters. `domain:` on the plot limits the function to part of the x scale.

Several function plots in one chart are told apart with `foregroundStyle(by:)` and a constant value, which also gives them a legend. Code: `gallery/function-plot.md`.

A function plot over a `PointPlot` draws a trend line. The framework does not fit the line; compute the coefficients in the model and pass the range of the data as `domain:`, so the line stops where the points do.

<!-- snippet: Plots/FunctionPlots.swift#trend -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct ScatterWithTrendLine: View {
    let beans: [Bean]
    /// The fitted line, computed by the model: body = intercept + slope × acidity.
    let intercept: Double
    let slope: Double
    /// The range of acidity in the data, so the line stops where the points do.
    let acidityRange: ClosedRange<Double>

    var body: some View {
        Chart {
            PointPlot(beans, x: .value("Acidity", \.acidity), y: .value("Body", \.body))
                .opacity(0.5)
            LinePlot(x: "Acidity", y: "Trend", domain: acidityRange) { [intercept, slope] acidity in
                intercept + slope * acidity
            }
            .foregroundStyle(Color.primary)
        }
    }
}
```

**Pitfalls:**

- Where the function is undefined, return `Double.nan`; where it is infinite, `Double.infinity`. Guard any input that would trap and return `.nan` for it.
- A curve that leaves the y scale's domain is drawn outside the plot area, over the axis labels and beyond the chart's frame. Clip the plot area (`plot-area.md`):

<!-- snippet: Plots/FunctionPlots.swift#undefined -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct ReciprocalPlot: View {
    var body: some View {
        Chart {
            LinePlot(x: "x", y: "1/x") { x in
                x == 0 ? .nan : 1 / x
            }
        }
        .chartXScale(domain: -3 ... 3)
        .chartYScale(domain: -5 ... 5)
        // The curve runs far past the y domain near zero; keep it inside the plot area.
        .chartPlotStyle { plotArea in
            plotArea.clipped()
        }
    }
}
```

- With no x scale domain and no other data in the chart, the function is drawn from -10 to 10. Set the domain.
- The closure is `@Sendable`. Capture the values it needs (`[intercept, slope]`), not `self`.

### `AreaPlot` - iOS 18.0+

**Use for:** filling the area under a function, or between two functions, besides plotting a collection as an area.

- `AreaPlot(x:y:domain:function:)` fills between zero and f(x).
- `AreaPlot(x:yStart:yEnd:domain:function:)` fills between two functions: the closure returns `(yStart:, yEnd:)`.

<!-- snippet: Plots/FunctionPlots.swift#area -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct FunctionAreaPlot: View {
    var body: some View {
        Chart {
            AreaPlot(x: "x", yStart: "x", yEnd: "x² − 1") { x in
                (yStart: x, yEnd: x * x - 1)
            }
            .opacity(0.4)
        }
        .chartXScale(domain: -2 ... 2)
        .chartYScale(domain: -4 ... 4)
    }
}
```

The same pitfalls as `LinePlot` apply. Recipe for both: `gallery/function-plot.md`.

## Pitfalls

- Naming the opaque content types (`VectorizedBarPlotContent`, `FunctionLinePlotContent` and the rest). They are implementation types; write `BarPlot`, `LinePlot`.
