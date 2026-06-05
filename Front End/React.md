# React Notes

React is a JavaScript library for building interactive UI rather manually selecting DOM elements and updating them yourself.

If classical browser JavaScript often feels like:

- find the element
- listen for an event
- manually change the DOM

React changes the workflow to:

- store data in state
- describe the UI for that state
- let React update the DOM

---

## Suggested learning path

This document is organized in the order that tends to make React easiest to understand:

1. React foundations
2. Components and JSX
3. Props and state
4. Events and forms
5. Conditional rendering and lists
6. Effects and data fetching
7. Styling and UI organization
8. Component design patterns
9. Reusable logic with custom hooks
10. Common feature-building patterns
11. React with TypeScript
12. Debugging mindset and common mistakes

---

## 1. React foundations

### Imperative vs Declarative UI

#### Imperative UI

Imperative code tells the computer the step-by-step instructions to change the interface.

Example:

```js
count = count + 1;
countText.textContent = count;
button.disabled = count > 10; //button.disabled = true when count>10
message.textContent = count > 10 ? "Limit reached" : "";
```

#### Declarative UI

Declarative code describes what the UI should look like for the current state/data.

A React-style mental model looks more like:

```js
if (count > 10) {
  // UI should show "Limit reached"
  // UI should disable the button
} else {
  // UI should hide the message
  // UI should keep the button enabled
}
```

In React code, you do not directly write to DOM nodes. Instead, you return UI based on state, and React applies the right DOM changes.

### UI as a function of state

This idea is worth remembering exactly:

**UI = function(state)**

That means the screen is determined by the current data.

If state changes, the UI should change too.

Example idea:

- if `isLoggedIn` is `true`, show the dashboard
- if `isLoggedIn` is `false`, show the login screen
- if `isLoading` is `true`, show a spinner
- if `items` has 5 values, render 5 list items

In React, state is the source of truth for the UI.

That means you try not to manually force the DOM into the correct appearance. Instead, you update state, and React derives the UI from that state.

This is why React code often feels more predictable once you understand it.

### Components

A component is a reusable piece of UI.

You can think of a component as a JavaScript function that returns interface markup.

For example, instead of thinking of a page as one giant HTML file, React encourages thinking in reusable pieces such as:

- `Navbar`
- `Sidebar`
- `ProductCard`
- `TodoItem`
- `LoginForm`

This is useful because each piece can:

- have its own purpose
- receive its own data
- manage some local behavior
- be reused in multiple places

A simple component looks like this:

```jsx
function Welcome() {
  return <h1>Hello</h1>;
}
```

This is just a function, but instead of returning a number or string, it returns UI.

### JSX

JSX is the syntax React uses that looks like HTML inside JavaScript.

Example:

```jsx
function Welcome() {
  return <h1>Hello, world</h1>; //looks like HTML but is JSX, later compiled to react using Babel or SWC(Next.js)
}
```

### Rendering and Re-rendering

Rendering means React calls your component function, the component rerturns JSX and react uses that to update the browser DOM.

Re-rendering means React runs the component again because the state changed, prop changed or the parent component re-rendered.

### Plain JS vs React Mental Model

#### Plain JavaScript

```html
<button id="toggle">Toggle</button>
<p id="message"></p>
```

```js
let isVisible = false;

const button = document.getElementById("toggle");
const message = document.getElementById("message");

button.addEventListener("click", function () {
  isVisible = !isVisible;
  message.textContent = isVisible ? "Now visible" : "";
});
```

#### React-style thinking

```jsx
function App() {
  const isVisible = true;

  return (
    <div>
      <button>Toggle</button>
      {isVisible ? <p>Now visible</p> : null}
    </div>
  );
}
```

This example is simplified, but the main idea is visible:

- the UI is expressed from data
- visibility is controlled by state/values
- you do not directly set `textContent`

---

## 2. Components and JSX

### Function components in real files

Most teams place one main component per file and export it.

Example:

```jsx
// ProductCard.jsx
export default function ProductCard() {
  return <article>...</article>;
}
```

For utility or shared UI pieces, named exports are also common:

```jsx
export function PriceTag() {
  return <span>...</span>;
}
```

Practical conventions:

- use `PascalCase` for component names and filenames
- keep render output predictable (same inputs -> same output)
- avoid doing side effects directly during render
- move non-UI helper logic outside the component when possible

### Nesting and composing components

Composition is most useful when each component has one responsibility.

Instead of one large component:

- `ProductPage` handles page layout
- `ProductGallery` handles images
- `ProductInfo` handles text/details
- `AddToCartButton` handles button UI

A small composition example:

```jsx
function ProductPage() {
  return (
    <main>
      <ProductGallery />
      <ProductInfo />
      <AddToCartButton />
    </main>
  );
}
```

When reviewing component structure, ask:

- can this section be reused elsewhere?
- is one component doing too many unrelated things?
- does each component have a clear name tied to one purpose?

### JSX rules that usually cause bugs

1. **Return one parent node from a component**

Use one wrapper element or a fragment.

1. **Use valid JSX attribute names**

- `className` instead of `class`
- `htmlFor` instead of `for`
- camelCase event/DOM props like `onClick`, `tabIndex`

1. **All tags must close**

```jsx
<img src="/avatar.png" alt="avatar" />
```

1. **Use parentheses for multiline return JSX**

```jsx
return (
  <section>
    <h2>Title</h2>
  </section>
);
```

This avoids JavaScript automatic semicolon insertion mistakes.

### JavaScript expressions inside JSX

Inside JSX braces (`{}`), you can use expressions, not statements.

Valid examples:

```jsx
<h1>{userName}</h1>
<p>{price * quantity}</p>
<p>{isOpen ? "Open" : "Closed"}</p>
```

Invalid directly inside JSX:

- `if (...) { ... }`
- `for (...) { ... }`

If you need statements, calculate values before `return`:

```jsx
function Status({ isOpen }) {
  let text = "Closed";
  if (isOpen) text = "Open";

  return <p>{text}</p>;
}
```

### Fragments and wrapping elements

Use fragments when you only need grouping and no extra DOM node.

```jsx
function HeaderBlock() {
  return (
    <>
      <h1>Dashboard</h1>
      <p>Welcome back</p>
    </>
  );
}
```

Use semantic wrappers (`<section>`, `<article>`, `<nav>`) when structure or accessibility meaning matters.

Also remember:

- short fragment syntax `<>...</>` cannot take props
- use `React.Fragment` when you need a `key`

```jsx
items.map((item) => (
  <React.Fragment key={item.id}>
    <dt>{item.label}</dt>
    <dd>{item.value}</dd>
  </React.Fragment>
));
```

These habits keep JSX readable now and make later topics (props, state, lists, effects) easier to reason about.

---

## 3. Props and state

This section will explain how components receive input through props and manage changing local data through state.

Planned expansion topics:

- what props are
- one-way data flow
- local component state
- why state updates trigger re-renders
- props vs state

---

## 4. Events and forms

This section will connect browser events from plain JavaScript to React event handling and form management.

Planned expansion topics:

- click handlers
- input/change events
- controlled inputs
- form submission
- preventing default behavior

---

## 5. Conditional rendering and lists

This section will show how React renders different UI based on conditions and how it builds repeated UI from arrays.

Planned expansion topics:

- `if` logic and ternaries
- showing/hiding UI
- rendering arrays with `.map()`
- keys and why they matter
- empty states

---

## 6. Effects and data fetching

This section will explain how React handles work that touches the outside world, such as timers, subscriptions, and API requests.

Planned expansion topics:

- why effects exist
- syncing with outside systems
- data fetching basics
- loading and error states
- effect cleanup

---

## 7. Styling and UI organization

This section will cover the common ways React components are styled and organized in real projects.

Planned expansion topics:

- CSS files
- conditional classes
- inline styles
- component folder structure
- separating UI and utility code

---

## 8. Component design patterns

This section will focus on structuring React apps so that data flow stays understandable as the UI grows.

Planned expansion topics:

- lifting state up
- parent/child communication
- controlled vs uncontrolled patterns
- shared state placement
- breaking large components into smaller ones

---

## 9. Reusable logic with custom hooks

This section will explain how reusable stateful behavior can be extracted into custom hooks.

Planned expansion topics:

- what a custom hook is
- when to extract one
- separating logic from presentation
- examples like form state or data fetching

---

## 10. Common feature-building patterns

This section will connect React concepts to the kinds of features you actually build in applications.

Planned expansion topics:

- counters and toggles
- todo lists
- modal visibility
- tabbed interfaces
- loading/error/success UI
- API-driven lists

---

## 11. React with TypeScript

This section will build on `TS.md` and show how types improve React code clarity and safety.

Planned expansion topics:

- typing props
- typing state
- typing event handlers
- typing API data
- when to infer vs when to annotate

---

## 12. Common mistakes and debugging mindset

This section will cover the issues React beginners commonly face and how to reason through them.

Planned expansion topics:

- confusing props and state
- mutating arrays/objects directly
- expecting state updates to feel synchronous
- missing keys in lists
- effect misuse
- reading errors from the component tree

---

## 13. How to continue from here

Use this document in layers instead of trying to memorize everything at once.

Suggested order:

1. Understand the React foundation section until the declarative model feels natural
2. Learn components and JSX
3. Learn props and state
4. Practice events, forms, and list rendering
5. Then move to effects and data fetching
6. After that, connect React to TypeScript

If the foundation section is clear, the rest of React will feel far more logical instead of feeling like disconnected APIs.
