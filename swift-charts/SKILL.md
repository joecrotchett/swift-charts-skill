---
name: swift-charts
description: Designs, builds, and reviews charts made with Apple's Swift Charts framework, including custom charts from a screenshot or mockup, chart accessibility (VoiceOver, Audio Graphs), and which APIs exist on which OS version. Use when reading, writing, designing, or reviewing code that imports Charts, or when deciding whether Swift Charts can produce a given chart design.
license: MIT
metadata:
  author: Joe Crotchett
  version: "0.1"
---

Design and engineer charts with Swift Charts so the result needs little correction in code review: the right marks, the intended API for each job, correct availability for the app's deployment target, and accessibility built in.

## Always

- **Establish the deployment target first.** Read it from the project (`IPHONEOS_DEPLOYMENT_TARGET`, `Package.swift` platforms, `Project.swift`) before choosing any API. Ask only if it cannot be read. For a design with no project, state the target you assumed and mark each part that needs a newer OS.
- **Check `references/availability/README.md` before using an API.** If the API is newer than the deployment target, either gate it with `if #available` and supply a fallback, or choose the older equivalent. Say which you did.
- **Keep what the framework gives for free.** Dark Mode, Dynamic Type, localization, and a starting point for VoiceOver come with the standard marks, scales and axes (`references/composition.md`). Do not rebuild or defeat them.
- **Use the framework's intended API before a workaround.** Reach for an escape hatch only after `references/limits-and-escapes.md` confirms Swift Charts cannot do it natively.
- **Never guess an API.** If the references do not cover something, say so and check Apple's documentation rather than inventing a signature or modifier.
- **Accessibility is part of done.** Every chart delivered has been through `references/accessibility.md` and has a chart descriptor written from `references/audio-graph.md`.
- **Self-review before presenting code** using `references/review-checklist.md`.

## Workflows

Load only the references the task needs.

### Design a chart

1. Decide whether a chart is right at all, and name the question it answers (`references/design.md`). Note the shape of the data (`references/data.md`).
2. Work through `references/design.md` in its order: marks, axes, descriptions, interaction, colour, then how the chart sits in the app.
3. Write the chart's headline or summary in words. If the message cannot be stated, the design is not ready.
4. Confirm the form is buildable for the deployment target (`references/availability/README.md`, `references/limits-and-escapes.md`).
5. Describe the design, naming the API for each decision, and the accessibility experience: what an element covers, what it says, and what the Audio Graph says (`references/accessibility.md`, `references/audio-graph.md`).

### Build a chart from a screenshot or mockup

1. **Inventory what is visible.** List every element: mark shapes, how data maps to position, colour, size and symbol, axes (ticks, grid lines, labels), legend, annotations, backgrounds, and any implied interaction or animation. Match the overall form against `references/gallery/README.md`, and compare the chart's chrome with its list of defaults: every difference is something to customise.
2. **Map each element to API** using the topic references.
3. **Classify each element** as one of:
   - *Native*: Swift Charts API available at the deployment target.
   - *Native, newer OS*: Swift Charts API whose minimum OS is above the deployment target; needs `if #available` and a fallback.
   - *Escape hatch*: needs SwiftUI outside Swift Charts (`references/limits-and-escapes.md`).
   - *Not feasible*: say so and propose the nearest achievable design.
4. **Report the feasibility table before writing code** when anything is not *Native*.
5. Implement, then apply `references/accessibility.md`, `references/accessibility-beyond-voiceover.md`, `references/audio-graph.md` and `references/review-checklist.md`.

### Implement or modify a chart

1. For a known form, start from its recipe (`references/gallery/README.md`). Shape the data (`references/data.md`), then choose and compose marks (`references/marks.md`, `references/composition.md`), configure them (`references/mark-configuration.md`) and style them (`references/mark-modifiers.md`).
2. Configure scales (`references/scales.md`), axes (`references/axes.md`), legend (`references/legend.md`) and plot area (`references/plot-area.md`) where the design differs from the defaults, and check every axis's labels at phone width and the largest text size (`references/axes.md`, *`AxisValueLabel`*).
3. Add annotations (`references/annotations.md`) and interaction (`references/interaction.md`) if required, a visible title or headline saying what the chart shows (`references/design.md`, *Descriptions*), and a placeholder for when there is no data.
4. For large data sets, draw fewer points: bin or aggregate; a plot (iOS 18+) suits a large uniform collection, but measured no faster than marks (`references/vectorized-plots.md`).
5. Apply `references/accessibility.md`, `references/accessibility-beyond-voiceover.md`, `references/audio-graph.md` and `references/review-checklist.md`.

### Diagnose a chart that looks or behaves wrong

1. Identify the form from `references/gallery/README.md` and read that recipe's pitfalls first.
2. Compare what is on screen with the list of defaults in `references/gallery/README.md`: an unintended default is the usual cause.
3. Then by symptom: a slow chart, `references/vectorized-plots.md`; a wrong selection or gesture, `references/interaction.md` and `references/plot-area.md`; marks spilling outside the plot, `references/plot-area.md`; a missing legend or colours that change when the data is filtered, `references/legend.md` and `references/scales.md`; a chart VoiceOver reads oddly, `references/accessibility-defaults.md`.

