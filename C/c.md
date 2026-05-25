# ⚡ C LANGUAGE — INTERVIEW REVISION CHEAT SHEET
> **Goal:** 30-minute pre-interview revision. Pointer handbook + Memory cheat sheet + Last-minute guide.

---

# 1. MOST ASKED C INTERVIEW TOPICS

### 🔴 Very Frequently Asked
1. Pointers (arithmetic, double, void, function pointers)
2. Memory layout (stack, heap, BSS, text)
3. Dangling / Wild / NULL pointers
4. `malloc` / `calloc` / `realloc` / `free`
5. Segmentation fault causes
6. `static` keyword behavior
7. `sizeof(array)` vs `sizeof(pointer)`
8. String literals vs char arrays
9. Structure padding & alignment
10. Output prediction — `*p++`, `(*p)++`, `++*p`

### 🟡 Frequently Asked
- `volatile` and `extern`
- Undefined behavior examples
- Recursion stack behavior
- Function pointer syntax and use
- Memory leaks and how to detect
- `const` pointer variants
- Deep copy vs shallow copy
- Buffer overflow

### 🟢 Rarely Asked
- `register` keyword
- Bit manipulation tricks
- `setjmp` / `longjmp`
- Variable-length arrays (VLA)

---

# 2. MEMORY LAYOUT OF C PROGRAM

```
High Address  ┌──────────────────┐
              │     Stack        │  ← grows downward ↓
              │  (local vars,    │
              │   return addrs)  │
              ├──────────────────┤
              │       ↓          │
              │    (gap)         │
              │       ↑          │
              ├──────────────────┤
              │     Heap         │  ← grows upward ↑
              │  (malloc/calloc) │
              ├──────────────────┤
              │  BSS Segment     │  ← uninit globals/static (zeroed)
              ├──────────────────┤
              │  Data Segment    │  ← init globals/static
              ├──────────────────┤
Low Address   │  Text/Code       │  ← read-only, machine code
              └──────────────────┘
```

| Segment | What lives here | Initialized? |
|---|---|---|
| Text | Machine code, string literals | R/O |
| Data | `int x = 5;` (global) | Yes |
| BSS | `int x;` (global, uninitialized) | Zero-filled |
| Heap | `malloc()` allocations | No |
| Stack | Local vars, params, return addr | No |

**Key interview points:**
- Stack grows **downward**, heap grows **upward**
- Global `int x;` → BSS. Global `int x = 5;` → Data
- String literals go in **Text segment** (read-only!)
- `static int x;` inside a function → Data/BSS (not stack)

**Typical interviewer questions:**
- "Where does a local variable live?" → Stack
- "Where does `static int x = 0` inside main live?" → BSS/Data
- "Can you modify a string literal?" → No → segfault

---

# 3. STACK VS HEAP

| Property | Stack | Heap |
|---|---|---|
| Allocation | Automatic (compiler) | Manual (`malloc`) |
| Deallocation | Automatic (function return) | Manual (`free`) |
| Speed | Very fast (just move SP) | Slower (OS/allocator involved) |
| Size | Small (~1–8 MB) | Large (limited by RAM) |
| Lifetime | Scope of function | Until `free()` is called |
| Thread safety | Each thread has own stack | Shared; needs synchronization |
| Fragmentation | None | Yes |
| Memory leak risk | None | Yes |

**Real interview explanation:**
> "Stack memory is automatically managed — when a function returns, its frame is popped. Heap memory persists until explicitly freed. If I allocate on heap and lose the pointer without freeing, that's a memory leak."

**Common trap:** Returning address of a local (stack) variable → dangling pointer!

```c
int* bad() {
    int x = 10;
    return &x;  // ❌ x is destroyed after return
}
```

---

# 4. POINTER MASTER SECTION

## Basics

```c
int x = 5;
int *p = &x;   // p holds address of x

*p = 10;       // dereference — changes x to 10
p++;           // moves p by sizeof(int) bytes
```

## Pointer Arithmetic

```c
int arr[] = {10, 20, 30};
int *p = arr;

p + 1  → points to arr[1]    // moves by sizeof(int) = 4 bytes
p + 2  → points to arr[2]

*(p+1) == arr[1] == 20        // TRUE
```

**Rule:** `p + n` moves by `n * sizeof(*p)` bytes

## Double Pointers

```c
int x = 5;
int *p = &x;
int **pp = &p;

**pp = 99;   // x is now 99
```

**Use case:** Modifying a pointer inside a function (pass-by-pointer-to-pointer)

