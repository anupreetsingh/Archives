HTML is the core of the of the webpage that gives structure to the content on our webpage. The browser builds an in memory representation of all the elements on the HTML page which looks like a tree of objects called the DOM. 

---

## Document structure and semantics

HTML is made up of **elements**. Each element has:

- An **opening tag** — e.g. `<p>`
- **Content** — text and/or other elements (nesting is allowed)
- A **closing tag** — e.g. `</p>`

Some **Void elements** could also just have the opening tag. Common examples: `<br>` (line break), `<hr>` (horizontal rule), `<img src="..." alt="...">`, `<input type="text">`, `<meta charset="UTF-8">`, `<link rel="stylesheet" href="...">`.

Elements have a default **display type**: 

1. **Block** elements start on a new line and take the full width of their container (e.g. `<p>`, `<div>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<h1>`–`<h6>`, `<ul>`, `<ol>`, `<li>`, `<main>`, `<nav>`, `<aside>`, `<figure>`, `<form>`, `<blockquote>`, `<hr>`).
2. **Inline** elements flow with the text and only take as much width as their content (e.g. `<span>`, `<a>`, `<strong>`, `<em>`, `<img>`, `<br>`, `<input>`, `<label>`, `<code>`, `<abbr>`).

This affects how the page is laid out; you can override it with CSS (`display: ...`).

You can also have **attributes** in the opening tag of an element to add extra information (e.g. link URL, image source, CSS class). Format: `name="value"`.

Examples:

- `<a href="https://example.com">Click here</a>` — `href` is the attribute (where the link goes).
- `<img src="photo.jpg" alt="A photo">` — `src` (image file) and `alt` (description for accessibility).
- `<p class="intro">...</p>` — `class` for styling or scripting.

Every valid HTML page has a standard structure:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Page Title</title>
  </head>
  <body>
    <!-- HTML comments: visible only in source; use for notes. -->
  </body>
</html>
```

- `<!DOCTYPE html>` — One-off instruction at the very start of the file that tells the browser this is an HTML5 document.
- `<html>` — Wraps the entire page. Use the `lang` attribute (e.g. `lang="en"`) for accessibility(screen reader) and SEO.
- `<head>` — Holds non-visible page info:
  - `<meta>` — Metadata via attributes such as 
    - `charset` : for encoding.
    - `name` + `content`: where `name` defines the type; `content` holds the value.
  - `<title>` — Shown in the browser tab and used in search results / bookmarks.
  - `<style>` - To include CSS styling directly
  - `<link>` - To link documents(like CSS files) to the HTML file. `rel="stylesheet"` for rel attribute is most commonly used to link .css files
  - `<script>` — Embeds or links JavaScript. Use `src="path/to/file.js"` to link an external script, or put code between the tags. Often placed at the end of `<body>` so the DOM is ready, or in `<head>` with `defer`/`async`. Example: `<script src="app.js" defer></script>` in `<head>` or before `</body>`.
- `<body>` — Contains all visible content: text, images, links, sections, etc. There are two broad categories of body elements:


### 1. Semantic elements

Semantic elements are more often than not containers that group  other elements(blocks or inline) together and use tags that describe the **meaning** of the content to the browser and the reader of the code; and in doing so helps with:

- **Accessibility** — Screen readers and assistive tech can understand the page.
- **SEO(Search Engine Optimization)** — Search engines can better interpret your content.
- **Maintainability** — Other developers (and you later) understand the structure.

Some Example of semantic Elements are:


| Element        | Purpose                                                                                         |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `<header>`     | Introductory content or nav for the page or a section (e.g. logo, main nav).                    |
| `<nav>`        | Navigation links (main menu, table of contents, etc.).                                          |
| `<main>`       | The primary content of the page. Only one per page; skip repeated chrome (header, footer).      |
| `<article>`    | Self-contained piece of content that could stand alone (blog post, news item, product card).    |
| `<section>`    | Thematic part of something larger, usually with its own heading.                                      |
| `<aside>`      | Tangentially related content (sidebar, pull quotes, ads).                                       |
| `<figure>`     | Self-contained content (media, diagram, code block) that can have a caption.|
| `<figcaption>` | Caption for a `<figure>`. Place inside the `<figure>`, after or before the main content.        |
| `<footer>`     | Footer for the page or a section (copyright, links, contact).                                   |


### 2. Content Elements

Besides semantic sectioning elements, you use these common elements to add and structure content:


| Element                          | Purpose                                                                                                                                                                                                                                                                   |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<h1>`–`<h6>`                    | Headings define the outline of the page. One `<h1>` per page; then `<h2>` for major sections, `<h3>` for subsections. Do not skip levels. Screen readers and search engines use this hierarchy.                                                                           |
| `<p>`                            | Paragraph of text. Block-level (stacks vertically). Use for body copy; avoid wrapping interactive elements (links, buttons) that contain block content inside a `<p>`.                                                                                                    |
| `<ul>` / `<ol>`                  | List containers: `<ul>` = unordered list (bullets), `<ol>` = ordered list(numbers). Use for nav menus, steps, any list. Direct children should be `<li>`.                                                                                                                 |
| `<li>`                           | List item. Must be inside `<ul>` or `<ol>`. Can contain text, links, or nested lists (e.g. `<ul>` inside `<li>` for sub-items).                                                                                                                                           |
| `<a href="...">`                 | `a`= **Anchor** is used to add a link. `href` = destination (URL, path, or `#id` for same-page jump). Text between tags is clickable.                                                                                                                                     |
| `<img src="..." alt="...">`      | For embedding Image. Common attributes include `src` = path/URL of the image, `alt` = short description (if image does not load, and for screen readers); required for accessibility. It's a Void element.                                                                |
| `<video src="..." controls>` | For embedding Video. Common attributes include valued attributes like `src` and `poster`(thumbnail) or boolean attributes like `controls`, `autoplay`, `muted`, `loop`. Use void element `<track>` inside it for giving captions or description.                          |
| `<audio src="..." controls>` | For embedding Audio. Common attributes include valued attributes like `src` or boolean attributes like `controls`, `autoplay`, `muted`, `loop`. No visual output except controls. Use void element`<track>` inside for captions or description (e.g. for screen readers). |
| `<div>`                          | Generic block-level container. No meaning; use when no semantic element fits. Handy for layout or as a hook for CSS/JavaScript.                                                                                                                                           |
| `<span>`                         | Generic inline wrapper. No meaning; use to apply `class` or `id` to a bit of text. Does not change layout by itself — add CSS for visual effect.                                                                                                                          |
| `<br>` | Adds Line break inside a block (e.g. in `<p>`). It's a Void element.  |
| `<strong>`                       | Indicates importance for the reader(or screen reader) and is rendered as bold by the browser(warnings, key terms).                                                                                                                                                        |
| `<em>`                           | Indicates Emphasis (stress, tone) for the reader(or screen reader) and rendered as Italic by default. Use when emphasis changes meaning (e.g. "I *did* submit it" vs "I did *submit* it").                                                                                |
| `<table>`  | For tabular data. Structure: optional `<caption>` (table title); `<thead>` (header rows), `<tbody>` (body rows), optional `<tfoot>` (footer rows); inside each, `<tr>` (row) containing `<th>` (header cell) or `<td>` (data cell).                                                                                                               |

