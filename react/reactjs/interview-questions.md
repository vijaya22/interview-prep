# React.js Interview Questions & Answers

---

## 1. What is React?

React is a JavaScript library developed by Facebook for building user interfaces, specifically single-page applications. It allows developers to create reusable UI components and efficiently update the DOM using a virtual DOM.

---

## 2. What is JSX?

JSX stands for JavaScript XML. It is a syntax extension for JavaScript that lets you write HTML-like code inside JavaScript. JSX is not valid JavaScript on its own — it gets transpiled by tools like Babel into `React.createElement()` calls.

```jsx
const element = <h1>Hello, world!</h1>;
// Transpiles to:
const element = React.createElement('h1', null, 'Hello, world!');
```

---

## 3. What is the Virtual DOM and how does it work?

The Virtual DOM is a lightweight in-memory representation of the real DOM. When state changes occur:

1. React creates a new Virtual DOM tree.
2. It diffs the new tree with the previous one (reconciliation).
3. It calculates the minimal set of changes needed.
4. It batches and applies only those changes to the real DOM.

This makes updates faster because direct DOM manipulation is expensive.

---

## 4. What is the difference between a functional component and a class component?

| Feature | Functional Component | Class Component |
|---|---|---|
| Syntax | Plain function | ES6 class extending `React.Component` |
| State | Via `useState` hook | Via `this.state` |
| Lifecycle | Via `useEffect` hook | Via lifecycle methods (`componentDidMount`, etc.) |
| `this` keyword | Not used | Required |
| Performance | Slightly lighter | Slightly heavier |

Functional components with hooks are the modern standard.

---

## 5. What are React Hooks?

Hooks are functions introduced in React 16.8 that let you use state and other React features in functional components. Key hooks:

- **useState** — manage local state
- **useEffect** — handle side effects (data fetching, subscriptions, DOM mutations)
- **useContext** — consume context values
- **useRef** — persist mutable values across renders without causing re-renders
- **useMemo** — memoize expensive computations
- **useCallback** — memoize function references
- **useReducer** — manage complex state logic
- **useLayoutEffect** — like useEffect but fires synchronously after DOM mutations

---

## 6. What are the rules of Hooks?

1. **Only call hooks at the top level** — never inside loops, conditions, or nested functions.
2. **Only call hooks from React functions** — either functional components or custom hooks.

These rules ensure React can correctly track hook state between re-renders using call order.

---

## 7. What is `useState`?

`useState` is a hook that adds local state to a functional component. It returns an array with the current state value and a setter function.

```jsx
const [count, setCount] = useState(0);

// Updating with a value
setCount(5);

// Updating based on previous state
setCount(prev => prev + 1);
```

State updates are asynchronous and batched.

---

## 8. What is `useEffect`?

`useEffect` lets you perform side effects in functional components. It runs after render.

```jsx
// Runs on every render
useEffect(() => { /* ... */ });

// Runs only on mount
useEffect(() => { /* ... */ }, []);

// Runs when `id` changes
useEffect(() => { /* ... */ }, [id]);

// Cleanup function (runs before next effect or unmount)
useEffect(() => {
  const sub = subscribe(id);
  return () => sub.unsubscribe();
}, [id]);
```

---

## 9. What is the difference between `useMemo` and `useCallback`?

- **`useMemo`** memoizes a **computed value**. It recalculates only when dependencies change.
- **`useCallback`** memoizes a **function reference**. It returns the same function instance unless dependencies change.

```jsx
const expensiveValue = useMemo(() => computeExpensive(a, b), [a, b]);
const handleClick = useCallback(() => doSomething(id), [id]);
```

`useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

---

## 10. What is `useRef`?

`useRef` returns a mutable ref object whose `.current` property persists across renders without triggering re-renders.

Common uses:
- Accessing DOM elements directly
- Storing mutable values (like timers, previous state)

```jsx
const inputRef = useRef(null);

const focusInput = () => inputRef.current.focus();

