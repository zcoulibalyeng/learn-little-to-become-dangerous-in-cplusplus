# Module 9: Practical Projects

## Overview

This module provides hands-on projects that combine concepts from all previous modules. Each project is designed to reinforce specific skills while building something practical.

## Project 1: Command-Line Calculator

### Requirements
- Parse mathematical expressions
- Support +, -, *, /, % operations
- Handle parentheses
- Proper error handling
- History feature

### Implementation

```cpp
// calculator.hpp
#pragma once
#include <string>
#include <stack>
#include <vector>
#include <stdexcept>
#include <cctype>
#include <cmath>

class Calculator {
public:
    double evaluate(const std::string& expression);
    const std::vector<std::string>& history() const { return history_; }
    void clear_history() { history_.clear(); }

private:
    std::vector<std::string> history_;
    
    std::vector<std::string> tokenize(const std::string& expr);
    std::vector<std::string> to_postfix(const std::vector<std::string>& tokens);
    double evaluate_postfix(const std::vector<std::string>& postfix);
    int precedence(const std::string& op);
    bool is_operator(const std::string& token);
    double apply_operator(const std::string& op, double a, double b);
};

// calculator.cpp
#include "calculator.hpp"
#include <sstream>

std::vector<std::string> Calculator::tokenize(const std::string& expr) {
    std::vector<std::string> tokens;
    std::string current;
    
    for (size_t i = 0; i < expr.length(); ++i) {
        char c = expr[i];
        
        if (std::isspace(c)) {
            continue;
        }
        
        if (std::isdigit(c) || c == '.') {
            current += c;
        } else {
            if (!current.empty()) {
                tokens.push_back(current);
                current.clear();
            }
            tokens.push_back(std::string(1, c));
        }
    }
    
    if (!current.empty()) {
        tokens.push_back(current);
    }
    
    return tokens;
}

int Calculator::precedence(const std::string& op) {
    if (op == "+" || op == "-") return 1;
    if (op == "*" || op == "/" || op == "%") return 2;
    if (op == "^") return 3;
    return 0;
}

bool Calculator::is_operator(const std::string& token) {
    return token == "+" || token == "-" || token == "*" || 
           token == "/" || token == "%" || token == "^";
}

std::vector<std::string> Calculator::to_postfix(const std::vector<std::string>& tokens) {
    std::vector<std::string> output;
    std::stack<std::string> operators;
    
    for (const auto& token : tokens) {
        if (std::isdigit(token[0]) || (token.length() > 1 && token[0] == '.')) {
            output.push_back(token);
        } else if (is_operator(token)) {
            while (!operators.empty() && 
                   operators.top() != "(" &&
                   precedence(operators.top()) >= precedence(token)) {
                output.push_back(operators.top());
                operators.pop();
            }
            operators.push(token);
        } else if (token == "(") {
            operators.push(token);
        } else if (token == ")") {
            while (!operators.empty() && operators.top() != "(") {
                output.push_back(operators.top());
                operators.pop();
            }
            if (operators.empty()) {
                throw std::runtime_error("Mismatched parentheses");
            }
            operators.pop();
        }
    }
    
    while (!operators.empty()) {
        if (operators.top() == "(") {
            throw std::runtime_error("Mismatched parentheses");
        }
        output.push_back(operators.top());
        operators.pop();
    }
    
    return output;
}

double Calculator::apply_operator(const std::string& op, double a, double b) {
    if (op == "+") return a + b;
    if (op == "-") return a - b;
    if (op == "*") return a * b;
    if (op == "/") {
        if (b == 0) throw std::runtime_error("Division by zero");
        return a / b;
    }
    if (op == "%") {
        if (b == 0) throw std::runtime_error("Modulo by zero");
        return std::fmod(a, b);
    }
    if (op == "^") return std::pow(a, b);
    throw std::runtime_error("Unknown operator: " + op);
}

double Calculator::evaluate_postfix(const std::vector<std::string>& postfix) {
    std::stack<double> values;
    
    for (const auto& token : postfix) {
        if (is_operator(token)) {
            if (values.size() < 2) {
                throw std::runtime_error("Invalid expression");
            }
            double b = values.top(); values.pop();
            double a = values.top(); values.pop();
            values.push(apply_operator(token, a, b));
        } else {
            values.push(std::stod(token));
        }
    }
    
    if (values.size() != 1) {
        throw std::runtime_error("Invalid expression");
    }
    
    return values.top();
}

double Calculator::evaluate(const std::string& expression) {
    auto tokens = tokenize(expression);
    auto postfix = to_postfix(tokens);
    double result = evaluate_postfix(postfix);
    
    std::ostringstream entry;
    entry << expression << " = " << result;
    history_.push_back(entry.str());
    
    return result;
}

// main.cpp
#include <iostream>
#include <string>
#include "calculator.hpp"

int main() {
    Calculator calc;
    std::string line;
    
    std::cout << "C++ Calculator (type 'quit' to exit, 'history' to see history)\n";
    std::cout << "Supports: +, -, *, /, %, ^ and parentheses\n\n";
    
    while (true) {
        std::cout << "> ";
        std::getline(std::cin, line);
        
        if (line == "quit" || line == "exit") {
            break;
        }
        
        if (line == "history") {
            for (const auto& entry : calc.history()) {
                std::cout << "  " << entry << "\n";
            }
            continue;
        }
        
        if (line == "clear") {
            calc.clear_history();
            std::cout << "History cleared\n";
            continue;
        }
        
        if (line.empty()) {
            continue;
        }
        
        try {
            double result = calc.evaluate(line);
            std::cout << "= " << result << "\n";
        } catch (const std::exception& e) {
            std::cout << "Error: " << e.what() << "\n";
        }
    }
    
    return 0;
}
```

