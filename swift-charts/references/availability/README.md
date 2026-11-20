# API availability

Which Swift Charts API exists on which OS version. This file is the overview; the complete tables are one file per release, generated from the SDK.

## Using this

1. Find the app's deployment target for each platform it ships on.
2. Every API from a later release is **gated**. The table under *Designing for an older deployment target* says what that rules out, and every API heading in the topic references carries its release. Open a release file (`ios-17.md`, `ios-18.md`, ...) only to look up an API's exact forms or its versions on other platforms.
3. Use a gated API only inside `if #available` with a fallback, or inside a type marked `@available`. See *Availability patterns*.
4. Never raise the deployment target to reach an API without asking.
5. When a design depends on a gated API, say what people on the older OS will see instead.

An API name that appears in none of the release files is not Swift Charts API. Check the spelling, or it belongs to SwiftUI.

## Releases

| Release file | iOS / iPadOS | macOS | tvOS | watchOS | visionOS | Headline additions |
|---|---|---|---|---|---|---|
| `ios-16.md` | 16.0 | 13.0 | 16.0 | 9.0 | 1.0 | The framework: `Chart`, the six marks (`AreaMark`, `BarMark`, `LineMark`, `PointMark`, `RectangleMark`, `RuleMark`), scales, axes, legends, annotations, `chartOverlay` / `chartBackground` with `ChartProxy`, binning. |
| `ios-16.md` | 16.4 | 13.3 | 16.4 | 9.4 | 1.0 | `blur` and `shadow` on marks; `ScaleType` `power`, `squareRoot`, `symmetricLog`; `ChartProxy` lookups for domains and resolved styles. |
| `ios-17.md` | 17.0 | 14.0 | 17.0 | 10.0 | 1.0 | `SectorMark` (pie and donut); selection (`chartXSelection`, `chartYSelection`, `chartAngleSelection`); scrolling (`chartScrollableAxes`, visible domain, scroll position, scroll target behavior); `chartGesture`; annotation overflow resolution; `zIndex`; `compositingLayer`; `chartXAxisStyle` / `chartYAxisStyle`. |
| `ios-18.md` | 18.0 | 15.0 | 18.0 | 11.0 | 2.0 | Vectorized plots (`AreaPlot`, `BarPlot`, `LinePlot`, `PointPlot`, `RectanglePlot`, `RulePlot`, `SectorPlot`); function plots on `LinePlot` and `AreaPlot`; `PlottableProjection`; `MarkDimensions`. |
| `ios-26.md` | 26.0 | 26.0 | none | none | 26.0 | 3D: `Chart3D`, `SurfacePlot`, `z` initializers on `PointMark`, `RectangleMark`, `RuleMark`, `chart3DPose`, Z axis, scale and selection. No 3D API exists on tvOS or watchOS. |
| `ios-27.md` | 27.0 | 27.0 | 27.0 | 27.0 | 27.0 | No new chart API. SwiftUI's `EmptyView` and `TupleContent` conform to `ChartContent`. |

The framework is on every visionOS release, so API from iOS 16 and 17 needs no visionOS check.

## Platform differences

- **tvOS and watchOS have no 3D charts.** Everything in `ios-26.md` marked *Not on tvOS, watchOS* must be excluded from those builds with `#if os(...)`, not only gated at run time.
- `chart3DCameraProjection` and `Chart3DCameraProjection` are iOS and macOS only.
- `chart3DRenderingStyle` and `Chart3DRenderingStyle` are visionOS only.

## Deprecated and changed

| API | Change | Use instead |
|---|---|---|
| `ChartProxy.plotAreaFrame` | Deprecated in iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0 | `ChartProxy.plotFrame` (iOS 17.0+) |
| `ChartProxy.plotAreaSize` | Deprecated in iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0 | `ChartProxy.plotSize` (iOS 17.0+) |

With a deployment target below iOS 17 the replacements are themselves gated, so the deprecated properties remain the correct choice outside an `if #available` block.

