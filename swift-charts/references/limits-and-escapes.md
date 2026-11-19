# Limits and escape hatches

A lookup from a design element to whether Swift Charts can do it, from which OS, with which API. Check here before telling the user a design is impossible, and before reaching outside the framework.

## Capability table

One row per design element people ask for. *Verdict* is one of:

- **Native:** chart content or a chart modifier does it. *Minimum OS* says from when; if that is above the deployment target, gate it and supply the fallback the row or its reference names.
- **Escape hatch:** it needs a SwiftUI view or technique that is not chart content, even where a chart modifier (`chartOverlay`, `chartBackground`) attaches it. It works, and its position, accessibility and upkeep are yours.
- **Not feasible:** say so and offer the nearest design that is.

Whole chart forms are in `gallery/README.md` with their own minimum OS; a form appears here only when part of it is an escape hatch.


### Outside the framework

Designs that need SwiftUI beside or around the chart, or cannot be done.

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| Text or a view in the hole of a donut | Escape hatch | iOS 17.0 | `chartBackground` with `ChartProxy.plotFrame`, a SwiftUI view positioned at the plot centre | `gallery/donut.md` |
| Selecting a value or dragging out a range on iOS 16 | Escape hatch | iOS 16.0 | A gesture in `chartOverlay`, converted with `ChartProxy.value(atX:as:)`; the indicator is still a mark | `interaction.md`, `gallery/selection-line.md` |
| Scrolling on iOS 16 | Escape hatch | iOS 16.0 | A chart wider than the screen inside a horizontal `ScrollView`; its value axis scrolls with it | `interaction.md` |
| Half-ring gauge, or any sector chart with a chosen start angle or partial sweep | Escape hatch | iOS 17.0 | No modifier for it: `SectorMark` angle ranges with a clear sector, and `rotationEffect` on the chart | `gallery/gauge.md` |
| A view pinned to a data value or a category's band | Escape hatch | iOS 16.0 | `chartOverlay` with `ChartProxy.position(forX:)` or `positionRange(forX:)` | `plot-area.md` |
| Custom legend layout in the chart's own colours | Escape hatch | iOS 16.4 | Hide the legend; build one from `ChartProxy.foregroundStyle(for:)` | `legend.md` |
| Small multiples (a grid or stack of aligned charts) | Escape hatch | iOS 16.0 | No faceting in the framework: one `Chart` per group in SwiftUI layout, with a shared scale domain | `gallery/small-multiples.md` |
| 3D bars, lines, areas or pies | Not feasible | | `Chart3D` accepts only points, rules, rectangles and surfaces. Nearest: a 2D grouped bar chart or heat map when the third dimension is a category, a 3D scatter when all three are numbers | `chart3d.md` |
| Image export of a 3D chart | Escape hatch | iOS 26.0 | `ImageRenderer` draws a placeholder for `Chart3D`; draw a hosted window with `drawHierarchy`, which captures it in full (iOS): a `UIHostingController` (`safeAreaRegions = []`) in a `UIWindow(windowScene:)` of the presenting view's scene (a bare `UIWindow(frame:)` draws blank), shown and left to draw, then `drawHierarchy(in:afterScreenUpdates: true)` in a `UIGraphicsImageRenderer`; or export a 2D chart of the same data | `chart3d.md` |
| Image export of a chart | Native | iOS 16.0 | `ImageRenderer(content:)` with `scale` set to the display scale and an explicit frame and background. It renders without the presenter's environment, the locale included (iOS): set `\.locale` and the colour scheme on the content | `limits-and-escapes.md` |
| Image export of a scrolling chart | Escape hatch | iOS 17.0 | `ImageRenderer` draws a scrollable chart with no marks: render a second chart without the scrolling modifiers, its x domain fixed to the visible window | `interaction.md` |