### Build and Test

```bash
g++ -std=c++23 -Wall -Wextra -o calculator main.cpp calculator.cpp
./calculator
```

---

## Project 2: Memory Pool Allocator

### Requirements
- Pre-allocate memory pool
- O(1) allocation/deallocation
- No fragmentation for fixed-size blocks
- Thread-safe option

### Implementation

```cpp
// memory_pool.hpp
#pragma once
#include <cstddef>
#include <vector>
#include <mutex>
#include <memory>

template<typename T, size_t BlockSize = 4096>
class MemoryPool {
public:
    using value_type = T;
    
    MemoryPool() : current_block_(nullptr), current_slot_(nullptr),
                   last_slot_(nullptr), free_slots_(nullptr) {}
    
    ~MemoryPool() {
        // Free all allocated blocks
        Slot* curr = current_block_;
        while (curr != nullptr) {
            Slot* prev = curr->next;
            operator delete(reinterpret_cast<void*>(curr));
            curr = prev;
        }
    }
    
    // Non-copyable
    MemoryPool(const MemoryPool&) = delete;
    MemoryPool& operator=(const MemoryPool&) = delete;
    
    // Movable
    MemoryPool(MemoryPool&& other) noexcept {
        current_block_ = other.current_block_;
        current_slot_ = other.current_slot_;
        last_slot_ = other.last_slot_;
        free_slots_ = other.free_slots_;
        other.current_block_ = nullptr;
    }
    
    T* allocate() {
        if (free_slots_ != nullptr) {
            T* result = reinterpret_cast<T*>(free_slots_);
            free_slots_ = free_slots_->next;
            return result;
        }
        
        if (current_slot_ >= last_slot_) {
            allocate_block();
        }
        
        return reinterpret_cast<T*>(current_slot_++);
    }
    
    void deallocate(T* p) {
        if (p != nullptr) {
            reinterpret_cast<Slot*>(p)->next = free_slots_;
            free_slots_ = reinterpret_cast<Slot*>(p);
        }
    }
    
    template<typename... Args>
    T* construct(Args&&... args) {
        T* result = allocate();
        new (result) T(std::forward<Args>(args)...);
        return result;
    }
    
    void destroy(T* p) {
        p->~T();
        deallocate(p);
    }
    
    size_t block_size() const { return BlockSize; }
    size_t slot_size() const { return sizeof(Slot); }

private:
    union Slot {
        T element;
        Slot* next;
    };
    
    Slot* current_block_;
    Slot* current_slot_;
    Slot* last_slot_;
    Slot* free_slots_;
    
    size_t pad_pointer(char* p, size_t align) const {
        uintptr_t result = reinterpret_cast<uintptr_t>(p);
        return ((align - result) % align);
    }
    
    void allocate_block() {
        char* new_block = reinterpret_cast<char*>(
            operator new(BlockSize)
        );
        
        reinterpret_cast<Slot*>(new_block)->next = current_block_;
        current_block_ = reinterpret_cast<Slot*>(new_block);
        
        char* body = new_block + sizeof(Slot*);
        size_t body_padding = pad_pointer(body, alignof(Slot));
        
        current_slot_ = reinterpret_cast<Slot*>(body + body_padding);
        last_slot_ = reinterpret_cast<Slot*>(
            new_block + BlockSize - sizeof(Slot) + 1
        );
    }
};

// Thread-safe version
template<typename T, size_t BlockSize = 4096>
class ThreadSafeMemoryPool {
public:
    T* allocate() {
        std::lock_guard<std::mutex> lock(mutex_);
        return pool_.allocate();
    }
    
    void deallocate(T* p) {
        std::lock_guard<std::mutex> lock(mutex_);
        pool_.deallocate(p);
    }
    
    template<typename... Args>
    T* construct(Args&&... args) {
        std::lock_guard<std::mutex> lock(mutex_);
        return pool_.construct(std::forward<Args>(args)...);
    }
    
    void destroy(T* p) {
        std::lock_guard<std::mutex> lock(mutex_);
        pool_.destroy(p);
    }

private:
    MemoryPool<T, BlockSize> pool_;
    std::mutex mutex_;
};

// Usage example
#include <iostream>
#include <chrono>

struct TestObject {
    int data[10];
    TestObject() { std::fill(std::begin(data), std::end(data), 0); }
};

int main() {
    const int iterations = 1000000;
    
    // Standard allocation
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        auto* obj = new TestObject();
        delete obj;
    }
    auto end = std::chrono::high_resolution_clock::now();
    auto std_time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    std::cout << "Standard new/delete: " << std_time.count() << "ms\n";
    
    // Memory pool allocation
    MemoryPool<TestObject> pool;
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        auto* obj = pool.construct();
        pool.destroy(obj);
    }
    end = std::chrono::high_resolution_clock::now();
    auto pool_time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    std::cout << "Memory pool: " << pool_time.count() << "ms\n";
    
    std::cout << "Speedup: " << (double)std_time.count() / pool_time.count() << "x\n";
    
    return 0;
}
```

