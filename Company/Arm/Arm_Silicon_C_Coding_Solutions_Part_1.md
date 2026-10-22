# Arm & Silicon Companies C Coding Practice — Detailed Solutions
## Part 1: Core C Screening Problems (with step-by-step walkthroughs)

**How to use this guide:** These are practice problems based on common C, systems, and firmware screening themes. They are not claimed to be the exact or leaked questions from any company's assessment. Try each problem yourself first, then read the walkthrough.

The explanations use a story-like approach: what the program knows at the beginning, what changes after each important step, and why the result is correct.

---

## 1. Find the Temperature Closest to Zero

### Problem

You receive an array of integer temperatures. Return the temperature closest to zero. If two temperatures are equally close, return the positive temperature. If the array is empty, return `0`.

Example: `[-8, -2, 2, 5]` returns `2`, because `-2` and `2` are equally far from zero, and the positive one wins.

### C solution

```c
#include <stddef.h>

int closest_to_zero(const int temperatures[], size_t count)
{
    if (count == 0)
    {
        return 0; // No temperatures available
    }

    int closest = temperatures[0]; // Initial closest = -8
    long long current_distance = 0;
    long long closest_distance = 0;

    for (size_t i = 1; i < count; i++)
    {
        int current = temperatures[i];

        // Convert both values to positive distances from zero.
        // Cast to long long BEFORE negation to safely handle INT_MIN.
        if (current < 0)
        {
            // Example: current = -2147483648 (INT_MIN for a 32-bit int).
            // If we write -current, the result would be 2147483648,
            // which cannot fit in a 32-bit int, causing undefined behavior.
            //
            // (long long)current converts the value to a wider type FIRST.
            // Then negation happens using long long:
            // -(-2147483648LL) = 2147483648LL
            //
            // This avoids signed integer overflow.
            current_distance = -(long long)current;
        }
        else
        {
            // Non-negative values can be converted directly.
            // Example: current = 25 -> current_distance = 25.
            current_distance = (long long)current;
        }

        if (closest < 0)
        {
            // Example: current = -2147483648 (INT_MIN for a 32-bit int).
            // If we write -current, the result would be 2147483648,
            // which cannot fit in a 32-bit int, causing undefined behavior.
            //
            // (long long)current converts the value to a wider type FIRST.
            // Then negation happens using long long:
            // -(-2147483648LL) = 2147483648LL
            //
            // This avoids signed integer overflow.
            closest_distance = -(long long)closest;
        }
        else
        {
            // Non-negative values can be converted directly.
            // Example: current = 25 -> current_distance = 25.
            closest_distance = (long long)closest;
        }
        // Choose the smaller distance.
        // If distances are equal, choose the positive temperature.
        if ((current_distance < closest_distance) ||
            ((current_distance == closest_distance) &&
             (current > closest)))
        {
            closest = current;
        }

        /*
         * DRY RUN for {-8, -2, 2, 5}:
         *
         * i = 1: current = -2, current_distance = 2,
         *        closest = -8, closest_distance = 8
         *        2 < 8 -> TRUE
         *        closest = -2
         *
         * i = 2: current = 2, current_distance = 2,
         *        closest = -2, closest_distance = 2
         *        2 < 2 -> FALSE
         *        2 == 2 && 2 > -2 -> TRUE
         *        closest = 2
         *
         * i = 3: current = 5, current_distance = 5,
         *        closest = 2, closest_distance = 2
         *        5 < 2 -> FALSE
         *        5 == 2 -> FALSE
         *        closest remains 2
         *
         * Final return: 2
         */
    }

    return closest; // Result: 2
}
```

### Walkthrough, step by step

Imagine `closest` is the current champion in a contest: the champion is whichever temperature is nearest to zero. We start with the first element because we need a real candidate before we can compare anything.

For `[-8, -2, 2, 5]`, the first value is `-8`, so initially `closest = -8`. Its distance from zero is `8`.

At index `1`, `current = -2`. Its distance is `2`, which is smaller than `8`, so `-2` becomes the new champion. Now `closest = -2`.

At index `2`, `current = 2`. Its distance is also `2`. The distances tie, so we apply the tie rule: choose the greater numerical value. Since `2 > -2`, the answer changes from `-2` to `2`.

