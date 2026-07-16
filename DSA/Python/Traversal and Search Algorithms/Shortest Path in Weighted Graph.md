# Shortest Path in Weighted Graph

> Prims and Kruskal

## Dijkstra's Algorithm — Key Notes

**Purpose:**

- Finds the shortest path from a single source node to all other nodes
- Works on weighted graphs with NON-NEGATIVE weights only

**Why not negative?**

- Algorithm assumes once the shortest path to a node is found, it never improves later.
- Negative edges could invalidate that assumption → incorrect results

**Data Structures Used:**

- Graph stored as adjacency list → `{node: [(neighbor, weight), ...]}`
- min-heap (priority queue) to always pick the shortest distance node next

**Complexity:**

- Time: O(E log V) using priority queue
- Space: O(V)

**Steps:**

1️⃣ Initialize distances (0 to source, ∞ to others)
2️⃣ Push source into priority queue
3️⃣ While queue not empty:

- → Pop smallest distance node
- → Relax edges (update if shorter path found)
4️⃣ Return distance array

```python
import heapq  # For priority queue (min-heap)

def dijkstra(graph, source):
    # Step 1: Initialize distances (∞ for all except source)
    distances = {node: float('inf') for node in graph}
    distances[source] = 0

    # Min-heap storing pairs (distance, node)
    pq = [(0, source)]

    while pq:
        current_dist, current_node = heapq.heappop(pq)

        # Optimization: Skip outdated distances
        if current_dist > distances[current_node]:
            continue

        # Step 3: Relax edges
        for neighbor, weight in graph[current_node]:
            distance = current_dist + weight

            # Found shorter path?
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(pq, (distance, neighbor))

    return distances
```
