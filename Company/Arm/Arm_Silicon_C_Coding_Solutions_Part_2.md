# Arm & Silicon Companies C Coding Practice — Detailed Solutions
## Part 2: Pointers, Memory, Strings, and Linked Lists

**Learning approach:** Treat each function like a small story. Track what each variable points to, what memory changes after each statement, and what happens for edge cases. These are practice problems based on common C and systems-screening themes, not claims about exact private company assessment questions.

---

## 11. Swap Two Integers Using Pointers

### Problem
Write a function that swaps two integers without returning either one.

### C solution
```c

#include <stddef.h>

void swap_int(int *a, int *b)
{
    // Validate both pointers before dereferencing them.
    // If either pointer is NULL, return without changing anything.
    if (a == NULL || b == NULL) {
        return;
    }

    // DRY RUN:
    // int x = 10, y = 20;
    // swap_int(&x, &y);
    //
    // a points to x, so *a = 10.
    // b points to y, so *b = 20.
    //
    // STEP 1:
    // Save the value at address a in temp.
    // temp = *a = 10.
    int temp = *a;

    // STEP 2:
    // Copy the value at address b into the location pointed to by a.
    // *a = *b -> x = 20.
    // Current values: x = 20, y = 20.
    *a = *b;

    // STEP 3:
    // Copy the saved original value into the location pointed to by b.
    // *b = temp -> y = 10.
    // Final values: x = 20, y = 10.
    *b = temp;

    // The values are now swapped.
}

```

### Walkthrough, step by step
Imagine two labelled boxes, `x` and `y`. The pointer `a` tells us where box `x` is, and `b` tells us where box `y` is. The `*` operator means “go to the box this pointer refers to and access its contents.”

Suppose `x = 10` and `y = 25`. We call `swap_int(&x, &y)`, so `a` points to `x` and `b` points to `y`. The first statement, `temp = *a`, reads the value inside `x`, so `temp` becomes `10`. Nothing has changed in either original variable yet.

Next, `*a = *b` reads `25` from `y` and writes it into the box pointed to by `a`, which is `x`. Now `x = 25`, while `y` is still `25`; temporarily both variables contain `25`. That is why we needed `temp` before overwriting anything.

Finally, `*b = temp` writes the saved `10` into `y`. The final state is `x = 25`, `y = 10`. The caller sees the change because the function modified the original objects through their addresses, rather than modifying local copies.

The null checks avoid dereferencing a null pointer. If both pointers point to the same valid integer, the function still behaves safely: it reads the value, writes the same value back, and completes.

**Complexity:** \(O(1)\) time and \(O(1)\) extra space.

**Interview point:** C passes arguments by value. Even though we pass pointers by value, those copied pointer values still refer to the caller's objects, so dereferencing them can modify the original objects.

---

## 12. Reverse an Integer Array In Place

### Problem
Reverse the order of elements in an array without allocating a second array.

Example: `[10, 20, 30, 40]` becomes `[40, 30, 20, 10]`.

### C solution
```c

#include <stddef.h>

void reverse_array(int a[], size_t n)
{
    // If the array pointer is NULL, or fewer than 2 elements exist,
    // there is nothing to reverse.
    if (a == NULL || n < 2U) {
        return;
    }

    // left starts at the first element.
    size_t left = 0U;

    // right starts at the last valid index.
    // For n = 4, right = 4 - 1 = 3.
    size_t right = n - 1U;

    // DRY RUN:
    // Input: [10, 20, 30, 40]
    // Index:   0   1   2   3
    //
    // ITERATION 1:
    // left = 0, right = 3
    // a[left] = 10, a[right] = 40
    //
    // Step 1: temp = a[left] -> temp = 10
    // Step 2: a[left] = a[right] -> a[0] = 40
    //         Array becomes [40, 20, 30, 40]
    // Step 3: a[right] = temp -> a[3] = 10
    //         Array becomes [40, 20, 30, 10]
    // Step 4: ++left -> left = 1
    // Step 5: --right -> right = 2
    //
    // ITERATION 2:
    // left = 1, right = 2
    // a[left] = 20, a[right] = 30
    //
    // Step 1: temp = a[left] -> temp = 20
    // Step 2: a[left] = a[right] -> a[1] = 30
    //         Array becomes [40, 30, 30, 10]
    // Step 3: a[right] = temp -> a[2] = 20
    //         Array becomes [40, 30, 20, 10]
    // Step 4: ++left -> left = 2
    // Step 5: --right -> right = 1
    //
    // LOOP CHECK:
    // left = 2, right = 1
    // left < right is false, so the loop stops.
    //
    // Final array: [40, 30, 20, 10]

    while (left < right) {

        // Save the left element before overwriting it.
        int temp = a[left];

        // Copy the right element into the left position.
        a[left] = a[right];

        // Put the saved left element into the right position.
        a[right] = temp;

        // Move inward from the left.
        ++left;

        // Move inward from the right.
        --right;
    }
}

```

