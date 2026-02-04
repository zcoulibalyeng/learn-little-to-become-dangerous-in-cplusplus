# Module 4: Pointers & Memory Management

## Understanding Pointers

### What is a Pointer?

A pointer is a variable that stores the memory address of another variable.

```cpp
#include <iostream>

int main() {
    int x = 42;
    int* ptr = &x;  // ptr stores the address of x
    
    std::cout << "Value of x: " << x << "\n";
    std::cout << "Address of x: " << &x << "\n";
    std::cout << "Value of ptr: " << ptr << "\n";
    std::cout << "Value at ptr: " << *ptr << "\n";  // Dereference
    
    // Modify x through pointer
    *ptr = 100;
    std::cout << "x after modification: " << x << "\n";
    
    return 0;
}
```

### Pointer Declaration and Initialization

```cpp
#include <iostream>

int main() {
    // Declaration styles (all equivalent)
    int* ptr1;      // Preferred in C++
    int *ptr2;      // Common in C
    int * ptr3;     // Also valid
    
    // Multiple declarations - be careful!
    int* p1, p2;    // p1 is int*, p2 is int!
    int *q1, *q2;   // Both are int*
    
    // Initialize to null
    int* null_ptr = nullptr;  // C++11/C23 (preferred)
    int* null_ptr2 = NULL;    // C-style (avoid in C++)
    int* null_ptr3 = 0;       // Also works (avoid)
    
    // Initialize to variable address
    int value = 42;
    int* ptr = &value;
    
    // Pointer to pointer
    int** ptr_ptr = &ptr;
    std::cout << "Value: " << **ptr_ptr << "\n";  // 42
    
    return 0;
}
```

### Pointer Types

```cpp
#include <iostream>

int main() {
    // Pointers have types
    int i = 42;
    double d = 3.14;
    
    int* ip = &i;
    double* dp = &d;
    
    // void pointer - generic pointer
    void* vp = &i;
    vp = &d;  // Can point to any type
    
    // Must cast void* to use
    int* ip2 = static_cast<int*>(vp);  // Dangerous if wrong type!
    
    // const pointer variations
    int value = 10;
    
    const int* ptr1 = &value;      // Pointer to const int (data is const)
    // *ptr1 = 20;                 // ERROR: can't modify data
    ptr1 = nullptr;                // OK: can modify pointer
    
    int* const ptr2 = &value;      // Const pointer to int (pointer is const)
    *ptr2 = 20;                    // OK: can modify data
    // ptr2 = nullptr;             // ERROR: can't modify pointer
    
    const int* const ptr3 = &value; // Both are const
    // *ptr3 = 20;                 // ERROR
    // ptr3 = nullptr;             // ERROR
    
    return 0;
}
```

## Pointer Arithmetic

```cpp
#include <iostream>

int main() {
    int arr[] = {10, 20, 30, 40, 50};
    int* ptr = arr;  // Points to first element
    
    std::cout << "ptr points to: " << *ptr << "\n";     // 10
    
    // Increment moves to next element
    ptr++;
    std::cout << "After ptr++: " << *ptr << "\n";       // 20
    
    // Add offset
    ptr = arr;
    std::cout << "*(ptr + 2): " << *(ptr + 2) << "\n";  // 30
    
    // Array indexing is pointer arithmetic
    std::cout << "arr[3]: " << arr[3] << "\n";          // 40
    std::cout << "*(arr + 3): " << *(arr + 3) << "\n";  // 40
    std::cout << "3[arr]: " << 3[arr] << "\n";          // 40 (weird but valid!)
    
    // Pointer difference
    int* p1 = &arr[1];
    int* p2 = &arr[4];
    std::ptrdiff_t diff = p2 - p1;
    std::cout << "Elements between: " << diff << "\n";  // 3
    
    // Comparing pointers
    if (p1 < p2) {
        std::cout << "p1 comes before p2\n";
    }
    
    return 0;
}
```

### Pointers and Arrays

```cpp
#include <iostream>

void print_array(const int* arr, size_t size) {
    for (size_t i = 0; i < size; i++) {
        std::cout << arr[i] << " ";
    }
    std::cout << "\n";
}

void print_array_ptr(const int* begin, const int* end) {
    for (const int* p = begin; p != end; ++p) {
        std::cout << *p << " ";
    }
    std::cout << "\n";
}

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    size_t size = sizeof(arr) / sizeof(arr[0]);
    
    // Array decays to pointer when passed to function
    print_array(arr, size);
    
    // Using begin/end pointers
    print_array_ptr(arr, arr + size);
    
    // 2D array and pointers
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    
    // matrix is int(*)[4] - pointer to array of 4 ints
    int (*row_ptr)[4] = matrix;
    
    std::cout << "matrix[1][2]: " << matrix[1][2] << "\n";
    std::cout << "*(*(matrix + 1) + 2): " << *(*(matrix + 1) + 2) << "\n";
    
    return 0;
}
```

## Dynamic Memory Allocation

