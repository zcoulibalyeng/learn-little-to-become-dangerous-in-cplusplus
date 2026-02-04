# Module 2: Fundamentals

## Data Types and Variables

### Fundamental Data Types

C and C++ provide several fundamental data types that map closely to hardware capabilities.

### Integer Types

```cpp
#include <iostream>
#include <cstdint>   // Fixed-width integers
#include <climits>   // Type limits

int main() {
    // Character types (also integers)
    char c = 'A';                    // At least 8 bits
    signed char sc = -128;           // -128 to 127
    unsigned char uc = 255;          // 0 to 255
    
    // Short integers
    short s = 32767;                 // At least 16 bits
    unsigned short us = 65535;
    
    // Integers
    int i = 2147483647;              // At least 16 bits (usually 32)
    unsigned int ui = 4294967295u;
    
    // Long integers
    long l = 2147483647L;            // At least 32 bits
    unsigned long ul = 4294967295UL;
    
    // Long long integers (C99/C++11)
    long long ll = 9223372036854775807LL;  // At least 64 bits
    unsigned long long ull = 18446744073709551615ULL;
    
    // Print sizes
    std::cout << "sizeof(char): " << sizeof(char) << " bytes\n";
    std::cout << "sizeof(short): " << sizeof(short) << " bytes\n";
    std::cout << "sizeof(int): " << sizeof(int) << " bytes\n";
    std::cout << "sizeof(long): " << sizeof(long) << " bytes\n";
    std::cout << "sizeof(long long): " << sizeof(long long) << " bytes\n";
    
    return 0;
}
```

### Fixed-Width Integer Types (Recommended)

```cpp
#include <cstdint>

int main() {
    // Exact width (guaranteed)
    int8_t i8 = 127;                 // Exactly 8 bits
    int16_t i16 = 32767;             // Exactly 16 bits
    int32_t i32 = 2147483647;        // Exactly 32 bits
    int64_t i64 = 9223372036854775807LL;  // Exactly 64 bits
    
    // Unsigned versions
    uint8_t u8 = 255;
    uint16_t u16 = 65535;
    uint32_t u32 = 4294967295U;
    uint64_t u64 = 18446744073709551615ULL;
    
    // Fastest types (at least N bits, fastest for platform)
    int_fast32_t fast = 42;
    
    // Smallest types (at least N bits, smallest size)
    int_least32_t small = 42;
    
    // Pointer-sized integers
    intptr_t ptr_sized = 0;
    uintptr_t uptr_sized = 0;
    
    // Maximum-width integers
    intmax_t max_int = INTMAX_MAX;
    
    return 0;
}
```

### C23: Bit-Precise Integers

```c
// C23 only
#include <stdio.h>

int main(void) {
    // Arbitrary-width integers
    _BitInt(8) b8 = 127;
    _BitInt(24) b24 = 8388607;       // Exactly 24 bits
    _BitInt(128) b128 = 0;           // 128-bit integer
    unsigned _BitInt(256) huge = 0;  // 256-bit unsigned
    
    printf("sizeof(_BitInt(128)): %zu\n", sizeof(b128));
    
    return 0;
}
```

### Floating-Point Types

```cpp
#include <iostream>
#include <cfloat>
#include <limits>
#include <numbers>  // C++20

int main() {
    // Standard floating-point types
    float f = 3.14159f;              // ~7 decimal digits precision
    double d = 3.141592653589793;    // ~15-17 decimal digits
    long double ld = 3.141592653589793238L;  // Extended precision
    
    // Print precision info
    std::cout << "float digits: " << std::numeric_limits<float>::digits10 << "\n";
    std::cout << "double digits: " << std::numeric_limits<double>::digits10 << "\n";
    
    // C++20 mathematical constants
    constexpr double pi = std::numbers::pi;
    constexpr double e = std::numbers::e;
    constexpr double phi = std::numbers::phi;  // Golden ratio
    
    // Special values
    double inf = std::numeric_limits<double>::infinity();
    double nan = std::numeric_limits<double>::quiet_NaN();
    
    std::cout << "Infinity: " << inf << "\n";
    std::cout << "NaN: " << nan << "\n";
    
    return 0;
}
```

### Boolean Type

```cpp
#include <iostream>

int main() {
    // C++: bool is a fundamental type
    bool flag = true;
    bool active = false;
    
    // Implicit conversions
    bool from_int = 42;      // true (non-zero)
    bool from_zero = 0;      // false
    bool from_ptr = nullptr; // false
    
    int to_int = flag;       // 1
    
    std::cout << std::boolalpha;  // Print "true"/"false" instead of 1/0
    std::cout << "flag: " << flag << "\n";
    std::cout << "active: " << active << "\n";
    
    return 0;
}
```

