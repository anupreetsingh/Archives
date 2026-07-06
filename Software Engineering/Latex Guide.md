# LaTeX Guide

## High Level Terminology

### Markup Language

A **markup language** annotates("marks up") content with meaning, structure, or metadata — for example, marking parts of the content as headings, paragraphs, lists, links, tables, or emphasized text. But it stops there: the final appearance usually depends on another system, such as a CSS configuration or the renderer (a browser, the VS Code preview, GitHub, or a PDF renderer).

Example: HTML, Markdown, XML, and JSX.

### Typesetting System

A **typesetting system** goes one level further. It has a source file covering the annotation of the content along with details about its final appearance and formatting such as fonts, spacing, margins, page numbering, references, citations, and mathematical notation.

The most famous markup based typesetting system is **Tex** and **LaTex** is a package on top of it that provides higher-level macros such as `\section`, `\item`, `\textbf`, and `\begin{equation}`, that make describing a document much easier than writing raw TeX.

The **source file** in this case is `.tex` and a **compiler engine**, such as `pdflatex` or `lualatex` processes it and converts it into the finished document, usually a PDF.

Tools like **MS Word**, **Apple Pages** and **Adobe In Design** are WYSIWYG(What You See Is What You Get) word processors that give you high-level controls such as font menus, heading styles, page breaks, and tables which are essentially abstractions for their own typesetting systems.

### Latex Installs

"Installing LaTeX" means installing a Latex **distribution**: a bundle that puts several different things on your machine at once:

Common distributions include **TeX Live**(cross-platform), **MacTeX**(macOS TeX Live bundle), **BasicTeX**(small macOS install), **MiKTeX**(common on Windows), and **TinyTeX**(small TeX Live build). Full distributions install most packages up front; minimal distributions save disk space but may require installing missing packages later.

1. **LaTeX format**: This is the high-level macros (`\section`, `\item`, `\textbf`, `\begin{equation}`) and the conventions for how a `.tex` file is structured. Essentially this is "LaTeX the language" — the vocabulary you actually write your document in.

2. **Compiler engines**: These are the programs that actually run your source `.tex` file and produces the PDF. Eg: `pdflatex`, `xelatex`, and `lualatex`. They differ mainly in font and Unicode support: `pdflatex` is the traditional default, while `xelatex` and `lualatex` work better with modern system fonts and Unicode.

3. **Auxiliary programs**: These are separate helper tools that the engine calls for specialized jobs during the build, such as `bibtex` or `biber` for bibliographies, `makeindex` for indexes, and `latexmk` for build automation. A full build may need multiple passes, so `latexmk` is useful because it runs the needed tools in the right order.

4. **package library**: Hundreds of optional `.sty` packages, like `amsmath`, `graphicx`, and `geometry`, that you import into your source document with `\usepackage`. In TeX Live based installs, missing packages are usually installed with `tlmgr`, the TeX Live package manager.

## Commands and Environments

Almost everything in LaTeX is either a **command** or an **environment**.

### Commands

A command starts with a backslash and optionally takes arguments. Arguments in curly braces `{}` are required; arguments in square brackets `[]` are optional.

```latex
\textbf{bold text}              % required argument
\section{Introduction}          % required argument
\documentclass[12pt]{article}   % optional [12pt], required {article}
```

The general shape is:

```latex
\commandname[optional]{required}
```

Common inline text commands:

| Command | Effect |
| --- | --- |
| `\textbf{...}` | **Bold** |
| `\textit{...}` | *Italic* |
| `\underline{...}` | Underlined |
| `\texttt{...}` | Monospace (typewriter) |
| `\emph{...}` | Emphasis (usually italic) |
| `\textsuperscript{...}` | Superscript |
| `\footnote{...}` | Adds a footnote |

#### No Arguments

A command only takes an argument when it needs extra information. For example, `\section{Introduction}` needs to know the section title, and `\textbf{bold text}` needs to know which text should be bold.

**Meaning by itself** Some commands do not take arguments because the command already has a complete meaning by itself:

```latex
\today          % prints the current date
\LaTeX          % prints the LaTeX logo
\newline        % starts a new line
\hrule          % Prints a horizontal line
```

**Declarations**: they change the current formatting or state from that point onward until the current group or environment ends. For example, `\bfseries` switches to bold text, but it does not take `{...}` as an argument:

```latex
{\bfseries This text is bold.}
{\itshape This text is italic.}
{\centering This paragraph is centered.\par}
```