At index `3`, `current = 5`. Its distance is `5`, which is larger than the champion's distance `2`, so `closest` remains `2`. The function returns `2`.

The code converts values to `long long` before negating them. That detail matters because the most negative `int` may not have a positive counterpart representable by `int`; blindly calculating `-INT_MIN` can overflow. A screening answer should explain both the algorithm and this edge case.

**Complexity:** \(O(n)\) time because each temperature is visited once, and \(O(1)\) extra space.

---

## 2. Reverse a String In Place

### Problem

Reverse a writable, null-terminated C string without allocating another string.

Example: `"ARM"` becomes `"MRA"`.

### C solution

```c

#include <stddef.h>
#include <string.h>

void reverse_string(char s[])
{
    // If s is NULL, no valid string is available to reverse.
    // Return immediately to avoid accessing invalid memory.
    if (s == NULL) {
        return;
    }

    // strlen() returns the number of characters before '\0'.
    // Example: "ARM" has length 3.
    // Index:    0    1    2
    // String:  'A'  'R'  'M' '\0'
    size_t len = strlen(s);

    // left starts at the first character.
    size_t left = 0U;

    // If the string is empty, there is nothing to reverse.
    if (len == 0U) {
        return;
    }

    // right points to the last actual character, not '\0'.
    // For "ARM", right = 3 - 1 = 2.
    size_t right = len - 1U;

    // Continue swapping while left is before right.
    // Once the pointers meet or cross, reversal is complete.
    while (left < right) {

        // STEP 1: Save the left character temporarily.
        // First iteration: temp = 'A'.
        char temp = s[left];

        // STEP 2: Copy the right character to the left position.
        // First iteration: s[0] = 'M'.
        // String becomes "MRM".
        s[left] = s[right];

        // STEP 3: Put the saved left character at the right position.
        // First iteration: s[2] = 'A'.
        // String becomes "MRA".
        s[right] = temp;

        // Move left one position to the right.
        // Next iteration: left = 1.
        ++left;

        // Move right one position to the left.
        // Next iteration: right = 1.
        --right;
    }

    // For "ARM":
    // Iteration 1: swap indexes 0 and 2 -> "MRA".
    // Iteration 2 check: left = 1, right = 1.
    // left < right is false, so the loop stops.
    // Final string: "MRA".
}

```

### Walkthrough, step by step

Think of two people standing at opposite ends of a line of letters. The person on the left and the person on the right exchange places, then both move one position inward. They stop when they meet or cross.

For `"ARM"`, the indices are `0:A`, `1:R`, and `2:M`. Initially `left = 0` and `right = 2`. The first swap stores `A` in `temp`, puts `M` at index `0`, and then puts the saved `A` at index `2`. The string is now `"MRA"`. We increment `left` to `1` and decrement `right` to `1`. Since `left < right` is now false, the loop stops. The middle character `R` does not need to move.

For an even-length string such as `"CODE"`, the first swap produces `"EODC"`, then the second swap exchanges `O` and `D`, producing `"EDOC"`. The pointers cross after that, and the loop ends.

The `len == 0` check prevents `len - 1` from underflowing. Since `size_t` is unsigned, if `len` were zero and we calculated `len - 1`, it would wrap to a very large value. We also check for a null pointer before calling `strlen`, because `strlen(NULL)` is undefined behavior.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

**Interview note:** This assumes `s` points to writable memory and contains a terminating `'\0'`. It cannot safely reverse a string literal such as `char *s = "ARM";`, because modifying a string literal is undefined behavior. Use `char s[] = "ARM";` for a writable array.

---

## 3. Count the Number of Set Bits

### Problem

Count how many bits are `1` in an unsigned integer.

Example: `13` is binary `1101`, so the answer is `3`.

### C solution

