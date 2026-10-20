# LeetCode 3886 — Sum of Sortable Integers

> **Core idea:** A block can be sorted by cyclic rotation only when its circular sequence has at most one descent. After rotating each block, the values in earlier blocks must also all be no greater than the values in later blocks.

![Language: C](https://img.shields.io/badge/Language-C-A8B9CC?style=flat-square&logo=c&logoColor=white)
![Time: O(n × τ(n))](https://img.shields.io/badge/Time-O(n%C3%97%CF%84(n))-2ea44f?style=flat-square)
![Space: O(n)](https://img.shields.io/badge/Space-O(n)-0366d6?style=flat-square)

## Problem

For every divisor `k` of `n`, split `nums` into consecutive blocks of length `k`. Each block may be cyclically rotated independently. Add every `k` for which the whole array can become non-decreasing.

## Key observations

### 1. Check whether a block can be rotated into sorted order

Count descents around the circular block:

- Every adjacent pair `nums[j - 1] > nums[j]` is a descent.
- Also count the wraparound pair `nums[end] > nums[start]`.

The block can be rotated into non-decreasing order exactly when this circular descent count is at most one.

### 2. Check that the blocks can form one sorted array

Let `prefixMax[i]` be the maximum value in `nums[0..i]`, and `suffixMin[i]` the minimum value in `nums[i..n-1]`.

At every block boundary `s`, all values before the boundary must be no greater than all values from the boundary onward:

`prefixMax[s - 1] <= suffixMin[s]`

Checking this at every boundary ensures the sorted blocks fit together in non-decreasing order.

## Final C solution

```c
int sortableIntegers(int* nums, int numsSize) {
    int n = numsSize;
    int prefixMax[n];
    int suffixMin[n];

    prefixMax[0] = nums[0];
    for (int i = 1; i < n; i++) {
        prefixMax[i] = prefixMax[i - 1] > nums[i]
                     ? prefixMax[i - 1] : nums[i];
    }

    suffixMin[n - 1] = nums[n - 1];
    for (int i = n - 2; i >= 0; i--) {
        suffixMin[i] = suffixMin[i + 1] < nums[i]
                     ? suffixMin[i + 1] : nums[i];
    }

    int answer = 0;

    for (int k = 1; k <= n; k++) {
        if (n % k != 0)
            continue;

        int valid = 1;

        for (int start = 0; start < n; start += k) {
            int end = start + k - 1;
            int descents = 0;

            for (int j = start + 1; j <= end; j++) {
                if (nums[j - 1] > nums[j])
                    descents++;
            }

            if (nums[end] > nums[start])
                descents++;

            if (descents > 1 ||
                (start > 0 && prefixMax[start - 1] > suffixMin[start])) {
                valid = 0;
                break;
            }
        }

        if (valid)
            answer += k;
    }

    return answer;
}
```

## Complexity

Let `τ(n)` be the number of positive divisors of `n`.

- **Time:** `O(n × τ(n))` — each valid divisor may require scanning the array once.
- **Extra space:** `O(n)` — prefix-maximum and suffix-minimum arrays.

## Key learning

Do not check only whether each block is individually rotatable. The boundary condition matters too: even individually sortable blocks cannot produce a globally sorted array if a value in an earlier block is greater than a value in a later block.
