# Design Patterns in C++ — Complete Interview Guide

> **How to use this guide:** Each pattern follows the same structure — *What it is → Why it exists → C++ code → When to use → Interview traps.* Read SOLID Principles first; they are the "why" behind every pattern.

---

## Table of Contents

1. [SOLID Principles](#solid-principles)
2. [Creational Patterns](#creational-patterns)
3. [Structural Patterns](#structural-patterns)
4. [Behavioral Patterns](#behavioral-patterns)
5. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

# SOLID Principles

> SOLID is the foundation. Every design pattern is essentially a SOLID principle in disguise.

---

## S — Single Responsibility Principle (SRP)

**"A class should have only one reason to change."**

A class should do *one thing* and do it well. If a class handles business logic AND logging AND database access, you have three reasons to change it — that's three responsibilities.

### Bad Example

```cpp
class Employee {
public:
    string name;
    double salary;

    // Responsibility 1: Business logic
    double calculateBonus() {
        return salary * 0.10;
    }

    // Responsibility 2: Persistence — WRONG, not Employee's job
    void saveToDatabase() {
        cout << "INSERT INTO employees..." << endl;
    }

    // Responsibility 3: Reporting — WRONG, not Employee's job
    void printReport() {
        cout << "Employee: " << name << ", Salary: " << salary << endl;
    }
};
```

### Good Example

```cpp
// Responsibility 1: Employee data & business logic only
class Employee {
public:
    string name;
    double salary;

    double calculateBonus() {
        return salary * 0.10;
    }
};

// Responsibility 2: Persistence
class EmployeeRepository {
public:
    void save(const Employee& e) {
        cout << "Saving " << e.name << " to DB..." << endl;
    }
};

// Responsibility 3: Reporting
class EmployeeReporter {
public:
    void print(const Employee& e) {
        cout << "Employee: " << e.name << ", Salary: " << e.salary << endl;
    }
};
```

**Interview Tip:** SRP doesn't mean one method per class. It means one *axis of change*. A class managing Employee data can have many methods — as long as they all relate to Employee data.

---

## O — Open/Closed Principle (OCP)

**"Software entities should be open for extension, but closed for modification."**

You should be able to add new behavior without editing existing, tested code. Achieve this through abstraction (interfaces/abstract classes).

### Bad Example

```cpp
class DiscountCalculator {
public:
    double calculate(string customerType, double price) {
        if (customerType == "Regular")   return price * 0.95;
        if (customerType == "Premium")   return price * 0.90;
        if (customerType == "VIP")       return price * 0.80;
        // Adding "Wholesale" means EDITING this function — violates OCP
        return price;
    }
};
```

### Good Example

```cpp
// Abstract base — closed for modification
class DiscountStrategy {
public:
    virtual double calculate(double price) = 0;
    virtual ~DiscountStrategy() = default;
};

// Open for extension — just add new classes
class RegularDiscount : public DiscountStrategy {
public:
    double calculate(double price) override { return price * 0.95; }
};

class PremiumDiscount : public DiscountStrategy {
public:
    double calculate(double price) override { return price * 0.90; }
};

class VIPDiscount : public DiscountStrategy {
public:
    double calculate(double price) override { return price * 0.80; }
};

// Adding Wholesale? Just add a new class — zero changes to existing code
class WholesaleDiscount : public DiscountStrategy {
public:
    double calculate(double price) override { return price * 0.70; }
};

class DiscountCalculator {
    DiscountStrategy* strategy;
public:
    DiscountCalculator(DiscountStrategy* s) : strategy(s) {}
    double calculate(double price) { return strategy->calculate(price); }
};
```

**Interview Tip:** OCP is achieved via polymorphism and the Strategy/Template Method patterns. If you find yourself adding `if/else` or `switch` for new types, you're likely violating OCP.

---

## L — Liskov Substitution Principle (LSP)

**"Objects of a derived class must be substitutable for objects of the base class without altering program correctness."**

If `S` is a subtype of `T`, then objects of type `T` may be replaced with objects of type `S` without breaking the program.

### Bad Example — Classic Rectangle/Square Trap

```cpp
class Rectangle {
protected:
    int width, height;
public:
    virtual void setWidth(int w)  { width = w; }
    virtual void setHeight(int h) { height = h; }
    int area() { return width * height; }
};

// Mathematically a square IS-A rectangle, but...
class Square : public Rectangle {
public:
    void setWidth(int w) override  { width = height = w; } // Forces equal sides
    void setHeight(int h) override { width = height = h; } // Forces equal sides
};

// This function breaks when Square is passed — LSP VIOLATED
void testRectangle(Rectangle* r) {
    r->setWidth(5);
    r->setHeight(4);
    // Expects area = 20, but Square gives 16 (4*4)
    assert(r->area() == 20); // FAILS for Square!
}
```

### Good Example

```cpp
// Don't force inheritance where behavior contracts differ
class Shape {
public:
    virtual int area() = 0;
    virtual ~Shape() = default;
};

class Rectangle : public Shape {
    int width, height;
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    int area() override { return width * height; }
};

class Square : public Shape {
    int side;
public:
    Square(int s) : side(s) {}
    int area() override { return side * side; }
};
```

**Interview Tip:** LSP violations often appear when you use `instanceof` / `dynamic_cast` checks. If a function needs to check what type it received, it means the abstraction is broken.

---

## I — Interface Segregation Principle (ISP)

**"Clients should not be forced to depend on interfaces they do not use."**

Don't create fat interfaces. Split large interfaces into smaller, focused ones so classes only implement what they actually need.

### Bad Example

```cpp
// Fat interface — forces every "worker" to implement everything
class Worker {
public:
    virtual void work() = 0;
    virtual void eat() = 0;   // Robots don't eat!
    virtual void sleep() = 0; // Robots don't sleep!
};

class HumanWorker : public Worker {
public:
    void work()  override { cout << "Human working\n"; }
    void eat()   override { cout << "Human eating\n"; }
    void sleep() override { cout << "Human sleeping\n"; }
};

class RobotWorker : public Worker {
public:
    void work()  override { cout << "Robot working\n"; }
    void eat()   override { /* FORCED — makes no sense */ }  // ISP VIOLATED
    void sleep() override { /* FORCED — makes no sense */ }  // ISP VIOLATED
};
```

### Good Example

```cpp
// Segregated interfaces
class Workable {
public:
    virtual void work() = 0;
    virtual ~Workable() = default;
};

class Feedable {
public:
    virtual void eat() = 0;
    virtual ~Feedable() = default;
};

class Restable {
public:
    virtual void sleep() = 0;
    virtual ~Restable() = default;
};

// Human implements all that apply
class HumanWorker : public Workable, public Feedable, public Restable {
public:
    void work()  override { cout << "Human working\n"; }
    void eat()   override { cout << "Human eating\n"; }
    void sleep() override { cout << "Human sleeping\n"; }
};

// Robot only implements what makes sense
class RobotWorker : public Workable {
public:
    void work() override { cout << "Robot working\n"; }
};
```

**Interview Tip:** ISP is about *role interfaces*, not just splitting randomly. Group methods by the *client* that uses them.

---

## D — Dependency Inversion Principle (DIP)

**"High-level modules should not depend on low-level modules. Both should depend on abstractions."**

Don't hardcode dependencies. Depend on interfaces/abstract classes, not concrete implementations. This enables Dependency Injection.

### Bad Example

```cpp
// Low-level module
class MySQLDatabase {
public:
    void save(string data) {
        cout << "Saving to MySQL: " << data << endl;
    }
};

// High-level module DIRECTLY depends on low-level — DIP VIOLATED
class OrderService {
    MySQLDatabase db; // Hardcoded — can't swap to PostgreSQL/MongoDB
public:
    void placeOrder(string order) {
        // ... business logic ...
        db.save(order); // Tightly coupled
    }
};
```

### Good Example

```cpp
// Abstraction (the interface both depend on)
class IDatabase {
public:
    virtual void save(string data) = 0;
    virtual ~IDatabase() = default;
};

// Low-level modules depend on abstraction
class MySQLDatabase : public IDatabase {
public:
    void save(string data) override {
        cout << "Saving to MySQL: " << data << endl;
    }
};

class MongoDatabase : public IDatabase {
public:
    void save(string data) override {
        cout << "Saving to MongoDB: " << data << endl;
    }
};

// High-level module depends on abstraction — NOT concrete class
class OrderService {
    IDatabase* db; // Depends on abstraction
public:
    OrderService(IDatabase* database) : db(database) {} // Injected from outside
    void placeOrder(string order) {
        db->save(order); // Works with any DB
    }
};

int main() {
    MySQLDatabase mysql;
    OrderService service(&mysql);
    service.placeOrder("Order #1001");

    // Swap to Mongo — zero changes to OrderService
    MongoDatabase mongo;
    OrderService service2(&mongo);
    service2.placeOrder("Order #1002");
}
```

**Interview Tip:** DIP enables unit testing. When `OrderService` depends on `IDatabase`, you can inject a mock database in tests.

---

# Creational Patterns

> These patterns deal with **object creation mechanisms**, aiming to create objects in a manner suitable to the situation.

---

## 1. Singleton

**Intent:** Ensure a class has only one instance and provide a global access point to it.

**Real-world analogy:** A country can have only one president. No matter where you ask, you get the same person.

**When to use:** Logger, Configuration, Thread Pool, Cache, Connection Pool.

```cpp
#include <iostream>
#include <mutex>
using namespace std;

class Logger {
private:
    static Logger* instance;     // The single instance
    static mutex mtx;            // Thread-safety

    // Private constructor — prevents direct instantiation
    Logger() {
        cout << "Logger initialized\n";
    }

public:
    // Deleted copy constructor and assignment — prevents copying
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;

    // Thread-safe lazy initialization (C++11 guarantees static local is thread-safe)
    static Logger& getInstance() {
        static Logger instance; // Guaranteed single instance
        return instance;
    }

    void log(const string& message) {
        cout << "[LOG]: " << message << endl;
    }
};

int main() {
    Logger::getInstance().log("Application started");
    Logger::getInstance().log("User logged in");

    // Both refer to the SAME object
    Logger& l1 = Logger::getInstance();
    Logger& l2 = Logger::getInstance();
    cout << "Same instance? " << (&l1 == &l2 ? "YES" : "NO") << endl;

    return 0;
}
```

**Output:**
```
Logger initialized
[LOG]: Application started
[LOG]: User logged in
Same instance? YES
```

**Interview Traps:**
- Why use static local variable? — Thread-safe in C++11 (no mutex needed for initialization)
- Singleton vs Global variable? — Singleton controls instantiation timing; globals are created at startup
- Is Singleton always bad? — Overuse is a code smell; it hides dependencies and makes testing hard

---

## 2. Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

**Real-world analogy:** A logistics company (Creator) has a `createTransport()` method. Road logistics creates Trucks; Sea logistics creates Ships. The company doesn't know which vehicle it'll use until runtime.

```cpp
#include <iostream>
#include <memory>
using namespace std;

// Product interface
class Transport {
public:
    virtual void deliver() = 0;
    virtual ~Transport() = default;
};

// Concrete Products
class Truck : public Transport {
public:
    void deliver() override {
        cout << "Delivering by road in a Truck\n";
    }
};

class Ship : public Transport {
public:
    void deliver() override {
        cout << "Delivering by sea in a Ship\n";
    }
};

// Creator (abstract) — declares the Factory Method
class Logistics {
public:
    // Factory Method — subclasses override this
    virtual unique_ptr<Transport> createTransport() = 0;

    // Template method uses the factory method
    void planDelivery() {
        auto transport = createTransport();
        cout << "Planning route... ";
        transport->deliver();
    }

    virtual ~Logistics() = default;
};

// Concrete Creators — override the factory method
class RoadLogistics : public Logistics {
public:
    unique_ptr<Transport> createTransport() override {
        return make_unique<Truck>();
    }
};

class SeaLogistics : public Logistics {
public:
    unique_ptr<Transport> createTransport() override {
        return make_unique<Ship>();
    }
};

int main() {
    unique_ptr<Logistics> logistics;

    string type = "road";
    if (type == "road")
        logistics = make_unique<RoadLogistics>();
    else
        logistics = make_unique<SeaLogistics>();

    logistics->planDelivery();
    return 0;
}
```

**Interview Tip:** Factory Method relies on *inheritance* (subclass decides). Abstract Factory relies on *composition* (object contains factories).

---

## 3. Abstract Factory

**Intent:** Provide an interface for creating *families* of related objects without specifying their concrete classes.

**Real-world analogy:** A furniture shop (Abstract Factory) produces matching sets. Victorian Factory produces Victorian Chair + Victorian Sofa. Modern Factory produces Modern Chair + Modern Sofa. Client gets a matched set regardless of style.

```cpp
#include <iostream>
#include <memory>
using namespace std;

// Abstract Products
class Button {
public:
    virtual void render() = 0;
    virtual ~Button() = default;
};

class Checkbox {
public:
    virtual void render() = 0;
    virtual ~Checkbox() = default;
};

// Concrete Products — Windows family
class WindowsButton : public Button {
public:
    void render() override { cout << "Rendering Windows Button\n"; }
};

class WindowsCheckbox : public Checkbox {
public:
    void render() override { cout << "Rendering Windows Checkbox\n"; }
};

// Concrete Products — Mac family
class MacButton : public Button {
public:
    void render() override { cout << "Rendering Mac Button\n"; }
};

class MacCheckbox : public Checkbox {
public:
    void render() override { cout << "Rendering Mac Checkbox\n"; }
};

// Abstract Factory — creates a FAMILY of related products
class GUIFactory {
public:
    virtual unique_ptr<Button>   createButton()   = 0;
    virtual unique_ptr<Checkbox> createCheckbox() = 0;
    virtual ~GUIFactory() = default;
};

// Concrete Factories
class WindowsFactory : public GUIFactory {
public:
    unique_ptr<Button>   createButton()   override { return make_unique<WindowsButton>(); }
    unique_ptr<Checkbox> createCheckbox() override { return make_unique<WindowsCheckbox>(); }
};

class MacFactory : public GUIFactory {
public:
    unique_ptr<Button>   createButton()   override { return make_unique<MacButton>(); }
    unique_ptr<Checkbox> createCheckbox() override { return make_unique<MacCheckbox>(); }
};

// Client — works with any factory without knowing concrete types
class Application {
    unique_ptr<Button>   button;
    unique_ptr<Checkbox> checkbox;
public:
    Application(GUIFactory& factory) {
        button   = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    void render() {
        button->render();
        checkbox->render();
    }
};

int main() {
    string os = "Windows";
    unique_ptr<GUIFactory> factory;

    if (os == "Windows")
        factory = make_unique<WindowsFactory>();
    else
        factory = make_unique<MacFactory>();

    Application app(*factory);
    app.render();
    return 0;
}
```

**Key Difference:**
| | Factory Method | Abstract Factory |
|---|---|---|
| Creates | ONE product | FAMILY of products |
| Uses | Inheritance | Composition |
| Override | createProduct() method | Inject factory object |

---

## 4. Builder

**Intent:** Separate the construction of a complex object from its representation. The same construction process can create different representations.

**Real-world analogy:** Building a house. Director says "build a house." Builder handles "pour foundation, build walls, install roof" step by step. You can swap builders (WoodHouseBuilder vs StoneHouseBuilder) with the same director.

```cpp
#include <iostream>
#include <string>
using namespace std;

// Product
class Pizza {
public:
    string dough, sauce, topping, size;

    void describe() {
        cout << "Pizza [" << size << "]: " << dough << " dough, "
             << sauce << " sauce, " << topping << " topping\n";
    }
};

// Abstract Builder
class PizzaBuilder {
protected:
    Pizza pizza;
public:
    virtual void buildDough()   = 0;
    virtual void buildSauce()   = 0;
    virtual void buildTopping() = 0;
    virtual void buildSize()    = 0;
    Pizza getResult() { return pizza; }
    virtual ~PizzaBuilder() = default;
};

// Concrete Builders
class MargheritaBuilder : public PizzaBuilder {
public:
    void buildDough()   override { pizza.dough   = "Thin Crust"; }
    void buildSauce()   override { pizza.sauce   = "Tomato"; }
    void buildTopping() override { pizza.topping = "Mozzarella"; }
    void buildSize()    override { pizza.size    = "Medium"; }
};

class BBQChickenBuilder : public PizzaBuilder {
public:
    void buildDough()   override { pizza.dough   = "Thick Crust"; }
    void buildSauce()   override { pizza.sauce   = "BBQ"; }
    void buildTopping() override { pizza.topping = "Chicken + Peppers"; }
    void buildSize()    override { pizza.size    = "Large"; }
};

// Director — controls the build ORDER
class Chef {
public:
    Pizza construct(PizzaBuilder& builder) {
        builder.buildSize();
        builder.buildDough();
        builder.buildSauce();
        builder.buildTopping();
        return builder.getResult();
    }
};

// Modern Fluent Builder (no Director needed — popular in interviews)
class Burger {
    string bun, patty, sauce;
    bool lettuce = false, cheese = false;
public:
    class Builder {
        string bun, patty, sauce;
        bool lettuce = false, cheese = false;
    public:
        Builder& setBun(string b)   { bun = b;       return *this; }
        Builder& setPatty(string p) { patty = p;     return *this; }
        Builder& setSauce(string s) { sauce = s;     return *this; }
        Builder& withLettuce()      { lettuce = true; return *this; }
        Builder& withCheese()       { cheese = true;  return *this; }
        Burger build() {
            Burger b;
            b.bun = bun; b.patty = patty; b.sauce = sauce;
            b.lettuce = lettuce; b.cheese = cheese;
            return b;
        }
    };

    void describe() {
        cout << "Burger: " << bun << " bun, " << patty << " patty, "
             << sauce << " sauce"
             << (lettuce ? ", lettuce" : "")
             << (cheese  ? ", cheese"  : "") << "\n";
    }
};

int main() {
    // Director-based
    Chef chef;
    MargheritaBuilder margBuilder;
    Pizza pizza = chef.construct(margBuilder);
    pizza.describe();

    // Fluent Builder
    Burger burger = Burger::Builder()
        .setBun("Sesame")
        .setPatty("Beef")
        .setSauce("Ketchup")
        .withCheese()
        .withLettuce()
        .build();
    burger.describe();

    return 0;
}
```

**Interview Tip:** Builder vs Constructor — use Builder when you have many optional parameters (avoids "telescoping constructor" anti-pattern). Builder vs Abstract Factory — Builder constructs step-by-step; Abstract Factory returns product immediately.

---

## 5. Prototype

**Intent:** Create new objects by copying (cloning) an existing object.

**Real-world analogy:** A biologist clones a cell — the clone starts with all the same DNA. You don't rebuild from scratch.

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

// Prototype interface
class Shape {
protected:
    string color;
public:
    Shape(string c) : color(c) {}
    virtual unique_ptr<Shape> clone() = 0; // The clone method
    virtual void draw() = 0;
    virtual ~Shape() = default;
};

// Concrete Prototypes
class Circle : public Shape {
    int radius;
public:
    Circle(int r, string c) : Shape(c), radius(r) {}

    // Deep copy via clone
    unique_ptr<Shape> clone() override {
        return make_unique<Circle>(*this); // Copy constructor
    }

    void draw() override {
        cout << "Circle [radius=" << radius << ", color=" << color << "]\n";
    }
};

class Rectangle : public Shape {
    int width, height;
public:
    Rectangle(int w, int h, string c) : Shape(c), width(w), height(h) {}

    unique_ptr<Shape> clone() override {
        return make_unique<Rectangle>(*this);
    }

    void draw() override {
        cout << "Rectangle [" << width << "x" << height
             << ", color=" << color << "]\n";
    }
};

int main() {
    // Original shapes
    Circle original(10, "Red");
    original.draw();

    // Clone — fast copy, no need to rebuild
    auto cloned = original.clone();
    cloned->draw();

    // Prototype registry (cache of pre-built prototypes)
    // Useful when object creation is expensive
    map<string, unique_ptr<Shape>> registry;
    registry["small_circle"] = make_unique<Circle>(5, "Blue");
    registry["big_rect"]     = make_unique<Rectangle>(100, 50, "Green");

    // Clone from registry whenever needed
    auto shape = registry["small_circle"]->clone();
    shape->draw();

    return 0;
}
```

**When to use:** When object creation is expensive (DB call, network request, complex initialization) and you need many similar objects. Also used in game engines (clone NPC prototypes).

---

# Structural Patterns

> These patterns deal with **object composition**, creating relationships between objects to form larger structures.

---

## 6. Adapter

**Intent:** Convert the interface of a class into another interface that clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Real-world analogy:** A power adapter lets your US laptop work in a European socket. The laptop (client) expects US socket; the wall (adaptee) provides EU socket; the adapter bridges the gap.

```cpp
#include <iostream>
using namespace std;

// Target Interface — what the client expects
class MediaPlayer {
public:
    virtual void play(string filename) = 0;
    virtual ~MediaPlayer() = default;
};

// Adaptee — existing class with incompatible interface
class VLCPlayer {
public:
    void playVLC(string filename) {
        cout << "VLC playing: " << filename << endl;
    }
};

class MP4Player {
public:
    void playMP4(string filename) {
        cout << "MP4 player playing: " << filename << endl;
    }
};

// Adapter — wraps Adaptee, implements Target interface
class MediaAdapter : public MediaPlayer {
    VLCPlayer  vlcPlayer;
    MP4Player  mp4Player;
    string     format;

public:
    MediaAdapter(string audioType) : format(audioType) {}

    void play(string filename) override {
        if (format == "vlc")
            vlcPlayer.playVLC(filename);
        else if (format == "mp4")
            mp4Player.playMP4(filename);
        else
            cout << "Unsupported format: " << format << endl;
    }
};

// Client — only knows about MediaPlayer
class AudioPlayer : public MediaPlayer {
public:
    void play(string filename) override {
        string ext = filename.substr(filename.find_last_of('.') + 1);

        if (ext == "mp3") {
            cout << "Native MP3 playing: " << filename << endl;
        } else if (ext == "vlc" || ext == "mp4") {
            MediaAdapter adapter(ext);  // Use adapter for other formats
            adapter.play(filename);
        } else {
            cout << "Unknown format: " << ext << endl;
        }
    }
};

int main() {
    AudioPlayer player;
    player.play("song.mp3");
    player.play("movie.vlc");
    player.play("video.mp4");
    return 0;
}
```

**Two types of Adapter:**
- **Object Adapter** (above) — uses composition (wraps the adaptee)
- **Class Adapter** — uses multiple inheritance (less common in modern C++)

---

## 7. Bridge

**Intent:** Decouple an abstraction from its implementation so that the two can vary independently.

**Real-world analogy:** A remote control (abstraction) works with any TV brand (implementation). The remote doesn't need to change when you switch from Samsung to Sony.

```cpp
#include <iostream>
using namespace std;

// Implementation interface (the "back end")
class DrawingAPI {
public:
    virtual void drawCircle(int x, int y, int radius) = 0;
    virtual ~DrawingAPI() = default;
};

// Concrete Implementations
class OpenGLAPI : public DrawingAPI {
public:
    void drawCircle(int x, int y, int radius) override {
        cout << "OpenGL Circle at (" << x << "," << y << ") r=" << radius << "\n";
    }
};

class DirectXAPI : public DrawingAPI {
public:
    void drawCircle(int x, int y, int radius) override {
        cout << "DirectX Circle at (" << x << "," << y << ") r=" << radius << "\n";
    }
};

// Abstraction (the "front end") — holds a reference to Implementation
class Shape {
protected:
    DrawingAPI* api; // Bridge to implementation
public:
    Shape(DrawingAPI* a) : api(a) {}
    virtual void draw() = 0;
    virtual void resize(float factor) = 0;
    virtual ~Shape() = default;
};

// Refined Abstraction
class Circle : public Shape {
    int x, y, radius;
public:
    Circle(int x, int y, int r, DrawingAPI* api)
        : Shape(api), x(x), y(y), radius(r) {}

    void draw() override {
        api->drawCircle(x, y, radius); // Delegates to implementation
    }

    void resize(float factor) override {
        radius *= factor;
    }
};

int main() {
    OpenGLAPI opengl;
    DirectXAPI directx;

    Circle c1(0, 0, 10, &opengl);
    Circle c2(5, 5, 20, &directx);

    c1.draw();  // OpenGL renders it
    c2.draw();  // DirectX renders it

    // Swap implementation at runtime!
    Circle c3(1, 1, 15, &opengl);
    c3.draw();
    c3.resize(2);
    c3.draw();

    return 0;
}
```

**Bridge vs Adapter:** Adapter makes *existing* incompatible things work together. Bridge is designed *upfront* to let abstraction and implementation evolve independently.

---

## 8. Composite

**Intent:** Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions uniformly.

**Real-world analogy:** A file system — a Folder can contain Files OR other Folders. You can call `getSize()` on a File (returns its size) or a Folder (returns sum of children sizes). Same interface, different behavior.

```cpp
#include <iostream>
#include <vector>
#include <memory>
using namespace std;

// Component — common interface for leaf and composite
class FileSystemItem {
public:
    virtual void display(int depth = 0) = 0;
    virtual int  getSize() = 0;
    virtual ~FileSystemItem() = default;
};

// Leaf — no children
class File : public FileSystemItem {
    string name;
    int    size;
public:
    File(string n, int s) : name(n), size(s) {}

    void display(int depth = 0) override {
        cout << string(depth * 2, ' ') << "- " << name
             << " (" << size << "KB)\n";
    }

    int getSize() override { return size; }
};

// Composite — has children
class Folder : public FileSystemItem {
    string name;
    vector<unique_ptr<FileSystemItem>> children;
public:
    Folder(string n) : name(n) {}

    void add(unique_ptr<FileSystemItem> item) {
        children.push_back(move(item));
    }

    void display(int depth = 0) override {
        cout << string(depth * 2, ' ') << "+ " << name
             << " (" << getSize() << "KB)\n";
        for (auto& child : children)
            child->display(depth + 1);
    }

    // Recursively sums children
    int getSize() override {
        int total = 0;
        for (auto& child : children)
            total += child->getSize();
        return total;
    }
};

int main() {
    auto root = make_unique<Folder>("root");

    auto docs = make_unique<Folder>("Documents");
    docs->add(make_unique<File>("resume.pdf", 200));
    docs->add(make_unique<File>("cover_letter.docx", 50));

    auto pics = make_unique<Folder>("Pictures");
    pics->add(make_unique<File>("photo1.jpg", 3000));
    pics->add(make_unique<File>("photo2.jpg", 2500));

    root->add(move(docs));
    root->add(move(pics));
    root->add(make_unique<File>("notes.txt", 10));

    root->display();
    cout << "Total size: " << root->getSize() << "KB\n";
    return 0;
}
```

**Output:**
```
+ root (5760KB)
  + Documents (250KB)
    - resume.pdf (200KB)
    - cover_letter.docx (50KB)
  + Pictures (5500KB)
    - photo1.jpg (3000KB)
    - photo2.jpg (2500KB)
  - notes.txt (10KB)
Total size: 5760KB
```

---

## 9. Decorator

**Intent:** Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**Real-world analogy:** Coffee order — you start with plain coffee, then add milk (+₹20), then add sugar (+₹5), then whipped cream (+₹30). Each addition wraps the previous order.

```cpp
#include <iostream>
using namespace std;

// Component interface
class Coffee {
public:
    virtual string getDescription() = 0;
    virtual double getCost() = 0;
    virtual ~Coffee() = default;
};

// Concrete Component
class SimpleCoffee : public Coffee {
public:
    string getDescription() override { return "Simple Coffee"; }
    double getCost()        override { return 50.0; }
};

// Abstract Decorator — wraps a Coffee
class CoffeeDecorator : public Coffee {
protected:
    Coffee* coffee; // Wraps the decorated coffee
public:
    CoffeeDecorator(Coffee* c) : coffee(c) {}
};

// Concrete Decorators
class MilkDecorator : public CoffeeDecorator {
public:
    MilkDecorator(Coffee* c) : CoffeeDecorator(c) {}
    string getDescription() override {
        return coffee->getDescription() + " + Milk";
    }
    double getCost() override { return coffee->getCost() + 20.0; }
};

class SugarDecorator : public CoffeeDecorator {
public:
    SugarDecorator(Coffee* c) : CoffeeDecorator(c) {}
    string getDescription() override {
        return coffee->getDescription() + " + Sugar";
    }
    double getCost() override { return coffee->getCost() + 5.0; }
};

class WhipDecorator : public CoffeeDecorator {
public:
    WhipDecorator(Coffee* c) : CoffeeDecorator(c) {}
    string getDescription() override {
        return coffee->getDescription() + " + Whip";
    }
    double getCost() override { return coffee->getCost() + 30.0; }
};

int main() {
    Coffee* order = new SimpleCoffee();
    cout << order->getDescription() << " -> ₹" << order->getCost() << "\n";

    order = new MilkDecorator(order);
    cout << order->getDescription() << " -> ₹" << order->getCost() << "\n";

    order = new SugarDecorator(order);
    order = new WhipDecorator(order);
    cout << order->getDescription() << " -> ₹" << order->getCost() << "\n";

    return 0;
}
```

**Decorator vs Inheritance:** Decorator is runtime-flexible (wrap at runtime); Inheritance is compile-time. With N features, inheritance creates 2^N subclasses; Decorator composes N classes dynamically.

---

## 10. Facade

**Intent:** Provide a simplified interface to a complex subsystem.

**Real-world analogy:** A hotel concierge. You say "I need a taxi, restaurant reservation, and wake-up call." The concierge handles all the complexity behind the scenes. You don't talk to the taxi dispatcher, restaurant, and front desk individually.

```cpp
#include <iostream>
using namespace std;

// Complex subsystem classes
class DVDPlayer {
public:
    void on()         { cout << "DVD Player ON\n"; }
    void play(string movie) { cout << "DVD playing: " << movie << "\n"; }
    void off()        { cout << "DVD Player OFF\n"; }
};

class Projector {
public:
    void on()         { cout << "Projector ON\n"; }
    void setInput(string input) { cout << "Projector input: " << input << "\n"; }
    void off()        { cout << "Projector OFF\n"; }
};

class SoundSystem {
public:
    void on()         { cout << "Sound System ON\n"; }
    void setVolume(int v) { cout << "Volume set to " << v << "\n"; }
    void off()        { cout << "Sound System OFF\n"; }
};

class Lights {
public:
    void dim(int level) { cout << "Lights dimmed to " << level << "%\n"; }
    void on()           { cout << "Lights ON\n"; }
};

// Facade — simple interface over the complex subsystem
class HomeTheaterFacade {
    DVDPlayer   dvd;
    Projector   projector;
    SoundSystem sound;
    Lights      lights;

public:
    void watchMovie(string movie) {
        cout << "\n--- Starting Movie Mode ---\n";
        lights.dim(20);
        projector.on();
        projector.setInput("DVD");
        sound.on();
        sound.setVolume(8);
        dvd.on();
        dvd.play(movie);
        cout << "--- Enjoy your movie! ---\n\n";
    }

    void endMovie() {
        cout << "\n--- Shutting Down ---\n";
        dvd.off();
        sound.off();
        projector.off();
        lights.on();
        cout << "--- Goodnight! ---\n";
    }
};

int main() {
    HomeTheaterFacade theater;
    theater.watchMovie("Inception");
    theater.endMovie();
    return 0;
}
```

**Interview Tip:** Facade doesn't prevent direct access to subsystem — it just provides a simpler path. It's about convenience, not restriction (that's Proxy).

---

## 11. Flyweight

**Intent:** Use sharing to support a large number of fine-grained objects efficiently. Separate *intrinsic* (shared, immutable) state from *extrinsic* (context-specific) state.

**Real-world analogy:** A forest with 1 million trees. Each tree type (Oak, Pine, Birch) shares the same texture and color data (intrinsic). Only position and size differ per tree (extrinsic).

```cpp
#include <iostream>
#include <map>
#include <vector>
#include <memory>
using namespace std;

// Flyweight — stores INTRINSIC (shared) state
class TreeType {
    string name;
    string color;
    string texture; // Imagine this is megabytes of texture data

public:
    TreeType(string n, string c, string t)
        : name(n), color(c), texture(t) {}

    void draw(int x, int y) {
        cout << "Drawing " << name << " [" << color << "] at (" << x << "," << y << ")\n";
    }
};

// Flyweight Factory — manages shared flyweight objects
class TreeFactory {
    map<string, shared_ptr<TreeType>> types;

public:
    shared_ptr<TreeType> getTreeType(string name, string color, string texture) {
        string key = name + "_" + color;
        if (types.find(key) == types.end()) {
            cout << "Creating new TreeType: " << key << "\n";
            types[key] = make_shared<TreeType>(name, color, texture);
        }
        return types[key]; // Return SHARED instance
    }

    int getTypeCount() { return types.size(); }
};

// Context — stores EXTRINSIC (per-object) state
struct Tree {
    int x, y;                      // Extrinsic — unique per tree
    shared_ptr<TreeType> type;     // Intrinsic — shared

    Tree(int x, int y, shared_ptr<TreeType> t) : x(x), y(y), type(t) {}

    void draw() { type->draw(x, y); }
};

// Forest
class Forest {
    vector<Tree> trees;
    TreeFactory  factory;

public:
    void plantTree(int x, int y, string name, string color, string texture) {
        auto type = factory.getTreeType(name, color, texture);
        trees.emplace_back(x, y, type);
    }

    void draw() {
        for (auto& tree : trees) tree.draw();
        cout << "Total trees: " << trees.size()
             << ", Unique types: " << factory.getTypeCount() << "\n";
    }
};

int main() {
    Forest forest;
    forest.plantTree(1,  1,  "Oak",  "Green",  "oak_texture.png");
    forest.plantTree(5,  10, "Oak",  "Green",  "oak_texture.png"); // Reuses same type
    forest.plantTree(3,  7,  "Pine", "DarkGreen", "pine_texture.png");
    forest.plantTree(9,  2,  "Pine", "DarkGreen", "pine_texture.png"); // Reuses same type
    forest.plantTree(12, 4,  "Oak",  "Green",  "oak_texture.png"); // Reuses Oak again
    forest.draw();
    return 0;
}
```

**When to use:** When you need a HUGE number of similar objects (particles, characters in a game, glyphs in a text editor) and RAM is a constraint.

---

## 12. Proxy

**Intent:** Provide a surrogate or placeholder for another object to control access to it.

**Three main types:**
- **Virtual Proxy** — lazy initialization (create expensive object only when needed)
- **Protection Proxy** — access control
- **Remote Proxy** — represents object in a different address space

```cpp
#include <iostream>
using namespace std;

// Subject interface
class Image {
public:
    virtual void display() = 0;
    virtual ~Image() = default;
};

// Real Subject — expensive to create
class RealImage : public Image {
    string filename;
public:
    RealImage(string f) : filename(f) {
        cout << "Loading image from disk: " << filename << "\n"; // Heavy operation
    }
    void display() override {
        cout << "Displaying: " << filename << "\n";
    }
};

// Virtual Proxy — delays creation until absolutely needed
class ImageProxy : public Image {
    string              filename;
    mutable RealImage*  realImage = nullptr; // Lazy

public:
    ImageProxy(string f) : filename(f) {
        cout << "Proxy created for: " << filename << " (not loaded yet)\n";
    }

    void display() override {
        if (!realImage) {
            realImage = new RealImage(filename); // Load on first access
        }
        realImage->display();
    }

    ~ImageProxy() { delete realImage; }
};

// Protection Proxy
class SecureImage : public Image {
    Image*  realImage;
    string  userRole;

public:
    SecureImage(Image* img, string role) : realImage(img), userRole(role) {}

    void display() override {
        if (userRole == "admin" || userRole == "editor") {
            realImage->display();
        } else {
            cout << "Access Denied: insufficient privileges\n";
        }
    }
};

int main() {
    cout << "--- Virtual Proxy ---\n";
    ImageProxy proxy1("photo.jpg");
    ImageProxy proxy2("document.png");

    // Images loaded only when actually displayed
    cout << "\nDisplaying proxy1:\n";
    proxy1.display(); // Loads NOW
    cout << "\nDisplaying proxy1 again:\n";
    proxy1.display(); // Uses cached — no reload

    cout << "\n--- Protection Proxy ---\n";
    RealImage   real("secret.jpg");
    SecureImage adminAccess(&real, "admin");
    SecureImage guestAccess(&real, "guest");

    adminAccess.display(); // Allowed
    guestAccess.display(); // Denied

    return 0;
}
```

---

# Behavioral Patterns

> These patterns deal with **communication and responsibility between objects**.

---

## 13. Chain of Responsibility

**Intent:** Pass requests along a chain of handlers. Each handler decides to process the request or pass it to the next handler.

**Real-world analogy:** Customer support — Level 1 handles basic issues; escalates to Level 2 for complex issues; escalates to Manager for critical issues.

```cpp
#include <iostream>
using namespace std;

// Handler interface
class SupportHandler {
protected:
    SupportHandler* next = nullptr;
public:
    void setNext(SupportHandler* n) { next = n; }

    virtual void handle(int level) {
        if (next) next->handle(level);
        else cout << "Issue unresolved — no handler available\n";
    }
    virtual ~SupportHandler() = default;
};

// Concrete Handlers
class Level1Support : public SupportHandler {
public:
    void handle(int level) override {
        if (level == 1) {
            cout << "Level 1 Support: Resolved basic issue\n";
        } else {
            cout << "Level 1: Escalating to Level 2...\n";
            SupportHandler::handle(level);
        }
    }
};

class Level2Support : public SupportHandler {
public:
    void handle(int level) override {
        if (level == 2) {
            cout << "Level 2 Support: Resolved complex issue\n";
        } else {
            cout << "Level 2: Escalating to Manager...\n";
            SupportHandler::handle(level);
        }
    }
};

class Manager : public SupportHandler {
public:
    void handle(int level) override {
        if (level == 3) {
            cout << "Manager: Resolved critical issue\n";
        } else {
            SupportHandler::handle(level); // Goes to next or unresolved
        }
    }
};

int main() {
    Level1Support l1;
    Level2Support l2;
    Manager       mgr;

    // Build the chain
    l1.setNext(&l2);
    l2.setNext(&mgr);

    cout << "Issue Level 1: "; l1.handle(1);
    cout << "Issue Level 2: "; l1.handle(2);
    cout << "Issue Level 3: "; l1.handle(3);
    cout << "Issue Level 4: "; l1.handle(4);

    return 0;
}
```

---

## 14. Command

**Intent:** Encapsulate a request as an object, allowing you to parameterize clients, queue requests, log them, or support undoable operations.

**Real-world analogy:** A restaurant order slip — the waiter takes the order (command), passes it to the kitchen (invoker), which executes it. The slip can be queued, cancelled, or reordered.

```cpp
#include <iostream>
#include <stack>
#include <memory>
using namespace std;

// Receiver — the object that actually does the work
class TextEditor {
    string text;
public:
    void insertText(string t, int pos) {
        text.insert(pos, t);
        cout << "Text: \"" << text << "\"\n";
    }
    void deleteText(int pos, int len) {
        text.erase(pos, len);
        cout << "Text: \"" << text << "\"\n";
    }
    string getText() { return text; }
};

// Command interface
class Command {
public:
    virtual void execute() = 0;
    virtual void undo()    = 0;
    virtual ~Command() = default;
};

// Concrete Commands
class InsertCommand : public Command {
    TextEditor& editor;
    string      text;
    int         position;
public:
    InsertCommand(TextEditor& e, string t, int p)
        : editor(e), text(t), position(p) {}

    void execute() override { editor.insertText(text, position); }
    void undo()    override { editor.deleteText(position, text.length()); }
};

// Invoker — manages command history
class CommandHistory {
    stack<unique_ptr<Command>> history;
public:
    void execute(unique_ptr<Command> cmd) {
        cmd->execute();
        history.push(move(cmd));
    }

    void undo() {
        if (!history.empty()) {
            cout << "Undoing last command...\n";
            history.top()->undo();
            history.pop();
        }
    }
};

int main() {
    TextEditor    editor;
    CommandHistory invoker;

    invoker.execute(make_unique<InsertCommand>(editor, "Hello", 0));
    invoker.execute(make_unique<InsertCommand>(editor, " World", 5));
    invoker.execute(make_unique<InsertCommand>(editor, "!", 11));

    invoker.undo();
    invoker.undo();

    return 0;
}
```

---

## 15. Iterator

**Intent:** Provide a way to sequentially access elements of a collection without exposing its underlying representation.

```cpp
#include <iostream>
#include <vector>
using namespace std;

// Custom iterator for a collection
template<typename T>
class NumberCollection {
    vector<T> items;

public:
    void add(T item) { items.push_back(item); }

    // Iterator class
    class Iterator {
        vector<T>& items;
        size_t     index;
    public:
        Iterator(vector<T>& v, size_t i = 0) : items(v), index(i) {}

        T& operator*()  { return items[index]; }
        Iterator& operator++() { ++index; return *this; }
        bool operator!=(const Iterator& other) { return index != other.index; }
    };

    Iterator begin() { return Iterator(items, 0); }
    Iterator end()   { return Iterator(items, items.size()); }
};

int main() {
    NumberCollection<int> nums;
    nums.add(10); nums.add(20); nums.add(30); nums.add(40);

    // Range-for loop works thanks to begin()/end()
    for (int n : nums)
        cout << n << " ";
    cout << "\n";

    // Modern C++ STL iterators (use these in practice)
    vector<int> vec = {1, 2, 3, 4, 5};
    auto it = vec.begin();
    while (it != vec.end()) {
        cout << *it << " ";
        ++it;
    }
    return 0;
}
```

---

## 16. Observer

**Intent:** Define a one-to-many dependency between objects so that when one object (Subject) changes state, all dependents (Observers) are notified automatically.

**Real-world analogy:** YouTube channel — you subscribe (observe) to a channel (subject). When they post (state change), all subscribers get notified.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// Observer interface
class Observer {
public:
    virtual void update(string event, int data) = 0;
    virtual ~Observer() = default;
};

// Subject
class StockMarket {
    vector<Observer*> observers;
    int               stockPrice;
    string            stockName;

public:
    StockMarket(string name, int price) : stockName(name), stockPrice(price) {}

    void subscribe(Observer* obs)   { observers.push_back(obs); }
    void unsubscribe(Observer* obs) {
        observers.erase(remove(observers.begin(), observers.end(), obs), observers.end());
    }

    void setPrice(int price) {
        string event = (price > stockPrice) ? "PRICE_RISE" : "PRICE_FALL";
        stockPrice = price;
        cout << "\n[Market] " << stockName << " price updated to " << stockPrice << "\n";
        notify(event);
    }

    void notify(string event) {
        for (auto* obs : observers)
            obs->update(event, stockPrice);
    }
};

// Concrete Observers
class Investor : public Observer {
    string name;
public:
    Investor(string n) : name(n) {}
    void update(string event, int data) override {
        cout << "  [Investor " << name << "] " << event
             << " -> price is now " << data << "\n";
    }
};

class TradingBot : public Observer {
public:
    void update(string event, int data) override {
        if (event == "PRICE_FALL" && data < 100)
            cout << "  [TradingBot] BUY signal triggered at " << data << "\n";
        if (event == "PRICE_RISE" && data > 150)
            cout << "  [TradingBot] SELL signal triggered at " << data << "\n";
    }
};

int main() {
    StockMarket apple("AAPL", 120);

    Investor    raj("Raj");
    Investor    priya("Priya");
    TradingBot  bot;

    apple.subscribe(&raj);
    apple.subscribe(&priya);
    apple.subscribe(&bot);

    apple.setPrice(90);   // Fall — bot BUY signal
    apple.setPrice(160);  // Rise — bot SELL signal

    apple.unsubscribe(&priya);
    apple.setPrice(100);  // Priya doesn't get notified

    return 0;
}
```

---

## 17. Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**Real-world analogy:** Navigation app — you can choose "Fastest route," "Shortest route," or "Avoid highways." The strategy (algorithm) is swappable at runtime.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// Strategy interface
class SortStrategy {
public:
    virtual void sort(vector<int>& data) = 0;
    virtual string name() = 0;
    virtual ~SortStrategy() = default;
};

// Concrete Strategies
class BubbleSort : public SortStrategy {
public:
    void sort(vector<int>& data) override {
        int n = data.size();
        for (int i = 0; i < n - 1; i++)
            for (int j = 0; j < n - i - 1; j++)
                if (data[j] > data[j + 1])
                    swap(data[j], data[j + 1]);
    }
    string name() override { return "BubbleSort"; }
};

class QuickSort : public SortStrategy {
public:
    void sort(vector<int>& data) override {
        std::sort(data.begin(), data.end()); // Simulating quicksort
    }
    string name() override { return "QuickSort"; }
};

class MergeSort : public SortStrategy {
public:
    void sort(vector<int>& data) override {
        stable_sort(data.begin(), data.end()); // Simulating merge sort
    }
    string name() override { return "MergeSort"; }
};

// Context
class Sorter {
    SortStrategy* strategy;
public:
    Sorter(SortStrategy* s) : strategy(s) {}

    void setStrategy(SortStrategy* s) { strategy = s; } // Swap at runtime

    void sort(vector<int>& data) {
        cout << "Sorting with " << strategy->name() << "...\n";
        strategy->sort(data);
        for (int x : data) cout << x << " ";
        cout << "\n";
    }
};

int main() {
    vector<int> data = {64, 34, 25, 12, 22, 11, 90};

    BubbleSort bubble;
    QuickSort  quick;

    Sorter sorter(&bubble);
    sorter.sort(data);

    data = {64, 34, 25, 12, 22, 11, 90};
    sorter.setStrategy(&quick);
    sorter.sort(data);

    return 0;
}
```

**Strategy vs State:** Both look similar in code. The difference is intent — Strategy is about swappable algorithms (client picks); State is about object transitions (state itself drives change).

---

## 18. State

**Intent:** Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

**Real-world analogy:** A vending machine — in "HasMoney" state it can dispense; in "Empty" state it can't; in "NoMoney" state it waits. Same machine, different behavior per state.

```cpp
#include <iostream>
using namespace std;

class VendingMachine; // Forward declaration

// State interface
class VendingState {
public:
    virtual void insertCoin(VendingMachine* machine) = 0;
    virtual void selectItem(VendingMachine* machine) = 0;
    virtual void dispense(VendingMachine* machine)   = 0;
    virtual ~VendingState() = default;
};

// States
class NoCoinState : public VendingState {
public:
    void insertCoin(VendingMachine* m) override;
    void selectItem(VendingMachine* m) override { cout << "Insert a coin first!\n"; }
    void dispense(VendingMachine* m)   override { cout << "Insert a coin first!\n"; }
};

class HasCoinState : public VendingState {
public:
    void insertCoin(VendingMachine* m) override { cout << "Coin already inserted\n"; }
    void selectItem(VendingMachine* m) override;
    void dispense(VendingMachine* m)   override { cout << "Select an item first\n"; }
};

class DispensingState : public VendingState {
public:
    void insertCoin(VendingMachine* m) override { cout << "Please wait, dispensing...\n"; }
    void selectItem(VendingMachine* m) override { cout << "Please wait, dispensing...\n"; }
    void dispense(VendingMachine* m)   override;
};

// Context
class VendingMachine {
    VendingState* currentState;
    int           itemCount;

    NoCoinState     noCoinState;
    HasCoinState    hasCoinState;
    DispensingState dispensingState;

public:
    VendingMachine(int items) : itemCount(items), currentState(&noCoinState) {}

    void setState(VendingState* s) { currentState = s; }
    NoCoinState*     getNoCoinState()     { return &noCoinState; }
    HasCoinState*    getHasCoinState()    { return &hasCoinState; }
    DispensingState* getDispensingState() { return &dispensingState; }
    int              getItems()           { return itemCount; }
    void             decrementItem()      { itemCount--; }

    void insertCoin() { currentState->insertCoin(this); }
    void selectItem() { currentState->selectItem(this); }
    void dispense()   { currentState->dispense(this); }
};

// State method implementations (after VendingMachine is defined)
void NoCoinState::insertCoin(VendingMachine* m) {
    cout << "Coin inserted!\n";
    m->setState(m->getHasCoinState());
}

void HasCoinState::selectItem(VendingMachine* m) {
    cout << "Item selected!\n";
    m->setState(m->getDispensingState());
}

void DispensingState::dispense(VendingMachine* m) {
    if (m->getItems() > 0) {
        cout << "Item dispensed! Enjoy.\n";
        m->decrementItem();
    }
    m->setState(m->getNoCoinState());
}

int main() {
    VendingMachine vm(3);

    vm.selectItem();  // No coin — rejected
    vm.insertCoin();  // Coin inserted
    vm.selectItem();  // Item selected
    vm.dispense();    // Item dispensed

    return 0;
}
```

---

## 19. Template Method

**Intent:** Define the skeleton of an algorithm in a base class, deferring some steps to subclasses. Subclasses can redefine certain steps without changing the algorithm's structure.

```cpp
#include <iostream>
using namespace std;

// Abstract class with the template method
class DataProcessor {
public:
    // Template Method — final skeleton, cannot be overridden
    void process() {
        readData();    // Step 1
        processData(); // Step 2 — subclass specific
        writeData();   // Step 3
        generateReport(); // Optional hook
    }

protected:
    virtual void readData()    = 0; // Must override
    virtual void processData() = 0; // Must override
    virtual void writeData()   = 0; // Must override

    // Hook — optional override (has default behavior)
    virtual void generateReport() {
        cout << "  [Default] No report generated.\n";
    }
};

class CSVProcessor : public DataProcessor {
protected:
    void readData()    override { cout << "  Reading CSV file...\n"; }
    void processData() override { cout << "  Processing CSV rows...\n"; }
    void writeData()   override { cout << "  Writing to SQL DB...\n"; }
    void generateReport() override { cout << "  Generating CSV summary report.\n"; }
};

class XMLProcessor : public DataProcessor {
protected:
    void readData()    override { cout << "  Parsing XML file...\n"; }
    void processData() override { cout << "  Transforming XML nodes...\n"; }
    void writeData()   override { cout << "  Writing to NoSQL DB...\n"; }
    // Uses default report hook
};

int main() {
    cout << "--- CSV Processing ---\n";
    CSVProcessor csv;
    csv.process();

    cout << "\n--- XML Processing ---\n";
    XMLProcessor xml;
    xml.process();

    return 0;
}
```

**Template Method vs Strategy:** Template Method uses *inheritance* (subclass fills in steps). Strategy uses *composition* (inject algorithm object). Template Method controls the skeleton; Strategy swaps the whole algorithm.

---

## 20. Mediator

**Intent:** Define an object that encapsulates how a set of objects interact. Promotes loose coupling by keeping objects from referring to each other explicitly.

**Real-world analogy:** Air traffic control tower — pilots don't talk directly to each other; they go through the tower. The tower (mediator) coordinates all communication.

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Colleague; // Forward declaration

// Mediator interface
class ChatMediator {
public:
    virtual void sendMessage(string msg, Colleague* sender) = 0;
    virtual void addUser(Colleague* user) = 0;
    virtual ~ChatMediator() = default;
};

// Colleague
class Colleague {
protected:
    ChatMediator* mediator;
    string        name;
public:
    Colleague(ChatMediator* m, string n) : mediator(m), name(n) {}
    virtual void send(string msg)    = 0;
    virtual void receive(string msg) = 0;
    string getName() { return name; }
};

// Concrete Colleague
class User : public Colleague {
public:
    User(ChatMediator* m, string n) : Colleague(m, n) {}

    void send(string msg) override {
        cout << name << " sends: " << msg << "\n";
        mediator->sendMessage(msg, this); // Go through mediator
    }

    void receive(string msg) override {
        cout << "  >> " << name << " received: " << msg << "\n";
    }
};

// Concrete Mediator
class ChatRoom : public ChatMediator {
    vector<Colleague*> users;
public:
    void addUser(Colleague* user) override {
        users.push_back(user);
    }

    void sendMessage(string msg, Colleague* sender) override {
        for (auto* user : users) {
            if (user != sender) // Don't send to self
                user->receive(msg);
        }
    }
};

int main() {
    ChatRoom room;

    User raj("room", "Raj"), priya("room", "Priya"), ankit("room", "Ankit");
    // Hack for demo: reassign mediator properly
    User u1(&room, "Raj"), u2(&room, "Priya"), u3(&room, "Ankit");

    room.addUser(&u1);
    room.addUser(&u2);
    room.addUser(&u3);

    u1.send("Hey everyone!");
    u2.send("Hi Raj!");

    return 0;
}
```

---

## 21. Memento

**Intent:** Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later. (Undo mechanism)

```cpp
#include <iostream>
#include <stack>
using namespace std;

// Memento — stores state snapshot
class TextMemento {
    string state;
public:
    TextMemento(string s) : state(s) {}
    string getState() const { return state; }
};

// Originator — creates and restores from mementos
class TextEditor {
    string text;
public:
    void write(string content) {
        text += content;
        cout << "Text: \"" << text << "\"\n";
    }

    TextMemento save() {
        return TextMemento(text); // Snapshot
    }

    void restore(TextMemento m) {
        text = m.getState(); // Restore
        cout << "Restored to: \"" << text << "\"\n";
    }
};

// Caretaker — manages mementos
class UndoManager {
    stack<TextMemento> history;
public:
    void save(TextMemento m)  { history.push(m); }
    TextMemento undo()        {
        auto m = history.top();
        history.pop();
        return m;
    }
    bool canUndo() { return !history.empty(); }
};

int main() {
    TextEditor  editor;
    UndoManager undo;

    undo.save(editor.save()); // Save initial (empty)
    editor.write("Hello");
    undo.save(editor.save());

    editor.write(", World");
    undo.save(editor.save());

    editor.write("! Extra text");

    cout << "\n--- Undoing ---\n";
    editor.restore(undo.undo()); // Back to "Hello, World"
    editor.restore(undo.undo()); // Back to "Hello"
    editor.restore(undo.undo()); // Back to ""

    return 0;
}
```

---

## 22. Visitor

**Intent:** Let you add further operations to objects without modifying them. Separate algorithm from the object structure.

```cpp
#include <iostream>
using namespace std;

class Circle; class Rectangle; // Forward declarations

// Visitor interface — one visit() per element type
class ShapeVisitor {
public:
    virtual void visit(Circle& c)    = 0;
    virtual void visit(Rectangle& r) = 0;
    virtual ~ShapeVisitor() = default;
};

// Element interface
class Shape {
public:
    virtual void accept(ShapeVisitor& v) = 0; // Key: calls visitor's visit()
    virtual ~Shape() = default;
};

// Concrete Elements
class Circle : public Shape {
public:
    int radius;
    Circle(int r) : radius(r) {}
    void accept(ShapeVisitor& v) override { v.visit(*this); }
};

class Rectangle : public Shape {
public:
    int width, height;
    Rectangle(int w, int h) : width(w), height(h) {}
    void accept(ShapeVisitor& v) override { v.visit(*this); }
};

// Concrete Visitors — operations added without modifying shapes
class AreaCalculator : public ShapeVisitor {
public:
    double totalArea = 0;
    void visit(Circle& c)    override { totalArea += 3.14 * c.radius * c.radius; }
    void visit(Rectangle& r) override { totalArea += r.width * r.height; }
};

class XMLExporter : public ShapeVisitor {
public:
    void visit(Circle& c)    override {
        cout << "<circle radius=\"" << c.radius << "\"/>\n";
    }
    void visit(Rectangle& r) override {
        cout << "<rect width=\"" << r.width << "\" height=\"" << r.height << "\"/>\n";
    }
};

int main() {
    vector<Shape*> shapes = {
        new Circle(5), new Rectangle(4, 6), new Circle(3)
    };

    AreaCalculator calc;
    for (auto* s : shapes) s->accept(calc);
    cout << "Total Area: " << calc.totalArea << "\n\n";

    XMLExporter exporter;
    cout << "<shapes>\n";
    for (auto* s : shapes) s->accept(exporter);
    cout << "</shapes>\n";

    for (auto* s : shapes) delete s;
    return 0;
}
```

---

## 23. Interpreter

**Intent:** Define a grammatical representation for a language and provide an interpreter to deal with that grammar.

```cpp
#include <iostream>
#include <map>
using namespace std;

// Abstract Expression
class Expression {
public:
    virtual int interpret(map<string, int>& context) = 0;
    virtual ~Expression() = default;
};

// Terminal Expressions
class NumberExpression : public Expression {
    int number;
public:
    NumberExpression(int n) : number(n) {}
    int interpret(map<string, int>&) override { return number; }
};

class VariableExpression : public Expression {
    string varName;
public:
    VariableExpression(string v) : varName(v) {}
    int interpret(map<string, int>& context) override {
        return context.count(varName) ? context[varName] : 0;
    }
};

// Non-Terminal Expressions
class AddExpression : public Expression {
    Expression *left, *right;
public:
    AddExpression(Expression* l, Expression* r) : left(l), right(r) {}
    int interpret(map<string, int>& ctx) override {
        return left->interpret(ctx) + right->interpret(ctx);
    }
};

class MultiplyExpression : public Expression {
    Expression *left, *right;
public:
    MultiplyExpression(Expression* l, Expression* r) : left(l), right(r) {}
    int interpret(map<string, int>& ctx) override {
        return left->interpret(ctx) * right->interpret(ctx);
    }
};

int main() {
    // Expression: x + (y * 3)
    map<string, int> context = {{"x", 10}, {"y", 5}};

    Expression* expr = new AddExpression(
        new VariableExpression("x"),
        new MultiplyExpression(
            new VariableExpression("y"),
            new NumberExpression(3)
        )
    );

    cout << "x + (y * 3) = " << expr->interpret(context) << "\n"; // 10 + 15 = 25
    return 0;
}
```

---

## 24. Chain of Responsibility — Extended (Middleware)

See section 13 above for the full example with support levels.

---

# Quick Reference Cheat Sheet

## Pattern Selection Guide

| Situation | Pattern to Use |
|---|---|
| Only one instance needed | Singleton |
| Let subclass decide which object to create | Factory Method |
| Create families of related objects | Abstract Factory |
| Complex step-by-step object construction | Builder |
| Copy expensive objects | Prototype |
| Incompatible interfaces need to work together | Adapter |
| Decouple abstraction from implementation | Bridge |
| Tree structures, part-whole hierarchies | Composite |
| Add responsibilities dynamically | Decorator |
| Simple interface to complex subsystem | Facade |
| Huge number of similar fine-grained objects | Flyweight |
| Control/log/cache access to an object | Proxy |
| Pass request through chain of handlers | Chain of Responsibility |
| Encapsulate request as object (undo support) | Command |
| Traverse collection without exposing internals | Iterator |
| Notify many objects when state changes | Observer |
| Many objects communicate through a hub | Mediator |
| Save/restore object state | Memento |
| Swap algorithms at runtime | Strategy |
| Object behavior changes with internal state | State |
| Skeleton algorithm, subclasses fill in steps | Template Method |
| Add operations to objects without modifying them | Visitor |
| Interpret a language's grammar | Interpreter |

---

## SOLID → Pattern Mapping

| SOLID Principle | Patterns That Implement It |
|---|---|
| Single Responsibility | Facade, Command, Iterator |
| Open/Closed | Strategy, Observer, Decorator |
| Liskov Substitution | All patterns using polymorphism |
| Interface Segregation | Adapter, Proxy, Facade |
| Dependency Inversion | Factory Method, Abstract Factory, Builder |

---

## Top 10 Interview Questions & Answers

**Q1: Difference between Factory Method and Abstract Factory?**
A: Factory Method creates ONE product via subclass override. Abstract Factory creates a FAMILY of related products via a factory object. Factory Method uses inheritance; Abstract Factory uses composition.

**Q2: When would you use Builder over a constructor?**
A: When a class has many optional parameters (5+). Builder avoids "telescoping constructors" and improves readability. Also use when the object creation requires multiple steps in a specific order.

**Q3: What's the difference between Decorator and Proxy?**
A: Both wrap an object. Decorator ADDS new behavior/responsibilities. Proxy CONTROLS access (lazy loading, access control, logging). Decorator is about enhancement; Proxy is about control.

**Q4: Adapter vs Facade?**
A: Adapter makes two EXISTING incompatible interfaces work together. Facade creates a NEW simplified interface over a COMPLEX subsystem. Adapter changes the interface; Facade simplifies it.

**Q5: Strategy vs State?**
A: Both use polymorphism and look similar in code. Strategy = client picks algorithm, which doesn't change on its own. State = object transitions between states automatically based on behavior. In State, the state object itself can trigger transitions; in Strategy, the client swaps the strategy.

**Q6: Observer vs Mediator?**
A: Both deal with object communication. Observer is one-to-many (Subject notifies all Observers). Mediator is many-to-many (all objects communicate THROUGH the mediator). Observer is simpler; Mediator avoids "spaghetti" connections between many objects.

**Q7: Why is Singleton considered an anti-pattern?**
A: It creates global state (hidden dependency), makes unit testing hard (can't mock), violates SRP (manages both instance count AND business logic), and causes tight coupling. Use Dependency Injection instead where possible.

**Q8: Thread-safe Singleton in C++?**
A: Use a static local variable (Meyers Singleton). C++11 guarantees static local initialization is thread-safe — no mutex needed for the initialization itself.

**Q9: What is the "double dispatch" problem and which pattern solves it?**
A: C++ virtual dispatch only works on one object. When you need dispatch on TWO objects (e.g., `shape.accept(visitor)`), the Visitor pattern solves it by combining dynamic dispatch (virtual accept) and static dispatch (overloaded visit methods).

**Q10: How does the Bridge pattern differ from Adapter?**
A: Adapter is a retrofit — you have two existing incompatible things and bridge them. Bridge is designed upfront — you intentionally separate abstraction and implementation so they can evolve independently without being glued together.

---

*Guide prepared for interview preparation — covers all 23 Gang of Four patterns + complete SOLID principles with C++ examples.*