```c
void allocate(int **ptr) {
    *ptr = malloc(sizeof(int));
    **ptr = 42;
}
```

## Void Pointers

```c
void *vp;
int x = 5;
vp = &x;           // OK — void* accepts any pointer
*(int*)vp = 10;    // Must cast before dereference
```
- Cannot dereference without casting
- Cannot do pointer arithmetic on `void*` (in standard C)
- `malloc` returns `void*`

## Dangling Pointer ⚠️

```c
int *p = malloc(sizeof(int));
free(p);
*p = 5;   // ❌ UNDEFINED BEHAVIOR — p is dangling

// Fix:
free(p);
p = NULL;  // ✅ nullify after free
```

**Definition:** Pointer that points to memory that has been freed or is out of scope.

## Wild Pointer ⚠️

```c
int *p;     // uninitialized — points to garbage
*p = 5;     // ❌ CRASH — wild pointer
```

**Fix:** Always initialize: `int *p = NULL;`

## NULL Pointer

```c
int *p = NULL;   // p = 0 (doesn't point anywhere)
*p = 5;          // ❌ segfault

// Safe pattern:
if (p != NULL) *p = 5;
```

**`NULL` vs `0` vs `'\0'`:**
| Symbol | Value | Used for |
|---|---|---|
| `NULL` | `(void*)0` | Null pointer |
| `0` | integer zero | Arithmetic |
| `'\0'` | char 0 | String terminator |

## Function Pointers

```c
int add(int a, int b) { return a + b; }

int (*fp)(int, int) = add;   // declare
int result = fp(2, 3);       // call → 5

// Typedef for readability:
typedef int (*Op)(int, int);
Op fp2 = add;
```

**Interview use:** Callbacks, dispatch tables, plugins.

## Pointer to Array vs Array of Pointers

```c
// Pointer to array of 5 ints
int (*p)[5];      // p points to entire array
(*p)[0] = 1;

// Array of 5 int pointers
int *arr[5];      // 5 separate pointers
arr[0] = &x;
```

**Memory trick:**
```
int *arr[5]  → arr is [pointer][pointer][pointer][pointer][pointer]
int (*p)[5]  → p → [int][int][int][int][int]
```

## const Pointer Variants

```c
int x = 5, y = 10;

const int *p = &x;    // can't change *p, CAN change p
int *const p = &x;    // CAN change *p, can't change p
const int *const p = &x; // can't change either
```

**Memory trick:** Read right to left.
- `const int *p` → p is a pointer to const int
- `int *const p` → p is a const pointer to int

## Memory Visualization

```
Stack:                 Heap:
┌───────┐              ┌───────┐
│  p ──────────────→   │  42   │
│ (addr)│              └───────┘
│  x=5  │
└───────┘
```

## Top Pointer Questions Asked in Interviews

**Q1:** What is the difference between `*p++`, `(*p)++`, `++*p`?

```c
int arr[] = {10, 20, 30};
int *p = arr;

*p++    // returns *p (10), then increments p → p now points to arr[1]
(*p)++  // increments the value at p → arr[0] becomes 11
++*p    // increments value at p first → arr[0] becomes 11, returns 11
```

**Q2:** What does this print?
```c
int arr[] = {1,2,3,4,5};
int *p = arr + 2;
printf("%d %d", *p, *(p-1));   // Output: 3 2
```

**Q3:** Size of pointer?
```c
sizeof(int*)   // 4 on 32-bit, 8 on 64-bit — same for ALL pointer types
sizeof(char*)  // also 4 or 8 — NOT 1!
```

**Segfault causes from pointers:**
- Dereferencing NULL
- Dereferencing dangling pointer
- Dereferencing uninitialized (wild) pointer
- Buffer overrun (going past array bounds)
- Writing to read-only memory (string literal)

---

# 5. ARRAYS VS POINTERS

| Feature | Array | Pointer |
|---|---|---|
| `sizeof` | Total bytes of array | Size of pointer (4 or 8) |
| Reassignable | ❌ No (`arr = something` illegal) | ✅ Yes |
| Memory | Allocated inline | Stores an address |
| `&arr` | Address of whole array | Address of pointer variable |

```c
int arr[5] = {1,2,3,4,5};
int *p = arr;

sizeof(arr)  // 20 (5 * 4 bytes)
sizeof(p)    // 8 (pointer size on 64-bit)
```

**Array decays to pointer when:**
- Passed to a function
- Used in expressions (except `sizeof` and `&`)

