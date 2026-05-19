---
mode: agent
description: Diagnose and fix OnPush change detection issues in the selected component
---

The selected component is not updating its view as expected — likely an `OnPush` issue.
Diagnose and fix.

## Diagnostic checklist

Walk through these and report findings:

1. **Is the source of the data a signal?**
   - If yes: just call it in the template `{{ data() }}`. OnPush picks it up automatically.
   - If no (Observable, plain field, manual mutation): identify which.

2. **Is the data assigned via mutation instead of replacement?**
   ```ts
   this.items.push(newItem);          // ❌ OnPush won't fire
   this.items = [...this.items, x];   // ✅ new reference
   this._items.update(arr => [...arr, x]);  // ✅ signal update
   ```

3. **Is the change happening outside Angular's zone or detection cycle?**
   - Third-party callbacks (Web SDK, WebSocket, IntersectionObserver) often fire outside
   - Fix: wrap in `NgZone.run(...)`, OR (preferred) use a signal that you `.set()` — signals trigger CD regardless of zone

4. **Is the component receiving an `@Input()` that mutates internally?**
   - With OnPush, inputs trigger CD only on **reference change**
   - Convert to signal input (`input<T>()`) and mutate properly

5. **Is `async` pipe used correctly?**
   - `async` pipe auto-triggers CD on emission
   - If using `subscribe()` manually, you need `cdr.markForCheck()` — or convert to signal

6. **Is the parent using OnPush and not firing CD?**
   - Child's OnPush is fine, but parent must propagate change

## Fixes — in order of preference

### A. Convert to signals (best)

```ts
// BEFORE
@Input() items: Item[] = [];
addItem(i: Item) { this.items.push(i); }   // OnPush misses this

// AFTER
readonly items = input<Item[]>([]);
private readonly _localItems = signal<Item[]>([]);
addItem(i: Item) { this._localItems.update(arr => [...arr, i]); }
```

### B. Use immutable update

```ts
this.items = [...this.items, newItem];   // new reference triggers CD
```

### C. Manual `markForCheck` (last resort)

```ts
private readonly cdr = inject(ChangeDetectorRef);

onAsyncCallback(data: Data) {
  this.data = data;
  this.cdr.markForCheck();
}
```

### D. `NgZone.run` for outside-zone callbacks (if not using signals)

```ts
private readonly zone = inject(NgZone);

setupSocket() {
  socket.on('message', msg => {
    this.zone.run(() => {
      this.lastMessage = msg;
      this.cdr.markForCheck();
    });
  });
}
```

## Process

1. Identify the symptom (what doesn't update?)
2. Trace the data flow from source to template
3. Apply the highest-tier fix (prefer A over D)
4. Verify by re-reading the resulting code
5. Add a regression test if reasonable
