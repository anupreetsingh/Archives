# Dictionary Basics in Python

A dictionary is a key-value data structure.

## Example

```python
person = {
    "name": "Alice",
    "age": 30,
    "city": "New York"
}
```

## Accessing values using keys

```python
print(person["name"])  # Output: Alice
```

If the key does not exist, `dictionary[key]` raises a `KeyError`.

Safer syntax:

```python
dictionary.get(key, default_value)
```

`.get()` returns the value if the key exists. If the key is missing, it returns `None` or the provided default value.

```python
print(person.get("job"))             # Output: None
print(person.get("job", "Unknown"))  # Output: Unknown
```

## Adding or updating key-value pairs

```python
person["age"] = 31        # Updates 'age' since it was already there
person["job"] = "Engineer"  # Adds new key 'job' and assigns value "Engineer"
```

## Removing a key-value pair

```python
del person["city"]  # Removes the 'city' key
```

## Insertion order

Modern dictionaries of the built-in `dict` type preserve insertion order like a list.

This means keys stay in the order they were added. If a key is deleted, the remaining keys keep their order. If new keys are added later, they are added to the end.

```python
pages = {}

pages["home"] = "Home Page"
pages["search"] = "Search Page"
pages["profile"] = "Profile Page"

print(pages)
# {'home': 'Home Page', 'search': 'Search Page', 'profile': 'Profile Page'}

del pages["search"]

print(pages)
# {'home': 'Home Page', 'profile': 'Profile Page'}

pages["settings"] = "Settings Page"
pages["help"] = "Help Page"

print(pages)
# {'home': 'Home Page', 'profile': 'Profile Page', 'settings': 'Settings Page', 'help': 'Help Page'}
```

## Check if a key exists

```python
if "name" in person:
    print("Name is present.")
```

## Get all keys, values, or key-value pairs

```python
keys = person.keys()       # dict_keys(['name', 'age', 'job']), View Object(Iterable but not Mutable)
values = person.values()   # dict_values(['Alice', 31, 'Engineer']),  View Object
items = person.items()     # dict_items([('name', 'Alice'), ('age', 31), ('job', 'Engineer')]), View object that is like a list of tuples
```

## Looping through a dictionary

```python
for key in person:
    print(key, person[key])

for key, value in person.items():
    print(f"{key}: {value}")
```

## Dictionary comprehension (quick way to build a dict)

Syntax: `{key_expression: value_expression for item in iterable if optional_condition}`

```python
squares = {x: x*x for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

## Other useful operations

```python
# Using .pop() to remove and return a value
age = person.pop("age")  # Removes 'age' and returns its value

# .clear() removes all items
person.clear()  # Dictionary is now empty

# .update() adds key-value pairs from another dict
person.update({"name": "Bob", "city": "LA"})  # Adds/updates keys
```
