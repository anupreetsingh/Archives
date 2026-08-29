ruff check . scans your Python code for linting problems—mostly things that are suspicious, inconsistent, unused, or likely mistakes.

For example, it can catch:

import os   # imported but never used
x = 10
if x == None:   # should usually use "is None"
    ...
def add(a, b):
    unused = 5   # assigned but never used
    return a + b

Ruff for python and ESLint for Javascript.
