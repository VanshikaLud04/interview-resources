# DSA Patterns — Backend/AI Interview Focus

> Priority: Medium/Hard on Sliding Window, Two Pointers, Graphs, Heaps, DP. These cover 80% of what Backend/AI roles test.

---

## Must-Do 30 (FAANG Core)

| # | Problem | Pattern | TC | SC |
|---|---------|---------|----|----|
| 1 | [Two Sum](https://leetcode.com/problems/two-sum) | HashMap | O(n) | O(n) |
| 2 | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters) | Sliding Window | O(n) | O(min(m,n)) |
| 3 | [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring) | Expand Around Center | O(n²) | O(1) |
| 4 | [Container With Most Water](https://leetcode.com/problems/container-with-most-water) | Two Pointers | O(n) | O(1) |
| 5 | [3Sum](https://leetcode.com/problems/3sum) | Sort + Two Pointers | O(n²) | O(1) |
| 6 | [Remove Nth Node From End](https://leetcode.com/problems/remove-nth-node-from-end-of-list) | Two Pointers | O(n) | O(1) |
| 7 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses) | Stack | O(n) | O(n) |
| 8 | [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists) | Two Pointers | O(n+m) | O(1) |
| 9 | [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists) | Min Heap | O(n log k) | O(k) |
| 10 | [Search in Rotated Array](https://leetcode.com/problems/search-in-rotated-sorted-array) | Binary Search | O(log n) | O(1) |
| 11 | [Combination Sum](https://leetcode.com/problems/combination-sum) | Backtracking | O(2ⁿ) | O(n) |
| 12 | [Maximum Subarray](https://leetcode.com/problems/maximum-subarray) | Kadane's | O(n) | O(1) |
| 13 | [Merge Intervals](https://leetcode.com/problems/merge-intervals) | Sort + Greedy | O(n log n) | O(n) |
| 14 | [Climbing Stairs](https://leetcode.com/problems/climbing-stairs) | DP | O(n) | O(1) |
| 15 | [LRU Cache](https://leetcode.com/problems/lru-cache) | HashMap + DLL | O(1) | O(n) |
| 16 | [Number of Islands](https://leetcode.com/problems/number-of-islands) | BFS/DFS | O(m×n) | O(m×n) |
| 17 | [Course Schedule II](https://leetcode.com/problems/course-schedule-ii) | Topological Sort | O(V+E) | O(V+E) |
| 18 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water) | Two Pointers | O(n) | O(1) |
| 19 | [Kth Largest Element](https://leetcode.com/problems/kth-largest-element-in-an-array) | Min Heap/Quickselect | O(n log k) | O(k) |
| 20 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring) | Sliding Window | O(n) | O(k) |
| 21 | [Word Search](https://leetcode.com/problems/word-search) | DFS + Backtracking | O(m×n×4ᵏ) | O(k) |
| 22 | [Validate BST](https://leetcode.com/problems/validate-binary-search-tree) | DFS with bounds | O(n) | O(h) |
| 23 | [Binary Tree Level Order](https://leetcode.com/problems/binary-tree-level-order-traversal) | BFS | O(n) | O(n) |
| 24 | [Construct BT from Preorder+Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal) | Recursion | O(n) | O(n) |
| 25 | [Best Time to Buy/Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock) | Greedy | O(n) | O(1) |
| 26 | [Unique Paths](https://leetcode.com/problems/unique-paths) | DP | O(m×n) | O(m×n) |
| 27 | [Jump Game](https://leetcode.com/problems/jump-game) | Greedy | O(n) | O(1) |
| 28 | [Rotate Image](https://leetcode.com/problems/rotate-image) | Matrix | O(n²) | O(1) |
| 29 | [Group Anagrams](https://leetcode.com/problems/group-anagrams) | HashMap | O(n·k·log k) | O(n·k) |
| 30 | [Spiral Matrix](https://leetcode.com/problems/spiral-matrix) | Simulation | O(m×n) | O(1) |

---

## Pattern Templates

### Sliding Window
**When:** Subarray/substring problems with a size or condition constraint.
```python
left = 0
window = {}
for right in range(len(s)):
    # expand window: add s[right]
    window[s[right]] = window.get(s[right], 0) + 1
    while WINDOW_INVALID:  # e.g. len(window) > k
        # shrink: remove s[left]
        window[s[left]] -= 1
        if window[s[left]] == 0: del window[s[left]]
        left += 1
    result = max(result, right - left + 1)
```

### Two Pointers
**When:** Sorted array, finding pairs/triplets, partitioning.
```python
left, right = 0, len(arr) - 1
while left < right:
    if condition_met(arr[left], arr[right]):
        result.append(...)
        left += 1; right -= 1
    elif too_small:
        left += 1
    else:
        right -= 1
```

### BFS (Level Order / Shortest Path)
**When:** Shortest path in unweighted graph, level-by-level traversal.
```python
from collections import deque
q = deque([start])
visited = {start}
while q:
    node = q.popleft()
    for neighbor in graph[node]:
        if neighbor not in visited:
            visited.add(neighbor)
            q.append(neighbor)
```

### DFS + Backtracking
**When:** Permutations, combinations, grid exploration, constraint satisfaction.
```python
def backtrack(start, path):
    if BASE_CASE: result.append(path[:]); return
    for i in range(start, len(nums)):
        if SKIP_CONDITION: continue
        path.append(nums[i])
        backtrack(i + 1, path)
        path.pop()  # undo choice
```

### Topological Sort (Kahn's BFS)
**When:** Dependency ordering, detecting cycles in directed graphs.
```python
in_degree = {node: 0 for node in graph}
for node in graph:
    for neighbor in graph[node]:
        in_degree[neighbor] += 1
q = deque([n for n in in_degree if in_degree[n] == 0])
order = []
while q:
    node = q.popleft(); order.append(node)
    for neighbor in graph[node]:
        in_degree[neighbor] -= 1
        if in_degree[neighbor] == 0: q.append(neighbor)
# cycle exists if len(order) != len(graph)
```

### Binary Search (Generic)
**When:** Sorted array search, "find minimum valid X" (search space problems).
```python
left, right = 0, len(arr) - 1
while left <= right:
    mid = (left + right) // 2
    if arr[mid] == target: return mid
    elif arr[mid] < target: left = mid + 1
    else: right = mid - 1
# For "find minimum valid": change to left < right, return left
```

### Min Heap (Top K)
**When:** Kth largest/smallest, merge K sorted lists, task scheduling.
```python
import heapq
heap = []
for num in nums:
    heapq.heappush(heap, num)
    if len(heap) > k:
        heapq.heappop(heap)  # maintain size k
return heap[0]  # kth largest
```

### Dynamic Programming (1D)
**When:** Optimal substructure (Fibonacci-type, coin change, climb stairs).
```python
dp = [0] * (n + 1)
dp[0] = BASE_CASE
for i in range(1, n + 1):
    dp[i] = RECURRENCE(dp[i-1], dp[i-2], ...)
return dp[n]
# Space optimization: often just need prev two values
```

### LRU Cache (HashMap + Doubly Linked List)
**When:** Cache design, O(1) get and put.
```python
class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = {}  # key -> node
        # Dummy head and tail
        self.head, self.tail = Node(), Node()
        self.head.next = self.tail; self.tail.prev = self.head

    def get(self, key):
        if key in self.cache:
            self._move_to_front(self.cache[key])
            return self.cache[key].val
        return -1

    def put(self, key, val):
        if key in self.cache: self._remove(self.cache[key])
        node = Node(key, val)
        self.cache[key] = node
        self._add_to_front(node)
        if len(self.cache) > self.cap:
            lru = self.tail.prev
            self._remove(lru)
            del self.cache[lru.key]
```

---

## Google-Specific (Most Frequent 2025-26)

| # | Problem | Category |
|---|---------|---------|
| 1 | [Number of Islands](https://leetcode.com/problems/number-of-islands) | Graph/DFS |
| 2 | [LRU Cache](https://leetcode.com/problems/lru-cache) | Design |
| 3 | [Course Schedule II](https://leetcode.com/problems/course-schedule-ii) | Topological Sort |
| 4 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water) | Two Pointers |
| 5 | [Serialize/Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree) | Trees |
| 6 | [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays) | Binary Search |
| 7 | [Word Ladder](https://leetcode.com/problems/word-ladder) | BFS |
| 8 | [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii) | Heap/Greedy |

## Meta-Specific

| # | Problem | Category |
|---|---------|---------|
| 1 | [Valid Palindrome II](https://leetcode.com/problems/valid-palindrome-ii) | Two Pointers |
| 2 | [Minimum Remove to Make Valid Parentheses](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses) | Stack |
| 3 | [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view) | BFS |
| 4 | [Random Pick with Weight](https://leetcode.com/problems/random-pick-with-weight) | Binary Search |
| 5 | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k) | HashMap/Prefix Sum |

## Amazon-Specific

| # | Problem | Category |
|---|---------|---------|
| 1 | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements) | Heap |
| 2 | [Reorder Data in Log Files](https://leetcode.com/problems/reorder-data-in-log-files) | Sorting |
| 3 | [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin) | Heap |
| 4 | [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer) | HashMap |
| 5 | [LFU Cache](https://leetcode.com/problems/lfu-cache) | Design |

## AI Company-Specific (Perplexity, Cohere, Scale AI)

For AI/backend roles at AI companies, expect:
- **System design**: RAG systems, vector search, LLM inference infrastructure
- **Production infra coding** (Python/Go) — no LeetCode tricks (Cohere)
- **Concurrency follow-ups**: "Now make it thread-safe" on any solution
- **Debugging rounds**: Given broken code, find and fix it

Key problems for AI roles:
| Problem | Why relevant |
|---------|-------------|
| [LRU Cache](https://leetcode.com/problems/lru-cache) | Semantic cache implementation |
| [Design Twitter](https://leetcode.com/problems/design-twitter) | Feed ranking, rate limiting |
| [Rate Limiter](https://leetcode.com/problems/design-rate-limiter-using-sliding-window-log) | Directly mirrors LLM Cost Guard |
| [Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree) | Token prefix matching |
| [Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words) | Analytics/ranking |

---

## Complexity Quick Reference

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| Binary Search | O(log n) | O(1) | Requires sorted input |
| BFS/DFS | O(V+E) | O(V) | Graph traversal |
| Merge Sort | O(n log n) | O(n) | Stable sort |
| Quick Sort | O(n log n) avg | O(log n) | Unstable, bad pivot = O(n²) |
| Heap Push/Pop | O(log n) | O(n) | heapq in Python = min heap |
| HashMap Get/Put | O(1) avg | O(n) | O(n) worst case collision |
| Dynamic Programming | Varies | Varies | Subproblem count × per-subproblem cost |
| Topological Sort | O(V+E) | O(V+E) | Only for DAGs |
