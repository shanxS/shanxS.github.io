---
layout: post
title:  "Intersection of Two Arrays - some comments"
date:   2018-08-10 18:16:10 -0700
categories: coding
---


For algorithmic question below, aka intersection of 2 arrays, evaluate `hashtable` vs `sort+search` approach

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2,2]
```

```
Let length of smaller array = SMALL
Let length of bigger array = BIG
```

**tl;dr**
From evaluation of theoretical time complexity`Hashtable` solution will be slower than `sort + search` solution if `BIG + SMALL` < `(BIG + 1)log2(SMALL)` use hashtable else sort+search.
Which basically boils down to `SMALL <= 2`, you can evaluate this claim at: https://www.desmos.com/calculator/p5gp3ak6uy


**Details**

**case 1**
Let's start with case where we know that smaller array is sorted.
All we have to do is search each element in of bigger array in smaller array.
```
Time complexity = O(BIG * log2(SMALL))
Space complexity = O(1)
```
For same case let's put smaller array in a HashSet and then search all elements of bigger array in smaller one.
```
Time complexity = O(BIG + SMALL)
Space complexity = O(SMALL)
```
Let's assume BIG = 10. Pick up any online graphing calculator and plot:
```
y = 10 * log2(x)
y = 10 + x
```
Don't forget the fact that log is base 2 and SMALL <= BIG.

You'll find that using `sort+search` is better for some values of SMALL.

**case 2**
Now, consider 2 unsorted arrays of different lengths.
From case 1, we know using hashset has following complexities:
```
Time complexity = O(BIG + SMALL)
Space complexity = O(SMALL)
```
But, if we sort the smaller array and then search in it complexities are:
```
Time complexity to sort = O(log(SMALL))
Time complexity to search = O(BIG * log(SMALL))
Total time complexity = O((BIG + 1) * log(SMALL))

Total space complexity = O(1)
```
If we repeat the plotting experiment from case 1 with different values of BIG, `sort+search` is better than `hashtable` only if BIG <= 2.


***corollary***
 Boundary condition is (I don't know how to solve this algebraically, I used graphing calculator):
```
k + x = (k + 1)log2(x)
so, (k + 1)log2(x) - (k + x) < 0 such that x <= k
where k is constant and is equal to size of bigger array.
x is variable and is size of smaller array.

Graph for above equation: https://www.desmos.com/calculator/p5gp3ak6uy
```

Please correct me if I am wrong.