# APIs added in iOS 17

Generated from the Xcode SDK by `tools/build_availability_tables.py`. Do not edit by hand.

Platform versions that ship together, unless a row's *Platforms* cell says otherwise:

| iOS / iPadOS | macOS | tvOS | watchOS | visionOS |
|---|---|---|---|---|
| 17.0 | 14.0 | 17.0 | 10.0 | 1.0 |

## Types

| API | Kind | iOS | Members | Platforms | Topic |
|---|---|---|---|---|---|
| `ChartAxisContent` | struct | 17.0 |  |  | axes |
| `ChartScrollTargetBehaviorContext` | struct | 17.0 | `chartProxy` |  | interaction |
| `ChartScrollTargetBehavior` | protocol | 17.0 | `valueAligned(unit:majorAlignment:limitBehavior:)`, `valueAligned(matching:majorAlignment:limitBehavior:)`, `valueAligned(xUnit:yUnit:xMajorAlignment:yMajorAlignment:limitBehavior:)`, `valueAligned(xMatching:yMatching:xMajorAlignment:yMajorAlignment:limitBehavior:)`, `valueAligned(xMatching:yUnit:xMajorAlignment:yMajorAlignment:limitBehavior:)`, `valueAligned(xUnit:yMatching:xMajorAlignment:yMajorAlignment:limitBehavior:)` |  | interaction |
| `ValueAlignedLimitBehavior` | struct | 17.0 | `automatic`, `always`, `never` |  | interaction |
| `MajorValueAlignment` | struct | 17.0 | `unit(_:)`, `matching(_:)`, `page` |  | interaction |
| `ValueAlignedChartScrollTargetBehavior` | struct | 17.0 | `init(unit:majorAlignment:limitBehavior:)`, `init(matching:majorAlignment:limitBehavior:)`, `init(xUnit:yUnit:xMajorAlignment:yMajorAlignment:limitBehavior:)`, `init(xMatching:yMatching:xMajorAlignment:yMajorAlignment:limitBehavior:)`, `init(xMatching:yUnit:xMajorAlignment:yMajorAlignment:limitBehavior:)`, `init(xUnit:yMatching:xMajorAlignment:yMajorAlignment:limitBehavior:)` |  | interaction |
| `AnnotationOverflowResolution` | struct | 17.0 | `init(x:y:)`, `automatic` |  | annotations |
| `AnnotationOverflowResolution.Strategy` | struct | 17.0 | `fit`, `fit(to:)`, `padScale`, `disabled`, `automatic` |  | annotations |
| `AnnotationOverflowResolution.Boundary` | struct | 17.0 | `plot`, `chart`, `automatic` |  | annotations |
| `SectorMark` | struct | 17.0 | `init(angle:innerRadius:outerRadius:angularInset:)` |  | marks |

## Chart modifiers (applied to the `Chart` view)

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `chartXAxisStyle` | 17.0 | `(content:)` |  | axes |
| `chartYAxisStyle` | 17.0 | `(content:)` |  | axes |
| `chartScrollableAxes` | 17.0 | `(_:)` |  | interaction |
| `chartXVisibleDomain` | 17.0 | `(length:)` |  | interaction |
| `chartYVisibleDomain` | 17.0 | `(length:)` |  | interaction |
| `chartScrollTargetBehavior` | 17.0 | `(_:)` |  | interaction |
| `chartScrollPosition` | 17.0 | `(initialX:)`, `(x:)`, `(initialY:)`, `(y:)` |  | interaction |
| `chartXSelection` | 17.0 | `(value:)`, `(range:)` |  | interaction |
| `chartYSelection` | 17.0 | `(value:)`, `(range:)` |  | interaction |
| `chartAngleSelection` | 17.0 | `(value:)` |  | interaction |
| `chartGesture` | 17.0 | `(_:)` |  | interaction |

## Mark modifiers (applied to marks and other `ChartContent`)

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `annotation` | 17.0 | `(position:alignment:spacing:overflowResolution:content:)` |  | annotations |
| `compositingLayer` | 17.0 | `()`, `(style:)` |  | mark-modifiers |
| `zIndex` | 17.0 | `(_:)` |  | mark-modifiers |

## Axis content modifiers

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `compositingLayer` | 17.0 | `()`, `(style:)` |  | axes |

## Members added to earlier types

| API | iOS | Platforms | Topic |
|---|---|---|---|
| `ChartProxy.plotFrame` | 17.0 |  | plot-area |
| `ChartProxy.plotContainerFrame` | 17.0 |  | plot-area |
| `ChartProxy.plotSize` | 17.0 |  | plot-area |
| `ChartProxy.selectXValue(at:)` | 17.0 |  | interaction |
| `ChartProxy.selectYValue(at:)` | 17.0 |  | interaction |
| `ChartProxy.selectXRange(from:to:)` | 17.0 |  | interaction |
| `ChartProxy.selectYRange(from:to:)` | 17.0 |  | interaction |
| `ChartProxy.selectAngleValue(at:)` | 17.0 |  | interaction |
| `ChartProxy.angle(at:)` | 17.0 |  | interaction |
| `ChartProxy.value(atAngle:as:)` | 17.0 |  | interaction |

## Conformances

| Protocol | iOS | Conforming types | Platforms |
|---|---|---|---|
| `ChartScrollTargetBehavior` | 17.0 | `PagingScrollTargetBehavior` |  |
