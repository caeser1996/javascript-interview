# TypeScript Quick Revision Cheatsheet

## Basic Types

```typescript
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b"];

// Tuple
let tuple: [string, number] = ["John", 30];

// Any (avoid)
let anything: any = "hello";

// Unknown (safer than any)
let value: unknown = "hello";
if (typeof value === "string") {
  value.toUpperCase(); // OK after type check
}

// Never
function error(message: string): never {
  throw new Error(message);
}

// Void
function log(message: string): void {
  console.log(message);
}
```

## Interfaces

```typescript
interface User {
  name: string;
  age: number;
  email?: string;        // Optional
  readonly id: number;   // Readonly
}

// Function interface
interface SearchFunc {
  (source: string, subString: string): boolean;
}

// Extending interfaces
interface Employee extends User {
  employeeId: number;
}

// Index signature
interface StringArray {
  [index: number]: string;
}
```

## Type Aliases

```typescript
type ID = string | number;
type Point = { x: number; y: number };
type Status = "active" | "inactive";

// Intersection
type NamedPoint = Point & { name: string };

// Union
type Result = Success | Error;
```

## Generics

```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

// Generic interface
interface Container<T> {
  value: T;
}

// Generic class
class Box<T> {
  constructor(public value: T) {}
}

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}
```

## Utility Types

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

// Partial - All properties optional
type PartialUser = Partial<User>;

// Required - All properties required
type RequiredUser = Required<User>;

// Readonly - All properties readonly
type ReadonlyUser = Readonly<User>;

// Pick - Select properties
type UserNameAge = Pick<User, "name" | "age">;

// Omit - Exclude properties
type UserWithoutEmail = Omit<User, "email">;

// Record - Create object type
type Roles = Record<string, string[]>;

// ReturnType - Extract return type
function getUser() {
  return { name: "John", age: 30 };
}
type User = ReturnType<typeof getUser>;

// Parameters - Extract parameter types
function greet(name: string, age: number) {}
type GreetParams = Parameters<typeof greet>; // [string, number]
```

## Type Guards

```typescript
// typeof
function print(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}

// instanceof
if (obj instanceof Dog) {
  obj.bark();
}

// in operator
if ("swim" in animal) {
  animal.swim();
}

// Custom type guard
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; sideLength: number };

function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  } else {
    return shape.sideLength ** 2;
  }
}
```

## Union & Intersection

```typescript
// Union (OR)
type StringOrNumber = string | number;

function printId(id: StringOrNumber) {
  console.log(id);
}

// Intersection (AND)
type Person = { name: string };
type Employee = { employeeId: number };
type EmployeePerson = Person & Employee;
```

## Literal Types

```typescript
type Direction = "north" | "south" | "east" | "west";
type StatusCode = 200 | 404 | 500;

function move(direction: Direction) {
  // direction can only be one of the four values
}
```

## Type Assertions

```typescript
// as syntax (preferred)
let value: unknown = "hello";
let length = (value as string).length;

// Angle bracket syntax
let length2 = (<string>value).length;

// Non-null assertion
let name: string | null = getName();
let upperName = name!.toUpperCase(); // Assert non-null

// Const assertion
const config = {
  endpoint: "/api",
  timeout: 3000
} as const; // All properties become readonly
```

## Classes

```typescript
class Person {
  // Properties
  name: string;
  private age: number;
  protected id: number;
  readonly createdAt: Date;

  // Constructor
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
    this.id = Math.random();
    this.createdAt = new Date();
  }

  // Method
  greet(): void {
    console.log(`Hi, I'm ${this.name}`);
  }

  // Static method
  static species(): string {
    return "Homo sapiens";
  }

  // Getter
  get info(): string {
    return `${this.name}, ${this.age}`;
  }

  // Setter
  set nickname(value: string) {
    this.name = value;
  }
}

// Inheritance
class Employee extends Person {
  constructor(name: string, age: number, public role: string) {
    super(name, age);
  }
}

// Abstract class
abstract class Animal {
  abstract makeSound(): void;

  move(): void {
    console.log("Moving...");
  }
}

// Implements interface
interface Movable {
  move(): void;
}

class Car implements Movable {
  move() {
    console.log("Driving...");
  }
}
```

## Enums

```typescript
// Numeric enum
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

// String enum
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
  Pending = "PENDING"
}

// Const enum (inlined at compile time)
const enum Color {
  Red,
  Green,
  Blue
}
```

## Advanced Types

### Mapped Types

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};
```

### Conditional Types

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// With infer
type ArrayElement<T> = T extends (infer U)[] ? U : never;
type C = ArrayElement<string[]>; // string
```

### Template Literal Types

```typescript
type Greeting = `Hello ${string}`;

type EventName = "click" | "focus";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus"
```

## Decorators

```typescript
// Enable in tsconfig.json:
// "experimentalDecorators": true

// Class decorator
function Component(config: any) {
  return function<T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      config = config;
    };
  };
}

@Component({ selector: "app-user" })
class UserComponent {}

// Method decorator
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey}`);
    return originalMethod.apply(this, args);
  };
}

class Calculator {
  @log
  add(a: number, b: number) {
    return a + b;
  }
}
```

## Type Narrowing

```typescript
// Type narrowing with typeof
function process(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // TypeScript knows it's string
  }
  return value.toFixed(2); // TypeScript knows it's number
}

// Truthiness narrowing
function print(str: string | null | undefined) {
  if (str) {
    console.log(str.trim()); // TypeScript knows str is string
  }
}

// Equality narrowing
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    x.toUpperCase(); // Both must be string
  }
}
```

## Configuration (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

## Common Interview Questions

### Q: Interface vs Type?
- **Interface**: Can be extended, declaration merging, OOP
- **Type**: Unions, intersections, computed properties

Use interface for object shapes, type for complex types.

### Q: What is `keyof`?
Gets keys of a type as union of string literals.

```typescript
interface User {
  name: string;
  age: number;
}
type UserKeys = keyof User; // "name" | "age"
```

### Q: What is `typeof`?
Gets type of a value.

```typescript
const user = { name: "John", age: 30 };
type User = typeof user; // { name: string; age: number }
```

### Q: What is `never`?
Represents values that never occur. Used for exhaustive checks.

```typescript
function error(message: string): never {
  throw new Error(message);
}
```

### Q: What is `unknown`?
Type-safe counterpart of `any`. Must narrow before use.

```typescript
let value: unknown;
if (typeof value === "string") {
  value.toUpperCase(); // OK
}
```

### Q: Generics benefits?
- Type safety
- Code reusability
- Better IntelliSense
- Compile-time checks

### Q: When to use `as const`?
Makes properties readonly and literal types.

```typescript
const colors = ["red", "green"] as const;
// readonly ["red", "green"]
```

## Best Practices

✅ Enable `strict` mode in tsconfig.json
✅ Avoid `any`, use `unknown` instead
✅ Use interfaces for public APIs
✅ Use type aliases for unions and complex types
✅ Leverage utility types (Partial, Pick, etc.)
✅ Create custom type guards for complex checks
✅ Use const assertions for readonly data
✅ Enable `noImplicitAny` and `strictNullChecks`
✅ Use generics for reusable components
✅ Prefer union types over enums when possible

## Key Takeaways

- TypeScript adds static typing to JavaScript
- Interfaces define contracts for objects
- Generics enable type-safe reusable code
- Utility types transform existing types
- Type guards narrow union types
- Strict mode catches more errors
- Decorators add metadata to classes
- Use type inference when possible
