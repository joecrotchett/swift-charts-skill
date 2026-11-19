# Interaction

Selection, scrolling and custom gestures.

## Selection

### `chartXSelection` - iOS 17.0+

**Use for:** letting people touch or drag across the chart to pick a position on the x axis. `chartYSelection` is the same for y.

- `chartXSelection(value:)` binds an optional value of the x axis type. It is `nil` when nothing is selected.
- `chartXSelection(range:)` binds an optional `ClosedRange`.

Each has a default gesture that differs by platform, and a chart can carry both modifiers at once:

| Selection | iOS | macOS |
|---|---|---|
| A value | Touch and hold, or press and drag | Hover |
| A range | Two-finger tap (not yet checked by a press) | Drag |

**On iOS a value selection lasts only while the finger is down** (pressed on the iOS simulator, chart inside a vertical `ScrollView`). A tap selects nothing, lifting the finger sets the binding back to `nil`, and a vertical swipe that starts on the plot scrolls the page and leaves no selection. Two consequences:

- Nothing that acts only on a live selection can sit outside the chart. A toggle or slider that changes how the selection is drawn cannot be reached while the selection exists, so it changes nothing anyone sees.
- Whatever a selection reveals has to be reachable another way, normally through the chart's own elements (`accessibility.md`), since a VoiceOver user does not hold a finger on the plot.

<!-- snippet: Interaction/ProxySnippets.swift#value-and-range -->
```swift
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
extension View {
    /// A chart can offer both: a single value and a range, each with its own default gesture.
    func selectable(value: Binding<Date?>, range: Binding<ClosedRange<Date>?>) -> some View {
        chartXSelection(value: value)
            .chartXSelection(range: range)
    }
}
```

The modifier only reports. Drawing the indicator and the label is your job, and they are drawn as marks inside the chart: a `RuleMark` at the selection with an `annotation` holding the label, added with `if let` while a selection exists.

The bound value is the exact axis position under the finger, not a data point: on a date axis it has a time of day. Snap it to the data in a computed property and draw from the snapped value.

Recipe: `gallery/selection-line.md`.

**Pitfalls:**

- Using the raw value to look up data finds nothing, because it rarely equals a data point.
- Drawing the indicator in an overlay instead of as a mark; it then needs manual positioning and drifts on resize.

### Selecting on both axes

`chartXSelection` and `chartYSelection` can be bound on the same chart. Together they give the position of the touch in data values, which is how a scatter plot or a map selects the point nearest the finger. Accept the nearest point only within a small distance, so touching empty space selects nothing, and hide the highlight mark from assistive technologies, since it repeats a point that is already described.

Measure that distance in a unit the two axes share. In data units, an axis spanning hundreds swamps one spanning ten: on a chart of waits (0 to 360 seconds) against queue length (0 to 12 people), one person is about 25 points on screen and one second about half a point. Divide each axis's difference by its span, as below, or for a radius in screen points convert both positions with the proxy (`plot-area.md`).

