# Module 5: Object-Oriented Programming (C++)

## Classes and Objects

### Basic Class Definition

```cpp
#include <iostream>
#include <string>

class Person {
public:
    // Member variables (attributes)
    std::string name;
    int age;
    
    // Member function (method)
    void introduce() {
        std::cout << "Hi, I'm " << name << ", " << age << " years old.\n";
    }
    
    void birthday() {
        age++;
        std::cout << name << " is now " << age << "!\n";
    }
};

int main() {
    // Create object (instance)
    Person alice;
    alice.name = "Alice";
    alice.age = 30;
    alice.introduce();
    alice.birthday();
    
    // Another object
    Person bob;
    bob.name = "Bob";
    bob.age = 25;
    bob.introduce();
    
    return 0;
}
```

### Access Specifiers

```cpp
#include <iostream>
#include <string>

class BankAccount {
private:
    // Only accessible within this class
    std::string account_number_;
    double balance_;

protected:
    // Accessible in this class and derived classes
    std::string owner_name_;

public:
    // Accessible everywhere
    
    // Constructor
    BankAccount(const std::string& owner, double initial_balance)
        : owner_name_(owner), balance_(initial_balance) {
        static int counter = 1000;
        account_number_ = "ACC" + std::to_string(++counter);
    }
    
    // Getters (accessors)
    double get_balance() const {
        return balance_;
    }
    
    std::string get_owner() const {
        return owner_name_;
    }
    
    // Methods
    void deposit(double amount) {
        if (amount > 0) {
            balance_ += amount;
            std::cout << "Deposited $" << amount << "\n";
        }
    }
    
    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance_) {
            balance_ -= amount;
            std::cout << "Withdrew $" << amount << "\n";
            return true;
        }
        std::cout << "Insufficient funds\n";
        return false;
    }
};

int main() {
    BankAccount account("Alice", 1000.0);
    
    account.deposit(500);
    account.withdraw(200);
    
    std::cout << "Balance: $" << account.get_balance() << "\n";
    
    // account.balance_ = 1000000;  // ERROR: private
    
    return 0;
}
```

## Constructors and Destructors

### Types of Constructors

```cpp
#include <iostream>
#include <string>

class Widget {
public:
    // Default constructor
    Widget() : id_(++counter_), name_("Unnamed") {
        std::cout << "Default constructor: Widget " << id_ << "\n";
    }
    
    // Parameterized constructor
    Widget(const std::string& name) : id_(++counter_), name_(name) {
        std::cout << "Param constructor: Widget " << id_ << " (" << name_ << ")\n";
    }
    
    // Copy constructor
    Widget(const Widget& other) : id_(++counter_), name_(other.name_ + " (copy)") {
        std::cout << "Copy constructor: Widget " << id_ << " from " << other.id_ << "\n";
    }
    
    // Move constructor (C++11)
    Widget(Widget&& other) noexcept : id_(++counter_), name_(std::move(other.name_)) {
        std::cout << "Move constructor: Widget " << id_ << " from " << other.id_ << "\n";
        other.name_ = "Moved";
    }
    
    // Copy assignment operator
    Widget& operator=(const Widget& other) {
        if (this != &other) {
            name_ = other.name_ + " (assigned)";
            std::cout << "Copy assignment: Widget " << id_ << " from " << other.id_ << "\n";
        }
        return *this;
    }
    
    // Move assignment operator (C++11)
    Widget& operator=(Widget&& other) noexcept {
        if (this != &other) {
            name_ = std::move(other.name_);
            other.name_ = "Moved";
            std::cout << "Move assignment: Widget " << id_ << " from " << other.id_ << "\n";
        }
        return *this;
    }
    
    // Destructor
    ~Widget() {
        std::cout << "Destructor: Widget " << id_ << " (" << name_ << ")\n";
    }
    
    void info() const {
        std::cout << "Widget " << id_ << ": " << name_ << "\n";
    }

private:
    int id_;
    std::string name_;
    static int counter_;
};

int Widget::counter_ = 0;

int main() {
    Widget w1;                    // Default constructor
    Widget w2("Custom");          // Parameterized constructor
    Widget w3 = w2;               // Copy constructor
    Widget w4 = std::move(w2);    // Move constructor
    
    w1 = w3;                      // Copy assignment
    w1 = std::move(w4);           // Move assignment
    
    return 0;
}  // Destructors called in reverse order
```

### Member Initializer List

