# Plot area and chart proxy

The plot area is the rectangle the marks are drawn in, inside the axes and legend. This file covers styling it, placing SwiftUI views against it, and `ChartProxy`, which exposes the chart's geometry and scales.

## Styling the plot area

### `chartPlotStyle` - iOS 16.0+

**Use for:** styling the plot area without affecting the axis labels around it. The closure is a view builder that receives the plot area as a view (a `ChartPlotContent`): add a background, border, frame or clip to it, or branch on state with `if` inside it. Labels drawn inside the plot are covered too: the category names of horizontal bars sit on the plot area's background, while the value labels stay outside. Example: `gallery/bar.md`.

`frame(height:)` on the plot area sets its size directly, with the axes and legend added around it. Use it to make the plot as tall as its content needs, such as a fixed height per row of a horizontal bar chart:

<!-- snippet: Sessions/SessionPatterns.swift#plot-height -->
```swift
/// The plot area grows with the number of rows, so every bar keeps a comfortable thickness.
struct RowSizedBars: View {
    let shares: [DrinkSales]

    var body: some View {
        Chart(shares) { share in
            BarMark(x: .value("Cups", share.cups), y: .value("Drink", share.drink))
        }
        .chartPlotStyle { plotArea in
            plotArea.frame(height: 44 * CGFloat(shares.count))
        }
    }
}
```

A plot taller than the chart's own frame is not clipped: it is drawn past the frame, over the views around it. Leave the chart's height free, and when the plot can outgrow the screen put the chart in a `ScrollView`; move the value axis to the top (`AxisMarks(position: .top)`) so its scale is in view where the scroll starts.

`aspectRatio(_:contentMode:)` on the plot area fixes its shape, whatever the chart's frame. Use it when distances on the two axes must be comparable: with equal domains and a ratio of 1, a circle is drawn round and a map keeps its proportions. With `.fit` the plot is centred in the chart's frame, the axes follow it, and the space left over stays empty. Example: `gallery/function-plot.md`.

Marks are not clipped to the plot area. When a scale's domain is narrower than the data, or a function plot runs off the scale, add `.chartPlotStyle { $0.clipped() }` to cut them off at its edge. Symbols on the domain's ends are cut in half with it, and so are annotations: a value label above the tallest bar was cut. Padding the scale's range, `chartXScale(range: .plotDimension(padding:))` or `chartYScale(range: .plotDimension(endPadding:))`, lies inside the clip and keeps both whole. The other scale's automatic domain still comes from every mark, including those cut off. **Clipping removes the axis titles:** on iOS, `.clipped()`, `.clipShape(Rectangle())` and `.mask(Rectangle())` inside `chartPlotStyle` all stopped `chartXAxisLabel` and `chartYAxisLabel` being drawn, in either order and in the closure form; their space stays reserved and the value labels stay. When a clipped chart needs a unit, put it in the heading or caption, or make the clip unnecessary: domains that hold every mark, and `domain:` on a function plot. Example under `LinePlot` in `vectorized-plots.md`.

## Views behind and in front of the plot

### `chartBackground` - iOS 16.0+

**Use for:** a SwiftUI view behind the marks that needs to know where the plot area is. The closure receives a `ChartProxy`. Use it for content positioned relative to the plot, such as text in the hole of a donut (`gallery/donut.md`): a sector chart draws its ring as the largest circle that fits the plot area, centred in it, so the hole's centre is the plot frame's centre. `chartOverlay` is the same in front of the marks. Neither is clipped to the chart: a path built from `plotFrame` on a scrolling chart is drawn its full width, past the chart and the screen, so add `.clipped()` to the `GeometryReader`.

**Prefer over** a `ZStack` around the chart: the chart's frame includes axes and legend, so centring on it misses the centre of the plot.

**Prefer a mark with an `annotation`** when the content belongs to a data value (`annotations.md`).

## Chart proxy

### `ChartProxy` - iOS 16.0+

**Use for:** reading the chart's geometry and scales from inside `chartBackground`, `chartOverlay`, `chartGesture` and a custom scroll behaviour. It converts between positions on screen and values in the data, in both directions.

