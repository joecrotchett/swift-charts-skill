# Mark modifiers

The modifiers every mark shares. They come from the `ChartContent` protocol, behave like SwiftUI view modifiers but act on chart content, and can be applied to a single mark, a `ForEach`, or a `Plot`. All forms are listed in `availability/`.

## Styling marks

### `foregroundStyle(_:)` - iOS 16.0+

**Use for:** painting a mark with a fixed colour, gradient or other `ShapeStyle`.

**Do not use the hierarchical styles on marks or axis marks.** `.primary` draws in the chart's blue, and `.secondary`, `.tertiary` and `.quaternary` in paler tints of it, not in black and greys. Use a colour: `Color.primary`, `Color.secondary` or `.gray`, with an opacity for lighter shades. This applies to marks and axis marks; text and shapes in ordinary SwiftUI views, including the views inside an annotation, take `.secondary` as usual.

A `foregroundStyle` applied to the `Chart` view itself colours every mark that has no style of its own, which is the shortest way to tint a single-series chart. `tint(_:)` does not do it: unstyled marks stay blue.

**Prefer `foregroundStyle(by:)`** when the colour stands for a data value. Use the fixed form for a single series, for a mark that is not data (a threshold rule), and for emphasising one element by a condition:

<!-- snippet: Marks/MarkModifiers.swift#highlight -->
```swift
Chart(sales) { item in
    BarMark(
        x: .value("Drink", item.drink),
        y: .value("Cups", item.cups)
    )
    .foregroundStyle(item.drink == highlighted ? Color.red : Color.blue)
}
```

A fixed style produces no legend.

### `lineStyle(_:)` - iOS 16.0+

**Use for:** the stroke of line and rule marks: width, dash, cap.

The `StrokeStyle` you pass replaces the mark's default line width and line cap, not only the properties you set. Always set the `lineWidth`, and set `lineCap` and `lineJoin` too where a line's default ends and joins should be kept. A rule needs only the width.

### Other styling modifiers

| Modifier | iOS | Does |
|---|---|---|
| `opacity(_:)` | 16.0 | Fades a mark. Used to de-emphasise all but one element, or to keep overlapping marks visible. The legend's swatches stay at full strength |
| `cornerRadius(_:style:)` | 16.0 | Rounds bars, rectangles and sectors. A bar rising from the baseline is rounded at its free end only |
| `interpolationMethod(_:)` | 16.0 | How line and area marks connect their points, for example `.catmullRom` or `.stepEnd` |
| `blur(radius:)` | 16.4 | Gaussian blur |
| `shadow(color:radius:x:y:)` | 16.4 | Drop shadow |

`blur` and `shadow` arrived in iOS 16.4, so they are gated on an iOS 16.0 target (`availability/README.md`, *Gate a mark modifier*).

## Positioning marks

### `offset` - iOS 16.0+

**Use for:** moving or stretching a mark by a distance in points, independent of the data.

- `offset(x:y:)` moves the whole mark.
- `offset(x:yStart:yEnd:)`, `offset(xStart:xEnd:y:)` and `offset(xStart:xEnd:yStart:yEnd:)` move one end, for marks that have a start and an end such as bars and rules. `offset(yStart: -10)` extends a vertical rule 10 points past the top of the plot (`gallery/selection-line.md`).

### `alignsMarkStylesWithPlotArea` - iOS 16.0+

**Use for:** making a gradient (or any style with geometry) span the whole plot area instead of each mark.

Without it, every bar is filled with the full gradient, so a short bar and a tall bar both run from the first colour to the last. With it, the gradient is laid across the plot area and each mark shows the part it covers, so colour corresponds to height.

<!-- snippet: Marks/MarkModifiers.swift#gradient -->
```swift
Chart(sales) { item in
    BarMark(
        x: .value("Drink", item.drink),
        y: .value("Cups", item.cups)
    )
    .foregroundStyle(.linearGradient(colors: [.yellow, .red], startPoint: .bottom, endPoint: .top))
    .alignsMarkStylesWithPlotArea(alignsWithPlotArea)
}
```

## Setting symbol appearance

Symbols are the shapes drawn for `PointMark`s and at the points of a `LineMark`.

