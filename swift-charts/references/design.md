# Chart design

Deciding whether to chart, what to build, and how it fits the app. Each guideline names the API that carries it out, so a design decision becomes a line of code. It starts with whether to chart at all, then goes through the five things a chart is designed with (marks, axes, descriptions, interaction, colour), then how charts sit in an app.

## Whether to use a chart

- **Use a chart to highlight something about a dataset.** A chart draws the eye, so it should have a point: a trend, the current state of something that changes, or a comparison across items or across time.
- **If people only need the values, use a list or table** they can scroll, search and sort. A chart is for conveying information about the data, not for delivering it.
- **Charts earn their place** in three common cases: values over time, past or predicted; part of a whole, such as progress toward a goal; and comparison between items or categories.
- **Start from the goal, and narrow it.** Write the question the chart answers ("how were sales in the last 30 days?"), list what it could show (pattern, range, individual values, the maximum, outliers, comparisons), and choose the few that matter most to the people using it. An effective chart focuses on a few pieces of information.
- **Look at the data from three distances** for things worth showing: the whole (a total or an average, as a headline or a `RuleMark`), subsets (a period, a category), and single values (the best day, as a highlighted `PointMark` or an annotation).
- **Design with real data, and for its awkward cases.** A form that looks right on smooth invented data can fail on the real thing. Check days with zero, gaps, outliers and very small values, and the case of no data at all, or only zeros: an empty chart draws nothing, and a chart of zeros whose marks grow from zero (bars, areas, sectors) draws axes with no marks under a headline that is no longer true. Points at zero are still drawn, so a scatter plot of zeros is a chart, not an empty state. Show a placeholder that says why, such as `ContentUnavailableView` (iOS 17.0+), in place of the `Chart` view, and make the headline and the chart descriptor's summary handle the same case.
- **Keep it simple and let people ask for more.** Do not pack in everything available. Offer detail gradually: a period switch, a subset picker, a larger version behind a tap.

## Marks

- **Choose the mark by what the chart should communicate,** not by the shape of the data: bars for comparison and for totals per period, a line for change over time, points for individual values and how two properties relate. The full table is *Choosing a mark* in `marks.md`.

- **How bars are arranged changes what they say:** in a row along time, change; stacked, the parts of a total; side by side, a comparison between categories.
- **Lines overstate what lies between distant points.** With gaps or zero days, the segments joining far-apart values become more prominent than the values. Bars keep each day separate and show zeros without distraction, and when values are totals the amount of bar on screen matches the amount sold.
- **Combine marks when it adds clarity.** Points on a line show the trend and the individual readings together: `symbol` on a `LineMark` (`mark-modifiers.md`). A line over a faint filled area gives the trend more weight (`gallery/area.md`).
- **Add a dimension only when a question needs it.** Splitting bars by a second property answers "and how is each made up?". Leave the second encoding out unless that question is being asked.
- **Prefer common chart types.** People already know how to read bars and lines. A scatter plot is less familiar and may need a sentence of guidance. A novel form has to be taught, for instance by introducing its parts one at a time, and belongs at the centre of the app, not in a supporting chart.
- **Pie and donut charts** suit casual settings where precision is not critical and the point is how parts make up a whole. Their round shape is approachable and reads well large. Keep to 5 to 7 sectors and sum the rest into "Other"; with more, or when people need to compare values accurately, use horizontal bars.
- **3D only when it earns interaction.** A 3D chart works when the shape of the data matters more than exact values, most naturally when the data is itself three-dimensional, such as positions in space. It has to be rotated to be understood, so use one only where requiring that interaction improves the experience (`chart3d.md`).

## Axes

- **Defaults first.** The automatic scales, axes and legend suit most charts. Customise when a default misleads or the design calls for something specific.
- **Fixed or dynamic range, by meaning.** Use a fixed range when the smallest and largest possible values mean something for any data: a battery runs from 0% to 100% whatever today's readings are (`chartYScale(domain: 0 ... 100)`). Use a dynamic range, which is the default, when values vary widely and the marks should fill the plot: the top of a step-count axis follows the largest count in the period shown. See `ScaleDomain` in `scales.md`.
- **Choose the lower bound from the mark and the use.** Bars should start at zero, because people compare their lengths. Lines and points far from zero should not: a heart-rate chart from zero hides the difference between resting and active readings (`.automatic(includesZero: false)`).
- **Use familiar sequences for labels.** 0, 5, 10 is read at a glance; 1, 6, 11 follows the same rule and makes people stop and work it out. Check what a custom domain or stride produces (`AxisMarkValues` in `axes.md`).
- **Fit grid lines and labels to the chart's job.** Too many distract from the data; too few make values hard to estimate. Apple's worked example settles on about four horizontal grid lines for a detail chart, at round steps people count in (multiples of 20; weeks on a 30-day axis). A preview chart needs none. Where people can inspect values by touching the chart, use fewer grid lines and lighter labels so the data stays prominent:

