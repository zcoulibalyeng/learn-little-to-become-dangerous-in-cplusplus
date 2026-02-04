# QUICK GUIDE - C/C++ Cheat Sheet

## 📝 C/C++ Quick Reference (C23 & C++23)

### Compilation Commands

```bash
# C23 compilation
gcc -std=c23 -Wall -Wextra -o program program.c
clang -std=c23 -Wall -Wextra -o program program.c

# C++23 compilation
g++ -std=c++23 -Wall -Wextra -o program program.cpp
clang++ -std=c++23 -Wall -Wextra -o program program.cpp

# With debugging symbols
g++ -std=c++23 -g -O0 -o program program.cpp

# With optimization
g++ -std=c++23 -O2 -o program program.cpp

# With sanitizers
g++ -std=c++23 -fsanitize=address,undefined -o program program.cpp
```

---

## 🔢 Data Types

### C/C++ Fundamental Types

```cpp
// Boolean
bool flag = true;              // C++: native, C23: keyword
bool active = false;

// Characters
char c = 'A';                  // Usually 8 bits
wchar_t wc = L'Ω';            // Wide character
char8_t u8c = u8'A';          // UTF-8 (C++20)
char16_t u16c = u'A';         // UTF-16 (C++11)
char32_t u32c = U'A';         // UTF-32 (C++11)

// Integers
short s = 32767;              // At least 16 bits
int i = 42;                   // At least 16 bits (usually 32)
long l = 100000L;             // At least 32 bits
long long ll = 9223372036854775807LL;  // At least 64 bits

// Unsigned integers
unsigned int ui = 42u;
unsigned long ul = 100000UL;
size_t sz = sizeof(int);      // Unsigned, for sizes

// Fixed-width integers (from <cstdint> or <stdint.h>)
int8_t i8 = 127;
int16_t i16 = 32767;
int32_t i32 = 2147483647;
int64_t i64 = 9223372036854775807LL;
uint8_t u8 = 255;
uint32_t u32 = 4294967295U;

// Floating-point
float f = 3.14f;              // Usually 32 bits
double d = 3.14159265359;     // Usually 64 bits
long double ld = 3.14159265358979323846L;

// C23: Bit-precise integers
_BitInt(128) big_int;         // Exactly 128 bits

// Size type (for array sizes, etc.)
size_t size = 100;
ptrdiff_t diff = ptr2 - ptr1;
```

### Type Inference (C++11 / C23)

```cpp
// C++11 auto
auto x = 42;                  // int
auto y = 3.14;               // double
auto z = "hello";            // const char*
auto vec = std::vector<int>{1, 2, 3};

// C++14 auto in functions
auto add(int a, int b) { return a + b; }

// C++11 decltype
int a = 5;
decltype(a) b = 10;          // int

// C23 auto (type inference)
auto n = 42;                 // int (C23)

// C23 typeof
typeof(42) m = 100;          // int
```

---

## 📝 Variables and Constants

### Variable Declaration

```cpp
// Basic declaration
int count;                    // Uninitialized (dangerous!)
int count = 0;               // Initialized
int count{0};                // C++11 uniform initialization
int count(0);                // Constructor-style

// Multiple declarations
int a = 1, b = 2, c = 3;

// References (C++)
int x = 10;
int& ref = x;                // Reference to x
ref = 20;                    // x is now 20

// Pointers
int* ptr = &x;               // Pointer to x
int* ptr = nullptr;          // Null pointer (C++11/C23)
```

### Constants

```cpp
// Compile-time constants
const int MAX_SIZE = 100;
constexpr int BUFFER_SIZE = 256;  // C++11

// C++20 consteval (must be evaluated at compile-time)
consteval int square(int n) { return n * n; }

// C++20 constinit (constant initialization)
constinit int global_value = 42;

// Preprocessor macros (avoid when possible)
#define PI 3.14159

// Enumerations
enum Color { RED, GREEN, BLUE };
enum class Color { Red, Green, Blue };  // C++11 scoped enum
```

### Literals (C23/C++23)

```cpp
// Integer literals
int dec = 42;                // Decimal
int oct = 052;               // Octal
int hex = 0x2A;              // Hexadecimal
int bin = 0b101010;          // Binary (C++14/C23)

// Digit separators (C++14/C23)
int million = 1'000'000;
long long big = 0xFFFF'FFFF'FFFF'FFFFLL;

// Floating-point literals
double d = 3.14;
double sci = 6.022e23;       // Scientific notation
float f = 3.14f;
long double ld = 3.14L;

// String literals
const char* s = "Hello";
const char* raw = R"(Line 1
Line 2)";                    // Raw string (C++11)
const char8_t* u8s = u8"UTF-8 string";
const char16_t* u16s = u"UTF-16 string";
const char32_t* u32s = U"UTF-32 string";

// User-defined literals (C++11)
using namespace std::chrono_literals;
auto duration = 100ms;
auto text = "hello"s;        // std::string
```

