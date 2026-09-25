# 2. FIXED-SIZE ARRAYS & STACK OPERATIONS

## BEGINNER

### 51. Find maximum in fixed-size array without dynamic allocation.

**Answer**

To find the maximum value in a fixed-size array, initialize the maximum with the first element and scan the remaining elements once.

This requires:

```text
Time  = O(n)
Space = O(1)
```

No dynamic memory allocation is required.

```c
#include <stdint.h>
#include <stdbool.h>
#include <stddef.h>

static bool find_max(const int32_t *array,
                     size_t size,
                     int32_t *max_value)
{
    /*
     * Empty array has no maximum.
     */
    if (array == NULL ||
        max_value == NULL ||
        size == 0U)
    {
        return false;
    }

    /*
     * Initialize with first element rather than zero.
     *
     * This is important when all array values are negative.
     */
    int32_t max = array[0];

    /*
     * Scan remaining elements.
     */
    for (size_t i = 1U; i < size; ++i)
    {
        if (array[i] > max)
        {
            max = array[i];
        }
    }

    *max_value = max;

    return true;
}
```

**Counter: Stack-allocated array vs heap?**

For a fixed-size embedded buffer, stack allocation can be simple:

```c
int32_t values[100];
```

Advantages:

```text
No malloc/free
Deterministic allocation
Simple lifetime
```

But large arrays can consume significant stack space.

Heap allocation:

```c
int32_t *values = malloc(...);
```

can introduce:

```text
Fragmentation
Allocation failure
Non-deterministic allocation time
Lifetime-management complexity
```

For many embedded systems, fixed static/stack storage is preferred when the maximum size is known.

**Counter: Time complexity: O(n) guaranteed on embedded?**

Yes, this algorithm examines each element once:

```text
Worst case = n - 1 comparisons
```

So the algorithm is:

```text
O(n)
```

The actual execution time depends on:

```text
CPU frequency
memory wait states
cache behavior
branch behavior
compiler optimization
```

But the algorithmic upper bound remains linear.

**Counter: How to handle empty array safely?**

A zero-length array has no maximum.

Do not access:

```c
array[0]
```

when:

```c
size == 0
```

Return an error/status such as `false`.

**Interview point**

> “I would use a single linear scan with O(n) time and O(1) extra space. I initialize from the first element so negative values are handled correctly, and I explicitly reject an empty array.”

---

### 52. Reverse array in-place with limited stack space.

**Answer**

To reverse an array in-place, use two indexes:

```text
left  → first element
right → last element
```

Swap them and move toward the center.

This requires only O(1) extra space.

```c
#include <stdint.h>
#include <stddef.h>

static void reverse_array(int32_t *array, size_t size)
{
    if (array == NULL || size == 0U)
    {
        return;
    }

    size_t left = 0U;
    size_t right = size - 1U;

    while (left < right)
    {
        int32_t temp = array[left];

        array[left] = array[right];
        array[right] = temp;

        ++left;
        --right;
    }
}
```

**Counter: Only 4-8 bytes of extra variables allowed?**

Yes. The algorithm uses only a few scalar variables:

```text
left
right
temp
```

The exact stack usage depends on their types and compiler/ABI.

No second array is required.

**Counter: What if array size is unknown at compile-time?**

That is not a problem if the size is supplied at runtime:

```c
reverse_array(array, size);
```

The array itself can be fixed-size, static, stack-allocated, or otherwise provided by the caller. The algorithm does not require the size to be a compile-time constant.

**Counter: Verify in-place doesn't cause undefined behavior?**

Important checks:

```text
array != NULL
size > 0
```

Then ensure the indexes remain within:

```text
0 ... size - 1
```

The `left < right` condition prevents crossing and ensures every pair is swapped once.

Avoid writing code such as:

```c
size_t right = size - 1U;
```

before checking `size == 0`, because unsigned subtraction would wrap.

**Interview point**

> “I can reverse the array in-place using two indexes and one temporary variable. The algorithm is O(n) time and O(1) extra space.”

---

### 53. Linear search with early termination.

**Answer**

Linear search checks elements one by one until the target is found or the array ends.

```c
#include <stdint.h>
#include <stddef.h>

static int32_t linear_search(const int32_t *array,
                             size_t size,
                             int32_t target)
{
    if (array == NULL)
    {
        return -1;
    }

    for (size_t i = 0U; i < size; ++i)
    {
        if (array[i] == target)
        {
            /*
             * Early termination.
             */
            return (int32_t)i;
        }
    }

    return -1;
}
```

Complexity:

```text
Best case  = O(1)
Worst case = O(n)
Average    = O(n)
```

**Counter: Cache efficiency on Harvard architecture?**

Harvard architecture separates instruction and data memory.

The search is still fundamentally sequential data access:

```text
array[0]
array[1]
array[2]
...
```

Sequential access is generally friendly to memory systems, but the exact behavior depends on whether the data is in:

```text
RAM
flash
cache
external memory
```

