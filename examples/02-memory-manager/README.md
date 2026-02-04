# Example Project 2: Memory Manager

A custom memory management system demonstrating various allocation strategies and debugging capabilities.

## Features

- Multiple allocation strategies (pool, stack, freelist)
- Memory tracking and leak detection
- Alignment support
- Debug bounds checking
- Statistics and profiling
- Thread-safe allocators

## Project Structure

```
02-memory-manager/
├── README.md
├── CMakeLists.txt
├── include/
│   ├── allocator_base.hpp
│   ├── pool_allocator.hpp
│   ├── stack_allocator.hpp
│   ├── freelist_allocator.hpp
│   ├── memory_tracker.hpp
│   └── memory_arena.hpp
├── src/
│   ├── main.cpp
│   ├── pool_allocator.cpp
│   ├── stack_allocator.cpp
│   ├── freelist_allocator.cpp
│   └── memory_tracker.cpp
└── tests/
    └── test_allocators.cpp
```

## Building

```bash
mkdir build && cd build
cmake ..
cmake --build .
./memory_manager
```

## Code

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)
project(MemoryManager VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(memory_manager
    src/main.cpp
    src/pool_allocator.cpp
    src/stack_allocator.cpp
    src/freelist_allocator.cpp
    src/memory_tracker.cpp
)

target_include_directories(memory_manager PRIVATE include)
target_compile_options(memory_manager PRIVATE -Wall -Wextra -Wpedantic)
```

### include/allocator_base.hpp

```cpp
#pragma once
#include <cstddef>
#include <cstdint>
#include <string>

// Alignment utilities
inline size_t align_up(size_t n, size_t alignment) {
    return (n + alignment - 1) & ~(alignment - 1);
}

inline void* align_ptr(void* ptr, size_t alignment) {
    uintptr_t addr = reinterpret_cast<uintptr_t>(ptr);
    uintptr_t aligned = align_up(addr, alignment);
    return reinterpret_cast<void*>(aligned);
}

// Base allocator interface
class AllocatorBase {
public:
    virtual ~AllocatorBase() = default;
    
    virtual void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) = 0;
    virtual void deallocate(void* ptr) = 0;
    virtual void reset() = 0;
    
    virtual size_t used_memory() const = 0;
    virtual size_t total_memory() const = 0;
    virtual size_t allocation_count() const = 0;
    
    virtual std::string name() const = 0;
    
    // Convenience methods
    template<typename T, typename... Args>
    T* create(Args&&... args) {
        void* mem = allocate(sizeof(T), alignof(T));
        return new (mem) T(std::forward<Args>(args)...);
    }
    
    template<typename T>
    void destroy(T* obj) {
        if (obj) {
            obj->~T();
            deallocate(obj);
        }
    }
};
```

### include/pool_allocator.hpp

```cpp
#pragma once
#include "allocator_base.hpp"
#include <vector>
#include <mutex>

// Fixed-size block pool allocator
// O(1) allocation and deallocation
// No fragmentation for same-size allocations
class PoolAllocator : public AllocatorBase {
public:
    PoolAllocator(size_t block_size, size_t block_count);
    ~PoolAllocator() override;
    
    // Non-copyable
    PoolAllocator(const PoolAllocator&) = delete;
    PoolAllocator& operator=(const PoolAllocator&) = delete;
    
    // Movable
    PoolAllocator(PoolAllocator&& other) noexcept;
    PoolAllocator& operator=(PoolAllocator&& other) noexcept;
    
    void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) override;
    void deallocate(void* ptr) override;
    void reset() override;
    
    size_t used_memory() const override { return used_blocks_ * block_size_; }
    size_t total_memory() const override { return total_size_; }
    size_t allocation_count() const override { return used_blocks_; }
    
    std::string name() const override { return "PoolAllocator"; }
    
    size_t block_size() const { return block_size_; }
    size_t block_count() const { return block_count_; }
    size_t free_blocks() const { return block_count_ - used_blocks_; }

private:
    struct FreeBlock {
        FreeBlock* next;
    };
    
    void* memory_;
    FreeBlock* free_list_;
    size_t block_size_;
    size_t block_count_;
    size_t total_size_;
    size_t used_blocks_;
};

// Thread-safe version
class ThreadSafePoolAllocator : public PoolAllocator {
public:
    using PoolAllocator::PoolAllocator;
    
