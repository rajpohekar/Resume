# 1. C++ Fundamentals

## 🔷 1.1 OOP Concepts

Object-Oriented Programming (OOP) is a programming paradigm that organizes software around **objects and classes** rather than functions and logic.

A class acts as a blueprint, and objects are instances of that class.

### Advantages of OOP
- Code reusability
- Better maintainability
- Modularity
- Data security
- Easy scalability

---

# 4 Pillars of OOP

---

## 1. Encapsulation

### Definition
Encapsulation is the process of binding data members and member functions together into a single unit (class) and restricting direct access to data using access modifiers.

### Real-World Analogy
An ATM machine allows users to perform transactions through buttons and menus. Users cannot directly access the internal cash storage.

### Example

```cpp
#include <iostream>
using namespace std;

class BankAccount {
private:
    double balance;

public:
    BankAccount() {
        balance = 0;
    }

    void deposit(double amount) {
        if(amount > 0)
            balance += amount;
    }

    double getBalance() {
        return balance;
    }
};

int main() {
    BankAccount account;

    account.deposit(5000);

    cout << "Balance: " << account.getBalance();

    return 0;
}
```

### Benefits
- Data hiding
- Better security
- Controlled access
- Improved maintainability

### Interview Question
**Q: Difference between Encapsulation and Data Hiding?**

**Answer:**
- Encapsulation = Wrapping data and methods together.
- Data Hiding = Restricting direct access to data.
- Data hiding is achieved using encapsulation.

---

## 2. Inheritance

### Definition
Inheritance allows one class to acquire properties and behaviors of another class.

### Real-World Analogy
A Car is a Vehicle. It inherits common features such as speed, engine, and start functionality while adding its own features.

### Example

```cpp
#include <iostream>
using namespace std;

class Vehicle {
public:
    void start() {
        cout << "Vehicle Started" << endl;
    }
};

class Car : public Vehicle {
public:
    void playMusic() {
        cout << "Playing Music" << endl;
    }
};

int main() {
    Car car;

    car.start();
    car.playMusic();

    return 0;
}
```

---

### Types of Inheritance

#### 1. Single Inheritance

```cpp
class A {};
class B : public A {};
```

#### 2. Multilevel Inheritance

```cpp
class A {};
class B : public A {};
class C : public B {};
```

#### 3. Multiple Inheritance

```cpp
class A {};
class B {};

class C : public A, public B {};
```

#### 4. Hierarchical Inheritance

```cpp
class A {};

class B : public A {};
class C : public A {};
```

#### 5. Hybrid Inheritance

Combination of two or more inheritance types.

---

### Diamond Problem

```cpp
class A {};

class B : public A {};
class C : public A {};

class D : public B, public C {};
```

Here D gets two copies of A causing ambiguity.

### Solution: Virtual Inheritance

```cpp
class A {};

class B : virtual public A {};
class C : virtual public A {};

class D : public B, public C {};
```

---

### Benefits
- Code reuse
- Reduced redundancy
- Easier maintenance

### Interview Question

**Q: Why use inheritance?**

**Answer:**
Inheritance promotes code reusability and establishes an "IS-A" relationship between classes.

Example:
- Car IS-A Vehicle
- Dog IS-A Animal

---

## 3. Polymorphism

### Definition
Polymorphism means "many forms."

The same function or interface can behave differently depending on the object.

---

## Types of Polymorphism

### A. Compile-Time Polymorphism (Static Binding)

Achieved using:
- Function Overloading
- Operator Overloading

---

### Function Overloading

```cpp
#include <iostream>
using namespace std;

class Calculator {
public:
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
};

int main() {
    Calculator c;

    cout << c.add(2,3) << endl;
    cout << c.add(2.5,3.5) << endl;
    cout << c.add(1,2,3) << endl;
}
```

---

### Operator Overloading

```cpp
#include <iostream>
using namespace std;

class Complex {
public:
    int real, imag;

    Complex(int r, int i) {
        real = r;
        imag = i;
    }

    Complex operator+(Complex const& obj) {
        return Complex(real + obj.real,
                       imag + obj.imag);
    }

    void display() {
        cout << real << " + " << imag << "i" << endl;
    }
};

int main() {
    Complex c1(2,3);
    Complex c2(4,5);

    Complex c3 = c1 + c2;

    c3.display();
}
```

---

### B. Runtime Polymorphism (Dynamic Binding)