```c
void func(int arr[]) {  // actually int *arr — decay!
    sizeof(arr);         // ❌ gives pointer size, NOT array size
}
```

**Tricky interview example:**
```c
int arr[5];
int *p = arr;

arr[2] == *(arr + 2) == p[2] == *(p + 2)  // ALL equivalent

&arr      // type: int(*)[5] — pointer to entire array
&arr + 1  // jumps 20 bytes (past the whole array)
&p        // type: int** — pointer to pointer
&p + 1    // jumps sizeof(pointer) bytes
```

---

# 6. STATIC / VOLATILE / EXTERN

## `static`

| Context | Effect |
|---|---|
| Local variable | Persists across function calls (in Data/BSS) |
| Global variable | File scope only (not visible to other files) |
| Function | File scope only |

```c
void counter() {
    static int count = 0;  // initialized once, persists
    count++;
    printf("%d\n", count);
}
counter(); // 1
counter(); // 2
counter(); // 3
```

**Why interviewers ask:** Tests understanding of lifetime vs scope.

**Trap:** `static` local initializes only ONCE.

## `volatile`

```c
volatile int flag = 0;  // tells compiler: DO NOT optimize this away
```

**Use cases:**
- Memory-mapped hardware registers
- Signal handlers
- Variables shared between ISR and main

**Why:** Without `volatile`, compiler may cache value in register and miss changes from hardware/another thread.

**Interview line:** *"volatile prevents compiler from optimizing accesses to a variable — used in embedded/ISR contexts."*

## `extern`

```c
// file1.c
int global = 5;

// file2.c
extern int global;  // declaration only — defined elsewhere
```

- Declares without defining
- Links across translation units
- No memory allocated at `extern` declaration

---

# 7. DYNAMIC MEMORY ALLOCATION

## The Four Functions

| Function | Initializes? | Usage |
|---|---|---|
| `malloc(size)` | ❌ garbage | `int *p = malloc(n * sizeof(int));` |
| `calloc(n, size)` | ✅ zero-fills | `int *p = calloc(n, sizeof(int));` |
| `realloc(ptr, newsize)` | ❌ new part is garbage | `p = realloc(p, new_n * sizeof(int));` |
| `free(ptr)` | N/A | `free(p); p = NULL;` |

```c
int *p = malloc(5 * sizeof(int));
if (!p) { /* ALWAYS check for NULL */ }

p = realloc(p, 10 * sizeof(int));  // may return new pointer
// old p is invalid if realloc moved it!

free(p);
p = NULL;  // prevent dangling pointer
```

**Common bugs:**

```c
// 1. Memory leak
int *p = malloc(100);
p = malloc(200);  // ❌ first block leaked — no free!

// 2. Double free
free(p);
free(p);  // ❌ undefined behavior

// 3. Use after free
free(p);
*p = 5;   // ❌ dangling pointer

// 4. Wrong size
int *p = malloc(5);  // ❌ should be malloc(5 * sizeof(int))
```

**Interviewer follow-ups:**
- "What does `malloc(0)` return?" → Implementation-defined (NULL or unique pointer)
- "What happens if you `free(NULL)`?" → Safe, no-op
- "Difference between `malloc` and `calloc`?" → calloc zero-initializes and takes count + size
- "Can `realloc` return NULL?" → Yes, if allocation fails (original pointer still valid!)

---

# 8. STRINGS IN C

## char array vs string literal

```c
char arr[] = "hello";    // mutable — copy on stack
char *p = "hello";       // pointer to read-only literal in text segment

arr[0] = 'H';  // ✅ OK
p[0] = 'H';    // ❌ SEGFAULT — modifying read-only memory
```

**Key rule:** String literals are **immutable**. Always use `char[]` if you need to modify.

## Null Terminator

```c
char s[] = "hi";   // stored as: ['h', 'i', '\0']
strlen(s) == 2     // doesn't count '\0'
sizeof(s) == 3     // DOES count '\0'
```

**Trap:** `sizeof` vs `strlen`
```c
char s[] = "hello";
sizeof(s)   // 6 (with \0)
strlen(s)   // 5 (without \0)
```

## Dangerous Functions

| Function | Problem | Safe alternative |
|---|---|---|
| `gets()` | No bounds check → buffer overflow | `fgets()` |
| `strcpy()` | No bounds check | `strncpy()` |
| `strcat()` | No bounds check | `strncat()` |
| `sprintf()` | No bounds check | `snprintf()` |

## Segfault examples with strings

