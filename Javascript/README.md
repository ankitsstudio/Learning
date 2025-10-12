# JavaScript Key Concepts to Always Keep in Mind

This README covers some fundamental JavaScript concepts that are essential for writing efficient, clean, and maintainable code. These concepts include **Throttling and Debouncing**, **Event Delegation**, and **Closures**. Each section contains explanations, code examples, and practical thoughts to deepen your understanding.

***

## 1. Throttling and Debouncing

* These techniques help control how often a function is executed in response to events, which is crucial for performance optimization, especially in events that fire rapidly like scrolling or resizing.

* **Debouncing**
  * What it does: Delays the function execution until there is a pause in events.
  * Use case: Search input where filtering happens after the user stops typing.

    ``` javascript
    function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
        func.apply(this, args);
        }, delay);
    };
    }

    // Usage example
    const handleInput = debounce((event) => {
    console.log('Search query:', event.target.value);
    }, 300);

    document.getElementById('searchInput').addEventListener('input', handleInput);
    ```

* **Throttling**
  * What it does: Limits the function to execute not more than once every N milliseconds.
  * Use case: Window resizing or scroll event handlers where you want to avoid excessive calls.

    ``` javascript
    function throttle(func, limit) {
    let lastFunc;
    let lastRan;
    return function(...args) {
        if (!lastRan) {
        func.apply(this, args);
        lastRan = Date.now();
        } else {
        clearTimeout(lastFunc);
        lastFunc = setTimeout(() => {
            if ((Date.now() - lastRan) >= limit) {
            func.apply(this, args);
            lastRan = Date.now();
            }
        }, limit - (Date.now() - lastRan));
        }
    };
    }

    // Usage example
    window.addEventListener('scroll', throttle(() => {
    console.log('Scrolled!');
    }, 1000));
    ```

***

## 2. Event Delegation

* Event Delegation involves adding a single event listener to a common parent element instead of multiple listeners to individual child elements. The event bubbles up from the target element to the parent, where you can handle it efficiently.

* **Advantages:**
  * Reduces memory usage.
  * Supports dynamic elements added after initial page load.
  * Simplifies event management.

* **Example: Handling clicks on dynamic list items**

    ``` xml
    <ul id="taskList">
    <li>Task 1</li>
    <li>Task 2</li>
    <!-- Additional tasks might be added dynamically -->
    </ul>
    ```

    ``` javascript
    document.getElementById('taskList').addEventListener('click', function(event) {
    if (event.target.tagName === 'LI') {
        event.target.classList.toggle('completed');
        console.log(`Toggled: ${event.target.textContent}`);
    }
    });
    ```

* **More complex example with data attributes**

    ``` xml
    <div id="menu">
    <button data-action="save">Save</button>
    <button data-action="load">Load</button>
    <button data-action="search">Search</button>
    </div>
    ```

    ```javascript
    const menu = document.getElementById('menu');
    menu.addEventListener('click', (event) => {
    const action = event.target.dataset.action;
    if (action) {
        switch(action) {
        case 'save':
            alert('Saving...');
            break;
        case 'load':
            alert('Loading...');
            break;
        case 'search':
            alert('Searching...');
            break;
        }
    }
    });
    ```

* **Combining concepts:** Event delegation often works well with throttling or debouncing, especially when dealing with scroll or input events on many elements.

***

## 3. Closure

* A **closure** gives you access to an outer function’s scope from an inner function, **even after the outer function has executed**. This is crucial for creating private variables and stateful functions.

* **Example**: Stateful button counter

    ```xml
    <button id="btn1">Like</button>
    <button id="btn2">Like</button>
    ```

    ```javascript
    function createLikeButton(buttonId) {
    let likes = 0; // This variable is private to this instance (closure)

    document.getElementById(buttonId).addEventListener('click', () => {
        likes++;
        console.log(`${buttonId} has ${likes} likes`);
    });
    }

    createLikeButton('btn1');
    createLikeButton('btn2');
    ```

    Each button maintains its own independent like count due to closures preserving the **likes** variable.

* **Memory management:** Closures are powerful but can cause memory leaks if references to outer scopes persist unnecessarily—be mindful when using closures inside long-running apps.

***

## 4. Promise & Async/Await