Achieved using:
- Method Overriding
- Virtual Functions

### Example

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    virtual void sound() {
        cout << "Animal Sound" << endl;
    }
};

class Dog : public Animal {
public:
    void sound() override {
        cout << "Bark" << endl;
    }
};

class Cat : public Animal {
public:
    void sound() override {
        cout << "Meow" << endl;
    }
};

int main() {

    Animal* a1 = new Dog();
    Animal* a2 = new Cat();

    a1->sound();
    a2->sound();

    delete a1;
    delete a2;
}
```

---

### Why Virtual Function?

Without virtual:

```cpp
Animal* a = new Dog();
a->sound();
```

Output:
```text
Animal Sound
```

With virtual:

```text
Bark
```

Runtime decides which function to call.

---

### Interview Questions

#### Q: Difference between Overloading and Overriding?

| Overloading | Overriding |
|------------|------------|
| Same function name | Same function signature |
| Same class | Parent-child classes |
| Compile-time | Runtime |
| No inheritance required | Inheritance required |

---

#### Q: What is a Virtual Function?

**Answer:**
A virtual function allows runtime polymorphism by enabling dynamic binding.

---

#### Q: What is a Pure Virtual Function?

```cpp
class Shape {
public:
    virtual void draw() = 0;
};
```

A pure virtual function makes a class abstract.

---

## 4. Abstraction

### Definition

Abstraction means hiding implementation details and exposing only essential functionality.

### Real-World Analogy

When driving a car, you only use the steering wheel, brake, and accelerator. You don't need to know how the engine internally works.

---

### Abstraction using Abstract Class

```cpp
#include <iostream>
using namespace std;

class Shape {
public:
    virtual double area() = 0;
};

class Circle : public Shape {
private:
    double radius;

public:
    Circle(double r) {
        radius = r;
    }

    double area() override {
        return 3.14 * radius * radius;
    }
};

int main() {

    Shape* shape;

    Circle c(5);

    shape = &c;

    cout << shape->area();
}
```

---

### Benefits
- Hides complexity
- Improves security
- Makes systems easier to use
- Reduces dependency

---

### Interview Question

**Q: Difference between Abstraction and Encapsulation?**

| Abstraction | Encapsulation |
|------------|------------|
| Hides implementation | Hides data |
| Focus on what | Focus on how |
| Achieved using abstract classes and interfaces | Achieved using access modifiers |

---

# OOP Relationships

## IS-A Relationship

Achieved through inheritance.

```cpp
Car IS-A Vehicle
Dog IS-A Animal
```

---

## HAS-A Relationship

Achieved through composition.

```cpp
class Engine {};

class Car {
private:
    Engine engine;
};
```

Car HAS-A Engine.

---

# Constructor

### Definition
Special member function automatically called when object is created.

```cpp
class Student {
public:
    Student() {
        cout << "Constructor Called";
    }
};
```

### Types
- Default Constructor
- Parameterized Constructor
- Copy Constructor

---

## Copy Constructor

```cpp
class Student {
public:
    int age;

    Student(int a) {
        age = a;
    }

    Student(const Student &s) {
        age = s.age;
    }
};
```

---

# Destructor

### Definition
Automatically called when object is destroyed.

```cpp
class Student {
public:
    ~Student() {
        cout << "Destructor Called";
    }
};
```

---

# Virtual Destructor

```cpp
class Base {
public:
    virtual ~Base() {
        cout << "Base Destructor";
    }
};
```

Used when deleting derived objects using base pointers.

---

# this Pointer

```cpp
class Student {
private:
    int age;

public:
    void setAge(int age) {
        this->age = age;
    }
};
```

`this` points to the current object.

---

# Friend Function

```cpp
class Demo {
private:
    int x = 10;

    friend void show(Demo d);
};

void show(Demo d) {
    cout << d.x;
}
```

Friend functions can access private members.

---

# Static Members

```cpp
class Employee {
public:
    static int count;
};

int Employee::count = 0;
```

Static members belong to the class, not objects.

---

# Final Interview Revision

### Must Know Topics

- Class & Object
- Constructor
- Destructor
- Copy Constructor
- Encapsulation
- Inheritance
- Polymorphism
- Function Overloading
- Function Overriding
- Virtual Function
- Pure Virtual Function
- Abstract Class
- Friend Function
- Static Members
- IS-A vs HAS-A
- Diamond Problem
- Virtual Inheritance
- Dynamic Binding
- this Pointer
