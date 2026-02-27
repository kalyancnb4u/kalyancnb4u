---
title: "Complete C++ Mastery Part 9: Design Patterns & Software Architecture"
date: 2024-02-21 02:00:00 +0000
categories: [C++, Programming, Design-Patterns]
tags: [cpp, design-patterns, architecture, solid, patterns, best-practices]
---

# Complete C++ Mastery Part 9: Design Patterns & Software Architecture

## Introduction

Welcome to Part 9 of the Complete C++ Mastery series. Design patterns are proven solutions to recurring software design problems. Combined with solid architectural principles, they enable developers to build maintainable, scalable, and robust systems. This part explores SOLID principles, classic Gang of Four patterns adapted for modern C++, C++-specific idioms, and architectural patterns.

Understanding design patterns isn't just about memorizing solutions—it's about recognizing when and why to apply them, and how modern C++ features (smart pointers, lambdas, move semantics) enhance traditional patterns.

**Prerequisites:** This part assumes knowledge from Parts 1-8, especially OOP, templates, smart pointers, and move semantics.

---

## 9.1 SOLID Principles

### Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change.

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <vector>

// ❌ BAD: Multiple responsibilities
class BadEmployee {
    std::string name;
    double salary;
    
public:
    BadEmployee(const std::string& n, double s) : name(n), salary(s) {}
    
    // Responsibility 1: Employee data management
    void set_salary(double s) { salary = s; }
    double get_salary() const { return salary; }
    
    // Responsibility 2: Persistence (violates SRP)
    void save_to_database() {
        std::cout << "Saving to database...\n";
    }
    
    // Responsibility 3: Reporting (violates SRP)
    void generate_payslip() {
        std::cout << "Generating payslip...\n";
    }
};

// ✅ GOOD: Single responsibility per class
class Employee {
    std::string name;
    double salary;
    
public:
    Employee(const std::string& n, double s) : name(n), salary(s) {}
    
    void set_salary(double s) { salary = s; }
    double get_salary() const { return salary; }
    const std::string& get_name() const { return name; }
};

class EmployeeRepository {
public:
    void save(const Employee& emp) {
        std::cout << "Saving " << emp.get_name() << " to database\n";
    }
    
    Employee load(const std::string& name) {
        return Employee(name, 50000);
    }
};

class PayslipGenerator {
public:
    void generate(const Employee& emp) {
        std::cout << "Payslip for " << emp.get_name() 
                  << ": $" << emp.get_salary() << "\n";
    }
};
```

### Open/Closed Principle (OCP)

**Definition:** Software entities should be open for extension but closed for modification.

```cpp
#include <iostream>
#include <vector>
#include <memory>

// ❌ BAD: Must modify class to add new shapes
class BadAreaCalculator {
public:
    double calculate_area(const std::vector<std::string>& shapes) {
        double total = 0;
        for (const auto& shape : shapes) {
            if (shape == "circle") {
                total += 3.14 * 5 * 5;  // Hardcoded radius
            } else if (shape == "rectangle") {
                total += 10 * 20;  // Hardcoded dimensions
            }
            // Adding new shape requires modifying this code!
        }
        return total;
    }
};

// ✅ GOOD: Open for extension, closed for modification
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override { return 3.14159 * radius * radius; }
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double area() const override { return width * height; }
};

class Triangle : public Shape {
    double base, height;
public:
    Triangle(double b, double h) : base(b), height(h) {}
    double area() const override { return 0.5 * base * height; }
};

class AreaCalculator {
public:
    double calculate_total_area(const std::vector<std::unique_ptr<Shape>>& shapes) {
        double total = 0;
        for (const auto& shape : shapes) {
            total += shape->area();
        }
        return total;
    }
};

int main() {
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5));
    shapes.push_back(std::make_unique<Rectangle>(10, 20));
    shapes.push_back(std::make_unique<Triangle>(10, 15));
    
    AreaCalculator calc;
    std::cout << "Total area: " << calc.calculate_total_area(shapes) << "\n";
    
    return 0;
}
```

### Liskov Substitution Principle (LSP)

**Definition:** Derived classes must be substitutable for their base classes.

```cpp
#include <iostream>

// ❌ BAD: Square violates LSP (changes rectangle behavior)
class BadRectangle {
protected:
    int width, height;
public:
    virtual void set_width(int w) { width = w; }
    virtual void set_height(int h) { height = h; }
    int area() const { return width * height; }
};

class BadSquare : public BadRectangle {
public:
    void set_width(int w) override {
        width = height = w;  // Changes height too!
    }
    
    void set_height(int h) override {
        width = height = h;  // Changes width too!
    }
};

void process_rectangle(BadRectangle& rect) {
    rect.set_width(5);
    rect.set_height(10);
    // Expected area: 50
    // With BadSquare: area is 100! (violates LSP)
    std::cout << "Area: " << rect.area() << "\n";
}

