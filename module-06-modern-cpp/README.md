# Module 6: Modern C++ Features

## C++11/14/17/20 Essentials

### C++11 Core Features

```cpp
#include <iostream>
#include <vector>
#include <memory>

int main() {
    // 1. auto type deduction
    auto x = 42;              // int
    auto y = 3.14;           // double
    auto vec = std::vector<int>{1, 2, 3};
    
    // 2. Range-based for loop
    for (const auto& v : vec) {
        std::cout << v << " ";
    }
    std::cout << "\n";
    
    // 3. Uniform initialization
    int arr[]{1, 2, 3};
    std::vector<int> v{1, 2, 3};
    
    // 4. nullptr
    int* ptr = nullptr;
    
    // 5. Lambda expressions
    auto add = [](int a, int b) { return a + b; };
    std::cout << add(3, 4) << "\n";
    
    // 6. Smart pointers
    auto sp = std::make_shared<int>(42);
    auto up = std::unique_ptr<int>(new int(42));  // make_unique in C++14
    
    // 7. Move semantics
    std::string s1 = "Hello";
    std::string s2 = std::move(s1);  // s1 is now empty
    
    // 8. constexpr
    constexpr int square(int n) { return n * n; }
    constexpr int sq = square(5);
    
    return 0;
}
```

### C++14 Features

```cpp
#include <iostream>
#include <memory>

int main() {
    // 1. Generic lambdas
    auto print = [](const auto& x) { std::cout << x << "\n"; };
    print(42);
    print("Hello");
    
    // 2. Return type deduction
    auto add(int a, int b) { return a + b; }
    
    // 3. make_unique
    auto ptr = std::make_unique<int>(42);
    
    // 4. Binary literals and digit separators
    int binary = 0b1010'1010;
    int million = 1'000'000;
    
    // 5. [[deprecated]] attribute
    // [[deprecated("Use newFunc instead")]]
    // void oldFunc() {}
    
    return 0;
}
```

### C++17 Features

```cpp
#include <iostream>
#include <optional>
#include <variant>
#include <string_view>
#include <filesystem>
#include <tuple>

int main() {
    // 1. Structured bindings
    auto [a, b, c] = std::tuple{1, 2.0, "three"};
    
    // 2. if/switch with initializer
    if (auto x = compute(); x > 0) {
        std::cout << "Positive: " << x << "\n";
    }
    
    // 3. std::optional
    std::optional<int> opt = 42;
    if (opt) {
        std::cout << "Value: " << *opt << "\n";
    }
    auto empty = std::optional<int>{};
    std::cout << "Has value: " << empty.has_value() << "\n";
    
    // 4. std::variant
    std::variant<int, double, std::string> var = "Hello";
    std::cout << std::get<std::string>(var) << "\n";
    
    // 5. std::string_view (non-owning string reference)
    std::string_view sv = "Hello, World!";
    std::cout << sv.substr(0, 5) << "\n";
    
    // 6. if constexpr
    auto process = []<typename T>(T value) {
        if constexpr (std::is_integral_v<T>) {
            return value * 2;
        } else {
            return value;
        }
    };
    
    // 7. Fold expressions
    auto sum = [](auto... args) {
        return (args + ...);  // Unary right fold
    };
    std::cout << sum(1, 2, 3, 4, 5) << "\n";
    
    // 8. std::filesystem
    namespace fs = std::filesystem;
    std::cout << "Current path: " << fs::current_path() << "\n";
    
    return 0;
}

int compute() { return 42; }
```

### C++20 Features

```cpp
#include <iostream>
#include <vector>
#include <ranges>
#include <concepts>
#include <span>
#include <format>
#include <numbers>
#include <compare>
#include <coroutine>

// 1. Concepts
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<Numeric T>
T square(T x) {
    return x * x;
}

// 2. Ranges
void ranges_demo() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // Filter and transform with pipes
    auto result = nums 
        | std::views::filter([](int n) { return n % 2 == 0; })
        | std::views::transform([](int n) { return n * n; });
    
    for (int n : result) {
        std::cout << n << " ";  // 4 16 36 64 100
    }
    std::cout << "\n";
}

// 3. Three-way comparison (spaceship operator)
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;
};

// 4. Modules (conceptual - requires build system support)
// export module mymodule;
// export int add(int a, int b) { return a + b; }

int main() {
    // Concepts
    std::cout << square(5) << "\n";
    std::cout << square(3.14) << "\n";
    
    // Ranges
    ranges_demo();
    
    // std::format
    std::string msg = std::format("Hello, {}! You are {} years old.", "Alice", 30);
    std::cout << msg << "\n";
    
    // std::span (non-owning view of contiguous data)
    int arr[] = {1, 2, 3, 4, 5};
    std::span<int> sp(arr);
    for (int n : sp) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    // Mathematical constants
    std::cout << "Pi: " << std::numbers::pi << "\n";
    std::cout << "E: " << std::numbers::e << "\n";
    
    // Three-way comparison
    Point p1{1, 2}, p2{1, 3};
    auto cmp = p1 <=> p2;
    if (cmp < 0) std::cout << "p1 < p2\n";
    
    return 0;
}
```