## Designing for an older deployment target

What a deployment target rules out. Find every row whose range includes the app's target: what it lists is unavailable there without gating.

| Minimum OS | Not available | What exists instead at that OS |
|---|---|---|
| iOS 16.0 to 16.3 | `blur` and `shadow` on marks; power, square-root and symmetric-log scales | `linear`, `log`, `date` and `category` scale types |
| iOS 16.0 to 16.x | Pie and donut charts (`SectorMark`) | The six rectangular-coordinate marks only |
| iOS 16.0 to 16.x | Selection modifiers | `chartOverlay` with `ChartProxy.value(atX:as:)`, `value(atY:as:)` or `value(at:as:)` and your own gesture |
| iOS 16.0 to 16.x | Scrolling modifiers and visible domain | A chart with a fixed width wider than the screen, inside a horizontal `ScrollView` (`interaction.md`) |
| iOS 16.0 to 16.x | Annotation overflow resolution, `zIndex`, `compositingLayer`, axis styles | `annotation(position:alignment:spacing:content:)` without overflow control |
| iOS 17.x and earlier | Vectorized plots and function plots | One mark per element, with `ForEach` or `Chart(data)`; gated fallback in `vectorized-plots.md`. A function has to be sampled into data points |
| iOS 18.x and earlier | All 3D charts | 2D charts only |

## Availability patterns

Each of these compiles with an iOS 16.0 / macOS 13.0 deployment target. List every platform the app ships on in the check; a platform left out falls under `*` and is treated as available.

### Gate a mark inside the chart

`if #available` works directly in a chart's content. Give the older OS a different mark.

<!-- snippet: Availability/GatedMark.swift#gate-mark -->
```swift
Chart(shares) { share in
    if #available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *) {
        SectorMark(angle: .value("Value", share.value))
            .foregroundStyle(by: .value("Name", share.name))
    } else {
        BarMark(x: .value("Value", share.value))
            .foregroundStyle(by: .value("Name", share.name))
    }
}
```

### Gate a chart modifier

Wrap the newer modifier in a `View` extension so the chart itself is written once. The example below does nothing on the older OS, which suits an enhancement people can live without. When the feature is required, the `else` branch needs a real fallback: for selection, the overlay gesture in `interaction.md`.

<!-- snippet: Availability/GatedChartModifier.swift#gate-chart-modifier -->
```swift
extension View {
    @ViewBuilder
    func chartXSelectionIfAvailable(_ value: Binding<Date?>) -> some View {
        if #available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *) {
            chartXSelection(value: value)
        } else {
            self
        }
    }
}
```

### Gate a mark modifier

The same idea for a modifier on a mark, using `@ChartContentBuilder`.

<!-- snippet: Availability/GatedMarkModifier.swift#gate-mark-modifier -->
```swift
extension ChartContent {
    @ChartContentBuilder
    func zIndexIfAvailable(_ value: Double) -> some ChartContent {
        if #available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *) {
            zIndex(value)
        } else {
            self
        }
    }
}
```

### Gate a whole chart

When the chart only makes sense on the newer OS, mark the type and choose at the call site. State of a newer type (a `Chart3DPose`) lives in the marked type too: held by the view that makes the check, it does not compile.

<!-- snippet: Availability/GatedWholeChart.swift#gate-whole-chart -->
```swift
@available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *)
struct DonutChart: View {
    let shares: [Share]

    var body: some View {
        Chart(shares) { share in
            SectorMark(angle: .value("Value", share.value), innerRadius: .ratio(0.6))
                .foregroundStyle(by: .value("Name", share.name))
        }
    }
}

struct ShareChart: View {
    let shares: [Share]

    var body: some View {
        if #available(iOS 17.0, macOS 14.0, tvOS 17.0, watchOS 10.0, *) {
            DonutChart(shares: shares)
        } else {
            ShareBarChart(shares: shares)
        }
    }
}
```