<!-- snippet: Design/HIGPatterns.swift#quiet-axis -->
```swift
func quietAxes() -> some View {
    chartYAxis {
        AxisMarks(values: .automatic(desiredCount: 3)) { _ in
            AxisGridLine().foregroundStyle(Color.secondary.opacity(0.2))
            AxisValueLabel().foregroundStyle(Color.secondary.opacity(0.6))
        }
    }
}
```

- **In a narrow space, give the width to the plot.** Keep value labels short (`1M`, not `1,000,000`; `axes.md`, `AxisValueLabel`), move the unit into the chart's heading, and put long category names inside the plot area, which is where Swift Charts puts them by default on a horizontal bar chart.

- **Keep a clean leading edge** so the chart lines up with the views around it. The defaults already do this: the value axis is on the trailing side, and each time label sits on the trailing side of its grid line. Moving the value axis to the leading edge gives that up; do it only when the design calls for it.

## Descriptions

- **Say what the chart is before people read it.** A title and labels that state its purpose give everyone the context to interpret it, and matter most to people using VoiceOver and people with some cognitive disabilities, who decide from the description whether to explore further.
- **Summarise the main message in words.** A headline such as "Heavy rain in the next hour" or "1,577 steps on average" gives the takeaway without reading the chart. The chart is the supporting evidence. Read on its own, the text should still inform. In rising order of usefulness:

| Description | Example |
|---|---|
| A title that only names the chart | Sales in the past 30 days |
| A title that carries the key value | Total sales: 1,234 pancakes |
| A sentence, when the chart is not straightforward | Sales in the past 30 days totaled 1,234 pancakes |
| An interpretation, when the data is unfamiliar | Sales for the past 30 days are up 12%, totaling 1,234 pancakes |

- **Prefer a heading to an axis title for saying what is measured.** An axis title is small and off to the side; a heading puts the meaning front and centre. Add an axis title (`axes.md`) when the heading cannot carry the unit, or when the two axes need telling apart, as on a scatter plot. Context can also come from the interface around the chart: the screen's title, the label of a period switch.
- **Words that name a direction** ("left to right", "on the right") are false in a right-to-left layout, where the chart mirrors. Read `layoutDirection`, or write "in order" and "before" instead.
- **A headline is true of what is drawn.** When a control changes the chart, the headline is worked out again from what the chart now shows. The same holds for any note beside the chart: base it on what is in effect (the text size and layout direction the chart is drawn in), not on a control's setting.
- **A headline is not an accessibility label.** It helps, and each still has to be written (`accessibility.md`).
- **Label directly where possible.** A value written on a rule, or a name and value in the centre of a donut, is read faster than a legend lookup (`annotations.md`). Hide the legend when direct labels make it redundant.
- **Establish a hierarchy: the data first.** Marks are the most prominent thing; descriptions and axes support them without competing. Secondary text styles and light grid lines keep that order.

The pattern Apple's own apps use for a chart with its summary is a caption, a large figure with its unit, the period covered, then the chart:

![The caption AVERAGE, a bold 167 cups, the dates Mar 1 to 7, 2026, and a bar chart of seven days with its value axis.](../assets/gallery/summary-card.png)

<!-- snippet: Design/HIGPatterns.swift#summary-card -->
```swift
struct SummaryCard: View {
    let days: [DailySales]

    private var average: Int {
        days.isEmpty ? 0 : days.map(\.cups).reduce(0, +) / days.count
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 2) {
            Text("AVERAGE")
                .font(.caption)
                .foregroundStyle(.secondary)
            Text("\(average, format: .number) ")
                .font(.title.bold())
                // foregroundColor keeps this a Text, so the two can be joined on iOS 16.
                + Text("cups").font(.headline).foregroundColor(.secondary)
            if let first = days.first?.day, let last = days.last?.day {
                Text(first ..< last, format: .interval.month().day().year())
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
            Chart(days) { day in
                BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
            }
            .padding(.top, 6)
        }
    }
}
```

## Interaction

- **Let people explore, but never hide critical information behind interaction.** The chart at rest has to communicate its message. Selection reveals detail on top of that: dragging an indicator along a line to read the value at a time (`interaction.md`).
- **Make the whole plot the target.** Individual marks are often too small to hit with a finger or pointer. Let people scrub anywhere across the plot area to move the selection. `chartXSelection` works this way; a custom gesture should too (`interaction.md`).
- **The title gives way to the selection.** A selection label can occupy the space above the plot where the title and legend sit, with those hidden while a selection is active. The layout does not jump when a selection appears.
- **Make it navigable by keyboard and Switch Control,** and help people notice when a chart changes. Both are covered in `accessibility-beyond-voiceover.md`.

