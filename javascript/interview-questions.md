# JavaScript Interview Questions & Answers

---

## 1. What are the different data types in JavaScript?

JavaScript has 8 data types:

**Primitive types (7):**
- `string` — textual data
- `number` — integers and floats (64-bit IEEE 754)
- `bigint` — arbitrary precision integers
- `boolean` — `true` or `false`
- `undefined` — declared but not assigned
- `null` — intentional absence of value
- `symbol` — unique, immutable identifier

**Non-primitive (1):**
- `object` — collections of key-value pairs (includes arrays, functions, dates, etc.)

```js
typeof 'hello'     // 'string'
typeof 42          // 'number'
typeof true        // 'boolean'
typeof undefined   // 'undefined'
typeof null        // 'object' (historical bug)
typeof Symbol()    // 'symbol'
typeof 10n         // 'bigint'
typeof {}          // 'object'
typeof []          // 'object'
typeof function(){} // 'function'
```

---

## 2. What is the difference between `var`, `let`, and `const`?

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function-scoped | Block-scoped | Block-scoped |
| Hoisting | Hoisted (initialized as `undefined`) | Hoisted (not initialized, TDZ) | Hoisted (not initialized, TDZ) |
| Re-declaration | Allowed | Not allowed | Not allowed |
| Re-assignment | Allowed | Allowed | Not allowed |
| Global object property | Yes (`window.x`) | No | No |

```js
// var — function scoped
function example() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 (accessible outside if block)
}

// let — block scoped
function example() {
  if (true) {
    let y = 10;
  }
  console.log(y); // ReferenceError
}

// const — must be initialized, binding is immutable
const obj = { a: 1 };
obj.a = 2;       // OK — mutating the object is allowed
obj = { a: 2 };  // TypeError — reassignment not allowed
```

---

## 3. What is hoisting?

Hoisting is JavaScript's behavior of moving declarations to the top of their scope during the compilation phase (before execution).

```js
// What you write:
console.log(x); // undefined (not ReferenceError)
var x = 5;

// How JS interprets it:
var x;
console.log(x); // undefined
x = 5;

// Functions are fully hoisted
greet(); // "Hello" — works
function greet() { console.log("Hello"); }

// Function expressions are NOT fully hoisted
greet(); // TypeError: greet is not a function
var greet = function() { console.log("Hello"); };

// let/const — hoisted but in Temporal Dead Zone (TDZ)
console.log(a); // ReferenceError: Cannot access 'a' before initialization
let a = 10;
```

---

## 4. What is the Temporal Dead Zone (TDZ)?

The TDZ is the period between entering a scope and the point where a `let` or `const` variable is declared. Accessing the variable during TDZ throws a `ReferenceError`.

```js
{
  // TDZ starts for 'x'
  console.log(x); // ReferenceError
  let x = 10;     // TDZ ends
  console.log(x); // 10
}
```

TDZ exists to catch errors — using variables before declaration is almost always a bug.

---

## 5. What are closures?

A closure is a function that retains access to variables from its outer (enclosing) scope, even after the outer function has returned.

```js
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
counter.getCount(); // 2
```

Common use cases:
- Data privacy / encapsulation
- Function factories
- Callbacks and event handlers
- Partial application and currying
- Module pattern

---

## 6. What is the event loop?

The event loop is the mechanism that allows JavaScript to perform non-blocking I/O operations despite being single-threaded.

**Execution order:**
1. Execute the current call stack (synchronous code)
2. Process all microtasks (Promises, `queueMicrotask`, `MutationObserver`)
3. Process one macrotask (setTimeout, setInterval, I/O, UI rendering)
4. Repeat

```js
console.log('1');                          // Sync

setTimeout(() => console.log('2'), 0);     // Macrotask

Promise.resolve().then(() => console.log('3')); // Microtask

console.log('4');                          // Sync

// Output: 1, 4, 3, 2
```

Microtasks always run before macrotasks. The microtask queue is fully drained before the next macrotask executes.

---

## 7. What is the difference between `==` and `===`?

