# Review checklist

The mistakes a reviewer would catch in chart code. Run every section that applies before presenting code, and use it as the agenda when reviewing someone else's. Each item names the rule and points to the reference that explains it.

## Availability

- Every API newer than the deployment target is inside `if #available` or an `@available` type, with a fallback. (`availability/README.md`)
- The availability check lists every platform the app ships on. (`availability/README.md`)
- 3D chart API is excluded from tvOS and watchOS builds with `#if os`, not only gated at run time. (`availability/README.md`)
- `ChartProxy.plotFrame` and `plotSize` are used on iOS 17+; `plotAreaFrame` and `plotAreaSize` only below it. (`availability/README.md`)
- `blur` and `shadow` on marks are gated at iOS 16.4, not 16.0. (`mark-modifiers.md`)
- Camera projection is inside `#if os(iOS) || os(macOS)` and rendering style inside `#if os(visionOS)`. (`chart3d.md`)

## API usage

- Nothing the framework provides is rebuilt or defeated: no hard-coded light-only colours, fixed label font sizes or hand-formatted numbers. (`composition.md`)
- `Chart(data)` is used when every element yields its own mark or group of marks; a standalone mark such as a rule goes in `Chart { }`. (`composition.md`)
- Every chart with more than one line gives each `LineMark` a `series:` or a `by:` encoding. (`composition.md`)
- Colour that carries meaning uses `foregroundStyle(by:)`, not a fixed colour chosen per element in code, except to emphasise one element or as a second cue for something position already shows (the sign of a diverging bar). Series styled by hand with `series:` have no legend, so each is labelled directly. (`mark-modifiers.md`, `annotations.md`)
- Shared styling is applied once to the `ForEach` or `Plot`, not repeated on each mark. (`composition.md`)
- A label about a data value is an `annotation` on a mark, not an overlay with computed positions. (`annotations.md`)
- A selection popover names an overflow strategy (`.fit(to: .chart)` on x); a label on the last point of a line may keep the default. `overflowResolution:` is gated at iOS 17. With overflow disabled, space is reserved for the annotation. (`annotations.md`)
- A custom legend takes its colours from the chart or from the same scale, never from a second list. (`legend.md`)
- A chart whose data can be filtered, or that shares categories with another chart, fixes its colour scale domain (and size scale domain). (`scales.md`)
- Greys on marks and axis marks are a colour (`Color.secondary`, `.gray`), not the hierarchical styles `.secondary` or `.tertiary`. (`mark-modifiers.md`)
- An `AxisMarks` closure lists every part the design shows: grid line, tick and label. (`axes.md`)
- An axis is customised at the shallowest level that works: `AxisMarks` parameters before a content closure. (`axes.md`)
- An `AxisValue` is read with `if let value.as(Type.self)`, never force-cast. (`axes.md`)
- A conditional axis label's test matches values the stride actually produces. (`axes.md`)
- An axis that changes with state has its `if` inside the axis closure, not in a conditional view modifier around the chart, which makes two different charts. (`axes.md`)
- An explicit axis `format:`, the headline, element text and the descriptor all take the environment's locale, read in the view and passed in. (`composition.md`)
- A fixed set of categories is an enum conforming to `Plottable`, not strings typed at each use. (`data.md`)
- Square-root, power and symmetric-log scales are gated at iOS 16.4. (`scales.md`)
- No force unwraps: every `ChartProxy` result, `plotFrame` included, is unwrapped, and a `chartForegroundStyleScale` closure has a fallback for unknown values. (`plot-area.md`, `scales.md`)
- `lineStyle(_:)` always sets `lineWidth`, and `lineCap` and `lineJoin` too where a line's default ends and joins should be kept; the style passed replaces all of them. (`mark-modifiers.md`)
- `symbolSize(by:)` is paired with `chartSymbolSizeScale(range:)`. (`mark-modifiers.md`)
- `symbol(by:)` encodes a categorical value. (`mark-modifiers.md`)
- A bare line is a `LineMark`, not an `AreaMark` with a clear fill. (`marks.md`, `gallery/area.md`)
- A recurring chart element is a `ChartContent` type, not copied between charts, and sets inside itself only the styles a caller must not change. (`composition.md`)
- A `SurfacePlot` or function plot returns `Double.nan` where the function is undefined. (`chart3d.md`, `vectorized-plots.md`)
- A region or band is a `RectangleMark` with start and end values, declared before the marks it sits behind. (`marks.md`)
- A 3D rule has one range and two single values; a 3D rectangle two ranges and one single value. (`chart3d.md`)

## Data and scales