Do not assume a Harvard architecture automatically makes linear search faster.

**Counter: Predictable timing for real-time?**

Linear search has a variable execution time because it may find the target early.

If the target is found at index 0:

```text
very short
```

If not found:

```text
full O(n) scan
```

For hard real-time code, you need to consider the worst-case execution time.

If constant/predictable timing is required, an early-exit search may not be appropriate.

**Counter: Sentinel value optimization?**

A sentinel places the target at the end of the search area so the loop only checks for equality.

Conceptually:

```c
array[size] = target;

size_t i = 0U;

while (array[i] != target)
{
    ++i;
}
```

Then the loop is guaranteed to terminate because the sentinel exists.

But this requires:

```text
One extra writable element
```

and is unsuitable if:

```text
array cannot be modified
array has no spare space
target could conflict with required data semantics
```

**Interview point**

> “Linear search is simple and cache-friendly for sequential data, but its execution time is data-dependent. In real-time systems I need to consider the worst-case scan length and whether early termination is acceptable.”

---

### 54. Copy array with size bounds checking.

**Answer**

When copying an array, the destination capacity must be known and checked before writing.

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

static bool copy_array(uint8_t *destination,
                       size_t destination_size,
                       const uint8_t *source,
                       size_t source_size)
{
    if (destination == NULL ||
        source == NULL)
    {
        return false;
    }

    /*
     * Prevent destination buffer overflow.
     */
    if (source_size > destination_size)
    {
        return false;
    }

    /*
     * This implementation is only valid for non-overlapping
     * source and destination buffers.
     */
    for (size_t i = 0U; i < source_size; ++i)
    {
        destination[i] = source[i];
    }

    return true;
}
```

**Counter: Prevent buffer overflow?**

Yes.

The fundamental rule is:

```text
bytes_to_copy <= destination_capacity
```

Never assume the destination is large enough.

Also be careful with calculations involving multiplication:

```c
count * sizeof(element)
```

because the size calculation itself can overflow in poorly designed code.

**Counter: What if source == destination (overlapping)?**

If:

```c
source == destination
```

copying is unnecessary.

If the regions partially overlap, a plain forward loop may corrupt the source before it has been read.

For overlapping memory, use:

```c
memmove()
```

rather than:

```c
memcpy()
```

**Counter: Use memcpy() or manual loop?**

For ordinary non-overlapping byte copies:

```c
memcpy(destination, source, size);
```

is usually the preferred API.

Advantages include:

```text
Clear intent
Compiler optimization
Highly optimized library implementation
```

A manual loop may be appropriate when:

```text
Special hardware semantics exist
Need per-element transformation
Need instrumentation
Copy operation has unusual constraints
```

**Important**

`memcpy()` requires the source and destination regions to be non-overlapping.

**Interview point**

> “I first validate destination capacity. For non-overlapping memory I would normally use `memcpy()`. If overlap is possible, I use `memmove()`.”

---

### 55. Find index of maximum value.

**Answer**

This is similar to finding the maximum, but instead of returning only the value, we retain the index where it occurs.

```c
#include <stdint.h>
#include <stddef.h>

static int find_max_index(const int32_t *array,
                          size_t size)
{
    if (array == NULL || size == 0U)
    {
        return -1;
    }

    size_t max_index = 0U;

    for (size_t i = 1U; i < size; ++i)
    {
        if (array[i] > array[max_index])
        {
            max_index = i;
        }
    }

    return (int)max_index;
}
```

**Counter: Return first or last occurrence?**

That is an API contract decision.

Using:

```c
>
```

returns the **first occurrence** of the maximum.

Example:

```text
[5, 9, 3, 9]

first maximum index = 1
```

If we use:

```c
>=
```

the function returns the **last occurrence**:

```text
last maximum index = 3
```

The important thing is to define the behavior clearly.

**Counter: What if array is empty?**

There is no valid index.

Possible API choices:

```text
return -1
return false + output parameter
return optional/result structure
```

For a C API, `-1` is convenient when the returned type is signed.

**Counter: Can you eliminate branching?**

In theory, branchless implementations are possible, but that does not automatically make them faster.

A simple branch:

```c
if (array[i] > array[max_index])
```

is usually clearer and often optimized effectively.

Branchless code can be considered when:

```text
profiling shows a bottleneck
branch misprediction is significant
target benefits from conditional instructions
```

**Interview point**

> “I maintain the index of the current maximum. Using `>` gives the first maximum; `>=` gives the last maximum.”

---

### 56. Check if array contains specific value (presence).

**Answer**

Presence checking is simply a search that returns true/false.

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

static bool contains_value(const int32_t *array,
                           size_t size,
                           int32_t target)
{
    if (array == NULL)
    {
        return false;
    }

    for (size_t i = 0U; i < size; ++i)
    {
        if (array[i] == target)
        {
            return true;
        }
    }

    return false;
}
```

Complexity:

```text
Best case  = O(1)
Worst case = O(n)
Space      = O(1)
```

**Counter: Time limit for real-time system?**

If this runs in a real-time path, the worst-case execution time must fit within the task's budget.

For an unsorted array:

```text
Worst case = n comparisons
```

If `n` is large and the timing requirement is strict, consider a better data structure or preprocessing.

**Counter: Cache-friendly search order?**

For a normal contiguous array, sequential search is naturally sequential:

```text
0 → 1 → 2 → 3 → ...
```

This is generally good for memory locality.

But again, exact behavior depends on:

```text
RAM/cache
flash
memory bus
architecture
data size
```

**Counter: Early exit optimization?**

Yes.

Once the target is found, return immediately.

```c
if (array[i] == target)
{
    return true;
}
```

This improves average-case work but does not improve the worst-case O(n) bound.

**Interview point**

> “For an unsorted fixed array, linear search is simple and O(1) space. Early exit improves average latency, but the worst-case execution time remains linear.”

---

### 57. Count occurrences of element in array.

**Answer**

Count every occurrence of the target.

```c
#include <stdint.h>
#include <stddef.h>

static size_t count_occurrences(const int32_t *array,
                                size_t size,
                                int32_t target)
{
    if (array == NULL)
    {
        return 0U;
    }

    size_t count = 0U;

    for (size_t i = 0U; i < size; ++i)
    {
        if (array[i] == target)
        {
            ++count;
        }
    }

    return count;
}
```

Complexity:

```text
Time  = O(n)
Space = O(1)
```

**Counter: O(n) time, O(1) space always?**

For this direct algorithm:

```text
O(n) time
O(1) extra space
```

Yes.

But not every way of solving the problem has O(1) space. A frequency table/hash table could use extra memory.

**Counter: Integer overflow: count > 2^32?**

If `size_t` is 64-bit, it can represent a much larger count than `uint32_t`.

For a `uint32_t count`, ensure:

```text
count cannot exceed UINT32_MAX
```

In normal C arrays, the number of elements cannot exceed what `size_t` can represent, so `size_t` is generally the natural type for array counts.

**Counter: How to make this atomic for thread safety?**

The function itself reads the array. If another thread/ISR can modify the array simultaneously, simply making `count` atomic does not make the operation thread-safe.

You need synchronization around the data snapshot.

Possible approaches:

```text
Mutex
Critical section
Disable relevant interrupt
Immutable snapshot
Double buffering
Atomic producer/consumer design
```

The key issue is consistency of the array, not just atomicity of the final counter.

**Interview point**

> “The simple algorithm is O(n) time and O(1) extra space. For concurrency, protecting the array data is more important than merely making the local count atomic.”

---

### 58. Initialize fixed array with pattern (zeros, ones, alternating).

**Answer**

Initialization depends on the element type and pattern.

For an array of bytes:

```c
#include <stdint.h>
#include <stddef.h>
#include <string.h>

static void init_zero(uint8_t *array, size_t size)
{
    if (array == NULL)
    {
        return;
    }

    memset(array, 0, size);
}
```

For a byte array filled with `0xFF`:

```c
static void init_ff(uint8_t *array, size_t size)
{
    if (array == NULL)
    {
        return;
    }

    memset(array, 0xFF, size);
}
```

**Important:** `memset()` fills bytes, not arbitrary integer values.

For example:

```c
uint32_t array[4];

memset(array, 1, sizeof(array));
```

does NOT create:

```text
1
1
1
1
```

Instead, each byte becomes `0x01`, so each 32-bit element becomes:

```text
0x01010101
```

For integer patterns, use a loop:

```c
static void init_u32(uint32_t *array,
                     size_t size,
                     uint32_t pattern)
{
    if (array == NULL)
    {
        return;
    }

    for (size_t i = 0U; i < size; ++i)
    {
        array[i] = pattern;
    }
}
```

**Counter: memset() vs loop vs compiler optimizations?**

Use `memset()` when the desired pattern is byte-oriented.

Use a loop when the desired pattern is element-oriented.

The compiler may optimize both.

Modern compilers can recognize simple loops and lower them to efficient memory operations.

**Counter: Volatile vs non-volatile data?**

For ordinary RAM:

```c
uint32_t array[10];
```

do not use `volatile` unnecessarily.

For memory-mapped hardware or data modified asynchronously by hardware/ISR in a context where `volatile` is appropriate:

```c
volatile uint32_t register_array[10];
```

the compiler must treat accesses differently.

However:

> `volatile` does not provide atomicity or thread synchronization.

It only affects compiler optimization/visibility of accesses.

**Counter: Size-specific optimizations (8/16/32-bit elements)?**

Yes, especially in low-level embedded systems.

Examples:

```text
uint8_t  → byte operations
uint16_t → half-word operations
uint32_t → word operations
```

The compiler normally handles alignment and instruction selection.

Forcing manual tricks is usually unnecessary unless profiling demonstrates a need.

**Alternating pattern example**