| Job | Members | Since |
|---|---|---|
| Where is the plot area? | `plotFrame`, `plotSize` | iOS 17.0. With a deployment target below that, use `plotAreaFrame` and `plotAreaSize`, which are deprecated from iOS 17 and not optional (`availability/README.md`) |
| Where is the visible part of a scrolling plot? | `plotContainerFrame` | iOS 17.0 |
| Which value is at this position? | `value(atX:as:)`, `value(atY:as:)`, `value(at:as:)` | iOS 16.0 |
| Where is this value drawn? | `position(forX:)`, `position(forY:)`, `position(for:)` | iOS 16.0 |
| Where is this category's band? | `positionRange(forX:)`, `positionRange(forY:)`, `positionRange(for:)` | iOS 16.0 |
| Sectors: which angle, which value? | `angle(at:)`, `value(atAngle:as:)` | iOS 17.0 |
| Set the chart's selection from a gesture | `selectXValue(at:)`, `selectYValue(at:)`, `selectXRange(from:to:)`, `selectYRange(from:to:)`, `selectAngleValue(at:)` | iOS 17.0 |
| Which colour, symbol, size or line style did the chart give this value? | `foregroundStyle(for:)`, `symbol(for:)`, `symbolSize(for:)`, `lineStyle(for:)` | iOS 16.4 |
| Which values are on a scale? | `xDomain(dataType:)`, `yDomain(dataType:)`, `foregroundStyleDomain(dataType:)` and the symbol, size and line-style equivalents | iOS 16.4 |

**Positions are relative to the plot area, not to the overlay or the chart.** The plot frames are anchors: resolve one in a `GeometryReader` with `geometry[anchor]` to get its rectangle. Then subtract its origin from the location of a gesture in an overlay (or one measured in a named coordinate space) before asking for a value, and add it to a position before placing a view. A gesture given to `chartGesture` is different: its location is already measured from the plot area, so pass it straight to the proxy (pressed on the iOS simulator in left to right and right to left, with the plot starting 20 points in); `selectXValue(at:)` also takes plot-area positions. With the default value axis on the trailing edge the plot starts at the overlay's origin, so leaving the origin out goes unnoticed until the axis moves to the leading edge. Ask a category scale for its own type: `value(atX: x, as: String.self)` for names.

**Right to left.** The resolved plot frame, the proxy's positions and a gesture's location all still count x from the left edge; only `.position(x:)` counts from the right. Reading a value needs no change (`location.x - plot.minX`). Placing a view does: `.position(x: geometry.size.width - (plot.minX + x), y: plot.minY + y)`, with `x` from the proxy. The snippet below is written for left to right; written as it is, its star lands over the mirrored category.

<!-- snippet: Interaction/ProxySnippets.swift#position-overlay -->
```swift
/// A SwiftUI view placed at a data value, when a mark's annotation cannot do the job.
/// Written for an iOS 16 target, where `plotAreaFrame` is the property to use; from iOS 17 it is `plotFrame`.
struct BadgeAtValue: View {
    let sales: [DrinkSales]
    let drink: String

    var body: some View {
        Chart(sales) { item in
            BarMark(x: .value("Drink", item.drink), y: .value("Cups", item.cups))
        }
        .chartOverlay { proxy in
            GeometryReader { geometry in
                // The band of a category is a range of positions; a number or date is a single point.
                if let band = proxy.positionRange(forX: drink) {
                    let plot = geometry[proxy.plotAreaFrame]
                    Image(systemName: "star.fill")
                        .foregroundStyle(.yellow)
                        .position(x: plot.minX + (band.lowerBound + band.upperBound) / 2, y: plot.minY + 12)
                }
            }
        }
    }
}
```

Every lookup returns an optional: `nil` when the scale is unavailable, or the position or value is not valid for it. For a number or a date, `positionRange` is a single point; for a category it is the band.

In a scrolling chart, `plotFrame` covers the whole plot, including what is scrolled out of view, and `plotContainerFrame` only the visible part, which stays still while the chart scrolls. The value axis's domain covers all the data, not only the part in view:

<!-- snippet: Interaction/ProxySnippets.swift#container-frame -->
```swift
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
extension View {
    /// Outlines the visible part of the plot. It stays put while the chart scrolls beneath it.
    func plotContainerBorder() -> some View {
        chartOverlay { proxy in
            GeometryReader { geometry in
                if let frame = proxy.plotContainerFrame {
                    Path(geometry[frame])
                        .stroke(.secondary, lineWidth: 1)
                        .allowsHitTesting(false)
                }
            }
        }
    }
}
```

The scale lookups let the app draw things the chart would otherwise draw itself, in the chart's own colours: a custom legend is in `legend.md`. Gestures built on the proxy are in `interaction.md`.

## Pitfalls

- `plotFrame` and `plotContainerFrame` are optional; unwrap them.
- Placing a view from the proxy without mirroring x in right to left.
- Converting a position that lies outside the plot area. A number or date scale still answers, with a value beyond the axis; a category scale returns `nil`. Check the position against the plot's size first.
- Passing an overlay gesture's location to the proxy without subtracting the plot area's origin (a `chartGesture` location needs no subtraction). Values come out shifted whenever the plot area does not start at the overlay's origin, as with an axis on the leading edge.
- Using `plotFrame` to draw a border or a fixed overlay on a scrolling chart. It moves with the content; use `plotContainerFrame`.
- Using the proxy to place a label that belongs to a data value. An `annotation` does that without any geometry (`annotations.md`).
