# 🔧 Complete C/C++ Course - From Zero to Master

Welcome to the complete C/C++ course! This course will take you from basic programming concepts to advanced systems programming with modern C23 and C++23/C++26 standards.

## 📚 Course Index

### **[Module 1: Introduction to C/C++](module-01-introduction/README.md)**

* History and evolution of C and C++
* C23 and C++23/C++26 standards overview
* Setting up your development environment
* Your first programs in C and C++
* Compilation process and build systems

### **[Module 2: Fundamentals](module-02-fundamentals/README.md)**

* Data types and variables
* Operators and expressions
* Type conversions and casting
* Constants and literals (including C23/C++23 features)
* Input/Output basics

### **[Module 3: Control Flow & Functions](module-03-control-flow-functions/README.md)**

* Conditional statements
* Loops and iteration
* Functions and parameter passing
* Recursion
* Function overloading (C++)

### **[Module 4: Pointers & Memory Management](module-04-pointers-memory/README.md)**

* Understanding pointers
* Pointer arithmetic
* Dynamic memory allocation
* Memory safety and common pitfalls
* Smart pointers introduction (C++)

### **[Module 5: Object-Oriented Programming (C++)](module-05-oop-cpp/README.md)**

* Classes and objects
* Constructors and destructors
* Inheritance and polymorphism
* Access specifiers and encapsulation
* Virtual functions and abstract classes

### **[Module 6: Modern C++ Features](module-06-modern-cpp/README.md)**

* C++11/14/17/20 essentials
* C++23 new features (deducing this, etc.)
* C++26 preview (reflection, contracts)
* Move semantics and rvalue references
* Lambda expressions

### **[Module 7: STL & Standard Library](module-07-stl-standard-library/README.md)**

* Containers (vector, map, set, etc.)
* Iterators and algorithms
* Strings and string_view
* Ranges library (C++20/23)
* std::format and std::print (C++23)

### **[Module 8: Advanced Topics](module-08-advanced-topics/README.md)**

* Templates and generic programming
* RAII and resource management
* Smart pointers deep dive
* Concurrency and multithreading
* Constexpr and compile-time programming

### **[Module 9: Practical Projects](module-09-practical-projects/README.md)**

* Progressive exercises
* Real-world applications
* Systems programming examples
* Portfolio-ready projects

## 🚀 How to Use This Course

### Option 1: Using Containers (Recommended)
Each module has a pre-configured container environment:

```bash
# Build and run the development container
cd containers
podman build -f Dockerfile.dev -t cpp-course-dev .
podman run -it --rm -v $(pwd)/..:/app:Z cpp-course-dev

# Or use the multi-service setup
podman-compose up -d
```

### Option 2: Local Development
1. **Read each module in order** - The course is designed to be progressive
2. **Compile and run all examples** - Practice is fundamental
3. **Complete the exercises** - Each module has hands-on exercises
4. **Build the projects** - Apply what you learn in real applications

## 📋 Prerequisites

* Basic computer literacy
* Command line familiarity
* A modern computer (Windows, macOS, or Linux)
* At least 4GB RAM and 10GB disk space
* Willingness to dive deep into systems programming 😊

## 🎯 Course Objectives

Upon completing this course, you will be able to:

* ✅ Write efficient, safe C and C++ code
* ✅ Understand memory management and pointers
* ✅ Apply object-oriented design principles
* ✅ Use modern C++23 features effectively
* ✅ Work with the STL and standard library
* ✅ Write concurrent and multithreaded programs
* ✅ Build real-world systems and applications
* ✅ Debug and optimize C/C++ programs

---

## 📖 Let's Begin

👉 **[Go to Module 1: Introduction to C/C++](module-01-introduction/README.md)**

---

## 📚 Reference Guides

* **COURSE-INFO.md** - Complete course information and structure
* **QUICK-GUIDE.md** - C/C++ syntax cheat sheet
* **ADDITIONAL-RESOURCES.md** - Links, books, videos, community

---

## 💻 Practical Examples

The `examples/` directory contains ready-to-compile projects:

1. **CLI Calculator** (`examples/01-cli-calculator/`)
2. **Memory Manager** (`examples/02-memory-manager/`)
3. **Game Engine Basics** (`examples/03-game-engine-basics/`)

---

## 🛠️ Technology Stack Covered

* **C23** (ISO/IEC 9899:2024) - Latest C standard
* **C++23** (ISO/IEC 14882:2024) - Current C++ standard
* **C++26** (Preview) - Upcoming standard features
* **GCC 14+** / **Clang 18+** - Modern compilers
* **CMake** - Cross-platform build system
* **GDB/LLDB** - Debuggers
* **Valgrind/AddressSanitizer** - Memory analysis tools

---

## 🆕 What's New in C23 and C++23

### C23 Highlights
* `nullptr` keyword (finally!)
* `bool`, `true`, `false` as keywords
* `typeof` operator standardized
* `_BitInt(N)` for bit-precise integers
* `#embed` directive for binary data
* `[[nodiscard]]`, `[[deprecated]]` attributes
* Binary literals and digit separators

### C++23 Highlights
* `std::print` and `std::println`
* Deducing `this` for simpler member functions
* `std::expected` for error handling
* `std::mdspan` for multidimensional views
* `import std;` for modular standard library
* `std::flat_map` and `std::flat_set`
* Enhanced `constexpr` support

### C++26 Preview
* **Reflection** - Compile-time introspection
* **Contracts** - Pre/postconditions
* **Pattern Matching** - Expressive matching
* **std::simd** - Data-parallel types

---

_Remember: C and C++ reward deep understanding. Take your time with each concept, and don't hesitate to experiment!_
