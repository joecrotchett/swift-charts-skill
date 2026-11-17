# APIs added in iOS 26

Generated from the Xcode SDK by `tools/build_availability_tables.py`. Do not edit by hand.

Platform versions that ship together, unless a row's *Platforms* cell says otherwise:

| iOS / iPadOS | macOS | tvOS | watchOS | visionOS |
|---|---|---|---|---|
| 26.0 | 26.0 | 26.0 | 26.0 | 26.0 |

## Types

| API | Kind | iOS | Members | Platforms | Topic |
|---|---|---|---|---|---|
| `SurfacePlot` | struct | 26.0 | `init(x:y:z:function:)` | Not on tvOS, watchOS | chart3d |
| `Chart3DCameraProjection` | struct | 26.0 | `orthographic`, `perspective`, `automatic` | Not on tvOS, watchOS, visionOS | chart3d |
| `Chart3DRenderingStyle` | struct | n/a | `automatic`, `flat`, `volumetric` | visionOS only | chart3d |
| `Chart3DPose` | struct | 26.0 | `init(azimuth:inclination:)`, `default`, `front`, `back`, `top`, `bottom`, `left`, `right`, `azimuth`, `inclination` | Not on tvOS, watchOS | chart3d |
| `Chart3D` | struct | 26.0 | `init(content:)`, `init(_:content:)`, `init(_:id:content:)` | Not on tvOS, watchOS | chart3d |
| `Chart3DContent` | protocol | 26.0 | see modifier tables | Not on tvOS, watchOS | chart3d |
| `Chart3DContentBuilder` | struct | 26.0 |  | Not on tvOS, watchOS | chart3d |
| `Chart3DSymbolShape` | protocol | 26.0 | `sphere`, `cylinder`, `cone`, `cube` | Not on tvOS, watchOS | chart3d |
| `BasicChart3DSymbolShape` | struct | 26.0 |  | Not on tvOS, watchOS | chart3d |
| `BasicChart3DSurfaceStyle` | struct | 26.0 |  | Not on tvOS, watchOS | chart3d |
| `Chart3DSurfaceStyle` | protocol | 26.0 | `heightBased`, `heightBased(_:yRange:)`, `heightBased(yRange:)`, `normalBased` | Not on tvOS, watchOS | chart3d |

## Chart modifiers (applied to the `Chart` view)

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `chart3DPose` | 26.0 | `(_:)` | Not on tvOS, watchOS | chart3d |
| `chart3DCameraProjection` | 26.0 | `(_:)` | Not on tvOS, watchOS, visionOS | chart3d |
| `chart3DRenderingStyle` | n/a | `(_:)` | visionOS only | chart3d |
| `chartZAxis` | 26.0 | `(_:)`, `(content:)` | Not on tvOS, watchOS | chart3d |
| `chartZAxisLabel` | 26.0 | `(_:position:alignment:spacing:)` | Not on tvOS, watchOS | chart3d |
| `chartZScale` | 26.0 | `(domain:range:type:)`, `(domain:type:)`, `(range:type:)` | Not on tvOS, watchOS | chart3d |
| `chartZSelection` | 26.0 | `(value:)`, `(range:)` | Not on tvOS, watchOS | chart3d |

## 3D content modifiers

| API | iOS | Forms | Platforms | Topic |
|---|---|---|---|---|
| `foregroundStyle` | 26.0 | `(_:)`, `(by:)` | Not on tvOS, watchOS | chart3d |
| `metalness` | 26.0 | `(_:)` | Not on tvOS, watchOS | chart3d |
| `roughness` | 26.0 | `(_:)` | Not on tvOS, watchOS | chart3d |
| `symbol` | 26.0 | `(_:)` | Not on tvOS, watchOS | chart3d |
| `symbolRotation` | 26.0 | `(_:)` | Not on tvOS, watchOS | chart3d |
| `symbolSize` | 26.0 | `(_:)` | Not on tvOS, watchOS | chart3d |

## Members added to earlier types

| API | iOS | Platforms | Topic |
|---|---|---|---|
| `PointMark.init(x:y:z:)` | 26.0 | Not on tvOS, watchOS | chart3d |
| `RectangleMark.init(x:y:z:)` | 26.0 | Not on tvOS, watchOS | chart3d |
| `RuleMark.init(x:y:z:)` | 26.0 | Not on tvOS, watchOS | chart3d |

## Conformances

| Protocol | iOS | Conforming types | Platforms |
|---|---|---|---|
| `Chart3DContent` | 26.0 | `ForEach`, `PointMark`, `RectangleMark`, `RuleMark` | Not on tvOS, watchOS |
