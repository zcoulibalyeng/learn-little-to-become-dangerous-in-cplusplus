# Module 1: Introduction to C/C++

## History and Evolution

### The Birth of C (1972)

C was created by Dennis Ritchie at Bell Labs in 1972. It was designed to rewrite the UNIX operating system, which was originally written in assembly language. C provided:

- **Portability** - Write once, compile anywhere
- **Efficiency** - Close to the hardware, minimal runtime overhead
- **Flexibility** - Low-level memory access with high-level constructs

### Key C Milestones

| Year | Standard | Key Features |
|------|----------|--------------|
| 1972 | Original C | Created by Dennis Ritchie |
| 1978 | K&R C | "The C Programming Language" book |
| 1989 | ANSI C (C89) | First official standard |
| 1999 | C99 | `inline`, `restrict`, `//` comments, VLAs |
| 2011 | C11 | Threads, atomics, `_Generic` |
| 2017 | C17 | Bug fixes and clarifications |
| 2024 | **C23** | `nullptr`, `typeof`, `_BitInt`, attributes |

### The Evolution of C++ (1983-Present)

C++ was created by Bjarne Stroustrup at Bell Labs, starting in 1979 as "C with Classes." It evolved into C++ (named in 1983) and has grown into a multi-paradigm language.

### Key C++ Milestones

| Year | Standard | Key Features |
|------|----------|--------------|
| 1983 | C++ Named | Classes, inheritance |
| 1998 | C++98 | First ISO standard, STL |
| 2003 | C++03 | Bug fixes |
| 2011 | C++11 | `auto`, lambdas, move semantics, smart pointers |
| 2014 | C++14 | Generic lambdas, relaxed `constexpr` |
| 2017 | C++17 | `if constexpr`, structured bindings, `std::optional` |
| 2020 | C++20 | Concepts, ranges, coroutines, modules |
| 2024 | **C++23** | `std::print`, deducing `this`, `std::expected` |
| 2026 | C++26 | Reflection, contracts (expected) |

## C23 and C++23 Standards Overview

### C23 New Features

```c
// nullptr keyword (finally!)
int* ptr = nullptr;  // Instead of NULL or (void*)0

// bool, true, false as keywords
bool flag = true;    // No need for <stdbool.h>

// typeof operator (standardized)
int x = 42;
typeof(x) y = 100;   // y is int

// Digit separators
int million = 1'000'000;
int binary = 0b1010'1010;

// Binary literals
int b = 0b11110000;

// _BitInt for arbitrary-precision integers
_BitInt(128) big_number;

// #embed directive for binary data
static const unsigned char icon[] = {
    #embed "icon.png"
};

// Attributes
[[nodiscard]] int important_result(void);
[[deprecated("Use new_func instead")]] void old_func(void);
[[maybe_unused]] int debug_var;

// Empty initializer
int arr[100] = {};   // Zero-initialize

// constexpr functions
constexpr int square(int x) { return x * x; }
```

### C++23 New Features

```cpp
// std::print and std::println
#include <print>
std::print("Hello, {}!\n", "World");
std::println("Value: {}", 42);

// Deducing this (explicit object parameter)
struct Widget {
    // Instead of const/non-const overloads
    template<typename Self>
    auto& getValue(this Self&& self) {
        return self.value;
    }
private:
    int value;
};

// std::expected for error handling
#include <expected>
std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) return std::unexpected("Division by zero");
    return a / b;
}

// import std; (modular standard library)
import std;  // Import entire standard library

// std::flat_map and std::flat_set
#include <flat_map>
std::flat_map<int, std::string> fast_map;

// std::mdspan for multidimensional views
#include <mdspan>
std::vector<int> data(100);
std::mdspan<int, std::extents<size_t, 10, 10>> matrix(data.data());

// Enhanced constexpr
constexpr std::vector<int> make_vec() {
    return {1, 2, 3, 4, 5};
}
```

## Setting Up Your Development Environment

### Option 1: Linux (Recommended for Learning)

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install build-essential gcc-14 g++-14 gdb cmake

# Set GCC 14 as default
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 100
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-14 100

# Verify installation
gcc --version
g++ --version
```

### Option 2: macOS

```bash
# Install Xcode Command Line Tools
xcode-select --install

# Or install via Homebrew
brew install gcc llvm cmake

