# Complete C++ OOP Interview Preparation Guide

> This guide contains beginner-to-advanced Object Oriented Programming interview questions in C++ with detailed explanations, examples, interview tips, and important edge cases.

---

# Table of Contents

1. OOP Basics
2. Class and Object
3. Constructors and Destructors
4. Access Specifiers
5. Inheritance
6. Polymorphism
7. Abstraction and Encapsulation
8. Virtual Functions
9. Abstract Classes and Interfaces
10. Static Members
11. Friend Keyword
12. Copy Constructor and Assignment Operator
13. Deep Copy vs Shallow Copy
14. Operator Overloading
15. Runtime vs Compile Time Polymorphism
16. Virtual Destructor
17. Diamond Problem
18. this Pointer
19. Singleton Design Pattern
20. RAII
21. Memory Management
22. Dynamic Casting
23. Important Interview Coding Examples
24. Real-Life Examples
25. Frequently Asked Interview Follow-Ups
26. Advanced OOP Interview Questions
27. HR + Project-Oriented OOP Questions
28. Common Interview Mistakes
29. Rapid Revision Notes

---

# 1. What are the four pillars of OOP?

## Answer

Object Oriented Programming is based on four major principles:

## 1. Encapsulation

Encapsulation means wrapping data and functions into a single unit called a class.

### Purpose

* Data hiding
* Security
* Controlled access

### Example

```cpp
class BankAccount {
private:
    int balance;

public:
    void setBalance(int b) {
        balance = b;
    }

    int getBalance() {
        return balance;
    }
};
```

Here `balance` is private and accessed only using public methods.

---

## 2. Abstraction

Abstraction means hiding implementation details and showing only essential functionality.

### Real-Life Example

Driving a car without knowing engine internals.

### Example

```cpp
class Car {
public:
    void startCar() {
        cout << "Car Started";
    }
};
```

User only knows `startCar()`.

---

## 3. Inheritance

Inheritance allows one class to acquire properties and methods of another class.

### Advantages

* Code reuse
* Better organization
* Hierarchical structure

### Example

```cpp
class Animal {
public:
    void eat() {
        cout << "Eating";
    }
};

class Dog : public Animal {
public:
    void bark() {
        cout << "Barking";
    }
};
```

---

## 4. Polymorphism

Polymorphism means one function behaving differently in different situations.

### Types

1. Compile Time Polymorphism
2. Runtime Polymorphism

### Example

```cpp
class Animal {
public:
    virtual void sound() {
        cout << "Animal sound";
    }
};

class Dog : public Animal {
public:
    void sound() override {
        cout << "Dog Bark";
    }
};
```

---

# 2. Difference Between Class and Object

| Class               | Object            |
| ------------------- | ----------------- |
| Blueprint           | Instance          |
| Logical entity      | Physical entity   |
| No memory allocated | Memory allocated  |
| Defines structure   | Holds actual data |

### Example

```cpp
class Student {
public:
    string name;
};

Student s1;
```

`Student` is class.
`s1` is object.

---

# 3. Constructors in C++

## What is a Constructor?

A constructor is a special member function automatically called when an object is created.

### Characteristics

* Same name as class
* No return type
* Automatically invoked

---

## Types of Constructors

### 1. Default Constructor

```cpp
class Demo {
public:
    Demo() {
        cout << "Default Constructor";
    }
};
```

---

### 2. Parameterized Constructor

```cpp
class Student {
public:
    int age;

    Student(int a) {
        age = a;
    }
};
```

---

### 3. Copy Constructor

```cpp
class Demo {
public:
    int x;

    Demo(int a) {
        x = a;
    }

    Demo(const Demo &d) {
        x = d.x;
    }
};
```

---

## Constructor Overloading

Same constructor name with different parameters.

```cpp
class Test {
public:
    Test() {}
    Test(int x) {}
    Test(string name) {}
};
```

---

# 4. Destructor in C++

## What is Destructor?

Destructor is automatically called when object goes out of scope.

### Syntax

```cpp
~ClassName() {}
```

### Example

```cpp
class Demo {
public:
    ~Demo() {
        cout << "Destructor Called";
    }
};
```

---

# 5. Access Specifiers

| Specifier | Accessible Inside Class | Derived Class | Outside Class |
| --------- | ----------------------- | ------------- | ------------- |
| private   | Yes                     | No            | No            |
| protected | Yes                     | Yes           | No            |
| public    | Yes                     | Yes           | Yes           |

---

