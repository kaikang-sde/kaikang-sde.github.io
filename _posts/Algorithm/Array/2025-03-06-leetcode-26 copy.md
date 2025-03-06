---
title: LeetCode 26 - Remove Duplicates from Sorted Array
date: 2025-03-06
categories: [Algorithm, Array]
tags: [Array, Java, LeetCode, Two Pointers]
author: kai
---

# LeetCode 26 - Remove Duplicates from Sorted Array

link: [26 - Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/).

## Description - simple version
- an array sorted in non-decreasing order
- remove the duplicates in-place
- same relative order of the elements
- return the number of the unique elements, not the index

### Example

```
Input: nums = [1,1,2]
Output: 2, nums = [1,2,_]
Explanation: Your function should return k = 2, with the first two elements of nums being 1 and 2 respectively.
It does not matter what you leave beyond the returned k (hence they are underscores).
```

### Process
![Remove Duplicates from Sorted Array](/assets/img/posts/Algorithm/Array/LC26.png)

## Code example - Jave
- time complexity - O(N)
- space complexity - O(1)

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums == null || nums.length < 1) {
            return 0;
        }

        int right = 1;
        int left = 0;
        while (right < nums.length) {
            if (nums[left] != nums[right]) {
                nums[++left] = nums[right];
            } 
            right++;
        }
        return left + 1;  
    }
}
```

---

Stay tuned for more insights into algorithm fundamentals!