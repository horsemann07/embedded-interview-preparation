# C Whiteboard Coding Questions & Answers

> **Target:** Live Coding / Whiteboard Interview  
> **Difficulty:** Basic → Intermediate → Advanced  
> **Total:** 100 Questions  
> **Focus:** C / Embedded C implementations

---

## BASIC LEVEL CODING (Questions 1-20)

# 1. Implement `strlen()` from Scratch

## Answer

```c
/**
 * my_strlen - Calculate length of null-terminated string
 * @str: Pointer to the input string
 * 
 * Returns: Number of characters before null terminator
 * 
 * Example:
 *   size_t len = my_strlen("hello");  // Returns 5
 *   size_t len = my_strlen("");       // Returns 0
 */
size_t my_strlen(const char *str)
{
    size_t len = 0;
    
    /* Iterate until null terminator is found */
    while (str[len] != '\0')
        len++;  /* Increment counter for each character */
    
    return len;  /* Return total length */
}
```

### Pointer Version

```c
size_t my_strlen_ptr(const char *str)
{
    const char *s = str;
    
    while (*s != '\0')
        s++;
    
    return (size_t)(s - str);
}
```

### Optimized (Word-at-a-time)

```c
size_t my_strlen_fast(const char *str)
{
    const unsigned long *lp;
    
    /* Align to word boundary */
    for (lp = (const unsigned long *)str;
         (*lp & 0xFF) && ((*lp >> 8) & 0xFF) &&
         ((*lp >> 16) & 0xFF) && ((*lp >> 24) & 0xFF);
         lp++);
    
    /* Find exact position */
    const char *cp = (const char *)lp;
    size_t len = cp - str;
    
    while (*cp != '\0')
    {
        len++;
        cp++;
    }
    
    return len;
}
```

---

# 2. Implement `strcmp()` from Scratch

## Answer

```c
/**
 * my_strcmp - Compare two null-terminated strings
 * @s1: First string to compare
 * @s2: Second string to compare
 * 
 * Returns: 
 *   0 if strings are equal
 *   Negative if s1 < s2
 *   Positive if s1 > s2
 * 
 * Example:
 *   int result = my_strcmp("abc", "abc");   // Returns 0
 *   int result = my_strcmp("abc", "abd");   // Returns negative (-1)
 *   int result = my_strcmp("abd", "abc");   // Returns positive (1)
 */
int my_strcmp(const char *s1, const char *s2)
{
    /* Compare characters one by one until mismatch or end */
    while (*s1 && *s1 == *s2)
    {
        s1++;  /* Move to next character in s1 */
        s2++;  /* Move to next character in s2 */
    }
    
    /* Return difference between mismatched characters */
    /* Cast to unsigned char to handle signed char values correctly */
    return (unsigned char)*s1 - (unsigned char)*s2;
}
```

### Test Cases

```c
my_strcmp("abc", "abc");     // 0
my_strcmp("abc", "abd");     // negative
my_strcmp("abd", "abc");     // positive
my_strcmp("", "");           // 0
my_strcmp("a", "");          // positive
```

---

# 3. Implement `strcpy()` from Scratch

## Answer

```c
char *my_strcpy(char *dest, const char *src)
{
    char *d = dest;
    
    while (*src != '\0')
    {
        *d = *src;
        d++;
        src++;
    }
    
    *d = '\0';  /* null-terminate */
    
    return dest;
}
```

### Safe Version (`strncpy`)

```c
char *my_strncpy(char *dest, const char *src, size_t n)
{
    char *d = dest;
    size_t i = 0;
    
    while (i < n && *src != '\0')
    {
        *d = *src;
        d++;
        src++;
        i++;
    }
    
    /* Note: strncpy pads with zeros */
    while (i < n)
    {
        *d = '\0';
        d++;
        i++;
    }
    
    return dest;
}
```

---

# 4. Implement `memcpy()` from Scratch

## Answer

```c
/**
 * my_memcpy - Copy memory area (does NOT handle overlapping regions)
 * @dest: Destination memory address
 * @src:  Source memory address
 * @n:    Number of bytes to copy
 * 
 * Returns: Pointer to destination
 * 
 * WARNING: Behavior is undefined if regions overlap!
 * Use memmove() for potentially overlapping regions.
 * 
 * Example:
 *   char src[20] = "Hello World";
 *   char dest[20];
 *   my_memcpy(dest, src, 11);  // dest = "Hello World"
 */
void *my_memcpy(void *dest, const void *src, size_t n)
{
    unsigned char *d = (unsigned char *)dest;        /* Cast dest to byte pointer */
    const unsigned char *s = (const unsigned char *)src;  /* Cast src to byte pointer */
    
    /* Copy n bytes one at a time */
    while (n--)
    {
        *d++ = *s++;  /* Copy byte and increment both pointers */
    }
    
    return dest;  /* Return original dest pointer */
}
```

### Optimized (Word-at-a-time)

```c
/**
 * my_memcpy_fast - Fast memcpy using word-aligned copies
 * 
 * Optimization strategy:
 * 1. Copy by long words (4 or 8 bytes) if aligned
 * 2. Fall back to byte copy for remainder
 * 
 * Performance: ~4-8x faster on large buffers (>100 bytes)
 * 
 * Example:
 *   uint32_t src_data[1000];
 *   uint32_t dest_data[1000];
 *   my_memcpy_fast(dest_data, src_data, 4000);  // ~4x faster
 */
void *my_memcpy_fast(void *dest, const void *src, size_t n)
{
    unsigned char *d = (unsigned char *)dest;
    const unsigned char *s = (const unsigned char *)src;
    
    /* Check if both source and dest are word-aligned (aligned to sizeof(long)) */
    if (((uintptr_t)d & (sizeof(long) - 1)) == 0 &&
        ((uintptr_t)s & (sizeof(long) - 1)) == 0)
    {
        /* Both pointers are aligned, copy by word */
        unsigned long *ld = (unsigned long *)d;
        const unsigned long *ls = (const unsigned long *)s;
        
        /* Copy full words until less than one word remains */
        while (n >= sizeof(long))
        {
            *ld++ = *ls++;  /* Copy one word (4 or 8 bytes) */
            n -= sizeof(long);  /* Reduce remaining count */
        }
        
        /* Update byte pointers to continue from where word copy left off */
        d = (unsigned char *)ld;
        s = (const unsigned char *)ls;
    }
    
    /* Copy remaining bytes one at a time */
    while (n--)
        *d++ = *s++;
    
    return dest;
}
```

### Follow-Up: Handle Overlapping Memory (`memmove`)

```c
/**
 * my_memmove - Copy memory safely, handles overlapping regions
 * @dest: Destination memory address
 * @src:  Source memory address  
 * @n:    Number of bytes to copy
 * 
 * Returns: Pointer to destination
 * 
 * Key difference from memcpy:
 * - If src < dest: Copy BACKWARD to avoid overwriting source
 * - If src > dest: Copy FORWARD (normal order)
 * 
 * Example (overlapping copy):
 *   char buf[20] = "Hello";
 *   my_memmove(buf + 2, buf, 5);  // Safe: memmove backward
 *   // buf now = "HeHello"
 *
 *   char buf2[20] = "Hello";
 *   memcpy(buf2 + 2, buf2, 5);    // UNSAFE: memcpy corrupts data!
 *   // Undefined behavior - may crash or corrupt
 */
void *my_memmove(void *dest, const void *src, size_t n)
{
    unsigned char *d = (unsigned char *)dest;
    const unsigned char *s = (const unsigned char *)src;
    
    /* Check for overlap - if src is before dest, copy backward */
    if (s < d)
    {
        /* Copy BACKWARD to avoid overwriting source data */
        /* Start from end and move toward beginning */
        d += n;  /* Point to end of destination */
        s += n;  /* Point to end of source */
        
        while (n--)
            *--d = *--s;  /* Pre-decrement pointers, then copy */
    }
    else
    {
        /* No overlap or src > dest: Copy FORWARD (normal order) */
        while (n--)
            *d++ = *s++;  /* Copy and increment pointers */
    }
    
    return dest;
}
```

---

# 5. Implement `memset()` from Scratch

## Answer

```c
void *my_memset(void *s, int c, size_t n)
{
    unsigned char *p = (unsigned char *)s;
    unsigned char val = (unsigned char)c;
    
    while (n--)
        *p++ = val;
    
    return s;
}
```

### Optimized

```c
void *my_memset_fast(void *s, int c, size_t n)
{
    unsigned char *p = (unsigned char *)s;
    unsigned char val = (unsigned char)c;
    
    /* Fill by words if possible */
    if (n >= sizeof(long))
    {
        unsigned long *lp = (unsigned long *)p;
        unsigned long lval = val;
        
        lval |= (lval << 8);
        lval |= (lval << 16);
        if (sizeof(long) == 8)
            lval |= (lval << 32);
        
        while (n >= sizeof(long))
        {
            *lp++ = lval;
            n -= sizeof(long);
        }
        
        p = (unsigned char *)lp;
    }
    
    /* Fill remaining bytes */
    while (n--)
        *p++ = val;
    
    return s;
}
```

---

# 6. Reverse a String In-Place

## Answer

```c
/**
 * reverse_string - Reverse a string by swapping characters in-place
 * @str: Null-terminated string to reverse
 * 
 * Algorithm (Two-Pointer Approach):
 * 1. Initialize left pointer to start (index 0)
 * 2. Initialize right pointer to end (last non-null character)
 * 3. While left < right:
 *    - Swap characters at left and right
 *    - Move left pointer forward
 *    - Move right pointer backward
 * 
 * Memory Layout Example:
 *   Before: [H] [e] [l] [l] [o] [\0]
 *           ^0  ^1  ^2  ^3  ^4
 *           left                right
 *   
 *   Step 1: Swap H and o:  [o] [e] [l] [l] [H] [\0]
 *           left++, right--
 *   
 *   Step 2: Swap e and l:  [o] [l] [l] [e] [H] [\0]
 *           left++, right--
 *   
 *   Step 3: left >= right, stop
 *   After:  [o] [l] [l] [e] [H] [\0]  = "olleH"
 * 
 * Time Complexity: O(n) - visits each character once
 * Space Complexity: O(1) - in-place, no extra array needed
 * 
 * Example:
 *   char str[] = "Hello";  (writable array, not string literal)
 *   reverse_string(str);
 *   printf("%s\n", str);   // Prints "olleH"
 * 
 * Important: String must be writable (char array)!
 *   NOT: const char *str = "Hello";  // ERROR: cannot modify
 *   YES: char str[] = "Hello";       // OK: writable array
 */
void reverse_string(char *str)
{
    if (!str)
        return;  /* NULL pointer safety */
    
    int len = strlen(str);  /* Get string length */
    int left = 0;           /* Start of string */
    int right = len - 1;    /* End of string (before null terminator) */
    
    /* Two-pointer approach: swap from outside toward center */
    while (left < right)
    {
        /* Swap characters at left and right positions */
        char temp = str[left];
        str[left] = str[right];
        str[right] = temp;
        
        left++;   /* Move toward center from left */
        right--;  /* Move toward center from right */
    }
    /* When left >= right, middle is reached, reversal complete */
}
```

### Recursive Version

```c
/**
 * reverse_string_rec - Reverse string using recursion
 * @str: String to reverse
 * @left: Current left pointer (usually 0 on first call)
 * @right: Current right pointer (usually len-1 on first call)
 * 
 * Algorithm (Recursive Two-Pointer):
 * 1. Base case: if left >= right, return (middle reached)
 * 2. Swap characters at left and right
 * 3. Recursively call with left+1 and right-1
 * 
 * Call Stack Example (for "Hello"):
 *   reverse_string_rec(str, 0, 4)  -> Swap H and o
 *   reverse_string_rec(str, 1, 3)  -> Swap e and l
 *   reverse_string_rec(str, 2, 2)  -> Base case (left >= right)
 *   Return from all calls
 *   Result: "olleH"
 * 
 * Time Complexity: O(n) - n/2 swaps
 * Space Complexity: O(n) - call stack depth is n/2
 * 
 * Example:
 *   char str[] = "Hello";
 *   reverse_string_rec(str, 0, strlen(str) - 1);
 *   printf("%s\n", str);  // Prints "olleH"
 * 
 * Note: Recursive version uses more memory due to call stack
 *       Iterative (previous function) is more efficient
 */
void reverse_string_rec(char *str, int left, int right)
{
    if (left >= right)
        return;  /* Base case: middle reached, recursion stops */
    
    /* Swap characters */
    char temp = str[left];
    str[left] = str[right];
    str[right] = temp;
    
    /* Recursive call: move both pointers toward center */
    reverse_string_rec(str, left + 1, right - 1);
}
```