<!-- snippet: Dashboard/DashboardPatterns.swift#nearest-point -->
```swift
/// Selecting on both axes at once, to find the point nearest the touch.
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct NearestPointScatter: View {
    let beans: [Bean]
    @State var selectedAcidity: Double?
    @State var selectedBody: Double?

    private var nearest: Bean? {
        guard let selectedAcidity, let selectedBody else { return nil }
        // The axes have different units, so measure each as a fraction of its own extent:
        // a step of one unit of body and one of acidity are not the same distance on screen.
        func span(_ values: [Double]) -> Double {
            max((values.max() ?? 0) - (values.min() ?? 0), .ulpOfOne)
        }
        let acidityRange = span(beans.map(\.acidity)), bodyRange = span(beans.map(\.body))
        func distance(_ bean: Bean) -> Double {
            hypot((bean.acidity - selectedAcidity) / acidityRange, (bean.body - selectedBody) / bodyRange)
        }
        // Nothing is selected unless the touch is close to a point: here within 5% of the plot.
        return beans.filter { distance($0) < 0.05 }.min { distance($0) < distance($1) }
    }

    var body: some View {
        Chart {
            ForEach(beans) { bean in
                PointMark(x: .value("Acidity", bean.acidity), y: .value("Body", bean.body))
                    .opacity(0.5)
            }
            if let nearest {
                PointMark(x: .value("Acidity", nearest.acidity), y: .value("Body", nearest.body))
                    .foregroundStyle(Color.primary)
                    .symbolSize(90)
                    // The highlight repeats a point that is already described.
                    .accessibilityHidden(true)
                    // Fitted to the plot, so near the top it does not cover the axis titles.
                    .annotation(position: .top, overflowResolution: .init(x: .fit(to: .plot), y: .fit(to: .plot))) {
                        Text("\(nearest.origin), sweetness \(nearest.sweetness, format: .number.precision(.fractionLength(1)))")
                            .font(.caption)
                            .padding(4)
                            .background(.thinMaterial, in: .rect(cornerRadius: 4))
                    }
            }
        }
        .chartXSelection(value: $selectedAcidity)
        .chartYSelection(value: $selectedBody)
    }
}
```

### A band as the selection indicator

For bars, a `RectangleMark` with only an x value and `width: .ratio(1)` shades the whole selected unit from top to bottom, behind the bars. Give it the same `unit:` as the bars. As a `ChartContent` type it can be reused in every chart that shows the selection. Code: `gallery/linked-charts.md`.

### Linking charts

Charts that bind `chartXSelection` to the same value share a selection: touching one highlights the same position in all of them, and other views can filter to it. Keep the value in a shared model when the charts live in different views. The charts need the same x domain, so the shared value falls at the same place in each: give them the same `chartXScale(domain:)` unless they are certain to cover the same span, and the same y domain where they can, so their axis labels and so their plots are the same width. A chart with no record for the selected day draws no band there; say so in any text that promises a band in every chart. Recipe: `gallery/linked-charts.md`.

### `chartAngleSelection` - iOS 17.0+

**Use for:** letting people touch and hold, or drag, around a pie or donut. On iOS it behaves like a value selection (pressed on the iOS simulator): a held press selects, a tap leaves nothing selected, and lifting the finger sets the binding back to `nil`, so a label in the hole that names the selection shows only while the finger is down.

`chartAngleSelection(value:)` binds an optional number; the recipe binds `Double?` for sectors whose angle is an `Int`. The value is a position along the running total of all sectors' values, not the touched element. With sectors of 916, 760 and 540, a touch in the second sector reports something between 916 and 1676. Map it to an element by accumulating the values until the total passes it.

Recipe: `gallery/donut.md`.

**Pitfalls:**

- Treating the value as an element's own value selects the wrong sector or none.
- **Read the selection in the view's body, not only inside `Chart(data) { item in ... }`.** That closure runs outside the body, so when the selection lives in an `@Observable` model and is read only there, a change does not redraw the marks: under a held press the headline above changed and the sectors did not fade (pressed on the iOS simulator). Reading it in the body and passing plain values into the closure (the set of faded sectors, the labels) fixed it.

## Scrolling

### `chartScrollableAxes` - iOS 17.0+

**Use for:** making a chart scroll along an axis, for a time range too long to show at once. It is used with three companions:

| Modifier | Does |
|---|---|
| `chartScrollableAxes(.horizontal)` | Turns scrolling on for the axis. |
| `chartXVisibleDomain(length:)` | How much of the axis is visible at once. On a date axis the length is in seconds. |
| `chartScrollTargetBehavior(_:)` | Where scrolling comes to rest: see `ChartScrollTargetBehavior` below. |
| `chartScrollPosition(x:)` | Binds the leading edge of the visible window. |

Read the bound position to describe what is on screen: the window is from the position to the position plus the visible length, and a headline can total the data inside it.

Recipe: `gallery/scrolling-bar.md`.

**Pitfalls:**

