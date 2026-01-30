# TypeScript Interview Questions & Answers

---

## 1. What is TypeScript?

TypeScript is a statically typed superset of JavaScript developed by Microsoft. It adds optional type annotations, interfaces, generics, and other features that help catch errors at compile time rather than runtime. TypeScript compiles down to plain JavaScript.

```ts
// TypeScript
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Compiled JavaScript
function greet(name) {
  return `Hello, ${name}`;
}
```

---

## 2. What are the benefits of TypeScript over JavaScript?

- **Static type checking** — catch errors at compile time
- **Better IDE support** — autocompletion, refactoring, inline documentation
- **Self-documenting code** — types serve as documentation
- **Safer refactoring** — compiler catches breaking changes
- **Advanced OOP features** — interfaces, generics, enums, access modifiers
- **Gradual adoption** — valid JavaScript is valid TypeScript
- **Better tooling** — navigation, find references, rename across files

---

## 3. What are the basic types in TypeScript?

```ts
// Primitives
let name: string = 'Alice';
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;
let id: symbol = Symbol('id');
let big: bigint = 100n;

// Arrays
let nums: number[] = [1, 2, 3];
let names: Array<string> = ['Alice', 'Bob'];

// Tuple
let pair: [string, number] = ['Alice', 30];

// Enum
enum Direction { Up, Down, Left, Right }

// Any (opt out of type checking)
let value: any = 42;
value = 'hello'; // no error

// Unknown (type-safe any)
let input: unknown = 42;
// input.toFixed(); // Error — must narrow first
if (typeof input === 'number') {
  input.toFixed(); // OK
}

// Void (no return value)
function log(msg: string): void {
  console.log(msg);
}

// Never (function never returns)
function throwError(msg: string): never {
  throw new Error(msg);
}

// Object
let obj: object = { a: 1 };
let user: { name: string; age: number } = { name: 'Alice', age: 30 };
```

---

## 4. What is the difference between `any`, `unknown`, and `never`?

| Type | Description | Type-safe | Use when |
|---|---|---|---|
| `any` | Disables type checking entirely | No | Migrating JS, quick prototyping |
| `unknown` | Type-safe counterpart of `any` | Yes | Handling values of unknown type |
| `never` | Represents values that never occur | Yes | Exhaustive checks, functions that never return |

```ts
// any — no checks
let a: any = 'hello';
a.foo.bar; // no error (but will crash at runtime)

// unknown — must narrow before use
let b: unknown = 'hello';
// b.toUpperCase(); // Error
if (typeof b === 'string') {
  b.toUpperCase(); // OK
}

// never — exhaustive checking
type Shape = 'circle' | 'square';

function getArea(shape: Shape): number {
  switch (shape) {
    case 'circle': return Math.PI;
    case 'square': return 1;
    default:
      const _exhaustive: never = shape; // Error if a case is missing
      return _exhaustive;
  }
}
```

---

## 5. What is the difference between `interface` and `type`?

| Feature | `interface` | `type` |
|---|---|---|
| Object shapes | Yes | Yes |
| Extend/inherit | `extends` | `&` (intersection) |
| Declaration merging | Yes | No |
| Unions | No | Yes |
| Primitives/tuples/mapped | No | Yes |
| `implements` | Yes | Yes |

```ts
// Interface
interface User {
  name: string;
  age: number;
}

interface Admin extends User {
  role: string;
}

// Declaration merging
interface User {
  email: string; // merged with original User
}

// Type alias
type User = {
  name: string;
  age: number;
};

type Admin = User & { role: string }; // intersection

// Only type can do these:
type ID = string | number;              // union
type Pair = [string, number];           // tuple
type Callback = (data: string) => void; // function type
type Keys = keyof User;                 // mapped/computed types
```

**Rule of thumb:** Use `interface` for object shapes and public APIs (declaration merging is useful for libraries). Use `type` for unions, intersections, mapped types, and complex type expressions.

---

## 6. What are generics?

Generics allow writing reusable code that works with multiple types while maintaining type safety.

```ts
// Generic function
function identity<T>(value: T): T {
  return value;
}

identity<string>('hello'); // type is string
identity(42);              // type is inferred as number

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

const userResponse: ApiResponse<User> = {
  data: { name: 'Alice', age: 30 },
  status: 200,
  message: 'OK',
};

// Generic class
class Stack<T> {
  private items: T[] = [];

  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
  peek(): T | undefined { return this.items[this.items.length - 1]; }
}

const numStack = new Stack<number>();
numStack.push(1);
numStack.push(2);

// Generic constraints
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

getLength('hello');    // OK
getLength([1, 2, 3]); // OK
getLength(42);         // Error — number has no 'length'
```

---

## 7. What are generic constraints?

Generic constraints restrict what types can be used with a generic using `extends`.

```ts
// Constrain to objects with specific properties
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Alice', age: 30 };
getProperty(user, 'name'); // OK, returns string
getProperty(user, 'foo');  // Error — 'foo' is not in keyof user

// Multiple constraints
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}

// Constraint with interface
interface HasId {
  id: number;
}

function findById<T extends HasId>(items: T[], id: number): T | undefined {
  return items.find(item => item.id === id);
}
```