// ✅ GOOD: Proper hierarchy respecting LSP
class Shape {
public:
    virtual ~Shape() = default;
    virtual int area() const = 0;
};

class Rectangle : public Shape {
    int width, height;
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    int area() const override { return width * height; }
};

class Square : public Shape {
    int side;
public:
    Square(int s) : side(s) {}
    int area() const override { return side * side; }
};

int main() {
    BadSquare sq;
    process_rectangle(sq);  // Unexpected behavior
    
    return 0;
}
```

### Interface Segregation Principle (ISP)

**Definition:** No client should be forced to depend on methods it doesn't use.

```cpp
#include <iostream>

// ❌ BAD: Fat interface forces implementations to provide unused methods
class BadWorker {
public:
    virtual ~BadWorker() = default;
    virtual void work() = 0;
    virtual void eat() = 0;
    virtual void sleep() = 0;
};

class Human : public BadWorker {
public:
    void work() override { std::cout << "Human working\n"; }
    void eat() override { std::cout << "Human eating\n"; }
    void sleep() override { std::cout << "Human sleeping\n"; }
};

class Robot : public BadWorker {
public:
    void work() override { std::cout << "Robot working\n"; }
    void eat() override { /* Robot doesn't eat! */ }
    void sleep() override { /* Robot doesn't sleep! */ }
};

// ✅ GOOD: Segregated interfaces
class Workable {
public:
    virtual ~Workable() = default;
    virtual void work() = 0;
};

class Eatable {
public:
    virtual ~Eatable() = default;
    virtual void eat() = 0;
};

class Sleepable {
public:
    virtual ~Sleepable() = default;
    virtual void sleep() = 0;
};

class GoodHuman : public Workable, public Eatable, public Sleepable {
public:
    void work() override { std::cout << "Human working\n"; }
    void eat() override { std::cout << "Human eating\n"; }
    void sleep() override { std::cout << "Human sleeping\n"; }
};

class GoodRobot : public Workable {
public:
    void work() override { std::cout << "Robot working\n"; }
    // No need to implement eat() or sleep()
};
```

### Dependency Inversion Principle (DIP)

**Definition:** Depend on abstractions, not concretions.

```cpp
#include <iostream>
#include <memory>

// ❌ BAD: High-level depends on low-level concrete class
class MySQLDatabase {
public:
    void connect() { std::cout << "Connecting to MySQL\n"; }
    void execute(const std::string& query) {
        std::cout << "Executing: " << query << "\n";
    }
};

class BadUserService {
    MySQLDatabase db;  // Tightly coupled!
public:
    void save_user(const std::string& name) {
        db.connect();
        db.execute("INSERT INTO users VALUES ('" + name + "')");
    }
};

// ✅ GOOD: Depend on abstraction
class IDatabase {
public:
    virtual ~IDatabase() = default;
    virtual void connect() = 0;
    virtual void execute(const std::string& query) = 0;
};

class MySQLDatabase2 : public IDatabase {
public:
    void connect() override { std::cout << "Connecting to MySQL\n"; }
    void execute(const std::string& query) override {
        std::cout << "MySQL: " << query << "\n";
    }
};

class PostgreSQLDatabase : public IDatabase {
public:
    void connect() override { std::cout << "Connecting to PostgreSQL\n"; }
    void execute(const std::string& query) override {
        std::cout << "PostgreSQL: " << query << "\n";
    }
};

class GoodUserService {
    std::unique_ptr<IDatabase> db;
public:
    GoodUserService(std::unique_ptr<IDatabase> database)
        : db(std::move(database)) {}
    
    void save_user(const std::string& name) {
        db->connect();
        db->execute("INSERT INTO users VALUES ('" + name + "')");
    }
};

int main() {
    // Can easily swap database implementations
    auto service1 = std::make_unique<GoodUserService>(
        std::make_unique<MySQLDatabase2>()
    );
    service1->save_user("Alice");
    
    auto service2 = std::make_unique<GoodUserService>(
        std::make_unique<PostgreSQLDatabase>()
    );
    service2->save_user("Bob");
    
    return 0;
}
```

---

## 9.2 Creational Patterns

### Singleton

```cpp
#include <iostream>
#include <memory>
#include <mutex>

// ❌ Old-style Singleton (pre-C++11)
class OldSingleton {
    static OldSingleton* instance;
    static std::mutex mutex_;
    
    OldSingleton() = default;
    
public:
    static OldSingleton* get_instance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mutex_);
            if (instance == nullptr) {
                instance = new OldSingleton();
            }
        }
        return instance;
    }
    
    OldSingleton(const OldSingleton&) = delete;
    OldSingleton& operator=(const OldSingleton&) = delete;
};

OldSingleton* OldSingleton::instance = nullptr;
std::mutex OldSingleton::mutex_;