```c
char *p = "hello";
p[0] = 'H';        // ❌ segfault — text segment is read-only

char *p;
strcpy(p, "hello"); // ❌ segfault — p is wild (uninitialized)

char s[3];
strcpy(s, "hello"); // ❌ buffer overflow — s only holds 3 chars
```

---

# 9. STRUCTURE PADDING & ALIGNMENT

**Why padding?** CPU accesses memory aligned to its natural boundaries (2/4/8 bytes). Compiler inserts padding to align members.

**Rule:** Each member aligned to its own size. Struct size is multiple of largest member.

```c
struct A {
    char a;    // 1 byte + 3 padding
    int b;     // 4 bytes
    char c;    // 1 byte + 3 padding
};
// sizeof(A) = 12, NOT 6!

struct B {
    int b;     // 4 bytes
    char a;    // 1 byte
    char c;    // 1 byte + 2 padding
};
// sizeof(B) = 8 — reordered → smaller!
```

**Memory layout of struct A:**
```
[a][pad][pad][pad][b  b  b  b][c][pad][pad][pad]
 1   2    3    4   5  6  7  8  9  10   11   12
```

**Minimize size:** Order members from largest to smallest.

**`#pragma pack(1)`** — eliminates padding (use with caution, may be slow).

```c
#pragma pack(1)
struct A {
    char a;  // no padding!
    int b;
    char c;
};
// sizeof(A) = 6
```

**sizeof trap:**
```c
struct { char a; int b; } s;
sizeof(s) // NOT 5 — answer is 8 due to padding
```

---

# 10. UNDEFINED BEHAVIOR & SEGMENTATION FAULTS

## Segfault Common Causes

```c
// 1. NULL dereference
int *p = NULL;
*p = 5;          // ❌ segfault

// 2. Stack overflow (deep recursion)
void f() { f(); }  // ❌ infinite recursion → stack exhausted

// 3. Buffer overrun
int arr[5];
arr[10] = 99;    // ❌ may segfault or corrupt memory

// 4. Freed memory access
int *p = malloc(4);
free(p);
*p = 5;          // ❌ undefined behavior

// 5. String literal modification
char *s = "hello";
s[0] = 'H';      // ❌ segfault

// 6. Misaligned access (some architectures)
char buf[5];
int *p = (int*)(buf + 1);
*p = 5;          // ❌ undefined on strict alignment systems
```

## Common Undefined Behaviors

```c
int x = INT_MAX;
x++;              // ❌ signed integer overflow — UB

int arr[5];
arr[5] = 10;      // ❌ out of bounds — UB

int x;
printf("%d", x);  // ❌ reading uninitialized variable — UB

int x = 5;
int y = x++ + x++;  // ❌ multiple modifications in expression — UB
```

## Debugging Mindset

- **Segfault:** Use GDB, AddressSanitizer (`-fsanitize=address`)
- **Memory leak:** Use Valgrind
- **UB detection:** `-fsanitize=undefined`
- **First suspect:** pointers — check for NULL, dangling, wild

---

# 11. TOP OUTPUT-BASED QUESTIONS

**Q1 — Pointer increment**
```c
int arr[] = {10, 20, 30};
int *p = arr;
printf("%d\n", *p++);
printf("%d\n", *p);
```
**Output:** `10` then `20`
**Trap:** `*p++` returns value at p THEN increments p. Post-increment.

---

**Q2 — `(*p)++` vs `*p++`**
```c
int x = 5;
int *p = &x;
printf("%d\n", (*p)++);
printf("%d\n", x);
```
**Output:** `5` then `6`
**Trap:** `(*p)++` increments the value, returns old value.

---

**Q3 — Static variable**
```c
void f() {
    static int x = 0;
    x++;
    printf("%d ", x);
}
int main() { f(); f(); f(); }
```
**Output:** `1 2 3`
**Trap:** static initialized once, persists across calls.

---

**Q4 — sizeof**
```c
int arr[] = {1,2,3,4,5};
int *p = arr;
printf("%zu %zu\n", sizeof(arr), sizeof(p));
```
**Output (64-bit):** `20 8`
**Trap:** `sizeof(arr)` = 5*4 = 20. `sizeof(p)` = pointer size = 8.

---

**Q5 — String sizeof vs strlen**
```c
char s[] = "hello";
printf("%zu %zu\n", sizeof(s), strlen(s));
```
**Output:** `6 5`
**Trap:** sizeof includes `\0`, strlen does not.

---