---

## 8. What are union and intersection types?

**Union (`|`)** — a value can be one of several types:

```ts
type ID = string | number;

function printId(id: ID) {
  if (typeof id === 'string') {
    console.log(id.toUpperCase()); // narrowed to string
  } else {
    console.log(id.toFixed(2));    // narrowed to number
  }
}

// Discriminated union
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
  | { kind: 'rectangle'; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'square': return shape.side ** 2;
    case 'rectangle': return shape.width * shape.height;
  }
}
```

**Intersection (`&`)** — a value must satisfy all types:

```ts
type HasName = { name: string };
type HasAge = { age: number };
type Person = HasName & HasAge;

const person: Person = { name: 'Alice', age: 30 }; // must have both
```

---

## 9. What are type guards?

Type guards are expressions that narrow a type within a conditional block.

```ts
// typeof guard
function process(value: string | number) {
  if (typeof value === 'string') {
    value.toUpperCase(); // narrowed to string
  }
}

// instanceof guard
function handleError(err: Error | string) {
  if (err instanceof Error) {
    console.log(err.message); // narrowed to Error
  }
}

// in operator
interface Dog { bark(): void; }
interface Cat { meow(): void; }

function speak(animal: Dog | Cat) {
  if ('bark' in animal) {
    animal.bark(); // narrowed to Dog
  }
}

// Custom type guard (type predicate)
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function process(value: unknown) {
  if (isString(value)) {
    value.toUpperCase(); // narrowed to string
  }
}

// Discriminated union
type Result =
  | { success: true; data: string }
  | { success: false; error: Error };

function handle(result: Result) {
  if (result.success) {
    console.log(result.data);  // narrowed to success branch
  } else {
    console.log(result.error); // narrowed to error branch
  }
}
```

---

## 10. What are mapped types?

Mapped types create new types by transforming properties of an existing type.

```ts
// Built-in mapped types
type User = { name: string; age: number; email: string };

// All properties optional
type PartialUser = Partial<User>;
// { name?: string; age?: number; email?: string }

// All properties required
type RequiredUser = Required<PartialUser>;

// All properties readonly
type ReadonlyUser = Readonly<User>;

// Pick specific properties
type UserPreview = Pick<User, 'name' | 'email'>;

// Omit specific properties
type UserWithoutEmail = Omit<User, 'email'>;

// Custom mapped type
type Nullable<T> = { [K in keyof T]: T[K] | null };
type NullableUser = Nullable<User>;
// { name: string | null; age: number | null; email: string | null }

// Conditional mapped type
type ReadonlyExceptName<T> = {
  readonly [K in keyof T as K extends 'name' ? never : K]: T[K];
} & { name: string };
```

---

## 11. What are conditional types?

Conditional types select one of two types based on a condition, using the syntax `T extends U ? X : Y`.

```ts
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Extract and Exclude (built-in)
type T1 = Extract<'a' | 'b' | 'c', 'a' | 'c'>; // 'a' | 'c'
type T2 = Exclude<'a' | 'b' | 'c', 'a'>;       // 'b' | 'c'

// Infer keyword — extract types within conditional types
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Fn = (x: number) => string;
type R = ReturnType<Fn>; // string

// Unwrap Promise
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;
type Result = Awaited<Promise<Promise<string>>>; // string

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;
type Arr = ToArray<string | number>; // string[] | number[]
```

---

## 12. What is the `keyof` operator?

`keyof` produces a union of all property names (keys) of a type.

```ts
interface User {
  name: string;
  age: number;
  email: string;
}

type UserKeys = keyof User; // 'name' | 'age' | 'email'

// Use with generics for type-safe property access
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = { name: 'Alice', age: 30, email: 'a@b.com' };
const name = getValue(user, 'name'); // type is string
const age = getValue(user, 'age');   // type is number
getValue(user, 'foo');               // Error
```

---

## 13. What is the `typeof` operator in TypeScript?

In TypeScript, `typeof` can be used in type context to get the type of a variable.

```ts
const config = {
  host: 'localhost',
  port: 3000,
  debug: true,
};

// typeof in type context — extracts the type from a value
type Config = typeof config;
// { host: string; port: number; debug: boolean }

// Useful with const assertions
const COLORS = ['red', 'green', 'blue'] as const;
type Color = typeof COLORS[number]; // 'red' | 'green' | 'blue'

// With functions
function createUser(name: string, age: number) {
  return { name, age, id: Math.random() };
}
type User = ReturnType<typeof createUser>;
// { name: string; age: number; id: number }
```

---

## 14. What are template literal types?

Template literal types build string types using template literal syntax.

```ts
type Color = 'red' | 'blue';
type Size = 'small' | 'large';

type ClassNames = `${Size}-${Color}`;
// 'small-red' | 'small-blue' | 'large-red' | 'large-blue'

// Built-in string manipulation types
type Upper = Uppercase<'hello'>;     // 'HELLO'
type Lower = Lowercase<'HELLO'>;     // 'hello'
type Cap = Capitalize<'hello'>;      // 'Hello'
type Uncap = Uncapitalize<'Hello'>;  // 'hello'

// Event handler pattern
type EventName = 'click' | 'focus' | 'blur';
type Handler = `on${Capitalize<EventName>}`;
// 'onClick' | 'onFocus' | 'onBlur'

// Getter pattern
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User { name: string; age: number; }
type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }
```

