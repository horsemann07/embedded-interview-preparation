# Google DSA Interview Questions — 8 Years Experience
> ~500 Questions | Beginner → Intermediate → Advanced → Expert
> Topics: Arrays, Strings, Trees, Graphs, DP, System Design DSA
> ⚠️ No answers — test yourself
> Format: Question → Counter Questions

---
## GRADING REFERENCE

| Level | Expected Performance |
|-------|---------------------|
| Beginner | Solve in < 10 mins, explain complexity |
| Intermediate | Solve in < 20 mins, optimize on follow-up |
| Advanced | Solve in < 30 mins, handle all edge cases |
| Expert | Design + solve in < 45 mins, prove correctness |

---

## TOPIC DIFFICULTY MATRIX

| Topic | Beginner | Intermediate | Advanced | Expert |
|-------|----------|-------------|----------|--------|
| Arrays | Q1–15 | Q16–30 | Q31–40 | Q41–45 |
| Hashing | Q46–55 | Q56–65 | Q66–75 | — |
| Two Pointers | Q76–80 | Q81–90 | Q91–95 | — |
| Binary Search | Q96–100 | Q101–110 | Q111–115 | — |
| Trees | Q191–195 | Q196–205 | Q206–210 | — |
| Graphs | Q251–255 | Q256–265 | Q266–275 | — |
| DP | Q301–305 | Q306–315 | Q316–345 | — |
| Expert Round | — | — | — | Q461–500 |

---

*Google interviews test clarity of thought > memorization.*
*If you can explain WHY, not just HOW — you are ready.*

---

## TABLE OF CONTENTS
1. Arrays & Strings
2. Hashing & Maps
3. Two Pointers & Sliding Window
4. Binary Search
5. Sorting & Searching
6. Recursion & Backtracking
7. Linked Lists
8. Stacks & Queues
9. Trees & Binary Trees
10. Binary Search Trees
11. Heaps & Priority Queues
12. Tries
13. Graphs — BFS/DFS
14. Graphs — Shortest Path
15. Graphs — Advanced
16. Dynamic Programming — 1D
17. Dynamic Programming — 2D
18. Dynamic Programming — Advanced
19. Greedy Algorithms
20. Divide & Conquer
21. Bit Manipulation
22. Math & Number Theory
23. Intervals
24. Matrix Problems
25. Segment Trees & BITs
26. Union Find / DSU
27. Monotonic Stack/Queue
28. String Algorithms
29. Advanced Data Structures
30. System Design DSA

---

## 1. ARRAYS & STRINGS

### BEGINNER

1. Find the maximum element in an array.
   - Counter: What if array has duplicates — return index or value?
   - Counter: What if all elements are negative?
   - Counter: Can you do it in one pass?



```c++
#include <climits>

int findMax(int arr[], int n)
{
    if(arr == nullptr || n <= 0)
    {
        return INT_MIN; // or throw exception
    }

    int max = INT_MIN;

    for(int i = 0; i < n; i++)
    {
        if(arr[i] > max)
        {
            max = arr[i];
        }
    }

    return max;
}


#include <climits>

int findMaxIndex(int arr[], int n)
{
    if(arr == nullptr || n <= 0)
    {
        return -1;
    }

    int max = arr[0];
    int index = 0;

    for(int i = 1; i < n; i++)
    {
        if(arr[i] > max)
        {
            max = arr[i];
            index = i;
        }
    }

    return index;

}
```
* Follow up 1: 
  * Given an integer array arr of size n, find:
    * Maximum element
    * Second maximum element
    * Index of the first occurrence of the maximum element

```c++

/*
====================================================
Question:
Find Maximum, Second Maximum and Index of Maximum
in a Single Pass
====================================================

Complexity:
Time  : O(n)
Space : O(1)

----------------------------------------------------
Example 1: Normal Case
----------------------------------------------------
Input:
arr = {10, 5, 20, 15}

Output:
max = 20
secondMax = 15
index = 2

----------------------------------------------------
Example 2: Duplicate Maximum
----------------------------------------------------
Input:
arr = {8, 8, 3, 2}

Output:
max = 8
secondMax = 3
index = 0

Note:
First occurrence of maximum is returned.

----------------------------------------------------
Example 3: All Negative Numbers
----------------------------------------------------
Input:
arr = {-10, -5, -20, -3}

Output:
max = -3
secondMax = -5
index = 3

----------------------------------------------------
Example 4: Single Element
----------------------------------------------------
Input:
arr = {10}

Output:
max = 10
secondMax = INT_MIN   // Not Found
index = 0

----------------------------------------------------
Example 5: All Elements Same
----------------------------------------------------
Input:
arr = {8, 8, 8, 8}

Output:
max = 8
secondMax = INT_MIN   // Not Found
index = 0

Note:
No valid second maximum exists.

----------------------------------------------------
Example 6: INT_MIN Present
----------------------------------------------------
Input:
arr = {INT_MIN, -5, -10}

Output:
max = -5
secondMax = -10
index = 1

----------------------------------------------------
Example 7: Empty Array
----------------------------------------------------
Input:
arr = {}

Output:
max = INT_MIN
secondMax = INT_MIN
index = -1

Note:
Indicates invalid input.

----------------------------------------------------
Corner Cases Covered
----------------------------------------------------
✓ Empty Array
✓ Null Pointer
✓ Single Element
✓ Duplicate Maximum Values
✓ All Elements Same
✓ All Negative Numbers
✓ INT_MIN Present
✓ One Pass Solution
✓ O(1) Extra Space
*/
#include <iostream>
#include <climits>

struct Result
{
    int max;        // Maximum element
    int secondMax;  // Second maximum element
    int index;      // Index of first occurrence of maximum
};

Result findMaxAndSecondMax(int arr[], int n)
{
    Result res;

    // Corner Case 1:
    // Empty array or null pointer
    if(arr == nullptr || n <= 0)
    {
        res.max = INT_MIN;
        res.secondMax = INT_MIN;
        res.index = -1;
        return res;
    }

    // Initialize with first element
    res.max = arr[0];
    res.secondMax = INT_MIN;
    res.index = 0;

    // One-pass traversal
    for(int i = 1; i < n; i++)
    {
        // New maximum found
        if(arr[i] > res.max)
        {
            res.secondMax = res.max;
            res.max = arr[i];
            res.index = i;
        }

        // Update second maximum
        else if(arr[i] < res.max &&
                arr[i] > res.secondMax)
        {
            res.secondMax = arr[i];
        }
    }

    return res;
}

```

2. Reverse an array in place.
   - Counter: What is space complexity?
   - Counter: How do you handle odd-length arrays?
   - Counter: What if the array is immutable?

```
/*
====================================================
Question:
Reverse an Array In-Place
====================================================

Approach:
Use two pointers:
- st (start)
- bk (back/end)

Swap elements from both ends and move
towards the center until st >= bk.

Example:
arr = {2, 4, 5, 6, 7}

Iteration 1:
Swap arr[0] and arr[4]

{7, 4, 5, 6, 2}

Iteration 2:
Swap arr[1] and arr[3]

{7, 6, 5, 4, 2}

Done.

====================================================
Complexity
====================================================

Time Complexity:
O(n)

Reason:
We process approximately n/2 swaps.

Example:
n = 10

swap(0,9)
swap(1,8)
swap(2,7)
swap(3,6)
swap(4,5)

Only 5 swaps.

Still O(n).

----------------------------------------------------

Space Complexity:
O(1)

Reason:
Only three extra variables are used:

int st;
int bk;
int temp;

Regardless of array size.

n = 10      -> 3 variables
n = 1000    -> 3 variables
n = 100000  -> 3 variables

Extra memory does NOT grow with input size.

Therefore:

Space = O(1)

This is called an In-Place Algorithm.

====================================================
Corner Cases
====================================================

Case 1: Empty Array

Input:
{}

Output:
Invalid Input

----------------------------------------------------

Case 2: Null Pointer

Input:
nullptr

Output:
Invalid Input

----------------------------------------------------

Case 3: Single Element

Input:
{5}

Output:
{5}

No swap needed.

----------------------------------------------------

Case 4: Two Elements

Input:
{1,2}

Output:
{2,1}

----------------------------------------------------

Case 5: Odd Length Array

Input:
{1,2,3,4,5}

Output:
{5,4,3,2,1}

Note:
Middle element remains unchanged.

----------------------------------------------------

Case 6: Even Length Array

Input:
{1,2,3,4}

Output:
{4,3,2,1}

----------------------------------------------------

Case 7: Negative Numbers

Input:
{-1,-2,-3}

Output:
{-3,-2,-1}

----------------------------------------------------

Case 8: Duplicate Values

Input:
{7,7,7,7}

Output:
{7,7,7,7}

====================================================
Follow-Up Questions
====================================================

Q1. What if array length is odd?

Example:

{1,2,3,4,5}

When st == bk

Middle element (3) stays as-is.

No special handling needed.

----------------------------------------------------

Q2. What if array is immutable?

Example:

const int arr[]

Cannot modify original array.

Create a new array:

newArr[n]

Copy elements in reverse order.

Time  : O(n)
Space : O(n)

----------------------------------------------------

Q3. Can we do it recursively?

Yes.

Time  : O(n)
Space : O(n) due to recursion stack.

====================================================
Code
====================================================
*/

#include <iostream>

int reverseArr(int a[], int n)
{
    // Validate input
    if(a == nullptr || n <= 0)
    {
        return -1;
    }

    int st = 0;
    int bk = n - 1;

    // Reverse in-place
    while(st < bk)
    {
        int temp = a[st];
        a[st] = a[bk];
        a[bk] = temp;

        st++;
        bk--;
    }

    return 0; // Success
}

```
3. Find second largest element in array.
   - Counter: What if all elements are same?
   - Counter: Can you do it in single pass without sorting?
   - Counter: What if array has only one element?

4. Check if array is sorted.
   - Counter: Ascending or descending or both?
   - Counter: Handle duplicates — is `[1,1,2]` sorted?
   - Counter: What about circular sorted array?

5. Remove duplicates from sorted array.
   - Counter: In-place or new array?
   - Counter: What is the optimal space complexity?
   - Counter: What if array is unsorted?

6. Move all zeros to end of array.
   - Counter: Maintain relative order of non-zero elements?
   - Counter: Can you do it in one pass?
   - Counter: What if all elements are zero?

7. Find missing number in `[1..N]`.
   - Counter: What if there are multiple missing numbers?
   - Counter: How does XOR approach work here?
   - Counter: What if range is `[0..N]`?

8. Count occurrences of element in sorted array.
   - Counter: Can you do better than O(n)?
   - Counter: What is binary search approach?
   - Counter: Handle when element not present?

9. Find intersection of two unsorted arrays.
   - Counter: With duplicates or without?
   - Counter: What if one array is much larger?
   - Counter: What is hash set approach time complexity?

10. Find union of two arrays.
    - Counter: With or without duplicates in result?
    - Counter: Sort-merge approach vs hash approach — tradeoffs?
    - Counter: What if both arrays are sorted — can you beat O(n log n)?

11. Rotate array by K positions.
    - Counter: Left rotate or right rotate?
    - Counter: What if K > array length?
    - Counter: Can you do it in O(1) space?
    - Counter: Explain the reverse-three-times trick.

12. Find pair with given sum in sorted array.
    - Counter: What if no pair exists?
    - Counter: Return indices or values?
    - Counter: Multiple pairs — return all?

13. Find leaders in array (element greater than all to its right).
    - Counter: Rightmost element is always leader — why?
    - Counter: Can you do it in O(n)?
    - Counter: What if array is sorted ascending?

14. Find equilibrium index in array.
    - Counter: What if multiple equilibrium indices exist?
    - Counter: Can prefix sum help here?
    - Counter: What if no equilibrium index exists?

15. Check if array can be divided into pairs with equal sum.
    - Counter: What is necessary condition on array length?
    - Counter: How does sorting help?
    - Counter: What if elements can repeat?

---

### INTERMEDIATE