* **Promise:**
  * A **Promise** is an object representing the eventual **completion or failure** of an **asynchronous operation**.

  * **Use Promise when:**
    * When you want to manage asynchronous tasks with chaining, or handle multiple tasks in sequence or parallel.
    * When targeting environments where async/await might not be supported.

  **States of a Promise:**
  * *Pending*: Initial state
  * *Fulfilled*: The operation completed successfully
  * *Rejected*: The operation failed
  
  **Basic Syntax:**

  ``` javascript
  const promise = new Promise(function(resolve, reject) {
    // async operation
    if (/* success */) {
        resolve(result);
    } else {
        reject(error);
    }
  });

  promise
    .then(msg => console.log(msg))   // Runs on resolve
    .catch(err => console.error(err));  // Runs on reject
  ```

  **Example of a Promise:**

  ```javascript
  function fakeApiCall() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = true; // change to false to simulate failure
            if (success) {
                resolve("Data received from API");
            } else {
                reject("API error");
            }
        }, 2000);
    });
  }

  fakeApiCall()
    .then(response => {
        console.log("Success:", response);
    })
    .catch(error => {
        console.error("Failed:", error);
    });
  ```
  
  * **Promise Methods**
    JavaScript provides several static methods for working with promises:

    * **Promise.all():** Waits for all promises to be fulfilled or any to be rejected.
    * **Promise.allSettled():** Waits for all promises to settle (either fulfilled or rejected).
    * **Promise.any():** Waits for any promise to be fulfilled.
    * **Promise.race():** Waits for the first promise to settle (either fulfilled or rejected).

  **Promise Chaining:**
    We can chain **.then()** calls to perform operations in sequence:

  ```javascript
  function step1(data) {
    return new Promise((resolve) => {
        setTimeout(() => resolve(data + " → Step 1"), 1000);
    });
    }

  function step2(data) {
    return new Promise((resolve) => {
        setTimeout(() => resolve(data + " → Step 2"), 1000);
    });
  }

  step1("Start")
    .then(step2)
    .then(result => console.log(result)); // Start → Step 1 → Step 2

* **Async/Await**
  * Async/Await is syntax sugar over Promises. It makes asynchronous code look and behave like synchronous code.
  * Functions marked **async** alwyas return a promise.
  * **await** pauses the execution of that async function until the awaited Promise settles, then resumes with the resolved value.

  * **Use Async/Await when**
    * you want clearer, more readable asynchronous code, resembling linear, step-by-step logic.
    * For most modern JavaScript projects where ES2017+ is supported.

  **Example**

  ```javascript
  async function fetchUserData(userId) {
    try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
        if (!response.ok) throw new Error("User not found");

        const user = await response.json();
        console.log("User Data:", user);
    } catch (error) {
        console.error("Fetch error:", error.message);
    }
  }

  fetchUserData(1);
  ```

  **Parallel Asynchronous Tasks**
  Using **Promise.all** for multiple async operations:

  ```javascript
  async function getMultipleUsers() {
    const userIds = [1, 2, 3];
    try {
        const responses = await Promise.all(
            userIds.map(id => fetch(`https://jsonplaceholder.typicode.com/users/${id}`))
        );
        const users = await Promise.all(responses.map(res => res.json()));
        console.log("Users:", users);
    } catch (error) {
        console.error("One or more requests failed", error);
    }
  }
  ```
  
  **Advanced Example: Uploading Files**

  ```javascript
  async function uploadFiles(files) {
    try {
        const uploadPromises = files.map(file => uploadToServer(file));
        const results = await Promise.all(uploadPromises);
        console.log("All files uploaded successfully:", results);
    } catch (error) {
        console.error("Upload failed:", error);
    }
  }

  function uploadToServer(file) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
          if (file.size > 0) resolve(`${file.name} uploaded`);
          else reject(`${file.name} is empty`);
        }, 1000);
    });
  }

  // Example files
  uploadFiles([
    { name: "file1.txt", size: 100 },
    { name: "file2.txt", size: 0 }, // will trigger error
  ]);
  ```

  **More to Promises and Async/Await in Javascript**
  * Asynchronous programming is core part of Javascript, helping you handle oplerations that take time like - fetching data or reading files - without blocking your entire program.

  * In older JavaScript, callbacks were used extensively for async tasks. But multiple nested callbacks lead to **callback hell**, making code hard to read and maintain. Promises and Async/Await solve this by providing more structured, readable ways to write asynchronous code, making it easier to handle success, failure, and complex chains.

***

## 5. Higher Order Functions (HOFs)

* A **Higher-Order Function (HOF)** is a function that **does one or both** of the following:
  * Takes another function as an argument
  * Returns a function as its result

* Common Examples of **Built-in HOFs** in JavaScript:
  * **map()**: Transforms array elements
  * **filter()**: Filters elements based on a condition
  * **reduce()**: Reduces array to single value
  * **forEach()**: Iterates over each element (no return value)
  * **Sort()**: Sorts array based on a comparator fucntion

* **Custom Higher-Order Function Example**

  ```javascript
  function repeat(operation, times) {
    for (let i = 0; i < times; i++) {
        operation();
    }
  }

  repeat(() => console.log("Hello!"), 3);
  // Output:
  // Hello!
  // Hello!
  // Hello!
  ```

* **Real-World Project use case:**
  * **Scenerio: Middleware in Express.js (Web Server)**
    * Middleware functions in Express.js are HOFs — they take other functions and control how/when they are run.
  
    ```javascript
    const express = require('express');
    const app = express();

    // Higher-order middleware function
    function logger(message) {
        return function (req, res, next) {
            console.log(`[${new Date().toISOString()}] ${message} ${req.method} ${req.url}`);
            next(); // pass control to the next middleware
        };
    }

    // Use HOF logger with different messages
    app.use(logger("API Request"));

    app.get('/', (req, res) => {
        res.send("Home Page");
    });

    app.listen(3000, () => console.log('Server started on port 3000'));
    ```

  * **Scenerio: Event Handling Wrapper**
    * This is useful in search input boxes, where you don’t want to make an API call on every keystroke.
  
    ```javascript
    // Higher-order function to debounce an event
    function debounce(fn, delay) {
    let timeout;
    return function (...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => fn.apply(this, args), delay);
    };
    }

    // Usage in DOM
    document.getElementById("search").addEventListener(
    "input",
    debounce((event) => {
        console.log("Searching for:", event.target.value);
    }, 300)
    );
    ```

  * **Scenerio: API Retry Mechanism (HOF for Robustness)**

    ```javascript
    function withRetry(fn, retries = 3) {
        return async function (...args) {
            let attempts = 0;
            while (attempts < retries) {
            try {
                return await fn(...args);
            } catch (error) {
                attempts++;
                if (attempts >= retries) throw error;
                console.log(`Retrying... (${attempts})`);
            }
            }
        };
    }

    // Simulated API function
    async function fetchUser() {
    const fail = Math.random() > 0.5;
    if (fail) throw new Error("API failed");
    return { name: "John" };
    }

    const fetchUserWithRetry = withRetry(fetchUser, 3);

    fetchUserWithRetry().then(console.log).catch(console.error);
    ```

## 6. Functional Programming Concepts

Functional Programming is a **paradigm** that treats computation as the **evaluation of mathematical functions** and **avoids changing state and mutable data**.

***

### 1. Pure Functions

A pure function is a function that:

* Always returns the same output for the same input.
* Does not cause side effects (like modifying external variables, DOM, file systems, etc.)

* **Impure function Example (Bad)**

    ```javascript
    let counter = 0;

    function incrementCounter() {
    counter += 1;
    return counter;
    }

    console.log(incrementCounter()); // 1
    console.log(incrementCounter()); // 2 (Different result for same call)
    ```

* **Why it is impure?**
  * It modifies an external variable *counter*
  * It doesn't always return the same result.

* **Pure Function Example**

    ```javascript
    function increment(value) {
    return value + 1;
    }

    console.log(increment(5)); // 6
    console.log(increment(5)); // 6 (Always same result)
    ```

* **Why it is Pure?**
  * It doesn’t change anything outside its scope.
  * It always returns the same result for the same input.

### 2. Immutability

Immutability means once data is created, it cannot be changed. Instead of modifying existing objects/arrays, we create new copies with updated data.

* **Mutable Example (Bad)**

    ```javascript
    const user = { name: "Alice", age: 25 };

    function makeOlder(user) {
    user.age += 1;
    return user;
    }

    console.log(makeOlder(user)); // { name: "Alice", age: 26 }
    console.log(user);            // Original object mutated!
    ```

* **What's wrong?**
  * The original user object is changed (side effect).
  * Makes debugging and state tracking harder in real apps.

* **Immutable Example**

    ```javascript
    const user = { name: "Alice", age: 25 };

    function makeOlder(user) {
    return { ...user, age: user.age + 1 };
    }

    const newUser = makeOlder(user);
    console.log(newUser); // { name: "Alice", age: 26 }
    console.log(user);    // { name: "Alice", age: 25 } — original remains unchanged
    ```

* **Why it is good?**
  * We don’t change the original object.
  * Safer, especially in React, Redux, or concurrent environments.

### 3. Function Composition

Function Composition is the process of combining multiple smaller functions to build a more complex one.

* **Mutable composition Example**

  ```javascript
  const trim = str => str.trim();
  const toLower = str => str.toLowerCase();
  const wrapInDiv = str => `<div>${str}</div>`;

  // Compose manually
  const transform = str => wrapInDiv(toLower(trim(str)));

  console.log(transform("   Hello World   ")); 
  // <div>hello world</div>
  ```
  
***

#### Scenario: Transform product data in an immutable and functional way

```javascript
const products = [
  { name: "  Laptop ", price: 1000 },
  { name: "  Phone ", price: 500 },
];

