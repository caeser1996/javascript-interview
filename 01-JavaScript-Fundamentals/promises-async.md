# Promises and Async/Await in JavaScript

## Table of Contents
- [Promises](#promises)
- [Async/Await](#asyncawait)
- [Common Interview Questions](#common-interview-questions)
- [Error Handling](#error-handling)
- [Advanced Patterns](#advanced-patterns)

## Promises

### What is a Promise?

A Promise is an object representing the eventual completion or failure of an asynchronous operation.

**States**:
- **Pending**: Initial state, neither fulfilled nor rejected
- **Fulfilled**: Operation completed successfully
- **Rejected**: Operation failed

### Basic Promise Example

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve("Operation successful!");
    } else {
      reject("Operation failed!");
    }
  }, 1000);
});

promise
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### Promise Methods

#### Promise.all()
Waits for all promises to resolve or any to reject.

```javascript
const promise1 = Promise.resolve(3);
const promise2 = new Promise(resolve => setTimeout(() => resolve(42), 100));
const promise3 = Promise.resolve("foo");

Promise.all([promise1, promise2, promise3])
  .then(values => console.log(values)); // [3, 42, "foo"]

// If any promise rejects, Promise.all() rejects immediately
const p1 = Promise.resolve(1);
const p2 = Promise.reject("Error!");
const p3 = Promise.resolve(3);

Promise.all([p1, p2, p3])
  .catch(error => console.log(error)); // "Error!"
```

#### Promise.allSettled()
Waits for all promises to settle (either fulfilled or rejected).

```javascript
const promises = [
  Promise.resolve(1),
  Promise.reject("Error"),
  Promise.resolve(3)
];

Promise.allSettled(promises).then(results => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: 1 },
  //   { status: 'rejected', reason: 'Error' },
  //   { status: 'fulfilled', value: 3 }
  // ]
});
```

#### Promise.race()
Returns the first promise that settles (fulfills or rejects).

```javascript
const promise1 = new Promise(resolve => setTimeout(() => resolve('one'), 500));
const promise2 = new Promise(resolve => setTimeout(() => resolve('two'), 100));

Promise.race([promise1, promise2])
  .then(value => console.log(value)); // "two"
```

#### Promise.any()
Returns the first promise that fulfills, ignores rejections.

```javascript
const promise1 = Promise.reject('Error 1');
const promise2 = new Promise(resolve => setTimeout(() => resolve('Success'), 100));
const promise3 = Promise.reject('Error 2');

Promise.any([promise1, promise2, promise3])
  .then(value => console.log(value)); // "Success"
```

## Async/Await

### What is Async/Await?

Async/await is syntactic sugar built on top of Promises, making asynchronous code look and behave more like synchronous code.

### Basic Example

```javascript
// Using Promises
function fetchUserData() {
  return fetch('https://api.example.com/user')
    .then(response => response.json())
    .then(data => {
      console.log(data);
      return data;
    })
    .catch(error => console.error(error));
}

// Using Async/Await
async function fetchUserData() {
  try {
    const response = await fetch('https://api.example.com/user');
    const data = await response.json();
    console.log(data);
    return data;
  } catch (error) {
    console.error(error);
  }
}
```

### Key Points

1. `async` functions always return a Promise
2. `await` can only be used inside `async` functions
3. `await` pauses execution until the Promise resolves

```javascript
async function example() {
  return "Hello";
}

// Same as:
function example() {
  return Promise.resolve("Hello");
}
```

## Common Interview Questions

### Q1: What's the output?

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

Promise.resolve().then(() => {
  console.log('3');
});

console.log('4');
```

**Answer**: `1 4 3 2`

**Explanation**:
- `1` and `4` are synchronous, execute first
- Promises (microtasks) execute before setTimeout (macrotasks)
- `3` executes before `2`

### Q2: Promise Chaining

```javascript
// ❌ Wrong way (nesting)
fetch('/api/user')
  .then(response => {
    return response.json().then(user => {
      return fetch(`/api/posts/${user.id}`).then(posts => {
        return posts.json();
      });
    });
  });

// ✅ Right way (chaining)
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error));

// ✅ Best way (async/await)
async function getUserPosts() {
  try {
    const userResponse = await fetch('/api/user');
    const user = await userResponse.json();
    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();
    return posts;
  } catch (error) {
    console.error(error);
  }
}
```

### Q3: Parallel vs Sequential Execution

```javascript
// ❌ Sequential (slower)
async function sequential() {
  const user = await fetchUser();      // Takes 1s
  const posts = await fetchPosts();    // Takes 1s
  const comments = await fetchComments(); // Takes 1s
  // Total: ~3 seconds
}

// ✅ Parallel (faster)
async function parallel() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  // Total: ~1 second (all run in parallel)
}

// Use sequential when operations depend on each other
async function dependentOperations() {
  const user = await fetchUser();
  const posts = await fetchUserPosts(user.id); // Needs user.id
  return posts;
}
```

### Q4: Error Handling

```javascript
// Multiple ways to handle errors

