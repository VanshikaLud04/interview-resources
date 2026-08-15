# DSA Maths, Bit Manipulation, and Binary Search Companion

> Use this with [DSA Pattern Guide](patterns.md). The source PDF's core patterns are already covered there; this companion adds the high-frequency maths, bit, and binary-search reasoning that interview prep often leaves too thin.

## 1. Pattern Recognition in 20 Seconds

| Signal in the question | Reach for | First question to ask yourself |
|---|---|---|
| Sorted array / first or last occurrence | Binary search | What is the monotonic boundary? |
| "Minimum possible maximum" / "maximum possible minimum" | Binary search on answer | Can I write `feasible(x)` that changes only once? |
| Power of two, subset, XOR, one odd element | Bits | Is each bit independent or does XOR cancellation help? |
| Divisibility, fractions, coordinates, repeated cycles | Maths | Do `gcd`, modular arithmetic, or overflow change the approach? |
| Count subarrays / pairs with a numeric target | Prefix sum / combinatorics | Can a prefix or complement encode past work? |

**Interview habit:** say the invariant before code. Example: "`feasible(x)` is true for all capacities at or above the answer, so I can binary-search the first true capacity."

---

## 2. Binary Search: Three Templates You Should Know

### A. Exact search

```cpp
int binarySearch(const vector<int>& a, int target) {
    int lo = 0, hi = (int)a.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2; // avoids overflow
        if (a[mid] == target) return mid;
        if (a[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

### B. First index satisfying a predicate (lower bound)

Use when the answer is a boundary: first `>= target`, first bad version, minimum valid capacity.

```cpp
int firstTrue(int lo, int hi) { // answer is known to lie in [lo, hi]
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

### C. Last index satisfying a predicate

```cpp
int lastTrue(int lo, int hi) {
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2; // upper mid prevents infinite loop
        if (feasible(mid)) lo = mid;
        else hi = mid - 1;
    }
    return lo;
}
```

### Binary search on answer: the reusable recipe

1. Identify a numeric answer range, not an array index range.
2. Write a greedy or linear `feasible(x)` checker.
3. Prove monotonicity: `false false false true true`, or the reverse.
4. Pick first-true or last-true template.
5. State total complexity: `O(log(range) * cost(feasible))`.

| Problem | Search space | `feasible(x)` | Goal |
|---|---|---|---|
| Koko Eating Bananas | speed `1..maxPile` | finishes within `h` hours | first true |
| Capacity To Ship Packages | capacity `maxWeight..sum` | ships in `d` days | first true |
| Aggressive Cows | distance `1..max-min` | place all cows distance apart | last true |
| Split Array Largest Sum | max subarray sum | split into at most `m` parts | first true |
| Median / kth in sorted arrays | partition position | left side has correct size/order | binary-search smaller array |

### Follow-ups interviewers ask

**Why not sort and scan?** If one check is linear but the answer range is large, binary search makes `O(n log range)`; scan is `O(n * range)`.

**How do you know it is monotonic?** Test two values. For shipping, if capacity `C` succeeds, any capacity larger than `C` can mimic the same shipment groups, so it also succeeds.

**What are common bugs?** Wrong bounds, using `<=` where `lo < hi` is expected, a non-progressing midpoint, and returning `mid` rather than the converged boundary.

---

## 3. Bit Manipulation: The Small Set of Laws That Matter

| Operation | Meaning | Useful fact |
|---|---|---|
| `x & 1` | last bit | odd iff result is 1 |
| `x & (x - 1)` | clears lowest set bit | zero iff `x` has exactly one set bit (for `x > 0`) |
| `x & -x` | isolates lowest set bit | useful in Fenwick trees / masks |
| `x ^ x` | 0 | pairs cancel under XOR |
| `x ^ 0` | x | identity for XOR |
| `x << k` | multiply by `2^k` | watch signed overflow |
| `x >> k` | divide by `2^k` for non-negative x | signed right shift is language-sensitive |

### Core templates

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

int countSetBits(int n) {
    int count = 0;
    while (n) {
        n &= (n - 1); // removes one set bit
        ++count;
    }
    return count;
}

int singleNumber(const vector<int>& nums) {
    int ans = 0;
    for (int x : nums) ans ^= x;
    return ans;
}
```

### Common patterns

| Pattern | Idea | Representative problem |
|---|---|---|
| Single unique value, all others twice | XOR all values | Single Number |
| Two unique values, all others twice | split by a set bit in `xorAll` | Single Number III |
| Subsets | bitmask from `0` to `(1<<n)-1` | Subsets |
| State of a small alphabet / visited set | one bit per feature | Valid Words for Puzzles |
| Count bits `0..n` | `dp[i] = dp[i >> 1] + (i & 1)` | Counting Bits |

### Bitmask subset template

```cpp
vector<vector<int>> subsets(const vector<int>& nums) {
    int n = nums.size();
    vector<vector<int>> out;
    for (int mask = 0; mask < (1 << n); ++mask) {
        vector<int> cur;
        for (int i = 0; i < n; ++i)
            if (mask & (1 << i)) cur.push_back(nums[i]);
        out.push_back(cur);
    }
    return out;
}
```

**Follow-up: Why does XOR solve "every element twice except one"?** XOR is associative, commutative, and `x ^ x = 0`; equal pairs disappear and the unique value remains.

**Follow-up: When not to use bitmasks?** `2^n` is only practical for small `n` (roughly `n <= 20–25`, depending on work per subset). Also use unsigned types when shifting near the sign bit.

---

## 4. Interview Maths Toolkit

### GCD and LCM

```cpp
long long gcdll(long long a, long long b) {
    while (b) { long long t = a % b; a = b; b = t; }
    return a;
}

long long lcmll(long long a, long long b) {
    return a / gcdll(a, b) * b; // divide first to reduce overflow risk
}
```

- `gcd(a, b)` appears in fraction reduction, repeated cycles, grid-line points, and making equal-sized groups.
- `lcm(a, b)` appears in periodic schedules and "when do both repeat together?"

### Prime checks and sieve

```cpp
bool isPrime(int n) {
    if (n < 2) return false;
    for (int d = 2; d <= n / d; ++d) // safer than d*d <= n
        if (n % d == 0) return false;
    return true;
}

vector<bool> sieve(int n) {
    vector<bool> prime(n + 1, true);
    if (n >= 0) prime[0] = false;
    if (n >= 1) prime[1] = false;
    for (int p = 2; p <= n / p; ++p)
        if (prime[p])
            for (long long multiple = 1LL * p * p; multiple <= n; multiple += p)
                prime[multiple] = false;
    return prime;
}
```

Use trial division for one number; use a sieve when you need facts for every number up to `n`.

### Modular arithmetic

For `MOD = 1e9 + 7`:

```cpp
long long modPow(long long base, long long exp, long long mod) {
    long long ans = 1;
    base %= mod;
    while (exp) {
        if (exp & 1) ans = ans * base % mod;
        base = base * base % mod;
        exp >>= 1;
    }
    return ans;
}
```

Rules: reduce after addition/multiplication; normalize subtraction with `(a - b + MOD) % MOD`; **do not divide modulo `MOD` normally**. If `MOD` is prime and denominator is not divisible by it, multiply by modular inverse `x^(MOD-2) mod MOD`.

### Combinatorics you should recognise

| Question shape | Formula / approach |
|---|---|
| Choose `r` unordered items from `n` | `nCr = n! / (r!(n-r)!)` |
| Number of pairs among `n` items | `n(n-1)/2` |
| Every item combines with every item in another group | `a*b` |
| Count paths with only right/down moves | `C(m+n-2, m-1)` |
| Permutations of duplicates | `n! / (c1! c2! ...)` |

For coding questions, prefer a DP solution unless constraints require factorials + modular inverses; it is easier to explain and less error-prone.

### Geometry essentials

```cpp
long long cross(long long ax, long long ay, long long bx, long long by) {
    return ax * by - ay * bx;
}
// For A->B and A->C: cross(B-A, C-A) > 0 means C is to the left.
```

- Squared distance avoids floating point: `(x1-x2)^2 + (y1-y2)^2`.
- Cross product determines turn direction / collinearity.
- For points on a line segment between integer lattice endpoints: `gcd(abs(dx), abs(dy)) + 1` points including endpoints.

### Numeric safety checklist

1. Use `long long` for sums, products, coordinates, and binary-search mid calculations when constraints might exceed `2^31-1`.
2. Avoid `pow` for integer results; rounding can surprise you.
3. State whether negatives are allowed before applying modulo or bit tricks.
4. For doubles, compare with epsilon; for exact comparisons, avoid doubles entirely.

---

## 5. High-Return Practice Set

| Topic | Problems to solve | What to explain aloud |
|---|---|---|
| Binary search | First/Last Position, Koko, Ship Packages, Median of Two Sorted Arrays | boundary + monotonicity |
| Bits | Single Number I/II/III, Counting Bits, Subsets, Maximum XOR | why each bit operation is valid |
| Maths | GCD of Strings, Count Primes, Pow(x,n), Happy Number, Rotate Function | overflow and complexity |
| Prefix / numeric | Subarray Sum Equals K, Product Except Self, Continuous Subarray Sum | invariant and map meaning |

## 6. Final Interview Drill

Before coding any numeric problem, say:

> "I’ll first check constraints and whether I need 64-bit arithmetic. Then I’ll identify the invariant: a sorted boundary, a monotonic feasibility function, or a property that can be represented bit-by-bit. I’ll test the approach on a small edge case before implementation."

That sounds structured, prevents most bugs, and gives the interviewer a way to follow your reasoning.