---

## 15. What are enums and what are their alternatives?

**Numeric enums:**
```ts
enum Direction {
  Up = 0,    // default starts at 0
  Down = 1,
  Left = 2,
  Right = 3,
}

Direction.Up;        // 0
Direction[0];        // 'Up' (reverse mapping)
```

**String enums:**
```ts
enum Status {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE',
  Pending = 'PENDING',
}
```

**Const enums (inlined at compile time):**
```ts
const enum Color {
  Red = 'RED',
  Blue = 'BLUE',
}
// Inlined: no runtime object generated
```

**Alternatives to enums (often preferred):**

```ts
// Union of string literals
type Direction = 'up' | 'down' | 'left' | 'right';

// Object with as const
const Direction = {
  Up: 'up',
  Down: 'down',
  Left: 'left',
  Right: 'right',
} as const;

type Direction = typeof Direction[keyof typeof Direction];
// 'up' | 'down' | 'left' | 'right'
```

String literal unions are simpler and don't add runtime code. `as const` objects give you both a runtime object and type safety.

---

## 16. What is `as const` (const assertion)?

`as const` makes the value deeply readonly and narrows types to their literal values.

```ts
// Without as const
const config = { port: 3000, host: 'localhost' };
// type: { port: number; host: string }

// With as const
const config = { port: 3000, host: 'localhost' } as const;
// type: { readonly port: 3000; readonly host: 'localhost' }

// Arrays
const colors = ['red', 'green', 'blue'] as const;
// type: readonly ['red', 'green', 'blue']

type Color = typeof colors[number]; // 'red' | 'green' | 'blue'

// Useful for discriminated unions
const actions = {
  INCREMENT: 'INCREMENT',
  DECREMENT: 'DECREMENT',
} as const;
```

---

## 17. What is type assertion?

Type assertions tell the compiler to treat a value as a specific type. They don't perform runtime checks or conversions.

```ts
// as syntax (preferred)
const input = document.getElementById('name') as HTMLInputElement;
input.value = 'Alice';

// Angle bracket syntax (not usable in JSX/TSX files)
const input = <HTMLInputElement>document.getElementById('name');

// Non-null assertion (!)
function process(value: string | null) {
  const length = value!.length; // assert value is not null
}

// Double assertion (escape hatch — use sparingly)
const value = 'hello' as unknown as number; // force incompatible types
```

Type assertions should be used carefully. They override the compiler's type checking and can lead to runtime errors if used incorrectly.

---

## 18. What are utility types?

TypeScript provides built-in utility types for common type transformations.

```ts
interface User {
  name: string;
  age: number;
  email: string;
  address: string;
}

// Partial — all properties optional
type PartialUser = Partial<User>;

// Required — all properties required
type RequiredUser = Required<Partial<User>>;

// Readonly — all properties readonly
type ReadonlyUser = Readonly<User>;

// Pick — select specific properties
type UserName = Pick<User, 'name' | 'email'>;

// Omit — remove specific properties
type UserWithoutEmail = Omit<User, 'email'>;

// Record — construct type with set of properties
type Roles = Record<'admin' | 'user' | 'guest', { permissions: string[] }>;

// Exclude — remove types from union
type T = Exclude<'a' | 'b' | 'c', 'a'>; // 'b' | 'c'

// Extract — keep types in union
type T = Extract<'a' | 'b' | 'c', 'a' | 'c'>; // 'a' | 'c'

// NonNullable — remove null and undefined
type T = NonNullable<string | null | undefined>; // string

// ReturnType — extract function return type
type T = ReturnType<() => string>; // string

// Parameters — extract function parameter types
type T = Parameters<(a: string, b: number) => void>; // [string, number]

// Awaited — unwrap Promise type
type T = Awaited<Promise<string>>; // string
```

---

## 19. What are decorators?

Decorators are special declarations that modify classes, methods, properties, or parameters. They are enabled with the `experimentalDecorators` flag (legacy) or available as stage 3 TC39 decorators (TypeScript 5.0+).

```ts
// Class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class User {
  name: string;
  constructor(name: string) { this.name = name; }
}

// Method decorator
function log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key} with`, args);
    return original.apply(this, args);
  };
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

// Property decorator
function minLength(length: number) {
  return function (target: any, key: string) {
    let value: string;
    Object.defineProperty(target, key, {
      get: () => value,
      set: (newVal: string) => {
        if (newVal.length < length) throw new Error(`${key} must be at least ${length} chars`);
        value = newVal;
      },
    });
  };
}
```

---

## 20. What is declaration merging?

Declaration merging is TypeScript's ability to merge multiple declarations with the same name into a single definition.

```ts
// Interface merging
interface User {
  name: string;
}

interface User {
  age: number;
}

