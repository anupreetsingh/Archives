# 🧱 Default Stack & Queue in Python

Python does not have built-in "Stack" or "Queue" types like in Java or C++, but we can use built-in modules or data structures to implement them efficiently.

---

## 🥞 Stack (LIFO: Last In, First Out)

✅ Recommended: Use list (`append` + `pop`)

```python
stack = []

# Push onto stack
stack.append(10)
stack.append(20)
stack.append(30)

# Pop from stack
top = stack.pop()  # 30

# Peek at top element
peek = stack[-1]   # 20

# Check if empty
is_empty = len(stack) == 0

# Result
print(stack)  # [10, 20]
```

---

## 🚦 Queue (FIFO: First In, First Out)

✅ Recommended: Use `collections.deque` for efficient O(1) operations

```python
from collections import deque

queue = deque()

# Enqueue (add to right end)
queue.append('A')
queue.append('B')
queue.append('C')

# Dequeue (remove from left end)
front = queue.popleft()  # 'A' # This is O(1) time, if you instead use a normal list and do .pop(0) to remove the leftmost element it shifts all the elements to the left which is inefficient

# Peek at front element
peek_front = queue[0]    # 'B'

# Check if empty
is_empty = len(queue) == 0

# Result
print(queue)  # deque(['B', 'C'])
```

---

## 🧵 Advanced Queue: queue.Queue (Thread-safe)

```python
from queue import Queue

q = Queue()

# Enqueue
q.put(1)
q.put(2)

# Dequeue
first = q.get()  # 1

# Check size
size = q.qsize()  # 1
```

---

⛔ **Note:** `list.pop(0)` can simulate a queue, but it's O(n) — not efficient.
