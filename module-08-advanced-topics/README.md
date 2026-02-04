# Module 8: Advanced Topics

## Templates and Generic Programming

### Function Templates

```cpp
#include <iostream>
#include <string>

// Basic function template
template<typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

// Multiple type parameters
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

// C++20: Abbreviated function template
auto multiply(auto a, auto b) {
    return a * b;
}

// Non-type template parameter
template<int N>
void print_n_times(const std::string& msg) {
    for (int i = 0; i < N; ++i) {
        std::cout << msg << "\n";
    }
}

// Template specialization
template<typename T>
void print(T value) {
    std::cout << "Generic: " << value << "\n";
}

template<>
void print<bool>(bool value) {
    std::cout << "Bool: " << (value ? "true" : "false") << "\n";
}

int main() {
    std::cout << max(3, 5) << "\n";           // int
    std::cout << max(3.14, 2.71) << "\n";     // double
    std::cout << max<double>(3, 3.14) << "\n"; // Explicit type
    
    std::cout << add(1, 2.5) << "\n";         // double
    std::cout << multiply(3, 4.5) << "\n";    // double
    
    print_n_times<3>("Hello");
    
    print(42);
    print(true);
    
    return 0;
}
```

### Class Templates

```cpp
#include <iostream>
#include <stdexcept>

template<typename T, size_t Size>
class Array {
public:
    T& operator[](size_t index) {
        if (index >= Size) throw std::out_of_range("Index out of bounds");
        return data_[index];
    }
    
    const T& operator[](size_t index) const {
        if (index >= Size) throw std::out_of_range("Index out of bounds");
        return data_[index];
    }
    
    constexpr size_t size() const { return Size; }
    
    T* begin() { return data_; }
    T* end() { return data_ + Size; }
    const T* begin() const { return data_; }
    const T* end() const { return data_ + Size; }

private:
    T data_[Size];
};

// Partial specialization
template<typename T>
class Array<T, 0> {
public:
    constexpr size_t size() const { return 0; }
};

int main() {
    Array<int, 5> arr;
    for (size_t i = 0; i < arr.size(); ++i) {
        arr[i] = i * 10;
    }
    
    for (int n : arr) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### Variadic Templates (C++11)

```cpp
#include <iostream>
#include <sstream>

// Recursive variadic template
template<typename T>
void print(T value) {
    std::cout << value << "\n";
}

template<typename T, typename... Args>
void print(T first, Args... rest) {
    std::cout << first << " ";
    print(rest...);
}

// C++17 fold expressions
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // Unary right fold
}

template<typename... Args>
auto sum_init(Args... args) {
    return (0 + ... + args);  // Binary left fold
}

template<typename... Args>
void print_all(Args... args) {
    ((std::cout << args << " "), ...);  // Comma fold
    std::cout << "\n";
}

// sizeof... operator
template<typename... Args>
constexpr size_t count_args(Args... args) {
    return sizeof...(args);
}

int main() {
    print(1, 2, 3, "hello", 3.14);
    
    std::cout << "Sum: " << sum(1, 2, 3, 4, 5) << "\n";
    
    print_all(1, "two", 3.0, "four");
    
    std::cout << "Count: " << count_args(1, 2, 3, 4) << "\n";
    
    return 0;
}
```

### Concepts (C++20)

```cpp
#include <iostream>
#include <concepts>
#include <string>

// Define a concept
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

template<typename T>
concept Printable = requires(T a, std::ostream& os) {
    { os << a } -> std::same_as<std::ostream&>;
};

template<typename T>
concept Number = std::integral<T> || std::floating_point<T>;

// Use concept in template
template<Addable T>
T add(T a, T b) {
    return a + b;
}

// Require clause
template<typename T>
    requires Number<T>
T square(T x) {
    return x * x;
}

// Abbreviated syntax
void print_number(Number auto n) {
    std::cout << n << "\n";
}

// Combining concepts
template<typename T>
concept PrintableNumber = Printable<T> && Number<T>;

int main() {
    std::cout << add(1, 2) << "\n";
    std::cout << add(std::string("Hello, "), std::string("World!")) << "\n";
    
    std::cout << square(5) << "\n";
    std::cout << square(3.14) << "\n";
    
    print_number(42);
    print_number(3.14);
    // print_number("hello");  // ERROR: not a number
    
    return 0;
}
```

## RAII (Resource Acquisition Is Initialization)

```cpp
#include <iostream>
#include <fstream>
#include <mutex>
#include <memory>