The outer `{...}` creates a group that limits the scope of the declaration. So `\textbf{bold text}` takes the text as an argument, while `{\bfseries bold text}` uses a declaration whose effect lasts until the group ends.

**Structural Boundary:** Some commands affect the text that comes after them without wrapping that text in `{}`. For example, `\item` starts a new list entry; the content that follows belongs to that item:

```latex
\begin{itemize}
  \item First bullet
  \item Second bullet
\end{itemize}
```

An `\item` continues until LaTeX reaches the next `\item` or the end of the list environment. In the example above, `First bullet` belongs to the first item because the next `\item` starts a new item, and `Second bullet` belongs to the second item until `\end{itemize}` ends the list.

#### Custom Macros

A **macro** is a command that LaTeX expands it into its underlying definition and then processes the result.

Many everyday LaTeX commands are macros. For example, `\section{Introduction}` expands into the lower-level instructions needed to format the heading, number it, add spacing, and register it for the table of contents.

You can define your own macros with `\newcommand`. This is useful when you repeat the same formatting or phrase many times.

For example, suppose you want important vocabulary words to always appear in bold monospace:

```latex
\newcommand{\keyword}[1]{\textbf{\texttt{#1}}}
```

The syntax is:

| Part | Meaning |
| --- | --- |
| `\newcommand` | Defines a new command |
| `{\keyword}` | The name of the command being created |
| `[1]` | The new command takes one argument |
| `{\textbf{\texttt{#1}}}` | The replacement text; `#1` is where the first argument goes |

Now you can write:

```latex
\keyword{API}
```

and LaTeX treats it like:

```latex
\textbf{\texttt{API}}
```

You can also define commands that take no arguments:

```latex
\newcommand{\latexname}{LaTeX}
```

Then:

```latex
\latexname{} is a typesetting system.
```

When executing `\latexname` the `{}` isn't required but it helps separate the command from the following space or text.

### Environments

An environment wraps a block of content and applies environment specific formatting to everything inside it.

```latex
\begin{center}
This text is centered.
\end{center}
```

You wrap content in an environment using the `\begin{env-name}` and `\end{env-name}` commands

Some common LaTeX environments are:

| Environment | Purpose |
|---|---|
| `document` | Contains the visible content of the document |
| `center` | Centers the content inside it |
| `itemize` | Creates a bulleted list |
| `enumerate` | Creates a numbered list |
| `description` | Creates a list with custom labels |
| `tabular` | Creates the table grid|
| `table` | Holds tables with captions |
| `quote` | Formats a short quotation |
| `verbatim` | Displays text exactly as written |
| `figure` | Holds figures/images with captions |
| `equation` | Displays a numbered equation |
| `align` | Aligns multiple equations, usually requires `amsmath` |

## LaTeX document Structure

Every LaTeX document(`.tex` source file) has the same skeleton:

```latex
\documentclass{article}   % 1. document class

\usepackage{amsmath}      % 2. preamble (packages + settings)

\begin{document}          % 3. body starts

Hello, world.

\end{document}            % body ends
```

The three parts are:

| Part | What it does |
| --- | --- |
| `\documentclass{...}` | Chooses the overall document type and default layout |
| Preamble | Everything between `\documentclass` and `\begin{document}`; loads packages and configures settings |
| Body | Everything inside `\begin{document} ... \end{document}`; the actual content |

Anything after `\end{document}` is ignored.

> Indentation is not enforced in Latex and is only used for improving readability.

### Document Classes

The document class sets the foundational layout, default font size, and which structural commands are available. You can optionally pass it **options** in square brackets.

```latex
\documentclass[12pt, a4paper]{article}
```

Common document classes:

| Class | Use case |
| --- | --- |
| `article` | Papers, short reports, essays |
| `report` | Longer documents with chapters |
| `book` | Books; two-sided, chapters, parts |
| `beamer` | Presentation slides |
| `letter` | Formal letters |

Common class options:

| Option | Effect |
| --- | --- |
| `10pt`, `11pt`, `12pt` | Base font size |
| `a4paper`, `letterpaper` | Paper size |
| `twocolumn` | Two-column layout |
| `twoside` | Different margins for odd/even pages |
| `landscape` | Landscape orientation |

The class also determines which sectioning commands exist. `article` has no `\chapter`; `report` and `book` do.

### Packages

The base LaTeX system is deliberately small. **Packages** extend it with extra commands, environments, and features. You load them in the preamble with `\usepackage`.

Think of a package like an import in a programming language: it must be loaded before the body if you want to use what it provides.

