# PDF DSA Pattern and Question Checklist

> Exact coverage companion for **DSA Patterns - Complete C++ Notes**. Every named pattern and worked question from the supplied PDF is listed here. Use the linked template guides for the reusable code; this file tells you the invariant and the first thing to say in an interview.

## How to practise each row

1. Recognise the pattern from the trigger.
2. State the invariant in one sentence.
3. Implement in C++ without looking.
4. Test the listed edge case aloud.

---

## 1. Sliding Window

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Fixed Window Template (maximum sum of size `k`) | contiguous fixed-size range; `sum` equals the current window | add `a[r]`, subtract `a[r-k]` once `r >= k` | `k == n` |
| Longest Substring Without Repeating Characters | substring + unique; window has no duplicate | `unordered_map<char,int>` / last index; shrink until valid | all same character |
| Max Consecutive Ones III | longest range with at most `k` zeros | increment `zeros`, shrink while `zeros > k` | `k == 0` |

**Follow-up:** exactly `k` distinct often becomes `atMost(k) - atMost(k-1)`; do not force an awkward exact-window invariant.

## 2. Two Pointers

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Container With Most Water | maximize `min(height[l], height[r]) * width` | move the shorter wall only | equal heights |
| Remove Element | remove in place, order irrelevant | swap unwanted item with the end or use a write pointer | every item removed |
| 3Sum | sorted values + triples | sort, fix `i`, converge `l/r`, skip duplicates | duplicate triples |

**Why move the shorter wall in Container?** Width always shrinks; keeping the shorter wall cannot improve the limiting height, so it cannot improve area.

## 3. Fast and Slow Pointers

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Linked List Cycle | repeated traversal / cycle | Floyd: `slow += 1`, `fast += 2` | empty / one node |
| Linked List Cycle II (entry) | find where cycle starts | after meeting, reset one pointer to head; move both one step | cycle begins at head |
| Middle of Linked List | one pass, midpoint | fast moves 2; slow moves 1 | even length: define upper/lower middle |

**Reasoning:** once pointers meet in a cycle, the distance from head to entry equals distance from meeting point to entry modulo cycle length.

## 4. In-Place Linked-List Reversal

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Reverse Linked List | reverse pointers | keep `prev`, `cur`, `next` before rewiring | one node |
| Reverse Nodes in k-Group | reverse fixed chunks only when complete | find kth node first; reverse the bounded segment | final chunk shorter than `k` |

**Invariant to say:** `prev` is the already reversed prefix; `cur` is the first node not processed yet.

## 5. HashMap + Linked List

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Copy List with Random Pointer | clone graph-like next/random references | map old node -> clone, then wire references in second pass | random points to self / null |
| LRU Cache | O(1) get and put + recency | `unordered_map<Key, Node*>` + doubly linked list | update existing key / capacity zero |

**Follow-up:** a DLL gives O(1) removal once the map gives the node address. A list alone cannot find an arbitrary key in O(1).

## 6. Monotonic Stack

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Daily Temperatures | next greater value | stack of unresolved indices with decreasing temperatures | no warmer future day |
| Largest Rectangle in Histogram | nearest smaller on both sides | increasing stack of indices; append sentinel height 0 | all increasing bars |

**Why O(n), not O(n^2)?** Each index is pushed once and popped once.

## 7. Stack: Strings and Expressions

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Remove All Adjacent Duplicates in String | cancel neighbouring equal characters | use `string` as a stack; pop equal top | chain reaction after a pop |
| Basic Calculator (`+`, `-`, parentheses) | nested expression | accumulate number; stack previous result/sign at `(` | spaces, nested parentheses, leading `-` |

## 8. DFS

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| DFS Graph Template | connected traversal / components | mark visited before recursive call | disconnected graph |
| Subtree of Another Tree | compare a candidate subtree | `sameTree(a,b)` then DFS every root in main tree | null subtree convention |
| Pacific Atlantic Water Flow | reachability from two boundaries | reverse DFS/BFS from each ocean; intersect visited sets | one-row matrix |

**Follow-up:** reverse the Pacific/Atlantic flow. From an ocean you move to neighbours of equal-or-higher height, avoiding a DFS from every cell.

## 9. BFS

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Tree Level Order Traversal | level-by-level output | capture `q.size()` before processing a level | null root |
| Snakes and Ladders | fewest dice throws | BFS board states; convert square number to row/column carefully | ladder/snake at destination |
| 01 Matrix | nearest source with many sources | enqueue all zeros first; expand outward once | no zeros (define expected output) |

