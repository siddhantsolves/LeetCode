# LeetCode 169 — Majority Element

## Problem

Given an integer array `nums` of size `n`, return the element that appears more than `⌊n / 2⌋` times.

The problem guarantees that a majority element exists.

## Initial approach: brute-force counting

The original approach was to consider each element as a possible answer and count how many times it appears by scanning the array again. As soon as an element's count is greater than `n / 2`, return it.

This is easy to reason about and works correctly, but it repeats the same comparisons many times.

- **Time complexity:** O(n²)
- **Space complexity:** O(1)

## Moving to Boyer–Moore Voting Algorithm

The key observation is that the majority element occurs more often than every other value combined. Pairing one majority-element occurrence with one non-majority occurrence can never eliminate all occurrences of the majority element.

Maintain:

- a **candidate** `x`
- a **counter** `c`

For each number:

1. If it matches the candidate, increment the counter.
2. Otherwise, decrement the counter, treating the two different values as a cancelled pair.
3. When the counter becomes zero, choose the current number as the new candidate and reset the counter to one.

Because a majority element is guaranteed, the final candidate is the answer.

## Final C solution

```c
int majorityElement(int* nums, int numsSize) {
    int i,x=nums[0],c=1;
    for(i=1;i<numsSize;i++){
        if(nums[i]==x){
            c++;
        }
        else{
            c--;
        }
        if(c==0){
            x=nums[i];
            c=1;
        }
    }
    return x;
}
```

## Complexity

- **Time complexity:** O(n) — one pass through the array.
- **Space complexity:** O(1) — only the candidate, counter, and loop index are stored.

## How the algorithm was derived

The solution began with the direct O(n²) idea: count the frequency of each number and check whether it exceeds half the array length. The improvement came from recognizing that different values can cancel each other out in pairs. Since the majority element appears more than half the time, it survives every possible cancellation. This leads naturally to keeping just one running candidate and a count instead of maintaining many frequency counts.

## Key learnings

- Start with a correct brute-force solution; it reveals what information is actually needed.
- A guaranteed majority element is a powerful condition that enables Boyer–Moore Voting.
- “Cancelling” different pairs is a useful way to reduce a frequency problem to constant space.
- Boyer–Moore returns a candidate. If a majority element were **not** guaranteed, a second pass would be needed to verify its count.
