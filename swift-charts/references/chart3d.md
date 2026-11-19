# 3D charts

Three-dimensional charts, iOS 26.0+. They exist on iOS, macOS and visionOS only: there is no 3D chart API on tvOS or watchOS, so where a target also builds for tvOS or watchOS, code that uses it must be excluded from those builds with `#if !os(tvOS) && !os(watchOS)`, not only gated at run time; an app that ships only on iOS needs only the availability check. On an older deployment target, keep every `Chart3D` and any state of a 3D type (a `@State` `Chart3DPose`) in a view marked `@available(iOS 26, *)`: such state in the view that holds the `if #available` does not compile. Every snippet in this file is gated with `@available`; the `#if` is not repeated in them.

## Chart3D

### `Chart3D` - iOS 26.0+

**Use for:** the view that displays a 3D chart. It is a separate view from `Chart`, with its own content type, and it is interactive.

**Prefer a 2D chart** unless the data's shape in three dimensions is the point. A 3D chart is read by rotating it: seen from a side it shows two properties at a time, like the 2D chart it replaces, and turned it shows how clusters separate. Choose it when the shape matters more than the exact values, when the data is itself three-dimensional (positions in space), and only if requiring interaction suits the app.

Converting a 2D scatter plot's marks is three changes: `Chart` becomes `Chart3D`, each `PointMark` gains a `z:`, and the z axis gets a label. What does not come across: accessibility (*Accessibility* below), the legend, a colour scale, `symbol(by:)` and labelled `Plot`s.

Its initializers mirror `Chart`: `Chart3D(data) { }`, `Chart3D(data, id:) { }` and `Chart3D { }`, chosen the same way (`composition.md`).

Only four kinds of content can go in a 3D chart:

- `SurfacePlot`, which exists only in 3D.
- `PointMark(x:y:z:)`, `RuleMark(x:y:z:)` and `RectangleMark(x:y:z:)`, the 3D initializers of three existing marks.

There are no 3D bars, lines, areas or sectors. A range is half-open (`0 ..< 10`), and the content builder takes `if` and `switch`. A rule, rectangle or surface has no thickness: seen exactly along it (a rule along z from `.front`, a surface from 90/0) it draws nothing.

All three axes of a 3D mark are numeric. What separates the three marks is how many axes are given a range instead of a single value:

| Mark | Single values | Ranges | Draws |
|---|---|---|---|
| `PointMark(x:y:z:)` | 3 | 0 | A symbol at a point |
| `RuleMark(x:y:z:)` | 2 | 1 | A line along the axis given the range, a hairline with no width setting |
| `RectangleMark(x:y:z:)` | 1 | 2 | A flat rectangle across the two axes given ranges, centred on them. Its width is the range's length passed through the scale as a position (iOS 27): true only where the scale puts 0 at the box's centre (a domain such as −10…10); on 0…10, 0..<10 drew 2.5–7.5 and 0..<5 nothing |

## 3D marks

A 3D scatter plot, with the pose bound so the app can read or set the viewing angle:

<!-- snippet: Gallery/Chart3DRecipes.swift#scatter -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct Scatter3DRecipe: View {
    let beans: [Bean]
    @State private var pose: Chart3DPose = .default

    var body: some View {
        Chart3D(beans) { bean in
            PointMark(
                x: .value("Acidity", bean.acidity),
                y: .value("Body", bean.body),
                z: .value("Sweetness", bean.sweetness)
            )
            .foregroundStyle(by: .value("Origin", bean.origin))
        }
        .chart3DPose($pose)
    }
}
```

Points take a 3D symbol and a size:

<!-- snippet: Gallery/Chart3DRecipes.swift#symbols -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct StyledScatter3D: View {
    let beans: [Bean]

    var body: some View {
        Chart3D(beans) { bean in
            PointMark(
                x: .value("Acidity", bean.acidity),
                y: .value("Body", bean.body),
                z: .value("Sweetness", bean.sweetness)
            )
            .symbol(.cone)
            .symbolSize(0.05)
            .foregroundStyle(by: .value("Origin", bean.origin))
        }
    }
}
```