- The position is the window's leading edge, not its centre.
- A scrolling chart cannot be captured with `ImageRenderer`: it comes out with its value axis and grid but no marks. For an image, draw a second chart without the scrolling modifiers, its x domain fixed to the visible window: from the scroll position to the position plus the visible length.
- Its `accessibilityChartDescriptor` is ignored: the Audio Graph gets the framework's default ("Chart", no summary, dates as numbers). Put an `accessibilityRepresentation` on it holding a non-scrolling chart of all the data with the descriptor inside; read on iOS, that descriptor is kept. Every mark is an element either way, scrolled out of view or not, so say whether a headline describes the window or the whole.
- On screen it drew no x axis title and cut the y title short (iOS 27): put the unit in the heading or caption.
- Laid out first at another width (a split view collapsing on iPhone), it keeps its offset in points and opens on the wrong window; a full x domain did not help. Rebuild it at its settled width, `.id(width)` with the width read by `onGeometryChange`.

### `ChartScrollTargetBehavior` - iOS 17.0+

**Use for:** making a scrolling chart come to rest on meaningful values instead of anywhere. Written `.valueAligned(...)`:

| Parameter | Sets |
|---|---|
| `matching: DateComponents(hour: 0)` | Date axis: rest only where the date matches the components, here midnight, so whole days. Name the data's calendar and time zone in the components |
| `unit: 10` | Number axis: rest on multiples of the unit |
| `majorAlignment:` | Where a swipe, as opposed to a slow drag, lands: `.page` (one visible window on), `.matching(DateComponents(day: 1))` (the next first of the month) or `.unit(100)` |
| `limitBehavior:` | Whether one gesture is limited in how far it can travel: `.automatic` (limited when the chart is compact along the scrolling axis), `.always`, `.never` |

Forms with `xMatching`/`xUnit` and `yMatching`/`yUnit` cover charts that scroll both ways.

<!-- snippet: Interaction/ProxySnippets.swift#scroll-behaviors -->
```swift
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
extension View {
    /// Rest on whole days; a swipe moves a page (one visible window) at a time.
    func scrollByPagesOfDays() -> some View {
        chartScrollTargetBehavior(
            .valueAligned(matching: DateComponents(hour: 0), majorAlignment: .page)
        )
    }

    /// A number axis: rest on multiples of 10, and let a hard swipe travel as far as it likes.
    func scrollByTens() -> some View {
        chartScrollTargetBehavior(
            .valueAligned(unit: 10, limitBehavior: .never)
        )
    }
}
```

`valueAligned(matching:)` takes no calendar. On a simulator six hours behind UTC, charts of days recorded at UTC midnight came to rest on UTC day boundaries, with `DateComponents(hour: 0)` alone and with a UTC calendar and time zone set in the components; they did not rest at the device's midnight. Name the data's calendar in the components anyway, so the rule says which midnight is meant.

For behaviour none of these gives, conform a type to `ChartScrollTargetBehavior` and implement `updateTarget(_:context:)`. The context has the scroll view's usual properties and a `chartProxy` for converting the proposed offset to data values.

### Scrolling below iOS 17

The whole chart moves, so its value axis scrolls out of view; put that axis on the leading edge so it is seen first, or draw it as a separate fixed chart beside the scroll view. There is no snapping and no reported scroll position.

<!-- snippet: Community/CommunityPatterns.swift#scroll-ios16 -->
```swift
/// Scrolling on iOS 16, which has no scrolling modifiers: a chart wider than the screen inside a scroll view.
/// The whole chart moves, its value axis included.
struct ScrollingChartFallback: View {
    let days: [DailySales]
    /// Points of width given to each day.
    var dayWidth: CGFloat = 24

    var body: some View {
        ScrollView(.horizontal) {
            Chart(days) { day in
                BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
            }
            // Leading, so the axis is visible before any scrolling.
            .chartYAxis { AxisMarks(position: .leading) }
            .frame(width: CGFloat(days.count) * dayWidth)
        }
    }
}
```

## Custom gestures