---

## 🔀 Operators

### Arithmetic Operators

```cpp
int a = 10, b = 3;
int sum = a + b;             // 13
int diff = a - b;            // 7
int prod = a * b;            // 30
int quot = a / b;            // 3 (integer division)
int rem = a % b;             // 1 (modulo)

// Compound assignment
a += b;  // a = a + b
a -= b;  // a = a - b
a *= b;  // a = a * b
a /= b;  // a = a / b
a %= b;  // a = a % b

// Increment/Decrement
++a;     // Pre-increment
a++;     // Post-increment
--a;     // Pre-decrement
a--;     // Post-decrement
```

### Comparison Operators

```cpp
bool eq = (a == b);          // Equal
bool ne = (a != b);          // Not equal
bool lt = (a < b);           // Less than
bool gt = (a > b);           // Greater than
bool le = (a <= b);          // Less than or equal
bool ge = (a >= b);          // Greater than or equal

// C++20 three-way comparison (spaceship operator)
auto result = a <=> b;       // Returns ordering
```

### Logical Operators

```cpp
bool x = true, y = false;
bool and_result = x && y;    // Logical AND
bool or_result = x || y;     // Logical OR
bool not_result = !x;        // Logical NOT
```

### Bitwise Operators

```cpp
int a = 0b1100, b = 0b1010;
int and_bit = a & b;         // 0b1000 (AND)
int or_bit = a | b;          // 0b1110 (OR)
int xor_bit = a ^ b;         // 0b0110 (XOR)
int not_bit = ~a;            // Bitwise NOT
int left = a << 2;           // Left shift
int right = a >> 2;          // Right shift
```

---

## 🔁 Control Flow

### Conditionals

```cpp
// If-else
if (condition) {
    // code
} else if (other_condition) {
    // code
} else {
    // code
}

// C++17 if with initializer
if (auto result = compute(); result > 0) {
    // use result
}

// C++23 consteval if
if consteval {
    // compile-time only
}

// Ternary operator
int max = (a > b) ? a : b;

// Switch
switch (value) {
    case 1:
        // code
        break;
    case 2:
    case 3:
        // code for 2 or 3
        break;
    default:
        // default code
        break;
}

// C++17 switch with initializer
switch (auto x = get_value(); x) {
    case 1: break;
}

// C++23 [[fallthrough]] attribute
switch (x) {
    case 1:
        do_something();
        [[fallthrough]];
    case 2:
        do_more();
        break;
}
```

### Loops

```cpp
// While loop
while (condition) {
    // code
}

// Do-while loop
do {
    // code
} while (condition);

// For loop
for (int i = 0; i < 10; ++i) {
    // code
}

// C++11 range-based for
std::vector<int> vec = {1, 2, 3, 4, 5};
for (int x : vec) {
    // read-only
}
for (int& x : vec) {
    // modifiable
}
for (const auto& x : vec) {
    // read-only, efficient
}

// C++20 range-based for with initializer
for (auto vec = get_vector(); auto& x : vec) {
    // code
}

// Loop control
break;                       // Exit loop
continue;                    // Skip to next iteration
```

---

## 📦 Functions

### Function Declaration and Definition

```cpp
// Declaration (prototype)
int add(int a, int b);

// Definition
int add(int a, int b) {
    return a + b;
}

// Default parameters
void greet(std::string name = "World") {
    std::cout << "Hello, " << name << "!\n";
}

// C++11 trailing return type
auto multiply(int a, int b) -> int {
    return a * b;
}

// C++14 return type deduction
auto divide(double a, double b) {
    return a / b;  // Returns double
}
```

### Parameter Passing

```cpp
// Pass by value (copy)
void func(int x);

// Pass by reference
void func(int& x);           // Can modify x

// Pass by const reference
void func(const int& x);     // Read-only, no copy

// Pass by pointer
void func(int* x);           // Can be nullptr

// C++11 pass by rvalue reference
void func(int&& x);          // Move semantics
```

### Function Overloading (C++)

```cpp
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
std::string add(const std::string& a, const std::string& b) {
    return a + b;
}
```

### Lambda Expressions (C++11+)

```cpp
// Basic lambda
auto add = [](int a, int b) { return a + b; };

// Capture by value
int x = 10;
auto func = [x]() { return x; };

// Capture by reference
auto func = [&x]() { x++; };

// Capture all by value/reference
auto func = [=]() { return x + y; };
auto func = [&]() { x++; y++; };

// C++14 generic lambda
auto add = [](auto a, auto b) { return a + b; };

// C++14 init capture
auto func = [y = x + 1]() { return y; };

// C++20 template lambda
auto func = []<typename T>(T x) { return x * 2; };

// C++23 simplified this capture
struct S {
    void f() {
        auto lambda = [this]() { return value; };
    }
    int value;
};
```