---

## Project 3: Entity Component System (ECS)

### Requirements
- Entity management
- Component storage
- System processing
- Cache-friendly design

### Implementation

```cpp
// ecs.hpp
#pragma once
#include <vector>
#include <unordered_map>
#include <typeindex>
#include <memory>
#include <bitset>
#include <queue>
#include <functional>

const size_t MAX_COMPONENTS = 32;

using Entity = uint32_t;
using ComponentMask = std::bitset<MAX_COMPONENTS>;

// Base component array interface
class IComponentArray {
public:
    virtual ~IComponentArray() = default;
    virtual void entity_destroyed(Entity entity) = 0;
};

// Typed component array
template<typename T>
class ComponentArray : public IComponentArray {
public:
    void insert(Entity entity, T component) {
        size_t new_index = size_;
        entity_to_index_[entity] = new_index;
        index_to_entity_[new_index] = entity;
        components_[new_index] = component;
        ++size_;
    }
    
    void remove(Entity entity) {
        if (entity_to_index_.find(entity) == entity_to_index_.end()) {
            return;
        }
        
        size_t removed_index = entity_to_index_[entity];
        size_t last_index = size_ - 1;
        components_[removed_index] = components_[last_index];
        
        Entity last_entity = index_to_entity_[last_index];
        entity_to_index_[last_entity] = removed_index;
        index_to_entity_[removed_index] = last_entity;
        
        entity_to_index_.erase(entity);
        index_to_entity_.erase(last_index);
        --size_;
    }
    
    T& get(Entity entity) {
        return components_[entity_to_index_[entity]];
    }
    
    bool has(Entity entity) const {
        return entity_to_index_.find(entity) != entity_to_index_.end();
    }
    
    void entity_destroyed(Entity entity) override {
        if (has(entity)) {
            remove(entity);
        }
    }

private:
    std::array<T, 1000> components_;
    std::unordered_map<Entity, size_t> entity_to_index_;
    std::unordered_map<size_t, Entity> index_to_entity_;
    size_t size_ = 0;
};

// Component manager
class ComponentManager {
public:
    template<typename T>
    void register_component() {
        std::type_index type = typeid(T);
        component_types_[type] = next_component_type_++;
        component_arrays_[type] = std::make_shared<ComponentArray<T>>();
    }
    
    template<typename T>
    size_t get_component_type() {
        return component_types_[typeid(T)];
    }
    
    template<typename T>
    void add_component(Entity entity, T component) {
        get_component_array<T>()->insert(entity, component);
    }
    
    template<typename T>
    void remove_component(Entity entity) {
        get_component_array<T>()->remove(entity);
    }
    
    template<typename T>
    T& get_component(Entity entity) {
        return get_component_array<T>()->get(entity);
    }
    
    template<typename T>
    bool has_component(Entity entity) {
        return get_component_array<T>()->has(entity);
    }
    
    void entity_destroyed(Entity entity) {
        for (auto& [type, array] : component_arrays_) {
            array->entity_destroyed(entity);
        }
    }

private:
    std::unordered_map<std::type_index, size_t> component_types_;
    std::unordered_map<std::type_index, std::shared_ptr<IComponentArray>> component_arrays_;
    size_t next_component_type_ = 0;
    
    template<typename T>
    std::shared_ptr<ComponentArray<T>> get_component_array() {
        return std::static_pointer_cast<ComponentArray<T>>(
            component_arrays_[typeid(T)]
        );
    }
};

// Entity manager
class EntityManager {
public:
    EntityManager() {
        for (Entity entity = 0; entity < 1000; ++entity) {
            available_entities_.push(entity);
        }
    }
    
    Entity create_entity() {
        Entity id = available_entities_.front();
        available_entities_.pop();
        ++living_entity_count_;
        return id;
    }
    
    void destroy_entity(Entity entity) {
        signatures_[entity].reset();
        available_entities_.push(entity);
        --living_entity_count_;
    }
    
    void set_signature(Entity entity, ComponentMask signature) {
        signatures_[entity] = signature;
    }
    
    ComponentMask get_signature(Entity entity) {
        return signatures_[entity];
    }

private:
    std::queue<Entity> available_entities_;
    std::array<ComponentMask, 1000> signatures_;
    uint32_t living_entity_count_ = 0;
};

// World - main ECS coordinator
class World {
public:
    void init() {
        entity_manager_ = std::make_unique<EntityManager>();
        component_manager_ = std::make_unique<ComponentManager>();
    }
    
    Entity create_entity() {
        return entity_manager_->create_entity();
    }
    
    void destroy_entity(Entity entity) {
        entity_manager_->destroy_entity(entity);
        component_manager_->entity_destroyed(entity);
    }
    
    template<typename T>
    void register_component() {
        component_manager_->register_component<T>();
    }
    
    template<typename T>
    void add_component(Entity entity, T component) {
        component_manager_->add_component<T>(entity, component);
        
        auto signature = entity_manager_->get_signature(entity);
        signature.set(component_manager_->get_component_type<T>(), true);
        entity_manager_->set_signature(entity, signature);
    }
    
    template<typename T>
    T& get_component(Entity entity) {
        return component_manager_->get_component<T>(entity);
    }
    
    template<typename T>
    bool has_component(Entity entity) {
        return component_manager_->has_component<T>(entity);
    }

private:
    std::unique_ptr<EntityManager> entity_manager_;
    std::unique_ptr<ComponentManager> component_manager_;
};

// Example components
struct Position {
    float x, y, z;
};

struct Velocity {
    float dx, dy, dz;
};

struct Health {
    int current, max;
};

// Example system
void movement_system(World& world, const std::vector<Entity>& entities, float dt) {
    for (Entity entity : entities) {
        if (world.has_component<Position>(entity) && 
            world.has_component<Velocity>(entity)) {
            auto& pos = world.get_component<Position>(entity);
            auto& vel = world.get_component<Velocity>(entity);
            
            pos.x += vel.dx * dt;
            pos.y += vel.dy * dt;
            pos.z += vel.dz * dt;
        }
    }
}

// Usage
#include <iostream>

int main() {
    World world;
    world.init();
    
    // Register components
    world.register_component<Position>();
    world.register_component<Velocity>();
    world.register_component<Health>();
    
    // Create entities
    std::vector<Entity> entities;
    for (int i = 0; i < 100; ++i) {
        Entity e = world.create_entity();
        world.add_component(e, Position{0.0f, 0.0f, 0.0f});
        world.add_component(e, Velocity{1.0f, 0.5f, 0.0f});
        if (i % 2 == 0) {
            world.add_component(e, Health{100, 100});
        }
        entities.push_back(e);
    }
    
    // Run systems
    float dt = 1.0f / 60.0f;  // 60 FPS
    for (int frame = 0; frame < 60; ++frame) {
        movement_system(world, entities, dt);
    }
    
    // Check results
    auto& pos = world.get_component<Position>(entities[0]);
    std::cout << "Entity 0 position after 1 second: ("
              << pos.x << ", " << pos.y << ", " << pos.z << ")\n";
    
    return 0;
}
```