| Modifier | iOS | Does |
|---|---|---|
| `symbol(_:)` | 16.0 | A fixed shape: `.circle`, `.square`, `.triangle`, `.diamond`, `.pentagon`, `.plus`, `.cross`, `.asterisk`, or a custom `ChartSymbolShape` (`mark-configuration.md`) |
| `symbol(symbol:)` | 16.0 | Any SwiftUI view as the symbol. Drawn as written: `symbolSize(_:)`, `symbolSize(by:)` and `foregroundStyle(by:)` do not reach it (their legends still appear); size the view itself with a `frame` |
| `symbolSize(_:)` | 16.0 | Size, as an area in square points (`symbolSize(100)` is a 10 by 10 point square's worth) or as a `CGSize` bounding box |

A view as a symbol:

<!-- snippet: Marks/MarkModifiers.swift#viewsymbol -->
```swift
Chart(beans) { bean in
    PointMark(
        x: .value("Acidity", bean.acidity),
        y: .value("Body", bean.body)
    )
    .symbol {
        Image(systemName: "leaf.fill")
            .font(.caption)
            .foregroundStyle(.green)
    }
}
```

## Encoding data into mark characteristics

A `by:` modifier lets a data property decide a characteristic of the mark. Each one creates a scale (configured in `scales.md`), and each except `position(by:)` adds to the legend (`legend.md`).

### `foregroundStyle(by:)` - iOS 16.0+

**Use for:** letting a data property decide each mark's colour.

**Prefer over** `foregroundStyle(_:)` whenever colour means something. Encoding a category gives three things at once: marks are coloured per distinct value, a legend is added, and stacked bars or areas show their segments.

Encoding a number instead shades every mark along one gradient and adds a gradient legend. That is how a heat map is coloured (`gallery/heat-map.md`).

Choose the colours with `chartForegroundStyleScale` (`scales.md`).

### `position(by:axis:span:)` - iOS 16.0+

**Use for:** placing marks that share a category side by side instead of stacking them: a grouped bar chart. `axis:` is the direction to spread along, and `span:` sets how much of the band the group uses. By default a group leaves room either side; `span: .ratio(1)` fills the band, and the bars inside a group still keep a gap between them.

Recipe: `gallery/grouped-bar.md`.

### `symbol(by:)` - iOS 16.0+

**Use for:** giving each series its own point shape. On a `LineMark` it adds a marker at every point and separates the series (`composition.md`, *Series*).

The value must be categorical, such as a `String`. Use it with `foregroundStyle(by:)` on the same property so series differ by both colour and shape. Choose the shapes with `chartSymbolScale` (`scales.md`).

### `symbolSize(by:)` - iOS 16.0+

**Use for:** a third quantity on a scatter plot, shown as the size of each point (a bubble chart).

Always set the size range with `chartSymbolSizeScale(range:)` (`scales.md`). The default range is large enough that points on a small chart merge into a blob. It also takes a category, `symbolSize(by: .value("Store", store))`: give each category its size with the mapping form, `.chartSymbolSizeScale(["Harbour": 30, "Station": 300])`, which draws a legend of the sizes.

<!-- snippet: Marks/MarkModifiers.swift#sizescale -->
```swift
Chart(beans) { bean in
    PointMark(
        x: .value("Acidity", bean.acidity),
        y: .value("Body", bean.body)
    )
    .symbolSize(by: .value("Sweetness", bean.sweetness))
}
.chartSymbolSizeScale(range: 20 ... 200)
```

### `lineStyle(by:)` - iOS 16.0+

**Use for:** telling series apart by dash pattern (solid, then dashed) instead of, or as well as, colour. The legend shows a sample of each line.

<!-- snippet: Marks/MarkModifiers.swift#linestyle -->
```swift
Chart {
    ForEach(stores) { store in
        ForEach(store.days) { item in
            LineMark(
                x: .value("Day", item.day, unit: .day),
                y: .value("Cups", item.cups)
            )
        }
        .lineStyle(by: .value("Store", store.store))
    }
}
```

## Annotating marks

`annotation` attaches a SwiftUI view to a mark. See `annotations.md`.

## Layering chart content

### `zIndex` - iOS 17.0+

**Use for:** controlling which of two overlapping marks is in front. The default is 0; a negative value puts a mark behind the rest, wherever it is written in the chart. Example: `gallery/selection-line.md`.

## Masking and clipping

| Modifier | iOS | Does |
|---|---|---|
| `clipShape(_:style:)` | 16.0 | Clips each mark to a shape that fills the mark's frame. `clipShape(Capsule())` on a bar gives a pill with both ends rounded |
| `mask(content:)` | 16.0 | Masks a mark with the alpha channel of other chart content |

<!-- snippet: Marks/MarkModifiers.swift#shape -->
```swift
Chart(sales) { item in
    BarMark(
        x: .value("Drink", item.drink),
        y: .value("Cups", item.cups),
        width: .ratio(0.4)
    )
    .clipShape(Capsule())
}
```

## Configuring accessibility

`accessibilityLabel`, `accessibilityValue`, `accessibilityHidden` and `accessibilityIdentifier` exist on every mark, but a label or value is reliably read only when applied to a `Plot` around the mark. See `accessibility.md` before using them.

## Pitfalls

- Styling by hand and expecting a legend. Only a `by:` style encoding produces one.
- Repeating a modifier on every mark when it could be applied once to the `ForEach`.