**Character entities:** To show special characters(like `<`, `>`, `&`, `"`, etc ) that HTML uses for markup, we use entities where The `&` (ampersand) starts the entity; the **`;`** (semicolon) ends it. Eg: `&lt;` displays as `<`, `&amp;` as `&`, and `&copy;` as ©. Format: `&name;` or `&#code;` (e.g. `&#169;` for ©). 


**Note:** `<span>` and `<div>` do not change how content looks on their own; use them with `class` or `id` and then style or target them with CSS/JavaScript.

---

## Forms and input

Forms collect data from the user and send it to a server (or handle it with JavaScript). The data is sent when the user submits the form (e.g. clicks a submit button).

### The form element

`<form>` wraps all form control elements. Two attributes define where and how data is sent:

- `action` — URL that receives the submitted data (e.g. `/login`, `https://api.example.com/signup`). Can be empty or omitted if you handle submit with JavaScript.
- `method` — How the data is sent:
  - `get` — Data appended to the URL as a query string (`?name=value&...`). Good for searches, but visible in the address bar so security concern.
  - `post` — Data sent in the request body. Good for login, signup, or any submission that should not appear in the URL.

Only control elements with a `name` attribute contribute to the submitted data.

### Common elements inside a form


| Element  | Purpose and attributes  |
| ---| --- |
| `<label>`  | Displays text that describes a control. Some common attributes: `for` — set to the control’s `id` so clicking the label focuses the control. |
| `<input>`  | Takes in single-line or special input. It's a void element. Some common attributes: <br>`name` — It is the key sent with the value of `<input>` on submit. It's required for value to be sent. `type="radio"` share one name per group. <br>`id` — Used in `<label for="id">`. <br>`type` can be  `"text"`, `"email"`, `"password"`, `"number"`, `"tel"`, `"url"`, `"date"`, `"checkbox"`, `"radio"`, `"file"`, `"hidden"`, `"submit"`, `"reset"`, `"button"`, `"search"`, etc. <br>`value` — default text show/submitted.<br>  `required` — a boolean that blocks submit until filled. <br>`placeholder` — A hint when empty. |
| `<textarea>` | Use for adding multi-line text; default value is the content between the tags. Some common attributes: <br>`name` — key sent with value of `textarea`on submit. <br>`id` — Used in `<label for="id">`. <br>`rows`, `cols` — size (or use CSS). <br>`required` — a boolean that blocks submit until filled. <br>`placeholder` — A hint when empty.|
| `<select>`   | Encapsulates `<option>` elements to create a dropdown; the selected option's `value` is submitted. Some common attributes: <br>`name` — key sent with value on submit. <br>`id` — Used in `<label for="id">`. <br>`required` — blocks submit until an option is chosen. |
| `<option>`   | One choice inside `<select>`. The text between the tags is the visible label in the dropdown. Some common attributes: <br>`value` — what gets submitted when this option is selected. |
| `<button>`   | A clickable button. Label is the content between the tags. Some common attributes: <br>`type` — `"submit"` (sends form), `"button"` (custom JS), or `"reset"` (clears form). <br>`name` — optional on submit buttons.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |


