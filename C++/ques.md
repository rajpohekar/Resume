# C++ OOP Interview Preparation Guide

## 1. What are the four main principles of OOP and define each of them?
1. **Encapsulation:** The bundling of data (variables) and methods (functions) that operate on that data into a single unit (class), while restricting access to some details (using access specifiers like `private`).
2. **Abstraction:** Hiding complex implementation details and showing only the necessary features of an object. (e.g., using a TV remote without knowing how the circuit board works).
3. **Inheritance:** The mechanism where a new class (derived class) acquires the properties and behaviors of an existing class (base class), promoting code reuse.
4. **Polymorphism:** Polymorphism allows the same method call to behave differently depending on the object type at runtime.

### Example (Overview of Principles):
```cpp
#include <iostream>
using namespace std;

// Abstraction & Encapsulation
class Animal {
private:
    int age; // Hidden data
public:
    void setAge(int a) { age = a; } // Controlled access
    virtual void makeSound() = 0;   // Abstraction (pure virtual)
};

// Inheritance
class Dog : public Animal {
public:
    // Polymorphism (Overriding behavior)
    void makeSound() override {
        cout << "Woof!" << endl;
    }
};
```

## 2. What is the difference between a class and an object in C++?
* **Class:** A user-defined data type that acts as a blueprint or template. It defines the structure (attributes) and behavior (methods) but does not occupy memory (until instantiated).
* **Object:** An instance of a class. It is a real-world entity created from the class blueprint that occupies memory and holds specific data values.

### Example:
```cpp
class Car {          // Class (Blueprint)
public:
    string brand;
    void honk() { cout << "Beep!" << endl; }
};

int main() {
    Car myCar;       // Object (Memory allocated)
    myCar.brand = "Toyota";
    myCar.honk();
}
```

## 3. What is the difference between shallow copy and deep copy in C++?
* **Shallow Copy:** Copies the member values of one object to another. If the object contains pointers, only the memory address is copied, not the data it points to. Both objects end up pointing to the *same* memory location.
* **Deep Copy:** Copies the member values *and* allocates new memory for any pointers, copying the actual data into the new memory. The objects remain independent.

### Example:
```cpp
class Shallow {
public:
    int* data;
    Shallow(int d) { data = new int(d); }
    // Default copy constructor does a Shallow Copy
};

class Deep {
public:
    int* data;
    Deep(int d) { data = new int(d); }
    // Custom Copy Constructor for Deep Copy
    Deep(const Deep& other) {
        data = new int(*(other.data)); // Allocates new memory
    }
    ~Deep() { delete data; }
};
```

## 4. What is constructor overloading in C++?
It is the concept of having multiple constructors in a class with the same name but different parameter lists (different number or types of arguments). This allows objects to be initialized in different ways.

### Example:
```cpp
class Point {
public:
    int x, y;
    Point() { x = 0; y = 0; }                // Constructor 1 (Default)
    Point(int val) { x = val; y = val; }     // Constructor 2 (One param)
    Point(int xVal, int yVal) { x = xVal; y = yVal; } // Constructor 3 (Two params)
};
```

## 5. What is a virtual function in C++?
A member function declared in a base class using the `virtual` keyword that is redefined (overridden) by a derived class. It ensures that the correct function is called for an object, regardless of the type of reference (or pointer) used for the function call. This enables **Runtime Polymorphism**.

### Example:
```cpp
class Base {
public:
    virtual void show() { cout << "Base" << endl; }
};

class Derived : public Base {
public:
    void show() override { cout << "Derived" << endl; }
};

int main() {
    Base* b = new Derived();
    b->show(); // Outputs "Derived" because of 'virtual'
    delete b;
}
```

## 6. Explain the concept of multiple inheritance in C++?
Multiple inheritance is a feature where a class can inherit from more than one base class. The derived class acquires the properties of all parent classes.

