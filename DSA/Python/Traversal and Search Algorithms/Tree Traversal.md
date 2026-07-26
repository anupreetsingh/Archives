# Tree Traversal

## BFS Traversal: Level Order (Breadth-First Search)

Unlike DFS (which goes deep first), BFS explores all nodes at the current depth before moving to nodes at the next level. It's typically implemented using a queue.

```python
from collections import deque

# Definition for a binary tree node
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def bfs_level_order(root):
    """
    Performs level-order traversal (Breadth-First Search) of a binary tree.

    Returns:
        List of lists, where each sublist contains the node values at that level.

    Example:
        Input tree:
            1
           / \
          2   3
         / \
        4   5

        Output:
        [
            [1],
            [2, 3],
            [4, 5]
        ]
    """
    # Edge case of empty Tree
    if not root:
        return []

    result = []                 # Final list of levels
    queue = deque([root])      # Queue for BFS

    while queue: # Goes across all nodes in the tree
        level_size = len(queue)       # Number of nodes at current level
        current_level = []            # List to hold values of this level

        for _ in range(level_size):# Goes across all nodes in current level
            node = queue.popleft()    # Dequeue current node
            current_level.append(node.val)

            # Add child nodes to queue (if they exist)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        # After processing the level, append it to result
        result.append(current_level)

    return result
```

## DFS Traversal Orders

Tree traversal means visiting every node in a tree exactly once in some specific order. There are three common types of **Depth-First Traversals (DFS)**:

1. **Inorder Traversal (Left → Node → Right)**:
   - For Binary Search Trees (BST), this gives nodes in **sorted order**.
2. **Preorder Traversal (Node → Left → Right)**:
   - Useful for copying the tree or serializing it.
3. **Postorder Traversal (Left → Right → Node)**:
   - Commonly used in deleting/freeing nodes, or evaluating expressions.

Each of these can be implemented in two ways:

- **Recursive**: Clean and natural due to the recursive nature of trees.
- **Iterative**: Uses an explicit stack instead of the call stack and avoids recursion limits.

Below are Python implementations for all three traversal types using both techniques.

```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

### 1. Inorder (Left → Node → Right)

```python
def inorder_recursive(node):
    """Recursive Inorder Traversal"""
    if not node:
        return
    inorder_recursive(node.left)     # Visit left subtree
    print(node.val)                  # Process current node
    inorder_recursive(node.right)    # Visit right subtree


def inorder_iterative(root):
    """Iterative Inorder Traversal"""
    result = []
    stack = []
    curr = root

    while curr or stack:
        while curr:
            stack.append(curr)
            curr = curr.left

        curr = stack.pop()
        result.append(curr.val)      # Process current node
        curr = curr.right

    return result
```

### 2. Preorder (Node → Left → Right)

```python
def preorder_recursive(node):
    """Recursive Preorder Traversal"""
    if not node:
        return
    print(node.val)                 # Process current node
    preorder_recursive(node.left)   # Visit left subtree
    preorder_recursive(node.right)  # Visit right subtree


def preorder_iterative(root):
    """Iterative Preorder Traversal"""
    if not root:
        return []

    result = []
    stack = [root]

    while stack:
        node = stack.pop()
        result.append(node.val)     # Process current node

        # Push left second so it is processed first from the stack(LIFO)
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)

    return result
```

### 3. Postorder (Left → Right → Node)

```python
def postorder_recursive(node):
    """Recursive Postorder Traversal"""
    if not node:
        return
    postorder_recursive(node.left)   # Visit left subtree
    postorder_recursive(node.right)  # Visit right subtree
    print(node.val)                  # Process current node


def postorder_iterative(root):
    """Iterative Postorder Traversal"""
    result = []
    stack = []
    curr = root
    last_visited = None

    while curr or stack:
        while curr:
            stack.append(curr)
            curr = curr.left

        peek = stack[-1]

        # Visit right subtree before processing current node
        if peek.right and last_visited != peek.right:
            curr = peek.right
        else:
            result.append(peek.val)  # Process current node
            last_visited = stack.pop()

    return result
```

## Predecessor and Successor

Whatever traversal order you use, the tree gets converted into a sequence of visited nodes.

```text
A, B, C, D, E
```

For any node in that sequence:

- **Predecessor**: the node visited immediately before it.
- **Successor**: the node visited immediately after it.

The first node has no predecessor. The last node has no successor.

Example tree:

```text
        A
       / \
      B   C
     / \
    D   E
```

Different traversal orders create different predecessor/successor relationships:

```text
Inorder:   D, B, E, A, C
Preorder:  A, B, D, E, C
Postorder: D, E, B, C, A
```

| Traversal | Node | Predecessor | Successor |
| --- | --- | --- | --- |
| Inorder | `B` | `D` | `E` |
| Inorder | `A` | `E` | `C` |
| Inorder | `D` | none | `B` |
| Preorder | `A` | none | `B` |
| Preorder | `B` | `A` | `D` |
| Preorder | `C` | `E` | none |
| Postorder | `D` | none | `E` |
| Postorder | `B` | `E` | `C` |
| Postorder | `A` | `C` | none |

### Inorder: Left → Node → Right

For inorder traversal, the current node is visited between its left subtree and right subtree.

- **Predecessor**:
  - If the node has a left subtree, the predecessor is the rightmost node of the left subtree.
  - If the node does not have a left subtree, move up until you find an ancestor where the node is in the ancestor's right subtree.
  - If no such ancestor exists, the node has no inorder predecessor.

- **Successor**:
  - If the node has a right subtree, the successor is the leftmost node of the right subtree.
  - If the node does not have a right subtree, move up until you find an ancestor where the node is in the ancestor's left subtree.
  - If no such ancestor exists, the node has no inorder successor.

### Preorder: Node → Left → Right

For preorder traversal, the current node is visited before its children.

- **Predecessor**:
  - If the node is the root, it has no predecessor.
  - If the node is a left child, the predecessor is usually its parent.
  - If the node is a right child and there is no left sibling subtree, the predecessor is its parent.
  - If the node is a right child and there is a left sibling subtree, the predecessor is the last node visited in that left sibling subtree.

- **Successor**:
  - If the node has a left child, the successor is the left child.
  - Else if the node has a right child, the successor is the right child.
  - Else, move up until you find an ancestor with an unvisited right subtree. The successor is the root of that unvisited right subtree.

The last node in a preorder subtree is found by going right when possible, otherwise left, until reaching a leaf.

### Postorder: Left → Right → Node

For postorder traversal, the current node is visited after its children.

- **Predecessor**:
  - If the node has a right child, the predecessor is the right child.
  - Else if the node has a left child, the predecessor is the left child.
  - If the node is a leaf, it is easiest to reason from the full postorder sequence unless you have parent pointers.

- **Successor**:
  - If the node is the root, it has no successor.
  - If the node is a right child, the successor is its parent.
  - If the node is a left child and the parent has no right child, the successor is its parent.
  - If the node is a left child and the parent has a right subtree, the successor is the first node visited in that right subtree.

The first node in a postorder subtree is found by going left when possible, otherwise right, until reaching a leaf.

Preorder and postorder predecessor/successor are about **traversal order**, not sorted value. The "closest smaller" and "closest larger" meaning comes specifically from **inorder traversal of a BST**.

### BST Context

Because the **inorder** traversal of a BST gives a sorted sequence. People generally talk about **inorder predecessor** and **inorder successor**.

- **Inorder predecessor**: the closest smaller value.
- **Inorder successor**: the closest larger value.
