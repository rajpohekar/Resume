# C++ Interview Preparation — Complete Guide
> OOP Concepts · Memory Management · Advanced Topics · Interview Q&A

---

## Table of Contents

1. [OOP — The Four Pillars](#1-oop--the-four-pillars)
   - [Encapsulation](#11-encapsulation)
   - [Abstraction](#12-abstraction)
   - [Inheritance](#13-inheritance)
   - [Polymorphism](#14-polymorphism)
2. [Class & Object](#2-class--object)
3. [Constructors & Destructors](#3-constructors--destructors)
4. [Access Specifiers](#4-access-specifiers)
5. [Inheritance — Deep Dive](#5-inheritance--deep-dive)
6. [Polymorphism — Deep Dive](#6-polymorphism--deep-dive)
7. [Virtual Functions & vtable](#7-virtual-functions--vtable)
8. [Abstract Class & Interface](#8-abstract-class--interface)
9. [Friend Keyword](#9-friend-keyword)
10. [Static Members](#10-static-members)
11. [this Pointer](#11-this-pointer)
12. [Operator Overloading](#12-operator-overloading)
13. [Copy Constructor & Deep vs Shallow Copy](#13-copy-constructor--deep-vs-shallow-copy)
14. [Diamond Problem](#14-diamond-problem)
15. [Memory Management — Complete](#15-memory-management--complete)
    - [Stack vs Heap](#151-stack-vs-heap)
    - [new / delete](#152-new--delete)
    - [Memory Leaks](#153-memory-leaks)
    - [Smart Pointers](#154-smart-pointers)
    - [RAII](#155-raii)
    - [Memory Layout of a Program](#156-memory-layout-of-a-program)
16. [Singleton Design Pattern](#16-singleton-design-pattern)
17. [Dynamic Casting](#17-dynamic-casting)
18. [Exception Handling](#18-exception-handling)
19. [Struct vs Class](#19-struct-vs-class)
20. [STL Essentials](#20-stl-essentials)
21. [Common Interview Mistakes](#21-common-interview-mistakes)
22. [Rapid Revision Sheet](#22-rapid-revision-sheet)

---

## 1. OOP — The Four Pillars

---

### 1.1 Encapsulation

**Definition:**
Bundling data (variables) and the methods that operate on that data within a single unit (class), and **restricting direct external access** to internal data.

**Real-Life Tech Example — Payment Gateway:**
When you integrate Razorpay or Stripe, you only call `processPayment(amount)`. You never directly touch the internal `merchantKey`, `bankRoutingLogic`, or `fraudDetectionScore`. All of that is hidden inside the SDK. You interact only through the exposed API. That's encapsulation.

```cpp
class PaymentGateway {
private:
    string merchantKey;         // hidden — never exposed
    double fraudScore;          // hidden — computed internally
    string bankRoutingLogic;    // hidden — internal implementation

    bool isFraudulent(double amount) {
        return fraudScore > 0.8; // internal logic
    }

public:
    // Only this is exposed to the outside world
    bool processPayment(double amount) {
        if (isFraudulent(amount)) {
            return false;
        }
        // process...
        return true;
    }

    double getTransactionFee(double amount) {
        return amount * 0.02; // 2% fee
    }
};

int main() {
    PaymentGateway pg;
    pg.processPayment(5000);   // ✅ allowed
    // pg.merchantKey = "..."; // ❌ compile error — private
}
```

**What Encapsulation Provides:**
- **Security** — internal state cannot be corrupted from outside
- **Maintainability** — internal implementation can change without breaking callers
- **Controlled Access** — validation logic lives inside setters (e.g., reject negative amounts)
- **Modularity** — each class is a self-contained unit

---

### 1.2 Abstraction

**Definition:**
Hiding the **implementation complexity** and exposing only the **essential interface** the user needs.

> Difference from Encapsulation: Encapsulation hides *data*. Abstraction hides *implementation/complexity*.

**Real-Life Tech Example — ATM Machine:**
When you press "Withdraw ₹5000", you don't need to know:
- Which bank's core banking system is called
- How the network request is authenticated
- How the cash dispenser motor is triggered
- How the transaction journal is written

You only know: insert card → enter PIN → get money. The complexity is abstracted away.

```cpp
// Abstract interface — what the ATM exposes
class ATM {
public:
    virtual bool authenticateUser(string pin) = 0;
    virtual bool withdraw(double amount) = 0;
    virtual double checkBalance() = 0;
    virtual ~ATM() {}
};

// Concrete implementation — hidden from the user
class ICICIAtm : public ATM {
private:
    // All this complexity is hidden
    void connectToCoreBanking() { /* TCP call to banking server */ }
    void logAuditTrail() { /* Write to transaction journal */ }
    void triggerCashDispenser(double amount) { /* Motor control */ }
    double balance = 10000.0;

public:
    bool authenticateUser(string pin) override {
        connectToCoreBanking();
        return pin == "1234"; // simplified
    }

    bool withdraw(double amount) override {
        if (amount <= balance) {
            triggerCashDispenser(amount);
            logAuditTrail();
            balance -= amount;
            return true;
        }
        return false;
    }

    double checkBalance() override {
        return balance;
    }
};

int main() {
    ATM* myAtm = new ICICIAtm();
    myAtm->authenticateUser("1234");
    myAtm->withdraw(2000); // user doesn't know HOW this works
    delete myAtm;
}
```

**What Abstraction Provides:**
- **Reduced Complexity** — user sees only what they need
- **Loose Coupling** — callers depend on the interface, not the implementation
- **Interchangeability** — swap `ICICIAtm` with `HDFCAtm` without changing user code
- **Focus** — developer focuses on "what" not "how"

---

### 1.3 Inheritance

**Definition:**
A child class (subclass) acquires the **properties and behaviors** of a parent class (superclass), enabling code reuse and hierarchical classification.

**Real-Life Tech Example — User Roles in a SaaS App:**
In a ticket management system (like Jira/Freshdesk), every user has a name, email, and login ability. But an `Admin` can also delete tickets and manage teams. An `Agent` can resolve and assign. A `Customer` can only create tickets. Instead of rewriting common fields for each role, you inherit from a base `User` class.

```cpp
class User {
protected:
    string name;
    string email;
    string passwordHash;

public:
    User(string n, string e) : name(n), email(e) {}

    bool login(string password) {
        // hash and verify
        return true;
    }

    string getName() { return name; }
    string getEmail() { return email; }
};

class Admin : public User {
public:
    Admin(string n, string e) : User(n, e) {}

    void deleteTicket(int ticketId) {
        cout << "Admin " << name << " deleted ticket " << ticketId << endl;
    }

    void manageTeam() {
        cout << "Managing team..." << endl;
    }
};

class Agent : public User {
public:
    Agent(string n, string e) : User(n, e) {}

    void resolveTicket(int ticketId) {
        cout << "Agent " << name << " resolved ticket " << ticketId << endl;
    }
};

class Customer : public User {
public:
    Customer(string n, string e) : User(n, e) {}

    void createTicket(string issue) {
        cout << "Ticket created: " << issue << endl;
    }
};

int main() {
    Admin admin("Rohit", "rohit@company.com");
    admin.login("pass123");         // inherited from User
    admin.deleteTicket(42);         // Admin's own method

    Customer c("Priya", "priya@gmail.com");
    c.login("pass456");             // inherited
    c.createTicket("Login not working");
}
```

**Types of Inheritance in C++:**

| Type | Structure | Example |
|---|---|---|
| Single | A → B | User → Admin |
| Multilevel | A → B → C | User → Employee → Manager |
| Hierarchical | A → B, A → C | User → Admin, User → Agent |
| Multiple | A + B → C | Flyable + Swimmable → Duck |
| Hybrid | Combination of above | Mix of multiple and hierarchical |

**What Inheritance Provides:**
- **Code Reuse** — write common logic once in parent
- **Extensibility** — extend without modifying existing code (Open/Closed Principle)
- **Hierarchy** — models real-world relationships naturally
- **Polymorphism** — foundation for runtime polymorphism

---

### 1.4 Polymorphism

**Definition:**
"One interface, many forms." The same method call **behaves differently** depending on the actual object it is invoked on.

**Real-Life Tech Example — Notification Service:**
A notification system needs to send alerts. Whether it sends via Email, SMS, or Push Notification — the calling code just calls `send()`. The system figures out HOW based on the actual object.

```cpp
// Base class — the interface
class NotificationService {
public:
    virtual void send(string message) = 0;  // pure virtual
    virtual ~NotificationService() {}
};

// Different behaviors for the same interface
class EmailNotification : public NotificationService {
public:
    void send(string message) override {
        cout << "Sending EMAIL: " << message << endl;
        // SMTP logic here
    }
};

class SMSNotification : public NotificationService {
public:
    void send(string message) override {
        cout << "Sending SMS via Twilio: " << message << endl;
        // Twilio API call here
    }
};

class PushNotification : public NotificationService {
public:
    void send(string message) override {
        cout << "Sending PUSH via Firebase: " << message << endl;
        // FCM call here
    }
};

// Caller doesn't care which channel — just calls send()
void alertUser(NotificationService* service, string msg) {
    service->send(msg);  // runtime polymorphism — decided at runtime
}

int main() {
    NotificationService* email = new EmailNotification();
    NotificationService* sms   = new SMSNotification();
    NotificationService* push  = new PushNotification();

    alertUser(email, "Your OTP is 4521");   // Sending EMAIL...
    alertUser(sms,   "Your OTP is 4521");   // Sending SMS...
    alertUser(push,  "Your OTP is 4521");   // Sending PUSH...

    delete email;
    delete sms;
    delete push;
}
```

**Two Types:**

#### Compile-Time Polymorphism (Method Overloading / Operator Overloading)
Resolved at **compile time**. Same name, different signatures.

```cpp
class Logger {
public:
    void log(string msg) {
        cout << "[LOG] " << msg << endl;
    }

    void log(string msg, int errorCode) {
        cout << "[ERROR " << errorCode << "] " << msg << endl;
    }

    void log(string msg, string level) {
        cout << "[" << level << "] " << msg << endl;
    }
};

// All three are different functions resolved at compile time
Logger l;
l.log("Server started");
l.log("DB connection failed", 500);
l.log("Cache miss", "WARN");
```

#### Runtime Polymorphism (Virtual Function Overriding)
Resolved at **runtime** via vtable. Requires `virtual` keyword and base class pointer.

```cpp
// (See NotificationService example above)
// Key: Base* ptr = new Derived(); ptr->method(); — calls Derived's method
```

**What Polymorphism Provides:**
- **Flexibility** — add new types without modifying existing code
- **Extensibility** — new `WhatsAppNotification` class works with existing `alertUser()` function
- **Clean Code** — eliminates chains of `if-else` or `switch` based on type

---

## 2. Class & Object

| Concept | Class | Object |
|---|---|---|
| Definition | Blueprint / Template | Instance of a class |
| Memory | No memory allocated | Memory allocated |
| Existence | Logical | Physical |
| Example | `class User {}` | `User u1;` |

```cpp
class DatabaseConnection {
public:
    string host;
    int port;
    string dbName;

    void connect() {
        cout << "Connecting to " << host << ":" << port << "/" << dbName << endl;
    }
};

int main() {
    DatabaseConnection prod;   // object 1
    prod.host = "prod-db.internal";
    prod.port = 5432;
    prod.dbName = "app_db";
    prod.connect();

    DatabaseConnection staging; // object 2 — separate memory
    staging.host = "staging-db.internal";
    staging.port = 5432;
    staging.dbName = "app_db_staging";
    staging.connect();
}
```

---

## 3. Constructors & Destructors

### What is a Constructor?
A special member function automatically called when an object is created. Same name as class, no return type.

### Types of Constructors

#### 1. Default Constructor
```cpp
class Server {
public:
    string status;

    Server() {
        status = "STOPPED";
        cout << "Server initialized" << endl;
    }
};

Server s; // Default constructor called automatically
```

#### 2. Parameterized Constructor
```cpp
class Server {
public:
    string host;
    int port;

    Server(string h, int p) {
        host = h;
        port = p;
    }
};

Server s("localhost", 8080);
```

#### 3. Copy Constructor
```cpp
class Config {
public:
    string apiKey;
    int timeout;

    Config(string key, int t) : apiKey(key), timeout(t) {}

    // Copy constructor
    Config(const Config& other) {
        apiKey  = other.apiKey;
        timeout = other.timeout;
        cout << "Config copied" << endl;
    }
};

Config original("abc123", 30);
Config backup = original;  // copy constructor called
```

#### 4. Constructor Initializer List (preferred)
```cpp
class Connection {
private:
    const string host;  // const — MUST be initialized via initializer list
    int port;

public:
    Connection(string h, int p) : host(h), port(p) {}
    // More efficient — initializes directly instead of default-init then assign
};
```

### Destructor
Automatically called when object goes out of scope or is deleted. Used to release resources.

```cpp
class DatabaseConnection {
private:
    // Simulate a DB connection handle
    bool connected = false;

public:
    DatabaseConnection() {
        connected = true;
        cout << "DB Connected" << endl;
    }

    ~DatabaseConnection() {
        if (connected) {
            connected = false;
            cout << "DB Connection closed — cleanup done" << endl;
        }
    }
};

// When function ends, destructor is automatically called
void runQuery() {
    DatabaseConnection db;  // constructor called
    // ... do work ...
}   // destructor called here — connection safely closed
```

**Key Rules:**
- Only one destructor per class
- No parameters, no return type
- Called in reverse order of construction
- Should be `virtual` in base classes (see Section 7)

---

## 4. Access Specifiers

| Specifier | Same Class | Derived Class | Outside Class |
|---|---|---|---|
| `private` | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ |

```cpp
class APIClient {
private:
    string secretKey = "sk-xyz";     // only this class

protected:
    string baseUrl = "https://api.example.com"; // this class + derived

public:
    void makeRequest(string endpoint) {  // anyone can call
        // internally uses secretKey and baseUrl
        cout << "GET " << baseUrl << endpoint << endl;
    }
};

class AuthenticatedClient : public APIClient {
public:
    void fetchUser(int id) {
        // baseUrl accessible (protected) ✅
        cout << "Fetching user from " << baseUrl << "/users/" << id << endl;
        // secretKey NOT accessible (private) ❌
    }
};
```

---

## 5. Inheritance — Deep Dive

### Single Inheritance
```cpp
class Vehicle {
public:
    int speed;
    void start() { cout << "Vehicle started" << endl; }
};

class Car : public Vehicle {
public:
    int doors;
    void honk() { cout << "Beep!" << endl; }
};
```

### Multiple Inheritance
```cpp
class Serializable {
public:
    virtual string toJSON() = 0;
};

class Cacheable {
public:
    virtual void cache() = 0;
};

// User can be serialized AND cached
class User : public Serializable, public Cacheable {
public:
    string name;

    string toJSON() override {
        return "{\"name\":\"" + name + "\"}";
    }

    void cache() override {
        cout << "User cached in Redis" << endl;
    }
};
```

### Multilevel Inheritance
```cpp
class Entity {
public:
    int id;
    void save() { cout << "Saving to DB..." << endl; }
};

class User : public Entity {
public:
    string email;
};

class PremiumUser : public User {
public:
    string plan;
    void accessPremiumFeature() {
        cout << "Premium access granted for user " << id << endl;
    }
};
```

### Inheritance Access Table

| Base Member | `public` inheritance | `protected` inheritance | `private` inheritance |
|---|---|---|---|
| `public` | public | protected | private |
| `protected` | protected | protected | private |
| `private` | inaccessible | inaccessible | inaccessible |

---

## 6. Polymorphism — Deep Dive

### Overloading vs Overriding

| Feature | Overloading | Overriding |
|---|---|---|
| Where | Same class | Parent & Child class |
| When resolved | Compile time | Runtime |
| Parameters | Different | Same |
| `virtual` needed | No | Yes |
| Return type | Can differ | Must be same (or covariant) |

### Object Slicing (Interview Trap)
```cpp
class Base {
public:
    int x = 10;
    virtual void show() { cout << "Base" << endl; }
};

class Derived : public Base {
public:
    int y = 20;  // extra field
    void show() override { cout << "Derived, y=" << y << endl; }
};

Derived d;
Base b = d;   // ⚠️ OBJECT SLICING — y is lost!
b.show();     // prints "Base" — not "Derived"

Base* ptr = &d;
ptr->show();  // prints "Derived" ✅ — pointer/reference avoids slicing
```

**Rule:** Always use pointer or reference for polymorphism. Never copy by value.

---

## 7. Virtual Functions & vtable

### Why Virtual?
Without `virtual`, base class pointer always calls **base class method** regardless of the actual object type.

```cpp
class Shape {
public:
    // Without virtual: always calls Shape::area()
    // With virtual: calls the actual derived class's area()
    virtual double area() = 0;

    void printArea() {
        cout << "Area = " << area() << endl; // uses virtual dispatch
    }
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() override { return 3.14 * r * r; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h) : w(w), h(h) {}
    double area() override { return w * h; }
};

int main() {
    Shape* s1 = new Circle(5);
    Shape* s2 = new Rectangle(4, 6);

    s1->printArea(); // Area = 78.5
    s2->printArea(); // Area = 24

    delete s1;
    delete s2;
}
```

### How vtable Works (Internals)

```
Each class with virtual functions has a vtable (virtual function table).
Each object has a hidden vptr (virtual pointer) pointing to its class's vtable.

Circle's vtable:
  [0] → Circle::area()

Rectangle's vtable:
  [0] → Rectangle::area()

When ptr->area() is called:
  1. Follow ptr to the object
  2. Follow vptr to the vtable
  3. Look up area() in vtable
  4. Call that function
```

### Virtual Destructor — CRITICAL

```cpp
class Base {
public:
    Base() { cout << "Base created" << endl; }
    virtual ~Base() { cout << "Base destroyed" << endl; }  // MUST be virtual
};

class Derived : public Base {
    int* data;
public:
    Derived() {
        data = new int[1000];
        cout << "Derived created" << endl;
    }
    ~Derived() {
        delete[] data;  // MEMORY LEAK if this is never called!
        cout << "Derived destroyed" << endl;
    }
};

Base* ptr = new Derived();
delete ptr;
// WITHOUT virtual ~Base(): only Base destructor called → data leaks!
// WITH    virtual ~Base(): Derived destructor → Base destructor → safe ✅
```

**Rule:** If a class has virtual functions, its destructor MUST be virtual.

---

## 8. Abstract Class & Interface

### Abstract Class
A class with **at least one pure virtual function** (`= 0`). Cannot be instantiated.

```cpp
class PaymentProcessor {
public:
    virtual bool charge(double amount) = 0;    // pure virtual — must override
    virtual bool refund(double amount) = 0;    // pure virtual — must override

    // Can have concrete methods too
    void printReceipt(double amount) {
        cout << "Payment of Rs." << amount << " processed." << endl;
    }

    virtual ~PaymentProcessor() {}
};

class RazorpayProcessor : public PaymentProcessor {
public:
    bool charge(double amount) override {
        cout << "Charging Rs." << amount << " via Razorpay" << endl;
        return true;
    }

    bool refund(double amount) override {
        cout << "Refunding Rs." << amount << " via Razorpay" << endl;
        return true;
    }
};

// PaymentProcessor pp;  // ❌ Error — cannot instantiate abstract class
PaymentProcessor* pp = new RazorpayProcessor(); // ✅ pointer is fine
pp->charge(999.0);
pp->printReceipt(999.0);
```

### Interface in C++
C++ has no `interface` keyword. Simulate it with a **pure abstract class** (all methods = 0, no data).

```cpp
class ILogger {
public:
    virtual void info(string msg) = 0;
    virtual void error(string msg) = 0;
    virtual void warn(string msg) = 0;
    virtual ~ILogger() {}
};

class ConsoleLogger : public ILogger {
public:
    void info(string msg)  override { cout << "[INFO] "  << msg << endl; }
    void error(string msg) override { cout << "[ERROR] " << msg << endl; }
    void warn(string msg)  override { cout << "[WARN] "  << msg << endl; }
};

class FileLogger : public ILogger {
public:
    void info(string msg)  override { /* write to file */ }
    void error(string msg) override { /* write to file */ }
    void warn(string msg)  override { /* write to file */ }
};
```

### Abstract Class vs Interface

| Feature | Abstract Class | Interface (Pure Abstract) |
|---|---|---|
| Pure virtual methods | At least one | All |
| Concrete methods | Yes | No (in strict interface) |
| Data members | Yes | No |
| Constructor | Yes | No (convention) |
| Use when | Shared code + contract | Contract only |

---

## 9. Friend Keyword

### Friend Function
Can access private/protected members of a class. Declared inside the class but defined outside.

```cpp
class BankAccount {
private:
    double balance;
    string accountNo;

public:
    BankAccount(double b, string a) : balance(b), accountNo(a) {}

    // Grant access to audit function
    friend void auditAccount(BankAccount acc);
};

void auditAccount(BankAccount acc) {
    // Can access private members
    cout << "Auditing account: " << acc.accountNo
         << ", Balance: " << acc.balance << endl;
}
```

### Friend Class
```cpp
class Server;  // forward declaration

class Monitor {
public:
    void showStats(Server& s);  // needs access to Server's private data
};

class Server {
private:
    int cpuUsage = 85;
    int memoryUsage = 60;

    friend class Monitor;  // Monitor can access all private members
};

void Monitor::showStats(Server& s) {
    cout << "CPU: " << s.cpuUsage << "%, RAM: " << s.memoryUsage << "%" << endl;
}
```

**Note:** Friendship is not inherited. Not transitive. Use sparingly — it breaks encapsulation.

---

## 10. Static Members

### Static Variable
Shared across **all objects** of the class. Only one copy exists.

```cpp
class DatabaseConnectionPool {
private:
    static int activeConnections;  // shared across all instances

public:
    DatabaseConnectionPool() {
        activeConnections++;
        cout << "Connection opened. Active: " << activeConnections << endl;
    }

    ~DatabaseConnectionPool() {
        activeConnections--;
        cout << "Connection closed. Active: " << activeConnections << endl;
    }

    static int getActiveConnections() {
        return activeConnections;
    }
};

// Must define static member outside class
int DatabaseConnectionPool::activeConnections = 0;

int main() {
    DatabaseConnectionPool c1, c2, c3;
    cout << "Total: " << DatabaseConnectionPool::getActiveConnections() << endl; // 3
}
```

### Static Function
- Can only access static members
- Called on class, not object
- No `this` pointer

```cpp
class MathUtils {
public:
    static int clamp(int val, int min, int max) {
        if (val < min) return min;
        if (val > max) return max;
        return val;
    }

    static double percentOf(double part, double total) {
        return (part / total) * 100.0;
    }
};

int clamped = MathUtils::clamp(150, 0, 100);     // 100
double pct  = MathUtils::percentOf(35, 200);     // 17.5
```

---

## 11. this Pointer

`this` is a hidden pointer available in every non-static member function. It holds the address of the current object.

```cpp
class Builder {
private:
    string host;
    int port;
    string dbName;

public:
    // Method chaining using this pointer
    Builder& setHost(string h) {
        this->host = h;
        return *this;  // return current object
    }

    Builder& setPort(int p) {
        this->port = p;
        return *this;
    }

    Builder& setDatabase(string db) {
        this->dbName = db;
        return *this;
    }

    void build() {
        cout << "Connecting to " << host << ":" << port << "/" << dbName << endl;
    }
};

int main() {
    Builder b;
    b.setHost("localhost")
     .setPort(5432)
     .setDatabase("myapp_db")
     .build();  // method chaining via this
}
```

---

## 12. Operator Overloading

Define how operators (`+`, `-`, `==`, etc.) work on user-defined types.

```cpp
class Money {
private:
    double amount;
    string currency;

public:
    Money(double a, string c) : amount(a), currency(c) {}

    // Overload +
    Money operator+(const Money& other) {
        if (currency != other.currency) throw runtime_error("Currency mismatch");
        return Money(amount + other.amount, currency);
    }

    // Overload ==
    bool operator==(const Money& other) {
        return amount == other.amount && currency == other.currency;
    }

    // Overload < for sorting
    bool operator<(const Money& other) {
        return amount < other.amount;
    }

    // Overload << for printing
    friend ostream& operator<<(ostream& os, const Money& m) {
        os << m.currency << " " << m.amount;
        return os;
    }
};

int main() {
    Money price(999.0, "INR");
    Money tax(179.82, "INR");
    Money total = price + tax;   // uses operator+
    cout << total << endl;       // INR 1178.82
}
```

---

## 13. Copy Constructor & Deep vs Shallow Copy

### Shallow Copy (default behavior)
Copies pointer *address* — both objects point to same memory. **Dangerous.**

```cpp
class Cache {
public:
    int* data;
    int size;

    Cache(int s) : size(s) {
        data = new int[size];
    }

    // Default copy constructor does SHALLOW copy:
    // newObj.data = original.data  ← SAME pointer!
};

Cache c1(10);
Cache c2 = c1;   // shallow copy — c2.data == c1.data (same address!)
delete[] c1.data; // c2.data now points to freed memory → undefined behavior!
```

### Deep Copy (safe)
Allocates **separate memory** and copies contents.

```cpp
class Cache {
public:
    int* data;
    int size;

    Cache(int s) : size(s) {
        data = new int[size];
        fill(data, data + size, 0);
    }

    // Deep copy constructor
    Cache(const Cache& other) : size(other.size) {
        data = new int[size];          // NEW allocation
        copy(other.data, other.data + size, data);  // copy values
        cout << "Deep copy performed" << endl;
    }

    // Deep copy assignment operator
    Cache& operator=(const Cache& other) {
        if (this == &other) return *this;  // self-assignment check
        delete[] data;                     // free old memory
        size = other.size;
        data = new int[size];
        copy(other.data, other.data + size, data);
        return *this;
    }

    ~Cache() {
        delete[] data;
    }
};
```

**Rule of Three:** If you define any of — destructor, copy constructor, copy assignment — define all three.
**Rule of Five (C++11):** Also define move constructor and move assignment.

### Copy Constructor vs Assignment Operator

| | Copy Constructor | Assignment Operator |
|---|---|---|
| When called | During initialization | After object exists |
| Example | `Config c2 = c1;` | `c3 = c1;` |
| `this` valid | Not yet | Yes (object exists) |
| Self-assign check | Not needed | Always check `if (this == &other)` |

---

## 14. Diamond Problem

```
        Person
       /      \
  Employee  Taxpayer
       \      /
       Contractor
```

`Contractor` gets **two copies** of `Person`. Ambiguous. Solved with **virtual inheritance**.

```cpp
class Person {
public:
    string name;
    Person(string n) : name(n) {}
    void showName() { cout << "Name: " << name << endl; }
};

class Employee : virtual public Person {
public:
    int empId;
    Employee(string n, int id) : Person(n), empId(id) {}
};

class Taxpayer : virtual public Person {
public:
    string panNumber;
    Taxpayer(string n, string pan) : Person(n), panNumber(pan) {}
};

// Only ONE copy of Person thanks to virtual inheritance
class Contractor : public Employee, public Taxpayer {
public:
    Contractor(string n, int id, string pan)
        : Person(n), Employee(n, id), Taxpayer(n, pan) {}
};

int main() {
    Contractor c("Amit", 101, "ABCDE1234F");
    c.showName();  // No ambiguity — single Person base ✅
}
```

---

## 15. Memory Management — Complete

---

### 15.1 Stack vs Heap

| Feature | Stack | Heap |
|---|---|---|
| Allocation | Automatic (at function entry) | Manual (`new`) |
| Deallocation | Automatic (at function exit) | Manual (`delete`) |
| Size | Small (~1-8 MB typically) | Large (limited by RAM) |
| Speed | Very fast (just move stack pointer) | Slower (OS allocates) |
| Access | LIFO | Random |
| Thread safety | Each thread has its own stack | Shared — needs synchronization |
| Risk | Stack overflow | Memory leak |

```cpp
void demonstrate() {
    // Stack allocation — automatic
    int x = 10;             // stack — freed when function returns
    char buf[256];          // stack — 256 bytes reserved on stack

    // Heap allocation — manual
    int* y = new int(20);   // heap — stays until delete
    int* arr = new int[100]; // heap — 400 bytes on heap

    // Must manually free heap memory
    delete y;
    delete[] arr;
    // x and buf freed automatically
}
```

### 15.2 new / delete

```cpp
// Allocate single object
int* p = new int(42);      // allocates int, initializes to 42
delete p;                  // free single object

// Allocate array
int* arr = new int[10];    // allocates 10 ints
delete[] arr;              // MUST use delete[] for arrays (not delete)

// Allocate object
class Node {
public:
    int val;
    Node(int v) : val(v) {}
};

Node* n = new Node(5);
delete n;  // calls ~Node() then frees memory

// Allocating with exception safety
try {
    int* p = new int[1000000000]; // throws std::bad_alloc if not enough memory
} catch (bad_alloc& e) {
    cout << "Allocation failed: " << e.what() << endl;
}

// nothrow version
int* p = new (nothrow) int[1000000000];
if (!p) {
    cout << "Allocation failed" << endl;
}
```

### malloc vs new

| | `malloc` | `new` |
|---|---|---|
| Language | C | C++ |
| Returns | `void*` (must cast) | Correct type |
| Constructor | NOT called | Called |
| Failure | Returns `NULL` | Throws `bad_alloc` |
| Sizing | You specify bytes | Compiler calculates |
| Pair | `free()` | `delete` |

```cpp
// malloc — C style
int* p1 = (int*)malloc(sizeof(int));  // manual cast required
if (p1) { *p1 = 10; }
free(p1);   // not delete!

// new — C++ style
int* p2 = new int(10);   // cleaner
delete p2;
```

### 15.3 Memory Leaks

A memory leak occurs when heap memory is allocated but never freed.

```cpp
// LEAK — classic mistake
void processRequest() {
    int* buffer = new int[1000];
    // ... do work ...
    if (someError) return;  // ⚠️ LEAK — delete[] never reached
    delete[] buffer;
}

// LEAK — exception path
void riskyFunction() {
    int* data = new int[100];
    mightThrow();   // if throws, delete never called ⚠️
    delete[] data;
}

// Detecting leaks:
// Valgrind: valgrind --leak-check=full ./your_program
// AddressSanitizer: compile with -fsanitize=address
```

### 15.4 Smart Pointers

Smart pointers automatically manage memory. Defined in `<memory>`. **Prefer over raw pointers.**

#### unique_ptr — Exclusive Ownership

```cpp
#include <memory>

// Only ONE owner at a time
unique_ptr<int> p1 = make_unique<int>(42);
cout << *p1 << endl;   // 42

// Transfer ownership
unique_ptr<int> p2 = move(p1);  // p1 is now null
// p1.get() == nullptr

// Auto-freed when p2 goes out of scope — no delete needed
```

```cpp
// Real-world: DB connection that only one component owns
unique_ptr<DatabaseConnection> db = make_unique<DatabaseConnection>("localhost");
db->query("SELECT * FROM users");
// Automatically closed when db goes out of scope
```

#### shared_ptr — Shared Ownership (Reference Counted)

```cpp
#include <memory>

shared_ptr<int> p1 = make_shared<int>(100);
cout << p1.use_count() << endl;  // 1

{
    shared_ptr<int> p2 = p1;    // both own the same object
    cout << p1.use_count() << endl;  // 2
    cout << p2.use_count() << endl;  // 2
}   // p2 destroyed — count drops to 1

// Object freed only when count reaches 0
cout << p1.use_count() << endl;  // 1
```

```cpp
// Real-world: shared config loaded once, used by multiple services
shared_ptr<AppConfig> config = make_shared<AppConfig>("config.json");

AuthService auth(config);     // both hold shared_ptr
DatabaseService db(config);   // config freed when both are done
```

#### weak_ptr — Non-Owning Reference (Breaks Circular References)

```cpp
#include <memory>

// Circular reference problem — MEMORY LEAK:
struct Node {
    shared_ptr<Node> next;  // strong reference — prevents deallocation
    // If two nodes point to each other, neither is ever freed!
};

// Fix with weak_ptr:
struct TreeNode {
    shared_ptr<TreeNode> leftChild;
    shared_ptr<TreeNode> rightChild;
    weak_ptr<TreeNode> parent;  // weak — doesn't prevent deallocation

    void doSomethingWithParent() {
        // Must lock() to use — may be null if parent was freed
        if (auto p = parent.lock()) {
            cout << "Parent exists" << endl;
        }
    }
};
```

#### Smart Pointer Comparison

| | `unique_ptr` | `shared_ptr` | `weak_ptr` |
|---|---|---|---|
| Ownership | Exclusive | Shared (ref-counted) | None (observer) |
| Copy | ❌ (move only) | ✅ | ✅ |
| Overhead | Minimal | Ref count overhead | Minimal |
| Use when | One clear owner | Multiple owners | Break cycles |
| Auto-free | When owner destroyed | When count = 0 | Never (no ownership) |

### 15.5 RAII

**Resource Acquisition Is Initialization** — acquire resources in constructor, release in destructor.

```cpp
// RAII wrapper for file handle
class FileHandle {
private:
    FILE* file;

public:
    FileHandle(const string& path, const string& mode) {
        file = fopen(path.c_str(), mode.c_str());
        if (!file) throw runtime_error("Cannot open file: " + path);
        cout << "File opened" << endl;
    }

    ~FileHandle() {
        if (file) {
            fclose(file);
            cout << "File closed automatically" << endl;
        }
    }

    FILE* get() { return file; }

    // Prevent copying
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
};

void processFile() {
    FileHandle f("data.txt", "r");  // opened
    // ... read data ...
    // f automatically closed when function exits — even on exception!
}
```

**RAII Benefits:**
- No resource leaks even when exceptions occur
- No need to remember to call `close()`, `free()`, etc.
- Cleanup is deterministic and tied to scope

### 15.6 Memory Layout of a Program

```
High Address
┌─────────────────────────────┐
│        Command Line Args    │ (argv, envp)
├─────────────────────────────┤
│           Stack             │ ← grows downward
│  (local vars, return addr,  │
│   function call frames)     │
├─────────────────────────────┤
│             ↓               │
│          (gap)              │
│             ↑               │
├─────────────────────────────┤
│           Heap              │ ← grows upward (new/malloc)
├─────────────────────────────┤
│    BSS Segment              │ (uninitialized global/static vars)
├─────────────────────────────┤
│    Data Segment             │ (initialized global/static vars)
├─────────────────────────────┤
│    Text Segment (Code)      │ (executable instructions — read-only)
└─────────────────────────────┘
Low Address
```

```cpp
int globalInit = 10;        // Data segment
int globalUninit;           // BSS segment

int main() {
    int localVar = 20;      // Stack
    static int s = 30;      // Data segment

    int* heapVar = new int(40); // Heap
    delete heapVar;
}
```

---

## 16. Singleton Design Pattern

Ensures only **one instance** of a class exists globally.

```cpp
class ConfigManager {
private:
    string dbUrl;
    string apiKey;
    int maxConnections;

    // Private constructor — cannot call from outside
    ConfigManager() {
        dbUrl = "postgresql://localhost:5432/prod";
        apiKey = "sk-prod-abc123";
        maxConnections = 50;
        cout << "Config loaded" << endl;
    }

public:
    // Delete copy and move to prevent duplication
    ConfigManager(const ConfigManager&) = delete;
    ConfigManager& operator=(const ConfigManager&) = delete;

    static ConfigManager& getInstance() {
        static ConfigManager instance;  // created once, on first call
        return instance;
    }

    string getDbUrl() { return dbUrl; }
    string getApiKey() { return apiKey; }
    int getMaxConnections() { return maxConnections; }
};

int main() {
    ConfigManager& cfg1 = ConfigManager::getInstance();
    ConfigManager& cfg2 = ConfigManager::getInstance();

    // Both refer to the same object
    cout << (&cfg1 == &cfg2) << endl;  // 1 (true)
    cout << cfg1.getDbUrl() << endl;
}
```

---

## 17. Dynamic Casting

Safe downcast in polymorphic hierarchies. Returns `nullptr` if the cast is invalid.

```cpp
class Shape {
public:
    virtual void draw() = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
public:
    void draw() override { cout << "Drawing circle" << endl; }
    void setRadius(double r) { cout << "Radius set to " << r << endl; }
};

class Rectangle : public Shape {
public:
    void draw() override { cout << "Drawing rectangle" << endl; }
    void setDimensions(double w, double h) { cout << w << "x" << h << endl; }
};

void process(Shape* s) {
    s->draw();

    // Try to treat it as a Circle
    Circle* c = dynamic_cast<Circle*>(s);
    if (c) {
        c->setRadius(5.0);  // safe — it IS a Circle
    } else {
        cout << "Not a circle" << endl;
    }
}

int main() {
    Shape* s1 = new Circle();
    Shape* s2 = new Rectangle();

    process(s1);  // draws circle, sets radius
    process(s2);  // draws rectangle, prints "Not a circle"

    delete s1;
    delete s2;
}
```

**static_cast vs dynamic_cast:**
- `static_cast` — compile-time, no safety check, faster
- `dynamic_cast` — runtime check, requires RTTI, returns nullptr on failure, safe

---

## 18. Exception Handling

```cpp
#include <stdexcept>

class InsufficientFundsException : public exception {
private:
    double requested;
    double available;

public:
    InsufficientFundsException(double r, double a)
        : requested(r), available(a) {}

    const char* what() const noexcept override {
        return "Insufficient funds in account";
    }

    double getShortfall() const { return requested - available; }
};

class BankAccount {
    double balance;

public:
    BankAccount(double b) : balance(b) {}

    void withdraw(double amount) {
        if (amount > balance) {
            throw InsufficientFundsException(amount, balance);
        }
        balance -= amount;
    }
};

int main() {
    BankAccount acc(5000.0);

    try {
        acc.withdraw(3000.0);    // OK
        acc.withdraw(4000.0);    // throws!
    }
    catch (const InsufficientFundsException& e) {
        cout << "Error: " << e.what() << endl;
        cout << "Shortfall: " << e.getShortfall() << endl;
    }
    catch (const exception& e) {
        cout << "General error: " << e.what() << endl;
    }
    catch (...) {
        cout << "Unknown error" << endl;
    }
}
```

---

## 19. Struct vs Class

| Feature | struct | class |
|---|---|---|
| Default access | `public` | `private` |
| Inheritance default | `public` | `private` |
| Typical use | POD data grouping | Full OOP with behavior |

```cpp
// Struct — use for simple data
struct Point {
    double x;   // public by default
    double y;
};

// Class — use when behavior + data encapsulation needed
class Vector2D {
private:
    double x, y;
public:
    Vector2D(double x, double y) : x(x), y(y) {}
    double magnitude() { return sqrt(x*x + y*y); }
};
```

---

## 20. STL Essentials

```cpp
#include <vector>
#include <map>
#include <unordered_map>
#include <set>
#include <queue>
#include <stack>
#include <algorithm>

// vector — dynamic array O(1) access
vector<int> v = {1, 2, 3};
v.push_back(4);
v.erase(v.begin() + 1);  // remove index 1

// map — sorted key-value O(log n)
map<string, int> wordCount;
wordCount["hello"] = 5;
wordCount["world"]++;

// unordered_map — hash table O(1) average
unordered_map<int, string> userMap;
userMap[101] = "Alice";
userMap[102] = "Bob";

// set — unique sorted elements
set<int> primes = {2, 3, 5, 7, 11};
primes.insert(13);  // auto-sorted

// priority_queue — max-heap by default
priority_queue<int> pq;
pq.push(5); pq.push(1); pq.push(10);
cout << pq.top() << endl;  // 10

// Algorithms
sort(v.begin(), v.end());
auto it = find(v.begin(), v.end(), 3);
int cnt = count(v.begin(), v.end(), 3);
```

---

## 21. Common Interview Mistakes

| Mistake | Correct Approach |
|---|---|
| Confusing overloading and overriding | Overloading = same class, different params. Overriding = parent-child, same signature |
| Forgetting `virtual` destructor | Any class with virtual methods must have `virtual ~Class()` |
| Using `delete` instead of `delete[]` | Arrays always need `delete[]` |
| Shallow copy when deep copy needed | Implement copy constructor + assignment operator when class has pointers |
| Only giving definitions, no examples | Always follow: Definition → Example → Use case |
| Not checking `nullptr` after `dynamic_cast` | Always `if (ptr = dynamic_cast<T*>(base)) { ... }` |
| Circular `shared_ptr` causing leaks | Use `weak_ptr` to break cycles |
| Returning address of local variable | Local variables are destroyed; returned pointer is dangling |

---

## 22. Rapid Revision Sheet

### OOP Pillars in 1 Line Each
- **Encapsulation** — Hide data, expose interface. Provides security & maintainability.
- **Abstraction** — Hide implementation, show only what user needs. Reduces complexity.
- **Inheritance** — Child acquires parent's properties. Provides code reuse & hierarchy.
- **Polymorphism** — Same interface, different behavior. Provides flexibility & extensibility.

### Key Keywords
```
virtual      → enables runtime polymorphism
override     → explicitly marks overriding (compile-time safety)
final        → prevents further overriding/inheritance
= 0          → pure virtual → abstract class
friend       → grants access to private members (use sparingly)
static       → belongs to class, not object
this         → pointer to current object
explicit     → prevents implicit conversion in constructors
delete       → disable compiler-generated functions
noexcept     → function won't throw exceptions
```

### Smart Pointer 1-Liners
```
unique_ptr  → one owner, zero overhead, use by default
shared_ptr  → multiple owners, ref-counted, slightly heavier
weak_ptr    → observer only, breaks shared_ptr cycles
```

### Memory Segments (Quick)
```
Stack   → local vars, fast, auto-managed
Heap    → new/delete, manual, large
Data    → global/static initialized
BSS     → global/static uninitialized
Text    → code (read-only)
```

### Polymorphism Quick Ref
```
Compile-time: function overloading, operator overloading
Runtime:      virtual functions, function overriding
Key:          Base* ptr = new Derived(); ptr->method(); → calls Derived's method
```

### Rules to Remember
```
Rule of Three:  define destructor → also define copy ctor + copy assignment
Rule of Five:   + move ctor + move assignment
Rule of Zero:   use smart pointers → don't define any of the five
```

### Interview Answer Template
For any OOP concept:
1. **Definition** — 1-2 sentences
2. **Tech Example** — ATM, payment gateway, notification service, etc.
3. **Code** — minimal but complete
4. **What it provides** — security / reuse / flexibility / etc.
5. **Project hook** — tie to something you've built

---

*Good luck with your interview. You've got this.*