### C-Style: malloc/free

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void) {
    // Allocate single value
    int* p = (int*)malloc(sizeof(int));
    if (p == NULL) {
        fprintf(stderr, "Allocation failed\n");
        return 1;
    }
    *p = 42;
    printf("Value: %d\n", *p);
    free(p);
    
    // Allocate array
    size_t n = 10;
    int* arr = (int*)malloc(n * sizeof(int));
    if (arr == NULL) {
        return 1;
    }
    for (size_t i = 0; i < n; i++) {
        arr[i] = i * 10;
    }
    free(arr);
    
    // calloc - allocates and zero-initializes
    int* zeroed = (int*)calloc(n, sizeof(int));
    // All elements are 0
    free(zeroed);
    
    // realloc - resize allocation
    int* dynamic = (int*)malloc(5 * sizeof(int));
    dynamic = (int*)realloc(dynamic, 10 * sizeof(int));
    free(dynamic);
    
    return 0;
}
```

### C++: new/delete

```cpp
#include <iostream>

int main() {
    // Allocate single value
    int* p = new int;
    *p = 42;
    std::cout << "Value: " << *p << "\n";
    delete p;
    
    // Allocate with initialization
    int* q = new int(100);
    int* r = new int{200};  // C++11 uniform init
    delete q;
    delete r;
    
    // Allocate array
    int* arr = new int[10];
    for (int i = 0; i < 10; i++) {
        arr[i] = i;
    }
    delete[] arr;  // Must use delete[] for arrays!
    
    // Zero-initialized array
    int* zeroed = new int[10]();  // All zeros
    delete[] zeroed;
    
    // C++11: Initialized array
    int* init_arr = new int[5]{1, 2, 3, 4, 5};
    delete[] init_arr;
    
    // nothrow version (returns nullptr on failure)
    int* safe = new (std::nothrow) int[1000000000];
    if (safe == nullptr) {
        std::cout << "Allocation failed\n";
    } else {
        delete[] safe;
    }
    
    return 0;
}
```

## Memory Safety and Common Pitfalls

### Common Errors

```cpp
#include <iostream>

void memory_errors() {
    // 1. Memory leak - forgetting to free
    int* leak = new int(42);
    // Missing: delete leak;
    
    // 2. Dangling pointer - using after free
    int* dangling = new int(42);
    delete dangling;
    // *dangling = 10;  // UNDEFINED BEHAVIOR!
    dangling = nullptr;  // Good practice after delete
    
    // 3. Double free
    int* dbl = new int(42);
    delete dbl;
    // delete dbl;  // UNDEFINED BEHAVIOR!
    
    // 4. Array/single mismatch
    int* arr = new int[10];
    // delete arr;   // WRONG! Must use delete[]
    delete[] arr;    // Correct
    
    // 5. Using uninitialized pointer
    int* uninit;
    // *uninit = 42;  // UNDEFINED BEHAVIOR!
    
    // 6. Buffer overflow
    int* buffer = new int[5];
    // buffer[10] = 42;  // Out of bounds!
    delete[] buffer;
    
    // 7. Returning pointer to local variable
    // int* bad() {
    //     int local = 42;
    //     return &local;  // DANGLING POINTER!
    // }
}

int main() {
    std::cout << "Memory safety examples\n";
    return 0;
}
```

### Using Sanitizers

```bash
# Compile with AddressSanitizer
g++ -std=c++23 -fsanitize=address -g program.cpp -o program

# Compile with UndefinedBehaviorSanitizer
g++ -std=c++23 -fsanitize=undefined -g program.cpp -o program

# Both sanitizers
g++ -std=c++23 -fsanitize=address,undefined -g program.cpp -o program

# Memory leak detection
g++ -std=c++23 -fsanitize=leak -g program.cpp -o program
```

## Smart Pointers (C++11)

Smart pointers automatically manage memory, preventing leaks and dangling pointers.

### std::unique_ptr

```cpp
#include <iostream>
#include <memory>

class Resource {
public:
    Resource(int id) : id_(id) {
        std::cout << "Resource " << id_ << " created\n";
    }
    ~Resource() {
        std::cout << "Resource " << id_ << " destroyed\n";
    }
    void use() {
        std::cout << "Using resource " << id_ << "\n";
    }
private:
    int id_;
};

int main() {
    // Create unique_ptr
    std::unique_ptr<Resource> ptr1 = std::make_unique<Resource>(1);
    ptr1->use();
    
    // Transfer ownership
    std::unique_ptr<Resource> ptr2 = std::move(ptr1);
    // ptr1 is now nullptr
    
    if (!ptr1) {
        std::cout << "ptr1 is null\n";
    }
    
    ptr2->use();
    
    // Array of unique_ptr
    auto arr = std::make_unique<int[]>(10);
    arr[0] = 42;
    
    // Release ownership (returns raw pointer)
    Resource* raw = ptr2.release();
    delete raw;  // Must manually delete!
    
    // Reset with new resource
    ptr2 = std::make_unique<Resource>(3);
    ptr2.reset();  // Deletes and sets to nullptr
    
    return 0;
}  // Automatic cleanup
```

### std::shared_ptr

```cpp
#include <iostream>
#include <memory>