Rules and rectangles draw guides such as an axis line or a reference plane:

<!-- snippet: Gallery/Chart3DRecipes.swift#guides -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct Guides3D: View {
    var body: some View {
        Chart3D {
            // A rule: a range on one axis, single values on the other two.
            RuleMark(
                x: .value("x", -0.5 ..< 0.5),
                y: .value("y", 0),
                z: .value("z", 0)
            )
            .foregroundStyle(.red)

            // A rectangle: ranges on two axes, a single value on the third. Its size is true only
            // where the scale puts 0 at the box's centre, as these values do.
            RectangleMark(
                x: .value("x", -0.5 ..< 0.5),
                y: .value("y", 0),
                z: .value("z", -0.5 ..< 0.5)
            )
            .foregroundStyle(.blue.opacity(0.6))
        }
    }
}
```

3D content has its own, smaller set of modifiers:

| Modifier | Does |
|---|---|
| `foregroundStyle(_:)`, `foregroundStyle(by:)` | Fixed or data-driven colour. In 3D `foregroundStyle(by:)` draws no legend and ignores `chartForegroundStyleScale` (iOS 27): where a key is needed, colour each point from one table the key also reads |
| `symbol(_:)` | The point shape: `.sphere`, `.cube`, `.cylinder`, `.cone`. Seen face on, a cube and a cylinder both look square: as a second cue, use sphere, cube and cone. There is no `symbol(by:)` |
| `symbolSize(_:)`, `symbolRotation(_:)` | Size as a fraction of the box's edge, and orientation as a `Rotation3D`. At `0.1` a cube is a tenth of the edge but a sphere's diameter a fifth (the value is its radius); the default sizes differ by shape, a cone's barely visible, so set it. Measured on iOS, sizes that look alike: sphere 0.57, cube 1, cone 1.75 times the cube's value |
| `opacity` in a style | Translucent, but the colour washes toward white far faster than the value: at 0.6 a plane still reads, at 0.3 it nearly vanishes yet still hides the grid behind it |
| `metalness(_:)` | 0 for a non-metallic surface, 1 for metallic. Real materials are one or the other. At 1 with roughness 0 a point mirrors a dark scene and loses its colour |
| `roughness(_:)` | 0 for smooth, 1 for completely rough |

The 2D mark modifiers in `mark-modifiers.md` do not apply to 3D content.

## Surface plots

### `SurfacePlot` - iOS 26.0+

**Use for:** drawing a function of two variables, y = f(x, z), as a surface. Its closure is `@Sendable`: capture values, not `self`, and in a module isolated to the main actor by default mark any function it calls `nonisolated`.

`SurfacePlot(x:y:z:function:)` takes a label for each axis and a closure from `(x, z)` to `y`. Set the region to plot with `chartXScale(domain:)`, `chartYScale(domain:)` and `chartZScale(domain:)`.

It is the three-dimensional counterpart of a function `LinePlot` (`vectorized-plots.md`). Drawn in the same chart as points, a surface shows a fitted model against the data; as with a trend line, the app computes the fit:

<!-- snippet: Gallery/Chart3DRecipes.swift#regression -->
```swift
/// Points with a fitted plane through them. The app supplies the fit.
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct ScatterWithSurface3D: View {
    let beans: [Bean]
    /// body = intercept + a x acidity + b x sweetness
    let intercept, acidityWeight, sweetnessWeight: Double

    var body: some View {
        Chart3D {
            ForEach(beans) { bean in
                PointMark(
                    x: .value("Acidity", bean.acidity),
                    y: .value("Body", bean.body),
                    z: .value("Sweetness", bean.sweetness)
                )
            }
            SurfacePlot(x: "Acidity", y: "Body", z: "Sweetness") { [intercept, acidityWeight, sweetnessWeight] acidity, sweetness in
                intercept + acidityWeight * acidity + sweetnessWeight * sweetness
            }
            .foregroundStyle(.gray)
        }
    }
}
```

Style it with `foregroundStyle`:

- `.heightBased` colours by y value, which makes the shape easier to read. Use it by default. The ramp runs over the y scale's domain, not the data: widen the domain and the peaks lose their warmest colour. `heightBased(yRange:)` exists but changed nothing seen on iOS 27.
- `.normalBased` colours by the direction the surface faces.

- A gradient such as `LinearGradient` or `EllipticalGradient`, or a plain colour.

<!-- snippet: Gallery/Chart3DRecipes.swift#surface-gradient -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct GradientSurfacePlot: View {
    var body: some View {
        Chart3D {
            SurfacePlot(x: "x", y: "y", z: "z") { x, z in
                let distance = hypot(x, z)
                return distance == 0 ? 1 : sin(distance) / distance
            }
            .foregroundStyle(EllipticalGradient(colors: [.red, .orange, .yellow, .green, .blue]))
        }
        .chartXScale(domain: -10 ... 10, range: -0.5 ... 0.5)
        .chartZScale(domain: -10 ... 10, range: -0.5 ... 0.5)
        .chartYScale(domain: -0.23 ... 1, range: -0.5 ... 0.5)
    }
}
```

