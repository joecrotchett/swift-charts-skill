# APIs added in iOS 16

Generated from the Xcode SDK by `tools/build_availability_tables.py`. Do not edit by hand.

Platform versions that ship together, unless a row's *Platforms* cell says otherwise:

| iOS / iPadOS | macOS | tvOS | watchOS | visionOS |
|---|---|---|---|---|
| 16.0 | 13.0 | 16.0 | 9.0 | 1.0 |
| 16.4 | 13.3 | 16.4 | 9.4 | 1.0 |

## Types

| API | Kind | iOS | Members | Platforms | Topic |
|---|---|---|---|---|---|
| `AxisContent` | protocol | 16.0 | see modifier tables |  | axes |
| `AnyAxisContent` | struct | 16.0 | `init(_:)` |  | axes |
| `AxisContentBuilder` | struct | 16.0 |  |  | axes |
| `AxisGridLine` | struct | 16.0 | `init(centered:stroke:)` |  | axes |
| `AxisMark` | protocol | 16.0 | see modifier tables |  | axes |
| `AnyAxisMark` | struct | 16.0 | `init(_:)` |  | axes |
| `AxisMarkBuilder` | struct | 16.0 |  |  | axes |
| `AxisMarks` | struct | 16.0 | `init(preset:position:values:content:)`, `init(format:preset:position:values:stroke:)`, `init(preset:position:values:stroke:)` |  | axes |
| `AxisMarkPreset` | struct | 16.0 | `automatic`, `extended`, `aligned`, `inset` |  | axes |
| `AxisMarkPosition` | struct | 16.0 | `automatic`, `leading`, `trailing`, `top`, `bottom` |  | axes |
| `AxisMarkValues` | struct | 16.0 | `automatic`, `automatic(desiredCount:roundLowerBound:roundUpperBound:)`, `automatic(minimumStride:desiredCount:roundLowerBound:roundUpperBound:)`, `stride(by:count:roundLowerBound:roundUpperBound:calendar:)`, `stride(by:roundLowerBound:roundUpperBound:)` |  | axes |
| `AxisTick` | struct | 16.0 | `init(centered:length:stroke:)` |  | axes |
| `AxisTick.Length` | struct | 16.0 | `automatic`, `label`, `label(extendPastBy:)`, `longestLabel`, `longestLabel(extendPastBy:)` |  | axes |
| `AxisValue` | struct | 16.0 | `as(_:)`, `index`, `count` |  | axes |
| `AxisValueLabel` | struct | 16.0 | `init(centered:anchor:multiLabelAlignment:collisionResolution:offsetsMarks:orientation:horizontalSpacing:verticalSpacing:)`, `init(centered:anchor:multiLabelAlignment:collisionResolution:offsetsMarks:orientation:horizontalSpacing:verticalSpacing:content:)`, `init(_:centered:anchor:multiLabelAlignment:collisionResolution:offsetsMarks:orientation:horizontalSpacing:verticalSpacing:)`, `init(format:centered:anchor:multiLabelAlignment:collisionResolution:offsetsMarks:orientation:horizontalSpacing:verticalSpacing:)` |  | axes |
| `AxisValueLabelOrientation` | struct | 16.0 | `automatic`, `horizontal`, `vertical`, `verticalReversed` |  | axes |
| `AxisValueLabelCollisionResolution` | struct | 16.0 | `automatic`, `greedy`, `greedy(priority:minimumSpacing:)`, `truncate`, `disabled` |  | axes |
| `ChartBinRange` | struct | 16.0 | `lowerBound`, `upperBound`, `contains(_:)`, `relative(to:)` |  | data |
| `DateBins` | struct | 16.0 | `thresholds`, `init(thresholds:)`, `init(timeInterval:range:)`, `init(unit:by:range:calendar:)`, `init(range:desiredCount:calendar:)`, `init(data:desiredCount:calendar:)` |  | data |
| `NumberBins` | struct | 16.0 | `thresholds`, `init(thresholds:)`, `init(size:range:)`, `init(range:count:)`, `init(range:desiredCount:minimumStride:)`, `init(data:desiredCount:minimumStride:)` |  | data |
| `Chart` | struct | 16.0 | `init(content:)`, `init(_:content:)`, `init(_:id:content:)` |  | composition |
| `ChartContent` | protocol | 16.0 | see modifier tables |  | composition |
| `AnnotationPosition` | struct | 16.0 | `automatic`, `overlay`, `top`, `bottom`, `leading`, `trailing`, `topLeading`, `topTrailing`, `bottomLeading`, `bottomTrailing` |  | annotations |
| `AnnotationContext` | struct | 16.0 | `targetSize` |  | annotations |
| `ChartContentBuilder` | struct | 16.0 |  |  | composition |
| `AnyChartContent` | struct | 16.0 | `init(_:)` |  | composition |
| `ChartPlotContent` | struct | 16.0 |  |  | plot-area |
| `ChartProxy` | struct | 16.0 | `plotAreaFrame`, `plotAreaSize`, `position(forX:)`, `position(forY:)`, `position(for:)`, `positionRange(forX:)`, `positionRange(forY:)`, `positionRange(for:)`, `value(atX:as:)`, `value(atY:as:)`, `value(at:as:)` |  | plot-area |
| `Plot` | struct | 16.0 | `init(content:)` |  | composition |
| `ChartSymbolShape` | protocol | 16.0 | `strokeBorder(lineWidth:)`, `strokeBorder(style:)`, `circle`, `square`, `triangle`, `diamond`, `pentagon`, `plus`, `cross`, `asterisk` |  | mark-configuration |
| `AnyChartSymbolShape` | struct | 16.0 | `init(_:)` |  | mark-configuration |
| `BasicChartSymbolShape` | struct | 16.0 | `strokeBorder(lineWidth:)` |  | mark-configuration |
| `AreaMark` | struct | 16.0 | `init(x:y:stacking:)`, `init(xStart:xEnd:y:)`, `init(x:yStart:yEnd:)`, `init(x:y:series:stacking:)`, `init(xStart:xEnd:y:series:)`, `init(x:yStart:yEnd:series:)` |  | marks |
| `BarMark` | struct | 16.0 | `init(x:y:width:height:stacking:)`, `init(x:yStart:yEnd:width:stacking:)`, `init(xStart:xEnd:y:height:stacking:)`, `init(xStart:xEnd:y:height:)`, `init(xStart:xEnd:yStart:yEnd:)`, `init(x:yStart:yEnd:width:)` |  | marks |
| `LineMark` | struct | 16.0 | `init(x:y:)`, `init(x:y:series:)` |  | marks |
| `MarkStackingMethod` | struct | 16.0 | `standard`, `normalized`, `center`, `unstacked` |  | mark-configuration |
| `MarkDimension` | struct | 16.0 | `automatic`, `ratio(_:)`, `inset(_:)`, `fixed(_:)` |  | mark-configuration |
| `InterpolationMethod` | struct | 16.0 | `linear`, `cardinal`, `cardinal(tension:)`, `catmullRom`, `catmullRom(alpha:)`, `monotone`, `stepStart`, `stepCenter`, `stepEnd` |  | mark-configuration |
| `PointMark` | struct | 16.0 | `init(x:y:)` |  | marks |
| `RectangleMark` | struct | 16.0 | `init(x:y:width:height:)`, `init(x:yStart:yEnd:width:)`, `init(xStart:xEnd:y:height:)`, `init(xStart:xEnd:yStart:yEnd:)` |  | marks |
| `RuleMark` | struct | 16.0 | `init(xStart:xEnd:y:)`, `init(x:yStart:yEnd:)` |  | marks |
| `Plottable` | protocol | 16.0 |  |  | data |
| `PrimitivePlottableProtocol` | protocol | 16.0 |  |  | data |
| `PlottableValue` | struct | 16.0 | `value(_:_:)`, `value(_:_:unit:calendar:)` |  | data |
| `ScaleType` | struct | 16.0 | `linear`, `log`, `date`, `category` |  | scales |
| `ScaleDomain` | protocol | 16.0 | `automatic`, `automatic(includesZero:reversed:)`, `automatic(includesZero:reversed:dataType:modifyInferredDomain:)` |  | scales |
| `ScaleRange` | protocol | 16.0 |  |  | scales |
| `PositionScaleRange` | protocol | 16.0 | `plotDimension`, `plotDimension(padding:)`, `plotDimension(startPadding:endPadding:)` |  | scales |
| `AutomaticScaleDomain` | struct | 16.0 |  |  | scales |
| `PlotDimensionScaleRange` | struct | 16.0 |  |  | scales |