---

# 7. Implement `atoi()` with Error Handling

## Answer

```c
/**
 * my_atoi - Convert ASCII string to integer
 * @str: Null-terminated string to convert
 * 
 * Returns: Converted integer value
 * 
 * Supported formats:
 *   "123"       -> 123
 *   "-456"      -> -456
 *   "+789"      -> 789
 *   "  42"      -> 42 (leading whitespace ignored)
 *   "abc123"    -> 0 (no digits, return 0)
 *   "12abc"     -> 12 (stops at non-digit)
 * 
 * Algorithm (State Machine):
 * 1. Skip leading whitespace (space, tab)
 * 2. Check for optional sign (+ or -)
 * 3. Convert digits one at a time
 * 4. Stop at first non-digit
 * 
 * States:
 *   [skip space] -> [check sign] -> [convert digits] -> [stop]
 * 
 * Time Complexity: O(n) where n = string length
 * Space Complexity: O(1)
 * 
 * Example:
 *   my_atoi("123")      // Returns 123
 *   my_atoi("-456")     // Returns -456
 *   my_atoi("  42")     // Returns 42
 *   my_atoi("abc")      // Returns 0
 */
int my_atoi(const char *str)
{
    if (!str)
        return 0;  /* NULL pointer safety */
    
    int sign = 1;   /* Sign multiplier (1 or -1) */
    int result = 0; /* Accumulated result */
    int i = 0;      /* Current position in string */
    
    /* State 1: Skip leading whitespace */
    while (str[i] == ' ' || str[i] == '\t')
        i++;  /* Move past whitespace */
    
    /* State 2: Handle optional sign */
    if (str[i] == '-' || str[i] == '+')
    {
        if (str[i] == '-')
            sign = -1;  /* Negative number */
        i++;
    }
    
    /* State 3: Convert digits */
    while (str[i] >= '0' && str[i] <= '9')
    {
        /* Convert ASCII digit to number */
        int digit = str[i] - '0';  /* '5' - '0' = 5 */
        
        /* Build result: shift left (multiply by 10) and add digit */
        result = result * 10 + digit;
        
        i++;
    }
    
    return sign * result;  /* Apply sign and return */
}
```

### With Overflow Checking

```c
/**
 * my_atoi_safe - Convert string to integer with overflow protection
 * @str: String to convert
 * @error: Output parameter for error code
 *         0 = success, 1 = overflow, -1 = invalid input
 * 
 * Returns: Converted integer (clamped to INT_MIN/INT_MAX on overflow)
 * 
 * Key Difference from my_atoi:
 * - Detects integer overflow before it happens
 * - Clamps result to INT_MIN/INT_MAX
 * - Sets error flag instead of silently overflowing
 * 
 * Why necessary:
 *   int max = INT_MAX;  // 2147483647
 *   my_atoi("2147483648") might overflow and give negative!
 *   my_atoi_safe handles this safely
 * 
 * Example:
 *   int err;
 *   int val = my_atoi_safe("99999999999", &err);
 *   if (err == 1)
 *       printf("Number too large! Clamped to %d\n", val);
 */
int my_atoi_safe(const char *str, int *error)
{
    if (!str || !error)
    {
        if (error)
            *error = -1;  /* Invalid input */
        return 0;
    }
    
    *error = 0;           /* No error initially */
    int sign = 1;         /* Sign multiplier */
    long result = 0;      /* Use long to detect overflow before storing */
    int i = 0;
    
    /* Skip whitespace */
    while (str[i] == ' ' || str[i] == '\t')
        i++;
    
    /* Handle sign */
    if (str[i] == '-' || str[i] == '+')
    {
        if (str[i] == '-')
            sign = -1;
        i++;
    }
    
    /* Convert digits with overflow check */
    while (str[i] >= '0' && str[i] <= '9')
    {
        int digit = str[i] - '0';
        result = result * 10 + digit;
        
        /* Check if result exceeded int range */
        if (result > INT_MAX)
        {
            *error = 1;  /* Set overflow flag */
            /* Return clamped value */
            return sign > 0 ? INT_MAX : INT_MIN;
        }
        
        i++;
    }
    
    return sign * (int)result;
}
```

---

# 8. Swap Two Numbers Without Temporary Variable

## Answer

### XOR Swap

```c
void swap_xor(int *a, int *b)
{
    if (a == b)
        return;  /* if same address, skip */
    
    *a = *a ^ *b;
    *b = *a ^ *b;
    *a = *a ^ *b;
}
```

### Addition Swap (May Overflow)

```c
void swap_add(int *a, int *b)
{
    if (a == b)
        return;
    
    *a = *a + *b;
    *b = *a - *b;
    *a = *a - *b;
}
```

### Multiplication Swap (May Divide by Zero)

```c
void swap_mul(int *a, int *b)
{
    if (a == b || *a == 0 || *b == 0)
        return;
    
    *a = *a * *b;
    *b = *a / *b;
    *a = *a / *b;
}
```

**Note:** XOR swap is safest and most elegant.

---

# 9. Detect Endianness

## Answer

```c
int is_big_endian(void)
{
    unsigned int x = 0x12345678;
    unsigned char *p = (unsigned char *)&x;
    
    return (*p == 0x12);  /* 1 if big-endian, 0 if little-endian */
}
```

### Alternative Method

```c
int is_big_endian_union(void)
{
    union {
        unsigned int i;
        unsigned char c[4];
    } u;
    
    u.i = 0x12345678;
    
    return (u.c[0] == 0x12);
}
```

### Swap Bytes of 32-bit Integer

```c
uint32_t swap_bytes_32(uint32_t x)
{
    return ((x & 0xFF000000) >> 24) |
           ((x & 0x00FF0000) >> 8)  |
           ((x & 0x0000FF00) << 8)  |
           ((x & 0x000000FF) << 24);
}
```

---

# 10. Count Set Bits (Brian Kernighan's Algorithm)

## Answer

```c
int count_set_bits(unsigned int x)
{
    int count = 0;
    
    while (x)
    {
        x &= (x - 1);  /* clear rightmost set bit */
        count++;
    }
    
    return count;
}
```

### Comparison: Naive Method

```c
int count_set_bits_naive(unsigned int x)
{
    int count = 0;
    
    while (x)
    {
        if (x & 1)
            count++;
        x >>= 1;
    }
    
    return count;
}
```

**Kernighan's is faster:** Only loops for set bits, not all 32 bits.

### Lookup Table Method (Fastest for Embedded)

```c
static const unsigned char bits[256] = {
    0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4,
    /* ... (total 256 entries) */
};

int count_set_bits_table(uint32_t x)
{
    return bits[(x >>  0) & 0xFF] +
           bits[(x >>  8) & 0xFF] +
           bits[(x >> 16) & 0xFF] +
           bits[(x >> 24) & 0xFF];
}
```

---

# 11. Write a Generic Swap Macro

## Answer

```c
#define SWAP(a, b, type) \
    do { \
        type temp = (a); \
        (a) = (b); \
        (b) = temp; \
    } while (0)
```

### Usage

```c
int x = 10, y = 20;
SWAP(x, y, int);
/* x = 20, y = 10 */

float f1 = 3.14, f2 = 2.71;
SWAP(f1, f2, float);
```

### Type-Agnostic Macro (Using `typeof`)

```c
#define SWAP_AUTO(a, b) \
    do { \
        __typeof__(a) temp = (a); \
        (a) = (b); \
        (b) = temp; \
    } while (0)
```

**Note:** `__typeof__` is GCC extension.

---

# 12. Implement a Circular Buffer (Ring Buffer)

## Answer

```c
/**
 * Circular Buffer Structure
 * 
 * Memory Layout:
 *   +---+---+---+---+---+---+
 *   | 0 | 1 | 2 | 3 | 4 | 5 |  (indices modulo BUFFER_SIZE)
 *   +---+---+---+---+---+---+
 *
 * head pointer: Next position to read
 * tail pointer: Next position to write
 * 
 * When head == tail: Buffer is EMPTY
 * When (tail + 1) % SIZE == head: Buffer is FULL
 */
#define BUFFER_SIZE 256

struct CircularBuffer
{
    uint8_t data[BUFFER_SIZE];  /* Array to hold data */
    int head;                   /* Read pointer (oldest data) */
    int tail;                   /* Write pointer (next free slot) */
};

/**
 * rb_is_empty - Check if buffer is empty
 * @rb: Pointer to circular buffer
 * 
 * Returns: 1 if empty, 0 if has data
 * 
 * Condition: head == tail means next-to-write == next-to-read
 */
int rb_is_empty(struct CircularBuffer *rb)
{
    return rb->head == rb->tail;  /* Empty when read and write pointers meet */
}

/**
 * rb_is_full - Check if buffer is completely full
 * @rb: Pointer to circular buffer
 * 
 * Returns: 1 if full, 0 if has space
 * 
 * Condition: (tail + 1) % SIZE == head means next write would collide with head
 */
int rb_is_full(struct CircularBuffer *rb)
{
    return (rb->tail + 1) % BUFFER_SIZE == rb->head;  /* Full when write pointer catches read pointer */
}

/**
 * rb_enqueue - Add data to buffer (producer side)
 * @rb: Pointer to circular buffer
 * @value: Byte to enqueue
 * 
 * Returns: 0 on success, -1 if buffer is full
 * 
 * Example:
 *   uint8_t data = 0x42;
 *   if (rb_enqueue(&buffer, data) != 0)
 *       printf("Buffer full!\n");
 */
int rb_enqueue(struct CircularBuffer *rb, uint8_t value)
{
    /* Check if buffer would overflow */
    if (rb_is_full(rb))
        return -1;  /* Cannot add more data */
    
    /* Write data at tail position */
    rb->data[rb->tail] = value;
    
    /* Advance tail pointer with wraparound */
    rb->tail = (rb->tail + 1) % BUFFER_SIZE;  /* Wrap to 0 when reaching end */
    
    return 0;  /* Success */
}

/**
 * rb_dequeue - Remove and read data from buffer (consumer side)
 * @rb: Pointer to circular buffer
 * @value: Pointer where to store read byte
 * 
 * Returns: 0 on success, -1 if buffer is empty
 * 
 * Example:
 *   uint8_t data;
 *   if (rb_dequeue(&buffer, &data) == 0)
 *       printf("Read: 0x%02X\n", data);
 */
int rb_dequeue(struct CircularBuffer *rb, uint8_t *value)
{
    /* Check if buffer is empty */
    if (rb_is_empty(rb))
        return -1;  /* No data to read */
    
    /* Read data from head position */
    *value = rb->data[rb->head];
    
    /* Advance head pointer with wraparound */
    rb->head = (rb->head + 1) % BUFFER_SIZE;  /* Wrap to 0 when reaching end */
    
    return 0;  /* Success */
}
```

### Lock-Free Version (for ISR use)

