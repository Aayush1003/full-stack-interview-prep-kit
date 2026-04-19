# React Practical Guidelines & Interview Questions (3 Years Experience)

**Target Profile:** Mid-Level Developer (TCS Wings1 / Digital / Prime / Elevate style)
**Focus Areas:** Practical Custom Hooks, API Integration, Component Architecture, State Management implementations, and Testing.

At the 3-year level, evaluations shift from "What is an element?" to "Here is a broken component, how do you fix it?" and "How do you structure an authentication flow?"

---

## 1. Scenario-Based Component Architecture

### Scenario: The Token Refresh / Axios Interceptor 
**Question:** Your application's JWT access token expires every 15 minutes. How do you quietly refresh the token without logging the user out and forcing them to re-click whatever button they just clicked?

**Practical Guideline Response:**
I would implement an **Axios Response Interceptor**. 
1. When Axios makes a request, if it receives a `401 Unauthorized` error, the interceptor catches it.
2. It pauses the original request, makes a call to the `/api/refresh-token` endpoint natively.
3. If the refresh succeeds, the interceptor updates the local storage/context with the new access token.
4. It then modifies the original, paused request's authorization header and *retries* the request seamlessly. The user never notices.
5. If the refresh fails, it redirects to the `/login` route.

### Scenario: Protected Routes
**Question:** How precisely do you implement Protected Routes in React Router DOM (v6)?

**Practical Guideline Response:**
I create a custom wrapper component called `<ProtectedRoute>`.
```jsx
// Practical Implementation
const ProtectedRoute = ({ children, allowedRoles }) => {
  const { user, isAuthenticated } = useAuth(); // From Context or Redux

  if (!isAuthenticated) return <Navigate to="/login" replace />;
  if (allowedRoles && !allowedRoles.includes(user.role)) return <Navigate to="/unauthorized" replace />;
  
  return children; // Renders the protected page
};

// Route Definition
<Route path="/admin" element={
  <ProtectedRoute allowedRoles={['ADMIN']}>
    <AdminDashboard />
  </ProtectedRoute>
} />
```

---

## 2. Practical Hooks & Optimization

### Scenario: The Autocomplete Search Bar
**Question:** You have a search input. Every time the user types a letter, an API call is fired. If they type "Laptop", 6 API calls are fired, lagging the server. Fix this practically.

**Practical Guideline Response:**
I would implement a **Debounce** mechanism using a Custom Hook (`useDebounce`).
```jsx
// The custom hook
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    // Crucial: Cleanup function to clear the timeout if value changes before delay
    return () => clearTimeout(handler); 
  }, [value, delay]);
  return debouncedValue;
}
```
In the component, I pass the user's input to `useDebounce(input, 500)`, and I only trigger the `useEffect` API fetch when the `debouncedValue` changes, successfully waiting exactly 500ms after the user *stops* typing.

### Scenario: Identifying Unnecessary Re-renders
**Question:** A parent component holds an `onClick` callback function and passes it to a child component. Every time the parent's state updates, the child component unnecessarily re-renders, even though the child is wrapped in `React.memo()`. Why?

**Practical Guideline Response:**
`React.memo` does a shallow comparison of props. When the parent re-renders, the `onClick` function is *re-created* in memory, meaning the child sees a brand new function reference and re-renders anyway.
**The Fix:** Wrap the `onClick` function in the parent inside a `useCallback` hook. This preserves the function's memory reference across parent re-renders, allowing `React.memo` in the child to do its job.

---

## 3. State Management (When to use What)

### Scenario: Form Handling
**Question:** Your form has 15 input fields. Do you create 15 `useState` variables? 

**Practical Guideline Response:**
No. Creating 15 separate states is terrible practice. 
- **Approach 1:** I use a single `useState` holding an object: `const [formData, setFormData] = useState({ name: '', email: '', ... })` and use a unified `onChange` handler leveraging the `e.target.name` property to update the specific key.
- **Approach 2 (Better):** I use **React Hook Form**. It uses uncontrolled components under the hood (Refs), dramatically reducing re-renders while typing, and provides robust validation handling (like integrating with `Yup` or `Zod`).

### Scenario: Redux vs Context Practicality
**Question:** When would you absolutely NOT use Redux, and when would you absolutely NOT use Context API?

**Practical Guideline Response:**
- **Skip Redux when:** I only need to pass a theming variable (dark/light) or simple user auth session data down a component tree. It's too much boilerplate for data that changes rarely. Context API is perfect here.
- **Skip Context when:** I have high-frequency, complex changing data (like a trading dashboard or multiple interactive tables). Context causes all consumers to re-render when *any* part of the context changes. Here, Redux Toolkit (with precise selectors) or Zustand is mandatory for performance.

---

## 4. Testing & Code Quality

### Scenario: Testing an API Component
**Question:** How do you test a component that fetches data when it mounts using Jest and React Testing Library? Do you hit the real API?

**Practical Guideline Response:**
No, we never hit a real real API in unit tests.
1. I mock the global `fetch` or `axios` module using `jest.mock('axios')`.
2. I provide a mock response: `axios.get.mockResolvedValue({ data: [...] })`.
3. I use `render(<MyComponent />)` from React Testing Library.
4. I assert that a loading spinner appears first (`getByTestId`).
5. I use `waitFor` or `findByText` to assert that the mock data eventually appears in the DOM.
