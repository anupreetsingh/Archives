# Terminal and Shell

## Terminal

A **terminal** is an application or window that lets you enter text for a shell program and view text output from a shell program.

You could have a variety of terminals, Examples include:

- macOS Terminal
- VS Code integrated terminal
- iTerm2
- Windows Terminal
- Linux terminal emulators like GNOME Terminal or Konsole

Different terminal applications can provide different user experiences, including differences in keyboard shortcuts, text rendering, tab and pane management, scrolling, copy/paste behavior, and configuration options.

Terminal Tab: Separate terminal sessions, but you switch between them.
Terminal Pane: Separate terminal sessions shown at the same time in a split view.

For example, macOS Terminal may not treat `Command + Right Arrow` as "move the cursor to the end of the line" or `Command + Left Arrow` as "move the cursor to the beginning of the line". The VS Code integrated terminal can support those shortcuts.

Modern terminal apps are usually **terminal emulators**.

Older computers used physical terminal devices connected to larger computers. A modern terminal app emulates that old text-based interface in a window.

So when people say "open the terminal", they usually mean "Open a terminal emulator app so you can interact with a shell".

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

### Command Manuals

`man <command-name>` gives you a list of commands you can call for a program.

## Shell

A **shell** is a command interpreter.

The terminal calls upon the shell program. The shell interprets commands and asks the operating system to run other programs. It sits on top of the kernel and communicates with it to launch child processes.

![Shell](<Media/Shell.png>)

The name *shell* comes from the program's role as the protective outermost layer of the operating system. Commands can be typed interactively in a terminal window or passed to a shell through a `.sh` shell script.

### Shell Programs

Although the differences between shells are mostly invisible during basic use, each kind of shell is a separate program. Examples include:

- Bash (Bourne Again Shell, the default on many Linux systems)
- Zsh (Z Shell, the default on modern macOS systems)
- Fish
- PowerShell
- POSIX `sh`

You can check your configured default shell with:

```bash
echo $SHELL
# Output: /bin/zsh
```

`/bin/zsh` is the location on disk of the executable file for the `zsh` shell program. Because the path begins with /, it is an **absolute path**, meaning it starts from the root directory / of the filesystem.

**Filesystem** is the OS-managed structure and rules used to organize, store, locate, and manage files and directories on storage devices.

### Interactive Shell

An **interactive shell** is the shell you use directly in a terminal by typing in commands sequentially.

Example:

```bash
cd Projects
ls
git status
```

The shell is what understands commands such as:

```bash
cd Documents
ls
export NAME="Maya"
```

Some commands are built directly into the shell. Others are external programs.

### Built-in Commands

A **built-in command** is handled by the shell itself.

`echo` is a command for the shell and repeats back anything passed in it back on the terminal screen. It is a **shell-builtin** so the shell executes it itself without launching any child processes.

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

The external program is a **child process** launched by the main shell process. `PPID` (Parent Process ID) stores the process ID of the current process's parent.

## Shell Scripts

A **shell script** is a file containing commands written for a shell to execute.

Shells use scripting languages that can be typed interactively into the terminal or saved in a `.sh` file. A **scripting language** is a programming language commonly used to automate a sequence of tasks.

Bash and Zsh provide shell scripting languages. Python is also commonly used as a scripting language, but it is a general-purpose programming language rather than a shell language.

### Running a Shell Script

To pass commands to a shell using a shell script, execute the `.sh` file from the terminal. For example, this command explicitly asks Bash to interpret `script.sh`:

```bash
bash script.sh
```

The `.sh` extension identifies the file as a shell script by convention, but it does not determine which shell interprets it.

Example Bash script:

```bash
#!/usr/bin/env bash

name="Maya"

echo "Starting script..."

if [ "$name" = "Maya" ]; then
  echo "Hello, $name"
fi
```

### Shebang

The first line that is commented out is called a **shebang**.

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

Shell scripting language conventions are mostly the same across related shells, but more advanced syntax and behavior can differ. For example, Bash arrays start at index `0`, while Zsh arrays normally start at index `1`:

```bash
# Bash
names=("Ryan" "Maya")
echo "${names[0]}"

# Zsh
names=("Ryan" "Maya")
echo "${names[1]}"
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

An **environment variable** is a named text value that the shell stores in its process memory and makes available to the programs started from that shell.

Internally the shell maintains a map of the variable names to values and whether a variable is exported or not.

### Shell Variables and Exported Variables

| Variable | Value | Exported |
|----------|-------|----------|
| `USER` | `rbaker` | 1 |
| `SHELL` | `/bin/zsh` | 1 |
| `NAME` | `ryan` | 0 |

An **exported** variable means that variable is exported to child processes. When a shell spawns a child process it passes a *snapshot* of its exported variables at that instance to the child process.

It is common to refer to an exported variable as the environment variable and non-exported variable as a shell variable.

Example:

```bash
export APP_ENV="development"
```

A program started from that shell can read `APP_ENV`.

#### Creating a Shell Variable

We can also create our own variable in the terminal:

```bash
NAME=ryan
# Syntax does not have a spaces because shell uses whitespace to separate commands and arguments.
echo $NAME
# ryan
```

`$` is the expansion operator and prints the value of the environment variable `NAME`.

#### Reading and Exporting Variables

The command `printenv` launches the child process `printenv` to print out the map of the environment variables and their values in case no argument is given. If a variable name is given as argument then it prints its value.

```bash
printenv SHELL
# Output: /bin/zsh

printenv NAME
# Output:
# Doesn't print anything because NAME was not marked as an exported variable, so it's not carried over to the printenv child process
```

The `printenv` is a child process launched by the main shell process when we type in `printenv`.

![PrintEnv](<Media/printenv.png>)

We can promote a shell variable to an environment variable using `export` built-in keyword.

```bash
export NAME
printenv NAME
# Output: ryan
```

### Common Environment Variables

| Variable | Meaning |
|----------|---------|
| `HOME` | Path to the current user's home directory |
| `PATH` | Directories where the shell looks for executable programs |
| `SHELL` | Path to the user's default shell |
| `PWD` | Current working directory |
| `USER` | Current username |

#### PATH

`PATH` is especially important. It tells the shell where to search for commands.

To figure out which programs to launch when you type in a command name, the shell keeps a list of directories to search through and it exposes that list as an environment variable called `PATH` so you're able to modify it.

```text
/usr/bin
/bin
/usr/local/bin
/sbin
```

When you type:

```bash
python
```

the shell searches through the directories listed in `PATH` until it finds an executable named `python`.

It walks the directories in order and stops at the first match.

You can print and modify your `PATH` with:

```bash
echo $PATH
# Output: /usr/bin:/bin:/usr/local/bin:/sbin
PATH=.:$PATH
# . is the current directory
# : is the separator
# $PATH expands the current value
echo $PATH
# Output: .:/usr/bin:/bin:/usr/local/bin:/sbin
# The current directory now sits first so if you have a matching program name in it, it will get called.
```