### Example:
```cpp
class LandAnimal {
public:
    void walk() { cout << "Walking..." << endl; }
};

class WaterAnimal {
public:
    void swim() { cout << "Swimming..." << endl; }
};

// Inherits from both
class Amphibian : public LandAnimal, public WaterAnimal {};

int main() {
    Amphibian frog;
    frog.walk();
    frog.swim();
}
```

## 7. What is the difference between public, private, and protected access specifiers?
* **Public:** Members are accessible from anywhere (inside and outside the class).
* **Private:** Members are accessible *only* within the class defining them (and by friends). They are hidden from derived classes and the outside world.
* **Protected:** Members are accessible within the class and by derived classes (subclasses), but not by the outside world.

### Example:
```cpp
class Base {
public:    int pub = 1;  // Accessible everywhere
protected: int prot = 2; // Accessible here and in Derived classes
private:   int priv = 3; // Accessible ONLY in Base
};

class Derived : public Base {
    void test() {
        cout << pub;  // OK
        cout << prot; // OK
        // cout << priv; // ERROR
    }
};
```

## 8. What are friend classes?
A `friend` class is a class that is granted access to the `private` and `protected` members of another class. If Class B is a friend of Class A, B can access A's private data.

### Example:
```cpp
class SecretInfo {
private:
    int password = 1234;
    friend class Hacker; // Grants access to Hacker class
};

class Hacker {
public:
    void stealInfo(SecretInfo& obj) {
        cout << "Stolen password: " << obj.password << endl; // Accesses private data
    }
};
```

## 9. What is a pure virtual function?
A virtual function that has no implementation in the base class and is set to zero (`= 0`). It forces derived classes to implement this function. A class containing a pure virtual function becomes an **Abstract Class**.

### Example:
```cpp
class Shape {
public:
    virtual void draw() = 0; // Pure virtual function
};

class Circle : public Shape {
public:
    void draw() override { cout << "Drawing Circle" << endl; }
};
```

## 10. What is the "diamond problem" in C++ and how is it solved?
This occurs in multiple inheritance when two classes (`B` and `C`) inherit from the same base class (`A`), and a fourth class (`D`) inherits from both `B` and `C`. This creates two copies of `A` inside `D`, causing ambiguity.
* **Solution:** Use **Virtual Inheritance**. `class B : virtual public A { ... };` ensures only one instance of the base class is present.

### Example:
```cpp
class A { public: int x; };
class B : virtual public A {}; // Virtual inheritance
class C : virtual public A {}; // Virtual inheritance
class D : public B, public C {};

int main() {
    D obj;
    obj.x = 10; // No ambiguity! Only one copy of 'x' exists.
}
```

## 11. What are static members in C++?
* **Static Variable:** Shared by all objects of the class. It is stored in a single memory location, not separately for each object.
* **Static Function:** Can be called without creating an object (using `ClassName::func()`). It can only access static data members of the class.

### Example:
```cpp
class Player {
public:
    static int playerCount; // Declaration
    Player() { playerCount++; }
    static void showCount() { cout << playerCount << endl; }
};

int Player::playerCount = 0; // Initialization outside class

int main() {
    Player p1, p2;
    Player::showCount(); // Outputs 2. Called without an object.
}
```

## 12. What is the difference between function and operator overloading?
* **Function Overloading:** Creating multiple functions with the same name but different parameters.
* **Operator Overloading:** Redefining the way standard operators (like `+`, `-`, `<<`) work with user-defined types (objects). e.g., enabling `obj1 + obj2`.

### Example:
```cpp
class MathInfo {
public:
    // Function Overloading
    void print(int x) { cout << "Int: " << x << endl; }
    void print(double x) { cout << "Double: " << x << endl; }
};

class Complex {
public:
    int real, imag;
    Complex(int r, int i) : real(r), imag(i) {}
    
    // Operator Overloading (+)
    Complex operator+(const Complex& obj) {
        return Complex(real + obj.real, imag + obj.imag);
    }
};
```

## 13. What is the difference between Aggregation, Association, and Composition?
* **Association:** A generic "uses-a" relationship between two independent objects.
* **Aggregation:** A "has-a" relationship where the child can exist independently of the parent (Weak bond).
* **Composition:** A "has-a" relationship where the child *cannot* exist without the parent (Strong bond).

