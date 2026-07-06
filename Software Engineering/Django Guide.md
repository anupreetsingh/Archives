# Django Guide

## High-Level Mental Model

Django is a Python based backend web framework. It gives you a structured way to handle HTTP requests, route URLs, talk to a database, render HTML, validate forms, manage users, and run an admin interface.

### Project Structure

A Django **project** is the full web application. It contains the global configuration for the site.

We create a new Django project with `django-admin`, the command-line tool installed with Django.

```bash
django-admin startproject my_project
```

This creates the initial project package and a `manage.py` file. After that, most Django framework commands are usually run through the project's own `manage.py` entry point.

Typical project layout:

```text
my_project/
├── manage.py
├── my_project/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── app_name/
    ├── migrations/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
└── app_name_2/
    .
    .
```

Important project files:

| File | Purpose |
| --- | --- |
| `manage.py` | CLI Entry point for development tasks |
| `settings.py` | Global project configuration |
| `urls.py` | Main URL routing table |
| `asgi.py` | ASGI entry point for async-capable servers |
| `wsgi.py` | WSGI entry point for traditional Python web servers |

### CLI Entry Point Django Commands

`manage.py` is the project-specific CLI entry point. It loads the project's settings and lets us run Django management commands against that project.

Common commands run through `manage.py`:

| Command | Purpose |
| --- | --- |
| `python manage.py runserver` | Start the local development server |
| `python manage.py startapp catalog` | Create a new Django app |
| `python manage.py makemigrations` | Create migration files from model changes |
| `python manage.py migrate` | Apply migrations to the database |
| `python manage.py createsuperuser` | Create an admin user |
| `python manage.py test` | Run the project's automated tests |
| `python manage.py shell` | Open a Django-aware Python shell |

### Core Packages

Django itself ships with many core packages that you import directly like:

| Package | Purpose |
| --- | --- |
| `django.conf` | Project settings and configuration |
| `django.urls` | URL routing |
| `django.http` | Request and response objects |
| `django.shortcuts` | Helper functions like `render`, `redirect`, and `get_object_or_404` |
| `django.views` | View utilities and class-based views |
| `django.db` | ORM and database layer |
| `django.forms` | Forms and validation |
| `django.template` | Template engine |
| `django.core` | Core utilities, management commands, mail, files, and exceptions |
| `django.middleware` | Built-in middleware classes |

### Apps

A Django **app** is a Python package inside a project that owns one area of functionality. A project can contain many apps.

#### First Party Apps

Django has built-in first party apps under `django.contrib` that can be added to `INSTALLED_APPS` in `settings.py`.

Common built-in apps from `django.contrib`:

| App | Purpose |
| --- | --- |
| `django.contrib.admin` | Built-in admin site. Creates admin log tables during migration. |
| `django.contrib.auth` | Users, groups, permissions, and authentication. Creates user, group, and permission tables during migration. |
| `django.contrib.contenttypes` | Tracks installed models and enables generic relationships. Creates a content type table during migration. |
| `django.contrib.sessions` | Session storage. Creates a session table during migration. |
| `django.contrib.messages` | Temporary flash messages. Does not usually create its own database table; commonly uses sessions or cookies for storage. |
| `django.contrib.staticfiles` | Static file discovery and collection. Does not create database tables. |

#### Third Party Apps

Third-party apps are installed with a package manager like `pip` or `uv`. If the package provides a Django app, we also add that app to `INSTALLED_APPS`.

For example, Django REST Framework is a third-party app package used to build REST APIs. After installing it, we include it as  `rest_framework`, in `INSTALLED_APPS`.

#### Custom Apps

Your own project apps are the custom packages you create for your application's features.

Examples:

```text
accounts/       # users, login, registration, profiles
catalog/        # products, categories, inventory
cart/           # shopping cart behavior
checkout/       # order creation and checkout flow
billing/        # payments, invoices, refunds
notifications/  # emails, alerts, user messages
```

We use the CLI entry point `manage.py` to create an app.

```bash
python manage.py startapp catalog
```

This creates an `catalog/` package with the standard app files. After creating the app, you usually add it to `INSTALLED_APPS` in `settings.py` so Django knows the app is part of the project.

`INSTALLED_APPS` can include your own apps, built-in Django apps from `django.contrib`, and third-party apps. For example, if we are building a REST API, we would install Django REST Framework and add `rest_framework` to `INSTALLED_APPS`.

Files created by `startapp`:

| File | Purpose |
| --- | --- |
| `migrations/` | Database schema history |
| `__init__.py` | Marks the app directory as a Python package |
| `admin.py` | Admin interface registration |
| `apps.py` | App configuration |
| `models.py` | Database models |
| `tests.py` | Automated tests |
| `views.py` | Request-handling logic |

Files commonly added later:

| File | Purpose |
| --- | --- |
| `urls.py` | App-specific URL routes, if created |
| `forms.py` | Form definitions, if created |
| `serializers.py` | API data conversion and validation when using Django REST Framework |
| `tests/` | Larger test package when one `tests.py` file becomes too small |

## Models

Data models are classes that define the schema and behavior of tables in your database. Django uses these model classes through its ORM.

```python
from django.db import models

# Inheriting models.Model makes Post a data model class that defines schema for a table.
class Post(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
```

Because `Post` inherits from `models.Model`, Django provides the initializer for model instances. When we write:

```python
post = Post(title="Intro to Django", body="...")
```

`models.Model` initializer does something like:

```python
post.title = "Intro to Django"
post.body = "..."
```

At this point, `post` is only a Python object in memory. It becomes a row in the database only when it is saved:

```python
post.save()
```

So the model class defines the table schema, and each saved model instance represents one row in that table.

`save()` adds or updates a row in an already-migrated database table immediately.

### Fields and Field Arguments

The values passed into field definitions are field arguments. They configure the rules, defaults, validation, database behavior, or display behavior for that field.

```python
title = models.CharField(max_length=200, unique=True, blank=False)
name = forms.CharField(max_length=100, required=True)
name = serializers.CharField(max_length=100, required=True)
```

Read a field definition like this:

```python
genre = models.CharField(max_length=100, blank=True)
```

- `models.CharField` tells Django this is text.
- `max_length=100` says the text can be up to 100 characters.
- `blank=True` says model/form validation allows this field to be empty.

Common model field arguments:

| Argument | Meaning |
| --- | --- |
| `max_length=200` | Limits text to 200 characters |
| `blank=True` | Allows the field to be empty during validation. For text fields, this usually means `""` is allowed |
| `blank=False` | Requires a non-empty value during validation |
| `null=True` | Allows the database column to store `NULL` |
| `null=False` | Requires the database column to store a non-`NULL` value |
| `default=0` | Uses `0` if no value is provided |
| `default="draft"` | Uses `"draft"` if no value is provided |
| `unique=True` | Requires every row to have a different value for this field |
| `choices=[("draft", "Draft")]` | Limits the field to specific allowed values |
| `db_index=True` | Adds a database index to make lookups on this field faster |
| `auto_now_add=True` | Sets the value once when the row is first created |
| `auto_now=True` | Updates the value every time the row is saved |
| `max_digits=8` | For `DecimalField`, allows 8 total digits |
| `decimal_places=2` | For `DecimalField`, stores 2 digits after the decimal point |

Common relationship field arguments:

```python
author = models.ForeignKey(Author, related_name="books", on_delete=models.CASCADE)
```

| Argument | Meaning |
| --- | --- |
| `Author` | The related model this field points to |
| `on_delete=models.CASCADE` | If the related `Author` is deleted, delete this object too |
| `on_delete=models.SET_NULL` | If the related object is deleted, set this field to `NULL`. Usually needs `null=True` |
| `on_delete=models.PROTECT` | Prevent deleting the related object while this object still points to it |
| `related_name="books"` | Lets the related object access these objects with `author.books.all()` |

Common form and serializer field arguments:

| Argument | Meaning |
| --- | --- |
| `required=True` | Input must include this field |
| `required=False` | Input may omit this field |
| `label="Title"` | Changes the display label for a form/API field |
| `initial="draft"` | Gives a form field a starting display value |
| `widget=forms.Textarea` | Displays a form field using a larger text box |
| `read_only=True` | The serializer includes this field in output. The client can read it, but cannot send it as input to create or edit it. |
| `write_only=True` | The client can send this field as input. The serializer accepts it, but does not include it in output(client writes only). |
| `allow_null=True` | Serializer accepts `null` as a valid API value |

| Field type | What the arguments affect |
| --- | --- |
| Model field | Database schema and model-level validation, like `max_length`, `default`, `null`, `blank`, and `unique` |
| Form field | HTML form validation and display, like `required`, `label`, `initial`, and `widget` |
| Serializer field | API validation and output behavior, like `required`, `read_only`, `write_only`, and `allow_null` |