```c
/*
This algorithm uses Brian Kernighan’s algorithm. The key idea is that value & (value - 1U) removes the rightmost set bit (1) from the binary number in every iteration.
*/
unsigned int count_set_bits(unsigned int value)
{
    // Initially, we have not counted any set bits.
    unsigned int count = 0U;

    // Continue until all the set bits have been removed.
    // When value becomes 0, no more bits are equal to 1.
    while (value != 0U) {

        // Subtracting 1 flips the rightmost set bit to 0
        // and changes any trailing 0s into 1s.
        //
        // Example:
        // value     = 1101 (13)
        // value - 1 = 1100 (12)
        //
        // Bitwise AND:
        //   1101
        // & 1100
        // -------
        //   1100
        //
        // The rightmost set bit has been removed.
        /* value &= (value - 1U); */
        unsigned int minus1Value = value - 1U
        unsigned int result = value & minus1Value;
        value = result;

        // One set bit was removed in this iteration,
        // so increase the number of set bits counted.
        ++count;
    }

    // Return the total number of bits that were 1.
    return count;
}

// --------------------------------------------------------

unsigned int count_set_bits(unsigned int value)
{
    unsigned int count = 0U;

    // Check every bit until no bits remain.
    while (value != 0U)
    {
        // Check whether the rightmost bit is 1.
        // value & 1U extracts only the least significant bit.
        //
        // Example:
        // value = 1101
        // value & 0001 = 0001 -> bit is 1
        if ((value & 1U) != 0U)
        {
            ++count;  // Count this set bit.
        }

        // Shift all bits one position to the right.
        // The bit we just checked is discarded.
        //
        // Example:
        // 1101 >> 1 = 0110
        value >>= 1U;
    }

    return count;
}


```

### Walkthrough, step by step

The trick is that `value & (value - 1)` clears the rightmost set bit. Rather than checking every bit position, the loop removes one `1` at a time. Each successful removal increases `count` by one.

Start with `value = 13`, which is binary `1101`, and `count = 0`.

On the first iteration, `value - 1` is `12`, binary `1100`. ANDing `1101` with `1100` gives `1100`, which is decimal `12`. The rightmost `1` has disappeared, so `count` becomes `1`.

On the second iteration, `value = 12` (`1100`). Subtracting one gives `11` (`1011`). The AND result is `1000`, decimal `8`. One more set bit was cleared, so `count = 2`.

On the third iteration, `value = 8` (`1000`). Subtracting one gives `7` (`0111`). The AND result is `0000`, so `value` becomes zero and `count = 3`.

The loop ends because there are no set bits left. It returns `3`. This method is efficient for sparse values because it loops only once per set bit.

**Complexity:** \(O(k)\), where \(k\) is the number of set bits; worst case \(O(w)\), where \(w\) is the width of `unsigned int`.

**Why use unsigned?** Bitwise operations are generally easier to reason about with unsigned integers. Signed shifts and signed overflow introduce extra pitfalls.

---

## 4. Set, Clear, Toggle, and Test a Bit

### Problem

Given an unsigned register-like value and a bit index `n`, demonstrate the common bit operations:

- Set bit `n` to `1`.
- Clear bit `n` to `0`.
- Toggle bit `n`.
- Test whether bit `n` is `1`.

### C solution

```c

#include <stdbool.h>
#include <limits.h>

// Check whether bit index n is valid for unsigned int.
// Example: if unsigned int is 32 bits, valid indexes are 0 to 31.
bool bit_index_valid(unsigned int n)
{
    return n < sizeof(unsigned int) * CHAR_BIT;
}

// SET BIT: Make bit n equal to 1.
// Example: set bit 2 in 00001000 -> 00001100.
unsigned int set_bit(unsigned int value, unsigned int n)
{
    // 1U << n creates a mask with only bit n set.
    // OR (|) makes that bit 1 without changing other bits.
    return value | (1U << n);
}

// CLEAR BIT: Make bit n equal to 0.
// Example: clear bit 2 in 00001100 -> 00001000.
unsigned int clear_bit(unsigned int value, unsigned int n)
{
    // 1U << n creates a mask with bit n = 1.
    // ~ inverts the mask, making bit n = 0 and other bits = 1.
    // AND (&) clears bit n and preserves all other bits.
    return value & ~(1U << n);
}

// TOGGLE BIT: Change bit n from 0 to 1 or from 1 to 0.
unsigned int toggle_bit(unsigned int value, unsigned int n)
{
    // XOR (^) changes a bit when the corresponding mask bit is 1.
    // If bit n is 0, it becomes 1; if it is 1, it becomes 0.
    return value ^ (1U << n);
}

// TEST BIT: Check whether bit n is 1.
bool test_bit(unsigned int value, unsigned int n)
{
    // AND (&) isolates bit n.
    // If the result is nonzero, bit n is 1; otherwise it is 0.
    return (value & (1U << n)) != 0U;
}

```

