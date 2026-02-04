# Module 7: STL & Standard Library

## Containers Overview

The Standard Template Library (STL) provides a rich set of containers for different use cases.

### Container Categories

| Category | Containers | Use Case |
|----------|------------|----------|
| Sequence | vector, array, deque, list, forward_list | Ordered collections |
| Associative | set, map, multiset, multimap | Sorted key-based access |
| Unordered | unordered_set, unordered_map, etc. | Hash-based fast access |
| Adapters | stack, queue, priority_queue | Restricted access patterns |

## Sequence Containers

### std::vector

```cpp
#include <iostream>
#include <vector>

int main() {
    // Creation
    std::vector<int> v1;                    // Empty
    std::vector<int> v2(5);                 // 5 zeros
    std::vector<int> v3(5, 10);             // 5 tens
    std::vector<int> v4{1, 2, 3, 4, 5};     // Initializer list
    std::vector<int> v5(v4.begin(), v4.end()); // From iterators
    
    // Adding elements
    v1.push_back(1);
    v1.push_back(2);
    v1.emplace_back(3);  // Construct in-place (more efficient)
    
    // Accessing elements
    std::cout << "Front: " << v1.front() << "\n";
    std::cout << "Back: " << v1.back() << "\n";
    std::cout << "v1[1]: " << v1[1] << "\n";          // No bounds check
    std::cout << "v1.at(1): " << v1.at(1) << "\n";    // Bounds checked
    
    // Size and capacity
    std::cout << "Size: " << v1.size() << "\n";
    std::cout << "Capacity: " << v1.capacity() << "\n";
    std::cout << "Empty: " << v1.empty() << "\n";
    
    v1.reserve(100);     // Pre-allocate
    v1.shrink_to_fit();  // Release unused capacity
    
    // Modifying
    v1.pop_back();       // Remove last
    v1.clear();          // Remove all
    v1.resize(10);       // Change size
    
    // Accessing 
    v1.at()
    
    // Iterating
    v4.push_back(6);
    for (size_t i = 0; i < v4.size(); ++i) {
        std::cout << v4[i] << " ";
    }
    std::cout << "\n";
    
    for (const auto& elem : v4) {
        std::cout << elem << " ";
    }
    std::cout << "\n";
    
    // Insert and erase
    v4.insert(v4.begin() + 2, 100);  // Insert at position 2
    v4.erase(v4.begin());            // Erase first element
    v4.erase(v4.begin(), v4.begin() + 2);  // Erase range
    
    return 0;
}
```

### std::array (C++11)

```cpp
#include <iostream>
#include <array>

int main() {
    // Fixed-size array
    std::array<int, 5> arr = {1, 2, 3, 4, 5};
    
    // Size known at compile time
    constexpr size_t size = arr.size();
    
    // Access
    std::cout << "arr[0]: " << arr[0] << "\n";
    std::cout << "arr.at(0): " << arr.at(0) << "\n";
    std::cout << "Front: " << arr.front() << "\n";
    std::cout << "Back: " << arr.back() << "\n";
    
    // Raw pointer access
    int* data = arr.data();
    
    // Fill
    arr.fill(0);
    
    // Swap
    std::array<int, 5> arr2 = {5, 4, 3, 2, 1};
    arr.swap(arr2);
    
    return 0;
}
```

### std::deque

```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> dq = {3, 4, 5};
    
    // Efficient insertion at both ends
    dq.push_front(2);
    dq.push_front(1);
    dq.push_back(6);
    dq.push_back(7);
    
    // Remove from both ends
    dq.pop_front();
    dq.pop_back();
    
    // Random access (slightly slower than vector)
    std::cout << "dq[2]: " << dq[2] << "\n";
    
    for (int n : dq) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

### std::list

```cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> lst = {1, 2, 3, 4, 5};
    
    // Efficient insertion/deletion anywhere
    auto it = lst.begin();
    std::advance(it, 2);
    lst.insert(it, 100);  // Insert before iterator
    
    // No random access!
    // lst[0];  // ERROR!
    
    // List-specific operations
    lst.push_front(0);
    lst.push_back(6);
    lst.reverse();
    lst.sort();
    lst.unique();  // Remove consecutive duplicates
    
    // Splice - move elements between lists
    std::list<int> lst2 = {10, 20, 30};
    lst.splice(lst.begin(), lst2);  // Move lst2 to front of lst
    
    for (int n : lst) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

