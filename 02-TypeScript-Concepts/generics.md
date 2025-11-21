# TypeScript Generics

## Table of Contents
- [What are Generics?](#what-are-generics)
- [Generic Functions](#generic-functions)
- [Generic Interfaces](#generic-interfaces)
- [Generic Classes](#generic-classes)
- [Generic Constraints](#generic-constraints)
- [Utility Types with Generics](#utility-types-with-generics)
- [Common Interview Questions](#common-interview-questions)

## What are Generics?

Generics allow you to write reusable, type-safe code that works with multiple types.

### Why Generics?

```typescript
// Without generics (not type-safe)
function identityAny(arg: any): any {
  return arg;
}

let output1 = identityAny("hello"); // type: any (lost type information)
let output2 = identityAny(123);     // type: any

// With generics (type-safe)
function identity<T>(arg: T): T {
  return arg;
}

let output3 = identity<string>("hello"); // type: string
let output4 = identity<number>(123);     // type: number
let output5 = identity("hello");         // type inferred as string
```

## Generic Functions

### Basic Generic Function

```typescript
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const numbers = [1, 2, 3];
const first = firstElement(numbers); // type: number | undefined

const strings = ["a", "b", "c"];
const firstStr = firstElement(strings); // type: string | undefined
```

### Multiple Type Parameters

```typescript
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const p1 = pair("hello", 42);        // type: [string, number]
const p2 = pair(true, "world");      // type: [boolean, string]

// Swapping function
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]];
}

const original: [string, number] = ["hello", 42];
const swapped = swap(original); // type: [number, string]
```

### Generic Arrow Functions

```typescript
// TSX syntax (to avoid conflict with JSX)
const identity = <T,>(arg: T): T => arg;

// Array syntax
const toArray = <T>(arg: T): T[] => [arg];

const nums = toArray(42);      // type: number[]
const strs = toArray("hello"); // type: string[]
```

## Generic Interfaces

### Basic Generic Interface

```typescript
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

const stringContainer: Container<string> = {
  value: "hello",
  getValue() {
    return this.value;
  },
  setValue(value: string) {
    this.value = value;
  }
};

const numberContainer: Container<number> = {
  value: 42,
  getValue() {
    return this.value;
  },
  setValue(value: number) {
    this.value = value;
  }
};
```

### Generic Function Interface

```typescript
interface GenericFunction<T> {
  (arg: T): T;
}

const identity: GenericFunction<string> = (arg) => arg;
const result = identity("hello"); // type: string
```

### Array-like Interface

```typescript
interface ReadonlyArray<T> {
  readonly length: number;
  [index: number]: T;
  map<U>(fn: (value: T) => U): U[];
  filter(fn: (value: T) => boolean): T[];
}
```

## Generic Classes

### Basic Generic Class

```typescript
class Box<T> {
  private value: T;

  constructor(value: T) {
    this.value = value;
  }

  getValue(): T {
    return this.value;
  }

  setValue(value: T): void {
    this.value = value;
  }
}

const stringBox = new Box("hello");
console.log(stringBox.getValue()); // "hello"

const numberBox = new Box(42);
console.log(numberBox.getValue()); // 42
```

### Generic Collection Class

```typescript
class Collection<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  remove(item: T): void {
    const index = this.items.indexOf(item);
    if (index > -1) {
      this.items.splice(index, 1);
    }
  }

  get(index: number): T | undefined {
    return this.items[index];
  }

  getAll(): T[] {
    return [...this.items];
  }

  map<U>(fn: (item: T) => U): U[] {
    return this.items.map(fn);
  }

  filter(fn: (item: T) => boolean): T[] {
    return this.items.filter(fn);
  }
}

const numbers = new Collection<number>();
numbers.add(1);
numbers.add(2);
numbers.add(3);

const doubled = numbers.map(n => n * 2); // type: number[]
const evens = numbers.filter(n => n % 2 === 0); // type: number[]
```

### Generic Stack Implementation

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }

  size(): number {
    return this.items.length;
  }
}

const stack = new Stack<number>();
stack.push(1);
stack.push(2);
stack.push(3);
console.log(stack.pop()); // 3
console.log(stack.peek()); // 2
```

## Generic Constraints

### Basic Constraints

```typescript
// Constraint: T must have a 'length' property
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(arg: T): void {
  console.log(arg.length);
}

logLength("hello");       // OK
logLength([1, 2, 3]);     // OK
logLength({ length: 10 }); // OK
// logLength(123);        // Error: number doesn't have 'length'
```

### Using keyof Constraint

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

const name = getProperty(person, "name");   // type: string
const age = getProperty(person, "age");     // type: number
// getProperty(person, "address"); // Error: 'address' doesn't exist
```

### Multiple Constraints

```typescript
interface Lengthwise {
  length: number;
}

interface Printable {
  print(): void;
}

function process<T extends Lengthwise & Printable>(arg: T): void {
  console.log(arg.length);
  arg.print();
}

// Usage
const obj = {
  length: 5,
  print() {
    console.log("Printing...");
  }
};

process(obj); // OK
```

### Class Type Constraints

```typescript
function create<T>(constructor: new () => T): T {
  return new constructor();
}

class Person {
  name: string = "";
}

const person = create(Person); // type: Person

// With constructor parameters
function createWith<T>(
  constructor: new (...args: any[]) => T,
  ...args: any[]
): T {
  return new constructor(...args);
}

class User {
  constructor(public name: string, public age: number) {}
}

const user = createWith(User, "John", 30); // type: User
```

## Utility Types with Generics

### Partial<T>

```typescript
// Makes all properties optional
interface User {
  name: string;
  age: number;
  email: string;
}

type PartialUser = Partial<User>;
// Same as: { name?: string; age?: number; email?: string; }

// Implementation
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

// Usage
function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

const user: User = { name: "John", age: 30, email: "john@example.com" };
const updated = updateUser(user, { age: 31 }); // Only update age
```

### Required<T>

```typescript
// Makes all properties required
interface UserOptional {
  name?: string;
  age?: number;
}

type RequiredUser = Required<UserOptional>;
// Same as: { name: string; age: number; }

// Implementation
type MyRequired<T> = {
  [P in keyof T]-?: T[P];
};
```

### Readonly<T>

```typescript
// Makes all properties readonly
type ReadonlyUser = Readonly<User>;

// Implementation
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Usage
const user: Readonly<User> = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

// user.age = 31; // Error: Cannot assign to 'age'
```

### Pick<T, K>

```typescript
// Pick specific properties
type UserNameAndAge = Pick<User, "name" | "age">;
// Same as: { name: string; age: number; }

// Implementation
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

### Omit<T, K>

```typescript
// Omit specific properties
type UserWithoutEmail = Omit<User, "email">;
// Same as: { name: string; age: number; }

// Implementation
type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

### Record<K, T>

```typescript
// Create object type with specific keys and value type
type Roles = "admin" | "user" | "guest";
type RolePermissions = Record<Roles, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// Implementation
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
};
```

### ReturnType<T>

```typescript
// Extract return type of a function
function getUser() {
  return { name: "John", age: 30 };
}

type User = ReturnType<typeof getUser>; // { name: string; age: number; }

// Implementation
type MyReturnType<T extends (...args: any[]) => any> =
  T extends (...args: any[]) => infer R ? R : never;
```

## Common Interview Questions

### Q1: Implement a Generic Queue

```typescript
class Queue<T> {
  private items: T[] = [];

  enqueue(item: T): void {
    this.items.push(item);
  }

  dequeue(): T | undefined {
    return this.items.shift();
  }

  peek(): T | undefined {
    return this.items[0];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }

  size(): number {
    return this.items.length;
  }

  clear(): void {
    this.items = [];
  }

  toArray(): T[] {
    return [...this.items];
  }
}

// Usage
const queue = new Queue<number>();
queue.enqueue(1);
queue.enqueue(2);
queue.enqueue(3);
console.log(queue.dequeue()); // 1
console.log(queue.peek());    // 2
```

### Q2: Implement a Generic Linked List

```typescript
class ListNode<T> {
  value: T;
  next: ListNode<T> | null = null;

  constructor(value: T) {
    this.value = value;
  }
}

class LinkedList<T> {
  private head: ListNode<T> | null = null;
  private tail: ListNode<T> | null = null;
  private length: number = 0;

  append(value: T): void {
    const node = new ListNode(value);

    if (!this.head) {
      this.head = node;
      this.tail = node;
    } else {
      this.tail!.next = node;
      this.tail = node;
    }

    this.length++;
  }

  prepend(value: T): void {
    const node = new ListNode(value);
    node.next = this.head;
    this.head = node;

    if (!this.tail) {
      this.tail = node;
    }

    this.length++;
  }

  find(value: T): ListNode<T> | null {
    let current = this.head;

    while (current) {
      if (current.value === value) {
        return current;
      }
      current = current.next;
    }

    return null;
  }

  toArray(): T[] {
    const result: T[] = [];
    let current = this.head;

    while (current) {
      result.push(current.value);
      current = current.next;
    }

    return result;
  }

  size(): number {
    return this.length;
  }
}
```

### Q3: Implement DeepPartial

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface User {
  name: string;
  age: number;
  address: {
    street: string;
    city: string;
    country: string;
  };
}

type PartialUser = DeepPartial<User>;

const user: PartialUser = {
  name: "John",
  address: {
    city: "NYC" // Other address fields are optional
  }
};
```

### Q4: Implement DeepReadonly

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface Config {
  database: {
    host: string;
    port: number;
  };
  api: {
    endpoint: string;
  };
}

type ReadonlyConfig = DeepReadonly<Config>;

const config: ReadonlyConfig = {
  database: { host: "localhost", port: 5432 },
  api: { endpoint: "/api" }
};

// config.database.host = "newhost"; // Error: readonly
```

### Q5: Implement a Type-Safe Event Emitter

```typescript
type EventMap = Record<string, any>;

class TypedEventEmitter<Events extends EventMap> {
  private listeners: {
    [K in keyof Events]?: Array<(data: Events[K]) => void>;
  } = {};

  on<K extends keyof Events>(
    event: K,
    listener: (data: Events[K]) => void
  ): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }

  emit<K extends keyof Events>(event: K, data: Events[K]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }

  off<K extends keyof Events>(
    event: K,
    listener: (data: Events[K]) => void
  ): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      const index = eventListeners.indexOf(listener);
      if (index > -1) {
        eventListeners.splice(index, 1);
      }
    }
  }
}

// Usage
interface AppEvents {
  userLogin: { userId: string; timestamp: number };
  userLogout: { userId: string };
  dataUpdate: { data: any[] };
}

const emitter = new TypedEventEmitter<AppEvents>();

emitter.on("userLogin", (data) => {
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.emit("userLogin", { userId: "123", timestamp: Date.now() });
// emitter.emit("userLogin", { userId: 123 }); // Error: wrong type
```

### Q6: Implement a Generic API Client

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

class ApiClient {
  async get<T>(url: string): Promise<ApiResponse<T>> {
    const response = await fetch(url);
    const data = await response.json();
    return {
      data,
      status: response.status,
      message: response.statusText
    };
  }

  async post<T, U>(url: string, body: U): Promise<ApiResponse<T>> {
    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(body)
    });
    const data = await response.json();
    return {
      data,
      status: response.status,
      message: response.statusText
    };
  }
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
}

interface CreateUserData {
  name: string;
  email: string;
}

const client = new ApiClient();

// Type-safe API calls
async function getUser(id: string) {
  const response = await client.get<User>(`/api/users/${id}`);
  return response.data; // type: User
}

async function createUser(data: CreateUserData) {
  const response = await client.post<User, CreateUserData>("/api/users", data);
  return response.data; // type: User
}
```

## Advanced Generic Patterns

### Conditional Types with Generics

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type A = NonNullable<string | null>; // string
type B = NonNullable<number | undefined>; // number

// Extract types
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type StringArray = ArrayElement<string[]>; // string
type NumberArray = ArrayElement<number[]>; // number
```

### Mapped Types with Generics

```typescript
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

interface User {
  name: string;
  age: number;
}

type NullableUser = Nullable<User>;
// { name: string | null; age: number | null; }

// With modifiers
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]+?: T[P];
};
```

## Best Practices

1. **Use generic type inference when possible**
   ```typescript
   // ✅ Good: Let TypeScript infer
   const result = identity("hello");

   // ❌ Unnecessary: Explicit type
   const result = identity<string>("hello");
   ```

2. **Add constraints to make generics safer**
   ```typescript
   // ❌ Too permissive
   function merge<T, U>(obj1: T, obj2: U) {
     return { ...obj1, ...obj2 };
   }

   // ✅ Better: Constrain to objects
   function merge<T extends object, U extends object>(obj1: T, obj2: U) {
     return { ...obj1, ...obj2 };
   }
   ```

3. **Use descriptive generic names for complex types**
   ```typescript
   // ❌ Unclear
   function map<T, U>(arr: T[], fn: (x: T) => U): U[] {
     return arr.map(fn);
   }

   // ✅ Clear
   function map<Input, Output>(
     arr: Input[],
     transform: (value: Input) => Output
   ): Output[] {
     return arr.map(transform);
   }
   ```

## Key Takeaways

1. Generics enable writing reusable, type-safe code
2. Use constraints (`extends`) to limit acceptable types
3. TypeScript can infer generic types in most cases
4. Utility types are built with generics
5. Conditional types enable advanced type transformations
6. Generics work with functions, interfaces, and classes
7. `keyof` and `typeof` are powerful with generics

## Practice Problems

1. Implement a generic binary search tree
2. Create a type-safe state management system
3. Build a generic async cache with TTL
4. Implement a type-safe form validator
5. Create a generic pub/sub system

---

**Next Topic**: [Utility Types](./utility-types.md)