## Colour

- **Never rely on colour alone** to tell data apart or to carry essential information. Add a second difference: symbol shape for points and lines (`symbol(by:)`), dash pattern for lines (`lineStyle(by:)`), or direct labels. For example, one measure as red circles and the other as black diamonds.

<!-- snippet: Design/HIGPatterns.swift#two-series-points -->
```swift
struct TwoSeriesPoints: View {
    let stores: [StoreSales]

    var body: some View {
        Chart {
            ForEach(stores) { store in
                ForEach(store.days) { day in
                    PointMark(
                        x: .value("Day", day.day, unit: .day),
                        y: .value("Cups", day.cups)
                    )
                }
                // Colour and shape together, so the series can be told apart without colour.
                .foregroundStyle(by: .value("Store", store.store))
                .symbol(by: .value("Store", store.store))
            }
        }
        .chartYScale(domain: .automatic(includesZero: false))
    }
}
```

- **Emphasise one thing.** To draw the eye to one element, keep it at full strength and fade the rest (opacity around 0.3), or grey the supporting marks and colour the one that matters. When an overlay such as an average line is switched on, mute what is underneath so there is one focus at a time.
- **Assign meaningful colours explicitly.** Automatic colours are assigned in order and mean nothing. When a category has a colour people already associate with it, set it; a mismatch (the bar for "green" drawn in pink) actively misleads (`scales.md`).
- **Balance the visual weight of category colours.** A vivid colour beside a dark one makes its series look more important. Even out saturation and brightness unless a hierarchy is intended; when one series should stand out, a heavier colour is the tool.
- **Choose colours that are easy to name apart** (purple and green, not purple and purple-pink) and that contrast with the background.
- **Colour meanings are cultural.** Green for gains and red for losses is expected in some countries and reversed in others, so do not hard-code that pairing in an app that ships widely; let it follow the locale or a setting.
- **Separate neighbouring areas of colour.** Where coloured segments touch, as in a stacked bar, a thin gap between them helps people see each one (`gallery/stacked-bar.md`). For a pie or donut it is `angularInset` (`marks.md`).

## Charts in an app experience

- **Size follows function:**

| Size | Typically | Has |
|---|---|---|
| Small | A widget, a watch complication, a thumbnail in a list, a preview platter | Static. No grid lines, labels or interaction; it shows what can be taken in at a glance and promises more detail a tap away, on this device or a larger one |
| Medium | The width of the view, not its full height | Axis lines and labels, selection for exact values, a period switch |
| Large | Most of the screen | Deep exploration: overlays, summaries, several controls |

- **Reveal complexity progressively.** Put a small static chart high in the navigation and let people move to fuller versions as their interest grows.
- **Overview and detail.** A compact chart in a list row opens a detail screen with the same data in a larger chart (around 240 points tall on iPhone) with axes and interaction. The overview pairs a caption, a headline stating the takeaway in words and numbers, and a small chart with axes and legend hidden (`gallery/compact.md`). The headline above a detail chart reflects its current state: the visible range of a scrolling chart, or the selected element.
- **Keep continuity between charts of the same data.** The preview and the full version use the same chart type, colours, annotations and wording, so people recognise them as one thing. Someone who taps a chart wants more of what they saw: keep its shape and every number that was visible, and add to it.
- **Be consistent across charts with a similar purpose.** One type and one style let people carry what they learned from one chart to the next. Differ only to signal a real difference, and make the difference as large as the change:

| What changed between two charts | Change in the design |
|---|---|
| The period | The description and the data |
| The kind of data | Also a different colour |
| The measure and how it is represented | Also a different mark style or orientation |

- **Several aligned charts, one per measure,** as on Apple's Activity screen, show related measures side by side without forcing them onto one scale (`gallery/small-multiples.md`).
- **Link related charts.** A selection in one chart highlighted in the others lets people follow one period or item across several views (`gallery/linked-charts.md`).
- **A chart can be the legend.** In a dashboard, a small bar chart of the categories, coloured by the shared scale, tells people what each colour means and gives them something to tap; the other charts then hide their own legends.
- **Keep category order the same** in every view of the same data.
- **Keep colours the same** in every chart and every filtered state: fix the colour scale's domain (`scales.md`).

## Pitfalls

- A chart where a table was wanted, or a chart with no point.
- Everything available packed into one chart.
- A compact chart with hidden axes and no accompanying figure: a shape without a meaning.
- Changing the order of categories, or the chart type, between two views of the same data.