> The same general idea appears in models, forms, and serializers: the field class chooses the kind of value, and the arguments configure that field. The exact argument names are not always the same. For `ModelForm` and `ModelSerializer`, many fields are generated from the model, so model field arguments can also influence form or API validation.

### Migrations

To change the table structure after updating a model class, Django uses **migrations**.

Example:

```bash
python manage.py makemigrations
python manage.py migrate
```

Like many other tasks in Django development, we use the CLI entry point `manage.py` to run framework commands.

Here, `makemigrations` looks at your model changes and creates migration files inside the corresponding app's `migrations/` directory.

`migrate` applies those migration files to the actual database.

### Using Django's ORM

Every model gets a manager at `Model.objects`. The manager is the entry point for creating, finding, updating, and deleting rows for that model.

```python
Post.objects.all()
Post.objects.filter(title="Intro to Django")
Post.objects.get(id=1)
Post.objects.create(title="Intro to Django", body="...")
```

Most ORM query methods return a **QuerySet**. A QuerySet represents a database query and usually contains zero, one, or many model objects.

Common methods for getting rows:

| Method | Returns | Purpose |
| --- | --- | --- |
| `Model.objects.all()` | QuerySet | Get all rows for that model |
| `Model.objects.filter(...)` | QuerySet | Get rows that match the given conditions |
| `Model.objects.exclude(...)` | QuerySet | Get rows that do not match the given conditions |
| `Model.objects.get(...)` | One model object | Get exactly one matching row |
| `Model.objects.first()` | One model object or `None` | Get the first row from a query |
| `Model.objects.last()` | One model object or `None` | Get the last row from a query |
| `Model.objects.exists()` | Boolean | Check whether at least one matching row exists |
| `Model.objects.count()` | Integer | Count matching rows |
| `Model.objects.order_by(...)` | QuerySet | Sort rows |
| `Model.objects.values(...)` | QuerySet of dictionaries | Return selected fields as dictionaries instead of model objects |
| `Model.objects.values_list(...)` | QuerySet of tuples or values | Return selected fields as tuples or plain values |

Common methods for changing rows:

| Method | Returns | Purpose |
| --- | --- | --- |
| `Model.objects.create(...)` | One saved model object | Create a new row immediately |
| `object.save()` | `None` | Save a new object or update an existing object |
| `QuerySet.update(...)` | Number of updated rows | Update matching rows directly in the database |
| `object.delete()` | Delete result tuple | Delete one model object |
| `QuerySet.delete()` | Delete result tuple | Delete all rows in a QuerySet |

`create()` takes model field names as keyword arguments.

```python
post = Post.objects.create(
    title="Intro to Django",
    body="...",
    published=True,
)
```

This creates a `Post` object and saves it as a database row immediately.

This:

```python
Post.objects.create(title="Intro to Django", body="...")
```

is a shortcut for:

```python
post = Post(title="Intro to Django", body="...")
post.save()
```

#### `filter()` Arguments

`filter()`, `exclude()`, and `get()` receive lookup arguments that describe which rows should match.

A lookup argument usually has this shape:

```python
field_name=value
```

or:

```python
field_name__lookup_type=value
```

Examples:

```python
Post.objects.filter(title="Intro to Django")
Post.objects.filter(author="Alex", published=True)
Post.objects.exclude(published=False)
Post.objects.get(id=1)
```

`title="Intro to Django"` means the `title` field must equal `"Intro to Django"`.

`published=True` means the `published` field must equal `True`.

`title__icontains="django"` means the `title` field must contain `"django"` without caring about uppercase/lowercase.

`views__gt=100` means the `views` field must be greater than `100`.

Multiple arguments usually mean **AND**:

```python
Post.objects.filter(author="Alex", published=True)
```

This means:

```text
author is Alex AND published is True
```

For **OR** logic, use `Q` objects:

```python
from django.db.models import Q


Post.objects.filter(Q(author="Alex") | Q(author="Sam"))
```

This means:

```text
author is Alex OR author is Sam
```

You can also negate a condition with `~Q(...)`:

```python
Post.objects.filter(~Q(title__icontains="draft"))
```

Common lookup arguments:

| Lookup | Example | Meaning |
| --- | --- | --- |
| exact match | `title="Intro to Django"` | Field equals this value |
| `__iexact` | `title__iexact="intro to django"` | Case-insensitive exact match |
| `__contains` | `title__contains="Django"` | Field contains this text |
| `__icontains` | `title__icontains="django"` | Field contains this text, case-insensitive |
| `__startswith` | `title__startswith="Intro"` | Field starts with this text |
| `__istartswith` | `title__istartswith="intro"` | Field starts with this text, case-insensitive |
| `__endswith` | `title__endswith="Guide"` | Field ends with this text |
| `__gt` | `views__gt=100` | Field is greater than this value |
| `__gte` | `views__gte=100` | Field is greater than or equal to this value |
| `__lt` | `views__lt=100` | Field is less than this value |
| `__lte` | `views__lte=100` | Field is less than or equal to this value |
| `__in` | `id__in=[1, 2, 3]` | Field is one of these values |
| `__isnull` | `published_at__isnull=True` | Field is `NULL` or not `NULL` |
| `__range` | `views__range=(10, 100)` | Field is between two values |
| date part lookup | `created_at__year=2026` | Match part of a date/datetime field |

The part before the double underscore is the model field name. The part after the double underscore is the lookup type.

Queries can be chained because methods like `filter()`, `exclude()`, and `order_by()` return another QuerySet.

```python
posts = (
    Post.objects
    .filter(published=True)
    .exclude(title__icontains="draft")
    .order_by("-created_at")
)
```

`order_by("created_at")` sorts oldest first.

`order_by("-created_at")` sorts newest first.

You can limit results with slicing:

```python
latest_five_posts = Post.objects.filter(published=True).order_by("-created_at")[:5]
```

Use `get()` only when exactly one row should match.

```python
post = Post.objects.get(id=1)
```

If no row matches, `get()` raises `Post.DoesNotExist`.

If more than one row matches, `get()` raises `Post.MultipleObjectsReturned`.

If zero or many rows are normal possibilities, use `filter()` instead:

```python
posts = Post.objects.filter(author="Alex")
```

## URLs

URLs can be defined directly in the project-level `urls.py` file, or they can be defined inside an app-level `urls.py` file and included by the main project `urls.py`.

A URLconf, usually named `urls.py`, maps URL patterns to **views**, which are the handler functions or classes for requests routed to them.

