# Linked List Tips

## Dummy

```python
dummy = Node(0, head)
```

- Use a **dummy** node pointing to head when the head might change.
- Return `dummy.next` at the end.

## Traversing Linked List

```python
while curr:
    # Reaches and processes the last node.
    curr = curr.next
```

```python
while curr.next:
    # Stops at the last node, but does not process it.
    curr = curr.next
```

## Nested While Loops

- If you use a `while` loop to iterate through a data structure and then use a nested `while` loop, the nested loop needs its own out of bounds check.

List example:

```python
i = 0

while i < len(nums):
    # Skip duplicate values.
    while i + 1 < len(nums) and nums[i] == nums[i + 1]:
        i += 1

    # Move to the next unique value.
    i += 1
```

Linked list example:

```python
while curr:
    # Skip duplicate nodes.
    while curr.next and curr.val == curr.next.val:
        curr.next = curr.next.next

    # Move to the next unique node.
    curr = curr.next
return dummy.next
```