---

## 🏗️ Classes (C++)

### Class Declaration

```cpp
class MyClass {
public:
    // Constructor
    MyClass() : value(0) {}
    MyClass(int v) : value(v) {}
    
    // Destructor
    ~MyClass() {}
    
    // Copy constructor
    MyClass(const MyClass& other) : value(other.value) {}
    
    // Move constructor (C++11)
    MyClass(MyClass&& other) noexcept : value(other.value) {
        other.value = 0;
    }
    
    // Copy assignment
    MyClass& operator=(const MyClass& other) {
        value = other.value;
        return *this;
    }
    
    // Move assignment (C++11)
    MyClass& operator=(MyClass&& other) noexcept {
        value = other.value;
        other.value = 0;
        return *this;
    }
    
    // Member functions
    int getValue() const { return value; }
    void setValue(int v) { value = v; }
    
private:
    int value;
};

// C++11 default and delete
class NonCopyable {
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

### Inheritance

```cpp
class Base {
public:
    virtual void speak() { std::cout << "Base\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void speak() override { std::cout << "Derived\n"; }
};

// Multiple inheritance
class C : public A, public B { };

// Virtual inheritance (for diamond problem)
class A : virtual public Base { };
class B : virtual public Base { };
class C : public A, public B { };
```

---

## 📚 STL Containers

### Sequence Containers

```cpp
#include <vector>
#include <array>
#include <deque>
#include <list>
#include <forward_list>

// Vector (dynamic array)
std::vector<int> vec = {1, 2, 3, 4, 5};
vec.push_back(6);
vec.pop_back();
vec[0] = 10;
vec.at(0) = 10;              // Bounds-checked
vec.size();
vec.empty();
vec.clear();

// Array (fixed-size, C++11)
std::array<int, 5> arr = {1, 2, 3, 4, 5};

// Deque (double-ended queue)
std::deque<int> dq;
dq.push_front(1);
dq.push_back(2);

// List (doubly-linked)
std::list<int> lst = {1, 2, 3};

// Forward list (singly-linked, C++11)
std::forward_list<int> flst = {1, 2, 3};
```

### Associative Containers

```cpp
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

// Map (ordered key-value)
std::map<std::string, int> ages;
ages["Alice"] = 30;
ages.insert({"Bob", 25});
ages.contains("Alice");      // C++20

// Set (ordered unique values)
std::set<int> nums = {3, 1, 4, 1, 5};  // {1, 3, 4, 5}

// Unordered map (hash table)
std::unordered_map<std::string, int> hash_map;

// Unordered set
std::unordered_set<int> hash_set;

// C++23 flat_map and flat_set
#include <flat_map>
std::flat_map<int, std::string> flat;
```

### Container Adapters

```cpp
#include <stack>
#include <queue>
#include <priority_queue>

// Stack (LIFO)
std::stack<int> stk;
stk.push(1);
stk.top();
stk.pop();

// Queue (FIFO)
std::queue<int> q;
q.push(1);
q.front();
q.back();
q.pop();

// Priority queue (max-heap by default)
std::priority_queue<int> pq;
pq.push(3);
pq.push(1);
pq.top();  // 3 (largest)
```

---

## 🔄 Algorithms

```cpp
#include <algorithm>
#include <numeric>

std::vector<int> vec = {3, 1, 4, 1, 5, 9, 2, 6};

// Sorting
std::sort(vec.begin(), vec.end());
std::sort(vec.begin(), vec.end(), std::greater<int>());  // Descending

// Searching
auto it = std::find(vec.begin(), vec.end(), 4);
bool found = std::binary_search(vec.begin(), vec.end(), 4);

// Transforming
std::transform(vec.begin(), vec.end(), vec.begin(),
               [](int x) { return x * 2; });

// Accumulating
int sum = std::accumulate(vec.begin(), vec.end(), 0);

// Min/Max
auto [min_it, max_it] = std::minmax_element(vec.begin(), vec.end());

// Counting
int count = std::count(vec.begin(), vec.end(), 1);
int count_if = std::count_if(vec.begin(), vec.end(),
                             [](int x) { return x > 3; });

// C++20 Ranges
#include <ranges>
namespace rv = std::views;

auto even = vec | rv::filter([](int x) { return x % 2 == 0; });
auto squared = vec | rv::transform([](int x) { return x * x; });
```

---

## 🧵 Memory Management

### Pointers

```cpp
int x = 42;
int* ptr = &x;               // Pointer to x
int value = *ptr;            // Dereference
*ptr = 100;                  // Modify through pointer

// Null pointer
int* null_ptr = nullptr;     // C++11/C23

// Pointer arithmetic
int arr[5] = {1, 2, 3, 4, 5};
int* p = arr;
p++;                         // Points to arr[1]
p += 2;                      // Points to arr[3]
```

### Dynamic Memory (C)

```c
#include <stdlib.h>

// Allocation
int* p = (int*)malloc(sizeof(int) * 10);
int* q = (int*)calloc(10, sizeof(int));  // Zero-initialized

// Reallocation
p = (int*)realloc(p, sizeof(int) * 20);

// Deallocation
free(p);
```

### Dynamic Memory (C++)

```cpp
// Raw new/delete (avoid when possible)
int* p = new int(42);
delete p;

int* arr = new int[10];
delete[] arr;

// Smart pointers (C++11)
#include <memory>

// unique_ptr (exclusive ownership)
std::unique_ptr<int> up = std::make_unique<int>(42);
auto up2 = std::make_unique<int[]>(10);  // Array

// shared_ptr (shared ownership)
std::shared_ptr<int> sp = std::make_shared<int>(42);
auto sp2 = sp;  // Reference count: 2

// weak_ptr (non-owning observer)
std::weak_ptr<int> wp = sp;
if (auto locked = wp.lock()) {
    // Use locked
}
```

---

## 🔧 Modern C++ Features

### Move Semantics (C++11)

```cpp
std::vector<int> createVector() {
    std::vector<int> v = {1, 2, 3};
    return v;  // Moved, not copied
}

std::string s1 = "Hello";
std::string s2 = std::move(s1);  // s1 is now empty

// Move constructor/assignment in class
class MyClass {
public:
    MyClass(MyClass&& other) noexcept 
        : data(std::move(other.data)) {}
    