return <input ref={inputRef} />;
```

---

## 11. What is `useReducer`?

`useReducer` is an alternative to `useState` for managing complex state logic. It works similarly to Redux reducers.

```jsx
const reducer = (state, action) => {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    default: return state;
  }
};

const [state, dispatch] = useReducer(reducer, { count: 0 });

dispatch({ type: 'increment' });
```

Use `useReducer` when state transitions depend on previous state or when you have multiple related state values.

---

## 12. What is Context API?

Context provides a way to pass data through the component tree without manually passing props at every level (prop drilling).

```jsx
const ThemeContext = React.createContext('light');

// Provider
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>

// Consumer (using hook)
const theme = useContext(ThemeContext);
```

---

## 13. What is prop drilling and how do you avoid it?

Prop drilling is passing props through multiple intermediate components that don't need the data themselves, just to get it to a deeply nested child.

Solutions:
- **Context API** — built-in React solution
- **State management libraries** — Redux, Zustand, Jotai, Recoil
- **Component composition** — restructure components to avoid deep nesting

---

## 14. What is the difference between controlled and uncontrolled components?

**Controlled component:** Form data is handled by React state. The component's value is driven by state, and changes go through event handlers.

```jsx
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />
```

**Uncontrolled component:** Form data is handled by the DOM itself. You use refs to access values.

```jsx
const inputRef = useRef();
<input ref={inputRef} defaultValue="hello" />
// Access: inputRef.current.value
```

Controlled components are preferred in most cases because they give you full control over form behavior.

---

## 15. What is React.memo?

`React.memo` is a higher-order component that memoizes a functional component. It prevents re-renders if props haven't changed (shallow comparison).

```jsx
const ExpensiveList = React.memo(({ items }) => {
  return items.map(item => <div key={item.id}>{item.name}</div>);
});
```

You can pass a custom comparison function as the second argument.

---

## 16. What are keys in React and why are they important?

Keys are unique identifiers assigned to elements in a list. They help React identify which items have changed, been added, or removed during reconciliation.

```jsx
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

Rules:
- Keys must be **stable**, **unique**, and **predictable**.
- Never use array index as a key if the list can be reordered, filtered, or items can be inserted/removed.
- Using index as a key can cause bugs with component state and incorrect rendering.

---

## 17. What is the difference between `useEffect` and `useLayoutEffect`?

| Feature | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Timing | Runs asynchronously after paint | Runs synchronously after DOM mutation, before paint |
| Use case | Most side effects | DOM measurements, synchronous visual updates |
| Blocking | Does not block rendering | Blocks rendering until complete |

Use `useLayoutEffect` when you need to read DOM layout and synchronously re-render before the user sees the update (e.g., measuring element size, preventing flicker).

---

## 18. What is React's reconciliation algorithm?

Reconciliation is the process React uses to diff two Virtual DOM trees and determine the minimum changes to apply to the real DOM.

Key assumptions:
1. Elements of different types produce different trees — React tears down the old tree and builds a new one.
2. Keys indicate which child elements are stable across renders.

This allows React to achieve O(n) diffing instead of O(n^3).

---

## 19. What are Higher-Order Components (HOCs)?

A HOC is a function that takes a component and returns a new component with additional props or behavior.

```jsx
function withLoading(WrappedComponent) {
  return function WithLoading({ isLoading, ...props }) {
    if (isLoading) return <Spinner />;
    return <WrappedComponent {...props} />;
  };
}

const EnhancedList = withLoading(UserList);
```

HOCs are a pattern from the class component era. Custom hooks are the modern alternative for sharing logic.

---

## 20. What are render props?

Render props is a pattern where a component receives a function as a prop (or children) and calls it to determine what to render.

```jsx
<DataFetcher url="/api/users">
  {({ data, loading }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )}
</DataFetcher>
```

Like HOCs, this pattern has largely been replaced by custom hooks.

---

## 21. What are custom hooks?

Custom hooks are JavaScript functions whose names start with `use` that encapsulate reusable stateful logic.

