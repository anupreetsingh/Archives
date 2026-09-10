# BFS vs DFS Graph Traversal

## Nodes and Edges in a Graph

### Undirected Graph

An **undirected graph** has edges with no direction, so an edge between `A` and `B` can be used both from `A` to `B` and from `B` to `A`.

For a simple undirected graph with `V` nodes and `E` edges:

- `E = 0` means there are no edges, so the graph is disconnected when `V > 1`.
- `E = V - 1` is the fewest number of edges required to make a graph connected, so every node can be reached from every other node. A **tree** is a connected graph with exactly `V - 1` edges.
- `E = VC2 = V(V - 1) / 2` is the maximum number of edges for a graph with `V` nodes. In this case every pair of nodes is directly connected to each other.

So the edge-count bounds are:

```text
Connected graph:  V - 1 <= E <= V(V - 1) / 2
Any graph:        0 <= E <= V(V - 1) / 2
```

### Directed Graph

A **directed graph** has edges with direction, so an edge from `A` to `B` can be used from `A` to `B`, but not automatically from `B` to `A`.

For a simple directed graph with `V` nodes and `E` directed edges:

- `E = 0` means there are no edges, so the graph is disconnected when `V > 1`.
- `E = V - 1` is the fewest number of edges required to make the graph **weakly connected**, meaning it would be connected if edge directions were ignored.
- `E = V` is the fewest number of edges required to make the graph **strongly connected** when `V > 1`, meaning every node can reach every other node by following edge directions.
- `E = V(V - 1)` is the maximum number of directed edges, because each pair of nodes can have 2 ordered edges.

So the edge-count bounds are:

```text
Weakly connected directed graph:    V - 1 <= E <= V(V - 1)
Strongly connected directed graph:  V <= E <= V(V - 1)
Any directed graph:                 0 <= E <= V(V - 1)
```

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

### Time and Space Complexity

For basic graph traversal, BFS and DFS have the same Big-O time and space complexity.

Graphs are commonly implemented or represented in a few different ways. The traversal complexity depends on which representation is used.

#### Adjacency List

An **adjacency list** stores the graph as each node with a list of its direct neighbors.

Example:

```python
graph = {
    "A": ["B", "C"],
    "B": ["A"],
    "C": ["A"],
}
```

For both BFS and DFS:

```text
Time:  O(V + E)
Space: O(V)
```

Where:

- `V` = number of vertices/nodes.
- `E` = number of edges.

The time is `O(V + E)` because each reachable node is visited once, and each edge from those nodes is checked once while scanning neighbors.

The space is `O(V)` because of the `visited` structure. BFS can also hold up to `O(V)` nodes in the queue, and DFS can use up to `O(V)` space in the recursion call stack or explicit stack.

#### Adjacency Matrix

An **adjacency matrix** stores the graph as a `V x V` table. Each row and column represents a node. A value like `1` means an edge exists, and `0` means no edge exists.

Example:

```python
graph = [
    # A  B  C
    [0, 1, 1],  # A
    [1, 0, 0],  # B
    [1, 0, 0],  # C
]
```

For an adjacency matrix, checking all possible neighbors of one node takes `O(V)`, so traversing all nodes takes:

```text
Time:  O(V^2)
Space: O(V)
```

#### 2D Grid

A **2D grid** as graph means that if the grid has `R` rows and `C` columns, each cell is treated like a node:

```text
Time:  O(R * C)
Space: O(R * C)
```

Each cell is visited at most once, and each cell has only a constant number of neighbors, usually up, down, left, and right.

### Tracking Visited State

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

**Common ways to store `visited`**:

- `set`: most common when nodes are hashable values like strings, numbers, or `(row, col)` tuples.
- `dict`: useful when you also want to store information like parent, distance, color, or discovery state.
- Boolean list: useful when nodes are numbered from `0` to `V - 1`.
- 2D boolean matrix: useful for grid traversal.
- Parent check: works only for an undirected tree, where the only backward edge is the edge to the parent. For a general graph, use `visited`.

## DAG (Directed Acyclic Graph)

A **DAG** is a directed graph with no cycles.

- **Directed** means edges have a one-way direction.
- **Acyclic** means there is no path that eventually comes back to the same node.

But there can be multiple paths leading to the same node.

Example:

```text
0 -> 1
|    |
v    v
2 -> 3
```

A DAG is useful when something must happen before something else:

- Course prerequisites.
- Task scheduling.
- Build systems.
- Dependency graphs.

### Topological Sort

A **topological sort** is an ordering of nodes in a DAG where every node appears before the nodes that depend on it.

If there is an edge:

```text
A -> B
```

Then `A` must come before `B` in the topological order.

For this graph:

```text
0 -> 1
|    |
v    v
2 -> 3
```

One valid topological order is:

```text
0, 1, 2, 3
```

Another valid topological order is:

```text
0, 2, 1, 3
```

Both are valid because:

- `0` comes before `1`.
- `0` comes before `2`.
- `1` comes before `3`.
- `2` comes before `3`.

Topological sort is only possible for a **DAG**. If the graph has a cycle, there is no valid topological order.

### DFS vs Kahn's Algorithm

There are two common ways to find a topological order in a DAG:

- **DFS-based topological sort**: When the graph is modeled as each node pointing to its prerequisites, DFS adds each node only after all of its prerequisites have been visited. Useful when the problem naturally fits recursion, postorder traversal, or DFS-based cycle detection.
- **Kahn's Algorithm**: When the graph is modeled as each prerequisite pointing to the nodes that depend on it, the algorithm repeatedly processes nodes with an in-degree of `0`. An in-degree of `0` means the node has no remaining prerequisites, so it can be safely added to the ordering. Useful the problem is about prerequisites, dependency counts, or repeatedly choosing nodes with no remaining prerequisites.

> **LeetCode # 210**: Course Schedule II, is the classic course-prerequisite problem for topological sorting. The submission for that problem shows how to find the topological order using both DFS topological sort and Kahn's Algorithm.

Both have the same complexity:

```text
Time:  O(V + E)
Space: O(V)
```

Where:

- `V` = number of vertices/nodes.
- `E` = number of edges.
