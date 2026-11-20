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

**Use a plot** for a larger data set whose points are all styled the same way, from the same properties, and to draw a function. A plot treats the collection as one thing and can process it in parallel. Do not promise a reader that it draws faster: in a single render on the iOS simulator (`ImageRenderer`, 1,000 to 20,000 points) a `PointPlot` and a `ForEach` of the same `PointMark`s took the same time, within 2%. Choose a plot for its shape, and measure before claiming speed.

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
| `accessibilityLabel(_:)`, `accessibilityValue(_:)`, `accessibilityHidden(_:)`, `accessibilityIdentifier(_:)` | A key path to the text, `Bool` or `String`. The label and value are not read by VoiceOver, on a few points or many (iOS): see *Accessibility*, below |

A plot is also `ChartContent`, so every modifier in `mark-modifiers.md` works on it with a plain value, applied to the whole plot: `cornerRadius(4)`, `opacity(0.7)`, `interpolationMethod(.catmullRom)`.

**Apply key-path modifiers first, plain-value modifiers after.** A plain-value modifier returns ordinary chart content, and the key-path modifiers no longer exist on the result.

**Pitfalls:**

- A key-path modifier after a plain-value one does not compile, and no error says why. What it says depends on the plain-value modifier (Xcode 27.0). After one with no key-path form, such as `symbol(.square)`, there are two errors on the key-path modifier's line: "instance method 'foregroundStyle(by:)' requires that 'KeyPath<Root, Value>' conform to 'Plottable'" and "cannot infer key path type from context; consider explicitly specifying a root type", whose suggestion does not help. After one that also has a key-path form (`opacity`, `symbolSize`, `lineStyle`, `foregroundStyle(_:)`), the compiler picks that form and the error lands on the plain-value line instead, reading as though that modifier wanted a key path: "cannot convert value of type 'Double' to expected argument type 'KeyPath<…, CGFloat>'" for `opacity(0.6)`, "no exact matches in call to instance method 'symbolSize'" for `symbolSize(40)`; with several, each line gets its own. In a longer expression the build may instead give up: "the compiler is unable to type-check this expression in reasonable time". Either way, move the key-path modifiers up.
- A stored key path works as well as a literal (`.value("Colour", choice.keyPath)`), so a control can switch what a modifier reads. All the choices for one modifier must share a value type. Switching the key path inside `withAnimation`, or through a binding with `.animation(...)`, moves every point of a `PointPlot` to its new place.
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

VoiceOver does not read an interval's start and end: on a date axis each rectangle's element is the one-hour stretch around its middle, then the category ("8:00 AM to 9:00 AM Tom" for a shift from 11:00 to 18:00), and, for a plain date, in the device's time zone (`accessibility-defaults.md`). With a few intervals, give each a labelled mark in a representation (*Accessibility*, below) in place of stretches.

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

### Accessibility

A plot's elements are the automatic ones, merged into stretches of the axis with bare numbers and no units, and plain dates in the device's time zone (`accessibility-defaults.md`); its key-path `accessibilityLabel` and `accessibilityValue` are not read. Two routes give VoiceOver units and the right times, both read on iOS. A labelled `Plot` around a `PointPlot` of each portion's elements is read, but in the order of the frames' tops, not along x (portions of a scatter came out 6–8, 8–10, 12–14, 10–12); the clear full-height band that puts marks in x order (`accessibility.md`, *One element per `Plot`*; read on iOS with `LineMark`s, stacked bars and one `PointMark` per day) has not been read with a `PointPlot`. Or put a representation on the chart: one labelled stretch of the axis per element, each spanning the plot's full height (a `RectangleMark` from the bottom of the y domain to the top), read left to right. The chart descriptor on the representation reaches the Audio Graph, and since the plot stays as drawn, marks and plots can share one representation. A function plot alone is simpler: a labelled `Plot` around a function `LinePlot` is one element with that label and value (iOS). Beside other elements it is read in the order of its frame's top (a curve over a histogram came between the first two bars), so a curve over data goes into the stretches' values instead.