// Merged result: { name: string; age: number }
const user: User = { name: 'Alice', age: 30 };

// Namespace merging with class
class Album {
  label: Album.AlbumLabel;
}

namespace Album {
  export interface AlbumLabel {
    name: string;
  }
}

// Module augmentation
declare module 'express' {
  interface Request {
    user?: { id: string; role: string };
  }
}
```

This is commonly used to extend third-party library types.

---

## 21. What are type predicates?

Type predicates are return types that narrow the type of a parameter in the calling scope.

```ts
// Type predicate with 'is'
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function process(value: unknown) {
  if (isString(value)) {
    // value is narrowed to string here
    console.log(value.toUpperCase());
  }
}

// With interfaces
interface Fish { swim(): void; }
interface Bird { fly(): void; }

function isFish(animal: Fish | Bird): animal is Fish {
  return (animal as Fish).swim !== undefined;
}

// Assertion functions (asserts)
function assertString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error('Expected string');
  }
}

function process(value: unknown) {
  assertString(value);
  // value is narrowed to string after this point
  console.log(value.toUpperCase());
}
```

---

## 22. What is the `infer` keyword?

`infer` is used within conditional types to declare a type variable that TypeScript infers.

```ts
// Extract return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type T1 = ReturnType<() => string>; // string

// Extract first argument type
type FirstArg<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;
type T2 = FirstArg<(name: string, age: number) => void>; // string

// Unwrap Promise
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;
type T3 = UnwrapPromise<Promise<number>>; // number

// Extract array element type
type ElementType<T> = T extends (infer E)[] ? E : T;
type T4 = ElementType<string[]>; // string

// Extract props from React component
type PropsOf<T> = T extends React.ComponentType<infer P> ? P : never;
```

---

## 23. What are index signatures and index access types?

**Index signatures** define the type for dynamic property keys:

```ts
// Index signature
interface StringMap {
  [key: string]: number;
}

const scores: StringMap = {
  alice: 95,
  bob: 87,
};

// With known and dynamic properties
interface Config {
  name: string;                  // required known property
  [key: string]: string | number; // must be compatible with known properties
}
```

**Index access types** look up a specific property type:

```ts
interface User {
  name: string;
  age: number;
  address: {
    street: string;
    city: string;
  };
}

type Name = User['name'];           // string
type Address = User['address'];     // { street: string; city: string }
type Street = User['address']['city']; // string

// With unions
type NameOrAge = User['name' | 'age']; // string | number

// With keyof
type UserValues = User[keyof User]; // string | number | { street: string; city: string }
```

---

## 24. What is the `satisfies` operator?

`satisfies` (TypeScript 4.9+) validates that a value conforms to a type without widening the inferred type.

```ts
type Colors = Record<string, [number, number, number] | string>;

// With type annotation — loses specific type info
const colors: Colors = {
  red: [255, 0, 0],
  green: '#00ff00',
};
colors.red.map(c => c); // Error — could be string | number[]

// With satisfies — validates AND preserves narrow types
const colors = {
  red: [255, 0, 0],
  green: '#00ff00',
} satisfies Colors;

colors.red.map(c => c);      // OK — TypeScript knows it's a tuple
colors.green.toUpperCase();   // OK — TypeScript knows it's a string

// Catches errors at the validation point
const colors = {
  red: [255, 0, 0],
  green: true,           // Error — boolean doesn't satisfy Colors
} satisfies Colors;
```

---

## 25. What is the difference between `type` assertions and `satisfies`?

| Feature | `as` (assertion) | `satisfies` |
|---|---|---|
| Direction | Overrides compiler's type | Validates against a type |
| Type narrowing | Widens or narrows | Preserves the narrowest type |
| Safety | Can hide errors | Catches errors at validation |
| Runtime | No effect | No effect |

```ts
type Config = { port: number; debug: boolean };

// as — overrides type, can hide errors
const config = { port: '3000', debug: true } as Config; // no error!

// satisfies — validates, catches errors
const config = { port: '3000', debug: true } satisfies Config; // Error: string not assignable to number
```

---

## 26. What are discriminated unions?

Discriminated unions are union types where each member has a common literal property (the discriminant) that TypeScript uses for narrowing.

```ts
type Result<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }
  | { status: 'loading' };

function handleResult(result: Result<User>) {
  switch (result.status) {
    case 'success':
      console.log(result.data);  // TypeScript knows data exists
      break;
    case 'error':
      console.log(result.error); // TypeScript knows error exists
      break;
    case 'loading':
      console.log('Loading...');
      break;
  }
}

// Exhaustive check with never
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

function handleResult(result: Result<User>) {
  switch (result.status) {
    case 'success': return result.data;
    case 'error': throw result.error;
    case 'loading': return null;
    default: return assertNever(result); // Error if a case is missing
  }
}
```

---

## 27. What is the `readonly` modifier?

`readonly` prevents property reassignment after initialization.

```ts
// Readonly properties
interface User {
  readonly id: number;
  name: string;
}

const user: User = { id: 1, name: 'Alice' };
user.name = 'Bob'; // OK
user.id = 2;       // Error