```c
// C23: bool, true, false are keywords
#include <stdio.h>

int main(void) {
    bool flag = true;
    bool active = false;
    
    printf("flag: %d\n", flag);     // 1
    printf("active: %d\n", active); // 0
    
    return 0;
}
```

### Character Types

```cpp
#include <iostream>

int main() {
    // Basic character
    char c = 'A';
    char newline = '\n';
    char tab = '\t';
    char backslash = '\\';
    char quote = '\"';
    
    // Wide character
    wchar_t wc = L'Ω';
    
    // C++11 Unicode characters
    char8_t u8c = u8'A';     // UTF-8 code unit
    char16_t u16c = u'A';    // UTF-16 code unit
    char32_t u32c = U'😀';   // UTF-32 code unit (full Unicode)
    
    // Character as number
    char digit = '0' + 5;    // '5'
    int ascii = 'A';         // 65
    
    std::cout << "Character: " << c << ", ASCII: " << static_cast<int>(c) << "\n";
    
    return 0;
}
```

## Variable Declaration and Initialization

### Declaration Styles

```cpp
#include <iostream>
#include <vector>
#include <string>

int main() {
    // C-style initialization
    int a = 42;
    
    // Constructor-style (C++)
    int b(42);
    
    // Uniform initialization (C++11) - RECOMMENDED
    int c{42};
    int d{};         // Zero-initialized
    
    // auto type deduction (C++11)
    auto e = 42;     // int
    auto f = 3.14;   // double
    auto g = "hello"; // const char*
    auto h = std::string{"hello"}; // std::string
    
    // Multiple declarations (avoid for readability)
    int x = 1, y = 2, z = 3;
    
    // Constants
    const int MAX_SIZE = 100;
    constexpr int BUFFER_SIZE = 256;  // Compile-time constant
    
    // Uninitialized (DANGEROUS - contains garbage)
    int uninitialized;  // Don't do this!
    
    // Structured bindings (C++17)
    auto [first, second] = std::pair{1, 2};
    
    return 0;
}
```

### Initialization Pitfalls

```cpp
#include <iostream>
#include <vector>

int main() {
    // Narrowing conversions - uniform init catches errors
    // int x{3.14};  // ERROR: narrowing conversion
    int x = 3.14;    // WARNING only, x = 3
    
    // Most vexing parse
    // std::vector<int> v();  // Declares a function, not a vector!
    std::vector<int> v{};     // Creates empty vector
    std::vector<int> v2(10);  // Vector with 10 elements
    std::vector<int> v3{10};  // Vector with one element: 10
    
    std::cout << "v2.size(): " << v2.size() << "\n";  // 10
    std::cout << "v3.size(): " << v3.size() << "\n";  // 1
    
    return 0;
}
```

## Operators and Expressions

### Arithmetic Operators

```cpp
#include <iostream>

int main() {
    int a = 17, b = 5;
    
    std::cout << "a + b = " << (a + b) << "\n";   // 22
    std::cout << "a - b = " << (a - b) << "\n";   // 12
    std::cout << "a * b = " << (a * b) << "\n";   // 85
    std::cout << "a / b = " << (a / b) << "\n";   // 3 (integer division)
    std::cout << "a % b = " << (a % b) << "\n";   // 2 (remainder)
    
    // Floating-point division
    double c = 17.0, d = 5.0;
    std::cout << "c / d = " << (c / d) << "\n";   // 3.4
    
    // Compound assignment
    int x = 10;
    x += 5;   // x = x + 5 = 15
    x -= 3;   // x = x - 3 = 12
    x *= 2;   // x = x * 2 = 24
    x /= 4;   // x = x / 4 = 6
    x %= 4;   // x = x % 4 = 2
    
    // Increment/Decrement
    int i = 5;
    std::cout << "i++: " << i++ << "\n";  // 5 (post-increment)
    std::cout << "i: " << i << "\n";      // 6
    std::cout << "++i: " << ++i << "\n";  // 7 (pre-increment)
    
    // Unary minus and plus
    int neg = -a;
    int pos = +a;
    
    return 0;
}
```

### Comparison Operators

