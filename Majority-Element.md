# LeetCode 169 — Majority Element

> **Core idea:** when different values cancel each other in pairs, the majority element is the one left standing.

![Language: C](https://img.shields.io/badge/Language-C-A8B9CC?style=flat-square&logo=c&logoColor=white)
![Time: O(n)](https://img.shields.io/badge/Time-O(n)-2ea44f?style=flat-square)
![Space: O(1)](https://img.shields.io/badge/Space-O(1)-0366d6?style=flat-square)

## Problem

Given an integer array <code>nums</code> of size <code>n</code>, return the element that appears more than <code>⌊n / 2⌋</code> times.

The problem guarantees that a majority element exists.

---

## The journey: from brute force to a one-pass solution

### 1) Initial approach — brute-force counting

The original approach was to treat each element as a possible answer and count its occurrences by scanning the array again. As soon as an element's count is greater than <code>n / 2</code>, return it.

✅ Straightforward and correct  
⚠️ Repeats the same comparisons many times

| Time | Space |
| :---: | :---: |
| **O(n²)** | **O(1)** |

### 2) Key observation — cancel different pairs

A majority element appears **more times than all other values combined**. So, if one occurrence of the majority element is paired with one different value, removing that pair cannot remove the majority completely.

~~~text
[2, 2, 1, 1, 1, 2, 2]

2 ↔ 1   cancel
2 ↔ 1   cancel
2 ↔ 1   cancel
2       survives
2       survives

Majority element: 2
~~~

This is the intuition behind the **Boyer–Moore Voting Algorithm**.

---

## Boyer–Moore Voting Algorithm

Keep only two pieces of information:

| Variable | Meaning |
| --- | --- |
| <code>x</code> | current candidate |
| <code>c</code> | its current vote count |

~~~mermaid
flowchart TD
    A([Read next number]) --> B{Does it match<br/>the candidate?}
    B -- Yes --> C[Increase count]
    B -- No --> D[Decrease count]
    C --> E{More numbers?}
    D --> F{Count is zero?}
    F -- Yes --> G[Set current number<br/>as candidate and count = 1]
    F -- No --> E
    G --> E
    E -- Yes --> A
    E -- No --> H([Candidate is the majority element])
~~~

### Worked example

For <code>[2, 2, 1, 1, 1, 2, 2]</code>:

| Number | Candidate <code>x</code> | Count <code>c</code> | What happened |
| :---: | :---: | :---: | --- |
| 2 | 2 | 1 | Start with 2 |
| 2 | 2 | 2 | Same candidate → increment |
| 1 | 2 | 1 | Different value → cancel one vote |
| 1 | 1 | 1 | Count reached 0 → choose 1 |
| 1 | 1 | 2 | Same candidate → increment |
| 2 | 1 | 1 | Different value → cancel one vote |
| 2 | 2 | 1 | Count reached 0 → choose 2 |

🎯 Final candidate: **2**

---

## Final C solution

~~~c
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
~~~

## Complexity

| Time | Space | Why |
| :---: | :---: | --- |
| **O(n)** | **O(1)** | One pass through the array; only the candidate, counter, and loop index are stored. |

---

## How the algorithm was derived

The solution began with the direct **O(n²)** idea: count the frequency of each number and check whether it exceeds half the array length. The improvement came from recognizing that different values can cancel each other in pairs. Since the majority element appears more than half the time, it survives every possible cancellation.

That insight replaces repeated frequency counting with just one running candidate and one counter.

## Key learnings

- Start with a correct brute-force solution; it reveals what information is actually needed.
- A guaranteed majority element is the condition that makes Boyer–Moore work.
- “Cancelling” different pairs can turn a frequency problem into a constant-space solution.
- Boyer–Moore returns a **candidate**. If a majority element were **not** guaranteed, use a second pass to verify its count.
