# TypeScript Types and Interfaces

## Table of Contents
- [Basic Types](#basic-types)
- [Interfaces](#interfaces)
- [Type Aliases](#type-aliases)
- [Union and Intersection Types](#union-and-intersection-types)
- [Literal Types](#literal-types)
- [Type Assertions](#type-assertions)
- [Common Interview Questions](#common-interview-questions)

## Basic Types

### Primitive Types

```typescript
// Boolean
let isDone: boolean = false;

// Number
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// String
let color: string = "blue";
let fullName: string = `John Doe`;

// Array
let list1: number[] = [1, 2, 3];
let list2: Array<number> = [1, 2, 3];

// Tuple
let tuple: [string, number] = ["hello", 10];

// Enum
enum Color {
  Red,
  Green,
  Blue
}
let c: Color = Color.Green;

// Any (avoid when possible)
let notSure: any = 4;
notSure = "maybe a string";
notSure = false;

// Void
function warnUser(): void {
  console.log("Warning!");
}

// Null and Undefined
let u: undefined = undefined;
let n: null = null;

// Never (function that never returns)
function error(message: string): never {
  throw new Error(message);
}

// Object
let obj: object = { name: "John" };
```

### Type Inference

```typescript
// TypeScript infers types
let message = "Hello"; // inferred as string
// message = 42; // Error

let numbers = [1, 2, 3]; // inferred as number[]
let mixed = [1, "hello"]; // inferred as (string | number)[]

// Return type inference
function add(a: number, b: number) {
  return a + b; // inferred return type: number
}
```

## Interfaces

### Basic Interface

```typescript
interface User {
  name: string;
  age: number;
  email?: string; // Optional property
  readonly id: number; // Read-only property
}

const user: User = {
  name: "John",
  age: 30,
  id: 1
};

// user.id = 2; // Error: Cannot assign to 'id'
```

### Function Types in Interfaces

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = (src, sub) => {
  return src.includes(sub);
};

// Method syntax
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};
```

### Indexable Types

```typescript
// String index signature
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = ["Bob", "Fred"];

// Dictionary pattern
interface Dictionary {
  [key: string]: any;
}

const dict: Dictionary = {
  name: "John",
  age: 30,
  active: true
};
```

### Extending Interfaces

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: number;
  department: string;
}

const employee: Employee = {
  name: "John",
  age: 30,
  employeeId: 12345,
  department: "IT"
};

// Multiple inheritance
interface Animal {
  species: string;
}

interface Pet {
  name: string;
}

interface Dog extends Animal, Pet {
  breed: string;
}

const myDog: Dog = {
  species: "Canis familiaris",
  name: "Rex",
  breed: "Labrador"
};
```

### Interface for Classes

```typescript
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();

  setTime(d: Date) {
    this.currentTime = d;
  }
}
```

## Type Aliases

### Basic Type Alias

```typescript
type ID = string | number;

let userId: ID = "abc123";
userId = 123; // Also valid

type Point = {
  x: number;
  y: number;
};

const point: Point = { x: 10, y: 20 };
```

### Generic Type Aliases

```typescript
type Container<T> = {
  value: T;
};

const stringContainer: Container<string> = { value: "hello" };
const numberContainer: Container<number> = { value: 42 };

// Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type User = {
  name: string;
  age: number;
};

type ReadonlyUser = Readonly<User>;
// Same as: { readonly name: string; readonly age: number; }
```

## Union and Intersection Types

### Union Types

```typescript
// Union type (OR)
type StringOrNumber = string | number;

function printId(id: StringOrNumber) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id);
  }
}

printId("abc"); // OK
printId(123);   // OK

// Union of object types
type Success = {
  status: "success";
  data: any;
};

type Error = {
  status: "error";
  message: string;
};

type Response = Success | Error;

function handleResponse(response: Response) {
  if (response.status === "success") {
    console.log(response.data);
  } else {
    console.log(response.message);
  }
}
```

### Intersection Types

```typescript
// Intersection type (AND)
type HasName = {
  name: string;
};

type HasAge = {
  age: number;
};

type Person = HasName & HasAge;

const person: Person = {
  name: "John",
  age: 30
};

// Combining multiple types
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;

const cc: ColorfulCircle = {
  color: "red",
  radius: 10
};
```

## Literal Types

```typescript
// String literal types
type Direction = "north" | "south" | "east" | "west";

function move(direction: Direction) {
  console.log(`Moving ${direction}`);
}

move("north"); // OK
// move("up"); // Error

// Numeric literal types
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

function rollDice(): DiceRoll {
  return (Math.floor(Math.random() * 6) + 1) as DiceRoll;
}

// Boolean literal types
type True = true;
type False = false;

// Combining literals with unions
type Status = "loading" | "success" | "error";
type StatusCode = 200 | 404 | 500;

type ApiResponse = {
  status: Status;
  code: StatusCode;
};
```

## Type Assertions

```typescript
// "as" syntax (preferred)
let someValue: unknown = "this is a string";
let strLength: number = (someValue as string).length;

// Angle-bracket syntax (not usable in JSX)
let strLength2: number = (<string>someValue).length;

// Non-null assertion
function getValue(key: string): string | null {
  return key === "name" ? "John" : null;
}

const name = getValue("name")!; // Assert non-null
const upperName = name.toUpperCase(); // No error

// Const assertions
let x = "hello"; // type: string
let y = "hello" as const; // type: "hello"

const colors = ["red", "green", "blue"] as const;
// type: readonly ["red", "green", "blue"]

const config = {
  endpoint: "/api",
  timeout: 3000
} as const;
// All properties become readonly
```

## Common Interview Questions

### Q1: Interface vs Type Alias - What's the difference?

```typescript
// Interface
interface UserInterface {
  name: string;
  age: number;
}

// Type Alias
type UserType = {
  name: string;
  age: number;
};
```

**Key Differences**:

1. **Declaration Merging** (Interfaces only)
```typescript
interface Window {
  title: string;
}

interface Window {
  ts: number;
}

// Merged into one interface
const w: Window = {
  title: "Browser",
  ts: Date.now()
};

// Type aliases cannot be merged
// type Window = { title: string; }
// type Window = { ts: number; } // Error: Duplicate identifier
```

2. **Extending**
```typescript
// Interface extends
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Type intersection
type Animal = {
  name: string;
};

type Dog = Animal & {
  breed: string;
};
```

3. **Union Types** (Type only)
```typescript
// Type alias can represent unions
type ID = string | number;

// Interface cannot
// interface ID = string | number; // Error
```

**When to use what**:
- Use **interface** for object shapes, especially when building libraries (declaration merging)
- Use **type** for unions, intersections, and complex type compositions

### Q2: What is Type Narrowing?

```typescript
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    // TypeScript knows padding is number here
    return " ".repeat(padding) + value;
  }
  // TypeScript knows padding is string here
  return padding + value;
}

// Discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number };

function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    // TypeScript knows shape has radius here
    return Math.PI * shape.radius ** 2;
  }
  // TypeScript knows shape has sideLength here
  return shape.sideLength ** 2;
}

// Using 'in' operator
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();
  } else {
    animal.fly();
  }
}

// Using instanceof
class Dog {
  bark() {
    console.log("Woof!");
  }
}

class Cat {
  meow() {
    console.log("Meow!");
  }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

### Q3: What are Index Signatures?

```typescript
// Basic index signature
interface StringDictionary {
  [key: string]: string;
}

const dict: StringDictionary = {
  name: "John",
  city: "NYC"
};

// Mixed with defined properties
interface NumberDictionary {
  length: number; // Known property
  [index: string]: number; // All other properties must be number
}

// Record utility type (preferred)
type Dictionary = Record<string, string>;

const myDict: Dictionary = {
  key1: "value1",
  key2: "value2"
};

// Nested records
type NestedDict = Record<string, Record<string, number>>;

const nested: NestedDict = {
  user1: { age: 30, score: 100 },
  user2: { age: 25, score: 95 }
};
```

### Q4: How to make all properties optional/required?

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

// Make all optional (Partial)
type PartialUser = Partial<User>;
// Same as: { name?: string; age?: number; email?: string; }

// Make all required (Required)
interface UserOptional {
  name?: string;
  age?: number;
  email?: string;
}

type RequiredUser = Required<UserOptional>;
// Same as: { name: string; age: number; email: string; }

// Make all readonly (Readonly)
type ReadonlyUser = Readonly<User>;
// Same as: { readonly name: string; readonly age: number; readonly email: string; }

// Pick specific properties
type UserNameAndAge = Pick<User, "name" | "age">;
// Same as: { name: string; age: number; }

// Omit specific properties
type UserWithoutEmail = Omit<User, "email">;
// Same as: { name: string; age: number; }
```

### Q5: What are Conditional Types?

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// Exclude null/undefined
type NonNullable<T> = T extends null | undefined ? never : T;

type C = NonNullable<string | null>; // string
type D = NonNullable<number | undefined>; // number

// Extract function return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet(): string {
  return "Hello";
}

type GreetReturn = ReturnType<typeof greet>; // string

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;

type StrOrNumArray = ToArray<string | number>;
// Same as: string[] | number[]
```

### Q6: Explain 'keyof' and 'typeof'

```typescript
// keyof: get keys of an interface/type
interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonKeys = keyof Person; // "name" | "age" | "email"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person: Person = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

const name = getProperty(person, "name"); // type: string
const age = getProperty(person, "age");   // type: number

// typeof: get type of a value
const user = {
  name: "John",
  age: 30
};

type User = typeof user; // { name: string; age: number; }

// Combined usage
const colors = {
  red: "#ff0000",
  green: "#00ff00",
  blue: "#0000ff"
} as const;

type ColorName = keyof typeof colors; // "red" | "green" | "blue"
type ColorValue = typeof colors[ColorName]; // "#ff0000" | "#00ff00" | "#0000ff"
```

## Best Practices

1. **Prefer interfaces for object shapes**
   ```typescript
   // ✅ Good
   interface User {
     name: string;
     age: number;
   }

   // ❌ Less preferred (unless you need union/intersection)
   type User = {
     name: string;
     age: number;
   };
   ```

2. **Use readonly for immutable properties**
   ```typescript
   interface Config {
     readonly apiUrl: string;
     readonly timeout: number;
   }
   ```

3. **Avoid 'any', use 'unknown' instead**
   ```typescript
   // ❌ Bad
   function processData(data: any) {
     return data.value;
   }

   // ✅ Good
   function processData(data: unknown) {
     if (typeof data === "object" && data !== null && "value" in data) {
       return (data as { value: any }).value;
     }
   }
   ```

4. **Use type guards for narrowing**
   ```typescript
   function isString(value: unknown): value is string {
     return typeof value === "string";
   }

   function process(value: unknown) {
     if (isString(value)) {
       console.log(value.toUpperCase());
     }
   }
   ```

## Key Takeaways

1. TypeScript provides static type checking for JavaScript
2. Interfaces define contracts for object shapes
3. Type aliases can represent any type, including unions
4. Use union types for "OR" relationships, intersections for "AND"
5. Literal types restrict values to specific constants
6. Type narrowing helps TypeScript understand your code better
7. Utility types (Partial, Required, etc.) transform types
8. 'keyof' and 'typeof' are powerful for type manipulation

## Practice Problems

1. Create a type-safe event emitter
2. Implement a deep readonly type
3. Build a type-safe API client with proper types
4. Create discriminated unions for different response types
5. Implement a type-safe Redux-like store

---

**Next Topic**: [Generics](./generics.md)