- **`==` (loose equality)** — compares values after type coercion
- **`===` (strict equality)** — compares values without type coercion (type + value must match)

```js
0 == false        // true (false coerced to 0)
0 === false       // false (different types)
'' == false       // true
'' === false      // false
null == undefined // true
null === undefined // false
NaN == NaN        // false
NaN === NaN       // false
```

Always use `===` unless you intentionally need coercion (e.g., `x == null` to check for both `null` and `undefined`).

---

## 8. What is `this` in JavaScript?

`this` refers to the context in which a function is called. Its value depends on how the function is invoked:

| Call Pattern | `this` Value |
|---|---|
| Global scope | `window` (browser) / `globalThis` |
| Object method `obj.fn()` | `obj` |
| Standalone function `fn()` | `window` (sloppy mode) / `undefined` (strict mode) |
| `new` constructor | New instance |
| `call` / `apply` / `bind` | Explicitly set |
| Arrow function | Inherited from enclosing lexical scope |
| Event handler | DOM element that received the event |

```js
const obj = {
  name: 'Alice',
  greet() { console.log(this.name); },
  greetArrow: () => { console.log(this.name); },
};

obj.greet();      // 'Alice' — method call
obj.greetArrow(); // undefined — arrow inherits outer `this`

const fn = obj.greet;
fn();             // undefined — standalone call (strict mode)
```

---

## 9. What are arrow functions and how are they different?

Arrow functions (`=>`) are a concise syntax for writing functions with key differences:

| Feature | Regular Function | Arrow Function |
|---|---|---|
| `this` binding | Dynamic (based on call site) | Lexical (from enclosing scope) |
| `arguments` object | Has own | No own (inherits from outer) |
| `new` keyword | Can be used as constructor | Cannot be used as constructor |
| `prototype` property | Has one | Does not have one |
| Implicit return | No | Yes (for single expressions) |

```js
// Concise syntax
const add = (a, b) => a + b;
const square = x => x * x;
const getObj = () => ({ key: 'value' }); // wrap object in parens

// Lexical this
function Timer() {
  this.seconds = 0;
  setInterval(() => {
    this.seconds++; // `this` refers to Timer instance
  }, 1000);
}
```

---

## 10. What is prototypal inheritance?

JavaScript uses prototypal inheritance — objects can inherit properties and methods from other objects via the prototype chain.

```js
const animal = {
  speak() { console.log(`${this.name} makes a sound`); }
};

const dog = Object.create(animal);
dog.name = 'Rex';
dog.bark = function() { console.log('Woof!'); };

dog.speak(); // 'Rex makes a sound' — inherited from animal
dog.bark();  // 'Woof!'
```

Every object has an internal `[[Prototype]]` link. When a property is not found on the object, JavaScript looks up the prototype chain until it reaches `null`.

```js
dog.__proto__ === animal          // true
animal.__proto__ === Object.prototype // true
Object.prototype.__proto__ === null   // true
```

---

## 11. What are Promises?

A Promise represents the eventual result of an asynchronous operation. It has three states:

- **Pending** — initial state
- **Fulfilled** — operation completed successfully
- **Rejected** — operation failed

```js
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      if (success) resolve({ data: 'Hello' });
      else reject(new Error('Failed'));
    }, 1000);
  });
};

fetchData()
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));
```

**Promise combinators:**
- `Promise.all([...])` — resolves when all resolve, rejects if any rejects
- `Promise.allSettled([...])` — resolves when all settle (regardless of outcome)
- `Promise.race([...])` — resolves/rejects with the first settled promise
- `Promise.any([...])` — resolves with first fulfilled, rejects if all reject

---

## 12. What is async/await?

`async/await` is syntactic sugar over Promises that makes asynchronous code look synchronous.

```js
async function fetchUsers() {
  try {
    const response = await fetch('/api/users');
    const users = await response.json();
    return users;
  } catch (error) {
    console.error('Failed:', error);
    throw error;
  }
}
```

Key rules:
- `await` can only be used inside `async` functions (or at top level in ES modules)
- `async` functions always return a Promise
- `await` pauses execution until the Promise settles
- Errors can be caught with `try/catch`