- Dates that stand for a day, week or month are plotted with `unit:` on bars and points (on an area it costs half a period of empty plot at each side, a choice to make knowingly), the same unit on every mark sharing the axis. Dates recorded in another time zone pass that calendar to the value and to the axis. (`data.md`)
- Identifiers stored as numbers are converted to strings before being used as categories. (`data.md`)
- Bars and stacked segments are ordered by ordering the data, and are ranked when the chart's message is which is largest or smallest, unless the same categories keep one order across several views. (`gallery/bar.md`, `composition.md`, `design.md`)
- A plain bar chart has one row per category; several rows silently stack into one bar. (`marks.md`)
- Distributions and per-period totals are binned with `NumberBins` or `DateBins`, not by hand-rolled rounding. (`data.md`)
- A bin is plotted as the x value of a `BarMark`, with the bars touching or a hairline apart. (`gallery/histogram.md`)
- A scale domain narrower than the data is paired with a clipped plot area. (`plot-area.md`)
- A value with a meaningful minimum and maximum (a percentage, a rating) has that fixed range as its scale domain; otherwise the dynamic default stays, unless another item here fixes the domain. (`scales.md`, `design.md`)
- The value axis of bars that grow from zero includes zero; floating bars (a range, a box), lines and points far from zero do not have to. (`scales.md`, `design.md`)
- An exact domain is passed directly, not built with `modifyInferredDomain`. (`scales.md`)
- Axis mark values that lie outside the data are intended to widen the scale. (`axes.md`)
- Data spanning several orders of magnitude uses a log scale, and has no zero or negative values on it. (`scales.md`)

## Interaction

- A selection value is snapped to the data before it is used. (`interaction.md`)
- Charts meant to be read together share a selection and an x domain. (`gallery/linked-charts.md`)
- A nearest-point selection on two axes accepts a point only within a small distance of the touch. (`interaction.md`)
- A gesture's location has the plot area's origin subtracted before it goes to `ChartProxy`. (`plot-area.md`)
- A gesture position outside the plot area is rejected before it is converted to a value. (`plot-area.md`)
- A custom gesture on iOS 17+ drives the chart's selection through the proxy instead of keeping a parallel state. (`interaction.md`)
- A fixed overlay on a scrolling chart uses `plotContainerFrame`. (`plot-area.md`)
- A scrolling chart comes to rest on meaningful values with `.valueAligned`: `matching:` for dates (midnight for daily bars), `unit:` for numbers. (`interaction.md`)
- A sector selection is mapped through cumulative ranges. (`interaction.md`)
- The selection indicator is a mark in the chart. (`interaction.md`)
- A label below a zero-based bar has the scale's domain fixed, or is moved above or inside the bar. (`annotations.md`)
- A switch that changes what the marks stand for (days to months) is not animated; new values for the same categories are. (`composition.md`)

## Design

Judgement first, then the specifics.

- The chart was designed from a stated goal and shows the few things that goal needs, checked against real data including zeros, gaps and outliers. With no data, it shows a placeholder, not a blank space or an empty plot; so does a chart of only zeros whose marks grow from zero (bars, areas, sectors). Points at zero are real marks and are drawn. (`design.md`)
- The chart has a point, stated in a visible title or headline; data people only need to look up is in a list or table. (`design.md`)
- The headline is true of the chart as drawn, and is still true when a control changes the chart. (`design.md`)
- Every other sentence beside the chart (a note, an explanation) is true of the chart as it is now drawn, including when the device's own text size or layout direction, not a control, produced it. (`design.md`)
- The mark matches what is being communicated: bars for comparison and totals, a line for change, points for relationships. (`design.md`)
- Axis labels follow a familiar sequence. (`design.md`)
- A 3D chart is justified by three-dimensional data and an app where rotating it is welcome; its initial pose shows the point. (`chart3d.md`)
- Grid lines and labels are no heavier than the chart's job needs, and lighter where the chart is interactive. (`design.md`)
- The message does not depend on interaction, and scrubbing along one axis works from anywhere in the plot area. (`design.md`)
- Series differ by more than colour. (`design.md`)
- Category colours have similar visual weight unless one is meant to dominate, are easy to name apart, and hold up in Dark Mode and Increase Contrast. (`design.md`, `accessibility-beyond-voiceover.md`)
- A small preview chart is static and bare; a detail chart has axes and selection. (`design.md`)
- Charts that differ in subject differ visibly: colour for a different kind of data, mark style for a different measure. (`design.md`)
- Touching areas of colour have separation. (`design.md`)
- A preview and its detail chart use the same type, colours and wording; charts with the same purpose share a style. (`design.md`)
- An explicit colour scale is set wherever the automatic colours could contradict what the categories mean. (`scales.md`)
- The colour scale is written in the same order as the data, so legend and stack agree. (`gallery/stacked-bar.md`)
- A hidden legend or hidden axis is replaced by something that carries the same information. (`design.md`)
- One element is emphasised at a time. (`design.md`)
- Category order is the same in every view of the same data. (`design.md`)
- Unstacked areas have opacity so none is hidden. (`gallery/area.md`)
- A gradient meant to show level uses `alignsMarkStylesWithPlotArea`. (`mark-modifiers.md`)
- Grouped bars are used to compare sub-categories, stacked bars to show totals. (`gallery/grouped-bar.md`)
- A heat map sets cell width and height and moves the row labels outside the plot. (`gallery/heat-map.md`)
- No more than seven series rely on the automatic colours or symbols; both repeat after seven. (`gallery/README.md`)
- A pie or donut has a small `angularInset` and corner radius, not large ones. (`marks.md`)
- The unit of every axis is stated once where the reader will see it: in the heading, in an axis title, or in the axis labels' format. There are no more axis labels than the reader needs, and values in the millions have an explicit format; the automatic one is scientific notation. (`design.md`, `axes.md`)
- Category labels fit: none are truncated to an ellipsis at the smallest supported width. (`axes.md`)
- Custom axis labels are checked at phone width. (`gallery/custom-axes.md`)
- `centered:` is true on a date axis label only when every unit is labelled; with a longer stride it puts the label mid-stride. (`axes.md`)
- A smoothed line uses `.monotone` when its peaks must be real values; a value that holds until it changes uses a stepped line. (`mark-configuration.md`)
- Category names on a horizontal chart are moved outside the plot when they squeeze the rows' height; where width is the constraint they stay inside. (`axes.md`, `design.md`)

