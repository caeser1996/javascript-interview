# TypeScript Decorators

## Table of Contents
- [What are Decorators?](#what-are-decorators)
- [Class Decorators](#class-decorators)
- [Method Decorators](#method-decorators)
- [Property Decorators](#property-decorators)
- [Parameter Decorators](#parameter-decorators)
- [Accessor Decorators](#accessor-decorators)
- [Decorator Factories](#decorator-factories)
- [Real-World Examples](#real-world-examples)

## What are Decorators?

Decorators are a special kind of declaration that can be attached to classes, methods, properties, or parameters. They use the `@` syntax.

**Note**: Enable decorators in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

## Class Decorators

Class decorators are applied to the constructor of a class.

### Basic Class Decorator

```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

// Prevent extending the class
// Person.prototype.age = 30; // Error in strict mode
```

### Class Decorator with Metadata

```typescript
function Component(config: { selector: string; template: string }) {
  return function <T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      selector = config.selector;
      template = config.template;

      render() {
        console.log(`Rendering ${config.selector}`);
        console.log(config.template);
      }
    };
  };
}

@Component({
  selector: "app-user",
  template: "<div>User Component</div>"
})
class UserComponent {
  constructor(public name: string) {}
}

const user = new UserComponent("John");
// user.render(); // "Rendering app-user"
```

### Injectable Class Decorator (Dependency Injection)

```typescript
const Injectable = (): ClassDecorator => {
  return (target) => {
    // Register class in DI container
    console.log(`${target.name} registered for injection`);
  };
};

@Injectable()
class DatabaseService {
  connect() {
    console.log("Connected to database");
  }
}

@Injectable()
class UserService {
  constructor(private db: DatabaseService) {}

  getUsers() {
    this.db.connect();
    return ["User1", "User2"];
  }
}
```

## Method Decorators

Method decorators are applied to methods of a class.

### Basic Method Decorator

```typescript
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(5, 3);
// Logs:
// Calling add with args: [5, 3]
// add returned: 8
```

### Timing Decorator

```typescript
function measure(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = async function (...args: any[]) {
    const start = performance.now();
    const result = await originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };

  return descriptor;
}

class DataService {
  @measure
  async fetchData() {
    await new Promise(resolve => setTimeout(resolve, 1000));
    return { data: "some data" };
  }
}
```

### Retry Decorator

```typescript
function retry(maxAttempts: number, delay: number = 1000) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          if (attempt === maxAttempts) {
            throw error;
          }
          console.log(`Attempt ${attempt} failed, retrying...`);
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    };

    return descriptor;
  };
}

class ApiService {
  @retry(3, 1000)
  async fetchData(url: string) {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error("Failed to fetch");
    }
    return response.json();
  }
}
```

### Memoize Decorator

```typescript
function memoize(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  const cache = new Map();

  descriptor.value = function (...args: any[]) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log("Returning cached result");
      return cache.get(key);
    }

    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };

  return descriptor;
}

class MathOperations {
  @memoize
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}

const math = new MathOperations();
console.log(math.fibonacci(10)); // Calculated
console.log(math.fibonacci(10)); // Cached
```

## Property Decorators

Property decorators are applied to class properties.

### Basic Property Decorator

```typescript
function readonly(target: any, propertyKey: string) {
  Object.defineProperty(target, propertyKey, {
    writable: false,
    configurable: false
  });
}

class User {
  @readonly
  role: string = "admin";
}

const user = new User();
// user.role = "user"; // Error: Cannot assign to read only property
```

### Validation Decorator

```typescript
function min(limit: number) {
  return function (target: any, propertyKey: string) {
    let value: number;

    const getter = function () {
      return value;
    };

    const setter = function (newVal: number) {
      if (newVal < limit) {
        throw new Error(`${propertyKey} must be at least ${limit}`);
      }
      value = newVal;
    };

    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

class Product {
  @min(0)
  price: number;

  constructor(price: number) {
    this.price = price;
  }
}

const product = new Product(100); // OK
// const invalidProduct = new Product(-10); // Error
```

### Required Property Decorator

```typescript
function required(target: any, propertyKey: string) {
  let value: any;

  const getter = function () {
    return value;
  };

  const setter = function (newVal: any) {
    if (newVal === undefined || newVal === null) {
      throw new Error(`${propertyKey} is required`);
    }
    value = newVal;
  };

  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter,
    enumerable: true,
    configurable: true
  });
}

class User {
  @required
  email: string;

  constructor(email: string) {
    this.email = email;
  }
}
```

## Parameter Decorators

Parameter decorators are applied to function parameters.

### Basic Parameter Decorator

```typescript
function logParameter(
  target: any,
  propertyKey: string,
  parameterIndex: number
) {
  const existingParameters: number[] =
    Reflect.getOwnMetadata("log_parameters", target, propertyKey) || [];

  existingParameters.push(parameterIndex);

  Reflect.defineMetadata(
    "log_parameters",
    existingParameters,
    target,
    propertyKey
  );
}

class UserService {
  greet(
    @logParameter name: string,
    @logParameter age: number
  ): string {
    return `Hello ${name}, you are ${age} years old`;
  }
}
```

### Required Parameter Decorator

```typescript
function required(
  target: any,
  propertyKey: string,
  parameterIndex: number
) {
  const existingRequiredParameters: number[] =
    Reflect.getOwnMetadata("required_parameters", target, propertyKey) || [];

  existingRequiredParameters.push(parameterIndex);

  Reflect.defineMetadata(
    "required_parameters",
    existingRequiredParameters,
    target,
    propertyKey
  );
}

function validate(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    const requiredParameters: number[] =
      Reflect.getOwnMetadata("required_parameters", target, propertyKey) || [];

    for (const index of requiredParameters) {
      if (args[index] === undefined || args[index] === null) {
        throw new Error(`Parameter at index ${index} is required`);
      }
    }

    return originalMethod.apply(this, args);
  };
}

class UserController {
  @validate
  createUser(
    @required name: string,
    @required email: string,
    age?: number
  ) {
    return { name, email, age };
  }
}
```

## Accessor Decorators

Accessor decorators are applied to getters and setters.

### Basic Accessor Decorator

```typescript
function configurable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.configurable = value;
  };
}

class Point {
  private _x: number = 0;
  private _y: number = 0;

  @configurable(false)
  get x() {
    return this._x;
  }

  set x(value: number) {
    this._x = value;
  }

  @configurable(false)
  get y() {
    return this._y;
  }

  set y(value: number) {
    this._y = value;
  }
}
```

### Format Accessor Decorator

```typescript
function format(formatString: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalGet = descriptor.get;

    descriptor.get = function () {
      const value = originalGet?.call(this);
      return formatString.replace("{0}", value);
    };
  };
}

class Person {
  private _name: string = "";

  @format("Name: {0}")
  get name() {
    return this._name;
  }

  set name(value: string) {
    this._name = value;
  }
}

const person = new Person();
person.name = "John";
console.log(person.name); // "Name: John"
```

## Decorator Factories

Decorator factories are functions that return decorators.

### Parameterized Decorators

```typescript
function Component(options: { selector: string; template: string }) {
  return function <T extends { new(...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      selector = options.selector;
      template = options.template;
    };
  };
}

@Component({
  selector: "app-header",
  template: "<header>Header Content</header>"
})
class HeaderComponent {}
```

### Conditional Decorator

```typescript
function deprecate(message: string, condition: boolean = true) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    if (!condition) return descriptor;

    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      console.warn(`DEPRECATED: ${message}`);
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class LegacyAPI {
  @deprecate("Use newMethod() instead", true)
  oldMethod() {
    console.log("Old method executed");
  }

  newMethod() {
    console.log("New method executed");
  }
}
```

## Real-World Examples

### Authorization Decorator

```typescript
enum Role {
  Admin = "admin",
  User = "user",
  Guest = "guest"
}

function authorize(...roles: Role[]) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      // Assume we have a way to get current user
      const currentUserRole = getCurrentUserRole();

      if (!roles.includes(currentUserRole)) {
        throw new Error("Unauthorized access");
      }

      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

function getCurrentUserRole(): Role {
  // Implementation to get current user role
  return Role.User;
}

class AdminPanel {
  @authorize(Role.Admin)
  deleteUser(userId: string) {
    console.log(`Deleting user ${userId}`);
  }

  @authorize(Role.Admin, Role.User)
  viewDashboard() {
    console.log("Viewing dashboard");
  }
}
```

### Rate Limiter Decorator

```typescript
function rateLimit(maxCalls: number, timeWindow: number) {
  const calls: number[] = [];

  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const now = Date.now();

      // Remove old calls outside time window
      while (calls.length > 0 && calls[0] < now - timeWindow) {
        calls.shift();
      }

      if (calls.length >= maxCalls) {
        throw new Error("Rate limit exceeded");
      }

      calls.push(now);
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class ApiController {
  @rateLimit(5, 60000) // 5 calls per minute
  async handleRequest(req: any) {
    return { status: "success" };
  }
}
```

### Validation Decorator

```typescript
function validate(schema: any) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const [data] = args;

      // Simple validation (in real app, use Joi, Yup, etc.)
      for (const [key, rules] of Object.entries(schema)) {
        const value = data[key];
        const ruleSet = rules as any;

        if (ruleSet.required && !value) {
          throw new Error(`${key} is required`);
        }

        if (ruleSet.type && typeof value !== ruleSet.type) {
          throw new Error(`${key} must be of type ${ruleSet.type}`);
        }

        if (ruleSet.min && value < ruleSet.min) {
          throw new Error(`${key} must be at least ${ruleSet.min}`);
        }
      }

      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class UserController {
  @validate({
    name: { required: true, type: "string" },
    age: { required: true, type: "number", min: 0 },
    email: { required: true, type: "string" }
  })
  createUser(data: any) {
    return { id: 1, ...data };
  }
}

const controller = new UserController();

try {
  controller.createUser({ name: "John", age: 30, email: "john@example.com" });
  // controller.createUser({ name: "John" }); // Error: age is required
} catch (error) {
  console.error(error);
}
```

### Cache Decorator with TTL

```typescript
function cache(ttl: number = 60000) {
  const cacheStore = new Map<string, { value: any; timestamp: number }>();

  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const key = `${propertyKey}-${JSON.stringify(args)}`;
      const cached = cacheStore.get(key);

      if (cached && Date.now() - cached.timestamp < ttl) {
        console.log("Returning cached result");
        return cached.value;
      }

      const result = await originalMethod.apply(this, args);
      cacheStore.set(key, { value: result, timestamp: Date.now() });

      return result;
    };

    return descriptor;
  };
}

class DataService {
  @cache(5000) // Cache for 5 seconds
  async fetchUserData(userId: string) {
    console.log(`Fetching data for user ${userId}`);
    await new Promise(resolve => setTimeout(resolve, 1000));
    return { userId, name: "John", age: 30 };
  }
}
```

### Debounce Decorator

```typescript
function debounce(delay: number) {
  let timeoutId: NodeJS.Timeout;

  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      clearTimeout(timeoutId);

      return new Promise((resolve) => {
        timeoutId = setTimeout(() => {
          resolve(originalMethod.apply(this, args));
        }, delay);
      });
    };

    return descriptor;
  };
}

class SearchComponent {
  @debounce(300)
  async search(query: string) {
    console.log(`Searching for: ${query}`);
    // API call here
    return [`Result 1 for ${query}`, `Result 2 for ${query}`];
  }
}
```

## Decorator Composition

Multiple decorators can be composed together.

```typescript
class UserController {
  @authorize(Role.Admin)
  @rateLimit(10, 60000)
  @measure
  @log
  async deleteUser(userId: string) {
    console.log(`Deleting user ${userId}`);
    return { success: true };
  }
}

// Execution order: bottom to top
// 1. @log
// 2. @measure
// 3. @rateLimit
// 4. @authorize
```

## Best Practices

1. **Keep decorators focused**: Each decorator should do one thing
2. **Use decorator factories for configuration**: Allow customization
3. **Preserve metadata**: Use `Reflect.metadata` when needed
4. **Type safety**: Ensure proper typing for decorator parameters
5. **Performance**: Be mindful of overhead in frequently called methods
6. **Error handling**: Handle errors gracefully in decorators
7. **Documentation**: Document decorator behavior clearly

## Key Takeaways

1. Decorators modify behavior without changing code
2. Five types: Class, Method, Property, Parameter, Accessor
3. Decorators execute at class definition time
4. Multiple decorators compose bottom-to-top
5. Decorator factories enable parameterization
6. Use for cross-cutting concerns (logging, caching, auth)
7. Enable with `experimentalDecorators` in tsconfig.json

## Practice Problems

1. Create a `@timeout` decorator that throws if method takes too long
2. Implement a `@singleton` class decorator
3. Build a `@validate` decorator using a schema library
4. Create a `@transaction` decorator for database operations
5. Implement a `@bind` decorator to bind class methods to instance

---

**Next Section**: [MongoDB](../03-MongoDB/)