## Chart modifiers (applied to the `Chart` view)

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `chartXAxis` | 16.0 | `(_:)`, `(content:)` |  | axes |
| `chartYAxis` | 16.0 | `(_:)`, `(content:)` |  | axes |
| `chartXAxisLabel` | 16.0 | `(position:alignment:spacing:content:)`, `(_:position:alignment:spacing:)` |  | axes |
| `chartYAxisLabel` | 16.0 | `(position:alignment:spacing:content:)`, `(_:position:alignment:spacing:)` |  | axes |
| `chartPlotStyle` | 16.0 | `(content:)` |  | plot-area |
| `chartOverlay` | 16.0 | `(alignment:content:)` |  | plot-area |
| `chartBackground` | 16.0 | `(alignment:content:)` |  | plot-area |
| `chartLegend` | 16.0 | `(position:alignment:spacing:content:)`, `(position:alignment:spacing:)`, `(_:)` |  | legend |
| `chartXScale` | 16.0 | `(domain:range:type:)`, `(domain:type:)`, `(range:type:)`, `(type:)` |  | scales |
| `chartYScale` | 16.0 | `(domain:range:type:)`, `(domain:type:)`, `(range:type:)`, `(type:)` |  | scales |
| `chartForegroundStyleScale` | 16.0 | `(domain:range:type:)`, `(domain:type:)`, `(range:type:)`, `(type:)`, `(_:)`, `(domain:mapping:)`, `(mapping:)` |  | scales |
| `chartSymbolScale` | 16.0 | `(domain:range:)`, `(domain:)`, `(range:)`, `(_:)`, `(domain:mapping:)`, `(mapping:)` |  | scales |
| `chartLineStyleScale` | 16.0 | `(domain:range:)`, `(domain:)`, `(range:)`, `(_:)`, `(domain:mapping:)`, `(mapping:)` |  | scales |
| `chartSymbolSizeScale` | 16.0 | `(domain:range:type:)`, `(domain:type:)`, `(range:type:)`, `(type:)`, `(_:)`, `(domain:mapping:)`, `(mapping:)` |  | scales |