## C++23 New Features

```cpp
#include <iostream>
#include <print>
#include <expected>
#include <mdspan>
#include <generator>
#include <ranges>
#include <flat_map>

// 1. std::print and std::println
void print_demo() {
    std::println("Hello, World!");
    std::print("Value: {}\n", 42);
    std::println("Name: {}, Age: {}", "Alice", 30);
    std::println("Hex: {:#x}, Binary: {:b}", 255, 42);
}

// 2. std::expected - better error handling
std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) {
        return std::unexpected("Division by zero");
    }
    return a / b;
}

// 3. Deducing this
struct Widget {
    int value = 42;
    
    // Single function instead of const/non-const overloads
    template<typename Self>
    auto&& getValue(this Self&& self) {
        return std::forward<Self>(self).value;
    }
};

// 4. std::generator (coroutine-based)
std::generator<int> fibonacci(int n) {
    int a = 0, b = 1;
    for (int i = 0; i < n; ++i) {
        co_yield a;
        auto next = a + b;
        a = b;
        b = next;
    }
}

// 5. Multidimensional subscript operator
struct Matrix {
    double data[3][3];
    
    double& operator[](int i, int j) {
        return data[i][j];
    }
};

// 6. if consteval
constexpr int factorial(int n) {
    if consteval {
        // Compile-time only code
    }
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// 7. static operator()
struct Adder {
    static int operator()(int a, int b) {
        return a + b;
    }
};

int main() {
    // std::print
    print_demo();
    
    // std::expected
    auto result = divide(10, 2);
    if (result) {
        std::println("Result: {}", *result);
    }
    
    auto error = divide(10, 0);
    if (!error) {
        std::println("Error: {}", error.error());
    }
    
    // std::flat_map (faster for small maps)
    std::flat_map<int, std::string> fm;
    fm[1] = "one";
    fm[2] = "two";
    
    // std::mdspan
    std::vector<int> data(12);
    std::mdspan<int, std::extents<size_t, 3, 4>> matrix(data.data());
    matrix[1, 2] = 42;
    
    // Generator
    for (int fib : fibonacci(10)) {
        std::print("{} ", fib);
    }
    std::println("");
    
    // Static operator()
    std::println("Adder: {}", Adder{}(3, 4));
    
    return 0;
}
```

## C++26 Preview

```cpp
#include <iostream>

// 1. Reflection (P2996) - compile-time introspection
/*
struct Point {
    int x, y;
};

// Get member names at compile time
constexpr auto names = std::meta::members_of(^Point);
for (auto name : names) {
    std::println("{}", std::meta::name_of(name));
}
*/

// 2. Contracts (P2900) - preconditions and postconditions
/*
int divide(int a, int b)
    pre (b != 0)
    post (r: r * b == a)
{
    return a / b;
}
*/

// 3. Pattern Matching (proposed)
/*
int describe(auto value) {
    return inspect(value) {
        0 => "zero";
        1 => "one";
        x if x < 0 => "negative";
        _ => "other";
    };
}
*/

// 4. std::simd - data-parallel types
/*
#include <simd>
std::simd<float, 4> a = {1.0f, 2.0f, 3.0f, 4.0f};
std::simd<float, 4> b = {5.0f, 6.0f, 7.0f, 8.0f};
auto c = a + b;  // SIMD addition
*/

// 5. Pack indexing
/*
template<typename... Ts>
auto get_second(Ts... args) {
    return args...[1];  // Get second element of pack
}
*/

int main() {
    std::cout << "C++26 features are still in development!\n";
    std::cout << "Key features expected:\n";
    std::cout << "- Reflection\n";
    std::cout << "- Contracts\n";
    std::cout << "- std::simd\n";
    std::cout << "- Pack indexing\n";
    std::cout << "- Pattern matching (possible)\n";
    
    return 0;
}
```

## Move Semantics Deep Dive

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <utility>

class HeavyResource {
public:
    HeavyResource() : data_(new int[1000]) {
        std::cout << "Default constructor\n";
    }
    
    // Copy constructor - expensive
    HeavyResource(const HeavyResource& other) : data_(new int[1000]) {
        std::copy(other.data_, other.data_ + 1000, data_);
        std::cout << "Copy constructor (expensive)\n";
    }
    
