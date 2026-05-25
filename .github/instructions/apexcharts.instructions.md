---
applyTo: "**/*chart*.ts,**/*chart*.html,**/charts/**/*.ts,**/charts/**/*.html"
description: ApexCharts in Angular — senior-level rules (v20/v21, signals, OnPush)
---

# ApexCharts rules

Charts are heavy, async, and side-effect-y — they break naive Angular patterns
(OnPush + signals) if integrated carelessly. Follow these rules.

## Library choice

- **Wrapper**: `ng-apexcharts` — first-class Angular wrapper, provides typed
  `ApexOptions`, `ChartComponent`, lifecycle hooks. **Never use raw
  `new ApexCharts(el, opts)` in components** unless there's a specific reason
  (e.g. dynamic chart inside a non-Angular library) — and document why in the file.
- **Imports**: only the types you need from `ng-apexcharts` —
  `NgApexchartsModule` for declaration, `ChartComponent` for `viewChild`,
  `ApexOptions` and per-section types (`ApexAxisChartSeries`, `ApexChart`,
  `ApexXAxis`, `ApexTooltip`, `ApexLegend`, `ApexFill`, `ApexStroke`,
  `ApexDataLabels`, `ApexGrid`, `ApexResponsive`, `ApexYAxis`).
- **Never** `import 'apexcharts/dist/apexcharts.css'` per-component — load
  globally in `styles.scss` or `angular.json`.

## Module loading & SSR

- ApexCharts touches `window` and `document` — **never render server-side**.
- In SSR/zoneless apps, wrap with `@defer (on viewport)` for below-the-fold,
  `@defer (on idle)` for above-the-fold but non-critical, or guard with
  `afterNextRender(() => { ... })` for charts that must mount eagerly.
- **Bundle**: ApexCharts core is ~400 KB. **Always lazy-load** the chart
  component via route-level `loadComponent` or `@defer` — never put it in the
  app shell. If the chart is reused across features, place it in
  `shared/charts/` and ensure consumers `@defer` it.

## Component skeleton (signal-first)

```ts
import {
  ChangeDetectionStrategy, Component, computed, inject, input, viewChild,
} from '@angular/core';
import {
  ChartComponent, NgApexchartsModule,
  ApexChart, ApexAxisChartSeries, ApexXAxis, ApexYAxis,
  ApexTooltip, ApexLegend, ApexStroke, ApexFill, ApexDataLabels, ApexGrid,
} from 'ng-apexcharts';
import { ChartThemeService } from '../../core/chart-theme.service';

type Options = {
  chart: ApexChart;
  series: ApexAxisChartSeries;
  xaxis: ApexXAxis;
  yaxis: ApexYAxis;
  tooltip: ApexTooltip;
  legend: ApexLegend;
  stroke: ApexStroke;
  fill: ApexFill;
  dataLabels: ApexDataLabels;
  grid: ApexGrid;
};

@Component({
  selector: 'app-revenue-chart',
  templateUrl: './revenue-chart.component.html',
  styleUrl: './revenue-chart.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [NgApexchartsModule],
})
export class RevenueChartComponent {
  private readonly theme = inject(ChartThemeService);

  readonly data = input.required<readonly { x: number; y: number }[]>();
  readonly title = input<string>('');
  readonly height = input<number>(320);

  private readonly chart = viewChild<ChartComponent>('chart');

  readonly options = computed<Options>(() => ({
    chart: {
      type: 'area',
      height: this.height(),
      toolbar: { show: false },
      zoom: { enabled: false },
      animations: { enabled: !this.theme.reducedMotion() },
      fontFamily: 'inherit',
      background: 'transparent',
    },
    series: [{ name: this.title(), data: [...this.data()] }],
    xaxis: { type: 'datetime' },
    yaxis: { labels: { formatter: (v) => this.theme.formatNumber(v) } },
    tooltip: { theme: this.theme.mode() },
    legend: { show: false },
    stroke: { curve: 'smooth', width: 2 },
    fill: { type: 'gradient', gradient: { opacityFrom: 0.4, opacityTo: 0 } },
    dataLabels: { enabled: false },
    grid: { borderColor: this.theme.gridColor() },
  }));
}
```