    MyClass& operator=(MyClass&& other) noexcept {
        data = std::move(other.data);
        return *this;
    }
private:
    std::vector<int> data;
};
```

### Structured Bindings (C++17)

```cpp
// Pair/Tuple decomposition
std::pair<int, std::string> p = {1, "hello"};
auto [num, str] = p;

// Map iteration
std::map<std::string, int> m = {{"a", 1}, {"b", 2}};
for (const auto& [key, value] : m) {
    std::cout << key << ": " << value << "\n";
}

// Array decomposition
int arr[3] = {1, 2, 3};
auto [a, b, c] = arr;
```

### std::optional (C++17)

```cpp
#include <optional>

std::optional<int> divide(int a, int b) {
    if (b == 0) return std::nullopt;
    return a / b;
}

auto result = divide(10, 2);
if (result) {
    std::cout << *result << "\n";
}

// Or with value_or
int value = divide(10, 0).value_or(-1);
```

### std::expected (C++23)

```cpp
#include <expected>

std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) return std::unexpected("Division by zero");
    return a / b;
}

auto result = divide(10, 2);
if (result) {
    std::cout << *result << "\n";
} else {
    std::cout << "Error: " << result.error() << "\n";
}
```

### std::print (C++23)

```cpp
#include <print>

std::print("Hello, {}!\n", "World");
std::println("Value: {}", 42);
std::println("Hex: {:#x}", 255);
std::println("Binary: {:b}", 42);
```

---

## 🛠️ Common Patterns

### RAII (Resource Acquisition Is Initialization)

```cpp
class FileHandle {
public:
    FileHandle(const char* filename) {
        file = fopen(filename, "r");
    }
    ~FileHandle() {
        if (file) fclose(file);
    }
    // Delete copy operations
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
private:
    FILE* file;
};

// Usage - file automatically closed when out of scope
{
    FileHandle f("data.txt");
    // use f
}  // f.~FileHandle() called here
```

### Singleton Pattern

```cpp
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance;  // Thread-safe in C++11
        return instance;
    }
    // Delete copy/move
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
private:
    Singleton() = default;
};
```

---

## 📚 Useful Headers

```cpp
// Core
#include <iostream>          // I/O streams
#include <string>            // std::string
#include <string_view>       // std::string_view (C++17)
#include <cstdint>           // Fixed-width integers

// Containers
#include <vector>
#include <array>
#include <map>
#include <unordered_map>
#include <set>

// Algorithms
#include <algorithm>
#include <numeric>
#include <ranges>            // C++20

// Memory
#include <memory>            // Smart pointers

// Utilities
#include <optional>          // C++17
#include <variant>           // C++17
#include <expected>          // C++23
#include <format>            // C++20
#include <print>             // C++23

// Concurrency
#include <thread>
#include <mutex>
#include <atomic>
#include <future>

// I/O
#include <fstream>
#include <sstream>
#include <filesystem>        // C++17
```

---

**Keep this cheat sheet handy while coding! 📖**
