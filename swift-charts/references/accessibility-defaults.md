# What assistive technologies get from a chart by default

What VoiceOver reads, and what the Audio Graph is given, for a chart with no accessibility code. Use it to predict what a chart will say, and to explain one that says something odd. Everything below was read on iOS (the iOS 27 simulator, with the device six hours behind UTC) unless it says macOS.

## Elements

The chart is a group holding one accessibility element per position, not one per mark. The position is along the axis that holds the categories or the independent value: x for a vertical chart, y for horizontal bars ("the x value" below means that one).

**Not elements:** the axes, the automatic legend and the views inside annotations. Views given to `chartLegend(content:)` are elements on iOS, and in the accessibility tree they carry the chart's descriptor too. **Elements:** views in a `chartOverlay`, and on iOS views in a `chartBackground` too (on macOS they were not), so text placed behind the plot is read. A symbol image there is read by its system label ("Favorite" for `star.fill`); hide or label it. An `accessibilityLabel` on the whole chart replaces the label of every view in its overlay and background, as well as naming the chart.

What an element says depends on how many marks sit at its position:

| At one x position | Label | Value | Read as |
|---|---|---|---|
| One mark | The x value | The y value | "Latte, 128" |
| Stacked marks | The x value | Their total, and how many categories | "Latte, total 128, 3 Sizes" |
| Marks side by side (`position(by:)`) | The x value | The same as stacked: their total, not their average | "Latte, total 128, 3 Sizes" |
| Several series of lines or points | The x value | The first series' value to the last's (not lowest to highest), and how many series | "Mar 1, 2026 at 12 AM to Mar 2, 2026 at 12 AM, 190 to 155, 2 Stores". With a colour and a symbol encoding on the same field, the count is "2 series" |
| A mark drawn between two values | The x value and the two values, with no comma between them | Nothing | "Item 1 1 to 10" for a bar from 1 to 10 |
| An interval on a date axis, one per category | A timeline row: the category, then its start and end. A shift within one day: the one-hour stretch around its middle, then the category | Nothing | "Item 1 Feb 28, 2026 at 6:00 PM to Mar 2, 2026 at 6:00 PM"; "Mar 1, 2026 at 8:00 AM to Mar 1, 2026 at 9:00 AM Tom" for a shift from 11:00 to 18:00 UTC |
| A `RuleMark` across the plot | Nothing | Its value | "400", after all the data marks |
| A second layer over the same data (an area under a line, points over a line) | Each mark is its own element | | Every position twice, and which of the pair comes first varies from one position to the next. Points drawn as a separate layer are read after the whole line, in no x order |

Pies, heat maps and single stacked bars have elements as marks do: one per sector, one per column of a heat map (its average and a count), one per segment. **A strip of points with only an x value has no elements at all.**

Two things change on a continuous axis (numbers or dates):

- **The label is a range.** A point at 3 is "3 to 3.5"; a day is "Mar 1, 2026 at 12 AM to Mar 2, 2026 at 12 AM". A date is written with its full date. A day bin (`unit: .day`) is read in the time zone of the calendar passed to `.value(…, unit: .day, calendar:)`: with a UTC calendar, days made at midnight UTC read "Mar 1, 2026 at 12 AM to Mar 2, 2026 at 12 AM" on a device six hours behind. With no calendar the device's is used, and the same days are binned and read as the local day before ("Feb 28, 2026 at 12 AM to Mar 1, 2026 at 12 AM"). A plain date with no unit is read in the device's time zone: orders from 6 AM to 6 PM UTC as "12:00 AM" to "12:00 PM" six hours behind.
- **Neighbouring marks are merged** once there are more than a few, into one element per stretch of the axis. A stretch takes one of three forms: "starts at 8, varies between 9 and 2, ends at 3, 60 values", the highest and lowest in the order they are reached, not lowest first; "3 to 1, decreases inconsistently, 60 values" when its first and last values are its extremes; and "193 to 146, 2 values" for a stretch of two. **How many stretches you get depends on the data, not only on the number of marks.** For example: 12 bars at number positions made 6; 30 days at local midnight made 15 two-day stretches, while 30 days at UTC midnight made 6 one-week stretches; 330 points over the hours 6 to 18 made 6 two-hour stretches, 400 points over the same hours made 12 one-hour ones, and 2,000 made 6. Six points, or seven days, stay one each. Part of a stretch with no data (a missing hour) does not split it: the gap is folded inside, and breaking the drawn line with `series:` does not change that. A category axis is never merged: 60 categories are 60 elements. So a 30-day chart needs a labelled `Plot` per day to be read day by day.

The words come from the chart's code. "3 Sizes" is the label in `.value("Size", ...)` made plural, and the Audio Graph names its axes from the same labels. Write each label as a singular noun a listener will understand, not a variable name.

## Audio Graph

VoiceOver offers *Describe chart*, *Chart Details* and *Play Audio Graph*. It names the axes and the number of series, gives summaries of the data, and plays the chart as tones that follow its values. What the framework hands it by default:

| Part | Default |
|---|---|
| Title | "Chart", whatever the chart shows. An `accessibilityLabel` on the chart does not change it |
| Summary | None |
| Axis titles | The `.value` labels |
| Value axis range | The drawn scale, not the data: 0 to 300 for data up to 219 |
| Series | One per value of a colour encoding, named by that value ("Small", "Medium"). With no encoding, one series with no name. With a colour and a symbol encoding on the same field, each name is doubled ("Harbour Harbour"). Lines are continuous, bars and points separate. A line broken with `series:` but no encoding is still one continuous series: the gap is not heard |
| Each point | The x value and the bare number. Labels and values set on marks or on `Plot`s are not used, so there are no units |
| A date axis | Handed over as a plain number axis of seconds since 2001 (the middle of each date bin), with both ends described as empty strings: no date is ever said |
| Rules | Left out. A chart that is only a rule has no Audio Graph |
| A vectorized plot | As the same marks would be: a `PointPlot` of 2,000 orders and a `ForEach` of the same `PointMark`s gave identical elements. Its key-path labels and values are not used, in the elements or here |
| Charts with no value on each axis | No Audio Graph at all: pies, donuts and gauges, heat maps and calendar grids, timelines, range bars (`yStart`/`yEnd` over categories), a single stacked bar, a one-axis strip of points, a candlestick |

Marks layered over the same data distort it. A line with an area under it gives every point twice in one series; a line with a `PointMark` layer gives two unnamed series, one continuous and one not. Hide the layer that adds nothing with `accessibilityHidden(true)`, and put points on a line with `symbol`, which adds no marks.

How reshaping the elements affects the Audio Graph:

- `accessibilityElement(children: .ignore)` with a label on the chart makes one element and removes the Audio Graph. On a `Button` it also drops the button trait.
- `accessibilityChildren` on the chart replaces every element, the legend's included, and removes the Audio Graph. Inside `chartPlotStyle` it replaces only the plot's elements and keeps the Audio Graph.
- A chart descriptor replaces all of the Audio Graph's data exactly as written, and leaves the elements alone.

What the app adds to all this is in `accessibility.md` and `audio-graph.md`. What VoiceOver *says* from this data (the words of *Describe chart*, the tones) has not been heard in these checks; hear it on a device. Read your own chart's elements before relying on the exact wording (`accessibility.md`, *Testing a chart's accessibility*).