### Walkthrough, step by step

Suppose `value = 10`, which is binary `1010`, and we want to operate on bit `0` (the rightmost bit). The mask is `1U << 0`, which is binary `0001`.

To **set** the bit, OR the value with the mask: `1010 | 0001 = 1011`, so the result is `11`. OR keeps a bit at `1` if either input has a `1` there.

To **clear** the bit, invert the mask and AND it with the value. The mask `0001` becomes `...1110` after `~`. AND preserves all other bits but forces bit zero to zero: `1010 & ...1110 = 1010`. Since bit zero was already zero, the value remains `10`.

To **toggle** the bit, XOR the value with the mask: `1010 ^ 0001 = 1011`. XOR flips the selected bit: zero becomes one and one becomes zero.

To **test** the bit, AND with the mask. `1010 & 0001` equals zero, so bit zero is not set. If the result is nonzero, the bit is set.

In production code, validate the bit index before shifting. Shifting by an amount greater than or equal to the width of the left operand is undefined behavior in C. The four operation functions above assume the caller has already validated `n`; a robust public API should reject invalid indices rather than blindly shifting.

**Complexity:** \(O(1)\) time and \(O(1)\) extra space.

---

## 5. Find the Second-Largest Distinct Number

### Problem

Return the second-largest **distinct** value in an integer array. For `[4, 1, 9, 9, 7]`, the answer is `7`, not `9`. Decide how your function signals that no second-largest distinct value exists.

### C solution

This version returns `true` when a result exists and writes the answer through `out`.

```c

#include <stdbool.h>
#include <stddef.h>
#include <limits.h>

bool second_largest(const int a[], size_t n, int *out)
{
    // Return false if the array or output pointer is NULL,
    // or if fewer than two elements are provided.
    if (a == NULL || out == NULL || n < 2U) {
        return false;
    }

    int largest = INT_MIN;
    int second = INT_MIN;

    // Flags indicate whether valid values have been found.
    bool have_largest = false;
    bool have_second = false;

    // DRY RUN INPUT: {4, 1, 9, 9, 7}
    //
    // Initially:
    // largest = INT_MIN, second = INT_MIN
    // have_largest = false, have_second = false

    for (size_t i = 0U; i < n; ++i) {
        int current = a[i];

        // ITERATION 1: i = 0, current = 4
        // !have_largest is true because no largest ecurrentists yet.
        // Set largest = 4 and have_largest = true.
        // State: largest = 4, second = INT_MIN

        // ITERATION 2: i = 1, current = 1
        // current > largest? 1 > 4 -> false.
        // current < largest? 1 < 4 -> true.
        // !have_second is true because second is not found yet.
        // Set second = 1 and have_second = true.
        // State: largest = 4, second = 1

        // ITERATION 3: i = 2, current = 9
        // current > largest? 9 > 4 -> true.
        // The previous largest (4) becomes second.
        // Set second = 4, then largest = 9.
        // State: largest = 9, second = 4

        // ITERATION 4: i = 3, current = 9
        // current > largest? 9 > 9 -> false.
        // current < largest? 9 < 9 -> false.
        // Both conditions fail, so the duplicate 9 is ignored.
        // State: largest = 9, second = 4

        // ITERATION 5: i = 4, current = 7
        // current > largest? 7 > 9 -> false.
        // current < largest? 7 < 9 -> true.
        // current > second? 7 > 4 -> true.
        // Update second = 7.
        // State: largest = 9, second = 7

        if ((have_largest == false) || (current > largest)) 
        {
            if (have_largest) 
            {
                // Move the previous largest into second place.
                second = largest;
                have_second = true;
            }
            largest = current;
            have_largest = true;
        } 
        else if ((current < largest) && ((have_second == false) || (current > second)))
        {
            // Update second only when current is distinct from largest
            // and is larger than the current second candidate.
            second = current;
            have_second = true;
        }
    }

    // If no second distinct value was found, report failure.
    // Example: {5, 5, 5} has no second-largest distinct value.
    if (have_second == false) 
    {
        return false;
    }

    // Store the answer in the caller's variable.
    *out = second;

    // For {4, 1, 9, 9, 7}:
    // largest = 9, second = 7.
    // Function returns true, and *out contains 7.
    return true;
}

```