```c
static void init_alternating(uint32_t *array, size_t size)
{
    if (array == NULL)
    {
        return;
    }

    for (size_t i = 0U; i < size; ++i)
    {
        array[i] = (i & 1U) ? 0x55555555U
                            : 0xAAAAAAAAU;
    }
}
```

**Interview point**

> “`memset()` is byte-oriented. For typed integer patterns I prefer an element-wise loop. I also don't use `volatile` unless the memory is accessed asynchronously or has hardware semantics that justify it.”

---

### 59. Find two elements that sum to target.

**Answer**

There are multiple solutions depending on whether the array is sorted and how much memory is available.

For an unsorted array with O(1) extra space, the straightforward approach is nested loops:

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

typedef struct
{
    size_t first;
    size_t second;
} Pair;

static bool find_pair(const int32_t *array,
                      size_t size,
                      int32_t target,
                      Pair *result)
{
    if (array == NULL || result == NULL)
    {
        return false;
    }

    for (size_t i = 0U; i < size; ++i)
    {
        for (size_t j = i + 1U; j < size; ++j)
        {
            /*
             * Be careful with integer overflow if the data
             * type can reach its limits.
             */
            if (array[i] == target - array[j])
            {
                result->first = i;
                result->second = j;
                return true;
            }
        }
    }

    return false;
}
```

Complexity:

```text
Time  = O(n²)
Space = O(1)
```

**Counter: Sorted array vs unsorted?**

If the array is sorted, the two-pointer technique gives O(n) time.

```text
left  → beginning
right → end

sum < target
    → left++

sum > target
    → right--

sum == target
    → found
```

Example:

```c
static bool find_pair_sorted(const int32_t *array,
                             size_t size,
                             int32_t target,
                             size_t *left_result,
                             size_t *right_result)
{
    if (array == NULL ||
        left_result == NULL ||
        right_result == NULL ||
        size < 2U)
    {
        return false;
    }

    size_t left = 0U;
    size_t right = size - 1U;

    while (left < right)
    {
        /*
         * Wider arithmetic can be used if overflow is a concern.
         */
        int64_t sum =
            (int64_t)array[left] +
            (int64_t)array[right];

        if (sum == target)
        {
            *left_result = left;
            *right_result = right;
            return true;
        }

        if (sum < target)
        {
            ++left;
        }
        else
        {
            --right;
        }
    }

    return false;
}
```

**Counter: O(n) vs O(n log n) space constraint?**

The common choices are:

```text
Brute force:
    O(n²) time
    O(1) space

Sort + two pointers:
    O(n log n) sorting time
    O(1) or implementation-dependent extra space
    BUT original index information may need to be preserved

Hash table:
    O(n) expected time
    O(n) space
```

If the array is already sorted:

```text
two pointers = O(n) time, O(1) space
```

**Counter: Multiple pairs: find all or just one?**

That changes the algorithm.

If only one pair is required:

```text
return immediately
```

If all pairs are required:

```text
continue searching
```

For duplicate values, define whether you want:

```text
All index pairs
Unique value pairs
First pair only
```

These are different problems.

**Interview point**

> “For an unsorted array, I can choose O(n²)/O(1) brute force, O(n) expected time with a hash table, or sort and use two pointers. The right answer depends on whether the array is already sorted, whether I may modify it, and the memory limit.”

---

### 60. Shift array elements (rotate/slide) without extra storage.

**Answer**

There are several operations that can be called a "shift", so the requirement should be clarified:

```text
1. Left shift with data loss
2. Right shift with data loss
3. Left rotation
4. Right rotation
```

A simple left shift by one with data loss is:

```c
#include <stdint.h>
#include <stddef.h>

static void shift_left_one(int32_t *array, size_t size)
{
    if (array == NULL || size == 0U)
    {
        return;
    }

    for (size_t i = 1U; i < size; ++i)
    {
        array[i - 1U] = array[i];
    }

    /*
     * The last element can be filled according to the API
     * requirement.
     */
    array[size - 1U] = 0;
}
```

**Counter: Left shift by K positions?**

If data loss is intended:

```text
[0 1 2 3 4 5]
K = 2

→ [2 3 4 5 0 0]
```

A simple O(n) implementation moves each surviving element left.

For a rotation, however, we want to preserve all elements.

---

**Counter: Right shift / rotate?**

For rotation:

```text
[1 2 3 4 5]
rotate right by 2

→ [4 5 1 2 3]
```

A very useful O(n) / O(1)-space technique is the reversal algorithm.

For right rotation by `k`:

```text
1. Reverse entire array
2. Reverse first k elements
3. Reverse remaining elements
```

Example:

```text
[1 2 3 4 5 6 7]

Reverse all:
[7 6 5 4 3 2 1]

Reverse first 2:
[6 7 5 4 3 2 1]

Reverse remaining:
[6 7 1 2 3 4 5]
```

Implementation:

```c
#include <stdint.h>
#include <stddef.h>

