# Module 3: Control Flow & Functions

## Conditional Statements

### If-Else Statements

```cpp
#include <iostream>

int main() {
    int x = 10;
    
    // Simple if
    if (x > 0) {
        std::cout << "x is positive\n";
    }
    
    // If-else
    if (x % 2 == 0) {
        std::cout << "x is even\n";
    } else {
        std::cout << "x is odd\n";
    }
    
    // If-else if-else chain
    if (x < 0) {
        std::cout << "x is negative\n";
    } else if (x == 0) {
        std::cout << "x is zero\n";
    } else {
        std::cout << "x is positive\n";
    }
    
    // Nested if
    if (x > 0) {
        if (x < 100) {
            std::cout << "x is between 0 and 100\n";
        }
    }
    
    return 0;
}
```

### C++17 If with Initializer

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores = {{"Alice", 95}, {"Bob", 87}};
    
    // C++17: if with initializer
    if (auto it = scores.find("Alice"); it != scores.end()) {
        std::cout << "Alice's score: " << it->second << "\n";
    } else {
        std::cout << "Alice not found\n";
    }
    // 'it' is not accessible here
    
    // Useful for error checking
    if (int result = compute_something(); result >= 0) {
        std::cout << "Success: " << result << "\n";
    } else {
        std::cout << "Error: " << result << "\n";
    }
    
    return 0;
}

int compute_something() { return 42; }
```

### Ternary Operator

```cpp
#include <iostream>

int main() {
    int a = 10, b = 20;
    
    // Ternary operator: condition ? if_true : if_false
    int max = (a > b) ? a : b;
    std::cout << "Max: " << max << "\n";
    
    // Can be nested (but avoid for readability)
    int x = 5;
    std::string result = (x > 0) ? "positive" : (x < 0) ? "negative" : "zero";
    
    // Better: use if-else for complex conditions
    
    return 0;
}
```

### Switch Statement

```cpp
#include <iostream>

int main() {
    int day = 3;
    
    switch (day) {
        case 1:
            std::cout << "Monday\n";
            break;
        case 2:
            std::cout << "Tuesday\n";
            break;
        case 3:
            std::cout << "Wednesday\n";
            break;
        case 4:
            std::cout << "Thursday\n";
            break;
        case 5:
            std::cout << "Friday\n";
            break;
        case 6:
        case 7:
            std::cout << "Weekend!\n";
            break;
        default:
            std::cout << "Invalid day\n";
            break;
    }
    
    // C++17: switch with initializer
    switch (int n = get_number(); n) {
        case 1: std::cout << "One\n"; break;
        case 2: std::cout << "Two\n"; break;
        default: std::cout << "Other\n"; break;
    }
    
    return 0;
}

int get_number() { return 2; }
```

### C++17 [[fallthrough]] Attribute

```cpp
#include <iostream>

int main() {
    int level = 2;
    
    switch (level) {
        case 3:
            std::cout << "Level 3 bonus\n";
            [[fallthrough]];  // Explicit fallthrough
        case 2:
            std::cout << "Level 2 bonus\n";
            [[fallthrough]];
        case 1:
            std::cout << "Level 1 bonus\n";
            break;
        default:
            std::cout << "No bonus\n";
    }
    
    return 0;
}
```

### C++23 consteval if

```cpp
#include <iostream>