### Walkthrough, step by step

We keep two running values: `largest`, the best value seen so far, and `second`, the best distinct value below it. The two Boolean flags tell us whether those values have actually been found. This is safer than using `INT_MIN` alone as a sentinel, because `INT_MIN` itself might be a valid array element.

For `[4, 1, 9, 9, 7]`, the first element `4` becomes the largest. There is no second value yet. Next, `1` is below `4`, so `second = 1`. When we reach `9`, it is larger than `4`; the old largest (`4`) becomes the second-largest, and `9` becomes the largest. The next `9` is equal to the largest, so it is ignored: the problem asks for distinct values. Finally, `7` is below `9` but greater than the current second value `4`, so `second` becomes `7`. The function returns success and writes `7` to `*out`.

If the array is `[5, 5, 5]`, every value equals the largest and there is no distinct second-largest number. The function returns `false` rather than inventing an answer.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

**C API lesson:** A Boolean status return plus an output pointer separates “the operation succeeded” from “the computed value.” It also avoids confusing a legitimate number with an error sentinel.

---

## 6. Find the Missing Number from `0` to `n`

### Problem

An array contains `n` distinct numbers from the range `0` through `n`, with one number missing. Find the missing value.

Example: `[3, 0, 1]` represents the range `0, 1, 2, 3`; the missing value is `2`.

### C solution using XOR

```c
#include <stddef.h>

unsigned int missing_number(const unsigned int a[], size_t n)
{
    unsigned int result = (unsigned int)n;  // n = 3, result = 3

    for (size_t i = 0U; i < n; ++i) 
    {

        // ITERATION 1: i = 0
        // result = 3 ^ 0 = 011 ^ 000 = 011 (3)
        //
        // ITERATION 2: i = 1
        // result = 0 ^ 1 = 000 ^ 001 = 001 (1)
        //
        // ITERATION 3: i = 2
        // result = 1 ^ 2 = 001 ^ 010 = 011 (3)
        // The first statement contributes the expected numbers 0, 1, 2. The initial value n contributes 3. 
        result ^= (unsigned int)i;
       
        // ITERATION 1: a[0] = 3
        // result = 3 ^ 3 = 011 ^ 011 = 000 (0)
        //
        // ITERATION 2: a[1] = 0
        // result = 1 ^ 0 = 001 ^ 000 = 001 (1)
        //
        // ITERATION 3: a[2] = 1
        // result = 3 ^ 1 = 011 ^ 001 = 010 (2)
        // The second statement contributes the actual array elements 3, 0, 1.
        result ^= a[i];
    }

    return result; // Missing number = 2
}
```

### Walkthrough, step by step

XOR has two useful properties: `x ^ x` becomes zero, and `x ^ 0` remains `x`. If we XOR every number in the full range with every number present in the array, all numbers that appear in both places cancel out. Only the missing number remains.

For `[3, 0, 1]`, the array length is `3`, so the full range is `0, 1, 2, 3`. We initialize `result = 3`, representing the final range element. At index zero, XOR with index `0` and array value `3`: the value `3` cancels with the initial `3`, leaving `0`. At index one, XOR with index `1` and array value `0`: XORing `1` makes the result `1`, and XORing `0` changes nothing. At index two, XOR with index `2` and array value `1`: `1 ^ 2 ^ 1` reduces to `2`, because the two `1` values cancel. The function returns `2`.

This avoids calculating `n * (n + 1) / 2`, which can overflow for sufficiently large input values. It assumes the input satisfies the problem's rules: every array value is in the range and all present values are distinct.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

---

## 7. Binary Search in a Sorted Array

### Problem

Given a sorted array in ascending order and a target, return the target's index or `-1` if it is absent.

### C solution

