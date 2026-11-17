# Plot area and chart proxy

The plot area is the rectangle the marks are drawn in, inside the axes and legend. This file covers styling it, placing SwiftUI views against it, and `ChartProxy`, which exposes the chart's geometry and scales.

## Styling the plot area

### `chartPlotStyle` - iOS 16.0+

**Use for:** styling the plot area without affecting the axis labels around it. The closure receives the plot area as a view (a `ChartPlotContent`): add a background, border, frame or clip to it. Example: `gallery/bar.md`.

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

`aspectRatio(_:contentMode:)` on the plot area fixes its shape, whatever the chart's frame. Use it when distances on the two axes must be comparable: with equal domains and a ratio of 1, a circle is drawn round and a map keeps its proportions. Example: `gallery/function-plot.md`.

Marks are not clipped to the plot area. When a scale's domain is narrower than the data, or a function plot runs off the scale, add `.chartPlotStyle { $0.clipped() }` to cut them off at its edge. Example under `LinePlot` in `vectorized-plots.md`.

## Views behind and in front of the plot

### `chartBackground` - iOS 16.0+

**Use for:** a SwiftUI view behind the marks that needs to know where the plot area is. The closure receives a `ChartProxy`. Use it for content positioned relative to the plot, such as text in the hole of a donut (`gallery/donut.md`). `chartOverlay` is the same in front of the marks.

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

**Positions are relative to the plot area, not to the overlay or the chart.** The plot frames are anchors: resolve one in a `GeometryReader` with `geometry[anchor]` to get its rectangle. Then subtract its origin from a gesture's location before asking for a value, and add it to a position before placing a view.

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

In a scrolling chart, `plotFrame` covers the whole plot, including what is scrolled out of view, and `plotContainerFrame` only the visible part, which stays still while the chart scrolls:

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
- Converting a position that lies outside the plot area. A number or date scale still answers, with a value beyond the axis; a category scale returns `nil`. Check the position against the plot's size first.
- Passing a gesture's location to the proxy without subtracting the plot area's origin. Values come out shifted whenever the plot area does not start at the overlay's origin, as with an axis on the leading edge.
- Using `plotFrame` to draw a border or a fixed overlay on a scrolling chart. It moves with the content; use `plotContainerFrame`.
- Using the proxy to place a label that belongs to a data value. An `annotation` does that without any geometry (`annotations.md`).