`path()` is a function that defines one URL route. It takes a route pattern, the view that should handle matching requests, and optionally a URL-route name(that is helpful for looking up the URL path using functions like `reverse()`[Used Here](#testing-views)).

```python
# In the catalog app
# catalog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("products/", views.product_list, name="product_list"),
    path("categories/", views.category_list, name="category_list"),
]
```

The main project-level `urls.py` usually includes routes from each app using `include()`.

```python
# my_project/urls.py
from django.urls import include, path

urlpatterns = [
    path("catalog/", include("catalog.urls")),
]
```

`include("catalog.urls")` attaches all `urlpatterns` from `catalog/urls.py` under the `catalog/` prefix.

Final routes:

```text
/catalog/products/    -> catalog.views.product_list
/catalog/categories/  -> catalog.views.category_list
```

## Views

Views are the request handler functions or classes that Django runs after a URL pattern matches. They are equivalent to handler functions in FastAPI.

After Django's URLconf routes the request to the matching view, Django passes an `HttpRequest` object into the view. The view reads the request, coordinates whatever work is needed, and returns an HTTP response.

- Read request data
- Query models
- Apply business logic
- Return JSON for an API
- Return HTML directly or by rendering a template
- Redirect users
- Return HTTP errors when needed

### View Input(HttpRequest)

`request` is the `HttpRequest` object Django passes into the view.

Information we get from `request` includes:

| Attribute | Meaning |
| --- | --- |
| `request.method` | HTTP method, such as `"GET"` or `"POST"` |
| `request.GET` | Query string data from the URL |
| `request.POST` | HTML form data from a POST request, commonly used with Django forms (`ModelForm`) |
| `request.FILES` | Uploaded files from form submissions |
| `request.body` | Raw request body bytes, useful when manually parsing request data |
| `request.data` | Parsed API input, often JSON; commonly used in Django REST Framework requests |
| `request.headers` | HTTP request headers |
| `request.COOKIES` | Cookies sent by the browser |
| `request.user` | The authenticated user, if authentication middleware is enabled |
| `request.session` | Session data for the current browser/client |

In normal Django form views, submitted browser form data is usually read from `request.POST`. In DRF API views, parsed API data is usually read from `request.data`.

### View Responses

A view must return an HTTP response object, or raise an HTTP exception such as `Http404`.

#### Normal HTTP Response

For simple text or manually written HTML, return `HttpResponse` directly:

```python
from django.http import HttpResponse


return HttpResponse("Hello from Django")
return HttpResponse("<h1>Products</h1>")
```

#### Rendered HTML Response

For server-rendered HTML, use `render()`. `render()` finds a template, fills it with context data, and returns an `HttpResponse` containing the final HTML. For detail visit [Rendering the Template](#rendering-the-template).

#### API Response

For API responses, return `JsonResponse`:

```python
from django.http import JsonResponse


return JsonResponse({"products": list(products)})
```

#### Redirect HTTP Response

For redirects, return `redirect()`. A redirect is also an HTTP response, usually an `HttpResponseRedirect` with status code `302`. It tells the browser to make a new request to another URL:

```python
from django.shortcuts import redirect


return redirect("product_list")
```

#### Error Response

For missing resources, raise `Http404`:

```python
from django.http import Http404


raise Http404("Product not found")
```

### Function-Based Views

A function-based view is a regular Python function that accepts a `request` object and returns an HTTP response.

```python
from django.http import JsonResponse
from .models import Product


def product_list(request):
    products = Product.objects.all().values("id", "name", "price")
    return JsonResponse({"products": list(products)})
```

If `catalog/urls.py` contains:

```python
path("products/", views.product_list, name="product_list")
```

then a request to `/catalog/products/` eventually runs:

```python
catalog.views.product_list(request)
```

### Class-Based Views

A class-based view organizes request-handling logic inside a class. Different HTTP methods can be represented by different class methods, such as `get()` or `post()`.

```python
import json

from django.http import JsonResponse
from django.views import View
from .models import Product


class ProductListView(View):
    def get(self, request):
        products = Product.objects.all().values("id", "name", "price")
        return JsonResponse({"products": list(products)})

    def post(self, request):
        data = json.loads(request.body)
        product = Product.objects.create(
            name=data["name"],
            price=data["price"],
        )
        return JsonResponse({"id": product.id, "name": product.name, "price": product.price})
```

Here, `get()` handles reading products, while `post()` handles creating a new product. This keeps different HTTP method behavior separated inside the same endpoint class.

In `catalog/urls.py`, class-based views are integrated into `urlpatterns` with `.as_view()`. `.as_view()` turns the class into a callable view function that Django can route requests to.

```python
path("products/", views.ProductListView.as_view(), name="product_list")
```

Now that `ProductListView` is included in `urlpatterns`, a request to `/catalog/products/` is routed to that view. The request object's HTTP method then determines which class method runs: `GET` runs `get()`, while `POST` runs `post()`.

## Templates

Templates are HTML files that Django renders when a view returns server-side HTML instead of JSON.

A template decides what the user sees in the browser, while the view decides which data should be passed into that page.

### Loading Templates

In `settings.py`, Django's `TEMPLATES` setting usually has `APP_DIRS` set to `True`.

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [],
        },
    },
]
```

`APP_DIRS=True` tells Django to look inside the `templates/` directory of every app listed in `INSTALLED_APPS`.

That means templates from different apps are loaded into one shared template search system.

If a view does this:

```python
render(request, "product_list.html")
```

These files would all be candidates for the same template name because they have been loaded in the same search system even though they come from different apps:

```text
catalog/templates/product_list.html
orders/templates/product_list.html
accounts/templates/product_list.html
```

This is why templates usually live inside an extra app-named folder inside `templates/`. A common app-level layout is:

```text
catalog/
├── templates/
│   └── catalog/
│       ├── product_list.html
│       └── product_detail.html
└── views.py
```

With that structure, the view asks for:

```python
render(request, "catalog/product_list.html")
```

and Django searches for the more specific template name `catalog/product_list.html` in its template search system.

### Rendering the Template

Basic syntax:

```python
render(request, template_name, context=None, status=None)
```

- `request` is the `HttpRequest` object passed into the view
- `template_name` is the template file Django should render
- `context` is an optional dictionary of data made available inside the template
- `status` is an optional HTTP status code for the response

```python
from django.shortcuts import render
from .models import Product


def product_list(request):
    products = Product.objects.all()
    return render(
        request,
        "catalog/product_list.html",
        {"products": products},
    )