A fix to an existing chart is held to the same *Always* rules as a new one only for the part that is changed; mention accessibility gaps you notice without rewriting the chart.

### Review chart code

Walk `references/review-checklist.md`. Most items state their rule in full; load the reference an item names only when the fix is not evident from the item. Report only genuine problems, organised by file, each with the rule and a before/after fix. Flag any API used without the availability its deployment target requires.

### Make a chart accessible

Follow `references/accessibility.md` in full for the elements and everything beyond VoiceOver, then write the chart descriptor from `references/audio-graph.md`. Use `references/accessibility-defaults.md` to know what the chart says before you change anything.

## References

- `references/availability/README.md` - what each OS release added, what an older deployment target rules out, and `if #available` patterns, with a complete API table per release beside it; read when choosing any API or supporting an older OS.
- `references/data.md` - shaping data for charts, plottable values and ranges, dates and units, making your own types and enums plottable, binning numbers and dates for histograms and totals; read before composing marks.
- `references/marks.md` - each mark type (area, line, point, rectangle, rule, bar, sector) and how a mark's position is given as a value, an interval or nothing; read when choosing a mark.
- `references/mark-configuration.md` - stacking methods, bar and rectangle dimensions, line interpolation, built-in, outlined and custom symbol shapes; read when setting how marks stack, how wide they are, how a line travels between points, or what a point looks like.
- `references/mark-modifiers.md` - the modifiers every mark shares: styling, positioning, symbols, encoding data as colour, position, symbol, size or line style, layering, clipping; read when styling marks or matching a visual design.
- `references/composition.md` - the model (marks, mark properties, scales), what the framework handles for you (and the locale and right-to-left rules), the `Chart` initializers, series, layering, stacking and grouping, `Plot`, animation, reusable `ChartContent` types; read first for any chart, and again when it has more than one mark or series.
- `references/scales.md` - position scales: exact, zero-free, reversed and adjusted domains, plot padding, log and other scale types; colour and symbol scales; read when an axis range or direction, or a colour mapping, differs from the default.
- `references/axes.md` - hiding or rebuilding an axis: which values are marked, on which edge, label format, placement presets, grid lines, ticks, custom and conditional labels, crowded labels, axis titles; read when an axis is not the default.
- `references/legend.md` - hiding and placing the legend, and building your own in the chart's colours; read when the legend is not the default.
- `references/plot-area.md` - styling, sizing and clipping the plot area, placing views against it, and `ChartProxy`: converting between positions and data values, plot frames, reading the chart's scales; read for backgrounds, centre labels, overlays, marks that spill out of the plot, and custom geometry.
- `references/annotations.md` - labelling marks: value labels, positions and alignment, and what happens at the chart's edge; read when the chart shows labels, callouts or badges.
- `references/interaction.md` - selection on one axis, both axes or sectors, linking charts through a shared selection (the basis of a dashboard), scrolling and where it comes to rest, and custom gestures on iOS 16 and 17; read when the chart responds to touch or pointer.
- `references/vectorized-plots.md` - the plot types (iOS 18+): one plot for a whole collection, key-path modifiers and their ordering rule, keeping a large plot fast, function and parametric plots, trend lines; read for large data sets, slow charts or mathematical curves.
- `references/chart3d.md` - `Chart3D`, surface plots, 3D points, pose and projection, and what 3D cannot do; read for any 3D chart.
- `references/accessibility.md` - choosing what an accessibility element covers, labelling marks through `Plot`, grouping marks into one element, hiding what adds nothing, testing; read for every chart before it is done.
- `references/accessibility-beyond-voiceover.md` - keyboard and Switch Control navigation, colour and contrast, Dynamic Type, announcing changes; read for every chart before it is done, and again when colour or text size carries meaning.
- `references/audio-graph.md` - writing a chart descriptor: title, summary, axes, series, units and dates for the Audio Graph, for category, number and single-value charts, and the charts that have no Audio Graph without one; read for every chart before it is done.
- `references/accessibility-defaults.md` - exactly what VoiceOver reads and what the Audio Graph is given for each kind of chart with no accessibility code; read when predicting or explaining what a chart says.
- `references/design.md` - Apple's design guidance mapped to API: whether to chart, marks, axes, descriptions, interaction, colour, charts in an app experience (sizes from a widget or watch face to a full screen, empty states, a family of charts); read when deciding what to build, and when judging whether a chart is well designed.
- `references/limits-and-escapes.md` - a lookup from design element to verdict (native, escape hatch, not feasible), minimum OS and API, including exporting a chart as an image; read to find whether and how something can be done, and before reaching for a workaround.
- `references/review-checklist.md` - the mistakes a reviewer would catch; read before presenting chart code and when reviewing it.
- `references/gallery/README.md` - common chart forms with their visual signature, a rendered image and a recipe, plus what an uncustomised chart looks like; read when identifying a chart from an image or starting a chart of a known form.
