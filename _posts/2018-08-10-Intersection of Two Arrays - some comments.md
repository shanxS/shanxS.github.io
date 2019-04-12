---
layout: post
title:  "Intersection of Two Arrays - some comments"
date:   2018-08-10 18:16:10 -0700
categories: coding
---

For problems like:
```
**Input:** nums1 = [1,2,2,1], nums2 = [2,2]
**Output:** [2,2]
```

When would you use set vs sort+search approach?

**tl;dr**
`Hashtable` solution will be slower than `sort + search` solution under certain conditions. Boundary condition is (I don't know how to solve this algebraically, I used graphing calculator):
```
k + x = (k + 1)log2(x)
where k is constant and is equal to size of bigger array.
x is variable and is size of smaller array.

For certain values of k there will be no solution, 
in those cases sort+search will be always better than hashtable.
Else there will be 2 roots of this equation and when x will be
between those roots hashtable will be better than sort+search.
```

**Details**
```
Let length of smaller array = SMALL
Let length of bigger array = BIG
```
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
You'll find that using hashtable is better for any value of SMALL.
Set BIG = 100 and there will be certain range of SMALL where where hashtable will be better than searching in sorted array.


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
If we repeat the plotting experiment from case 1 with BIG = 999 and BIG = 10, we will have same observation as in case 1.
Here is the link to graphs for BIG = 10: https://www.desmos.com/calculator/pvg5vj3qnf
and BIG = 999: https://www.desmos.com/calculator/tcxm8pcowp

**which one to use for given inputs**
if `BIG + SMALL` < `(BIG + 1)log2(SMALL)` use hashtable
else sort+search

Please correct me if I am wrong.