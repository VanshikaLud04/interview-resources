# DSA Pattern Recognition Guide for Interviews

## 1. Two Pointers
**🎯 When to recognize it:**
- Dealing with sorted arrays or linked lists.
- Needing to find a set of elements that fulfill certain constraints (e.g., pairs summing to a target).
- Signal words: "Sorted array", "Find a pair", "In-place modification", "Palindrome".

**🧠 Approach:**
Initialize two pointers (often at the start and end, or both at the start). Move them towards each other or in the same direction based on conditions.

**⏱ Complexity:**
- Time: $O(N)$
- Space: $O(1)$

**💻 Template:**
```python
def two_pointers(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        if condition(arr[left], arr[right]):
            return True
        elif move_left_condition(arr[left], arr[right]):
            left += 1
        else:
            right -= 1
    return False
```

## 2. Sliding Window
**🎯 When to recognize it:**
- Finding a contiguous subarray or substring that satisfies a condition (e.g., max sum, shortest length).
- Signal words: "Contiguous subarray", "Substring", "Maximum/Minimum length", "At most K".

**🧠 Approach:**
Maintain a window `[left, right]`. Expand `right` to include elements. If the window violates the condition, shrink from `left` until valid. Update the answer during this process.

**⏱ Complexity:**
- Time: $O(N)$
- Space: $O(1)$ or $O(K)$ for a hash map

**💻 Template:**
```python
def sliding_window(arr):
    left = 0
    result = 0
    for right in range(len(arr)):
        # Add arr[right] to window state
        while invalid_window_condition():
            # Remove arr[left] from window state
            left += 1
        result = max(result, right - left + 1)
    return result
```

## 3. Binary Search
**🎯 When to recognize it:**
- Searching in a sorted array.
- Finding a boundary (e.g., first true value).
- "Minimize the maximum" or "Maximize the minimum".
- Signal words: "Sorted array", "Logarithmic time required", "Maximum/Minimum possible value".

**🧠 Approach:**
Define a search space `[low, high]`. Check the `mid` point. If it satisfies the condition, discard half the search space and continue.

**⏱ Complexity:**
- Time: $O(\log N)$
- Space: $O(1)$

**💻 Template:**
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

## 4. BFS/DFS (Graph)
**🎯 When to recognize it:**
- Traversing a tree, graph, or matrix.
- Finding the shortest path in an unweighted graph (BFS).
- Exploring all paths, connectivity, or cycle detection (DFS).
- Signal words: "Shortest path", "Connected components", "Tree/Graph", "Matrix paths".

**🧠 Approach:**
- **BFS**: Use a queue. Add neighbors to the queue and mark them visited. Good for shortest path.
- **DFS**: Use recursion or a stack. Explore as deep as possible before backtracking. Good for exhaustive search.

**⏱ Complexity:**
- Time: $O(V + E)$
- Space: $O(V)$

**💻 Template (BFS):**
```python
from collections import deque
def bfs(graph, start):
    queue = deque([start])
    visited = set([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

## 5. Tree Traversals
**🎯 When to recognize it:**
- Any problem involving a Binary Tree or N-ary Tree.
- Signal words: "Preorder/Inorder/Postorder", "Level order", "Lowest Common Ancestor".

**🧠 Approach:**
Use recursion for DFS traverses or a queue for BFS (level-order).

**💻 Template (Inorder DFS):**
```python
def inorder(root):
    if not root:
        return
    inorder(root.left)
    print(root.val)
    inorder(root.right)
```

## 6. Dynamic Programming
**🎯 When to recognize it:**
- Finding an optimal solution (max/min/longest/shortest).
- Counting the number of ways to do something.
- The problem can be broken down into overlapping subproblems.
- Signal words: "Maximum/Minimum", "Number of ways", "Optimal", "Longest".

**🧠 Approach:**
Define a state (e.g., `dp[i]` is the max profit up to index `i`). Find the transition relation (how `dp[i]` relates to previous states). Initialize base cases.

**⏱ Complexity:**
- Time: $O(\text{States} \times \text{Transitions})$
- Space: $O(\text{States})$

**💻 Template (Memoization):**
```python
def solve(n):
    memo = {}
    def dp(i):
        if i == 0: return base_case
        if i in memo: return memo[i]
        memo[i] = transition(dp(i-1), ...)
        return memo[i]
    return dp(n)
```

## 7. Greedy
**🎯 When to recognize it:**
- Making a locally optimal choice at each step leads to a global optimum.
- Often involves sorting.
- Signal words: "Minimum/Maximum", "Jump game", "Interval scheduling".

**🧠 Approach:**
Sort the input if needed. Iterate through and make the best possible choice at the current moment without looking back.

**⏱ Complexity:**
- Time: $O(N \log N)$ (due to sorting)
- Space: $O(1)$ or $O(N)$

## 8. Stack/Monotonic Stack
**🎯 When to recognize it:**
- Matching pairs (e.g., parentheses).
- Finding the "Next Greater Element" or "Next Smaller Element".
- Signal words: "Valid parentheses", "Next greater", "Daily temperatures", "Histogram".

**🧠 Approach:**
Maintain a stack of elements (or their indices) such that the elements are strictly increasing or decreasing. When a new element breaks the monotonic property, pop from the stack and process.

**⏱ Complexity:**
- Time: $O(N)$
- Space: $O(N)$

**💻 Template (Next Greater Element):**
```python
def next_greater(arr):
    stack = []
    res = [-1] * len(arr)
    for i in range(len(arr)):
        while stack and arr[stack[-1]] < arr[i]:
            res[stack.pop()] = arr[i]
        stack.append(i)
    return res