```

In this example, `render()` returns an `HttpResponse` containing the final HTML from `catalog/product_list.html`. The queryset stored in the Python variable `products` is passed into the context dictionary under the key `"products"`, so Django makes it available in the template as `products`.

### Template Syntax And Context Data

Django templates use a small syntax on top of normal HTML.

In the examples below, names like `product`, `products`, and `product.name` are sample values from the template context. The Django syntax is the surrounding marker, such as `{{ ... }}` or `{% ... %}`.

| Syntax pattern | Example | Meaning |
| --- | --- | --- |
| `{{ value }}` | `{{ product.name }}` | Output a value into the HTML |
| `{# comment #}` | `{# show product name #}` | Write a template comment that will not appear in the final HTML |
| `{ value }` | `{ product.name }` | Plain text, because single curly braces have no special meaning |

**Filters**

A filter formats or transforms a value before Django outputs it. Filters use a pipe, `|`, after the value.

| Syntax pattern | Example | Meaning |
| --- | --- | --- |
| `{{ value|filter }}` | `{{ product.name|lower }}` | Output the product name in lowercase |
| `{{ value|filter:argument }}` | `{{ product.price|floatformat:2 }}` | Output a price like `4.99` with 2 decimal places |
| `{{ value|default:"text" }}` | `{{ product.description|default:"No description" }}` | Show fallback text if the value is empty |

**Template Tags**

A **template tag** is a Django template command written with `{% ... %}`. Tags do not usually print a value directly. They control rendering, build URLs, or connect templates together.

Common template tags:

| Tag pattern | Example | Purpose |
| --- | --- | --- |
| `{% for item in values %}` | `{% for product in products %}` | Loop over values |
| `{% empty %}` | `{% empty %}` | Render fallback HTML when a loop has no items |
| `{% endfor %}` | `{% endfor %}` | End a `for` loop |
| `{% if condition %}` | `{% if product.in_stock %}` | Conditionally render HTML |
| `{% elif condition %}` | `{% elif product.backordered %}` | Add another condition after an `if` |
| `{% else %}` | `{% else %}` | Render fallback HTML when previous `if` conditions are false |
| `{% endif %}` | `{% endif %}` | End an `if` block |
| `{% url "route_name" arg %}` | `{% url "product_detail" item.id %}` | Build a URL from a route name using Django's URL reversing behavior, like `reverse("product_detail", args=[item.id])` |
| `{% csrf_token %}` | `{% csrf_token %}` | Add the CSRF token required in POST forms |
| `{% include "template.html" %}` | `{% include "catalog/card.html" %}` | Render another template inside this template |
| `{% with name=value %}` | `{% with total=products|length %}` | Create a temporary template variable |
| `{% endwith %}` | `{% endwith %}` | End a `with` block |
| `{% block block_name %}` | `{% block content %}` | Define a named section that a child template can replace |
| `{% endblock %}` | `{% endblock %}` | End a block section |
| `{% extends "template.html" %}` | `{% extends "base.html" %}` | Reuse a parent template layout; put this at the top of a child template |

In `{% block content %}`, `content` is just the block name. It could be `title`, `main`, `sidebar`, or any other name, as long as the child template uses the same block name when replacing it.

Django template syntax is intentionally more limited than Python. Templates should format and display data, not perform heavy business logic. If the logic is complicated, it usually belongs in the view, model methods, querysets, or a dedicated helper.

#### Example: Using Context Data

The context dictionary passed to `render()` is the bridge between the view and the template. If a view returns:

```python
def product_detail(request, product_id):
    product = Product.objects.get(id=product_id)
    related_products = Product.objects.filter(category=product.category)

    return render(
        request,
        "catalog/product_detail.html",
        {
            "product": product,
            "related_products": related_products,
        },
    )
```

then the template can use `product` and `related_products` because those are the keys in the context dictionary.

```html
<h1>{{ product.name }}</h1>
<p>${{ product.price }}</p>

{% if product.in_stock %}
  <p>In stock</p>
{% else %}
  <p>Out of stock</p>
{% endif %}

<h2>Related products</h2>

{% for item in related_products %}
  <a href="{% url 'product_detail' item.id %}">
    {{ item.name }}
  </a>
{% empty %}
  <p>No related products found.</p>
{% endfor %}
```

The template does not use the original Python variable names directly. It uses the names from the context dictionary.

```python
{"product": product}
```

The left side, `"product"`, is the template variable name. The right side, `product`, is the Python object being passed from the view.

- `{{ product.name }}` uses dot lookup to read the `name` attribute from the `product` object.
- The `$` in `<p>${{ product.price }}</p>` is normal HTML text. Only `{{ product.price }}` is Django syntax.
- `{% if product.in_stock %}` renders different HTML depending on whether `product.in_stock` is true.
- `{% for item in related_products %}` loops over the value stored under `"related_products"`. Inside the loop, `item` is the current product.
- `{% empty %}` renders fallback HTML when `related_products` has no items.
- `{% url 'product_detail' item.id %}` builds a URL using the route name and the product ID.

A good convention is:

- The view prepares and names the data
- The template displays that data
- The template uses simple loops, conditionals, filters, and URL tags
- Complicated decisions stay in Python code

This is similar to URL names in templates. The template should refer to stable names and display-ready data instead of depending on hard-coded paths or complicated Python logic.

### Template Inheritance

Template inheritance lets you define a shared base page layout once and fill in sections from child templates.

For example, `base.html` is commonly used as the shared site-wide layout. It is often placed in a project-level `templates/` folder, then app-level templates like `catalog/product_list.html` extend it and fill in the page-specific sections.

```html
<!-- templates/base.html -->
<!doctype html>
<html>
  <head>
    <title>{% block title %}My Site{% endblock %}</title>
  </head>
  <body>
    <nav>
      <a href="{% url 'product_list' %}">Products</a>
    </nav>

    {% block content %}{% endblock %}
  </body>
</html>
```

`{% block title %}` and `{% block content %}` define regions that child templates can replace.

```html
<!-- catalog/templates/catalog/product_list.html -->
{% extends "base.html" %}

{% block title %}Products{% endblock %}

{% block content %}
  <h1>Products</h1>

  {% for product in products %}
    <p>{{ product.name }} - {{ product.price }}</p>
  {% empty %}
    <p>No products found.</p>
  {% endfor %}
{% endblock %}
```

`{% extends "base.html" %}` tells Django to use `base.html` as the outer layout. The child template only fills in the named blocks.

### URL Names In Templates

When a URL pattern has a `name`, templates can refer to that name instead of hard-coding the path.

If `catalog/urls.py` contains:

```python
path("products/", views.product_list, name="product_list")
```

then a template can link to it with:

```html
<a href="{% url 'product_list' %}">Products</a>
```

This is useful because the route path can change later while the name stays the same, so you do not have to update every template link manually.

## Forms

Forms are a core Django package that helps us define classes for validating and organizing submitted input.

When data is already trusted, we can create model rows directly using model classes:

```python
Product.objects.create(name="Keyboard", price=Decimal("49.99")) # `Product` is a model class in this case
```

But when data comes from a browser request in the form of an `HttpRequest`, it usually arrives as raw strings in `request.POST`. Instead of manually pulling values out of `request.POST`, converting them, checking required fields, and building model instances ourselves, we give the submitted data to a form.

```python
form = ProductForm(request.POST)

if form.is_valid():
    ...
```

The `.is_valid()` would help us find out if the input is validated or not.

In that sense, a Django form plays a similar role to a Pydantic model: it validates and normalizes input before the rest of the application trusts it.

Forms are important because they centralize validation instead of scattering input checks across views. The view stays focused on request flow, while the form owns the rules for whether submitted data is acceptable.

### Types of Forms

#### Plain Forms: `forms.Form`

A plain `forms.Form` is useful when the submitted data does not map directly to one model.

Unlike a model class that describes the fields a database table is supposed to have, a form class describes incoming user input.

```python
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
```

Django uses the 3 attributes and their definitions to validate submitted values.

If the browser submits:

```text
name=Sam
email=sam@example.com
message=Hello
```

then the form can validate that:

- `name` is present and not longer than 100 characters
- `email` looks like an email address
- `message` is present

After validation succeeds, normalized values are available through `form.cleaned_data`.

```python
form.cleaned_data["email"]
```

A plain form does not save a database row automatically. It validates input, stores cleaned Python values in `form.cleaned_data`, and then the view decides what to do with that cleaned data.

```python
form = ContactForm(request.POST)

if form.is_valid():
    email = form.cleaned_data["email"]
```

The important idea is that `request.POST` contains raw submitted strings, while `cleaned_data` contains validated Python values that the view can trust more.

#### Model-Backed Forms: `forms.ModelForm`

A `ModelForm` is tied to a model class. It builds form fields from model fields, and after `form.is_valid()` passes, `form.save()` can create or update a database row using an instance of that model class.

```python
from django import forms
from .models import Post


class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ["title", "body"]
```

In this example:

`Meta` class is configuration for the Model-Form class `PostForm` and inside it:

- `model = Post` tells Django which model the form is based on
- `fields = ["title", "body"]` tells Django which model fields/attributes should appear in the form

If `Post.title` is a `CharField`, Django creates a corresponding form field. If `Post.body` is a `TextField`, Django creates a larger text input by default.

Because this form is tied to `Post`, a view can call `form.save()` after `form.is_valid()` to create and save a new `Post` model instance as a row, or update an existing `Post` when the form is bound to an `instance`.

```python
form = PostForm(request.POST)

if form.is_valid():
    post = form.save()
```

This is the main difference from a plain form: a `ModelForm` can use valid submitted data to build and save the model instance for us.

### Creating Form Instances

Both plain forms and model-backed forms can be instantiated with arguments that Django's form initializer understands.

For a plain form, a simplified version looks like:

```python
ContactForm(data=None, files=None, initial=None)
```

For a model-backed form, a simplified version looks like:

```python
PostForm(data=None, files=None, instance=None, initial=None)
```

Common arguments:

| Argument | Meaning |
| --- | --- |
| `data` | Submitted form values, usually from `request.POST` |
| `files` | Uploaded files, usually from `request.FILES` |
| `initial` | Starting values used when displaying an unbound form |
| `instance` | Existing model object used by a `ModelForm` for editing instead of creating |

When we write:

```python
form = ContactForm(request.POST)
```

or:

```python
form = PostForm(request.POST)
```

`request.POST` is passed as the `data` argument. That makes the instance a **bound form**, meaning it now has submitted values attached to it and can be validated with `form.is_valid()`.

When we write:

```python
form = PostForm()
```

Since no submitted data is attached yet, the instance is an **unbound form**. This is usually used when displaying a blank form for the first time.

### Using Forms

Forms are usually integrated into the request/response workflow through views.

The form class is defined in `forms.py`, imported into `views.py`, created or bound inside a view, and then passed into a template for display.

```text
forms.py defines the form class
    |
    v
views.py imports and uses the form
    |
    v
template renders the form fields and errors
```

For the purpose of seeing how a form is used for validation, A typical form-handling view has two main paths:

- `GET` request: show an empty form
- `POST` request: validate the submitted form data

```python
from django.shortcuts import redirect, render
from .forms import PostForm


def create_post(request):
    if request.method == "POST":
        form = PostForm(request.POST)

        if form.is_valid():
            post = form.save() # creates a Post row after validation
            return redirect("post_detail", pk=post.pk)
    else:
        form = PostForm()

    return render(
        request,
        "blog/create_post.html",
        {"form": form},
    )
```

In a `GET` request, the user is asking to see the page. Since no form data has been submitted yet, `PostForm()` creates an unbound blank form that can be displayed in the template.

In a `POST` request, the user has submitted the HTML form. `PostForm(request.POST)` creates a bound form by attaching the submitted values to the form instance.

Those **submitted values** are the raw input that the form instance is supposed to validate.

The form instance compares the submitted values against its field definitions, such as required fields, max lengths, field types, and custom validation rules. Those field definitions may come from a plain form class or from a model class in the case of a `ModelForm`.

Then `form.is_valid()` decides whether the submitted values are acceptable.

If the values are valid, Django stores the cleaned Python values in the instance attribute `form.cleaned_data`. If the values are invalid, Django stores validation messages in `form.errors`.

If validation passes, `form.save()` creates and saves a `Post` model instance as a database row because `PostForm` is a `ModelForm`. The `redirect()` call then sends the browser to another route, such as the detail page for the newly created record.

If validation fails, the view does not redirect. It reaches the final `render()` call and sends the same bound form back to the template, including the user's submitted values and the validation error messages.

The final `render()` call passes the form instance(bound or unbound) into the template context:

```python
{"form": form}
```

That makes the form available in the template as `form`.

The simplest way to display it is:

```html
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Save</button>
</form>
```

`{{ form.as_p }}` renders each field wrapped in a paragraph tag. This is useful for examples, but real projects often render fields manually for more control over layout and styling.

```html
<form method="post">
  {% csrf_token %}

  <label for="{{ form.title.id_for_label }}">Title</label>
  {{ form.title }}
  {{ form.title.errors }}

  <label for="{{ form.body.id_for_label }}">Body</label>
  {{ form.body }}
  {{ form.body.errors }}

  <button type="submit">Save</button>
</form>
```

`{% csrf_token %}` is required for normal POST forms in Django templates. It protects against cross-site request forgery by including a token that Django checks when the form is submitted.

## Django REST Framework

Django REST Framework, often called **DRF**, is a third-party package for building APIs with Django.

Normal Django can return JSON with `JsonResponse`, but DRF adds a more complete API toolkit:

- Serializers for converting model objects to JSON-like data and validating incoming data
- API views and viewsets for organizing API behavior
- Routers for generating API URL patterns automatically
- Request and response helpers designed for APIs
- Authentication, permissions, pagination, filtering, and browsable API support

### Basic Setup

After installing Django REST Framework, add it to `INSTALLED_APPS`:

```python
# settings.py
INSTALLED_APPS = [
    ...
    "rest_framework",
]
```

DRF code is commonly organized like this:

```text
catalog/
├── models.py
├── serializers.py 
├── views.py
├── urls.py        
└── api_urls.py    
```

### Serializers

A serializer is used when an API needs to send model data as JSON-like data, or receive JSON-like input and turn it into validated model data.

A `ModelSerializer` maps model fields into API fields.

It is similar to a `ModelForm`, but it is designed for API workflows: validating JSON-like input, returning JSON-like output, and creating/updating model rows from API data (`request.data`) instead of Browser HTML form data (`request.POST`).

For a model like:

```python
# catalog/models.py
from django.db import models


class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    in_stock = models.BooleanField(default=True)
```

```python
# catalog/serializers.py
from rest_framework import serializers
from .models import Product


class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "price", "in_stock"]
```

`model = Product` tells DRF which model this serializer is based on.

`fields` controls which model fields are exposed through the API.

### ViewSets

A viewset groups related API actions into one class. It is similar to a class-based view, but instead of organizing behavior around request types like `get()` and `post()` , it organizes API behavior around actions like `list`, `retrieve`, `create`, `update`, and `destroy`.

For a specific model, those actions might include:

- Listing many records
- Retrieving one record
- Creating a record
- Updating a record
- Deleting a record

A `ModelViewSet` provides the standard CRUD handlers for that model in one class.

```python
# catalog/views.py
from rest_framework import viewsets
from .models import Product
from .serializers import ProductSerializer


class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

`queryset` tells DRF which objects this API handler(`ProductViewSet`) works with. It usually comes from the model’s manager, like `Product.objects.all()`, which returns `Product` records from the database.

`serializer_class` tells DRF how to manipulate those objects to and from API data. Usually, the serializer is based on the same model through its `Meta` class, like `model = Product`.

### Routers And `register()`

A router connects a viewset to URL patterns.

```python
# catalog/api_urls.py
from rest_framework.routers import DefaultRouter
from .views import ProductViewSet

# Creates a router object. Its job is to generate URL patterns for viewsets.
router = DefaultRouter()
router.register("products", ProductViewSet, basename="product")

# Exposes the generated API URL patterns so project-level urls.py can include them.
urlpatterns = router.urls
```

`router.register()` takes:

| Argument | Meaning |
| --- | --- |
| `"products"` | The URL prefix used in the actual path, like `/products/`, `/products/1/`, or `/products/in_stock/` |
| `ProductViewSet` | The viewset class that handles requests |
| `basename="product"` | The base name used for generated URL names, like `product-list`, `product-detail`, or `product-in-stock` |

For each URL pattern generated by `router.register()` for this ViewSet(`ProductViewSet`):

- The URL path starts with the URL prefix, such as `"products"`.
- The URL name starts with the `basename`, then adds the route/action name, such as `product-list`, `product-detail`, or `product-in-stock`.

This creates routes like:

| HTTP method | URL | ViewSet action | URL name |
| --- | --- | --- | --- |
| `GET` | `/products/` | `list` | `product-list` |
| `POST` | `/products/` | `create` | `product-list` |
| `GET` | `/products/1/` | `retrieve` | `product-detail` |
| `PUT` | `/products/1/` | `update` | `product-detail` |
| `PATCH` | `/products/1/` | `partial_update` | `product-detail` |
| `DELETE` | `/products/1/` | `destroy` | `product-detail` |

#### Custom Router Actions

Use `@action` when a `ViewSet` needs an extra endpoint that is related to the same resource, but is not one of the default CRUD actions.

```python
# catalog/views.py
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response

from .models import Product
from .serializers import ProductSerializer


class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    @action(detail=False, methods=["get"])
    def in_stock(self, request):
        products = Product.objects.filter(in_stock=True)
        serializer = self.get_serializer(products, many=True)
        return Response(serializer.data)

    @action(detail=True, methods=["post"])
    def mark_sold_out(self, request, pk=None):
        product = self.get_object()
        product.in_stock = False
        product.save()
        return Response({"status": "sold out"})
```

Because this viewset is registered with:

```python
router.register("products", ProductViewSet, basename="product")
```

DRF adds routes like:

| HTTP method | URL | ViewSet action | URL name |
| --- | --- | --- | --- |
| `GET` | `/products/in_stock/` | `in_stock` | `product-in-stock` |
| `POST` | `/products/1/mark_sold_out/` | `mark_sold_out` | `product-mark-sold-out` |

The URL path uses the action method name. The URL name uses the `basename` plus the action name, with underscores changed to hyphens.

`detail` controls whether the action belongs to the whole collection or one object:

| Option | Meaning | URL shape |
| --- | --- | --- |
| `detail=False` | Collection action | `/products/action_name/` |
| `detail=True` | Detail action for one object | `/products/1/action_name/` |

`methods` controls which HTTP methods the custom action accepts:

| `methods` value | Allowed HTTP method |
| --- | --- |
| `["get"]` | `GET` |
| `["post"]` | `POST` |
| `["put"]` | `PUT` |
| `["patch"]` | `PATCH` |
| `["delete"]` | `DELETE` |
| `["get", "post"]` | `GET` and `POST` |

This app's router URLs can then be included in the project-level `urls.py`:

```python
# my_project/urls.py
from django.urls import include, path


urlpatterns = [
    path("api/", include("catalog.api_urls")),
]
```

With that prefix, the final URLs become:

```text
/api/products/
/api/products/1/
```

### How The Pieces Fit Together

Because these router URLs are included under `path("api/", ...)`, reversing the URL name `product-list` points to `/api/products/`, and reversing `product-detail` with a product ID points to `/api/products/1/`.

When an API request hits one of those routes, DRF uses both the URL and the HTTP method to choose the appropriate viewset action. If the request contains JSON, DRF parses the JSON body and makes it available as `request.data`.

This is similar to a normal class-based view, where the URL routes to the class and the HTTP method decides whether `get()` or `post()` runs. In a viewset, the HTTP method maps to actions like `list`, `create`, `retrieve`, `update`, and `destroy`.

Since `ProductViewSet` is based on the `Product` queryset, those actions work with `Product` records.

For example:

```http
POST /api/products/
Content-Type: application/json

{
  "name": "Keyboard",
  "price": "49.99",
  "in_stock": true
}
```

The workflow is:

```text
POST /api/products/
    |
    v
POST maps to ProductViewSet.create()
    |
    v
DRF parses the JSON body into request.data
    |
    v
ProductSerializer validates request.data
    |
    v
ProductViewSet creates a Product row
    |
    v
DRF returns an API response
```

For a detail request:

```text
GET /api/products/1/
```

DRF matches the detail route `/api/products/1/` to `ProductViewSet`, and because the request method is `GET`, it maps the request to `ProductViewSet.retrieve()`.

## Admin

Django includes a built-in admin site for managing database records.

The admin is an internal web interface generated from your models. It is useful for manipulating or inspecting database records without building a custom management screen for every model.

The admin functionality comes from the first party app `django.contrib.admin`.

```python
# my_project/urls.py
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path("admin/", admin.site.urls),
]
```

This makes the admin available at `/admin/`.

To log in, we usually create an admin user with:

```bash
python manage.py createsuperuser
```

### Making Models Available in the Admin

A data model does not automatically appear in the admin just because it exists. We register the model in the app's `admin.py` file.

```python
# admin.py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

