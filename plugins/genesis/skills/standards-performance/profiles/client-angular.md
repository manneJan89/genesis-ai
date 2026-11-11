# Performance: client profile, Angular
- **Never bind a freshly created value into an input or template.** An object
  literal (`[config]="{ size: 'lg' }"`), array literal, `.map()`/`.filter()`
  result, or a getter or method that builds a new object produces a new
  reference on every change-detection pass. The child sees a "change" every
  time; with `ngOnChanges`, an `effect`, or a setter that emits back to the
  parent, that becomes an endless loop that hangs the page. Build the value once
  (a field set when its inputs change, a `computed` signal, or a pure pipe) and
  bind that.
- **No function calls in templates** except pure pipes and signals; a method
  call runs on every change-detection pass.
- **`ChangeDetectionStrategy.OnPush`** for new components, with immutable
  inputs.
- **Track lists:** `@for (...; track item.id)` (or `trackBy`), never track by
  index for lists that reorder.
- **Subscriptions end:** `takeUntilDestroyed()`, the `async` pipe, or signals;
  no manual `subscribe` without teardown.
- **Lazy-load routes** and keep the initial bundle within the project's budget
  (`angular.json` budgets).
