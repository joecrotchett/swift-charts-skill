# Marks

Each mark type and what it is for, and how mark positions are given. The types that configure marks (stacking, dimensions, interpolation, symbol shapes) are in `mark-configuration.md`; the modifiers shared by every mark are in `mark-modifiers.md`. The availability tables list initializers by their parameter labels, which hides the one-axis forms and which parameters are data values; the table below says which combinations exist.

## Choosing a mark

| To show | Use | Form |
|---|---|---|
| Amounts compared across categories | `BarMark` | Bar chart |
| A value changing over time or another continuous measure | `LineMark` | Line chart |
| A quantity over time where the filled amount, a total made of parts, or a range matters | `AreaMark` | Area, stacked area, range area |
| The relationship between two measures, one point per item | `PointMark` | Scatter plot |
| A range from a low to a high value | `BarMark` with start and end | Range bar |
| When things start and end, per row | `BarMark` or `RuleMark` with `xStart` and `xEnd` | Timeline, Gantt chart |
| A value for every combination of two categories | `RectangleMark` coloured by the value | Heat map |
| A constant: threshold, target, average, or a selected position | `RuleMark` | A straight line across the plot |
| A region of interest behind other marks | `RectangleMark` with start and end | Shaded band or box |
| Parts of a whole | `SectorMark` (iOS 17.0+), or one stacked `BarMark` | Pie, donut, single stacked bar |

The mark follows from what the chart should communicate, not from the shape of the data.

## How mark positions are given

`BarMark`, `RectangleMark`, `RuleMark` and `PointMark` have many initializers, but they follow one pattern. On each axis a mark takes one of three things:

| Given on an axis | Parameter | The mark there |
|---|---|---|
| A value | `x: .value(...)` | Sits at that value. A bar with a category or nothing on the other axis also extends to zero; with an interval on the other axis the value is only its position. With a number on both axes a bar sits at x and grows from zero to y |
| An interval | `xStart: .value(...), xEnd: .value(...)` | Spans from one value to the other |
| Nothing | Parameter left out | A bar, rectangle or rule spans the whole plot area; a point is centred, and so is a rule that has an interval on the other axis |

Pick the parameters from what the mark should do on each axis. `RuleMark(y:)` is "a value on y, nothing on x", so it crosses the plot. `RectangleMark()` with no arguments fills the plot area. Not every combination has an initializer:

| Mark | Combinations that do not exist |
|---|---|
| `RectangleMark` | None: all nine exist |
| `BarMark` | An interval on both axes; nothing on both |
| `RuleMark` | A value on both axes; an interval on both; nothing on both |
| `PointMark` | An interval on either axis; nothing on both |

A size parameter exists only on the form that has it. With a value on both axes it is `BarMark(x:y:width:)` for vertical bars and `BarMark(x:y:height:)` for horizontal ones. A bar with no category takes the one named for the axis that has the value, `BarMark(x:width:)` or `BarMark(y:height:)`, and there it changes nothing: the bar fills the plot across. Size such a bar with the chart's frame (`gallery/stacked-bar.md`).

The start and end parameters of the left-out axis also accept a `CGFloat`. That is a distance in points from the plot area's top-leading corner, not a data value. Use it only for decoration at a fixed place.

## Mark types

### `AreaMark` - iOS 16.0+

**Use for:** data shown as a filled region: a quantity over time, a total made of stacked parts, or a band between a low and a high value.

**Prefer `LineMark`** when only the line matters and nothing should be filled beneath it.

- `AreaMark(x:y:stacking:)`: fills from the baseline to the value. Put a date or an ordered string on x and a number on y.
- `AreaMark(x:yStart:yEnd:)`: a range area between two values at each x. `AreaMark(xStart:xEnd:y:)` is the horizontal form.
- Each form has a `series:` variant that separates several regions without a colour encoding.

Regions that share x positions stack (`composition.md`); `stacking:` takes a `MarkStackingMethod` (`mark-configuration.md`) and defaults to `.standard`.

Recipe: `gallery/area.md`.

**Pitfalls:** unstacked areas cover each other; add opacity.