### Marks and styling

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| A point symbol shape that is not built in | Native | iOS 16.0 | Conform a `Shape` to `ChartSymbolShape` | `mark-configuration.md` |
| An image, icon or any view as the point symbol | Native | iOS 16.0 | `symbol(symbol:)` | `mark-modifiers.md` |
| Outlined (hollow) point symbols | Native | iOS 16.0 | `.symbol(.circle.strokeBorder(lineWidth:))` | `mark-configuration.md` |
| Bubble chart (points sized by a value) | Native | iOS 16.0 | `symbolSize(by:)` with `chartSymbolSizeScale(range:)` | `mark-modifiers.md` |
| Series told apart by dash pattern | Native | iOS 16.0 | `lineStyle(by:)` | `mark-modifiers.md` |
| Stepped (staircase) or smoothed line | Native | iOS 16.0 | `interpolationMethod` | `gallery/line-shapes.md` |
| Shaded band between a low and a high line | Native | iOS 16.0 | `AreaMark(x:yStart:yEnd:)` | `gallery/area.md` |
| Line over a fading gradient fill | Native | iOS 16.0 | `LineMark` over an `AreaMark` with a gradient `foregroundStyle` | `gallery/area.md` |
| Shaded region or band behind the data | Native | iOS 16.0 | `RectangleMark` with start and end values | `marks.md` |
| Outline of a map or other shape with data points on it | Native | iOS 16.0 | Projected coordinates plotted as a line and as points, axes hidden, fixed aspect ratio. Not a map view: no tiles, panning or geography | `plot-area.md` |
| A gradient that runs across the whole chart, not per bar | Native | iOS 16.0 | `alignsMarkStylesWithPlotArea` | `mark-modifiers.md` |
| Pill-shaped bars rounded at both ends | Native | iOS 16.0 | `clipShape(Capsule())` | `mark-modifiers.md` |
| Gaps between the segments of a stacked bar | Native | iOS 16.0 | `clipShape(Rectangle().inset(by: 1))` on each mark | `gallery/stacked-bar.md` |
| One bar divided into parts of a whole | Native | iOS 16.0 | `BarMark(x:)` with `foregroundStyle(by:)` | `gallery/stacked-bar.md` |
| Bars above and below zero in one stack | Native | iOS 16.0 | Negative values with standard stacking | `gallery/stacked-bar.md` |
| Bars of irregular or sub-day duration on a time axis | Native | iOS 16.0 | A range of dates as the x value | `data.md` |
| Continuous colour ramp with a gradient legend | Native | iOS 16.0 | `foregroundStyle(by:)` on a number; `chartForegroundStyleScale(range: Gradient(...))` | `scales.md` |
| Drop shadow or blur on marks | Native | iOS 16.4 | `shadow`, `blur` | `mark-modifiers.md` |
| A reusable chart element with its own type | Native | iOS 16.0 | Conform a struct to `ChartContent` | `composition.md` |
| Graph of a mathematical function, a trend line over a scatter plot, a fitted curve over a histogram, or a band around any of them | Native | iOS 18.0 | `LinePlot` and `AreaPlot` with a function; the app computes any fit. Below iOS 18, sample the function into `LineMark` data | `gallery/function-plot.md`, `vectorized-plots.md` |
| A very large collection drawn efficiently | Native | iOS 18.0 | Vectorized plots. Below iOS 18, bin or aggregate so there are fewer marks | `vectorized-plots.md` |

### Axes and scales

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| Chart with no axes, grid or legend | Native | iOS 16.0 | `chartXAxis(.hidden)`, `chartYAxis(.hidden)`, `chartLegend(.hidden)` | `gallery/compact.md` |
| Axis with a fixed range, such as 0 to 100% | Native | iOS 16.0 | `chartYScale(domain:)` | `scales.md` |
| Axis that does not start at zero | Native | iOS 16.0 | `.automatic(includesZero: false)` or an exact domain | `scales.md` |
| Reversed axis (rank 1 at the top) | Native | iOS 16.0 | `.automatic(reversed: true)` | `scales.md` |
| Logarithmic axis | Native | iOS 16.0 | `chartXScale(type: .log)`; square-root, power and symmetric-log need iOS 16.4 | `scales.md` |
| Categories in a chosen order, or an empty slot for a missing category | Native | iOS 16.0 | An array of categories as the scale's domain | `scales.md` |
| Chosen number, spacing or exact positions of axis labels | Native | iOS 16.0 | `AxisMarks(values:)` | `axes.md` |
| Abbreviated axis values (250K, 1M) | Native | iOS 16.0 | `AxisMarks(format:)` with compact notation | `axes.md` |
| Axis labels with a unit, percent or currency | Native | iOS 16.0 | `AxisMarks(format:)` or `AxisValueLabel(format:)` | `axes.md` |
| Axis title with units | Native | iOS 16.0 | `chartXAxisLabel`, `chartYAxisLabel` | `axes.md` |
| Grid lines without labels, or labels without grid lines | Native | iOS 16.0 | Two `AxisMarks` with different values and content | `gallery/custom-axes.md` |
| Axis on the left, or along the top | Native | iOS 16.0 | `AxisMarks(position:)` | `axes.md` |
| Two-line or fully custom axis labels | Native | iOS 16.0 | `AxisValueLabel { }` with any view | `gallery/custom-axes.md` |
| Rotated axis labels | Native | iOS 16.0 | `AxisValueLabel(orientation: .vertical)`. Vertical only; no diagonal option | `axes.md` |
| Coloured, dashed or heavier grid lines | Native | iOS 16.0 | `AxisGridLine(stroke:)` with `foregroundStyle` | `axes.md` |
| Colours that stay put when data is filtered, or match across charts | Native | iOS 16.0 | `chartForegroundStyleScale(domain:)` | `scales.md` |