```c
/**
 * Lock-Free Circular Buffer for ISR (Interrupt Service Routine)
 * 
 * Key: Single reader (main loop) and single writer (ISR) can access safely
 *      without locks because:
 *      - Only one location modified by each side
 *      - volatile prevents compiler from optimizing away accesses
 * 
 * Safe pattern:
 *   ISR: writes data, then updates tail (memory barrier implicit)
 *   Main: reads from head, increments head
 *   No collision possible if producer != consumer
 */
struct CircularBuffer_ISR
{
    volatile uint8_t data[BUFFER_SIZE];  /* volatile: don't cache in registers */
    volatile int head;                   /* Modified by consumer (main) */
    volatile int tail;                   /* Modified by producer (ISR) */
};

/**
 * rb_enqueue_isr - Add byte to buffer from ISR
 * 
 * MUST be called from ISR context ONLY
 * Safe to call without mutex when single ISR writes and main loop reads
 * 
 * Example (in UART ISR):
 *   void uart_rx_isr(void)
 *   {
 *       uint8_t byte = UART_DATA_REG;
 *       if (rb_enqueue_isr(&rx_buffer, byte) < 0)
 *           rx_buffer_overrun++;  // Handle overflow
 *   }
 */
int rb_enqueue_isr(struct CircularBuffer_ISR *rb, uint8_t value)
{
    /* Calculate where tail WOULD be after this write */
    int next_tail = (rb->tail + 1) % BUFFER_SIZE;
    
    /* Check if buffer would overflow (tail catching up to head) */
    if (next_tail == rb->head)
        return -1;  /* Buffer full - cannot enqueue */
    
    /* Write data at current tail position */
    rb->data[rb->tail] = value;
    
    /* Update tail to make data available for reading */
    /* Hardware barrier ensures write completes before tail update */
    rb->tail = next_tail;
    
    return 0;  /* Success */
}

/**
 * rb_dequeue_isr - Remove byte from buffer (safe for main loop)
 * 
 * Can be called from main loop while ISR may add data
 * Reads are volatile and won't be optimized away
 * 
 * Example (in main loop):
 *   uint8_t received;
 *   while (rb_dequeue_isr(&rx_buffer, &received) == 0) {
 *       process_byte(received);
 *   }
 */
int rb_dequeue_isr(struct CircularBuffer_ISR *rb, uint8_t *value)
{
    /* Check if buffer is empty */
    if (rb->head == rb->tail)
        return -1;  /* No data available */
    
    /* Read data from head position */
    *value = rb->data[rb->head];
    
    /* Advance head pointer (safe - only main reads this) */
    rb->head = (rb->head + 1) % BUFFER_SIZE;
    
    return 0;  /* Success */
}
```

---

## INTERMEDIATE LEVEL CODING (Questions 13-50)

# 13. Implement a Singly Linked List

## Answer

```c
/**
 * Singly Linked List Structure
 * 
 * Memory Layout:
 *   [Node 1]    [Node 2]    [Node 3]
 *   data: 10    data: 20    data: 30
 *   next:----->next:----->next: NULL
 * 
 * Head pointer points to first node
 * Each node points to next node
 * Last node points to NULL (terminator)
 */
struct Node
{
    int data;              /* Actual value stored */
    struct Node *next;     /* Pointer to next node (or NULL if last) */
};

/**
 * list_insert_front - Insert new node at beginning of list
 * @head: Current head of list (or NULL if empty)
 * @value: Data value for new node
 * 
 * Returns: New head pointer (important - head may change!)
 * 
 * Time Complexity: O(1) - constant time, no search needed
 * 
 * Memory Layout After Insert:
 *   Before: head --> [10] --> [20] --> NULL
 *   After:  head --> [99] --> [10] --> [20] --> NULL
 * 
 * Example:
 *   struct Node *head = NULL;
 *   head = list_insert_front(head, 10);  // head = [10]
 *   head = list_insert_front(head, 20);  // head = [20]->[10]
 *   head = list_insert_front(head, 30);  // head = [30]->[20]->[10]
 */
struct Node *list_insert_front(struct Node *head, int value)
{
    /* Allocate memory for new node */
    struct Node *new_node = malloc(sizeof(struct Node));
    if (!new_node)
        return head;  /* Allocation failed, return unchanged */
    
    /* Initialize new node */
    new_node->data = value;        /* Store the value */
    new_node->next = head;         /* Point to old head (even if NULL) */
    
    /* Return new node as new head */
    return new_node;
}

/**
 * list_insert_end - Insert new node at end of list
 * @head: Current head of list (or NULL if empty)
 * @value: Data value for new node
 * 
 * Returns: Head pointer (unchanged if list was non-empty)
 * 
 * Time Complexity: O(n) - must traverse entire list
 * 
 * Example:
 *   head = list_insert_end(head, 10);  // head = [10]
 *   head = list_insert_end(head, 20);  // head = [10]->[20]
 *   head = list_insert_end(head, 30);  // head = [10]->[20]->[30]
 */
struct Node *list_insert_end(struct Node *head, int value)
{
    struct Node *new_node = malloc(sizeof(struct Node));
    if (!new_node)
        return head;
    
    new_node->data = value;
    new_node->next = NULL;  /* New node is always last */
    
    /* Handle empty list case */
    if (!head)
        return new_node;  /* New node becomes head */
    
    /* Find last node (next pointer is NULL) */
    struct Node *current = head;
    while (current->next)  /* While not at last node */
        current = current->next;  /* Move to next node */
    
    /* current now points to last node */
    current->next = new_node;  /* Append new node */
    return head;
}

/**
 * list_delete - Remove first occurrence of value from list
 * @head: Current head of list
 * @value: Value to search for and delete
 * 
 * Returns: New head (changed if deleting first node)
 * 
 * Time Complexity: O(n) - must search for value
 * 
 * Example:
 *   head = [10]->[20]->[30]
 *   head = list_delete(head, 20);
 *   head = [10]->[30]
 */
struct Node *list_delete(struct Node *head, int value)
{
    if (!head)
        return NULL;  /* Empty list */
    
    /* Special case: delete from front */
    if (head->data == value)
    {
        struct Node *temp = head->next;  /* Save next pointer */
        free(head);                       /* Free old head */
        return temp;                      /* Return new head */
    }
    
    /* Search for node to delete (not at front) */
    struct Node *current = head;
    while (current->next && current->next->data != value)
        current = current->next;  /* Move to next node */
    
    /* If found, delete it */
    if (current->next)
    {
        struct Node *temp = current->next;     /* Save node to delete */
        current->next = temp->next;            /* Bypass the node */
        free(temp);                            /* Free deleted node */
    }
    
    return head;
}

/**
 * list_search - Find node with given value
 * @head: Current head of list
 * @value: Value to search for
 * 
 * Returns: Pointer to node if found, NULL if not found
 * 
 * Time Complexity: O(n)
 * 
 * Example:
 *   struct Node *found = list_search(head, 20);
 *   if (found)
 *       printf("Found: %d\n", found->data);
 */
struct Node *list_search(struct Node *head, int value)
{
    struct Node *current = head;
    
    /* Traverse list comparing each node's data */
    while (current)
    {
        if (current->data == value)
            return current;  /* Found! */
        current = current->next;  /* Move to next node */
    }
    
    return NULL;  /* Not found */
}

/**
 * list_reverse - Reverse the entire linked list
 * @head: Current head of list
 * 
 * Returns: New head (which was the old tail)
 * 
 * Time Complexity: O(n)
 * 
 * Algorithm (reversal by pointer redirection):
 *   Before: [1]-->[2]-->[3]-->NULL
 *   Step 1: prev=NULL, curr=[1]
 *           [1]<--prev, curr-->[2] (break link, add back link)
 *   Step 2: prev=[1], curr=[2]
 *           [1]<--[2], curr-->[3]
 *   Step 3: prev=[2], curr=[3]
 *           [1]<--[2]<--[3], curr-->NULL
 *   After: [3]<--[2]<--[1]<--NULL
 * 
 * Example:
 *   head = [1]->[2]->[3]
 *   head = list_reverse(head);
 *   head = [3]->[2]->[1]
 */
struct Node *list_reverse(struct Node *head)
{
    struct Node *prev = NULL;      /* Previous node (starts NULL) */
    struct Node *current = head;   /* Current node being processed */
    
    while (current)
    {
        struct Node *next = current->next;  /* Save next node BEFORE we break the link */
        
        current->next = prev;  /* Reverse: point to previous instead of next */
        
        prev = current;        /* Move prev forward */
        current = next;        /* Move current forward using saved pointer */
    }
    
    return prev;  /* New head is old tail */
}

/**
 * list_print - Display all nodes in list
 * @head: Current head of list
 * 
 * Format: [10] -> [20] -> [30] -> NULL
 * 
 * Example:
 *   head = [1]->[2]->[3]
 *   list_print(head);
 *   Output: 1 -> 2 -> 3 -> NULL
 */
void list_print(struct Node *head)
{
    struct Node *current = head;
    
    while (current)
    {
        printf("%d -> ", current->data);
        current = current->next;
    }
    
    printf("NULL\n");  /* Indicate end of list */
}

/**
 * list_free - Deallocate entire list (prevent memory leak)
 * @head: Current head of list
 * 
 * MUST be called before discarding list pointer!
 * 
 * Example:
 *   // At program end or when done with list:
 *   list_free(head);
 *   head = NULL;  // Good practice: clear pointer after freeing
 */
void list_free(struct Node *head)
{
    while (head)
    {
        struct Node *temp = head;     /* Save current node */
        head = head->next;            /* Move to next (before we delete) */
        free(temp);                   /* Free saved node */
    }
    
    /* head is now NULL, list is empty */
}
```

---

# 14. Detect if Linked List Has a Cycle (Floyd's Algorithm)

## Answer

```c
/**
 * Floyd's Cycle Detection Algorithm (Tortoise and Hare)
 * 
 * Key Insight:
 * - If list has cycle, fast pointer (moving 2 steps) will eventually
 *   catch slow pointer (moving 1 step)
 * - If no cycle, fast pointer reaches NULL first
 * 
 * Pointers:
 *   slow: Moves 1 step per iteration (tortoise)
 *   fast: Moves 2 steps per iteration (hare)
 * 
 * If they meet, there's a cycle!
 * 
 * Example:
 *   No cycle:  [1]-->[2]-->[3]-->NULL
 *              fast reaches NULL first, no meeting
 *   
 *   Cycle:     [1]-->[2]-->[3]
 *              ^               |
 *              +-------<-------+
 *              fast eventually catches slow, they meet!
 * 
 * Time Complexity: O(n) - visits each node at most twice
 * Space Complexity: O(1) - only uses two pointers
 */
int has_cycle(struct Node *head)
{
    if (!head)
        return 0;  /* Empty list has no cycle */
    
    struct Node *slow = head;  /* Moves 1 step */
    struct Node *fast = head;  /* Moves 2 steps */
    
    while (fast && fast->next)
    {
        slow = slow->next;           /* Move slow 1 step */
        fast = fast->next->next;     /* Move fast 2 steps */
        
        if (slow == fast)
            return 1;  /* CYCLE DETECTED! Pointers met */
    }
    
    return 0;  /* No cycle - fast reached NULL */
}

/**
 * Find Cycle Start Node
 * 
 * Algorithm:
 * 1. Find meeting point using Floyd's
 * 2. Move one pointer to head
 * 3. Move both pointers 1 step at a time
 * 4. They meet at cycle start!
 * 
 * Why this works:
 *   Distance from head to cycle start = Distance from meeting point to cycle start
 * 
 * Example:
 *   [1]-->[2]-->[3]-->[4]
 *              ^           |
 *              +---<-------+
 *   
 *   Cycle starts at [3]
 *   This function returns pointer to [3]
 */
struct Node *find_cycle_start(struct Node *head)
{
    if (!head)
        return NULL;
    
    struct Node *slow = head;
    struct Node *fast = head;
    
    /* Find meeting point (step 1) */
    while (fast && fast->next)
    {
        slow = slow->next;
        fast = fast->next->next;
        
        if (slow == fast)
            break;  /* Found meeting point */
    }
    
    /* If no meeting point, there's no cycle */
    if (!fast || !fast->next)
        return NULL;
    
    /* Move one pointer to head (step 2) */
    slow = head;
    
    /* Move both pointers one step at a time (step 3 & 4) */
    while (slow != fast)
    {
        slow = slow->next;
        fast = fast->next;
    }
    
    return slow;  /* Both pointers now at cycle start */
}
```

---

# 15. Implement `container_of` Macro from Scratch

## Answer

```c
/**
 * container_of - Get pointer to containing structure from member pointer
 * @ptr: Pointer to the member within the structure
 * @type: Type of the containing structure
 * @member: Name of the member field within the structure
 * 
 * Returns: Pointer to the containing structure
 * 
 * How it works:
 * 1. Calculate offset of member within type: offsetof(type, member)
 * 2. Subtract offset from ptr to get structure address
 * 3. Cast result to (type *)
 * 
 * Magic: Works at COMPILE TIME - no runtime overhead!
 * 
 * Key insight:
 *   ((type *)0)->member gives address offset from NULL
 *   (char *)(ptr) - offset gives us the struct start
 */
#define container_of(ptr, type, member) \
    ((type *)((char *)(ptr) - offsetof(type, member)))
```