// Readonly arrays and tuples
const nums: readonly number[] = [1, 2, 3];
nums.push(4);  // Error
nums[0] = 99;  // Error

const pair: readonly [string, number] = ['Alice', 30];

// ReadonlyArray
const arr: ReadonlyArray<number> = [1, 2, 3];

// Readonly utility type (deep for one level)
type ReadonlyUser = Readonly<User>;

// Deep readonly (custom)
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

---

## 28. What are access modifiers in TypeScript?

```ts
class User {
  public name: string;         // accessible everywhere (default)
  protected email: string;     // accessible in class and subclasses
  private password: string;    // accessible only in this class
  readonly id: number;         // cannot be reassigned after construction

  constructor(name: string, email: string, password: string) {
    this.id = Math.random();
    this.name = name;
    this.email = email;
    this.password = password;
  }
}

class Admin extends User {
  getEmail() {
    return this.email;    // OK — protected
    // return this.password; // Error — private
  }
}

const user = new User('Alice', 'a@b.com', 'secret');
user.name;     // OK — public
// user.email;    // Error — protected
// user.password; // Error — private

// Parameter properties (shorthand)
class User {
  constructor(
    public name: string,
    protected email: string,
    private password: string,
    readonly id: number = Math.random()
  ) {}
}
```

Note: TypeScript's `private` is compile-time only. JavaScript's `#` prefix provides true runtime privacy.

---

## 29. What are abstract classes?

Abstract classes cannot be instantiated directly. They serve as base classes that define a contract for subclasses.

```ts
abstract class Shape {
  abstract area(): number;     // must be implemented by subclasses
  abstract perimeter(): number;

  // Concrete method — shared implementation
  describe(): string {
    return `Area: ${this.area()}, Perimeter: ${this.perimeter()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }

  perimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

// const shape = new Shape(); // Error — cannot instantiate abstract class
const circle = new Circle(5);
circle.describe(); // "Area: 78.54, Perimeter: 31.42"
```

---

## 30. What is the difference between `implements` and `extends`?

```ts
// extends — inherits implementation from a class
class Animal {
  move() { console.log('Moving'); }
}

class Dog extends Animal {
  bark() { console.log('Woof'); }
}

const dog = new Dog();
dog.move(); // inherited
dog.bark();

// implements — enforces a contract (no implementation inherited)
interface Serializable {
  serialize(): string;
}

interface Loggable {
  log(): void;
}

// Can implement multiple interfaces
class User implements Serializable, Loggable {
  constructor(public name: string) {}

  serialize(): string {        // must implement
    return JSON.stringify(this);
  }

  log(): void {                // must implement
    console.log(this.name);
  }
}
```

`extends` = "inherit behavior from a class" (single inheritance).
`implements` = "promise to match a shape" (multiple interfaces).

---

## 31. What are namespaces?

Namespaces organize code and prevent naming collisions. They are largely replaced by ES modules but still used in declaration files and some patterns.

```ts
namespace Validation {
  export interface Validator {
    validate(value: string): boolean;
  }

  export class EmailValidator implements Validator {
    validate(value: string): boolean {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
    }
  }

  // Not exported — internal to namespace
  const MAX_LENGTH = 255;
}

const validator = new Validation.EmailValidator();
validator.validate('test@example.com');
```

Prefer ES modules (`import`/`export`) over namespaces in modern TypeScript.

---

## 32. What are declaration files (`.d.ts`)?

Declaration files contain only type information (no implementation). They describe the shape of JavaScript libraries for TypeScript.

```ts
// math-lib.d.ts
declare module 'math-lib' {
  export function add(a: number, b: number): number;
  export function subtract(a: number, b: number): number;
  export const PI: number;
}

// Global declarations
declare const __DEV__: boolean;
declare function fetch(url: string): Promise<Response>;

// Ambient module declaration (for non-JS imports)
declare module '*.css' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.svg' {
  const content: React.FC<React.SVGProps<SVGSVGElement>>;
  export default content;
}
```

Type packages on npm follow the `@types/package-name` convention (e.g., `@types/react`).

---

## 33. What is the `Record` utility type?

`Record<Keys, Type>` constructs a type with a set of properties of type `Keys` with values of type `Type`.

```ts
// Basic usage
type UserRoles = Record<'admin' | 'user' | 'guest', { permissions: string[] }>;

const roles: UserRoles = {
  admin: { permissions: ['read', 'write', 'delete'] },
  user: { permissions: ['read', 'write'] },
  guest: { permissions: ['read'] },
};

// Dynamic keys
type StringMap = Record<string, number>;

// With enums
enum Status { Active, Inactive, Pending }
type StatusLabels = Record<Status, string>;

// Practical example — grouping
function groupBy<T>(items: T[], key: keyof T): Record<string, T[]> {
  return items.reduce((groups, item) => {
    const value = String(item[key]);
    return { ...groups, [value]: [...(groups[value] || []), item] };
  }, {} as Record<string, T[]>);
}
```

---

## 34. What is module augmentation?

Module augmentation allows extending types from external modules without modifying the original source.

```ts
// Extend Express Request
import 'express';