class Node {
public:
    int value;
    std::shared_ptr<Node> next;
    
    Node(int v) : value(v) {
        std::cout << "Node " << value << " created\n";
    }
    ~Node() {
        std::cout << "Node " << value << " destroyed\n";
    }
};

int main() {
    // Create shared_ptr
    std::shared_ptr<Node> ptr1 = std::make_shared<Node>(1);
    std::cout << "Count: " << ptr1.use_count() << "\n";  // 1
    
    {
        // Share ownership
        std::shared_ptr<Node> ptr2 = ptr1;
        std::cout << "Count: " << ptr1.use_count() << "\n";  // 2
        
        std::shared_ptr<Node> ptr3 = ptr1;
        std::cout << "Count: " << ptr1.use_count() << "\n";  // 3
    }  // ptr2 and ptr3 go out of scope
    
    std::cout << "Count: " << ptr1.use_count() << "\n";  // 1
    
    // Linked list example
    ptr1->next = std::make_shared<Node>(2);
    ptr1->next->next = std::make_shared<Node>(3);
    
    return 0;
}  // All nodes destroyed
```

### std::weak_ptr

```cpp
#include <iostream>
#include <memory>

class Person {
public:
    std::string name;
    std::shared_ptr<Person> partner;  // Strong reference
    std::weak_ptr<Person> friend_;    // Weak reference (no ownership)
    
    Person(const std::string& n) : name(n) {
        std::cout << name << " created\n";
    }
    ~Person() {
        std::cout << name << " destroyed\n";
    }
};

int main() {
    // Circular reference problem with shared_ptr
    // Using weak_ptr to break the cycle
    
    auto alice = std::make_shared<Person>("Alice");
    auto bob = std::make_shared<Person>("Bob");
    
    // Weak reference - doesn't prevent destruction
    alice->friend_ = bob;
    bob->friend_ = alice;
    
    // Using weak_ptr
    if (auto locked = alice->friend_.lock()) {
        std::cout << alice->name << "'s friend is " << locked->name << "\n";
    } else {
        std::cout << "Friend has been destroyed\n";
    }
    
    // Check if weak_ptr is valid
    std::weak_ptr<Person> weak = alice;
    std::cout << "Expired: " << weak.expired() << "\n";  // false
    
    alice.reset();
    std::cout << "Expired: " << weak.expired() << "\n";  // true
    
    return 0;
}
```

### When to Use Which

```cpp
/*
 * unique_ptr:
 * - Exclusive ownership
 * - Default choice for dynamic allocation
 * - Zero overhead compared to raw pointer
 * - Can be moved but not copied
 *
 * shared_ptr:
 * - Shared ownership (multiple owners)
 * - Reference counted
 * - Thread-safe reference counting
 * - Higher overhead than unique_ptr
 *
 * weak_ptr:
 * - Non-owning observer of shared_ptr
 * - Breaks circular references
 * - Must lock() before using
 *
 * Raw pointer:
 * - Non-owning observer
 * - Interface with C code
 * - When you need pointer arithmetic
 */
```

## Function Pointers

```cpp
#include <iostream>
#include <functional>

// Function pointer type
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }

// C-style function pointer
typedef int (*Operation)(int, int);

// C++11 using alias
using OperationFunc = int (*)(int, int);

// With std::function (more flexible)
using OperationStd = std::function<int(int, int)>;

void apply(int a, int b, Operation op) {
    std::cout << "Result: " << op(a, b) << "\n";
}

int main() {
    // Function pointer
    int (*fp)(int, int) = add;
    std::cout << "fp(3, 4): " << fp(3, 4) << "\n";
    
    // Using typedef
    Operation op = subtract;
    std::cout << "op(10, 3): " << op(10, 3) << "\n";
    
    // Array of function pointers
    Operation ops[] = {add, subtract, multiply};
    for (auto& operation : ops) {
        std::cout << operation(10, 5) << " ";
    }
    std::cout << "\n";
    
    // std::function can hold lambdas too
    OperationStd func = [](int a, int b) { return a * b + 1; };
    std::cout << "Lambda: " << func(3, 4) << "\n";
    
    // Pass function to another function
    apply(5, 3, add);
    apply(5, 3, multiply);
    
    return 0;
}
```

## Exercises

### Exercise 1: Swap Function
Write a function that swaps two integers using pointers.

### Exercise 2: Dynamic Array
Implement a simple dynamic array class with:
- Constructor that allocates memory
- Destructor that frees memory
- `push_back` method
- `operator[]` for access

### Exercise 3: Linked List
Implement a singly linked list using smart pointers.

### Exercise 4: Memory Pool
Create a simple memory pool that pre-allocates memory and hands out chunks.

### Exercise 5: Reference Counting
Implement your own simple reference-counted smart pointer.

## Next Steps

In the next module, we'll cover Object-Oriented Programming:
- Classes and objects
- Constructors and destructors
- Inheritance and polymorphism
- Virtual functions

---

**Practice Tip:** Use Valgrind and AddressSanitizer religiously until memory management becomes second nature!
