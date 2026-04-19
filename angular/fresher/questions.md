# Angular Interview Questions for Freshers

**Target Profile:** Entry-Level / Fresher
**Focus Areas:** Components, Directives, Data Binding, Lifecycle Hooks, and Basic TypeScript.

---

### 1. What is Data Binding in Angular?
**Question:** Explain the different types of Data Binding in Angular with syntax.
**Answer:**
Data Binding is the communication between the component's TypeScript code and the HTML template.
- **Interpolation (`{{ value }}`):** One-way binding from component to view.
- **Property Binding (`[property]="value"`):** One-way binding from component to an HTML property.
- **Event Binding (`(event)="handler()"`):** One-way binding from view to component when an event (like a click) happens.
- **Two-Way Binding (`[(ngModel)]="value"`):** Keeps the component and the view strictly in sync. Changes in one update the other immediately.

### 2. Directives: `*ngIf` vs `[hidden]`
**Question:** When conditionally showing an element, when should you use `*ngIf` vs `[hidden]`?
**Answer:**
- `*ngIf` is a structural directive. If the condition is false, Angular completely removes the element and its children from the DOM, saving performance and memory.
- `[hidden]` is a property binding that simply applies `display: none`. The element still exists in the DOM. Use `[hidden]` if the element is heavy to initialize and you need to toggle it frequently.

### 3. Components vs Modules
**Question:** What is the difference between a Component and a Module (`@NgModule`)?
**Answer:**
- A **Component** represents a section of the screen. It has its own HTML, CSS, and TS logic.
- An **NgModule** is a container that groups related components, services, and pipes together so Angular knows how they depend on each other.

### 4. Lifecycle: `constructor` vs `ngOnInit`
**Question:** Why do we define logic in `ngOnInit()` instead of the class `constructor()`?
**Answer:**
The `constructor()` is a standard JavaScript feature used to inject dependencies. When it runs, Angular's input properties (`@Input`) haven't been resolved yet. `ngOnInit()` is an Angular lifecycle hook that runs *after* Angular initializes the component and all input bindings are ready. 

### 5. Services and Dependency Injection
**Question:** What is a Service in Angular, and what does `@Injectable({ providedIn: 'root' })` mean?
**Answer:**
A Service is a class used to share data across components or interact with external APIs. Adding `@Injectable({ providedIn: 'root' })` registers the service at the root level, making a single shared instance (Singleton) available throughout the entire app without needing to list it in a module's providers array.