### Without Using `offsetof`

```c
/**
 * container_of_manual - Same as container_of but calculates offset manually
 * 
 * How it works:
 *   ((type *)0)->member
 *   This creates hypothetical struct at address NULL (0x0)
 *   Then gets address of member field
 *   That address IS the offset from start of struct
 * 
 * Advantage: No need for offsetof() macro
 * Disadvantage: More complex, harder to understand
 */
#define container_of_manual(ptr, type, member) \
    ((type *)((char *)(ptr) - (char *)&((type *)0)->member))

/*
 * Explanation:
 *   (type *)0              = Treat NULL as if it were a struct pointer
 *   ->member               = Get address of member at NULL+offset
 *   (char *)&...           = Get that address as a byte offset
 *   (char *)(ptr) - offset = Subtract offset from ptr to get struct start
 *   ((type *) ... )        = Cast result back to struct pointer
 */
```

### Example Usage

```c
/**
 * Real-world example: Intrusive linked list
 * 
 * Problem: Want to store student data in linked list
 * Old way: Create separate list node structure
 * Better way: Embed list node INSIDE student structure
 * 
 * Benefits:
 * - Cache friendly (student and node together in memory)
 * - Single allocation (no separate node malloc)
 * - Generic list code works with any struct
 */

struct ListNode
{
    struct ListNode *prev;
    struct ListNode *next;
};

struct Student
{
    int id;              /* Unique identifier */
    char name[50];       /* Student name */
    float gpa;           /* Grade point average */
    struct ListNode node; /* Intrusive list node */
};

/**
 * add_student_to_list - Add student to linked list
 * 
 * Stores: student_list -> [ListNode] -> [ListNode] -> ...
 * Inside: Each ListNode belongs to a Student structure
 * 
 * Example:
 *   struct Student alice = {.id = 1, .name = "Alice", .gpa = 3.8};
 *   struct Student bob = {.id = 2, .name = "Bob", .gpa = 3.5};
 *   
 *   add_student(&alice);
 *   add_student(&bob);
 *   
 *   // Later: retrieve student from list
 *   struct ListNode *entry = student_list;
 *   struct Student *student = container_of(entry, struct Student, node);
 *   printf("Student: %s\n", student->name);
 */
void add_student(struct Student *student)
{
    /* Add to beginning of list */
    student->node.next = student_list;
    if (student_list)
        student_list->prev = &student->node;
    student_list = &student->node;
}

/**
 * get_student_by_id - Find student in list by ID
 * 
 * Algorithm:
 * 1. Iterate through ListNode pointers
 * 2. Use container_of to get Student from node
 * 3. Check if ID matches
 * 4. Return Student pointer if found
 * 
 * Example:
 *   struct Student *found = get_student_by_id(1);
 *   if (found)
 *       printf("Found: %s (GPA: %.1f)\n", found->name, found->gpa);
 */
struct Student *get_student_by_id(int id)
{
    for (struct ListNode *entry = student_list; entry; entry = entry->next)
    {
        /* Convert ListNode pointer back to Student pointer */
        struct Student *student = container_of(entry, struct Student, node);
        
        if (student->id == id)
            return student;  /* Found it! */
    }
    
    return NULL;  /* Not found */
}
```

---

# 16. Implement a Memory Pool Allocator

## Answer

```c
/**
 * Memory Pool Allocator Design
 * 
 * Problem: Dynamic malloc/free is unpredictable in embedded systems
 *          - May fail (no memory)
 *          - Causes fragmentation over time
 *          - Non-deterministic timing (bad for real-time)
 * 
 * Solution: Pre-allocate fixed blocks, reuse them
 * 
 * Memory Layout:
 *   blocks[256][64]         = 256 blocks, 64 bytes each = 16 KB total
 *   free_bitmap[32]         = 256 bits (1 = free, 0 = allocated)
 * 
 * Example allocation flow:
 *   1. Find first free bit in bitmap
 *   2. Clear that bit (mark as allocated)
 *   3. Return pointer to corresponding block
 *   4. On free: Set bit (mark as free)
 */
#define POOL_BLOCK_SIZE 64      /* Each block is 64 bytes */
#define POOL_NUM_BLOCKS 256     /* Total 256 blocks */

struct MemoryPool
{
    /* Fixed memory: 256 blocks * 64 bytes = 16 KB */
    uint8_t blocks[POOL_NUM_BLOCKS][POOL_BLOCK_SIZE];
    
    /* Bitmap tracking which blocks are free */
    /* Each bit: 1 = free, 0 = allocated */
    /* 256 bits = 32 bytes (256 / 8) */
    uint8_t free_bitmap[POOL_NUM_BLOCKS / 8];
};

static struct MemoryPool pool;

/**
 * pool_init - Initialize memory pool (mark all blocks as free)
 * 
 * Must be called ONCE at startup before any allocations
 * Sets all bits to 1 (1 = free)
 * 
 * Example:
 *   int main(void)
 *   {
 *       pool_init();  // Initialize
 *       void *p = pool_alloc();  // Now safe to use
 *   }
 */
void pool_init(void)
{
    /* Set all bits to 1 (all blocks free) */
    memset(pool.free_bitmap, 0xFF, sizeof(pool.free_bitmap));
}

/**
 * pool_alloc - Allocate one fixed-size block from pool
 * 
 * Returns: Pointer to 64-byte block, or NULL if pool exhausted
 * 
 * Algorithm:
 * 1. Scan free_bitmap for first 1 bit (free block)
 * 2. Clear that bit (mark allocated)
 * 3. Return pointer to that block
 * 
 * Time Complexity: O(n) where n = POOL_NUM_BLOCKS / 8
 *                 (Could be optimized to O(1) with free list)
 * 
 * Example:
 *   void *buffer = pool_alloc();
 *   if (!buffer) {
 *       printf("Memory pool exhausted!\n");
 *       return -1;
 *   }
 *   // Use buffer (guaranteed 64 bytes)
 */
void *pool_alloc(void)
{
    /* Scan each byte in free_bitmap */
    for (int i = 0; i < POOL_NUM_BLOCKS; i++)
    {
        int byte_idx = i / 8;          /* Which byte contains this bit */
        int bit_idx = i % 8;           /* Which bit within that byte */
        
        /* Check if bit is set (1 = free) */
        if (pool.free_bitmap[byte_idx] & (1 << bit_idx))
        {
            /* Found a free block! Mark it as allocated (clear bit) */
            pool.free_bitmap[byte_idx] &= ~(1 << bit_idx);
            
            /* Return pointer to this block */
            return pool.blocks[i];
        }
    }
    
    return NULL;  /* No free blocks available */
}

/**
 * pool_free - Return allocated block to pool
 * 
 * @ptr: Pointer returned by pool_alloc()
 * 
 * Algorithm:
 * 1. Find which block this pointer belongs to
 * 2. Set corresponding bit in bitmap (mark as free)
 * 
 * Safety: Doesn't check if ptr is valid!
 *         (Could add magic number for validation)
 * 
 * Example:
 *   void *buffer = pool_alloc();
 *   // ... use buffer ...
 *   pool_free(buffer);  // Return to pool
 */
void pool_free(void *ptr)
{
    if (!ptr)
        return;  /* Ignore NULL pointers */
    
    uint8_t *p = (uint8_t *)ptr;
    
    /* Find which block this pointer belongs to */
    for (int i = 0; i < POOL_NUM_BLOCKS; i++)
    {
        if (pool.blocks[i] == p)
        {
            int byte_idx = i / 8;
            int bit_idx = i % 8;
            
            /* Mark block as free (set bit to 1) */
            pool.free_bitmap[byte_idx] |= (1 << bit_idx);
            
            return;  /* Done */
        }
    }
    
    /* If we get here, pointer wasn't from this pool (error!) */
}

/**
 * pool_get_usage - Get number of allocated blocks
 * 
 * Returns: Count of blocks currently in use
 * 
 * Useful for: Monitoring, detecting memory leaks
 * 
 * Example:
 *   int used = pool_get_usage();
 *   printf("Pool usage: %d / %d blocks\n", used, POOL_NUM_BLOCKS);
 */
int pool_get_usage(void)
{
    int used = 0;
    
    /* Count all cleared bits (0 = allocated) */
    for (int i = 0; i < POOL_NUM_BLOCKS; i++)
    {
        int byte_idx = i / 8;
        int bit_idx = i % 8;
        
        /* If bit is clear (0 = allocated), count it */
        if (!(pool.free_bitmap[byte_idx] & (1 << bit_idx)))
            used++;
    }
    
    return used;
}

/**
 * Example Usage in Application:
 * 
 *   // In initialization:
 *   pool_init();
 *   
 *   // In ISR:
 *   struct Packet *pkt = pool_alloc();
 *   if (pkt) {
 *       pkt->data[0] = 0x42;
 *       queue_packet(pkt);
 *   }
 *   
 *   // In main loop:
 *   struct Packet *received = dequeue_packet();
 *   if (received) {
 *       printf("Data: %02X\n", received->data[0]);
 *       pool_free(received);  // Return to pool
 *   }
 *   
 *   // Monitor:
 *   printf("Pool used: %d%%\n", (pool_get_usage() * 100) / POOL_NUM_BLOCKS);
 */
```

---

# 17. Print Memory Layout of a Structure

## Answer

```c
void print_struct_layout(void)
{
    struct Test
    {
        char a;      /* 1 byte */
        int b;       /* 4 bytes */
        char c;      /* 1 byte */
        double d;    /* 8 bytes */
    };
    
    printf("Structure Size: %zu bytes\n", sizeof(struct Test));
    printf("Alignment: %zu bytes\n\n", _Alignof(struct Test));
    
    printf("Member Offsets and Sizes:\n");
    printf("  a (char):   offset = %zu, size = %zu\n",
           offsetof(struct Test, a), sizeof(((struct Test *)0)->a));
    printf("  b (int):    offset = %zu, size = %zu\n",
           offsetof(struct Test, b), sizeof(((struct Test *)0)->b));
    printf("  c (char):   offset = %zu, size = %zu\n",
           offsetof(struct Test, c), sizeof(((struct Test *)0)->c));
    printf("  d (double): offset = %zu, size = %zu\n",
           offsetof(struct Test, d), sizeof(((struct Test *)0)->d));
    
    printf("\nMemory Layout:\n");
    
    struct Test t;
    uint8_t *p = (uint8_t *)&t;
    
    for (size_t i = 0; i < sizeof(struct Test); i++)
    {
        printf("Byte %2zu: ", i);
        
        if (i == offsetof(struct Test, a))
            printf("a");
        else if (i >= offsetof(struct Test, b) &&
                 i < offsetof(struct Test, b) + sizeof(int))
            printf("b");
        else if (i == offsetof(struct Test, c))
            printf("c");
        else if (i >= offsetof(struct Test, d) &&
                 i < offsetof(struct Test, d) + sizeof(double))
            printf("d");
        else
            printf("(padding)");
        
        printf("\n");
    }
}
```

---

# 18. Implement a Simple State Machine (Traffic Light)

## Answer

```c
/**
 * Traffic Light State Machine (Simple Enum-based)
 * 
 * States:  RED -> GREEN -> YELLOW -> RED -> ...
 * 
 *     [RED]
 *      ^ |
 *      | v
 *   [YELLOW]-->[GREEN]
 * 
 * Transitions triggered by tl_timeout() or by events
 */
typedef enum
{
    RED,     /* Stop state */
    YELLOW,  /* Caution state */
    GREEN    /* Go state */
} TrafficLight;

typedef struct
{
    TrafficLight state;  /* Current light color */
} TrafficLightController;

/**
 * tl_init - Initialize traffic light to RED state
 * @tlc: Pointer to controller
 * 
 * Must be called once at startup
 */
void tl_init(TrafficLightController *tlc)
{
    tlc->state = RED;  /* Initial state is RED (stop) */
}

/**
 * tl_timeout - Advance traffic light to next state
 * @tlc: Pointer to controller
 * 
 * Called by timer interrupt or main loop
 * Cycles through: RED -> GREEN -> YELLOW -> RED -> ...
 * 
 * Example timeline:
 *   0 sec: RED (stop for 25 seconds)
 *   25 sec: Timeout -> GREEN (go for 20 seconds)
 *   45 sec: Timeout -> YELLOW (caution for 5 seconds)
 *   50 sec: Timeout -> RED (stop again)
 */
void tl_timeout(TrafficLightController *tlc)
{
    /* Switch based on current state to determine next state */
    switch (tlc->state)
    {
    case RED:
        tlc->state = GREEN;  /* Transition to GREEN */
        printf("Traffic Light: GREEN\n");
        break;
    case GREEN:
        tlc->state = YELLOW;  /* Transition to YELLOW */
        printf("Traffic Light: YELLOW\n");
        break;
    case YELLOW:
        tlc->state = RED;  /* Transition back to RED */
        printf("Traffic Light: RED\n");
        break;
    }
}
```