// ✅ Modern Singleton (C++11+): Thread-safe, no memory leak
class Singleton {
    Singleton() {
        std::cout << "Singleton created\n";
    }
    
public:
    static Singleton& get_instance() {
        static Singleton instance;  // Thread-safe in C++11
        return instance;
    }
    
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    void do_something() {
        std::cout << "Singleton doing work\n";
    }
};

int main() {
    Singleton& s1 = Singleton::get_instance();
    Singleton& s2 = Singleton::get_instance();
    
    std::cout << "Same instance: " << (&s1 == &s2) << "\n";
    s1.do_something();
    
    return 0;
}
```

### Factory Method

```cpp
#include <iostream>
#include <memory>
#include <string>

// Product hierarchy
class Document {
public:
    virtual ~Document() = default;
    virtual void open() = 0;
    virtual void save() = 0;
};

class PDFDocument : public Document {
public:
    void open() override { std::cout << "Opening PDF document\n"; }
    void save() override { std::cout << "Saving PDF document\n"; }
};

class WordDocument : public Document {
public:
    void open() override { std::cout << "Opening Word document\n"; }
    void save() override { std::cout << "Saving Word document\n"; }
};

// Creator hierarchy
class Application {
public:
    virtual ~Application() = default;
    
    // Factory method
    virtual std::unique_ptr<Document> create_document() = 0;
    
    void new_document() {
        auto doc = create_document();
        doc->open();
    }
};

class PDFApplication : public Application {
public:
    std::unique_ptr<Document> create_document() override {
        return std::make_unique<PDFDocument>();
    }
};

class WordApplication : public Application {
public:
    std::unique_ptr<Document> create_document() override {
        return std::make_unique<WordDocument>();
    }
};

int main() {
    std::unique_ptr<Application> app = std::make_unique<PDFApplication>();
    app->new_document();
    
    app = std::make_unique<WordApplication>();
    app->new_document();
    
    return 0;
}
```

### Abstract Factory

```cpp
#include <iostream>
#include <memory>

// Abstract products
class Button {
public:
    virtual ~Button() = default;
    virtual void paint() = 0;
};

class Checkbox {
public:
    virtual ~Checkbox() = default;
    virtual void paint() = 0;
};

// Concrete products - Windows
class WindowsButton : public Button {
public:
    void paint() override { std::cout << "Rendering Windows button\n"; }
};

class WindowsCheckbox : public Checkbox {
public:
    void paint() override { std::cout << "Rendering Windows checkbox\n"; }
};

// Concrete products - Mac
class MacButton : public Button {
public:
    void paint() override { std::cout << "Rendering Mac button\n"; }
};

class MacCheckbox : public Checkbox {
public:
    void paint() override { std::cout << "Rendering Mac checkbox\n"; }
};

// Abstract factory
class GUIFactory {
public:
    virtual ~GUIFactory() = default;
    virtual std::unique_ptr<Button> create_button() = 0;
    virtual std::unique_ptr<Checkbox> create_checkbox() = 0;
};

// Concrete factories
class WindowsFactory : public GUIFactory {
public:
    std::unique_ptr<Button> create_button() override {
        return std::make_unique<WindowsButton>();
    }
    
    std::unique_ptr<Checkbox> create_checkbox() override {
        return std::make_unique<WindowsCheckbox>();
    }
};

class MacFactory : public GUIFactory {
public:
    std::unique_ptr<Button> create_button() override {
        return std::make_unique<MacButton>();
    }
    
    std::unique_ptr<Checkbox> create_checkbox() override {
        return std::make_unique<MacCheckbox>();
    }
};

// Client code
class Application {
    std::unique_ptr<GUIFactory> factory;
    
public:
    Application(std::unique_ptr<GUIFactory> f) : factory(std::move(f)) {}
    
    void create_ui() {
        auto button = factory->create_button();
        auto checkbox = factory->create_checkbox();
        
        button->paint();
        checkbox->paint();
    }
};

int main() {
    std::cout << "=== Windows UI ===\n";
    Application win_app(std::make_unique<WindowsFactory>());
    win_app.create_ui();
    
    std::cout << "\n=== Mac UI ===\n";
    Application mac_app(std::make_unique<MacFactory>());
    mac_app.create_ui();
    
    return 0;
}
```

### Builder Pattern

```cpp
#include <iostream>
#include <string>
#include <memory>

class Pizza {
    std::string dough;
    std::string sauce;
    std::string topping;
    
public:
    void set_dough(const std::string& d) { dough = d; }
    void set_sauce(const std::string& s) { sauce = s; }
    void set_topping(const std::string& t) { topping = t; }
    
    void show() const {
        std::cout << "Pizza: " << dough << " dough, "
                  << sauce << " sauce, " << topping << " topping\n";
    }
};

// Builder interface
class PizzaBuilder {
protected:
    std::unique_ptr<Pizza> pizza;
    
public:
    PizzaBuilder() : pizza(std::make_unique<Pizza>()) {}
    virtual ~PizzaBuilder() = default;
    