```latex
\usepackage{amsmath}                % advanced math
\usepackage[margin=1in]{geometry}   % with options
```

Frequently used packages:

| Package | Provides |
| --- | --- |
| `amsmath`, `amssymb` | Advanced math environments and symbols |
| `graphicx` | Including images with `\includegraphics` |
| `geometry` | Control page margins and size |
| `hyperref` | Clickable links and PDF bookmarks |
| `babel` | Language-specific typesetting rules |
| `xcolor` | Colored text and backgrounds |
| `booktabs` | Better-looking table rules |
| `listings` | Source code listings with syntax highlighting |
| `fontenc`, `inputenc` | Font and input encoding (often needed for accents) |

If you compile and get an "undefined control sequence" error, a missing `\usepackage` is a common cause.

### Sectioning the Document

You usually organize the body into titled, numbered parts using **sectioning commands**. You only declare the *level* of a heading; LaTeX handles the numbering, the font, the spacing, and the table-of-contents entry for you.

```latex
\section{Introduction}
\subsection{Background}
\subsubsection{Prior Work}
```

The hierarchy, from largest to smallest:

| Command | Level | Available in Document class|
| --- | --- | --- |
| `\part{...}` | Highest | book, report |
| `\chapter{...}` | | book, report |
| `\section{...}` | | all |
| `\subsection{...}` | | all |
| `\subsubsection{...}` | | all |
| `\paragraph{...}` | | all |
| `\subparagraph{...}` | Lowest | all |

Adding a `*` to any sectioning command suppresses its number and keeps it out of the table of contents. This is common for headings like acknowledgements that you don't want numbered:

```latex
\section*{Acknowledgements}   % no number, not in the table of contents
```

### Title and Table of Contents

The title block is declared in the preamble and then rendered in the body with `\maketitle`. The table of contents is generated from your sectioning commands with `\tableofcontents`.

```latex
\title{My Report}
\author{Anupreet Singh}
\date{\today}             % \today inserts the date of compilation

\begin{document}
\maketitle                % renders the title block
\tableofcontents          % renders the table of contents
...
```

