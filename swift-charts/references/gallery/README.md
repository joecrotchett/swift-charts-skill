# Chart gallery

Common chart forms, each with a written visual signature so a chart can be identified from an image, and a recipe file with the implementation. Images in `../../assets/gallery/` are rendered from the recipe code.

## Identifying a chart

Match what is visible against the *Visual signature* column, then open the recipe. A recipe shows how the form is constructed, not a finished chart, and its code leaves out what every delivered chart needs. Before delivering one, add: its title or headline (`../design.md`); the labelled `Plot`s its *Accessibility* section describes and a chart descriptor (`../accessibility.md`, `../audio-graph.md`); the view's locale on every explicit format, and the data's calendar where dates were recorded in another time zone (`../composition.md`, `../data.md`); and a look at the largest text sizes and at right-to-left (`../accessibility-beyond-voiceover.md`). Each recipe's *Accessibility* section says what the form reads as by default and what it needs; only some recipes include that code. A design that combines forms is built by combining recipes (see `../composition.md`).

| Form | Visual signature | Minimum OS | Recipe |
|---|---|---|---|
| Bar chart | One solid rectangle per category rising from a shared baseline; category names along one axis, a number scale on the other. Horizontal variant: bars grow left to right. Either may carry a value above or inside each bar. | iOS 16.0 | `bar.md` |
| Histogram | Bars that touch, on a continuous number or date axis labelled at the boundaries, against a count. | iOS 16.0 | `histogram.md` |
| Stacked bar chart | Each bar is split into coloured segments that sit on top of each other; a legend names the colours. Single-bar variant: one horizontal bar divided into segments. Diverging variant: segments above and below a zero line. Segments may have thin gaps between them. | iOS 16.0 | `stacked-bar.md` |
| Grouped bar chart | Each category holds a cluster of narrower bars side by side, one colour per sub-category; a legend names the colours. | iOS 16.0 | `grouped-bar.md` |
| Area chart | A filled region under a line, sometimes fading toward the baseline beneath a solid line; or several coloured bands stacked on each other; or a floating band of varying thickness. Bands filling the full height are normalized; bands symmetric about a centre line are a streamgraph. | iOS 16.0 | `area.md` |
| Line shapes | One line drawn as straight segments, as a smooth curve, or as a staircase. | iOS 16.0 | `line-shapes.md` |
| Multi-series line chart | Two or more continuous lines over a shared time or number axis, told apart by colour or point symbol; often a straight horizontal line marks a threshold. | iOS 16.0 | `multi-series-line.md` |
| Candlestick chart | Along a time axis, thin vertical lines each with a wider box over part of it, the boxes in two colours. | iOS 16.0 | `candlestick.md` |
| Box plot | Per category, a thin line from lowest to highest value, a box over the middle of it, and a line across the box. | iOS 16.0 | `box-plot.md` |
| Diverging bar chart | Horizontal bars growing left or right from one zero line, one signed value each, sorted into a sweep; names beside the bars, a signed axis along the top. | iOS 16.0 | `diverging-bar.md` |
| Population pyramid | Horizontal bars growing left and right from a centre line, one row per group, one colour per side. | iOS 16.0 | `population-pyramid.md` |
| Vector field | A grid of small arrows, each rotated to show a direction. | iOS 16.0 | `vector-field.md` |
| Difference chart | The gap between two lines over time, filled in a colour that changes whenever the lead changes. | iOS 16.0 | `difference.md` |
| Scatter plot | Separate points placed by a number on each axis, not joined; groups differ by colour or symbol shape. Variants: a shaded box behind the points, a trend line through them, a single row of points, or a row whose crowded points pile up into columns (beeswarm). | iOS 16.0 | `scatter.md` |
| Beeswarm | Points along one value axis, nudged into rows so none overlap, piling up where values crowd. | iOS 16.0 | `scatter.md`, *Beeswarm* |
| Heat map | A grid of touching cells, one per pair of categories, shaded along one gradient by value (light to dark in Light Mode, dark to bright in Dark Mode); the legend is a gradient strip. | iOS 16.0 | `heat-map.md` |
| Calendar heat map | A grid seven columns wide, one tile per day, shaded by value, with day numbers in the tiles and weekday names across the top; rows are weeks. | iOS 16.0 | `calendar.md` |
| Timeline (Gantt chart) | Named rows, each with one or more horizontal bars or thin lines that start and end at different points along a time axis. | iOS 16.0 | `timeline.md` |
| Range bar with average | Floating bars that do not touch the baseline, each spanning a low to a high value, with a thin tick inside for a typical value; often a full-width line for an overall average. | iOS 16.0 | `range-bar.md` |
| Chart with customised axes | Any form whose axes differ from the defaults below: a fixed round maximum, fewer or formatted labels, unlabelled grid lines, an axis on another edge, two-line time labels. | iOS 16.0 | `custom-axes.md` |
| Small multiples | Several small charts of one type stacked or in a grid, one per category or measure, sharing an axis. | iOS 16.0 | `small-multiples.md` |
| Linked charts | Two or more charts stacked or side by side with the same period or item highlighted in each, often by a shaded band. | iOS 17.0 (16.0 with a gesture fallback) | `linked-charts.md` |
| Compact overview chart | A small chart with no axes, labels or legend, sitting under a short headline and a large number, usually in a list row or card. | iOS 16.0 | `compact.md` |
| Donut or pie chart | A ring (or full disc) cut into coloured wedges; a donut often has text in the hole. | iOS 17.0 | `donut.md` |
| Gauge (half ring) | An arch partly filled with colour from one end, the rest a pale track. | iOS 17.0 | `gauge.md` |
| Scrolling bar chart | A bar chart showing a window of a longer time range; bars are cut off at an edge, or a date range above the chart changes as it moves. | iOS 17.0 | `scrolling-bar.md` |
| Line chart with selection | A line chart with a thin vertical line at one position and a floating label above it giving the values there; or two vertical lines bracketing a range. | iOS 17.0 | `selection-line.md` |
| Function plot | A perfectly smooth curve with no data points, on axes centred on zero: a wave, a parabola, a region between two curves, or a looping figure. | iOS 18.0 | `function-plot.md` |