**Sequential vs parallel:**

```js
// Sequential — each waits for the previous
const a = await fetchA();
const b = await fetchB();

// Parallel — both start immediately
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

---

## 13. What is the difference between `null` and `undefined`?

| Feature | `undefined` | `null` |
|---|---|---|
| Meaning | Variable declared but not assigned | Intentional absence of value |
| Type | `typeof undefined === 'undefined'` | `typeof null === 'object'` |
| Default | Default value for uninitialized variables, missing function params, missing object properties | Must be explicitly assigned |
| Equality | `null == undefined` is `true` | `null === undefined` is `false` |

```js
let x;          // undefined
let y = null;   // null

function greet(name) {
  console.log(name); // undefined if no argument passed
}

const obj = { a: 1 };
console.log(obj.b); // undefined
```

---

## 14. What is destructuring?

Destructuring allows extracting values from arrays or properties from objects into distinct variables.

```js
// Object destructuring
const { name, age, city = 'Unknown' } = { name: 'Alice', age: 30 };

// Renaming
const { name: userName } = { name: 'Alice' };

// Nested
const { address: { street } } = { address: { street: '123 Main St' } };

// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

// Skipping values
const [, , third] = [1, 2, 3]; // third=3

// Swapping
let a = 1, b = 2;
[a, b] = [b, a]; // a=2, b=1

// Function parameters
function greet({ name, age }) {
  console.log(`${name} is ${age}`);
}
```

---

## 15. What is the spread operator and rest parameter?

**Spread (`...`)** — expands an iterable into individual elements:

```js
// Arrays
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4]; // [1, 2, 3, 4]

// Objects (shallow copy)
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }

// Function arguments
Math.max(...[1, 5, 3]); // 5
```

**Rest (`...`)** — collects remaining elements into an array:

```js
// Function parameters
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3); // 6

// Destructuring
const { a, ...rest } = { a: 1, b: 2, c: 3 };
// a=1, rest={ b: 2, c: 3 }
```

---

## 16. What are higher-order functions?

A higher-order function either takes a function as an argument or returns a function.

```js
// Takes a function as argument
const numbers = [1, 2, 3, 4, 5];
numbers.map(n => n * 2);       // [2, 4, 6, 8, 10]
numbers.filter(n => n > 3);    // [4, 5]
numbers.reduce((sum, n) => sum + n, 0); // 15

// Returns a function
function multiply(factor) {
  return (number) => number * factor;
}
const double = multiply(2);
double(5); // 10
```

Common built-in higher-order functions: `map`, `filter`, `reduce`, `forEach`, `find`, `some`, `every`, `sort`, `flatMap`.

---

## 17. What is currying?

Currying transforms a function with multiple arguments into a sequence of functions, each taking a single argument.

```js
// Regular function
const add = (a, b, c) => a + b + c;
add(1, 2, 3); // 6

// Curried version
const curriedAdd = (a) => (b) => (c) => a + b + c;
curriedAdd(1)(2)(3); // 6

// Partial application
const add5 = curriedAdd(5);
const add5and3 = add5(3);
add5and3(2); // 10

// Generic curry utility
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}
```

---

## 18. What is debouncing and throttling?

**Debouncing** — delays execution until a pause in events. The function runs only after the caller stops triggering for a specified duration.

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Use case: search input
input.addEventListener('input', debounce(handleSearch, 300));
```

**Throttling** — limits execution to at most once per interval, regardless of how many times the event fires.

```js
function throttle(fn, interval) {
  let lastTime = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}

// Use case: scroll handler
window.addEventListener('scroll', throttle(handleScroll, 100));
```

---

## 19. What is the difference between shallow copy and deep copy?

**Shallow copy** — copies the top-level properties. Nested objects are still references.

```js
const original = { a: 1, b: { c: 2 } };

// Shallow copy methods
const copy1 = { ...original };
const copy2 = Object.assign({}, original);

copy1.b.c = 99;
console.log(original.b.c); // 99 — nested object is shared
```

**Deep copy** — recursively copies all levels. No shared references.