After this, the `Post` model appears in the admin interface, and staff users with the right permissions can manage `Post` records.

### Controlling How Models Appear in the Admin

For small data models like `Post`, `admin.site.register(Post)` is enough.

For larger models, we usually define a `ModelAdmin` class to control how the model appears in the admin.

```python
from django.contrib import admin
from .models import Post


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ["title", "created_at"]
    search_fields = ["title", "body"]
    list_filter = ["created_at"]
```

Here:

- `list_display` controls which columns appear on the model's list page
- `search_fields` adds a search box for selected fields
- `list_filter` adds sidebar filters

## Middleware

Middleware is a stack of processing layers around Django's request/response cycle. An incoming request passes through middleware before it reaches the view, and the response passes back through middleware before it is sent to the client.

The rough flow is:

```text
Request from browser
    |
    v
Middleware
    |
    v
URL routing
    |
    v
View
    |
    v
Middleware
    |
    v
Response to browser
```

Common middleware responsibilities:

- Security headers
- Sessions
- Authentication
- CSRF protection
- Messages
- Logging
- Request/response modification

Middleware is configured in `settings.py` with the `MIDDLEWARE` list.

```python
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
]
```

Order matters. Request middleware runs from top to bottom. Response middleware runs back out in the reverse direction.

That means `SessionMiddleware` runs before `AuthenticationMiddleware` on the incoming request, which matters because authentication often depends on session data.