```c

#include <stddef.h>
#include <stdint.h>
#include <limits.h>

int binary_search(const int a[], size_t n, int target)
{
    // If the array pointer is NULL, searching is not possible.
    if (a == NULL) {
        return -1;
    }

    // left is the first index to search.
    size_t left = 0U;

    // right is an EXCLUSIVE upper bound.
    // For an array of 5 elements, valid indexes are 0 to 4,
    // so the initial search range is [0, 5).
    size_t right = n;

    // Example: a = {1, 3, 5, 7, 9}, target = 7, n = 5.
    //
    // ITERATION 1:
    // left = 0, right = 5
    // mid = 0 + (5 - 0) / 2 = 2
    // a[2] = 5
    // 5 == 7? No.
    // 5 < 7? Yes, so discard indexes 0 through 2.
    // Update left = mid + 1 = 3.
    //
    // ITERATION 2:
    // left = 3, right = 5
    // mid = 3 + (5 - 3) / 2 = 4
    // a[4] = 9
    // 9 == 7? No.
    // 9 < 7? No, so discard index 4 and everything after it.
    // Update right = mid = 4.
    //
    // ITERATION 3:
    // left = 3, right = 4
    // mid = 3 + (4 - 3) / 2 = 3
    // a[3] = 7
    // a[3] == target -> found!
    // Return index 3.

    while (left < right) 
    {

        // Find the middle index without calculating left + right,
        // which could overflow for very large index values.
        size_t mid = left + (right - left) / 2U;

        // If the middle element matches the target, return its index.
        if (a[mid] == target) {

            // The function returns int, so ensure mid fits in int.
            if (mid > (size_t)INT_MAX) {
                return -1;
            }

            return (int)mid;

        } 
        else if (a[mid] < target)
        {

            // The middle element is too small.
            // Search only the right half.
            // mid itself cannot be the answer, so use mid + 1.
            left = mid + 1U;

        } 
        else 
        {

            // The middle element is too large.
            // Search only the left half.
            // right is exclusive, so set right = mid.
            right = mid;
        }
    }

    // left == right means the search range is empty.
    // The target was not found.
    return -1;
}

```

### Walkthrough, step by step

Binary search works like looking up a word in a dictionary. Instead of checking every page, we open the middle and decide which half can still contain the answer. This only works if the array is sorted.

Take `[2, 4, 7, 10, 13, 18, 21]` and target `13`. Initially, the search range is indices `0` through `6`, represented in this implementation as `[left, right)` or `left = 0`, `right = 7`. The middle index is `0 + (7 - 0) / 2 = 3`. The value at index `3` is `10`. Because `10 < 13`, none of the indices at or below `3` can contain `13`, so we move `left` to `4`.

Now the remaining range is indices `4` through `6`. The middle is `4 + (7 - 4) / 2 = 5`, whose value is `18`. Since `18 > 13`, index `5` and everything to its right are too large, so `right` becomes `5`.

The remaining range is indices `4` only (`left = 4`, `right = 5`). The middle is `4`, and `a[4]` is `13`, so we return index `4`.

Notice the midpoint formula `left + (right - left) / 2`. It avoids the possible overflow of `(left + right) / 2`. The implementation uses an exclusive upper bound: `right` is one past the final candidate. That convention makes the loop condition `left < right` and the updates easy to reason about.

**Complexity:** \(O(\log n)\) time and \(O(1)\) extra space.

**API caveat:** Returning an index as `int` limits the maximum representable index. In most coding assessments, arrays are small enough that this is irrelevant; in production code, prefer `ptrdiff_t` or a status-plus-output design when very large arrays are possible.

---

## 8. First Non-Repeating Character

### Problem

Given a string, return the first character that occurs exactly once. For `"swiss"`, the answer is `'w'`. If none exists, return `'\0'` (this convention means the API cannot distinguish “not found” from a real null character in the input, which ordinary text strings do not contain).

### C solution for byte-oriented text