const trim = str => str.trim();
const toUpper = str => str.toUpperCase();
const applyTax = price => price * 1.1;

const transformProduct = product => ({
  ...product,
  name: toUpper(trim(product.name)),
  price: applyTax(product.price),
});

const transformedProducts = products.map(transformProduct);

console.log(transformedProducts);
// [
//   { name: "LAPTOP", price: 1100 },
//   { name: "PHONE", price: 550 }
// ]
```
## 7. Prototype & Inheritance
A prototype is a regular object that an object delegates to for property and method lookup. Think of it as a fallback object: if property x is not on obj, the runtime asks obj's prototype for x, and continues up the chain until it finds the property or reaches null.

* The internal prototype pointer is commonly called [[Prototype]].
* You can get it via Object.getPrototypeOf(obj) and (non-standard but common) via obj.__proto__.
* Functions (constructor functions) have a .prototype property which becomes the [[Prototype]] of objects created via new.

### 1. Why JavaScript uses prototypes — what problem it solves
* **Memory efficiency**: Instead of each instance holding copies of methods, instances can delegate to a single prototype object that contains shared methods.
* **Dynamic extension**: You can add methods to a prototype at runtime and all existing instances immediately see the new methods.

### 2. Property lookup & prototype chain
when evaluating `obj.prop`: 
  1.  Check if `prop` exists as an own property on `obj`. If yes, return it.
  2.  Otherwise, get `proto = Object.getPrototypeOf(obj)`.
  3.  If `proto` is `null`, property not found (result `undefined`).
  4.  Else repeat step 1 with `proto`.

**Shadowing**: If an own property has the same name as a prototype the own property hides (shadows) the prototype one.

**Important Operators**: 
  1. **in**: returns `true` if property exists on object or anywhere up the prototype chain.
  2. **hasOwnProperty**: returns `true` only for own properties.
  3. **instanceof**: checks whether an object's prototype chain contains `Ctor.prototype`

### 3. How to create objects & patterns
1. **Constructor function + prototype (classic)**: 
  
   ```javascript
   function Person(name) {
     this.name = name;
     // DON'T put methods here (they'd be created per-instance)
   }

   Person.prototype.greet = function() {
     return `Hi, I'm ${this.name}`;
   };

   const a = new Person('Alice');
   const b = new Person('Bob');
   console.log(a.greet()); // "Hi, I'm Alice"
   console.log(a.greet === b.greet); // true -> same function on prototype
   ```

2. **`class` syntax (ES6) - syntatic sugar**
   
   ```javascript
   class Person {
   constructor(name) { this.name = name; }
     greet() { return `Hi, I'm ${this.name}`; }
   }


   const p = new Person('Sam');
   console.log(Object.getPrototypeOf(p) === Person.prototype); // true
   ```
   `class` does under-the-hood the same prototype wiring as constructor functions; it's mainly clearer syntax + some niceties (like `super`).

3. **`Object.create()` — pure delegation**
   
   ```javascript
    const animalProto = {
      speak() { return `${this.name} makes a noise` }
    };


    const dog = Object.create(animalProto);
    dog.name = 'Rex';
    console.log(dog.speak()); // Rex makes a noise
   ```
   `Object.create(proto)` creates an object whose `[[Prototype]]` is `proto` without needing constructor functions. Great for composing behavior and avoiding `new`.

4. **`Object.create()` — pure delegation**
   
   ```javascript
    function createUser(name) {
    const user = Object.create(userMethods);
    user.name = name;
    return user;
    }


    const userMethods = {
    sayName() { return this.name; }
    };
   ```
   This pattern avoids classes and is explicit about delegation.

### 4. Best practice and cautions
1. **Best practices**
   * Prefer composition over deep inheritance chains. Use small objects that compose behavior instead of long prototype chains.
   * Put methods on prototypes, not inside constructors, unless method needs unique per-instance closure state.
   * Use Object.create(null) for plain hash maps to avoid prototype collisions.
  
2. **Cautions**
   * **Never modify built-in prototypes** (like `Array.prototype`) in production libraries—this can break other code. (Acceptable in small apps or polyfills with caution.)
   * **Be careful with this**: methods expect `this` to be the instance. If you pass methods as callbacks, `this` can be lost — use `.bind()`, arrow functions (but arrow functions have lexical `this` and cannot be used as prototype methods expecting dynamic `this`), or call with `.call`/`.apply`.
   * **Arrow functions as prototype methods**: Don’t use `arrow` functions when you expect `this` to refer to the instance — arrow functions capture `this` lexically at creation time.


## 8. Event Loop, Call Stack, Microtasks & Macrotasks (Deep Notes)

### 1. The Core Concept — How JavaScript Executes Code
JavaScript is **single-threaded** — meaning it has only one main thread of execution. That means **only one piece of code runs at a time** in the main thread.

But JavaScript also handles asynchronous operations (like timers, network calls, DOM events) without blocking. This is achieved using the **event loop, call stack, and task queues**.

### 2. Call Stack — The Execution Stack
The `call stack` is a data structure that keeps track of where JavaScript is in program execution.
* When a function is invoked, it’s pushed onto the stack.
* When the function finishes, it’s popped off.

**Example**

```javascript
function greet() {
console.log('Hello');
}