## What an uncustomised chart looks like

Knowing the defaults tells you which parts of a design need a modifier: anything in the image that differs from this list was customised. These are the framework's defaults in the iOS 27 SDK.

- **Value axis:** labels on the trailing edge, outside the plot and centred on solid horizontal grid lines, one per label, four or five of them, formatted as plain numbers with grouping separators, until the axis reaches several million: 2,000,000 is written in full, 8,000,000 as "8.0E6". A number scale includes zero, on either axis and for every mark type, and rounds up past the largest value. A colour scale does not: a number encoded as colour starts near the smallest value (144 to 279 gives a legend from 100 to 300). A scatter plot of values from 3 to 8 still starts at 0.
- **Category or date axis:** labels along the bottom, with dashed vertical grid lines between categories or at each date label. A date label sits to the right of its grid line, which extends down beside it. Category labels that do not fit are cut short, with an ellipsis while there is room for one; labels listed in an `AxisMarks` closure are not, and overlap instead.
- **Categories on the y axis:** the names sit inside the plot area, above each row's marks at the leading edge, not outside to the left. The marks get what is left of the row. This applies to horizontal bars, timelines and heat maps.
- **Bars:** fill most of their band, with slightly rounded corners.
- **Category order:** the order the values first appear in the data, not alphabetical. Stacked segments stack bottom to top in the same order, and the legend lists them in that order.
- **Colour:** a single series is blue. A colour encoding of categories assigns blue, green, orange, purple, red, cyan, yellow in order, then starts again at blue. A colour encoding of a number shades every mark along one gradient, light to dark in Light Mode and dark to bright in Dark Mode; its lowest values are pale enough to vanish against a white background.
- **Legend:** appears only when a mark uses a `by:` style encoding. It sits below the chart at the leading edge, with a circle swatch per entry (or the series' own symbol when `symbol(by:)` is used). A number encoded as colour gets a gradient strip with values beneath it; a number encoded as size gets a row of circles of increasing size.
- **Scale direction and type:** numbers increase upward and to the trailing side, evenly spaced (linear). Dates on the y axis run the other way, earliest at the top.
- **No data:** a chart given an empty collection draws nothing at all, not even axes.
- **Lines:** straight segments between points, no point symbols. Encoded line styles are assigned solid, then dashes of increasing length.
- **Symbols:** a point is a filled circle. Encoded symbols on points are assigned circle, cross, triangle, plus, square, asterisk, diamond in order, then start again at circle. On a line, a fixed `symbol(.circle)` and `symbol(by:)` both draw them outlined, as rings, and `symbol(by:)` assigns them in another order, circle, square, triangle, diamond, while the legend shows them filled.
- **An encoding other than colour leaves the colour alone:** with only `symbol(by:)` or `lineStyle(by:)`, every series is drawn in the same blue.
- **Corner radius:** a bar rising from the baseline is rounded at its free end only; a floating bar is rounded at both ends.
- **Sectors:** start at 12 o'clock and run clockwise in data order.
- **Rectangles:** 70% of the band wide and high on category axes, so a default heat map has gaps between cells.
- **Stacking:** bars that share a position stack even with no colour encoding, with a hairline between the pieces.
- **Out-of-range marks:** a mark beyond a scale's domain is not clipped; it is drawn outside the plot area.