Template (`revenue-chart.component.html`):

```html
<div class="chart-host"
     role="img"
     [attr.aria-label]="title() || 'Revenue chart'">
  <apx-chart #chart
             [series]="options().series"
             [chart]="options().chart"
             [xaxis]="options().xaxis"
             [yaxis]="options().yaxis"
             [tooltip]="options().tooltip"
             [legend]="options().legend"
             [stroke]="options().stroke"
             [fill]="options().fill"
             [dataLabels]="options().dataLabels"
             [grid]="options().grid" />
</div>
```

## Reactivity rules

- **Options must be `computed()`** from input signals — when an input changes,
  Angular re-binds, `apx-chart` re-renders. **Do not** mutate a static
  `chartOptions` object on input changes — `OnPush` will swallow it.
- **Large series**: if `data()` is huge (>10k points) and updates often, skip
  the `computed()` rebind. Instead grab the chart ref and call:
  ```ts
  effect(() => this.chart()?.updateSeries([{ data: [...this.data()] }], false));
  ```
  Second arg `animate=false` for fast updates. **Use this sparingly** — it's
  imperative escape hatch.
- **`updateOptions` vs `updateSeries`**: prefer `updateSeries` when only data
  changes — it's an order of magnitude faster than rebuilding options.
- **Never** call `this.chart()?.render()` manually — `apx-chart` handles it.

## Forbidden in chart components

- `BehaviorSubject<ApexOptions>` — use signals + `computed()`
- Mutating `this.options.series[0].data.push(...)` — Angular won't see it
- Inline `template: ` with chart markup — separate `.html` always
- Constructor-injected services
- Calling chart methods inside `ngOnInit` — use `afterNextRender` or `effect()`
- Multiple `apx-chart` per host without unique container IDs (ApexCharts uses
  global IDs internally — collisions break tooltips)

## Theming & dark mode

- **Single source of truth**: `ChartThemeService` in `core/` exposing signals:
  `mode()` ('light' | 'dark'), `colors()`, `gridColor()`, `tooltipBg()`,
  `formatNumber(v)`, `formatDate(v)`, `reducedMotion()`.
- All chart components read from this service — **never hardcode hex colors**
  in chart options. Theme changes propagate via signals → `computed()` →
  rebind.
- Match project palette (Tailwind / Material / brand) — pull from CSS custom
  properties when possible:
  ```ts
  const grid = getComputedStyle(document.documentElement)
    .getPropertyValue('--color-border').trim();
  ```
  Compute once in the service, cache as signal.
- ApexCharts `theme.mode: 'dark'` is too aggressive for most apps — prefer
  manual `tooltip.theme`, `chart.background: 'transparent'`, custom
  `grid.borderColor`.

## Performance defaults

- `chart.animations.enabled: false` for data refreshes; `true` only on initial
  mount (or driven by `reducedMotion()`).
- `chart.toolbar.show: false` unless export/zoom is a feature. The toolbar
  bloats the chart and is rarely useful.
- `chart.zoom.enabled: false` unless explicitly needed.
- `dataLabels.enabled: false` by default — they kill perf at >50 points.
- For sparklines: `chart.sparkline: { enabled: true }` strips axes, grids,
  paddings — use it for KPI cards.
- For >5k points: use `decimate` plugin or downsample server-side; ApexCharts
  is not designed for huge datasets, switch to ECharts/uPlot if needed.

## Accessibility (non-negotiable)

ApexCharts is **not accessible by default**. Every chart component must:

- Wrap in `<div role="img" [attr.aria-label]="...">` (concise description).
- Provide `aria-describedby` pointing to a hidden `<table>` with the underlying
  data — screen readers can't read SVG charts otherwise.