# 6. Types of Inheritance

## 1. Single Inheritance

```cpp
class A {};
class B : public A {};
```

---

## 2. Multiple Inheritance

```cpp
class A {};
class B {};
class C : public A, public B {};
```

---

## 3. Multilevel Inheritance

```cpp
class A {};
class B : public A {};
class C : public B {};
```

---

## 4. Hierarchical Inheritance

```cpp
class A {};
class B : public A {};
class C : public A {};
```

---

## 5. Hybrid Inheritance

Combination of multiple inheritance types.

---

# 7. Compile-Time vs Runtime Polymorphism

| Compile-Time                | Runtime                     |
| --------------------------- | --------------------------- |
| Resolved during compilation | Resolved during execution   |
| Faster                      | Slightly slower             |
| Function overloading        | Virtual function overriding |
| Operator overloading        | Runtime dispatch            |

---

# 8. Function Overloading

Same function name with different parameters.

```cpp
class Math {
public:
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
};
```

---

# 9. Function Overriding

Derived class redefines base class method.

```cpp
class Base {
public:
    virtual void show() {
        cout << "Base";
    }
};

class Derived : public Base {
public:
    void show() override {
        cout << "Derived";
    }
};
```

---

# 10. Virtual Function

A virtual function enables runtime polymorphism.

## Why Needed?

Without virtual functions, base class pointer calls base function.

### Example

```cpp
class Base {
public:
    virtual void display() {
        cout << "Base Display";
    }
};

class Derived : public Base {
public:
    void display() override {
        cout << "Derived Display";
    }
};

int main() {
    Base* ptr;
    Derived d;
    ptr = &d;
    ptr->display();
}
```

### Output

```text
Derived Display
```

---

# 11. Pure Virtual Function

A virtual function initialized with `= 0`.

```cpp
class Shape {
public:
    virtual void draw() = 0;
};
```

---

# 12. Abstract Class

A class containing at least one pure virtual function.

### Important Points

* Cannot create objects
* Used for abstraction
* Used as blueprint/interface

---

# 13. Interface in C++

C++ does not have explicit interface keyword.

Interface is created using pure abstract class.

```cpp
class Vehicle {
public:
    virtual void start() = 0;
    virtual void stop() = 0;
};
```

---

# 14. Deep Copy vs Shallow Copy

## Shallow Copy

Copies pointer address.

## Problem

Both objects point to same memory.

### Example

```cpp
class Demo {
public:
    int* ptr;

    Demo(int val) {
        ptr = new int(val);
    }
};
```

Default copy constructor performs shallow copy.

---

## Deep Copy

Creates separate memory.

```cpp
class Demo {
public:
    int* ptr;

    Demo(int val) {
        ptr = new int(val);
    }

    Demo(const Demo &d) {
        ptr = new int(*d.ptr);
    }
};
```

---

# 15. Copy Constructor vs Assignment Operator

| Copy Constructor             | Assignment Operator     |
| ---------------------------- | ----------------------- |
| Creates new object           | Assigns existing object |
| Called during initialization | Called after creation   |

### Example

```cpp
Demo d1(10);
Demo d2 = d1; // Copy Constructor

Demo d3(20);
d3 = d1; // Assignment Operator
```

---

# 16. Friend Function

Friend function can access private members.

```cpp
class Test {
private:
    int x = 10;

    friend void show(Test t);
};

void show(Test t) {
    cout << t.x;
}
```

---

# 17. Friend Class

Entire class gets access.

```cpp
class A {
private:
    int x = 5;

    friend class B;
};

class B {
public:
    void display(A a) {
        cout << a.x;
    }
};
```

---

# 18. Static Members

## Static Variable

Shared among all objects.

```cpp
class Test {
public:
    static int count;
};

int Test::count = 0;
```

---

## Static Function

```cpp
class Test {
public:
    static void show() {
        cout << "Static Function";
    }
};
```

Called using:

```cpp
Test::show();
```

---

# 19. this Pointer

`this` stores address of current object.

```cpp
class Demo {
private:
    int x;

public:
    void setX(int x) {
        this->x = x;
    }
};
```

---

# 20. Virtual Destructor

## Why Important?

Ensures derived destructor also gets called.

```cpp
class Base {
public:
    virtual ~Base() {
        cout << "Base Destructor";
    }
};

class Derived : public Base {
public:
    ~Derived() {
        cout << "Derived Destructor";
    }
};
```

---

# 21. Diamond Problem

## Problem

