# Backtracking

## Decision Tree (Backtracking Context)

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

## Backtracking

- Depth-First Search (DFS) over the decision tree/solution space.
- General approach:
  1. Choose an option
  2. move deeper into the tree in dfs fashion -> If end condition met → record the solution.
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
    for c in choices: # The for loop lets you go over choices at the current level in decision tree
        if c in path:
            continue
        path.append(c)            # Choose
        backtrack(path, choices, target_sum)  # Explore deeper
        path.pop()               # Undo choice (backtrack)
```

### Normal DFS vs Backtracking

- **Normal DFS:** Use when you need to explore a graph or tree—for example, check whether a node is reachable, find connected components, or visit every tree node. For graph reachability, once a node has been explored, reaching it through a different route adds no new information, so it stays marked as visited.

- **Backtracking:** Use when you need to find valid solutions built from a sequence of choices. Earlier choices affect what you can choose next, so you restore the previous state before trying an alternative. sExamples include permutations, subsets, combinations, Sudoku, N-Queens, all simple paths, and word search without reusing cells within a path.

**Example:** Checking whether any path exists from A to B uses normal DFS. Finding every simple path from A to B uses backtracking: a node cannot repeat within one path, but it can appear in other paths, so you unmark it from visited when returning from the current path.
