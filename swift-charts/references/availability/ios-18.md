# APIs added in iOS 18

Generated from the Xcode SDK by `tools/build_availability_tables.py`. Do not edit by hand.

Platform versions that ship together, unless a row's *Platforms* cell says otherwise:

| iOS / iPadOS | macOS | tvOS | watchOS | visionOS |
|---|---|---|---|---|
| 18.0 | 15.0 | 18.0 | 11.0 | 2.0 |

## Types

| API | Kind | iOS | Members | Platforms | Topic |
|---|---|---|---|---|---|
| `VectorizedChartContent` | protocol | 18.0 | see modifier tables |  | vectorized-plots |
| `AreaPlot` | struct | 18.0 | `init(_:x:y:stacking:)`, `init(_:xStart:xEnd:y:)`, `init(_:x:yStart:yEnd:)`, `init(_:x:y:series:stacking:)`, `init(_:xStart:xEnd:y:series:)`, `init(_:x:yStart:yEnd:series:)`, `init(x:yStart:yEnd:domain:function:yEnd:)`, `init(x:y:domain:function:)` |  | vectorized-plots |
| `FunctionAreaPlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `VectorizedAreaPlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `BarPlot` | struct | 18.0 | `init(_:x:y:width:height:stacking:)`, `init(_:xStart:xEnd:y:height:)`, `init(_:x:yStart:yEnd:width:)`, `init(_:x:yStart:yEnd:width:stacking:)`, `init(_:xStart:xEnd:y:height:stacking:)`, `init(_:xStart:xEnd:yStart:yEnd:)` |  | vectorized-plots |
| `VectorizedBarPlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `LinePlot` | struct | 18.0 | `init(_:x:y:)`, `init(_:x:y:series:)`, `init(x:y:domain:function:)`, `init(x:y:t:domain:function:y:)` |  | vectorized-plots |
| `FunctionLinePlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `VectorizedLinePlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `MarkDimensions` | struct | 18.0 | `automatic`, `ratio(_:)`, `inset(_:)`, `fixed(_:)` |  | vectorized-plots |
| `PointPlot` | struct | 18.0 | `init(_:x:y:)` |  | vectorized-plots |
| `VectorizedPointPlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `RectanglePlot` | struct | 18.0 | `init(_:x:y:width:height:)`, `init(_:xStart:xEnd:y:height:)`, `init(_:x:yStart:yEnd:width:)`, `init(_:xStart:xEnd:yStart:yEnd:)` |  | vectorized-plots |
| `VectorizedRectanglePlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `RulePlot` | struct | 18.0 | `init(_:xStart:xEnd:y:)`, `init(_:x:yStart:yEnd:)` |  | vectorized-plots |
| `VectorizedRulePlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `SectorPlot` | struct | 18.0 | `init(_:angle:innerRadius:outerRadius:angularInset:)` |  | vectorized-plots |
| `VectorizedSectorPlotContent` | struct | 18.0 |  |  | vectorized-plots |
| `PlottableProjection` | struct | 18.0 | `value(_:_:)`, `value(_:_:_:)`, `value(_:_:unit:calendar:)` |  | vectorized-plots |

## Plot modifiers (applied to vectorized plots)

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `accessibilityLabel` | 18.0 | `(_:)` |  | accessibility |
| `accessibilityValue` | 18.0 | `(_:)` |  | accessibility |
| `accessibilityIdentifier` | 18.0 | `(_:)` |  | accessibility |
| `accessibilityHidden` | 18.0 | `(_:)` |  | accessibility |
| `foregroundStyle` | 18.0 | `(_:)`, `(by:)` |  | vectorized-plots |
| `position` | 18.0 | `(by:axis:span:)` |  | vectorized-plots |
| `opacity` | 18.0 | `(_:)` |  | vectorized-plots |
| `lineStyle` | 18.0 | `(_:)`, `(by:)` |  | vectorized-plots |
| `symbol` | 18.0 | `(by:)` |  | vectorized-plots |
| `symbolSize` | 18.0 | `(_:)`, `(by:)` |  | vectorized-plots |