declare module 'express' {
  interface Request {
    user?: {
      id: string;
      role: string;
    };
  }
}

// Now you can use req.user in your code
app.get('/profile', (req, res) => {
  const userId = req.user?.id; // no error
});

// Extend a library's types
import 'axios';

declare module 'axios' {
  interface AxiosRequestConfig {
    retry?: number;
    retryDelay?: number;
  }
}
```

---

## 35. What is the `Omit` vs `Pick` difference?

Both create new types from existing ones by selecting properties.

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Pick — select specific properties to KEEP
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: number; name: string }

// Omit — select specific properties to REMOVE
type SafeUser = Omit<User, 'password'>;
// { id: number; name: string; email: string; createdAt: Date }

// Practical usage
type CreateUserInput = Omit<User, 'id' | 'createdAt'>;
type UpdateUserInput = Partial<Omit<User, 'id'>>;
```

Use `Pick` when keeping a few properties from a large type. Use `Omit` when removing a few properties from a large type.

---

## 36. What are variadic tuple types?

Variadic tuple types (TypeScript 4.0+) allow generic spreading in tuple types.

```ts
// Spread in tuple types
type Concat<A extends any[], B extends any[]> = [...A, ...B];
type T1 = Concat<[1, 2], [3, 4]>; // [1, 2, 3, 4]

// Prepend to tuple
type Prepend<T, Arr extends any[]> = [T, ...Arr];
type T2 = Prepend<0, [1, 2, 3]>; // [0, 1, 2, 3]

// Function composition types
type Head<T extends any[]> = T extends [infer First, ...any[]] ? First : never;
type Tail<T extends any[]> = T extends [any, ...infer Rest] ? Rest : never;

type H = Head<[1, 2, 3]>; // 1
type R = Tail<[1, 2, 3]>; // [2, 3]

// Strongly typed function arguments
function curry<A extends any[], B extends any[], R>(
  fn: (...args: [...A, ...B]) => R,
  ...headArgs: A
): (...tailArgs: B) => R {
  return (...tailArgs) => fn(...headArgs, ...tailArgs);
}
```

---

## 37. What are recursive types?

TypeScript supports recursive type definitions for representing nested data structures.

```ts
// JSON type
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

// Nested menu
type MenuItem = {
  label: string;
  href: string;
  children?: MenuItem[];
};

// Deep partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

// Flatten nested arrays type
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;
type T1 = Flatten<number[][][]>; // number

// File system
type FileSystem = {
  [name: string]: string | FileSystem;
};

const fs: FileSystem = {
  src: {
    'index.ts': 'content',
    utils: {
      'helpers.ts': 'content',
    },
  },
};
```

---

## 38. What is the `NoInfer` utility type?

`NoInfer` (TypeScript 5.4+) prevents TypeScript from inferring a type parameter from a specific position.

```ts
// Without NoInfer — TypeScript infers from both arguments
function createFSM<S extends string>(initial: S, states: S[]) { }
createFSM('idle', ['idle', 'loading', 'error']);
// S inferred as 'idle' | 'loading' | 'error' — works

// Problem: incorrect initial state is accepted
createFSM('idel', ['idle', 'loading', 'error']); // no error!
// S inferred as 'idle' | 'loading' | 'error' | 'idel'

// With NoInfer — inference blocked on second argument
function createFSM<S extends string>(initial: S, states: NoInfer<S>[]) { }
createFSM('idel', ['idle', 'loading', 'error']);
// Error: 'idel' is not assignable to 'idle' | 'loading' | 'error'
```

---

## 39. What is `strictNullChecks` and why is it important?

`strictNullChecks` is a compiler option that makes `null` and `undefined` distinct types instead of being assignable to any type.

```ts
// With strictNullChecks: false (default in older configs)
let name: string = null;      // OK (unsafe)
let age: number = undefined;  // OK (unsafe)

// With strictNullChecks: true (recommended)
let name: string = null;      // Error
let name: string | null = null; // OK — explicit

function getUser(id: number): User | null {
  // ...
}

const user = getUser(1);
// user.name; // Error — user might be null
if (user) {
  user.name; // OK — narrowed to User
}

// Optional chaining works well with strict null checks
user?.name;              // string | undefined
user?.name ?? 'Unknown'; // string
```

Always enable `strictNullChecks` (or the umbrella `strict` flag) for type safety.

---

## 40. What are the important `tsconfig.json` compiler options?

```json
{
  "compilerOptions": {
    // Strictness
    "strict": true,                    // enables all strict checks
    "noUncheckedIndexedAccess": true,  // array/object index returns T | undefined
    "exactOptionalPropertyTypes": true, // distinguish undefined from missing

    // Modules
    "module": "esnext",                // module system
    "moduleResolution": "bundler",     // how modules are resolved
    "esModuleInterop": true,           // CJS/ESM interop
    "resolveJsonModule": true,         // import JSON files

    // Output
    "target": "es2022",               // JS version target
    "outDir": "./dist",               // output directory
    "declaration": true,              // generate .d.ts files
    "sourceMap": true,                // generate source maps

    // Path aliases
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },

    // Checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    // JSX (React)
    "jsx": "react-jsx",

    // Type roots
    "typeRoots": ["./node_modules/@types", "./src/types"]
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 41. What is structural typing (duck typing)?

TypeScript uses structural typing — type compatibility is based on structure (shape), not name.

```ts
interface Point {
  x: number;
  y: number;
}