### Walkthrough, step by step
We put one index at the first element and another at the last element. Every iteration exchanges the two values and moves both indices toward the middle.

For `[10, 20, 30, 40]`, the first pair is index `0` (`10`) and index `3` (`40`). We save `10` in `temp`, put `40` at index `0`, then put the saved `10` at index `3`. The array is now `[40, 20, 30, 10]`. Next, `left` becomes `1` and `right` becomes `2`.

The second exchange swaps `20` and `30`, making the array `[40, 30, 20, 10]`. Now `left` becomes `2` and `right` becomes `1`. Since `left < right` is false, the loop ends. Every element is in its reversed position.

The guard `n < 2U` handles both an empty array and a one-element array. Without it, computing `n - 1U` for `n == 0` would wrap around because `size_t` is unsigned.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

---

## 13. Copy a String Into a Bounded Destination Buffer

### Problem
Copy a source C string into a destination buffer without writing past the destination's capacity. The destination must be null-terminated whenever its capacity is greater than zero.

### C solution
```c
#include <stddef.h>
#include <stdbool.h>

bool copy_string(char *dst, size_t dst_capacity, const char *src)
{
    if (dst == NULL || src == NULL) {
        return false;
    }

    if (dst_capacity == 0U) {
        return false;
    }

    size_t i = 0U;

    while (i + 1U < dst_capacity && src[i] != '\0') {
        dst[i] = src[i];
        ++i;
    }

    dst[i] = '\0';

    /* False means the source did not fit completely. */
    return src[i] == '\0';
}
```

### Walkthrough, step by step
Suppose the source is `"FIRMWARE"` and the destination buffer has capacity `5`. The destination can hold at most four visible characters plus the terminating null byte. It cannot hold all eight letters.

The loop starts at `i = 0`. It copies `F`, then increments `i` to `1`; next it copies `I`, then `R`, then `M`. After these four characters, `i = 4`. The condition `i + 1 < dst_capacity` is now `5 < 5`, which is false, so the loop stops before writing outside the buffer.

The statement `dst[i] = '\0'` writes the terminator at index `4`. The destination now contains `"FIRM"` and is a valid C string. The return expression checks `src[i] == '\0'`. At source index `4`, the character is `W`, not the terminator, so the function returns `false` to tell the caller that truncation occurred.

If the source were `"ARM"` and the buffer capacity were `5`, the loop would copy `A`, `R`, and `M`. Then it would encounter the source terminator and stop. The destination becomes `"ARM"` and the function returns `true`, because the entire string fit.

A capacity of zero is rejected because no terminator can be written. This function assumes `dst` and `src` refer to valid memory and do not overlap in a way that makes the copy unsafe. If overlapping ranges must be supported, use a carefully specified `memmove`-style design instead.

**Complexity:** \(O(\min(n, capacity))\) time and \(O(1)\) extra space.

**Important:** `strncpy` is not a drop-in “safe `strcpy`”: it may leave the destination unterminated when the source is too long, and it may pad many bytes with zeros.

---

## 14. Understand `malloc`, `free`, and Use After Free

### Problem
Explain what is wrong with code that frees allocated memory and then reads it.

### Incorrect example
```c

#include <stdlib.h>

int bad_example(void)
{
    // Allocate enough heap memory to store one int.
    int *p = malloc(sizeof *p);

    // If allocation fails, malloc returns NULL.
    if (p == NULL) 
    {
        return -1;
    }

    // Store 42 in the allocated memory.
    // p points to valid allocated memory.
    *p = 42;

    // Release the allocated memory.
    // After this call, the memory is no longer owned by p.
    free(p);

    // BUG: Do not read memory through p after free(p).
    // The pointer p still contains the old address, but that
    // address no longer refers to memory you are allowed to access.
    // Reading *p here is undefined behavior (use-after-free).
    return *p;
}

```