```c

#include <stddef.h>
#include <limits.h>

char first_non_repeating(const char *s)
{
    // If s is NULL, there is no valid string to process.
    if (s == NULL) {
        return '\0';
    }

    // Create a frequency counter for every possible unsigned char value.
    // counts[c] stores how many times character c appears.
    // Initially, all counts are zero.
    size_t counts[UCHAR_MAX + 1U] = {0U};

    // PASS 1: Count the frequency of every character.
    //
    // Input: "swiss"
    //
    // i = 0: s[0] = 's' -> counts['s'] = 1
    // i = 1: s[1] = 'w' -> counts['w'] = 1
    // i = 2: s[2] = 'i' -> counts['i'] = 1
    // i = 3: s[3] = 's' -> counts['s'] = 2
    // i = 4: s[4] = 's' -> counts['s'] = 3
    //
    // Final frequencies:
    // 's' -> 3, 'w' -> 1, 'i' -> 1
    //
    // Note: "swiss" actually contains three 's' characters and
    // two 'i'/'s'? Let's count carefully: s, w, i, s, s.
    // Therefore 's' occurs 3 times, 'w' once, and 'i' once.

    for (size_t i = 0U; s[i] != '\0'; ++i) {
        // Convert to unsigned char so the array index is never negative.
        unsigned char c = (unsigned char)s[i];

        // Increment the frequency of this character.
        ++counts[c];
    }

    // PASS 2: Scan from the beginning to find the first character
    // whose total frequency is exactly 1.
    //
    // i = 0: 's' -> counts['s'] = 3, skip.
    // i = 1: 'w' -> counts['w'] = 1, return 'w'.
    //
    // We return immediately, so 'i' is never checked in this pass.
    for (size_t i = 0U; s[i] != '\0'; ++i) {
        unsigned char c = (unsigned char)s[i];

        // Exactly one occurrence means this is non-repeating.
        if (counts[c] == 1U) {
            return s[i];
        }
    }

    // No character occurred exactly once.
    return '\0';
}

```

### Walkthrough, step by step

We make two passes. The first pass counts how often each byte appears. The second pass walks through the original string from left to right and returns the first character whose count is exactly one. The second pass is essential: knowing which characters are unique is not enough; we must preserve their original order.

For `"swiss"`, the first pass sees `s` and increments its count to one. It sees `w`, so `w` becomes one. The next `i` becomes one, the next `s` increases the `s` count to two, and the final `s` increases it to three. At the end, the counts are `s:3`, `w:1`, `i:1`.

The second pass starts again at the first character. It sees `s`, whose count is three, so it skips it. The next character is `w`, whose count is one, so it returns `'w'` immediately. It does not return `'i'`, even though `i` is also unique, because `w` appears earlier in the string.

The cast to `unsigned char` matters because plain `char` may be signed on a platform. Using a possibly negative `char` as an array index could access memory before the counts array. This solution treats text as bytes; it does not correctly count Unicode code points in UTF-8 text.

**Complexity:** \(O(n + R)\) time, where \(R\) is the number of possible byte values, and \(O(R)\) space. Since `R` is fixed for a given C implementation, this is effectively \(O(n)\) time with constant-sized extra storage.

---

## 9. Implement `strlen`

### Problem

Write a function that counts characters before the first null terminator.

### C solution

```c

#include <stddef.h>

size_t my_strlen(const char *s)
{
    // Initialize the length counter to zero.
    // We have not checked any characters yet.
    size_t length = 0U;

    // If s is NULL, return 0 as our chosen API policy.
    // Note: standard strlen() does not accept a NULL pointer.
    if (s == NULL) {
        return 0U;
    }

    // DRY RUN INPUT: s = "ARM"
    //
    // Memory representation:
    // Index:      0     1     2     3
    // Character: 'A'   'R'   'M'   '\0'
    //
    // Iteration 1:
    // length = 0, s[0] = 'A'
    // 'A' != '\0', so increment length to 1.
    //
    // Iteration 2:
    // length = 1, s[1] = 'R'
    // 'R' != '\0', so increment length to 2.
    //
    // Iteration 3:
    // length = 2, s[2] = 'M'
    // 'M' != '\0', so increment length to 3.
    //
    // Iteration 4:
    // length = 3, s[3] = '\0'
    // Condition is false, so the loop stops.
    //
    // The terminating '\0' is not included in the length.

    while (s[length] != '\0') {
        ++length;  // Move to the next character.
    }

    // Return the number of characters before '\0'.
    // For "ARM", the returned length is 3.
    return length;
}

```

### Walkthrough, step by step

A C string is not stored with its length automatically attached. Instead, the string ends when the program reaches the special byte `'\0'`. So the function starts counting at zero and moves through the string one byte at a time until it sees that terminator.

