# JavaScript — Real Company Interview Questions

> Questions asked in actual JavaScript interviews at real companies.

---

## 1. What is `this` in JavaScript?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

`this` is a keyword that refers to the object that is currently executing the code. Its value is determined by *how* a function is called, not where it is defined.

**Rules (in order of precedence):**

| Call style | `this` value |
|---|---|
| `new Foo()` | The newly created object |
| `foo.call(obj)` / `foo.apply(obj)` / `foo.bind(obj)` | `obj` (explicit binding) |
| `obj.foo()` | `obj` (implicit binding) |
| `foo()` | `undefined` in strict mode, `window` in non-strict |
| Arrow function | Inherits `this` from enclosing lexical scope (no own `this`) |

```js
const obj = {
  name: 'Alice',
  greet() {
    console.log(this.name); // 'Alice' — implicit binding
  },
  greetLater() {
    setTimeout(function () {
      console.log(this.name); // undefined — `this` is window/undefined
    }, 100);

    setTimeout(() => {
      console.log(this.name); // 'Alice' — arrow function inherits `this`
    }, 100);
  }
};
```

In React class components, this is why you had to `.bind(this)` event handlers or use arrow functions — otherwise `this` would be `undefined` when the handler was called.

---

## 2. Does JavaScript have inheritance?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

Yes, but JavaScript uses **prototypal inheritance**, not classical inheritance like Java or C++.

Every JavaScript object has an internal `[[Prototype]]` link (accessible via `__proto__` or `Object.getPrototypeOf()`) pointing to another object. When you access a property that doesn't exist on the object itself, JavaScript walks up the prototype chain until it finds it or reaches `null`.

```js
const animal = {
  speak() {
    return `${this.name} makes a sound`;
  }
};

const dog = Object.create(animal);
dog.name = 'Rex';
dog.speak(); // "Rex makes a sound" — found via prototype chain
```

ES6 `class` syntax provides a cleaner way to set up prototypal inheritance, but it's syntactic sugar — under the hood, it still uses prototypes.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  bark() {
    return `${this.name} barks`;
  }
}

const d = new Dog('Rex');
d.speak(); // "Rex makes a sound" — inherited
d.bark();  // "Rex barks" — own method
```

---

## 3. Is JavaScript object-oriented?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

Yes, JavaScript is object-oriented, but it is **multi-paradigm**. It supports:

- **Object-oriented programming** — via prototypal inheritance, classes (ES6 sugar), encapsulation (closures, private fields with `#`)
- **Functional programming** — first-class functions, closures, higher-order functions, array methods (`map`, `filter`, `reduce`)
- **Event-driven / asynchronous** — callbacks, promises, async/await

JavaScript's OOP differs from classical OOP languages:

| Feature | Classical (Java/C++) | JavaScript |
|---|---|---|
| Inheritance | Class-based | Prototype-based |
| Classes | Blueprints that create instances | Syntactic sugar over prototypes |
| Encapsulation | `private`, `protected`, `public` | Closures, `#` private fields (ES2022) |
| Polymorphism | Method overriding, interfaces | Duck typing, method overriding |
| Abstract classes | Supported | Not natively (can be simulated) |

---

## 4. How will you show inheritance using objects in JavaScript?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

There are several ways to implement inheritance using objects:

**1. `Object.create()` — direct prototypal inheritance:**

```js
const vehicle = {
  start() {
    return `${this.type} is starting`;
  }
};

const car = Object.create(vehicle);
car.type = 'Car';
car.drive = function () {
  return `${this.type} is driving`;
};

car.start(); // "Car is starting" — inherited from vehicle
car.drive(); // "Car is driving" — own method
```

**2. Constructor functions (pre-ES6):**

```js
function Vehicle(type) {
  this.type = type;
}
Vehicle.prototype.start = function () {
  return `${this.type} is starting`;
};

function Car(type) {
  Vehicle.call(this, type); // call parent constructor
}
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

const c = new Car('Sedan');
c.start(); // "Sedan is starting"
```

**3. ES6 classes (syntactic sugar):**

```js
class Vehicle {
  constructor(type) {
    this.type = type;
  }
  start() {
    return `${this.type} is starting`;
  }
}

class Car extends Vehicle {
  drive() {
    return `${this.type} is driving`;
  }
}

const c = new Car('SUV');
c.start(); // "SUV is starting"
c.drive(); // "SUV is driving"
```

All three approaches work via the prototype chain. `Object.create()` is the most direct demonstration of prototypal inheritance.

---

## 5. Name some JavaScript engines.
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

| Engine | Developed by | Used in |
|---|---|---|
| **V8** | Google | Chrome, Chromium-based browsers (Edge, Opera, Brave), Node.js, Deno |
| **SpiderMonkey** | Mozilla | Firefox |
| **JavaScriptCore (Nitro)** | Apple | Safari, Bun |
| **Chakra** | Microsoft | Legacy Edge (pre-Chromium) |
| **Hermes** | Meta | React Native |

All modern engines use JIT (Just-In-Time) compilation to convert JavaScript into optimized machine code at runtime, rather than interpreting it line by line.

---

## 6. Do you know about V8 and SpiderMonkey?
**Company:** Refyne
**Role:** Senior Frontend Engineer

**Answer:**

**V8 (Google):**
- Open-source, written in C++
- Powers Chrome, Node.js, Deno, and all Chromium-based browsers
- Compiles JavaScript directly to machine code (no bytecode interpreter initially, though it now has Ignition + TurboFan)
- **Pipeline:** Source → Parser → AST → Ignition (bytecode interpreter) → TurboFan (optimizing JIT compiler)
- Uses hidden classes and inline caching for fast property access
- Manages memory with a generational garbage collector (Orinoco)

**SpiderMonkey (Mozilla):**
- The first-ever JavaScript engine, created by Brendan Eich in 1995
- Open-source, written in C/C++ and Rust
- Powers Firefox
- **Pipeline:** Source → Parser → AST → Bytecode → Baseline (quick JIT) → Warp/IonMonkey (optimizing JIT)
- Has a multi-tier compilation strategy — code starts interpreted, then gets compiled at increasing optimization levels based on how often it runs

**Key differences:**

| Aspect | V8 | SpiderMonkey |
|---|---|---|
| JIT tiers | 2 (Ignition → TurboFan) | 3 (Interpreter → Baseline → Warp) |
| Initial strategy | Bytecode interpretation | Bytecode interpretation |
| Used in | Chrome, Node.js, Deno | Firefox |
| Wasm support | TurboFan-based | Cranelift-based |

---