```cpp
#include <iostream>
#include <string>

class Rectangle {
public:
    // Using member initializer list (preferred)
    Rectangle(double w, double h)
        : width_(w)        // Initialize width_
        , height_(h)       // Initialize height_
        , area_(w * h)     // Initialize area_
    {
        std::cout << "Rectangle created: " << width_ << "x" << height_ << "\n";
    }
    
    // Without initializer list (less efficient for complex types)
    // Rectangle(double w, double h) {
    //     width_ = w;   // Assignment, not initialization
    //     height_ = h;
    //     area_ = w * h;
    // }
    
    double area() const { return area_; }

private:
    const double width_;   // const members MUST use initializer list
    const double height_;
    double area_;
};

// Reference members also require initializer list
class Logger {
public:
    Logger(std::ostream& os) : output_(os) {}
    
    void log(const std::string& msg) {
        output_ << "[LOG] " << msg << "\n";
    }

private:
    std::ostream& output_;  // Reference member
};

int main() {
    Rectangle rect(10, 5);
    std::cout << "Area: " << rect.area() << "\n";
    
    Logger logger(std::cout);
    logger.log("Hello!");
    
    return 0;
}
```

### Rule of Five (C++11)

```cpp
#include <iostream>
#include <cstring>
#include <utility>

class String {
public:
    // Constructor
    String(const char* str = "") {
        size_ = std::strlen(str);
        data_ = new char[size_ + 1];
        std::strcpy(data_, str);
        std::cout << "Constructor: \"" << data_ << "\"\n";
    }
    
    // 1. Destructor
    ~String() {
        std::cout << "Destructor: \"" << (data_ ? data_ : "null") << "\"\n";
        delete[] data_;
    }
    
    // 2. Copy Constructor
    String(const String& other) {
        size_ = other.size_;
        data_ = new char[size_ + 1];
        std::strcpy(data_, other.data_);
        std::cout << "Copy Constructor: \"" << data_ << "\"\n";
    }
    
    // 3. Copy Assignment
    String& operator=(const String& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new char[size_ + 1];
            std::strcpy(data_, other.data_);
            std::cout << "Copy Assignment: \"" << data_ << "\"\n";
        }
        return *this;
    }
    
    // 4. Move Constructor
    String(String&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
        std::cout << "Move Constructor: \"" << data_ << "\"\n";
    }
    
    // 5. Move Assignment
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
            std::cout << "Move Assignment: \"" << data_ << "\"\n";
        }
        return *this;
    }
    
    const char* c_str() const { return data_; }
    size_t size() const { return size_; }

private:
    char* data_;
    size_t size_;
};
```

## Inheritance

### Basic Inheritance

```cpp
#include <iostream>
#include <string>

class Animal {
public:
    Animal(const std::string& name) : name_(name) {}
    
    void eat() {
        std::cout << name_ << " is eating.\n";
    }
    
    void sleep() {
        std::cout << name_ << " is sleeping.\n";
    }

protected:
    std::string name_;
};

class Dog : public Animal {
public:
    Dog(const std::string& name, const std::string& breed)
        : Animal(name), breed_(breed) {}
    
    void bark() {
        std::cout << name_ << " the " << breed_ << " says: Woof!\n";
    }
    
    void fetch() {
        std::cout << name_ << " is fetching the ball!\n";
    }

private:
    std::string breed_;
};

class Cat : public Animal {
public:
    Cat(const std::string& name) : Animal(name) {}
    
    void meow() {
        std::cout << name_ << " says: Meow!\n";
    }
    
    void scratch() {
        std::cout << name_ << " is scratching!\n";
    }
};

int main() {
    Dog buddy("Buddy", "Golden Retriever");
    buddy.eat();      // Inherited from Animal
    buddy.sleep();    // Inherited from Animal
    buddy.bark();     // Dog-specific
    buddy.fetch();    // Dog-specific
    
    Cat whiskers("Whiskers");
    whiskers.eat();
    whiskers.meow();
    
    return 0;
}
```

### Inheritance Access Specifiers

```cpp
class Base {
public:
    int pub;
protected:
    int prot;
private:
    int priv;
};

// Public inheritance: maintains access levels
class PublicDerived : public Base {
    // pub is public
    // prot is protected
    // priv is not accessible
};

// Protected inheritance: public becomes protected
class ProtectedDerived : protected Base {
    // pub is protected
    // prot is protected
    // priv is not accessible
};

// Private inheritance: everything becomes private
class PrivateDerived : private Base {
    // pub is private
    // prot is private
    // priv is not accessible
};
```

## Polymorphism and Virtual Functions

### Virtual Functions

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <vector>

class Shape {
public:
    Shape(const std::string& name) : name_(name) {}
    virtual ~Shape() = default;  // Virtual destructor!
    
    // Virtual function - can be overridden
    virtual double area() const {
        return 0.0;
    }
    
    // Pure virtual function - MUST be overridden
    virtual void draw() const = 0;
    
    std::string name() const { return name_; }

protected:
    std::string name_;
};

class Circle : public Shape {
public:
    Circle(double radius)
        : Shape("Circle"), radius_(radius) {}
    
    double area() const override {
        return 3.14159 * radius_ * radius_;
    }
    
    void draw() const override {
        std::cout << "Drawing circle with radius " << radius_ << "\n";
    }

private:
    double radius_;
};

class Rectangle : public Shape {
public:
    Rectangle(double width, double height)
        : Shape("Rectangle"), width_(width), height_(height) {}
    