function start() {
greet();
console.log('Start done');
}

start();
```

**Step-by-step execution:**
1. Global execution context created → pushed to the stack.
2. `start()` is called → pushed to the stack.
3. Inside `start()`, `greet()` is called → pushed to the stack.
4. `console.log('Hello')` runs → printed.
5. `greet()` finishes → popped off.
6. `console.log('Start done')` runs.
7. `start()` finishes → popped off.
   * The **call stack** executes code synchronously — one frame at a time.

If a long-running operation (like a network call or timer) were handled here, it would block everything. That’s where the **event loop** comes in.

### 3. Event Loop 
The **event loop** is a constantly running process that checks **if the call stack is empty**, and if so, **pushes the next task from the queues** (micro or macro) to be executed.

It checks tasks in the following order:
1. Execute all tasks in the **call stack**.
2. When stack is empty, run all **microtasks** (if any).
3. Then take one **macrotask** (if any) and execute it.
4. Repeat the cycle.

### 4. Microtasks vs Macrotasks

1. **Microtasks**:
   * Come from **Promise callbacks**, `queueMicrotask()`, and **MutationObserver**.
   * Run **immediately after** the current stack is empty but **before** the next macrotask.
   * They have **higher priority** — meaning they are executed before rendering or timers.
   * 
2. **Macrotasks**: 
   * Come from **setTimeout**, **setInterval**, **setImmediate (Node.js)**, and **I/O events**.
   * Run after the microtask queue is empty.

### 5. Understanding Event Loop by code

1. **Mixing Promises and setTimeout**
    ```javascript
    console.log('1');
    setTimeout(() => console.log('2'), 0);
    Promise.resolve().then(() => console.log('3'));
    console.log('4');
    ```
    **Step-by-step breakdown**:
    1. `console.log('1')` → runs immediately (sync).
    2. `setTimeout(..., 0)` → callback scheduled as a **macrotask**.
    3. `Promise.resolve().then(...)` → callback scheduled as a **microtask**.
    4. `console.log('4')` → runs (sync).
    5. Stack is now empty.
    6. **Event loop** checks **microtask** queue → finds `console.log('3')` → runs.
    7. Then moves to **macrotask** queue → runs `console.log('2')`.
    
    **Output**:

    ```
    1
    4
    3
    2
    ```
    Even though both `setTimeout` and `Promise` are asynchronous, **microtasks always run before macrotasks**.

## 9. LocalStorage, SessionStorage & Cookies

### 1. The Core Idea — Why Client-Side Storage Exists
In web applications, you often need to store small amounts of data on the user’s browser — such as authentication tokens, preferences, shopping cart items, or session state — **without relying on the server every time.**

Before HTML5, **cookies** were the only way to persist data in the browser. But cookies have limitations:
 * They’re sent with every HTTP request (increasing payload size).
 * They have a small storage limit (~4KB).
 * Managing them can be clunky.

To solve this, **Web Storage API** introduced:
 * `localStorage` — persistent storage that remains even after the browser is closed.
 * `sessionStorage` — temporary storage that lasts until the browser tab is closed.

### 2. LocalStorage — Persistent Key-Value Storage
`localStorage` allows web apps to store data persistently in the browser across sessions and tabs. Data persists even after a page refresh or closing the browser.

#### Example 1. Storing and Retrieving Data

```javascript
// Save user preferences
localStorage.setItem('theme', 'dark');
localStorage.setItem('fontSize', '16px');

// Retrieve
const theme = localStorage.getItem('theme');
const fontSize = localStorage.getItem('fontSize');

console.log(`Theme: ${theme}, Font Size: ${fontSize}`);

// Remove a single item
localStorage.removeItem('fontSize');

// Clear everything
localStorage.clear();
```

**Explanation**
   * setItem(key, value) → stores as string key-value pairs.
   * Values are persisted per domain, not per page.
   * All data is stringified, so you need JSON.stringify() for objects.
  
#### Example 2. Storing Complex Data Objects

```javascript
const user = {
name: 'Alice',
loggedInAt: new Date().toISOString(),
roles: ['admin', 'editor']
};

// Store
localStorage.setItem('user', JSON.stringify(user));

// Retrieve
const userData = JSON.parse(localStorage.getItem('user'));
console.log(userData.name); // Alice
```

**Use `localStorage` when**:
* You need **persistent** client-side state.
* Examples:
  * Dark mode / language preference.
  * Caching small JSON API responses.
  * Remembering user progress (e.g., learning apps).

### 3. SessionStorage - Temporary Tab-Scoped Storage
`sessionStorage` works just like `localStorage` but the data lasts **only until the tab or window is closed**. It’s not shared across tabs.

**Example**:

```javascript
// Set and get temporary session data
sessionStorage.setItem('sessionId', 'abc123xyz');
console.log(sessionStorage.getItem('sessionId')); // abc123xyz