```js
// structuredClone (modern, built-in)
const deep = structuredClone(original);

// JSON trick (doesn't handle functions, undefined, Date, RegExp, etc.)
const deep2 = JSON.parse(JSON.stringify(original));
```

---

## 20. What is the difference between `call`, `apply`, and `bind`?

All three set the `this` context for a function.

```js
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: 'Alice' };

// call — invokes immediately, args passed individually
greet.call(user, 'Hello', '!');    // "Hello, Alice!"

// apply — invokes immediately, args passed as array
greet.apply(user, ['Hello', '!']); // "Hello, Alice!"

// bind — returns a new function with `this` bound, does NOT invoke
const bound = greet.bind(user, 'Hello');
bound('!');                         // "Hello, Alice!"
```

---

## 21. What is event delegation?

Event delegation leverages event bubbling by attaching a single event listener to a parent element instead of individual listeners on each child.

```js
// Instead of adding listeners to every <li>
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log('Clicked:', e.target.textContent);
  }
});
```

Benefits:
- Better memory efficiency (fewer listeners)
- Automatically works for dynamically added elements
- Simpler cleanup

---

## 22. What is event bubbling and capturing?

When an event occurs on a DOM element, it goes through three phases:

1. **Capturing phase** — event travels from `window` down to the target element
2. **Target phase** — event reaches the target element
3. **Bubbling phase** — event travels back up from target to `window`

```js
// Bubbling (default)
element.addEventListener('click', handler);

// Capturing
element.addEventListener('click', handler, true);
// or
element.addEventListener('click', handler, { capture: true });

// Stop propagation
event.stopPropagation();          // stops further propagation
event.stopImmediatePropagation(); // stops all handlers on current element too
```

---

## 23. What are generators?

Generators are functions that can pause and resume execution using `yield`. They return an iterator.

```js
function* idGenerator() {
  let id = 0;
  while (true) {
    yield ++id;
  }
}

const gen = idGenerator();
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }
gen.next(); // { value: 3, done: false }

// Iterable
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

for (const num of range(1, 5)) {
  console.log(num); // 1, 2, 3, 4, 5
}

// Async generators
async function* fetchPages(url) {
  let page = 1;
  while (true) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();
    if (data.length === 0) return;
    yield data;
    page++;
  }
}
```

---

## 24. What are Symbols?

Symbols are unique, immutable primitive values used as object property keys to avoid naming collisions.

```js
const id = Symbol('id');
const user = {
  name: 'Alice',
  [id]: 123,
};

console.log(user[id]);       // 123
console.log(user.id);        // undefined
console.log(Object.keys(user)); // ['name'] — Symbols are not enumerable

// Well-known Symbols
Symbol.iterator    // defines iteration behavior
Symbol.toPrimitive // defines type conversion
Symbol.hasInstance  // customizes instanceof

// Global Symbol registry
const s1 = Symbol.for('app.id'); // creates/retrieves from global registry
const s2 = Symbol.for('app.id');
s1 === s2; // true
```

---

## 25. What are WeakMap and WeakSet?

**WeakMap** — a map where keys must be objects and are weakly referenced (garbage collected if no other references exist).

```js
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveComputation(obj);
  cache.set(obj, result);
  return result;
}

// When obj is garbage collected, the cache entry is automatically removed
```

**WeakSet** — a set where values must be objects and are weakly referenced.

```js
const visited = new WeakSet();

function track(obj) {
  if (visited.has(obj)) return;
  visited.add(obj);
  // process obj
}
```

Key differences from Map/Set:
- Keys/values must be objects (not primitives)
- Not iterable (no `forEach`, `keys`, `values`, `entries`)
- No `size` property
- Entries are garbage-collectable

---

## 26. What is the difference between `Map` and a plain object?

| Feature | Object | Map |
|---|---|---|
| Key types | Strings and Symbols only | Any value (objects, functions, primitives) |
| Key order | Not fully guaranteed | Insertion order guaranteed |
| Size | Manual (`Object.keys(obj).length`) | `map.size` |
| Iteration | `for...in`, `Object.entries()` | `for...of`, `forEach` |
| Performance | Slower for frequent add/delete | Optimized for frequent add/delete |
| Prototype keys | Has inherited keys | No inherited keys |
| Serialization | Native JSON support | Must convert manually |

