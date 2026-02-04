# Example Project 1: CLI Calculator

A fully-featured command-line calculator with expression parsing, history, and variables.

## Features

- Mathematical expression evaluation
- Support for +, -, *, /, %, ^ operators
- Parentheses for grouping
- Variable storage
- History with recall
- Built-in functions (sin, cos, sqrt, etc.)
- Clean error handling

## Project Structure

```
01-cli-calculator/
├── README.md
├── CMakeLists.txt
├── include/
│   ├── calculator.hpp
│   ├── tokenizer.hpp
│   ├── parser.hpp
│   └── evaluator.hpp
├── src/
│   ├── main.cpp
│   ├── calculator.cpp
│   ├── tokenizer.cpp
│   ├── parser.cpp
│   └── evaluator.cpp
└── tests/
    └── test_calculator.cpp
```

## Building

```bash
mkdir build && cd build
cmake ..
cmake --build .
./calculator
```

## Code

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)
project(Calculator VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(calculator
    src/main.cpp
    src/calculator.cpp
    src/tokenizer.cpp
    src/parser.cpp
    src/evaluator.cpp
)

target_include_directories(calculator PRIVATE include)
target_compile_options(calculator PRIVATE -Wall -Wextra -Wpedantic)
```

### include/tokenizer.hpp

```cpp
#pragma once
#include <string>
#include <vector>
#include <variant>

enum class TokenType {
    Number,
    Identifier,
    Plus,
    Minus,
    Star,
    Slash,
    Percent,
    Caret,
    LeftParen,
    RightParen,
    Equals,
    Comma,
    EndOfInput,
    Error
};

struct Token {
    TokenType type;
    std::variant<double, std::string> value;
    size_t position;
    
    Token(TokenType t, size_t pos) : type(t), value(0.0), position(pos) {}
    Token(TokenType t, double v, size_t pos) : type(t), value(v), position(pos) {}
    Token(TokenType t, std::string v, size_t pos) : type(t), value(std::move(v)), position(pos) {}
};

class Tokenizer {
public:
    explicit Tokenizer(const std::string& input);
    std::vector<Token> tokenize();

private:
    std::string input_;
    size_t pos_ = 0;
    
    char peek() const;
    char advance();
    void skip_whitespace();
    Token number();
    Token identifier();
};
```

### include/parser.hpp

```cpp
#pragma once
#include "tokenizer.hpp"
#include <memory>

// AST Nodes
struct ASTNode {
    virtual ~ASTNode() = default;
    virtual double evaluate(class Evaluator& eval) const = 0;
};

using ASTNodePtr = std::unique_ptr<ASTNode>;

struct NumberNode : ASTNode {
    double value;
    explicit NumberNode(double v) : value(v) {}
    double evaluate(Evaluator& eval) const override;
};

struct VariableNode : ASTNode {
    std::string name;
    explicit VariableNode(std::string n) : name(std::move(n)) {}
    double evaluate(Evaluator& eval) const override;
};

struct BinaryOpNode : ASTNode {
    TokenType op;
    ASTNodePtr left;
    ASTNodePtr right;
    
    BinaryOpNode(TokenType o, ASTNodePtr l, ASTNodePtr r)
        : op(o), left(std::move(l)), right(std::move(r)) {}
    double evaluate(Evaluator& eval) const override;
};

struct UnaryOpNode : ASTNode {
    TokenType op;
    ASTNodePtr operand;
    
    UnaryOpNode(TokenType o, ASTNodePtr op) : op(o), operand(std::move(op)) {}
    double evaluate(Evaluator& eval) const override;
};

struct FunctionCallNode : ASTNode {
    std::string name;
    std::vector<ASTNodePtr> args;
    
    FunctionCallNode(std::string n, std::vector<ASTNodePtr> a)
        : name(std::move(n)), args(std::move(a)) {}
    double evaluate(Evaluator& eval) const override;
};

struct AssignmentNode : ASTNode {
    std::string name;
    ASTNodePtr value;
    
    AssignmentNode(std::string n, ASTNodePtr v)
        : name(std::move(n)), value(std::move(v)) {}
    double evaluate(Evaluator& eval) const override;
};

// Parser
class Parser {
public:
    explicit Parser(std::vector<Token> tokens);
    ASTNodePtr parse();

private:
    std::vector<Token> tokens_;
    size_t pos_ = 0;
    
    Token& current();
    Token& peek(int offset = 0);
    bool match(TokenType type);
    Token consume(TokenType type, const std::string& error);
    
    ASTNodePtr expression();
    ASTNodePtr assignment();
    ASTNodePtr additive();
    ASTNodePtr multiplicative();
    ASTNodePtr power();
    ASTNodePtr unary();
    ASTNodePtr primary();
    ASTNodePtr function_call(const std::string& name);
};
```

### include/evaluator.hpp

```cpp
#pragma once
#include <string>
#include <unordered_map>
#include <functional>
#include <cmath>

class Evaluator {
public:
    Evaluator();
    
    double get_variable(const std::string& name) const;
    void set_variable(const std::string& name, double value);
    bool has_variable(const std::string& name) const;
    
    double call_function(const std::string& name, const std::vector<double>& args) const;
    bool has_function(const std::string& name) const;
    
    const std::unordered_map<std::string, double>& variables() const { return variables_; }

private:
    std::unordered_map<std::string, double> variables_;
    std::unordered_map<std::string, std::function<double(const std::vector<double>&)>> functions_;
    
    void register_builtins();
};
```

### include/calculator.hpp

```cpp
#pragma once
#include "evaluator.hpp"
#include <string>
#include <vector>
#include <optional>

struct CalculatorResult {
    double value;
    std::string expression;
};

class Calculator {
public:
    Calculator();
    
    std::optional<double> evaluate(const std::string& expression);
    const std::vector<CalculatorResult>& history() const { return history_; }
    void clear_history() { history_.clear(); }
    
    Evaluator& evaluator() { return evaluator_; }
    const Evaluator& evaluator() const { return evaluator_; }
    
    std::string last_error() const { return last_error_; }

private:
    Evaluator evaluator_;
    std::vector<CalculatorResult> history_;
    std::string last_error_;
};
```

### src/main.cpp

```cpp
#include "calculator.hpp"
#include <iostream>
#include <string>
#include <iomanip>

void print_help() {
    std::cout << R"(
Calculator Commands:
  help     - Show this help message
  history  - Show calculation history
  vars     - Show defined variables
  clear    - Clear history
  quit     - Exit the calculator

Operators: +, -, *, /, %, ^ (power)
Functions: sin, cos, tan, sqrt, abs, log, log10, exp, floor, ceil
Constants: pi, e
Variables: x = 5 (then use x in expressions)

Examples:
  2 + 3 * 4
  (2 + 3) * 4
  sin(pi / 2)
  x = 10
  x * 2 + 5
)";
}