constexpr int factorial(int n) {
    if consteval {
        // Only executed at compile time
        // Can use consteval-only operations
    }
    
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int main() {
    constexpr int fact5 = factorial(5);  // Computed at compile time
    std::cout << "5! = " << fact5 << "\n";
    
    return 0;
}
```

## Loops

### While Loop

```cpp
#include <iostream>

int main() {
    // Basic while
    int i = 0;
    while (i < 5) {
        std::cout << i << " ";
        i++;
    }
    std::cout << "\n";
    
    // Infinite loop (use with break)
    int count = 0;
    while (true) {
        std::cout << "Count: " << count << "\n";
        count++;
        if (count >= 3) break;
    }
    
    return 0;
}
```

### Do-While Loop

```cpp
#include <iostream>

int main() {
    // Executes at least once
    int num;
    do {
        std::cout << "Enter a positive number: ";
        std::cin >> num;
    } while (num <= 0);
    
    std::cout << "You entered: " << num << "\n";
    
    return 0;
}
```

### For Loop

```cpp
#include <iostream>
#include <vector>

int main() {
    // Basic for loop
    for (int i = 0; i < 5; i++) {
        std::cout << i << " ";
    }
    std::cout << "\n";
    
    // Multiple variables
    for (int i = 0, j = 10; i < j; i++, j--) {
        std::cout << "i=" << i << ", j=" << j << "\n";
    }
    
    // Iterating backwards
    for (int i = 5; i > 0; i--) {
        std::cout << i << " ";
    }
    std::cout << "\n";
    
    // Infinite for loop
    // for (;;) { /* infinite */ }
    
    return 0;
}
```

### Range-Based For Loop (C++11)

```cpp
#include <iostream>
#include <vector>
#include <map>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // Read-only iteration
    for (int n : numbers) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    // Modify elements (use reference)
    for (int& n : numbers) {
        n *= 2;
    }
    
    // Efficient read-only (const reference)
    for (const int& n : numbers) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    // With auto
    for (const auto& n : numbers) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    // Iterating maps
    std::map<std::string, int> ages = {{"Alice", 30}, {"Bob", 25}};
    for (const auto& [name, age] : ages) {  // C++17 structured binding
        std::cout << name << " is " << age << "\n";
    }
    
    // C-style arrays
    int arr[] = {10, 20, 30};
    for (int x : arr) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Loop Control

```cpp
#include <iostream>

int main() {
    // break - exit loop immediately
    for (int i = 0; i < 10; i++) {
        if (i == 5) break;
        std::cout << i << " ";
    }
    std::cout << "\n";  // 0 1 2 3 4
    
    // continue - skip to next iteration
    for (int i = 0; i < 10; i++) {
        if (i % 2 == 0) continue;
        std::cout << i << " ";
    }
    std::cout << "\n";  // 1 3 5 7 9
    
    // Nested loops with labeled break (using goto - avoid if possible)
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (i == 1 && j == 1) goto done;
            std::cout << "(" << i << "," << j << ") ";
        }
    }
    done:
    std::cout << "\n";
    
    // Better: use a flag or extract to function
    bool found = false;
    for (int i = 0; i < 3 && !found; i++) {
        for (int j = 0; j < 3 && !found; j++) {
            if (i == 1 && j == 1) {
                found = true;
            }
        }
    }
    
    return 0;
}
```

## Functions

### Function Declaration and Definition

```cpp
#include <iostream>

// Function declaration (prototype)
int add(int a, int b);
void greet(const std::string& name);
double average(double x, double y);

int main() {
    std::cout << "Sum: " << add(3, 4) << "\n";
    greet("World");
    std::cout << "Average: " << average(10.0, 20.0) << "\n";
    
    return 0;
}

// Function definitions
int add(int a, int b) {
    return a + b;
}

void greet(const std::string& name) {
    std::cout << "Hello, " << name << "!\n";
}

double average(double x, double y) {
    return (x + y) / 2.0;
}
```

### Parameter Passing

```cpp
#include <iostream>
#include <vector>

// Pass by value (copy)
void by_value(int x) {
    x = 100;  // Only modifies local copy
}

// Pass by reference
void by_reference(int& x) {
    x = 100;  // Modifies original
}

// Pass by const reference (efficient, read-only)
void by_const_ref(const std::vector<int>& vec) {
    // Can read but not modify
    for (int n : vec) {
        std::cout << n << " ";
    }
    std::cout << "\n";
}

// Pass by pointer
void by_pointer(int* x) {
    if (x != nullptr) {
        *x = 100;  // Modifies original
    }
}

// C++11: Pass by rvalue reference (move semantics)
void by_rvalue_ref(std::vector<int>&& vec) {
    // Takes ownership of vec
    std::vector<int> local = std::move(vec);
}

int main() {
    int a = 10;
    
    by_value(a);
    std::cout << "After by_value: " << a << "\n";  // 10
    
    by_reference(a);
    std::cout << "After by_reference: " << a << "\n";  // 100
    
    a = 10;
    by_pointer(&a);
    std::cout << "After by_pointer: " << a << "\n";  // 100
    
    std::vector<int> vec = {1, 2, 3};
    by_const_ref(vec);
    
    return 0;
}
```

### Default Parameters

```cpp
#include <iostream>
#include <string>

void greet(const std::string& name = "World", int times = 1) {
    for (int i = 0; i < times; i++) {
        std::cout << "Hello, " << name << "!\n";
    }
}

// Default parameters must be rightmost
// void bad(int a = 1, int b);  // ERROR

int main() {
    greet();                    // "Hello, World!" once
    greet("Alice");            // "Hello, Alice!" once
    greet("Bob", 3);           // "Hello, Bob!" three times
    
    return 0;
}
```

### Function Overloading (C++)

```cpp
#include <iostream>
#include <string>

// Same name, different parameters
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}

std::string add(const std::string& a, const std::string& b) {
    return a + b;
}

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    std::cout << add(1, 2) << "\n";           // int version
    std::cout << add(1.5, 2.5) << "\n";       // double version
    std::cout << add("Hello, ", "World!") << "\n";  // string version
    std::cout << add(1, 2, 3) << "\n";        // three-arg version
    
    return 0;
}
```

### Return Types

```cpp
#include <iostream>
#include <tuple>
#include <optional>

// Single return value
int square(int x) {
    return x * x;
}

// Return reference (be careful with lifetime!)
int& get_element(int arr[], int index) {
    return arr[index];
}

// C++14: Return type deduction
auto multiply(int a, int b) {
    return a * b;  // Deduced as int
}

// C++11: Trailing return type
auto divide(int a, int b) -> double {
    return static_cast<double>(a) / b;
}

// Multiple return values with tuple
std::tuple<int, int, int> get_rgb(int color) {
    int r = (color >> 16) & 0xFF;
    int g = (color >> 8) & 0xFF;
    int b = color & 0xFF;
    return {r, g, b};
}

// C++17: Optional return
std::optional<int> safe_divide(int a, int b) {
    if (b == 0) return std::nullopt;
    return a / b;
}

int main() {
    // Structured binding with tuple (C++17)
    auto [r, g, b] = get_rgb(0xFF8040);
    std::cout << "R=" << r << " G=" << g << " B=" << b << "\n";
    
    // Using optional
    if (auto result = safe_divide(10, 2)) {
        std::cout << "Result: " << *result << "\n";
    }
    
    if (auto result = safe_divide(10, 0)) {
        std::cout << "Result: " << *result << "\n";
    } else {
        std::cout << "Division by zero!\n";
    }
    
    return 0;
}
```

## Recursion

### Basic Recursion

```cpp
#include <iostream>

// Factorial: n! = n * (n-1) * ... * 1
int factorial(int n) {
    // Base case
    if (n <= 1) return 1;
    
    // Recursive case
    return n * factorial(n - 1);
}

// Fibonacci: F(n) = F(n-1) + F(n-2)
int fibonacci(int n) {
    if (n <= 0) return 0;
    if (n == 1) return 1;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// Sum of array
int sum_array(const int arr[], int size) {
    if (size == 0) return 0;
    return arr[size - 1] + sum_array(arr, size - 1);
}

int main() {
    std::cout << "5! = " << factorial(5) << "\n";       // 120
    std::cout << "F(10) = " << fibonacci(10) << "\n";   // 55
    
    int arr[] = {1, 2, 3, 4, 5};
    std::cout << "Sum: " << sum_array(arr, 5) << "\n";  // 15
    
    return 0;
}
```

### Tail Recursion

```cpp
#include <iostream>

// Regular recursion (stack grows)
int factorial_regular(int n) {
    if (n <= 1) return 1;
    return n * factorial_regular(n - 1);
}

// Tail recursion (can be optimized by compiler)
int factorial_tail(int n, int accumulator = 1) {
    if (n <= 1) return accumulator;
    return factorial_tail(n - 1, n * accumulator);  // Tail call
}

// Iterative (most efficient)
int factorial_iterative(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

int main() {
    std::cout << "10! (regular): " << factorial_regular(10) << "\n";
    std::cout << "10! (tail): " << factorial_tail(10) << "\n";
    std::cout << "10! (iterative): " << factorial_iterative(10) << "\n";
    
    return 0;
}
```

## Lambda Expressions (C++11)

### Basic Lambdas

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    // Basic lambda
    auto greet = []() {
        std::cout << "Hello!\n";
    };
    greet();
    
    // Lambda with parameters
    auto add = [](int a, int b) {
        return a + b;
    };
    std::cout << "Sum: " << add(3, 4) << "\n";
    
    // Lambda with explicit return type
    auto divide = [](int a, int b) -> double {
        return static_cast<double>(a) / b;
    };
    
    // Using with algorithms
    std::vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6};
    
    std::sort(nums.begin(), nums.end(), [](int a, int b) {
        return a > b;  // Descending order
    });
    
    for (int n : nums) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Lambda Captures

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int multiplier = 3;
    std::vector<int> nums = {1, 2, 3, 4, 5};
    
    // Capture by value [=] or specific [multiplier]
    auto times = [multiplier](int x) {
        return x * multiplier;
    };
    
    // Capture by reference [&] or specific [&multiplier]
    auto increment_mult = [&multiplier]() {
        multiplier++;
    };
    
    // Mixed captures
    int a = 1, b = 2;
    auto mixed = [a, &b]() {
        // a is captured by value (copy)
        // b is captured by reference
        b = a + 10;
    };
    
    // Capture all by value
    auto all_by_value = [=]() {
        return multiplier + a + b;
    };
    
    // Capture all by reference
    auto all_by_ref = [&]() {
        multiplier = 100;
    };
    
    // C++14: Init capture
    auto init_capture = [x = multiplier * 2]() {
        return x;
    };
    
    // Mutable lambda (can modify captured values)
    int count = 0;
    auto counter = [count]() mutable {
        return ++count;  // Modifies the lambda's copy
    };
    std::cout << counter() << " " << counter() << "\n";  // 1 2
    std::cout << "Original count: " << count << "\n";    // 0
    
    return 0;
}
```

### Generic Lambdas (C++14)

```cpp
#include <iostream>
#include <string>