// 1. Try-Catch
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
    throw error; // Re-throw if needed
  }
}

// 2. .catch()
async function fetchData() {
  const data = await fetch('/api/data')
    .then(res => res.json())
    .catch(error => {
      console.error('Error:', error);
      return null; // Return default value
    });
  return data;
}

// 3. Promise.allSettled for multiple operations
async function fetchMultiple() {
  const results = await Promise.allSettled([
    fetch('/api/users'),
    fetch('/api/posts'),
    fetch('/api/comments')
  ]);

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => r.reason);

  return { successful, failed };
}
```

### Q5: Implement Promise.all

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      reject(new TypeError('Argument must be an array'));
      return;
    }

    const results = [];
    let completedCount = 0;

    if (promises.length === 0) {
      resolve(results);
      return;
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completedCount++;

          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          reject(error);
        });
    });
  });
}

// Test
promiseAll([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
]).then(console.log); // [1, 2, 3]
```

### Q6: Implement Promise.race

```javascript
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      reject(new TypeError('Argument must be an array'));
      return;
    }

    promises.forEach(promise => {
      Promise.resolve(promise)
        .then(resolve)
        .catch(reject);
    });
  });
}
```

### Q7: Retry Failed Promises

```javascript
async function retry(fn, maxAttempts = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }
      console.log(`Attempt ${attempt} failed. Retrying...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
retry(() => fetch('/api/data'), 3, 1000)
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('All attempts failed:', error));
```

### Q8: Promise with Timeout

```javascript
function promiseWithTimeout(promise, timeout) {
  return Promise.race([
    promise,
    new Promise((_, reject) => {
      setTimeout(() => {
        reject(new Error('Operation timed out'));
      }, timeout);
    })
  ]);
}

// Usage
promiseWithTimeout(fetch('/api/data'), 5000)
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

## Advanced Patterns

### 1. Queue/Sequential Execution

```javascript
async function executeSequentially(tasks) {
  const results = [];
  for (const task of tasks) {
    results.push(await task());
  }
  return results;
}

// Usage
const tasks = [
  () => fetchUser(1),
  () => fetchUser(2),
  () => fetchUser(3)
];

executeSequentially(tasks).then(console.log);
```

### 2. Concurrent Execution with Limit

```javascript
async function executeWithLimit(tasks, limit) {
  const results = [];
  const executing = [];

  for (const task of tasks) {
    const promise = Promise.resolve().then(() => task());
    results.push(promise);

    if (limit <= tasks.length) {
      const e = promise.then(() => {
        executing.splice(executing.indexOf(e), 1);
      });
      executing.push(e);

      if (executing.length >= limit) {
        await Promise.race(executing);
      }
    }
  }

  return Promise.all(results);
}
```

### 3. Cancellable Promises

```javascript
function makeCancellable(promise) {
  let cancelled = false;

  const wrappedPromise = new Promise((resolve, reject) => {
    promise
      .then(value => {
        if (!cancelled) {
          resolve(value);
        }
      })
      .catch(error => {
        if (!cancelled) {
          reject(error);
        }
      });
  });

  return {
    promise: wrappedPromise,
    cancel: () => {
      cancelled = true;
    }
  };
}

// Usage
const { promise, cancel } = makeCancellable(fetch('/api/data'));

// Cancel after 2 seconds
setTimeout(() => {
  cancel();
  console.log('Request cancelled');
}, 2000);

promise
  .then(response => console.log('Success'))
  .catch(error => console.log('Error or Cancelled'));
```

## Common Pitfalls

### 1. Forgetting to await

```javascript
// ❌ Wrong
async function wrong() {
  const data = fetchData(); // Returns Promise, not data!
  console.log(data.name); // undefined or error
}

// ✅ Right
async function right() {
  const data = await fetchData();
  console.log(data.name);
}
```

### 2. Not handling errors

```javascript
// ❌ Wrong - unhandled rejection
async function wrong() {
  const data = await fetchData(); // If this fails, error is unhandled
}

// ✅ Right
async function right() {
  try {
    const data = await fetchData();
  } catch (error) {
    console.error(error);
  }
}
```

### 3. Using async without await

```javascript
// ❌ Unnecessary async
async function unnecessary() {
  return 42; // No await, async is unnecessary
}

// ✅ Better
function better() {
  return 42;
}
```

## Key Takeaways

1. Promises handle asynchronous operations
2. Async/await makes asynchronous code easier to read
3. Use `Promise.all()` for parallel execution
4. Always handle errors with try-catch or .catch()
5. Understand microtasks vs macrotasks
6. Don't forget to `await` promises
7. Use appropriate Promise methods based on requirements

## Practice Problems

1. Implement `Promise.allSettled()`
2. Create a function that fetches data with retry logic
3. Implement a debounced API call using Promises
4. Create a Promise-based queue system
5. Build a simple async rate limiter

---

**Next Topic**: [Event Loop](./event-loop.md)