## Mark modifiers (applied to marks and other `ChartContent`)

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `accessibilityLabel` | 16.0 | `(_:)` |  | accessibility |
| `accessibilityValue` | 16.0 | `(_:)` |  | accessibility |
| `accessibilityIdentifier` | 16.0 | `(_:)` |  | accessibility |
| `accessibilityHidden` | 16.0 | `(_:)` |  | accessibility |
| `annotation` | 16.0 | `(position:alignment:spacing:content:)` |  | annotations |
| `mask` | 16.0 | `(content:)` |  | mark-modifiers |
| `foregroundStyle` | 16.0 | `(_:)`, `(by:)` |  | mark-modifiers |
| `alignsMarkStylesWithPlotArea` | 16.0 | `(_:)` |  | mark-modifiers |
| `position` | 16.0 | `(by:axis:span:)` |  | mark-modifiers |
| `opacity` | 16.0 | `(_:)` |  | mark-modifiers |
| `clipShape` | 16.0 | `(_:style:)` |  | mark-modifiers |
| `lineStyle` | 16.0 | `(_:)`, `(by:)` |  | mark-modifiers |
| `symbol` | 16.0 | `(_:)`, `(by:)`, `(symbol:)` |  | mark-modifiers |
| `symbolSize` | 16.0 | `(_:)`, `(by:)` |  | mark-modifiers |
| `offset` | 16.0 | `(x:y:)`, `(_:)`, `(x:yStart:yEnd:)`, `(xStart:xEnd:y:)`, `(xStart:xEnd:yStart:yEnd:)` |  | mark-modifiers |
| `cornerRadius` | 16.0 | `(_:style:)` |  | mark-modifiers |
| `interpolationMethod` | 16.0 | `(_:)` |  | mark-modifiers |
| `blur` | 16.4 | `(radius:)` |  | mark-modifiers |
| `shadow` | 16.4 | `(color:radius:x:y:)` |  | mark-modifiers |

## Axis mark modifiers

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `foregroundStyle` | 16.0 | `(_:)` |  | axes |
| `font` | 16.0 | `(_:)` |  | axes |
| `offset` | 16.0 | `(x:y:)`, `(_:)` |  | axes |

## Members added to earlier types

| API | iOS | Platforms | Topic |
|---|---|---|---|
| `ChartProxy.foregroundStyle(for:)` | 16.4 |  | plot-area |
| `ChartProxy.lineStyle(for:)` | 16.4 |  | plot-area |
| `ChartProxy.symbol(for:)` | 16.4 |  | plot-area |
| `ChartProxy.symbolSize(for:)` | 16.4 |  | plot-area |
| `ChartProxy.xDomain(dataType:)` | 16.4 |  | plot-area |
| `ChartProxy.yDomain(dataType:)` | 16.4 |  | plot-area |
| `ChartProxy.foregroundStyleDomain(dataType:)` | 16.4 |  | plot-area |
| `ChartProxy.symbolDomain(dataType:)` | 16.4 |  | plot-area |
| `ChartProxy.symbolSizeDomain(dataType:)` | 16.4 |  | plot-area |
| `ChartProxy.lineStyleDomain(dataType:)` | 16.4 |  | plot-area |
| `ScaleType.power(exponent:)` | 16.4 |  | scales |
| `ScaleType.squareRoot` | 16.4 |  | scales |
| `ScaleType.symmetricLog(slopeAtZero:)` | 16.4 |  | scales |
| `ScaleType.symmetricLog` | 16.4 |  | scales |

## Conformances

| Protocol | iOS | Conforming types | Platforms |
|---|---|---|---|
| `ChartContent` | 16.0 | `ForEach` |  |
| `ChartSymbolShape` | 16.0 | `Circle` |  |
| `Plottable` | 16.0 | `Double`, `String`, `Date`, `Float16`, `Float`, `Decimal`, `Int`, `Int8`, `Int16`, `Int32`, `Int64`, `UInt`, `UInt8`, `UInt16`, `UInt32`, `UInt64` |  |
| `ScaleDomain` | 16.0 | `ClosedRange`, `Array`, `ArraySlice` |  |
| `ScaleRange` | 16.0 | `ClosedRange`, `Array`, `ArraySlice`, `Gradient`, `AnyGradient` |  |
| `PositionScaleRange` | 16.0 | `ClosedRange`, `Array`, `ArraySlice` |  |
