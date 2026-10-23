# LeetCode 206 — Reverse Linked List

## Problem

Given the head of a singly linked list, reverse the list and return the new head.

## Approach: iterative pointer reversal

Walk through the list once while maintaining three pointers:

1. `prev` holds the already-reversed portion.
2. `curr` is the node currently being processed.
3. `next` temporarily saves the original next node before the link is reversed.

For each node, redirect `curr->next` to `prev`, then advance all pointers. When the traversal ends, `prev` is the head of the reversed list.

## Final C solution

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* reverseList(struct ListNode* head) {
    struct ListNode *prev = NULL;
    struct ListNode *curr = head;
    struct ListNode *next = NULL;

    while (curr != NULL) {
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }

    return prev;
}
```

## Complexity

- **Time complexity:** O(n) — every node is visited once.
- **Space complexity:** O(1) — only three pointers are used.