With more than one surface in a chart, give each a different style to tell them apart.

<!-- snippet: Gallery/Chart3DRecipes.swift#surface -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct SurfacePlotRecipe: View {
    var body: some View {
        Chart3D {
            SurfacePlot(x: "x", y: "y", z: "z") { x, z in
                sin(2 * x) * cos(2 * z)
            }
            .foregroundStyle(.heightBased)
        }
        .chartXScale(domain: -2 ... 2)
        .chartYScale(domain: -1 ... 1)
        .chartZScale(domain: -2 ... 2)
    }
}
```

**Pitfalls:** where a surface's function is undefined or infinite, return `Double.nan`.

## Camera and pose

### `chart3DPose` - iOS 26.0+

**Use for:** tracking the angle the chart is viewed from. Bind a `Chart3DPose` state value; it is updated as people rotate the chart.

`Chart3DPose` has presets (`.default`, `.front`, `.back`, `.top`, `.bottom`, `.left`, `.right`) and `Chart3DPose(azimuth:inclination:)` for a specific angle: azimuth turns the chart left and right, inclination tilts it up and down. A bound pose reads back as `pose.azimuth` and `pose.inclination` (`Angle2D`), so sliders can follow a drag. Read on iOS: `.front` 0/0 shows x across and y up; `.left` 90/0 shows z across and y up; `.bottom` 0/−89.9 shows x across and z up; `.default` is −30/15. `.back` (180/0), `.right` (−90/0) and `.top` (0/89.9) show the same pairs mirrored. Inclination stops at ±89.9; azimuth is not normalised.

**Choose the initial pose deliberately.** The same data looks different from each angle: one pose separates the clusters, another shows a linear relationship edge-on. Open on the pose that shows what the chart is for. With data not known in advance, choose one that works for typical data of that kind.

### Projection and rendering style

| Modifier | Platforms | Does |
|---|---|---|
| `chart3DCameraProjection(_:)` | iOS and macOS only | `.orthographic` (the default): points keep their size whatever their depth, so sizes and distances can be compared and a side view reads as a 2D chart. `.perspective`: farther points are smaller and parallel lines converge, which helps the sense of depth |
| `chart3DRenderingStyle(_:)` | visionOS only | `.flat` draws the chart onto a plane, `.volumetric` in depth for both eyes; `.automatic` chooses |

Each needs a platform check as well as the availability gate:

<!-- snippet: Gallery/Chart3DRecipes.swift#projection -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
extension View {
    /// Perspective projection exists on iOS and macOS only.
    func perspectiveProjectionWhereAvailable() -> some View {
        #if os(iOS) || os(macOS)
        chart3DCameraProjection(.perspective)
        #else
        self
        #endif
    }
}
```