function logPoint(point: Point) {
  console.log(`${point.x}, ${point.y}`);
}

// This works even though it's not declared as Point
const obj = { x: 10, y: 20, z: 30 };
logPoint(obj); // OK — has x and y

// This also works
class Coordinate {
  constructor(public x: number, public y: number) {}
}

logPoint(new Coordinate(5, 10)); // OK — same shape as Point
```

This is different from nominal typing (used in Java/C#) where types must be explicitly declared.

---

## 42. What is the difference between `type` narrowing techniques?

```ts
type Shape = Circle | Square | Triangle;

// 1. typeof — for primitives
function process(value: string | number) {
  if (typeof value === 'string') { /* string */ }
}

// 2. instanceof — for classes
function handle(err: Error | TypeError) {
  if (err instanceof TypeError) { /* TypeError */ }
}

// 3. in operator — for property existence
function draw(shape: Circle | Square) {
  if ('radius' in shape) { /* Circle */ }
}

// 4. Discriminant property — for discriminated unions
function area(shape: Shape) {
  if (shape.kind === 'circle') { /* Circle */ }
}

// 5. Custom type guard — for complex logic
function isCircle(shape: Shape): shape is Circle {
  return shape.kind === 'circle';
}

// 6. Truthiness narrowing
function greet(name: string | null) {
  if (name) { /* string (non-empty) */ }
}

// 7. Equality narrowing
function compare(a: string | number, b: string | boolean) {
  if (a === b) { /* both are string */ }
}
```

---

## 43. What are function overloads?

Function overloads provide multiple type signatures for a single function implementation.

```ts
// Overload signatures
function createElement(tag: 'a'): HTMLAnchorElement;
function createElement(tag: 'canvas'): HTMLCanvasElement;
function createElement(tag: 'input'): HTMLInputElement;
function createElement(tag: string): HTMLElement;

// Implementation signature (not directly callable)
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}

const anchor = createElement('a');     // HTMLAnchorElement
const canvas = createElement('canvas'); // HTMLCanvasElement
const div = createElement('div');       // HTMLElement

// Alternative: conditional types (often preferred)
type ElementMap = {
  a: HTMLAnchorElement;
  canvas: HTMLCanvasElement;
  input: HTMLInputElement;
};

function createElement<T extends keyof ElementMap>(tag: T): ElementMap[T];
function createElement(tag: string): HTMLElement;
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}
```

---

## 44. What is the `using` keyword?

`using` (TypeScript 5.2+) implements the TC39 Explicit Resource Management proposal. Resources declared with `using` are automatically disposed when they go out of scope.

```ts
// Disposable interface
interface Disposable {
  [Symbol.dispose](): void;
}

// Creating a disposable resource
class DatabaseConnection implements Disposable {
  constructor() { console.log('Connected'); }
  query(sql: string) { /* ... */ }

  [Symbol.dispose]() {
    console.log('Disconnected'); // automatic cleanup
  }
}

// using declaration — auto-disposes at end of scope
{
  using conn = new DatabaseConnection();
  conn.query('SELECT * FROM users');
} // conn[Symbol.dispose]() called automatically here

// await using — for async disposal
class FileHandle implements AsyncDisposable {
  async [Symbol.asyncDispose]() {
    await this.close();
  }
}

{
  await using file = await openFile('data.txt');
  await file.write('hello');
} // file automatically closed
```

---

## 45. What is covariance and contravariance?

These describe how type relationships work with generics.

```ts
// Covariance (output position) — preserves type relationship
// If Dog extends Animal, then Array<Dog> extends Array<Animal>
type Animal = { name: string };
type Dog = { name: string; breed: string };

const dogs: Dog[] = [{ name: 'Rex', breed: 'Lab' }];
const animals: readonly Animal[] = dogs; // OK — covariant (readonly)

// Contravariance (input position) — reverses type relationship
// If Dog extends Animal, then (animal: Animal) => void extends (dog: Dog) => void
type AnimalHandler = (animal: Animal) => void;
type DogHandler = (dog: Dog) => void;

const handleAnimal: AnimalHandler = (a) => console.log(a.name);
const handleDog: DogHandler = handleAnimal; // OK — contravariant

// TypeScript 4.7+ — explicit variance annotations
interface Producer<out T> {  // covariant
  get(): T;
}

interface Consumer<in T> {   // contravariant
  accept(value: T): void;
}

interface Transform<in T, out U> { // contravariant input, covariant output
  transform(input: T): U;
}
```

---

## 46. What is the `extends` keyword used for in TypeScript?

`extends` has multiple uses:

```ts
// 1. Class inheritance
class Admin extends User { }

// 2. Interface inheritance
interface Admin extends User {
  role: string;
}

// 3. Generic constraints
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

// 4. Conditional types
type IsString<T> = T extends string ? true : false;

