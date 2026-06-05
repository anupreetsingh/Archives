CSS is the language that helps us controls the visual appearance of  HTML elements.

# Linking CSS to HTML

You can add CSS in three ways:

1. **External stylesheet** — Best for real projects. In `<head>`:  
   `<link rel="stylesheet" href="styles.css">`  
   The `.css` file contains only CSS (no HTML tags).

2. **Internal (embedded)** — Inside `<head>`:  
   `<style> ... CSS rules ... </style>`

3. **Inline** — On a single element:  
   `<p style="color: red;">...</p>`  
    Use sparingly; hard to maintain and overrides are messy.


# Rules
Using CSS you set **rules**. A rule is one complete instruction, it has:
  - **Selector** that decides which element(s) the rule applies to, and a
  - **Declaration block** which has one or more `property: value;` pairs in curly braces.

For example:
```
p { 
  color: blue; 
  font-size: 16px; 
}
``` 
is one rule: the selector is `p`, and the declaration block has multiple property:value pairs (`color: blue;` and `font-size: 16px;`).

## Selectors

A **selector** picks which element(s) the rule applies to. It can be implemented in a variety of ways:

### 1. Basic selectors

| Selector      | Example        | Targets                                      |
| ------------- | -------------- | -------------------------------------------- |
| **Element**   | `p { }`        | All `<p>` elements.                           |
| **Class**     | `.intro { }`   | Elements with `class="intro"`. Reusable.      |
| **ID**        | `#main { }`    | The element with `id="main"`. One per page.   |
| **Universal** | `* { }`        | Every element. Use with care.                 |

**Targeting elements that have Multiple classes:** One HTML element can have several class names in its `class` attribute, separated by spaces, e.g. `class="intro highlight"`. In CSS you can target it with any single class (`.intro` or `.highlight` — both will match this element) or with the combined selector `.intro.highlight` (no space), which matches only elements that have *both* classes.

**Applying Same rules for Multiple selectors:** A comma (`,`) between selectors groups them so they share the same declaration block. So `p, .intro { color: blue; }` applies to every `<p>` element *and* every element with `class="intro"` (including `<p class="intro">`). All of them get `color: blue`. Use this to avoid repeating the same declaration blocks for different selectors.

### 2. Combinator Selectors

**Combinators** are symbols used to target elements that stand in a certain relationship to another element in the HTML(DOM). There are four kinds of combinators:

| Combinator | Example       | Meaning                                      |
| ---------- | ------------- | -------------------------------------------- |
| **Descendant** | `div p { }` | Any `p` inside a `div` (child, grandchild, …). |
| **Child**      | `div > p { }` | Only `p` that are **direct** children of `div`. |
| **Adjacent sibling** | `h1 + p { }` | The first `p` that immediately follows `h1`. |
| **General sibling** | `h1 ~ p { }` | Any `p` that comes *after* `h1` (meaning they both have the same parent; and appear in the given order). |

### 3. Attribute selectors

**Attribute selectors** target elements based on their HTML attributes — either the presence of an attribute, its exact value, or a pattern (starts with, ends with, contains). They use square brackets `[ ]`. Common patterns:

- `[href]` — has `href` attribute.
- `[type="text"]` — `type` equals `"text"`.
- `[class^="btn"]` — `class` **starts with** `"btn"`.
- `[class$="btn"]` — **ends with** `"btn"`.
- `[class*="btn"]` — **contains** `"btn"`.

### 4. Pseudo-classes (state or position)

A **pseudo-class** is a keyword you add to a selector (with a single colon `:`) so the rule applies only when a certain **condition** is true — for example the element is hovered, focused, or is the first child. You’re not selecting a different element; you’re selecting the same element in a particular state or position. Syntax: `selector:pseudo-class`.

| Pseudo-class   | Use case                          |
| -------------- | --------------------------------- |
| `:hover`       | Mouse over the element.            |
| `:focus`       | Element has keyboard focus.        |
| `:active`      | Element is being clicked.          |
| `:first-child` | Element is the first child.        |
| `:last-child`  | Element is the last child.        |
| `:nth-child(n)`| nth child (e.g. `2`, `odd`, `2n`).|
| `:not(selector)` | Elements that do **not** match. |

Example: `a:hover { color: red; }`

### 5. Pseudo-elements (part of an element)

