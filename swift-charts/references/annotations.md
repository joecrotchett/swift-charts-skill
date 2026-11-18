# Annotations

Attaching labels and views to marks.

## Annotating marks

### `annotation` - iOS 16.0+

**Use for:** a SwiftUI view attached to a mark: a value label, a callout, a selection popover. The view is positioned by the chart, so it follows the mark when data or size changes.

**Prefer over** an overlay with hand-computed positions.

`annotation(position:alignment:spacing:content:)`. `position` says which side of the mark the view goes on, `alignment` where along that side, and `spacing` how far away.

The most common use is a value label on each bar:

<!-- snippet: Annotations/AnnotationSnippets.swift#value-labels -->
```swift
struct LabelledBarChart: View {
    let sales: [DrinkSales]

    var body: some View {
        Chart(sales) { item in
            BarMark(
                x: .value("Drink", item.drink),
                y: .value("Cups", item.cups)
            )
            .annotation(position: .top) {
                Text(item.cups, format: .number)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }
        }
    }
}
```

Good anchors besides the mark being labelled: a `RuleMark` for a label about a constant or a selected position; a `Plot` for one label spanning several marks.

Examples: `gallery/bar.md`, `gallery/range-bar.md`, `gallery/selection-line.md`.

### `AnnotationPosition` - iOS 16.0+

**Use for:** the `position:` of an annotation.

| Position | Places the view |
|---|---|
| `.top`, `.bottom`, `.leading`, `.trailing` | Outside the mark, centred on that edge. Beside a vertical bar the view has only the gap to the next bar: it is drawn under that bar, and beside the last bar it runs onto the value axis's labels |
| `.topLeading`, `.topTrailing`, `.bottomLeading`, `.bottomTrailing` | Outside the mark, diagonally off that corner, with the same crowding beside bars |
| `.overlay` | On top of the mark, centred. The view is offered the mark's width, so text in a narrow bar wraps, a digit to a line at large text sizes: give it `.lineLimit(1)` and a `.minimumScaleFactor`. On a sector that is the middle of its ring: halfway round its angle and halfway between the inner and outer radius |
| `.automatic` | The default. Above a vertical bar; other marks were not checked, so name a position when it matters |

`alignment:` then moves the view along the chosen side: `position: .top, alignment: .leading` sits above the mark with leading edges lined up. With `.overlay`, the alignment picks the spot inside the mark, which is how a label is put at the end of a bar, inside it:

<!-- snippet: Annotations/AnnotationSnippets.swift#inside-labels -->
```swift
struct InsideLabelledBars: View {
    let shares: [DrinkSales]

    var body: some View {
        Chart(shares) { share in
            BarMark(
                x: .value("Cups", share.cups),
                y: .value("Drink", share.drink)
            )
            .annotation(position: .overlay, alignment: .trailing) {
                Text(share.cups, format: .number)
                    .font(.caption)
                    .foregroundStyle(.white)
                    .padding(.trailing, 6)
            }
        }
        .chartYAxis {
            AxisMarks(preset: .aligned, position: .leading)
        }
    }
}
```

On horizontal bars the category name itself can be written this way, with the category axis hidden. Annotating at `.leading` and fitting to the plot (iOS 17.0+) pulls each name inside the start of its bar; the bars are drawn at reduced opacity so the text stays readable:

<!-- snippet: Dashboard/DashboardPatterns.swift#labels-on-bars -->
```swift
/// Category names written on the bars, in place of a category axis.
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct NamedBars: View {
    let shares: [DrinkSales]

    var body: some View {
        Chart(shares) { share in
            BarMark(x: .value("Cups", share.cups), y: .value("Drink", share.drink))
                .opacity(0.5)
                .cornerRadius(4)
                .annotation(position: .leading, alignment: .leading,
                            overflowResolution: .init(x: .fit(to: .plot))) {
                    Text(share.drink).padding(.leading, 4)
                }
        }
        .chartYAxis(.hidden)
    }
}
```

### `AnnotationContext` - iOS 16.0+

**Use for:** sizing an annotation from what it annotates. The content closure can take one; its `targetSize` is the size of the mark, or of the whole group when the annotation is on a `Plot`. It has been seen to be zero for an `.overlay` annotation on a `RectangleMark`, so check what it reports before sizing by it. Use it to make a label at least as wide as what it describes (`gallery/selection-line.md`).

