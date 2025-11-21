# TypeScript Utility Types

## Table of Contents
- [Built-in Utility Types](#built-in-utility-types)
- [Creating Custom Utility Types](#creating-custom-utility-types)
- [Advanced Type Manipulation](#advanced-type-manipulation)
- [Real-World Examples](#real-world-examples)

## Built-in Utility Types

### Partial<T>

Makes all properties optional.

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

type PartialUser = Partial<User>;
// { name?: string; age?: number; email?: string; }

// Use case: Update functions
function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

const user: User = { name: "John", age: 30, email: "john@example.com" };
const updated = updateUser(user, { age: 31 });
```

### Required<T>

Makes all properties required.

```typescript
interface Config {
  host?: string;
  port?: number;
  database?: string;
}

type RequiredConfig = Required<Config>;
// { host: string; port: number; database: string; }

function validateConfig(config: Required<Config>): boolean {
  // All properties are guaranteed to exist
  return config.host.length > 0 && config.port > 0;
}
```

### Readonly<T>

Makes all properties readonly.

```typescript
interface MutableUser {
  name: string;
  age: number;
}

type ImmutableUser = Readonly<MutableUser>;

const user: ImmutableUser = { name: "John", age: 30 };
// user.age = 31; // Error: Cannot assign to 'age'

// Use case: Configuration objects
const config: Readonly<{apiUrl: string; timeout: number}> = {
  apiUrl: "https://api.example.com",
  timeout: 5000
};
```

### Pick<T, K>

Creates a type with only specified properties from T.

```typescript
interface User {
  id: string;
  name: string;
  age: number;
  email: string;
  password: string;
}

type UserProfile = Pick<User, "id" | "name" | "email">;
// { id: string; name: string; email: string; }

// Use case: API responses (hide sensitive data)
function getUserProfile(user: User): UserProfile {
  const { id, name, email } = user;
  return { id, name, email };
}
```

### Omit<T, K>

Creates a type by omitting specified properties from T.

```typescript
type PublicUser = Omit<User, "password">;
// { id: string; name: string; age: number; email: string; }

// Use case: Database models
interface UserModel extends Omit<User, "id"> {
  _id: string; // MongoDB style
}
```

### Record<K, T>

Creates an object type with keys K and values T.

```typescript
type Role = "admin" | "user" | "guest";

type Permissions = Record<Role, string[]>;

const permissions: Permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// Dictionary pattern
type StringDictionary = Record<string, string>;

const translations: StringDictionary = {
  hello: "Hola",
  goodbye: "Adiós",
  thanks: "Gracias"
};

// Numeric keys
type PageMap = Record<number, { title: string; content: string }>;

const pages: PageMap = {
  1: { title: "Home", content: "Welcome" },
  2: { title: "About", content: "About us" }
};
```

### Exclude<T, U>

Excludes types from T that are assignable to U.

```typescript
type AllTypes = "a" | "b" | "c" | "d";
type Excluded = Exclude<AllTypes, "a" | "c">;
// "b" | "d"

// Use case: Remove specific types from union
type Primitive = string | number | boolean | null | undefined;
type NonNullPrimitive = Exclude<Primitive, null | undefined>;
// string | number | boolean

// Remove function types
type Mixed = string | number | (() => void);
type NonFunction = Exclude<Mixed, Function>;
// string | number
```

### Extract<T, U>

Extracts types from T that are assignable to U.

```typescript
type AllTypes = "a" | "b" | "c" | "d";
type Extracted = Extract<AllTypes, "a" | "c">;
// "a" | "c"

// Use case: Filter specific types
type Mixed = string | number | boolean | (() => void);
type OnlyFunctions = Extract<Mixed, Function>;
// () => void

// Extract string literals starting with certain prefix
type Routes = "/home" | "/about" | "/api/users" | "/api/posts";
type ApiRoutes = Extract<Routes, `/api/${string}`>;
// "/api/users" | "/api/posts"
```

### NonNullable<T>

Removes null and undefined from T.

```typescript
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// string

// Use case: Function parameters
function processValue(value: string | null): NonNullable<typeof value> {
  if (value === null) {
    throw new Error("Value cannot be null");
  }
  return value; // type: string
}
```

### ReturnType<T>

Extracts the return type of a function.

```typescript
function getUser() {
  return { name: "John", age: 30 };
}

type User = ReturnType<typeof getUser>;
// { name: string; age: number; }

// With generic functions
function createArray<T>(item: T): T[] {
  return [item];
}

type ArrayReturn = ReturnType<typeof createArray<string>>;
// string[]

// API response types
const fetchData = async () => {
  return { data: [], total: 0, page: 1 };
};

type FetchDataReturn = ReturnType<typeof fetchData>;
// Promise<{ data: never[]; total: number; page: number; }>

type UnwrappedData = Awaited<ReturnType<typeof fetchData>>;
// { data: never[]; total: number; page: number; }
```

### Parameters<T>

Extracts parameter types of a function as a tuple.

```typescript
function greet(name: string, age: number): string {
  return `Hello ${name}, you are ${age} years old`;
}

type GreetParams = Parameters<typeof greet>;
// [name: string, age: number]

// Use case: Function wrapper
function loggedGreet(...args: Parameters<typeof greet>): ReturnType<typeof greet> {
  console.log("Calling greet with:", args);
  return greet(...args);
}
```

### ConstructorParameters<T>

Extracts constructor parameter types.

```typescript
class User {
  constructor(public name: string, public age: number) {}
}

type UserConstructorParams = ConstructorParameters<typeof User>;
// [name: string, age: number]

// Use case: Factory function
function createUser(...args: ConstructorParameters<typeof User>): User {
  return new User(...args);
}
```

### InstanceType<T>

Extracts instance type of a class constructor.

```typescript
class User {
  constructor(public name: string) {}
}

type UserInstance = InstanceType<typeof User>;
// User

// Use case: Generic factory
function createInstance<T extends new (...args: any[]) => any>(
  constructor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new constructor(...args);
}

const user = createInstance(User, "John");
```

### Awaited<T>

Unwraps Promise type recursively.

```typescript
type P1 = Awaited<Promise<string>>;
// string

type P2 = Awaited<Promise<Promise<number>>>;
// number

// Use case: Async function return types
async function fetchUser() {
  return { name: "John", age: 30 };
}

type User = Awaited<ReturnType<typeof fetchUser>>;
// { name: string; age: number; }
```

## Creating Custom Utility Types

### DeepPartial<T>

Makes all properties and nested properties optional.

```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? DeepPartial<T[P]>
    : T[P];
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
    city: "NYC" // other address fields optional
  }
};
```

### DeepReadonly<T>

Makes all properties and nested properties readonly.

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P];
};

interface Config {
  database: {
    host: string;
    port: number;
  };
}

type ReadonlyConfig = DeepReadonly<Config>;
// All nested properties are readonly
```

### Nullable<T>

Makes all properties nullable.

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
```

### Optional<T, K>

Makes specific properties optional.

```typescript
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface User {
  id: string;
  name: string;
  age: number;
  email: string;
}

type UserWithOptionalEmail = Optional<User, "email">;
// { id: string; name: string; age: number; email?: string; }
```

### RequireAtLeastOne<T>

Requires at least one property from T.

```typescript
type RequireAtLeastOne<T, Keys extends keyof T = keyof T> =
  Pick<T, Exclude<keyof T, Keys>> &
  {
    [K in Keys]-?: Required<Pick<T, K>> & Partial<Pick<T, Exclude<Keys, K>>>;
  }[Keys];

interface Filters {
  name?: string;
  age?: number;
  email?: string;
}

type FilterWithAtLeastOne = RequireAtLeastOne<Filters>;

// Valid: At least one property required
const filter1: FilterWithAtLeastOne = { name: "John" };
const filter2: FilterWithAtLeastOne = { age: 30, email: "john@example.com" };
// const filter3: FilterWithAtLeastOne = {}; // Error
```

### Mutable<T>

Removes readonly modifiers.

```typescript
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

interface ReadonlyUser {
  readonly name: string;
  readonly age: number;
}

type MutableUser = Mutable<ReadonlyUser>;
// { name: string; age: number; }
```

### PickByValue<T, V>

Picks properties by value type.

```typescript
type PickByValue<T, V> = {
  [P in keyof T as T[P] extends V ? P : never]: T[P];
};

interface User {
  id: string;
  name: string;
  age: number;
  active: boolean;
}

type StringProperties = PickByValue<User, string>;
// { id: string; name: string; }

type NumberProperties = PickByValue<User, number>;
// { age: number; }
```

### OmitByValue<T, V>

Omits properties by value type.

```typescript
type OmitByValue<T, V> = {
  [P in keyof T as T[P] extends V ? never : P]: T[P];
};

type NonStringProperties = OmitByValue<User, string>;
// { age: number; active: boolean; }
```

### PromiseType<T>

Extracts type from Promise.

```typescript
type PromiseType<T> = T extends Promise<infer U> ? U : never;

type P = PromiseType<Promise<string>>;
// string

async function fetchUser() {
  return { name: "John", age: 30 };
}

type User = PromiseType<ReturnType<typeof fetchUser>>;
// { name: string; age: number; }
```

### FunctionKeys<T>

Gets keys of properties that are functions.

```typescript
type FunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

interface User {
  name: string;
  age: number;
  greet(): void;
  sayGoodbye(): void;
}

type UserMethods = FunctionKeys<User>;
// "greet" | "sayGoodbye"
```

## Advanced Type Manipulation

### Conditional Types

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// With infer
type ArrayElementType<T> = T extends (infer U)[] ? U : never;

type C = ArrayElementType<string[]>; // string
type D = ArrayElementType<number[]>; // number
```

### Template Literal Types

```typescript
type Greeting = `Hello ${string}`;

const greeting1: Greeting = "Hello World"; // OK
// const greeting2: Greeting = "Hi there"; // Error

// Event names
type EventName = "click" | "focus" | "blur";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// CSS properties
type CSSProperty = `${"margin" | "padding"}-${"top" | "bottom" | "left" | "right"}`;
// "margin-top" | "margin-bottom" | ... | "padding-right"
```

### Mapped Type Modifiers

```typescript
// Remove readonly
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Add readonly
type Immutable<T> = {
  +readonly [P in keyof T]: T[P];
};

// Remove optional
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// Add optional
type Partial<T> = {
  [P in keyof T]+?: T[P];
};
```

## Real-World Examples

### API Response Handler

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

type ApiError = {
  error: string;
  code: number;
};

type ApiResult<T> = ApiResponse<T> | ApiError;

function isApiError(response: ApiResult<any>): response is ApiError {
  return "error" in response;
}

async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  const result: ApiResult<T> = await response.json();

  if (isApiError(result)) {
    throw new Error(result.error);
  }

  return result.data;
}
```

### Form State Management

```typescript
interface FormValues {
  username: string;
  email: string;
  age: number;
}

type FormErrors<T> = {
  [K in keyof T]?: string;
};

type FormTouched<T> = {
  [K in keyof T]?: boolean;
};

interface FormState<T> {
  values: T;
  errors: FormErrors<T>;
  touched: FormTouched<T>;
  isSubmitting: boolean;
}

// Usage
const formState: FormState<FormValues> = {
  values: { username: "", email: "", age: 0 },
  errors: { username: "Required" },
  touched: { username: true },
  isSubmitting: false
};
```

### Database Models

```typescript
interface BaseModel {
  _id: string;
  createdAt: Date;
  updatedAt: Date;
}

type CreateInput<T> = Omit<T, keyof BaseModel>;
type UpdateInput<T> = Partial<CreateInput<T>>;

interface User extends BaseModel {
  name: string;
  email: string;
  age: number;
}

// Creating a user (no _id, timestamps)
type CreateUserInput = CreateInput<User>;
// { name: string; email: string; age: number; }

// Updating a user (all fields optional)
type UpdateUserInput = UpdateInput<User>;
// { name?: string; email?: string; age?: number; }
```

## Key Takeaways

1. Utility types reduce boilerplate and improve maintainability
2. Built-in utility types cover most common use cases
3. Custom utility types can be created for specific needs
4. Conditional types enable advanced type transformations
5. Template literal types are powerful for string manipulation
6. Mapped types with modifiers provide fine-grained control
7. Type inference with `infer` extracts types from complex structures

## Practice Problems

1. Create a utility type that makes specific keys required while keeping others optional
2. Implement a type that converts all methods of a class to Promise-returning versions
3. Build a utility type that flattens nested objects
4. Create a type-safe event emitter using utility types
5. Implement a utility type that creates a union of all possible paths in an object

---

**Next Topic**: [Decorators](./decorators.md)