int main() {
    Calculator calc;
    std::string line;
    
    std::cout << "C++ Calculator v1.0 (type 'help' for commands)\n\n";
    
    while (true) {
        std::cout << "> ";
        if (!std::getline(std::cin, line)) {
            break;
        }
        
        // Trim whitespace
        size_t start = line.find_first_not_of(" \t");
        if (start == std::string::npos) continue;
        size_t end = line.find_last_not_of(" \t");
        line = line.substr(start, end - start + 1);
        
        if (line.empty()) continue;
        
        if (line == "quit" || line == "exit") {
            break;
        }
        
        if (line == "help") {
            print_help();
            continue;
        }
        
        if (line == "history") {
            if (calc.history().empty()) {
                std::cout << "  (no history)\n";
            } else {
                for (size_t i = 0; i < calc.history().size(); ++i) {
                    std::cout << "  [" << i + 1 << "] " 
                              << calc.history()[i].expression << " = "
                              << calc.history()[i].value << "\n";
                }
            }
            continue;
        }
        
        if (line == "vars") {
            const auto& vars = calc.evaluator().variables();
            if (vars.empty()) {
                std::cout << "  (no variables)\n";
            } else {
                for (const auto& [name, value] : vars) {
                    std::cout << "  " << name << " = " << value << "\n";
                }
            }
            continue;
        }
        
        if (line == "clear") {
            calc.clear_history();
            std::cout << "  History cleared\n";
            continue;
        }
        
        auto result = calc.evaluate(line);
        if (result) {
            std::cout << "= " << std::setprecision(10) << *result << "\n";
        } else {
            std::cout << "Error: " << calc.last_error() << "\n";
        }
    }
    
    std::cout << "Goodbye!\n";
    return 0;
}
```

### src/tokenizer.cpp

```cpp
#include "tokenizer.hpp"
#include <cctype>

Tokenizer::Tokenizer(const std::string& input) : input_(input) {}

char Tokenizer::peek() const {
    if (pos_ >= input_.length()) return '\0';
    return input_[pos_];
}

char Tokenizer::advance() {
    return input_[pos_++];
}

void Tokenizer::skip_whitespace() {
    while (std::isspace(peek())) {
        advance();
    }
}

Token Tokenizer::number() {
    size_t start = pos_;
    std::string num;
    
    while (std::isdigit(peek())) {
        num += advance();
    }
    
    if (peek() == '.') {
        num += advance();
        while (std::isdigit(peek())) {
            num += advance();
        }
    }
    
    // Scientific notation
    if (peek() == 'e' || peek() == 'E') {
        num += advance();
        if (peek() == '+' || peek() == '-') {
            num += advance();
        }
        while (std::isdigit(peek())) {
            num += advance();
        }
    }
    
    return Token(TokenType::Number, std::stod(num), start);
}

Token Tokenizer::identifier() {
    size_t start = pos_;
    std::string id;
    
    while (std::isalnum(peek()) || peek() == '_') {
        id += advance();
    }
    
    return Token(TokenType::Identifier, id, start);
}