// RAII for file handling
class FileHandle {
public:
    explicit FileHandle(const std::string& filename)
        : file_(filename) {
        if (!file_.is_open()) {
            throw std::runtime_error("Failed to open file: " + filename);
        }
        std::cout << "File opened\n";
    }
    
    ~FileHandle() {
        if (file_.is_open()) {
            file_.close();
            std::cout << "File closed\n";
        }
    }
    
    // Non-copyable
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    
    // Movable
    FileHandle(FileHandle&& other) noexcept : file_(std::move(other.file_)) {}
    
    std::string readline() {
        std::string line;
        std::getline(file_, line);
        return line;
    }

private:
    std::ifstream file_;
};

// RAII for mutex locking
class LockGuard {
public:
    explicit LockGuard(std::mutex& mtx) : mutex_(mtx) {
        mutex_.lock();
    }
    
    ~LockGuard() {
        mutex_.unlock();
    }
    
    LockGuard(const LockGuard&) = delete;
    LockGuard& operator=(const LockGuard&) = delete;

private:
    std::mutex& mutex_;
};

// RAII for memory
class Buffer {
public:
    explicit Buffer(size_t size) : size_(size), data_(new char[size]) {
        std::cout << "Buffer allocated: " << size << " bytes\n";
    }
    
    ~Buffer() {
        delete[] data_;
        std::cout << "Buffer deallocated\n";
    }
    
    char* data() { return data_; }
    size_t size() const { return size_; }

private:
    size_t size_;
    char* data_;
};

int main() {
    // File automatically closed at end of scope
    try {
        FileHandle file("test.txt");
        // Use file...
    } catch (const std::exception& e) {
        std::cout << "Error: " << e.what() << "\n";
    }
    
    // Mutex automatically unlocked
    std::mutex mtx;
    {
        LockGuard lock(mtx);
        // Critical section...
    }  // Lock released here
    
    // Better: use standard RAII wrappers
    {
        std::lock_guard<std::mutex> lock(mtx);
        // Or C++17: std::scoped_lock lock(mtx1, mtx2);
    }
    
    // Buffer automatically freed
    {
        Buffer buf(1024);
        // Use buffer...
    }  // Memory freed here
    
    return 0;
}
```

## Concurrency and Multithreading

### Threads

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <chrono>

void hello() {
    std::cout << "Hello from thread " << std::this_thread::get_id() << "\n";
}

void worker(int id, int iterations) {
    for (int i = 0; i < iterations; ++i) {
        std::cout << "Worker " << id << ": iteration " << i << "\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

int main() {
    // Basic thread
    std::thread t1(hello);
    t1.join();  // Wait for thread to finish
    
    // Thread with arguments
    std::thread t2(worker, 1, 3);
    std::thread t3(worker, 2, 3);
    
    t2.join();
    t3.join();
    
    // Lambda thread
    std::thread t4([]() {
        std::cout << "Lambda thread\n";
    });
    t4.join();
    
    // Detached thread (runs independently)
    std::thread t5([]() {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        std::cout << "Detached thread finished\n";
    });
    t5.detach();
    
    // Hardware concurrency
    std::cout << "Hardware threads: " << std::thread::hardware_concurrency() << "\n";
    
    // Wait a bit for detached thread
    std::this_thread::sleep_for(std::chrono::seconds(2));
    
    return 0;
}
```

### Mutex and Locks

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

std::mutex cout_mutex;
int counter = 0;
std::mutex counter_mutex;

void safe_print(const std::string& msg) {
    std::lock_guard<std::mutex> lock(cout_mutex);
    std::cout << msg << "\n";
}

void increment(int times) {
    for (int i = 0; i < times; ++i) {
        std::lock_guard<std::mutex> lock(counter_mutex);
        ++counter;
    }
}

int main() {
    // Safe printing
    std::thread t1([](){ safe_print("Thread 1"); });
    std::thread t2([](){ safe_print("Thread 2"); });
    t1.join();
    t2.join();
    
    // Counter with mutex
    counter = 0;
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment, 1000);
    }
    for (auto& t : threads) {
        t.join();
    }
    std::cout << "Counter: " << counter << "\n";  // Should be 10000
    
    // C++17 scoped_lock (multiple mutexes)
    std::mutex m1, m2;
    {
        std::scoped_lock lock(m1, m2);
        // Use both resources safely
    }
    
    return 0;
}
```

### Atomics

```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <vector>

