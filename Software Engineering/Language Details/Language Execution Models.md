# Language Execution Models

Language execution models describe how source code becomes a running program.

## Compiled vs Interpreted Languages

These execution strategies describe how a language implementation runs code. A single runtime can combine several of them.

### Compiled Languages

Source code is fully translated into machine code *before* execution by a compiler. The resulting binary runs directly on the CPU without needing the compiler again at runtime.

**Examples:** C, C++, Go, Rust, Swift

**Flow:**

```text
source code (.c)
     |
compiler -> machine code (.exe / .out)
     |
CPU executes directly
```

| Pros | Cons |
|------|------|
| Very fast execution because the program is already machine code | Must recompile after each code change |
| Can be optimized heavily by the compiler | Harder to debug during execution |
| Fewer runtime errors if type-checked at compile time | Platform-dependent binaries, such as Windows vs Linux builds |

### Interpreted Languages

An interpreter executes a program's instructions during execution. Depending on the implementation, it may work with a parsed representation of the source or with bytecode produced first. The developer often runs the source file without a separate build command.

**Examples:** CPython, Ruby's standard implementation, PHP; JavaScript engines commonly combine interpretation with JIT compilation.

**Flow:**

```text
source code (.py)
     |
compile to bytecode (CPython)
     |
interpreter executes bytecode instructions
```

| Pros | Cons |
|------|------|
| Easy to test and debug quickly | Interpreting instructions can add overhead compared with native machine code |
| Portable when compatible interpreters and required APIs are available | Some errors are discovered only when the relevant code path executes |
| Good for scripting, prototyping, and education | |

### Bytecode and Virtual Machines

Some languages use an intermediate representation called **bytecode**.

Bytecode is lower-level than source code but not the same as native CPU machine code. A virtual machine/runtime executes that bytecode.

### Python's Hybrid Nature

Technically, Python is both *compiled and interpreted*:

1. The `.py` source is first compiled into **bytecode** (`.pyc`).
2. That bytecode is then interpreted by the **Python Virtual Machine (PVM)**.

So Python is a *hybrid* language: compiled to bytecode, then interpreted.

You can see bytecode in `__pycache__`:

```python
>>> import example
# creates: __pycache__/example.cpython-311.pyc
```

Example:

```python
def greet():
    print("hello")

greet()

# When you run this:
# 1. Python compiles greet() into bytecode instructions.
# 2. The interpreter executes those instructions one by one.

import dis
print("\nDisassembled bytecode for greet():")
dis.dis(greet)
# Disassembled bytecode for greet():
#   1           0 RESUME                   0

#   2           2 LOAD_GLOBAL              1 (NULL + print)
#              12 LOAD_CONST               1 ('hello')
#              14 CALL                     1
#              22 POP_TOP
#              24 RETURN_CONST             0 (None)
```

### Just-In-Time Compilation

Some environments use **JIT (Just-In-Time) compilation**. They compile frequently used parts of code *at runtime* into native machine code for better performance.

| Environment | JIT |
|-------------|-----|
| Java | HotSpot JIT in JVM |
| C# | .NET CLR |
| JavaScript | V8 engine in Chrome/Node |
| Python | PyPy |

### Summary Table

| Type | How it runs | Examples | Speed | Flexibility |
|------|-------------|----------|-------|-------------|
| **Compiled** | Translated to machine code before running | C, C++, Rust, Go | Fast | Less |
| **Interpreted** | Executed by an interpreter at runtime | Python, JavaScript, Ruby | Slower | High |
| **Hybrid** | Compiled to bytecode, then executed by a VM/runtime | Java, Python, C# | Medium | High |
| **JIT** | Compiled dynamically at runtime | Java, JavaScript, PyPy | Fast after warmup | High |

## Runtimes

A **runtime**, or **runtime environment**, is the software that executes a program and supplies the services it needs while running. Depending on the language and environment, these services can include memory management, loading modules, handling errors, scheduling asynchronous work, and accessing files or the network.

The word also has a time-related meaning: an error that happens **at runtime** happens while the program is executing. In contrast, **a runtime** names the software supporting that execution.

A runtime can be installed separately, bundled with an application, or included as support code in a compiled executable. Compiling a program to machine code does not necessarily remove its need for runtime services.

### JavaScript Runtimes

JavaScript makes the role of a runtime easy to see because the same language runs in several environments:

- **Language:** JavaScript defines syntax and behavior, such as functions, objects, arrays, and promises.
- **Engine:** software such as **V8** implements and executes that language, including managing JavaScript objects in memory. Engines can use interpretation and compilation together, as described above.
- **Runtime environment:** surrounds the engine with capabilities for the setting in which the program runs. A browser supplies page-related APIs; Node.js supplies APIs for tasks such as working with files and running servers.

For example, `Array.prototype.map()` belongs to JavaScript, while `document` and `setTimeout()` are supplied by the host environment. Some host APIs appear in multiple environments, but a shared language does not guarantee identical APIs.

Consider a small notes application. Its formatting logic can be shared across the following environments:

```js
// note.mjs — shared JavaScript module
export function formatNote(title, body) {
  return `${title.trim()}\n\n${body.trim()}`;
}
```

The engine can execute this function anywhere that supports these language features. Displaying or saving the result requires capabilities from the surrounding environment.

#### Browser Environments

A browser combines a JavaScript engine with facilities such as the **DOM** for interacting with a page, network requests, timers, and browser-managed storage. Chrome uses V8; other browsers can use different engines.

The notes app can display its formatted text using the browser's `document` object:

```html
<!-- index.html — serve alongside note.mjs through a local web server -->
<!doctype html>
<meta charset="utf-8">
<title>Notes</title>
<pre id="note"></pre>

<script type="module">
  import { formatNote } from "./note.mjs";

  const text = formatNote("Runtimes", "The environment supplies the APIs.");
  document.querySelector("#note").textContent = text;
</script>
```

The string manipulation is JavaScript; finding and updating the page element uses the DOM. Browser code operates within the browser's permissions and isolation rules. For example, access to user files goes through browser-provided APIs and user authorization rather than arbitrary filesystem paths.

##### Progressive Web Apps (PWAs)

A **Progressive Web App (PWA)** is a web application designed to offer capabilities associated with installed applications, such as an app icon, a separate window, and support for working offline. **Its execution environment is still provided by a browser**, including when the usual tabs and address bar are hidden. PWA describes how a web app is delivered and behaves; it does not name a new JavaScript engine or runtime.

The browser version of the notes app could become a PWA by adding:

- A **web app manifest**: a JSON file describing the app's name, icons, launch URL, and display preferences, such as opening in a standalone window.
- **HTTPS hosting**, with localhost available for development. Installation behavior and available capabilities depend on the browser and operating system.
- A **service worker** for an offline experience: a script the browser runs separately from the page, in response to events. It can intercept requests and return previously cached HTML, CSS, and JavaScript. It cannot directly access the page's DOM, and the browser can stop it between events.
- **Local data storage**, such as IndexedDB, to keep the user's notes on the device. The application must implement any synchronization with a server when connectivity returns.

For example, after the app has cached its files, the user could reopen it without a connection, view locally stored notes, and continue editing. Caching the interface and storing the user's data are separate parts of making this work.

**Installation alone does not make an app work offline.** Service workers are commonly used for offline support, but they are not a universal requirement for installation. Push notifications and background operations also depend on platform support and permissions; a service worker is not a continuously running server process.

The installed notes app still uses browser APIs. Installation does not give it Node.js APIs or unrestricted access to the operating system. Its backend, if it has one, can independently run on Node.js, Python, Java, or another server environment.

#### Node.js

**Node.js** is a JavaScript runtime that runs outside the browser. It combines V8 with APIs for filesystem access, networking, processes, and other system tasks, plus support for asynchronous I/O. It is used for command-line tools, development tools, and backend servers.

The same notes module can be used in a script that saves a file:

```js
// save-note.mjs — run with: node save-note.mjs
import { writeFile } from "node:fs/promises";
import { formatNote } from "./note.mjs";

const text = formatNote("Runtimes", "The environment supplies the APIs.");
await writeFile("note.txt", text, "utf8");
console.log("Saved note.txt");
```

Here, `node:fs/promises` supplies the filesystem operation, subject to the process's permissions. Node.js does not supply a browser page or its `document` object. Conversely, the browser cannot directly import Node's filesystem module. The formatting module works in both because it only depends on the language itself.

For how asynchronous work progresses while I/O is pending, see [Async and Event-Loop Concurrency](<../Concurrency Models.md#async-and-event-loop-concurrency>).

#### Electron

**Electron** is a desktop application framework that bundles **Chromium and Node.js**, along with APIs for desktop features such as windows, menus, and dialogs. Chromium supplies the browser environment for rendering the interface, while Node.js supports system operations. Users of a packaged Electron app do not need to install Node.js separately.

Its main parts have different responsibilities:

- **Main process:** runs in a Node.js environment, manages the application's lifecycle and windows, and handles operations involving the operating system.
- **Renderer processes:** display the HTML/CSS interface and run its JavaScript using Chromium. Page code normally has browser APIs, without direct access to Node.js.
- **Preload script and messaging:** expose selected operations to the interface through a controlled bridge. Inter-process communication (**IPC**) lets the renderer request work from the main process.

The notes app can reuse its browser interface and formatting module. To save a file, the interface sends the formatted text through the bridge; the main process opens a save dialog and writes to the chosen location using Node.js.

```mermaid
flowchart LR
    UI["Renderer: notes interface"] -->|Save request| Bridge["Preload bridge"]
    Bridge -->|IPC| Main["Main process: Node.js and Electron APIs"]
    Main --> Dialog["Save dialog"]
    Dialog --> File["Write note to selected file"]
```

Electron lets developers package a web interface with deeper desktop integration and a known Chromium version. Bundling that environment increases download size, and running its processes adds memory overhead. Application updates also need to deliver updates to the bundled components.

##### Electron vs PWA

Both can give the notes app an icon and its own window. Their execution environments and distribution differ:

| Aspect | PWA | Electron application |
|---|---|---|
| Runtime delivery | Uses a browser environment already available on the device | Ships Chromium and Node.js with the application |
| Distribution | Website, with installation on supported platforms | Packaged desktop application |
| OS integration | Browser APIs, permissions, and platform support | Node.js and Electron APIs, subject to OS permissions |
| Offline notes | Requires cached app resources and local data storage | Can ship local app resources and store notes in local files |
| Updates | Web deployment, with browser and service-worker caching affecting when changes appear | Application update mechanism distributes new code and bundled components |

A PWA suits an app whose needs can be met by the web platform, especially when access through a URL and use across desktop and mobile matter. Electron suits a desktop app that needs capabilities such as extensive local file operations or native menus. Either approach requires application logic to handle unavailable network services.

### Other Language Runtimes

The same idea appears elsewhere: **CPython** provides Python's interpreter and runtime services; the **JVM** executes Java bytecode and manages facilities such as memory; and **.NET's CLR** provides managed execution for languages such as C#. A compiled Go executable includes runtime support for services such as garbage collection and goroutine scheduling. A runtime therefore does not always mean a separately installed interpreter or virtual machine.
