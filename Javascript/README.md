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
  **Promise Methods**
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