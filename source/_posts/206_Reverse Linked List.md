---
title: "[Leetcode] #206_Reverse Linked List"
date: 2021-08-06 15:21:16
updated: 2021-11-30 15:21:16
tags: [Leetcode, EASY]
categories:
  - Leetcode Solutions
---

![example1](https://miro.medium.com/max/1084/0*KbEa1WlFKRnl9VWb.jpg)

[206_Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) belongs to the EASY category. I started learning about Data Structures this month and have never implemented an actual linked list on a problem before. So this one is quite challenging and fun to do. In this post I’m only covering the iterative solution.

<!-- more -->
### Problem description:

Given the head of a singly linked list, reverse the list, and return the reversed list.
Example 1:

![example1](https://miro.medium.com/max/1084/0*KbEa1WlFKRnl9VWb.jpg)

```
Input: head = [1,2,3,4,5]
Output: [5,4,3,2,1]
```

Example 2:

![example2](https://miro.medium.com/max/364/0*2rQmJi_cw_BBTxTG.jpg)

```
Input: head = [1,2]
Output: [2,1]
```

Example 3:

```
Input: head = []
Output: []
```

Constraints:
```
The number of nodes in the list is the range [0, 5000].
-5000 <= Node.val <= 5000
```
Follow up: A linked list can be reversed either iteratively or recursively. Could you implement both?


### Breaking down the problem in my own words:

- Our function takes in one parameter : the head of a singly linked list
- Our function should be able to reverse the linked list and return the new head
- There are two ways to do it: iteratively or recursively
- They also provide the definition of a singly linked list in the online editor:

```
* Definition for singly-linked list.
* function ListNode(val, next) {
*     this.val = (val===undefined ? 0 : val)
*     this.next = (next===undefined ? null : next)
* }
```

To my understanding, a singly linked list consists of nodes pointing to their next node with one-direction pointers, which means a node cannot be pointing to another node in two different directions at the same time:

![singlylinkedlist](https://miro.medium.com/max/1400/0*OLVK8DiyhWziU9ky.png)
source: JavaScript Algorithms and Data Structures Masterclass

And with the constructor function ListNode(val, next) provided, we can assign a value to the next node with this.next

#### Solution 1 — Iterative
In order to “reverse” the linked list, we can remain the order of the nodes but change the direction of the pointer to pointing backwards:

![node1to2](https://miro.medium.com/max/364/0*2rQmJi_cw_BBTxTG.jpg)

- Originally we have node 1 pointing to node 2, (node 2 pointing to null) like 1 -> 2 -> null. What we’re gonna do here is to make node 2 point backwards to node 1, (and node 1 points to null). 2 -> 1 -> null
- We need to change node 1’s pointer direction, and let node 1 (which is our current head) point to null. But if doing so without storing the node 2 (head.next) value beforehand, we would end up not being able to access it later.
- So now we are iterating through the list, we need 3 variables to keep track of where we are in the process: previousNode , currentNode , and nextNode
- Then in every iteration we update these three variables:
1. start from assigning head to currentNode
2. store the next node of currentNode in nextNode
3. reverse pointer direction of currentNode, from originally pointing to nextNode to now pointing to previousNode
4. now there’s nothing pointing to nextNode , also the pointer was gone
5. now make currentNode our previousNode
6. and make nextNode our currentNode
7. move on to a new loop of step 2.~6. until we reach the point when currentNode === null
8. break from the loop and return previousNode (the tail of the original linked list, which is now the new head)

```
const reverseList = function(head) {
    let previousNode = null
    let currentNode = head
    let nextNode = null
    
    while (currentNode !== null) {
        // store next node
        nextNode = currentNode.next
        // reverse pointer direction
        currentNode.next = previousNode
        // cut off pointer from currentNode to nextNode
        // good thing we have saved nextNode for later access
        // move previous node to current node
        previousNode = currentNode
        // move current node to nextNode that we've saved earlier
        currentNode = nextNode
    }
    return previousNode
}
```

![submission1](https://user-images.githubusercontent.com/51830382/128460640-bce0c0ce-4428-4ecf-b0a9-46082a691f40.png)


Results:  
Runtime: 72 ms  
Memory Usage: 41.1 MB  
