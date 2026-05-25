# C++ OOP Interview Preparation Guide

## 1. What are the four main principles of OOP and define each of them?
1. **Encapsulation:** The bundling of data (variables) and methods (functions) that operate on that data into a single unit (class), while restricting access to some details (using access specifiers like `private`).
2. **Abstraction:** Hiding complex implementation details and showing only the necessary features of an object. (e.g., using a TV remote without knowing how the circuit board works).
3. **Inheritance:** The mechanism where a new class (derived class) acquires the properties and behaviors of an existing class (base class), promoting code reuse.
4. **Polymorphism:** Polymorphism allows the same method call to behave differently depending on the object type at runtime.


## 2. What is the difference between a class and an object in C++?
* **Class:** A user-defined data type that acts as a blueprint or template. It defines the structure (attributes) and behavior (methods) but does not occupy memory (until instantiated).
* **Object:** An instance of a class. It is a real-world entity created from the class blueprint that occupies memory and holds specific data values.

## 3. What is the difference between shallow copy and deep copy in C++?
* **Shallow Copy:** Copies the member values of one object to another. If the object contains pointers, only the memory address is copied, not the data it points to. Both objects end up pointing to the *same* memory location.
* **Deep Copy:** Copies the member values *and* allocates new memory for any pointers, copying the actual data into the new memory. The objects remain independent.

## 4. What is constructor overloading in C++?
It is the concept of having multiple constructors in a class with the same name but different parameter lists (different number or types of arguments). This allows objects to be initialized in different ways.

## 5. What is a virtual function in C++?
A member function declared in a base class using the `virtual` keyword that is redefined (overridden) by a derived class. It ensures that the correct function is called for an object, regardless of the type of reference (or pointer) used for the function call. This enables **Runtime Polymorphism**.

## 6. Explain the concept of multiple inheritance in C++?
Multiple inheritance is a feature where a class can inherit from more than one base class. The derived class acquires the properties of all parent classes.
* *Example:* Class `Amphibian` inherits from both `LandAnimal` and `WaterAnimal`.

## 7. What is the difference between public, private, and protected access specifiers?
* **Public:** Members are accessible from anywhere (inside and outside the class).
* **Private:** Members are accessible *only* within the class defining them (and by friends). They are hidden from derived classes and the outside world.
* **Protected:** Members are accessible within the class and by derived classes (subclasses), but not by the outside world.

## 8. What are friend classes?
A `friend` class is a class that is granted access to the `private` and `protected` members of another class. If Class B is a friend of Class A, B can access A's private data.

## 9. What is a pure virtual function?
A virtual function that has no implementation in the base class and is set to zero (`= 0`).
* *Syntax:* `virtual void show() = 0;`
* It forces derived classes to implement this function. A class containing a pure virtual function becomes an **Abstract Class**.

## 10. What is the "diamond problem" in C++ and how is it solved?
This occurs in multiple inheritance when two classes (`B` and `C`) inherit from the same base class (`A`), and a fourth class (`D`) inherits from both `B` and `C`. This creates two copies of `A` inside `D`, causing ambiguity.
* **Solution:** Use **Virtual Inheritance**. `class B : virtual public A { ... };` ensures only one instance of the base class is present.

## 11. What are static members in C++?
* **Static Variable:** Shared by all objects of the class. It is stored in a single memory location, not separately for each object.
* **Static Function:** Can be called without creating an object (using `ClassName::func()`). It can only access static data members of the class.

## 12. What is the difference between function and operator overloading?
* **Function Overloading:** Creating multiple functions with the same name but different parameters.
* **Operator Overloading:** Redefining the way standard operators (like `+`, `-`, `<<`) work with user-defined types (objects). e.g., enabling `obj1 + obj2`.

## 13. What is the difference between Aggregation, Association, and Composition?
* **Association:** A generic "uses-a" relationship between two independent objects (e.g., Teacher and Student).
* **Aggregation:** A "has-a" relationship where the child can exist independently of the parent (Weak bond). E.g., A `Library` has `Books`. If the library is destroyed, the books still exist.
* **Composition:** A "has-a" relationship where the child *cannot* exist without the parent (Strong bond). E.g., A `House` has `Rooms`. If the house is destroyed, the rooms are destroyed too.

## 14. What is the role of the virtual keyword in C++?
It tells the compiler to perform **dynamic linkage** (late binding) on the function. It enables polymorphism by ensuring the most derived version of a function is called when a base class pointer points to a derived class object.

