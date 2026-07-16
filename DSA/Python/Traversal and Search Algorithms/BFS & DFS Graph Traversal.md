# BFS vs DFS Graph Traversal

## BFS (Breadth-First Search)

- Visits nodes level-by-level (distance from start).
- Explores all neighbors at "length 1" (i.e., one edge away) before moving to length 2, and so on.
- Uses a queue (FIFO) to explore neighbors in order.
- Finds shortest paths in unweighted graphs.
- Good for problems requiring minimum steps or closest nodes.

## DFS (Depth-First Search)

- Explores as deep as possible along a path before backtracking.
- Goes down one branch to the maximum depth ("as far as possible") before exploring other branches.
- Uses recursion or stack (LIFO).
- Does not guarantee shortest path.
- Good for connectivity, cycle detection, topological ordering.

## Both BFS and DFS

- Visit all reachable nodes from a start node.
- Differ mainly in order of visiting nodes and data structure used.
- Usually track visited nodes to avoid processing the same node again.

## Visited State and Cycles

Graphs are not always one-way structures. The same node can often be reached again through another edge.

In an **undirected graph**, every edge can be followed both ways. If `A` is connected to `B`, then from `B` we can immediately go back to `A`.

```text
A --- B
```

Without a `visited` check, traversal can keep moving back and forth forever:

```text
A -> B -> A -> B -> A -> ...
```

In a **directed graph**, a cycle happens when directed edges eventually point back to an earlier node.

```text
A -> B -> C
^         |
|_________|
```

Without a `visited` check, traversal can loop forever:

```text
A -> B -> C -> A -> B -> C -> ...
```

So BFS and DFS usually track a `visited` state to avoid infinite loops and duplicate work.

## Ways to Track Visited Nodes

Common ways to store `visited`:

- `set`: most common when nodes are hashable values like strings, numbers, or `(row, col)` tuples.
- `dict`: useful when you also want to store information like parent, distance, color, or discovery state.
- Boolean list: useful when nodes are numbered from `0` to `n - 1`.
- 2D boolean matrix: useful for grid traversal.
- Parent check: works only for an undirected tree, where the only backward edge is the edge to the parent. For a general graph, use `visited`.

## 2D grid as a graph

- Each cell (row, col) is a node.
- Edges connect a node to its valid neighbors (up/down/left/right).
- A grid with movement in all four directions behaves like an undirected graph, because if you can move from one cell to another, you can usually move back.
- The example below proves the main idea once: BFS and DFS use different traversal orders, but both need `visited` to avoid revisiting cells forever.

```python
from collections import deque

def bfs_grid(start_row, start_col, grid):
    rows, cols = len(grid), len(grid[0])
    visited = set()
    queue = deque([(start_row, start_col)])
    visited.add((start_row, start_col))  # Mark when adding to the queue.

    while queue:
        r, c = queue.popleft()
        print(grid[r][c], end=' ')
        for dr, dc in [(1,0), (-1,0), (0,1), (0,-1)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and (nr, nc) not in visited:
                queue.append((nr, nc))
                visited.add((nr, nc))

def dfs_grid(r, c, grid, visited=None):
    if visited is None:
        visited = set()
    rows, cols = len(grid), len(grid[0])

    if (r, c) in visited:
        return
    visited.add((r, c))  # Mark when the DFS reaches the cell.
    print(grid[r][c], end=' ')
    for dr, dc in [(1,0), (-1,0), (0,1), (0,-1)]:
        nr, nc = r + dr, c + dc
        if 0 <= nr < rows and 0 <= nc < cols:
            dfs_grid(nr, nc, grid, visited)

# 3x3 grid example
grid_3x3 = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

print("BFS traversal starting at (0,0):")
bfs_grid(0, 0, grid_3x3)
print("\nDFS traversal starting at (0,0):")
dfs_grid(0, 0, grid_3x3)

# BFS traversal starting at (0,0):
# 1 4 2 7 5 3 8 6 9
# DFS traversal starting at (0,0):
# 1 4 7 8 5 2 3 6 9
```
