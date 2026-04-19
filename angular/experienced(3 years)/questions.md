# Angular Practical Guidelines & Interview Questions (3 Years)

**Target Profile:** Mid-Level Developer
**Focus Areas:** RxJS, Routing, Forms, Performance, and Interceptors.

---

### 1. RxJS: Observables vs Promises
**Question:** In an Angular app, HTTP client returns Observables. How do they differ from Promises, and when would you use a `Subject`?
**Practical Guideline Response:**
- **Promises** are eager (execute immediately), can't be cancelled, and resolve only a single value.
- **Observables** are lazy (don't execute until `.subscribe()` is called), can be easily cancelled using `unsubscribe()`, and can emit multiple values over time.
- I use a **Subject / BehaviorSubject** when I need to multicast data (e.g., maintaining a user session state across multiple components) because it acts as both an Observable and an Observer.

### 2. Forms Architecture
**Question:** When would you choose Reactive Forms over Template-Driven Forms?
**Practical Guideline Response:**
I prefer **Reactive Forms** for complex applications. 
Template-Driven forms are strictly HTML-first and asynchronous, making testing difficult. Reactive Forms define the logic in the TypeScript class. They are synchronous, highly scalable, and allow writing complex custom validators (like cross-field validation or async API validation) easily.

### 3. Implementing Route Guards
**Question:** How do you prevent unauthorized users from accessing the `/dashboard` route?
**Practical Guideline Response:**
I would implement an Angular Route Guard (`CanActivate`).
1. Create a guard file that injects the `AuthService` and `Router`.
2. Inside `canActivate()`, check if the user has a valid token.
3. If valid, return `true`.
4. If invalid, inject the `Router` to navigate to `/login` and return `false`.
5. Attach it to the routing module: `{ path: 'dashboard', component: Dashboard, canActivate: [AuthGuard] }`.

### 4. Lazy Loading
**Question:** The initial bundle size of your app is 5MB, and load time is extremely slow. How do you resolve this?
**Practical Guideline Response:**
I implement **Lazy Loading**. Instead of importing all modules in the main `AppModule`, I segment the app by feature (e.g., `AdminModule`, `ShopModule`). In the main routing file, I use the `loadChildren` syntax utilizing dynamic imports (`import('./admin/admin.module').then(m => m.AdminModule)`). This ensures the Admin JavaScript is only downloaded when the user actually navigates to that route.

### 5. HTTP Interceptors
**Question:** Explain how to attach an Authorization token to every outbound API request without modifying every single `HttpClient.get()` call.
**Practical Guideline Response:**
I create an `HttpInterceptor`. Whenever `HttpClient` makes a request, the interceptor catches it. I clone the request (`req.clone()`) and attach the `Authorization: Bearer <token>` to the headers. I then pass the cloned request forward using `next.handle(clonedRequest)`. I provide this interceptor in the `AppModule` providers using `HTTP_INTERCEPTORS`.