### A label at the end of a line

A line has no single mark to annotate, and an annotation on every `LineMark` would label every point. Add a `PointMark` at the last value with a symbol size of zero and annotate that: nothing is drawn, and the label sits at the line's end. It replaces a legend for a few series.

<!-- snippet: Annotations/AnnotationSnippets.swift#end-label -->
```swift
/// A name at the end of each line, in place of a legend.
struct LinesLabelledAtTheirEnds: View {
    let stores: [StoreSales]

    var body: some View {
        Chart(stores) { store in
            ForEach(store.days) { day in
                LineMark(x: .value("Day", day.day), y: .value("Cups", day.cups), series: .value("Store", store.store))
            }
            if let last = store.days.last {
                // A point of no size at the last value: nothing is drawn, and the label has a mark to hang on.
                PointMark(x: .value("Day", last.day), y: .value("Cups", last.cups))
                    .symbolSize(0)
                    .accessibilityHidden(true)
                    .annotation(position: .trailing) {
                        Text(store.store).font(.caption)
                    }
            }
        }
    }
}
```

Lines that end close together get labels that overlap; nothing moves them apart for you.

## Positioning and overflow

### `AnnotationOverflowResolution` - iOS 17.0+

**Use for:** the `overflowResolution:` of an annotation, which decides what happens when the view would cross the edge of the chart. It takes a strategy for each axis: `.init(x: .fit(to: .chart), y: .disabled)`.

| Strategy | Result at the edge |
|---|---|
| `.automatic` | The default. On the last point of a line it extends the scale as `.padScale` does; name a strategy when the result matters |
| `.padScale` | Extends the scale past the data until the annotation fits. The marks shrink a little; the annotation stays centred on its mark. It cannot extend a domain you set: the annotation then goes past the edge as with `.disabled`. A date axis given `.stride(by:)` counts from the padded start, so weekly labels move off the data's weeks; give the axis an exact array of dates instead |
| `.fit(to: .plot)` | Slides the annotation inward to stay inside the plot area: sideways on x, down on y, where a label above a point at the top then covers its point. The scale is unchanged |
| `.fit(to: .chart)` | Slides it to stay inside the whole chart, so it may sit over the axis labels; on y it covers its point as `.fit(to: .plot)` does |
| `.disabled` | Leaves it where it is. It is cut off at the chart's edge, or drawn outside the chart |

<!-- snippet: Annotations/AnnotationSnippets.swift#overflow -->
```swift
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct EdgeAnnotation: View {
    let days: [DailySales]
    let strategy: AnnotationOverflowResolution.Strategy

    var body: some View {
        Chart {
            ForEach(days) { day in
                LineMark(x: .value("Day", day.day), y: .value("Cups", day.cups))
            }
            if let last = days.last {
                PointMark(x: .value("Day", last.day), y: .value("Cups", last.cups))
                    .annotation(
                        position: .top,
                        overflowResolution: .init(x: strategy, y: strategy)
                    ) {
                        Text("Latest: \(last.cups) cups")
                            .font(.caption)
                    }
            }
        }
    }
}
```

Choosing:

- A label on the last point of a line: the default or `.padScale`, so the label stays over its point.
- A popover that follows a selection: `.fit(to: .chart)` on x, so it slides inward at the edges; `.disabled` on y when it is meant to sit above the plot area.

## Pitfalls

- With overflow disabled, an annotation can draw outside the chart's frame and over neighbouring views. Reserve the space.
- On an iOS 16 target the `overflowResolution:` form is gated; the plain form has no edge handling.
- An annotation below a bar that rises from zero (`position: .bottom`) makes room for itself by extending the scale below zero by as much as the label needs, rounded to the axis's round numbers: bars of 0 to 164 cups got an axis from -100 to 200, a third of the plot empty, and in a shorter chart -200. The same happens at `.bottomLeading` and `.bottomTrailing`. Put the label above or inside the bar, or fix the scale's domain.
- An annotation's view is not an accessibility element. Anything written only there is invisible to VoiceOver; the label and value on the mark's `Plot` must carry it (`accessibility.md`).