    void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) override {
        std::lock_guard<std::mutex> lock(mutex_);
        return PoolAllocator::allocate(size, alignment);
    }
    
    void deallocate(void* ptr) override {
        std::lock_guard<std::mutex> lock(mutex_);
        PoolAllocator::deallocate(ptr);
    }
    
    std::string name() const override { return "ThreadSafePoolAllocator"; }

private:
    mutable std::mutex mutex_;
};
```

### include/stack_allocator.hpp

```cpp
#pragma once
#include "allocator_base.hpp"

// Stack allocator - LIFO allocation pattern
// Very fast allocation
// Deallocation only in reverse order
class StackAllocator : public AllocatorBase {
public:
    explicit StackAllocator(size_t size);
    ~StackAllocator() override;
    
    StackAllocator(const StackAllocator&) = delete;
    StackAllocator& operator=(const StackAllocator&) = delete;
    
    void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) override;
    void deallocate(void* ptr) override;
    void reset() override;
    
    // Stack-specific: deallocate back to a marker
    using Marker = size_t;
    Marker get_marker() const { return offset_; }
    void free_to_marker(Marker marker);
    
    size_t used_memory() const override { return offset_; }
    size_t total_memory() const override { return size_; }
    size_t allocation_count() const override { return allocation_count_; }
    
    std::string name() const override { return "StackAllocator"; }

private:
    void* memory_;
    size_t size_;
    size_t offset_;
    size_t allocation_count_;
    
    struct AllocationHeader {
        size_t adjustment;  // For alignment
        size_t size;
    };
};
```

### include/freelist_allocator.hpp

```cpp
#pragma once
#include "allocator_base.hpp"

// Free list allocator - general purpose
// Variable-size allocations
// First-fit allocation strategy
class FreeListAllocator : public AllocatorBase {
public:
    enum class PlacementPolicy {
        FirstFit,
        BestFit,
        WorstFit
    };
    
    explicit FreeListAllocator(size_t size, PlacementPolicy policy = PlacementPolicy::FirstFit);
    ~FreeListAllocator() override;
    
    FreeListAllocator(const FreeListAllocator&) = delete;
    FreeListAllocator& operator=(const FreeListAllocator&) = delete;
    
    void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) override;
    void deallocate(void* ptr) override;
    void reset() override;
    
    size_t used_memory() const override { return used_; }
    size_t total_memory() const override { return size_; }
    size_t allocation_count() const override { return allocation_count_; }
    
    std::string name() const override { return "FreeListAllocator"; }
    
    // Debug info
    size_t free_block_count() const;
    size_t largest_free_block() const;

private:
    struct FreeBlock {
        size_t size;
        FreeBlock* next;
    };
    
    struct AllocationHeader {
        size_t size;
        size_t adjustment;
    };
    
    void* memory_;
    FreeBlock* free_list_;
    size_t size_;
    size_t used_;
    size_t allocation_count_;
    PlacementPolicy policy_;
    
    FreeBlock* find_first_fit(size_t size, size_t alignment, FreeBlock** prev);
    FreeBlock* find_best_fit(size_t size, size_t alignment, FreeBlock** prev);
    void coalesce(FreeBlock* prev, FreeBlock* block);
};
```

### include/memory_tracker.hpp

```cpp
#pragma once
#include <unordered_map>
#include <string>
#include <mutex>
#include <vector>
#include <source_location>

struct AllocationInfo {
    void* ptr;
    size_t size;
    std::string file;
    int line;
    std::string function;
    uint64_t timestamp;
};

class MemoryTracker {
public:
    static MemoryTracker& instance();
    
    void track_allocation(void* ptr, size_t size, 
                         const std::source_location& loc = std::source_location::current());
    void track_deallocation(void* ptr);
    
    void print_stats() const;
    void print_leaks() const;
    
    size_t total_allocated() const { return total_allocated_; }
    size_t total_deallocated() const { return total_deallocated_; }
    size_t current_usage() const { return total_allocated_ - total_deallocated_; }
    size_t peak_usage() const { return peak_usage_; }
    size_t allocation_count() const { return allocation_count_; }
    size_t deallocation_count() const { return deallocation_count_; }
    
    const std::unordered_map<void*, AllocationInfo>& active_allocations() const {
        return allocations_;
    }

private:
    MemoryTracker() = default;
    