**Q6 — Pointer to pointer**
```c
int x = 10;
int *p = &x;
int **pp = &p;
**pp = 99;
printf("%d\n", x);
```
**Output:** `99`

---

**Q7 — Array pointer arithmetic**
```c
int arr[] = {1,2,3,4,5};
int *p = arr;
printf("%d\n", *(p+3));
printf("%d\n", p[3]);
printf("%d\n", 3[p]);   // ← tricky!
```
**Output:** `4 4 4`
**Trap:** `3[p]` is legal! `a[i]` = `*(a+i)` = `*(i+a)` = `i[a]`

---

**Q8 — Increment in expression**
```c
int i = 5;
printf("%d\n", i++ + i++);
```
**Output:** Undefined behavior! Don't answer confidently; say "UB."

---

**Q9 — Dangling pointer print**
```c
int *f() {
    int x = 10;
    return &x;       // returning address of local variable
}
int main() {
    int *p = f();
    printf("%d\n", *p);  // undefined behavior
}
```
**Output:** Undefined / garbage / crash
**Trap:** Local variable is destroyed after function returns.

---

**Q10 — Structure sizeof trap**
```c
struct S { char a; int b; };
printf("%zu\n", sizeof(struct S));
```
**Output:** `8` (not 5 — due to padding)

---

**Q11 — malloc + sizeof**
```c
int *p = malloc(10);    // only 10 bytes, not 10 ints!
p[2] = 100;             // might work but is a bug
```
**Correct:** `malloc(10 * sizeof(int))`

---

**Q12 — const pointer**
```c
int x = 5, y = 10;
const int *p = &x;
p = &y;        // ✅ OK — pointer can change
*p = 20;       // ❌ compile error — value is const
```

---

**Q13 — Recursive output**
```c
void f(int n) {
    if (n == 0) return;
    printf("%d ", n);
    f(n-1);
    printf("%d ", n);
}
f(3);
```
**Output:** `3 2 1 1 2 3`

---

**Q14 — void pointer cast**
```c
int x = 65;
void *vp = &x;
printf("%c\n", *(char*)vp);
```
**Output:** `A` (65 is ASCII 'A')

---

# 12. TOP INTERVIEW Q&A

**Stack vs Heap?**
Stack is automatic, fast, limited size, function-scoped. Heap is manual (`malloc/free`), slower, large, persists until freed. Stack overflow from deep recursion; heap leaks from forgetting `free`.

**malloc vs calloc?**
Both allocate heap memory. `malloc(n)` allocates n bytes uninitialized. `calloc(count, size)` allocates count×size bytes and zero-initializes. `calloc` is safer when you need zero-initialized data.

**Dangling pointer?**
Pointer that references freed or out-of-scope memory. Fix: set to NULL after free. Common source of use-after-free bugs and security vulnerabilities.

**Memory leak?**
Heap memory allocated but never freed, and the pointer is lost. Detected with Valgrind. Prevented by always calling `free` and setting pointer to NULL.

**Segmentation fault?**
OS signal when a process accesses memory it's not allowed to: NULL deref, stack overflow, out-of-bounds access, modifying read-only memory.

**Array vs pointer?**
Array is a fixed-size contiguous block; `sizeof` gives total size. Pointer is a variable storing an address; `sizeof` gives 4 or 8. Arrays decay to pointers when passed to functions, losing size info.

**static keyword?**
For local variables: persists across calls (stored in data segment, not stack). For globals/functions: limits visibility to the file. The scope and lifetime behaviors are different — know both.

**volatile keyword?**
Tells compiler not to optimize accesses — variable may change unexpectedly (hardware register, ISR, signal). Without it, compiler may cache value in register and miss updates.

**Deep vs shallow copy?**
Shallow copy copies the pointer (both point to same memory). Deep copy duplicates the pointed-to data too. Shallow copy risks double-free and shared mutation.

**Why no garbage collector in C?**
C is designed for systems programming — predictable performance, manual control. GC introduces pauses and overhead. C trusts the programmer to manage memory explicitly.

**Function pointer use?**
Callbacks (like `qsort` comparator), event handlers, plugins/dispatch tables, state machines. Allows runtime selection of behavior without `switch` statements.

---

# 13. COMMON INTERVIEW TRAPS

**Trap 1 — Return local variable address**
```c
int* f() {
    int x = 5;
    return &x;   // ❌ x is on stack, destroyed after return
}
```
**Why it fails:** Stack frame popped. Pointer is dangling. Any use is UB.
**Expected answer:** *"You'd get a dangling pointer — undefined behavior. Use heap allocation inside the function or pass a pointer in from the caller."*