### Corrected example
```c
#include <stdlib.h>

int safe_example(int *out)
{
    if (out == NULL) 
    {
        return -1;
    }

    int *p = malloc(sizeof *p);

    if (p == NULL) 
    {
        return -1;
    }

    *p = 42;
    *out = *p; /* Read while the allocation is still alive. */

    free(p);
    p = NULL;  /* Helps prevent accidental reuse of this local pointer. */

    return 0;
}
```

### Walkthrough, step by step
Think of `malloc` as asking the allocator for a small storage space. The returned pointer is the address of that space, if allocation succeeds. In `bad_example`, `p = malloc(sizeof *p)` requests enough bytes for one `int`. The null check is necessary because allocation can fail. Then `*p = 42` stores `42` in the allocated memory.

When `free(p)` runs, the program gives that allocation back to the allocator. The pointer variable `p` still contains the old address, but the allocation is no longer owned by this code. The next expression, `*p`, attempts to read memory through a pointer whose allocation lifetime has ended. That is undefined behavior: the program might appear to work, return a different value, crash, or fail only in an optimized build.

In `safe_example`, the value is copied to `*out` before `free(p)`. The output object belongs to the caller, so the result survives after the temporary allocation is released. Only then do we free the allocation.

Setting `p = NULL` after freeing it can prevent a later accidental dereference through this particular local variable. However, it does not repair other aliases. If `q = p` was made earlier, setting `p` to `NULL` does not change `q`; `q` would still be a dangling pointer.

**Complexity:** The algorithm itself is \(O(1)\) time and \(O(1)\) allocated space.

**Interview distinction:** A memory leak happens when allocated memory is never released after it is no longer needed. Use-after-free happens when code accesses an allocation after it has been released. They are different bugs.

---

## 15. Avoid a Memory Leak on an Error Path

### Problem
A function allocates two buffers. If the second allocation fails, it must release the first allocation before returning.

### C solution
```c

#include <stdlib.h>
#include <stddef.h>

int process_buffers(size_t n)
{
    // STEP 1: Allocate the first buffer.
    // Example: n = 100 bytes.
    // If successful, first points to 100 allocated bytes.
    unsigned char *first = malloc(n);

    // If n is nonzero and allocation fails, return an error.
    // Nothing needs to be freed because no allocation succeeded.
    if (first == NULL && n != 0U) 
    {
        return -1;
    }

    // STEP 2: Allocate the second buffer.
    // If successful, second points to another n bytes.
    unsigned char *second = malloc(n);

    // CASE 1: Second allocation fails for a nonzero size.
    //
    // Example:
    // first  = valid allocated buffer
    // second = NULL
    //
    // We must free first before returning.
    // Otherwise, the first allocation would leak.
    if (second == NULL && n != 0U) 
    {
        free(first);  // Release the first buffer.
        return -1;    // Report allocation failure.
    }

    // CASE 2: Both allocations succeeded,
    // or n == 0 and malloc returned NULL.
    // Example processing would happen here.
    // Use the buffers only when their validity is guaranteed.

    // STEP 3: Release the second allocation first.
    // free(NULL) is safe, so this is valid even if second is NULL.
    free(second);

    // STEP 4: Release the first allocation.
    // free(NULL) is also safe.
    free(first);

    // All successful allocations have been released.
    return 0;
}

```

### Walkthrough, step by step
The function first asks for the `first` buffer. If that allocation fails for a nonzero request, there is nothing to clean up, so it returns an error immediately. If it succeeds, the program now owns that allocation and must remember to release it.

Next, it requests `second`. Imagine this allocation fails. If the function simply returned, `first` would remain allocated but the caller would have no pointer to it. That is a leak. The failure branch therefore calls `free(first)` before returning.

If both allocations succeed, the function performs its work and then frees both buffers. Each successful allocation has one matching release. In more complicated code, a single cleanup section can make this pattern easier to maintain.

