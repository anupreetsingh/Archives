# Terminal and Shell

A terminal and a shell are related, but they are not the same thing.

The **terminal** is the text-based interface where you type commands and see output. The **shell** is the program that reads those commands, interprets them, and starts other programs.

Simple relationship:

```md
Terminal -> Shell -> Commands / CLI Programs -> Operating System
```

Example:

```bash
git status
```

When you run this command:

1. You type `git status` into the **terminal**.
2. The **shell** reads the command.
3. The shell finds the `git` program.
4. The shell runs `git` with `status` as an argument.
5. Git prints output back to the terminal.

---

## Terminal

A **terminal** is the application or window that gives you a text interface to the computer.

Examples:

- macOS Terminal
- VS Code integrated terminal
- iTerm2
- Windows Terminal
- Linux terminal emulators like GNOME Terminal or Konsole

The terminal handles input and output. It lets you type text, sends that text to the shell, and displays whatever the shell or program prints back.

The terminal itself usually does not understand commands like `cd`, `ls`, `git`, or `python`. Those are interpreted by the shell or executed as separate programs.

### Terminal Emulator

Modern terminal apps are usually **terminal emulators**.

Older computers used physical terminal devices connected to larger computers. A modern terminal app emulates that old text-based interface in a window.

So when people say "open the terminal", they usually mean:

```md
Open a terminal emulator app so you can interact with a shell.
```

### Terminal Behavior vs Shell Behavior

Different terminal emulators can handle the same keyboard input differently, even when they are running the same shell.

For example, macOS Terminal may not treat `Command + Right Arrow` as "move the cursor to the end of the line" or `Command + Left Arrow` as "move the cursor to the beginning of the line". The VS Code integrated terminal can support those shortcuts.

That difference is caused by the **terminal emulator**, not by the shell. The terminal decides how keyboard shortcuts are handled and what input gets sent to the shell.

So if two terminals both run `zsh`, but one supports a shortcut and the other does not, the difference is usually in the terminal app's key handling, not in `zsh` itself.

---

## Shell

A **shell** is a command interpreter. It interprets commands, and asks the operating system to run programs. Commands can be typed interactively in a terminal window or passed to a shell through a `.sh` shell script.

Examples:

- Bash
- Zsh
- Fish
- PowerShell
- POSIX `sh`

The shell is what understands commands such as:

```bash
cd Documents
ls
export NAME="Maya"
```

Some commands are built directly into the shell. Others are external programs.

### Built-in Commands

A **built-in command** is handled by the shell itself.

Example:

```bash
cd Documents
```

`cd` changes the current working directory of the shell process. This needs to be built into the shell because an external program cannot permanently change the parent shell's current directory.

Common shell built-ins include:

| Built-in | Purpose |
|----------|---------|
| `cd` | Change directory |
| `echo` | Print text |
| `export` | Set an environment variable |
| `alias` | Create a command shortcut |
| `pwd` | Print working directory |

### External Programs

An **external program** is a separate executable that the shell starts.

Examples:

```bash
git status
python script.py
npm install
docker compose up
```

Here, `git`, `python`, `npm`, and `docker` are separate command-line programs. The shell finds them, starts them, passes arguments to them, and displays their output.

---

## Bash and Zsh

**Bash** and **Zsh** are both shells.

```md
Bash = one specific shell
Zsh = another specific shell
Shell = the general category
```

Bash stands for **Bourne Again Shell**. It is very common in Linux environments and shell scripting.

Zsh stands for **Z Shell**. It is the default interactive shell on modern macOS systems and is popular because of features like improved autocomplete, themes, and plugins.

### Interactive Shell

An **interactive shell** is the shell you use directly in a terminal.

Example:

```bash
cd Projects
ls
git status
```

On macOS, the interactive shell is often `zsh`. On many Linux systems, it is often `bash`.

You can check your current shell with:

```bash
echo $SHELL
```

