# LeetCode 2916 — Subarrays Distinct Element Sum of Squares II

> **Core idea:** process subarrays by their right endpoint. When a new value is appended, only a contiguous range of starting positions gains one new distinct value.

![Language: C](https://img.shields.io/badge/Language-C-A8B9CC?style=flat-square&logo=c&logoColor=white)
![Time: O(n log n)](https://img.shields.io/badge/Time-O(n_log_n)-2ea44f?style=flat-square)
![Space: O(n)](https://img.shields.io/badge/Space-O(n)-0366d6?style=flat-square)

## Problem

For every non-empty contiguous subarray, find its number of distinct values, square that count, and add all results together.

Return the answer modulo <code>1,000,000,007</code>.

This is the large-constraint version of Problem 2913: <code>n</code> can be as large as <code>100,000</code>, so enumerating all O(n²) subarrays is too slow.

---

## How to reach the optimized idea

### Start from the easy version

In Problem I, choosing every left endpoint and expanding the right endpoint works because <code>n ≤ 100</code>. For each subarray, maintain its frequency table and its distinct count.

That takes **O(n²)** time. With <code>n = 100,000</code>, it is impossible: there are roughly five billion subarrays.

### Change the viewpoint: fix the right endpoint

Process the array from left to right. At index <code>r</code>, consider every subarray ending at <code>r</code>:

<code>nums[1..r], nums[2..r], ..., nums[r..r]</code>

For each possible starting position <code>l</code>, store:

<code>count[l] = distinct values in nums[l..r]</code>

When the new value <code>x = nums[r]</code> arrives, suppose its previous occurrence was at position <code>p</code>.

- If <code>l ≤ p</code>, the subarray already contains <code>x</code>; its distinct count does not change.
- If <code>p &lt; l ≤ r</code>, the subarray did not contain <code>x</code>; its distinct count increases by one.

So one new element becomes a **range increment**:

<code>count[p + 1 .. r] += 1</code>

~~~mermaid
flowchart TD
    A([Process next right endpoint]) --> B["Find previous position of current value"]
    B --> C["Add 1 to all affected start positions"]
    C --> D["Update sum and sum of squares for that range"]
    D --> E["Root sum of squares = contribution of all subarrays ending here"]
    E --> F["Add root sum of squares to answer"]
    F --> G["Record current position as last occurrence"]
    G --> H{"More elements?"}
    H -- Yes --> A
    H -- No --> I([Return answer modulo M])
~~~

---

## Why a segment tree is needed

We need two things after each range increment:

1. The sum of the current distinct counts.
2. The sum of their squares.

For one tree segment, store:

| Stored value | Meaning |
| --- | --- |
| <code>sum</code> | sum of all <code>count[l]</code> values in the segment |
| <code>sq</code> | sum of all <code>count[l] × count[l]</code> values in the segment |
| <code>lazy</code> | pending range increment |

If every count in a segment increases by <code>d</code>:

<code>(x + d)² = x² + 2dx + d²</code>

Therefore:

<code>newSq = oldSq + 2 × d × oldSum + length × d²</code>

and:

<code>newSum = oldSum + length × d</code>

That makes each range update **O(log n)**.

---

## Worked example

For <code>nums = [1, 2, 1]</code>:

| Right endpoint | Current value | Previous position | Updated start-position range | Distinct counts for subarrays ending here | Contribution |
| :---: | :---: | :---: | --- | --- | :---: |
| 1 | 1 | 0 | [1..1] | [1] | 1 |
| 2 | 2 | 0 | [1..2] | [2, 1] | 4 + 1 = 5 |
| 3 | 1 | 1 | [2..3] | [2, 2, 1] | 4 + 4 + 1 = 9 |

**Total: 1 + 5 + 9 = 15**

---

## Reference C implementation

> No prior submission was provided for this problem, so this is a reference implementation of the optimized approach.

~~~c
#include <stdlib.h>

#define MOD 1000000007LL

typedef long long ll;

typedef struct {
    ll sum;
    ll sq;
    ll lazy;
} Node;

static void apply(Node *tree, int node, int left, int right, ll add) {
    ll length = right - left + 1;
    ll oldSum = tree[node].sum;

    tree[node].sq = (tree[node].sq
        + 2LL * add * oldSum
        + length * add * add) % MOD;

    tree[node].sum = (tree[node].sum + length * add) % MOD;
    tree[node].lazy = (tree[node].lazy + add) % MOD;
}

static void push(Node *tree, int node, int left, int right) {
    int mid;
    ll pending;

    if (left == right || tree[node].lazy == 0) {
        return;
    }

    mid = left + (right - left) / 2;
    pending = tree[node].lazy;

    apply(tree, node * 2, left, mid, pending);
    apply(tree, node * 2 + 1, mid + 1, right, pending);

    tree[node].lazy = 0;
}

static void rangeAdd(Node *tree, int node, int left, int right,
                     int queryLeft, int queryRight) {
    int mid;

    if (queryRight < left || right < queryLeft) {
        return;
    }

    if (queryLeft <= left && right <= queryRight) {
        apply(tree, node, left, right, 1);
        return;
    }

    push(tree, node, left, right);

    mid = left + (right - left) / 2;
    rangeAdd(tree, node * 2, left, mid, queryLeft, queryRight);
    rangeAdd(tree, node * 2 + 1, mid + 1, right, queryLeft, queryRight);

    tree[node].sum = (tree[node * 2].sum + tree[node * 2 + 1].sum) % MOD;
    tree[node].sq = (tree[node * 2].sq + tree[node * 2 + 1].sq) % MOD;
}

int sumCounts(int* nums, int numsSize) {
    Node *tree = calloc(4 * numsSize + 5, sizeof(Node));
    int *last = calloc(numsSize + 1, sizeof(int));
    ll answer = 0;

    for (int right = 1; right <= numsSize; right++) {
        int value = nums[right - 1];
        int left = last[value] + 1;

        rangeAdd(tree, 1, 1, numsSize, left, right);

        answer = (answer + tree[1].sq) % MOD;
        last[value] = right;
    }

    free(tree);
    free(last);

    return (int)answer;
}
~~~

## Complexity

| Time | Space | Why |
| :---: | :---: | --- |
| **O(n log n)** | **O(n)** | Each index produces one range update and one segment-tree root read. The tree and last-occurrence array are linear in size. |

## Key learnings

- When O(n²) subarray enumeration is too slow, group subarrays by one shared endpoint.
- Track how adding one element changes many subarrays at once.
- The last occurrence divides starting positions into unchanged and incremented ranges.
- A lazy segment tree can maintain both a range sum and a range sum of squares.
- The identity <code>(x + 1)² = x² + 2x + 1</code> is the bridge between the problem statement and the segment-tree state.