Zero-size allocation behavior has special details in C, so the example treats `n == 0` separately in its checks. In real code, a clearer policy is often to reject zero capacity explicitly or define what an empty buffer means. Also, this example assumes `n` is a sensible size; production code should validate sizes before using them in calculations or allocations.

**Complexity:** Allocation cost depends on the runtime allocator; algorithmically, cleanup is \(O(1)\) in the number of allocations shown.

---

## 16. Detect a Cycle in a Linked List

### Problem
A singly linked list may accidentally point back to an earlier node, creating a cycle. Detect whether a cycle exists without using extra memory proportional to the list length.

### C solution
```c

#include <stdbool.h>

struct Node {
    int value;
    struct Node *next;
};

bool has_cycle(const struct Node *head)
{
    // Both pointers start at the head of the linked list.
    // slow moves one node at a time.
    // fast moves two nodes at a time.
    const struct Node *slow = head;
    const struct Node *fast = head;

    // Continue only while fast and the next node both exist.
    // This prevents dereferencing NULL when reaching the end
    // of a list that has no cycle.
    while (fast != NULL && fast->next != NULL) 
    {

        // Move slow forward by one node.
        // Example: slow = node 1 -> node 2.
        slow = slow->next;

        // Move fast forward by two nodes.
        // Example: node 1 -> node 2 -> node 3.
        fast = fast->next->next;

        // DRY RUN FOR A LIST WITH A CYCLE:
        //
        // List:
        // 1 -> 2 -> 3 -> 4 -> 5
        //           ^         |
        //           |_________|
        //
        // The next pointer of node 5 points back to node 3.
        //
        // Initially:
        // slow = node 1, fast = node 1
        //
        // Iteration 1:
        // slow = node 2
        // fast = node 3
        // slow == fast? No.
        //
        // Iteration 2:
        // slow = node 3
        // fast = node 5
        // slow == fast? No.
        //
        // Iteration 3:
        // slow = node 4
        // fast = node 4 (node 5 -> node 3 -> node 4)
        // slow == fast? Yes!
        // Return true because a cycle exists.

        // Compare node addresses, not the values stored in nodes.
        if (slow == fast) 
        {
            return true;
        }
    }

    // If fast reaches NULL, the list has an end and no cycle exists.
    return false;
}

```

### Walkthrough, step by step
Imagine two runners on a circular track. One moves one step at a time (`slow`), while the other moves two steps at a time (`fast`). If the track has a loop, the faster runner eventually catches the slower one. If the list ends, the fast runner reaches `NULL` and there is no cycle.

For a list `A -> B -> C -> D -> NULL`, both runners start at `A`. First iteration: `slow` moves to `B`, and `fast` moves to `C`. They are different. Second iteration: `slow` moves to `C`, and `fast` attempts two steps from `C`, landing at `NULL` after `D`. The loop condition then fails, so the function returns `false`.

Now imagine `A -> B -> C -> D -> B`, where `D` points back to `B`. Both start at `A`. After the first iteration, `slow = B` and `fast = C`. After the second, `slow = C` and `fast = B`. After the third, `slow = D` and `fast = D`. Their pointers are equal, so the function returns `true`.

The condition checks `fast` and `fast->next` before moving two steps. Without that check, evaluating `fast->next->next` could dereference `NULL`.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

**Important assumption:** All non-null `next` pointers must point to valid `Node` objects. No algorithm can safely inspect arbitrary invalid pointers.

---

## 17. Reverse a Singly Linked List

### Problem
Reverse a list such as `10 -> 20 -> 30 -> NULL` into `30 -> 20 -> 10 -> NULL`.