```js
const map = new Map();
map.set('key', 'value');
map.set(42, 'number key');
map.set(objRef, 'object key');
map.get('key');     // 'value'
map.has(42);        // true
map.delete(42);
map.size;           // 2
```

---

## 27. What are iterators and iterables?

An **iterable** is an object that implements `Symbol.iterator`, which returns an **iterator**. An iterator has a `next()` method that returns `{ value, done }`.

```js
// Custom iterable
const range = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { done: true };
      }
    };
  }
};

for (const num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}

// Spread also works on iterables
[...range]; // [1, 2, 3, 4, 5]
```

Built-in iterables: `Array`, `String`, `Map`, `Set`, `TypedArray`, `arguments`, `NodeList`.

---

## 28. What is the module system in JavaScript?

**ES Modules (ESM)** — the standard module system:

```js
// Named export/import
export const PI = 3.14;
export function add(a, b) { return a + b; }

import { PI, add } from './math.js';

// Default export/import
export default class User { }

import User from './User.js';

// Re-export
export { default as User } from './User.js';
export * from './utils.js';

// Dynamic import (code splitting)
const module = await import('./heavy-module.js');
```

**CommonJS (CJS)** — Node.js module system:

```js
// Export
module.exports = { add, subtract };
// or
exports.add = function(a, b) { return a + b; };

// Import
const { add } = require('./math');
```

Key differences:
- ESM is statically analyzed, CJS is dynamic
- ESM has `import/export`, CJS has `require/module.exports`
- ESM supports tree shaking, CJS does not
- ESM is asynchronous, CJS is synchronous

---

## 29. What is the difference between `for...in` and `for...of`?

```js
const arr = ['a', 'b', 'c'];

// for...in — iterates over enumerable PROPERTY KEYS (indices for arrays)
for (const key in arr) {
  console.log(key); // '0', '1', '2' (strings!)
}

// for...of — iterates over iterable VALUES
for (const value of arr) {
  console.log(value); // 'a', 'b', 'c'
}

const obj = { x: 1, y: 2 };

// for...in works with objects
for (const key in obj) {
  console.log(key); // 'x', 'y'
}

// for...of does NOT work with plain objects (not iterable)
for (const val of obj) { } // TypeError
// Use Object.entries() instead:
for (const [key, val] of Object.entries(obj)) {
  console.log(key, val);
}
```

Use `for...of` for arrays and iterables. Use `for...in` for object keys (with `hasOwnProperty` check).

---

## 30. What is optional chaining (`?.`)?

Optional chaining short-circuits to `undefined` if a reference is `null` or `undefined`.

```js
const user = { address: { street: '123 Main' } };

// Without optional chaining
const zip = user && user.address && user.address.zip;

// With optional chaining
const zip = user?.address?.zip; // undefined (no error)

// Method calls
user.getProfile?.(); // undefined if method doesn't exist

// Array access
const first = arr?.[0];

// With nullish coalescing
const city = user?.address?.city ?? 'Unknown';
```

---

## 31. What is nullish coalescing (`??`)?

`??` returns the right operand when the left is `null` or `undefined` (not other falsy values).

```js
// ?? vs ||
0 || 'default'     // 'default' (0 is falsy)
0 ?? 'default'     // 0 (0 is not null/undefined)

'' || 'default'    // 'default' ('' is falsy)
'' ?? 'default'    // '' ('' is not null/undefined)

false || 'default' // 'default'
false ?? 'default' // false

null ?? 'default'  // 'default'
undefined ?? 'default' // 'default'
```

Use `??` when `0`, `''`, or `false` are valid values.

---

## 32. What is the difference between `Object.freeze()`, `Object.seal()`, and `Object.preventExtensions()`?

| Method | Add properties | Delete properties | Modify values |
|---|---|---|---|
| `Object.preventExtensions()` | No | Yes | Yes |
| `Object.seal()` | No | No | Yes |
| `Object.freeze()` | No | No | No |