- Respect `prefers-reduced-motion` — disable animations when `reducedMotion()`
  is true.
- Color contrast for series colors must meet WCAG AA against the background.
- Don't rely on color alone — use shape/pattern/label for series differentiation.

Skeleton for a11y table fallback:

```html
<div role="img" [attr.aria-label]="title()" aria-describedby="chart-data-{{id()}}">
  <apx-chart ... />
</div>
<table id="chart-data-{{id()}}" class="visually-hidden">
  <caption>{{ title() }}</caption>
  <thead><tr><th>Date</th><th>Value</th></tr></thead>
  <tbody>
    @for (point of data(); track point.x) {
      <tr><td>{{ point.x | date }}</td><td>{{ point.y }}</td></tr>
    }
  </tbody>
</table>
```

## Common pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| Chart renders at 0×0 | Container has no height at mount time | Set fixed `height` on options + min-height on host |
| Chart doesn't update on data change | Mutated options instead of replacing | `computed()` returning new object reference |
| Tooltip stuck / wrong values after series change | Same chart `id` reused across instances | Don't set `chart.id` manually unless unique |
| Memory leak in SPA navigation | Raw `new ApexCharts` not destroyed | Use `ng-apexcharts` wrapper, which handles destroy |
| Chart shows during SSR then re-renders (flash) | Not deferred | `@defer (on viewport)` + `@placeholder` skeleton |
| Date axis shows UTC, not local | `chart.toolbar.export` defaults | Set `xaxis.labels.datetimeUTC: false` |
| Animation jank on data refresh | `animations.enabled: true` during updates | Disable on update, enable only on first render |
| Different fonts vs rest of app | ApexCharts injects own font | `chart.fontFamily: 'inherit'` + global CSS reset |
| Series colors inconsistent across charts | Per-chart palette | Centralize in `ChartThemeService.colors()` |

## Testing

- Spec files must mock `NgApexchartsModule` — don't render real chart in unit
  tests (it's slow and DOM-heavy). Use `MockComponent` from `ng-mocks` or
  a manual stub.
- Test the `options` `computed()` signal — give different input signals,
  assert the returned `Options` shape. Don't test what ApexCharts renders.
- Visual regression / Playwright for actual chart rendering.

## Chart type cheatsheet

| Use case | Type | Notes |
|---|---|---|
| Trends over time | `area` or `line` | `xaxis.type: 'datetime'`, smooth curve |
| Comparisons across categories | `bar` (horizontal for long labels) | `plotOptions.bar.horizontal` |
| Part-to-whole, ≤6 slices | `donut` | Never `pie` — donut leaves space for total |
| Part-to-whole, >6 slices | `bar` stacked or treemap | Pie/donut illegible above 6 |
| Distribution | `boxPlot` or `histogram` | |
| Correlation | `scatter` | Disable lines, enable markers |
| KPI card sparkline | `area` + `sparkline: true` | No axes, no toolbar |
| Heatmap (calendar, matrix) | `heatmap` | Diverging palette |
| Real-time stream | `line` + `chart.animations.dynamicAnimation` | Throttle updates |
| OHLC / financial | `candlestick` | Date axis, custom tooltip |

## Anti-patterns — NEVER suggest these

| Don't | Do |
|---|---|
| `new ApexCharts(el, opts).render()` in component | `<apx-chart [series]=... [chart]=...>` |
| Hardcoded hex colors per chart | `ChartThemeService.colors()` |
| `[options]="staticOptions"` (mutable object) | `[series]="opts().series" [chart]="opts().chart" ...` (computed signal) |
| Chart in app shell or eager-loaded route | Lazy via `@defer` or `loadComponent` |
| `subscribe()` to data stream + manual `.updateSeries()` | `effect()` reading the data signal |
| Same `chart.id` on multiple instances | Let ApexCharts auto-generate |
| `animations.enabled: true` on every refresh | True on first render, false on update |
| Pie chart with 12 slices | Bar chart |
| Chart without `aria-label` and data fallback | Both, always |
