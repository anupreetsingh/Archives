# Decorators

A decorator is a Python language feature that takes a function or class and returns a wrapped/modified version of it to update the behavior of that function or class.

## The Core Idea

This:

```python
def decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

@decorator
def greet():
    print("Hello")
```

is equivalent to this:

```python
def greet():
    print("Hello")

greet = decorator(greet)
```

For functions, the `@decorator` syntax is only shorthand for:

```python
function_name = decorator(function_name)
```

So the decorator is basically rebinding a modified object to the same name.

The original function object is passed into another function, and the name is rebound to whatever the decorator returns.

The same idea works for classes:

```python
@decorator
class User:
    ...
```

means:

```python
class User:
    ...

User = decorator(User)
```

So a decorator can receive a function or a class, then return the same object, a modified object, or a wrapper/replacement object.

You can make any sort of custom decorator of your choice. But keep a few things in mind:

### Preserving metadata of original function

See [`functools.wraps()`](<Functools.md#wraps>) for preserving the original function's metadata when writing a wrapper. The examples below use that pattern.

### Decorators With Arguments

Sometimes the decorator itself needs configuration.

Example:

```python
from functools import wraps

def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}")

greet("Ada")
```

This has three levels:

- `repeat(times)` receives the decorator argument
- `decorator(func)` receives the function being decorated
- `wrapper(*args, **kwargs)` runs when the decorated function is called

This:

```python
@repeat(3)
def greet(name):
    print(f"Hello, {name}")
```

means:

```python
def greet(name):
    print(f"Hello, {name}")

greet = repeat(3)(greet)
```

## Layer 1: General Programming Usage

Once the language mechanism is clear, decorators become useful for removing repeated code.

Many functions need the same surrounding behavior:

- log when they run
- measure how long they take
- validate inputs
- check permissions
- retry after failure
- cache expensive results

Instead of writing that logic inside every function, a decorator can wrap the repeated behavior around the function.

Example:

```python
@timer
def expensive_operation():
    total = 0
    for number in range(1_000_000):
        total += number
    return total
```

When `expensive_operation()` runs, the `@timer` decorator records the start time, runs the function, records the end time, and prints the elapsed time.

### Common General-Purpose Decorators

Most decorator names are not built into Python directly. Some come from the standard library, some come from frameworks or third-party libraries, and some are custom decorators you define yourself.

- [`@cache` and `@lru_cache`](<Functools.md#cache-and-lru_cache>): See the caching discussion for their differences, requirements, and example.

- Install with `python -m pip install codetiming`, then use `from codetiming import Timer`
  `@Timer(name="expensive_operation", text="{name} took {:.4f} seconds")`: Prints or logs how long a function takes to run. This is a third-party decorator.

- Define `timer` yourself, or import it from your own helper module if you created one.
  `@timer`: Measures how long a function takes to run. Python does not provide this decorator directly.

- Define `debug` yourself, or import it from your own helper module if you created one.
  `@debug`: Prints useful information when a function is called, such as arguments and return value. Python does not provide this decorator directly.

- `from tenacity import retry`
  `@retry`: Runs a function again if it fails, usually for temporary errors such as network requests or database calls. `tenacity` is a common third-party library for this.

- `from pydantic import validate_call`
  `@validate_call`: Checks function arguments against type hints before allowing the function to run. This is a practical library version of a validation decorator.

- `from django.contrib.auth.decorators import login_required`
  `@login_required`: Allows access only if the user is logged in. This is commonly used on Django views.

- `from django.contrib.auth.decorators import permission_required`
  `@permission_required("app.permission_name")`: Allows access only if the user has a specific permission. This is commonly used on Django views.

Example with a third-party timer decorator:

```python
from codetiming import Timer


@Timer(name="expensive_operation", text="{name} took {:.4f} seconds")
def expensive_operation():
    total = 0
    for number in range(1_000_000):
        total += number
    return total
```

Example with custom decorators:

```python
from functools import wraps
from time import perf_counter


def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = perf_counter()
        result = func(*args, **kwargs)
        end = perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result

    return wrapper


def debug(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result

    return wrapper


@timer
@debug
def add(a, b):
    return a + b
```

## Layer 2: Framework Usage

Many people first encounter decorators through frameworks.

Framework decorators often do not just wrap behavior. They may also register a function or class somewhere inside the framework.

For example, in FastAPI:

```python
@app.get("/users")
def get_users():
    return [{"id": 1, "name": "Ada"}]
```

`@app.get("/users")` registers `get_users` as the handler for the `GET /users` route.

Without decorator syntax, the FastAPI framework could use a more explicit approach:

```python
def get_users():
    return [{"id": 1, "name": "Ada"}]

app.add_api_route("/users", get_users, methods=["GET"])
```

The decorator version is cleaner because the route and handler are written together.

Frameworks can also use decorators on classes.

Example from Django admin:

```python
from django.contrib import admin

@admin.register(User)
class UserAdmin(admin.ModelAdmin):
    list_display = ["id", "email"]
```

Here, `@admin.register(User)` registers the `User` data model with Django’s admin site to make it appear there and uses `UserAdmin` configuration class, which inherits from `admin.ModelAdmin` and controls how the `User` model is displayed and managed in the admin web interface.

### Common Decorators At This Layer

#### FastAPI Decorators

FastAPI commonly uses decorators to register route handler functions.

```python
@app.get("/users")
def get_users():
    return [{"id": 1, "name": "Ada"}]
```

Here, `@app.get("/users")` registers `get_users` as the handler for `GET /users`.

To list out a few common FastAPI decorators:

1. `@app.get(...)`

Registers a handler for an HTTP `GET` request. This is commonly used for reading or fetching data.

2. `@app.post(...)`

Registers a handler for an HTTP `POST` request. This is commonly used for creating new data.

3. `@app.put(...)`

Registers a handler for an HTTP `PUT` request. This is commonly used for replacing or updating a full resource.

4. `@app.patch(...)`

Registers a handler for an HTTP `PATCH` request. This is commonly used for partial updates.

5. `@app.delete(...)`

Registers a handler for an HTTP `DELETE` request. This is commonly used for deleting data.

6. `@router.get(...)`, `@router.post(...)`, etc.

```python
@router.get("/orders")
def list_orders():
    ...
```

Router decorators work like app decorators, but they register routes on an `APIRouter`. This is common when a project is split into multiple route files.

#### Django Decorators

Django commonly uses decorators to add access control, permission checks, method restrictions, caching, CSRF behavior, or admin registration.

1. `@login_required`

```python
@login_required
def dashboard(request):
    ...
```

`@login_required` allows the view to run only if the current user is logged in.

2. `@permission_required(...)`

```python
@permission_required("orders.view_order")
def order_list(request):
    ...
```

`@permission_required(...)` allows the view to run only if the user has a specific Django permission.

3. `@require_http_methods(...)`

Restricts a view to specific HTTP methods.

```python
@require_http_methods(["GET", "POST"])
def profile(request):
    ...
```

4. `@require_GET` and `@require_POST`

These are shortcuts for allowing only one HTTP method. `@require_GET` allows only `GET` requests, and `@require_POST` allows only `POST` requests.

5. `@csrf_exempt`

Disables CSRF protection for a view. This should be used carefully because CSRF protection is an important security feature.

6. `@cache_page(...)`

Caches the response from a view for a fixed amount of time. This is useful when the same response is expensive to generate and does not change often.

7. `@admin.register(ModelName)`

Registers a model with Django's admin site and connects it to a custom `ModelAdmin` class.

#### pytest Decorators

pytest uses decorators to configure how tests are prepared, repeated, skipped, or labeled.

1. `@pytest.mark.parametrize(...)`

```python
@pytest.mark.parametrize("number, expected", [(2, 4), (3, 9)])
def test_square(number, expected):
    assert number * number == expected
```

`@pytest.mark.parametrize(...)` runs the same test multiple times with different values. This is useful when the test logic is the same, but the inputs and expected outputs change.

2. `@pytest.fixture`

```python
@pytest.fixture
def user():
    return {"id": 1, "email": "ada@example.com"}
```

`@pytest.fixture` marks a function as reusable test setup. Other tests can request it by using the fixture name as a parameter.

3. `@pytest.mark.skip(...)`

Skips a test. This is useful when a test should not run yet, usually because a feature is incomplete or temporarily unavailable.

4. `@pytest.mark.xfail(...)`

Marks a test as expected to fail. If it fails, pytest does not treat it like a normal failure.

5. `@pytest.mark.slow`

Labels a test with a custom marker. Teams often use this to run or exclude slower tests separately.

## Layer 3: OOP-Related Decorators

Decorators are also common in object-oriented code.

Many classes need the same method-related or class-related patterns:

- expose computed values like normal attributes
- define helper functions inside the class namespace
- create alternate constructors
- automatically generate common class methods
- register or modify the class object itself

Instead of writing all of that manually, OOP-related decorators can change how methods or classes behave.

### Common Method Decorators Used In OOP

Method decorators are placed on methods inside a class. They change how that method is accessed or how it receives context.

1. `@property`

```python
class Person:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name

    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
```

`@property` lets a method be accessed like an attribute. Even though `full_name` is defined as a method, it is used as `person.full_name`, not `person.full_name()`.

This is useful when a value should feel like stored data but is actually computed from other data.

2. `@property_name.setter`

```python
class Person:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Age cannot be negative")
        self._age = value
```

`@age.setter` defines what should happen when someone assigns to `person.age`.

This is useful when setting an attribute needs validation or extra logic.

3. `@staticmethod`

```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b
```

`@staticmethod` defines a method that belongs in the class namespace but does not receive `self` or `cls`.

Use it when the function is related to the class conceptually, but does not need object state or class state.

4. `@classmethod`

```python
class User:
    def __init__(self, username):
        self.username = username

    @classmethod
    def from_email(cls, email):
        # cls is the User class here.
        username = email.split("@")[0]
        # Same as User(username) in this example; this step in turn returns an instance of User class and now calls the __init__
        return cls(username)

# This uses from_email as an alternate constructor.
user = User.from_email("ada@example.com")
```

`@classmethod` defines a method that receives the class itself as `cls`.

Class methods are commonly used for alternate constructors. Here, `User.from_email("ada@example.com")` creates a `User` from an email instead of directly passing a username.

### Common Class Decorators Used In OOP

Class decorators are placed above a class definition. They receive the class object and can return the same class, a modified class, or a replacement class.

1. `@dataclass`

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
```

`@dataclass` modifies the class by adding common methods like `__init__`, `__repr__`, and `__eq__`.

This is useful for classes that mainly store data.

2. [`@functools.total_ordering`](<Functools.md#total_ordering>): See the comparison-method requirements and class example in the `functools` note.

3. Registration-style class decorators

```python
@register_model
class User:
    ...
```

Some libraries use class decorators to register a class somewhere, such as in a plugin registry, admin system, serializer system, or framework configuration.

The class may not visibly change, but the framework records it for later use.