---

**Trap 2 — Uninitialized pointer**
```c
int *p;
*p = 5;   // ❌ wild pointer — p holds garbage address
```
**Expected:** *"Always initialize pointers. `int *p = NULL;` or to a valid address."*

---

**Trap 3 — Accessing freed memory**
```c
int *p = malloc(4);
free(p);
printf("%d\n", *p);  // ❌ use-after-free — UB
```
**Expected:** *"After free, set pointer to NULL. Accessing freed memory is undefined behavior — may crash, may silently corrupt."*

---

**Trap 4 — sizeof(pointer) vs sizeof(array)**
```c
void f(int arr[]) {
    printf("%zu\n", sizeof(arr));  // prints 8, NOT array size!
}
int a[10];
f(a);
```
**Expected:** *"Arrays decay to pointers when passed to functions. sizeof gives pointer size. Pass array size separately."*

---

**Trap 5 — Modifying string literal**
```c
char *s = "hello";
s[0] = 'H';   // ❌ segfault — string literal in read-only .text
```
**Expected:** *"Use `char s[] = \"hello\"` for a mutable copy. String literals are read-only."*

---

**Trap 6 — Recursive stack overflow**
```c
int f(int n) { return n + f(n-1); }  // no base case
```
**Expected:** *"Missing base case → infinite recursion → stack overflow → segfault. Each call allocates a new stack frame."*

---

**Trap 7 — Off-by-one in malloc**
```c
char *s = malloc(strlen("hello"));  // ❌ forgot +1 for \0
strcpy(s, "hello");                  // writes past allocated memory
```
**Expected:** `malloc(strlen("hello") + 1)`

---

**Trap 8 — realloc losing original pointer**
```c
int *p = malloc(10 * sizeof(int));
p = realloc(p, 20 * sizeof(int));  // ❌ if realloc fails, p = NULL, original leaked!
```
**Fix:**
```c
int *tmp = realloc(p, 20 * sizeof(int));
if (!tmp) { /* handle error, p still valid */ }
else p = tmp;
```

---

# 14. 5-MINUTE FINAL REVISION SHEET

### ⚡ Pointer Rules
- `*p++` → dereference then advance pointer
- `(*p)++` → increment value at pointer
- `++*p` → increment value, return new value
- All pointer types are same size (4 or 8 bytes)
- `p + n` moves by `n * sizeof(*p)` bytes
- Array name is pointer to first element (except in `sizeof` and `&`)
- `a[i]` = `*(a+i)` = `i[a]` (all equivalent)

### ⚡ Memory Rules
- Local vars → Stack (auto-managed, fast, small)
- `malloc` → Heap (manual, large, persists)
- Global/static → Data or BSS segment
- String literals → Text segment (READ-ONLY)
- Stack grows down ↓, Heap grows up ↑
- `free(NULL)` is safe. `free(ptr)` twice → UB.

### ⚡ sizeof Quick Reference
```
sizeof(char)   = 1
sizeof(int)    = 4
sizeof(ptr)    = 4 or 8 (32/64-bit)
sizeof(arr)    = n * sizeof(element)   ← in scope only
sizeof(struct) ≥ sum of members        ← padding!
```

### ⚡ Common Traps — One-Liners
- Return `&local_var` → dangling pointer
- `char *p;` without init → wild pointer
- `free(p); *p = x;` → use-after-free
- Pass array to function → sizeof gives pointer size
- `char *s = "hi"; s[0]='H';` → segfault
- Deep recursion → stack overflow
- `malloc(strlen(s))` → off-by-one, need `+1`
- `p = realloc(p, n)` → leaks on failure, use temp

### ⚡ static — Three Meanings
1. Local: persists across calls
2. Global: file-private
3. Function: file-private

### ⚡ One-Line Interview Answers
| Question | Answer |
|---|---|
| Where does malloc allocate? | Heap |
| stack vs heap lifetime? | Scope vs until free() |
| What is a dangling pointer? | Points to freed/dead memory |
| What is a wild pointer? | Uninitialized pointer |
| calloc vs malloc? | calloc zero-initializes |
| sizeof("abc")? | 4 (includes \0) |
| Can you modify string literal? | No — segfault |
| What causes segfault? | Invalid memory access |
| volatile purpose? | Prevent compiler optimization |
| extern purpose? | Declare without defining |

---
*🎯 Good luck. You've got this.*
