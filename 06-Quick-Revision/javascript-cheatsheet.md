# JavaScript Quick Revision Cheatsheet

## Data Types

```javascript
// Primitives
typeof "hello"      // "string"
typeof 42           // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof Symbol()     // "symbol"
typeof 10n          // "bigint"
typeof null         // "object" (historical bug)

// Reference types
typeof {}           // "object"
typeof []           // "object"
typeof function(){} // "function"
```

## Variable Declarations

```javascript
var x = 1;    // Function-scoped, hoisted
let y = 2;    // Block-scoped, not hoisted (TDZ)
const z = 3;  // Block-scoped, immutable binding
```

## Arrow Functions

```javascript
// Syntax
const add = (a, b) => a + b;
const double = x => x * 2;
const log = () => console.log('hi');

// No own 'this', 'arguments', or 'super'
const obj = {
  name: 'John',
  greet: () => console.log(this.name) // undefined
};
```

## Destructuring

```javascript
// Array
const [a, b, ...rest] = [1, 2, 3, 4, 5];

// Object
const { name, age } = { name: 'John', age: 30 };
const { name: n } = { name: 'John' }; // Rename

// Nested
const { user: { name } } = { user: { name: 'John' } };
```

## Spread & Rest

```javascript
// Spread
const arr = [1, 2, 3];
const newArr = [...arr, 4, 5];
const obj = { a: 1, ...{ b: 2 } };

// Rest
function sum(...nums) {
  return nums.reduce((a, b) => a + b);
}
```

## Template Literals

```javascript
const name = 'John';
const greeting = `Hello, ${name}!`;
const multiline = `Line 1
Line 2`;
```

## Array Methods

```javascript
// map - Transform each element
[1, 2, 3].map(x => x * 2); // [2, 4, 6]

// filter - Keep elements that pass test
[1, 2, 3, 4].filter(x => x % 2 === 0); // [2, 4]

// reduce - Reduce to single value
[1, 2, 3].reduce((sum, x) => sum + x, 0); // 6

// forEach - Execute function for each
[1, 2, 3].forEach(x => console.log(x));

// find - First element that passes test
[1, 2, 3].find(x => x > 1); // 2

// some - At least one passes test
[1, 2, 3].some(x => x > 2); // true

// every - All pass test
[1, 2, 3].every(x => x > 0); // true

// includes - Contains element
[1, 2, 3].includes(2); // true

// slice - Extract portion (non-destructive)
[1, 2, 3, 4].slice(1, 3); // [2, 3]

// splice - Add/remove elements (destructive)
const arr = [1, 2, 3, 4];
arr.splice(1, 2, 'a', 'b'); // Removes [2, 3], adds 'a', 'b'

// sort - Sort array (in-place)
[3, 1, 2].sort((a, b) => a - b); // [1, 2, 3]
```

## String Methods

```javascript
'hello'.toUpperCase();        // 'HELLO'
'HELLO'.toLowerCase();        // 'hello'
'  hello  '.trim();           // 'hello'
'hello'.substring(1, 4);      // 'ell'
'hello'.slice(-2);            // 'lo'
'hello'.charAt(0);            // 'h'
'hello world'.split(' ');     // ['hello', 'world']
'hello'.repeat(3);            // 'hellohellohello'
'hello'.includes('ll');       // true
'hello'.startsWith('he');     // true
'hello'.endsWith('lo');       // true
'hello'.replace('l', 'x');    // 'hexlo'
'hello'.replaceAll('l', 'x'); // 'hexxo'
```

## Object Methods

```javascript
Object.keys({ a: 1, b: 2 });    // ['a', 'b']
Object.values({ a: 1, b: 2 });  // [1, 2]
Object.entries({ a: 1, b: 2 }); // [['a', 1], ['b', 2]]

Object.assign({}, { a: 1 }, { b: 2 }); // { a: 1, b: 2 }
Object.freeze(obj);   // Prevent modifications
Object.seal(obj);     // Prevent add/delete properties

obj.hasOwnProperty('key');
'key' in obj;
```

## Promises

```javascript
// Create promise
const promise = new Promise((resolve, reject) => {
  if (success) resolve(value);
  else reject(error);
});

// Consume promise
promise
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));

// Promise methods
Promise.all([p1, p2, p3]);        // All fulfill or any rejects
Promise.allSettled([p1, p2, p3]); // All settle
Promise.race([p1, p2, p3]);       // First to settle
Promise.any([p1, p2, p3]);        // First to fulfill
```

## Async/Await

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error(error);
  }
}

// Parallel execution
const [data1, data2] = await Promise.all([
  fetchData1(),
  fetchData2()
]);
```

## Closures

```javascript
function outer() {
  const x = 10;
  return function inner() {
    console.log(x); // Has access to x
  };
}

const fn = outer();
fn(); // 10
```

## Prototype Chain

```javascript
// Every object has __proto__
obj.__proto__ === Object.prototype; // true

// Constructor functions have prototype
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() {
  console.log(`Hi, I'm ${this.name}`);
};

const john = new Person('John');
john.greet(); // "Hi, I'm John"
```

## Classes

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`Hi, I'm ${this.name}`);
  }

  static species() {
    return 'Homo sapiens';
  }
}

class Employee extends Person {
  constructor(name, role) {
    super(name);
    this.role = role;
  }
}
```

## Event Loop

```javascript
console.log('1');                  // Synchronous
setTimeout(() => console.log('2'), 0); // Macrotask
Promise.resolve().then(() => console.log('3')); // Microtask
console.log('4');                  // Synchronous

// Output: 1 4 3 2
// Order: Sync → Microtasks → Macrotasks
```

## Common Patterns

### Debounce
```javascript
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

### Throttle
```javascript
function throttle(fn, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```

### Memoization
```javascript
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (key in cache) return cache[key];
    cache[key] = fn.apply(this, args);
    return cache[key];
  };
}
```

### Deep Clone
```javascript
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [key, deepClone(value)])
  );
}
```

## Common Interview Questions

### Q: `var` vs `let` vs `const`?
- `var`: Function-scoped, hoisted
- `let`: Block-scoped, TDZ
- `const`: Block-scoped, immutable binding

### Q: `==` vs `===`?
- `==`: Type coercion
- `===`: Strict equality (no coercion)

### Q: What is hoisting?
Variable and function declarations are moved to top of scope during compilation.

### Q: What is a closure?
Function that has access to outer function's variables even after outer function returns.

### Q: What is `this`?
Refers to execution context. In arrow functions, lexically bound.

### Q: Difference between `.call()`, `.apply()`, `.bind()`?
- `call(thisArg, arg1, arg2)`: Invoke immediately with args
- `apply(thisArg, [args])`: Invoke immediately with array
- `bind(thisArg)`: Returns new function with bound this

### Q: What is event bubbling?
Events propagate from target element up to root.

### Q: What is Promise?
Object representing eventual completion/failure of async operation.

### Q: Microtask vs Macrotask?
- Microtask: Promise callbacks, queueMicrotask
- Macrotask: setTimeout, setInterval, I/O
- Microtasks run before next macrotask

## Key Takeaways

✅ Use `const` by default, `let` when needed, avoid `var`
✅ Arrow functions don't have own `this`
✅ Promises > Callbacks, Async/Await > Promises
✅ Understand event loop and execution order
✅ Master array methods (map, filter, reduce)
✅ Know difference between shallow vs deep copy
✅ Understand closures and lexical scoping
✅ Use strict equality (`===`) unless type coercion needed