// On reload, data persists in same tab
// On new tab or close, data is gone
```
**Real-World Example**:
In **banking or payment gateways**, temporary data like a transaction ID or OTP session is stored in `sessionStorage`. This ensures that if a user opens another tab or closes the page, the sensitive data doesn’t persist.

```javascript
function initializePaymentSession(userId) {
const sessionToken = crypto.randomUUID();
sessionStorage.setItem('paymentSession', JSON.stringify({ userId, sessionToken }));
return sessionToken;
}
```

**Use sessionStorage when**:
* You need temporary, tab-specific data.
* Examples:
  * Current checkout session in an e-commerce flow.
  * Form data in a single-page flow.
  * Temporary OAuth flow tokens.

### 4. Cookies - The Old Guard of Browser Storage
Cookies are small key-value pairs stored in the browser and **sent with every HTTP request** to the same domain. They are used heavily for authentication and tracking.

**Example**:

```javascript
// Set cookie (expires in 7 days)
const expiry = new Date();
expiry.setDate(expiry.getDate() + 7);

document.cookie = `token=abc123; expires=${expiry.toUTCString()}; path=/; Secure; SameSite=Strict`;

// Read cookie
console.log(document.cookie); // token=abc123
```

**Explanation**:
   * Each cookie has attributes: expires, path, domain, Secure, and SameSite.
   * Secure means cookie is only sent over HTTPS.
   * HttpOnly means it’s not accessible via JavaScript (helps prevent XSS).

**Example 2**:

```javascript
function getCookie(name) {
return document.cookie
.split('; ')
.find(row => row.startsWith(name + '='))
?.split('=')[1];
}