```text
       A
      / \
     B   C
      \ /
       D
```

D gets two copies of A.

---

## Solution: Virtual Inheritance

```cpp
class A {};

class B : virtual public A {};
class C : virtual public A {};

class D : public B, public C {};
```

---

# 22. Operator Overloading

Allows operators to work with objects.

```cpp
class Complex {
public:
    int real, imag;

    Complex(int r, int i) {
        real = r;
        imag = i;
    }

    Complex operator + (Complex const &obj) {
        return Complex(real + obj.real, imag + obj.imag);
    }
};
```

---

# 23. Singleton Class

Ensures only one object exists.

```cpp
class Singleton {
private:
    Singleton() {}

public:
    static Singleton& getInstance() {
        static Singleton instance;
        return instance;
    }

    Singleton(const Singleton&) = delete;
};
```

---

# 24. RAII (Resource Acquisition Is Initialization)

Resources acquired in constructor and released in destructor.

## Benefits

* Prevents memory leaks
* Automatic cleanup
* Exception safety

---

# 25. Dynamic Memory Allocation

## new Operator

```cpp
int* ptr = new int(10);
```

---

## delete Operator

```cpp
delete ptr;
```

---

# 26. Smart Pointers

## unique_ptr

Only one owner.

```cpp
unique_ptr<int> ptr = make_unique<int>(10);
```

---

## shared_ptr

Multiple owners.

```cpp
shared_ptr<int> p1 = make_shared<int>(10);
shared_ptr<int> p2 = p1;
```

---

## weak_ptr

Avoids circular dependency.

---

# 27. dynamic_cast

Used for safe downcasting.

```cpp
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b);
```

---

# 28. Difference Between struct and class

| struct               | class           |
| -------------------- | --------------- |
| Default public       | Default private |
| Mainly data grouping | Full OOP        |

---

# 29. Difference Between C and C++

| C               | C++                   |
| --------------- | --------------------- |
| Procedural      | Object Oriented       |
| No classes      | Supports classes      |
| No inheritance  | Supports inheritance  |
| No polymorphism | Supports polymorphism |

---

# 30. Inline Function

```cpp
inline int square(int x) {
    return x * x;
}
```

Reduces function call overhead.

---

# 31. Namespace in C++

Avoids naming conflicts.

```cpp
namespace First {
    int x = 10;
}
```

---

# 32. Exception Handling

```cpp
try {
    throw 10;
}
catch(int x) {
    cout << x;
}
```

---

# 33. Difference Between Heap and Stack

| Stack             | Heap                |
| ----------------- | ------------------- |
| Automatic memory  | Dynamic memory      |
| Faster            | Slower              |
| Limited size      | Large memory        |
| Auto deallocation | Manual deallocation |

---

# 34. Real-Life Examples of OOP

## Encapsulation

ATM machine hiding internal logic.

## Abstraction

Car driving.

## Inheritance

Animal → Dog.

## Polymorphism

Same payment method behaving differently for UPI/Card/Cash.

---

# 35. Real-Life Example of Polymorphism in Your Project

## TicketWise Project Example

Different user roles perform same operation differently.

```text
Admin -> Can assign ticket
Agent -> Can resolve ticket
User -> Can create ticket
```

Same function behaves differently based on role.

---

# 36. Real-Life Example of Encapsulation in Your Project

In `TicketWise`, JWT token validation and database access are hidden inside backend APIs.

Frontend only interacts with API endpoints.

---

# 37. Real-Life Example of Abstraction in Your Project

User clicks "Create Ticket".

Internally:

* Validation happens
* Database query executes
* AI categorization occurs
* Notification service triggers

User only sees final result.

---

# 38. Real-Life Example of Inheritance

```cpp
class User {
public:
    string name;
};

class Admin : public User {};
class Agent : public User {};
```

---

# 39. Frequently Asked Interview Questions

## Q1. Why use virtual functions?

To achieve runtime polymorphism.

---

## Q2. Why constructor cannot be virtual?

Constructor initializes object before virtual table setup.

---

## Q3. Can static functions be virtual?

No.
Static functions belong to class, not object.

---

## Q4. Why destructor should be virtual?

To ensure proper cleanup.

---

## Q5. What is object slicing?

```cpp
Base b = Derived();
```

Derived-specific data gets sliced.

---

## Q6. What is late binding?

Function resolved at runtime.

---

## Q7. What is early binding?

Function resolved at compile time.

---

## Q8. Difference between overriding and overloading?

