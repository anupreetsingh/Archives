# Bellman Ford Algorithm

Bellman-Ford finds the shortest path from one source node to every other reachable node in a weighted graph, including graphs with negative edge weights. It can also detect negative-weight cycles reachable from the source.

## Core Idea

- Keep the best known distance to each node.
- Start the source at distance `0` and every other node at `infinity`.
- Repeatedly scan every edge. One complete scan is called a **pass**.
- For each edge `u -> v` with weight `w`, relax it: if `distances[u] + w < distances[v]`, update `distances[v]`.
- Only relax edges whose starting node is reachable from the source.
- A node's distance can improve across multiple passes; processing it once does not finalize its distance.
- Run at most `V - 1` passes, where `V` is the number of vertices.
- If a full pass makes no updates, stop early: no further relaxation can improve the distances.

### Why `V - 1` Passes?

Without a reachable negative-weight cycle, a shortest path can be chosen without repeating any vertex. With `V` vertices, such a path uses at most `V - 1` edges.

After pass `k`, every shortest path that needs at most `k` edges has been accounted for. Therefore, `V - 1` passes are enough to find all finite shortest distances.

Updates happen immediately within a pass, so a favorable edge order can propagate improvements through several edges in the same pass. `V - 1` is the upper bound, not a requirement to always run every pass.

### Detecting a Negative-Weight Cycle

A negative-weight cycle is a cycle whose edge weights add up to a negative number. Repeating it keeps reducing the path cost, so there is no finite shortest distance to nodes on the cycle or reachable from it.

After the `V - 1` passes, scan the edges once more. If an edge from a reachable node can still be relaxed, a negative-weight cycle is reachable from the source.

A negative cycle in an unreachable part of the graph does not affect this source's shortest paths. The implementation below raises an error if it finds a reachable negative cycle.

## When to Use

- Use it when negative edge weights may exist or you need to detect a negative-weight cycle reachable from the source.
- For graphs with only non-negative weights, [Dijkstra's algorithm](Dijkstra's%20Algorithm.md) is generally faster.
- Use an adjacency list. Each pass walks every node and relaxes its outgoing edges. Unlike [Dijkstra's algorithm](Dijkstra's%20Algorithm.md) there is no cheapest-next node to select, so no heap is needed.

## Directed vs Undirected Graphs

- In a directed graph, each edge only relaxes in its stored direction: `A -> B`.
- In an undirected graph, store each edge in both directions: `A -> B` and `B -> A`.
- A reachable negative undirected edge allows repeated travel back and forth with a negative total cost. Under the usual shortest-path rules that allow repeated edges, this means no finite shortest distance exists for nodes reachable from that edge.

## Implementation

`graph` is an adjacency list where `graph[node]` is a list of `(neighbour, weight)` pairs. Every vertex must be a key, including isolated ones, which map to an empty list. `distances` is built from the keys, so a vertex that only ever appears as a neighbour would be missing from the result.

```python
def bellman_ford(graph, source):
    distances = {node: float("inf") for node in graph}
    distances[source] = 0

    # A shortest path has at most V-1 edges, assuming no reachable negative cycle.
    # So we only need V-1 iterations to expand outward from the source
    # and find the lowest possible distance to every reachable vertex.
    # After i iterations, every path that uses at most i edges is guaranteed to have been found.
    for _ in range(len(graph) - 1):
        updated = False

        for start in graph:
            # Nothing to propagate from a node that is not yet reachable.
            if distances[start] == float("inf"):
                continue
            
            # Relaxing edges with neighbours
            for end, weight in graph[start]:
               
                new_distance = distances[start] + weight

                if new_distance < distances[end]:
                    distances[end] = new_distance
                    updated = True

        # If nothing was updated in this iteration, don't do further iterations
        if not updated:
            break

    # Any further improvement means a reachable negative-weight cycle exists.
    for start in graph:
        if distances[start] == float("inf"):
            continue
        
        # Relaxing edges with neighbours
        for end, weight in graph[start]:
            new_distance = distances[start] + weight

            if new_distance < distances[end]:
                raise ValueError("Negative-weight cycle reachable from source")

    return distances
```

### Example Walkthrough

A pass visits nodes in the order they appear in `graph`, here `C`, `B`, `A`, `D`. That order is deliberately unfavourable: some edges are relaxed before their starting node has received an improved distance, so those improvements only propagate in later passes.

| State | A | B | C | D |
| --- | --- | --- | --- | --- |
| Initial | 0 | infinity | infinity | infinity |
| After pass 1 | 0 | 4 | 5 | infinity |
| After pass 2 | 0 | 4 | 2 | 8 |
| After pass 3 | 0 | 4 | 2 | 5 |

The shortest route to `D` is `A -> B -> C -> D`, with total cost `4 + (-2) + 3 = 5`. The extra cycle-detection scan finds no further improvement. Any unreachable node would retain distance `infinity`.

## Complexity

- **Time: `O(V(V + E))` in the worst case for this implementation**, where `V` is the number of vertices and `E` is the number of edges.

  - Initializing distances takes `O(V)`.
  - Each pass visits all `V` vertices and examines up to `E` outgoing edges. Each relaxation takes `O(1)`, so a full pass takes `O(V + E)`.
  - The relaxation loop runs at most `V - 1` passes, giving `O((V - 1)(V + E))`.
  - The final negative-cycle check scans the vertices and their reachable outgoing edges once more, taking `O(V + E)`. Including initialization, the total is `O(V(V + E))`.
  - When `E` grows at least proportionally to `V`, as in a connected graph, `V + E = O(E)`. This simplifies the total to the usual **`O(VE)`** bound. Scanning a flat edge list instead also gives `O(VE)` for the repeated passes, plus `O(V)` initialization.
  - **Early stopping:** If the loop finishes after `k` passes, the total is `O(V + (k + 1)(V + E))`, including the final cycle check. If the first pass makes no updates, this becomes `O(V + E)`.

- **Space: `O(V)` auxiliary space**, plus `O(V + E)` for the adjacency list.

  - The `distances` dictionary stores one entry per vertex, taking `O(V)`.
  - The remaining variables take `O(1)` space; no heap or separate distance table for each pass is needed.