console.log(getCookie('token'));
```

**Use cookies when**:
* You need server-accessible data or secure authentication tokens.
* Examples:
  * JWTs or session IDs for login.
  * Analytics tracking.
  * CSRF protection tokens.

## 10. DOM manipulation
DOM manipulation is indeed one of the **most essential and powerful concepts** in JavaScript.

### 1.What is DOM:
**DOM (Document Object Model)** is an **in-memory** representation of the **HTML structure of a web page**.
When a browser loads a webpage, it parses the HTML and **creates a tree-like structure** — where every HTML element becomes a node (an object) that can be **accessed and manipulated via JavaScript**.

Before the DOM, HTML pages were **static** — no interactivity.
If you wanted to change content, you had to **reload the page**.
The DOM was introduced so JavaScript could **dynamically interact** with web pages —
add/remove elements, handle user actions, and update content **without page reloads**.

**Problems Solved by DOM**:
  * Enable **dynamic rendering** (e.g., add new cards, images, forms).
  * Handle **user interactions** in real-time (clicks, scrolls, inputs).
  * Power **Single Page Applications** (SPAs).

### 2. Internal Working — DOM Tree
When HTML is parsed:

```html
<html>
  <body>
    <div id="app">
      <h1>Hello World</h1>
      <button>Click Me</button>
    </div>
  </body>
</html>
```

It becomes a DOM Tree:

```less
Document
└── html
    └── body
        └── div#app
            ├── h1
            └── button
```

Every tag = a node, and you can access or modify it with JavaScript like:

```javascript
document.getElementById('app').children[0].textContent = "Hi Developer!";
```

### 3. DOM manipulation example - Mini task manager

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>DOM Manipulation Example</title>
  <style>
    .task { padding: 8px; margin: 6px 0; border-radius: 5px; background: #f5f5f5; }
    .completed { text-decoration: line-through; color: gray; }
  </style>
</head>
<body>
  <div id="app">
    <h2 id="title">Task Manager</h2>
    <input id="taskInput" type="text" placeholder="Add new task" />
    <button id="addBtn">Add Task</button>
    <ul id="taskList"></ul>
    <button id="clearAll">Clear All</button>
  </div>

  <script>
    // ======== DOM SELECTORS =========
    const input = document.getElementById('taskInput');
    const addBtn = document.querySelector('#addBtn');
    const list = document.querySelector('#taskList');
    const title = document.getElementById('title');
    const clearAll = document.getElementById('clearAll');

    // ======== MODIFY CONTENT =========
    title.textContent = "📝 My Task Manager";
    title.style.color = "#2a9d8f";

    // ======== EVENT HANDLER =========
    addBtn.addEventListener('click', () => {
      if (!input.value.trim()) return;

      // ======== CREATE ELEMENTS =========
      const li = document.createElement('li');
      li.classList.add('task');
      li.textContent = input.value;

      // Add a delete button dynamically
      const delBtn = document.createElement('button');
      delBtn.textContent = "❌";
      delBtn.style.marginLeft = "10px";
      li.appendChild(delBtn);

      // Append to DOM (insert at end)
      list.appendChild(li);

      // ======== TRAVERSAL EXAMPLE =========
      console.log("Parent:", li.parentElement.id);  // taskList
      console.log("First child of list:", list.firstElementChild?.textContent);

      // ======== EVENT DELEGATION =========
      li.addEventListener('click', (e) => {
        if (e.target.tagName === 'BUTTON') {
          // Delete this li
          list.removeChild(li);
        } else {
          // Toggle completed
          li.classList.toggle('completed');
        }
      });

      input.value = '';
    });

    // ======== REMOVE ALL TASKS =========
    clearAll.addEventListener('click', () => {
      // Use DocumentFragment for demonstration
      const fragment = document.createDocumentFragment();
      const msg = document.createElement('p');
      msg.textContent = "All tasks cleared ✅";
      msg.style.color = "#e76f51";
      fragment.appendChild(msg);

      list.replaceChildren(); // Clears all child elements
      list.appendChild(fragment);
    });

    // ======== ATTRIBUTE MANIPULATION =========
    input.setAttribute('maxlength', '50');
    console.log(input.getAttribute('maxlength')); // 50
  </script>
</body>
</html>

```