    virtual void build_dough() = 0;
    virtual void build_sauce() = 0;
    virtual void build_topping() = 0;
    
    std::unique_ptr<Pizza> get_pizza() {
        return std::move(pizza);
    }
};

// Concrete builders
class MargheritaBuilder : public PizzaBuilder {
public:
    void build_dough() override { pizza->set_dough("thin"); }
    void build_sauce() override { pizza->set_sauce("tomato"); }
    void build_topping() override { pizza->set_topping("mozzarella"); }
};

class PepperoniBuilder : public PizzaBuilder {
public:
    void build_dough() override { pizza->set_dough("thick"); }
    void build_sauce() override { pizza->set_sauce("spicy tomato"); }
    void build_topping() override { pizza->set_topping("pepperoni"); }
};

// Director
class Cook {
public:
    std::unique_ptr<Pizza> make_pizza(PizzaBuilder& builder) {
        builder.build_dough();
        builder.build_sauce();
        builder.build_topping();
        return builder.get_pizza();
    }
};

// ✅ Modern C++: Fluent Builder with method chaining
class ModernPizzaBuilder {
    Pizza pizza;
    
public:
    ModernPizzaBuilder& with_dough(const std::string& d) {
        pizza.set_dough(d);
        return *this;
    }
    
    ModernPizzaBuilder& with_sauce(const std::string& s) {
        pizza.set_sauce(s);
        return *this;
    }
    
    ModernPizzaBuilder& with_topping(const std::string& t) {
        pizza.set_topping(t);
        return *this;
    }
    
    Pizza build() {
        return std::move(pizza);
    }
};

int main() {
    Cook cook;
    
    MargheritaBuilder margherita_builder;
    auto margherita = cook.make_pizza(margherita_builder);
    margherita->show();
    
    PepperoniBuilder pepperoni_builder;
    auto pepperoni = cook.make_pizza(pepperoni_builder);
    pepperoni->show();
    
    // Modern fluent interface
    auto custom = ModernPizzaBuilder()
        .with_dough("sourdough")
        .with_sauce("pesto")
        .with_topping("vegetables")
        .build();
    custom.show();
    
    return 0;
}
```

### Prototype Pattern

```cpp
#include <iostream>
#include <memory>
#include <unordered_map>

class Shape {
public:
    virtual ~Shape() = default;
    virtual std::unique_ptr<Shape> clone() const = 0;
    virtual void draw() const = 0;
};

class Circle : public Shape {
    int radius;
    
public:
    Circle(int r) : radius(r) {}
    
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(*this);
    }
    
    void draw() const override {
        std::cout << "Circle with radius " << radius << "\n";
    }
};

class Rectangle : public Shape {
    int width, height;
    
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Rectangle>(*this);
    }
    
    void draw() const override {
        std::cout << "Rectangle " << width << "x" << height << "\n";
    }
};

// Prototype registry
class ShapeFactory {
    std::unordered_map<std::string, std::unique_ptr<Shape>> prototypes;
    
public:
    void register_prototype(const std::string& key, std::unique_ptr<Shape> prototype) {
        prototypes[key] = std::move(prototype);
    }
    
    std::unique_ptr<Shape> create(const std::string& key) const {
        auto it = prototypes.find(key);
        if (it != prototypes.end()) {
            return it->second->clone();
        }
        return nullptr;
    }
};

int main() {
    ShapeFactory factory;
    
    // Register prototypes
    factory.register_prototype("small_circle", std::make_unique<Circle>(5));
    factory.register_prototype("large_circle", std::make_unique<Circle>(20));
    factory.register_prototype("square", std::make_unique<Rectangle>(10, 10));
    
    // Create clones
    auto shape1 = factory.create("small_circle");
    auto shape2 = factory.create("large_circle");
    auto shape3 = factory.create("square");
    
    shape1->draw();
    shape2->draw();
    shape3->draw();
    
    return 0;
}
```

---

## 9.3 Structural Patterns

### Adapter Pattern

```cpp
#include <iostream>
#include <memory>

// Target interface
class MediaPlayer {
public:
    virtual ~MediaPlayer() = default;
    virtual void play(const std::string& filename) = 0;
};

// Adaptee (incompatible interface)
class AdvancedMediaPlayer {
public:
    virtual ~AdvancedMediaPlayer() = default;
    virtual void play_vlc(const std::string& filename) = 0;
    virtual void play_mp4(const std::string& filename) = 0;
};

class VlcPlayer : public AdvancedMediaPlayer {
public:
    void play_vlc(const std::string& filename) override {
        std::cout << "Playing VLC file: " << filename << "\n";
    }
    
    void play_mp4(const std::string& filename) override {
        // Do nothing
    }
};

class Mp4Player : public AdvancedMediaPlayer {
public:
    void play_vlc(const std::string& filename) override {
        // Do nothing
    }
    
    void play_mp4(const std::string& filename) override {
        std::cout << "Playing MP4 file: " << filename << "\n";
    }
};

