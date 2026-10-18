# LeetCode 2913 — Subarrays Distinct Element Sum of Squares I

> **Core idea:** fix the left boundary, expand the right boundary, and update the distinct-element count as each new value is added.

![Language: C](https://img.shields.io/badge/Language-C-A8B9CC?style=flat-square&logo=c&logoColor=white)
![Time: O(n²)](https://img.shields.io/badge/Time-O(n%C2%B2)-f59e0b?style=flat-square)
![Space: O(1)](https://img.shields.io/badge/Space-O(1)-0366d6?style=flat-square)

## Problem

For every non-empty contiguous subarray of <code>nums</code>:

1. Count its distinct values.
2. Square that count.
3. Add the result to the answer.

Return the final sum.

The constraints are small: <code>nums.length ≤ 100</code> and <code>1 ≤ nums[i] ≤ 100</code>. That makes direct subarray enumeration a practical solution.

---

## Approach — expand every starting point

Choose a starting index <code>i</code>. Then expand the ending index <code>j</code> from <code>i</code> to the end of the array.

For each new element:

- If its frequency is zero, it is a new distinct value, so increase <code>distinct</code>.
- Increase its frequency.
- Add <code>distinct × distinct</code> to <code>answer</code>.

The frequency array is reset for each new starting index, so it always represents the current subarray <code>nums[i..j]</code>.

~~~mermaid
flowchart TD
    A([Choose start index i]) --> B[Reset frequency array<br/>and distinct = 0]
    B --> C([Extend end index j])
    C --> D{Has nums[j] appeared<br/>in this subarray?}
    D -- No --> E[Increase distinct]
    D -- Yes --> F[Keep distinct unchanged]
    E --> G[Increase frequency of nums[j]]
    F --> G
    G --> H[Add distinct² to answer]
    H --> I{More end indices?}
    I -- Yes --> C
    I -- No --> J{More start indices?}
    J -- Yes --> A
    J -- No --> K([Return answer])
~~~

---

## Worked example

For <code>nums = [1, 2, 1]</code>:

| Start <code>i</code> | Current subarray | Distinct values | Added to answer |
| :---: | --- | :---: | :---: |
| 0 | [1] | 1 | 1² = 1 |
| 0 | [1, 2] | 2 | 2² = 4 |
| 0 | [1, 2, 1] | 2 | 2² = 4 |
| 1 | [2] | 1 | 1² = 1 |
| 1 | [2, 1] | 2 | 2² = 4 |
| 2 | [1] | 1 | 1² = 1 |

**Total: 1 + 4 + 4 + 1 + 4 + 1 = 15**

---

## Final C solution

~~~c
int sumCounts(int* nums, int numsSize) {
    int answer = 0;

    for (int i = 0; i < numsSize; i++) {
        int freq[101] = {0};
        int distinct = 0;

        for (int j = i; j < numsSize; j++) {
            if (freq[nums[j]] == 0) {
                distinct++;
            }

            freq[nums[j]]++;

            answer += distinct * distinct;
        }
    }

    return answer;
}
~~~

## Why <code>freq[101]</code> works

The problem guarantees that every value in <code>nums</code> is between 1 and 100. The array index is therefore a direct lookup for each number's frequency.

---

## Complexity

| Time | Space | Why |
| :---: | :---: | --- |
| **O(n²)** | **O(1)** | There are O(n²) subarrays. The frequency array has a fixed size of 101 because the input values are bounded. |

## Key learnings

- Enumerating all subarrays is often enough when the input limit is small.
- Reuse work while expanding a subarray: update the distinct count instead of recounting it from scratch.
- A bounded value range can replace a hash map with a simple fixed-size frequency array.
- The key update is: only increase <code>distinct</code> when the incoming value's previous frequency is zero.