**Why multi-source BFS?** It is equivalent to adding a virtual source connected to every zero. The first visit gives shortest distance in an unweighted graph.

## 10. Topological Sort

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Course Schedule | can prerequisites be completed? | Kahn's: indegree queue; processed count must equal `n` | disconnected courses |
| Course Schedule II | return valid prerequisite order | append nodes as indegree becomes zero | cycle -> empty answer |

**Reasoning:** a node with indegree zero has no unfulfilled prerequisite, so it is safe to schedule next.

## 11. Union-Find / DSU

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| DSU Template | repeated merge/connectivity checks | parent + size/rank; path-compressed `find` | union already-connected nodes |
| Number of Islands (DSU) | grid connectivity, merge adjacent land | map `(r,c)` to `r*cols+c`, decrement island count on successful union | water-only grid |

**Complexity:** amortized near O(1), formally O(alpha(n)), with path compression + union by size/rank.

## 12. Binary Search

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Classic Binary Search + Boundaries | sorted data / first or last index | lower/upper-bound template | target absent |
| Koko Eating Bananas | minimum rate that meets deadline | first-true binary search; `hours += ceil(pile / rate)` | one pile / huge `h` |
| Search a 2D Matrix | globally sorted matrix | treat index as `row = mid / cols`, `col = mid % cols` | one row / one column |

See [Maths, Bits & Binary Search](dsa_math_binary_companion.md) for full C++ templates, proof of monotonicity, and more binary-search-on-answer questions.

## 13. Intervals

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Merge Intervals | overlapping ranges | sort by start; merge when `next.start <= current.end` | touching intervals: clarify whether they merge |
| Insert Interval | one new range into sorted disjoint ranges | add before, merge overlaps, append after | insertion at both ends |

## 14. 1D Dynamic Programming

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Maximum Subarray (Kadane) | best contiguous sum | `bestEnding = max(x, bestEnding + x)` | all negative values |
| House Robber | max value with no adjacent picks | `dp[i] = max(dp[i-1], dp[i-2]+a[i])` | one/two houses |
| Coin Change | minimum coins to make amount | `dp[0]=0`, iterate amounts, unreachable = INF | impossible amount |

**DP explanation:** define the state in words first; then show the transition only uses smaller solved states.

## 15. 2D Dynamic Programming

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Longest Common Subsequence | two sequences / preserve order | `dp[i][j]` is LCS of prefixes | empty string |
| Edit Distance | min insert/delete/replace | `dp[i][j]` min edits for prefixes | equal strings / one empty |

**Transition for mismatch:** `1 + min(replace dp[i-1][j-1], delete dp[i-1][j], insert dp[i][j-1])`.

## 16. State-Machine DP

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Best Time to Buy and Sell Stock with Cooldown | buy/sell actions with delayed state | track `hold`, `sold`, `rest` and update from old values | one day / decreasing prices |

**Reasoning:** a state machine makes constraints explicit: you cannot buy the day after selling because the only next state is cooldown/rest.

## 17. Backtracking

| PDF question | Trigger / invariant | C++ move | Edge case |
|---|---|---|---|
| Permutations | all orderings | choose unused item, recurse, undo | duplicate values: sort + skip duplicates |
| Combination Sum | choices can repeat, target sum | recurse from same index after choosing | target zero |
| Word Search | path in a grid without reuse | mark cell visited, explore 4 dirs, restore cell | one-cell word |

**Universal invariant:** `path` contains only decisions currently active; every mutation is undone before the recursive call returns.

---

## Master PDF Pattern Checklist

- [x] Sliding Window
- [x] Two Pointers
- [x] Fast & Slow Pointers
- [x] In-Place Reversal of Linked List
- [x] Linked List + HashMap
- [x] Monotonic Stack
- [x] Stack / Expression Parsing
- [x] DFS
- [x] BFS / Multi-source BFS
- [x] Topological Sort
- [x] Union-Find
- [x] Binary Search: classic, boundary, answer, and 2D
- [x] Merge / Insert Intervals
- [x] 1D DP
- [x] 2D DP
- [x] State-machine DP
- [x] Backtracking
- [x] Complexity and interview-trick coverage through [DSA Pattern Guide](patterns.md) and [Maths, Bits & Binary Search](dsa_math_binary_companion.md)