## Associative Containers

### std::map

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    // Ordered map (red-black tree)
    std::map<std::string, int> ages;
    
    // Insert
    ages["Alice"] = 30;
    ages["Bob"] = 25;
    ages.insert({"Charlie", 35});
    ages.emplace("Diana", 28);
    
    // Access
    std::cout << "Alice: " << ages["Alice"] << "\n";
    std::cout << "Bob: " << ages.at("Bob") << "\n";
    
    // Find
    if (auto it = ages.find("Charlie"); it != ages.end()) {
        std::cout << "Found: " << it->first << " = " << it->second << "\n";
    }
    
    // C++20 contains
    if (ages.contains("Diana")) {
        std::cout << "Diana exists\n";
    }
    
    // Iterate (keys are sorted)
    for (const auto& [name, age] : ages) {
        std::cout << name << ": " << age << "\n";
    }
    
    // Erase
    ages.erase("Bob");
    
    return 0;
}
```

### std::set

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> nums = {5, 2, 8, 1, 9, 3};
    
    // Automatically sorted, unique values
    for (int n : nums) {
        std::cout << n << " ";  // 1 2 3 5 8 9
    }
    std::cout << "\n";
    
    // Insert
    auto [it, inserted] = nums.insert(4);
    std::cout << "Inserted: " << inserted << "\n";
    
    auto [it2, inserted2] = nums.insert(5);  // Already exists
    std::cout << "Inserted: " << inserted2 << "\n";
    
    // Find
    if (nums.count(5) > 0) {
        std::cout << "5 is in the set\n";
    }
    
    // Bounds
    auto lower = nums.lower_bound(4);  // First >= 4
    auto upper = nums.upper_bound(4);  // First > 4
    
    return 0;
}
```

### std::unordered_map

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    // Hash table - O(1) average access
    std::unordered_map<std::string, int> scores;
    
    scores["Alice"] = 95;
    scores["Bob"] = 87;
    scores["Charlie"] = 92;
    
    // Fast lookup
    std::cout << "Alice: " << scores["Alice"] << "\n";
    
    // Iteration (unordered!)
    for (const auto& [name, score] : scores) {
        std::cout << name << ": " << score << "\n";
    }
    
    // Bucket info
    std::cout << "Bucket count: " << scores.bucket_count() << "\n";
    std::cout << "Load factor: " << scores.load_factor() << "\n";
    
    return 0;
}
```

### C++23 std::flat_map

```cpp
#include <iostream>
#include <flat_map>

int main() {
    // Flat map - better cache performance for small maps
    std::flat_map<int, std::string> fm;
    
    fm[1] = "one";
    fm[2] = "two";
    fm[3] = "three";
    
    // Sorted iteration
    for (const auto& [key, value] : fm) {
        std::cout << key << ": " << value << "\n";
    }
    
    // Better for read-heavy, small datasets
    // Uses sorted vector internally
    
    return 0;
}
```

## Algorithms

### Sorting

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    
    // Sort ascending
    std::sort(nums.begin(), nums.end());
    
    // Sort descending
    std::sort(nums.begin(), nums.end(), std::greater<int>());
    
    // Sort with custom comparator
    std::sort(nums.begin(), nums.end(), [](int a, int b) {
        return a % 10 < b % 10;  // Sort by last digit
    });
    
    // Partial sort (top N)
    std::partial_sort(nums.begin(), nums.begin() + 3, nums.end());
    
    // Stable sort (preserves relative order of equal elements)
    std::stable_sort(nums.begin(), nums.end());
    
    // Nth element (partition around Nth)
    std::nth_element(nums.begin(), nums.begin() + 4, nums.end());
    // Element at position 4 is in its sorted position
    
    return 0;
}
```