# Use specific version
export CC=/opt/homebrew/bin/gcc-14
export CXX=/opt/homebrew/bin/g++-14
```

### Option 3: Windows

**Option A: MSYS2 (Recommended)**
1. Download MSYS2 from https://www.msys2.org/
2. Open MSYS2 MINGW64 terminal
3. Install GCC:
```bash
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-cmake mingw-w64-x86_64-gdb
```

**Option B: Visual Studio**
1. Download Visual Studio 2022+
2. Install "Desktop development with C++" workload
3. Select C++23 standard in project settings

### IDE Setup: VS Code

1. Install VS Code
2. Install extensions:
   - **C/C++** (Microsoft)
   - **CMake Tools**
   - **clangd** (recommended for better IntelliSense)

3. Configure `c_cpp_properties.json`:
```json
{
    "configurations": [
        {
            "name": "Linux",
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c23",
            "cppStandard": "c++23",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

## Your First Programs

### Hello World in C

```c
// hello.c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

Compile and run:
```bash
gcc -std=c23 -Wall -Wextra -o hello hello.c
./hello
```

### Hello World in C++

```cpp
// hello.cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

Compile and run:
```bash
g++ -std=c++23 -Wall -Wextra -o hello hello.cpp
./hello
```

### Modern C++23 Hello World

```cpp
// hello_modern.cpp
#include <print>

int main() {
    std::println("Hello, World!");
    return 0;
}
```

### Understanding the Code

#### C Version Breakdown:
```c
#include <stdio.h>  // Include standard I/O library
                    // Provides printf, scanf, etc.

int main(void) {    // Main function - program entry point
                    // Returns int, takes no parameters (void)
    
    printf("Hello, World!\n");  // Print string to stdout
                                // \n = newline character
    
    return 0;       // Return 0 to indicate success
}                   // Convention: 0 = success, non-zero = error
```

#### C++ Version Breakdown:
```cpp
#include <iostream>  // Include I/O stream library
                     // Provides std::cout, std::cin, etc.

int main() {         // Main function
                     // () is equivalent to (void) in C++
    
    std::cout << "Hello, World!" << std::endl;
    // std::cout - standard output stream
    // << - insertion operator
    // std::endl - newline + flush buffer
    
    return 0;        // Optional in C++ main()
}
```

## The Compilation Process

### From Source Code to Executable

```
Source Code (.c/.cpp)
        │
        ▼
   Preprocessor
   (#include, #define, macros)
        │
        ▼
   Compiler
   (Generates assembly)
        │
        ▼
   Assembler
   (Generates object code .o)
        │
        ▼
   Linker
   (Combines object files + libraries)
        │
        ▼
   Executable
```

### Compilation Stages

```bash
# Preprocessing only (-E)
g++ -E main.cpp -o main.i

# Compile to assembly (-S)
g++ -S main.cpp -o main.s

# Compile to object file (-c)
g++ -c main.cpp -o main.o

# Link object files
g++ main.o utils.o -o program

# All in one step
g++ main.cpp -o program
```

### Important Compiler Flags

```bash
# Standard selection
-std=c23           # Use C23 standard
-std=c++23         # Use C++23 standard

# Warnings
-Wall              # Enable common warnings
-Wextra            # Enable extra warnings
-Wpedantic         # Strict ISO compliance
-Werror            # Treat warnings as errors

# Optimization
-O0                # No optimization (debugging)
-O1                # Basic optimization
-O2                # Standard optimization
-O3                # Aggressive optimization
-Os                # Optimize for size

# Debugging
-g                 # Include debug symbols
-ggdb              # GDB-specific debug info

# Sanitizers
-fsanitize=address         # AddressSanitizer
-fsanitize=undefined       # UndefinedBehaviorSanitizer
-fsanitize=thread          # ThreadSanitizer
```

## Build Systems

### Simple Makefile

```makefile
# Makefile
CXX = g++
CXXFLAGS = -std=c++23 -Wall -Wextra -O2

TARGET = program
SRCS = main.cpp utils.cpp
OBJS = $(SRCS:.cpp=.o)

$(TARGET): $(OBJS)
	$(CXX) $(OBJS) -o $(TARGET)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

.PHONY: clean
```

### CMake (Recommended)

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.25)
project(MyProject VERSION 1.0 LANGUAGES CXX)

# Require C++23
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Add executable
add_executable(program main.cpp utils.cpp)

# Enable warnings
target_compile_options(program PRIVATE -Wall -Wextra -Wpedantic)

# Optional: Enable sanitizers for Debug builds
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(program PRIVATE -fsanitize=address,undefined)
    target_link_options(program PRIVATE -fsanitize=address,undefined)
endif()
```

Build with CMake:
```bash
mkdir build && cd build
cmake ..
cmake --build .
./program
```

## Differences Between C and C++

### C is a Subset (Mostly)

Valid C code is *mostly* valid C++, but there are differences:

```c
// Valid C, invalid C++
void* ptr = malloc(100);
int* arr = ptr;  // Implicit void* conversion - C only

// C++ requires explicit cast
int* arr = static_cast<int*>(ptr);

// Variable-length arrays (VLA) - C99/C11, not standard C++
int n = 10;
int arr[n];  // C: OK, C++: non-standard extension
```

### Key Differences

| Feature | C | C++ |
|---------|---|-----|
| Paradigm | Procedural | Multi-paradigm |
| Classes | No (struct only) | Yes |
| Function overloading | No | Yes |
| Default parameters | No | Yes |
| References | No | Yes |
| new/delete | No (malloc/free) | Yes |
| Namespaces | No | Yes |
| Templates | No | Yes |
| Exceptions | No | Yes |
| bool type | C23 keyword | Native |
| Comments // | C99+ | Always supported |

### When to Use Which?

**Use C when:**
- Writing OS kernels, device drivers
- Embedded systems with tight constraints
- Interfacing with hardware
- Maximum portability required
- Codebase is already C

**Use C++ when:**
- Building large applications
- Need object-oriented design
- Using STL containers/algorithms
- Want modern abstractions (RAII, smart pointers)
- Game development
- GUI applications

## Exercises

### Exercise 1: Hello World Variations
Write programs that:
1. Print your name
2. Print multiple lines
3. Print special characters (`\t`, `\n`, `\\`, `\"`)

### Exercise 2: Compile with Different Flags
Compile a simple program with:
1. `-O0` (no optimization) and `-O3` (full optimization)
2. Check the file sizes
3. Add `-g` for debugging, note the size increase

### Exercise 3: Create a CMake Project
Set up a project with:
1. `CMakeLists.txt`
2. `main.cpp`
3. `utils.cpp` and `utils.hpp`
4. Build and run

### Exercise 4: Explore Compiler Explorer
1. Go to https://godbolt.org/
2. Write a simple function
3. Compare assembly output for `-O0` vs `-O3`
4. Compare GCC vs Clang output

## Next Steps

In the next module, we'll cover fundamentals:
- Data types and variables
- Operators and expressions
- Type conversions
- Constants and literals
- Input/Output

---

**Practice Tip:** Set up your development environment now. Having a working compiler and editor is essential before proceeding!