// Adapter
class MediaAdapter : public MediaPlayer {
    std::unique_ptr<AdvancedMediaPlayer> advanced_player;
    
public:
    MediaAdapter(const std::string& audio_type) {
        if (audio_type == "vlc") {
            advanced_player = std::make_unique<VlcPlayer>();
        } else if (audio_type == "mp4") {
            advanced_player = std::make_unique<Mp4Player>();
        }
    }
    
    void play(const std::string& filename) override {
        if (advanced_player) {
            std::string ext = filename.substr(filename.find_last_of(".") + 1);
            if (ext == "vlc") {
                advanced_player->play_vlc(filename);
            } else if (ext == "mp4") {
                advanced_player->play_mp4(filename);
            }
        }
    }
};

// Client
class AudioPlayer : public MediaPlayer {
    std::unique_ptr<MediaAdapter> adapter;
    
public:
    void play(const std::string& filename) override {
        std::string ext = filename.substr(filename.find_last_of(".") + 1);
        
        if (ext == "mp3") {
            std::cout << "Playing MP3 file: " << filename << "\n";
        } else if (ext == "vlc" || ext == "mp4") {
            adapter = std::make_unique<MediaAdapter>(ext);
            adapter->play(filename);
        } else {
            std::cout << "Invalid format: " << ext << "\n";
        }
    }
};

int main() {
    AudioPlayer player;
    
    player.play("song.mp3");
    player.play("movie.mp4");
    player.play("video.vlc");
    
    return 0;
}
```

### Decorator Pattern

```cpp
#include <iostream>
#include <memory>
#include <string>

// Component interface
class Coffee {
public:
    virtual ~Coffee() = default;
    virtual std::string get_description() const = 0;
    virtual double get_cost() const = 0;
};

// Concrete component
class SimpleCoffee : public Coffee {
public:
    std::string get_description() const override {
        return "Simple coffee";
    }
    
    double get_cost() const override {
        return 2.0;
    }
};

// Decorator base class
class CoffeeDecorator : public Coffee {
protected:
    std::unique_ptr<Coffee> coffee;
    
public:
    CoffeeDecorator(std::unique_ptr<Coffee> c) : coffee(std::move(c)) {}
};

// Concrete decorators
class MilkDecorator : public CoffeeDecorator {
public:
    MilkDecorator(std::unique_ptr<Coffee> c) : CoffeeDecorator(std::move(c)) {}
    
    std::string get_description() const override {
        return coffee->get_description() + " + milk";
    }
    
    double get_cost() const override {
        return coffee->get_cost() + 0.5;
    }
};

class SugarDecorator : public CoffeeDecorator {
public:
    SugarDecorator(std::unique_ptr<Coffee> c) : CoffeeDecorator(std::move(c)) {}
    
    std::string get_description() const override {
        return coffee->get_description() + " + sugar";
    }
    
    double get_cost() const override {
        return coffee->get_cost() + 0.2;
    }
};

class WhipDecorator : public CoffeeDecorator {
public:
    WhipDecorator(std::unique_ptr<Coffee> c) : CoffeeDecorator(std::move(c)) {}
    
    std::string get_description() const override {
        return coffee->get_description() + " + whipped cream";
    }
    
    double get_cost() const override {
        return coffee->get_cost() + 0.7;
    }
};

int main() {
    // Simple coffee
    std::unique_ptr<Coffee> coffee = std::make_unique<SimpleCoffee>();
    std::cout << coffee->get_description() << " = $" << coffee->get_cost() << "\n";
    
    // Coffee with milk
    coffee = std::make_unique<MilkDecorator>(std::move(coffee));
    std::cout << coffee->get_description() << " = $" << coffee->get_cost() << "\n";
    
    // Coffee with milk and sugar
    coffee = std::make_unique<SugarDecorator>(std::move(coffee));
    std::cout << coffee->get_description() << " = $" << coffee->get_cost() << "\n";
    
    // Coffee with milk, sugar, and whip
    coffee = std::make_unique<WhipDecorator>(std::move(coffee));
    std::cout << coffee->get_description() << " = $" << coffee->get_cost() << "\n";
    
    return 0;
}
```

### Facade Pattern

```cpp
#include <iostream>
#include <memory>

// Complex subsystem classes
class CPU {
public:
    void freeze() { std::cout << "CPU: Freezing\n"; }
    void jump(long position) { std::cout << "CPU: Jumping to " << position << "\n"; }
    void execute() { std::cout << "CPU: Executing\n"; }
};

class Memory {
public:
    void load(long position, const std::string& data) {
        std::cout << "Memory: Loading data at " << position << "\n";
    }
};

class HardDrive {
public:
    std::string read(long lba, int size) {
        std::cout << "HardDrive: Reading " << size << " bytes from " << lba << "\n";
        return "boot_data";
    }
};