| Overloading          | Overriding      |
| -------------------- | --------------- |
| Same scope           | Different scope |
| Compile time         | Runtime         |
| Different parameters | Same parameters |

---

## Q9. Can we overload main()?

No.

---

## Q10. Can we override private methods?

No.

---

# 40. Advanced OOP Interview Questions

## Q1. What is vtable?

Virtual table storing addresses of virtual functions.

Compiler uses it for runtime polymorphism.

---

## Q2. What is vptr?

Hidden pointer added by compiler pointing to vtable.

---

## Q3. What happens when virtual function is called?

1. Object's vptr accessed
2. vtable searched
3. Correct function called at runtime

---

## Q4. What is covariant return type?

Derived override returns derived pointer/reference.

---

## Q5. What is object slicing?

Derived object copied into base object loses derived data.

---

# 41. OOP Coding Questions Asked in Interviews

## Reverse a String

```cpp
string s = "hello";
reverse(s.begin(), s.end());
```

---

## Swap Two Numbers Without Third Variable

```cpp
int a = 5, b = 10;
a = a + b;
b = a - b;
a = a - b;
```

---

## Check Palindrome

```cpp
string s = "madam";
string t = s;
reverse(t.begin(), t.end());
cout << (s == t);
```

---

# 42. Interview Trap Questions

## Q1. Difference between malloc and new?

| malloc              | new              |
| ------------------- | ---------------- |
| C function          | C++ operator     |
| No constructor call | Constructor call |
| Returns void*       | Proper type      |

---

## Q2. Difference between free and delete?

| free          | delete            |
| ------------- | ----------------- |
| No destructor | Destructor called |

---

## Q3. Can constructor return value?

No.

---

## Q4. Can destructor take parameters?

No.

---

## Q5. Can we make constructor private?

Yes.
Used in Singleton.

---

# 43. Important STL Concepts Asked with OOP

## Vector

```cpp
vector<int> v;
```

---

## Map

```cpp
map<int, string> mp;
```

---

## Set

```cpp
set<int> st;
```

---

# 44. Most Asked HR + OOP Combined Questions

## Explain OOP Concepts Using Your Project

### TicketWise

#### Encapsulation

JWT and DB logic hidden.

#### Abstraction

User only creates ticket.

#### Inheritance

Admin/User/Agent derived from User.

#### Polymorphism

Different roles behave differently.

---

# 45. Interviewer Follow-Up Questions

## Why OOP is better than Procedural Programming?

* Better modularity
* Reusability
* Maintainability
* Security
* Scalability

---

## Why C++ for system-level programming?

* Fast
* Direct memory control
* OOP + low-level support
* High performance

---

## Why encapsulation important?

Protects data from unauthorized access.

---

## Why abstraction important?

Reduces complexity.

---

# 46. Common Mistakes in Interviews

## Mistake 1

Confusing overloading and overriding.

---

## Mistake 2

Forgetting virtual destructor.

---

## Mistake 3

Explaining only definitions without examples.

---

## Mistake 4

Not giving project-based examples.

---

# 47. Rapid Revision Notes

## OOP Pillars

* Encapsulation
* Abstraction
* Inheritance
* Polymorphism

## Compile-Time Polymorphism

* Function Overloading
* Operator Overloading

## Runtime Polymorphism

* Virtual Functions
* Function Overriding

## Access Specifiers

* private
* protected
* public

## Memory

* Stack
* Heap

## Important Keywords

* virtual
* override
* friend
* static
* this
* explicit

---

# 48. Most Important Interview Lines

## For Runtime Polymorphism

"Runtime polymorphism is achieved using virtual functions and base class pointers."

---

## For Encapsulation

"Encapsulation improves security by restricting direct access to data members."

---

## For Abstraction

"Abstraction hides implementation complexity and exposes only necessary functionality."

---

## For Inheritance

"Inheritance promotes code reusability and hierarchical relationships."

---

# 49. Best Final Revision Before Interview

Focus strongly on:

1. Virtual Functions
2. Virtual Destructor
3. Constructor vs Destructor
4. Deep Copy vs Shallow Copy
5. Overloading vs Overriding
6. Runtime vs Compile-Time Polymorphism
7. Diamond Problem
8. Abstract Class
9. Singleton Pattern
10. Real-life project examples

---

# 50. Final Interview Tip

Do not only define concepts.

Always explain using:

1. Definition
2. Syntax
3. Small Example
4. Real-life Example
5. Project Example

That makes your answer strong and interview-ready.