### Searching

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    // Linear search
    auto it = std::find(nums.begin(), nums.end(), 5);
    if (it != nums.end()) {
        std::cout << "Found at index: " << (it - nums.begin()) << "\n";
    }
    
    // Find with predicate
    auto it2 = std::find_if(nums.begin(), nums.end(), [](int n) {
        return n > 5;
    });
    
    // Binary search (requires sorted range)
    bool found = std::binary_search(nums.begin(), nums.end(), 5);
    
    // Lower/upper bound
    auto lower = std::lower_bound(nums.begin(), nums.end(), 5);
    auto upper = std::upper_bound(nums.begin(), nums.end(), 5);
    
    // Equal range
    auto [lo, hi] = std::equal_range(nums.begin(), nums.end(), 5);
    
    // Count
    int count = std::count(nums.begin(), nums.end(), 5);
    int count_if = std::count_if(nums.begin(), nums.end(), [](int n) {
        return n % 2 == 0;
    });
    
    return 0;
}
```

### Transformations

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    std::vector<int> result(nums.size());
    
    // Transform
    std::transform(nums.begin(), nums.end(), result.begin(),
                   [](int n) { return n * n; });
    
    // Transform two ranges
    std::vector<int> nums2 = {10, 20, 30, 40, 50};
    std::transform(nums.begin(), nums.end(), nums2.begin(), result.begin(),
                   [](int a, int b) { return a + b; });
    
    // Copy
    std::copy(nums.begin(), nums.end(), result.begin());
    
    // Copy if
    std::vector<int> evens;
    std::copy_if(nums.begin(), nums.end(), std::back_inserter(evens),
                 [](int n) { return n % 2 == 0; });
    
    // Replace
    std::replace(nums.begin(), nums.end(), 3, 100);
    
    // Replace if
    std::replace_if(nums.begin(), nums.end(),
                    [](int n) { return n > 3; }, 0);
    
    // Remove (returns new end iterator)
    auto new_end = std::remove(nums.begin(), nums.end(), 0);
    nums.erase(new_end, nums.end());  // Actually erase
    
    // Remove if
    nums.erase(std::remove_if(nums.begin(), nums.end(),
                              [](int n) { return n % 2 == 0; }),
               nums.end());
    
    // Reverse
    std::reverse(nums.begin(), nums.end());
    
    // Rotate
    std::rotate(nums.begin(), nums.begin() + 2, nums.end());
    
    return 0;
}
```

### Numeric Algorithms

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};
    
    // Accumulate (sum)
    int sum = std::accumulate(nums.begin(), nums.end(), 0);
    
    // Accumulate with custom operation
    int product = std::accumulate(nums.begin(), nums.end(), 1,
                                  [](int a, int b) { return a * b; });
    
    // Inner product
    std::vector<int> nums2 = {1, 2, 3, 4, 5};
    int dot = std::inner_product(nums.begin(), nums.end(),
                                 nums2.begin(), 0);
    
    // Partial sum
    std::vector<int> partial(nums.size());
    std::partial_sum(nums.begin(), nums.end(), partial.begin());
    // partial = {1, 3, 6, 10, 15}
    
    // Adjacent difference
    std::vector<int> diff(nums.size());
    std::adjacent_difference(nums.begin(), nums.end(), diff.begin());
    // diff = {1, 1, 1, 1, 1}
    
    // Iota
    std::vector<int> seq(10);
    std::iota(seq.begin(), seq.end(), 1);  // Fill with 1, 2, 3, ...
    
    // C++17: Reduce (parallelizable accumulate)
    int sum2 = std::reduce(nums.begin(), nums.end());
    
    // C++17: Transform reduce
    int sum_of_squares = std::transform_reduce(
        nums.begin(), nums.end(),
        0,
        std::plus<>(),
        [](int n) { return n * n; }
    );
    
    std::cout << "Sum: " << sum << "\n";
    std::cout << "Product: " << product << "\n";
    std::cout << "Dot product: " << dot << "\n";
    
    return 0;
}
```

## C++20 Ranges

```cpp
#include <iostream>
#include <vector>
#include <ranges>
#include <algorithm>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // Filter view
    auto evens = nums | std::views::filter([](int n) { return n % 2 == 0; });
    
    // Transform view
    auto squared = nums | std::views::transform([](int n) { return n * n; });
    
    // Chained views
    auto result = nums 
        | std::views::filter([](int n) { return n % 2 == 0; })
        | std::views::transform([](int n) { return n * n; })
        | std::views::take(3);
    
    for (int n : result) {
        std::cout << n << " ";  // 4 16 36
    }
    std::cout << "\n";
    
    // Range-based algorithms
    std::ranges::sort(nums);
    std::ranges::reverse(nums);
    
    auto it = std::ranges::find(nums, 5);
    
    // Views
    for (int n : std::views::iota(1, 11)) {
        std::cout << n << " ";  // 1 2 3 4 5 6 7 8 9 10
    }
    std::cout << "\n";
    
    // Drop and take
    for (int n : nums | std::views::drop(3) | std::views::take(4)) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    // Reverse view
    for (int n : nums | std::views::reverse) {
        std::cout << n << " ";
    }
    std::cout << "\n";
    
    // Keys and values from map
    std::map<int, std::string> m = {{1, "one"}, {2, "two"}, {3, "three"}};
    for (const auto& key : m | std::views::keys) {
        std::cout << key << " ";
    }
    std::cout << "\n";
    
    return 0;
}
```

## Strings

```cpp
#include <iostream>
#include <string>
#include <string_view>
#include <sstream>

