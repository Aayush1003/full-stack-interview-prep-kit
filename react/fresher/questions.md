# React Fresher Interview Questions

This document contains a list of commonly asked React interview questions for a fresher (entry-level) position.

## 1. What is React?
React is an open-source frontend JavaScript library used for building user interfaces, especially for single-page applications. It is maintained by Meta (formerly Facebook) and a community of developers.

## 2. What are the major features of React?
- **JSX:** A JavaScript syntax extension used in React to write HTML within JavaScript.
- **Components:** The UI is divided into independent, reusable pieces called components.
- **Virtual DOM:** React uses a virtual representation of the DOM. When state changes, the Virtual DOM is updated, and React efficiently computes the difference (diffing) to update only the changed parts of the real DOM.
- **One-way data binding (Unidirectional data flow):** Data flows in one direction, from parent to child components, making it easier to track changes.

## 3. What is JSX?
JSX stands for JavaScript XML. It allows us to write HTML directly within JavaScript, which makes the code easier to understand and write. Behind the scenes, JSX is compiled into plain JavaScript by Babel.

## 4. What is the difference between Element and Component?
- **Element:** An element is a plain object describing what you want to appear on the screen in terms of DOM nodes or other components. Elements are immutable.
- **Component:** A component is a function or class that accepts inputs (props) and returns a React element. It can hold state and handle side effects.

## 5. What is the Virtual DOM? How does it differ from the Real DOM?
The Virtual DOM is a lightweight JavaScript representation of the Real DOM. Updating the Real DOM is slow and expensive. When the state of a React component changes, React updates the Virtual DOM first (which is fast). Then it compares the updated Virtual DOM with a snapshot of the previous Virtual DOM (a process called "diffing"). Finally, it updates the Real DOM but only with the parts that actually changed (a process called "reconciliation").

## 6. What are Props in React?
Props (short for properties) are used to pass data from a parent component to a child component. They are read-only (immutable), meaning child components cannot modify the props they receive.

## 7. What is State in React?
State is an object that holds some information that may change over the lifetime of the component. When the state object changes, the component re-renders to reflect the new changes in the UI.

## 8. What is the difference between State and Props?
- **State:** Managed within the component, can change over time, and is used to store data specific to that component.
- **Props:** Passed from parent to child, are read-only, and are used to configure the child component.

## 9. What are React Hooks?
React Hooks are functions that allow you to use state and other React features in functional components without writing a class.

## 10. Can you explain the `useState` hook?
`useState` is a built-in Hook that allows functional components to have state variables. It returns an array with two elements: the current state value and a function to update it.
```jsx
const [count, setCount] = useState(0);
```

## 11. Can you explain the `useEffect` hook?
`useEffect` allows you to perform side effects in functional components. Side effects include data fetching, manually changing the DOM, and setting up subscriptions. It runs automatically after every render by default, but you can control when it runs using the dependency array.
```jsx
useEffect(() => {
  // perform side effect
}, [dependencies]);
```

## 12. What are synthetic events in React?
Synthetic Events are cross-browser wrappers around the browser's native event system. React provides these so that events have a consistent interface and behavior across all browsers.

## 13. What is the significance of `key` props in React?
Keys help React identify which items have changed, are added, or are removed from a list. Keys should be given to the elements inside an array to give the elements a stable identity. This makes the diffing process and rendering of lists more efficient.

## 14. What are the Rules of Hooks?
There are two main rules to follow when using React Hooks:
- **Only Call Hooks at the Top Level:** Never call Hooks inside loops, conditions, or nested functions. Always use them at the top level of your React function.
- **Only Call Hooks from React Functions:** You must call Hooks from React functional components or from custom Hooks. Do not call them from regular JavaScript functions.

## 15. What is the `useContext` hook?
`useContext` is a hook that allows you to access and consume data from a React Context. It provides a way to pass data deeply through the component tree without having to pass props down manually at every level (avoiding "prop drilling").

## 16. What is the `useRef` hook?
`useRef` is a hook that returns a mutable ref object whose `.current` property is initialized to the passed argument. It is primarily used to access a DOM element directly or to store a mutable value that does not cause a re-render when it is updated.

## 17. What are Custom Hooks?
Custom hooks are regular JavaScript functions whose names start with "use" and that may call other Hooks. They allow you to extract common component logic into reusable functions, making it easier to share logic across multiple components without duplicating code.