### Example:
```cpp
class Engine {}; 
class Teacher {};

// Composition (Strong bond - Engine dies with Car)
class Car {
private:
    Engine eng; // Object created internally
};

// Aggregation (Weak bond - Teacher exists outside Department)
class Department {
private:
    Teacher* t; // Pointer to external object
public:
    Department(Teacher* teacherPtr) : t(teacherPtr) {}
};
```

## 14. What is the role of the virtual keyword in C++?
It tells the compiler to perform **dynamic linkage** (late binding) on the function. It enables polymorphism by ensuring the most derived version of a function is called when a base class pointer points to a derived class object. *(See Example in Question 5).*

## 15. What is the difference between Static (Compile-time) and Dynamic (Runtime) Polymorphism?
* **Static Polymorphism:** Resolved at compile time (Faster execution). Example: Function/Operator Overloading, Templates.
* **Dynamic Polymorphism:** Resolved at runtime (Slower due to V-Table lookups). Example: Virtual Functions.

### Example:
```cpp
// Static (Resolved at compile time based on arguments)
void add(int a, int b) {}
void add(double a, double b) {}

// Dynamic (Resolved at runtime based on object type)
class Base { virtual void run() {} };
class Derived : public Base { void run() override {} };
```

## 16. What is the difference between a constructor and a copy constructor?
* **Constructor:** Initializes a new object.
* **Copy Constructor:** Initializes a new object as a copy of an *existing* object.

### Example:
```cpp
class Box {
public:
    int size;
    Box(int s) { size = s; }            // Normal Constructor
    Box(const Box& b) { size = b.size; } // Copy Constructor
};

int main() {
    Box b1(10);  // Calls normal constructor
    Box b2 = b1; // Calls copy constructor
}
```

## 17. What is the use of the friend keyword in C++?
It is used to grant non-member functions or other classes access to the `private` and `protected` members of a class. Commonly used for operator overloading (like `operator<<`).

### Example:
```cpp
class Point {
private:
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    // Global function made a friend
    friend void printPoint(const Point& p); 
};

void printPoint(const Point& p) {
    // Can access private x and y
    cout << p.x << ", " << p.y << endl; 
}
```

## 18. What is an abstract class in C++?
A class that contains at least one **pure virtual function**. You cannot instantiate an abstract class. It serves as an interface for derived classes. *(See Example in Question 9).*

## 19. What is the role of the 'this' pointer in C++?
`this` is an implicit pointer available inside non-static member functions. It points to the object for which the function was called. Used to distinguish between member variables and parameters, and to return the current object.

### Example:
```cpp
class Rectangle {
private:
    int length, width;
public:
    // 1. Distinguish parameter from member
    void setDimensions(int length, int width) {
        this->length = length; 
        this->width = width;
    }
    
    // 2. Return current object for chaining
    Rectangle& grow() {
        this->length++;
        this->width++;
        return *this; 
    }
};
// Usage: rect.setDimensions(5,5); rect.grow().grow();
```

## 20. What is the difference between function overloading and function overriding?
* **Overloading:** Same function name, different parameters, *same scope* (inside one class).
* **Overriding:** Same function name, same parameters, *different scope* (Base vs Derived class). Used with `virtual` functions. *(See Examples in Q12 for Overloading and Q5 for Overriding).*

## 21. What is a "Deep Copy" constructor, and when would you use it?
A copy constructor that performs a deep copy (allocates new memory for pointers). Use it when your class handles dynamic memory (pointers like `int* ptr = new int;`). *(See Example in Question 3).*

## 22. What is the purpose of the explicit keyword in C++?
It prevents the compiler from using constructors for **implicit type conversions**.

### Example:
```cpp
class Weight {
public:
    explicit Weight(int kg) {} 
};

int main() {
    // Weight w1 = 50; // ERROR: Implicit conversion blocked
    Weight w2(50);     // OK: Explicit call
}
```