---

## Project 4: Simple HTTP Client

### Requirements
- TCP socket connection
- HTTP/1.1 GET requests
- Response parsing
- Error handling

```cpp
// http_client.hpp
#pragma once
#include <string>
#include <map>
#include <optional>
#include <sstream>

#ifdef _WIN32
    #include <winsock2.h>
    #include <ws2tcpip.h>
    #pragma comment(lib, "ws2_32.lib")
#else
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <netdb.h>
    #include <unistd.h>
    #define SOCKET int
    #define INVALID_SOCKET -1
    #define SOCKET_ERROR -1
    #define closesocket close
#endif

struct HttpResponse {
    int status_code;
    std::string status_text;
    std::map<std::string, std::string> headers;
    std::string body;
};

class HttpClient {
public:
    HttpClient() {
#ifdef _WIN32
        WSADATA wsaData;
        WSAStartup(MAKEWORD(2, 2), &wsaData);
#endif
    }
    
    ~HttpClient() {
#ifdef _WIN32
        WSACleanup();
#endif
    }
    
    std::optional<HttpResponse> get(const std::string& host, 
                                     const std::string& path = "/",
                                     int port = 80) {
        SOCKET sock = socket(AF_INET, SOCK_STREAM, 0);
        if (sock == INVALID_SOCKET) {
            return std::nullopt;
        }
        
        // Resolve hostname
        struct hostent* server = gethostbyname(host.c_str());
        if (server == nullptr) {
            closesocket(sock);
            return std::nullopt;
        }
        
        // Connect
        struct sockaddr_in server_addr{};
        server_addr.sin_family = AF_INET;
        server_addr.sin_port = htons(port);
        memcpy(&server_addr.sin_addr.s_addr, server->h_addr, server->h_length);
        
        if (connect(sock, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
            closesocket(sock);
            return std::nullopt;
        }
        
        // Send request
        std::ostringstream request;
        request << "GET " << path << " HTTP/1.1\r\n";
        request << "Host: " << host << "\r\n";
        request << "Connection: close\r\n";
        request << "\r\n";
        
        std::string req_str = request.str();
        send(sock, req_str.c_str(), req_str.length(), 0);
        
        // Receive response
        std::string response;
        char buffer[4096];
        int bytes_received;
        
        while ((bytes_received = recv(sock, buffer, sizeof(buffer) - 1, 0)) > 0) {
            buffer[bytes_received] = '\0';
            response += buffer;
        }
        
        closesocket(sock);
        
        return parse_response(response);
    }

private:
    std::optional<HttpResponse> parse_response(const std::string& raw) {
        HttpResponse response;
        
        size_t header_end = raw.find("\r\n\r\n");
        if (header_end == std::string::npos) {
            return std::nullopt;
        }
        
        std::string headers = raw.substr(0, header_end);
        response.body = raw.substr(header_end + 4);
        
        // Parse status line
        size_t first_line_end = headers.find("\r\n");
        std::string status_line = headers.substr(0, first_line_end);
        
        size_t space1 = status_line.find(' ');
        size_t space2 = status_line.find(' ', space1 + 1);
        
        response.status_code = std::stoi(status_line.substr(space1 + 1, space2 - space1 - 1));
        response.status_text = status_line.substr(space2 + 1);
        
        // Parse headers
        std::istringstream header_stream(headers.substr(first_line_end + 2));
        std::string line;
        while (std::getline(header_stream, line) && line != "\r") {
            size_t colon = line.find(':');
            if (colon != std::string::npos) {
                std::string key = line.substr(0, colon);
                std::string value = line.substr(colon + 2);
                if (!value.empty() && value.back() == '\r') {
                    value.pop_back();
                }
                response.headers[key] = value;
            }
        }
        
        return response;
    }
};

// Usage
#include <iostream>

int main() {
    HttpClient client;
    
    auto response = client.get("example.com", "/");
    
    if (response) {
        std::cout << "Status: " << response->status_code 
                  << " " << response->status_text << "\n\n";
        
        std::cout << "Headers:\n";
        for (const auto& [key, value] : response->headers) {
            std::cout << "  " << key << ": " << value << "\n";
        }
        
        std::cout << "\nBody (first 500 chars):\n";
        std::cout << response->body.substr(0, 500) << "...\n";
    } else {
        std::cout << "Request failed\n";
    }
    
    return 0;
}
```

