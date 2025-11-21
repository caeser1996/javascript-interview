# Prototypes and Inheritance in JavaScript

## Table of Contents
- [Understanding Prototypes](#understanding-prototypes)
- [Prototype Chain](#prototype-chain)
- [Constructor Functions](#constructor-functions)
- [ES6 Classes](#es6-classes)
- [Inheritance Patterns](#inheritance-patterns)
- [Common Interview Questions](#common-interview-questions)

## Understanding Prototypes

Every JavaScript object has an internal property called `[[Prototype]]` (accessed via `__proto__` or `Object.getPrototypeOf()`).

### Basic Concept

```javascript
const person = {
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

const john = Object.create(person);
john.name = 'John';

john.greet(); // "Hello, I'm John"

console.log(john.__proto__ === person); // true
console.log(Object.getPrototypeOf(john) === person); // true
```

### Key Points

1. Objects inherit properties from their prototype
2. Prototype chain enables property lookup
3. Changes to prototype affect all instances
4. Each function has a `prototype` property

## Prototype Chain

When accessing a property, JavaScript:
1. Checks the object itself
2. Checks the object's prototype
3. Checks the prototype's prototype
4. Continues until reaching `null`

```javascript
const obj = { a: 1 };

// Prototype chain:
// obj -> Object.prototype -> null

console.log(obj.toString()); // From Object.prototype
console.log(obj.__proto__ === Object.prototype); // true
console.log(obj.__proto__.__proto__); // null
```

### Visualizing the Chain

```
object
  ↓ __proto__
Object.prototype
  ↓ __proto__
null
```

## Constructor Functions

Before ES6 classes, constructor functions were used for creating objects.

### Basic Constructor

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  console.log(`Hello, I'm ${this.name}`);
};

const john = new Person('John', 30);
john.greet(); // "Hello, I'm John"

console.log(john.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
```

### What `new` Does

```javascript
function Person(name) {
  this.name = name;
}

const john = new Person('John');

// Equivalent to:
const john = Object.create(Person.prototype);
Person.call(john, 'John');
// (return john if Person doesn't return an object)
```

### Implementing `new`

```javascript
function myNew(constructor, ...args) {
  // 1. Create new object with constructor's prototype
  const obj = Object.create(constructor.prototype);

  // 2. Execute constructor with 'this' bound to new object
  const result = constructor.apply(obj, args);

  // 3. Return object (or constructor's return value if it's an object)
  return result instanceof Object ? result : obj;
}

// Usage
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const john = myNew(Person, 'John');
john.greet(); // "Hello, John"
```

## ES6 Classes

Classes are syntactic sugar over constructor functions and prototypes.

### Basic Class

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }

  // Static method
  static species() {
    return 'Homo sapiens';
  }

  // Getter
  get info() {
    return `${this.name}, ${this.age} years old`;
  }

  // Setter
  set nickname(value) {
    this._nickname = value;
  }
}

const john = new Person('John', 30);
john.greet(); // "Hello, I'm John"
console.log(john.info); // "John, 30 years old"
console.log(Person.species()); // "Homo sapiens"
```

### Class vs Constructor Function

```javascript
// Constructor Function
function PersonFunc(name) {
  this.name = name;
}
PersonFunc.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

// Class (equivalent)
class PersonClass {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

// Both produce similar results
const p1 = new PersonFunc('John');
const p2 = new PersonClass('Jane');
```

## Inheritance Patterns

### 1. Prototypal Inheritance (Object.create)

```javascript
const animal = {
  speak() {
    console.log(`${this.name} makes a sound`);
  }
};

const dog = Object.create(animal);
dog.name = 'Rex';
dog.bark = function() {
  console.log('Woof!');
};

dog.speak(); // "Rex makes a sound"
dog.bark();  // "Woof!"
```

### 2. Constructor Inheritance (Old Way)

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Set up prototype chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log('Woof!');
};

const rex = new Dog('Rex', 'German Shepherd');
rex.speak(); // "Rex makes a sound"
rex.bark();  // "Woof!"
```

### 3. Class Inheritance (ES6)

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Call parent constructor
    this.breed = breed;
  }

  bark() {
    console.log('Woof!');
  }

  speak() {
    super.speak(); // Call parent method
    console.log('Woof woof!');
  }
}

const rex = new Dog('Rex', 'German Shepherd');
rex.speak();
// "Rex makes a sound"
// "Woof woof!"
```

## Common Interview Questions

### Q1: What's the difference between `__proto__` and `prototype`?

```javascript
function Person(name) {
  this.name = name;
}

const john = new Person('John');

// prototype: Property of constructor functions
console.log(Person.prototype); // { constructor: Person }

// __proto__: Points to object's prototype
console.log(john.__proto__ === Person.prototype); // true

// Relationship
console.log(john.__proto__ === Person.prototype); // true
console.log(Person.prototype.__proto__ === Object.prototype); // true
```

**Answer**:
- `prototype`: Property on constructor functions, used as prototype for new instances
- `__proto__`: Property on all objects, references the object's prototype

### Q2: How do you check if a property is own vs inherited?

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log('Hello');
};

const john = new Person('John');

// Check own property
console.log(john.hasOwnProperty('name'));  // true
console.log(john.hasOwnProperty('greet')); // false

// Check if property exists (own or inherited)
console.log('name' in john);  // true
console.log('greet' in john); // true

// Get own properties only
console.log(Object.keys(john));              // ['name']
console.log(Object.getOwnPropertyNames(john)); // ['name']
```

### Q3: Implement inheritance without ES6 classes

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}

// Proper inheritance setup
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log('Woof!');
};

const rex = new Dog('Rex', 'Labrador');
rex.speak(); // "Rex makes a sound"
rex.bark();  // "Woof!"

console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
```

### Q4: What happens if you forget `new`?

```javascript
function Person(name) {
  this.name = name;
}

// With 'new'
const john = new Person('John');
console.log(john.name); // "John"

// Without 'new'
const jane = Person('Jane');
console.log(jane); // undefined
console.log(window.name); // "Jane" (in browser, 'this' = window)

// Solution: Safe constructor pattern
function SafePerson(name) {
  if (!(this instanceof SafePerson)) {
    return new SafePerson(name);
  }
  this.name = name;
}

const bob = SafePerson('Bob'); // Works even without 'new'
console.log(bob.name); // "Bob"
```

### Q5: How to create an object without a prototype?

```javascript
// Objects without prototype
const obj = Object.create(null);

console.log(obj.__proto__);        // undefined
console.log(obj.toString);         // undefined
console.log(Object.getPrototypeOf(obj)); // null

// Useful for dictionaries/maps
const dict = Object.create(null);
dict.toString = 'custom value'; // No conflict with inherited toString
```

### Q6: Implement Object.create

```javascript
function myCreate(proto) {
  function F() {}
  F.prototype = proto;
  return new F();
}

// Usage
const animal = {
  speak() {
    console.log('Animal sound');
  }
};

const dog = myCreate(animal);
dog.speak(); // "Animal sound"
console.log(Object.getPrototypeOf(dog) === animal); // true
```

### Q7: Prototype Chain Lookup

```javascript
function Animal() {}
Animal.prototype.type = 'Animal';

function Dog() {}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.breed = 'Dog';

function Poodle() {}
Poodle.prototype = Object.create(Dog.prototype);
Poodle.prototype.name = 'Poodle';

const myPoodle = new Poodle();

console.log(myPoodle.name);  // "Poodle" (from Poodle.prototype)
console.log(myPoodle.breed); // "Dog" (from Dog.prototype)
console.log(myPoodle.type);  // "Animal" (from Animal.prototype)

// Prototype chain:
// myPoodle -> Poodle.prototype -> Dog.prototype -> Animal.prototype -> Object.prototype -> null
```

### Q8: Method Overriding

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  speak() {
    return `${super.speak()} - Woof!`;
  }
}

class Cat extends Animal {
  speak() {
    return `${this.name} says Meow!`;
  }
}

const dog = new Dog('Rex');
const cat = new Cat('Whiskers');

console.log(dog.speak()); // "Rex makes a sound - Woof!"
console.log(cat.speak()); // "Whiskers says Meow!"
```

### Q9: Multiple Inheritance (Mixins)

JavaScript doesn't support multiple inheritance, but you can use mixins:

```javascript
// Mixin pattern
const canEat = {
  eat() {
    console.log(`${this.name} is eating`);
  }
};

const canWalk = {
  walk() {
    console.log(`${this.name} is walking`);
  }
};

const canSwim = {
  swim() {
    console.log(`${this.name} is swimming`);
  }
};

class Animal {
  constructor(name) {
    this.name = name;
  }
}

// Apply mixins
Object.assign(Animal.prototype, canEat, canWalk);

class Duck extends Animal {
  constructor(name) {
    super(name);
  }
}

Object.assign(Duck.prototype, canSwim);

const donald = new Duck('Donald');
donald.eat();  // "Donald is eating"
donald.walk(); // "Donald is walking"
donald.swim(); // "Donald is swimming"
```

### Q10: Private Fields (ES2022)

```javascript
class BankAccount {
  #balance = 0; // Private field

  constructor(initialBalance) {
    this.#balance = initialBalance;
  }

  deposit(amount) {
    this.#balance += amount;
  }

  withdraw(amount) {
    if (amount <= this.#balance) {
      this.#balance -= amount;
      return true;
    }
    return false;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150
console.log(account.#balance);     // SyntaxError: Private field
```

## Advanced Patterns

### 1. Factory Pattern

```javascript
function createPerson(name, age) {
  return {
    name,
    age,
    greet() {
      console.log(`Hi, I'm ${this.name}`);
    }
  };
}

const john = createPerson('John', 30);
john.greet();
```

### 2. Singleton Pattern

```javascript
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    Singleton.instance = this;
    this.timestamp = Date.now();
  }

  getTimestamp() {
    return this.timestamp;
  }
}

const s1 = new Singleton();
const s2 = new Singleton();

console.log(s1 === s2); // true
```

### 3. Revealing Module Pattern

```javascript
const Calculator = (function() {
  let result = 0;

  function add(x) {
    result += x;
    return this;
  }

  function subtract(x) {
    result -= x;
    return this;
  }

  function getResult() {
    return result;
  }

  return {
    add,
    subtract,
    getResult
  };
})();

Calculator.add(5).subtract(2);
console.log(Calculator.getResult()); // 3
```

## Key Takeaways

1. Every object has a prototype (except objects created with `Object.create(null)`)
2. Prototype chain enables property inheritance
3. `prototype` is on constructor functions, `__proto__` is on instances
4. ES6 classes are syntactic sugar over prototypes
5. Use `hasOwnProperty()` to check for own properties
6. Inheritance can be achieved via prototypes or classes
7. Private fields (#) are true private in ES2022+
8. Mixins enable composition over inheritance

## Practice Problems

1. Implement your own class system without ES6 classes
2. Create a deep clone function that handles prototypes
3. Implement method chaining with prototypes
4. Build a plugin system using prototypes
5. Create a custom inheritance system with multiple inheritance support

---

**Next Topic**: [ES6+ Features](./es6-features.md)
