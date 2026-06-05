# BFS Traversal: Level Order (Breadth-First Search)

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