---

## Portfolio Project Ideas

### Beginner Level
1. **Todo List CLI** - CRUD operations, file persistence
2. **Number Guessing Game** - Random numbers, user input
3. **Unit Converter** - Multiple units, accurate conversions
4. **Password Generator** - Configurable strength, character sets

### Intermediate Level
1. **Text Editor** - ncurses/terminal UI, file operations
2. **JSON Parser** - Recursive descent, proper error handling
3. **Image Processor** - Basic filters, format conversion
4. **Chat Application** - Sockets, multiple clients

### Advanced Level
1. **Database Engine** - B-tree indexes, query parsing
2. **Compiler/Interpreter** - Lexer, parser, code generation
3. **Game Engine** - Rendering, physics, ECS
4. **Operating System Kernel** - Memory management, scheduling

---

## Best Practices Summary

### Code Organization
```
project/
├── CMakeLists.txt
├── README.md
├── include/
│   └── project/
│       ├── module1.hpp
│       └── module2.hpp
├── src/
│   ├── main.cpp
│   ├── module1.cpp
│   └── module2.cpp
├── tests/
│   ├── test_module1.cpp
│   └── test_module2.cpp
└── docs/
    └── api.md
```

### CMake Template
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Add sanitizers for Debug
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_options(-fsanitize=address,undefined)
    add_link_options(-fsanitize=address,undefined)
