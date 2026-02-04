# Example Project 3: Game Engine Basics

A foundational game engine demonstrating core architecture patterns including Entity-Component-System (ECS), event systems, and resource management.

## Features

- Entity-Component-System architecture
- Event/Message system
- Resource management with reference counting
- Simple transform hierarchy
- Basic game loop
- Input handling abstraction

## Project Structure

```
03-game-engine-basics/
├── README.md
├── CMakeLists.txt
├── include/
│   ├── engine.hpp
│   ├── ecs/
│   │   ├── entity.hpp
│   │   ├── component.hpp
│   │   ├── system.hpp
│   │   └── world.hpp
│   ├── events/
│   │   ├── event.hpp
│   │   └── event_dispatcher.hpp
│   ├── resources/
│   │   ├── resource.hpp
│   │   └── resource_manager.hpp
│   ├── core/
│   │   ├── time.hpp
│   │   └── input.hpp
│   └── math/
│       ├── vector.hpp
│       └── transform.hpp
├── src/
│   ├── main.cpp
│   └── engine.cpp
└── examples/
    └── simple_game.cpp
```

## Building

```bash
mkdir build && cd build
cmake ..
cmake --build .
./game_engine
```

## Code

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)
project(GameEngine VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(game_engine
    src/main.cpp
    src/engine.cpp
)

target_include_directories(game_engine PRIVATE include)
target_compile_options(game_engine PRIVATE -Wall -Wextra -Wpedantic)
```

### include/math/vector.hpp

```cpp
#pragma once
#include <cmath>
#include <iostream>

struct Vec2 {
    float x = 0, y = 0;
    
    Vec2() = default;
    Vec2(float x, float y) : x(x), y(y) {}
    
    Vec2 operator+(const Vec2& other) const { return {x + other.x, y + other.y}; }
    Vec2 operator-(const Vec2& other) const { return {x - other.x, y - other.y}; }
    Vec2 operator*(float s) const { return {x * s, y * s}; }
    Vec2 operator/(float s) const { return {x / s, y / s}; }
    
    Vec2& operator+=(const Vec2& other) { x += other.x; y += other.y; return *this; }
    Vec2& operator-=(const Vec2& other) { x -= other.x; y -= other.y; return *this; }
    Vec2& operator*=(float s) { x *= s; y *= s; return *this; }
    
    float length() const { return std::sqrt(x * x + y * y); }
    float length_squared() const { return x * x + y * y; }
    
    Vec2 normalized() const {
        float len = length();
        return len > 0 ? *this / len : Vec2{};
    }
    
    float dot(const Vec2& other) const { return x * other.x + y * other.y; }
    
    friend std::ostream& operator<<(std::ostream& os, const Vec2& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }
};

struct Vec3 {
    float x = 0, y = 0, z = 0;
    
    Vec3() = default;
    Vec3(float x, float y, float z) : x(x), y(y), z(z) {}
    
    Vec3 operator+(const Vec3& other) const { return {x + other.x, y + other.y, z + other.z}; }
    Vec3 operator-(const Vec3& other) const { return {x - other.x, y - other.y, z - other.z}; }
    Vec3 operator*(float s) const { return {x * s, y * s, z * s}; }
    Vec3 operator/(float s) const { return {x / s, y / s, z / s}; }
    
    Vec3& operator+=(const Vec3& other) { x += other.x; y += other.y; z += other.z; return *this; }
    
    float length() const { return std::sqrt(x * x + y * y + z * z); }
    Vec3 normalized() const {
        float len = length();
        return len > 0 ? *this / len : Vec3{};
    }
    
    float dot(const Vec3& other) const { return x * other.x + y * other.y + z * other.z; }
    
    Vec3 cross(const Vec3& other) const {
        return {
            y * other.z - z * other.y,
            z * other.x - x * other.z,
            x * other.y - y * other.x
        };
    }
    
    friend std::ostream& operator<<(std::ostream& os, const Vec3& v) {
        return os << "(" << v.x << ", " << v.y << ", " << v.z << ")";
    }
};
```

### include/math/transform.hpp

```cpp
#pragma once
#include "vector.hpp"
#include <vector>
#include <memory>

class Transform {
public:
    Vec3 position{0, 0, 0};
    Vec3 rotation{0, 0, 0};  // Euler angles in degrees
    Vec3 scale{1, 1, 1};
    
    Transform* parent = nullptr;
    std::vector<Transform*> children;
    
    void set_parent(Transform* new_parent) {
        if (parent) {
            auto& siblings = parent->children;
            siblings.erase(std::remove(siblings.begin(), siblings.end(), this), siblings.end());
        }
        parent = new_parent;
        if (parent) {
            parent->children.push_back(this);
        }
    }
    
    Vec3 world_position() const {
        if (parent) {
            return parent->world_position() + position;
        }
        return position;
    }
    
    void translate(const Vec3& delta) {
        position += delta;
    }
    
    void rotate(const Vec3& euler_delta) {
        rotation += euler_delta;
    }
};
```

### include/ecs/entity.hpp

```cpp
#pragma once
#include <cstdint>
#include <limits>

using EntityID = uint32_t;
constexpr EntityID INVALID_ENTITY = std::numeric_limits<EntityID>::max();

struct Entity {
    EntityID id = INVALID_ENTITY;
    bool active = true;
    
    bool valid() const { return id != INVALID_ENTITY; }
    
    bool operator==(const Entity& other) const { return id == other.id; }
    bool operator!=(const Entity& other) const { return id != other.id; }
};
```

### include/ecs/component.hpp

```cpp
#pragma once
#include "../math/transform.hpp"
#include "../math/vector.hpp"
#include <string>

using ComponentTypeID = uint32_t;

// Component type ID generator
inline ComponentTypeID next_component_type_id() {
    static ComponentTypeID id = 0;
    return id++;
}

template<typename T>
inline ComponentTypeID get_component_type_id() {
    static ComponentTypeID id = next_component_type_id();
    return id;
}

// Base component marker
struct Component {
    bool enabled = true;
};

// Common components
struct TransformComponent : Component {
    Transform transform;
};

struct TagComponent : Component {
    std::string tag;
    
    TagComponent() = default;
    explicit TagComponent(std::string t) : tag(std::move(t)) {}
};

struct VelocityComponent : Component {
    Vec3 velocity{0, 0, 0};
    Vec3 angular_velocity{0, 0, 0};
};

struct RenderComponent : Component {
    std::string mesh_id;
    std::string material_id;
    bool visible = true;
};

struct ColliderComponent : Component {
    enum class Type { Box, Sphere, Capsule };
    Type type = Type::Box;
    Vec3 size{1, 1, 1};
    Vec3 offset{0, 0, 0};
};

struct RigidbodyComponent : Component {
    float mass = 1.0f;
    float drag = 0.0f;
    Vec3 force{0, 0, 0};
    bool use_gravity = true;
    bool is_kinematic = false;
};

struct HealthComponent : Component {
    int current = 100;
    int max = 100;
    
    float percentage() const { return static_cast<float>(current) / max; }
    bool alive() const { return current > 0; }
    
    void damage(int amount) { current = std::max(0, current - amount); }
    void heal(int amount) { current = std::min(max, current + amount); }
};

struct ScriptComponent : Component {
    std::string script_name;
};
```

### include/ecs/system.hpp

```cpp
#pragma once
#include "entity.hpp"
#include <vector>

class World;

// Base system class
class System {
public:
    virtual ~System() = default;
    
    virtual void init(World& world) { (void)world; }
    virtual void update(World& world, float dt) = 0;
    virtual void shutdown(World& world) { (void)world; }
    
    bool enabled = true;
    int priority = 0;  // Lower = runs first
};

// Movement system
class MovementSystem : public System {
public:
    void update(World& world, float dt) override;
};

// Physics system
class PhysicsSystem : public System {
public:
    Vec3 gravity{0, -9.81f, 0};
    
    void update(World& world, float dt) override;
};

// Render system (placeholder)
class RenderSystem : public System {
public:
    void update(World& world, float dt) override;
};
```

### include/ecs/world.hpp

```cpp
#pragma once
#include "entity.hpp"
#include "component.hpp"
#include "system.hpp"
#include <vector>
#include <unordered_map>
#include <memory>
#include <typeindex>
#include <algorithm>

// Type-erased component storage
class IComponentPool {
public:
    virtual ~IComponentPool() = default;
    virtual void remove(EntityID id) = 0;
    virtual bool has(EntityID id) const = 0;
};

template<typename T>
class ComponentPool : public IComponentPool {
public:
    T& add(EntityID id, T component = T{}) {
        components_[id] = std::move(component);
        return components_[id];
    }
    
    void remove(EntityID id) override {
        components_.erase(id);
    }
    
    T* get(EntityID id) {
        auto it = components_.find(id);
        return it != components_.end() ? &it->second : nullptr;
    }
    
    bool has(EntityID id) const override {
        return components_.find(id) != components_.end();
    }
    
    auto begin() { return components_.begin(); }
    auto end() { return components_.end(); }

private:
    std::unordered_map<EntityID, T> components_;
};

class World {
public:
    // Entity management
    Entity create_entity() {
        Entity entity;
        if (!free_ids_.empty()) {
            entity.id = free_ids_.back();
            free_ids_.pop_back();
        } else {
            entity.id = next_id_++;
        }
        entities_.push_back(entity);
        return entity;
    }
    
    void destroy_entity(Entity entity) {
        for (auto& [type, pool] : component_pools_) {
            pool->remove(entity.id);
        }
        
        entities_.erase(
            std::remove_if(entities_.begin(), entities_.end(),
                [entity](const Entity& e) { return e.id == entity.id; }),
            entities_.end()
        );
        
        free_ids_.push_back(entity.id);
    }
    
    // Component management
    template<typename T, typename... Args>
    T& add_component(Entity entity, Args&&... args) {
        auto& pool = get_or_create_pool<T>();
        return pool.add(entity.id, T{std::forward<Args>(args)...});
    }
    
    template<typename T>
    void remove_component(Entity entity) {
        if (auto* pool = get_pool<T>()) {
            pool->remove(entity.id);
        }
    }
    
    template<typename T>
    T* get_component(Entity entity) {
        if (auto* pool = get_pool<T>()) {
            return pool->get(entity.id);
        }
        return nullptr;
    }
    
    template<typename T>
    bool has_component(Entity entity) const {
        if (auto* pool = get_pool<T>()) {
            return pool->has(entity.id);
        }
        return false;
    }
    
    // View entities with specific components
    template<typename... Components>
    std::vector<Entity> view() {
        std::vector<Entity> result;
        for (const auto& entity : entities_) {
            if ((has_component<Components>(entity) && ...)) {
                result.push_back(entity);
            }
        }
        return result;
    }
    
    // System management
    template<typename T, typename... Args>
    T& add_system(Args&&... args) {
        auto system = std::make_unique<T>(std::forward<Args>(args)...);
        T& ref = *system;
        systems_.push_back(std::move(system));
        
        // Sort by priority
        std::sort(systems_.begin(), systems_.end(),
            [](const auto& a, const auto& b) {
                return a->priority < b->priority;
            });
        
        return ref;
    }
    
    void update(float dt) {
        for (auto& system : systems_) {
            if (system->enabled) {
                system->update(*this, dt);
            }
        }
    }
    
    void init() {
        for (auto& system : systems_) {
            system->init(*this);
        }
    }
    
    void shutdown() {
        for (auto& system : systems_) {
            system->shutdown(*this);
        }
    }
    
    const std::vector<Entity>& entities() const { return entities_; }

private:
    std::vector<Entity> entities_;
    std::vector<EntityID> free_ids_;
    EntityID next_id_ = 0;
    
    std::unordered_map<std::type_index, std::unique_ptr<IComponentPool>> component_pools_;
    std::vector<std::unique_ptr<System>> systems_;
    
    template<typename T>
    ComponentPool<T>* get_pool() {
        auto it = component_pools_.find(typeid(T));
        if (it != component_pools_.end()) {
            return static_cast<ComponentPool<T>*>(it->second.get());
        }
        return nullptr;
    }
    
    template<typename T>
    const ComponentPool<T>* get_pool() const {
        auto it = component_pools_.find(typeid(T));
        if (it != component_pools_.end()) {
            return static_cast<const ComponentPool<T>*>(it->second.get());
        }
        return nullptr;
    }
    
    template<typename T>
    ComponentPool<T>& get_or_create_pool() {
        auto it = component_pools_.find(typeid(T));
        if (it != component_pools_.end()) {
            return *static_cast<ComponentPool<T>*>(it->second.get());
        }
        
        auto pool = std::make_unique<ComponentPool<T>>();
        ComponentPool<T>& ref = *pool;
        component_pools_[typeid(T)] = std::move(pool);
        return ref;
    }
};
```

### include/events/event.hpp

```cpp
#pragma once
#include <cstdint>
#include <any>
#include <string>

using EventTypeID = uint32_t;

inline EventTypeID next_event_type_id() {
    static EventTypeID id = 0;
    return id++;
}

template<typename T>
inline EventTypeID get_event_type_id() {
    static EventTypeID id = next_event_type_id();
    return id;
}

struct Event {
    virtual ~Event() = default;
    virtual EventTypeID type() const = 0;
    bool handled = false;
};

// Common events
struct WindowResizeEvent : Event {
    int width, height;
    
    WindowResizeEvent(int w, int h) : width(w), height(h) {}
    EventTypeID type() const override { return get_event_type_id<WindowResizeEvent>(); }
};

struct KeyEvent : Event {
    int key;
    bool pressed;
    
    KeyEvent(int k, bool p) : key(k), pressed(p) {}
    EventTypeID type() const override { return get_event_type_id<KeyEvent>(); }
};

struct MouseMoveEvent : Event {
    float x, y;
    float dx, dy;
    
    MouseMoveEvent(float x, float y, float dx, float dy) 
        : x(x), y(y), dx(dx), dy(dy) {}
    EventTypeID type() const override { return get_event_type_id<MouseMoveEvent>(); }
};

struct CollisionEvent : Event {
    EntityID entity_a;
    EntityID entity_b;
    Vec3 contact_point;
    
    CollisionEvent(EntityID a, EntityID b, Vec3 point)
        : entity_a(a), entity_b(b), contact_point(point) {}
    EventTypeID type() const override { return get_event_type_id<CollisionEvent>(); }
};

struct EntityDestroyedEvent : Event {
    EntityID entity;
    
    explicit EntityDestroyedEvent(EntityID e) : entity(e) {}
    EventTypeID type() const override { return get_event_type_id<EntityDestroyedEvent>(); }
};
```

### include/events/event_dispatcher.hpp

```cpp
#pragma once
#include "event.hpp"
#include <functional>
#include <unordered_map>
#include <vector>
#include <queue>
#include <memory>

using EventHandler = std::function<void(Event&)>;

class EventDispatcher {
public:
    template<typename T>
    void subscribe(std::function<void(T&)> handler) {
        auto wrapper = [handler](Event& e) {
            handler(static_cast<T&>(e));
        };
        handlers_[get_event_type_id<T>()].push_back(wrapper);
    }
    
    void dispatch(Event& event) {
        auto it = handlers_.find(event.type());
        if (it != handlers_.end()) {
            for (auto& handler : it->second) {
                if (!event.handled) {
                    handler(event);
                }
            }
        }
    }
    
    void queue(std::unique_ptr<Event> event) {
        event_queue_.push(std::move(event));
    }
    
    void process_queue() {
        while (!event_queue_.empty()) {
            auto event = std::move(event_queue_.front());
            event_queue_.pop();
            dispatch(*event);
        }
    }
    
    void clear() {
        handlers_.clear();
        while (!event_queue_.empty()) event_queue_.pop();
    }

private:
    std::unordered_map<EventTypeID, std::vector<EventHandler>> handlers_;
    std::queue<std::unique_ptr<Event>> event_queue_;
};
```

### include/engine.hpp

```cpp
#pragma once
#include "ecs/world.hpp"
#include "events/event_dispatcher.hpp"
#include <chrono>

class Engine {
public:
    Engine();
    ~Engine();
    
    void init();
    void run();
    void shutdown();
    void stop() { running_ = false; }
    
    World& world() { return world_; }
    EventDispatcher& events() { return events_; }
    
    float delta_time() const { return delta_time_; }
    float time() const { return time_; }
    int fps() const { return fps_; }

private:
    World world_;
    EventDispatcher events_;
    
    bool running_ = false;
    float delta_time_ = 0.0f;
    float time_ = 0.0f;
    int fps_ = 0;
    
    using Clock = std::chrono::high_resolution_clock;
    Clock::time_point last_frame_time_;
};
```

### src/main.cpp

```cpp
#include "engine.hpp"
#include <iostream>

// Custom system for demo
class PrintSystem : public System {
public:
    PrintSystem() { priority = 100; }
    
    void update(World& world, float dt) override {
        frame_count_++;
        time_accumulator_ += dt;
        
        if (time_accumulator_ >= 1.0f) {
            std::cout << "FPS: " << frame_count_ << "\n";
            
            auto entities = world.view<TransformComponent, TagComponent>();
            std::cout << "Entities with Transform+Tag: " << entities.size() << "\n";
            
            for (auto entity : entities) {
                auto* tag = world.get_component<TagComponent>(entity);
                auto* transform = world.get_component<TransformComponent>(entity);
                
                std::cout << "  " << tag->tag << " at " 
                          << transform->transform.position << "\n";
            }
            
            frame_count_ = 0;
            time_accumulator_ -= 1.0f;
        }
    }

private:
    int frame_count_ = 0;
    float time_accumulator_ = 0.0f;
};

int main() {
    std::cout << "=== Game Engine Basics Demo ===\n\n";
    
    Engine engine;
    engine.init();
    
    // Add systems
    engine.world().add_system<MovementSystem>();
    engine.world().add_system<PhysicsSystem>();
    engine.world().add_system<PrintSystem>();
    
    // Create some entities
    {
        auto player = engine.world().create_entity();
        engine.world().add_component<TagComponent>(player, "Player");
        auto& transform = engine.world().add_component<TransformComponent>(player);
        transform.transform.position = {0, 0, 0};
        auto& velocity = engine.world().add_component<VelocityComponent>(player);
        velocity.velocity = {1, 0, 0};
    }
    
    {
        auto enemy = engine.world().create_entity();
        engine.world().add_component<TagComponent>(enemy, "Enemy");
        auto& transform = engine.world().add_component<TransformComponent>(enemy);
        transform.transform.position = {10, 0, 0};
        auto& velocity = engine.world().add_component<VelocityComponent>(enemy);
        velocity.velocity = {-0.5f, 0, 0};
        engine.world().add_component<HealthComponent>(enemy);
    }
    
    // Subscribe to events
    engine.events().subscribe<CollisionEvent>([](CollisionEvent& e) {
        std::cout << "Collision between " << e.entity_a 
                  << " and " << e.entity_b << "\n";
    });
    
    // Run for a few seconds
    std::cout << "Running for 3 seconds...\n\n";
    
    auto start = std::chrono::steady_clock::now();
    while (std::chrono::steady_clock::now() - start < std::chrono::seconds(3)) {
        // Simulate frame
        std::this_thread::sleep_for(std::chrono::milliseconds(16));  // ~60 FPS
        engine.world().update(0.016f);
        engine.events().process_queue();
    }
    
    engine.shutdown();
    
    std::cout << "\nEngine shutdown complete.\n";
    return 0;
}
```

## Learning Objectives

1. **ECS Architecture** - Composition over inheritance
2. **Data-Oriented Design** - Cache-friendly data layouts
3. **Event Systems** - Decoupled communication
4. **Game Loops** - Fixed timestep, frame timing
5. **Resource Management** - Handle-based systems

## Architecture Patterns

### Entity-Component-System Benefits
- Flexible composition
- Easy serialization
- Cache-friendly iteration
- Parallel processing ready

### Event System Benefits
- Decoupled systems
- Easy debugging
- Replay capability
- Network synchronization

## Extensions

- Add scene serialization (JSON/binary)
- Implement spatial partitioning
- Add scripting integration (Lua)
- Implement rendering abstraction
- Add audio system
- Network replication

## Usage Example

```cpp
// Create a game using the engine
Engine engine;
engine.init();

// Add custom game systems
engine.world().add_system<AISystem>();
engine.world().add_system<CombatSystem>();

// Spawn game objects
auto hero = spawn_hero(engine.world());
spawn_enemies(engine.world(), 10);

// Run game loop
engine.run();
```