**Pseudo-element** is a keyword you add to an element(with a double colon `::`) so the rule applies to **part** of the element (e.g. the first line or letter) or a virtual “fake” element (e.g. `::before`, `::after`) — so you’re targeting something that isn’t a full standalone HTML element. Syntax: `selector::pseudo-element`.

| Pseudo-element | Meaning                          |
| -------------- | -------------------------------- |
| `::before`     | Insert content before the element. |
| `::after`      | Insert content after the element.  |
| `::first-line` | First line of text.               |
| `::first-letter` | First letter.                   |

`::before` and `::after` need `content: ""` (can be empty) to appear.

---
### Specificity and Order

When more than one rule targets the **same element**, the **overlapping** properties contend and the final value for those are decided by **specificity** and **order**. All other properties in each rule’s block still apply; they are merged. 

- **Specificity** — The “strength” of the selector. Stronger selectors override weaker ones for overlapping properties. Order (strongest to weakest):
  1. Inline style (e.g. `style="..."`)
  2. ID (e.g. `#id`)
  3. Class, attribute, pseudo-class (e.g. `.class`, `[href]`, `:hover`)
  4. Element, pseudo-element (e.g. `p`, `::before`)
  

So if `p { color: blue; font-size: 16px; }` and `#news { color: red; margin: 20px; }` both match with `<p id="news">`, the two rules overlap on `color` (ID wins → red), but `font-size` and `margin` are not overlapping, so the element gets `font-size: 16px` from the first rule and `margin: 20px` from the second.
- **Order** — If two rules have the **same** specificity, the one that appears **later** in the CSS (or in the cascade) wins for any overlapping properties. So when specificity is tied, the last rule applies.

> Prefer classes over IDs for styling because that leaves more overhead for overriding using high selectors with higher specificity or using selectors later


> `!important` — Adding `!important` to a declaration (e.g. `color: red !important;`) forces that value to win over normal rules, including higher-specificity ones (except another `!important` with equal or higher specificity). Use it sparingly. 

---

## Declaration Block

### 1. The Box model and sizing

The first thing to understand is that every element is drawn as a **box** with four **conceptual areas** (from inside out). These are not properties but we style them with CSS as discussed in the sections below.

| Area      | Meaning | Properties that style it |
| --------- | ------- | ------------------------- |
| **Content** | Text or inner content | `width`, `height`, `box-sizing` |
| **Padding** | Space between content and border (background extends here) | `padding`, `padding-top`, etc. |
| **Border**  | Line around the padding | `border`, `border-width`, `border-right`, etc. |
| **Margin**  | Space outside the border (transparent) | `margin`, `margin-top`, etc. |


> color, background and size manipulation.


These are the main scaling/size units you’ll use:

- **Pixels:** `10px` — Fixed size.
- **`em`** — Relative to the element’s **font size** (e.g. `1.5em` = 1.5× current font size).
- **`rem`** — Relative to the **root** (`<html>`) font size. Predictable; good for spacing and typography.
- **`%`** — Percentage of the **parent** (for width, height, etc.).
- **`vw` / `vh`** — Percentage of current viewport size. 1vw= 1% of viewport width.
- **`fr`** — Fraction of free space in a Grid; 1fr= 1 part of total available space. grid-template-columns: 200px 1fr 2fr means second and third column gets 1/3 and 2/3 space after 200px removed from total grid. 


3. Positioning

`position` controls where the box sits in the flow or relative to the viewport:

| Value      | Behavior  |
| --- | ---|
| `static`   | Default. Normal flow; `top`, `right`, `bottom`, `left` have no effect.    |
| `relative` | Stays in flow; `top`/`right`/`bottom`/`left` offset it from its normal position. |
| `absolute` | Removed from flow; positioned relative to the nearest positioned ancestor (non-`static`). |
| `fixed`    | Removed from flow; positioned relative to the **viewport** (stays on scroll). |
| `sticky`   | Acts like relative until a scroll threshold, then sticks (e.g. sticky header). |

With `absolute` or `fixed`, use `top`, `right`, `bottom`, `left` to place the element.  
**Stacking:** `z-index` (number) controls which positioned element appears on top (only works when `position` is not `static`).


For typography and spacing, `rem` is often preferred so the layout scales with user font settings.

Typography

Text inside the box is styled with **typography** properties:

| Property        | Purpose                          | Example                    |
| --------------- | --------------------------------- | -------------------------- |
| `font-family`   | Font stack (fallbacks)            | `font-family: Arial, sans-serif;` |
| `font-size`     | Text size                         | `font-size: 1rem;`         |
| `font-weight`   | Boldness (100–900 or `normal`, `bold`) | `font-weight: 700;`  |
| `font-style`    | Italic or normal                  | `font-style: italic;`      |
| `line-height`   | Line spacing (unitless = multiple of font size) | `line-height: 1.5;` |
| `text-align`    | Horizontal alignment              | `text-align: center;`      |
| `text-decoration` | Underline, overline, line-through | `text-decoration: none;`   |
| `letter-spacing` | Space between letters             | `letter-spacing: 0.05em;`   |

Shorthand: `font: style weight size/line-height family;` — e.g. `font: italic 700 1rem/1.5 Georgia, serif;`

8. Responsive design and media queries

**responsive** techniques tie them to the viewport and breakpoints:

**Viewport meta tag** (in HTML `<head>`) is required for mobile-friendly scaling:  
`<meta name="viewport" content="width=device-width, initial-scale=1.0">`

**Media queries** apply different CSS based on screen size (or other features):

```css
/* Default (mobile-first) */
.container { width: 100%; }

/* Tablet and up */
@media (min-width: 768px) {
  .container { width: 750px; margin: 0 auto; }
}

/* Desktop */
@media (min-width: 1024px) {
  .container { width: 960px; }
}
```

- **Mobile-first:** Base styles for small screens; use `min-width` to add styles for larger screens.
- Other features: `(max-width: 600px)`, `(orientation: portrait)`, `(prefers-reduced-motion: reduce)` for accessibility.

---

### 2. Layout: display and the two main systems(Flex and Grid)

How the box is laid out is set by the `display` property.  `display` property can have the following values:

| Value | Behavior |
| --- | ---|
| `block`        | New line, full width of container (e.g. `div`, `p`).                     |
| `inline`       | Flows with text; width is content only (e.g. `span`, `a`).               |
| `inline-block`| Like inline but you can set width, height, and vertical margin/padding.   |
| `none`         | Element not shown and does not take space.                                |
| `flex`         | Children become flex items; use Flexbox for 1D layouts (row or column).   |
| `grid`         | Children become grid items; use Grid for 2D layouts.                      |

**Flexbox (one dimension)** — Use `display: flex;` on the parent when you have a single row or column of items.

These properties set on the parent(flex-container) control how the items are arranged and aligned:

- `flex-direction` accepts values like `row,  column, row-reverse, column-reverse` to determine which way the main axis runs. 
- `justify-content` accepts values like `flex-start, center, flex-end, space-between, space-around` to determine how items are aligned along the main axis (e.g. center, or space between).
- `align-items` accepts values like `flex-start,  center, flex-end, stretch` to determine how items are aligned perpendicular to the main axis.
- `gap` — Space between items (e.g. `1rem` = 1× the font size of the root element `<html>`).

These properties are set on the direct children(flex-items):

- `flex: 1` — Shorthand to let the item grow and fill available space (grow/shrink/basis).
- `flex-grow`, `flex-shrink`, `flex-basis`— Fine control over how the item grows or shrinks.


**Grid (two dimensions)** — Use `display: grid;` on the parent when you need rows and columns together.

These properties set on the parent(grid-container) define the grid and control how items are placed:

- **`grid-template-columns`** — Defines columns (e.g. `1fr 1fr 1fr` = three equal columns; `fr` = fraction of available space).
- **`grid-template-rows`** — Defines row sizes (e.g. `auto 100px` = first row by content, second row 100px).
- **`gap`** — Space between rows and columns (e.g. `1rem`).
- **`grid-template-areas`** — Names regions so you can place items by area name using `grid-area` on children.

These properties are set on its direct children(grid-items):

- **`grid-column`** — Which column(s), e.g. `1 / 3` (from line 1 to 3 = spans 2 columns).
- **`grid-row`** — Which row(s), e.g. `1` (first row).
- **`grid-area`** — Place in a named area (must match a name in the parent’s `grid-template-areas`).


---

## 3. CSS custom properties (variables)

To reuse values (colors, spacing) across the stylesheet, use **variables** — define once, use in many places. Defined on a selector (often `:root` for global use):

```css
:root {
  --primary-color: #3498db;
  --spacing: 1rem;
}

.button {
  background: var(--primary-color);
  padding: var(--spacing);
}
```

- Fallback: `var(--primary-color, blue);` — uses `blue` if `--primary-color` is not set.
- You can override variables inside any selector (e.g. a dark theme on `body.dark`).

---
