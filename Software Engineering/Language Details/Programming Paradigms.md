# Programming Paradigms

This category is about **programming paradigms**. A programming paradigm describes the main style a language encourages for organizing data, behavior, and program flow.

The same idea can be written in different paradigms. Most modern languages are **multi-paradigm**, meaning they support more than one style. For example, Python can be written procedurally, with classes and objects, or in a more functional style.

### Procedural Programming

Procedural programming organizes code around **procedures**, also called functions or routines. The program is usually written as a sequence of steps that operate on data.

The main focus is:

- What steps should happen?
- In what order should they happen?
- What data should each function read or modify?

Procedural code often keeps data and behavior separate. The data may be stored in variables, dictionaries, structs, or records, while functions operate on that data.

**Examples:** C, Pascal, Bash, Python, JavaScript

Example Usage

```python
account = {"owner": "Maya", "balance": 100}

def deposit(account, amount):
    account["balance"] += amount

def withdraw(account, amount):
    if amount <= account["balance"]:
        account["balance"] -= amount
    else:
        print("Insufficient funds")

deposit(account, 50)
withdraw(account, 30)

print(account["balance"])  # 120
```

In this example, the account data is stored in a dictionary, and the functions receive that dictionary as an argument. The functions directly modify the account's balance.

Procedural programming is useful when the program is naturally a sequence of operations, such as scripts, small utilities, data processing tasks, and system-level code.

### Object-Oriented Programming

Object-oriented programming organizes code around **objects**. An object combines data and behavior into one unit.

The main focus is:

- What objects exist in the system?
- What data does each object store?
- What behavior does each object provide?

In OOP, related data and functions are bundled together in a **class**. The class defines the structure and behavior, while each object is a specific instance of that class.

**Examples:** Java, C++, C#, Ruby, Python, JavaScript

Example Usage

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if amount <= self.balance:
            self.balance -= amount
        else:
            print("Insufficient funds")

account = BankAccount("Maya", 100)

account.deposit(50)
account.withdraw(30)

print(account.balance)  # 120
```

Here, the account's data and behavior are grouped inside the `BankAccount` class. Instead of passing the account into separate functions, the account object knows how to deposit and withdraw from itself.

OOP is useful when a program has many related entities with their own state and behavior, such as users, accounts, orders, files, windows, game characters, or network connections.

### Functional Programming

Functional programming organizes code around **functions** and values. It emphasizes computing new values instead of changing existing state.

The main focus is:

- What value should this function return?
- Can this function avoid changing outside state?
- Can larger behavior be built by combining smaller functions?

Functional code often prefers **pure functions**. A pure function returns the same output for the same input and does not modify outside state.

**Examples:** Haskell, Elixir, Erlang, F#, Lisp, Clojure, Scala, Python, JavaScript

Example Usage

```python
def deposit(balance, amount):
    return balance + amount

def withdraw(balance, amount):
    if amount <= balance:
        return balance - amount
    return balance

balance = 100
balance = deposit(balance, 50)
balance = withdraw(balance, 30)

print(balance)  # 120
```

In this version, the functions do not modify an account object or dictionary. They receive values and return new values. The caller decides what to do with the returned result.

A more structured functional version might return a new account dictionary instead of mutating the original one:

```python
def deposit(account, amount):
    return {
        "owner": account["owner"],
        "balance": account["balance"] + amount,
    }

account = {"owner": "Maya", "balance": 100}
updated_account = deposit(account, 50)

print(account["balance"])          # 100
print(updated_account["balance"])  # 150
```

Functional programming is useful when predictable data transformation matters, such as mathematical computation, data pipelines, concurrency, testing, and programs where avoiding shared mutable state reduces bugs.

Important functional ideas include:

| Idea | Meaning |
|------|---------|
| **Pure function** | A function with no side effects and the same output for the same input |
| **Immutability** | Avoiding changes to existing values after they are created |
| **First-class functions** | Treating functions like normal values that can be passed around |
| **Higher-order function** | A function that takes another function as input or returns a function |
| **Composition** | Building larger behavior by combining smaller functions |

### Comparison

| Paradigm | Main organizing idea | Data and behavior | Common strength |
|----------|----------------------|-------------------|-----------------|
| **Procedural** | Step-by-step procedures | Data and functions are usually separate | Simple control flow and direct scripts |
| **Object-Oriented** | Objects with state and behavior | Data and behavior are bundled together | Modeling systems with related entities |
| **Functional** | Functions transforming values | Behavior is functions; data is often treated as immutable | Predictable transformations and fewer state bugs |

The paradigms are not mutually exclusive. A Python program might use procedural code for a simple script, OOP for modeling application entities, and functional techniques for transforming lists or processing data.