// Facade
class ComputerFacade {
    CPU cpu;
    Memory memory;
    HardDrive hard_drive;
    
public:
    void start() {
        std::cout << "=== Starting Computer ===\n";
        cpu.freeze();
        memory.load(0, hard_drive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
        std::cout << "=== Computer Started ===\n";
    }
};

int main() {
    ComputerFacade computer;
    computer.start();  // Simple interface hides complex subsystem
    
    return 0;
}
```

[Due to length constraints, this is approximately 60% of Part 9. Would you like me to continue with sections 9.4 (Behavioral Patterns) and 9.5 (C++-Specific Idioms)?]

---

## 9.4 Behavioral Patterns

### Observer Pattern

```cpp
#include <iostream>
#include <vector>
#include <memory>
#include <algorithm>

// Observer interface
class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(float temperature) = 0;
};

// Subject
class WeatherStation {
    std::vector<std::weak_ptr<Observer>> observers;
    float temperature = 0;
    
public:
    void attach(std::shared_ptr<Observer> observer) {
        observers.push_back(observer);
    }
    
    void set_temperature(float temp) {
        temperature = temp;
        notify();
    }
    
    void notify() {
        // Remove expired observers
        observers.erase(
            std::remove_if(observers.begin(), observers.end(),
                [](const std::weak_ptr<Observer>& wp) { return wp.expired(); }),
            observers.end()
        );
        
        // Notify all observers
        for (auto& weak_obs : observers) {
            if (auto obs = weak_obs.lock()) {
                obs->update(temperature);
            }
        }
    }
};

// Concrete observers
class PhoneDisplay : public Observer {
    std::string name;
public:
    PhoneDisplay(const std::string& n) : name(n) {}
    
    void update(float temperature) override {
        std::cout << name << " phone: Temperature is " 
                  << temperature << "°C\n";
    }
};

class WindowDisplay : public Observer {
    std::string name;
public:
    WindowDisplay(const std::string& n) : name(n) {}
    
    void update(float temperature) override {
        std::cout << name << " window: Temperature is " 
                  << temperature << "°C\n";
    }
};

int main() {
    WeatherStation station;
    
    auto phone = std::make_shared<PhoneDisplay>("Alice's");
    auto window = std::make_shared<WindowDisplay>("Kitchen");
    
    station.attach(phone);
    station.attach(window);
    
    station.set_temperature(25.5);
    station.set_temperature(27.0);
    
    return 0;
}
```

### Strategy Pattern

```cpp
#include <iostream>
#include <memory>

// Strategy interface
class SortStrategy {
public:
    virtual ~SortStrategy() = default;
    virtual void sort(int* data, int size) = 0;
};

// Concrete strategies
class BubbleSort : public SortStrategy {
public:
    void sort(int* data, int size) override {
        std::cout << "Sorting using Bubble Sort\n";
        for (int i = 0; i < size - 1; ++i) {
            for (int j = 0; j < size - i - 1; ++j) {
                if (data[j] > data[j + 1]) {
                    std::swap(data[j], data[j + 1]);
                }
            }
        }
    }
};

class QuickSort : public SortStrategy {
public:
    void sort(int* data, int size) override {
        std::cout << "Sorting using Quick Sort\n";
        std::sort(data, data + size);
    }
};

// Context
class DataProcessor {
    std::unique_ptr<SortStrategy> strategy;
    
public:
    void set_strategy(std::unique_ptr<SortStrategy> s) {
        strategy = std::move(s);
    }
    
    void process(int* data, int size) {
        if (strategy) {
            strategy->sort(data, size);
        }
    }
};

int main() {
    int data[] = {5, 2, 8, 1, 9};
    DataProcessor processor;
    
    // Use bubble sort
    processor.set_strategy(std::make_unique<BubbleSort>());
    processor.process(data, 5);
    
    // Switch to quick sort
    processor.set_strategy(std::make_unique<QuickSort>());
    processor.process(data, 5);
    
    return 0;
}
```

### Command Pattern

```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>

// Receiver
class Light {
    std::string location;
    bool on = false;
    
public:
    Light(const std::string& loc) : location(loc) {}
    
    void turn_on() {
        on = true;
        std::cout << location << " light is ON\n";
    }
    
    void turn_off() {
        on = false;
        std::cout << location << " light is OFF\n";
    }
};

// Command interface
class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

// Concrete commands
class LightOnCommand : public Command {
    Light& light;
    
public:
    LightOnCommand(Light& l) : light(l) {}
    
    void execute() override {
        light.turn_on();
    }
    
    void undo() override {
        light.turn_off();
    }
};

class LightOffCommand : public Command {
    Light& light;
    
public:
    LightOffCommand(Light& l) : light(l) {}
    
    void execute() override {
        light.turn_off();
    }
    
    void undo() override {
        light.turn_on();
    }
};

// Invoker
class RemoteControl {
    std::vector<std::unique_ptr<Command>> history;
    
public:
    void execute_command(std::unique_ptr<Command> command) {
        command->execute();
        history.push_back(std::move(command));
    }
    