static void reverse_range(int32_t *array,
                          size_t left,
                          size_t right)
{
    while (left < right)
    {
        int32_t temp = array[left];

        array[left] = array[right];
        array[right] = temp;

        ++left;
        --right;
    }
}

static void rotate_right(int32_t *array,
                         size_t size,
                         size_t k)
{
    if (array == NULL || size == 0U)
    {
        return;
    }

    k %= size;

    if (k == 0U)
    {
        return;
    }

    /*
     * Reverse entire array.
     */
    reverse_range(array, 0U, size - 1U);

    /*
     * Reverse first k elements.
     */
    reverse_range(array, 0U, k - 1U);

    /*
     * Reverse remaining elements.
     */
    reverse_range(array, k, size - 1U);
}
```

For left rotation by `k`, a similar reversal method can be used.

For example:

```text
Left rotate by k:

reverse first k
reverse remaining n-k
reverse whole array
```

**Counter: Minimum number of swaps needed?**

There is an important distinction between:

```text
minimum number of element assignments/swaps
```

and:

```text
simple O(n) implementation
```

For arbitrary rotations, cycle decomposition can achieve the movement using the number of element positions that actually need to move, with O(1) extra storage apart from a temporary variable.

For a rotation by `k` positions:

```text
number of permutation cycles = gcd(n, k)
```

and the cycles can be moved directly.

However, the reversal algorithm is often preferred in interviews because it is:

```text
O(n) time
O(1) extra space
easy to explain
```

For a simple left/right shift with data loss, fewer operations may be possible because some elements are discarded.

**Interview point**

> “I first clarify whether the operation is a shift with data loss or a rotation. For rotation, the reversal algorithm gives O(n) time and O(1) extra space and avoids allocating another array.”

---

# FIXED-SIZE ARRAYS — INTERVIEW CHEAT SHEET

## 51. Find Maximum

```text
Initialize from array[0]
        ↓
Scan remaining elements
        ↓
Update maximum
```

```text
Time  = O(n)
Space = O(1)
```

Always handle:

```text
NULL
size == 0
negative values
```

---

## 52. Reverse In-Place

```text
left  → 0
right → n-1

while left < right:
    swap
    left++
    right--
```

```text
Time  = O(n)
Space = O(1)
```

---

## 53. Linear Search

```text
Best  = O(1)
Worst = O(n)
```

Early exit improves average-case work but not worst-case complexity.

For hard real-time systems:

```text
Worst-case execution time matters
```

---

## 54. Safe Array Copy

Always verify:

```text
copy_size <= destination_capacity
```

Use:

```text
memcpy()
```

for non-overlapping regions.

Use:

```text
memmove()
```

when overlapping is allowed.

---

## 55. Maximum Index

```c
if (array[i] > array[max_index])
```

→ first maximum.

```c
if (array[i] >= array[max_index])
```

→ last maximum.

---

## 56. Presence Check

```text
Unsorted array
    ↓
Linear search
    ↓
O(n)
```

Early return when found.

---

## 57. Count Occurrences

```text
Time  = O(n)
Space = O(1)
```

Use `size_t` for array element counts when appropriate.

For concurrency, protect the data being read, not merely the local counter.

---

## 58. Array Initialization

```text
memset()
    ↓
byte pattern

loop
    ↓
typed element pattern
```

Important:

```c
memset(uint32_array, 1, ...)
```

does not produce integer value `1` in each element.

It produces:

```text
0x01010101
```

per 32-bit element.

---

## 59. Two Sum

```text
Unsorted + O(1) space:
    O(n²)

Sorted:
    Two pointers → O(n)

Hash table:
    O(n) expected time
    O(n) space
```

Choose based on:

```text
Sorted?
Memory?
Need original indexes?
Need one pair or all pairs?
```

---

## 60. Shift / Rotate

First identify the operation:

```text
Shift
    → data can be lost

Rotate
    → all elements preserved
```

Rotation:

```text
Reversal algorithm
    ↓
O(n) time
O(1) space
```

---

# Embedded Interview Considerations for Fixed-Size Arrays

When discussing fixed-size arrays in embedded C, also mention:

```text
1. Deterministic memory usage
2. No malloc/free in timing-critical paths
3. Stack-size limits
4. Alignment
5. Integer overflow
6. Worst-case execution time
7. Interrupt/concurrency behavior
8. Buffer bounds
9. Memory-mapped/volatile data
10. Compiler optimization
```

For a real-time system, saying only:

> “It is O(n)”

is not enough.

A stronger answer considers:

```text
Algorithmic complexity
+
worst-case execution time
+
memory behavior
+
concurrency
+
overflow/bounds
+
target architecture
```

---

# Strong Interview Answer for This Topic

> **“For fixed-size arrays in embedded C, I prefer bounded, deterministic algorithms with no unnecessary dynamic allocation. I pay particular attention to bounds checking, integer overflow, stack usage, worst-case execution time, and whether the array is accessed concurrently by an ISR or another task. When an in-place O(1)-space algorithm exists, such as two-pointer reversal or reversal-based rotation, it is often a good fit for memory-constrained embedded systems.”**


# STACK OPERATIONS — BEGINNER

### 61. Implement a fixed-size stack using an array.

**Answer**

A stack follows **LIFO**:

```text
Last In → First Out
```

The most recently pushed element is the first one popped.

For a fixed-size embedded implementation, a static array plus a stack-top index is simple and deterministic.

```c
#include <stdint.h>
#include <stdbool.h>
#include <stddef.h>

