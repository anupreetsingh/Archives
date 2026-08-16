# Dijkstra's Algorithm

Dijkstra's algorithm finds the shortest path from one source node to every other reachable node in a weighted graph.

## Core Idea

- Keep the best known distance to each node.
- Start the source at distance `0` and every other node at `infinity`.
- Repeatedly choose the unvisited node with the smallest known distance.
- A min-heap is used to get this smallest-distance node efficiently.
- Once a node is removed from the heap with the smallest distance, its shortest distance is finalized.
- This works because all edge weights are non-negative: any future path through another unvisited node can only add more cost, so it cannot improve the current smallest distance.
- Relax each outgoing edge by checking whether going through the current node gives a shorter path.
- If a heap entry has an older distance than the current best distance, skip it.

## When to Use

- Use it for graphs with non-negative edge weights.
- Works on both directed graphs and undirected graphs.
- Use an adjacency list plus a min-heap for efficient implementation.
- Do not use it when negative edge weights exist; use Bellman-Ford instead.

## Directed vs Undirected Graphs

- In a directed graph, each edge only relaxes in its stored direction: `A -> B`.
- In an undirected graph, treat each edge as two directed edges: `A -> B` and `B -> A`.

## Implementation

```python
import heapq


def dijkstra(graph, source):
    distances = {node: float("inf") for node in graph}
    distances[source] = 0

    min_heap = [(0, source)]

    while min_heap:
        current_distance, current_node = heapq.heappop(min_heap)

        if current_distance > distances[current_node]:
            continue

        for neighbor, weight in graph[current_node]:
            new_distance = current_distance + weight

            if new_distance < distances[neighbor]:
                distances[neighbor] = new_distance
                heapq.heappush(min_heap, (new_distance, neighbor))

    return distances
```

## Complexity

- Time: `O(E log V)` with a priority queue, where `E` is edges and `V` is vertices.
- Space: `O(V)` for distances and heap state, plus the graph storage.
