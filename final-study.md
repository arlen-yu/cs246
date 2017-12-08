# Final Study

Trivia and a lot of examples, mostly from tutorials

# Table of Contents
1. [Abstract](#abstract)
2. [Design](#design)
   - [Abstract Iterator Pattern](#aip)
   - [Factory Design Pattern](#fdp)
   - [Template Method Pattern](#tmp)
   - [Visitor Pattern](#vp)
   - [pImpl Idiom](#pimpl)
3. [Measure of Design Quality](#measure)
4. [Compilation](#compilation)
5. [Resource Aquisition is Initialization (RAII)](#raii)
   - [Smart Pointers](#shared)
6. [Exception Safety](#exception)

## Abstract <a name="abstract"></a>
* a class is an abstract class if it has one or more pure virtual methods
* you can not create objects of abstract classes
* a subclass of an abstract class is, by default, also abstract, unless the subclass overrides ***all*** pure virtual methods of the base class
* a virtual destructor must be implemented even though it is pure virtual
  * this is because the destructor of the base class is called even when a derived class is destroyed
  * thus, the destructor of the base class must have some implementation (even if the implementation is empty)

## Design Patterns <a name="design"></a>
### Abstract Iterator Pattern <a name="aip"></a>
* used if we want to iterate through a collection of objects in a way that doesn't break encapsulation
* want a unified interface
* solution: create an abstract iterator class that defines the interface of iterators, all concrete iterators inherit from this class

```cpp
template<typename T>
class AbstractIterator {
 public:
  //dereference operation
  virtual T &operator*() const = 0;
  //prefix ++
  virtual AbstractIterator<T> &operator++() = 0;

  // NON-virtual != operator defined in terms of == operator
  bool operator!=(const AbstractIterator<T> &other) const{
    return !(*this == other);
  }

  // pure virtual == operator. Subclasses must implement,
  // otherwise abstract
  virtual bool operator==(const AbstractIterator<T> &other) const = 0;

  //virtual destructor to prevent any memory leaks
  virtual ~AbstractIterator() = default;
};
```
##### An example list iterator:

```cpp
// .h
#include "absiter.h"
class List {
  class Node;
  Node *theList = nullptr;

 public:
  class Iterator : public AbstractIterator<int> {
    Node *p;
    explicit Iterator(Node *p);
   public:
    int &operator*() const override;
    Iterator &operator++() override;
    bool operator==(const AbstractIterator<int> &other) const override;
    friend class List;
  };

  Iterator begin();
  Iterator end();

  void addToFront(int n);
  int ith(int i);
  ~List();
};
```

```cpp
// .cc
#include "listiter.h"
struct List::Node {
  int data;
  Node *next;

  Node (int data, Node *next): data{data}, next{next} {}
  ~Node() { delete next; }
};

List::Iterator::Iterator(Node *p): p{p} {}
int &List::Iterator::operator*() const { return p->data; }
List::Iterator &List::Iterator::operator++() {
  p = p -> next;
  return *this;
}
//Override operator== requires RHS to be AbstractIterator
//Will lead to mixed assignment unless we restrict RHS to List::Iterator
bool List::Iterator::operator==(const AbstractIterator<int> &other) const {
  //RHS must also be List::Iterator
  const List::Iterator &rhs = dynamic_cast<const List::Iterator &>(other);
  return p == rhs.p;
}

List::Iterator List::begin() { return Iterator(theList); }
List::Iterator List::end() {return Iterator(nullptr); }

List::~List() { delete theList; }

void List::addToFront(int n) { theList = new Node(n, theList); }

int List::ith(int i) {
  Node *cur = theList;
  for (int j = 0; j < i && cur; ++j, cur = cur -> next);
  return cur->data;
}
```

### Factory Design Pattern <a name="fdp"></a>
* want to be able to create instances of a subclasses based on criteria, ***at runtime***
  * e.g. based on input from user, or from probabilities
  * need to be able to easily change these criteria
* soln: create a class that creates instances of subclasses, must be abstract in order to switch the criteria

```cpp
// pizzaFactory.h
#include "pizza.h"

class PizzaFactory{
 public:
	// Create a pizza based on input:
	Pizza* makePizza();
};
```

```cpp
// pizzaFactory.cc
#include "pizzaFactory.h"
#include "crustandsauce.h"
#include "stuffedcrust.h"
#include "dippingsauce.h"
#include "topping.h"

#include <iostream>
#include <sstream>

using namespace std;

// This factory creates pizzas based on input:
Pizza* PizzaFactory::makePizza(){
	string descript;
	getline(cin, descript);

	istringstream ss{descript};

	Pizza* pizza = new CrustAndSauce;
	string topping;
	while ( ss >> topping ){
		if ( "stuffed" == topping ) pizza = new StuffedCrust(pizza);
		else if ( "ranch" == topping ) pizza = new DippingSauce(topping,pizza);
		else if ( "garlic" == topping ) pizza = new DippingSauce(topping,pizza);
		else if ( "cheese" == topping ) pizza = new Topping(topping, pizza);
		else if ( "pepperoni" == topping ) pizza = new Topping(topping, pizza);
		else if ( "mushroom" == topping ) pizza = new Topping(topping, pizza);
		else cerr << topping << " is not an available topping at this time" << endl;
	}
	return pizza;
}
```

### Template Method Pattern <a name="tmp"></a>
* we want to allow subclasses to have different implementations for some sections of a method, but enforcing a structure of the method and not allowing subclasses to have different implementations of other sections
* soln: implement an abstract superclass which has public non-virtual methods and private virtual methods

```cpp
class Turtle {
public:
  void draw() {
    drawHand();
    drawShell();
    drawFeet();
  }
private:
  void drawHead() { ... }
  void drawFeet() { ... }
  virtual void drawShell() = 0;
};

class RedTurtle:public Turtle {
  void drawShell() override { draw red shell }
}

class GreenTurtle:public Turtle {
  void drawShell() override { draw green shell }
}
```

### Visitor Pattern <a name="vp"></a>
* for selecting the right method to call based on two different hierarchies

```cpp
class Enemy {
public:
  virtual void beStruckBy(Weapon &w) = 0;
};

class Turtle:public Enemy {
public:
  void beStruckBy(Weapon &w) override { w.strike(* this); }
};

class Bullet:public Enemy {
public:
  void beStruckBy(Weapon &w) override { w.strike(* this); }
};

// Turtle and bullet MAY LOOK THE SAMe but they're calling different methods
// one version which takes a bullet, one that takes a turtle (overload)
class Weapon {
public:
  virtual void strike(Turtle &t) = 0;
  virtual void strike(Bullet &t) = 0;
};

class Stick:public Weapon {
public:
  void strike(Turtle &t) override {
    // strike a turtle with a stick
  }
  void strike(Bullet &b) override {
    // strike a bullet with a stick
  }
};

// similarly, rock.

Enemy * e = new Bullet (...);
Weapon * w = new Rock(...);

e->beStruckBy(*w); // what happens?
```

### pImpl <a name="pimpl"></a>
Consider:
```cpp
class GameGraphics {
  Window w;
  Controller c;
  ...
public:
  ...
};
```
* this exposes the fields to the user, we want to decouple the implementation with the interface
* soln: move all fields to a private implementation structure

```cpp
// GameGraphics.h
class GameGraphics {
  struct GameGraphicsImpl;
  GameGraphicsImpl * pImpl;
public:
  ...
};

// GameGraphics.cc
struct GameGraphics::GameGraphicsImpl {
  Window w;
  Controller c;
  ...
};
GameGraphics::GameGraphics(...):pImpl{ new GameGraphicsImpl{...} }{}
GameGraphics::~GameGraphics(){ delete pImpl; }
```

* thus, when the impls change, the header file doesn't have to, i.e. `GameGraphics.h` doesn't need to recompile

## Measure of Design Quality <a name="measure"></a>
##### In general, we want low coupling and high cohesion
### Coupling
* the degree to which distinct program modules depend on eachother
* from low to high:
  * modules communicating via function calls with basic params/results
  * modules pass arrays/structs back and forth
  * modules affect each other's control flow
  * modules have access to each other's implementation (friends)
* high coupling implies that the changes to a module require greater changes to other modules
* when we program to an interface, we exhibit low coupling
* when we program to an implementation, we exhibit low coupling

### Cohesion
* how closely elements of a module are related to each other
* low to high:
  * arbitrary grouping of unrelated elements (c++ utility)
  * elements sharing a common theme but otherwise unrelated (perhaps share some base code)
  * elements manipulate state over the lifetime of an object (e.g. open/read/close files)
  * elements pass data to eachother
  * elements cooperate to perform exactly one task

## Compilation  <a name="compilation"></a>
### Reducing Compilation Dependencies
```cpp
// consider
class A {...}; // a.h

class B:public A { ... };
class C {A a;};
class D {A*ap;};
class E{A f(A a);};
```
##### which of `B`, `C`, `D`, `E` require an include?
  * B needs `#include a.h`
  * `C` has `A` in it, needs to know how big A is, needs `#include a.h`
  * `D` only needs forward declaring, because pointers are all the same size (`class A;`)
  * `E` also doesn't need to know anything, so only needs `class A;`
  * when `class A` changes, only `A`, `B`, `C` need to recompile

##### In general, when to forward declare vs when to use class definition?
  * if we need to know something specific about the class, we need to have the class def
    * includes size, fields, methods
  * if we need to know that the class exists, we just forward declare
    * in the case we're using a pointer/reference to the class
  * *tutorial 10 has a good explanation of this*

##### Extra practice - what do we include and what do we forward declare?
```cpp
#include "A.h"
#include "B.h"
#include "C.h"
#include "D.h"
#include "E.h"
#include "F.h"
#include "G.h"
#include "H.h"
#include "I.h"
#include "J.h"
#include <iostream>

// class A;
// class B;
// class C;
// class D;
// class E;
// class F;
// class G;
// class H;
// class I;
// class J;


struct K: public A{
	B b;
	C* c;
	D& d;
	void foo(E);
	void foo(F*);
	void foo(G&);
	H func();
	I* func2();
	J& func3();

	K(B &, C &, D &);
};

std::ostream& operator<<(std::ostream&, const K&);
```

## Resource Aquisition is Initialization  <a name="raii"></a>
* every resource should be wrapped in a ***stack-allocated*** object whose dtor destroys it

```cpp
// e.g. files:
void h() {
  ifstream f("file"); // acquiring the resource, ("file")
  // initializing the object
  // ...
}
```
* under raii, resources are acquired using stack-allocated object initialization (i.e. through its constructor), so that the resource cannot be used before they are available and are “released” when the owning object is destroyed

Wrapper example from tut11:

```cpp
struct Wrapper {
  int * arr = nullptr;
  Wrapper(int length) {
    arr = new int[length];
  }
  ~Wrapper() {
    delete [] arr;
  }
};
```
```cpp
try {
  Wrapper w1(10), w2(20);
  ...
} catch (std::bad_alloc e) {
  ...
}
```

### Smart Pointers  <a name="shared"></a>
* solves the problem of implementing exception safety in dynamic memory
* example of the RAII
* there are wrapper classes provided in STL for pointers pointing to dynamic memory: unique_ptr, shared_ptr
  * `unique_ptr` means the only pointer that points to a block of heap memeory
    * `unique_ptr<Node> p = make_unique<Node>(2,nullptr);`
    * `unique_ptr`s are usually used to model composition relationship
  * `shared_ptr` aloows many pointers that all point to the same block of heap memory and only deletes that memory when no other `shared_ptr`s point to it
    * should only be used if the pointers are all sharing ownership; use unique pointers when there is a clear owner

```cpp
// A node for a doubly linked list
template <typename T> struct Node {
  T data;
  std::unique_ptr<Node <T>> next;
  // raw pointers are okay to use for modelling has-a relationship
  Node <T> * prev;
};
```
* when using smart pointers to manage heap-allocated objcts, use `std::make_shared` and `std::make_unique` rather than new


## Exception Safety  <a name="exception"></a>
##### Three levels of guarantee that you can provide with respect to exception safety:
1. basic guarantee
  * if an exception is thrown, data will be in a valid state and all class invariants are maintained
  * ex. if we change variables in an assignment operator before allocating heap memory with new
2. strong guarantee
  * if an exception is thrown, the data will appear as if nothing happened
  * ex. the copy and swap idiom for assignment operator
3. no-throw guarantee
  * an exception is never thrown, the function must always succeed