int main() {
    // C++14: auto parameters
    auto print = [](const auto& value) {
        std::cout << value << "\n";
    };
    
    print(42);
    print(3.14);
    print("Hello");
    
    // Generic lambda for any container
    auto print_all = [](const auto& container) {
        for (const auto& item : container) {
            std::cout << item << " ";
        }
        std::cout << "\n";
    };
    
    std::vector<int> nums = {1, 2, 3};
    std::string str = "Hello";
    
    print_all(nums);
    print_all(str);
    
    return 0;
}
```

### C++20 Template Lambdas

```cpp
#include <iostream>
#include <concepts>

int main() {
    // C++20: Template syntax in lambdas
    auto add = []<typename T>(T a, T b) {
        return a + b;
    };
    
    std::cout << add(1, 2) << "\n";
    std::cout << add(1.5, 2.5) << "\n";
    
    // With concepts
    auto numeric_add = []<typename T>(T a, T b) 
        requires std::integral<T> || std::floating_point<T>
    {
        return a + b;
    };
    
    return 0;
}
```

## Exercises

### Exercise 1: FizzBuzz
Write a function that prints numbers 1 to N, but prints "Fizz" for multiples of 3, "Buzz" for multiples of 5, and "FizzBuzz" for multiples of both.

### Exercise 2: Prime Checker
Write both iterative and recursive functions to check if a number is prime.

### Exercise 3: Binary Search
Implement binary search both iteratively and recursively.

### Exercise 4: Calculator with Functions
Create a calculator that uses function overloading for different operations and number types.

### Exercise 5: Lambda Sorter
Use lambdas to sort a vector of strings by:
1. Length
2. Alphabetically (case-insensitive)
3. Number of vowels

## Next Steps

In the next module, we'll cover:
- Understanding pointers
- Pointer arithmetic
- Dynamic memory allocation
- Memory safety
- Smart pointers introduction

---

**Practice Tip:** Master functions and lambdas—they are the building blocks of modular, maintainable code!
