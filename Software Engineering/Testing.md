# Testing

Testing is the process of checking that software behaves the way we expect. A test usually sets up some input, runs the code, and verifies the output or side effect.

```python
def add(a, b):
    return a + b

result = add(2, 3)

assert result == 5
```

The `assert` statement is the core idea behind most automated tests: if the expression is true, the test passes; if it is false, the test fails. A testing framework does not change this basic idea. It mainly reduces boilerplate, discovers tests automatically, gives better failure messages, and helps organize setup/cleanup.

Without a framework, a test file may need its own runner:

```python
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5

if __name__ == "__main__":
    test_add()
    print("All tests passed")
```

With `pytest`, the test can be written as a plain function whose name starts with `test_`. `pytest` finds and runs it automatically:

```python
def test_add():
    assert add(2, 3) == 5
```

## Types of Testing

Tests can be organized by level. Lower-level tests check smaller pieces of code in isolation. Higher-level tests check more pieces working together, usually closer to how a user actually uses the system.

### Unit Tests

Unit tests check one small unit of behavior, usually one function, class, or method. They should be fast, focused, and easy to understand.

For example, suppose an application has a cart calculation function:

```python
def calculate_total(items):
    return sum(item["price"] * item["quantity"] for item in items)
```

A unit test checks only this calculation:

```python
def test_calculate_total():
    items = [
        {"name": "Keyboard", "price": 50, "quantity": 1},
        {"name": "Mouse", "price": 25, "quantity": 2},
    ]

    assert calculate_total(items) == 100
```

The test has three common parts:

- Arrange: create the input data.
- Act: call the code being tested.
- Assert: check the expected result.

```python
def test_calculate_total_with_empty_cart():
    items = []

    total = calculate_total(items)

    assert total == 0
```

Unit tests are useful because they make small rules explicit. If `calculate_total` later changes and breaks empty carts or quantity handling, the failing unit test points directly to the broken behavior.

### Integration Tests

Integration tests check that multiple parts of the system work together correctly. They are broader than unit tests, but usually still smaller than a full user workflow.

Continuing the cart example, imagine the application stores products in a repository and has a service that builds a cart total from product IDs:

```python
class ProductRepository:
    def __init__(self, products):
        self.products = products

    def get(self, product_id):
        return self.products[product_id]


def calculate_total(items):
    return sum(item["price"] * item["quantity"] for item in items)


def checkout_total(cart, product_repository):
    items = []

    for product_id, quantity in cart.items():
        product = product_repository.get(product_id)
        items.append({
            "name": product["name"],
            "price": product["price"],
            "quantity": quantity,
        })

    return calculate_total(items)
```

The `calculate_total` unit tests already prove that the calculation works by itself. The integration test can now focus on whether the repository, cart shape, and calculation function connect correctly:

```python
def test_checkout_total_uses_product_prices_and_cart_quantities():
    products = {
        "keyboard": {"name": "Keyboard", "price": 50},
        "mouse": {"name": "Mouse", "price": 25},
    }
    repository = ProductRepository(products)
    cart = {
        "keyboard": 1,
        "mouse": 2,
    }

    total = checkout_total(cart, repository)

    assert total == 100
```

This test is not just testing arithmetic anymore. It checks whether product lookup, cart quantities, and total calculation work together. If the cart uses product IDs but the repository expects another key format, this kind of test can catch that mismatch.

### End to End Tests

End-to-end tests check a complete workflow from the outside, usually in the same way a real user or external client would use the application. They are the broadest and slowest type of automated test, but they give confidence that the whole system works together.

For the same checkout example, an end-to-end test might send an HTTP request to the application instead of calling `checkout_total` directly:

```python
def test_checkout_endpoint_returns_total(client):
    response = client.post("/checkout", json={
        "cart": {
            "keyboard": 1,
            "mouse": 2,
        }
    })

    assert response.status_code == 200
    assert response.json["total"] == 100
```

This test may involve routing, request parsing, application configuration, service code, repository access, and response formatting. Because it crosses more boundaries, it gives more system-level confidence, but failures are usually less precise than unit test failures.

The same behavior can therefore be tested at multiple levels:

- Unit test: `calculate_total` returns `100` for two products.
- Integration test: `checkout_total` combines cart data, product lookup, and calculation correctly.
- End-to-end test: `POST /checkout` returns the expected HTTP response for a real checkout request.

Good test suites usually contain many unit tests, fewer integration tests, and a smaller number of end-to-end tests. This keeps feedback fast while still checking the important system workflows.

### Manual Tests

Manual tests are checks performed by a person instead of an automated test runner. They are useful for exploratory testing, visual review, usability, and cases where human judgment matters.

For example, a developer might manually open the checkout page, add products to the cart, click checkout, and confirm that the total looks correct. This can catch layout or workflow problems, but it is slower and less repeatable than an automated test.

> Regression Testing Suite: The collection of tests (unit, integration, and E2E) that you run to make sure a new version of the program did not inadvertently regress any core features from the previous version

## Testing Frameworks

Programming languages usually provide a basic way to write tests, but teams often use testing frameworks to make test writing and test running easier.

### Python Tests

Python has a built-in `unittest` module:

```python
import unittest


class TestCartTotal(unittest.TestCase):
    def test_calculate_total(self):
        items = [
            {"name": "Keyboard", "price": 50, "quantity": 1},
            {"name": "Mouse", "price": 25, "quantity": 2},
        ]

        self.assertEqual(calculate_total(items), 100)


if __name__ == "__main__":
    unittest.main()
```

This works, but it requires class-based tests, `self.assertEqual`, and a runner block when executing the file directly.

`pytest` is an external Python testing framework that usually lets the same test be written with less boilerplate:

```python
def test_calculate_total():
    items = [
        {"name": "Keyboard", "price": 50, "quantity": 1},
        {"name": "Mouse", "price": 25, "quantity": 2},
    ]

    assert calculate_total(items) == 100
```

`pytest` helps by:

- Discovering files and functions named like tests.
- Using plain `assert` statements with readable failure output.
- Providing fixtures for shared setup, such as a test client or test database.
- Supporting unit, integration, and end-to-end style tests in the same test suite.

### Common Frameworks

- JavaScript/TypeScript: Jest, Vitest
- C++: GoogleTest
- Python: `pytest`, `unittest`
