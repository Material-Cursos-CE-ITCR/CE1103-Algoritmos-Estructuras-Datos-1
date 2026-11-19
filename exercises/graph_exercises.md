# Graphs: Traversal and Shortest Paths

> Preliminary exercise set (version 0.1). All problems are from LeetCode.
> Solutions are hidden in collapsible blocks
> Solutions are in C# 

## Reading material (before the exercises)

| Topic | Reading |
|---|---|
| DFS / BFS, connected components, grid graphs | [USACO Guide – Graph Traversal](https://usaco.guide/silver/graph-traversal) |
| Shortest path theory (Bellman-Ford, Dijkstra, Floyd-Warshall) | [Competitive Programmer's Handbook, ch. 13 (p. 127)](https://usaco.guide/CPH.pdf#page=127) |
| Dijkstra in depth (priority queue implementation, proof) | [cp-algorithms – Dijkstra](https://cp-algorithms.com/graph/dijkstra.html) |

---

## Contents

### Part 1 — Traversal (DFS / BFS)
| # | Problem | Difficulty | Key idea |
|---|---|---|---|
| 1 | [733. Flood Fill](https://leetcode.com/problems/flood-fill/) | Easy | DFS on a grid |
| 2 | [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | Counting components on a grid |
| 3 | [547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/) | Medium | Counting components (adjacency matrix) |
| 4 | [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium | Multi-source BFS by levels |
| 5 | [785. Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) | Medium | BFS with 2 colors |

### Part 2 — Shortest paths
| # | Problem | Difficulty | Key idea |
|---|---|---|---|
| 6 | [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium | BFS (unweighted edges) |
| 7 | [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium | Classic Dijkstra |
| 8 | [1514. Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Medium | "Reversed" Dijkstra (maximize) |
| 9 | [1631. Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Medium | Dijkstra on a grid with cost = maximum |

---

## Base templates

**BFS on a grid** — use it when all edges have the same weight. Complexity $O(R \cdot C)$.

```csharp
// Directions: down, up, right, left
int[] dr = { 1, -1, 0, 0 };
int[] dc = { 0, 0, 1, -1 };

Queue<(int r, int c)> queue = new Queue<(int r, int c)>();
queue.Enqueue((startR, startC));
visited[startR, startC] = true;

while (queue.Count > 0) {
    var (r, c) = queue.Dequeue();

    for (int d = 0; d < 4; d++) {
        int nr = r + dr[d];
        int nc = c + dc[d];

        // Check that the neighbor cell is valid and not visited
        if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
        if (visited[nr, nc]) continue;

        visited[nr, nc] = true;
        queue.Enqueue((nr, nc));
    }
}
```

**Dijkstra** — edges with **non-negative** weights. Complexity $O((V + E)\log V)$.

C#'s `PriorityQueue<TElement, TPriority>` is a **min-heap**: `Dequeue` removes the element with the smallest priority.

```csharp
// graph[u] = list of pairs (neighbor, weight)
int[] dist = new int[n];
Array.Fill(dist, int.MaxValue);

// Element = node, priority = distance
PriorityQueue<int, int> pq = new PriorityQueue<int, int>();

dist[source] = 0;
pq.Enqueue(source, 0);

while (pq.TryDequeue(out int u, out int d)) {
    // If we already found something better for u, this entry is stale
    if (d > dist[u]) continue;

    foreach (var (v, weight) in graph[u]) {
        if (dist[u] + weight < dist[v]) {
            dist[v] = dist[u] + weight;
            pq.Enqueue(v, dist[v]);
        }
    }
}
```

---

# Part 1 — Traversal

## 1. Flood Fill (733) — Easy

**Summary:** given an image (matrix), a starting cell and a color, paint the whole connected region (4 directions) that has the same color as the starting cell.

**Hint:** it's a DFS from `(sr, sc)` that only moves through cells of the original color.

<details>
<summary>Solution</summary>

**Idea:** save the original color, then DFS while changing the color. Careful: if the original color is already the new color, the DFS never ends (cells never "change"), so return the image as is.

**Complexity:** $O(R \cdot C)$ time and memory (recursion).

```csharp
public class Solution {
    private void Dfs(int[][] image, int r, int c, int originalColor, int newColor) {
        // Outside the image
        if (r < 0 || r >= image.Length || c < 0 || c >= image[0].Length) return;
        // Not part of the region
        if (image[r][c] != originalColor) return;

        // Paint (this also works as "visited")
        image[r][c] = newColor;

        Dfs(image, r + 1, c, originalColor, newColor);
        Dfs(image, r - 1, c, originalColor, newColor);
        Dfs(image, r, c + 1, originalColor, newColor);
        Dfs(image, r, c - 1, originalColor, newColor);
    }

    public int[][] FloodFill(int[][] image, int sr, int sc, int color) {
        int originalColor = image[sr][sc];

        // Special case: nothing to change
        if (originalColor == color) return image;

        Dfs(image, sr, sc, originalColor, color);
        return image;
    }
}
```
</details>

---

## 2. Number of Islands (200) — Medium

**Summary:** in a grid of `'1'` (land) and `'0'` (water), count how many islands there are (land connected in 4 directions).

**Hint:** every time you find an unvisited `'1'`, it's a new island. Then "sink" the whole island with DFS.

<details>
<summary>Solution</summary>

**Idea:** go through the grid. When you find a `'1'`, add 1 to the counter and run a DFS that turns the whole island into `'0'` so it isn't counted again. This is exactly **counting connected components**.

**Complexity:** $O(R \cdot C)$.

```csharp
public class Solution {
    private int rows;
    private int cols;

    // Turns the whole island containing (r, c) into water
    private void Sink(char[][] grid, int r, int c) {
        if (r < 0 || r >= rows || c < 0 || c >= cols) return;
        if (grid[r][c] != '1') return;

        grid[r][c] = '0'; // mark as visited

        Sink(grid, r + 1, c);
        Sink(grid, r - 1, c);
        Sink(grid, r, c + 1);
        Sink(grid, r, c - 1);
    }

    public int NumIslands(char[][] grid) {
        rows = grid.Length;
        cols = grid[0].Length;
        int islands = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    islands++;        // found a new island
                    Sink(grid, r, c); // erase it completely
                }
            }
        }
        return islands;
    }
}
```
</details>

---

## 3. Number of Provinces (547) — Medium

**Summary:** there are `n` cities and a matrix `isConnected` where `isConnected[i][j] = 1` if they are directly connected. Count how many groups (provinces) there are.

**Hint:** same as the previous one, but the graph comes as an **adjacency matrix** instead of a grid.

<details>
<summary>Solution</summary>

**Idea:** a `visited` array. For every unvisited city, count one province and run a DFS marking every reachable city.

**Complexity:** $O(n^2)$, because we check the whole row of each node.

```csharp
public class Solution {
    private void Dfs(int[][] isConnected, bool[] visited, int u) {
        visited[u] = true;
        int n = isConnected.Length;

        for (int v = 0; v < n; v++) {
            // If there is an edge u-v and v hasn't been visited, go to v
            if (isConnected[u][v] == 1 && !visited[v]) {
                Dfs(isConnected, visited, v);
            }
        }
    }

    public int FindCircleNum(int[][] isConnected) {
        int n = isConnected.Length;
        bool[] visited = new bool[n]; // everything starts as false
        int provinces = 0;

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                provinces++;
                Dfs(isConnected, visited, i);
            }
        }
        return provinces;
    }
}
```
</details>

---

## 4. Rotting Oranges (994) — Medium

**Summary:** grid with `0` (empty), `1` (fresh orange), `2` (rotten orange). Every minute, rotten oranges rot their neighbors in 4 directions. How many minutes until no fresh oranges are left? `-1` if it's impossible.

**Hint:** all rotten oranges start **at the same time** → put all of them in the queue from the beginning (multi-source BFS). Each BFS "level" is one minute.

<details>
<summary>Solution</summary>

**Idea:**
1. Put all rotten oranges in the queue and count the fresh ones.
2. BFS by levels: each round processes exactly the elements that were in the queue (one minute).
3. At the end, if fresh oranges remain, return `-1`.

**Complexity:** $O(R \cdot C)$.

```csharp
public class Solution {
    public int OrangesRotting(int[][] grid) {
        int rows = grid.Length;
        int cols = grid[0].Length;

        Queue<(int r, int c)> queue = new Queue<(int r, int c)>();
        int fresh = 0;

        // Step 1: find the initially rotten oranges and count fresh ones
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 2) queue.Enqueue((r, c));
                if (grid[r][c] == 1) fresh++;
            }
        }

        int[] dr = { 1, -1, 0, 0 };
        int[] dc = { 0, 0, 1, -1 };
        int minutes = 0;

        // Step 2: BFS by levels
        while (queue.Count > 0 && fresh > 0) {
            int levelSize = queue.Count; // oranges that rotted in the previous minute

            for (int i = 0; i < levelSize; i++) {
                var (r, c) = queue.Dequeue();

                for (int d = 0; d < 4; d++) {
                    int nr = r + dr[d];
                    int nc = c + dc[d];

                    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                    if (grid[nr][nc] != 1) continue; // only rot fresh ones

                    grid[nr][nc] = 2;
                    fresh--;
                    queue.Enqueue((nr, nc));
                }
            }
            minutes++;
        }

        // Step 3: check whether any fresh orange is left
        if (fresh > 0) return -1;
        return minutes;
    }
}
```
</details>

---

## 5. Is Graph Bipartite? (785) — Medium

**Summary:** given an undirected graph (adjacency list, may be disconnected), decide whether it can be split into two groups such that every edge goes from one group to the other.

**Hint:** try painting with 2 alternating colors. If two neighbors end up with the same color, it's not bipartite. Watch out: the graph can have several components!

<details>
<summary>Solution</summary>

**Idea:** `color[u] = -1` means unpainted. For every unpainted node, start a BFS that paints each neighbor with the opposite color (`1 - color[u]`).

**Complexity:** $O(V + E)$.

```csharp
public class Solution {
    public bool IsBipartite(int[][] graph) {
        int n = graph.Length;
        int[] color = new int[n];
        Array.Fill(color, -1); // -1 = unpainted, 0 or 1 = group

        for (int start = 0; start < n; start++) {
            if (color[start] != -1) continue; // already painted in another component

            Queue<int> queue = new Queue<int>();
            queue.Enqueue(start);
            color[start] = 0;

            while (queue.Count > 0) {
                int u = queue.Dequeue();

                foreach (int v in graph[u]) {
                    if (color[v] == -1) {
                        // Paint with the opposite color
                        color[v] = 1 - color[u];
                        queue.Enqueue(v);
                    } else if (color[v] == color[u]) {
                        // Two neighbors with the same color: conflict
                        return false;
                    }
                }
            }
        }
        return true;
    }
}
```
</details>

---

# Part 2 — Shortest paths

> **Quick rule for choosing the algorithm:**
> - All edges have the same weight → **BFS**
> - Non-negative weights → **Dijkstra**

## 6. Shortest Path in Binary Matrix (1091) — Medium

**Summary:** `n × n` grid of 0s and 1s. Find the shortest path from `(0,0)` to `(n-1,n-1)` going only through `0` cells, moving in **8 directions**. The length is counted in visited cells. `-1` if there is no path.

**Hint:** all edges weigh 1 → regular BFS. Only the directions change.

<details>
<summary>Solution</summary>

**Idea:** BFS storing in `dist[r, c]` the number of cells in the path. First check that the start and end aren't blocked.

**Complexity:** $O(n^2)$.

```csharp
public class Solution {
    public int ShortestPathBinaryMatrix(int[][] grid) {
        int n = grid.Length;

        // If the start or the end is blocked, there is no path
        if (grid[0][0] == 1 || grid[n - 1][n - 1] == 1) return -1;

        // 8 directions
        int[] dr = { 1, -1, 0, 0, 1, 1, -1, -1 };
        int[] dc = { 0, 0, 1, -1, 1, -1, 1, -1 };

        // -1 = not visited
        int[,] dist = new int[n, n];
        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                dist[r, c] = -1;
            }
        }

        Queue<(int r, int c)> queue = new Queue<(int r, int c)>();
        dist[0, 0] = 1; // the path already includes the starting cell
        queue.Enqueue((0, 0));

        while (queue.Count > 0) {
            var (r, c) = queue.Dequeue();

            for (int d = 0; d < 8; d++) {
                int nr = r + dr[d];
                int nc = c + dc[d];

                if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                if (grid[nr][nc] == 1) continue;  // blocked
                if (dist[nr, nc] != -1) continue; // already visited

                dist[nr, nc] = dist[r, c] + 1;
                queue.Enqueue((nr, nc));
            }
        }

        return dist[n - 1, n - 1];
    }
}
```
</details>

---

## 7. Network Delay Time (743) — Medium

**Summary:** `n` nodes (numbered 1..n) and directed edges `times[i] = [u, v, w]` (it takes `w` to go from `u` to `v`). A signal is sent from `k`. How long until it reaches **all** nodes? `-1` if some node never gets it.

**Hint:** it's straightforward Dijkstra from `k`. The answer is the largest distance.

<details>
<summary>Solution</summary>

**Idea:** Dijkstra from `k`, then take the maximum of `dist`. If any node stayed at infinity, return `-1`.

**Complexity:** $O((V + E)\log V)$.

```csharp
public class Solution {
    public int NetworkDelayTime(int[][] times, int n, int k) {
        // Build adjacency list (nodes 1 to n): pairs (neighbor, weight)
        List<(int v, int w)>[] graph = new List<(int v, int w)>[n + 1];
        for (int i = 0; i <= n; i++) {
            graph[i] = new List<(int v, int w)>();
        }
        foreach (int[] t in times) {
            graph[t[0]].Add((t[1], t[2]));
        }

        // Dijkstra
        int[] dist = new int[n + 1];
        Array.Fill(dist, int.MaxValue);

        PriorityQueue<int, int> pq = new PriorityQueue<int, int>(); // (node, distance)
        dist[k] = 0;
        pq.Enqueue(k, 0);

        while (pq.TryDequeue(out int u, out int d)) {
            if (d > dist[u]) continue; // stale entry

            foreach (var (v, w) in graph[u]) {
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.Enqueue(v, dist[v]);
                }
            }
        }

        // The answer is the node that takes the longest to reach
        int answer = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == int.MaxValue) return -1; // unreachable node
            answer = Math.Max(answer, dist[i]);
        }
        return answer;
    }
}
```
</details>

---

## 8. Path with Maximum Probability (1514) — Medium

**Summary:** **undirected** graph; each edge has a success probability. Find the maximum probability of going from `start_node` to `end_node` (probabilities along a path are multiplied). `0` if there is no path.

**Hint:** Dijkstra, but looking for the **maximum** instead of the minimum. Why does it work? Multiplying by a number in $[0, 1]$ never makes the probability grow, just like adding a non-negative weight never makes the distance shrink.

<details>
<summary>Solution</summary>

**Idea:** same as Dijkstra but relaxing with multiplication and taking the **largest** probability first. Since C#'s `PriorityQueue` is a min-heap, use the negative probability (`-p`) as the priority: the largest one comes out first.

**Complexity:** $O((V + E)\log V)$.

```csharp
public class Solution {
    public double MaxProbability(int n, int[][] edges, double[] succProb, int start_node, int end_node) {
        // Undirected graph: pairs (neighbor, probability)
        List<(int v, double p)>[] graph = new List<(int v, double p)>[n];
        for (int i = 0; i < n; i++) {
            graph[i] = new List<(int v, double p)>();
        }
        for (int i = 0; i < edges.Length; i++) {
            int u = edges[i][0];
            int v = edges[i][1];
            double p = succProb[i];
            graph[u].Add((v, p));
            graph[v].Add((u, p));
        }

        double[] prob = new double[n]; // everything starts at 0.0

        // Priority = -probability, so the largest comes out first
        PriorityQueue<int, double> pq = new PriorityQueue<int, double>();
        prob[start_node] = 1.0;
        pq.Enqueue(start_node, -1.0);

        while (pq.TryDequeue(out int u, out double priority)) {
            double currentProb = -priority;
            if (currentProb < prob[u]) continue; // stale entry

            foreach (var (v, edgeProb) in graph[u]) {
                // Relax: is reaching v through u more likely?
                if (prob[u] * edgeProb > prob[v]) {
                    prob[v] = prob[u] * edgeProb;
                    pq.Enqueue(v, -prob[v]);
                }
            }
        }

        return prob[end_node];
    }
}
```
</details>

---

## 9. Path With Minimum Effort (1631) — Medium

**Summary:** grid of heights. The **effort** of a path is the largest absolute height difference between two consecutive cells on the path. Find the minimum effort to go from the top-left corner to the bottom-right corner.

**Hint:** Dijkstra where the "cost" of a path is not the sum, but the **maximum**. When going from `u` to `v`, the new cost is $\max(\text{effort}[u], |h_u - h_v|)$.

<details>
<summary>Solution</summary>

**Idea:** Dijkstra on a grid; every cell is a node. The queue stores the cell `(r, c)` with priority equal to its effort.

**Complexity:** $O(R \cdot C \cdot \log(R \cdot C))$.

```csharp
public class Solution {
    public int MinimumEffortPath(int[][] heights) {
        int rows = heights.Length;
        int cols = heights[0].Length;

        int[] dr = { 1, -1, 0, 0 };
        int[] dc = { 0, 0, 1, -1 };

        int[,] effort = new int[rows, cols];
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                effort[r, c] = int.MaxValue;
            }
        }

        // Element = cell, priority = effort
        PriorityQueue<(int r, int c), int> pq = new PriorityQueue<(int r, int c), int>();
        effort[0, 0] = 0;
        pq.Enqueue((0, 0), 0);

        while (pq.TryDequeue(out var cell, out int e)) {
            int r = cell.r;
            int c = cell.c;

            if (e > effort[r, c]) continue; // stale entry

            // When the destination comes out of the queue, it's already optimal
            if (r == rows - 1 && c == cols - 1) return e;

            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d];
                int nc = c + dc[d];

                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;

                int jump = Math.Abs(heights[nr][nc] - heights[r][c]);
                int newEffort = Math.Max(e, jump); // cost = maximum, not sum

                if (newEffort < effort[nr, nc]) {
                    effort[nr, nc] = newEffort;
                    pq.Enqueue((nr, nc), newEffort);
                }
            }
        }

        return effort[rows - 1, cols - 1];
    }
}
```
</details>

---

## To do in future versions (ideas)

- Topological sort (e.g. 207. Course Schedule)
- Bellman-Ford (e.g. 787. Cheapest Flights Within K Stops)
- 0-1 BFS (e.g. 2290. Minimum Obstacle Removal to Reach Corner)
- Floyd-Warshall (all pairs)
- Path reconstruction (not just the distance)
- USACO Silver / CSES problems for practice outside LeetCode