### Labels on marks

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| Value label above, beside or inside each bar | Native | iOS 16.0 | `annotation(position:alignment:)`; `.overlay` for inside | `annotations.md` |
| Category names written on the bars | Native | iOS 17.0 | `annotation(position: .leading, ...)` fitted to the plot, with the axis hidden; `.overlay` on iOS 16 | `annotations.md` |
| Label that stays inside the chart at its edges | Native | iOS 17.0 | `overflowResolution:` with `.fit(to:)` or `.padScale`; no edge handling on iOS 16 | `annotations.md` |

### Plot area and layout

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| Tinted or bordered plot area | Native | iOS 16.0 | `chartPlotStyle` | `gallery/bar.md` |
| Marks cut off at the edge of the plot area | Native | iOS 16.0 | `chartPlotStyle { $0.clipped() }` | `plot-area.md` |
| Plot area with a fixed aspect ratio (equal units on both axes) | Native | iOS 16.0 | `chartPlotStyle { $0.aspectRatio(1, contentMode: .fit) }` | `plot-area.md` |
| Plot area sized by its content (a fixed height per row) | Native | iOS 16.0 | `chartPlotStyle { $0.frame(height:) }` | `plot-area.md` |
| Animated transition when the data changes | Native | iOS 16.0 | Change the data inside a SwiftUI animation | `composition.md` |

### Interaction

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| Tap or drag to select a value, with a popover | Native | iOS 17.0 | `chartXSelection` plus a `RuleMark` with an `annotation` | `gallery/selection-line.md` |
| Select a range between two positions | Native | iOS 17.0 | `chartXSelection(range:)` plus two rules in a `Plot` | `gallery/selection-line.md` |
| Select the nearest point in a scatter plot or map | Native | iOS 17.0 | `chartXSelection` and `chartYSelection` together, then a nearest-point search | `interaction.md` |
| Select a pie or donut sector | Native | iOS 17.0 | `chartAngleSelection`, mapping the cumulative value to an element | `gallery/donut.md` |
| Select on tap, long press or another gesture | Native | iOS 17.0 | `chartGesture` calling `ChartProxy.selectXValue(at:)` | `interaction.md` |
| Shaded band behind the selected bar | Native | iOS 16.0 | `RectangleMark` with an x value, a unit and `width: .ratio(1)` | `interaction.md` |
| Several charts sharing one selection | Native | iOS 17.0 | The same `chartXSelection` binding on each, and the same x domain | `gallery/linked-charts.md` |
| Horizontally scrolling chart, snapping to days, pages or multiples | Native | iOS 17.0 | The four scrolling modifiers, with `chartScrollTargetBehavior(.valueAligned(...))` | `gallery/scrolling-bar.md`, `interaction.md` |
| A border or overlay that stays still over a scrolling chart | Native | iOS 17.0 | `ChartProxy.plotContainerFrame` | `plot-area.md` |

### Data and accessibility

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| Totals per week or other period from finer data | Native | iOS 16.0 | `DateBins` and a sum per bin; the chart does not aggregate | `data.md` |
| Spoken title and summary for the Audio Graph | Native | iOS 16.0 | `accessibilityChartDescriptor` with an `AXChartDescriptorRepresentable` | `audio-graph.md` |

### 3D

| Design element | Verdict | Minimum OS | How | Detail |
|---|---|---|---|---|
| 3D scatter plot or 3D surface | Native | iOS 26.0 | `Chart3D` with `PointMark(x:y:z:)` or `SurfacePlot`; not on tvOS or watchOS | `chart3d.md` |
| A fitted plane or model surface through 3D points | Native | iOS 26.0 | `SurfacePlot` in the same `Chart3D` as the points; the app computes the fit | `chart3d.md` |

## Escape hatches

The techniques the table relies on, and where each is written up:

| Technique | Use for | Cost | Detail |
|---|---|---|---|
| A SwiftUI view positioned with `ChartProxy` in `chartOverlay` or `chartBackground` | Centre text, badges, a custom legend, anything pinned to a value | Its position is yours to keep right. Views in an overlay are accessibility elements and need labels; on iOS views in a background are read too, so hide decoration there | `plot-area.md` |
| A gesture in `chartOverlay` | Selection on iOS 16 | Two code paths to keep behaving alike | `interaction.md` |
| A horizontal `ScrollView` around a wide chart | Scrolling on iOS 16 | The value axis scrolls away; no snapping | `interaction.md` |
| `rotationEffect` and a clear sector | Gauges and partial rings | Everything in the chart rotates; half the frame is empty | `gallery/gauge.md` |
| One `Chart` per group in a SwiftUI layout | Small multiples | Scales must be fixed by hand to match | `gallery/small-multiples.md` |

Prefer a mark with an `annotation` whenever the thing being drawn belongs to a data value: marks stay aligned with the scales for free.
