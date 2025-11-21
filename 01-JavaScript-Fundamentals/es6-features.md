# ES6+ Modern JavaScript Features

## Table of Contents
- [Let and Const](#let-and-const)
- [Arrow Functions](#arrow-functions)
- [Destructuring](#destructuring)
- [Spread and Rest Operators](#spread-and-rest-operators)
- [Template Literals](#template-literals)
- [Modules](#modules)
- [Default Parameters](#default-parameters)
- [Enhanced Object Literals](#enhanced-object-literals)
- [Iterators and Generators](#iterators-and-generators)
- [Map and Set](#map-and-set)
- [Symbols](#symbols)
- [Optional Chaining and Nullish Coalescing](#optional-chaining-and-nullish-coalescing)

## Let and Const

### var vs let vs const

```javascript
// var: function-scoped, hoisted
var x = 1;
if (true) {
  var x = 2; // Same variable
  console.log(x); // 2
}
console.log(x); // 2

// let: block-scoped, not hoisted
let y = 1;
if (true) {
  let y = 2; // Different variable
  console.log(y); // 2
}
console.log(y); // 1

// const: block-scoped, cannot be reassigned
const z = 1;
// z = 2; // Error: Assignment to constant variable

// But const objects/arrays can be mutated
const obj = { name: 'John' };
obj.name = 'Jane'; // OK
obj.age = 30;      // OK
// obj = {};       // Error
```

### Temporal Dead Zone (TDZ)

```javascript
// var: undefined before declaration
console.log(a); // undefined
var a = 5;

// let/const: ReferenceError in TDZ
// console.log(b); // ReferenceError
let b = 5;

// const must be initialized
// const c; // SyntaxError
const c = 10;
```

## Arrow Functions

### Syntax

```javascript
// Traditional function
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// With single parameter (parentheses optional)
const double = x => x * 2;

// No parameters
const getRandom = () => Math.random();

// Multiple statements
const greet = (name) => {
  const message = `Hello, ${name}!`;
  return message;
};
```

### Key Differences from Regular Functions

```javascript
// 1. No 'this' binding - uses lexical this
const person = {
  name: 'John',
  regularFunc: function() {
    console.log(this.name); // 'John'
  },
  arrowFunc: () => {
    console.log(this.name); // undefined (or global)
  }
};

// 2. Useful for callbacks
const numbers = [1, 2, 3, 4, 5];

// Regular function (need to bind 'this')
function Multiplier() {
  this.factor = 2;
  this.multiply = function(arr) {
    return arr.map(function(num) {
      return num * this.factor; // 'this' is undefined
    }.bind(this));
  };
}

// Arrow function (lexical 'this')
function Multiplier() {
  this.factor = 2;
  this.multiply = function(arr) {
    return arr.map(num => num * this.factor); // Works!
  };
}

// 3. No arguments object
const regularFunc = function() {
  console.log(arguments);
};
regularFunc(1, 2, 3); // [1, 2, 3]

const arrowFunc = (...args) => {
  console.log(args);
};
arrowFunc(1, 2, 3); // [1, 2, 3]

// 4. Cannot be used as constructors
const Person = (name) => {
  this.name = name;
};
// new Person('John'); // TypeError
```

## Destructuring

### Array Destructuring

```javascript
const numbers = [1, 2, 3, 4, 5];

// Basic
const [first, second] = numbers;
console.log(first, second); // 1, 2

// Skip elements
const [a, , c] = numbers;
console.log(a, c); // 1, 3

// Rest pattern
const [head, ...tail] = numbers;
console.log(head, tail); // 1, [2, 3, 4, 5]

// Default values
const [x, y, z = 10] = [1, 2];
console.log(x, y, z); // 1, 2, 10

// Swapping variables
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q); // 2, 1
```

### Object Destructuring

```javascript
const person = {
  name: 'John',
  age: 30,
  city: 'New York'
};

// Basic
const { name, age } = person;
console.log(name, age); // 'John', 30

// Rename variables
const { name: personName, age: personAge } = person;
console.log(personName, personAge); // 'John', 30

// Default values
const { name, country = 'USA' } = person;
console.log(name, country); // 'John', 'USA'

// Nested destructuring
const user = {
  id: 1,
  info: {
    name: 'John',
    address: {
      city: 'New York',
      zip: '10001'
    }
  }
};

const { info: { name, address: { city } } } = user;
console.log(name, city); // 'John', 'New York'

// Function parameters
function greet({ name, age }) {
  console.log(`${name} is ${age} years old`);
}
greet(person); // 'John is 30 years old'
```

## Spread and Rest Operators

### Spread Operator (...)

```javascript
// Arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// Copy array
const copy = [...arr1];

// Math functions
const numbers = [5, 6, 2, 3, 7];
console.log(Math.max(...numbers)); // 7

// Objects
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// Overriding properties
const obj = { a: 1, b: 2 };
const updated = { ...obj, b: 3, c: 4 };
console.log(updated); // { a: 1, b: 3, c: 4 }

// Shallow copy
const original = { a: 1, nested: { b: 2 } };
const clone = { ...original };
clone.nested.b = 3;
console.log(original.nested.b); // 3 (shallow copy!)
```

### Rest Operator

```javascript
// Functions
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}
console.log(sum(1, 2, 3, 4)); // 10

// Destructuring
const [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first, rest); // 1, [2, 3, 4, 5]

const { a, ...others } = { a: 1, b: 2, c: 3 };
console.log(a, others); // 1, { b: 2, c: 3 }
```

## Template Literals

```javascript
const name = 'John';
const age = 30;

// Basic interpolation
const message = `Hello, ${name}!`;
console.log(message); // 'Hello, John!'

// Multi-line strings
const multiline = `
  This is
  a multi-line
  string
`;

// Expressions
const total = `Total: ${10 + 20}`; // 'Total: 30'

// Tagged templates
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const userName = 'John';
const userAge = 30;
const html = highlight`User ${userName} is ${userAge} years old`;
// 'User <mark>John</mark> is <mark>30</mark> years old'
```

## Modules

### Export

```javascript
// Named exports (math.js)
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export class Calculator {
  multiply(a, b) {
    return a * b;
  }
}

// Or export at the end
const subtract = (a, b) => a - b;
const divide = (a, b) => a / b;
export { subtract, divide };

// Default export (only one per module)
export default function square(x) {
  return x * x;
}
```

### Import

```javascript
// Named imports
import { PI, add, Calculator } from './math.js';

// Rename imports
import { add as sum } from './math.js';

// Import all
import * as math from './math.js';
console.log(math.PI);
console.log(math.add(2, 3));

// Default import
import square from './math.js';

// Mix default and named
import square, { PI, add } from './math.js';
```

## Default Parameters

```javascript
// Old way
function greetOld(name, message) {
  message = message || 'Hello';
  return `${message}, ${name}!`;
}

// ES6 way
function greet(name, message = 'Hello') {
  return `${message}, ${name}!`;
}

console.log(greet('John')); // 'Hello, John!'
console.log(greet('John', 'Hi')); // 'Hi, John!'

// With destructuring
function createUser({ name, age = 18, role = 'user' } = {}) {
  return { name, age, role };
}

console.log(createUser({ name: 'John' }));
// { name: 'John', age: 18, role: 'user' }
```

## Enhanced Object Literals

```javascript
const name = 'John';
const age = 30;

// Property shorthand
const person = {
  name,    // same as name: name
  age      // same as age: age
};

// Method shorthand
const obj = {
  // Old way
  sayHello: function() {
    console.log('Hello');
  },

  // ES6 way
  sayHi() {
    console.log('Hi');
  }
};

// Computed property names
const prop = 'age';
const user = {
  name: 'John',
  [prop]: 30,
  ['is' + 'Admin']: false
};
console.log(user); // { name: 'John', age: 30, isAdmin: false }
```

## Iterators and Generators

### For...of Loop

```javascript
const arr = [1, 2, 3, 4, 5];

// for...of (values)
for (const value of arr) {
  console.log(value);
}

// for...in (keys/indices)
for (const index in arr) {
  console.log(index); // '0', '1', '2', '3', '4'
}

// Works with strings
for (const char of 'hello') {
  console.log(char);
}

// With Map
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(key, value);
}
```

### Generators

```javascript
// Basic generator
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Infinite sequence
function* fibonacci() {
  let [prev, curr] = [0, 1];
  while (true) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

const fib = fibonacci();
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
console.log(fib.next().value); // 3
console.log(fib.next().value); // 5

// ID generator
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const getId = idGenerator();
console.log(getId.next().value); // 1
console.log(getId.next().value); // 2
```

## Map and Set

### Map

```javascript
// Creating a Map
const map = new Map();

// Setting values
map.set('name', 'John');
map.set('age', 30);
map.set(1, 'number key');
map.set({}, 'object key');

// Getting values
console.log(map.get('name')); // 'John'

// Check existence
console.log(map.has('name')); // true

// Size
console.log(map.size); // 4

// Delete
map.delete('age');

// Iterate
for (const [key, value] of map) {
  console.log(key, value);
}

// Convert to array
const arr = Array.from(map);
const entries = [...map.entries()];
const keys = [...map.keys()];
const values = [...map.values()];
```

### Set

```javascript
// Creating a Set
const set = new Set();

// Adding values
set.add(1);
set.add(2);
set.add(2); // Ignored (duplicates not allowed)
set.add('hello');

console.log(set.size); // 3

// Check existence
console.log(set.has(1)); // true

// Delete
set.delete(2);

// Iterate
for (const value of set) {
  console.log(value);
}

// Convert to array
const arr = Array.from(set);
const spread = [...set];

// Remove duplicates from array
const numbers = [1, 2, 2, 3, 3, 4, 5, 5];
const unique = [...new Set(numbers)];
console.log(unique); // [1, 2, 3, 4, 5]
```

## Symbols

```javascript
// Create unique identifiers
const sym1 = Symbol('description');
const sym2 = Symbol('description');

console.log(sym1 === sym2); // false (each Symbol is unique)

// Use as object property
const id = Symbol('id');
const user = {
  name: 'John',
  [id]: 123
};

console.log(user[id]); // 123
console.log(Object.keys(user)); // ['name'] (Symbol not included)

// Well-known Symbols
const arr = [1, 2, 3];
console.log(arr[Symbol.iterator]); // [Function]

// Global Symbol registry
const globalSym1 = Symbol.for('app.id');
const globalSym2 = Symbol.for('app.id');
console.log(globalSym1 === globalSym2); // true
```

## Optional Chaining and Nullish Coalescing

### Optional Chaining (?.)

```javascript
const user = {
  name: 'John',
  address: {
    city: 'New York'
  }
};

// Without optional chaining
const zipOld = user.address && user.address.zip;

// With optional chaining
const zip = user.address?.zip; // undefined (no error)

// Function calls
const result = obj.method?.();

// Array access
const item = arr?.[index];

// Real-world example
const city = user?.address?.city ?? 'Unknown';
console.log(city); // 'New York'

const country = user?.address?.country ?? 'USA';
console.log(country); // 'USA'
```

### Nullish Coalescing (??)

```javascript
// Returns right side if left is null or undefined
const value1 = null ?? 'default'; // 'default'
const value2 = undefined ?? 'default'; // 'default'
const value3 = 0 ?? 'default'; // 0 (not null/undefined)
const value4 = '' ?? 'default'; // '' (not null/undefined)
const value5 = false ?? 'default'; // false (not null/undefined)

// vs || operator
const a = 0 || 10; // 10 (0 is falsy)
const b = 0 ?? 10; // 0 (0 is not null/undefined)

const c = '' || 'default'; // 'default' ('' is falsy)
const d = '' ?? 'default'; // '' ('' is not null/undefined)

// Real use case
function greet(name) {
  const displayName = name ?? 'Guest';
  console.log(`Hello, ${displayName}!`);
}

greet('John'); // 'Hello, John!'
greet(null);   // 'Hello, Guest!'
greet('');     // 'Hello, !' (empty string is valid)
```

## Key Takeaways

1. Use `const` by default, `let` when reassignment is needed, avoid `var`
2. Arrow functions have lexical `this` binding
3. Destructuring makes code cleaner and more readable
4. Spread operator creates shallow copies
5. Template literals improve string formatting
6. Modules enable better code organization
7. `Map` and `Set` are useful for unique keys and values
8. Optional chaining prevents errors with nested properties
9. Nullish coalescing provides better default value handling
10. Generators enable lazy evaluation and infinite sequences

## Practice Problems

1. Convert old JavaScript code to ES6+
2. Implement a deep clone using spread operator and recursion
3. Create a custom iterator for an object
4. Build a unique ID generator using generators
5. Refactor callback-based code to use arrow functions

---

**Next Section**: [TypeScript Concepts](../02-TypeScript-Concepts/)