16. Maximum subarray sum (Kadane's algorithm).
    - Counter: What if all elements are negative?
    - Counter: Return sum or also indices?
    - Counter: What is the DP interpretation of Kadane's?
    - Counter: Circular array maximum subarray?

17. Maximum product subarray.
    - Counter: Why do you track both max and min?
    - Counter: What if array contains zero?
    - Counter: How does negative number affect max product?

18. Find all subsets of an array.
    - Counter: How many subsets does array of size N have?
    - Counter: Iterative vs recursive approach — which is faster?
    - Counter: What if array has duplicates — avoid duplicate subsets?

19. Three sum — find all triplets with zero sum.
    - Counter: How do you avoid duplicate triplets?
    - Counter: What is the time complexity after sorting?
    - Counter: Generalize to four sum?

20. Container with most water.
    - Counter: Why does greedy two-pointer work here?
    - Counter: What is the proof of correctness?
    - Counter: What if bars can be at non-integer positions?

21. Trapping rain water.
    - Counter: Explain precompute left-max right-max approach.
    - Counter: Can you do it in O(1) extra space?
    - Counter: Two pointer approach — prove it works.

22. Jump game — can you reach last index?
    - Counter: Greedy vs DP — which is better and why?
    - Counter: What is the invariant in the greedy approach?
    - Counter: Jump game II — minimum jumps?

23. Find minimum in rotated sorted array.
    - Counter: What if there are duplicates?
    - Counter: How does binary search work on rotated array?
    - Counter: What if array is not rotated at all?

24. Search in rotated sorted array.
    - Counter: How do you determine which half is sorted?
    - Counter: What if duplicates are present?
    - Counter: Can you extend to find all occurrences?

25. Spiral matrix traversal.
    - Counter: How do you track boundaries?
    - Counter: What if matrix is not square?
    - Counter: Generate spiral matrix from 1 to N²?

26. Product of array except self.
    - Counter: Without division operation?
    - Counter: O(1) extra space solution?
    - Counter: What if array contains zeros — one zero vs multiple?

27. Find duplicate number in array of N+1 integers from 1 to N.
    - Counter: Without modifying array and O(1) space — Floyd's cycle?
    - Counter: What is the proof that cycle exists?
    - Counter: What if there are multiple duplicates?

28. Longest consecutive sequence.
    - Counter: Can you do O(n) using hash set?
    - Counter: What is the key insight for O(n)?
    - Counter: Return the sequence or just the length?

29. Subarray sum equals K.
    - Counter: Prefix sum + hash map — explain the approach.
    - Counter: What if K is negative?
    - Counter: Count subarrays vs find one subarray?

30. Sort colors (Dutch National Flag).
    - Counter: What is three-way partition?
    - Counter: Single pass with O(1) space?
    - Counter: Generalize to K colors?

---

### ADVANCED

31. Median of two sorted arrays.
    - Counter: O(log(min(m,n))) solution — explain binary search on partitions.
    - Counter: What is the invariant you maintain?
    - Counter: What if arrays have different sizes?
    - Counter: What happens at edge cases (all elements of one array are smaller)?

32. Largest rectangle in histogram.
    - Counter: Brute force O(n²) vs stack O(n) — explain stack approach.
    - Counter: What does the stack represent?
    - Counter: How does this extend to maximal rectangle in binary matrix?

33. Sliding window maximum.
    - Counter: Why use deque instead of max-heap?
    - Counter: What invariant does the deque maintain?
    - Counter: What is the time complexity?

34. Minimum window substring.
    - Counter: How do you check if window contains all characters?
    - Counter: What is two-pointer shrink condition?
    - Counter: What if pattern has duplicate characters?

35. Find all anagrams in a string.
    - Counter: Sliding window with character frequency?
    - Counter: How to compare frequency maps efficiently?
    - Counter: Rolling hash approach?

36. Longest subarray with equal 0s and 1s.
    - Counter: Transform to prefix sum problem — how?
    - Counter: What is the key insight with hash map?
    - Counter: Generalize to count of two specific values being equal?

37. Maximum sum of non-adjacent elements.
    - Counter: What is the DP recurrence?
    - Counter: O(1) space solution?
    - Counter: What if it's a circular array?

38. Count inversions in array.
    - Counter: Brute force O(n²) vs merge sort O(n log n)?
    - Counter: How does merge step count inversions?
    - Counter: What is BIT/Fenwick tree approach?

39. Chocolate distribution problem — minimize difference.
    - Counter: Why does sorting help?
    - Counter: Sliding window of size M on sorted array?
    - Counter: What if children can get any number of packets?

40. Allocate minimum number of pages (book allocation).
    - Counter: Binary search on answer — what is the search space?
    - Counter: What is the feasibility check function?
    - Counter: Can you prove monotonicity of the feasibility function?

---

### EXPERT

41. Shortest subarray with sum at least K (with negatives).
    - Counter: Why doesn't standard sliding window work with negatives?
    - Counter: Monotonic deque on prefix sums — explain.
    - Counter: What is time complexity?

42. Maximum sum rectangle in 2D matrix.
    - Counter: How does Kadane's extend to 2D?
    - Counter: Fix two row boundaries — what problem does column become?
    - Counter: Time complexity of O(n²m) approach?

43. Count of range sum.
    - Counter: Merge sort on prefix sums?
    - Counter: Segment tree / BIT approach?
    - Counter: What is the time complexity?

44. Minimum number of jumps to reach end (with BFS approach).
    - Counter: BFS level = jump count — explain.
    - Counter: Why is greedy O(n) correct here?
    - Counter: Compare greedy vs BFS — same result?

45. Find the celebrity problem.
    - Counter: What is the O(n²) approach?
    - Counter: O(n) approach using elimination?
    - Counter: Prove the elimination step is correct.

---

## 2. HASHING & MAPS

### BEGINNER

46. Two sum — find indices of pair with target.
    - Counter: One pass hash map — explain.
    - Counter: What if multiple pairs exist?
    - Counter: Sorted array — two pointer vs hash map?

47. Check if two strings are anagrams.
    - Counter: Sorting approach vs frequency count?
    - Counter: Handle Unicode characters?
    - Counter: What is space complexity?

48. Group anagrams together.
    - Counter: What is the key for grouping?
    - Counter: Sorted string vs character count as key?
    - Counter: Time complexity?

49. Find first non-repeating character in string.
    - Counter: One pass vs two pass?
    - Counter: What data structure preserves insertion order?
    - Counter: What if all characters repeat?

50. Check if array contains duplicate within K distance.
    - Counter: Sliding window hash set of size K?
    - Counter: What do you add and remove from the set?
    - Counter: What if K is larger than array?

51. Intersection of two arrays II (with duplicates).
    - Counter: Hash map approach?
    - Counter: Sorted merge approach — when is it better?
    - Counter: What if one array doesn't fit in memory?

52. Find common characters in array of strings.
    - Counter: Minimum frequency across all strings?
    - Counter: How do you compute the minimum?
    - Counter: Edge case: empty string in array?

53. Isomorphic strings.
    - Counter: Bidirectional mapping — why both directions?
    - Counter: What if same character maps to different characters?
    - Counter: Can two strings be isomorphic to each other?

54. Word pattern matching.
    - Counter: Same as isomorphic strings but word-level?
    - Counter: What is edge case with "aa" pattern?
    - Counter: How do you split string into words?

55. Check if array is subset of another.
    - Counter: Hash map frequency count?
    - Counter: What if subset array has duplicates?
    - Counter: Sorted binary search approach?

---

### INTERMEDIATE

56. Longest substring without repeating characters.
    - Counter: Sliding window with hash map?
    - Counter: What do you store — character or last seen index?
    - Counter: When do you shrink the window?

57. Subarray with zero sum.
    - Counter: Prefix sum hash set?
    - Counter: What is the key insight?
    - Counter: Count all such subarrays?

58. Count distinct elements in every window of size K.
    - Counter: Sliding window with frequency map?
    - Counter: When does count increase/decrease?
    - Counter: What is time complexity?

59. Largest subarray with K distinct integers.
    - Counter: Two pointer with hash map?
    - Counter: What is the shrink condition?
    - Counter: Exactly K distinct vs at most K distinct?

60. Four sum — find all unique quadruplets.
    - Counter: Sort + two pointer after fixing two elements?
    - Counter: How do you skip duplicates?
    - Counter: Time complexity?

61. Longest palindrome by rearranging characters.
    - Counter: Count characters with odd frequency?
    - Counter: Can we place one odd-count character in center?
    - Counter: How to build the actual palindrome?

62. Check if permutation of string exists in another string.
    - Counter: Sliding window with character frequency?
    - Counter: When are two frequency maps equal?
    - Counter: Can you use rolling hash?

63. Pairs with difference K in array.
    - Counter: Hash set O(n) approach?
    - Counter: What if K is zero?
    - Counter: Count pairs vs find pairs?

64. Top K frequent elements.
    - Counter: Heap vs bucket sort — which is faster?
    - Counter: Bucket sort O(n) approach — explain.
    - Counter: What if multiple elements have same frequency?

65. Contiguous array with equal 0s and 1s.
    - Counter: Transform 0 to -1 — why?
    - Counter: Prefix sum hash map for longest subarray with sum 0?
    - Counter: What if we want exactly K more 1s than 0s?

---

### ADVANCED

66. Minimum window substring (revisit with detail).
    - Counter: When is window valid?
    - Counter: How do you shrink without missing answers?
    - Counter: What is space complexity?

67. Longest substring with at most K distinct characters.
    - Counter: What is the two-pointer invariant?
    - Counter: How do you remove character from window?
    - Counter: Generalize to exactly K distinct?

68. Alien dictionary — find character order.
    - Counter: Build graph from adjacent word comparisons?
    - Counter: Topological sort?
    - Counter: What if contradiction exists?

69. Design a data structure for O(1) insert, delete, getRandom.
    - Counter: Hash map + array combination?
    - Counter: How do you handle delete in O(1)?
    - Counter: What is the swap trick for delete?

70. LRU cache — implement with O(1) operations.
    - Counter: HashMap + doubly linked list?
    - Counter: How does HashMap point to list nodes?
    - Counter: What is sentinel node trick?

71. LFU cache — implement with O(1) operations.
    - Counter: What additional bookkeeping over LRU?
    - Counter: Min frequency tracking?
    - Counter: How do you handle frequency tie-breaking?

72. All O(1) data structure — inc, dec, getMaxKey, getMinKey.
    - Counter: Doubly linked list of frequency buckets?
    - Counter: HashMap from key to bucket?
    - Counter: How do you move a key between buckets?

73. Palindrome pairs in list of words.
    - Counter: Brute force O(n²k) vs hash map O(nk²)?
    - Counter: What are the three cases for palindrome pair?
    - Counter: Trie approach?

74. Brick wall — minimum bricks cut by vertical line.
    - Counter: Count edges at each position?
    - Counter: Why do you exclude last edge?
    - Counter: What if all rows are same width?

75. Random pick with weight.
    - Counter: Prefix sum + binary search?
    - Counter: How do you generate weighted random?
    - Counter: What is time complexity of each pick?

---

## 3. TWO POINTERS & SLIDING WINDOW

### BEGINNER

76. Two sum in sorted array using two pointers.
    - Counter: Why does two pointer work on sorted array?
    - Counter: What is the invariant?
    - Counter: What if no pair exists?

77. Remove element from array in-place.
    - Counter: Slow/fast pointer approach?
    - Counter: What is returned — length or array?
    - Counter: What happens to elements after new length?

78. Merge sorted arrays in-place.
    - Counter: Fill from back — why?
    - Counter: What if one array has no extra space?
    - Counter: Time and space complexity?

79. Squares of sorted array in sorted order.
    - Counter: Two pointer from ends — why?
    - Counter: Fill result from back?
    - Counter: Can you do it without two pointers?

80. Valid palindrome with only alphanumeric.
    - Counter: Two pointer from ends?
    - Counter: Skip non-alphanumeric characters?
    - Counter: Case-insensitive comparison?

---

### INTERMEDIATE

81. Longest substring with at most two distinct characters.
    - Counter: Sliding window with character count?
    - Counter: When do you shrink window?
    - Counter: Generalize to K distinct?

82. 3Sum closest to target.
    - Counter: Sort + two pointer?
    - Counter: How do you track minimum difference?
    - Counter: What if multiple solutions are equidistant?

83. Minimum size subarray with sum >= target.
    - Counter: Two pointer shrink when sum >= target?
    - Counter: What if no subarray exists?
    - Counter: What is time complexity?

84. Fruit into baskets (at most two distinct).
    - Counter: Same as longest subarray with 2 distinct?
    - Counter: What does fruit type represent?
    - Counter: How do you handle removing fruit from first basket?

85. Remove duplicates from sorted array II (allow at most 2).
    - Counter: What is the two-pointer condition?
    - Counter: Generalize to allow at most K duplicates?
    - Counter: What changes in your code for K=3?

86. Sort array by parity (even then odd).
    - Counter: Two pointer from both ends?
    - Counter: Stable vs unstable sort here?
    - Counter: What if you want odd then even?

87. Backspace string compare.
    - Counter: Build actual string vs two pointer from end?
    - Counter: Why process from end?
    - Counter: Multiple backspaces in a row?

88. Number of subarrays with product less than K.
    - Counter: Sliding window?
    - Counter: Why count `right - left + 1` for each right?
    - Counter: What if K <= 1?

89. Permutation in string.
    - Counter: Sliding window of fixed size?
    - Counter: Frequency map comparison?
    - Counter: Can you use single integer to compare maps?

90. Max consecutive ones III with at most K zeros.
    - Counter: Sliding window — what does the window represent?
    - Counter: Shrink when zeros in window > K?
    - Counter: What if K = 0?

---

### ADVANCED

91. Minimum window subsequence.
    - Counter: Find window then try to minimize from left?
    - Counter: Two pass two pointer?
    - Counter: How is this different from minimum window substring?

92. Subarrays with exactly K distinct integers.
    - Counter: atMost(K) - atMost(K-1) trick?
    - Counter: Why does this work?
    - Counter: Can you do it directly without subtraction?

93. Longest repeating character replacement.
    - Counter: Sliding window — what is the invariant?
    - Counter: Why `window_size - max_freq <= K`?
    - Counter: Do you need to decrease max_freq when shrinking?

94. Minimum number of swaps to group all 1s together.
    - Counter: Circular array?
    - Counter: Fixed window of size = count of 1s?
    - Counter: Count 0s in window = swaps needed?

95. Maximum number of vowels in substring of given length.
    - Counter: Sliding window of fixed size?
    - Counter: Rolling count — add new, remove old?
    - Counter: What if K > string length?

---

## 4. BINARY SEARCH

### BEGINNER

96. Binary search in sorted array.
    - Counter: Iterative vs recursive — stack overflow risk?
    - Counter: `mid = left + (right - left) / 2` — why not `(left + right) / 2`?
    - Counter: What does your while condition look like — `<` or `<=`?

97. Find first occurrence of element.
    - Counter: How does your binary search differ from standard?
    - Counter: When do you stop vs continue searching left?
    - Counter: What is `[left, right)` vs `[left, right]` template?

98. Find last occurrence of element.
    - Counter: What changes from first occurrence?
    - Counter: Can you combine first and last occurrence in one function?
    - Counter: Binary search on answer concept?

99. Count occurrences in sorted array.
    - Counter: Last occurrence - first occurrence + 1?
    - Counter: Handle element not found?
    - Counter: Two binary searches vs one?

100. Floor and ceiling in sorted array.
     - Counter: Floor = largest element <= X?
     - Counter: Ceiling = smallest element >= X?
     - Counter: What if X is outside array range?

---

### INTERMEDIATE

101. Search in rotated sorted array.
     - Counter: How do you identify which half is sorted?
     - Counter: What if pivot is at index 0?
     - Counter: With duplicates — when does O(log n) break down?

102. Find minimum in rotated sorted array.
     - Counter: What is the binary search condition?
     - Counter: How does comparing with rightmost element help?
     - Counter: With duplicates?

103. Search in 2D matrix (row and column sorted).
     - Counter: Start from top-right or bottom-left — why?
     - Counter: Can you use binary search differently?
     - Counter: What if only rows are sorted?

104. Find peak element.
     - Counter: Binary search on unsorted array — how?
     - Counter: What property allows binary search?
     - Counter: What if multiple peaks exist?

105. Kth smallest element in sorted matrix.
     - Counter: Binary search on value not index?
     - Counter: How do you count elements <= mid?
     - Counter: Heap approach vs binary search?

106. Sqrt(x) using binary search.
     - Counter: Integer sqrt — what is the search space?
     - Counter: How do you avoid overflow in mid*mid?
     - Counter: Newton's method approach?

107. Capacity to ship packages within D days.
     - Counter: Binary search on capacity?
     - Counter: What is search space lower and upper bound?
     - Counter: Feasibility check function?

108. Aggressive cows — maximum minimum distance.
     - Counter: Binary search on distance?
     - Counter: Feasibility check — can you place all cows?
     - Counter: What is the monotonicity property?

109. Painter's partition problem.
     - Counter: Same structure as book allocation?
     - Counter: Binary search on time — feasibility?
     - Counter: How does it differ from book allocation?

110. Find median of two sorted arrays.
     - Counter: Binary search on partition — what is invariant?
     - Counter: Edge cases when one array is fully on one side?
     - Counter: O(log(min(m,n))) vs O(log(m+n))?

---

### ADVANCED

111. Binary search on answer — template.
     - Counter: What makes a problem amenable to binary search on answer?
     - Counter: How do you identify search space?
     - Counter: What is the monotone function?

112. Split array largest sum.
     - Counter: Same as painter's partition?
     - Counter: Binary search vs DP — which is easier to code?
     - Counter: What is DP recurrence?

113. Minimize max difference between heights.
     - Counter: Binary search on difference?
     - Counter: Greedy after sorting?
     - Counter: What is the key observation about optimal solution?

114. Find K closest elements to X in sorted array.
     - Counter: Binary search to find insertion point?
     - Counter: Two pointer from insertion point?
     - Counter: Why not just sort by distance?

115. Count of smaller numbers after self.
     - Counter: Binary search on sorted list of visited elements?
     - Counter: Merge sort approach?
     - Counter: BIT/Fenwick tree approach?

---

## 5. SORTING & SEARCHING

### BEGINNER

116. Implement merge sort.
     - Counter: Time and space complexity?
     - Counter: Why is merge sort stable?
     - Counter: When is merge sort preferred over quick sort?

117. Implement quick sort.
     - Counter: Worst case — when does it occur?
     - Counter: How does pivot choice affect performance?
     - Counter: Three-way partition for duplicates?

118. Implement heap sort.
     - Counter: Is heap sort stable?
     - Counter: How do you build max-heap in O(n)?
     - Counter: Why is build-heap O(n) not O(n log n)?

119. Counting sort.
     - Counter: When is counting sort applicable?
     - Counter: How do you handle negative numbers?
     - Counter: Why is it O(n + k)?

120. Radix sort.
     - Counter: LSD vs MSD radix sort?
     - Counter: What underlying sort is used per digit?
     - Counter: Time complexity?

---

### INTERMEDIATE

121. Sort array of 0s, 1s, 2s (Dutch National Flag).
     - Counter: Three pointer approach?
     - Counter: What does each pointer maintain?
     - Counter: Single pass guarantee?

122. Merge K sorted arrays.
     - Counter: Min-heap approach — time complexity?
     - Counter: Divide and conquer approach?
     - Counter: Which is better for large K?

123. Sort array by frequency.
     - Counter: Frequency map + custom comparator?
     - Counter: Tie-breaking by value?
     - Counter: Bucket sort approach?

124. Wiggle sort (arr[0] < arr[1] > arr[2] < arr[3]...).
     - Counter: O(n) approach without sorting?
     - Counter: What is the swap condition?
     - Counter: Wiggle sort II — with median?

125. Find Kth largest element.
     - Counter: Heap of size K?
     - Counter: Quick select — expected O(n)?
     - Counter: What is worst case of quick select?

126. Custom sort string.
     - Counter: Build order map from order string?
     - Counter: Sort comparator using order map?
     - Counter: Elements not in order — where do they go?

127. Pancake sorting.
     - Counter: Find maximum, flip to front, flip to position?
     - Counter: Minimum number of flips?
     - Counter: Is this related to any other problem?

128. Sort linked list.
     - Counter: Why merge sort for linked list?
     - Counter: How do you find middle of linked list?
     - Counter: Bottom-up merge sort for O(1) space?

129. External sorting.
     - Counter: What is external sorting?
     - Counter: K-way merge with replacement selection?
     - Counter: How do you minimize disk I/O?

130. Find duplicates in O(n) time O(1) space.
     - Counter: Index as hash trick?
     - Counter: What if array has values outside [1,n]?
     - Counter: Floyd's cycle detection approach?

---

## 6. RECURSION & BACKTRACKING

### BEGINNER

131. Factorial and Fibonacci — recursion.
     - Counter: Stack overflow for large n?
     - Counter: Tail recursion optimization?
     - Counter: Memoized recursion vs DP?

132. Tower of Hanoi.
     - Counter: Recurrence relation?
     - Counter: Minimum number of moves?
     - Counter: Iterative solution?

133. Generate all subsets of array.
     - Counter: Include/exclude recursion tree?
     - Counter: Iterative bit manipulation approach?
     - Counter: Handle duplicates?

134. Generate all permutations of string.
     - Counter: Swap-based vs visited-array approach?
     - Counter: How to handle duplicates?
     - Counter: Count of permutations?

135. Print all subsequences of string.
     - Counter: Same as subsets?
     - Counter: Count vs print — different complexity?
     - Counter: What is the recursion tree?

---

### INTERMEDIATE

136. N-Queens problem.
     - Counter: What constraints do you check for each placement?
     - Counter: How do you check diagonal efficiently?
     - Counter: Count solutions vs print all?

137. Rat in a maze.
     - Counter: Four directional movement?
     - Counter: Mark visited cells — backtrack correctly?
     - Counter: Minimum path length?

138. Sudoku solver.
     - Counter: Constraint propagation before backtracking?
     - Counter: How do you check 3x3 box constraint?
     - Counter: Optimization — pick cell with fewest options?

139. Word search in grid.
     - Counter: DFS with backtracking?
     - Counter: Mark visited — restore on backtrack?
     - Counter: Multiple same cells — not allowed?

140. Generate parentheses — all valid combinations.
     - Counter: What invariant do you maintain?
     - Counter: When do you add open vs close?
     - Counter: Count of valid parentheses (Catalan number)?

141. Combination sum — unlimited use.
     - Counter: How do you avoid duplicate combinations?
     - Counter: Sort + start index trick?
     - Counter: What if candidates have duplicates?

142. Combination sum II — each element used once.
     - Counter: How to handle duplicates in candidates?
     - Counter: Skip same element at same level?
     - Counter: Why sort first?

143. Permutations with duplicates.
     - Counter: How to skip duplicate permutations?
     - Counter: Sort and visited array approach?
     - Counter: Condition for skipping?

144. Letter combinations of phone number.
     - Counter: Backtracking tree structure?
     - Counter: Base case?
     - Counter: What if input has 0 or 1?

145. Palindrome partitioning.
     - Counter: Check palindrome at each split?
     - Counter: Precompute palindrome table?
     - Counter: Minimum cuts vs all partitions?

---

### ADVANCED

146. Word break II — return all sentences.
     - Counter: Memoized backtracking?
     - Counter: Why pure backtracking TLEs?
     - Counter: What do you memoize?

147. Expression add operators — insert +,-,* to reach target.
     - Counter: How do you handle multiplication precedence?
     - Counter: Track `prevOperand` for multiplication?
     - Counter: Overflow handling?

148. Remove invalid parentheses.
     - Counter: BFS level by level removal?
     - Counter: How do you know minimum removals?
     - Counter: Avoid duplicate results?

149. Unique paths III — visit every cell.
     - Counter: Bitmask DP or backtracking?
     - Counter: How do you track visited cells?
     - Counter: What is the complexity?

150. Zuma game — minimum insertions.
     - Counter: Group consecutive same colors?
     - Counter: Memoized recursion on groups?
     - Counter: Why is naive backtracking too slow?

---

## 7. LINKED LISTS

### BEGINNER

151. Reverse a linked list.
     - Counter: Iterative vs recursive?
     - Counter: Recursive reversal — stack memory?
     - Counter: Reverse between positions L and R?

152. Find middle of linked list.
     - Counter: Slow/fast pointer?
     - Counter: Even length — first or second middle?
     - Counter: How many passes?

153. Detect cycle in linked list.
     - Counter: Floyd's algorithm?
     - Counter: Why does fast pointer meet slow inside cycle?
     - Counter: Find start of cycle?

154. Merge two sorted linked lists.
     - Counter: Iterative vs recursive?
     - Counter: Using dummy head node?
     - Counter: In-place merge?

155. Remove Nth node from end.
     - Counter: Two pointer with N gap?
     - Counter: One pass?
     - Counter: What if N equals list length?

---

### INTERMEDIATE

156. Add two numbers represented as linked lists.
     - Counter: Handle carry from last digit?
     - Counter: Different length lists?
     - Counter: Numbers stored in reverse order?

157. Intersection of two linked lists.
     - Counter: Length difference approach?
     - Counter: Two pointer cycling approach — explain.
     - Counter: What if lists don't intersect?

158. Clone linked list with random pointer.
     - Counter: Hash map approach?
     - Counter: O(1) space approach — interweaving?
     - Counter: Restore original list?

159. Flatten multilevel linked list.
     - Counter: DFS/recursion approach?
     - Counter: Stack-based iterative?
     - Counter: How do you handle `next` after flattening `child`?

160. Sort linked list in O(n log n).
     - Counter: Merge sort — find middle, split, merge?
     - Counter: Bottom-up merge sort for O(1) space?
     - Counter: Why not quick sort for linked list?

161. LRU cache implementation.
     - Counter: Hash map + doubly linked list?
     - Counter: Why doubly linked?
     - Counter: Dummy head and tail nodes?

162. Reverse nodes in K groups.
     - Counter: Check if K nodes remain?
     - Counter: Reverse K nodes, then recurse?
     - Counter: Iterative approach?

163. Reorder list (L0→Ln→L1→Ln-1...).
     - Counter: Find middle, reverse second half, merge?
     - Counter: Why split at middle first?
     - Counter: In-place?

164. Linked list cycle — find start of cycle.
     - Counter: Prove that meeting point + head simultaneously reach cycle start?
     - Counter: Mathematical proof?
     - Counter: What if there are multiple cycles?

165. Rotate linked list by K.
     - Counter: Make it circular, find new tail?
     - Counter: What is effective rotation when K > length?
     - Counter: Two pointer approach?

---

### ADVANCED

166. Design skiplist.
     - Counter: What is expected time complexity?
     - Counter: How do you determine level for new node?
     - Counter: How does search use multiple levels?

167. Merge K sorted linked lists.
     - Counter: Min-heap approach?
     - Counter: Divide and conquer?
     - Counter: Which is better for memory?

168. Reverse alternating K groups.
     - Counter: Reverse K, skip K, repeat?
     - Counter: How to track position?
     - Counter: Edge cases at end?

169. Linked list random node (reservoir sampling).
     - Counter: What is reservoir sampling?
     - Counter: Prove uniform distribution?
     - Counter: How does this work without knowing list length?

170. Design linked list with O(1) random access.
     - Counter: Is this possible? What tradeoff?
     - Counter: Skip list approach?
     - Counter: Unrolled linked list?

---

## 8. STACKS & QUEUES

### BEGINNER

171. Valid parentheses.
     - Counter: Stack push/pop approach?
     - Counter: Handle empty stack on pop?
     - Counter: Multiple bracket types?

172. Implement stack using queues.
     - Counter: Two queues vs one queue?
     - Counter: Push-heavy vs pop-heavy implementation?
     - Counter: Amortized complexity?

173. Implement queue using stacks.
     - Counter: Two stack approach — amortized O(1)?
     - Counter: When do you transfer between stacks?
     - Counter: Worst case for single operation?

174. Min stack — push, pop, getMin in O(1).
     - Counter: Two stack approach?
     - Counter: Single stack with pairs?
     - Counter: What if min is popped?

175. Evaluate reverse Polish notation.
     - Counter: Stack-based evaluation?
     - Counter: Handle division truncation toward zero?
     - Counter: What if invalid expression?

---

### INTERMEDIATE

176. Daily temperatures — next warmer day.
     - Counter: Monotonic decreasing stack?
     - Counter: What does the stack store?
     - Counter: Difference between storing index vs value?

177. Next greater element.
     - Counter: Monotonic stack — direction of traversal?
     - Counter: Circular array next greater element?
     - Counter: Next smaller element — what changes?

178. Largest rectangle in histogram.
     - Counter: Stack stores indices of increasing heights?
     - Counter: What does popping represent?
     - Counter: Why add sentinel values?

179. Maximal rectangle in binary matrix.
     - Counter: Compute histogram row by row?
     - Counter: Apply largest rectangle in histogram?
     - Counter: Time complexity?

180. Decode string (3[abc] → abcabcabc).
     - Counter: Stack for nested brackets?
     - Counter: What do you push onto stack?
     - Counter: Handle multi-digit counts?

181. Asteroid collision.
     - Counter: Stack-based simulation?
     - Counter: When does collision happen?
     - Counter: What about equal size asteroids?

182. Remove K digits to get smallest number.
     - Counter: Monotonic increasing stack?
     - Counter: What is the greedy choice?
     - Counter: Handle leading zeros?

183. Basic calculator II.
     - Counter: Stack for + - only vs + - * /?
     - Counter: How do you handle operator precedence?
     - Counter: Two-stack approach for full expression?

184. Score of parentheses.
     - Counter: Stack-based computation?
     - Counter: What is value of `()` and `(A)`?
     - Counter: O(1) space approach using depth?

185. Trapping rain water with stack.
     - Counter: How does stack approach differ from two-pointer?
     - Counter: What does the stack represent?
     - Counter: When do you pop and compute?

---

### ADVANCED

186. Sliding window maximum with deque.
     - Counter: Monotonic decreasing deque?
     - Counter: When do you remove from front?
     - Counter: When do you remove from back?

187. Shortest subarray with sum at least K using deque.
     - Counter: Prefix sum + monotonic deque?
     - Counter: What invariant does deque maintain?
     - Counter: Why two separate monotonic conditions?

188. Maximum frequency stack.
     - Counter: Group elements by frequency?
     - Counter: How do you track maximum frequency?
     - Counter: What is the time complexity?

189. Design circular deque.
     - Counter: Array-based with front/back pointers?
     - Counter: How do you handle wrap-around?
     - Counter: Full vs empty condition?

190. Simplify file path.
     - Counter: Stack for directory components?
     - Counter: Handle `..` — pop from stack?
     - Counter: Handle `.` — skip?

---

## 9. TREES & BINARY TREES

### BEGINNER

191. Inorder, preorder, postorder traversal.
     - Counter: Recursive vs iterative for each?
     - Counter: Morris traversal — O(1) space?
     - Counter: Use case for each traversal type?

192. Level order traversal (BFS).
     - Counter: Queue-based approach?
     - Counter: Level by level vs all at once?
     - Counter: Zigzag level order?

193. Height/depth of binary tree.
     - Counter: DFS recursive approach?
     - Counter: Iterative approach?
     - Counter: Height vs depth — what is difference?

194. Count nodes in binary tree.
     - Counter: Naive O(n) vs complete binary tree O(log²n)?
     - Counter: How does complete binary tree optimization work?
     - Counter: What is a complete binary tree?

195. Check if two trees are identical.
     - Counter: Recursive comparison?
     - Counter: Structure AND values must match?
     - Counter: Iterative with two queues?

---

### INTERMEDIATE

196. Lowest common ancestor (LCA) of binary tree.
     - Counter: Recursive approach — when do you return node?
     - Counter: Three cases in recursion?
     - Counter: Iterative approach with parent pointers?

197. Diameter of binary tree.
     - Counter: What is diameter — through root or not?
     - Counter: Track max during height calculation?
     - Counter: Where is the diameter computation?

198. Binary tree maximum path sum.
     - Counter: Path can start and end at any node?
     - Counter: What can you return vs what do you track?
     - Counter: Handle negative values?

199. Symmetric tree.
     - Counter: Mirror check?
     - Counter: Iterative with queue of pairs?
     - Counter: What is the base case?

200. Path sum — does root-to-leaf path with given sum exist?
     - Counter: Subtract from target as you go down?
     - Counter: What is leaf node condition?
     - Counter: All paths with given sum?

201. Flatten binary tree to linked list.
     - Counter: Preorder traversal into list?
     - Counter: Morris traversal in-place approach?
     - Counter: Right-first postorder trick?

202. Construct binary tree from preorder and inorder.
     - Counter: First preorder element is root?
     - Counter: Find root in inorder to split?
     - Counter: Hash map for O(n) vs O(n²) search?

203. Construct binary tree from postorder and inorder.
     - Counter: Last postorder element is root?
     - Counter: What changes from preorder version?
     - Counter: Can you construct from preorder and postorder?

204. Binary tree right side view.
     - Counter: BFS last element per level?
     - Counter: DFS right-first approach?
     - Counter: What if tree is left-skewed?

205. Cousins in binary tree.
     - Counter: What makes two nodes cousins?
     - Counter: BFS track parent and depth?
     - Counter: DFS approach?

---

### ADVANCED

206. Serialize and deserialize binary tree.
     - Counter: BFS vs DFS serialization?
     - Counter: How do you handle null nodes?
     - Counter: Level order with null markers?

207. Binary tree cameras — minimum cameras to monitor all nodes.
     - Counter: Greedy bottom-up?
     - Counter: Three states per node — covered, uncovered, has camera?
     - Counter: Why greedy from leaves?

208. Recover binary search tree (two nodes swapped).
     - Counter: Inorder traversal finds the two anomalies?
     - Counter: Adjacent vs non-adjacent swap?
     - Counter: O(1) space Morris traversal approach?

209. Count good nodes in binary tree.
     - Counter: Pass max value from root down?
     - Counter: A node is good if no greater node on path from root?

210. All nodes distance K in binary tree.
     - Counter: Convert to undirected graph first?
     - Counter: BFS from target node?
     - Counter: Or DFS with parent tracking?

---

## 10. BINARY SEARCH TREES

### BEGINNER

211. Search in BST.
     - Counter: Left if smaller, right if larger?
     - Counter: Iterative vs recursive?
     - Counter: What is time complexity on unbalanced BST?

212. Insert into BST.
     - Counter: Find correct leaf position?
     - Counter: Recursive insertion?
     - Counter: Does insertion preserve BST property?

213. Delete from BST.
     - Counter: Three cases — leaf, one child, two children?
     - Counter: Find inorder successor for two-child case?
     - Counter: Why inorder successor specifically?

214. Validate BST.
     - Counter: Pass min/max bounds?
     - Counter: Why not just check parent-child?
     - Counter: Inorder traversal approach?

215. Inorder successor in BST.
     - Counter: With parent pointer?
     - Counter: Without parent pointer?
     - Counter: What if node has right subtree?

---

### INTERMEDIATE

216. Kth smallest in BST.
     - Counter: Inorder traversal count?
     - Counter: Augmented BST with subtree size?
     - Counter: What if K changes frequently?

217. Range sum in BST.
     - Counter: Prune branches outside range?
     - Counter: Time complexity?
     - Counter: Vs brute force inorder?

218. Convert BST to greater sum tree.
     - Counter: Reverse inorder traversal?
     - Counter: Running sum from right?
     - Counter: Reconstruct original BST from greater sum tree?

219. Balanced BST from sorted array.
     - Counter: Mid element as root?
     - Counter: Height of resulting tree?
     - Counter: Is this unique?

220. Two sum in BST.
     - Counter: Inorder traversal + two pointer on sorted array?
     - Counter: BST iterator approach — two pointers?
     - Counter: O(h) space solution?

221. Trim BST to range [L, R].
     - Counter: What do you return for node outside range?
     - Counter: Recursive approach?
     - Counter: Why trim left/right subtree first?

222. Merge two BSTs.
     - Counter: Inorder both + merge sorted arrays + rebuild?
     - Counter: O(m+n) time and space?
     - Counter: Can you do it in O(h1 + h2) space?

223. BST iterator — next() in amortized O(1).
     - Counter: Controlled inorder traversal with stack?
     - Counter: What does the stack represent?
     - Counter: How is next() amortized O(1)?

224. Count BSTs with N nodes (structurally unique).
     - Counter: Catalan number?
     - Counter: DP recurrence?
     - Counter: With values — unique vs structurally unique?

225. Find mode in BST.
     - Counter: Inorder traversal + count consecutive?
     - Counter: What if multiple modes?
     - Counter: O(1) space Morris traversal?

---

## 11. HEAPS & PRIORITY QUEUES

### BEGINNER

226. Implement max-heap.
     - Counter: Array representation?
     - Counter: Parent, left child, right child index formulas?
     - Counter: Heapify up vs heapify down?

227. Heap sort.
     - Counter: Build heap O(n), extract O(n log n)?
     - Counter: Is heap sort stable?
     - Counter: Why use heap sort over merge sort?

228. K largest elements in array.
     - Counter: Min-heap of size K?
     - Counter: Why min-heap not max-heap?
     - Counter: Time complexity?

229. K closest points to origin.
     - Counter: Max-heap of size K?
     - Counter: Quick select alternative?
     - Counter: Custom comparator?

230. Find median from data stream.
     - Counter: Two heaps — max-heap and min-heap?
     - Counter: How do you balance two heaps?
     - Counter: What is median when odd vs even count?

---

### INTERMEDIATE

231. Merge K sorted lists.
     - Counter: Min-heap with one element from each list?
     - Counter: What is time complexity?
     - Counter: Divide and conquer alternative?

232. Task scheduler with cooldown.
     - Counter: Greedy — schedule most frequent first?
     - Counter: Idle slots calculation?
     - Counter: Max-heap approach?

233. Reorganize string — no two adjacent same.
     - Counter: Greedy with max-heap?
     - Counter: What if impossible?
     - Counter: When is it impossible?

234. K closest numbers to X in sorted array.
     - Counter: Binary search + two pointer?
     - Counter: Max-heap of size K?
     - Counter: Which is more efficient?

235. Ugly number II — Kth ugly number.
     - Counter: Three pointers for factors 2, 3, 5?
     - Counter: Min-heap approach?
     - Counter: Why does three-pointer work?

236. Swim in rising water.
     - Counter: Dijkstra-like with min-heap?
     - Counter: Binary search on answer?
     - Counter: Union-Find approach?

237. Minimum cost to connect sticks.
     - Counter: Always merge two smallest?
     - Counter: Min-heap for O(n log n)?
     - Counter: Greedy correctness proof?

238. IPO — maximize capital.
     - Counter: Sort projects by capital?
     - Counter: Max-heap of profits available?
     - Counter: Two pass approach?

239. Frequency sort — descending frequency.
     - Counter: Frequency map + custom sort?
     - Counter: Max-heap by frequency?
     - Counter: Bucket sort approach?

240. Smallest range covering K lists.
     - Counter: Min-heap with one element from each list?
     - Counter: Track current maximum?
     - Counter: When do you advance?

---

## 12. TRIES

### BEGINNER

241. Implement trie (insert, search, startsWith).
     - Counter: Node structure — array vs hashmap for children?
     - Counter: End-of-word marker?
     - Counter: Memory usage — array vs hashmap?

242. Search word with wildcard.
     - Counter: DFS through trie?
     - Counter: Dot matches any character?
     - Counter: Backtracking in trie?

243. Word search II — find all words from list in grid.
     - Counter: Build trie from word list?
     - Counter: DFS grid with trie traversal?
     - Counter: Pruning — remove matched words from trie?

244. Count words with given prefix.
     - Counter: Store count at each node?
     - Counter: Traverse to prefix end, count subtree?

245. Longest common prefix using trie.
     - Counter: Insert all strings, traverse while single child?
     - Counter: Binary search on length?
     - Counter: Sorting approach?

---

### INTERMEDIATE

246. Replace words with root in sentence.
     - Counter: Build trie from roots?
     - Counter: For each word, find shortest prefix in trie?
     - Counter: Hash set approach vs trie?

247. Map sum pairs.
     - Counter: Store sum at each prefix node?
     - Counter: Update when key exists — how to handle old value?
     - Counter: DFS to sum all leaf values?

248. Maximum XOR of two numbers.
     - Counter: Insert all numbers into binary trie?
     - Counter: For each number, greedily pick opposite bit?
     - Counter: Offline vs online query?

249. Palindrome pairs using trie.
     - Counter: How does trie help here vs hash map?
     - Counter: Three cases for palindrome pair?
     - Counter: Time complexity?

250. Word squares.
     - Counter: Prefix constraint — row i must share prefix with previous rows?
     - Counter: Trie for prefix lookup?
     - Counter: Backtracking approach?

---

## 13. GRAPHS — BFS/DFS

### BEGINNER

251. BFS traversal of graph.
     - Counter: Queue-based?
     - Counter: Visited array — why needed?
     - Counter: Disconnected graph handling?

252. DFS traversal of graph.
     - Counter: Recursive vs iterative?
     - Counter: Iterative DFS — stack order differs from recursive?
     - Counter: When does DFS use more memory than BFS?

253. Detect cycle in undirected graph.
     - Counter: BFS/DFS with parent tracking?
     - Counter: Union-Find approach?
     - Counter: Why track parent in undirected graph DFS?

254. Detect cycle in directed graph.
     - Counter: DFS with coloring (white/gray/black)?
     - Counter: Why doesn't undirected approach work?
     - Counter: Kahn's algorithm approach?

255. Number of islands.
     - Counter: DFS/BFS flood fill?
     - Counter: Mark visited in-place vs separate array?
     - Counter: Union-Find approach?

---

### INTERMEDIATE

256. Clone graph.
     - Counter: BFS/DFS with HashMap from original to clone?
     - Counter: When do you create the clone node?
     - Counter: Handle cycles?

257. Course schedule — can you finish all courses?
     - Counter: Detect cycle in directed graph?
     - Counter: Topological sort approach?
     - Counter: Return topological order?

258. Number of provinces (connected components).
     - Counter: DFS/BFS to find components?
     - Counter: Union-Find approach?
     - Counter: Adjacency matrix vs list?

259. Walls and gates — fill distance to nearest gate.
     - Counter: Multi-source BFS from all gates?
     - Counter: Why multi-source BFS over single-source?
     - Counter: Can you use DFS?

260. Rotten oranges — time to rot all.
     - Counter: Multi-source BFS from all rotten oranges?
     - Counter: Count fresh oranges to verify all rotted?
     - Counter: What if fresh orange is unreachable?

261. Pacific Atlantic water flow.
     - Counter: Reverse flow from oceans inward?
     - Counter: BFS/DFS from ocean borders?
     - Counter: Intersection of two reachable sets?

262. Word ladder — shortest transformation.
     - Counter: BFS for shortest path?
     - Counter: Build adjacency list or check on the fly?
     - Counter: Bidirectional BFS optimization?

263. Surrounded regions — capture 'O' surrounded by 'X'.
     - Counter: DFS from border 'O' cells?
     - Counter: Mark safe cells, then flip remaining?
     - Counter: Union-Find with virtual border node?

264. Flood fill.
     - Counter: DFS/BFS from starting cell?
     - Counter: Handle when newColor == oldColor?
     - Counter: 4-directional vs 8-directional?

265. Keys and rooms.
     - Counter: DFS/BFS from room 0?
     - Counter: Can you visit all rooms?
     - Counter: Relationship to reachability problem?

---

### ADVANCED

266. Critical connections (bridges) in network.
     - Counter: Tarjan's bridge finding algorithm?
     - Counter: What is discovery time and low time?
     - Counter: When is an edge a bridge?

267. Find all articulation points.
     - Counter: Tarjan's algorithm?
     - Counter: Difference from bridges?
     - Counter: Root node special case?

268. Strongly connected components (SCC).
     - Counter: Kosaraju's vs Tarjan's algorithm?
     - Counter: What is the two-pass DFS in Kosaraju's?
     - Counter: What does SCC represent semantically?

269. Topological sort.
     - Counter: DFS-based vs Kahn's (BFS-based)?
     - Counter: Multiple valid orderings — how many?
     - Counter: What if cycle exists?

270. Bipartite graph check.
     - Counter: Two-coloring with BFS/DFS?
     - Counter: What property makes it bipartite?
     - Counter: Application to real problems?

271. Minimum number of vertices to reach all nodes.
     - Counter: Nodes with in-degree 0?
     - Counter: Why exactly those nodes?
     - Counter: What if graph has cycles?

272. Redundant connection — make tree by removing one edge.
     - Counter: Union-Find — adding edge that creates cycle?
     - Counter: DFS cycle detection approach?
     - Counter: If multiple valid answers, return last edge?

273. Is graph bipartite (revisit with odd cycle).
     - Counter: Any odd cycle makes it non-bipartite?
     - Counter: Prove: bipartite ↔ no odd cycles?

274. Course schedule III — maximum number of courses.
     - Counter: Greedy with max-heap?
     - Counter: Sort by deadline?
     - Counter: Why greedy works?

275. Alien dictionary — character ordering.
     - Counter: Build graph from adjacent pairs in words?
     - Counter: Topological sort?
     - Counter: Detect cycle = invalid dictionary?

---

## 14. GRAPHS — SHORTEST PATH

### BEGINNER

276. Dijkstra's algorithm.
     - Counter: Why does Dijkstra fail with negative edges?
     - Counter: Min-heap vs array-based?
     - Counter: Lazy deletion in heap?

277. Bellman-Ford algorithm.
     - Counter: Why N-1 iterations?
     - Counter: Detect negative cycle?
     - Counter: When to use over Dijkstra?

278. BFS for unweighted shortest path.
     - Counter: Why BFS guarantees shortest path in unweighted graph?
     - Counter: What is level in BFS?

279. Floyd-Warshall — all pairs shortest path.
     - Counter: Time complexity O(V³)?
     - Counter: Detect negative cycle?
     - Counter: When to use over running Dijkstra V times?

280. Network delay time.
     - Counter: Dijkstra from source?
     - Counter: Maximum distance in shortest path tree?
     - Counter: What if graph is disconnected?

---

### INTERMEDIATE

281. Cheapest flights within K stops.
     - Counter: Modified Dijkstra or Bellman-Ford?
     - Counter: Why Dijkstra needs modification?
     - Counter: DP state definition?

282. Path with minimum effort.
     - Counter: Dijkstra with effort as weight?
     - Counter: What is "effort" on an edge?
     - Counter: Binary search + BFS/DFS alternative?

283. Swim in rising water (revisit).
     - Counter: Dijkstra where edge weight = max of two cells?
     - Counter: Binary search on time + BFS?

284. Find city with smallest number of reachable neighbors.
     - Counter: Floyd-Warshall for all-pairs distances?
     - Counter: Run Dijkstra from each city?
     - Counter: Which is better for small dense graph?

285. Minimum cost to reach destination (with forbidden states).
     - Counter: State = (node, extra_info)?
     - Counter: Dijkstra on extended state space?
     - Counter: How to define state transitions?

286. The maze — shortest path.
     - Counter: Ball rolls until hitting wall?
     - Counter: BFS/Dijkstra on resulting graph?
     - Counter: What is the edge weight?

287. Bus routes — minimum number of buses.
     - Counter: BFS where level = bus changes?
     - Counter: Build graph of route connections?
     - Counter: Virtual stop node approach?

288. Shortest path in binary matrix.
     - Counter: BFS from top-left?
     - Counter: 8-directional movement?
     - Counter: A* algorithm applicable here?

289. Word ladder II — all shortest paths.
     - Counter: BFS for distances + DFS/backtrack for paths?
     - Counter: Why BFS first, then backtrack?
     - Counter: Memory for storing all paths?

290. Minimum jumps with fuel.
     - Counter: State = (position, fuel)?
     - Counter: BFS on state space?
     - Counter: Dijkstra if fuel costs vary?

---

## 15. GRAPHS — ADVANCED

### ADVANCED

291. Minimum spanning tree — Kruskal's.
     - Counter: Sort edges + Union-Find?
     - Counter: Why greedy works?
     - Counter: Handle disconnected graph?

292. Minimum spanning tree — Prim's.
     - Counter: Greedy with min-heap?
     - Counter: Kruskal vs Prim — when to use each?
     - Counter: Dense vs sparse graph?

293. Euler path and circuit.
     - Counter: When does Euler circuit exist?
     - Counter: Hierholzer's algorithm?
     - Counter: Euler path vs Hamiltonian path?

294. Hamiltonian path/circuit.
     - Counter: Why is this NP-complete?
     - Counter: Bitmask DP for exact solution?
     - Counter: TSP relationship?

295. Graph coloring — minimum colors.
     - Counter: Greedy coloring?
     - Counter: Chromatic number — how to find exactly?
     - Counter: Bipartite = 2-colorable?

296. Maximum flow — Ford-Fulkerson.
     - Counter: Augmenting path concept?
     - Counter: Residual graph?
     - Counter: What is max-flow min-cut theorem?

297. Minimum cut.
     - Counter: Relationship to max-flow?
     - Counter: How to find actual min-cut edges?
     - Counter: Karger's randomized algorithm?

298. Traveling salesman problem.
     - Counter: Bitmask DP — O(2^n * n²)?
     - Counter: Approximation algorithms?
     - Counter: When does greedy give 2-approximation?

299. Graph isomorphism.
     - Counter: Is polynomial algorithm known?
     - Counter: Tree isomorphism — polynomial?
     - Counter: Weisfeiler-Lehman test?

300. Count paths in DAG.
     - Counter: Topological sort + DP?
     - Counter: Memoized DFS?
     - Counter: Handle long answer with modulo?

---

## 16. DYNAMIC PROGRAMMING — 1D

### BEGINNER

301. Fibonacci with memoization.
     - Counter: Top-down vs bottom-up?
     - Counter: Space optimization to O(1)?
     - Counter: Matrix exponentiation for O(log n)?

302. Climbing stairs — ways to reach top.
     - Counter: How is this same as Fibonacci?
     - Counter: What if you can take up to K steps?
     - Counter: Count vs check existence?

303. House robber — maximum money.
     - Counter: DP recurrence?
     - Counter: O(1) space solution?
     - Counter: Circular houses?

304. Min cost climbing stairs.
     - Counter: Pay cost to leave stair?
     - Counter: What is the base case?
     - Counter: Can you start from stair 0 or 1?

305. Decode ways — number of ways to decode string.
     - Counter: DP state = ways to decode first i characters?
     - Counter: Handle '0' carefully — when does it fail?
     - Counter: Two-digit decode condition?

---

### INTERMEDIATE

306. Longest increasing subsequence (LIS).
     - Counter: O(n²) DP vs O(n log n) patience sorting?
     - Counter: Reconstruct actual LIS?
     - Counter: Number of LIS?

307. Coin change — minimum coins.
     - Counter: DP or BFS?
     - Counter: Unbounded knapsack connection?
     - Counter: Count ways vs minimum coins?

308. Coin change II — count ways.
     - Counter: Order matters (permutations) vs doesn't (combinations)?
     - Counter: 2D DP vs 1D DP?
     - Counter: Why inner loop order matters?

309. Word break — can string be segmented.
     - Counter: DP or BFS?
     - Counter: Trie optimization?
     - Counter: Return all segmentations?

310. Perfect squares — minimum squares summing to N.
     - Counter: DP or BFS?
     - Counter: Math: Legendre's three-square theorem?
     - Counter: Why BFS gives minimum?

311. Jump game II — minimum jumps.
     - Counter: DP vs greedy — which is O(n)?
     - Counter: What is the greedy invariant?
     - Counter: BFS level approach?

312. Ugly numbers — Kth ugly number.
     - Counter: Three-pointer DP?
     - Counter: Min-heap approach?
     - Counter: Why does three-pointer avoid duplicates?

313. Maximum sum increasing subsequence.
     - Counter: How does it differ from LIS?
     - Counter: O(n²) DP?
     - Counter: O(n log n) approach?

314. Longest bitonic subsequence.
     - Counter: LIS from left + LIS from right?
     - Counter: At each index, longest bitonic through it?

315. Number of longest increasing subsequences.
     - Counter: Track both length and count arrays?
     - Counter: When do you update count?

---

### ADVANCED

316. Largest divisible subset.
     - Counter: Sort + LIS-style DP?
     - Counter: Divisibility condition?
     - Counter: Reconstruct actual subset?

317. Russian doll envelopes.
     - Counter: Sort by width ascending, height descending — why?
     - Counter: LIS on heights after sorting?
     - Counter: Why descending height for same width?

318. Minimum operations to make array non-decreasing.
     - Counter: LIS length gives answer?
     - Counter: Patience sort O(n log n)?
     - Counter: Prove reduction to LIS?

319. Tallest billboard.
     - Counter: DP with difference of two subsets?
     - Counter: State = difference between two groups?
     - Counter: Space optimization?

320. Number of dice rolls with target sum.
     - Counter: DP state = (dice used, current sum)?
     - Counter: Transition for each face value?
     - Counter: Modulo arithmetic?

---

## 17. DYNAMIC PROGRAMMING — 2D

### BEGINNER

321. Unique paths in grid.
     - Counter: DP vs combinatorics formula?
     - Counter: With obstacles?
     - Counter: Space optimization to 1D?

322. Minimum path sum in grid.
     - Counter: DP recurrence?
     - Counter: In-place modification?
     - Counter: Path reconstruction?

323. Longest common subsequence (LCS).
     - Counter: 2D DP table?
     - Counter: Reconstruct actual LCS?
     - Counter: Space optimization?

324. Edit distance.
     - Counter: Three operations — insert, delete, replace?
     - Counter: DP recurrence for each operation?
     - Counter: What does each cell mean?

325. 0/1 Knapsack.
     - Counter: 2D DP vs 1D DP?
     - Counter: Why traverse capacity in reverse for 1D?
     - Counter: Reconstruct selected items?

---

### INTERMEDIATE

326. Longest common substring.
     - Counter: Difference from LCS?
     - Counter: No skipping allowed — how does DP change?
     - Counter: Suffix array approach?

327. Wildcard matching.
     - Counter: DP state = (i, j) in pattern and string?
     - Counter: '*' matches zero or more — transition?
     - Counter: Greedy approach — does it work?

328. Regular expression matching.
     - Counter: '.' vs '*' handling?
     - Counter: 'a*' can match zero 'a's?
     - Counter: DP transition for '*'?

329. Interleaving string.
     - Counter: DP with two pointers into s1 and s2?
     - Counter: State = (i, j) — what does it represent?
     - Counter: Relation to LCS?

330. Maximal square of 1s in binary matrix.
     - Counter: DP recurrence — min of three neighbors + 1?
     - Counter: Why min of three?
     - Counter: Count all squares vs largest?

331. Dungeon game — minimum health to reach princess.
     - Counter: Why process right-to-left, bottom-to-top?
     - Counter: What if you process top-to-bottom?
     - Counter: Minimum health at each cell?

332. Cherry pickup — maximum cherries round trip.
     - Counter: Two simultaneous DFS from corner?
     - Counter: State = (r1, c1, r2) — why only three dimensions?
     - Counter: Same cell — count once?

333. Burst balloons.
     - Counter: Interval DP — which balloon to burst last?
     - Counter: Why last not first?
     - Counter: Recurrence relation?

334. Strange printer.
     - Counter: Interval DP?
     - Counter: What does dp[i][j] represent?
     - Counter: Transition when s[i] == s[j]?

335. Paint fence — number of ways.
     - Counter: Same as previous post vs different?
     - Counter: DP with same and diff counts?
     - Counter: Constraint: no 3 consecutive same?

---

## 18. DYNAMIC PROGRAMMING — ADVANCED

336. Matrix chain multiplication.
     - Counter: Interval DP — optimal split point?
     - Counter: How to define cost?
     - Counter: O(n³) complexity?

337. Optimal BST.
     - Counter: Interval DP with frequency?
     - Counter: What is the cost function?
     - Counter: How does root choice affect subtree costs?

338. Partition equal subset sum.
     - Counter: 0/1 knapsack with target = sum/2?
     - Counter: Odd sum — impossible immediately?
     - Counter: Bitset optimization?

339. Last stone weight II — minimize difference.
     - Counter: Same as partition equal subset?
     - Counter: DP with reachable sums?
     - Counter: What is the final answer?

340. Target sum — assign +/- to reach target.
     - Counter: DFS with memo?
     - Counter: Transform to subset sum problem?
     - Counter: Counting problem — not optimization?

341. Profitable schemes.
     - Counter: 3D DP — (schemes, members, profit)?
     - Counter: Modulo arithmetic throughout?
     - Counter: What is the transition?

342. Number of ways to paint N×3 grid.
     - Counter: Three column patterns — how many patterns per row?
     - Counter: Adjacent row compatibility?
     - Counter: O(n) with constant patterns?

343. Count all valid pickup/delivery options.
     - Counter: Math/combinatorics vs DP?
     - Counter: What is the recurrence?
     - Counter: Modular arithmetic?

344. Stone game — who wins optimally.
     - Counter: DP with minimax?
     - Counter: Why first player always wins in even piles?
     - Counter: Prove mathematically?

345. Palindrome removal — minimum moves to empty array.
     - Counter: Interval DP?
     - Counter: When two ends are equal — optimization?
     - Counter: Recurrence relation?

---

## 19. GREEDY ALGORITHMS

### BEGINNER

346. Activity selection — maximum non-overlapping activities.
     - Counter: Sort by end time — why end not start?
     - Counter: Proof of greedy optimality?
     - Counter: With weights — greedy still works?

347. Fractional knapsack.
     - Counter: Sort by value/weight ratio?
     - Counter: Why greedy works here but not 0/1 knapsack?
     - Counter: What if all items can be taken wholly?

348. Minimum number of coins (canonical coin systems).
     - Counter: Greedy works for US coins — why?
     - Counter: Counter-example where greedy fails?
     - Counter: When does greedy equal DP?

349. Gas station — can complete circuit?
     - Counter: Total gas >= total cost implies solution exists?
     - Counter: Where to start — accumulate from each point?
     - Counter: Prove: if total >= 0, starting from min-prefix point works?

350. Assign cookies to children — maximize satisfied.
     - Counter: Sort both, greedy match smallest sufficient?
     - Counter: Two pointer approach?
     - Counter: What if multiple cookies satisfy same child?

---

### INTERMEDIATE

351. Jump game — can reach last index (greedy).
     - Counter: Track maximum reachable index?
     - Counter: Why greedy is correct?
     - Counter: What is the invariant?

352. Merge intervals.
     - Counter: Sort by start time?
     - Counter: When do intervals overlap?
     - Counter: Non-overlapping intervals count?

353. Minimum number of arrows to burst balloons.
     - Counter: Sort by end coordinate?
     - Counter: Shoot at end of first balloon?
     - Counter: Relationship to merge intervals?

354. Queue reconstruction by height.
     - Counter: Sort tall people first, then insert by position?
     - Counter: Why tall people first?
     - Counter: Prove greedy works?

355. Partition labels — maximum partitions where each letter in one part.
     - Counter: Track last occurrence of each character?
     - Counter: Expand partition end as you go?
     - Counter: When do you cut?

356. Minimum number of platforms needed.
     - Counter: Sort arrival and departure separately?
     - Counter: Two pointer sweep?
     - Counter: Difference from meeting rooms?

357. Candy distribution — children with higher rating get more.
     - Counter: Two pass — left to right then right to left?
     - Counter: Why two passes?
     - Counter: O(1) space approach?

358. Non-overlapping intervals — minimum removals.
     - Counter: Greedy: keep interval with earliest end?
     - Counter: Similar to activity selection?
     - Counter: Count removals vs kept?

359. Maximum units on a truck.
     - Counter: Sort by units per box descending?
     - Counter: Greedy fill from most valuable?
     - Counter: Fractional knapsack variant?

360. Broken calculator — reach target from start.
     - Counter: Work backwards from target?
     - Counter: If target even — divide, if odd — add 1?
     - Counter: Prove this is optimal?

---

### ADVANCED

361. Minimum cost to hire K workers.
     - Counter: Enumerate highest quality ratio worker?
     - Counter: For fixed ratio, minimize total quality using heap?
     - Counter: Sort by wage/quality ratio?

362. Remove duplicate letters (lexicographically smallest).
     - Counter: Monotonic stack with last occurrence tracking?
     - Counter: When can you pop from stack?
     - Counter: Prove result is lexicographically smallest?

363. Create maximum number from two arrays.
     - Counter: Enumerate split + max single array subsequence?
     - Counter: Merge two subsequences maintaining order?
     - Counter: Compare and choose?

364. Minimum number of refueling stops.
     - Counter: Greedy with max-heap?
     - Counter: At each stop, decide to refuel or skip?
     - Counter: When you run out, retroactively pick largest past station?

365. IPO maximize capital (revisit deep).
     - Counter: What is the invariant maintained?
     - Counter: Prove greedy is optimal?
     - Counter: What if K is very large?

---

## 20. BIT MANIPULATION

### BEGINNER

366. Check if number is power of 2.
     - Counter: `n & (n-1) == 0` — why?
     - Counter: Handle n = 0?
     - Counter: Power of 4 — additional check?

367. Count set bits (Hamming weight).
     - Counter: `n & (n-1)` trick?
     - Counter: Lookup table approach?
     - Counter: Brian Kernighan's algorithm?

368. Find single number in array (every other appears twice).
     - Counter: XOR all elements — why?
     - Counter: XOR properties?
     - Counter: Every element appears three times — how to find single?

369. Reverse bits of integer.
     - Counter: Bit by bit shift and OR?
     - Counter: Divide and conquer bit reversal?
     - Counter: 32-bit vs 64-bit?

370. Sum of two integers without + operator.
     - Counter: XOR for sum without carry?
     - Counter: AND shifted left for carry?
     - Counter: Loop until no carry?

---

### INTERMEDIATE

371. Find two single numbers (two numbers appear once).
     - Counter: XOR gives XOR of two singles?
     - Counter: How to separate into two groups?
     - Counter: Use any set bit of XOR to partition?

372. Missing number in [0..N].
     - Counter: XOR approach?
     - Counter: Sum formula approach?
     - Counter: What if multiple missing?

373. Number of 1 bits in range [1..N] — count total.
     - Counter: Pattern for each bit position?
     - Counter: O(log n) approach?
     - Counter: Digit DP approach?

374. Bitwise AND of range [m..n].
     - Counter: Common prefix of m and n?
     - Counter: Right shift until equal?
     - Counter: Why does this work?

375. Maximum XOR subarray.
     - Counter: Trie approach?
     - Counter: Prefix XOR?
     - Counter: Linear basis / Gaussian elimination?

376. Single number III — two elements appear once.
     - Counter: XOR then separate by bit — full explanation?

377. Total Hamming distance.
     - Counter: Count 1s at each bit position?
     - Counter: O(32n) approach?
     - Counter: vs O(n²) brute force?

378. Subsets using bitmask.
     - Counter: Enumerate 0 to 2^n — 1?
     - Counter: Each bit represents inclusion?
     - Counter: Time and space complexity?

379. Bit tricks — swap without temp, min/max, etc.
     - Counter: XOR swap — when does it fail?
     - Counter: Overflow in min/max bitmask tricks?

380. Counting bits — for each number in [0..N].
     - Counter: `dp[i] = dp[i >> 1] + (i & 1)` — explain?
     - Counter: Can you do O(n) exactly?

---

## 21. MATH & NUMBER THEORY

### BEGINNER

381. Check prime number.
     - Counter: O(√n) approach?
     - Counter: Why √n is sufficient?
     - Counter: Sieve of Eratosthenes for range?

382. GCD and LCM.
     - Counter: Euclidean algorithm?
     - Counter: `gcd(a,b) = gcd(b, a%b)` — prove?
     - Counter: LCM from GCD?

383. Power function — fast exponentiation.
     - Counter: O(log n) binary exponentiation?
     - Counter: Modular exponentiation?
     - Counter: Handle negative exponent?

384. Factorial trailing zeros.
     - Counter: Count factor of 5?
     - Counter: Why 5 not 2?
     - Counter: `n/5 + n/25 + n/125 + ...`?

385. Reverse integer.
     - Counter: Overflow check?
     - Counter: Modulo and divide approach?
     - Counter: Negative numbers?

---

### INTERMEDIATE

386. Count primes up to N.
     - Counter: Sieve of Eratosthenes?
     - Counter: Linear sieve for O(n)?
     - Counter: Segmented sieve for large N?

387. Palindrome number without string conversion.
     - Counter: Reverse half the digits?
     - Counter: When to stop reversing?
     - Counter: Handle numbers ending in 0?

388. Excel sheet column number.
     - Counter: Base-26 with offset?
     - Counter: Why is 'A' = 1 not 0?
     - Counter: Column title from number?

389. Roman to integer and vice versa.
     - Counter: Subtractive notation — when?
     - Counter: Process right-to-left?
     - Counter: Edge cases IV, IX, XL, XC, CD, CM?

390. Happy number.
     - Counter: Floyd's cycle detection?
     - Counter: Which numbers eventually lead to 1?
     - Counter: Set vs cycle detection?

391. Pow(x, n) — handle negative n.
     - Counter: `x^(-n) = 1/x^n`?
     - Counter: Integer overflow when n = INT_MIN?
     - Counter: Binary exponentiation?

392. Sqrt without math library — Newton's method.
     - Counter: Convergence guarantee?
     - Counter: Integer vs float sqrt?
     - Counter: Binary search approach?

393. Fraction to recurring decimal.
     - Counter: Long division simulation?
     - Counter: Detect cycle in remainders?
     - Counter: Hash map — remainder to position?

394. Find all prime factors.
     - Counter: Trial division up to √n?
     - Counter: What if n itself is prime?
     - Counter: Pollard's rho for large numbers?

395. Modular arithmetic basics.
     - Counter: Modular addition and multiplication?
     - Counter: Modular inverse — Fermat's little theorem?
     - Counter: Chinese remainder theorem?

---

## 22. INTERVALS

396. Meeting rooms — can attend all?
     - Counter: Sort by start, check overlap?
     - Counter: What is overlap condition?

397. Meeting rooms II — minimum rooms needed.
     - Counter: Sort + min-heap of end times?
     - Counter: Sweep line approach?
     - Counter: Equivalence to maximum overlapping intervals?

398. Merge intervals.
     - Counter: Sort by start, merge if overlapping?
     - Counter: When exactly do you merge?
     - Counter: Insert new interval maintaining sorted order?

399. Insert interval.
     - Counter: Find overlapping intervals?
     - Counter: Merge all overlapping with new interval?
     - Counter: Binary search for insertion point?

400. Non-overlapping intervals minimum removal.
     - Counter: Greedy — keep interval with earliest end?
     - Counter: Count kept, subtract from total?
     - Counter: Relationship to LIS?

401. Interval list intersections.
     - Counter: Two pointer on both lists?
     - Counter: Advance pointer with earlier end?
     - Counter: What is intersection condition?

402. Employee free time.
     - Counter: Collect all intervals, sort, find gaps?
     - Counter: Priority queue approach?
     - Counter: What defines a gap?

403. Minimum number of intervals to cover range.
     - Counter: Greedy — pick farthest reaching interval covering current?
     - Counter: Sort by start time?
     - Counter: What if coverage is impossible?

404. Video stitching.
     - Counter: Same as interval coverage?
     - Counter: Sort by start, then pick farthest?
     - Counter: DP approach?

405. Partition into minimum number of disjoint intervals.
     - Counter: Greedy coloring?
     - Counter: Chromatic number of interval graph?
     - Counter: Meeting rooms II connection?

---

## 23. SEGMENT TREES & BIT (Fenwick Tree)

### INTERMEDIATE

406. Range sum query — mutable.
     - Counter: BIT (Fenwick tree) vs segment tree?
     - Counter: BIT update and query?
     - Counter: Why `i += i & (-i)` in BIT?

407. Range minimum query.
     - Counter: Segment tree with lazy propagation?
     - Counter: Sparse table for static array — O(1) query?
     - Counter: Why can't BIT do range minimum easily?

408. Count of smaller numbers after self.
     - Counter: BIT with coordinate compression?
     - Counter: Merge sort approach?
     - Counter: What is coordinate compression?

409. Range sum query 2D — mutable.
     - Counter: 2D BIT?
     - Counter: Update and query in 2D BIT?
     - Counter: Time complexity?

410. Count of range sum.
     - Counter: BIT with coordinate compression on prefix sums?
     - Counter: Merge sort on prefix sums?

---

### ADVANCED

411. Segment tree with lazy propagation.
     - Counter: What is lazy propagation?
     - Counter: When do you push down lazy tags?
     - Counter: Range update + range query?

412. Persistent segment tree.
     - Counter: What is persistence — version history?
     - Counter: How do you share nodes between versions?
     - Counter: Kth smallest in range using persistent tree?

413. Segment tree beats.
     - Counter: Range min/max update?
     - Counter: When does normal lazy fail?

414. Order statistics tree.
     - Counter: Rank and select operations?
     - Counter: Augmented BST with subtree sizes?
     - Counter: STL policy-based tree in C++?

415. Dynamic range sum with offline queries.
     - Counter: Mo's algorithm?
     - Counter: Block decomposition?
     - Counter: When is offline processing beneficial?

---

## 24. UNION FIND / DSU

416. Implement Union-Find.
     - Counter: Union by rank + path compression?
     - Counter: Why path compression helps?
     - Counter: Amortized complexity?

417. Number of connected components.
     - Counter: DSU vs BFS/DFS — when to choose?
     - Counter: Dynamic graph — edges added over time?

418. Redundant connection.
     - Counter: Adding edge that forms cycle?
     - Counter: Return last such edge?

419. Accounts merge.
     - Counter: Union emails by common accounts?
     - Counter: How to get canonical account name?
     - Counter: Graph BFS/DFS alternative?

420. Satisfiability of equality equations.
     - Counter: Union equals equations, check not-equals?
     - Counter: What if a==b and a!=b?

421. Number of islands II (online queries).
     - Counter: DSU with water as separate component?
     - Counter: How to add land cell?
     - Counter: Count components dynamically?

422. Most stones removed with same row or column.
     - Counter: Union stones by row/column?
     - Counter: Stones in same component can be removed?
     - Counter: Answer = N - components?

423. Swim in water (Union-Find).
     - Counter: Add edges in order of max elevation?
     - Counter: Stop when source and destination connected?
     - Counter: Vs Dijkstra approach?

424. Path with maximum probability.
     - Counter: Dijkstra with multiplication?
     - Counter: Log transformation to addition?
     - Counter: BFS with Union-Find?

425. Making a large island.
     - Counter: Label islands with DSU?
     - Counter: For each water cell, compute island sizes if connected?
     - Counter: Handle adjacent islands of same color?

---

## 25. MONOTONIC STACK/QUEUE

426. Next greater element I and II.
     - Counter: Monotonic decreasing stack?
     - Counter: Circular array — double traversal?

427. Sum of subarray minimums.
     - Counter: Previous smaller and next smaller for each element?
     - Counter: Contribution technique?
     - Counter: Modulo arithmetic?

428. Sum of subarray ranges.
     - Counter: Sum of maximums minus sum of minimums?
     - Counter: Monotonic stack for each?

429. Online stock span.
     - Counter: Stack of (price, span) pairs?
     - Counter: When do you pop?
     - Counter: What does span represent?

430. Maximum width ramp.
     - Counter: Build decreasing stack of candidates?
     - Counter: Then scan from right?
     - Counter: Prove correctness?

431. 132 pattern.
     - Counter: Find 1-3-2 subsequence?
     - Counter: Process right-to-left, maintain stack?
     - Counter: What invariant does stack maintain?

432. Maximum score of a good subarray.
     - Counter: Monotonic stack?
     - Counter: Two-pointer approach?
     - Counter: Similar to container with most water?

433. Jump game VI — maximum score jumping.
     - Counter: DP with deque for max in window?
     - Counter: Segment tree approach?
     - Counter: What is the window size?

434. Minimum number of visible people in queue.
     - Counter: Monotonic decreasing stack?
     - Counter: When can you see a person?

435. Removing stars from string.
     - Counter: Stack simulation?
     - Counter: What does star do to stack?
     - Counter: Build result from stack?

---

## 26. STRING ALGORITHMS

### INTERMEDIATE

436. KMP string matching.
     - Counter: What is failure function?
     - Counter: How does failure function enable O(n) matching?
     - Counter: Build failure function algorithm?

437. Rabin-Karp rolling hash.
     - Counter: What is rolling hash?
     - Counter: How do you handle hash collision?
     - Counter: When is Rabin-Karp better than KMP?

438. Z-algorithm.
     - Counter: What does Z-array represent?
     - Counter: How to compute Z-array in O(n)?
     - Counter: Application to string matching?

439. Manacher's algorithm — all palindromic substrings.
     - Counter: How does Manacher achieve O(n)?
     - Counter: What is the P-array?
     - Counter: Center and right boundary maintenance?

440. Aho-Corasick — multiple pattern matching.
     - Counter: Trie + failure links?
     - Counter: How does it differ from multiple KMP runs?
     - Counter: Time complexity for text length n, patterns total length m?

---

### ADVANCED

441. Suffix array construction.
     - Counter: Naive O(n² log n) vs DC3/SA-IS O(n)?
     - Counter: What is LCP array?
     - Counter: Application: longest repeated substring?

442. Suffix automaton.
     - Counter: What is it and when to use?
     - Counter: States and transitions?
     - Counter: Count distinct substrings?

443. Longest palindromic substring — Manacher.
     - Counter: O(n) vs O(n²) DP approach?
     - Counter: When to use each?

444. Shortest palindrome (add characters to front).
     - Counter: KMP on `s + "#" + reverse(s)`?
     - Counter: What does failure function give you?

445. Minimum window containing all characters.
     - Counter: Full solution walkthrough?
     - Counter: Character frequency tracking?
     - Counter: When do you have a valid window?

---

## 27. ADVANCED DATA STRUCTURES

446. Skip list — design and complexity.
     - Counter: Expected O(log n) for all operations?
     - Counter: How do you decide level for new node?
     - Counter: Comparison with balanced BST?

447. Fibonacci heap.
     - Counter: Why used in Dijkstra's algorithm?
     - Counter: Amortized decrease-key O(1)?
     - Counter: Practical performance vs binary heap?

448. B-tree and B+ tree.
     - Counter: Why used in databases and file systems?
     - Counter: Difference between B and B+?
     - Counter: How does B-tree reduce disk I/O?

449. Disjoint set with rollback.
     - Counter: Why path compression breaks rollback?
     - Counter: Union by rank without path compression?
     - Counter: Undo operation?

450. Van Emde Boas tree.
     - Counter: O(log log U) for integer keys?
     - Counter: What is the recursive structure?
     - Counter: Memory usage?

---

## 28. SYSTEM DESIGN DSA

451. Design LRU cache.
     - Counter: Thread-safe version?
     - Counter: Distributed LRU?
     - Counter: Segmented LRU (SLRU)?

452. Design LFU cache.
     - Counter: O(1) for all operations?
     - Counter: How to handle frequency tie?
     - Counter: Real-world vs LRU?

453. Design a rate limiter.
     - Counter: Token bucket vs leaky bucket vs sliding window?
     - Counter: Distributed rate limiter?
     - Counter: Data structure choices?

454. Design a URL shortener.
     - Counter: Hash function choice?
     - Counter: Collision handling?
     - Counter: Scale to billions of URLs?

455. Design a search autocomplete system.
     - Counter: Trie with frequency?
     - Counter: Top-K results efficiently?
     - Counter: Real-time update of frequencies?

456. Design a hit counter.
     - Counter: Rolling window of last 5 minutes?
     - Counter: Circular buffer approach?
     - Counter: Distributed hit counter?

457. Design a data structure for stream median.
     - Counter: Two heaps approach?
     - Counter: What if stream is too large for memory?
     - Counter: Approximate median with quantiles?

458. Design a data structure for stock price history.
     - Counter: Current price, maximum price, minimum price, find closest price?
     - Counter: What data structures combine here?

459. Implement HashMap from scratch.
     - Counter: Hash function — what makes a good one?
     - Counter: Collision resolution — chaining vs open addressing?
     - Counter: Dynamic resizing — load factor?

460. Design consistent hashing.
     - Counter: Why is it better than modular hashing for distributed systems?
     - Counter: Virtual nodes?
     - Counter: How do you handle node addition/removal?

---

## 29. EXPERT / COMPETITIVE LEVEL

461. Minimum cost to cut a stick.
     - Counter: Interval DP?
     - Counter: Transform cuts to interval endpoints?

462. Number of ways to reorder array to get same BST.
     - Counter: Relative order of left and right subtree elements?
     - Counter: Combinatorics + recursion?
     - Counter: Modular arithmetic?

463. Maximum score from performing multiplication operations.
     - Counter: DP with left and right pointers?
     - Counter: State = (left pointer, operations done)?

464. Selling diminishing-valued colored balls.
     - Counter: Greedy with heap?
     - Counter: Math: sell equal groups?

465. Count ways to build rooms in an ant colony.
     - Counter: Tree DP with subtree sizes?
     - Counter: Combinatorics on tree?

466. Strange Printer II — color layers.
     - Counter: Topological sort on color dependencies?
     - Counter: When can you print color A before B?

467. Minimum obstacle removal to reach corner.
     - Counter: 0-1 BFS (deque)?
     - Counter: Dijkstra with 0 or 1 cost?
     - Counter: Why 0-1 BFS is O(V+E) vs Dijkstra O(V log V)?

468. Minimum number of operations to make array continuous.
     - Counter: Sort unique elements?
     - Counter: Sliding window of size N?
     - Counter: Elements already in window need no change?

469. Count number of texts.
     - Counter: Similar to decode ways but with phone keypad?
     - Counter: How many repeats allowed per key?
     - Counter: Modular DP?

470. Maximum number of non-overlapping palindrome substrings.
     - Counter: Greedy from left?
     - Counter: DP for palindrome check?
     - Counter: Why greedy works — proof?

471. Recover the original array.
     - Counter: Sort, try each candidate for minimum k?
     - Counter: Check if valid pairing exists?
     - Counter: Hash map for matching?

472. Count subarrays with median K.
     - Counter: Transform: > K becomes +1, < K becomes -1?
     - Counter: Count prefix sums?

473. Maximum strength of a group.
     - Counter: All positives + even number of negatives?
     - Counter: Handle zeros?
     - Counter: Sort and decide inclusion?

474. Find the substring with exactly K distinct characters and given frequency.
     - Counter: Sliding window?
     - Counter: Frequency constraint simultaneously?

475. Minimum number of swaps to make string balanced.
     - Counter: Count unbalanced brackets?
     - Counter: Each swap fixes two unbalanced?

476. Minimum moves to reach target score.
     - Counter: Work backwards from target?
     - Counter: Divide when even, subtract 1 when odd?
     - Counter: Prove optimality?

477. Count palindromic subsequences.
     - Counter: DP — expand from each center or shrink from ends?
     - Counter: Distinct vs total count?

478. Maximum number of accepted invitations.
     - Counter: Bipartite matching?
     - Counter: Hungarian algorithm?
     - Counter: Hopcroft-Karp?

479. Number of valid move combinations on chessboard.
     - Counter: Backtracking with collision detection?
     - Counter: Check all future positions simultaneously?

480. Minimum weighted vertex cover.
     - Counter: NP-hard in general?
     - Counter: On tree — tree DP?
     - Counter: On bipartite graph — König's theorem?

---

## 30. GOOGLE-SPECIFIC PATTERNS

481. Problems with follow-up: "What if input is too large for memory?"
     - Counter: External sort, streaming algorithms?
     - Counter: Approximate algorithms?
     - Counter: Distributed processing?

482. Problems with follow-up: "What if you need to handle concurrent access?"
     - Counter: Lock-free data structures?
     - Counter: Concurrent hash map design?
     - Counter: Read-write lock tradeoffs?

483. Problems with follow-up: "Optimize your solution further."
     - Counter: Can you reduce space?
     - Counter: Can you reduce time by one log factor?
     - Counter: Can you use SIMD/hardware optimization?

484. Problems with follow-up: "What if the graph has 10^9 nodes?"
     - Counter: Implicit graph representation?
     - Counter: BFS/DFS without storing all nodes?
     - Counter: Bidirectional search?

485. Problems with follow-up: "Prove your greedy is optimal."
     - Counter: Exchange argument?
     - Counter: Matroid structure?
     - Counter: Contradiction proof?

486. Design a system to find top K queries in last hour.
     - Counter: Streaming top-K algorithms?
     - Counter: Count-Min sketch?
     - Counter: Space Saving algorithm?

487. Find the kth largest in a stream of numbers.
     - Counter: Min-heap of size K?
     - Counter: What if K changes dynamically?
     - Counter: Approximate Kth using sampling?

488. Nearest neighbor search in high dimensions.
     - Counter: KD-tree — curse of dimensionality?
     - Counter: LSH (Locality-Sensitive Hashing)?
     - Counter: Approximation vs exact?

489. Design a spell checker.
     - Counter: Trie for dictionary?
     - Counter: Edit distance for suggestions?
     - Counter: BK-tree for approximate search?

490. Optimal strategy for matrix chain multiplication in distributed system.
     - Counter: Communication cost model?
     - Counter: How does data placement affect cost?

491. Find shortest path in maze with teleportation edges.
     - Counter: BFS with teleport treated as edges?
     - Counter: Dijkstra if costs vary?
     - Counter: Can teleportation create negative cycles?

492. Maximum flow with lower bounds.
     - Counter: Reduce to standard max-flow?
     - Counter: Circulation with demands?
     - Counter: Feasibility check first?

493. Minimum cost maximum flow.
     - Counter: Successive shortest paths?
     - Counter: SPFA vs Dijkstra for cost?
     - Counter: Bellman-Ford for negative costs?

494. Online algorithms — competitive ratio.
     - Counter: Ski rental problem — when to buy vs rent?
     - Counter: Online caching — FIFO vs LRU competitive ratio?
     - Counter: What is competitive ratio?

495. Randomized algorithms — when and why.
     - Counter: QuickSort vs deterministic — expected vs worst case?
     - Counter: Bloom filter — false positive rate?
     - Counter: Randomized hash — universal hashing?

496. Approximation algorithms.
     - Counter: Vertex cover 2-approximation?
     - Counter: Set cover greedy — log n approximation?
     - Counter: When is approximation acceptable?

497. Parallel algorithms — map-reduce thinking.
     - Counter: Which sorting algorithms parallelize well?
     - Counter: Parallel BFS — level synchronization?
     - Counter: Work vs depth complexity model?

498. Cache-oblivious algorithms.
     - Counter: What is cache-oblivious model?
     - Counter: Recursive divide and conquer for cache efficiency?
     - Counter: Van Emde Boas tree layout?

499. What is amortized analysis — explain with examples.
     - Counter: Aggregate method?
     - Counter: Accounting method?
     - Counter: Potential function method?

500. Design an algorithm to detect plagiarism in code submissions.
     - Counter: AST-based comparison?
     - Counter: Rolling hash for similar blocks?
     - Counter: What transformations do students apply to evade detection?

---