    void undo_last() {
        if (!history.empty()) {
            history.back()->undo();
            history.pop_back();
        }
    }
};

int main() {
    Light living_room_light("Living Room");
    Light bedroom_light("Bedroom");
    
    RemoteControl remote;
    
    remote.execute_command(std::make_unique<LightOnCommand>(living_room_light));
    remote.execute_command(std::make_unique<LightOnCommand>(bedroom_light));
    remote.execute_command(std::make_unique<LightOffCommand>(living_room_light));
    
    std::cout << "\nUndo last command:\n";
    remote.undo_last();
    
    return 0;
}
```

---

## 9.5 C++-Specific Idioms

### RAII (Resource Acquisition Is Initialization)

```cpp
#include <iostream>
#include <mutex>
#include <fstream>

// RAII for mutex
void thread_safe_function() {
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);  // RAII: acquires on construction
    
    // Critical section
    std::cout << "Protected code\n";
    
    // Lock released automatically on destruction
}

// RAII for file handling
class FileRAII {
    std::ofstream file;
    
public:
    FileRAII(const std::string& filename) : file(filename) {
        if (!file) {
            throw std::runtime_error("Cannot open file");
        }
    }
    
    ~FileRAII() {
        if (file.is_open()) {
            file.close();
            std::cout << "File closed automatically\n";
        }
    }
    
    void write(const std::string& data) {
        file << data << "\n";
    }
};
```

### Pimpl (Pointer to Implementation)

```cpp
#include <memory>
#include <string>

// Widget.h (public header)
class Widget {
public:
    Widget();
    ~Widget();
    
    // Public interface
    void set_name(const std::string& name);
    std::string get_name() const;
    
private:
    class Impl;  // Forward declaration
    std::unique_ptr<Impl> pimpl;  // Pointer to implementation
};

// Widget.cpp (implementation file)
class Widget::Impl {
public:
    std::string name;
    int secret_data;
    // Complex private members hidden from clients
};

Widget::Widget() : pimpl(std::make_unique<Impl>()) {}

Widget::~Widget() = default;  // Must be in .cpp for unique_ptr to incomplete type

void Widget::set_name(const std::string& name) {
    pimpl->name = name;
}

std::string Widget::get_name() const {
    return pimpl->name;
}

/*
Benefits:
- Faster compilation (changes to Impl don't require recompiling clients)
- Binary compatibility (can change Impl without breaking ABI)
- Hides implementation details
*/
```

### CRTP (Curiously Recurring Template Pattern)

```cpp
#include <iostream>

// CRTP base class
template<typename Derived>
class Counter {
    static int count;
    
protected:
    Counter() { ++count; }
    ~Counter() { --count; }
    
public:
    static int get_count() { return count; }
};

template<typename Derived>
int Counter<Derived>::count = 0;

// Derived classes
class Widget : public Counter<Widget> {
public:
    void show() { std::cout << "Widget\n"; }
};

class Gadget : public Counter<Gadget> {
public:
    void show() { std::cout << "Gadget\n"; }
};

int main() {
    Widget w1, w2;
    Gadget g1, g2, g3;
    
    std::cout << "Widgets: " << Widget::get_count() << "\n";  // 2
    std::cout << "Gadgets: " << Gadget::get_count() << "\n";  // 3
    
    return 0;
}
```

### NVI (Non-Virtual Interface)

```cpp
#include <iostream>

class GameCharacter {
public:
    // Public non-virtual interface
    int health_value() const {
        // Before hook
        std::cout << "Calculating health...\n";
        
        // Call derived implementation
        int result = do_health_value();
        
        // After hook
        std::cout << "Health calculated: " << result << "\n";
        
        return result;
    }
    
    virtual ~GameCharacter() = default;
    
private:
    // Private virtual implementation
    virtual int do_health_value() const = 0;
};

class Warrior : public GameCharacter {
private:
    int do_health_value() const override {
        return 100;  // Warriors have high health
    }
};

class Mage : public GameCharacter {
private:
    int do_health_value() const override {
        return 50;  // Mages have lower health
    }
};

int main() {
    Warrior warrior;
    Mage mage;
    
    std::cout << "Warrior health: " << warrior.health_value() << "\n";
    std::cout << "Mage health: " << mage.health_value() << "\n";
    
    return 0;
}
```

---

## Comprehensive Example: Document Editor Architecture

```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>

// Strategy pattern for rendering
class RenderStrategy {
public:
    virtual ~RenderStrategy() = default;
    virtual void render(const std::string& content) = 0;
};

class HTMLRenderer : public RenderStrategy {
public:
    void render(const std::string& content) override {
        std::cout << "<html><body>" << content << "</body></html>\n";
    }
};

class MarkdownRenderer : public RenderStrategy {
public:
    void render(const std::string& content) override {
        std::cout << "# " << content << "\n";
    }
};