## Accessibility

Elements:

- Each accessibility element covers what the chart's purpose calls for: a mark, a portion, or the whole chart. (`accessibility.md`)
- Labels and values are on a `Plot` around the mark or marks, not on a bare mark that can share its element; a rule across the plot is its own element and is labelled directly, or through a labelled `Plot` around it. (`accessibility.md`)
- A control that changes element text without changing the data's ids gives the chart a new identity (`.id(state)`), or VoiceOver keeps the old text. (`accessibility.md`)
- A custom `ChartContent` made of several marks wraps them in one `Plot`. (`accessibility.md`)
- Labels give context first, then the value, with units and dates written out, in actual values and not words like "rapidly", naming what a series is and not its colour. (`accessibility.md`)
- A target or average rule has a label and a value with its unit; a selection indicator is hidden. (`accessibility.md`)
- Decoration and any layer that repeats another's data (a gauge's track, an area under a line, points over a line) are hidden with `accessibilityHidden`. (`accessibility.md`)
- Nothing is conveyed only by an annotation's view or a `chartBackground` view. (`accessibility.md`)
- The labels in `.value("...", ...)` and on function plots are real words, and a category's or series' label is singular: VoiceOver reads them as axis names and in summaries ("3 Sizes"). (`accessibility-defaults.md`, `vectorized-plots.md`)
- `accessibilityChildren`, where used, is applied to the plot area in `chartPlotStyle`. (`accessibility.md`)
- A compact chart inside a button is one element with one description. (`accessibility.md`)

Audio Graph:

- The chart has a chart descriptor with a title, a summary in actual values, units in its value descriptions, written-out dates, and a name for every series; a compact preview that is one element is the exception. (`audio-graph.md`, `accessibility.md`)
- The descriptor's `isContinuous` is true only for a connected line or area over an ordered x; separate totals, a bar per day included, are false, its value descriptions and summary agree in number ("1 cup") through the inflected form, not a hand-written test, and its value range matches the drawn scale. (`audio-graph.md`)

Beyond VoiceOver:

- Axis labels still fit at the accessibility text sizes; their number adapts to `dynamicTypeSize` where they would not. (`accessibility-beyond-voiceover.md`)
- A change to what the chart draws (its data, scale or kind of mark) is announced, not only animated, and the announcement follows every control that redraws the chart; a restyling control that speaks its own value needs none. (`accessibility-beyond-voiceover.md`)
- The elements were read, in Accessibility Inspector or with VoiceOver, not assumed. (`accessibility.md`)

## Large data sets

- Plot types are gated at iOS 18.0 with a mark-based fallback where the target is older. (`vectorized-plots.md`)
- On a plot, key-path modifiers come before plain-value modifiers. (`vectorized-plots.md`)
- A per-element colour is `foregroundStyle(\.property)`; `foregroundStyle(by:)` is for an encoded value with a legend. (`vectorized-plots.md`)
- A function plot alone in a chart sets its x scale domain; one that can leave the y domain sets it or clips the plot area. (`vectorized-plots.md`)
- A trend line is limited to the range of the data with `domain:`. (`vectorized-plots.md`)
- A function plot's closure captures values, not `self`. (`vectorized-plots.md`)
- A plot's key paths point at stored properties, and the collection is sorted by the property that drives its style. (`vectorized-plots.md`)
- Marks are used where elements differ in mark type, modifiers or layering; a plot where a large collection is styled uniformly. (`vectorized-plots.md`)