#define STACK_SIZE 16U

typedef struct
{
    uint32_t data[STACK_SIZE];

    /*
     * Number of valid elements currently in the stack.
     *
     * top = 0 -> empty
     * top = STACK_SIZE -> full
     */
    size_t top;

} Stack;

static void stack_init(Stack *stack)
{
    if (stack != NULL)
    {
        stack->top = 0U;
    }
}

static bool stack_push(Stack *stack, uint32_t value)
{
    if (stack == NULL || stack->top >= STACK_SIZE)
    {
        /*
         * Stack overflow.
         */
        return false;
    }

    stack->data[stack->top] = value;
    stack->top++;

    return true;
}

static bool stack_pop(Stack *stack, uint32_t *value)
{
    if (stack == NULL ||
        value == NULL ||
        stack->top == 0U)
    {
        /*
         * Stack underflow.
         */
        return false;
    }

    stack->top--;

    *value = stack->data[stack->top];

    return true;
}
```

Complexity:

```text
Push = O(1)
Pop  = O(1)
```

**Counter: Why fixed-size stack instead of malloc?**

In embedded systems, a fixed-size stack gives predictable memory usage and avoids:

```text
Heap fragmentation
Allocation failure
Non-deterministic allocation time
```

**Counter: What is the LIFO property?**

If we push:

```text
10
20
30
```

the pop order is:

```text
30
20
10
```

**Interview point**

> “For a fixed-size embedded stack, I would use an array and a top index. Push and pop are O(1), and overflow/underflow are explicitly checked.”

---

### 62. Check stack overflow and underflow.

**Answer**

A stack has two important invalid conditions:

```text
Overflow:
    Push when stack is full.

Underflow:
    Pop when stack is empty.
```

Example:

```c
static bool stack_is_empty(const Stack *stack)
{
    return stack == NULL || stack->top == 0U;
}

static bool stack_is_full(const Stack *stack)
{
    return stack != NULL && stack->top >= STACK_SIZE;
}
```

**Counter: What happens on overflow?**

Never write beyond the array boundary.

Possible policies:

```text
Return error
Set diagnostic flag
Reject new item
Enter safe/fault state
```

The correct behavior depends on the application.

**Counter: What happens on underflow?**

Return an error/status instead of reading invalid memory.

**Interview point**

> “Bounds checking is mandatory because stack overflow or underflow can become memory corruption.”

---

### 63. Peek at the top of a stack without removing it.

**Answer**

`peek()` returns the top element but does not decrement the stack pointer.

```c
static bool stack_peek(const Stack *stack,
                       uint32_t *value)
{
    if (stack == NULL ||
        value == NULL ||
        stack->top == 0U)
    {
        return false;
    }

    *value = stack->data[stack->top - 1U];

    return true;
}
```

Example:

```text
Stack:

10
20
30  ← top

peek() → 30

Stack remains unchanged.
```

**Counter: Difference between pop and peek?**

```text
peek → inspect, do not remove
pop  → inspect and remove
```

**Counter: Why useful?**

Useful when the next operation depends on the current top without changing the stack state.

**Interview point**

> “Peek provides read-only access to the top element, while pop changes the stack.”

---

### 64. Reverse a string using a stack.

**Answer**

Because a stack is LIFO, it naturally reverses a sequence.

Example:

```text
Input:

HELLO

Push:

H E L L O

Pop:

O L L E H
```

```c
#include <stddef.h>

