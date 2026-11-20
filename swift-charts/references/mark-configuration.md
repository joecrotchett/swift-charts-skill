# Mark configuration

The types that configure how marks are drawn: how they stack, how wide they are, how lines connect their points, and the shapes of point symbols. The marks themselves are in `marks.md`.

### `MarkStackingMethod` - iOS 16.0+

**Use for:** the `stacking:` parameter of `AreaMark` and `BarMark`, which decides how marks that share a position are combined.

| Method | Result |
|---|---|
| `.standard` | Stacked from zero. The default. Negative values stack downward from zero, so a chart with both grows in both directions (`gallery/stacked-bar.md`) |
| `.normalized` | Stacked and scaled so every stack fills the same height; the axis reads 0 to 100. Shows shares. With negative values each stack is scaled so its longer side, the rises or the falls, reaches 100 and the other side keeps its proportion beyond zero, so the axis runs from -100 to 100 and the figures are no longer shares of a whole |
| `.center` | Stacked, then shifted so each stack is centred on zero. A streamgraph for areas. The value axis then shows negative numbers that mean nothing; hide its labels |
| `.unstacked` | Not stacked; every mark starts at zero and they overlap. A later, longer bar or area hides an earlier one completely, so lower the opacity |

Pictured in `gallery/area.md`.

### `MarkDimension` - iOS 16.0+

**Use for:** the `width:` and `height:` of bars and rectangles, and the `span:` of `position(by:)`.

| Case | Size |
|---|---|
| `.automatic` | The default for the mark |
| `.ratio(0.6)` | A proportion of the scale's step: the band of a category, or one unit of a date plotted with `unit:`. `.ratio(1)` makes neighbours touch |
| `.inset(4)` | The step less 4 points on each side. Gaps stay the same width however many bars there are |
| `.fixed(8)` | 8 points, whatever the step. A bare number means the same: `width: 8`. Wider than the step, bars overlap and the outer ones run past the plot over the axis labels; clip the plot area (`plot-area.md`) or use a ratio |

How `.ratio` and `.inset` are measured for a sector's radii is under `SectorMark` in `marks.md`.

### `InterpolationMethod` - iOS 16.0+

**Use for:** the argument of `interpolationMethod(_:)`, which decides how line and area marks connect their points.

| Method | Draws | Use when |
|---|---|---|
| `.linear` | Straight segments. The default | The data is the points and nothing should be implied between them |
| `.catmullRom`, `.cardinal` | A smooth curve through every point | A softer look. The curve can rise above or dip below the data between points |
| `.monotone` | A smooth curve that never overshoots: it rises only where the data rises | A smooth line whose peaks and troughs must be real values |
| `.stepEnd` | Flat from each point until the next, then a vertical jump | A value that holds until it changes: a price, a setting, a status |
| `.stepStart` | A vertical jump at each point, then flat at the next point's value | Each value applies to the interval leading up to its point |
| `.stepCenter` | The jump halfway between points | Each value stands for a period centred on its point |

`.catmullRom` is `.catmullRom(alpha: 0.5)` and `.cardinal` is `.cardinal(tension: 0)`; the parameterised forms change that one value.

Apple's documentation says `.stepEnd` puts the data point "at the end of the step" and `.stepStart` "at the start": the step there is the vertical jump, so under `.stepEnd` each point sits at the top of the jump into it and then holds its value (rendered: `gallery/line-shapes.md`).

Pictured in `gallery/line-shapes.md`.

### `BasicChartSymbolShape` - iOS 16.0+

**Use for:** the built-in symbols: `.circle`, `.square`, `.triangle`, `.diamond`, `.pentagon`, `.plus`, `.cross`, `.asterisk`. The automatic `symbol(by:)` assignment uses seven of them; `.pentagon` appears only when asked for.

`strokeBorder(lineWidth:)` turns any of them into an outline, which keeps overlapping points readable:

<!-- snippet: Marks/MarkConfiguration.swift#outlined -->
```swift
struct OutlinedSymbols: View {
    let beans: [Bean]

    var body: some View {
        Chart(beans) { bean in
            PointMark(x: .value("Acidity", bean.acidity), y: .value("Body", bean.body))
                .symbol(.circle.strokeBorder(lineWidth: 2))
        }
    }
}
```

### `ChartSymbolShape` - iOS 16.0+

**Use for:** a symbol the framework does not provide. Conform a `Shape` to `ChartSymbolShape` and implement `perceptualUnitRect`, a rectangle in symbol units. The shape's path is drawn in that rectangle scaled by the symbol size, so a larger rectangle draws a larger symbol. `Circle` and every built-in symbol report `(-0.064, -0.064, 1.128, 1.128)`, a circle with the area of the unit square; a square path filling its rectangle matches that circle's area with `CGRect(x: 0, y: 0, width: 1, height: 1)`, and at 0.75 it draws visibly smaller (printed and rendered on macOS). Under main-actor default isolation mark the shape, and anything a symbol scale's mapping calls, `nonisolated`. Also conform to `InsettableShape` to get `strokeBorder`, which gives an outlined symbol instead of a filled one.

Example: `gallery/multi-series-line.md`.

### `AnyChartSymbolShape` - iOS 16.0+

**Use for:** returning different symbol shapes from one function or property. Filled and outlined symbols are different types, so a `switch` that returns both needs the type erased:

<!-- snippet: Marks/MarkConfiguration.swift#erased -->
```swift
func symbol(for origin: String) -> AnyChartSymbolShape {
    switch origin {
    case "Ethiopia": AnyChartSymbolShape(.diamond)
    case "Colombia": AnyChartSymbolShape(.square.strokeBorder(lineWidth: 2))
    default: AnyChartSymbolShape(.circle)
    }
}
```
