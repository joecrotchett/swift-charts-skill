# Legend

The legend explains the colour, symbol and other non-position scales. It appears when a mark uses a `by:` style encoding (`mark-modifiers.md`); fixed colours and `position(by:)` produce none. A scale modifier given a `domain:` also draws a legend of that domain, even when no mark is encoded, so do not set one on a chart with nothing to explain. Its entries, order and swatches come from the scales (`scales.md`).

### `chartLegend` - iOS 16.0+

**Use for:** hiding or placing the legend.

- `chartLegend(.hidden)` when the chart is compact or the series are labelled some other way. Something else must then identify the series, such as each line's name at its end (an `annotation` on the last point, `annotations.md`).
- `chartLegend(position:alignment:spacing:)` to move or align it, for example `chartLegend(alignment: .center, spacing: 18)` under a donut.

By default the legend sits below the chart at the leading edge, with a circle swatch per entry, or the series' own symbol when `symbol(by:)` is used. Two encodings of the same field, such as `foregroundStyle(by:)` and `lineStyle(by:)` on the café, make one entry per value, its swatch drawn in both (a dashed green line).

`alignment:` works along the side the legend is on: horizontal above or below the chart (`.leading`, `.center`, `.trailing`), vertical beside it (`.top`, `.center`, `.bottom`). A legend beside the chart takes its width from the plot, which matters at large text sizes.

When the legend comes and goes with state, keep one modifier chain and pass the state in: `.chartLegend(isShown ? .automatic : .hidden).chartLegend(position: position)`. `.hidden` wins whichever order the two are written in. An `if` around the chart's modifiers gives SwiftUI two different charts.

Examples: `gallery/donut.md`, `gallery/compact.md`.

### A legend of your own

When the design's legend cannot be produced by `chartLegend` (a different layout, values beside the names, a legend inside the plot), hide the automatic one and draw it yourself. `chartLegend(position:alignment:spacing:content:)` puts a view of your own in the legend's place instead, when only the content differs and the position is one the legend already offers. From iOS 16.4 the `ChartProxy` gives the scale's values and the colour the chart assigned to each, so the legend cannot drift from the marks:

<!-- snippet: Interaction/ProxySnippets.swift#custom-legend -->
```swift
/// A legend drawn by the app, using the colours the chart assigned (iOS 16.4).
@available(iOS 16.4, macOS 13.3, tvOS 16.4, watchOS 9.4, *)
struct ChartWithOwnLegend: View {
    let orders: [DrinkOrder]

    var body: some View {
        Chart(orders) { order in
            BarMark(x: .value("Drink", order.drink), y: .value("Cups", order.cups))
                .foregroundStyle(by: .value("Size", order.size))
        }
        .chartLegend(.hidden)
        .chartOverlay(alignment: .topLeading) { proxy in
            VStack(alignment: .leading, spacing: 2) {
                ForEach(proxy.foregroundStyleDomain(dataType: String.self), id: \.self) { size in
                    Label {
                        Text(size).font(.caption)
                    } icon: {
                        RoundedRectangle(cornerRadius: 2)
                            .fill(proxy.foregroundStyle(for: size) ?? AnyShapeStyle(.gray))
                            .frame(width: 10, height: 10)
                    }
                }
            }
            .padding(6)
        }
    }
}
```

`foregroundStyleDomain` lists the values in the colour scale's domain order, not the data's: sort the data in the same order if the legend must match a stack. A legend overlaid on the plot covers the marks under it; keep room for it with the scale's range, `chartXScale(range: .plotDimension(endPadding: width))` beside the marks, measured from the legend, or above them with the y scale at the largest text sizes.

On iOS 16.0 to 16.3 those lookups do not exist: set the colours explicitly with `chartForegroundStyleScale` and build the legend from the same table.

## Pitfalls

- Hiding the legend without another way to tell the series apart.
- Showing and hiding the legend with a conditional modifier around the chart instead of passing a `Visibility`.
- Expecting the legend to follow data order when a colour scale is written in a different order (`scales.md`).
