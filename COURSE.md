
# C++ Memory Management - Complete Course Reference

## Table of Contents
1. [Memory Fundamentals](#1-memory-fundamentals)
2. [Pointers and Pointer Arithmetic](#2-pointers-and-pointer-arithmetic)
3. [Dynamic Memory Allocation](#3-dynamic-memory-allocation)
4. [Copy Control and Move Semantics](#4-copy-control-and-move-semantics)
5. [Smart Pointers](#5-smart-pointers)
6. [Advanced Topics](#6-advanced-topics)

---

## 1. Memory Fundamentals

### 1.1 Virtual Memory

**Virtual Address Space Concept**
- Each process gets its own private, isolated address space
- OS translates virtual addresses to physical RAM addresses
- Provides protection between processes

**Benefits:**
- **Limited RAM**: Uses disk space as RAM extension when needed
- **Fragmentation**: OS maps contiguous virtual memory to non-contiguous physical memory
- **Protection**: Each process has isolated address space (Process A's 0x1000 ≠ Process B's 0x1000)

### 1.2 Program Memory Structure

#### Static Memory Areas

**1. Text Segment (.text)**
- Contains compiled machine code instructions
- Marked as read-only by OS
- Includes all functions and program logic

**2. Data Segment (.data)**
- Stores initialized global and static variables
- Values embedded in executable file
- Example: `int global = 100;`

**3. BSS Segment (Block Started by Symbol)**
- Reserved for uninitialized global/static variables
- Zero-initialized automatically by OS
- Optimization: saves space in executable file

#### Dynamic Memory Areas

**1. The Heap**
- Dynamic memory allocation (`new`/`malloc`)
- Programmer-controlled lifetime
- Grows upward toward higher addresses
- Managed by memory allocator
- Used for:
  - Data of unknown size at compile time
  - Objects needing lifetime beyond function scope
  - Large data structures

**2. The Stack**
- Manages function calls and automatic variables
- LIFO (Last-In, First-Out) principle
- Grows downward toward lower addresses
- Extremely fast allocation/deallocation
- Stores:
  - Local variables
  - Function parameters
  - Return addresses
  - Function call bookkeeping

**Stack vs Heap Comparison:**

| Feature | Stack | Heap |
|---------|-------|------|
| Speed | Very fast | Slower |
| Size | Limited (few MB) | Large |
| Lifetime | Automatic | Manual |
| Management | Compiler-managed | Programmer-managed |
| Risk | Stack overflow | Memory leaks |

### 1.3 Memory Addresses

**Address Representation**
- Every byte has a unique numerical address
- Represented in hexadecimal (base 16)
- Prefixed with `0x` (e.g., `0x7FFC1234ABCD`)

**Why Hexadecimal?**
- **Compactness**: 64-bit address = 16 hex digits vs 20 decimal digits
- **Easy Binary Conversion**: Each hex digit = 4 bits exactly

**Hexadecimal Reference:**
```
Hex: 0  1  2  3  4  5  6  7  8  9  A   B   C   D   E   F
Dec: 0  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15
Bin: 0000 0001 0010 0011 0100 0101 0110 0111 1000 1001 1010 1011 1100 1101 1110 1111
```

**Address Size:**
- 32-bit system: 4 bytes per address
- 64-bit system: 8 bytes per address

### 1.4 Memory Hierarchy

**From Fastest to Slowest:**

1. **CPU Registers**
   - Fastest, smallest
   - Directly inside CPU
   - Hold data CPU is actively working on
   - Access time: ~1 clock cycle

2. **Cache Memory (L1, L2, L3)**
   - L1: Smallest, fastest (~KB), split into L1d (data) and L1i (instructions)
   - L2: Larger, slightly slower (~MB), per-core or shared
   - L3: Largest cache, slowest cache, shared by all cores
   - Made of SRAM (Static RAM)

3. **RAM (Random Access Memory)**
   - Main operational workspace (GB)
   - Volatile (lost when power off)
   - Access time: tens to hundreds of nanoseconds

4. **Permanent Storage (HDD/SSD)**
   - Largest capacity (TB)
   - Non-volatile (persists without power)
   - Slowest: milliseconds access time
   - Used for swap space in virtual memory

**Trade-off Rule:**
- As you move down: Larger capacity, cheaper per byte, but MUCH slower

### 1.5 Cache Fundamentals

**Cache Operation:**
- **Cache Hit**: Data found in cache (fast access, ~few cycles)
- **Cache Miss**: Data not in cache (must fetch from RAM, ~100+ cycles)

**Cache Line:**
- Data fetched in fixed-size blocks (typically 64 or 128 bytes)
- When you access 1 byte, entire cache line is loaded

**Principles of Locality:**

1. **Temporal Locality** (Time)
   - If data is accessed once, it will likely be accessed again soon
   - Examples: loop variables, frequently called functions
   - Cache keeps recently accessed data

2. **Spatial Locality** (Space)
   - If data at address X is accessed, nearby addresses will likely be accessed
   - Examples: array iteration, struct member access
   - Cache prefetches neighboring data in cache lines

**Performance Impact:**
```cpp
// GOOD: Sequential access (spatial locality)
int arr[1000];
for (int i = 0; i < 1000; i++) {
    sum += arr[i];  // Cache-friendly
}

// BAD: Random access (poor locality)
for (int i = 0; i < 1000; i++) {
    sum += arr[rand() % 1000];  // Cache thrashing
}
```

### 1.6 Debugging with GDB

**Essential GDB Commands:**

```bash
# Compile with debug symbols
g++ -g program.cpp -o program

# Start GDB
gdb ./program

# Set breakpoints
break main
break function_name
break file.cpp:line_number

# Control execution
run                    # Start program
next (n)              # Step over
step (s)              # Step into
continue (c)          # Continue to next breakpoint

# Inspect variables
print variable_name    # Show value
print &variable_name   # Show address
print *pointer         # Dereference pointer

# Inspect memory
x/nfu address         # Examine memory
  n = count
  f = format (d=decimal, x=hex, c=char, s=string, i=instruction)
  u = unit (b=byte, h=halfword, w=word, g=giant)

# Examples
x/1dw &my_int         # 1 word as decimal
x/8cb &my_array       # 8 bytes as characters

# View memory sections
info file sections
info proc mappings <PID>

# Other useful commands
info locals           # Show local variables
backtrace (bt)       # Show call stack
quit                 # Exit GDB
```

---

## 2. Pointers and Pointer Arithmetic

### 2.1 Pointer Basics

**Definition:**
A pointer is a variable that stores a memory address.

**Declaration Syntax:**
```cpp
type *pointerName;

int *pNumber;      // Pointer to int
double *pPrice;    // Pointer to double
char *pLetter;     // Pointer to char
```

**Key Points:**
- All pointers are the same size (4 bytes on 32-bit, 8 bytes on 64-bit)
- The type specifies what kind of data the pointer points to
- Uninitialized pointers contain garbage addresses

**Address-of Operator (&):**
```cpp
int year = 2025;
int *pYear = &year;  // pYear now holds the address of year
```

**Dereference Operator (*):**
```cpp
int value = *pYear;  // Read: value = 2025
*pYear = 2028;       // Write: year is now 2028
```

**Complete Example:**
```cpp
int year = 2025;
int *pYear = &year;

std::cout << year << std::endl;        // Output: 2025
std::cout << &year << std::endl;       // Output: 0x7fff... (address)
std::cout << pYear << std::endl;       // Output: 0x7fff... (same address)
std::cout << *pYear << std::endl;      // Output: 2025

*pYear = 2028;
std::cout << year << std::endl;        // Output: 2028
```

### 2.2 Pointer Arithmetic

**Array and Pointer Relationship:**
```cpp
int scores[] = {85, 92, 78, 95, 88};
int *ptr = scores;  // Array name decays to pointer to first element

// These are equivalent:
scores[0]    <==>   *ptr
scores[1]    <==>   *(ptr + 1)
&scores[0]   <==>   ptr
&scores[1]   <==>   ptr + 1
```

**Pointer Arithmetic Rules:**
- `ptr + n` moves forward by `n * sizeof(type)` bytes
- `ptr - n` moves backward by `n * sizeof(type)` bytes
- `ptr++` moves to next element
- `ptr--` moves to previous element

**Example:**
```cpp
int grades[] = {70, 80, 90, 100};
int *p = grades;

std::cout << *p << std::endl;       // 70
p++;                                 // Move to next int
std::cout << *p << std::endl;       // 80
p += 2;                             // Move forward 2 ints
std::cout << *p << std::endl;       // 100
p--;                                // Move back 1 int
std::cout << *p << std::endl;       // 90

// Access without modifying pointer
std::cout << *(grades + 0) << std::endl;  // 70
std::cout << *(grades + 1) << std::endl;  // 80
std::cout << *(grades + 2) << std::endl;  // 90
```

### 2.3 The sizeof Operator

```cpp
std::cout << sizeof(int) << std::endl;     // Typically 4 or 8
std::cout << sizeof(double) << std::endl;  // Typically 8
std::cout << sizeof(char) << std::endl;    // Always 1

int arr[10];
std::cout << sizeof(arr) << std::endl;     // 40 or 80 (10 * sizeof(int))
```

### 2.4 Memory Alignment

**Definition:**
Computers read/write memory in chunks (4 or 8 bytes) for performance.

**Alignment Rules:**
- `int` (4 bytes) should start at address divisible by 4 (0, 4, 8, 12...)
- `double` (8 bytes) should start at address divisible by 8 (0, 8, 16...)

**Why It Matters:**
- **Performance**: Unaligned access is much slower
- **Correctness**: Some hardware crashes on unaligned access

**Struct Padding Example:**
```cpp
struct Example {
    char a;     // 1 byte
    // 3 bytes padding
    int b;      // 4 bytes
    char c;     // 1 byte
    // 3 bytes padding
};

// Expected: 1 + 4 + 1 = 6 bytes
// Actual: sizeof(Example) = 12 bytes
```

### 2.5 Pointers vs References

**Key Differences:**

| Feature | Pointer | Reference |
|---------|---------|-----------|
| Nullable | Can be `nullptr` | Cannot be null |
| Reassignment | Can point to different variables | Bound to one variable forever |
| Syntax | Requires `*` and `&` operators | Used like original variable |
| Memory address | Can get address with `&ptr` | Only address of referred variable |
| Initialization | Optional at declaration | Must be initialized |

**Examples:**

```cpp
// POINTERS
int x = 10, y = 20;
int *p = &x;
p = &y;           // Valid: p now points to y
p = nullptr;      // Valid: p points to nothing

if (p != nullptr) {
    *p = 30;      // Safe to use
}

// REFERENCES
int x = 10, y = 20;
int &r = x;       // Must initialize
r = y;            // Doesn't change what r refers to!
                  // This assigns value of y (20) to x
// r still refers to x, x is now 20

// int &r2;       // ERROR: Must be initialized
// r = nullptr;   // ERROR: Cannot be null
```

**When to Use Each:**

**Use References:**
- Function parameters (pass-by-reference)
- Return values (for chaining operations)
- Operator overloading
- Range-based for loops: `for (int &val : vec)`

**Use Pointers:**
- Dynamic memory allocation (`new`/`delete`)
- Optional values (can be `nullptr`)
- Arrays and pointer arithmetic
- Need to reassign (change what's pointed to)
- C API compatibility

### 2.6 Common Pointer Errors

#### 2.6.1 Dereferencing Null/Uninitialized Pointers

```cpp
// BAD: Uninitialized pointer
int *p;
*p = 10;          // CRASH! p contains garbage address

// BAD: Null pointer
int *p = nullptr;
*p = 10;          // CRASH! Dereferencing null

// GOOD: Check before use
int *p = nullptr;
if (p != nullptr) {
    *p = 10;
} else {
    // Handle error
}

// GOOD: Initialize properly
int *p = new int;
*p = 10;          // Safe
delete p;
p = nullptr;
```

#### 2.6.2 Dangling Pointers

```cpp
// BAD: Returning pointer to local variable
int* createPointer() {
    int x = 10;
    return &x;    // DANGER! x destroyed when function returns
}

int main() {
    int *p = createPointer();
    *p = 20;      // UNDEFINED BEHAVIOR! Memory no longer valid
}

// BAD: Using after delete
int *data = new int(50);
delete data;
*data = 100;      // DANGER! Memory freed, pointer dangling

// GOOD: Set to nullptr after delete
int *data = new int(50);
delete data;
data = nullptr;   // Safe dangling prevention

if (data != nullptr) {
    *data = 100;  // Safe check
}
```

#### 2.6.3 Out-of-Bounds Access

```cpp
int arr[3] = {1, 2, 3};
int *p = arr;

std::cout << *(p + 0) << std::endl;  // OK: arr[0]
std::cout << *(p + 1) << std::endl;  // OK: arr[1]
std::cout << *(p + 2) << std::endl;  // OK: arr[2]
std::cout << *(p + 3) << std::endl;  // DANGER! Out of bounds
std::cout << *(p - 1) << std::endl;  // DANGER! Out of bounds

// GOOD: Use std::vector with bounds checking
std::vector<int> vec = {1, 2, 3};
std::cout << vec.at(0) << std::endl;  // Safe: throws exception if out of bounds
```

### 2.7 Best Practices

**Golden Rules:**
1. **Always initialize pointers** (to valid address or `nullptr`)
2. **Set to nullptr after delete** (prevents dangling pointers)
3. **Check for nullptr before dereferencing**
4. **Match allocation with deallocation** (`new` with `delete`, `new[]` with `delete[]`)
5. **Prefer references over pointers when possible**
6. **Use smart pointers for ownership** (discussed later)

---

## 3. Dynamic Memory Allocation

### 3.1 The `new` and `delete` Operators

#### 3.1.1 Single Object Allocation

**Syntax:**
```cpp
Type *pointer = new Type;           // Default initialization
Type *pointer = new Type(value);    // Value initialization
Type *pointer = new Type{value};    // Brace initialization (C++11)
```

**Examples:**
```cpp
// Allocate and initialize
int *pInt = new int(42);
double *pDouble = new double(3.14159);

std::cout << *pInt << std::endl;     // Output: 42
std::cout << *pDouble << std::endl;  // Output: 3.14159

// Deallocate
delete pInt;
pInt = nullptr;       // Good practice

delete pDouble;
pDouble = nullptr;
```

**Memory Allocation Failure:**
- Modern C++: Throws `std::bad_alloc` exception
- Old behavior: Returned `nullptr` (can use `new (std::nothrow)`)

#### 3.1.2 Array Allocation

**Syntax:**
```cpp
Type *pointer = new Type[size];              // Uninitialized
Type *pointer = new Type[size]();            // Zero-initialized
Type *pointer = new Type[size]{val1, val2};  // List-initialized
```

**Examples:**
```cpp
int size = 5;
int *dynamicArray = new int[size];

// Initialize array
for (int i = 0; i < size; ++i) {
    dynamicArray[i] = i * 10;
}

// Use array
for (int i = 0; i < size; ++i) {
    std::cout << dynamicArray[i] << " ";  // Output: 0 10 20 30 40
}

// CRITICAL: Must use delete[] for arrays
delete[] dynamicArray;
dynamicArray = nullptr;
```

**CRITICAL RULE:**
```cpp
// CORRECT MATCHING
new Type      -->  delete ptr;
new Type[]    -->  delete[] ptr;

// WRONG: Undefined behavior, memory leak, crash
new Type      -->  delete[] ptr;    // WRONG!
new Type[]    -->  delete ptr;      // WRONG!
```

### 3.2 Memory Leaks

**Definition:**
Memory leak occurs when allocated memory is not deallocated, causing gradual memory consumption.

**Common Causes:**

```cpp
// 1. Forgetting to delete
void simpleLeak() {
    int *data = new int[100];
    // ... use data ...
    // No delete[] here! LEAK!
}

// 2. Early return without cleanup
bool processData(int* buffer) {
    int* temp = new int[100];
    
    if (buffer == nullptr) {
        return false;     // LEAK! Forgot to delete temp
    }
    
    // ... process ...
    delete[] temp;
    return true;
}

// 3. Reassigning pointers
void leak() {
    int *p = new int(10);
    int *q = new int(20);
    p = q;               // LEAK! Lost reference to first allocation
    delete q;            // Only deletes second allocation
}
```

**Solutions:**

```cpp
// 1. Always add matching delete
void noLeak() {
    int *data = new int[100];
    // ... use data ...
    delete[] data;
}

// 2. Ensure cleanup on all paths
bool processData(int* buffer) {
    int* temp = new int[100];
    
    if (buffer == nullptr) {
        delete[] temp;    // Cleanup before return
        return false;
    }
    
    // ... process ...
    delete[] temp;
    return true;
}

// 3. Better: Use RAII (covered later)
void bestPractice() {
    std::unique_ptr<int[]> data(new int[100]);
    // Automatic cleanup when scope ends
}
```

### 3.3 Memory Ownership

**Definition:**
Memory ownership is the concept that a specific part of your program is responsible for deleting dynamically allocated memory.

**Ownership Scenarios:**

#### 3.3.1 Local Ownership (Simple)
```cpp
void processData() {
    int *buffer = new int[10000];
    // Use buffer...
    delete[] buffer;
    buffer = nullptr;
}
// Clear ownership: function allocates and deletes
```

#### 3.3.2 Transferring Ownership (Function Returns Pointer)
```cpp
// Factory function transfers ownership to caller
int* createData() {
    int *data = new int(42);
    return data;    // Caller now owns this memory
}

void caller() {
    int *myData = createData();  // Caller takes ownership
    // ... use myData ...
    delete myData;               // Caller must delete
    myData = nullptr;
}

// IMPORTANT: Document ownership clearly!
// Comment: "Returns dynamically allocated int. Caller takes ownership and must delete."
```

#### 3.3.3 Passing Ownership (Function Takes Pointer)
```cpp
void destroyData(int* data) {
    // Takes ownership and deletes
    if (data != nullptr) {
        delete data;
        // Note: local copy of pointer set to null, not caller's
    }
}

void caller() {
    int *myData = new int(100);
    destroyData(myData);    // Transfers ownership
    // myData is now invalid (dangling)
    myData = nullptr;       // Good practice
}
```

#### 3.3.4 Shared Ownership (Complex - Use std::shared_ptr)
```cpp
// Manual shared ownership is error-prone
// Multiple owners need reference counting
// Solution: Use std::shared_ptr (covered in Section 5)
```

#### 3.3.5 Non-Ownership (Borrowing/Observing)
```cpp
void printData(const int* data) {
    // Does NOT own 'data'
    // Only reads, does not delete
    if (data != nullptr) {
        std::cout << "Data: " << *data << std::endl;
    }
    // NO delete here!
}

void owner() {
    int *ownedData = new int(99);
    printData(ownedData);    // Borrows pointer
    // ... 
    delete ownedData;        // Owner still deletes
    ownedData = nullptr;
}
```

### 3.4 Double Deletion

**The Problem:**
```cpp
int *p = new int(10);
int *q = p;           // Both point to same memory

delete p;
p = nullptr;          // Good!

delete q;             // CRASH! Double free
```

**Why It's Dangerous:**
- Corrupts heap management structures
- Can cause immediate crash or delayed corruption
- Security vulnerability

**Solution:**
```cpp
int *p = new int(10);
int *q = p;

delete p;
p = nullptr;

// Don't delete q if it points to same memory as p
// Or check:
if (q != nullptr && q != p) {
    delete q;
}
q = nullptr;

// BEST: Avoid raw pointer sharing - use smart pointers
```

### 3.5 Memory Safety Checklist

**Every time you write `new`, ask:**
1. ✓ Where is the matching `delete`?
2. ✓ Is it called on ALL execution paths (including exceptions)?
3. ✓ Who owns this memory?
4. ✓ Am I using `delete[]` for arrays?
5. ✓ Did I set the pointer to `nullptr` after delete?

**Every time you use a pointer, ask:**
1. ✓ Is it initialized?
2. ✓ Is it `nullptr`?
3. ✓ Has the memory been deleted already?
4. ✓ Am I staying within array bounds?

---

## 4. Copy Control and Move Semantics

### 4.1 The Problem: Shallow Copy

**Default Behavior:**
When you copy an object with raw pointers, C++ performs a shallow copy (copies the pointer value, not the data).

```cpp
class MyString {
public:
    char* m_buffer;
    
    MyString(const char* str) {
        m_buffer = new char[strlen(str) + 1];
        strcpy(m_buffer, str);
    }
    
    ~MyString() {
        delete[] m_buffer;
    }
    // No custom copy constructor/assignment
};

// DANGER:
MyString s1("Hello");
MyString s2 = s1;     // Default copy: s1.m_buffer == s2.m_buffer (same address!)

// When s2 is destroyed: deletes m_buffer
// When s1 is destroyed: tries to delete same m_buffer -> CRASH!
```

**Consequences:**
1. **Double Deletion**: Both objects try to delete the same memory
2. **Unintended Mutation**: Modifying one object affects the other
```cpp
s2.m_buffer[0] = 'J';
// Both s1 and s2 now say "Jello"!
```

### 4.2 The Rule of Three

**If a class defines ANY of these, it should define ALL:**
1. Destructor
2. Copy Constructor
3. Copy Assignment Operator

#### 4.2.1 Implementing Destructor

```cpp
class MyString {
private:
    char* m_buffer;
    
public:
    ~MyString() {
        delete[] m_buffer;
        m_buffer = nullptr;  // Good practice
    }
};
```

#### 4.2.2 Implementing Copy Constructor

**Purpose:** Create a new, independent deep copy when initializing from another object.

**Syntax:**
```cpp
ClassName(const ClassName& other);
```

**Called When:**
- `MyString s2 = s1;`
- `MyString s3(s1);`
- Passing by value to function
- Returning by value from function

**Implementation:**
```cpp
class MyString {
private:
    char* m_buffer;
    size_t m_size;
    
public:
    // Copy Constructor
    MyString(const MyString& other) {
        // 1. Allocate NEW memory
        m_size = other.m_size;
        m_buffer = new char[m_size + 1];
        
        // 2. Copy contents (deep copy)
        strcpy(m_buffer, other.m_buffer);
    }
};
```

#### 4.2.3 Implementing Copy Assignment Operator

**Purpose:** Assign one existing object to another, handling existing resources.

**Syntax:**
```cpp
ClassName& operator=(const ClassName& other);
```

**Called When:**
- `s2 = s1;` (both objects already exist)

**Implementation:**
```cpp
class MyString {
public:
    MyString& operator=(const MyString& other) {
        // 1. Self-assignment check
        if (this == &other) {
            return *this;
        }
        
        // 2. Deallocate old memory
        delete[] m_buffer;
        
        // 3. Allocate new memory
        m_size = other.m_size;
        m_buffer = new char[m_size + 1];
        
        // 4. Copy contents
        strcpy(m_buffer, other.m_buffer);
        
        // 5. Return *this
        return *this;
    }
};
```

**Complete Example:**
```cpp
class MyString {
private:
    char* m_buffer;
    size_t m_size;
    
public:
    // Constructor
    MyString(const char* str) {
        m_size = strlen(str);
        m_buffer = new char[m_size + 1];
        strcpy(m_buffer, str);
    }
    
    // Destructor
    ~MyString() {
        delete[] m_buffer;
        m_buffer = nullptr;
    }
    
    // Copy Constructor
    MyString(const MyString& other) {
        m_size = other.m_size;
        m_buffer = new char[m_size + 1];
        strcpy(m_buffer, other.m_buffer);
    }
    
    // Copy Assignment Operator
    MyString& operator=(const MyString& other) {
        if (this == &other) return *this;
        
        delete[] m_buffer;
        m_size = other.m_size;
        m_buffer = new char[m_size + 1];
        strcpy(m_buffer, other.m_buffer);
        
        return *this;
    }
};
```

### 4.3 Move Semantics (C++11)

**Problem:** Copying is expensive for large objects. What if we're copying from a temporary?

**Solution:** Move semantics - "steal" resources from temporary objects instead of copying.

#### 4.3.1 Lvalues vs Rvalues

**Lvalue:** Has a name, persists beyond expression
```cpp
int x = 10;           // x is lvalue
obj.member;           // lvalue
*ptr;                 // lvalue
```

**Rvalue:** Temporary, doesn't persist
```cpp
10;                   // literal is rvalue
x + y;                // expression result is rvalue
MyString("Hello");    // temporary object is rvalue
```

**Rvalue Reference:** `Type&&`
```cpp
void process(int& x);     // Takes lvalue
void process(int&& x);    // Takes rvalue

int a = 10;
process(a);               // Calls lvalue version
process(20);              // Calls rvalue version
process(a + 5);           // Calls rvalue version
```

#### 4.3.2 Move Constructor

**Purpose:** Create object by "stealing" resources from temporary.

**Syntax:**
```cpp
ClassName(ClassName&& other) noexcept;
```

**Implementation:**
```cpp
class MyString {
public:
    MyString(MyString&& other) noexcept 
        : m_buffer(nullptr), m_size(0) {
        
        // 1. Steal resources
        m_buffer = other.m_buffer;
        m_size = other.m_size;
        
        // 2. Leave 'other' in valid empty state
        other.m_buffer = nullptr;
        other.m_size = 0;
    }
};
```

**When Called:**
```cpp
MyString createString() {
    return MyString("Temporary");  // Move constructor used
}

MyString s1 = createString();  // Move constructor (or RVO)
```

#### 4.3.3 Move Assignment Operator

**Purpose:** Assign from temporary, stealing its resources.

**Syntax:**
```cpp
ClassName& operator=(ClassName&& other) noexcept;
```

**Implementation:**
```cpp
class MyString {
public:
    MyString& operator=(MyString&& other) noexcept {
        // 1. Self-assignment check
        if (this == &other) return *this;
        
        // 2. Deallocate existing resources
        delete[] m_buffer;
        
        // 3. Steal resources
        m_buffer = other.m_buffer;
        m_size = other.m_size;
        
        // 4. Leave 'other' in valid empty state
        other.m_buffer = nullptr;
        other.m_size = 0;
        
        // 5. Return *this
        return *this;
    }
};
```

### 4.4 std::move

**Purpose:** Convert lvalue to rvalue to force move semantics.

**Important:** `std::move` doesn't move anything! It just casts.

```cpp
#include <utility>  // for std::move

MyString s1("Hello");
MyString s2("World");

s2 = std::move(s1);  // Calls move assignment
                     // s1 is now empty/moved-from
                     // s2 now owns s1's old resources

// IMPORTANT: Don't use s1 after move (except to assign new value)
// s1 is in "valid but unspecified" state
```

**Common Use Cases:**
```cpp
// 1. Moving into containers
std::vector<MyString> vec;
MyString str("Hello");
vec.push_back(std::move(str));  // Move instead of copy

// 2. Swapping
MyString a("A"), b("B");
MyString temp = std::move(a);
a = std::move(b);
b = std::move(temp);

// 3. Returning from functions (usually automatic)
MyString createString() {
    MyString temp("Hello");
    return temp;  // Automatic move (or RVO)
}
```

### 4.5 The Rule of Five

**If a class defines ANY of these, it should define ALL (C++11+):**
1. Destructor
2. Copy Constructor
3. Copy Assignment Operator
4. **Move Constructor**
5. **Move Assignment Operator**

### 4.6 The Rule of Zero

**Best Practice:** Don't manage raw resources directly!

**Instead of:**
```cpp
class OldWidget {
    int* data;  // Raw pointer - needs Rule of Five!
    
public:
    OldWidget(int val) : data(new int(val)) {}
    ~OldWidget() { delete data; }
    // Need to define copy/move...
};
```

**Do this:**
```cpp
class NewWidget {
    std::unique_ptr<int> data;  // Smart pointer handles everything!
    
public:
    NewWidget(int val) : data(std::make_unique<int>(val)) {}
    // Compiler-generated destructor, copy, move all work correctly!
    // Rule of Zero achieved!
};
```

**Using Standard Library:**
```cpp
class BetterString {
    std::string m_buffer;  // std::string manages its own memory
    
public:
    BetterString(const char* str) : m_buffer(str) {}
    // No custom destructor, copy, or move needed!
};
```

### 4.7 Passing and Returning Objects

#### 4.7.1 Passing to Functions

**Options:**

1. **Pass by Value** (Copy/Move)
```cpp
void func(MyString str);  // Copies or moves
// Use for: Small objects, when function needs its own copy
```

2. **Pass by const Reference** (No Copy - PREFERRED)
```cpp
void func(const MyString& str);  // No copy, read-only
// Use for: Large objects, read-only access
```

3. **Pass by Reference** (No Copy, Can Modify)
```cpp
void func(MyString& str);  // No copy, can modify original
// Use for: Need to modify original object
```

4. **Pass by Pointer** (Can be null, Can Modify)
```cpp
void func(MyString* str);  // Can be nullptr, can modify
// Use for: Optional parameters, C API compatibility
```

**Recommendations:**
- **Default:** `const T&` for large objects
- **Small types:** Pass by value (int, double, bool)
- **Need to modify:** `T&`
- **Optional:** `T*` (check for nullptr)

#### 4.7.2 Returning from Functions

**Return by Value** (PREFERRED - uses RVO/Move)
```cpp
MyString createString() {
    MyString temp("Hello");
    return temp;  // RVO or move - very efficient!
}

MyString s = createString();  // No unnecessary copies
```

**Return by Reference** (For existing objects ONLY)
```cpp
class Widget {
    MyString m_name;
public:
    MyString& getName() { return m_name; }  // OK: m_name outlives function
};

// NEVER do this:
MyString& badFunction() {
    MyString temp("Hello");
    return temp;  // DANGER! temp destroyed, returning dangling reference
}
```

### 4.8 RAII (Resource Acquisition Is Initialization)

**Core Principle:**
- **Acquire resource in constructor**
- **Release resource in destructor**
- Automatic cleanup when object goes out of scope

**Example:**
```cpp
class FileHandler {
    FILE* m_file;
    
public:
    FileHandler(const char* filename) {
        m_file = fopen(filename, "r");
        if (!m_file) throw std::runtime_error("Can't open file");
    }
    
    ~FileHandler() {
        if (m_file) {
            fclose(m_file);
        }
    }
    
    // Use file...
};

void processFile() {
    FileHandler file("data.txt");  // File opened
    // Use file...
    // Exception? Early return? Doesn't matter!
}  // File automatically closed when 'file' goes out of scope
```

**Benefits:**
- Automatic cleanup
- Exception-safe
- No manual resource management
- Basis for smart pointers

---

## 5. Smart Pointers

### 5.1 Introduction

**Problem with Raw Pointers:**
- Manual `new`/`delete` error-prone
- Easy to forget `delete` → memory leaks
- Double deletion crashes
- Exception safety difficult

**Solution: Smart Pointers**
- RAII wrappers around raw pointers
- Automatic memory management
- Type-safe
- Zero or minimal overhead

**Include:**
```cpp
#include <memory>
```

### 5.2 std::unique_ptr - Exclusive Ownership

**Characteristics:**
- **Exclusive ownership:** Only ONE unique_ptr owns the resource
- **Non-copyable:** Cannot copy, can only move
- **Zero overhead:** Same size and performance as raw pointer
- **Default choice** for single ownership

#### 5.2.1 Creating unique_ptr

**Preferred: std::make_unique (C++14+)**
```cpp
auto ptr = std::make_unique<int>(42);
auto arr = std::make_unique<int[]>(10);  // Array

// Why preferred:
// 1. Exception-safe
// 2. Single allocation
// 3. Concise
```

**Alternative: Direct new**
```cpp
std::unique_ptr<int> ptr(new int(42));
std::unique_ptr<int[]> arr(new int[10]);
```

**Complete Example:**
```cpp
#include <memory>
#include <iostream>

class MyClass {
public:
    MyClass() { std::cout << "Constructed\n"; }
    ~MyClass() { std::cout << "Destroyed\n"; }
    void doSomething() { std::cout << "Doing something\n"; }
};

void demo() {
    auto ptr = std::make_unique<MyClass>();
    ptr->doSomething();
    
    // Automatic cleanup when ptr goes out of scope
}  // MyClass destructor called automatically

int main() {
    demo();
    // Output:
    // Constructed
    // Doing something
    // Destroyed
}
```

#### 5.2.2 Using unique_ptr

```cpp
auto ptr = std::make_unique<int>(42);

// Dereference
int value = *ptr;

// Member access
auto obj = std::make_unique<MyClass>();
obj->doSomething();

// Get raw pointer (for C APIs, don't delete!)
int* rawPtr = ptr.get();

// Check if valid
if (ptr) {
    // ptr is not null
}

// Array access
auto arr = std::make_unique<int[]>(10);
arr[0] = 5;
arr[5] = 10;
```

#### 5.2.3 Transferring Ownership

**Move Semantics:**
```cpp
auto owner1 = std::make_unique<MyClass>();
std::unique_ptr<MyClass> owner2;

owner2 = std::move(owner1);  // Ownership transferred
// owner1 is now nullptr
// owner2 now owns the object

if (!owner1) {
    std::cout << "owner1 is empty\n";
}
```

**Returning from Functions:**
```cpp
std::unique_ptr<MyClass> createObject() {
    auto obj = std::make_unique<MyClass>();
    return obj;  // Efficient move, no copy
}

auto myObj = createObject();  // myObj now owns the object
```

**Passing to Functions:**
```cpp
// By reference (doesn't transfer ownership)
void useObject(const std::unique_ptr<MyClass>& ptr) {
    ptr->doSomething();
}

// Transfer ownership (takes ownership)
void takeOwnership(std::unique_ptr<MyClass> ptr) {
    // ptr now owns the object
    // Will be destroyed when function ends
}

auto obj = std::make_unique<MyClass>();
useObject(obj);                    // obj still owns
takeOwnership(std::move(obj));     // Transfers ownership, obj is now null
```

#### 5.2.4 unique_ptr Best Practices

```cpp
// ✓ GOOD: Default choice for single ownership
auto ptr = std::make_unique<MyClass>();

// ✓ GOOD: Factory functions
std::unique_ptr<Base> createDerived() {
    return std::make_unique<Derived>();
}

// ✓ GOOD: Arrays
auto arr = std::make_unique<int[]>(100);

// ✗ BAD: Don't delete manually
auto ptr = std::make_unique<int>(42);
delete ptr.get();  // NO! Undefined behavior

// ✗ BAD: Don't create from unique_ptr::get()
auto ptr1 = std::make_unique<int>(42);
std::unique_ptr<int> ptr2(ptr1.get());  // NO! Double delete
```

### 5.3 std::shared_ptr - Shared Ownership

**Characteristics:**
- **Shared ownership:** Multiple shared_ptrs can own same resource
- **Reference counting:** Tracks number of owners
- **Automatic deletion:** Deletes when last owner destroyed
- **Thread-safe:** Reference counting uses atomic operations
- **Slight overhead:** Requires control block

#### 5.3.1 Creating shared_ptr

**Preferred: std::make_shared**
```cpp
auto ptr = std::make_shared<int>(42);

// Why preferred:
// 1. Single allocation (object + control block together)
// 2. More efficient
// 3. Exception-safe
```

**Alternative: Direct new**
```cpp
std::shared_ptr<int> ptr(new int(42));
// Less efficient: Two allocations (object + control block)
```

**Complete Example:**
```cpp
#include <memory>
#include <iostream>

class Resource {
public:
    Resource() { std::cout << "Resource created\n"; }
    ~Resource() { std::cout << "Resource destroyed\n"; }
    void use() { std::cout << "Using resource\n"; }
};

void demo() {
    std::cout << "Creating shared_ptr\n";
    auto ptr1 = std::make_shared<Resource>();
    std::cout << "Ref count: " << ptr1.use_count() << "\n";  // 1
    
    {
        auto ptr2 = ptr1;  // Copy - increments ref count
        std::cout << "Ref count: " << ptr1.use_count() << "\n";  // 2
        
        auto ptr3 = ptr1;
        std::cout << "Ref count: " << ptr1.use_count() << "\n";  // 3
    }  // ptr2 and ptr3 destroyed, ref count drops to 1
    
    std::cout << "Ref count: " << ptr1.use_count() << "\n";  // 1
}  // ptr1 destroyed, ref count drops to 0, Resource deleted

int main() {
    demo();
    // Output:
    // Creating shared_ptr
    // Resource created
    // Ref count: 1
    // Ref count: 2
    // Ref count: 3
    // Ref count: 1
    // Resource destroyed
}
```

#### 5.3.2 Using shared_ptr

```cpp
auto ptr = std::make_shared<int>(42);

// Same interface as unique_ptr
int value = *ptr;
ptr->member;
int* raw = ptr.get();
if (ptr) { /* valid */ }

// Reference counting
int count = ptr.use_count();

// Reset (release ownership)
ptr.reset();          // Decrements ref count
ptr.reset(new int(99));  // Replace with new resource
```

#### 5.3.3 Sharing Ownership

```cpp
class Data {
public:
    int value;
    Data(int v) : value(v) {}
};

void processData(std::shared_ptr<Data> data) {
    // data's ref count incremented when passed by value
    std::cout << "Value: " << data->value << "\n";
    std::cout << "Ref count in function: " << data.use_count() << "\n";
}  // Ref count decremented when function ends

int main() {
    auto data = std::make_shared<Data>(100);
    std::cout << "Initial ref count: " << data.use_count() << "\n";  // 1
    
    processData(data);  // Ref count temporarily becomes 2
    
    std::cout << "After function: " << data.use_count() << "\n";  // 1
}
```

### 5.4 std::weak_ptr - Non-Owning Observer

**Problem: Circular References**
```cpp
class B;

class A {
public:
    std::shared_ptr<B> b_ptr;  // A owns B
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::shared_ptr<A> a_ptr;  // B owns A
    ~B() { std::cout << "B destroyed\n"; }
};

void leak() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;  // A holds shared_ptr to B (B ref count = 2)
    b->a_ptr = a;  // B holds shared_ptr to A (A ref count = 2)
    
}  // a and b go out of scope
   // A ref count: 2 → 1 (still owned by B)
   // B ref count: 2 → 1 (still owned by A)
   // NEITHER DESTROYED! MEMORY LEAK!
```

**Solution: weak_ptr**

**Characteristics:**
- **Non-owning reference:** Doesn't increase ref count
- **Break cycles:** Prevents circular reference leaks
- **Check validity:** Can check if object still exists
- **Must lock():** Convert to shared_ptr before use

#### 5.4.1 Breaking Circular References

```cpp
class B;

class A {
public:
    std::shared_ptr<B> b_ptr;  // A owns B
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::weak_ptr<A> a_weak;  // B observes A (doesn't own)
    ~B() { std::cout << "B destroyed\n"; }
    
    void checkA() {
        if (auto a_shared = a_weak.lock()) {  // Convert to shared_ptr
            std::cout << "A is still alive\n";
        } else {
            std::cout << "A has been destroyed\n";
        }
    }
};

void noLeak() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;      // A owns B (B ref count = 2)
    b->a_weak = a;     // B observes A (A ref count still 1!)
    
    b->checkA();       // A is still alive
    
}  // a and b go out of scope
   // A ref count: 1 → 0 (DESTROYED)
   // B ref count: 2 → 1 → 0 (DESTROYED)
   // No leak!
```

#### 5.4.2 Using weak_ptr

```cpp
std::shared_ptr<int> shared = std::make_shared<int>(42);
std::weak_ptr<int> weak = shared;

// Cannot dereference directly!
// int value = *weak;  // ERROR!

// Must lock() first
if (auto locked = weak.lock()) {
    // locked is a shared_ptr
    int value = *locked;
    std::cout << "Value: " << value << "\n";
} else {
    std::cout << "Object no longer exists\n";
}

// Check without locking
if (weak.expired()) {
    std::cout << "Object destroyed\n";
}

// Get ref count
int count = weak.use_count();
```

#### 5.4.3 weak_ptr Use Cases

```cpp
// 1. Breaking cycles
class Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;  // Break cycle
};

// 2. Cache
class Cache {
    std::map<int, std::weak_ptr<Data>> cache;
    
public:
    std::shared_ptr<Data> get(int id) {
        auto it = cache.find(id);
        if (it != cache.end()) {
            if (auto data = it->second.lock()) {
                return data;  // Found in cache
            }
        }
        // Not in cache or expired, create new
        auto data = std::make_shared<Data>(id);
        cache[id] = data;
        return data;
    }
};

// 3. Observer pattern
class Subject {
    std::vector<std::weak_ptr<Observer>> observers;
    
public:
    void notify() {
        for (auto& weak : observers) {
            if (auto observer = weak.lock()) {
                observer->update();
            }
        }
    }
};
```

### 5.5 Choosing the Right Smart Pointer

| Scenario | Smart Pointer | Reason |
|----------|---------------|--------|
| Single owner | `unique_ptr` | Default choice, zero overhead |
| Need to copy/share | `shared_ptr` | Multiple owners |
| Optional parameter | `unique_ptr*` or `shared_ptr*` | Can be null |
| Break cycles | `weak_ptr` | Non-owning observer |
| Temporary access | Raw pointer from `get()` | Don't manage lifetime |
| Factory function return | `unique_ptr` | Transfer ownership |
| Containers of polymorphic objects | `unique_ptr<Base>` or `shared_ptr<Base>` | Lifetime management |

**Decision Flow:**
```
Need dynamic allocation?
├─ Yes
│  ├─ Single owner?
│  │  └─ Yes → unique_ptr
│  │
│  └─ Multiple owners?
│     └─ Yes → shared_ptr
│        └─ Circular reference?
│           └─ Yes → Use weak_ptr to break cycle
│
└─ No → Use automatic/stack allocation
```

### 5.6 Smart Pointer Best Practices

**DO:**
```cpp
// ✓ Use make_unique/make_shared
auto ptr = std::make_unique<T>();
auto sptr = std::make_shared<T>();

// ✓ Prefer unique_ptr by default
std::unique_ptr<T> ptr = ...;

// ✓ Pass by reference when not transferring ownership
void use(const std::unique_ptr<T>& ptr);
void use(const std::shared_ptr<T>& ptr);

// ✓ Return by value
std::unique_ptr<T> factory();
std::shared_ptr<T> factory();

// ✓ Use weak_ptr to break cycles
std::weak_ptr<T> observer;
```

**DON'T:**
```cpp
// ✗ Don't use new with smart pointers if possible
auto ptr = std::unique_ptr<T>(new T());  // Use make_unique instead

// ✗ Don't delete manually
auto ptr = std::make_unique<T>();
delete ptr.get();  // NO!

// ✗ Don't create multiple unique_ptr from same raw pointer
T* raw = new T();
std::unique_ptr<T> ptr1(raw);
std::unique_ptr<T> ptr2(raw);  // NO! Double delete

// ✗ Don't pass shared_ptr by value unnecessarily
void func(std::shared_ptr<T> ptr);  // Copies, increments ref count
void func(const std::shared_ptr<T>& ptr);  // Better

// ✗ Don't use shared_ptr if unique_ptr suffices
std::shared_ptr<T> ptr = ...;  // Overhead if not actually shared
```

### 5.7 Converting Raw Pointer Code

**Before (Raw Pointers):**
```cpp
class OldClass {
    int* data;
    
public:
    OldClass(int val) : data(new int(val)) {}
    
    ~OldClass() {
        delete data;
    }
    
    // Need copy constructor
    OldClass(const OldClass& other) {
        data = new int(*other.data);
    }
    
    // Need copy assignment
    OldClass& operator=(const OldClass& other) {
        if (this != &other) {
            delete data;
            data = new int(*other.data);
        }
        return *this;
    }
    
    // Need move constructor, move assignment...
};
```

**After (Smart Pointers - Rule of Zero):**
```cpp
class NewClass {
    std::unique_ptr<int> data;
    
public:
    NewClass(int val) : data(std::make_unique<int>(val)) {}
    
    // That's it! Compiler-generated destructor, copy, move all work!
    // Or delete copy if unique ownership desired:
    // NewClass(const NewClass&) = delete;
    // NewClass& operator=(const NewClass&) = delete;
};
```

**Factory Pattern Conversion:**

**Before:**
```cpp
Resource* getResource() {
    static Resource* instance = nullptr;
    if (!instance) {
        instance = new Resource();
    }
    return instance;  // Who deletes? Leak!
}
```

**After:**
```cpp
std::shared_ptr<Resource> getResource() {
    static std::shared_ptr<Resource> instance = 
        std::make_shared<Resource>();
    return instance;  // Automatic, safe sharing
}
```

---

## 6. Advanced Topics

### 6.1 Performance Considerations

#### unique_ptr Performance
- **Size:** Same as raw pointer (8 bytes on 64-bit)
- **Speed:** Zero overhead, identical to raw pointer
- **Move:** Optimized by compiler (RVO/NRVO)
- **Recommendation:** Use by default

#### shared_ptr Performance
- **Size:** Two pointers (16 bytes on 64-bit)
  - One to object
  - One to control block
- **Overhead:**
  - Atomic operations for thread-safety (slight cost)
  - Control block allocation
  - Reference counting
- **Optimization:** Use `make_shared` (single allocation)
- **Recommendation:** Only use when genuinely need shared ownership

### 6.2 Custom Deleters

**For special cleanup needs:**

```cpp
// Custom deleter for C file handle
auto fileDeleter = [](FILE* f) {
    if (f) fclose(f);
};

std::unique_ptr<FILE, decltype(fileDeleter)> file(
    fopen("data.txt", "r"),
    fileDeleter
);

// Custom deleter for shared_ptr
std::shared_ptr<FILE> file2(
    fopen("data.txt", "r"),
    [](FILE* f) { if (f) fclose(f); }
);

// Array delete for shared_ptr
std::shared_ptr<int> arr(new int[10], [](int* p) { delete[] p; });
// Better: Use unique_ptr<T[]> instead
```

### 6.3 Memory Pools (Advanced)

**When to Consider:**
- Allocating/deallocating same-size objects millions of times
- Predictable memory usage
- Performance-critical systems

**Concept:**
```cpp
class MemoryPool {
    char* pool;
    size_t blockSize;
    void* freeList;
    
public:
    MemoryPool(size_t size, size_t count) {
        blockSize = size;
        pool = new char[size * count];
        // Initialize free list...
    }
    
    void* allocate() {
        // Return block from pool (fast)
    }
    
    void deallocate(void* ptr) {
        // Return block to pool (fast)
    }
};

// Use with placement new
void* memory = pool.allocate();
MyObject* obj = new (memory) MyObject();

// Explicit destructor call
obj->~MyObject();
pool.deallocate(obj);
```

**Note:** Complex to implement correctly. Consider libraries or use smart pointers for 99% of cases.

### 6.4 Debugging Memory Issues

**Tools:**

1. **Valgrind** (Linux/Mac)
```bash
g++ -g program.cpp -o program
valgrind --leak-check=full --track-origins=yes ./program
```

2. **AddressSanitizer** (All platforms)
```bash
g++ -fsanitize=address -g program.cpp -o program
./program
```

3. **Visual Studio** (Windows)
- Use built-in memory leak detection
- CRT Debug Heap

**Common Issues Detected:**
- Memory leaks
- Use after free
- Double free
- Buffer overflows
- Uninitialized memory reads

### 6.5 Common Patterns

#### PIMPL (Pointer to Implementation)
```cpp
// Widget.h
class Widget {
    struct Impl;
    std::unique_ptr<Impl> pImpl;
    
public:
    Widget();
    ~Widget();
    void doSomething();
};

// Widget.cpp
struct Widget::Impl {
    // Private implementation details
    int data;
    std::vector<int> moreData;
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() = default;  // Must define in .cpp where Impl is complete

void Widget::doSomething() {
    pImpl->data = 42;
}
```

#### Observer Pattern with weak_ptr
```cpp
class Observer {
public:
    virtual void notify() = 0;
};

class Subject {
    std::vector<std::weak_ptr<Observer>> observers;
    
public:
    void attach(std::shared_ptr<Observer> obs) {
        observers.push_back(obs);
    }
    
    void notifyAll() {
        // Remove expired observers
        observers.erase(
            std::remove_if(observers.begin(), observers.end(),
                [](const std::weak_ptr<Observer>& wp) { 
                    return wp.expired(); 
                }),
            observers.end()
        );
        
        // Notify remaining
        for (auto& weak : observers) {
            if (auto obs = weak.lock()) {
                obs->notify();
            }
        }
    }
};
```

---

## Quick Reference

### Memory Management Checklist

**Every Allocation:**
- [ ] Is there a matching deallocation?
- [ ] Is it on all code paths (including exceptions)?
- [ ] Who owns this memory?
- [ ] Using correct delete (`delete` vs `delete[]`)?

**Every Pointer Use:**
- [ ] Is it initialized?
- [ ] Is it null?
- [ ] Has memory been deleted?
- [ ] Within array bounds?

### Golden Rules

1. **Prefer smart pointers** over raw pointers for ownership
2. **Use `make_unique`/`make_shared`** for creating smart pointers
3. **Default to `unique_ptr`** unless sharing required
4. **Pass by `const&`** for read-only access to large objects
5. **Set to `nullptr`** after delete (raw pointers)
6. **Always initialize** pointers
7. **Match new/delete** correctly
8. **Rule of Zero:** Use standard library types to avoid custom copy/move

### Syntax Quick Reference

```cpp
// Raw pointers
int* ptr = new int(42);
delete ptr;
ptr = nullptr;

int* arr = new int[10];
delete[] arr;
arr = nullptr;

// Unique pointer
auto ptr = std::make_unique<int>(42);
auto arr = std::make_unique<int[]>(10);

// Shared pointer
auto sptr = std::make_shared<int>(42);
auto sptr2 = sptr;  // Share ownership

// Weak pointer
std::weak_ptr<int> wptr = sptr;
if (auto locked = wptr.lock()) {
    // Use locked
}

// Access
*ptr              // Dereference
ptr->member       // Member access
ptr.get()         // Raw pointer
ptr.reset()       // Release ownership
```

---

## Resources

- **C++ Reference:** https://cppreference.com
- **Guidelines:** https://isocpp.github.io/CppCoreGuidelines/
- **Debugging:** Valgrind, AddressSanitizer, Visual Studio debugger
- **Books:** "Effective Modern C++" by Scott Meyers

---

**End of Course Reference**