### Shell Language

Shells also have programming language features. They support variables, conditionals, loops, functions, and scripts.

Example:

```bash
name="Maya"

if [ "$name" = "Maya" ]; then
  echo "Hello, $name"
fi
```

This means Bash is both:

- A shell for running commands interactively
- A scripting language for writing shell scripts

---

## CLI

**CLI** stands for **Command-Line Interface**.

A CLI is a way of interacting with software by typing commands instead of clicking buttons in a graphical interface.

Examples of CLI tools:

| CLI tool | Example command |
|----------|-----------------|
| Git | `git status` |
| Python | `python script.py` |
| Node/npm | `npm install` |
| Docker | `docker compose up` |
| SQLite | `sqlite3 database.db` |

The terminal is where you type the command. The shell interprets the command. The CLI program is often the actual tool being run.

Example:

```bash
git commit -m "Add notes"
```

Breakdown:

| Part | Meaning |
|------|---------|
| `git` | CLI program |
| `commit` | Git subcommand |
| `-m` | Option or flag |
| `"Add notes"` | Argument passed to the command |

---

## Shell Scripts

A **shell script** is a file containing commands written for a shell to execute.

Example Bash script:

```bash
#!/usr/bin/env bash

name="Maya"

echo "Starting script..."

if [ "$name" = "Maya" ]; then
  echo "Hello, $name"
fi
```

The first line is called a **shebang**.

```bash
#!/usr/bin/env bash
```

The shebang tells the operating system which program should run the script. In this case, the script should be run with Bash.

Other examples:

```bash
#!/bin/bash
```

```bash
#!/bin/zsh
```

```bash
#!/bin/sh
```

### Does the Shell Language Matter?

Yes. Shell scripts are written in a specific shell language.

Simple commands may work in many shells:

```sh
echo "hello"
ls
cd Documents
```

But more advanced syntax can differ between shells.

For example, Bash arrays use Bash-specific syntax:

```bash
names=("Maya" "Alex" "Sam")
echo "${names[0]}"
```

If a script uses Bash-specific syntax, it should use a Bash shebang:

```bash
#!/usr/bin/env bash
```

If a script is meant to be very portable across Unix-like systems, it is often written using POSIX `sh` syntax:

```bash
#!/bin/sh
```

Practical rule:

```md
Use Bash for common scripting features and readability.
Use POSIX sh when portability matters more than convenience.
Use Zsh mainly for interactive terminal customization, unless the script specifically needs Zsh features.
```

---

## Environment Variables

An **environment variable** is a named value available to a shell and the programs started from that shell.

Example:

```bash
export APP_ENV="development"
```

A program started from that shell can read `APP_ENV`.

Common environment variables:

| Variable | Meaning |
|----------|---------|
| `HOME` | Path to the current user's home directory |
| `PATH` | Directories where the shell looks for executable programs |
| `SHELL` | Path to the user's default shell |
| `PWD` | Current working directory |
| `USER` | Current username |

### PATH

`PATH` is especially important. It tells the shell where to search for commands.

When you type:

```bash
python
```

the shell searches through the directories listed in `PATH` until it finds an executable named `python`.

You can print your `PATH` with:

```bash
echo $PATH
```

---

## Summary

| Concept | Meaning |
|---------|---------|
| **Terminal** | The text window/app where you type commands and see output |
| **Terminal emulator** | A modern app that emulates old physical terminals |
| **Shell** | The command interpreter running inside the terminal |
| **Bash** | A specific shell and shell scripting language |
| **Zsh** | Another shell, commonly used interactively on macOS |
| **CLI** | A command-line interface, or a tool used through typed commands |
| **Shell script** | A file of commands written for a shell to execute |

The clean mental model:

```md
The terminal is where you type.
The shell is what interprets what you typed.
Bash and Zsh are specific shells.
CLI tools are programs you run through typed commands.
Shell scripts are files written in a shell language.
```