```cpp
#include <iostream>
#include <compare>  // C++20

int main() {
    int a = 10, b = 20;
    
    std::cout << std::boolalpha;
    std::cout << "a == b: " << (a == b) << "\n";  // false
    std::cout << "a != b: " << (a != b) << "\n";  // true
    std::cout << "a < b: " << (a < b) << "\n";    // true
    std::cout << "a > b: " << (a > b) << "\n";    // false
    std::cout << "a <= b: " << (a <= b) << "\n";  // true
    std::cout << "a >= b: " << (a >= b) << "\n";  // false
    
    // C++20 Three-way comparison (spaceship operator)
    auto result = a <=> b;
    if (result < 0) {
        std::cout << "a is less than b\n";
    } else if (result > 0) {
        std::cout << "a is greater than b\n";
    } else {
        std::cout << "a is equal to b\n";
    }
    
    return 0;
}
```

### Logical Operators

```cpp
#include <iostream>

int main() {
    bool a = true, b = false;
    
    std::cout << std::boolalpha;
    std::cout << "a && b: " << (a && b) << "\n";  // false (AND)
    std::cout << "a || b: " << (a || b) << "\n";  // true (OR)
    std::cout << "!a: " << (!a) << "\n";          // false (NOT)
    
    // Short-circuit evaluation
    int x = 0;
    if (x != 0 && 10 / x > 0) {
        // Second condition not evaluated if first is false
        // This prevents division by zero
    }
    
    // Chained comparisons (be careful!)
    int n = 5;
    // if (0 < n < 10)  // WRONG! Always true in C/C++
    if (0 < n && n < 10) {  // Correct
        std::cout << "n is between 0 and 10\n";
    }
    
    return 0;
}
```

### Bitwise Operators

```cpp
#include <iostream>
#include <bitset>

int main() {
    uint8_t a = 0b11001100;  // 204
    uint8_t b = 0b10101010;  // 170
    
    std::cout << "a:     " << std::bitset<8>(a) << "\n";
    std::cout << "b:     " << std::bitset<8>(b) << "\n";
    std::cout << "a & b: " << std::bitset<8>(a & b) << "\n";   // AND: 10001000
    std::cout << "a | b: " << std::bitset<8>(a | b) << "\n";   // OR:  11101110
    std::cout << "a ^ b: " << std::bitset<8>(a ^ b) << "\n";   // XOR: 01100110
    std::cout << "~a:    " << std::bitset<8>(~a) << "\n";      // NOT: 00110011
    
    // Bit shifting
    uint8_t c = 0b00000001;
    std::cout << "c << 3: " << std::bitset<8>(c << 3) << "\n"; // 00001000
    std::cout << "c << 7: " << std::bitset<8>(c << 7) << "\n"; // 10000000
    
    uint8_t d = 0b10000000;
    std::cout << "d >> 3: " << std::bitset<8>(d >> 3) << "\n"; // 00010000
    
    // Common bit operations
    uint8_t flags = 0;
    
    // Set bit 3
    flags |= (1 << 3);
    std::cout << "Set bit 3:   " << std::bitset<8>(flags) << "\n";
    
    // Clear bit 3
    flags &= ~(1 << 3);
    std::cout << "Clear bit 3: " << std::bitset<8>(flags) << "\n";
    
    // Toggle bit 5
    flags ^= (1 << 5);
    std::cout << "Toggle bit 5: " << std::bitset<8>(flags) << "\n";
    
    // Check bit 5
    if (flags & (1 << 5)) {
        std::cout << "Bit 5 is set\n";
    }
    
    return 0;
}
```

## Type Conversions

### Implicit Conversions

```cpp
#include <iostream>

int main() {
    // Integer promotion
    char c = 'A';
    int i = c;  // char promoted to int
    
    // Arithmetic conversions
    int a = 5;
    double b = 2.5;
    double result = a + b;  // a converted to double
    
    // Warning: narrowing
    double pi = 3.14159;
    int truncated = pi;  // 3 (fractional part lost)
    
    // Warning: sign conversion
    int negative = -1;
    unsigned int u = negative;  // Very large positive number!
    std::cout << "u: " << u << "\n";  // 4294967295 on 32-bit
    
    // Warning: overflow
    int8_t small = 127;
    small++;  // Undefined behavior! (overflow)
    
    return 0;
}
```

### Explicit Conversions (Casts)