endif()

# Warnings
add_compile_options(-Wall -Wextra -Wpedantic)

# Library
add_library(mylib
    src/module1.cpp
    src/module2.cpp
)
target_include_directories(mylib PUBLIC include)

# Executable
add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)

# Tests
enable_testing()
add_executable(tests tests/test_module1.cpp)
target_link_libraries(tests PRIVATE mylib)
add_test(NAME MyTests COMMAND tests)
```

### Error Handling Patterns

```cpp
// Use std::expected (C++23)
std::expected<Result, Error> operation() {
    if (failed) {
        return std::unexpected(Error{"reason"});
    }
    return Result{};
}

// Or exceptions for truly exceptional cases
class CustomException : public std::runtime_error {
public:
    CustomException(const std::string& msg) 
        : std::runtime_error(msg) {}
};
```

---

## Final Tips

1. **Start Simple** - Get basic functionality working first
2. **Test Early** - Write tests as you develop
3. **Profile** - Use profilers to find bottlenecks
4. **Document** - Write clear comments and documentation
5. **Review** - Read your code critically
6. **Refactor** - Improve code quality continuously
7. **Learn** - Study others' code and best practices

---

**Congratulations on completing the C/C++ course! 🎉**

You now have the knowledge to build sophisticated systems. Keep practicing and building projects!