static bool reverse_string(char *string,
                           size_t size)
{
    if (string == NULL || size == 0U)
    {
        return false;
    }

    /*
     * This implementation reverses the string in-place,
     * so an explicit stack is not actually necessary.
     *
     * The example is intended to illustrate the LIFO
     * behavior of a stack.
     */

    size_t left = 0U;
    size_t right = size - 1U;

    while (left < right)
    {
        char temp = string[left];

        string[left] = string[right];
        string[right] = temp;

        ++left;
        --right;
    }

    return true;
}
```

For an actual stack-based solution, characters would be pushed and then popped.

**Counter: Is a stack really necessary?**

No.

For an in-place string reversal, two pointers use:

```text
O(1) extra space
```

A stack would require:

```text
O(n) extra space
```

**Interview point**

> “A stack demonstrates the natural LIFO solution, but for embedded systems I would prefer the O(1)-space two-pointer solution when only reversal is required.”

---

### 65. Check balanced parentheses using a stack.

**Answer**

A stack is useful for matching nested delimiters.

For example:

```text
({[]})
```

is valid.

But:

```text
({[})
```

is invalid.

Algorithm:

```text
Opening bracket:
    push

Closing bracket:
    compare with top
    pop if matching
    otherwise error
```

```c
static bool is_matching(char open,
                        char close)
{
    return
        (open == '(' && close == ')') ||
        (open == '[' && close == ']') ||
        (open == '{' && close == '}');
}
```

Conceptually:

```text
Input: {[()]}

{ → push
[ → push
( → push
) → match + pop
] → match + pop
} → match + pop

Stack empty
→ valid
```

**Counter: What if a closing bracket appears on an empty stack?**

Invalid input.

There is nothing to match.

**Counter: Time and space complexity?**

```text
Time  = O(n)
Space = O(n) worst case
```

Worst case occurs when the input contains many nested opening brackets.

**Interview point**

> “The stack stores unmatched opening delimiters. Each closing delimiter must match the most recently opened delimiter, which is exactly LIFO behavior.”

---

### 66. Implement a stack using a linked list vs fixed array.

**Answer**

Two common implementations are:

```text
Fixed array
Linked list
```

Fixed array:

```text
+ Predictable memory
+ O(1) push/pop
+ Good locality
+ No heap allocation
- Fixed maximum capacity
```

Linked list:

```text
+ Dynamic size
- Heap allocation/deallocation
- Fragmentation risk
- Pointer overhead
- Poorer locality
- More complex failure handling
```

For many embedded systems with known maximum depth, a fixed array is preferable.

**Counter: Which is more deterministic?**

A fixed array.

Dynamic allocation introduces potential timing and memory-management variability.

**Interview point**

> “For embedded real-time software, if maximum stack depth is known, I would usually prefer a fixed array.”

---

### 67. Implement a stack with push/pop counters for diagnostics.

**Answer**

In embedded systems, diagnostics can help detect abnormal stack usage.

```c
typedef struct
{
    uint32_t data[STACK_SIZE];
    size_t top;

    uint32_t overflow_count;
    uint32_t underflow_count;

} DiagnosticStack;
```

On overflow:

```c
if (stack->top >= STACK_SIZE)
{
    stack->overflow_count++;
    return false;
}
```

On underflow:

```c
if (stack->top == 0U)
{
    stack->underflow_count++;
    return false;
}
```

**Counter: Why keep diagnostics?**

Because a system may continue running after an isolated invalid request, while diagnostics can tell us that the software attempted an invalid stack operation.

**Interview point**

> “For production embedded software, I often make resource-limit violations observable rather than silently ignoring them.”

---

# STACK OPERATIONS — INTERMEDIATE

### 68. Implement a minimum stack: get minimum value in O(1).

**Answer**

A normal stack gives O(1) push/pop, but finding the minimum by scanning all elements costs O(n).

A **min-stack** maintains an additional stack containing the minimum value seen at each depth.

Example:

```text
Data stack:
5
2
8
3

Minimum stack:
5
2
2
2
```

The current minimum is always at the top of the minimum stack.

```c
#define MIN_STACK_SIZE 16U

typedef struct
{
    int32_t data[MIN_STACK_SIZE];
    int32_t minimum[MIN_STACK_SIZE];

    size_t top;

} MinStack;

static bool min_stack_push(MinStack *stack,
                           int32_t value)
{
    if (stack == NULL ||
        stack->top >= MIN_STACK_SIZE)
    {
        return false;
    }

    stack->data[stack->top] = value;

    if (stack->top == 0U ||
        value < stack->minimum[stack->top - 1U])
    {
        stack->minimum[stack->top] = value;
    }
    else
    {
        stack->minimum[stack->top] =
            stack->minimum[stack->top - 1U];
    }

    stack->top++;

    return true;
}

static bool min_stack_get_min(const MinStack *stack,
                              int32_t *value)
{
    if (stack == NULL ||
        value == NULL ||
        stack->top == 0U)
    {
        return false;
    }

    *value = stack->minimum[stack->top - 1U];

    return true;
}
```

Now:

```text
push     = O(1)
pop      = O(1)
get_min  = O(1)
```

**Counter: Why extra storage?**

The second stack stores the minimum associated with every depth.

**Counter: Is there an alternative?**

Yes. Store:

```text
value
current minimum
```

as one structure per stack element.

**Interview point**

> “I trade O(n) additional storage for constant-time minimum lookup.”

---

### 69. Evaluate a postfix expression using a stack.

**Answer**

Postfix notation puts operators after operands.

Example:

```text
3 4 + 2 *
```

means:

```text
(3 + 4) * 2

= 14
```

Algorithm:

```text
Number → push

Operator:
    pop operand2
    pop operand1
    calculate
    push result
```

For:

```text
3 4 +
```

```text
push 3
push 4
+:
    a = 4
    b = 3
    result = 3 + 4 = 7
    push 7
```

**Counter: Why is operand order important?**

For subtraction and division:

```text
pop() → operand2
pop() → operand1
```

Then:

```text
operand1 - operand2
```

not:

```text
operand2 - operand1
```

**Counter: Complexity?**

```text
Time  = O(n)
Space = O(n)
```

**Embedded concern**

Check:

```text
Stack overflow
Stack underflow
Division by zero
Integer overflow
Invalid token
Unexpected end of expression
```

**Interview point**

> “The stack naturally handles the most recently encountered operands, making postfix evaluation a straightforward LIFO problem.”

---

### 70. Difference between software stack and CPU call stack.

**Answer**

This distinction is important in embedded interviews.

A **software stack data structure** is:

```text
An application data structure
used for LIFO operations.
```

The **CPU call stack** is:

```text
Memory used by the processor/compiler
to manage function calls, local variables,
return addresses, saved registers, etc.
```

Example:

```c
void function_a(void)
{
    function_b();
}
```

Conceptually:

```text
function_a()
    ↓
call function_b()
    ↓
return address saved
    ↓
function_b stack frame
```

**Counter: What can cause call-stack overflow?**

Examples:

```text
Deep recursion
Large local arrays
Large stack frames
Nested interrupts
Too many RTOS task calls/stack usage
```

**Counter: Why is this important in embedded systems?**

Because RAM is limited.

Excessive stack usage can corrupt:

```text
variables
return addresses
other task stacks
control structures
```

which can lead to hard faults or unpredictable behavior.

**Interview point**

> “A software stack is a data structure; the CPU call stack is part of program execution and stores call frames, return addresses, registers, and local data.”

---

### 71. How do you detect stack overflow in an embedded system?

**Answer**

Several techniques are commonly used.

**1. Stack canary / guard pattern**

Fill unused stack memory with a known pattern:

```text
0xA5A5A5A5
```

Later inspect how much of the pattern remains.

The overwritten region indicates maximum stack usage.

**2. Hardware stack protection**

Some MCUs provide:

```text
MPU
stack limit registers
guard regions
fault exceptions
```

**3. RTOS stack watermark**

Many RTOSes provide a stack-high-water-mark mechanism.

**4. Static analysis**

Estimate worst-case call depth and frame usage.

**Counter: Why not simply measure average usage?**

Because real-time and safety analysis cares about worst-case behavior.

Rare paths such as:

```text
error handler
deep interrupt nesting
diagnostic path
```

may use much more stack.

**Interview point**

> “For embedded systems I prefer combining static analysis with a runtime watermark/canary or hardware protection, depending on the MCU and safety requirements.”

---

### 72. Is a stack safe to use from both ISR and main code?

**Answer**

Not automatically.

If an ISR and main code modify the same stack object concurrently:

```text
Main:
    push()

ISR:
    pop()
```

the operations can race.

A stack implementation needs a concurrency strategy.

Possible approaches:

```text
Disable interrupts briefly
Use separate ISR and main stacks
Use atomic operations where appropriate
Use a lock in task context
Avoid sharing the structure
```

For very short ISR work, a common design is to keep the ISR data structure separate from task-owned data.

**Counter: Is `volatile` enough?**

No.

`volatile` does not make a multi-step operation atomic.

For example:

```c
stack->top++;
```

is conceptually:

```text
read
add
write
```

An interrupt or another thread can intervene.

**Interview point**

> “`volatile` controls compiler access optimization, not synchronization. Shared stack state requires an appropriate concurrency mechanism.”

---

# STACK INTERVIEW CHEAT SHEET

## Basic Stack

```text
LIFO

push:
    top++
    data[top]

pop:
    top--
    return data[top]
```

Complexity:

```text
Push = O(1)
Pop  = O(1)
Peek = O(1)
```

---

## Stack Errors

```text
Overflow
    ↓
push when full

Underflow
    ↓
pop when empty
```

Always check both.

---

## Common Stack Applications

```text
Parentheses matching
Expression evaluation
DFS
Backtracking
Undo operations
Function call management
Interrupt/context handling
Protocol parsing
```

---

## Embedded Software Stack vs CPU Call Stack

```text
Software stack
    ↓
Application LIFO data structure

CPU call stack
    ↓
Function calls
Local variables
Return addresses
Saved registers
Interrupt/task context
```

---

## Embedded Stack Risks

```text
Recursion
Large local arrays
Deep call chains
Nested interrupts
RTOS task stack exhaustion
```

Detection:

```text
Canary
Watermark
MPU/guard region
Static analysis
```

---

# Strong Interview Answer for the Stack Topic

> **“A stack is a LIFO data structure. For embedded systems, I would normally use a fixed-size array with a top index when the maximum depth is known, because that provides deterministic memory usage and O(1) push/pop/peek. I would explicitly handle overflow and underflow. I would also distinguish a software stack from the CPU call stack and consider stack-overflow detection using static analysis, watermarks, canaries, or hardware protection.”**