## Scales and axes in 3D

The x and y scale and axis modifiers apply to a `Chart3D`, and each has a z counterpart: `chartZScale`, `chartZAxis` (with `AxisMarks`, as in `axes.md`), `chartZAxisLabel`, and `chartZSelection(value:)` and `chartZSelection(range:)`. The z forms affect 3D charts only.

A 3D scale's `domain:` is in data values, as usual. Give every axis a `range:` of `-0.5 ... 0.5` (a `ClosedRange<CGFloat>`), as the snippets here do, so the chart is a cube centred in its view. The axis labels are the framework's and differ from 2D (iOS 27): the domain's end values are not labelled, a fractional value is rounded unless the label is given a format, the labels do not follow Dynamic Type, and an axis title can run into its labels. A figure a headline names must lie inside the domain to be labelled. Without `chartX/Y/ZAxisLabel` the axes are titled X, Y and Z, and `position:` does not move a 3D title. A `Chart3D` does not clip: marks outside a domain are drawn outside the box, so filter the data to the domains. The box scales with the chart's frame, so keep that from growing with text size (a fixed height), or titles are cut off at the sides.

<!-- snippet: Gallery/Chart3DRecipes.swift#configured -->
```swift
@available(iOS 26.0, macOS 26.0, visionOS 26.0, *)
struct ConfiguredScatter3D: View {
    let beans: [Bean]
    /// The angle the chart opens at. It updates as people rotate the chart.
    @State private var pose = Chart3DPose(azimuth: .degrees(20), inclination: .degrees(7))

    var body: some View {
        Chart3D(beans) { bean in
            PointMark(
                x: .value("Acidity", bean.acidity),
                y: .value("Body", bean.body),
                z: .value("Sweetness", bean.sweetness)
            )
            .foregroundStyle(by: .value("Origin", bean.origin))
        }
        .chart3DPose($pose)
        .chartXAxisLabel("Acidity")
        .chartYAxisLabel("Body")
        .chartZAxisLabel("Sweetness")
        // The domain is in data values; the range is the extent of the chart's box.
        .chartXScale(domain: 2 ... 8, range: -0.5 ... 0.5)
        .chartYScale(domain: 1 ... 8, range: -0.5 ... 0.5)
        .chartZScale(domain: 2 ... 8, range: -0.5 ... 0.5)
        // A 3D axis does not label its domain's ends (iOS): mark values inside it.
        .chartZAxis {
            AxisMarks(values: [4, 6]) {
                AxisTick()
                AxisGridLine()
                AxisValueLabel()
            }
        }
    }
}
```

## Pitfalls

- `ImageRenderer` draws a placeholder for a `Chart3D`. Drawing a hosted window with `drawHierarchy` captures it in full (iOS), or export a 2D twin.
- Expecting 2D marks or modifiers to work in `Chart3D`.

## Accessibility

A `Chart3D` gives assistive technologies nothing: no elements and no Audio Graph, one unlabelled element in the tree (read on iOS). Supply both. For points in groups, put `accessibilityRepresentation` on the chart holding an undrawn 2D `Chart` with one labelled `Plot` per group, and the chart descriptor inside the representation (`accessibility.md`); for a surface, make the chart one element (`accessibilityElement(children: .ignore)`, a label and a value) and attach the descriptor to it. Both were read on iOS, and a chart with points and a surface takes one representation with a `Plot` per group and one for the surface. Give the third dimension to the descriptor as an additional axis (`audio-graph.md`), and sample a surface's rows through every figure the summary names. Rotation needs a drag: offer the presets as buttons too (a picker keeps a stale selection after a drag). The `.id` that refreshes elements and descriptor also resets `@State` in the view it is on, so keep the pose as plain angles in the view model and bind the chart to them, or every toggle undoes the reader's rotation.