## 15. What is the difference between Static (Compile-time) and Dynamic (Runtime) Polymorphism?
* **Static Polymorphism:** Resolved at compile time.
    * *Examples:* Function Overloading, Operator Overloading, Templates.
    * *Speed:* Faster execution.
* **Dynamic Polymorphism:** Resolved at runtime.
    * *Examples:* Virtual Functions (Function Overriding).
    * *Speed:* Slower due to V-Table lookups.

## 16. What is the difference between a constructor and a copy constructor?
* **Constructor:** Initializes a new object.
* **Copy Constructor:** Initializes a new object as a copy of an *existing* object.
    * *Syntax:* `ClassName(const ClassName &other);`

## 17. What is the use of the friend keyword in C++?
It is used to grant non-member functions or other classes access to the `private` and `protected` members of a class. Commonly used for operator overloading (like `operator<<`).

## 18. What is an abstract class in C++?
A class that contains at least one **pure virtual function**.
* You cannot instantiate an abstract class (cannot create objects of it).
* It serves as an interface for derived classes.

## 19. What is the role of the 'this' pointer in C++?
`this` is an implicit pointer available inside non-static member functions. It points to the object for which the function was called. It is used to:
1. Distinguish between member variables and parameters with the same name (`this->x = x;`).
2. Return the current object (`return *this;`) for chaining calls.

## 20. What is the difference between function overloading and function overriding?
* **Overloading:** Same function name, different parameters, *same scope* (inside one class).
* **Overriding:** Same function name, same parameters, *different scope* (Base vs Derived class). Used with `virtual` functions.

## 21. What is a "Deep Copy" constructor, and when would you use it?
A copy constructor that performs a deep copy (allocates new memory for pointers).
* **Use case:** When your class handles dynamic memory (pointers like `int* ptr = new int;`). If you don't use a deep copy constructor, the default copy constructor will cause two objects to point to the same memory (Double Free Error).

## 22. What is the purpose of the explicit keyword in C++?
It prevents the compiler from using constructors for **implicit type conversions**.
* *Example:* `explicit MyClass(int x);` prevents `MyClass obj = 10;`. You must write `MyClass obj(10);`.

## 23. Explain RAII (Resource Acquisition Is Initialization).
A programming idiom where resource management (memory, file handles, locks) is tied to the lifecycle of an object.
* **Acquisition:** Resource is acquired in the constructor.
* **Release:** Resource is released in the destructor.
* *Advantage:* Prevents memory leaks automatically when the object goes out of scope. (Basis of Smart Pointers).

## 24. What is dynamic_cast in C++? How does it work?
A type cast used for safe downcasting (converting a base pointer to a derived pointer) at runtime.
* It works only with polymorphic classes (classes with at least one virtual function).
* If the cast fails, it returns `nullptr` (for pointers) or throws an exception (for references).

## 25. What is an "Interface"?
C++ does not have a specific `interface` keyword (unlike Java). An interface in C++ is simulated using an **Abstract Class** where:
1. All methods are pure virtual functions.
2. There are no data members.

## 26. What are Virtual Destructors and why are they important?
A destructor in the base class declared with `virtual`.
* **Importance:** If you delete a derived class object through a base class pointer (`Base* b = new Derived; delete b;`), a non-virtual base destructor will *only* call the base destructor, leaking the derived part's memory. A virtual destructor ensures *both* are called.

## 27. What is the difference between an "abstract class" and an "interface"?
* **Abstract Class:** Can have some implemented methods and member variables. Used for partial abstraction.
* **Interface (Pure Abstract Class):** Has *only* pure virtual functions and no data members. Used to define a strict contract.

## 28. What is Multiple Inheritance and how does C++ handle ambiguity?
(See Question 10). C++ handles ambiguity via **Virtual Inheritance** or explicitly using the Scope Resolution Operator (`obj.Base1::func()`).

## 29. What is a "Singleton" class in C++?
A design pattern that ensures a class has only **one instance** and provides a global point of access to it.
* **Key steps:**
  1. Private Constructor (prevents creating new objects).
  2. Deleted Copy Constructor (prevents cloning).
  3. Static Method to return the single instance.

```cpp
class Singleton {
    // Private constructor
    Singleton() {} 
public:
    // Meyers Singleton (Thread-safe in C++11)
    static Singleton& getInstance() {
        static Singleton instance; 
        return instance;
    }
    // Delete copy/move
    Singleton(const Singleton&) = delete; 
    void operator=(const Singleton&) = delete;
};