```js
const obj = { a: 1, b: 2 };

Object.freeze(obj);
obj.a = 99;     // Silently fails (or throws in strict mode)
obj.c = 3;      // Silently fails
delete obj.a;   // Silently fails
console.log(obj); // { a: 1, b: 2 }
```

All three are **shallow** — nested objects are not affected. For deep freeze, you need to recursively freeze nested objects.

---

## 33. What are template literals?

Template literals use backticks and support string interpolation, multi-line strings, and tagged templates.

```js
const name = 'Alice';
const age = 30;

// String interpolation
const msg = `Hello, ${name}! You are ${age} years old.`;

// Multi-line
const html = `
  <div>
    <p>${msg}</p>
  </div>
`;

// Expressions
`Total: ${price * quantity}`;
`Status: ${isActive ? 'Active' : 'Inactive'}`;

// Tagged templates
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) =>
    `${result}${str}<b>${values[i] || ''}</b>`, ''
  );
}

highlight`Hello ${name}, age ${age}`;
// "Hello <b>Alice</b>, age <b>30</b>"
```

---

## 34. What is the Proxy object?

`Proxy` wraps an object and lets you intercept and customize fundamental operations (property access, assignment, function calls, etc.).

```js
const handler = {
  get(target, prop) {
    return prop in target ? target[prop] : `Property '${prop}' not found`;
  },
  set(target, prop, value) {
    if (typeof value !== 'number') throw TypeError('Must be a number');
    target[prop] = value;
    return true;
  },
};

const obj = new Proxy({}, handler);
obj.x = 42;         // OK
obj.y = 'hello';    // TypeError: Must be a number
console.log(obj.z); // "Property 'z' not found"
```

Use cases: validation, logging, default values, data binding, access control.

---

## 35. What is the Reflect API?

`Reflect` provides methods for interceptable JavaScript operations (same as Proxy handler methods). It makes working with Proxy handlers cleaner.

```js
const handler = {
  get(target, prop, receiver) {
    console.log(`Accessing ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`Setting ${prop} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
};
```

Key `Reflect` methods: `get`, `set`, `has`, `deleteProperty`, `ownKeys`, `apply`, `construct`.

---

## 36. What is memoization?

Memoization is an optimization technique that caches function results based on their arguments to avoid redundant computations.

```js
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const factorial = memoize(function f(n) {
  return n <= 1 ? 1 : n * f(n - 1);
});

factorial(5); // computed
factorial(5); // cached
```

---

## 37. What is the difference between `setTimeout` and `setInterval`?

```js
// setTimeout — runs once after delay
const timerId = setTimeout(() => {
  console.log('Runs once after 1 second');
}, 1000);
clearTimeout(timerId);

// setInterval — runs repeatedly at interval
const intervalId = setInterval(() => {
  console.log('Runs every 1 second');
}, 1000);
clearInterval(intervalId);
```

**Problem with `setInterval`:** If the callback takes longer than the interval, calls can stack up. Recursive `setTimeout` is safer for consistent spacing:

```js
function poll() {
  doWork().then(() => {
    setTimeout(poll, 1000); // next call after work completes
  });
}
```

---

## 38. What are classes in JavaScript?

Classes (ES6) are syntactic sugar over prototypal inheritance.

```js
class Animal {
  // Private field
  #sound;

  // Static property
  static kingdom = 'Animalia';

  constructor(name, sound) {
    this.name = name;
    this.#sound = sound;
  }

  // Instance method
  speak() {
    console.log(`${this.name} says ${this.#sound}`);
  }

  // Getter
  get info() {
    return `${this.name} (${Animal.kingdom})`;
  }

  // Static method
  static create(name, sound) {
    return new Animal(name, sound);
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name, 'Woof');
  }

  fetch(item) {
    console.log(`${this.name} fetches ${item}`);
  }
}

