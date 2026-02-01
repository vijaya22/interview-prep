# React.js — Real Company Interview Questions

> Questions asked in actual React.js interviews at real companies.

---

## 1. What is React?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

React is an open-source JavaScript library developed by Facebook (now Meta) for building user interfaces, primarily for single-page applications. It follows a component-based architecture where the UI is broken down into reusable, self-contained components that manage their own state and compose together to form complex interfaces.

Key characteristics:

- **Declarative** — You describe *what* the UI should look like for a given state, and React handles updating the DOM efficiently. This is in contrast to imperative DOM manipulation where you manually specify each step.
- **Component-based** — Everything is a component. Components accept inputs (props), manage internal state, and return React elements describing what should appear on screen.
- **Virtual DOM** — React maintains a lightweight in-memory representation of the real DOM. When state changes, it creates a new Virtual DOM tree, diffs it against the previous one (reconciliation), and applies only the minimal set of changes to the real DOM.
- **Unidirectional data flow** — Data flows downward from parent to child via props, making the application more predictable and easier to debug.
- **Rich ecosystem** — While React itself is a view library (not a full framework), its ecosystem includes routing (React Router), state management (Redux, Zustand), server-side rendering (Next.js), and more.

React can be used for web apps (ReactDOM), mobile apps (React Native), and even desktop applications.

---

## 2. What is React's architecture?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

React's architecture is built around a few core layers:

1. **Component tree** — The UI is expressed as a tree of components. Each component is a function (or class) that receives props and returns React elements. Components can be nested, composed, and reused.

2. **React Fiber (reconciliation engine)** — Introduced in React 16, Fiber is the internal reconciliation engine. It represents the component tree as a linked list of "fiber nodes," enabling:
   - Incremental rendering — work can be split into chunks across multiple frames
   - Priority-based scheduling — urgent updates (user input) are processed before non-urgent ones (data fetching results)
   - Pause, abort, and resume — rendering work can be interrupted

3. **Virtual DOM and diffing** — When state or props change, React builds a new Virtual DOM tree, diffs it against the previous one using the reconciliation algorithm, and computes the minimal set of DOM mutations.

4. **Renderer layer** — React's core is renderer-agnostic. The reconciler determines *what* changed, and a renderer applies those changes to a specific target:
   - `react-dom` — for web browsers
   - `react-native` — for iOS/Android
   - `react-three-fiber` — for WebGL/3D

5. **Event system** — React implements its own synthetic event system that normalizes browser events and uses event delegation (attaching listeners to the root container rather than individual DOM nodes).

---

## 3. What is Virtual DOM?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

The Virtual DOM is a lightweight, in-memory JavaScript object representation of the real DOM tree. It acts as an abstraction layer between the developer's component code and actual browser DOM manipulation.

How it works:

1. **Render** — When a component renders, React creates a tree of plain JavaScript objects (React elements) describing the UI. This is the Virtual DOM.
2. **Diff** — When state changes trigger a re-render, React creates a *new* Virtual DOM tree and compares it with the previous one using the reconciliation algorithm (diffing).
3. **Patch** — React computes the minimal set of changes (insertions, deletions, updates) and applies only those changes to the real DOM in a batch.

Why it matters:

- Direct DOM manipulation is expensive because it triggers layout recalculations, repaints, and reflows.
- By batching and minimizing DOM operations, React reduces the performance cost of frequent UI updates.
- It also enables React to be renderer-agnostic — the same Virtual DOM diffing works whether the target is a browser, mobile app, or any other platform.

---

## 4. How can you distinguish between Virtual DOM vs ReactDOM?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

These are two completely different concepts:

**Virtual DOM:**
- A programming concept — a lightweight JavaScript object representation of the real DOM
- Lives in memory as a plain JS object tree
- Used internally by React's reconciliation algorithm to determine what changed
- Not a library or package — it's an implementation detail of React's core

**ReactDOM:**
- A package (`react-dom`) — the renderer that connects React to the browser DOM
- Provides methods like `createRoot()`, `render()`, `hydrate()`, and `createPortal()`
- Responsible for actually applying the changes (computed by the Virtual DOM diffing) to the real browser DOM
- One of many possible renderers (others include React Native, React Three Fiber)

In short: Virtual DOM figures out *what* needs to change; ReactDOM actually *makes* those changes in the browser.

```jsx
// ReactDOM in action — mounting React into the browser
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

---

## 5. Is the data flow in React bidirectional?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

No. React follows a **unidirectional (one-way) data flow**. Data flows downward from parent to child through props. A child component cannot directly modify its parent's state.

How it works:
- Parent owns state and passes it down as props
- Child communicates back to parent by calling callback functions passed as props
- This makes data flow predictable and easier to debug — you always know where data comes from

```jsx
function Parent() {
  const [value, setValue] = useState('');
  return <Child value={value} onChange={setValue} />;
}

function Child({ value, onChange }) {
  // Cannot modify parent state directly
  // Must use the callback prop
  return <input value={value} onChange={e => onChange(e.target.value)} />;
}
```

This is in contrast to frameworks like Angular which support two-way data binding via `ngModel`. React intentionally avoids this because one-way data flow makes it clearer where state mutations originate, which simplifies debugging in large applications.

---

## 6. What are hooks? Why do we use hooks?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

Hooks are functions introduced in React 16.8 that let you use state, lifecycle behavior, and other React features inside functional components — without writing class components.

**Core hooks:**
- `useState` — local state management
- `useEffect` — side effects (fetching, subscriptions, DOM mutations)
- `useContext` — consume context values
- `useRef` — mutable refs that persist across renders
- `useMemo` / `useCallback` — memoization for performance
- `useReducer` — complex state logic with a reducer pattern

**Why hooks were introduced:**

1. **Reuse stateful logic** — Before hooks, sharing logic between components required HOCs or render props, which led to "wrapper hell." Custom hooks allow extracting and reusing stateful logic without changing the component hierarchy.

2. **Simpler components** — Class components forced related logic to be split across lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). Hooks let you group related logic together in a single `useEffect`.

3. **No `this` keyword** — Classes required understanding JavaScript's `this` binding, which caused bugs (forgetting to bind event handlers). Functional components with hooks avoid this entirely.

4. **Better minification and optimization** — Functions minify better than classes, and hooks align well with future React optimizations like ahead-of-time compilation.

---

## 7. What is hydration in terms of SSR?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

Hydration is the process where React takes server-rendered HTML and "attaches" to it on the client side — adding event listeners and making the static HTML interactive, without recreating the DOM from scratch.

**How SSR + hydration works:**

1. **Server** renders the component tree to an HTML string and sends it to the browser. The user sees content immediately (good for SEO and perceived performance).
2. **Client** downloads the JavaScript bundle. React walks the existing server-rendered DOM, verifies it matches what it would have rendered, and attaches event handlers. This is hydration.
3. The page is now fully interactive.

```jsx
// Server
import { renderToString } from 'react-dom/server';
const html = renderToString(<App />);

// Client
import { hydrateRoot } from 'react-dom/client';
hydrateRoot(document.getElementById('root'), <App />);
```

**React 18 improvements:**
- **Selective hydration** — React can hydrate parts of the page independently using `Suspense` boundaries, rather than hydrating the entire tree at once
- **Streaming SSR** — `renderToPipeableStream` allows sending HTML in chunks as components resolve, combined with progressive hydration

**Key pitfall:** Hydration mismatches occur when the server-rendered HTML differs from what the client would render (e.g., using `Date.now()` or `window` on the server). React will warn about these in development.

---