    // Move constructor - cheap
    HeavyResource(HeavyResource&& other) noexcept : data_(other.data_) {
        other.data_ = nullptr;
        std::cout << "Move constructor (cheap)\n";
    }
    
    // Copy assignment
    HeavyResource& operator=(const HeavyResource& other) {
        if (this != &other) {
            delete[] data_;
            data_ = new int[1000];
            std::copy(other.data_, other.data_ + 1000, data_);
            std::cout << "Copy assignment (expensive)\n";
        }
        return *this;
    }
    
    // Move assignment
    HeavyResource& operator=(HeavyResource&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            other.data_ = nullptr;
            std::cout << "Move assignment (cheap)\n";
        }
        return *this;
    }
    
    ~HeavyResource() {
        delete[] data_;
        std::cout << "Destructor\n";
    }

private:
    int* data_;
};

// Perfect forwarding
template<typename T>
void wrapper(T&& arg) {
    process(std::forward<T>(arg));
}

void process(HeavyResource& r) {
    std::cout << "Processing lvalue\n";
}

void process(HeavyResource&& r) {
    std::cout << "Processing rvalue\n";
}

int main() {
    std::cout << "=== Creating resource ===\n";
    HeavyResource r1;
    
    std::cout << "\n=== Copy ===\n";
    HeavyResource r2 = r1;           // Copy
    
    std::cout << "\n=== Move ===\n";
    HeavyResource r3 = std::move(r1); // Move
    
    std::cout << "\n=== Vector operations ===\n";
    std::vector<HeavyResource> vec;
    vec.reserve(3);  // Prevent reallocation
    
    HeavyResource temp;
    vec.push_back(temp);             // Copy
    vec.push_back(std::move(temp));  // Move
    vec.push_back(HeavyResource());  // Move (temporary)
    
    std::cout << "\n=== Perfect forwarding ===\n";
    HeavyResource r4;
    wrapper(r4);              // lvalue
    wrapper(HeavyResource()); // rvalue
    
    std::cout << "\n=== End ===\n";
    return 0;
}
```

## Advanced Lambda Expressions

```cpp
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>

int main() {
    // 1. Immediately Invoked Lambda Expression (IILE)
    const int value = []() {
        int result = 0;
        for (int i = 1; i <= 10; ++i) {
            result += i;
        }
        return result;
    }();
    std::cout << "Sum 1-10: " << value << "\n";
    
    // 2. Recursive lambda (C++14)
    auto factorial = [](auto&& self, int n) -> int {
        if (n <= 1) return 1;
        return n * self(self, n - 1);
    };
    std::cout << "5! = " << factorial(factorial, 5) << "\n";
    
    // 3. C++23: Simpler recursive lambda with deducing this
    /*
    auto fib = [](this auto&& self, int n) -> int {
        if (n <= 1) return n;
        return self(n - 1) + self(n - 2);
    };
    */
    
    // 4. Lambda with state
    auto counter = [count = 0]() mutable {
        return ++count;
    };
    std::cout << counter() << counter() << counter() << "\n";  // 123
    
    // 5. Lambda returning lambda
    auto make_multiplier = [](int factor) {
        return [factor](int x) { return x * factor; };
    };
    auto times3 = make_multiplier(3);
    std::cout << "5 * 3 = " << times3(5) << "\n";
    
    // 6. Lambda with move capture (C++14)
    auto ptr = std::make_unique<int>(42);
    auto lambda = [p = std::move(ptr)]() {
        return *p;
    };
    std::cout << "Captured: " << lambda() << "\n";
    
    // 7. Generic lambda with concepts (C++20)
    auto add_numeric = []<typename T>(T a, T b) 
        requires std::integral<T> || std::floating_point<T>
    {
        return a + b;
    };
    
    // 8. Lambda in constexpr context (C++17)
    constexpr auto sq = [](int n) constexpr { return n * n; };
    static_assert(sq(5) == 25);
    
    return 0;
}
```

## Exercises

### Exercise 1: Concepts Practice
Create concepts for: Printable, Container, and Hashable types.

### Exercise 2: Ranges Pipeline
Use C++20 ranges to filter, transform, and sort a vector.

### Exercise 3: Move-Only Type
Create a move-only resource manager class.

### Exercise 4: Expected Error Handling
Refactor a function that uses exceptions to use std::expected.

## Next Steps

In the next module, we'll cover:
- STL Containers
- Algorithms
- Ranges library
- std::format and std::print

---

**Practice Tip:** Try the new features in a compiler that supports them. Compiler Explorer (godbolt.org) is great for this!
