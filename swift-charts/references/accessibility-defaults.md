# What assistive technologies get from a chart by default

What VoiceOver reads, and what the Audio Graph is given, for a chart with no accessibility code. Use it to predict what a chart will say, and to explain one that says something odd.

## Elements

The chart is a group holding one accessibility element per position, not one per mark. The position is along the axis that holds the categories or the independent value: x for a vertical chart, y for horizontal bars ("the x value" below means that one). The axes, the legend, the views inside annotations and views in a `chartBackground` are not elements. Views in a `chartOverlay` are: text there is read, and a decorative symbol is announced by its symbol name unless it is hidden or labelled. What an element says depends on how many marks sit at its position:

| At one x position | Label | Value | Read as |
|---|---|---|---|
| One mark | The x value | The y value | "Latte, 50" |
| Stacked marks | The x value | Their sum, and how many categories | "Latte, total 50, 3 Sizes" |
| Marks side by side (`position(by:)`) | The x value | Their average | "Latte, average 16.7, 3 Sizes, 3 Sizes" (the category count is said once per encoding that uses it) |
| Several series of lines or points | The x value | The span of their values | "Mar 1 ... to Mar 2 ..., 150 to 145, 2 Stores" |
| A mark drawn between two values | The x value and the two values | Nothing | "Jan, 1 to 6" for a bar from 1 to 6 at Jan |
| A `RuleMark` across the plot | Nothing | Its value | "400" |

Two things change on a continuous axis (numbers or dates):

- **The label is a range.** A point at 3 is "3 to 3.5"; a day is "Mar 1, 2026 at 12 AM to Mar 2, 2026 at 12 AM".
- **Neighbouring marks are merged** once there are more than a few, into one element per stretch of the axis that summarises them. For example: 400 samples make 8 elements such as "0 to 5, starts at 100, varies between 100 and 189, ends at 113, 50 values"; 12 bars at number positions make 6; 30 days make 15. Six points, or seven days, stay one each. A category axis is never merged: 60 categories are 60 elements. So a 30-day chart needs a labelled `Plot` per day to be read day by day.

The words come from the chart's code. "3 Sizes" is the label in `.value("Size", ...)` made plural, and the Audio Graph names its axes from the same labels. Write each label as a singular noun a listener will understand, not a variable name.

## Audio Graph

VoiceOver offers *Describe chart*, *Chart Details* and *Play Audio Graph*. It names the axes and the number of series ("The x-axis is time. The y-axis is sales. There is one data series"), gives summaries of the data, and plays the chart as tones that follow its values. What the framework hands it by default:

| Part | Default |
|---|---|
| Title | "Chart", whatever the chart shows. An `accessibilityLabel` on the chart does not change it |
| Summary | None |
| Axis titles | The `.value` labels |
| Series | One per value of a colour encoding, named by that value ("Small", "Medium"). With no encoding, one series with no name. Lines are continuous, bars and points separate |
| Each point | "Jan, 10": the x value and the bare number. Labels and values set on marks are not used, so there are no units |
| A date axis | Handed over as a plain number axis, and each point is described by its value alone, with no date |
| Rules | Left out |
| A vectorized plot | As the same marks would be. Its key-path labels and values are not used, in the elements or here |
| Charts with no value on each axis | No Audio Graph at all: pies, donuts and gauges, heat maps and calendar grids, timelines, a single stacked bar, a one-axis strip of points, a candlestick |

Marks layered over the same data distort it. A line with an area under it gives every point twice in one series; a line with a `PointMark` layer gives two unnamed series. Hide the layer that adds nothing with `accessibilityHidden(true)`, and put points on a line with `symbol`, which adds no marks.

What the app adds to all this is in `accessibility.md` and `audio-graph.md`.

Everything above is the accessibility tree as macOS builds it. Read your own chart's elements before relying on the exact wording (`accessibility.md`, *Testing a chart's accessibility*).