    std::unordered_map<void*, AllocationInfo> allocations_;
    mutable std::mutex mutex_;
    
    size_t total_allocated_ = 0;
    size_t total_deallocated_ = 0;
    size_t peak_usage_ = 0;
    size_t allocation_count_ = 0;
    size_t deallocation_count_ = 0;
};

// Tracking macros
#define TRACK_ALLOC(ptr, size) \
    MemoryTracker::instance().track_allocation(ptr, size)

#define TRACK_FREE(ptr) \
    MemoryTracker::instance().track_deallocation(ptr)
```

### include/memory_arena.hpp

```cpp
#pragma once
#include "allocator_base.hpp"
#include <vector>
#include <memory>

// Arena allocator - bulk allocate, bulk free
// Fast allocation, no individual deallocation
class MemoryArena : public AllocatorBase {
public:
    explicit MemoryArena(size_t chunk_size = 1024 * 1024);  // 1MB default
    ~MemoryArena() override;
    
    MemoryArena(const MemoryArena&) = delete;
    MemoryArena& operator=(const MemoryArena&) = delete;
    
    void* allocate(size_t size, size_t alignment = alignof(std::max_align_t)) override;
    void deallocate(void* ptr) override;  // No-op for arena
    void reset() override;
    
    size_t used_memory() const override { return total_used_; }
    size_t total_memory() const override { return total_allocated_; }
    size_t allocation_count() const override { return allocation_count_; }
    
    std::string name() const override { return "MemoryArena"; }

private:
    struct Chunk {
        void* memory;
        size_t size;
        size_t used;
        
        Chunk(size_t s) : size(s), used(0) {
            memory = std::malloc(s);
        }
        ~Chunk() {
            std::free(memory);
        }
    };
    
    std::vector<std::unique_ptr<Chunk>> chunks_;
    size_t chunk_size_;
    size_t total_used_ = 0;
    size_t total_allocated_ = 0;
    size_t allocation_count_ = 0;
    
    void add_chunk(size_t min_size);
};
```

### src/main.cpp

```cpp
#include "pool_allocator.hpp"
#include "stack_allocator.hpp"
#include "freelist_allocator.hpp"
#include "memory_arena.hpp"
#include "memory_tracker.hpp"
#include <iostream>
#include <chrono>
#include <vector>
#include <random>

struct TestObject {
    int data[16];  // 64 bytes
    
    TestObject() {
        for (int i = 0; i < 16; ++i) {
            data[i] = i;
        }
    }
};

template<typename Allocator>
void benchmark_allocator(Allocator& alloc, int iterations) {
    std::vector<void*> ptrs;
    ptrs.reserve(iterations);
    
    auto start = std::chrono::high_resolution_clock::now();
    
    // Allocate
    for (int i = 0; i < iterations; ++i) {
        void* ptr = alloc.allocate(sizeof(TestObject), alignof(TestObject));
        new (ptr) TestObject();
        ptrs.push_back(ptr);
    }
    
    // Deallocate
    for (void* ptr : ptrs) {
        static_cast<TestObject*>(ptr)->~TestObject();
        alloc.deallocate(ptr);
    }
    
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
    
    std::cout << alloc.name() << ": " << duration.count() << " us\n";
}

void benchmark_standard(int iterations) {
    std::vector<TestObject*> ptrs;
    ptrs.reserve(iterations);
    
    auto start = std::chrono::high_resolution_clock::now();
    
    for (int i = 0; i < iterations; ++i) {
        ptrs.push_back(new TestObject());
    }
    
    for (TestObject* ptr : ptrs) {
        delete ptr;
    }
    
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
    
    std::cout << "Standard new/delete: " << duration.count() << " us\n";
}

void demo_pool_allocator() {
    std::cout << "\n=== Pool Allocator Demo ===\n";
    
    PoolAllocator pool(sizeof(TestObject), 100);
    
    std::cout << "Block size: " << pool.block_size() << " bytes\n";
    std::cout << "Block count: " << pool.block_count() << "\n";
    
    // Allocate some objects
    TestObject* obj1 = pool.create<TestObject>();
    TestObject* obj2 = pool.create<TestObject>();
    TestObject* obj3 = pool.create<TestObject>();
    
    std::cout << "After 3 allocations:\n";
    std::cout << "  Used: " << pool.used_memory() << " bytes\n";
    std::cout << "  Free blocks: " << pool.free_blocks() << "\n";
    
    pool.destroy(obj2);
    
    std::cout << "After 1 deallocation:\n";
    std::cout << "  Free blocks: " << pool.free_blocks() << "\n";
    
    pool.destroy(obj1);
    pool.destroy(obj3);
}