std::atomic<int> atomic_counter{0};

void atomic_increment(int times) {
    for (int i = 0; i < times; ++i) {
        atomic_counter.fetch_add(1, std::memory_order_relaxed);
        // Or simply: ++atomic_counter;
    }
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(atomic_increment, 1000);
    }
    for (auto& t : threads) {
        t.join();
    }
    
    std::cout << "Atomic counter: " << atomic_counter.load() << "\n";
    
    // Atomic operations
    std::atomic<int> value{10};
    
    value.store(20);                    // Store
    int v = value.load();               // Load
    int old = value.exchange(30);       // Exchange
    
    int expected = 30;
    bool success = value.compare_exchange_strong(expected, 40);
    
    std::cout << "Value: " << value.load() << "\n";
    
    // Atomic flag (lock-free boolean)
    std::atomic_flag flag = ATOMIC_FLAG_INIT;
    flag.test_and_set();
    flag.clear();
    
    return 0;
}
```

### Futures and Promises

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

int compute_value(int x) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

int main() {
    // std::async - easiest way to run async task
    std::future<int> future1 = std::async(std::launch::async, compute_value, 5);
    
    std::cout << "Doing other work...\n";
    
    // Get result (blocks if not ready)
    int result1 = future1.get();
    std::cout << "Result: " << result1 << "\n";
    
    // Launch policy
    auto future2 = std::async(std::launch::async, compute_value, 6);    // New thread
    auto future3 = std::async(std::launch::deferred, compute_value, 7); // Lazy
    
    // std::promise and std::future
    std::promise<int> promise;
    std::future<int> future4 = promise.get_future();
    
    std::thread t([&promise]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        promise.set_value(42);  // Fulfill promise
    });
    
    std::cout << "Waiting for promise...\n";
    std::cout << "Got: " << future4.get() << "\n";
    t.join();
    
    // Check if future is ready
    auto future5 = std::async(std::launch::async, compute_value, 8);
    if (future5.wait_for(std::chrono::milliseconds(100)) == std::future_status::ready) {
        std::cout << "Ready!\n";
    } else {
        std::cout << "Still computing...\n";
    }
    future5.get();  // Must still call get()
    
    return 0;
}
```

## Constexpr and Compile-Time Programming

```cpp
#include <iostream>
#include <array>

// constexpr function
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

// constexpr class
class Point {
public:
    constexpr Point(double x = 0, double y = 0) : x_(x), y_(y) {}
    constexpr double x() const { return x_; }
    constexpr double y() const { return y_; }
    constexpr double distance_from_origin() const {
        return x_ * x_ + y_ * y_;  // sqrt not constexpr
    }
private:
    double x_, y_;
};

// C++17 if constexpr
template<typename T>
auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 0.5;
    } else {
        return value;
    }
}

// C++20 consteval (must be compile-time)
consteval int must_be_compile_time(int n) {
    return n * n;
}

// C++20 constinit (compile-time initialization)
constinit int global_value = factorial(5);

// Compile-time computation
constexpr auto make_array() {
    std::array<int, 10> arr{};
    for (int i = 0; i < 10; ++i) {
        arr[i] = i * i;
    }
    return arr;
}

constexpr auto squares = make_array();

int main() {
    // Compile-time evaluation
    constexpr int fact5 = factorial(5);
    static_assert(fact5 == 120, "Factorial incorrect!");
    
    // constexpr object
    constexpr Point p(3.0, 4.0);
    constexpr double dist = p.distance_from_origin();
    
    // if constexpr
    std::cout << process(42) << "\n";     // 84
    std::cout << process(3.14) << "\n";   // 1.57
    
    // consteval
    constexpr int sq = must_be_compile_time(5);  // OK
    // int n = 5;
    // int sq2 = must_be_compile_time(n);  // ERROR: not compile-time
    
    // Use compile-time array
    for (int n : squares) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

## Exercises

### Exercise 1: Generic Stack
Implement a stack template with push, pop, and peek operations.

### Exercise 2: Thread Pool
Create a simple thread pool that executes tasks from a queue.

### Exercise 3: Compile-Time Lookup Table
Create a constexpr lookup table (e.g., sin values).

### Exercise 4: Custom Concept
Define and use a concept for containers with begin/end.

## Next Steps

In the final module, we'll put everything together with practical projects:
- CLI applications
- Memory management systems
- Basic game engine components

---

**Practice Tip:** Templates and concurrency are complex. Start simple and gradually add complexity!