```jsx
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });

  useEffect(() => {
    const handleResize = () =>
      setSize({ width: window.innerWidth, height: window.innerHeight });

    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Usage
const { width, height } = useWindowSize();
```

---

## 22. What is React.lazy and Suspense?

`React.lazy` enables code-splitting by lazily loading components. `Suspense` provides a fallback UI while the lazy component loads.

```jsx
const LazyDashboard = React.lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <LazyDashboard />
    </Suspense>
  );
}
```

---

## 23. What are error boundaries?

Error boundaries are class components that catch JavaScript errors in their child component tree, log the errors, and display a fallback UI.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToService(error, info);
  }

  render() {
    if (this.state.hasError) return <h1>Something went wrong.</h1>;
    return this.props.children;
  }
}
```

Error boundaries do **not** catch errors in event handlers, async code, SSR, or errors thrown in the boundary itself.

---

## 24. What is React.Fragment?

`React.Fragment` lets you group multiple elements without adding an extra DOM node.

```jsx
// Verbose
<React.Fragment>
  <h1>Title</h1>
  <p>Description</p>
</React.Fragment>

// Short syntax
<>
  <h1>Title</h1>
  <p>Description</p>
</>
```

The verbose syntax supports the `key` prop, which is needed in lists.

---

## 25. What is the difference between state and props?

| Feature | Props | State |
|---|---|---|
| Ownership | Passed from parent | Owned by the component |
| Mutability | Read-only (immutable) | Mutable via setter |
| Purpose | Configure a component | Track internal data |
| Trigger re-render | When parent re-renders with new props | When state is updated |

---

## 26. What are synthetic events in React?

React wraps native browser events in `SyntheticEvent` objects to provide a consistent API across browsers. Key characteristics:

- Cross-browser compatibility
- Same interface as native events (`stopPropagation`, `preventDefault`)
- Events are pooled (recycled) in React < 17 for performance
- React 17+ removed event pooling

---

## 27. What is event delegation in React?

In React 16 and earlier, React attaches a single event listener at the `document` level for all events (event delegation). In React 17+, events are attached to the root DOM container instead of `document`. This improves interop when multiple React trees or non-React code exist on the same page.

---

## 28. What is lifting state up?

Lifting state up means moving shared state to the closest common ancestor of the components that need it. The parent component holds the state and passes it down as props along with update functions.

```jsx
function Parent() {
  const [value, setValue] = useState('');
  return (
    <>
      <InputComponent value={value} onChange={setValue} />
      <DisplayComponent value={value} />
    </>
  );
}
```

---

## 29. What is React Strict Mode?

`React.StrictMode` is a development-only wrapper that helps find potential problems:

- Detects unsafe lifecycle methods
- Warns about legacy API usage (string refs, findDOMNode)
- Detects unexpected side effects by double-invoking certain functions (render, constructors, effects)
- Warns about deprecated patterns

```jsx
<React.StrictMode>
  <App />
</React.StrictMode>
```

It does not render any visible UI and has no effect in production.

---

## 30. What is React Fiber?

React Fiber is the reimplementation of React's core reconciliation algorithm (introduced in React 16). Key improvements:

- **Incremental rendering** — ability to split rendering work into chunks and spread it over multiple frames
- **Prioritization** — ability to assign priority to different types of updates
- **Pause, abort, resume** — ability to pause work and come back later
- **Concurrency support** — foundation for concurrent features

---

## 31. What are portals in React?

Portals allow rendering children into a DOM node outside the parent component's DOM hierarchy while preserving React's event bubbling.

```jsx
import { createPortal } from 'react-dom';

function Modal({ children }) {
  return createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root')
  );
}
```

Common use cases: modals, tooltips, dropdowns.

---

## 32. What is `forwardRef`?

`forwardRef` lets a component receive a `ref` and forward it to a child DOM element or component.

```jsx
const FancyInput = React.forwardRef((props, ref) => (
  <input ref={ref} className="fancy" {...props} />
));