// Command pattern for undo/redo
class DocumentCommand {
public:
    virtual ~DocumentCommand() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

class InsertTextCommand : public DocumentCommand {
    std::string& text;
    std::string inserted;
    size_t position;
    
public:
    InsertTextCommand(std::string& t, const std::string& ins, size_t pos)
        : text(t), inserted(ins), position(pos) {}
    
    void execute() override {
        text.insert(position, inserted);
    }
    
    void undo() override {
        text.erase(position, inserted.length());
    }
};

// Observer pattern for document changes
class DocumentObserver {
public:
    virtual ~DocumentObserver() = default;
    virtual void on_document_changed() = 0;
};

class AutoSave : public DocumentObserver {
public:
    void on_document_changed() override {
        std::cout << "[AutoSave] Document changed, saving...\n";
    }
};

// Main document class
class Document {
    std::string content;
    std::unique_ptr<RenderStrategy> renderer;
    std::vector<std::unique_ptr<DocumentCommand>> history;
    std::vector<std::weak_ptr<DocumentObserver>> observers;
    size_t history_index = 0;
    
public:
    void set_renderer(std::unique_ptr<RenderStrategy> r) {
        renderer = std::move(r);
    }
    
    void attach_observer(std::shared_ptr<DocumentObserver> obs) {
        observers.push_back(obs);
    }
    
    void execute_command(std::unique_ptr<DocumentCommand> cmd) {
        // Remove commands after current position
        history.resize(history_index);
        
        cmd->execute();
        history.push_back(std::move(cmd));
        ++history_index;
        
        notify_observers();
    }
    
    void undo() {
        if (history_index > 0) {
            --history_index;
            history[history_index]->undo();
            notify_observers();
        }
    }
    
    void redo() {
        if (history_index < history.size()) {
            history[history_index]->execute();
            ++history_index;
            notify_observers();
        }
    }
    
    void render() const {
        if (renderer) {
            renderer->render(content);
        }
    }
    
    std::string& get_content() { return content; }
    
private:
    void notify_observers() {
        for (auto& weak_obs : observers) {
            if (auto obs = weak_obs.lock()) {
                obs->on_document_changed();
            }
        }
    }
};

int main() {
    Document doc;
    
    // Set rendering strategy
    doc.set_renderer(std::make_unique<HTMLRenderer>());
    
    // Attach observer
    auto auto_save = std::make_shared<AutoSave>();
    doc.attach_observer(auto_save);
    
    // Execute commands
    doc.execute_command(
        std::make_unique<InsertTextCommand>(doc.get_content(), "Hello", 0)
    );
    doc.execute_command(
        std::make_unique<InsertTextCommand>(doc.get_content(), " World", 5)
    );
    
    std::cout << "\nRendering:\n";
    doc.render();
    
    std::cout << "\nUndo:\n";
    doc.undo();
    doc.render();
    
    std::cout << "\nRedo:\n";
    doc.redo();
    doc.render();
    
    std::cout << "\nSwitch to Markdown:\n";
    doc.set_renderer(std::make_unique<MarkdownRenderer>());
    doc.render();
    
    return 0;
}
```

---

## Conclusion

Part 9 has covered design patterns and software architecture in C++:

1. **SOLID Principles** - Foundation of good design
2. **Creational Patterns** - Singleton, Factory, Abstract Factory, Builder, Prototype
3. **Structural Patterns** - Adapter, Decorator, Facade
4. **Behavioral Patterns** - Observer, Strategy, Command
5. **C++ Idioms** - RAII, Pimpl, CRTP, NVI

**Key Principles:**
- **SOLID**: Write maintainable, extensible code
- **Patterns**: Reusable solutions to common problems
- **Modern C++**: Smart pointers, move semantics enhance patterns
- **Composition**: Prefer over inheritance
- **Separation of Concerns**: Each class has one job

**When to Use Patterns:**
- Don't force patterns where they don't fit
- Patterns should solve actual problems
- Simple solutions are often better
- Recognize patterns in existing code
- Refactor to patterns as complexity grows

**Pattern Selection Guide:**
- **Need flexible object creation?** → Factory, Builder
- **Need to add behavior dynamically?** → Decorator, Strategy
- **Need to simplify complex interfaces?** → Facade
- **Need to decouple sender/receiver?** → Observer, Command
- **Need only one instance?** → Singleton

**Modern C++ Enhancements:**
- Smart pointers eliminate manual memory management
- Move semantics enable efficient resource transfer
- Lambdas simplify Strategy and Command patterns
- Templates enable compile-time patterns (CRTP)

**Next in the series:** Part 10 will cover Build Systems & Development Workflow.

---

## References

- **"Design Patterns"** by Gang of Four
- **"Modern C++ Design"** by Andrei Alexandrescu
- **"Effective C++"** by Scott Meyers
- **C++ Core Guidelines**: Design sections

---

*This blog post is part of the Complete C++ Mastery series.*