### Common Built-In Middleware

| Middleware | Purpose |
| --- | --- |
| `SecurityMiddleware` | Adds security-related request/response behavior |
| `SessionMiddleware` | Adds session support through `request.session` |
| `CsrfViewMiddleware` | Protects POST forms against CSRF attacks |
| `AuthenticationMiddleware` | Adds the logged-in user as `request.user` |
| `MessageMiddleware` | Enables temporary messages between requests |

Middleware is one reason views can access objects like `request.session` and `request.user` without manually creating them in every view.

### Custom Middleware

You can also write custom middleware when behavior should apply across many views.

```python
# catalog/middleware.py
class SimpleLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        print(f"Request path: {request.path}")

        response = self.get_response(request)

        print(f"Response status: {response.status_code}")
        return response
```

The middleware class can be placed in a file such as `catalog/middleware.py`, inside one of your Django apps. To make Django use it, include its dotted import path in the `MIDDLEWARE` list in `settings.py`.

```python
# settings.py
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "catalog.middleware.SimpleLoggingMiddleware",
]
```

When the server starts, Django reads the `MIDDLEWARE` list, imports each middleware class, and creates middleware objects from middleware classes. For this example, Django conceptually does something like this:

```python
middleware = SimpleLoggingMiddleware(get_response)
```

That calls `__init__`, which stores `self.get_response` as an instance attribute on the middleware object.

Middleware objects are reusable objects that Django calls when requests pass through the project. If this middleware is listed in `MIDDLEWARE`, it runs for every incoming request handled by Django, no matter which URL or view the request eventually matches.

When a request comes in, Django calls the middleware object like a function:

```python
response = middleware(request)
```

Because the object is callable, Python runs its `__call__` method. In this example, the middleware prints the request path before the view runs, calls `self.get_response(request)` to let the request continue on to next layer in the request/response cycle, waits for a response to come back, then prints the response status code before returning the response to Django.

## Authentication and Authorization

Django includes a built-in authentication and authorization system for working with users, login state, groups, and permissions.

The core functionality comes from the first party app `django.contrib.auth`, but authentication usually works together with sessions, middleware, views, forms, templates, and the admin.

Authentication answers: **Who is this user?**

Authorization answers: **What is this user allowed to do?**

### Combination of 3 Apps

```python
# settings.py
INSTALLED_APPS = [
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
]
```

These apps work together:

| App | Purpose |
| --- | --- |
| `django.contrib.auth` | Users, groups, permissions, password hashing, login/logout helpers |
| `django.contrib.contenttypes` | Tracks installed model types so permissions can be connected to models |
| `django.contrib.sessions` | Stores per-browser session data, including login state |

After these apps are installed, running migrations creates the database tables Django needs for users, groups, permissions, content types, and sessions.

```bash
python manage.py migrate
```

Common tables include:

| Table | Purpose |
| --- | --- |
| `auth_user` | Stores user accounts when using Django's default user model |
| `auth_group` | Stores groups of users |
| `auth_permission` | Stores permissions, such as add/change/delete/view permissions for models |
| `django_content_type` | Stores references to installed app models |
| `django_session` | Stores session data for browser/client sessions |

With Django's default user model, user accounts are stored in the `auth_user` table. Passwords are not stored as plain text; Django stores password hashes.

### Users, Groups, and Permissions

The default user model comes from:

```python
from django.contrib.auth.models import User
```

`User` is a Django model class from Django's built-in authentication app. Because it is a model, it maps to a database table. With Django's default user model, that table is usually called `auth_user`.

A user represents an account that can log in. Each user account is stored as a row in the `auth_user` table.

A group is a collection of users. Permissions can be assigned to a group so every user in that group receives the same access.

Groups are stored in their own table, usually `auth_group`. Users and groups are related through a many-to-many join table, usually `auth_user_groups`. That join table stores which users belong to which groups.

Django creates standard permission names for each model. These default permission names are generated by Django from the app name and model name. For a `catalog` app that has `Product` model, Django would create permission names like this:

```text
catalog.add_product
catalog.change_product
catalog.delete_product
catalog.view_product
```

A permission represents an allowed action. For example, `catalog.change_product` means the user is allowed to change `Product` records in the `catalog` app.

The format is:

```text
app_label.permission_codename
```

You usually do not manually invent these default names. Django creates the permission rows in `auth_permission`, and then you assign those existing permissions to users or groups.

A permission can be assigned directly to a user through `auth_user_user_permissions`, or assigned to a group through `auth_group_permissions`. A user's final permissions come from both direct user permissions and permissions from their groups.

Example of assigning one model permission directly to a user:

```python
from django.contrib.auth.models import Permission, User
from django.contrib.contenttypes.models import ContentType
from catalog.models import Product

user = User.objects.get(username="alex")
content_type = ContentType.objects.get_for_model(Product)

permission = Permission.objects.get(
    codename="change_product",
    content_type=content_type,
)

user.user_permissions.add(permission)
```

This gives `alex` the `catalog.change_product` permission directly.

### Creating Users

For an admin user, we usually create a superuser from the CLI:

```bash
python manage.py createsuperuser
```

That creates a user with admin-level permissions. The command creates a row in the default user table, usually `auth_user`.

That row stores values such as:

```text
username
email
password hash
is_staff = True
is_superuser = True
is_active = True
```

The password is not stored as plain text. Django stores a password hash. The account is created in the database by the command, but the login happens later through a web interface such as `/admin/` or a custom login page.

Normal users can be created through the admin, through a signup view, or through code:

```python
from django.contrib.auth.models import User

User.objects.create_user(
    username="alex",
    email="alex@example.com",
    password="strong-password",
)
```

`create_user()` creates a row in the user table, usually `auth_user`. It is important because it hashes the password correctly before saving the user. Django does not store the raw password text.

### Login State

Django remembers logged-in users through sessions.

A session is server-side data connected to a browser by a session cookie. The browser does not store the whole session. The browser stores a session key(`session_id`) in a cookie, and Django uses that key to find the matching session data on the server.

With Django's database-backed session engine, session records are stored in the `django_session` table.

The rough idea is:

```text
Browser stores sessionid cookie
    |
    v
Django uses that session key
    |
    v
Django loads matching data from django_session
```

This is how login state can survive across multiple requests. HTTP itself is stateless, so without a session each request would arrive without memory of previous requests.

### Authentication Middleware

Sessions and authentication are added to the request through middleware.

```python
# settings.py
MIDDLEWARE = [
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
]
```

`SessionMiddleware` reads the session cookie from the browser, loads the matching session data, and attaches it to the request as `request.session`.

`AuthenticationMiddleware` uses that session data to figure out the current user and attaches that user to the request as `request.user`.

The request flow looks like this:

```text
Browser sends request with session cookie
    |
    v
SessionMiddleware loads request.session
    |
    v
AuthenticationMiddleware adds request.user
    |
    v
View can check request.user
```

If the user is logged in, `request.user` represents that user.

If the user is not logged in, Django uses an anonymous user object instead of `None`.

```python
if request.user.is_authenticated:
    ...
```

This is why views can ask `request.user.is_authenticated` without first checking whether `request.user` exists.

### Logging In

Logging in means Django has verified the user's credentials and stored the user's ID in the session.

At a high level:

```text
User submits username and password
    |
    v
Django checks the credentials
    |
    v
Django stores the user ID in the session
    |
    v
Later requests can rebuild request.user from the session
```

Django provides helper functions for this:

```python
from django.contrib.auth import authenticate, login
from django.shortcuts import redirect, render


def login_view(request):
    if request.method == "POST":
        username = request.POST["username"]
        password = request.POST["password"]
        user = authenticate(request, username=username, password=password)

        if user is not None:
            login(request, user)
            return redirect("product_list")

    return render(request, "registration/login.html")
```

`authenticate()` checks the submitted credentials.

`login()` stores the authenticated user's ID in the session.

After `login(request, user)` runs, Django can identify that same browser on later requests through the session cookie.

### Logging Out

Logging out removes the authenticated user from the current session.

```python
from django.contrib.auth import logout
from django.shortcuts import redirect


def logout_view(request):
    logout(request)
    return redirect("login")
```

After logout, later requests from that browser will no longer have an authenticated `request.user`.

### Built-In Login and Logout Views

Django provides built-in login and logout view classes, but your project still needs to add URLs for them.

```python
# urls.py
from django.contrib.auth import views as auth_views
from django.urls import path

urlpatterns = [
    path("login/", auth_views.LoginView.as_view(), name="login"),
    path("logout/", auth_views.LogoutView.as_view(), name="logout"),
]
```

By default, `LoginView` expects a template at `registration/login.html`.