const dog = new Dog('Rex');
dog.speak();  // "Rex says Woof"
dog.fetch('ball'); // "Rex fetches ball"
```

---

## 39. What are private class fields?

Private fields (prefixed with `#`) are truly private — accessible only within the class body.

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    if (amount <= 0) throw new Error('Invalid amount');
    this.#balance += amount;
  }

  get balance() {
    return this.#balance;
  }
}

const account = new BankAccount();
account.deposit(100);
account.balance;     // 100
account.#balance;    // SyntaxError: Private field
```

You can also have private methods (`#method()`) and private static fields (`static #field`).

---

## 40. What is the difference between `Object.keys()`, `Object.values()`, and `Object.entries()`?

```js
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj);    // ['a', 'b', 'c']
Object.values(obj);  // [1, 2, 3]
Object.entries(obj); // [['a', 1], ['b', 2], ['c', 3]]

// Common patterns
Object.entries(obj).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});

// Convert entries back to object
const doubled = Object.fromEntries(
  Object.entries(obj).map(([k, v]) => [k, v * 2])
);
// { a: 2, b: 4, c: 6 }
```

All three return only **own, enumerable** properties (not inherited or Symbol-keyed).

---

## 41. What is a pure function?

A pure function:
1. Always returns the same output for the same input (deterministic)
2. Has no side effects (doesn't modify external state)

```js
// Pure
function add(a, b) { return a + b; }
function toUpper(str) { return str.toUpperCase(); }

// Impure — depends on external state
let counter = 0;
function increment() { return ++counter; }

// Impure — modifies input (side effect)
function addItem(arr, item) { arr.push(item); return arr; }

// Pure version
function addItem(arr, item) { return [...arr, item]; }
```

Pure functions are easier to test, reason about, and cache.

---

## 42. What is immutability and how do you achieve it?

Immutability means data cannot be changed after creation. Instead, you create new copies with modifications.

```js
// Arrays
const arr = [1, 2, 3];
const added = [...arr, 4];           // [1, 2, 3, 4]
const removed = arr.filter(x => x !== 2); // [1, 3]
const updated = arr.map(x => x === 2 ? 99 : x); // [1, 99, 3]

// Objects
const user = { name: 'Alice', age: 30 };
const updated = { ...user, age: 31 };   // new object

// Nested objects
const state = { user: { name: 'Alice', address: { city: 'NYC' } } };
const newState = {
  ...state,
  user: {
    ...state.user,
    address: { ...state.user.address, city: 'LA' },
  },
};

// structuredClone for deep copy
const deep = structuredClone(state);
```

Libraries like Immer simplify immutable updates with a mutable-looking API.

---

## 43. What is the difference between `Array.from()` and `Array.of()`?

```js
// Array.from — creates array from iterable or array-like object
Array.from('hello');           // ['h', 'e', 'l', 'l', 'o']
Array.from({ length: 3 }, (_, i) => i); // [0, 1, 2]
Array.from(new Set([1, 2, 2, 3]));      // [1, 2, 3]
Array.from(document.querySelectorAll('div')); // NodeList to Array

// Array.of — creates array from arguments
Array.of(1, 2, 3);  // [1, 2, 3]
Array.of(5);         // [5] (unlike Array(5) which creates 5 empty slots)
```

---

## 44. What are `try`, `catch`, `finally`, and custom errors?

```js
try {
  const data = JSON.parse(input);
  if (!data.name) throw new Error('Name is required');
} catch (error) {
  if (error instanceof SyntaxError) {
    console.error('Invalid JSON');
  } else {
    console.error(error.message);
  }
} finally {
  // Always runs (cleanup)
  closeConnection();
}

// Custom errors
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

throw new ValidationError('email', 'Invalid email format');
```

---

## 45. What is `Object.defineProperty()`?

`Object.defineProperty` adds or modifies a property with fine-grained control over its behavior.

```js
const obj = {};

Object.defineProperty(obj, 'name', {
  value: 'Alice',
  writable: false,      // cannot change value
  enumerable: true,     // shows up in for...in
  configurable: false,  // cannot delete or reconfigure
});

obj.name = 'Bob'; // silently fails (or throws in strict mode)

// Getter/setter
Object.defineProperty(obj, 'fullName', {
  get() { return `${this.first} ${this.last}`; },
  set(val) {
    [this.first, this.last] = val.split(' ');
  },
  enumerable: true,
  configurable: true,
});
```

---

## 46. What is `requestAnimationFrame`?

`requestAnimationFrame` schedules a callback to run before the next browser repaint (~60fps, every ~16ms).

```js
function animate() {
  element.style.left = `${position++}px`;

  if (position < 300) {
    requestAnimationFrame(animate);
  }
}

const animId = requestAnimationFrame(animate);
cancelAnimationFrame(animId); // cancel if needed
```

Advantages over `setInterval`:
- Synchronized with browser refresh rate
- Pauses when tab is inactive (saves battery)
- Smoother animations

---

## 47. What is the difference between `slice`, `splice`, and `split`?

```js
// slice(start, end) — returns new array, does NOT modify original
const arr = [1, 2, 3, 4, 5];
arr.slice(1, 3);  // [2, 3]
arr.slice(-2);    // [4, 5]
// arr is still [1, 2, 3, 4, 5]

// splice(start, deleteCount, ...items) — MODIFIES original array
const arr2 = [1, 2, 3, 4, 5];
arr2.splice(1, 2);        // returns [2, 3], arr2 is [1, 4, 5]
arr2.splice(1, 0, 'a');   // inserts 'a' at index 1, arr2 is [1, 'a', 4, 5]

// split(separator) — string method, splits into array
'hello world'.split(' '); // ['hello', 'world']
'a,b,c'.split(',');       // ['a', 'b', 'c']
'hello'.split('');        // ['h', 'e', 'l', 'l', 'o']
```

---

## 48. What are labeled statements and when are they useful?

Labels name a statement for use with `break` or `continue` in nested loops.

```js
outer: for (let i = 0; i < 3; i++) {
  inner: for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outer; // exits both loops
    console.log(i, j);
  }
}
// 0 0, 0 1, 0 2, 1 0
```

Rarely used in practice. Usually indicates code should be refactored into functions.

---

## 49. What are common JavaScript design patterns?

**Module Pattern:**
```js
const counter = (() => {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count,
  };
})();
```

**Singleton:**
```js
class Database {
  static #instance;
  static getInstance() {
    if (!Database.#instance) Database.#instance = new Database();
    return Database.#instance;
  }
}
```

**Observer:**
```js
class EventEmitter {
  #listeners = {};
  on(event, fn) { (this.#listeners[event] ??= []).push(fn); }
  emit(event, ...args) { this.#listeners[event]?.forEach(fn => fn(...args)); }
  off(event, fn) {
    this.#listeners[event] = this.#listeners[event]?.filter(f => f !== fn);
  }
}
```

**Factory:**
```js
function createUser(type) {
  switch (type) {
    case 'admin': return new AdminUser();
    case 'guest': return new GuestUser();
    default: return new RegularUser();
  }
}
```

---

## 50. What are common JavaScript coding challenges?

**Reverse a string:**
```js
const reverse = str => [...str].reverse().join('');
```

**Check palindrome:**
```js
const isPalindrome = str => {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === [...cleaned].reverse().join('');
};
```

**Flatten an array:**
```js
const flatten = arr => arr.flat(Infinity);
// or recursively
const flatten = arr =>
  arr.reduce((acc, val) =>
    acc.concat(Array.isArray(val) ? flatten(val) : val), []);
```

**Remove duplicates:**
```js
const unique = arr => [...new Set(arr)];
```

**Find missing number (1 to n):**
```js
const findMissing = arr => {
  const n = arr.length + 1;
  const expectedSum = (n * (n + 1)) / 2;
  const actualSum = arr.reduce((a, b) => a + b, 0);
  return expectedSum - actualSum;
};
```

**Debounce implementation:**
```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

**Deep equality check:**
```js
function deepEqual(a, b) {
  if (a === b) return true;
  if (typeof a !== typeof b || a == null || b == null) return false;
  if (typeof a !== 'object') return false;
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  return keysA.every(key => deepEqual(a[key], b[key]));
}
```