Interaction the selection and scrolling modifiers do not cover is built with a gesture of your own and `ChartProxy` (`plot-area.md`), which turns the gesture's location into a data value.

**On iOS 17 and later, drive the chart's own selection.** `chartGesture` receives the proxy; call `selectXValue(at:)` (or `selectXRange(from:to:)`, the y forms and the angle form, all iOS 17) and the value arrives through the same `chartXSelection` binding, so the rest of the chart is written exactly as for the built-in gesture. `chartGesture` overrides the default selection gesture. Use it to select on tap instead of on drag, or with a long press. Its gesture's location is measured from the plot area, so it goes to the proxy as it is. A selection made this way stays after the finger lifts (pressed), unlike the built-in one, so give the reader a way to clear it; a tap on the axis labels selects nothing:

<!-- snippet: Interaction/ProxySnippets.swift#chart-gesture -->
```swift
/// A custom gesture that drives the framework's own selection (iOS 17).
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct TapToSelectChart: View {
    let days: [DailySales]
    @State private var selected: Date?

    var body: some View {
        Chart {
            ForEach(days) { day in
                BarMark(x: .value("Day", day.day, unit: .day), y: .value("Cups", day.cups))
            }
            if let selected {
                RuleMark(x: .value("Selected", selected, unit: .day))
                    .foregroundStyle(Color.secondary)
                    .accessibilityHidden(true)
            }
        }
        .chartXSelection(value: $selected)
        .chartGesture { proxy in
            SpatialTapGesture()
                .onEnded { tap in
                    proxy.selectXValue(at: tap.location.x)
                }
        }
    }
}
```

**On iOS 16, or for anything that is not selection,** put a clear, hit-testable view in `chartOverlay`, attach the gesture to it, subtract the plot area's origin from its location, reject positions outside the plot, and convert the rest with the proxy. Selecting a range by dragging (brushing) is the same technique with two positions: `gallery/selection-line.md`.

**When the target is iOS 16 and the design needs selection,** do both: gate the framework's modifier and fall back to the overlay gesture, each writing the same binding. People on iOS 17 get the system gesture; people on iOS 16 still get the interaction.

<!-- snippet: Interaction/ProxySnippets.swift#selection-fallback -->
```swift
extension View {
    /// Selection on every supported OS: the framework's modifier where it exists, a gesture in an overlay below that.
    /// Both write the same binding, so the chart's marks are written once.
    @ViewBuilder
    func dateSelection(_ selection: Binding<Date?>) -> some View {
        if #available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *) {
            chartXSelection(value: selection)
        } else {
            chartOverlay { proxy in
                GeometryReader { geometry in
                    Rectangle().fill(.clear).contentShape(Rectangle())
                        .gesture(
                            DragGesture(minimumDistance: 0)
                                .onChanged { drag in
                                    let x = drag.location.x - geometry[proxy.plotAreaFrame].origin.x
                                    // Outside the plot a date scale still answers, with a date beyond the data.
                                    guard (0 ... proxy.plotAreaSize.width).contains(x) else { return }
                                    selection.wrappedValue = proxy.value(atX: x, as: Date.self)
                                }
                                .onEnded { _ in selection.wrappedValue = nil }
                        )
                }
            }
        }
    }
}
```

**Pitfalls:**

- Forgetting to subtract the plot area's origin in an overlay gesture (`plot-area.md`).
- An overlay that is not hit-testable receives nothing. Give the clear shape a `contentShape`.
- **A drag gesture in `chartOverlay` takes the page's vertical swipes.** In a vertical `ScrollView`, a swipe that starts on the chart does not scroll the page; it starts the drag. A SwiftUI `LongPressGesture` sequenced before the `DragGesture` does not fix it, on `.gesture` or `.simultaneousGesture` (both pressed on the iOS simulator). The built-in selection does not have the problem, so prefer it. Where an overlay drag is needed on iOS 18 or later, a UIKit `UILongPressGestureRecognizer` attached through `UIGestureRecognizerRepresentable`, with a short minimum press, lets a swipe scroll the page and a held drag select (pressed).