// Parent
const inputRef = useRef();
<FancyInput ref={inputRef} />
```

---

## 33. What is `useImperativeHandle`?

`useImperativeHandle` customizes the value exposed to parent components when using `ref` with `forwardRef`.

```jsx
const FancyInput = React.forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => { inputRef.current.value = ''; }
  }));

  return <input ref={inputRef} />;
});
```

---

## 34. How does React handle batching of state updates?

**React 17 and earlier:** State updates are batched only inside React event handlers. Updates in `setTimeout`, promises, or native event handlers are not batched.

**React 18+:** Automatic batching applies everywhere — event handlers, timeouts, promises, and native event handlers. All state updates are batched by default.

```jsx
// React 18: both updates cause a single re-render
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
}, 1000);
```

Use `flushSync` to opt out of batching when needed.

---

## 35. What are React Server Components (RSC)?

Server Components run only on the server and send rendered output (not JavaScript) to the client.

Benefits:
- Zero bundle size for server components
- Direct access to server-side resources (database, file system)
- Automatic code splitting

Key rules:
- Server Components cannot use state, effects, or browser-only APIs
- Server Components can import Client Components but not vice versa
- Client Components are marked with `'use client'` directive

---

## 36. What is Concurrent Rendering in React 18?

Concurrent rendering allows React to prepare multiple versions of the UI at the same time. Key features:

- **`useTransition`** — mark state updates as non-urgent, keeping the UI responsive
- **`useDeferredValue`** — defer re-rendering of a value until higher-priority updates are done
- **`startTransition`** — wrap state updates to mark them as transitions

```jsx
const [isPending, startTransition] = useTransition();

const handleSearch = (query) => {
  // Urgent: update input
  setInput(query);

  // Non-urgent: update results
  startTransition(() => {
    setSearchResults(filterResults(query));
  });
};
```

---

## 37. What is `useDeferredValue`?

`useDeferredValue` accepts a value and returns a deferred version that may lag behind the original. React will first render with the old value, then attempt a re-render with the new value in the background.

```jsx
const deferredQuery = useDeferredValue(query);

// Use deferredQuery for expensive rendering
<ExpensiveList query={deferredQuery} />
```

Useful for keeping the UI responsive when rendering expensive components.

---

## 38. What is the difference between `useTransition` and `useDeferredValue`?

| Feature | `useTransition` | `useDeferredValue` |
|---|---|---|
| What it wraps | State update (setter call) | A value |
| Control | You wrap the update | You wrap the consumed value |
| Use when | You own the state update | You receive the value as a prop |
| Returns | `[isPending, startTransition]` | Deferred value |

---

## 39. How do you optimize React performance?

Key techniques:
- **React.memo** — prevent unnecessary re-renders of components
- **useMemo / useCallback** — memoize values and functions
- **Code splitting** — `React.lazy` + `Suspense`
- **Virtualization** — render only visible items in long lists (react-window, react-virtuoso)
- **Avoid inline object/function creation** in render when passed as props
- **useTransition / useDeferredValue** — keep UI responsive during heavy updates
- **Key prop** — use stable keys in lists
- **Profiler** — use React DevTools Profiler to identify bottlenecks

---

## 40. What is the React component lifecycle (class components)?

**Mounting:**
1. `constructor()`
2. `static getDerivedStateFromProps()`
3. `render()`
4. `componentDidMount()`

**Updating:**
1. `static getDerivedStateFromProps()`
2. `shouldComponentUpdate()`
3. `render()`
4. `getSnapshotBeforeUpdate()`
5. `componentDidUpdate()`

**Unmounting:**
1. `componentWillUnmount()`

**Error Handling:**
1. `static getDerivedStateFromError()`
2. `componentDidCatch()`

---

## 41. How does `useEffect` map to class lifecycle methods?

| Class Lifecycle | Hook Equivalent |
|---|---|
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {}, [deps])` |
| `componentWillUnmount` | `useEffect(() => { return () => cleanup() }, [])` |
| `shouldComponentUpdate` | `React.memo` |
| `getDerivedStateFromProps` | Update state during render |

