JavaScript is the language that gives a webpage behavior, logic, and interactivity.

It can:

- Read and change the HTML structure (the DOM)
- Read and change CSS classes/styles
- Respond to user actions like clicks, typing, scrolling, and form submission
- Store and update data while the page is running
- Fetch data from servers/APIs

---

## Intro to JavaScript

In the browser, JavaScript runs inside the page and can work with the browser's built-in APIs, such as:

- `document` for reading/changing HTML DOM
- `window` for browser-level features
- `console` for debugging
- `fetch()` for HTTP requests
- `setTimeout()` / `setInterval()` for scheduling work

Introductory Example:

```html
<button id="btn">Click me</button>
<p id="message">Hello</p>
```

```js
const button = document.getElementById("btn"); // 
const message = document.getElementById("message");

button.addEventListener("click", function () {
  message.textContent = "Button clicked!";
});
```

Here JavaScript:

1. Finds elements in the page
2. Listens for a click
3. Changes the page when the click happens

---

## Core language ideas

### Variables

Variables store values.

```js
const name = "Anupreet";
let count = 0;
```

- `const` means the variable should not be reassigned, even to another value of same type
- `let` means the variable can be reassigned. But type cannot be changed in Typescript. 

Use `const` by default, and `let` when the value needs to change.

### Primitive Types

Common primitive types in JavaScript:

- `string` - text like `"hello"`
- `number` - values like `10`, `3.14`
- `boolean` - `true` or `false`
- `undefined` - a variable exists but has no value yet
- `null` - intentional "empty" value

```js
const username = "Sam";
const age = 22;
const isLoggedIn = true;
let selectedItem = null;
```

### Objects

Objects group related data using `key: value` pairs. They also hold functions which is discussed later on.

```js
const user = {
  name: "Sam",
  age: 22,
  isAdmin: false,
};

console.log(user.name); 
```

### Arrays

Arrays hold ordered lists of values.

```js
const fruits = ["apple", "banana", "orange"];

console.log(fruits[0]); // "apple"
```

Arrays are commonly used with loops and methods like:

- `.map()`, for calling a function on each element and returning an array with returns values from the function
- `.filter()`, keeps only items that pass a test, in the resulting array. 
- `.find()`, calls functions for each element and returns first element for which function returns true
- `.forEach()`, for calling function once per array element 

Example:

```js
const numbers = [1, 2, 3];
const doubled = numbers.map(function (n) {
  return n * 2;
});

console.log(doubled); // [2, 4, 6]
```


### Functions

Functions store reusable behavior.

```js
function add(a, b) {
  return a + b;
}

const result = add(2, 3);
```

Functions can also be stored in variables:

```js
//greet holds a reference to the function so we can call the function using greet()
const greet = function (name) {
  return "Hello " + name;
};
```

And with arrow function syntax:

```js
// instead of writing the keyword "function" just use => in js
const greet = (name) => {
  return "Hello " + name;
};
```

Short form(single line expression):

```js
const greet = (name) => "Hello " + name;
```
---

## Control flow

JavaScript uses normal programming control flow:

### Conditionals

```js
const age = 20;

if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}
```

### Loops

```js
const items = ["a", "b", "c"];

for (const item of items) {
  console.log(item);
}

// Incremental iterator loop (index-based)
for (let i = 0; i < items.length; i++) {
  console.log(i, items[i]);
}
```

---

## The DOM and classical browser JavaScript

Before libraries like React became popular, a lot of frontend development was done with **direct DOM manipulation**.

That classical pattern usually looked like this:
1. Write HTML for the page
2. Select elements with JavaScript
3. Listen for events
4. Manually update the DOM when data changes

Example:

```html
<input id="nameInput" type="text" />
<button id="saveBtn">Save</button>
<p id="output"></p>
```

```js
const input = document.getElementById("nameInput"); //input is a const variable that holds DOM node reference for the input element with id="nameInput".
const button = document.getElementById("saveBtn");
const output = document.getElementById("output");

button.addEventListener("click", function () {
  output.textContent = "Hello, " + input.value;
});
```

This is "classical" browser JavaScript:

- You manually grab elements
- You manually read values from them
- You manually update the screen

This works well for small pages, but as apps grow, it becomes harder to manage:

- State gets scattered across variables and DOM nodes
- UI updates become repetitive
- Different parts of the page can get out of sync

React exists largely to improve this workflow.

---

## Events

JavaScript in the browser is event-driven.

That means code often runs in response to something happening:

- click
- input
- submit
- keydown
- change
- load

Example:

```js
const form = document.getElementById("signupForm");

form.addEventListener("submit", function (event) {
  event.preventDefault();  // event.preventDefault() stops the browser's default behavior, such as reloading the page on form submission.
  console.log("Form submitted");
});
```

React also works heavily with events, but wraps them in a component-based system.

---

## State in plain JavaScript

State means "the data your UI depends on right now".

Example state:

