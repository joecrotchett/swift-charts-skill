# Legend

The legend explains the colour, symbol and other non-position scales. It appears when a mark uses a `by:` style encoding (`mark-modifiers.md`); fixed colours and `position(by:)` produce none. A scale modifier given a `domain:` also draws a legend of that domain, even when no mark is encoded, so do not set one on a chart with nothing to explain. Its entries, order and swatches come from the scales (`scales.md`).

### `chartLegend` - iOS 16.0+

**Use for:** hiding or placing the legend.

- `chartLegend(.hidden)` when the chart is compact or the series are labelled some other way. Something else must then identify the series.
- `chartLegend(position:alignment:spacing:)` to move or align it, for example `chartLegend(alignment: .center, spacing: 18)` under a donut.

By default the legend sits below the chart at the leading edge, with a circle swatch per entry, or the series' own symbol when `symbol(by:)` is used.

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

On iOS 16.0 to 16.3 those lookups do not exist: set the colours explicitly with `chartForegroundStyleScale` and build the legend from the same table.

## Pitfalls

- Hiding the legend without another way to tell the series apart.
- Expecting the legend to follow data order when a colour scale is written in a different order (`scales.md`).
