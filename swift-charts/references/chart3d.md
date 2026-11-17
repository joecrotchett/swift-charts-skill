# 3D charts

Three-dimensional charts, iOS 26.0+. They exist on iOS, macOS and visionOS only: there is no 3D chart API on tvOS or watchOS, so code that uses it must be excluded from those builds with `#if !os(tvOS) && !os(watchOS)`, not only gated at run time. Every snippet in this file is gated with `@available`; the `#if` is not repeated in them.

## Chart3D

### `Chart3D` - iOS 26.0+

**Use for:** the view that displays a 3D chart. It is a separate view from `Chart`, with its own content type, and it is interactive.

**Prefer a 2D chart** unless the data's shape in three dimensions is the point. A 3D chart is read by rotating it: seen from a side it shows two properties at a time, like the 2D chart it replaces, and turned it shows how clusters separate. Choose it when the shape matters more than the exact values, when the data is itself three-dimensional (positions in space), and only if requiring interaction suits the app.

Converting a 2D scatter plot is three changes: `Chart` becomes `Chart3D`, each `PointMark` gains a `z:`, and the z axis gets a label.

Its initializers mirror `Chart`: `Chart3D(data) { }`, `Chart3D(data, id:) { }` and `Chart3D { }`, chosen the same way (`composition.md`).

Only four kinds of content can go in a 3D chart:

- `SurfacePlot`, which exists only in 3D.
- `PointMark(x:y:z:)`, `RuleMark(x:y:z:)` and `RectangleMark(x:y:z:)`, the 3D initializers of three existing marks.

There are no 3D bars, lines, areas or sectors.

All three axes of a 3D mark are numeric. What separates the three marks is how many axes are given a range instead of a single value:

| Mark | Single values | Ranges | Draws |
|---|---|---|---|
| `PointMark(x:y:z:)` | 3 | 0 | A symbol at a point |
| `RuleMark(x:y:z:)` | 2 | 1 | A line along the axis given the range |
| `RectangleMark(x:y:z:)` | 1 | 2 | A flat rectangle across the two axes given ranges |

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

            // A rectangle: ranges on two axes, a single value on the third.
            RectangleMark(
                x: .value("x", -0.5 ..< 0.5),
                y: .value("y", 0),
                z: .value("z", -0.5 ..< 0.5)
            )
            .foregroundStyle(.blue.opacity(0.3))
        }
    }
}
```

3D content has its own, smaller set of modifiers:

| Modifier | Does |
|---|---|
| `foregroundStyle(_:)`, `foregroundStyle(by:)` | Fixed or data-driven colour, as in 2D |
| `symbol(_:)` | The point shape: `.sphere`, `.cube`, `.cylinder`, `.cone` |
| `symbolSize(_:)`, `symbolRotation(_:)` | Size, and orientation as a `Rotation3D` |
| `metalness(_:)` | 0 for a non-metallic surface, 1 for metallic. Real materials are one or the other |
| `roughness(_:)` | 0 for smooth, 1 for completely rough |

The 2D mark modifiers in `mark-modifiers.md` do not apply to 3D content.

## Surface plots

### `SurfacePlot` - iOS 26.0+

**Use for:** drawing a function of two variables, y = f(x, z), as a surface.

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

- `.heightBased` colours by y value, which makes the shape easier to read. Use it by default.
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

`Chart3DPose` has presets (`.default`, `.front`, `.back`, `.top`, `.bottom`, `.left`, `.right`) and `Chart3DPose(azimuth:inclination:)` for a specific angle: azimuth turns the chart left and right, inclination tilts it up and down.

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

The x and y scale and axis modifiers work on a `Chart3D` as they do in 2D, and each has a z counterpart: `chartZScale`, `chartZAxis` (with `AxisMarks`, as in `axes.md`), `chartZAxisLabel`, and `chartZSelection(value:)` and `chartZSelection(range:)`. The z forms affect 3D charts only.

A 3D scale's `domain:` is in data values, as usual. Give every axis a `range:` of `-0.5 ... 0.5`, as the snippets here do, so the chart is a cube centred in its view.

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
        .chartZAxis {
            AxisMarks(values: [2, 4, 6, 8]) {
                AxisTick()
                AxisGridLine()
                AxisValueLabel()
            }
        }
    }
}
```

## Pitfalls

- A `Chart3D` cannot be captured with `ImageRenderer`; it draws a placeholder. Use a 2D chart for exported images.
- Expecting 2D marks or modifiers to work in `Chart3D`.
