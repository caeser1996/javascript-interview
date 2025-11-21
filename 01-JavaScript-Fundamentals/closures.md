# Closures in JavaScript

## What is a Closure?

A closure is a function that has access to variables in its outer (enclosing) lexical scope, even after the outer function has returned.

**Simple Definition**: A closure gives you access to an outer function's scope from an inner function.

## Why Closures Matter

- Data privacy and encapsulation
- Factory functions
- Callbacks and event handlers
- Maintaining state in async operations

## Basic Example

```javascript
function outer() {
  const message = "Hello";

  function inner() {
    console.log(message); // Can access outer's variable
  }

  return inner;
}

const myFunc = outer();
myFunc(); // Output: "Hello"
```

## Common Interview Questions

### Q1: What will this code output?

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
```

**Answer**: `3 3 3`

**Explanation**: `var` is function-scoped, not block-scoped. By the time setTimeout executes, the loop has finished and `i` is 3.

**Solutions**:

```javascript
// Solution 1: Use let (block-scoped)
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
// Output: 0 1 2

// Solution 2: Use IIFE (Immediately Invoked Function Expression)
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);
    }, 1000);
  })(i);
}
// Output: 0 1 2

// Solution 3: Pass parameter to setTimeout
for (var i = 0; i < 3; i++) {
  setTimeout(function(j) {
    console.log(j);
  }, 1000, i);
}
// Output: 0 1 2
```

### Q2: Create a Counter Function

```javascript
function createCounter() {
  let count = 0;

  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount());  // 1
console.log(counter.count);       // undefined (private variable)
```

### Q3: Function Currying with Closures

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...nextArgs) {
        return curried.apply(this, args.concat(nextArgs));
      };
    }
  };
}

// Usage
function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3));     // 6
console.log(curriedAdd(1, 2)(3));     // 6
console.log(curriedAdd(1)(2, 3));     // 6
```

### Q4: Private Variables

```javascript
function bankAccount(initialBalance) {
  let balance = initialBalance;

  return {
    deposit: function(amount) {
      balance += amount;
      return balance;
    },
    withdraw: function(amount) {
      if (amount > balance) {
        return "Insufficient funds";
      }
      balance -= amount;
      return balance;
    },
    getBalance: function() {
      return balance;
    }
  };
}

const account = bankAccount(100);
console.log(account.deposit(50));    // 150
console.log(account.withdraw(30));   // 120
console.log(account.getBalance());   // 120
console.log(account.balance);        // undefined
```

### Q5: Once Function (Function that runs only once)

```javascript
function once(fn) {
  let called = false;
  let result;

  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

// Usage
const initialize = once(() => {
  console.log("Initialized!");
  return "Done";
});

initialize(); // "Initialized!" -> "Done"
initialize(); // Returns "Done" without logging
initialize(); // Returns "Done" without logging
```

### Q6: Memoization with Closures

```javascript
function memoize(fn) {
  const cache = {};

  return function(...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      console.log("Fetching from cache");
      return cache[key];
    }
    console.log("Calculating result");
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

// Usage
const fibonacci = memoize(function(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // Calculating result
console.log(fibonacci(10)); // Fetching from cache
```

## Common Pitfalls

### 1. Memory Leaks

```javascript
// ❌ Bad: Creates memory leak
function createHeavyObject() {
  const heavyData = new Array(1000000).fill('data');

  return function() {
    // Even if we don't use heavyData, it's kept in memory
    console.log('Function executed');
  };
}

// ✅ Good: Clean up when not needed
function createHeavyObject() {
  let heavyData = new Array(1000000).fill('data');

  return function() {
    if (heavyData) {
      // Use the data
      console.log(heavyData.length);
      // Clean up
      heavyData = null;
    }
  };
}
```

### 2. Closure in Loops (Already covered above)

### 3. Accidental Global Variables

```javascript
function createCounter() {
  count = 0; // ❌ Forgot 'let/const', creates global variable

  return function() {
    return ++count;
  };
}
```

## Real-World Use Cases

### 1. Event Handlers

```javascript
function setupButton(buttonId) {
  const button = document.getElementById(buttonId);
  let clickCount = 0;

  button.addEventListener('click', function() {
    clickCount++;
    console.log(`Button clicked ${clickCount} times`);
  });
}
```

### 2. Module Pattern

```javascript
const Calculator = (function() {
  let result = 0;

  return {
    add: (x) => { result += x; return result; },
    subtract: (x) => { result -= x; return result; },
    multiply: (x) => { result *= x; return result; },
    divide: (x) => { result /= x; return result; },
    reset: () => { result = 0; return result; },
    getResult: () => result
  };
})();

Calculator.add(10);
Calculator.multiply(2);
console.log(Calculator.getResult()); // 20
```

### 3. Debounce Function

```javascript
function debounce(fn, delay) {
  let timeoutId;

  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// Usage
const searchAPI = debounce((query) => {
  console.log('Searching for:', query);
}, 500);

searchAPI('react');
searchAPI('react hooks'); // Only this will execute after 500ms
```

## Key Takeaways

1. Closures allow functions to access variables from their outer scope
2. They're created every time a function is created
3. Useful for data privacy, callbacks, and maintaining state
4. Be aware of memory implications
5. Understand the difference between `var`, `let`, and `const` in closures

## Practice Problems

1. Create a `createSecret()` function that stores a secret and has methods to check if a guess is correct
2. Implement a `throttle()` function similar to debounce
3. Create a function that generates unique IDs using closures
4. Implement a simple event emitter using closures

---

**Next Topic**: [Promises & Async/Await](./promises-async.md)
