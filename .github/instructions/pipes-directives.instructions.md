---
applyTo: "**/*.pipe.ts,**/*.directive.ts"
description: Standalone pipes and directives
---

# Pipes & directives

## Pipe

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({ name: 'truncate', standalone: true, pure: true })
export class TruncatePipe implements PipeTransform {
  transform(value: string | null | undefined, max = 50, ellipsis = '…'): string {
    if (!value) return '';
    return value.length > max ? value.slice(0, max) + ellipsis : value;
  }
}
```

Rules:
- `pure: true` (default) — never `pure: false`. Impure pipes destroy performance.
- Handle `null` / `undefined` gracefully — return empty string or sensible default
- One pipe per file
- Use signal-based services if the pipe needs reactive data — but usually a `computed()` in the component is better than an impure pipe

## Directive

```ts
import { Directive, ElementRef, HostListener, inject, input } from '@angular/core';

@Directive({
  selector: '[appTooltip]',
  standalone: true,
  host: {
    '[attr.aria-describedby]': 'tooltipId()',
  },
})
export class TooltipDirective {
  private readonly host = inject(ElementRef<HTMLElement>);

  readonly text = input.required<string>({ alias: 'appTooltip' });
  readonly tooltipId = computed(() => `tt-${crypto.randomUUID()}`);

  @HostListener('mouseenter')
  onEnter() { /* show */ }

  @HostListener('mouseleave')
  onLeave() { /* hide */ }
}
```

Rules:
- `standalone: true` always
- Use `host: {}` config over `@HostBinding` / `@HostListener` when possible (preferred in v17+)
- Selectors: `[appKebab]` for attribute, `app-kebab` for element (rare)
- Use `input()` for directive inputs with `alias` matching the selector
- Avoid `ElementRef.nativeElement` mutations — prefer `Renderer2` or `host: {}` bindings
