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

**Representing a 2D grid as a graph:**

- Each cell (row, col) is a node.
- Edges connect a node to its valid neighbors (up/down/left/right).

```python
from collections import deque

def bfs_grid(start_row, start_col, grid):
    rows, cols = len(grid), len(grid[0])
    visited = set()
    queue = deque([(start_row, start_col)])
    visited.add((start_row, start_col))

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
    visited.add((r, c))
    print(grid[r][c], end=' ')
    for dr, dc in [(1,0), (-1,0), (0,1), (0,-1)]:
        nr, nc = r + dr, c + dc
        if 0 <= nr < rows and 0 <= nc < cols:
            dfs_grid(nr, nc, grid, visited)

# 5x5 grid example
grid_5x5 = [
    [ 1,  2,  3,  4,  5],
    [ 6,  7,  8,  9, 10],
    [11, 12, 13, 14, 15],
    [16, 17, 18, 19, 20],
    [21, 22, 23, 24, 25]
]

print("BFS traversal starting at (0,0):")
bfs_grid(0, 0, grid_5x5)
print("\nDFS traversal starting at (0,0):")
dfs_grid(0, 0, grid_5x5)

# BFS traversal starting at (0,0):
# 1 6 2 11 7 3 16 12 8 4 21 17 13 9 5 22 18 14 10 23 19 15 24 20 25
# DFS traversal starting at (0,0):
# 1 6 11 16 21 22 17 12 7 8 13 18 23 24 19 14 9 10 15 20 25 2 3 4 5
```