void demo_stack_allocator() {
    std::cout << "\n=== Stack Allocator Demo ===\n";
    
    StackAllocator stack(1024);
    
    auto marker = stack.get_marker();
    
    void* a = stack.allocate(64);
    void* b = stack.allocate(128);
    
    std::cout << "After 2 allocations: " << stack.used_memory() << " bytes\n";
    
    stack.free_to_marker(marker);
    
    std::cout << "After reset to marker: " << stack.used_memory() << " bytes\n";
}

void demo_freelist_allocator() {
    std::cout << "\n=== Free List Allocator Demo ===\n";
    
    FreeListAllocator freelist(4096, FreeListAllocator::PlacementPolicy::BestFit);
    
    void* a = freelist.allocate(100);
    void* b = freelist.allocate(200);
    void* c = freelist.allocate(50);
    
    std::cout << "After allocations:\n";
    std::cout << "  Used: " << freelist.used_memory() << " bytes\n";
    std::cout << "  Free blocks: " << freelist.free_block_count() << "\n";
    
    freelist.deallocate(b);
    
    std::cout << "After freeing middle block:\n";
    std::cout << "  Free blocks: " << freelist.free_block_count() << "\n";
    std::cout << "  Largest free: " << freelist.largest_free_block() << " bytes\n";
    
    freelist.deallocate(a);
    freelist.deallocate(c);
}

void demo_memory_arena() {
    std::cout << "\n=== Memory Arena Demo ===\n";
    
    MemoryArena arena(4096);
    
    for (int i = 0; i < 100; ++i) {
        arena.allocate(100);
    }
    
    std::cout << "After 100 allocations:\n";
    std::cout << "  Used: " << arena.used_memory() << " bytes\n";
    std::cout << "  Total: " << arena.total_memory() << " bytes\n";
    
    arena.reset();
    
    std::cout << "After reset:\n";
    std::cout << "  Used: " << arena.used_memory() << " bytes\n";
}

int main() {
    std::cout << "=== Memory Manager Demo ===\n";
    
    demo_pool_allocator();
    demo_stack_allocator();
    demo_freelist_allocator();
    demo_memory_arena();
    
    std::cout << "\n=== Benchmark (10000 allocations) ===\n";
    
    const int iterations = 10000;
    
    benchmark_standard(iterations);
    
    {
        PoolAllocator pool(sizeof(TestObject), iterations);
        benchmark_allocator(pool, iterations);
    }
    
    {
        StackAllocator stack(iterations * sizeof(TestObject) * 2);
        // Note: stack allocator requires LIFO deallocation
        std::cout << "StackAllocator: (not benchmarked - requires LIFO)\n";
    }
    
    {
        MemoryArena arena(iterations * sizeof(TestObject) * 2);
        benchmark_allocator(arena, iterations);
    }
    
    return 0;
}
```

## Learning Objectives

1. **Memory Layout** - Understanding how memory is organized
2. **Allocation Strategies** - Different approaches for different use cases
3. **Performance Trade-offs** - Speed vs flexibility vs fragmentation
4. **RAII Patterns** - Automatic resource management
5. **Thread Safety** - Protecting shared resources

## Allocation Strategy Comparison

| Allocator | Alloc Speed | Dealloc Speed | Fragmentation | Use Case |
|-----------|-------------|---------------|---------------|----------|
| Pool | O(1) | O(1) | None | Fixed-size objects |
| Stack | O(1) | O(1)* | None | Temporary allocations |
| FreeList | O(n) | O(1) | Medium | General purpose |
| Arena | O(1) | N/A | None | Bulk allocate/free |

*Stack requires LIFO deallocation order

## Extensions

- Add memory defragmentation
- Implement buddy allocator
- Add SIMD-aligned allocations
- Create custom STL-compatible allocator
- Add memory visualization

## Usage Example

```cpp
// Pool allocator for game entities
PoolAllocator entity_pool(sizeof(Entity), 10000);

// Stack allocator for temporary frame data
StackAllocator frame_allocator(1024 * 1024);

// Arena for level loading
MemoryArena level_arena(16 * 1024 * 1024);
```