### C solution
```c

struct Node {
    int value;
    struct Node *next;
};

struct Node *reverse_list(struct Node *head)
{
    // previous points to the already reversed part.
    // Initially, nothing is reversed, so previous = NULL.
    struct Node *previous = NULL;

    // current points to the node we are currently processing.
    // Initially, current points to node 10.
    struct Node *current = head;

    // Continue until every node has been processed.
    while (current != NULL) {

        // ITERATION 1: current points to node 10.
        //
        // Step 1: Save the next node before changing any links.
        // next_node = node 20.
        struct Node *next_node = current->next;

        // Step 2: Reverse the current node's link.
        // 10 -> 20 becomes 10 -> NULL.
        current->next = previous;

        // Step 3: Move previous to the node just reversed.
        // previous now points to node 10.
        previous = current;

        // Step 4: Move current to the saved next node.
        // current now points to node 20.
        current = next_node;

        // STATE AFTER ITERATION 1:
        // Reversed part: 10 -> NULL
        // Remaining part: 20 -> 30 -> NULL
        //
        // ITERATION 2: current points to node 20.
        //
        // next_node = node 30.
        // current->next = previous:
        // 20 -> 10 -> NULL
        // previous = current: previous points to node 20.
        // current = next_node: current points to node 30.
        //
        // STATE AFTER ITERATION 2:
        // Reversed part: 20 -> 10 -> NULL
        // Remaining part: 30 -> NULL
        //
        // ITERATION 3: current points to node 30.
        //
        // next_node = NULL.
        // current->next = previous:
        // 30 -> 20 -> 10 -> NULL
        // previous = current: previous points to node 30.
        // current = next_node: current becomes NULL.
        //
        // STATE AFTER ITERATION 3:
        // Reversed list: 30 -> 20 -> 10 -> NULL
        // current = NULL, so the loop ends.
    }

    // previous points to the new head, node 30.
    return previous;
}

```

### Walkthrough, step by step
Every node points forward to the next node. To reverse the list, we must change each arrow's direction. But if we overwrite `current->next` before saving its old value, we lose the route to the remaining list. So the first step of every iteration saves the next node.

Initially, `previous = NULL` and `current` points to node `10`. We save `next_node = 20`. Then we change `10->next` from `20` to `NULL`, making node `10` a one-node reversed list. We advance `previous` to `10` and `current` to `20`.

In the second iteration, we save `30`, change `20->next` to `10`, and advance. Now the reversed section is `20 -> 10 -> NULL`, while `current` points to `30`.

In the third iteration, `next_node` becomes `NULL`. We change `30->next` to `20`, so the list is now `30 -> 20 -> 10 -> NULL`. Then `previous` becomes `30` and `current` becomes `NULL`. The loop ends, and `previous` is the new head.

This algorithm does not create new nodes; it changes the `next` pointers in the existing nodes. Therefore, the caller must store the returned head. Calling `reverse_list(head)` and then continuing to use the old `head` as though it were still the first node would lose access to the front of the reversed list.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

---

## 18. Remove a Node by Value From a Linked List

### Problem
Remove the first node whose value matches `target`. Return the potentially changed head. This version frees the removed node, so it assumes the list owns nodes allocated in a way compatible with `free`.

### C solution
```c

#include <stdlib.h>

struct Node {
    int value;
    struct Node *next;
};

struct Node *remove_first(struct Node *head, int target)
{
    // previous tracks the node before current.
    // Initially, current is the head, so no previous node exists.
    struct Node *previous = NULL;
    struct Node *current = head;

    // Search until:
    // 1. current becomes NULL (end of list), or
    // 2. current->value matches target.
    //
    // DRY RUN:
    // List: 10 -> 20 -> 30 -> 20 -> NULL
    // Target = 20
    //
    // Initial:
    // previous = NULL, current = 10
    //
    // Iteration 1:
    // current->value = 10, not equal to 20.
    // previous = current -> previous points to 10.
    // current = current->next -> current points to 20.
    //
    // Stop searching because current->value == target.

    while (current != NULL && current->value != target) 
    {
        previous = current;
        current = current->next;
    }

    // If current is NULL, we reached the end without finding target.
    // Nothing needs to be removed, so return the original head.
    if (current == NULL)
    {
        return head;
    }

    // If previous is NULL, current is the first node (the head).
    // Example: 20 -> 30 -> NULL, target = 20.
    if (previous == NULL) 
    {
        // Move head to the next node.
        // The node containing 20 is no longer part of the list.
        head = current->next;

    } 
    else 
    {
        // Otherwise, bypass the node being removed.
        //
        // Before:
        // previous -> current -> next
        //    10         20       30
        //
        // Set previous->next = current->next.
        // After:
        // 10 -> 30
        //
        // In our dry run, the list becomes:
        // 10 -> 30 -> 20 -> NULL
        previous->next = current->next;
    }

    // Release the removed node's dynamically allocated memory.
    // Do not access current after this call.
    free(current);

    // Return the possibly updated head.
    // This matters when the first node was removed.
    return head;
}

```