std::vector<Token> Tokenizer::tokenize() {
    std::vector<Token> tokens;
    
    while (pos_ < input_.length()) {
        skip_whitespace();
        if (pos_ >= input_.length()) break;
        
        char c = peek();
        size_t start = pos_;
        
        if (std::isdigit(c) || (c == '.' && pos_ + 1 < input_.length() && std::isdigit(input_[pos_ + 1]))) {
            tokens.push_back(number());
        } else if (std::isalpha(c) || c == '_') {
            tokens.push_back(identifier());
        } else {
            advance();
            switch (c) {
                case '+': tokens.emplace_back(TokenType::Plus, start); break;
                case '-': tokens.emplace_back(TokenType::Minus, start); break;
                case '*': tokens.emplace_back(TokenType::Star, start); break;
                case '/': tokens.emplace_back(TokenType::Slash, start); break;
                case '%': tokens.emplace_back(TokenType::Percent, start); break;
                case '^': tokens.emplace_back(TokenType::Caret, start); break;
                case '(': tokens.emplace_back(TokenType::LeftParen, start); break;
                case ')': tokens.emplace_back(TokenType::RightParen, start); break;
                case '=': tokens.emplace_back(TokenType::Equals, start); break;
                case ',': tokens.emplace_back(TokenType::Comma, start); break;
                default:
                    tokens.emplace_back(TokenType::Error, std::string(1, c), start);
            }
        }
    }
    
    tokens.emplace_back(TokenType::EndOfInput, pos_);
    return tokens;
}
```

### src/evaluator.cpp

```cpp
#include "evaluator.hpp"
#include <stdexcept>
#include <numbers>

Evaluator::Evaluator() {
    register_builtins();
}

void Evaluator::register_builtins() {
    // Constants
    variables_["pi"] = std::numbers::pi;
    variables_["e"] = std::numbers::e;
    
    // Functions
    functions_["sin"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("sin requires 1 argument");
        return std::sin(args[0]);
    };
    
    functions_["cos"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("cos requires 1 argument");
        return std::cos(args[0]);
    };
    
    functions_["tan"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("tan requires 1 argument");
        return std::tan(args[0]);
    };
    
    functions_["sqrt"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("sqrt requires 1 argument");
        if (args[0] < 0) throw std::runtime_error("sqrt of negative number");
        return std::sqrt(args[0]);
    };
    
    functions_["abs"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("abs requires 1 argument");
        return std::abs(args[0]);
    };
    
    functions_["log"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("log requires 1 argument");
        if (args[0] <= 0) throw std::runtime_error("log of non-positive number");
        return std::log(args[0]);
    };
    
    functions_["log10"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("log10 requires 1 argument");
        if (args[0] <= 0) throw std::runtime_error("log10 of non-positive number");
        return std::log10(args[0]);
    };
    
    functions_["exp"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("exp requires 1 argument");
        return std::exp(args[0]);
    };
    
    functions_["floor"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("floor requires 1 argument");
        return std::floor(args[0]);
    };
    
    functions_["ceil"] = [](const std::vector<double>& args) {
        if (args.size() != 1) throw std::runtime_error("ceil requires 1 argument");
        return std::ceil(args[0]);
    };
    
    functions_["pow"] = [](const std::vector<double>& args) {
        if (args.size() != 2) throw std::runtime_error("pow requires 2 arguments");
        return std::pow(args[0], args[1]);
    };
    
    functions_["min"] = [](const std::vector<double>& args) {
        if (args.size() != 2) throw std::runtime_error("min requires 2 arguments");
        return std::min(args[0], args[1]);
    };
    
    functions_["max"] = [](const std::vector<double>& args) {
        if (args.size() != 2) throw std::runtime_error("max requires 2 arguments");
        return std::max(args[0], args[1]);
    };
}

double Evaluator::get_variable(const std::string& name) const {
    auto it = variables_.find(name);
    if (it == variables_.end()) {
        throw std::runtime_error("Undefined variable: " + name);
    }
    return it->second;
}

void Evaluator::set_variable(const std::string& name, double value) {
    variables_[name] = value;
}

bool Evaluator::has_variable(const std::string& name) const {
    return variables_.find(name) != variables_.end();
}

double Evaluator::call_function(const std::string& name, const std::vector<double>& args) const {
    auto it = functions_.find(name);
    if (it == functions_.end()) {
        throw std::runtime_error("Undefined function: " + name);
    }
    return it->second(args);
}

bool Evaluator::has_function(const std::string& name) const {
    return functions_.find(name) != functions_.end();
}
```

## Learning Objectives

1. **Lexical Analysis** - Breaking input into tokens
2. **Parsing** - Building an Abstract Syntax Tree
3. **Expression Evaluation** - Tree traversal and computation
4. **Error Handling** - Graceful error recovery
5. **REPL Design** - Read-Eval-Print Loop pattern

## Extensions

- Add more functions (asin, acos, atan, etc.)
- Implement complex number support
- Add plotting capability
- Create a GUI version
- Support for matrices

## Usage Examples

```
> 2 + 3 * 4
= 14

> (2 + 3) * 4
= 20

> x = 10
= 10

> x * 2 + 5
= 25

> sin(pi / 2)
= 1

> sqrt(2) ^ 2
= 2

> history
  [1] 2 + 3 * 4 = 14
  [2] (2 + 3) * 4 = 20
  [3] x = 10 = 10
  [4] x * 2 + 5 = 25
```