## 23. Explain RAII (Resource Acquisition Is Initialization).
A programming idiom where resource management (memory, file handles, locks) is tied to the lifecycle of an object.
* **Acquisition:** Resource is acquired in the constructor.
* **Release:** Resource is released in the destructor.

### Example:
```cpp
class FileHandler {
private:
    FILE* file;
public:
    FileHandler(const char* filename) {
        file = fopen(filename, "w"); // Acquire
    }
    ~FileHandler() {
        if (file) fclose(file);      // Release safely automatically
    }
};

void process() {
    FileHandler fh("log.txt");
    // If function throws error or ends, fh goes out of scope, file is closed!
}
```

## 24. What is dynamic_cast in C++? How does it work?
A type cast used for safe downcasting (converting a base pointer to a derived pointer) at runtime. It works only with polymorphic classes (classes with at least one virtual function).

### Example:
```cpp
class Base { virtual void dummy() {} }; // Polymorphic
class Derived : public Base { public: void specificFunc() {} };

int main() {
    Base* b = new Derived();
    
    // Safe downcasting
    Derived* d = dynamic_cast<Derived*>(b);
    if (d) {
        d->specificFunc(); // Cast succeeded
    }
    delete b;
}
```

## 25. What is an "Interface"?
C++ does not have a specific `interface` keyword. An interface in C++ is simulated using an **Abstract Class** where all methods are pure virtual functions and there are no data members.

### Example:
```cpp
class IPrintable { // The 'I' prefix conventionally denotes an Interface
public:
    virtual void print() const = 0;
    virtual ~IPrintable() = default; // Essential for interfaces
};
```

## 26. What are Virtual Destructors and why are they important?
A destructor in the base class declared with `virtual`.
* **Importance:** If you delete a derived class object through a base class pointer, a non-virtual base destructor will *only* call the base destructor, leaking the derived part's memory. A virtual destructor ensures *both* are called.

### Example:
```cpp
class Base {
public:
    virtual ~Base() { cout << "Base Destroyed" << endl; } // VIRTUAL!
};

class Derived : public Base {
    int* data;
public:
    Derived() { data = new int[100]; }
    ~Derived() { 
        delete[] data; 
        cout << "Derived Destroyed" << endl; 
    }
};

int main() {
    Base* b = new Derived();
    delete b; // Safely calls Derived destructor, THEN Base destructor
}
```

## 27. What is the difference between an "abstract class" and an "interface"?
* **Abstract Class:** Can have some implemented methods and member variables. Used for partial abstraction.
* **Interface (Pure Abstract Class):** Has *only* pure virtual functions and no data members. Used to define a strict contract. *(See Q25 for Interface example).*

## 28. What is Multiple Inheritance and how does C++ handle ambiguity?
C++ handles ambiguity via **Virtual Inheritance** (See Q10) or explicitly using the **Scope Resolution Operator**.

### Example (Scope Resolution):
```cpp
class A { public: void show() { cout << "A"; } };
class B { public: void show() { cout << "B"; } };
class C : public A, public B {}; // No virtual inheritance

int main() {
    C obj;
    // obj.show(); // ERROR: Ambiguous
    obj.A::show(); // Solved via Scope Resolution
}
```

## 29. What is a "Singleton" class in C++?
A design pattern that ensures a class has only **one instance** and provides a global point of access to it.

### Example:
```cpp
class Singleton {
private:
    // 1. Private constructor
    Singleton() { cout << "Singleton created\n"; } 
    
public:
    // 2. Static method to get the instance (Meyers Singleton - Thread-safe in C++11)
    static Singleton& getInstance() {
        static Singleton instance; 
        return instance;
    }
    
    // 3. Delete copy constructor and assignment operator to prevent clones
    Singleton(const Singleton&) = delete; 
    void operator=(const Singleton&) = delete;
};

int main() {
    Singleton& s1 = Singleton::getInstance();
    Singleton& s2 = Singleton::getInstance();
    // s1 and s2 refer to the exact same object. Constructor prints only once.
}
```
