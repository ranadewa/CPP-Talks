# CPP-Talks
Repository to note important points on C++ talks
- [CPP-Talks](#cpp-talks)
- [2020](#2020)
  - [Exceptions](#exceptions)
      - [How they work](#how-they-work)
      - [When to use exceptions](#when-to-use-exceptions)
      - [When not to use them](#when-not-to-use-them)
      - [How to use exceptions properly](#how-to-use-exceptions-properly)
      - [Exception Safety Guarantees](#exception-safety-guarantees)
      - [Functions that should not fail](#functions-that-should-not-fail)
  - [Concurrency](#concurrency)
      - [Thread safe Static initialisation](#thread-safe-static-initialisation)
      - [Initialize a member with once_flag](#initialize-a-member-with-once_flag)
      - [Comparison of C++11’s primitives](#comparison-of-c11s-primitives)
      - [C++17 shared_mutex (R/W lock)](#c17-shared_mutex-rw-lock)
      - [Synchronization with std::latch C++ 20](#synchronization-with-stdlatch-c-20)
      - [Synchronization with std::barrier C++ 20](#synchronization-with-stdbarrier-c-20)
      - [Comparison of C++20’s primitives](#comparison-of-c20s-primitives)
  - [Breaking Dependencies: The SOLID Principles](#breaking-dependencies-the-solid-principles)
      - [Single Responsibility](#single-responsibility)
      - [Open Close Principle](#open-close-principle)
      - [Liscov's Substitution Principal](#liscovs-substitution-principal)
      - [Interface Segregation](#interface-segregation)
      - [Dependency Inversion Principal](#dependency-inversion-principal)
      - [Summary](#summary)
  - [Embrace No Paradigm Programming](#embrace-no-paradigm-programming)
- [2019](#2019)
  - [Back to Basics: Smart Pointers](#back-to-basics-smart-pointers)
  - [Move Semantics](#move-semantics)

# 2020
## Exceptions
#### How they work
* Key words
  ```C++ 
  void f(){
      throw std::runtime_error();
  }

  try {
      f();
  } catch(std::exception const& ex)
  {

  }
  ```
*  Stack unwinding 
   *  Objects on the stack are destroyed in the reverse order they were created.
* Unhandle exceptions result in std::terminate getting called.

#### When to use exceptions
* For errors that are expected to happen rarely.
* For exceptional cases that cannot be handled locally. (I/O errors)
  *   File not found.
  *   Map key not found.
* For operators and constructors (i.e. When no other mechanism works)
#### When not to use them
* For errors that are expected to happen frequently.
* For functions that are excpeted to fail. (to_int(string))
* If you have to guarantee certain response times.
* For things that should never happen. (Assert)
  * Dereference nullptr
  * out of range access
  * Use after free
#### How to use exceptions properly
* Build on the std::exception Hierarchy
* Throw by Rvalue
    ```C++ 
    throw std::exception();
    ```
* Catch by reference
#### Exception Safety Guarantees
* Exception unsafe
  * No guarantees.
* Basic safety guarantee
  * Invariants are preserved.
  * Resources are not leaked.
  * Internal state might be changed.
* Strong Exception Safety Guarantee
  * Basic safety guarantees.
  * No state change. (commit or rollback)
  * Not always possible.
* No-Throw Guarantee
  * Operation cannot fail.
  * Expressed in the code with ```noexcept```
  
#### Functions that should not fail
* Destructors
* Move operations
* Swap Operations

## [Concurrency](https://www.youtube.com/watch?v=F6Ipn7gCOsY&list=PLHTh1InhhwT5o3GwbFYy3sR7HDNRA353e&index=14)
#### Thread safe Static initialisation
```C++ 
inline auto& SingletonFoo::getInstance() {
  static SingletonFoo instance;return instance;
  }
```
* The first thread to arrive will start initializing the static instance.Any more that arrive will block and wait until the first thread either succeeds (unblocking them all) or fails with an exception (unblocking one of them)
#### Initialize a member with once_flag
```C++ 
class Logger {    
  std::once_flag once_;
  std::optional<NetworkConnection> conn_; 
  
  NetworkConnection& getConn() {
    std::call_once(once_, []() {
      conn_ = NetworkConnection(defaultHost);
            });
    return *conn_;    }};
```
* Here, the first access to conn_ is protected by a once_flag.This mimics how C++ does static initialization, but for a non-static. Each Logger has its own conn_, protected by its own once_.

#### Comparison of C++11’s primitives

<img src="images/c11_syc_prim.png" width="600"/>

#### C++17 shared_mutex (R/W lock)
```C++ 
class ThreadSafeConfig
{
    std::map<std::string, int> settings_;
    mutable std::shared_mutex rw_;
    void set(const std::string &name, int value)
    {
        std::unique_lock<std::shared_mutex> lk(rw_);
        settings_.insert_or_assign(name, value);
    }
    int get(const std::string &name) const
    {
        std::shared_lock<std::shared_mutex> lk(rw_);
        return settings_.at(name);
    }
};
```
* unique_lock calls rw_.lock() in its constructor and rw_.unlock() in its destructor.shared_lock calls rw_.lock_shared()in its constructor and rw_.unlock_shared()in its destructor.

#### Synchronization with std::latch C++ 20
  ```C++ 
  std::latch myLatch(2);
  std::thread threadB = std::thread([&]() {
    myLatch.arrive_and_wait();  
    printf("Hello from B\n");
  });
  printf("Hello from A\n");
  myLatch.arrive_and_wait();
  threadB.join();
  printf("Hello again from A\n");
  ```
#### Synchronization with std::barrier C++ 20
  ```C++ 
  std::barrier b(2, [] { puts("Green flag, go!"); });
  std::thread threadB = std::thread([&]() {        
    printf("B is setting up\n");        
    b.arrive_and_wait();
    printf("B is running\n");
  });
  printf("A is setting up\n");
  b.arrive_and_wait();
  printf("A is running\n");
  threadB.join();
  ```
  #### Comparison of C++20’s primitives
  <img src="images/c20_sync_prim.png" width="600"/>
<br></br>
<br></br>

## [Breaking Dependencies: The SOLID Principles](https://www.youtube.com/watch?v=Ntraj80qN2k)
* **S**ingle Responsibility
* **O**pen Close 
* **L**iscove's Substitution
* **I**nterface Segregation
* **D**ependency Inversion

#### Single Responsibility
* Orthogonality: Software Components (functions, classes, modules) should be self contained, with a single, well defined purpose. One should be able to change without worrying about the others.
* Cohesion measures the strength of association inside a module. A highly cohesive module is a collection of statements and data that should be treated as a whole.

| BAD | Good|
|-----|-----|
| <img src="images/SRP_BAD.png" width="500"/> |  <img src="images/SRP_GOOD.png" width="800"/> |
* Guideline:  Prefer  cohesive  software  entities.  Everything  that  does  not strictly belong together, should be separated

#### Open Close Principle
* Software should be open for extension but closed for modification.
* Creating an object hierarchy based on types and switching between them violates open close principle. Creating an object heirarchy based on operations (virtual functions) can get rid of this but violates SRP.
* Guideline:  Prefer  software  design  that  allows  the  addition  of  types  or operations without the need to modify existing code
#### Liscov's Substitution Principal
* Subtypes must be substitutable for thier base type. 
* If you inherit from a base class make sure you keep the contracts of the base types. You guarantee that the behavior of base type is not broken.
  
Behavioral subtyping (aka “IS-A” relationship)
 * Contravariance of method arguments in a subtype 
 * Covariance of return types in a subtype 
 * Preconditions cannot be strengthened in a subtype 
 * Postconditions cannot be weakened in a subtype 
 * Invariants of the super type must be preserved in a subtype
#### Interface Segregation 
* Many client specific interfaces are better than one general-purpose interface
* Stratergy Pattern
<img src="images/Stratergy_Pattern.png" width="500"/>

```C++
class Circle;
class Square;
class DrawCircleStrategy
{
public:
    virtual ~DrawCircleStrategy() {}
    virtual void draw(const Circle &circle) const = 0;
};
class DrawSquareStrategy
{
public:
    virtual ~DrawSquareStrategy() {}
    virtual void draw(const Square &square) const = 0;
};

```

#### Dependency Inversion Principal
* The Dependency Inversion Principle (DIP) tells us that the most flexible systems are those in which source code dependencies refer only to abstractions, not to concretions.
  * High-level modules should not depend on low-level modules. Both should depend on abstractions.
  * Abstractions should not depend on details. Details should depend on abstractions.

  <img src="images/Dependency_inversion.png" width="600"/>

* Model Veiw controller
  <img src="images/MVC.png" width="600"/>

* Prefer  to  depend  on  abstractions  (i.e.  abstract  classes  or concepts) instead of concrete types

#### Summary
* The SOLID principles are more than just a set of OO guidelines
*  Use the SOLID principles to reduce coupling and facilitate change
   *  Separate concerns via the SRP to isolate changes
   *   Design by OCP to simplify additions/extensions
   *   Adhere to the LSP when using abstractions 
   *   Minimize the dependencies of interfaces via the ISP
   *    Introduce abstractions to steer dependencies (DIP)

## [Embrace No Paradigm Programming](https://www.youtube.com/watch?v=fwXaRH5ffJM)

# 2019
## [Back to Basics: Smart Pointers](https://www.youtube.com/watch?v=xGDLkt-jBJ4)

## Move Semantics
