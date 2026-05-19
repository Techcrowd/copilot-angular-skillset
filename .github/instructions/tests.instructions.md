---
applyTo: "**/*.spec.ts"
description: Unit / integration testing rules
---

# Testing rules

> Adjust framework to match your project: **Jest** (recommended for new) or **Karma + Jasmine** (legacy).
> The examples below show both; remove the one you don't use.

## What to test

- **Components**: rendering with inputs, output emissions, user interactions, accessibility (focus, ARIA)
- **Services**: public API behavior — given inputs, what state/HTTP calls result
- **Stores**: state transitions, computed selectors, async actions
- **Guards / resolvers / interceptors**: redirect logic, header injection, error handling
- **Pipes**: input → output mapping including edge cases

## What NOT to test

- Implementation details (private methods, internal signal names)
- Framework internals (Angular's CD, HttpClient internals)
- Trivial getters / pure data classes
- Style / visual rendering (use Playwright / Storybook for that)

## Component test (with TestBed)

```ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { provideHttpClientTesting } from '@angular/common/http/testing';
import { ThingCardComponent } from './thing-card.component';

describe('ThingCardComponent', () => {
  let fixture: ComponentFixture<ThingCardComponent>;
  let component: ThingCardComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ThingCardComponent],
      providers: [provideHttpClientTesting()],
    }).compileComponents();

    fixture = TestBed.createComponent(ThingCardComponent);
    component = fixture.componentInstance;

    // Set signal inputs via fixture
    fixture.componentRef.setInput('id', 'thing-1');
    fixture.detectChanges();
  });

  it('renders the display name', () => {
    const heading = fixture.nativeElement.querySelector('h2');
    expect(heading?.textContent).toContain('—');     // initial state before resource loads
  });

  it('emits opened event on click', () => {
    let emitted: Thing | undefined;
    component.opened.subscribe(t => (emitted = t));

    fixture.nativeElement.querySelector('button').click();

    expect(emitted).toBeDefined();
  });
});
```

## Service test (with HttpTestingController)

```ts
import { HttpTestingController, provideHttpClientTesting } from '@angular/common/http/testing';
import { provideHttpClient } from '@angular/common/http';
import { TestBed } from '@angular/core/testing';
import { ThingService } from './thing.service';

describe('ThingService', () => {
  let svc: ThingService;
  let http: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [provideHttpClient(), provideHttpClientTesting()],
    });
    svc = TestBed.inject(ThingService);
    http = TestBed.inject(HttpTestingController);
  });

  afterEach(() => http.verify());

  it('creates a thing', async () => {
    const promise = svc.create({ name: 'X' });

    const req = http.expectOne('/api/things');
    expect(req.request.method).toBe('POST');
    req.flush({ id: '1', name: 'X' });

    const result = await promise;
    expect(result.id).toBe('1');
  });
});
```

## Signal-based input testing

```ts
// Setting signal inputs in tests:
fixture.componentRef.setInput('id', 'abc');
fixture.componentRef.setInput('compact', true);

// Reading signals:
expect(component.thing.value()).toEqual({ ... });
```

## Rules
- One `describe()` per class under test
- `it()` names start with verb: `it('emits ...', ...)`, `it('renders ...', ...)`
- One assertion concept per test (multiple `expect` calls OK if same concept)
- AAA pattern: Arrange / Act / Assert with blank lines between
- Use `fakeAsync` + `tick` for timer/async logic — never `setTimeout` in tests
- Mock external services via `provide: Service, useValue: { ... }`
- Never use `xit` / `xdescribe` in committed code

## What to avoid

- `done()` callbacks — use `async/await` or `fakeAsync`
- Snapshot tests of HTML output (brittle, low signal)
- Testing private methods — test through public API
- `TestBed.inject(...)` for things the component owns — inject through DI override instead
