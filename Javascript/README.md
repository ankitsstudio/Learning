# JavaScript Key Concepts to Always Keep in Mind

This README covers some fundamental JavaScript concepts that are essential for writing efficient, clean, and maintainable code. These concepts include **Throttling and Debouncing**, **Event Delegation**, and **Closures**. Each section contains explanations, code examples, and practical thoughts to deepen your understanding.

***

# 1. Throttling and Debouncing

These techniques help control how often a function is executed in response to events, which is crucial for performance optimization, especially in events that fire rapidly like scrolling or resizing.

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

# 2. Event Delegation

Event Delegation involves adding a single event listener to a common parent element instead of multiple listeners to individual child elements. The event bubbles up from the target element to the parent, where you can handle it efficiently.

* Advantages:
* Reduces memory usage.
* Supports dynamic elements added after initial page load.
* Simplifies event management.

* Example: Handling clicks on dynamic list items

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

**Combining concepts:** Event delegation often works well with throttling or debouncing, especially when dealing with scroll or input events on many elements.

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

# 3. Closure

A **closure** gives you access to an outer function’s scope from an inner function, even after the outer function has executed. This is crucial for creating private variables and stateful functions.

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

**Memory management:** Closures are powerful but can cause memory leaks if references to outer scopes persist unnecessarily—be mindful when using closures inside long-running apps.