### `LineMark` - iOS 16.0+

**Use for:** a trend across a continuous axis. Put a date or category on x and a number on y.

- `LineMark(x:y:)` for a single line.
- `LineMark(x:y:series:)` when the chart has more than one line and they are styled by hand.

Which points join into a line is decided by `series:` or a `by:` encoding (`composition.md`, *Series*).

Style with `interpolationMethod` for the curve, `lineStyle(StrokeStyle(lineWidth:))` for the stroke, and `symbol` or `symbol(by:)` with `symbolSize` for point markers.

Example: `gallery/multi-series-line.md`.

**Pitfalls:**

- Several lines with no `series:` and no `by:` encoding join into one.
- An eighth line gets the first line's colour (`scales.md`). With that many lines, emphasise one and mute the rest (`design.md`).

### `PointMark` - iOS 16.0+

**Use for:** scatter plots, which show how two numeric properties relate, and for calling out a single data point on top of another mark, such as the best day on a line chart, with its own colour and `symbolSize`.

- `PointMark(x:y:)`: one point per element.
- `PointMark(x:)` or `PointMark(y:)`: every point on one line, showing how values are distributed along a single axis. Lower the opacity so overlapping points show as darker patches.
- `PointMark(x:y:z:)`: a point in a 3D chart (`chart3d.md`).

A point can carry three more properties besides its position: `foregroundStyle(by:)` for colour, `symbol(by:)` for shape and `symbolSize(by:)` for size (`mark-modifiers.md`).

Recipe: `gallery/scatter.md`.

### `RectangleMark` - iOS 16.0+

**Use for:** a rectangle positioned by data, when a bar's "rise from a baseline" meaning is wrong. Three jobs:

- **Heat map.** A category on each axis and `foregroundStyle(by:)` on a number: `RectangleMark(x:y:width:height:)`. Recipe: `gallery/heat-map.md`.
- **Shading a region** behind other marks. Give intervals for the axes the region is bounded on and leave out the others: `RectangleMark(xStart:xEnd:)` is a vertical band the full height of the plot; adding `yStart:yEnd:` makes a box. Declare it before the marks it sits behind, and keep its fill faint.
- **A tick at a value.** `RectangleMark(x:y:width:height:)` on a category axis with `height: .fixed(2)` draws a thin tick the same width as a bar on the same band. Example: `gallery/range-bar.md`.

<!-- snippet: Gallery/ScatterChart.swift#region -->
```swift
struct ScatterWithRegion: View {
    let beans: [Bean]

    var body: some View {
        Chart {
            RectangleMark(
                xStart: .value("Acidity from", 4),
                xEnd: .value("Acidity to", 6),
                yStart: .value("Body from", 3),
                yEnd: .value("Body to", 5)
            )
            .foregroundStyle(.green.opacity(0.15))

            ForEach(beans) { bean in
                PointMark(
                    x: .value("Acidity", bean.acidity),
                    y: .value("Body", bean.body)
                )
            }
        }
    }
}
```

`width` and `height` default to 70% of the step on an axis that has one, such as a category axis. `RectangleMark(x:y:z:)` is the 3D form (`chart3d.md`).

**Pitfalls:** on a number axis there is no band, so `RectangleMark(x:y:)` with single values draws only a small fixed-size rectangle. Give a region its edges with start and end values.

### `RuleMark` - iOS 16.0+

**Use for:** a straight horizontal or vertical line.

- `RuleMark(y:)`: horizontal across the whole plot, for a threshold, target or average.
- `RuleMark(x:)`: vertical across the whole plot, for a selected or current position.
- `RuleMark(xStart:xEnd:y:)`: a horizontal line between two x values, one per element: the thin-line form of a timeline (`gallery/timeline.md`). `RuleMark(x:yStart:yEnd:)` is the vertical form, for a line from a low to a high value.
- `RuleMark(x:y:z:)`: a line in a 3D chart (`chart3d.md`).