### Walkthrough, step by step
We use two pointers because deleting a node from a singly linked list requires changing the previous node's `next` pointer. `current` examines nodes; `previous` remembers the node immediately before it.

Suppose the list is `10 -> 20 -> 30 -> NULL`, and `target = 20`. Initially, `previous = NULL`, `current = 10`. Since `10` is not the target, we move `previous` to `10` and `current` to `20`. Now the loop stops because `current->value == 20`.

`previous` is not `NULL`, so we set `previous->next = current->next`. In plain terms, node `10` now points directly to node `30`, skipping node `20`. We then free the removed node. The list is now `10 -> 30 -> NULL`, and the original head remains correct.

If the target is `10`, `current` matches immediately and `previous` is still `NULL`. That means the first node is being removed, so the head must be updated to `current->next`, which points to `20`. This is why the function returns the new head: removing the first node changes it.

If the value does not exist, `current` eventually becomes `NULL`. The function returns the original head unchanged.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

**Ownership warning:** Never call `free(current)` if nodes are stack-allocated, embedded inside another object, or owned by another component. The code must match the list's memory-ownership contract.

---

## 19. Find the Middle Node of a Linked List

### Problem
Return the middle node using one traversal. For an even number of nodes, this implementation returns the **second** middle node.

### C solution
```c
struct Node {
    int value;
    struct Node *next;
};

struct Node *middle_node(struct Node *head)
{
    struct Node *slow = head;
    struct Node *fast = head;

    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;
}
```

### Walkthrough, step by step
The slow pointer moves one node each iteration, while the fast pointer moves two. When the fast pointer reaches the end, the slow pointer has covered roughly half the list.

For `A -> B -> C -> D -> E`, both begin at `A`. After the first iteration, slow is `B` and fast is `C`. After the second, slow is `C` and fast is `E`. One more loop is not possible because `E->next` is `NULL`. The function returns `C`, the middle node.

For `A -> B -> C -> D`, after the first iteration slow is `B` and fast is `C`. On the next iteration, slow becomes `C` and fast becomes `NULL`. The function returns `C`, the second of the two middle nodes (`B` and `C`).

If the list is empty, both pointers are `NULL`, the loop never runs, and the function returns `NULL`.

**Complexity:** \(O(n)\) time and \(O(1)\) extra space.

---

## 20. Common Pointer and Memory Mistakes

### Mistake A: Returning the address of a local variable
```c
int *bad_pointer(void)
{
    int local = 10;
    return &local; /* Invalid: local's lifetime ends when the function returns. */
}
```

**Story:** `local` exists only while `bad_pointer` is executing. When the function returns, that local object's lifetime ends. The returned address no longer designates a live `int`. Dereferencing it later is undefined behavior. The address may appear to work during one debug run and fail in another.

**Better options:** Return the integer by value; let the caller provide an output pointer; or allocate an object dynamically when dynamic lifetime is genuinely needed and document who frees it.

### Mistake B: Confusing `sizeof(pointer)` with array size
```c
void inspect(int a[])
{
    /* Here a is adjusted to int *; sizeof(a) is pointer size. */
}
```

**Story:** In a function parameter, `int a[]` is adjusted to `int *a`. The function receives a pointer, not a complete array object. `sizeof(a)` therefore reports the size of a pointer, not the number of elements. Pass the element count as a separate parameter.

### Mistake C: Dereferencing before checking for `NULL`
```c
if (p != NULL) {
    use_value(*p);
}
```

**Story:** A pointer may represent “no object” by containing `NULL`. Dereferencing it asks the program to read an object at an address that is not valid for that purpose. Check the pointer before dereferencing, and also ensure that non-null pointers really point to live objects of the right type.

### Mistake D: Double free
```c
free(p);
/* ... */
free(p); /* Undefined behavior if p was already freed and not reassigned. */
```

**Story:** The first `free` returns the allocation to the allocator. The second call attempts to release the same allocation again. That violates the allocator's contract and causes undefined behavior. Setting one pointer to `NULL` after freeing it helps only for later uses of that same variable; other aliases must still be managed.