---

## 42. What is `StrictMode` double rendering and why does it happen?

In development mode, `React.StrictMode` intentionally double-invokes:
- Component function bodies
- State updater functions
- `useReducer` reducers
- `useEffect` setup and cleanup (React 18)

This helps catch impure renders and side effects. If your component works correctly with double rendering, it follows React's rules. The double invocation does not happen in production.

---

## 43. What is `dangerouslySetInnerHTML`?

`dangerouslySetInnerHTML` is React's equivalent of setting `innerHTML`. It accepts an object with a `__html` key.

```jsx
<div dangerouslySetInnerHTML={{ __html: '<b>Bold text</b>' }} />
```

It is named "dangerous" because it exposes your application to XSS attacks. Always sanitize HTML content (e.g., with DOMPurify) before using it.

---

## 44. What is the difference between `createElement` and `cloneElement`?

- **`createElement(type, props, children)`** — creates a new React element.
- **`cloneElement(element, props, children)`** — clones an existing element and merges in new props.

```jsx
// createElement
const el = React.createElement('div', { className: 'box' }, 'Hello');

// cloneElement
const cloned = React.cloneElement(el, { id: 'main' });
// Result: <div className="box" id="main">Hello</div>
```

---

## 45. What are common patterns for state management in React?

| Approach | Best For |
|---|---|
| `useState` / `useReducer` | Local component state |
| Context API | Low-frequency global state (theme, auth, locale) |
| Redux / Redux Toolkit | Large-scale apps with complex state logic |
| Zustand | Simple global state with minimal boilerplate |
| Jotai / Recoil | Atomic state management |
| React Query / TanStack Query | Server state (caching, fetching, syncing) |

Rule of thumb: keep state as local as possible, lift it only when needed.

---

## 46. What is the difference between server-side rendering (SSR) and client-side rendering (CSR)?

| Feature | CSR | SSR |
|---|---|---|
| Initial load | Blank page + JS download | Full HTML from server |
| SEO | Poor (without workarounds) | Good |
| Time to interactive | Faster after initial load | May be slower (hydration needed) |
| Server load | Minimal | Higher |

Frameworks like Next.js support SSR, SSG (Static Site Generation), and ISR (Incremental Static Regeneration).

---

## 47. What is hydration?

Hydration is the process where React takes over server-rendered HTML and attaches event listeners to make it interactive. React reuses the existing DOM instead of recreating it.

React 18 introduced **selective hydration** — it can hydrate parts of the page independently and prioritize hydrating components the user is interacting with.

---

## 48. What is the `children` prop?

`children` is a special prop that contains the content between a component's opening and closing tags.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card>
  <h2>Title</h2>
  <p>Content here</p>
</Card>
```

You can manipulate children using `React.Children` utilities (`map`, `forEach`, `count`, `toArray`).

---

## 49. What are compound components?

Compound components is a pattern where multiple components work together to form a cohesive unit, sharing implicit state.

```jsx
<Select onChange={handleChange}>
  <Select.Option value="a">Option A</Select.Option>
  <Select.Option value="b">Option B</Select.Option>
  <Select.Option value="c">Option C</Select.Option>
</Select>
```

This is typically implemented using Context internally to share state between the parent and child components.

---

## 50. What are some common React anti-patterns?

1. **Mutating state directly** — always use the setter function
2. **Using index as key** in dynamic lists
3. **Overusing `useEffect`** — don't use effects for things that can be computed during render
4. **Prop drilling** through many layers instead of using Context or composition
5. **Premature optimization** — using `useMemo`/`useCallback` everywhere without measuring
6. **God components** — components with too many responsibilities
7. **Derived state in `useEffect`** — compute derived values during render instead
8. **Not cleaning up effects** — leads to memory leaks
9. **State that can be computed** — storing values in state that can be derived from existing state/props
10. **Syncing state between components** — use a single source of truth instead
