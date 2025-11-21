# JavaScript Event Loop

## Table of Contents
- [Understanding the Event Loop](#understanding-the-event-loop)
- [Call Stack](#call-stack)
- [Task Queues](#task-queues)
- [Microtasks vs Macrotasks](#microtasks-vs-macrotasks)
- [Common Interview Questions](#common-interview-questions)

## Understanding the Event Loop

The Event Loop is JavaScript's mechanism for handling asynchronous operations in a single-threaded environment.

### Key Components

1. **Call Stack**: Where function execution happens
2. **Web APIs**: Browser-provided APIs (setTimeout, fetch, DOM events)
3. **Callback Queue (Task Queue)**: Holds callbacks from macrotasks
4. **Microtask Queue**: Holds callbacks from microtasks
5. **Event Loop**: Coordinates execution between stack and queues

### Visual Representation

```
┌─────────────────────────────┐
│         Call Stack          │
└─────────────────────────────┘
              ↑
              │
┌─────────────┴───────────────┐
│        Event Loop           │
└─────────────┬───────────────┘
              ↓
    ┌─────────────────┐
    │  Microtask Queue│
    └─────────────────┘
              ↓
    ┌─────────────────┐
    │ Macrotask Queue │
    └─────────────────┘
```

## Call Stack

The call stack is a LIFO (Last In, First Out) data structure that keeps track of function execution.

```javascript
function first() {
  console.log('First');
  second();
  console.log('First Again');
}

function second() {
  console.log('Second');
  third();
}

function third() {
  console.log('Third');
}

first();

// Output:
// First
// Second
// Third
// First Again

// Call Stack Execution:
// 1. first() pushed
// 2. second() pushed
// 3. third() pushed
// 4. third() popped
// 5. second() popped
// 6. first() popped
```

## Task Queues

### Macrotask Queue (Task Queue)
Handles callbacks from:
- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- I/O operations
- UI rendering

### Microtask Queue
Handles callbacks from:
- `Promise.then/catch/finally`
- `queueMicrotask()`
- `MutationObserver`
- `process.nextTick()` (Node.js - has higher priority)

## Microtasks vs Macrotasks

### Execution Order

1. Execute all synchronous code (call stack)
2. Execute ALL microtasks in the microtask queue
3. Execute ONE macrotask from the macrotask queue
4. Repeat from step 2

### Example

```javascript
console.log('Start'); // 1. Synchronous

setTimeout(() => {
  console.log('setTimeout'); // 5. Macrotask
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1'); // 3. Microtask
  })
  .then(() => {
    console.log('Promise 2'); // 4. Microtask
  });

console.log('End'); // 2. Synchronous

// Output:
// Start
// End
// Promise 1
// Promise 2
// setTimeout
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
1. `1` - Synchronous
2. `4` - Synchronous
3. `3` - Microtask (Promise)
4. `2` - Macrotask (setTimeout)

### Q2: Complex Event Loop

```javascript
console.log('Script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
  })
  .then(() => {
    console.log('Promise 2');
  });

async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

async1();

console.log('Script end');
```

**Answer**:
```
Script start
async1 start
async2
Script end
Promise 1
async1 end
Promise 2
setTimeout
```

**Explanation**:
1. **Synchronous**: Script start, async1 start, async2, Script end
2. **Microtasks**: Promise 1, async1 end (await creates microtask), Promise 2
3. **Macrotask**: setTimeout

### Q3: Nested setTimeout vs Promise

```javascript
setTimeout(() => {
  console.log('Timeout 1');
  Promise.resolve().then(() => {
    console.log('Promise in Timeout 1');
  });
}, 0);

setTimeout(() => {
  console.log('Timeout 2');
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 1');
});
```

**Answer**:
```
Promise 1
Timeout 1
Promise in Timeout 1
Timeout 2
```

**Explanation**:
1. All synchronous code completes
2. `Promise 1` (microtask) executes
3. `Timeout 1` (macrotask) executes
4. `Promise in Timeout 1` (microtask created during macrotask) executes
5. `Timeout 2` (next macrotask) executes

### Q4: What happens here?

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 0);
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => {
    console.log(j);
  }, 0);
}
```

**Answer**:
```
3
3
3
0
1
2
```

**Explanation**:
- `var` is function-scoped: all callbacks reference same `i`, which is 3 after loop
- `let` is block-scoped: each callback has its own `j` value

### Q5: Promise + setTimeout Order

```javascript
setTimeout(() => console.log('1'), 0);

Promise.resolve().then(() => console.log('2'));

queueMicrotask(() => console.log('3'));

setTimeout(() => console.log('4'), 0);

Promise.resolve().then(() => console.log('5'));

console.log('6');
```

**Answer**: `6 2 3 5 1 4`

**Explanation**:
1. `6` - Synchronous
2. `2, 3, 5` - All microtasks execute before any macrotask
3. `1, 4` - Macrotasks in order

### Q6: Async/Await and Event Loop

```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2 start');
  return new Promise((resolve) => {
    console.log('promise');
    resolve();
  });
}

console.log('script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

async1();

new Promise((resolve) => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
});

console.log('script end');
```

**Answer**:
```
script start
async1 start
async2 start
promise
promise1
script end
async1 end
promise2
setTimeout
```

### Q7: Multiple Promises

```javascript
Promise.resolve()
  .then(() => {
    console.log('1');
    return Promise.resolve('2');
  })
  .then((res) => {
    console.log(res);
  });

Promise.resolve()
  .then(() => {
    console.log('3');
  })
  .then(() => {
    console.log('4');
  })
  .then(() => {
    console.log('5');
  });
```

**Answer**: `1 3 4 2 5`

**Explanation**:
- Returning a Promise in `.then()` adds extra microtasks
- First chain: 1 → (Promise resolution) → 2
- Second chain: 3 → 4 → 5
- They interleave based on microtask queue

### Q8: requestAnimationFrame

```javascript
console.log('start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(() => {
  console.log('promise');
});

requestAnimationFrame(() => {
  console.log('requestAnimationFrame');
});

console.log('end');
```

**Answer** (in browser):
```
start
end
promise
requestAnimationFrame
setTimeout
```

**Explanation**:
- `requestAnimationFrame` runs before macrotasks but after microtasks
- Priority: Synchronous → Microtasks → rAF → Macrotasks

## Node.js Event Loop

Node.js has a slightly different event loop with phases:

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

### process.nextTick() and setImmediate()

```javascript
setTimeout(() => console.log('setTimeout'), 0);

setImmediate(() => console.log('setImmediate'));

process.nextTick(() => console.log('nextTick'));

Promise.resolve().then(() => console.log('Promise'));

// Output:
// nextTick
// Promise
// setTimeout (or setImmediate, order can vary)
// setImmediate (or setTimeout)
```

**Priority in Node.js**:
1. `process.nextTick()` (highest priority microtask)
2. Promises (microtasks)
3. `setImmediate()` / `setTimeout()` (macrotasks)

## Best Practices

### 1. Avoid Blocking the Event Loop

```javascript
// ❌ Bad: Blocks event loop
function blockingOperation() {
  const start = Date.now();
  while (Date.now() - start < 5000) {
    // Blocks for 5 seconds
  }
  console.log('Done');
}

// ✅ Good: Non-blocking
function nonBlockingOperation() {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log('Done');
      resolve();
    }, 5000);
  });
}
```

### 2. Break Large Tasks

```javascript
// ❌ Bad: Process 10000 items at once
function processAll(items) {
  items.forEach(item => {
    // Heavy processing
    process(item);
  });
}

// ✅ Good: Process in chunks
async function processInChunks(items, chunkSize = 100) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    chunk.forEach(item => process(item));

    // Yield to event loop
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

### 3. Use Web Workers for CPU-Intensive Tasks

```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ data: largeDataset });

worker.onmessage = function(e) {
  console.log('Result:', e.data);
};

// worker.js
self.onmessage = function(e) {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};
```

## Debugging Event Loop Issues

```javascript
// Check pending async operations
console.log('Active handles:', process._getActiveHandles().length);
console.log('Active requests:', process._getActiveRequests().length);

// Monitor event loop lag
let lastCheck = Date.now();
setInterval(() => {
  const now = Date.now();
  const lag = now - lastCheck - 1000;
  console.log(`Event loop lag: ${lag}ms`);
  lastCheck = now;
}, 1000);
```

## Key Takeaways

1. JavaScript is single-threaded with an event loop for async operations
2. Microtasks execute before macrotasks
3. All microtasks execute before the next macrotask
4. `Promise` callbacks are microtasks
5. `setTimeout`/`setInterval` are macrotasks
6. `async/await` creates microtasks for code after `await`
7. Don't block the event loop with long synchronous operations
8. Use appropriate async patterns for different scenarios

## Practice Problems

1. Predict output of complex event loop scenarios
2. Debug event loop blocking issues
3. Implement a task scheduler respecting microtask/macrotask queues
4. Create a visualization of event loop execution
5. Optimize code to prevent event loop blocking

---

**Next Topic**: [Prototypes and Inheritance](./prototypes.md)
