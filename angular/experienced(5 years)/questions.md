# Angular Interview Questions for Experienced Developers (5+ Years)

**Target Profile:** Senior Developer / Architect
**Focus Areas:** Change Detection, NgRx State Management, RxJS Memory Management, and Modern Angular (Ivy, Standalone Components).

---

### 1. Change Detection Architecture
**Question:** Explain Angular's Change Detection strategy. How does `Zone.js` fit into this, and how can `ChangeDetectionStrategy.OnPush` improve performance in a large application?
**Expert Answer:**
Angular uses `Zone.js` to monkey-patch asynchronous browser APIs (like `setTimeout`, `fetch`, Promises). When an async event finishes, `Zone.js` alerts Angular, which then triggers a top-down change detection cycle across the component tree.
By default, this is inefficient for complex trees. By switching components to `ChangeDetectionStrategy.OnPush`, you tell Angular to *only* check the component if its `@Input` references change, or an event is triggered from within the component itself. This requires you to write immutable data structures, dramatically cutting down processing overhead.

### 2. Handling RxJS Memory Leaks
**Question:** What are the most robust ways to prevent memory leaks in components that subscribe to multiple Observables?
**Expert Answer:**
Subscribing manually via `.subscribe()` without cleanup holds references to components after they are destroyed.
1. **Best Practice:** Use the `async` pipe in the HTML template. It automatically subscribes and safely unsubscribes when the component unmounts.
2. **TakeUntil Pattern:** In TypeScript, I create a `private destroy$ = new Subject<void>()`. I pipe my observables via `takeUntil(this.destroy$)`. Inside `ngOnDestroy()`, I emit `this.destroy$.next()` and complete it.
3. **Angular 16+:** Use `takeUntilDestroyed()` hook which automatically infers the current component's destroy context.

### 3. NgRx State Management
**Question:** Explain the flow of state in NgRx. Why are `Effects` used instead of calling APIs inside reducers or components?
**Expert Answer:**
Reducers are strictly synchronous pure functions. They take the current state and an Action to produce a new state.
`Effects` are an RxJS-powered model to isolate side effects (like HTTP calls). 
**Flow:**
Component dispatches an Action -> Effect listens for the Action, makes the API call -> Effect dispatches a "Success" or "Error" Action -> Reducer updates the Store -> Component reads state via Selectors.
This decouples the UI completely from the data-fetching logic, making both heavily testable.

### 4. Standalone Components (Modern Angular)
**Question:** Angular 14+ introduced Standalone Components. What architectural paradigm shift does this bring to enterprise apps, and how does it affect `NgModules`?
**Expert Answer:**
Standalone components (`standalone: true` flag in the decorator) remove the necessity of `NgModules`. This dramatically simplifies mental models, pulling Angular closer to React's component-based mental model.
Architecturally, it removes module boilerplate, makes lazy-loading easier (you can lazy-load a component directly), and heavily streamlines testing. In greenfield projects, `NgModule` is considered optional/legacy.

### 5. Custom Structural Directives & ViewContainerRef
**Question:** Briefly explain how you would create a custom structural directive (like `*ngIfRole`) that checks if a user has a specific role before rendering the DOM element.
**Expert Answer:**
I'd inject `TemplateRef` (the HTML inside the directive) and `ViewContainerRef` (the container where the HTML is going to be rendered). If the user holds the valid role, I use `this.viewContainer.createEmbeddedView(this.templateRef)`. If not, I call `this.viewContainer.clear()`.