A rule across the plot is the natural anchor for a label about that value: attach an `annotation` to it. On a horizontal rule, `position: .top` with a leading or trailing alignment sits above one end; on a vertical rule, `position: .trailing, alignment: .top` sits beside its top, inside the plot. Near the plot's edge `.fit` slides that label back over the rule, so leave the default there or move the label to the other side. A rule's value takes part in the scale: a target beyond the data stretches the axis to reach it. Set its thickness and end caps with `lineStyle`.

Examples: `gallery/multi-series-line.md` (threshold), `gallery/range-bar.md` (labelled average), `gallery/selection-line.md` (selection indicator).

### `BarMark` - iOS 16.0+

**Use for:** comparing amounts. One bar per category, day or month.

- `BarMark(x:y:)`: a category (or date with a unit) on one axis and a number on the other. The bar rises from zero. Which axis holds the number sets the orientation.
- `BarMark(x:yStart:yEnd:)`: a floating bar from a low to a high value, for ranges. `BarMark(xStart:xEnd:y:)` is the horizontal form, one bar per interval, and builds a timeline or Gantt chart (`gallery/timeline.md`).
- `BarMark(x:)` with nothing on y: every element lands in one bar, end to end. With `foregroundStyle(by:)` that is a single stacked bar showing parts of a whole (`gallery/stacked-bar.md`).
- `width:` (or `height:` for horizontal bars, when both axes have a value) takes a `MarkDimension`: `.ratio(0.6)` for a fraction of the band or `.fixed(8)` for a size in points.

Bars that share a position stack, coloured or not (`composition.md`, *Stacking and grouping*); add `position(by:)` to group them side by side instead (`gallery/grouped-bar.md`). `stacking:` takes a `MarkStackingMethod`.

Examples: `gallery/bar.md`, `gallery/stacked-bar.md`, `gallery/range-bar.md`.

**Pitfalls:**

- Dates without `unit:` give bars of a fixed few points centred on an instant: slivers with a few periods, shifted half a period with many (see `data.md`).
- A bar taller than any single value, with faint lines across it, has several rows per category (`composition.md`).

### `SectorMark` - iOS 17.0+

**Use for:** showing how categories make up a total, as a pie or donut.

`SectorMark(angle:innerRadius:outerRadius:angularInset:)`. Only `angle` is required.

- `angle`: the value. Each sector's share of the circle is its value over the sum. It also accepts a range, `.value("Progress", 0 ..< 30)`, which places the sector between two explicit positions on a scale that runs from zero to the largest value in the chart; parts of that scale no sector covers stay empty.
- `innerRadius`: zero (the default) is a pie; anything larger is a donut. `.ratio(0.618)` is a well-proportioned ring; `.ratio` and `.inset` are measured against the outer radius, so `.inset(30)` is a ring 30 points thick.
- `outerRadius`: `.ratio` and `.inset` are measured against the plot area, so `.inset(10)` leaves a 10-point margin.
- `angularInset`: a gap between sectors, in points taken off each side, so `1.5` leaves a 3-point gap. It adds contrast between neighbouring sectors, which helps accessibility and readability. Keep it small (1 to 1.5), like the corner radius, so the sectors' shapes and relative sizes are not distorted. Very small sectors can disappear when an inset is set.
- `cornerRadius` rounds the corners; `foregroundStyle(by:)` colours the sectors and makes the legend.

The angle values are normalized to the full circle automatically; pass the raw amounts. Sectors start at 12 o'clock and run clockwise in data order, so order the data largest first, with any "Other" group last. A chart of sectors has no axes. Values must be positive.

Text in the hole of a donut is a view placed with `chartBackground` (`plot-area.md`). Selecting a sector uses `chartAngleSelection` (`interaction.md`). Recipe: `gallery/donut.md`.

**Pitfalls:**

- Keep sector order fixed when the same categories are shown for different periods, so sectors do not swap places when the data changes.
- Needs iOS 17.0. On an iOS 16 target, gate the chart and fall back to bars (`availability/README.md`).
- No modifier sets the start angle or the total angle of a sector chart. A half ring or gauge is built with angle ranges and a rotated chart (`gallery/gauge.md`).