int main() {
    // std::string
    std::string s1 = "Hello";
    std::string s2 = "World";
    std::string s3 = s1 + ", " + s2 + "!";
    
    // Access
    char first = s3[0];
    char last = s3.back();
    
    // Modification
    s3.append(" How are you?");
    s3.insert(5, "...");
    s3.erase(5, 3);
    s3.replace(0, 5, "Hi");
    
    // Search
    size_t pos = s3.find("World");
    size_t rpos = s3.rfind("o");
    
    // Substring
    std::string sub = s3.substr(0, 5);
    
    // Compare
    int cmp = s1.compare(s2);
    
    // C++17 string_view (non-owning)
    std::string_view sv = s3;  // No copy
    std::string_view sub_view = sv.substr(0, 5);  // No copy
    
    // String streams
    std::ostringstream oss;
    oss << "Value: " << 42 << ", Pi: " << 3.14;
    std::string result = oss.str();
    
    std::istringstream iss("42 3.14 hello");
    int i;
    double d;
    std::string word;
    iss >> i >> d >> word;
    
    // C++20 starts_with, ends_with
    bool starts = s3.starts_with("Hi");
    bool ends = s3.ends_with("!");
    
    // C++23 contains
    bool contains = s3.contains("World");
    
    return 0;
}
```

## std::format and std::print (C++20/C++23)

```cpp
#include <iostream>
#include <format>
#include <print>

int main() {
    // C++20 std::format
    std::string s = std::format("Hello, {}!", "World");
    std::string n = std::format("Value: {}", 42);
    
    // Positional arguments
    std::string pos = std::format("{1} {0}", "World", "Hello");
    
    // Formatting specifiers
    std::string hex = std::format("{:x}", 255);       // ff
    std::string HEX = std::format("{:#X}", 255);      // 0XFF
    std::string bin = std::format("{:b}", 42);        // 101010
    std::string oct = std::format("{:o}", 64);        // 100
    
    std::string flt = std::format("{:.2f}", 3.14159); // 3.14
    std::string sci = std::format("{:e}", 1234.5);    // 1.2345e+03
    
    std::string width = std::format("{:10}", 42);     // "        42"
    std::string left = std::format("{:<10}", 42);     // "42        "
    std::string center = std::format("{:^10}", 42);   // "    42    "
    std::string fill = std::format("{:0>5}", 42);     // "00042"
    
    // C++23 std::print and std::println
    std::println("Hello, World!");
    std::print("Value: {}\n", 42);
    std::println("Name: {}, Age: {}", "Alice", 30);
    std::println("{:.3f}", 3.14159);
    
    // Print to file
    // std::print(file, "Hello, {}!", "World");
    
    return 0;
}
```

## Exercises

### Exercise 1: Word Frequency Counter
Read text and count word frequencies using map/unordered_map.

### Exercise 2: Custom Container
Implement a simple container with iterators that works with STL algorithms.

### Exercise 3: Ranges Pipeline
Process data using a chain of range views.

### Exercise 4: String Parser
Parse a CSV string using string operations and algorithms.

## Next Steps

In the next module, we'll cover:
- Templates and generic programming
- RAII
- Smart pointers deep dive
- Concurrency and multithreading

---

**Practice Tip:** The STL is vast. Focus on vector, map, and the most common algorithms first!
