# Backtracking

## 🌳 Decision Tree (Backtracking Context)

- A conceptual tree representing all possible sequences of decisions.
- **Root**: state before making any decision.
- **Level**: a step in the decision-making process.
- **Branch**: one possible choice at that step.
- **Path**: sequence of choices from root to leaf → one complete solution.

Example (permutations of [A,B,C]):

```text
       []
    /   |   \
   A    B    C
  / \  / \  / \
 B  C C  A A  B
 |  | |  | |  |
 C  B A  C B  A
```

Even if no TreeNode object exists, by using recursion you often explore this implicit tree for problem solving.

## 🔄 Backtracking

- Depth-First Search (DFS) over the decision tree/solution space.
- General approach:
  1. Choose an option → move deeper into the tree.
  2. If end condition met → record the solution.
  3. Undo the choice → try the next option (backtrack).
- Core idea: "DFS + Undo Step".

Backtracking: explores all possible choices in a conceptual decision tree. Stops recursion when:

- **Success:** a valid solution is found (satisfactory condition).
- **Failure:** current path violates constraints and is pruned early.

```python
def backtrack(path, choices, target_sum):
    if sum(path) == target_sum:   # Success: found a valid solution
        print("Solution:", path)
        return
    if sum(path) > target_sum:    # Failure: prune this branch (no solution)
        return
    for c in choices:
        if c in path:
            continue
        path.append(c)            # Choose
        backtrack(path, choices, target_sum)  # Explore deeper
        path.pop()               # Undo choice (backtrack)
```

## Comparison: Normal Tree Recursion

Normal Tree recursion: traverses a fixed tree structure. Stops recursion when:

- **Base case:** reached a null node (end of a branch).

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def inorder(node):
    if not node:                  # Base case: no node to process
        return
    inorder(node.left)            # Traverse left subtree
    print(node.val)               # Process current node
    inorder(node.right)           # Traverse right subtree
```