<!-- snippet: Plots/VectorizedPlots.swift#spoken-portions -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct BeanPointPlotWithSpokenPortions: View {
    let beans: [Bean]
    /// Stretches of the acidity axis, one element each.
    let portions: [(start: Double, end: Double, count: Int)]

    var body: some View {
        Chart {
            PointPlot(beans, x: .value("Acidity", \.acidity), y: .value("Body", \.body))
        }
        .chartYScale(domain: 0 ... 10)
        .accessibilityRepresentation {
            Chart(portions, id: \.start) { portion in
                // Full height, so every element shares one top edge and is read along x.
                Plot {
                    RectangleMark(xStart: .value("Acidity", portion.start), xEnd: .value("Acidity", portion.end),
                                  yStart: .value("Body", 0), yEnd: .value("Body", 10))
                }
                .accessibilityLabel("Acidity \(portion.start.formatted()) to \(portion.end.formatted())")
                .accessibilityValue("\(portion.count) beans")
            }
            .chartYScale(domain: 0 ... 10)
            // The chart descriptor goes here, on the representation.
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

What costs time is the number of points. In single renders on the iOS simulator (`ImageRenderer`, a `PointPlot` coloured by category), 5,000 points took about 52 ms and 20,000 about 178 ms; none of the following changed that measurably, alone or all together (178 ms with all of them, 182 ms with none, inside the 5 to 13 ms spread between runs). Apple gives them as ways to keep a plot fast, and they may matter where a single render does not show it (live updates, scrolling, a device), so measure your own chart before promising a reader speed:

- **Key paths to stored properties**, not computed ones: a computed property runs its getter for every point.
- **The collection grouped by style**, so the colour changes as few times as possible along it.
- **Scale domains stated** where they are known, not `.automatic`.

Stating the domains also changes the picture, which is the better reason for it: an automatic y domain ended at 8 where the stated one ended at 10, and automatic colours went to the categories in the order they first appear in the data, so sorting the data reassigned them. Sorting also decides what is on top: sorted by drink, the last drink covered the others wherever points overlapped. State the domains for a stable axis and stable colours, and **fix the colour and size scale domains** when a selection filters the data, so nothing changes colour or size as the subset changes (`scales.md`).

**Small symbols** for a dense scatter (`symbolSize(4)`) keep crowded points apart; that is a reason of legibility, and their speed was not measured.

**What does cost less is fewer points,** on any release. Bin or aggregate the data first (`data.md`). A dense scatter becomes a two-dimensional histogram, one rectangle per cell coloured by count (`gallery/heat-map.md`), which shows the same density with a few hundred marks. Below iOS 18 there are no plots; since a heat map is a different chart form, gate the whole chart, not a mark inside it (`availability/README.md`, *Gate a whole chart*).

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

- Return `Double.nan` where the formula gives a number but the curve must not be drawn: outside the range a model is valid for, or a gap in a piecewise function. `.nan` and `.infinity` both leave a gap. A pole needs nothing: the plot never asks for y at the exact point (on iOS, about 3,000 calls for `1 / x` on -3 to 3, none at 0) and breaks the line there by itself, so a guard such as `x == 0 ? .nan : 1 / x` never runs. A pole where the sign flips is not joined either: `tan x` near π/2 was asked for 15,014 on one side and −3,522 on the other, and no line joined them. `log` and `sqrt` of a negative are already `.nan`, and `Double` division does not trap; guard only what would (integer arithmetic, an array index).
- A curve that leaves the y scale's domain is drawn outside the plot area, over the axis labels and beyond the chart's frame. Clip the plot area, which also removes the axis titles, so put the unit in the heading (`plot-area.md`):

<!-- snippet: Plots/FunctionPlots.swift#undefined -->
```swift
@available(iOS 18.0, macOS 15.0, tvOS 18.0, watchOS 11.0, visionOS 2.0, *)
struct ReciprocalPlot: View {
    var body: some View {
        Chart {
            // No guard at 0: the plot never asks for y there, and breaks the line at the pole by itself.
            LinePlot(x: "x", y: "1/x") { x in 1 / x }
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

Code, and the recipe for both: `gallery/function-plot.md`. The same pitfalls as `LinePlot` apply.

## Pitfalls

- Naming the opaque content types (`VectorizedBarPlotContent`, `FunctionLinePlotContent` and the rest). They are implementation types; write `BarPlot`, `LinePlot`.