```cpp
#include <iostream>

int main() {
    // C-style cast (avoid in C++)
    double d = 3.14;
    int i = (int)d;
    
    // C++ casts (preferred)
    
    // static_cast: compile-time type conversion
    double pi = 3.14159;
    int truncated = static_cast<int>(pi);
    
    // const_cast: remove/add const
    const int* cp = &i;
    int* p = const_cast<int*>(cp);  // Dangerous!
    
    // reinterpret_cast: bit-level reinterpretation
    int value = 0x12345678;
    char* bytes = reinterpret_cast<char*>(&value);
    
    // dynamic_cast: runtime polymorphic conversion
    // (requires polymorphic types - covered in OOP module)
    
    std::cout << "pi truncated: " << truncated << "\n";
    
    return 0;
}
```

### C23: typeof Operator

```c
// C23
#include <stdio.h>

int main(void) {
    int x = 42;
    typeof(x) y = 100;           // y is int
    typeof(x + 1.0) z = 3.14;    // z is double
    
    // typeof_unqual removes qualifiers
    const int cx = 10;
    typeof_unqual(cx) w = 20;    // w is int (not const int)
    
    printf("y: %d, z: %f, w: %d\n", y, z, w);
    
    return 0;
}
```

## Constants and Literals

### Literal Types

```cpp
#include <iostream>
#include <string>
using namespace std::string_literals;

int main() {
    // Integer literals
    int dec = 42;           // Decimal
    int oct = 052;          // Octal (starts with 0)
    int hex = 0x2A;         // Hexadecimal
    int bin = 0b101010;     // Binary (C++14/C23)
    
    // Suffixes
    unsigned int u = 42u;
    long l = 42L;
    long long ll = 42LL;
    unsigned long long ull = 42ULL;
    
    // Digit separators (C++14/C23)
    int million = 1'000'000;
    long long big = 0xFFFF'FFFF'FFFF'FFFFLL;
    int binary = 0b1010'1010'1010'1010;
    
    // Floating-point literals
    double d1 = 3.14;
    double d2 = 3.14e10;    // Scientific notation
    double d3 = 314e-2;     // 3.14
    float f = 3.14f;
    long double ld = 3.14L;
    
    // Character literals
    char c1 = 'A';
    char c2 = '\n';         // Newline
    char c3 = '\x41';       // Hex: 'A'
    char c4 = '\101';       // Octal: 'A'
    wchar_t wc = L'Ω';
    char32_t emoji = U'😀';
    
    // String literals
    const char* s1 = "Hello";
    const wchar_t* s2 = L"Wide string";
    const char8_t* s3 = u8"UTF-8 string";
    const char16_t* s4 = u"UTF-16 string";
    const char32_t* s5 = U"UTF-32 string";
    
    // Raw string literals (C++11)
    const char* raw = R"(Line 1
Line 2
Special chars: \n \t "quotes")";
    
    // C++14 string literal
    auto str = "Hello"s;    // std::string
    
    std::cout << "million: " << million << "\n";
    std::cout << "raw:\n" << raw << "\n";
    
    return 0;
}
```

### Constants

```cpp
#include <iostream>

// Preprocessor macro (avoid)
#define PI_MACRO 3.14159

// C-style constant
const double PI_CONST = 3.14159;

// C++11 constexpr (compile-time constant)
constexpr double PI = 3.14159;

// constexpr function
constexpr int square(int n) {
    return n * n;
}

// C++20 consteval (must be compile-time)
consteval int cube(int n) {
    return n * n * n;
}

// C++20 constinit (constant initialization)
constinit int global_value = 42;

int main() {
    // constexpr can be used at compile time
    constexpr int sq = square(5);  // Computed at compile time
    
    // Arrays require compile-time size
    int arr[square(3)];  // OK: size is 9
    
    // static_assert with constexpr
    static_assert(square(4) == 16, "Math is broken!");
    
    std::cout << "PI: " << PI << "\n";
    std::cout << "5 squared: " << sq << "\n";
    std::cout << "3 cubed: " << cube(3) << "\n";
    
    return 0;
}
```

## Input/Output Basics

### C++ I/O Streams

```cpp
#include <iostream>
#include <iomanip>
#include <string>

int main() {
    // Output
    std::cout << "Hello, World!\n";
    std::cout << "Value: " << 42 << ", Pi: " << 3.14 << "\n";
    
    // Input
    int age;
    std::cout << "Enter your age: ";
    std::cin >> age;
    
    std::string name;
    std::cout << "Enter your name: ";
    std::cin >> name;  // Reads until whitespace
    
    // Read entire line
    std::cin.ignore();  // Clear newline from buffer
    std::string full_name;
    std::cout << "Enter your full name: ";
    std::getline(std::cin, full_name);
    
    // Formatting
    double pi = 3.14159265359;
    std::cout << std::fixed << std::setprecision(2);
    std::cout << "Pi: " << pi << "\n";  // 3.14
    
    std::cout << std::setw(10) << std::right << 42 << "\n";  // Right-aligned
    std::cout << std::setw(10) << std::left << 42 << "\n";   // Left-aligned
    std::cout << std::setfill('0') << std::setw(5) << 42 << "\n";  // 00042
    
    // Hex, octal, binary output
    int num = 255;
    std::cout << "Dec: " << std::dec << num << "\n";
    std::cout << "Hex: " << std::hex << num << "\n";
    std::cout << "Oct: " << std::oct << num << "\n";
    
    return 0;
}
```