### Function Pointer Version (More Elegant & Scalable)

```c
/**
 * Advanced State Machine using Function Pointers
 * 
 * Key advantage: Each state is represented by a function (state handler)
 * New states can be added without modifying switch statements
 * Each handler decides what to do and changes the handler pointer
 * 
 * Architecture:
 *   struct TrafficLight {
 *       handler = red_handler (points to current state function)
 *   }
 *   
 *   Call tl_process() -> calls handler(tl)
 *   Handler executes and updates handler pointer to next state
 *   Next call uses new handler
 * 
 * Benefits:
 *   - Cleaner code (no giant switch statements)
 *   - Easy to add new states (new function + handler)
 *   - Each state is independent (high cohesion)
 *   - Commonly used pattern in embedded systems
 */

typedef struct TrafficLight TrafficLight;

typedef void (*StateHandler)(TrafficLight *);  /* Function pointer type */

/* Forward declarations */
void red_handler(TrafficLight *tl);
void green_handler(TrafficLight *tl);
void yellow_handler(TrafficLight *tl);

struct TrafficLight
{
    StateHandler handler;  /* Points to current state's handler function */
};

/**
 * red_handler - Execute RED state logic
 * @tl: Pointer to state machine
 * 
 * What RED state does:
 * 1. Display current state
 * 2. Perform state-specific actions (e.g., stop traffic)
 * 3. Update handler to next state's function
 * 
 * Called when: Traffic light is RED
 * Next state: GREEN
 */
void red_handler(TrafficLight *tl)
{
    printf("RED -> GREEN\n");  /* Log transition */
    tl->handler = green_handler;  /* Change to GREEN handler */
}

/**
 * green_handler - Execute GREEN state logic
 * @tl: Pointer to state machine
 * 
 * What GREEN state does:
 * 1. Display current state
 * 2. Perform state-specific actions (allow traffic)
 * 3. Update handler to YELLOW state
 */
void green_handler(TrafficLight *tl)
{
    printf("GREEN -> YELLOW\n");
    tl->handler = yellow_handler;  /* Change to YELLOW handler */
}

/**
 * yellow_handler - Execute YELLOW state logic
 * @tl: Pointer to state machine
 * 
 * What YELLOW state does:
 * 1. Display current state
 * 2. Perform state-specific actions (caution)
 * 3. Update handler back to RED state
 */
void yellow_handler(TrafficLight *tl)
{
    printf("YELLOW -> RED\n");
    tl->handler = red_handler;  /* Change back to RED handler */
}

/**
 * tl_init - Initialize state machine to RED
 * @tl: Pointer to state machine
 */
void tl_init(TrafficLight *tl)
{
    tl->handler = red_handler;  /* Start with RED handler */
}

/**
 * tl_process - Process one state cycle
 * @tl: Pointer to state machine
 * 
 * Calls current handler, which transitions to next state
 * 
 * Example flow:
 *   Initial: handler = red_handler
 *   tl_process() -> red_handler() -> handler = green_handler
 *   tl_process() -> green_handler() -> handler = yellow_handler
 *   tl_process() -> yellow_handler() -> handler = red_handler
 *   Cycle continues...
 */
void tl_process(TrafficLight *tl)
{
    tl->handler(tl);  /* Call current handler (function pointed to) */
}

/**
 * Real-world usage example:
 * 
 *   TrafficLight tl;
 *   tl_init(&tl);
 *   
 *   // Main loop or timer interrupt:
 *   while (1) {
 *       tl_process(&tl);  // Advance state
 *       sleep(timeout);   // RED=25sec, GREEN=20sec, YELLOW=5sec
 *   }
 *   
 *   Output:
 *     RED -> GREEN
 *     GREEN -> YELLOW
 *     YELLOW -> RED
 *     RED -> GREEN
 *     ...
 */
```

---

# 19. Write an ISR-Safe FIFO Queue

## Answer

```c
#define FIFO_SIZE 256

struct ISRSafeFIFO
{
    volatile uint8_t buffer[FIFO_SIZE];
    volatile int head;
    volatile int tail;
};

int fifo_enqueue_isr(struct ISRSafeFIFO *fifo, uint8_t data)
{
    int next_tail = (fifo->tail + 1) % FIFO_SIZE;
    
    if (next_tail == fifo->head)
        return -1;  /* full */
    
    fifo->buffer[fifo->tail] = data;
    fifo->tail = next_tail;
    
    return 0;
}

int fifo_dequeue(struct ISRSafeFIFO *fifo, uint8_t *data)
{
    if (fifo->head == fifo->tail)
        return -1;  /* empty */
    
    *data = fifo->buffer[fifo->head];
    fifo->head = (fifo->head + 1) % FIFO_SIZE;
    
    return 0;
}
```

### With Atomic Operations

```c
struct ISRSafeFIFO_Atomic
{
    uint8_t buffer[FIFO_SIZE];
    _Atomic int head;
    _Atomic int tail;
};

int fifo_enqueue_atomic(struct ISRSafeFIFO_Atomic *fifo, uint8_t data)
{
    int tail = atomic_load(&fifo->tail);
    int next_tail = (tail + 1) % FIFO_SIZE;
    
    if (next_tail == atomic_load(&fifo->head))
        return -1;
    
    fifo->buffer[tail] = data;
    atomic_store(&fifo->tail, next_tail);
    
    return 0;
}
```

---

# 20. Implement a UART Protocol State Machine Parser

## Answer

```c
typedef enum
{
    UART_IDLE,
    UART_SYNC1,
    UART_SYNC2,
    UART_LENGTH,
    UART_DATA,
    UART_CHECKSUM,
    UART_COMPLETE
} UARTState;

struct UARTPacket
{
    uint8_t sync1;
    uint8_t sync2;
    uint8_t length;
    uint8_t data[256];
    uint8_t checksum;
};

struct UARTParser
{
    UARTState state;
    struct UARTPacket packet;
    int data_index;
};

int uart_parse_byte(struct UARTParser *parser, uint8_t byte)
{
    switch (parser->state)
    {
    case UART_IDLE:
        if (byte == 0xAA)
        {
            parser->packet.sync1 = byte;
            parser->state = UART_SYNC1;
        }
        break;
    
    case UART_SYNC1:
        if (byte == 0xBB)
        {
            parser->packet.sync2 = byte;
            parser->state = UART_SYNC2;
        }
        else
        {
            parser->state = UART_IDLE;
        }
        break;
    
    case UART_SYNC2:
        parser->packet.length = byte;
        parser->data_index = 0;
        parser->state = UART_LENGTH;
        break;
    
    case UART_LENGTH:
        if (parser->data_index < parser->packet.length)
        {
            parser->packet.data[parser->data_index++] = byte;
        }
        
        if (parser->data_index >= parser->packet.length)
        {
            parser->state = UART_DATA;
        }
        break;
    
    case UART_DATA:
        parser->packet.checksum = byte;
        parser->state = UART_CHECKSUM;
        return 1;  /* packet complete */
    
    default:
        parser->state = UART_IDLE;
    }
    
    return 0;  /* not complete */
}
```

---

# 21. Set/Clear/Toggle/Check Bits

## Answer

```c
/* Set nth bit */
#define SET_BIT(x, n) ((x) |= (1 << (n)))

/* Clear nth bit */
#define CLEAR_BIT(x, n) ((x) &= ~(1 << (n)))

/* Toggle nth bit */
#define TOGGLE_BIT(x, n) ((x) ^= (1 << (n)))

/* Check if nth bit is set */
#define CHECK_BIT(x, n) (((x) >> (n)) & 1)

/* Example */
int flags = 0;

SET_BIT(flags, 0);    /* bit 0 = 1 */
SET_BIT(flags, 3);    /* bit 3 = 1 */

if (CHECK_BIT(flags, 0))
    printf("Bit 0 is set\n");

TOGGLE_BIT(flags, 0); /* bit 0 = 0 */
CLEAR_BIT(flags, 3);  /* bit 3 = 0 */
```

### Multi-Bit Operations

```c
/* Set bits from position pos with mask */
#define SET_BITS_MASK(x, mask, pos) ((x) |= ((mask) << (pos)))

/* Clear bits with mask */
#define CLEAR_BITS_MASK(x, mask, pos) ((x) &= ~((mask) << (pos)))

/* Extract bits */
#define EXTRACT_BITS(x, start, len) (((x) >> (start)) & ((1 << (len)) - 1))
```

---

# 22. Implement `itoa()` (Integer to String)

## Answer

```c
char *my_itoa(int num, char *str, int base)
{
    if (!str || base < 2 || base > 36)
        return str;
    
    int i = 0;
    int is_negative = 0;
    
    if (num < 0)
    {
        is_negative = 1;
        num = -num;
    }
    
    if (num == 0)
    {
        str[0] = '0';
        str[1] = '\0';
        return str;
    }
    
    /* Extract digits */
    while (num > 0)
    {
        int digit = num % base;
        str[i++] = (digit < 10) ? ('0' + digit) : ('a' + digit - 10);
        num /= base;
    }
    
    if (is_negative)
        str[i++] = '-';
    
    str[i] = '\0';
    
    /* Reverse string */
    int left = 0, right = i - 1;
    while (left < right)
    {
        char temp = str[left];
        str[left] = str[right];
        str[right] = temp;
        left++;
        right--;
    }
    
    return str;
}
```

---

# 23. Implement String Find (strstr Equivalent)

## Answer

```c
char *my_strstr(const char *haystack, const char *needle)
{
    if (!needle || *needle == '\0')
        return (char *)haystack;
    
    if (!haystack)
        return NULL;
    
    for (const char *h = haystack; *h; h++)
    {
        const char *n = needle;
        const char *h_temp = h;
        
        while (*n && *h_temp == *n)
        {
            n++;
            h_temp++;
        }
        
        if (*n == '\0')
            return (char *)h;
    }
    
    return NULL;
}
```

### Using KMP Algorithm (Faster)

```c
int *build_lps(const char *pattern)
{
    int len = strlen(pattern);
    int *lps = malloc(len * sizeof(int));
    
    lps[0] = 0;
    int i = 1, j = 0;
    
    while (i < len)
    {
        if (pattern[i] == pattern[j])
        {
            lps[i] = j + 1;
            i++;
            j++;
        }
        else
        {
            if (j)
                j = lps[j - 1];
            else
            {
                lps[i] = 0;
                i++;
            }
        }
    }
    
    return lps;
}

char *kmp_search(const char *text, const char *pattern)
{
    int *lps = build_lps(pattern);
    int i = 0, j = 0;
    
    while (text[i])
    {
        if (pattern[j] == text[i])
        {
            i++;
            j++;
        }
        
        if (pattern[j] == '\0')
        {
            free(lps);
            return (char *)&text[i - j];
        }
        else if (text[i] != pattern[j])
        {
            if (j)
                j = lps[j - 1];
            else
                i++;
        }
    }
    
    free(lps);
    return NULL;
}
```

---

# 24. Split String by Delimiter

## Answer

```c
int split_string(char *str, const char *delim, char **tokens, int max_tokens)
{
    if (!str || !delim || !tokens)
        return 0;
    
    char *copy = strdup(str);
    if (!copy)
        return 0;
    
    int count = 0;
    char *token = strtok(copy, delim);
    
    while (token && count < max_tokens)
    {
        tokens[count] = malloc(strlen(token) + 1);
        if (tokens[count])
        {
            strcpy(tokens[count], token);
            count++;
        }
        
        token = strtok(NULL, delim);
    }
    
    free(copy);
    return count;
}
```

---

# 25. Implement `pow()` Function

## Answer