```python
# settings.py
LOGIN_URL = "/login/"
LOGIN_REDIRECT_URL = "/catalog/products/"
LOGOUT_REDIRECT_URL = "/login/"
```

`LOGIN_URL` tells Django where to send unauthenticated users when a protected view requires login.

`LOGIN_REDIRECT_URL` tells Django where to send a user after a successful login when no more specific destination is provided.

`LOGOUT_REDIRECT_URL` tells Django where to send a user after logout.

### Authentication Forms

Django also provides form classes for common authentication workflows.

```python
from django.contrib.auth.forms import AuthenticationForm, UserCreationForm
```

`AuthenticationForm` validates login credentials.

`UserCreationForm` validates user signup data and handles password confirmation.

The built-in `LoginView` uses `AuthenticationForm` by default, so a common login template can render the form the same way as other Django forms:

```html
<!-- registration/login.html -->
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Log in</button>
</form>
```

This is the same form pattern discussed earlier:

- The view creates or binds a form
- The form validates submitted `request.POST` data
- The template renders fields, errors, and the CSRF token

### Using Authentication in Templates

When Django's authentication middleware and template context processors are configured, templates can use `user`.

```python
# settings.py
TEMPLATES = [
    {
        "OPTIONS": {
            "context_processors": [
                "django.contrib.auth.context_processors.auth",
            ],
        },
    },
]
```

The auth context processor makes `user` and `perms` available in templates rendered with `render()`.

```html
{% if user.is_authenticated %}
  <p>Signed in as {{ user.username }}</p>
  <form method="post" action="{% url 'logout' %}">
    {% csrf_token %}
    <button type="submit">Log out</button>
  </form>
{% else %}
  <a href="{% url 'login' %}">Log in</a>
{% endif %}
```

The template does not decide whether the user is truly authenticated. It displays different HTML based on the `user` object that Django added to the request/template context.

### Protecting Views

For your own views, you choose which pages require login.

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render
from .models import Product


@login_required
def product_list(request):
    products = Product.objects.all()
    return render(
        request,
        "catalog/product_list.html",
        {"products": products},
    )
```

The URL still routes to `product_list`, but Django calls a wrapped version of the view. The wrapper checks whether `request.user.is_authenticated` is true. If the user is logged in, the real view runs. If not, Django redirects the browser to the login page.

For permission checks, a view can require a specific permission:

```python
from django.contrib.auth.decorators import permission_required


@permission_required("catalog.add_product")
def create_product(request):
    ...
```

You can also check permissions directly:

```python
if request.user.has_perm("catalog.add_product"):
    ...
```

For class-based views, Django provides mixins:

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView


class ProductListPageView(LoginRequiredMixin, TemplateView):
    template_name = "catalog/product_list.html"
```

The idea is the same as the function decorator: the view is protected before its main handler logic runs.

### Admin and Authentication

The admin uses Django's same authentication system.

When you run:

```bash
python manage.py createsuperuser
```

the created user is stored in the same user table used by the rest of the project.

To access `/admin/`, a user usually needs:

- `is_staff=True`
- `is_active=True`
- The right model permissions for the admin actions they are trying to perform

A superuser bypasses normal permission checks and has access to everything in the admin.

### How the Pieces Fit Together

Authentication is not only one file or one feature. It is an interaction between several Django layers:

```text
models
    User, Group, Permission, Session tables

urls
    login, logout, signup, product list, protected routes

views
    authenticate users, call login/logout, protect pages

forms
    validate login and signup input

templates
    render login forms and show different navigation for authenticated users

admin
    uses the same user and permission system for staff access

middleware
    adds request.session and request.user to each request
```

The most important flow is:

```text
User logs in successfully
    |
    v
Django stores the user ID in the session
    |
    v
Browser keeps the session cookie
    |
    v
Next request includes the session cookie
    |
    v
SessionMiddleware loads request.session
    |
    v
AuthenticationMiddleware sets request.user
    |
    v
Views and templates can use request.user / user
```

That is the core relationship between authentication and sessions in Django: authentication identifies the user, while sessions remember that identity between requests.

## Tests

Tests are especially useful in Django because a typical feature is spread across several parts:

```text
model
    stores and queries data

url
    routes the request

view
    coordinates the request/response behavior
    
form
    validates submitted input

template
    renders the response shown to the user
```

### Where Tests Live

When you create an app with:

```bash
python manage.py startapp catalog
```

Django creates a `tests.py` file inside that app:

```text
catalog/
├── models.py
├── views.py
├── urls.py
└── tests.py
```

For small apps, `tests.py` is enough.

For larger apps, it is common to replace `tests.py` with a `tests/` package:

```text
catalog/
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_forms.py
│   └── test_views.py
```

Django's test runner discovers test files and test methods automatically when they follow common naming patterns.

Test methods usually start with `test_`.

### Running Tests

The standard Django test command is:

```bash
python manage.py test
```

This runs all tests in the project.

You can also run tests for one app:

```bash
python manage.py test catalog
```

Or one specific test class or method:

```bash
# In case of catalog/tests.py containing a test class ProductModulTests
python manage.py test catalog.tests.ProductModelTests
python manage.py test catalog.tests.ProductModelTests.test_product_string
```

Useful options:

```bash
python manage.py test --verbosity 2
python manage.py test --keepdb
python manage.py test --parallel
```

`--verbosity 2` prints more details.

`--keepdb` keeps the test database between runs, which can make repeated test runs faster.

`--parallel` runs tests in parallel when possible.

### Test Database

When tests use the database, Django does not use your normal development database directly.

Django creates a separate test database, runs migrations on it, executes the tests, and then destroys that test database when the run is finished.

This means test data should be created inside the test itself or inside test setup methods. Tests should not depend on records that happen to exist in your local development database.

### `TestCase` Class

Most model, form, and view tests inherit from `django.test.TestCase`.

```python
from django.test import TestCase


class ProductModelTests(TestCase):
    def test_example(self):
        self.assertEqual(1 + 1, 2)
```

`TestCase` builds on Python's standard `unittest` style and is a subclass of `SimpleTestCase`. `TestCase` is used for tests that need database access, whereas `SimpleTestCase` is used for tests that do not need the database.

When Django runs the test, it creates an instance of `ProductModelTests` and calls the test method on that instance. Each test method is run as an independent test. If a test method raises an assertion failure, Django reports that test method as failed.

Assertions like `self.assertEqual()` are instance methods inherited through Django's `TestCase` class, which ultimately builds on Python's `unittest.TestCase`.

Common assertions include:

| Assertion | Purpose |
| --- | --- |
| `self.assertEqual(a, b)` | Check that two values are equal |
| `self.assertIn(a, b)` | Check that `a` exists inside `b` |
| `self.assertTrue(value)` | Check that a value is truthy |
| `self.assertFalse(value)` | Check that a value is falsy |
| `self.assertContains(response, text)` | Check that a response contains text |
| `self.assertRedirects(response, url)` | Check that a response redirects to a URL |

### Testing Models

Model tests usually check model methods, default values, string representations, relationships, and query behavior.

Before running database tests, the model should already have migration files created with `makemigrations`.

When `python manage.py test` runs, Django creates a separate **test database** and applies those existing migrations automatically. Calls like `Product.objects.create(...)` insert rows into the test database, not the normal development database.

After the test is over the test database is deleted.

Example model:

```python
from django.db import models


class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    in_stock = models.BooleanField(default=True)

    def __str__(self):
        return self.name # self.name = name is set using the __init__ from models.Model
```

Example test:

```python
from decimal import Decimal

from django.test import TestCase
from .models import Product # Importing the model class


class ProductModelTests(TestCase):
    def test_product_string_is_name(self):
        product = Product.objects.create(
            name="Keyboard",
            price=Decimal("49.99"),
        )

        self.assertEqual(str(product), "Keyboard")

    def test_product_is_in_stock_by_default(self):
        product = Product.objects.create(
            name="Mouse",
            price=Decimal("24.99"),
        )

        self.assertTrue(product.in_stock)
```

These tests create records in the test database, then check that the data model behaves correctly.

### Testing Forms

Form tests check validation rules.

Example form:

```python
from django import forms
from .models import Product


class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = ["name", "price"]
```

Example test:

```python
from django.test import TestCase
from .forms import ProductForm


class ProductFormTests(TestCase):
    def test_form_accepts_valid_data(self):
        form = ProductForm(data={
            "name": "Keyboard",
            "price": "49.99",
        })

        self.assertTrue(form.is_valid())

    def test_form_rejects_missing_name(self):
        form = ProductForm(data={
            "name": "",
            "price": "49.99",
        })

        self.assertFalse(form.is_valid()) 
        self.assertIn("name", form.errors) # since form.errors is a dict like object and in is good way to find if "name" is in it
```