### C++23 std::print

```cpp
#include <print>
#include <string>

int main() {
    // Basic printing
    std::print("Hello, World!\n");
    std::println("Hello with newline!");
    
    // Formatted output
    std::println("Name: {}, Age: {}", "Alice", 30);
    
    // Positional arguments
    std::println("{1} is {0} years old", 30, "Alice");
    
    // Formatting specifiers
    double pi = 3.14159265359;
    std::println("Pi: {:.2f}", pi);           // 3.14
    std::println("Pi: {:10.4f}", pi);         // "    3.1416"
    
    int num = 255;
    std::println("Dec: {}", num);             // 255
    std::println("Hex: {:x}", num);           // ff
    std::println("Hex: {:X}", num);           // FF
    std::println("Hex: {:#x}", num);          // 0xff
    std::println("Bin: {:b}", num);           // 11111111
    std::println("Oct: {:o}", num);           // 377
    
    // Width and alignment
    std::println("{:>10}", 42);               // "        42"
    std::println("{:<10}", 42);               // "42        "
    std::println("{:^10}", 42);               // "    42    "
    std::println("{:0>5}", 42);               // "00042"
    
    return 0;
}
```

### C I/O

```c
#include <stdio.h>

int main(void) {
    // Output
    printf("Hello, World!\n");
    printf("Value: %d, Pi: %f\n", 42, 3.14);
    
    // Format specifiers
    printf("%d\n", 42);           // int
    printf("%u\n", 42u);          // unsigned int
    printf("%ld\n", 42L);         // long
    printf("%lld\n", 42LL);       // long long
    printf("%f\n", 3.14);         // double
    printf("%e\n", 3.14e10);      // Scientific
    printf("%c\n", 'A');          // char
    printf("%s\n", "Hello");      // string
    printf("%p\n", (void*)&main); // pointer
    printf("%x\n", 255);          // hex
    printf("%o\n", 255);          // octal
    printf("%b\n", 255);          // binary (C23)
    
    // Width and precision
    printf("%10d\n", 42);         // Width 10
    printf("%-10d\n", 42);        // Left-aligned
    printf("%010d\n", 42);        // Zero-padded
    printf("%.2f\n", 3.14159);    // 2 decimal places
    printf("%10.2f\n", 3.14159);  // Width 10, 2 decimals
    
    // Input
    int age;
    printf("Enter your age: ");
    scanf("%d", &age);
    
    char name[100];
    printf("Enter your name: ");
    scanf("%99s", name);  // Max 99 chars to prevent overflow
    
    // Read line (safer)
    char line[100];
    fgets(line, sizeof(line), stdin);
    
    return 0;
}
```

## Exercises

### Exercise 1: Data Type Sizes
Write a program that prints the size (in bytes) and range of all fundamental types.

```cpp
#include <iostream>
#include <limits>

int main() {
    std::cout << "Type\t\tSize\tMin\t\t\tMax\n";
    std::cout << "----\t\t----\t---\t\t\t---\n";
    
    // Your code here: print info for each type
    // Use sizeof() and std::numeric_limits
    
    return 0;
}
```

### Exercise 2: Temperature Converter
Write a program that converts between Celsius and Fahrenheit.

### Exercise 3: Bit Manipulation
Write functions to:
1. Set the Nth bit of a number
2. Clear the Nth bit
3. Toggle the Nth bit
4. Check if Nth bit is set

### Exercise 4: Number Base Converter
Write a program that reads a number and prints it in decimal, hexadecimal, octal, and binary.

### Exercise 5: Input Validation
Write a program that:
1. Reads an integer from the user
2. Handles invalid input gracefully
3. Keeps asking until valid input is received

## Next Steps

In the next module, we'll cover:
- Conditional statements
- Loops and iteration
- Functions and parameter passing
- Recursion

---

**Practice Tip:** Experiment with different data types and observe how they behave with various operations. Understanding types is fundamental to C/C++ programming!