### Mistake E: Losing the original allocation pointer
```c
p = malloc(100U);
p = malloc(200U); /* If the first allocation succeeded, its address is lost. */
```

**Story:** After the first allocation, `p` is the only reference shown to that block. Assigning a second allocation directly into `p` overwrites the address. If the first allocation succeeded, the program can no longer free it through `p`, causing a leak. Keep the first address or release the first allocation before replacing it.

---

## Practice prompts: Explain these aloud

1. Why does `swap_int` need a temporary variable?
2. Why must a bounded string-copy function reserve space for `'\0'`?
3. Why is a pointer still dangerous after its allocation has been freed, even if it is not `NULL`?
4. Why does Floyd's cycle detection use one pointer moving twice as fast?
5. Why must linked-list reversal save `current->next` before changing it?
6. Why does removing the head node require returning a new head pointer?
7. What is the difference between a memory leak, dangling pointer, use-after-free, and double free?
8. Why is `sizeof(a)` not the element count inside a function receiving `int a[]`?

## End-of-part summary

The central habit for pointer questions is to distinguish three things:

- **The pointer variable:** stores an address.
- **The object:** the memory at that address.
- **The object's lifetime and ownership:** whether the object is still valid and who is responsible for releasing it.

Most pointer bugs occur when code confuses these three. When you explain your solution in an interview, say what each pointer refers to before and after the important statements.


### Question: Calculate the Ant's Final Distance
An ant moves on a floor in four diagonal directions. The directions are defined from the perspective of a fixed aerial observer.

You are given an array moves containing integers from 0 to 3. Calculate the Euclidean distance between the ant's starting position and its final position. Return the integer part of the distance by truncating the decimal portion.

The directions are:
- 0: Up and right (x + 1, y + 1)
- 1: Down and right (x + 1, y - 1)
- 2: Down and left (x - 1, y - 1)
- 3: Up and left (x - 1, y + 1)
Example

Input: moves = [0, 3, 0, 0, 2, 0, 0]

Output: 5

Explanation: The ant finishes at position (3, 5). Its distance from the origin is \(\sqrt{3^2 + 5^2} = \sqrt{34} \approx 5.83\). After truncation, the answer is 5.

```c++

#include <math.h>

typedef struct intArray
{
    int length;
    int items[10];
} intArray;

int computeDistance(intArray moves)
{
    int h = 0; // Horizontal position
    int v = 0; // Vertical position

    // Process each move.
    for (int i = 0; i < moves.length; i++)
    {
        int curMove = moves.items[i];

        // Moves 0 and 1 go right.
        // Moves 2 and 3 go left.
        if (curMove == 0 || curMove == 1)
        {
            h++;
        }
        else if (curMove == 2 || curMove == 3)
        {
            h--;
        }
        else
        {
            return -1; // Invalid direction
        }

        // Moves 0 and 3 go up.
        // Moves 1 and 2 go down.
        if (curMove == 0 || curMove == 3)
        {
            v++;
        }
        else if (curMove == 1 || curMove == 2)
        {
            v--;
        }
        else
        {
           return -1; // Invalid direction
        }
    }

    // Calculate straight-line distance from (0, 0).
    double distance = sqrt((double)h * h + (double)v * v);

    // Convert to int to truncate the fractional part.
    return (int)distance;
}

```

### Question 2: Circular Track — Find the Final Position
Problem
A runner moves around a circular track containing N positions, numbered from 0 to N-1.

The runner starts at position 0. Each array element specifies the number of steps:
- Positive values move clockwise.
- Negative values move anticlockwise.
The runner wraps around the track when crossing either end. Return the final position.
```c++
//N = 10
//moves = [4, 8, -3, 2]

int circular_position(const int moves[], int length, int N, int X)
{
    if (N <= 0 || length < 0 ||
        X < 0 || X >= N ||
        (length > 0 && moves == NULL))
    {
        return -1;
    }

    int endpos = X;  // Runner starts at position X

    for (int i = 0; i < length; i++)
    {
        endpos = (int)(((long long)endpos + moves[i]) % N);

        // Convert a negative remainder to a valid position
        if (endpos < 0)
        {
            endpos += N;
        }
    }

    return endpos;
}


```