For `"ARM"`, index `0` contains `A`, so the condition is true and `length` becomes `1`. Index `1` contains `R`, so it becomes `2`. Index `2` contains `M`, so it becomes `3`. Index `3` contains `'\0'`, so the loop stops. The returned length is `3`; the null terminator itself is not counted.

The null-pointer check shows a deliberate custom policy. The standard library's `strlen` requires a valid pointer to a null-terminated string; passing `NULL` is undefined behavior. Returning zero for `NULL` may be convenient in a custom API, but it makes `NULL` indistinguishable from an empty string. In production code, decide and document the policy rather than assuming it is automatically safe.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

---

## 10. Implement `strcmp`

### Problem

Compare two null-terminated strings lexicographically. Return zero if they are equal, a negative value if the first differing byte in the first string is smaller, and a positive value if it is greater.

### C solution

```c

#include <stddef.h>

int my_strcmp(const char *a, const char *b)
{
    // Handle NULL pointers using our custom policy:
    // NULL == NULL -> return 0.
    // NULL comes before a non-NULL string -> return -1.
    // A non-NULL string comes after NULL -> return 1.
    if (a == NULL || b == NULL) 
    {
        if (a == b) 
        {
            return 0;
        }

        return (a == NULL) ? -1 : 1;
    }

    // DRY RUN:
    // a = "ARM", b = "ART"
    //
    // Initial characters:
    // a points to 'A', b points to 'A'.
    //
    // Iteration 1:
    // *a = 'A', *b = 'A'
    // Both characters are equal, so move both pointers forward.
    // a now points to 'R', b now points to 'R'.
    //
    // Iteration 2:
    // *a = 'R', *b = 'R'
    // Both characters are equal, so move both pointers forward.
    // a now points to 'M', b now points to 'T'.
    //
    // Next condition:
    // *a != '\0' -> true, because *a = 'M'.
    // *a == *b   -> false, because 'M' != 'T'.
    // The loop stops at the first differing character.

    while (*a != '\0' && *a == *b) {
        ++a;  // Move to the next character in the first string.
        ++b;  // Move to the next character in the second string.
    }

    // Compare the first differing bytes as unsigned values.
    //
    // 'M' = 77, 'T' = 84 in ASCII.
    // Return 77 - 84 = -7.
    //
    // If both strings end together, '\0' - '\0' = 0.
    // If one string ends first, '\0' is smaller than any nonzero byte.
    return (int)(unsigned char)*a - (int)(unsigned char)*b;
}

```

### Walkthrough, step by step

String comparison is like comparing two words alphabetically. Move through both strings together while the current characters match. The first mismatch decides the result. If both strings reach the terminator together, they are equal.

Compare `"cat"` with `"car"`. The first characters are both `c`, so both pointers advance. The second characters are both `a`, so they advance again. The third characters are `t` and `r`, which differ. Their byte values decide the result: `t` is greater than `r`, so the returned difference is positive. The exact positive number is not important; only its sign matters.

Compare `"cat"` with `"cat"`. Each character matches, and eventually both pointers reach `'\0'`. The loop stops and the subtraction is zero, so the strings compare equal.

The casts to `unsigned char` ensure bytes above the signed-char range are compared as nonnegative byte values. The null-pointer behavior is a custom extension, not standard `strcmp` behavior. As with the previous example, a real API should document this choice.

**Complexity:** \(O(n)\) time in the length of the common prefix plus the first mismatch, and \(O(1)\) extra space.

---

## Revision Checklist for Part 1

Before moving on, make sure you can explain these points without memorizing the code:

1. Why the closest-to-zero problem needs a tie rule.
2. Why reversing a string uses two indices and stops at the middle.
3. Why `value & (value - 1)` clears one set bit.
4. Why a shift count must be less than the operand's bit width.
5. Why duplicate maximum values do not count as the second-largest distinct value.
6. Why XOR cancels values that appear twice.
7. Why binary search requires sorted input and cuts the search range in half.
8. Why a character-counting array must be indexed using `unsigned char`.
9. Why `strlen` stops before counting `'\0'`.
10. Why `strcmp` only needs to return a negative, zero, or positive result.

**Next part:** pointer and memory problems, bounded string copy, swapping through pointers, array reversal, allocation/free safety, linked-list cycle detection, linked-list reversal, and common memory bugs.