`\tableofcontents` has to know each section's page number, and it gets those from the `.aux` file, a helper file LaTeX writes during a compile to remember numbers for the *next* compile. On a fresh document that file doesn't have the numbers yet, so a table of contents often needs **two compiles** to appear correctly. The same `.aux` mechanism drives [Cross-References](#cross-references-and-labels).

### Paragraphs and Spacing

- A **blank line** starts a new paragraph.
- `\\` adds a line break without starting a new paragraph

When you genuinely need to control spacing yourself, these commands help:

| Command | Effect |
| --- | --- |
| `\newpage` | Start a new page |
| `\vspace{1cm}` | Add vertical space |
| `\hspace{1cm}` | Add horizontal space |
| `\noindent` | Suppress the indent on the next paragraph |
| `~` | Non-breaking space, keeping two words on the same line. |

> LaTeX's default spacing is usually correct, and manual spacing tends to fight the layout so reach for these sparingly.

### Special Characters

Some characters have special meaning in LaTeX. To make them appear as normal text, you usually have to escape them:

| Symbol | Meaning in LaTeX | How to render it literally |
| --- | --- | --- |
| `%` | Starts a comment. Everything after `%` on the same line is ignored by LaTeX. | `\%` |
| `\` | Starts a command, such as `\textbf{...}` or `\section{...}`. | `\textbackslash{}` |
| `{ }` | Groups content or arguments passed to commands. For example, `\textbf{bold text}`. | `\{` and `\}` |
| `$` | Starts and ends inline math mode. For example, `$x + y = 5$`. | `\$` |
| `&` | Marks alignment points in tables and aligned equations. | `\&` |
| `_` | Creates a subscript in math mode. For example, `$x_1$`. | `\_` |
| `^` | Creates a superscript in math mode. For example, `$x^2$`. | `\textasciicircum{}` |
| `#` | Refers to parameters when defining custom commands. | `\#` |
| `~` | Creates a non-breaking space, keeping two words together on the same line. | `\textasciitilde{}` |

```latex
This is visible. % this is a comment and won't appear
```

### Lists

Lists are **environments**. The three main types are bulleted, numbered, and labeled, and each entry inside them begins with `\item`.

```latex
\begin{itemize}        % bulleted (unordered)
  \item First point
  \item Second point
\end{itemize}

\begin{enumerate}      % numbered (ordered)
  \item First step
  \item Second step
\end{enumerate}

\begin{description}    % labeled
  \item[Term] Definition of the term.
  \item[API] Application Programming Interface.
\end{description}
```

Lists can be **nested** by placing one list environment inside another. LaTeX automatically changes the bullet or numbering style at each deeper level:

```latex
\begin{itemize}
  \item Outer item
  \begin{itemize}
    \item Inner item
  \end{itemize}
\end{itemize}
```

## Floats: Tables and Figures

Tables and figures are usually placed as **floats**. A float is a block of content, such as a table or an image, that LaTeX positions for you instead of locking it exactly where you typed it, so that pages stay balanced and a figure never gets split across a page break. Every float can carry an automatic number and a caption.

The pattern is the same for both: you **build the content**, then **wrap it in a float environment**(`table` or `figure`) to give it a caption, a number, and automatic placement.

### Tables

The content of a table is built with the `tabular` environment.

Example:

```latex
\begin{tabular}{|l|c|c|r|p{4cm}|}
  \hline
  Item & Type & Color & Price & Note \\
  \hline
  Apple & Fruit & Red & 1.00 & Good for snacks. \\
  Chair & Furniture & Brown & 25.00 & Used for sitting. \\
  \hline
\end{tabular}
```

The column layout is declared in `{|l|c|c|r|p{4cm}|}`:

| Specifier | Column behavior |
| --- | --- |
| `l` | Left-aligned |
| `c` | Centered |
| `r` | Right-aligned |
| `p{width}` | Fixed-width paragraph cell that wraps text |
| `|` | Draws a vertical line between columns |

In this example, the layout creates five columns:

1. `l` makes the `Item` column left-aligned.
2. `c` makes the `Type` column centered.
3. `c` makes the `Color` column centered.
4. `r` makes the `Price` column right-aligned.
5. `p{4cm}` makes the `Note` column `4cm` wide and wraps longer text inside it.

The `|` characters draw vertical borders. Inside the table body, `&` separates cells within a row, `\\` ends a row, and `\hline` draws a horizontal line across the table.

**`tabular*`** is a variant of `tabular` that takes two arguments after the environment name: the total table width and the column layout.

```latex
\begin{tabular*}{0.8\textwidth}{|l|c|r|} %The table will be 0.8 times the textwidth
```

The `booktabs` package adds `\toprule`, `\midrule`, and `\bottomrule` for cleaner horizontal rules and is widely preferred over simply using `\hline`.

A bare `tabular` only builds the grid; it is not a float and has no caption or number. To get those, wrap it in the `table` float environment:

```latex
\begin{table}[h]
  \centering
  \begin{tabular}{l r}
    Item  & Price \\
    \hline
    Pen   & 2     \\
  \end{tabular}
  \caption{Price list}
  \label{tab:prices}
\end{table}
```

Here `\centering` centers the table, `\caption{...}` adds a numbered caption, and `\label{...}` names it so you can reference it later(see [Cross-References](#cross-references-and-labels)).

### Figures and Images

To include an image you load the `graphicx` package in the preamble, then insert the image with `\includegraphics` inside a `figure` float.

```latex
\usepackage{graphicx}   % in the preamble

\begin{figure}[h]
  \centering
  \includegraphics[width=0.5\textwidth]{diagram.png}
  \caption{System architecture}
  \label{fig:arch}
\end{figure}
```

- `\includegraphics[width=0.5\textwidth]{diagram.png}` inserts the image at half the text width. Sizing relative to `\textwidth` keeps it responsive to the margins instead of using a fixed size.
- `\caption{...}` adds a numbered caption.
- `\label{...}` names the figure so it can be referenced elsewhere.

### Float Placement

Because a float is positioned by LaTeX rather than fixed in place, the optional argument in `\begin{figure}[h]` or `\begin{table}[h]` is only a *hint* about where you'd prefer it to land:

| Specifier | Meaning |
| --- | --- |
| `h` | Here, approximately where it appears in the source |
| `t` | Top of a page |
| `b` | Bottom of a page |
| `p` | On a dedicated page of floats |
| `!` | Override LaTeX's internal placement restrictions |

You can combine them: `[ht]` means "here if possible, otherwise the top of a page." This is why a figure sometimes appears a little before or after where you wrote it.

## Math

Math is one of LaTeX's main strengths. Mathematical notation is written in **math mode**, which comes in two forms: **inline** math flows within a line of text, while **display** math is set apart on its own centered line.

### Inline vs Display

```latex
The equation $E = mc^2$ appears inline.

%The same equation, displayed:
\[
E = mc^2
\]

%The same equation, displayed and numbered:
\begin{equation}
E = mc^2
\end{equation}
```

| Mode | Delimiters | Result |
| --- | --- | --- |
| Inline | `$ ... $` | Math sits inside the text line |
| Display, unnumbered | `\[ ... \]` | Centered on its own line |
| Display, numbered | `\begin{equation} ... \end{equation}` | Centered, with an equation number |

### Common Math Syntax

Inside math mode, ordinary characters become math symbols, and backslash commands produce operators and structures:

| You want | You type |
| --- | --- |
| Superscript | `x^2` |
| Subscript | `x_i` |
| Fraction | `\frac{a}{b}` |
| Square root | `\sqrt{x}`, `\sqrt[3]{x}` |
| Summation | `\sum_{i=1}^{n}` |
| Integral | `\int_{0}^{1}` |
| Greek letters | `\alpha`, `\beta`, `\pi`, `\Sigma` |
| Infinity | `\infty` |
| Multiplication dot / times | `\cdot`, `\times` |
| Comparison | `\leq`, `\geq`, `\neq`, `\approx` |

Imp Note: Group anything longer than one character in braces: write `x^{10}`, not `x^10`, which would only raise the `1` and leave the `0` on the baseline.

### Aligned Equations

For multi-line equations that line up at a chosen point(usually the `=`), use the `align` environment from the `amsmath` package.

As in a table, `&` marks the alignment column and `\\` ends each line.

```latex
\begin{align}
y &= (x + 1)^2 \\
  &= x^2 + 2x + 1
\end{align}
```

Every line gets its own equation number; use `align*` to suppress the numbering.

## Cross-References and Labels

Once a document has numbered things(sections, tables, figures, equations), you can refer to them without hardcoding "Figure 3" or "Section 2.1", which break the moment you reorder content. You attach a `\label` to the item and refer to it with `\ref`; LaTeX fills in the correct number automatically.

```latex
\section{Methods}
\label{sec:methods}

As shown in Section~\ref{sec:methods}, ...
As illustrated in Figure~\ref{fig:arch} on page~\pageref{fig:arch}.
```

| Command | Inserts |
| --- | --- |
| `\label{key}` | Names a section/figure/table/equation so it can be referenced |
| `\ref{key}` | The number of the labeled item |
| `\pageref{key}` | The page number of the labeled item |
| `\eqref{key}` | The equation number in parentheses (from `amsmath`) |

A common convention is to prefix each label with its type(`sec:`, `fig:`, `tab:`, `eq:`) so a key like `fig:arch` is self-describing. The `~` before `\ref` is a non-breaking space, keeping the word and its number together on the same line.

## Bibliography and Citations

For references, you keep your sources in a separate `.bib` file, cite them by key in the text, and let LaTeX format and build the bibliography.

The general shape of an entry in the `.bib` file is `@type{citation-key, fields}`.

Example:

```bibtex
@article{einstein1905,
  author  = {Albert Einstein},
  title   = {On the Electrodynamics of Moving Bodies},
  journal = {Annalen der Physik},
  year    = {1905},
}
```

Here,

- `@article` says the source is a journal article. Common types include `@article`, `@book`, `@inproceedings`, `@misc`, and `@online`

- `einstein1905` is the key used when citing it, and the lines below it are fields describing the source. .

In the document, you load `biblatex` package:

```latex
\usepackage[backend=biber]{biblatex}
\addbibresource{references.bib}

\begin{document}
Time dilation was introduced in \cite{einstein1905}.

\printbibliography
\end{document}
```

`\addbibresource{references.bib}` register the `.bib` file
`\cite{einstein1905}` inserts the in-text citation marker, and `\printbibliography` renders the full reference list.

## Reading Errors

When a compile fails, LaTeX reports the error and a line number, both in the terminal and in the `.log` file. A few common ones and their usual causes:

| Error message | Likely cause |
| --- | --- |
| `Undefined control sequence` | A misspelled command, or a missing `\usepackage` for it |
| `Missing $ inserted` | A math symbol used outside math mode |
| `Runaway argument` | A missing closing brace `}` or `\end{...}` |
| `File not found` | A missing image or `.bib` file |
| `Something's wrong--perhaps a missing \item` | Content placed directly inside a list without `\item` |

The reported line number points *near* the problem, but the real cause is sometimes a few lines earlier, for example an unclosed brace that LaTeX only notices later. When you're stuck, comment out sections with `%` to isolate which part breaks the build.