The form receives raw submitted data, just like it would receive from `request.POST` in a real view. The test then checks whether `form.is_valid()` returns the expected result.

### Testing Views

View tests usually check status codes, templates, redirects, response content, and database changes.

Django gives each `TestCase` a test client at `self.client`.

The test client acts like a small browser inside the test. It can make requests to your Django app without starting a real development server.

Common request methods include:

```python
self.client.get(path)          # request a page or resource
self.client.post(path, data)   # submit form data or create something
self.client.put(path, data)    # replace a resource, often in APIs
self.client.patch(path, data)  # partially update a resource, often in APIs
self.client.delete(path)       # delete a resource, often in APIs
self.client.head(path)         # request headers without the response body
self.client.options(path)      # ask which HTTP methods/options a URL supports
```

The test client also has authentication helpers:

```python
self.client.login(username="alex", password="strong-password")  # log in by checking credentials
self.client.logout()                                           # clear the test client's login session
self.client.force_login(user)                                  # log in a user directly without a password check
```

`self.client.login()` logs the test client in by creating the same kind of session state Django would create during a real login.

```python
response = self.client.get("/catalog/products/")
```

Example view:

```python
from django.shortcuts import render
from .models import Product


def product_list(request):
    products = Product.objects.all()
    return render(
        request,
        "catalog/product_list.html",
        {"products": products},
    )
```

Example test:

```python
from decimal import Decimal

from django.test import TestCase
from django.urls import reverse
from .models import Product


class ProductViewTests(TestCase):
    def test_product_list_page_loads(self):
        response = self.client.get(reverse("product_list")) # reverse() searches for the URL path by this url_name provided

        self.assertEqual(response.status_code, 200)

    def test_product_list_shows_products(self):
        Product.objects.create(
            name="Keyboard",
            price=Decimal("49.99"),
        )

        response = self.client.get(reverse("product_list"))

        self.assertContains(response, "Keyboard") # looks up keyboard in the HttpResponse 
```

URL names are important because `reverse("product_list")` builds the URL from the URL route name. This is better than hard-coding `"/catalog/products/"` because the test will still point to the correct route if the URL path changes later.

### Testing POST Requests

The test client can also submit data with `post()`.

For a create view, a test often checks two things:

- The response redirects after a successful POST
- The database record was actually created

```python
from django.test import TestCase
from django.urls import reverse
from .models import Product


class ProductCreateViewTests(TestCase):
    def test_create_product(self):
        response = self.client.post(
            reverse("product_create"), # Assuming "product_create" leads to a view taht creates a Product row
            {
                "name": "Keyboard",
                "price": "49.99",
            },
        )

        self.assertEqual(response.status_code, 302) # Create object working correctly usually sends a redirect() response which is 302
        self.assertTrue(Product.objects.filter(name="Keyboard").exists())
        # checks that a Product row with this name was created
```

A `302` response means the view redirected the browser, which is common after a successful form submission.

### Testing Authentication Behavior

If a view requires login, the test should check both paths:

- Anonymous users should be redirected
- Logged-in users should be allowed through

Example:

```python
from django.contrib.auth.models import User
from django.test import TestCase
from django.urls import reverse


class ProtectedViewTests(TestCase):
    def test_anonymous_user_is_redirected(self):
        response = self.client.get(reverse("product_list")) # assumes product_list is protected with @login_required or LoginRequiredMixin

        self.assertEqual(response.status_code, 302)

    def test_logged_in_user_can_access_page(self):
        user = User.objects.create_user(
            username="alex",
            password="strong-password",
        )
        self.client.login(
            username="alex",
            password="strong-password",
        )

        response = self.client.get(reverse("product_list"))

        self.assertEqual(response.status_code, 200) # we logged in, so the login-required product_list view allows access
```

### Reading Test Logs, Failures, And Errors

When a Django test fails, the test output is usually enough to locate the problem if you read it from top to bottom.

Example:

```text
FAIL: test_available_endpoint_returns_only_in_stock_books (catalog.tests.test_api.BookApiTests.test_available_endpoint_returns_only_in_stock_books)
Traceback (most recent call last):
  File ".../catalog/tests/test_api.py", line 41, in test_available_endpoint_returns_only_in_stock_books
    self.assertIn("Has Stock", titles)
AssertionError: 'Has Stock' not found in ['Sold Out']
```

The first line tells you the result type and the test that failed.

```text
FAIL: test_available_endpoint_returns_only_in_stock_books (...)
```

`FAIL` means the test ran, but an assertion failed.

`test_available_endpoint_returns_only_in_stock_books` is the specific test method that failed.

The part in parentheses is the full Python path:

```text
catalog.tests.test_api.BookApiTests.test_available_endpoint_returns_only_in_stock_books
```

That path follows:

```text
fully_qualified_module.class.method
```

Then Django prints the traceback:

```text
Traceback (most recent call last):
```

This means Python is showing the call stack. "Most recent call last" means the deepest/latest call is shown near the bottom of the traceback.

Each `File ...` block is one stack frame in a traceback. A stack frame usually appears as two lines:

- The `File ...` line tells you the file, the line number where the call happened, and the function or method that was running.
- The next line prints the actual line of code from that file, indented underneath the `File ...` line.

```text
File ".../catalog/tests/test_api.py", line 41, in test_available_endpoint_returns_only_in_stock_books
    self.assertIn("Has Stock", titles)
```

In this stack frame, Django is saying in  `catalog/tests/test_api.py` at line `41` which falls inside the function `test_available_endpoint_returns_only_in_stock_books`, and the source line it ran was `self.assertIn("Has Stock", titles)`.

In a short assertion traceback, that indented source line may be the exact test assertion that failed. In a longer traceback, there may be many `File ...` blocks because stacked function calls

The final line of the traceback usually shows the actual error or failed assertion:

```text
AssertionError: 'Has Stock' not found in ['Sold Out']
```

In this example, the test expected `"Has Stock"` to be inside `titles`, but `titles` was actually `["Sold Out"]`.

#### Exception vs Error vs Failure

| Term | Meaning | Example |
| --- | --- | --- |
| Exception | The Python object/event raised when something goes wrong. | `raise ValueError("bad value")` |
| Error | The test crashed because of an unexpected exception before producing a normal assertion result. | A `NameError` or `NoReverseMatch` happens during the test |
| Failure | The test ran and reached an assertion, but the assertion result was wrong. | `self.assertEqual(actual, expected)` fails |

An **exception** is the thing Python raises.

An **error** is how Django/unittest classifies an unexpected exception during a test.

A **failure** is how Django/unittest classifies a failed assertion.

Common Django exceptions that usually become test errors:

| Exception | What It Usually Means |
| --- | --- |
| `django.urls.exceptions.NoReverseMatch` | Django could not find a URL pattern by that name or with those arguments |
| `catalog.models.Book.DoesNotExist` | A query expected one matching `Book`, but no matching row existed |
| `django.db.IntegrityError` | A database constraint failed, such as a missing required relationship or duplicate unique value |
| `django.core.exceptions.ValidationError` | A model, form, field, or validator rejected invalid data |
| `django.template.exceptions.TemplateDoesNotExist` | Django could not find the template file named by the view |

Common errors:

| Error Case | What It Usually Means |
| --- | --- |
| `ERROR` with `NameError` | The test crashed because a name was missing |
| `ERROR` with `AttributeError` | The test crashed because code used an attribute or method that does not exist |
| `ERROR` with `TypeError` | The test crashed because code called something with the wrong type or arguments |
| `ERROR` with `NoReverseMatch` | The test crashed because `reverse()` or a template URL tag could not find a route |
| `ERROR` with `IntegrityError` | The test crashed because a database constraint was violated |

Common failures:

| Failure Case | What It Usually Means |
| --- | --- |
| `FAIL` from `self.assertEqual(actual, expected)` | The actual value did not match the expected value |
| `FAIL` from `self.assertIn(item, collection)` | The expected item was missing from a list, string, queryset result, or other collection |
| `FAIL` from `self.assertTrue(value)` | The value was falsey, such as `False`, `None`, `0`, `""`, or an empty list |
| `FAIL` from `self.assertFalse(value)` | The value was truthy when the test expected it to be falsey |
| `FAIL` from `self.assertContains(response, text)` | The HTTP response did not contain the expected text |
| `FAIL` from `self.assertRedirects(response, url)` | The response did not redirect to the expected URL |

### Pytest In Django Projects

Some Django projects use `pytest` instead of Django's built-in `unittest` style runner.

In those projects, tests are usually run with:

```bash
pytest
```

The project usually installs `pytest-django` and configures the settings module in a file like `pytest.ini`:

```ini
[pytest]
DJANGO_SETTINGS_MODULE = my_project.settings
python_files = tests.py test_*.py *_tests.py
```

The core testing ideas are the same: create test data, make requests, check responses, and verify database changes. The syntax is different, but the goal is still to protect the behavior of your Django project.