- Whether a modal is open
- The current value of an input
- A list of todos
- Whether data is loading

In plain JavaScript, state is often just variables and objects:

```js
let count = 0;

const countText = document.getElementById("count");
const button = document.getElementById("increment");

button.addEventListener("click", function () {
  count = count + 1;
  countText.textContent = count;
});
```

Notice the key idea:

- Data changed on js side is `count` when the increment button is clicked 
- Then the UI had to be manually updated: `countText.textContent = count`

This manual syncing is one of the main problems React helps solve.

---

## Scope and closures

### Scope

Scope means "where a variable is available".

```js
function example() {
  const message = "hello";
  console.log(message);
}

console.log(message); // Error
```

`message` only exists inside the function.

### Closures

A closure happens when a function remembers variables from the place where it was created.

```js
function makeCounter() {
  let count = 0;

  return function () {
    count = count + 1;
    return count;
  };
}

const counter = makeCounter(); // returns reference to the function defined inside the return statement. The function closed over the variable count so now as long as counter exists, the enginer keeps the value of count in memory. 
//You execute the function
console.log(counter()); // 1 
console.log(counter()); // 2 
```

Closures are very important in modern JavaScript and help explain how callbacks, hooks, and component logic work.

---

## this, classes, and the classical OOP style

JavaScript supports objects and classes, and older codebases often use a more class-based style.

```js
class User {
  constructor(name) {
    this.name = name; //`this` refers to the current object instance in class methods
  }

  greet() {
    return "Hello " + this.name;
  }
}

const user = new User("Sam"); //`new` is used to create an instance of a class
console.log(user.greet());
```

Older React code often used **class components**, whereas Modern React mostly uses **functions and hooks** instead.

---

## Asynchronous JavaScript

JavaScript often needs to do work that finishes later:

- fetching API data
- waiting for a timer
- handling user actions

### Callbacks

```js
//setTimeout is a browser side or Node.js API function that allows us to callback(run something) after a certain time without hindering other processes
setTimeout(function () {
  console.log("Runs later");
}, 1000); //here the delay is 1000 ms(1 sec) before running the function defined inside. 
```

### Promises

```js
//fetch also comes from browser side API and sends an HTTP request to the given URL and returns a Promise: an object that represents a value(or error that will be available later )
fetch("/api/users")
  //.then is a method on a Promise with its first argument being a function that runs with the promis value
  .then(function (response) { // response is the raw HHTP response
    return response.json(); //.json() reads response as JSON and returns another Promise
  })
  .then(function (data) { //takes in parsed .json
    console.log(data); //displays parsed .json in console
  });
```

### async/await
`async/await` is just a cleaner way to work with promises.
```js
//declares an async function enables use of `await` inside the function to wait on Promises
async function loadUsers() {
  const response = await fetch("/api/users"); //Starts the fetch HTTP request and pauses the loadUsers() function until the request settles and response is stored in const response
  const data = await response.json(); 
  console.log(data);
}
```
This matters in React because components often fetch and display async data.

---

## Modules

Modern JavaScript is usually split into files called modules.

One file can export values:

```js
// `export` keyword makes a variable/functions/class, etc part of the module's public API so other files can import it.
export const API_URL = "/api";

export function add(a, b) {
  return a + b;
}
```

Another file can import them:

```js
import { API_URL, add } from "./utils.js";
```

React apps use this constantly: components, utility functions, hooks, and types are all usually separated into modules.

---

## How JavaScript is used classically

Before React, a classical frontend app often followed this mindset:

### 1. The HTML already exists

You start with HTML in the page.

### 2. JavaScript finds pieces of the page

Using things like:

- `document.getElementById()`
- `document.querySelector()`
- `document.querySelectorAll()`

### 3. JavaScript attaches behavior

Usually with:

- `addEventListener()`
- inline logic
- utility functions

### 4. JavaScript updates the page directly

Using things like:

- `textContent`
- `innerHTML`
- `classList.add()` / `classList.remove()`
- `style.propertyName`
- `appendChild()`

### 5. State and UI syncing are manual

If data changes, you must decide what to update and when.

That is the key contrast with React.

---

## The mental bridge to React

To understand React, it helps to compare the two styles.

### Plain JavaScript thinking

"When something happens, find the element and manually update it."

Example:

```js
count++;
countText.textContent = count;
```

### React thinking

"When state changes, describe what the UI should look like for that state."

React then updates the DOM for you.

So instead of:

- manually selecting nodes
- manually changing text
- manually toggling classes everywhere

You usually:

- store state
- render UI from that state
- let React handle DOM updates

This is one of the biggest mindset shifts.

---





## A simple way to study next

Study in this order:

1. JavaScript values, variables, functions, arrays, and objects
2. DOM selection, events, and updating the page
3. Modern JavaScript syntax (`=>`, destructuring, modules, async/await)
4. React components, props, and state
5. TypeScript types for functions, objects, props, and events