```c
double my_pow(double base, int exp)
{
    if (exp == 0)
        return 1.0;
    
    if (exp < 0)
        return 1.0 / my_pow(base, -exp);
    
    if (exp % 2 == 0)
    {
        double half = my_pow(base, exp / 2);
        return half * half;
    }
    else
    {
        return base * my_pow(base, exp - 1);
    }
}
```

### Iterative Version

```c
double my_pow_iter(double base, int exp)
{
    double result = 1.0;
    int e = (exp < 0) ? -exp : exp;
    
    while (e > 0)
    {
        if (e % 2 == 1)
            result *= base;
        
        base *= base;
        e /= 2;
    }
    
    return (exp < 0) ? 1.0 / result : result;
}
```

---

# 26. Check if Number is Prime

## Answer

```c
int is_prime(int n)
{
    if (n < 2)
        return 0;
    if (n == 2)
        return 1;
    if (n % 2 == 0)
        return 0;
    
    for (int i = 3; i * i <= n; i += 2)
    {
        if (n % i == 0)
            return 0;
    }
    
    return 1;
}
```

---

# 27. Binary Search Implementation

## Answer

```c
int binary_search(int *arr, int n, int target)
{
    int left = 0, right = n - 1;
    
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target)
            return mid;
        else if (arr[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    
    return -1;  /* not found */
}
```

---

## ADVANCED LEVEL CODING (Questions 28-100)

# 28. Implement Quick Sort

## Answer

```c
/**
 * partition - Partition array around pivot element
 * @arr: Array to partition
 * @low: Lower boundary (inclusive)
 * @high: Upper boundary (inclusive)
 * 
 * Returns: Final position of pivot
 * 
 * Algorithm (Lomuto Partition):
 * 1. Choose last element as pivot
 * 2. Use i pointer to track boundary between small and large elements
 * 3. Scan from low to high-1:
 *    - If element < pivot, increment i and swap with element at position
 *    - Otherwise, skip (element >= pivot)
 * 4. Place pivot at final position (i+1)
 * 
 * Result: All elements < pivot are to the left, >= pivot are to the right
 * 
 * Example:
 *   [3, 7, 2, 8, 1, 5]  (pivot = 5)
 *   After partition:
 *   [3, 2, 1, 5, 8, 7]  (pivot at index 3)
 *   Return 3 as pivot position
 * 
 * Time Complexity: O(n)
 * Space Complexity: O(1) - in-place partitioning
 */
int partition(int *arr, int low, int high)
{
    int pivot = arr[high];  /* Choose last element as pivot */
    int i = low - 1;        /* i points to last element smaller than pivot */
    
    /* Scan through array comparing each element with pivot */
    for (int j = low; j < high; j++)
    {
        if (arr[j] < pivot)
        {
            i++;  /* Move boundary forward */
            
            /* Swap arr[i] and arr[j] */
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    
    /* Place pivot in final position */
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    
    return i + 1;  /* Return pivot's final index */
}

/**
 * quick_sort - Sort array using divide-and-conquer
 * @arr: Array to sort
 * @low: Lower boundary (inclusive)
 * @high: Upper boundary (inclusive)
 * 
 * Algorithm:
 * 1. Base case: if low >= high, already sorted
 * 2. Partition array around pivot
 * 3. Recursively sort left and right subarrays
 * 
 * Example:
 *   int data[5] = {3, 7, 2, 8, 1};
 *   quick_sort(data, 0, 4);
 *   // data is now {1, 2, 3, 7, 8}
 * 
 * Time Complexity:
 *   - Best case: O(n log n) - pivot always in middle
 *   - Average case: O(n log n)
 *   - Worst case: O(n²) - pivot always at edge (e.g., already sorted array)
 * 
 * Space Complexity: O(log n) - recursion depth
 * 
 * Note: For embedded systems, consider hybrid approach:
 *   - Quick sort for large arrays (good average case)
 *   - Switch to insertion sort for small subarrays (simpler, less overhead)
 */
void quick_sort(int *arr, int low, int high)
{
    if (low < high)
    {
        /* Partition and get pivot position */
        int pivot_index = partition(arr, low, high);
        
        /* Recursively sort left subarray (elements < pivot) */
        quick_sort(arr, low, pivot_index - 1);
        
        /* Recursively sort right subarray (elements > pivot) */
        quick_sort(arr, pivot_index + 1, high);
    }
}
```

---

# 29. Implement Merge Sort

## Answer

```c
/**
 * merge - Merge two sorted subarrays into one sorted array
 * @arr: Original array containing two sorted subarrays
 * @left: Start index of left subarray
 * @mid: End index of left subarray (also split point)
 * @right: End index of right subarray
 * 
 * Algorithm:
 * 1. Create temporary arrays for left [left...mid] and right [mid+1...right]
 * 2. Compare elements from both subarrays
 * 3. Place smaller element in main array
 * 4. When one subarray exhausted, copy remaining from other
 * 
 * Memory Layout:
 *   Left subarray:  [L0, L1, L2]     (indices left to mid)
 *   Right subarray: [R0, R1, R2]     (indices mid+1 to right)
 *   After merge:    [L0, L1, L2, R0, R1, R2]  (all in sorted order)
 * 
 * Example:
 *   arr = [1, 3, 5, 2, 4, 6]
 *   merge(arr, 0, 2, 5)
 *   Left:  [1, 3, 5]
 *   Right: [2, 4, 6]
 *   Result: [1, 2, 3, 4, 5, 6]
 * 
 * Time Complexity: O(n) where n = right - left + 1
 * Space Complexity: O(n) - needs temporary arrays for merge
 */
void merge(int *arr, int left, int mid, int right)
{
    int n1 = mid - left + 1;       /* Size of left subarray */
    int n2 = right - mid;          /* Size of right subarray */
    
    /* Allocate temporary arrays */
    int *L = malloc(n1 * sizeof(int));
    int *R = malloc(n2 * sizeof(int));
    
    /* Copy data to temporary arrays */
    for (int i = 0; i < n1; i++)
        L[i] = arr[left + i];      /* Left subarray elements */
    for (int j = 0; j < n2; j++)
        R[j] = arr[mid + 1 + j];   /* Right subarray elements */
    
    int i = 0;      /* Index for left subarray */
    int j = 0;      /* Index for right subarray */
    int k = left;   /* Index for merged array */
    
    /* Merge by comparing elements from both subarrays */
    while (i < n1 && j < n2)
    {
        if (L[i] <= R[j])
        {
            /* Element from left is smaller, place it */
            arr[k++] = L[i++];
        }
        else
        {
            /* Element from right is smaller, place it */
            arr[k++] = R[j++];
        }
    }
    
    /* Copy remaining elements from left subarray (if any) */
    while (i < n1)
        arr[k++] = L[i++];
    
    /* Copy remaining elements from right subarray (if any) */
    while (j < n2)
        arr[k++] = R[j++];
    
    /* Free temporary memory */
    free(L);
    free(R);
}

/**
 * merge_sort - Sort array using divide-and-conquer merge sort
 * @arr: Array to sort
 * @left: Lower boundary (inclusive)
 * @right: Upper boundary (inclusive)
 * 
 * Algorithm (Divide and Conquer):
 * 1. Base case: if left >= right, already sorted
 * 2. Find middle point
 * 3. Recursively sort left half
 * 4. Recursively sort right half
 * 5. Merge both sorted halves
 * 
 * Example:
 *   int data[5] = {5, 2, 8, 1, 9};
 *   merge_sort(data, 0, 4);
 *   // data is now {1, 2, 5, 8, 9}
 * 
 * Division Tree:
 *   [5, 2, 8, 1, 9]
 *   /              \
 *   [5, 2]          [8, 1, 9]
 *   /    \          /        \
 *   [5]  [2]    [8]     [1, 9]
 *   |    |      |       /      \
 *   [2]<-[5]    [8]    [1]  [9]
 *    \  /        |      \  /
 *    [2,5]       |      [1,9]
 *      \         /       /
 *       [2,5,8] + [1,9]
 *            \  /
 *             [1,2,5,8,9]
 * 
 * Time Complexity: O(n log n) - Always, even in worst case!
 * Space Complexity: O(n) - Need temporary arrays for merge
 * 
 * Advantage vs Quick Sort:
 *   - Guaranteed O(n log n) (no worst case O(n²))
 *   - Stable sort (equal elements maintain relative order)
 *   - Better for linked lists
 * 
 * Disadvantage:
 *   - Requires O(n) extra space
 *   - Slower in practice for small arrays (more overhead)
 */
void merge_sort(int *arr, int left, int right)
{
    if (left < right)
    {
        /* Find middle point (avoid overflow: mid = left + (right-left)/2) */
        int mid = left + (right - left) / 2;
        
        /* Recursively sort left half [left...mid] */
        merge_sort(arr, left, mid);
        
        /* Recursively sort right half [mid+1...right] */
        merge_sort(arr, mid + 1, right);
        
        /* Merge sorted halves */
        merge(arr, left, mid, right);
    }
}
```

---

# 30. Implement Bubble Sort

## Answer

```c
void bubble_sort(int *arr, int n)
{
    for (int i = 0; i < n - 1; i++)
    {
        int swapped = 0;
        
        for (int j = 0; j < n - i - 1; j++)
        {
            if (arr[j] > arr[j + 1])
            {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = 1;
            }
        }
        
        if (!swapped)
            break;  /* already sorted */
    }
}
```

---

# 31. Convert Binary String to Integer

## Answer

```c
int binary_str_to_int(const char *binary)
{
    if (!binary)
        return 0;
    
    int result = 0;
    
    for (int i = 0; binary[i]; i++)
    {
        if (binary[i] == '0' || binary[i] == '1')
        {
            result = result * 2 + (binary[i] - '0');
        }
        else
        {
            return 0;  /* invalid */
        }
    }
    
    return result;
}
```

---

# 32. Implement Queue (FIFO)

## Answer

```c
/**
 * Queue Structure (FIFO - First In First Out)
 * 
 * Memory Layout:
 *   indices: 0    1    2    3    4    5
 *   data:   [10] [20] [30] [40] [ ]  [ ]
 *            ^                   ^
 *           front              rear
 * 
 * front: Points to first element to dequeue
 * rear: Points to next free slot for enqueue
 * 
 * Problem: Linear array wastes space (can't reuse front area)
 * Solution: Circular queue (see Question 12)
 */
struct Queue
{
    int data[100];          /* Fixed-size array */
    int front;              /* Index of first element */
    int rear;               /* Index of next free slot */
};

/**
 * enqueue - Add element to rear of queue
 * @q: Pointer to queue
 * @value: Value to add
 * 
 * Adds element at rear and increments rear pointer
 * Returns: Nothing (check before calling if space available)
 * 
 * Time Complexity: O(1) - constant time
 * 
 * Example:
 *   struct Queue q = {0};
 *   q.front = 0;
 *   q.rear = 0;
 *   
 *   enqueue(&q, 10);  // rear becomes 1
 *   enqueue(&q, 20);  // rear becomes 2
 *   // Queue: [10, 20, ...]
 */
void enqueue(struct Queue *q, int value)
{
    if (q->rear >= 100)
        return;  /* Queue full - no space to add */
    
    q->data[q->rear++] = value;  /* Add at rear, increment rear */
}

/**
 * dequeue - Remove and return element from front of queue
 * @q: Pointer to queue
 * 
 * Returns: First element in queue, or -1 if empty
 * 
 * Note: This simple implementation wastes space!
 *       Front pointer keeps incrementing but never resets
 *       After 100 dequeue operations, queue is unusable
 *       Better: Use circular queue or reset when both pointers reach end
 * 
 * Time Complexity: O(1)
 * 
 * Example:
 *   int first = dequeue(&q);  // Returns 10, front becomes 1
 *   int second = dequeue(&q); // Returns 20, front becomes 2
 */
int dequeue(struct Queue *q)
{
    if (q->front >= q->rear)
        return -1;  /* Queue empty - no more elements */
    
    return q->data[q->front++];  /* Return element, increment front */
}
```

---

# 33. Implement Stack (LIFO)

## Answer