    double area() const override {
        return width_ * height_;
    }
    
    void draw() const override {
        std::cout << "Drawing rectangle " << width_ << "x" << height_ << "\n";
    }

private:
    double width_, height_;
};

int main() {
    // Polymorphism in action
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5.0));
    shapes.push_back(std::make_unique<Rectangle>(4.0, 6.0));
    shapes.push_back(std::make_unique<Circle>(3.0));
    
    for (const auto& shape : shapes) {
        std::cout << shape->name() << " - Area: " << shape->area() << "\n";
        shape->draw();
    }
    
    return 0;
}
```

### Override and Final (C++11)

```cpp
#include <iostream>

class Base {
public:
    virtual void foo() { std::cout << "Base::foo\n"; }
    virtual void bar() { std::cout << "Base::bar\n"; }
    void baz() { std::cout << "Base::baz\n"; }  // Non-virtual
};

class Derived : public Base {
public:
    void foo() override {  // Explicitly override
        std::cout << "Derived::foo\n";
    }
    
    void bar() final {     // Cannot be overridden further
        std::cout << "Derived::bar\n";
    }
    
    // void baz() override { }  // ERROR: baz is not virtual
};

class Final final : public Derived {  // Cannot be inherited from
public:
    void foo() override {
        std::cout << "Final::foo\n";
    }
    
    // void bar() override { }  // ERROR: bar is final
};

// class MoreDerived : public Final { };  // ERROR: Final is final
```

## Operator Overloading

```cpp
#include <iostream>
#include <compare>  // C++20

class Vector2D {
public:
    double x, y;
    
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}
    
    // Addition
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(x + other.x, y + other.y);
    }
    
    // Subtraction
    Vector2D operator-(const Vector2D& other) const {
        return Vector2D(x - other.x, y - other.y);
    }
    
    // Scalar multiplication
    Vector2D operator*(double scalar) const {
        return Vector2D(x * scalar, y * scalar);
    }
    
    // Compound assignment
    Vector2D& operator+=(const Vector2D& other) {
        x += other.x;
        y += other.y;
        return *this;
    }
    
    // Equality
    bool operator==(const Vector2D& other) const {
        return x == other.x && y == other.y;
    }
    
    // C++20 spaceship operator
    auto operator<=>(const Vector2D& other) const = default;
    
    // Negation
    Vector2D operator-() const {
        return Vector2D(-x, -y);
    }
    
    // Index operator
    double& operator[](int i) {
        return (i == 0) ? x : y;
    }
    
    // Function call operator
    double operator()() const {
        return std::sqrt(x*x + y*y);  // Magnitude
    }
    
    // Stream output (friend function)
    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }
};

// Non-member operator (for scalar * vector)
Vector2D operator*(double scalar, const Vector2D& v) {
    return v * scalar;
}

int main() {
    Vector2D v1(3, 4);
    Vector2D v2(1, 2);
    
    std::cout << "v1: " << v1 << "\n";
    std::cout << "v2: " << v2 << "\n";
    std::cout << "v1 + v2: " << (v1 + v2) << "\n";
    std::cout << "v1 * 2: " << (v1 * 2) << "\n";
    std::cout << "2 * v1: " << (2 * v1) << "\n";
    std::cout << "|v1|: " << v1() << "\n";
    std::cout << "v1 == v2: " << (v1 == v2) << "\n";
    
    return 0;
}
```

## C++23 Deducing This

```cpp
#include <iostream>

class Widget {
    int value_ = 42;

public:
    // C++23: Deducing this
    // Replaces const and non-const overloads
    template<typename Self>
    auto&& getValue(this Self&& self) {
        return std::forward<Self>(self).value_;
    }
    
    // Traditional way required two overloads:
    // int& getValue() { return value_; }
    // const int& getValue() const { return value_; }
    
    // Recursive lambdas made easier
    void countDown(this auto&& self, int n) {
        if (n <= 0) {
            std::cout << "Done!\n";
            return;
        }
        std::cout << n << " ";
        self.countDown(n - 1);
    }
};

int main() {
    Widget w;
    std::cout << "Value: " << w.getValue() << "\n";
    
    w.countDown(5);
    
    return 0;
}
```

## Exercises

### Exercise 1: Shape Hierarchy
Create a shape hierarchy with Circle, Rectangle, and Triangle classes. Each should calculate area and perimeter.

### Exercise 2: Employee System
Design an employee management system with base Employee class and derived Manager, Engineer, and Intern classes.

### Exercise 3: Custom String Class
Implement a String class with all Rule of Five members plus operator overloading for concatenation.

### Exercise 4: Expression Calculator
Create a class hierarchy for mathematical expressions that can be evaluated and printed.

## Next Steps

In the next module, we'll cover Modern C++ Features:
- C++11/14/17/20 essentials
- C++23 new features
- Move semantics deep dive
- Lambda expressions advanced

---

**Practice Tip:** Understanding OOP is crucial for C++. Build projects that use inheritance and polymorphism!