// 5. Intersection of interfaces (multiple extends)
interface AdminUser extends User, Serializable {
  permissions: string[];
}
```

---

## 47. What are branded/opaque types?

Branded types create distinct types from the same underlying type, preventing accidental mixing.

```ts
// Brand type utility
type Brand<T, B extends string> = T & { readonly __brand: B };

type USD = Brand<number, 'USD'>;
type EUR = Brand<number, 'EUR'>;
type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;

// Constructor functions
function usd(amount: number): USD { return amount as USD; }
function eur(amount: number): EUR { return amount as EUR; }

// Type-safe — can't mix currencies
function addUSD(a: USD, b: USD): USD {
  return (a + b) as USD;
}

const price = usd(10);
const tax = usd(2);
const euroPrice = eur(10);

addUSD(price, tax);       // OK
addUSD(price, euroPrice); // Error — EUR not assignable to USD

// Prevents passing wrong ID types
function getUser(id: UserId): User { /* ... */ }
function getPost(id: PostId): Post { /* ... */ }

const userId = 'abc' as UserId;
const postId = 'xyz' as PostId;

getUser(userId); // OK
getUser(postId); // Error — PostId not assignable to UserId
```

---

## 48. What is the difference between `Object`, `object`, and `{}`?

```ts
// Object (uppercase) — matches any non-null/undefined value
// (includes primitives — avoid using this)
let a: Object = 'hello'; // OK
let b: Object = 42;      // OK
let c: Object = true;    // OK

// object (lowercase) — matches any non-primitive value
let d: object = {};        // OK
let e: object = [];        // OK
let f: object = () => {};  // OK
let g: object = 42;        // Error — primitive

// {} (empty object type) — matches any non-null/undefined value
// Similar to Object but preferred
let h: {} = 'hello';    // OK
let i: {} = 42;          // OK
let j: {} = {};          // OK
let k: {} = null;        // Error

// Best practice — use specific types
let user: { name: string; age: number }; // explicit shape
let map: Record<string, unknown>;         // dynamic keys
```

---

## 49. What are common TypeScript mistakes?

1. **Using `any` excessively** — defeats the purpose of TypeScript; use `unknown` and narrow instead
2. **Not enabling `strict` mode** — misses many potential bugs
3. **Type assertions instead of proper typing** — `as` hides errors; prefer type guards
4. **Ignoring `null` and `undefined`** — always handle nullable types explicitly
5. **Overcomplicating types** — complex conditional types when a simple union would work
6. **Not using discriminated unions** — using type assertions instead of narrowing
7. **Using `enum` when string unions suffice** — adds unnecessary runtime code
8. **Mutating readonly types with assertions** — undermines type safety
9. **Not using `satisfies`** — using type annotations that widen inferred types
10. **Ignoring TypeScript errors** — using `@ts-ignore` instead of fixing the underlying issue

---

## 50. What are some advanced TypeScript patterns?

**Builder pattern with types:**
```ts
class QueryBuilder<T extends Record<string, unknown>> {
  private conditions: Partial<T> = {};

  where<K extends keyof T>(key: K, value: T[K]): this {
    this.conditions[key] = value;
    return this;
  }

  build(): Partial<T> {
    return { ...this.conditions };
  }
}
```

**Type-safe event emitter:**
```ts
type EventMap = {
  login: { userId: string };
  logout: { reason: string };
  error: { code: number; message: string };
};

class TypedEmitter<T extends Record<string, unknown>> {
  private handlers = new Map<keyof T, Set<Function>>();

  on<K extends keyof T>(event: K, handler: (payload: T[K]) => void): void {
    const set = this.handlers.get(event) ?? new Set();
    set.add(handler);
    this.handlers.set(event, set);
  }

  emit<K extends keyof T>(event: K, payload: T[K]): void {
    this.handlers.get(event)?.forEach(fn => fn(payload));
  }
}

const emitter = new TypedEmitter<EventMap>();
emitter.on('login', ({ userId }) => console.log(userId)); // fully typed
emitter.emit('login', { userId: '123' });                 // validated
```

**Exhaustive pattern matching:**
```ts
type Action =
  | { type: 'ADD'; payload: string }
  | { type: 'REMOVE'; id: number }
  | { type: 'CLEAR' };

function reducer(state: string[], action: Action): string[] {
  switch (action.type) {
    case 'ADD': return [...state, action.payload];
    case 'REMOVE': return state.filter((_, i) => i !== action.id);
    case 'CLEAR': return [];
    default: {
      const _: never = action; // compile error if cases are missing
      return state;
    }
  }
}
```

**Path type for nested objects:**
```ts
type Path<T, Prefix extends string = ''> = T extends object
  ? {
      [K in keyof T & string]: T[K] extends object
        ? `${Prefix}${K}` | Path<T[K], `${Prefix}${K}.`>
        : `${Prefix}${K}`;
    }[keyof T & string]
  : never;

interface Config {
  db: { host: string; port: number };
  app: { name: string };
}

type ConfigPath = Path<Config>;
// 'db' | 'db.host' | 'db.port' | 'app' | 'app.name'
```