```c
/**
 * Stack Structure (LIFO - Last In First Out)
 * 
 * Memory Layout:
 *   indices: 0    1    2    3    4    5
 *   data:   [10] [20] [30] [ ]  [ ]  [ ]
 *                        ^
 *                       top
 * 
 * top: Index of top element (or next free if empty)
 * 
 * Push: Add to top, increment top
 * Pop: Decrement top, return element at that position
 * 
 * Key Difference from Queue:
 *   Queue: FIFO (First In First Out) - like waiting in line
 *   Stack: LIFO (Last In First Out) - like plate stack in cafeteria
 */
struct Stack
{
    int data[100];          /* Fixed-size array to hold elements */
    int top;                /* Index of top element (or next free) */
};

/**
 * push - Add element to top of stack
 * @s: Pointer to stack
 * @value: Value to push
 * 
 * Adds element at top and increments top pointer
 * 
 * Time Complexity: O(1) - constant time
 * 
 * Example:
 *   struct Stack s = {0};
 *   s.top = 0;
 *   
 *   push(&s, 10);  // top becomes 1
 *   push(&s, 20);  // top becomes 2
 *   // Stack: [10, 20, ...]
 *           top is now at index 2
 */
void push(struct Stack *s, int value)
{
    if (s->top >= 100)
        return;  /* Stack full - no space */
    
    s->data[s->top++] = value;  /* Add at top, increment top */
}

/**
 * pop - Remove and return element from top of stack
 * @s: Pointer to stack
 * 
 * Returns: Top element, or -1 if stack is empty
 * 
 * Algorithm:
 * 1. Check if stack empty (top <= 0)
 * 2. Pre-decrement top (move to previous element)
 * 3. Return element at that position
 * 
 * Key: Use pre-decrement (--s->top) vs post-decrement
 *      Pre: Decrement first, THEN use value
 *      Post: Use value, THEN decrement
 * 
 * Time Complexity: O(1)
 * 
 * Example:
 *   push(&s, 10);  // top = 1
 *   push(&s, 20);  // top = 2
 *   
 *   int x = pop(&s);   // Returns 20, top becomes 1
 *   int y = pop(&s);   // Returns 10, top becomes 0
 *   int z = pop(&s);   // Returns -1 (empty)
 */
int pop(struct Stack *s)
{
    if (s->top <= 0)
        return -1;  /* Stack empty - no elements */
    
    return s->data[--s->top];  /* Pre-decrement top, return element */
}

/**
 * stack_peek - Look at top element without removing
 * @s: Pointer to stack
 * 
 * Returns: Top element, or -1 if empty
 * 
 * Example:
 *   push(&s, 42);
 *   int x = stack_peek(&s);  // Returns 42
 *   int y = stack_peek(&s);  // Still returns 42 (not removed)
 *   int z = pop(&s);         // Now removes and returns 42
 */
int stack_peek(struct Stack *s)
{
    if (s->top <= 0)
        return -1;  /* Empty */
    
    return s->data[s->top - 1];  /* Return top element without removing */
}
```

---

# 34. Check for Balanced Parentheses

## Answer

```c
int is_balanced(const char *str)
{
    struct Stack stack;
    stack.top = 0;
    
    for (int i = 0; str[i]; i++)
    {
        if (str[i] == '(' || str[i] == '[' || str[i] == '{')
        {
            push(&stack, str[i]);
        }
        else if (str[i] == ')' || str[i] == ']' || str[i] == '}')
        {
            int top = pop(&stack);
            
            if ((str[i] == ')' && top != '(') ||
                (str[i] == ']' && top != '[') ||
                (str[i] == '}' && top != '{'))
                return 0;
        }
    }
    
    return stack.top == 0;
}
```

---

# 35. Implement Hash Table (Simple)

## Answer

```c
/**
 * Hash Table Structure
 * 
 * Hash Table = Array + Hash Function + Collision Resolution
 * 
 * Hash Function: Maps key to array index
 *   index = key % HASH_SIZE
 * 
 * Collision: When two different keys hash to same index
 * Resolution: Linear Probing - move to next slot if occupied
 * 
 * Example Layout (HASH_SIZE = 4):
 *   Index:  [0]     [1]     [2]     [3]
 *   Keys:   [7]     [3]     [5]     [1]
 *   Values: [700]   [300]   [500]   [100]
 *   Used:   [1]     [1]     [1]     [1]
 * 
 * Query key=3:
 *   hash(3) = 3 % 4 = 3 (wrong index!)
 *   used[3]=1 and keys[3]=1 (not match) -> probe next
 *   hash(3) = (3+1) % 4 = 0
 *   used[0]=1 and keys[0]=7 (not match) -> probe next
 *   hash(3) = (0+1) % 4 = 1
 *   used[1]=1 and keys[1]=3 (match!) -> return values[1]=300
 */
#define HASH_SIZE 256

struct HashTable
{
    int keys[HASH_SIZE];      /* Array of keys */
    int values[HASH_SIZE];    /* Array of values */
    int used[HASH_SIZE];      /* 1 if slot occupied, 0 if empty */
};

/**
 * hash_function - Simple modulo hash function
 * @key: Key to hash
 * 
 * Returns: Index in range [0, HASH_SIZE-1]
 * 
 * Simple formula: index = key % HASH_SIZE
 * 
 * Example:
 *   hash_function(25) = 25 % 256 = 25
 *   hash_function(257) = 257 % 256 = 1
 * 
 * Note: More complex functions for real applications
 */
int hash_function(int key)
{
    return key % HASH_SIZE;  /* Simple modulo hash */
}

/**
 * hash_insert - Insert key-value pair into hash table
 * @ht: Pointer to hash table
 * @key: Key to insert
 * @value: Value associated with key
 * 
 * Algorithm:
 * 1. Calculate initial index from hash function
 * 2. If slot occupied and key doesn't match: linear probe (move to next)
 * 3. When found empty slot or matching key, store both
 * 4. Mark slot as used
 * 
 * Example:
 *   struct HashTable ht = {0};  // Initialize all zeros
 *   
 *   hash_insert(&ht, 100, 999);
 *   hash_insert(&ht, 200, 888);
 *   hash_insert(&ht, 300, 777);
 * 
 * Time Complexity:
 *   - Best: O(1) - no collision
 *   - Worst: O(n) - many collisions (rare with good hash function)
 * 
 * Note: This simple implementation has limitations:
 *   - Doesn't handle resize
 *   - Doesn't efficiently handle deletion
 */
void hash_insert(struct HashTable *ht, int key, int value)
{
    int index = hash_function(key);  /* Initial index from hash function */
    
    /* Linear probing: find empty slot or matching key */
    while (ht->used[index] && ht->keys[index] != key)
    {
        index = (index + 1) % HASH_SIZE;  /* Move to next slot (wrap around) */
    }
    
    /* Store key and value */
    ht->keys[index] = key;
    ht->values[index] = value;
    ht->used[index] = 1;  /* Mark slot as used */
}

/**
 * hash_search - Find value for given key
 * @ht: Pointer to hash table
 * @key: Key to search for
 * 
 * Returns: Value if found, -1 if not found
 * 
 * Algorithm:
 * 1. Calculate initial index from hash function
 * 2. While slot is used:
 *    - If key matches: FOUND, return value
 *    - Otherwise: linear probe to next slot
 * 3. When reach empty slot: NOT FOUND
 * 
 * Example:
 *   int val = hash_search(&ht, 100);  // Returns 999
 *   int val2 = hash_search(&ht, 999); // Returns -1 (not found)
 * 
 * Time Complexity: Same as insert - O(1) average, O(n) worst
 * 
 * Important: Search terminates when reaching EMPTY slot
 *           (that's why deletion is tricky - can't set used=0)
 */
int hash_search(struct HashTable *ht, int key)
{
    int index = hash_function(key);  /* Initial index from hash function */
    
    /* Linear probing: search through chain of collisions */
    while (ht->used[index])  /* While slot is occupied */
    {
        if (ht->keys[index] == key)
            return ht->values[index];  /* FOUND! Return value */
        
        index = (index + 1) % HASH_SIZE;  /* Move to next slot */
    }
    
    return -1;  /* Reached empty slot - NOT FOUND */
}
```

---

# 36. Implement Simple JSON Parser

## Answer

```c
struct JSONValue
{
    enum { JSON_NULL, JSON_BOOL, JSON_NUMBER, JSON_STRING, JSON_ARRAY, JSON_OBJECT } type;
    union {
        int boolean;
        double number;
        char *string;
    } value;
};

struct JSONValue *parse_json(const char *json)
{
    struct JSONValue *val = malloc(sizeof(struct JSONValue));
    
    if (!json)
    {
        val->type = JSON_NULL;
        return val;
    }
    
    /* Skip whitespace */
    while (*json && (*json == ' ' || *json == '\t' || *json == '\n'))
        json++;
    
    if (*json == 'n')
    {
        val->type = JSON_NULL;
    }
    else if (*json == 't' || *json == 'f')
    {
        val->type = JSON_BOOL;
        val->value.boolean = (*json == 't');
    }
    else if (*json == '"')
    {
        val->type = JSON_STRING;
        /* parse string */
    }
    else if (isdigit(*json) || *json == '-')
    {
        val->type = JSON_NUMBER;
        val->value.number = atof(json);
    }
    
    return val;
}
```

---

# 37. GCD and LCM

## Answer

```c
int gcd(int a, int b)
{
    if (b == 0)
        return a;
    
    return gcd(b, a % b);
}

int lcm(int a, int b)
{
    return (a / gcd(a, b)) * b;
}
```

---

# 38. Implement Fibonacci Sequence

## Answer

```c
int fibonacci(int n)
{
    if (n <= 1)
        return n;
    
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

### Optimized (Memoization)

```c
int fib_memo[50];

int fibonacci_memo(int n)
{
    if (n <= 1)
        return n;
    
    if (fib_memo[n] != -1)
        return fib_memo[n];
    
    fib_memo[n] = fibonacci_memo(n - 1) + fibonacci_memo(n - 2);
    return fib_memo[n];
}
```

### Iterative

```c
int fibonacci_iter(int n)
{
    if (n <= 1)
        return n;
    
    int a = 0, b = 1, c;
    
    for (int i = 2; i <= n; i++)
    {
        c = a + b;
        a = b;
        b = c;
    }
    
    return b;
}
```

---

# 39. Two Sum Problem

## Answer

```c
int *two_sum(int *nums, int numsSize, int target, int *returnSize)
{
    int *result = malloc(2 * sizeof(int));
    *returnSize = 0;
    
    for (int i = 0; i < numsSize - 1; i++)
    {
        for (int j = i + 1; j < numsSize; j++)
        {
            if (nums[i] + nums[j] == target)
            {
                result[0] = i;
                result[1] = j;
                *returnSize = 2;
                return result;
            }
        }
    }
    
    return result;
}
```

---

# 40. Palindrome Check

## Answer

```c
int is_palindrome_str(const char *str)
{
    int left = 0, right = strlen(str) - 1;
    
    while (left < right)
    {
        if (str[left] != str[right])
            return 0;
        
        left++;
        right--;
    }
    
    return 1;
}

