# Binary Tree Traversals in Python

Tree traversal means visiting every node in a tree exactly once in some specific order. There are three common types of **Depth-First Traversals (DFS)**:

1. **Inorder Traversal (Left → Node → Right)**:
   - For Binary Search Trees (BST), this gives nodes in **sorted order**.
2. **Preorder Traversal (Node → Left → Right)**:
   - Useful for copying the tree or serializing it.
3. **Postorder Traversal (Left → Right → Node)**:
   - Commonly used in deleting/freeing nodes, or evaluating expressions.

Each of these can be implemented in two ways:

- **Recursive**: Clean and natural due to the recursive nature of trees.
- **Iterative**: Uses explicit Stacks and is often more complex but avoids recursion limits.

Below are Python implementations for all three traversal types using both techniques.

```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

---

## 1. Inorder (Left → Node → Right)

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
    stack = []
    curr = root
    while curr or stack:
        # Reach the leftmost node of the current subtree
        while curr:
            stack.append(curr)
            curr = curr.left
        # Node with no left child
        curr = stack.pop()
        print(curr.val)              # Process the node
        curr = curr.right            # Visit right subtree
```

## 2. Preorder (Node → Left → Right)

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
        return
    stack = [root]
    while stack:
        node = stack.pop()
        print(node.val)             # Process current node
        # Push right first so left is processed first (LIFO order)
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
```

## 3. Postorder (Left → Right → Node)

```python
def postorder_recursive(node):
    """Recursive Postorder Traversal"""
    if not node:
        return
    postorder_recursive(node.left)   # Visit left subtree
    postorder_recursive(node.right)  # Visit right subtree
    print(node.val)                  # Process current node


def postorder_iterative(root):
    """Iterative Postorder Traversal using two stacks"""
    if not root:
        return
    stack1 = [root]
    stack2 = []  # Will collect nodes in reverse postorder

    while stack1:
        node = stack1.pop()
        stack2.append(node)
        # Reverse the order: push left before right
        if node.left:
            stack1.append(node.left)
        if node.right:
            stack1.append(node.right)

    while stack2:
        print(stack2.pop().val)     # True postorder traversal
```
