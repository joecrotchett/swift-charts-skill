# Swift Charts skill

An agent skill for designing, building and reviewing charts made with Apple's [Swift Charts](https://developer.apple.com/documentation/charts) framework. It helps a coding agent choose the right marks, use the intended API for each job, gate APIs correctly for an app's deployment target, and build accessibility in from the start.

## What it covers

- **Building charts:** marks, mark configuration and modifiers, composition, data and binning, scales, axes, legends, the plot area, annotations, interaction, vectorized plots and 3D charts.
- **A gallery of forms** with recipes and pictures: bar, stacked and grouped bars, line, area, scatter, heat map, calendar, range bar, diverging bar, box plot, candlestick, timeline, population pyramid, difference chart, vector field, histogram and more.
- **Accessibility:** what VoiceOver gets by default, labelled elements, chart descriptors for the Audio Graph, Dynamic Type, Dark Mode, right-to-left and locales.
- **Availability:** which API exists on which OS version, from iOS 16 to iOS 27, and how to gate newer API.
- **Design and review:** choosing a form, a review checklist, and the limits of the framework with their escape hatches.

## Install

Copy the `swift-charts` folder into your agent's skills directory. For Claude Code:

```sh
# For every project
cp -R swift-charts ~/.claude/skills/

# Or for one project
cp -R swift-charts /path/to/your/project/.claude/skills/
```

The agent reads `swift-charts/SKILL.md` first and follows it to the reference each task needs.

## How it was made

The references were written from Apple's documentation and WWDC sessions, Apple's sample code, the SDK's own interface files and a set of community tutorials, each claim checked before it went in. Every code snippet in the references compiles, and behaviour that a compiler cannot prove was checked by rendering the chart. The skill has been tested by building a catalog app with it, screen by screen, and correcting whatever the builds and reviews found wrong.

## License

[MIT](LICENSE)
