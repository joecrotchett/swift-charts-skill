# Chart accessibility beyond VoiceOver

What a chart owes people who navigate without touch, who do not tell colours apart, who use large text, or who cannot see a change happen. VoiceOver elements and labels are in `accessibility.md`.

## Navigation

Keyboard commands, Full Keyboard Access and Switch Control visit elements one after another. In an interactive chart, make that path logical:

- Give the elements a predictable order, such as along the x axis, instead of one that jumps back and forth. On iOS the accessibility tree orders the elements by their place on screen, not by the data (read on the iOS simulator). Bars and lines along a category or date axis come out along x, which is usually the order wanted, the labelled segments of stacked bars bar by bar, each from the bottom segment up, and a labelled rule or band after all the bars. Everything else comes out by the tops of the elements' frames. A scatter plot's points come out highest first, and sorting the data by x does not change that (the Audio Graph does follow the data order); a rule or band among points takes its place by its top like them (a target line sixth of 21; a shaded region spanning the plot, first). Portions in labelled `Plot`s are ordered the same way unless each holds a clear full-height band, which puts them along x (`accessibility.md`, *One element per `Plot`*). A donut's sectors come out by their tops too: drawn largest first from 12 o'clock (Tea, Latte, Americano, Mocha, Cocoa), they came out Cocoa, Tea, Mocha, Americano, Latte, on every screen that drew them; chart content has no sort-priority modifier, and no way to reorder sectors has been read. Read your chart's order before describing it, and say it truthfully rather than promising one.
- For a very large dataset, let focus move between subsets of values, not through every point.

Both also improve VoiceOver, including on charts that are not interactive.

Every interaction designed for touch or pointer needs an equivalent for keyboard, Voice Control, Switch Control and VoiceOver. Make focus as easy to see as touch is to land: a focus indicator the full height of the plot, like the touch target.

Make the whole plot the target for touch too (`design.md`, *Interaction*).

## Colour, contrast and differentiation

- Never rely on colour alone, and separate neighbouring areas of colour: the second cue for each mark, the `.grayscale(1)` check and the gaps between segments and sectors are under *Colour* in `design.md`. The chart must still work with Differentiate Without Colour turned on.
- Choose colours that contrast with each other and with the background, check them with colour-blindness filters, and make sure they adapt to Dark Mode and Increase Contrast.

## Changes and motion

When marks or axes change (a new period, a new scale, a different kind of mark), people who do not notice misread the chart. Animation draws attention to the change, and is not enough: people using VoiceOver and people who have turned animations off need to be told another way. Post an accessibility announcement when the chart's content changes, saying what changed and what the chart now shows. Build the message from every control that redraws the chart, or the one left out changes it silently. The rest of this paragraph is this skill's convention, not Apple's: announce a change to the chart's data, scale or kind of mark, as one sentence on what changed followed by the headline; a control that only restyles the marks or moves labels, and already speaks its own value (a slider, a picker for where row names go), needs no announcement, unless the headline changes with it. A scale's range counts as restyling (padding, the range of symbol sizes); its domain, its type and which colour or shape each value gets change what the chart says, and are announced. Whatever changes the headline is announced, with one exception: what a reader's own touch moves, a selection or a scroll position, is not announced, since it changes the headline on every step of a drag and the reader is already looking at it. A tap whose effects land in other views, out of sight (a dashboard's other panels), is announced, once per tap. The same days reach VoiceOver users through the chart's elements, which they move through without a gesture. What a control shows and what it speaks describe the setting the same way (not "6 points" on screen and "6 points each way" aloud); a spoken value may add the headline after it, as below. A slider or stepper that changes what is drawn should not post one on every step either: make its spoken value the setting followed by the headline (or the headline alone when the control's label already shows the setting, as a stepper in a row showing its count does, or the count is spoken twice):

<!-- snippet: Accessibility/AccessibilitySnippets.swift#announce-change -->
```swift
/// Says aloud that a chart changed, for people who cannot see it happen.
@MainActor
func announceChartChange(_ message: String) {
    if #available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *) {
        AccessibilityNotification.Announcement(message).post()
    } else {
        #if canImport(UIKit) && !os(watchOS)
        UIAccessibility.post(notification: .announcement, argument: message)
        #endif
    }
}
```

A chart whose data changes many times a second, such as a live level meter, takes `accessibilityAddTraits(.updatesFrequently)`: the trait says the label or value changes too often for notifications, so an assistive technology can poll it instead.

## Dynamic Type and layout

The framework's own axis labels and legend follow Dynamic Type without any code (`composition.md`), but the legend does not keep its names whole: it wraps from XXX Large up, yet still clipped "Mocha" at `.accessibility3` and dropped a name, leaving its swatch alone, at `.accessibility5`, and in a fixed-height chart its extra lines come out of the plot. At those sizes give it your own legend that wraps (`legend.md`, *A legend of your own*). The number of labels does not: at the accessibility sizes, labels that fitted can collide or be cut short. On a horizontal bar chart the category names sit inside the plot, above each row, and take their height from the bars: in a fixed-height chart the bars shrink to strips at the largest sizes. Let the chart's height grow with the text (`@ScaledMetric`), or move the names outside the plot (`axes.md`, `AxisMarkPreset`). No single text size marks where labels start to collide: it depends on the labels' length and the chart's width, so check every size the app supports, not one.

For the labels along an axis, read `dynamicTypeSize` from the environment and ask for fewer. `desiredCount` is only a wish (3 on a 0 to 300 axis still gave four labels); an exact array of values is always drawn in full, so thin the array itself. At the largest sizes fewer may still not be enough for a date label such as "Mar 1" on a phone: shorten the format as well (the day alone). A label at the plot's trailing edge needs the room `axes.md` describes:

<!-- snippet: Community/CommunityPatterns.swift#dynamic-type-axis -->
```swift
struct DynamicTypeAwareAxis: View {
    let days: [DailySales]
    @Environment(\.dynamicTypeSize) private var dynamicTypeSize

    /// Fewer labels as the text grows, so they do not collide.
    private var daysPerLabel: Int {
        if dynamicTypeSize >= .accessibility3 { return 7 }
        if dynamicTypeSize >= .xxLarge { return 2 }
        return 1
    }

    var body: some View {
        Chart(days) { day in
            BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .day, count: daysPerLabel)) { _ in
                AxisGridLine()
                // Centred only when every day is labelled: with a longer stride the label would
                // sit under the middle of the stride, not under its own day.
                AxisValueLabel(format: .dateTime.weekday(.abbreviated), centered: daysPerLabel == 1)
            }
        }
    }
}
```

The value axis's top label is centred on the top of the domain, so in a chart inside a `ScrollView` it is cut in half; give the scale room above it that grows with the text (`chartYScale(domain:range: .plotDimension(endPadding:))` with a `@ScaledMetric` amount, capped, or it squeezes the plot at the largest sizes). `@ScaledMetric` follows a `dynamicTypeSize` set on the view, not only the device's. Test custom label views and annotations at the largest sizes too: a value written above a bar can crowd the value axis's labels, and `overflowResolution: .init(x: .fit(to: .plot))` keeps it inside the plot (read at `.accessibility5`). On x that slides a label at a bar's end back over the bar; to keep it clear, give the scale room that grows with the text, `range: .plotDimension(endPadding:)` with a capped `@ScaledMetric`.