```

## 9. Heap/Priority Queue
**🎯 When to recognize it:**
- Finding the "Kth largest/smallest" element.
- Continuously needing the max/min element from a dynamic collection.
- Signal words: "Top K", "Kth largest", "Median of stream", "Merge K sorted lists".

**🧠 Approach:**
Use `heapq` in Python. A min-heap keeps the smallest element at the top. For max-heap, insert negative values.

**⏱ Complexity:**
- Time: $O(N \log K)$ or $O(K \log N)$
- Space: $O(K)$ or $O(N)$

**💻 Template:**
```python
import heapq
def top_k(arr, k):
    heap = []
    for num in arr:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap
```

## 10. Union-Find
**🎯 When to recognize it:**
- Dealing with connected components in a graph.
- Checking if adding an edge creates a cycle.
- Dynamic connectivity problems.
- Signal words: "Connected components", "Redundant connection", "Islands".

**🧠 Approach:**
Maintain a `parent` array. Use `find` with path compression to find the root of a set, and `union` by rank to merge two sets.

**⏱ Complexity:**
- Time: $O(\alpha(N))$ per operation (Inverse Ackermann)
- Space: $O(N)$

## 11. Trie
**🎯 When to recognize it:**
- Problems involving dictionaries, word search, prefix matching, or autocomplete.
- Signal words: "Prefix", "Word search", "Autocomplete", "Dictionary".

**🧠 Approach:**
Build a tree where each node represents a character. Useful for fast prefix queries.

**⏱ Complexity:**
- Time: $O(L)$ per insert/search where L is word length
- Space: $O(N \times L)$

## 12. Topological Sort
**🎯 When to recognize it:**
- Finding an ordering of tasks with dependencies.
- Detecting cycles in a directed graph.
- Signal words: "Course schedule", "Dependencies", "Task ordering".

**🧠 Approach:**
Use Kahn's Algorithm (BFS) counting in-degrees, or DFS tracking visited states (unvisited, visiting, visited).

**⏱ Complexity:**
- Time: $O(V + E)$
- Space: $O(V + E)$

## 13. Backtracking
**🎯 When to recognize it:**
- Generating all permutations, combinations, or subsets.
- Solving puzzles like Sudoku, N-Queens.
- Signal words: "All possible", "Permutations", "Combinations", "Subsets".

**🧠 Approach:**
Explore all choices recursively. If a choice leads to a dead end, backtrack (undo the choice) and try the next one.

**⏱ Complexity:**
- Time: Exponential $O(2^N)$ or Factorial $O(N!)$
- Space: $O(N)$ recursion depth

**💻 Template:**
```python
def backtrack(path, choices):
    if is_goal(path):
        res.append(path[:])
        return
    for choice in choices:
        if valid(choice):
            path.append(choice)
            backtrack(path, new_choices)
            path.pop()
```

## 14. Interval Problems
**🎯 When to recognize it:**
- Merging overlapping intervals, finding conflicts.
- Signal words: "Intervals", "Overlapping", "Meeting rooms", "Merge".

**🧠 Approach:**
Sort the intervals by their start time. Iterate through and merge if the current interval's start is less than or equal to the previous interval's end.

**⏱ Complexity:**
- Time: $O(N \log N)$
- Space: $O(N)$ for output

## 15. Design/OOD
**🎯 When to recognize it:**
- Designing a system or data structure with specific constraints.
- Signal words: "Design a", "Implement", "LRU Cache".

**🧠 Approach:**
Focus on requirements, state, and choosing the right mix of data structures (e.g., Hash Map + Doubly Linked List for LRU Cache).

## 16. Bit Manipulation
**🎯 When to recognize it:**
- Extremely constrained space/time requirements, operations on bits.
- Signal words: "XOR", "Power of two", "Missing number".

## 17. Prefix Sum
**🎯 When to recognize it:**
- Needing the sum of elements in a range multiple times.
- Signal words: "Subarray sum", "Range query".

**🧠 Approach:**
Precompute a `prefix` array where `prefix[i]` is the sum of elements up to index `i`. Range sum `[i, j]` is `prefix[j] - prefix[i-1]`.

## 18. Linked List
**🎯 When to recognize it:**
- Manipulating nodes in a sequence without array indices.
- Signal words: "Reverse list", "Detect cycle", "Merge lists".

**🧠 Approach:**
Use pointers (`prev`, `curr`, `next`). Use Fast and Slow pointers (Floyd's Cycle Detection) for cycle detection or finding the middle.