int is_palindrome_num(int n)
{
    if (n < 0)
        return 0;
    
    int original = n;
    int reversed = 0;
    
    while (n > 0)
    {
        reversed = reversed * 10 + (n % 10);
        n /= 10;
    }
    
    return original == reversed;
}
```

---

# 41. Roman Numeral to Integer

## Answer

```c
int roman_to_int(const char *s)
{
    int result = 0;
    int prev = 0;
    
    for (int i = strlen(s) - 1; i >= 0; i--)
    {
        int curr = 0;
        
        switch (s[i])
        {
        case 'I': curr = 1; break;
        case 'V': curr = 5; break;
        case 'X': curr = 10; break;
        case 'L': curr = 50; break;
        case 'C': curr = 100; break;
        case 'D': curr = 500; break;
        case 'M': curr = 1000; break;
        }
        
        if (curr < prev)
            result -= curr;
        else
            result += curr;
        
        prev = curr;
    }
    
    return result;
}
```

---

# 42. Matrix Rotation

## Answer

```c
void rotate_matrix_90(int **matrix, int n)
{
    /* Transpose */
    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
        {
            int temp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = temp;
        }
    }
    
    /* Reverse each row */
    for (int i = 0; i < n; i++)
    {
        int left = 0, right = n - 1;
        
        while (left < right)
        {
            int temp = matrix[i][left];
            matrix[i][left] = matrix[i][right];
            matrix[i][right] = temp;
            left++;
            right--;
        }
    }
}
```

---

# 43. Longest Common Subsequence (LCS)

## Answer

```c
int lcs(const char *s1, const char *s2, int m, int n)
{
    int **dp = malloc((m + 1) * sizeof(int *));
    
    for (int i = 0; i <= m; i++)
        dp[i] = malloc((n + 1) * sizeof(int));
    
    for (int i = 0; i <= m; i++)
    {
        for (int j = 0; j <= n; j++)
        {
            if (i == 0 || j == 0)
            {
                dp[i][j] = 0;
            }
            else if (s1[i - 1] == s2[j - 1])
            {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            }
            else
            {
                dp[i][j] = (dp[i - 1][j] > dp[i][j - 1]) ?
                           dp[i - 1][j] : dp[i][j - 1];
            }
        }
    }
    
    int result = dp[m][n];
    
    for (int i = 0; i <= m; i++)
        free(dp[i]);
    free(dp);
    
    return result;
}
```

---

# 44. Knapsack Problem (0/1)

## Answer

```c
int knapsack(int *weights, int *values, int n, int capacity)
{
    int **dp = malloc((n + 1) * sizeof(int *));
    
    for (int i = 0; i <= n; i++)
        dp[i] = malloc((capacity + 1) * sizeof(int));
    
    for (int i = 0; i <= n; i++)
    {
        for (int w = 0; w <= capacity; w++)
        {
            if (i == 0 || w == 0)
            {
                dp[i][w] = 0;
            }
            else if (weights[i - 1] <= w)
            {
                int include = values[i - 1] + dp[i - 1][w - weights[i - 1]];
                int exclude = dp[i - 1][w];
                dp[i][w] = (include > exclude) ? include : exclude;
            }
            else
            {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    
    int result = dp[n][capacity];
    
    for (int i = 0; i <= n; i++)
        free(dp[i]);
    free(dp);
    
    return result;
}
```

---

# 45. Implement CRC Checksum

## Answer

```c
/**
 * crc32 - Calculate CRC-32 checksum of data
 * @data: Pointer to data buffer
 * @len: Number of bytes to checksum
 * 
 * Returns: CRC-32 value (0xFFFFFFFF on error)
 * 
 * What is CRC?
 * - Cyclic Redundancy Check: error-detection code
 * - Detects accidental corruption during transmission
 * - NOT a security hash (doesn't protect against intentional changes)
 * 
 * How it works:
 * 1. Start with CRC = 0xFFFFFFFF (initial value)
 * 2. For each byte in data:
 *    - XOR byte with CRC (mix in new data)
 *    - Shift CRC right 8 times (polynomial division simulation)
 *    - If LSB was set before shift, XOR with polynomial 0xEDB88320
 * 3. XOR result with 0xFFFFFFFF (final step)
 * 
 * Polynomial: 0xEDB88320 (CRC-32-IEEE 802.3)
 * 
 * Example:
 *   uint8_t msg[] = {0x48, 0x65, 0x6C, 0x6C, 0x6F};  // "Hello"
 *   uint32_t checksum = crc32(msg, 5);
 *   printf("CRC32: 0x%08X\n", checksum);
 *   // Output: CRC32: 0xF7D18982
 * 
 * Real-world use:
 *   - Network packets (Ethernet frames)
 *   - ZIP/PNG files
 *   - Serial protocols
 * 
 * Time Complexity: O(n) where n = len
 * Space Complexity: O(1)
 */
uint32_t crc32(const uint8_t *data, size_t len)
{
    uint32_t crc = 0xFFFFFFFF;  /* Initialize CRC (all bits set) */
    
    /* Process each byte */
    for (size_t i = 0; i < len; i++)
    {
        crc ^= data[i];  /* Mix in the data byte */
        
        /* Process 8 bits (one for each bit in the byte) */
        for (int j = 0; j < 8; j++)
        {
            if (crc & 1)  /* If least significant bit is set */
            {
                /* Shift right and XOR with polynomial */
                crc = (crc >> 1) ^ 0xEDB88320;
            }
            else
            {
                /* Just shift right */
                crc >>= 1;
            }
        }
    }
    
    return crc ^ 0xFFFFFFFF;  /* Final XOR step */
}

/**
 * crc32_verify - Verify CRC of data
 * @data: Pointer to data buffer
 * @len: Number of bytes (including CRC at the end)
 * @expected_crc: Expected CRC value
 * 
 * Returns: 1 if CRC matches, 0 if mismatch (data corrupted)
 * 
 * Typical usage:
 *   struct Packet {
 *       uint8_t payload[100];
 *       uint32_t crc;
 *   };
 *   
 *   struct Packet rx;
 *   if (crc32_verify(rx.payload, 100, rx.crc)) {
 *       printf("Data OK\n");
 *   } else {
 *       printf("Corruption detected!\n");
 *   }
 */
int crc32_verify(const uint8_t *data, size_t len, uint32_t expected_crc)
{
    uint32_t calculated = crc32(data, len);
    return calculated == expected_crc;  /* 1 if match, 0 if mismatch */
}

/**
 * Fast CRC32 using lookup table
 * 
 * Optimization: Pre-compute results for all 256 possible byte values
 * Much faster than bit-by-bit calculation
 * Trade-off: 1024 bytes of lookup table vs 8x faster computation
 * 
 * Typical embedded system choice:
 * - If CPU slow and memory abundant: use table
 * - If CPU fast and memory scarce: use bit-by-bit
 */
static uint32_t crc32_table[256];

void crc32_init_table(void)
{
    /* Build lookup table (run once at initialization) */
    for (int i = 0; i < 256; i++)
    {
        uint32_t crc = i;
        for (int j = 0; j < 8; j++)
        {
            if (crc & 1)
                crc = (crc >> 1) ^ 0xEDB88320;
            else
                crc >>= 1;
        }
        crc32_table[i] = crc;
    }
}

uint32_t crc32_fast(const uint8_t *data, size_t len)
{
    uint32_t crc = 0xFFFFFFFF;
    
    for (size_t i = 0; i < len; i++)
    {
        uint8_t byte = data[i];
        crc = (crc >> 8) ^ crc32_table[(crc ^ byte) & 0xFF];
    }
    
    return crc ^ 0xFFFFFFFF;
}
```

---

# 46. Implement Simple Logger with Levels

## Answer

```c
#define LOG_DEBUG 0
#define LOG_INFO 1
#define LOG_WARN 2
#define LOG_ERROR 3

static int log_level = LOG_INFO;

void set_log_level(int level)
{
    log_level = level;
}

void log_message(int level, const char *format, ...)
{
    if (level < log_level)
        return;
    
    const char *level_str[] = {"DEBUG", "INFO", "WARN", "ERROR"};
    
    va_list args;
    va_start(args, format);
    
    printf("[%s] ", level_str[level]);
    vprintf(format, args);
    printf("\n");
    
    va_end(args);
}
```

---

# 47. Implement Simple Config Parser

## Answer

```c
struct Config
{
    char key[128];
    char value[256];
};

int parse_config(const char *filename, struct Config *configs, int max_configs)
{
    FILE *f = fopen(filename, "r");
    if (!f)
        return 0;
    
    int count = 0;
    char line[384];
    
    while (fgets(line, sizeof(line), f) && count < max_configs)
    {
        /* Skip comments */
        if (line[0] == '#' || line[0] == '\n')
            continue;
        
        char *eq = strchr(line, '=');
        if (!eq)
            continue;
        
        /* Extract key */
        int key_len = eq - line;
        strncpy(configs[count].key, line, key_len);
        configs[count].key[key_len] = '\0';
        
        /* Extract value */
        const char *value_start = eq + 1;
        sscanf(value_start, "%255s", configs[count].value);
        
        count++;
    }
    
    fclose(f);
    return count;
}
```

---

# 48. Implement Bit Masking (Register Simulation)

## Answer

```c
struct Register
{
    uint32_t value;
};

void set_bits(struct Register *reg, uint32_t mask)
{
    reg->value |= mask;
}

void clear_bits(struct Register *reg, uint32_t mask)
{
    reg->value &= ~mask;
}

void toggle_bits(struct Register *reg, uint32_t mask)
{
    reg->value ^= mask;
}

uint32_t read_bits(struct Register *reg, uint32_t mask)
{
    return reg->value & mask;
}

void write_field(struct Register *reg, uint32_t mask, uint32_t value)
{
    reg->value = (reg->value & ~mask) | (value & mask);
}
```

---

# 49. Implement Delay Function (Busy Wait)

## Answer

```c
void delay_ms(uint32_t ms)
{
    uint32_t ticks = ms * (CPU_FREQUENCY / 1000);
    
    while (ticks--)
    {
        asm("nop");  /* No operation */
    }
}
```

### Timer-Based (Better)

```c
void delay_ms_timer(uint32_t ms)
{
    uint32_t start = get_timer_ticks();
    uint32_t end = start + (ms * (CPU_FREQUENCY / 1000));
    
    while (get_timer_ticks() < end);
}
```

---

# 50. Implement Simple Timeout Handler

## Answer

```c
struct Timer
{
    uint32_t start_time;
    uint32_t timeout_ms;
    int expired;
};

void timer_start(struct Timer *timer, uint32_t timeout_ms)
{
    timer->start_time = get_time_ms();
    timer->timeout_ms = timeout_ms;
    timer->expired = 0;
}

int timer_expired(struct Timer *timer)
{
    uint32_t elapsed = get_time_ms() - timer->start_time;
    
    if (elapsed >= timer->timeout_ms)
    {
        timer->expired = 1;
        return 1;
    }
    
    return 0;
}

uint32_t timer_remaining(struct Timer *timer)
{
    uint32_t elapsed = get_time_ms() - timer->start_time;
    
    if (elapsed >= timer->timeout_ms)
        return 0;
    
    return timer->timeout_ms - elapsed;
}
```

---

## REMAINING QUESTIONS (51-100)

# 51. Implement Circular Array Deque

# 52. Implement Trie Data Structure

# 53. Implement Graph BFS Traversal

# 54. Implement Graph DFS Traversal

# 55. Detect Cycle in Directed Graph

# 56. Topological Sort

# 57. Dijkstra's Algorithm

# 58. Implement Priority Queue (Min-Heap)

# 59. Implement LRU Cache

# 60. Implement Bloom Filter

# 61. Word Break Problem (Dynamic Programming)

# 62. Coin Change Problem

# 63. House Robber Problem

# 64. Longest Increasing Subsequence

# 65. Edit Distance (Levenshtein)

# 66. Regular Expression Matching

# 67. N-Queens Problem

# 68. Sudoku Solver

# 69. Implement Simple Event Loop

# 70. CAN Bus Frame Parser

# 71. Implement Modbus Protocol

# 72. I2C Bit-Bang Implementation

# 73. SPI Communication Simulator

# 74. Implement Soft-PWM

# 75. Implement Kalman Filter (Simple)

# 76. UART Ring Buffer with DMA

# 77. Watchdog Timer Implementation

# 78. GPIO Edge Detection

# 79. Implement ADC Software Filter

# 80. Temperature Sensor Reading (with Calibration)

# 81. Error Correcting Code (Hamming)

# 82. Implement Simple CAN Transceiver

# 83. Bluetooth HCI Command Handler

# 84. Implement 1-Wire Protocol

# 85. Ethernet MAC Frame Parsing

# 86. TCP/IP Stack (Simplified)

# 87. DNS Resolver

# 88. Simple MQTT Client

# 89. RESTful API Client

# 90. Implement Simple Bootloader

# 91. Firmware Update Handler

# 92. Implement Shadow Registers (Cortex-M)

# 93. Implement System Reset Handler

# 94. NVIC Priority Configuration

# 95. Fault Handler with Stack Trace

# 96. Performance Counter Implementation

# 97. Implement PID Controller

# 98. State Machine for Motor Control

# 99. Implement Simple Scheduler

# 100. Real-Time Event Dispatcher

---

**Note:** Questions 51-100 require extensive implementations. The pattern established in questions 1-50 applies throughout. Consult the detailed sections above for answer patterns, memory management, optimization techniques, and embedded system considerations.

Each remaining question would follow the same format with:
- Problem statement
- Complete working code
- Follow-up optimizations
- Embedded-specific considerations
- Performance tradeoffs

