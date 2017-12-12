# Final Study

Trivia and a lot of examples, mostly from tutorials

# Table of Contents
1. [References](#ref)
2. [The Big Five](#big)
3. [Invariants/Encapsulation](#inv)
4. [Relationships](#relationships)
   - [Types of Relationships](#types)
   - [Inheritance](#inherit)
   - [Copy/Move](#copy)
5. [Abstraction](#abstract)
   - [Destructor](#destruct)
6. [Templates](#templates)
7. [Design](#design)
   - [Abstract Iterator Pattern](#aip)
   - [Observer Pattern](#op)
   - [Factory Design Pattern](#fdp)
   - [Template Method Pattern](#tmp)
   - [Visitor Pattern](#vp)
   - [Decorator Pattern](#dec)
   - [pImpl Idiom](#pimpl)
8. [Measure of Design Quality](#measure)
9. [Compilation](#compilation)
10. [Resource Aquisition is Initialization (RAII)](#raii)
    - [Smart Pointers](#shared)
11. [Exception Safety](#exception)
12. [Casting](#casting)
13. [How Virtual Methods Work](#vmethod)

## References <a name="ref"></a>
References are like constant pointers with automatic dereferencing.
```cpp
int y = 10;
int &z = y; // Called an lvalue reference to int (to y)
```

##### Things you can't do with lvalue References
1. can't leave them unitialized (e.g. `int &x;` ❌)
2. they must be initialized with something which has an address since references are Pointers
   * `int &x = 4;` ❌
   * `int &x = y + z;` ❌
   * `int &x = y;` ✔️
3. create a pointer to a reference
   * `int &*x = .....;` ❌
   * reference to a pointer `int *&x = .....;` ✔️
4. create a reference to a reference
   * `int &&r = ......;` ❌
5. create an array of references
   * `int &r[3] = {.., .., ..}`

### Lvalues and Rvalues
* an lvalue is any entity which has an address
* an rvalue is anything which is not an lvalue
  * get their name because an rvalue can only occur on the right side of an assignment expression
  * an rvalue reference itself is considered as an lvalue
* use std::move() to convert any value to an rvalue
  * example [here](#move)

## The Big Five <a name="big"></a>

##### Steps for object creation
1. space is allocated (stack/heap)
2. default initialization of fields that are objects
3. constructor body runs

##### MIL is used to 'hijack' step 2
```cpp
struct Student {
  int assns, mt, final;
  const int id; // ya
  Student (int assn, int mt, int final, int id): assn(assn), mt(mt), final(final), id(id) {} // can only initialize constant here
}
```
```cpp
Node plusOne(Node n) {
  for (Node *p{&n}; p; p = p->next) ++p->data;
  return n;
}

Node n{1, new Node {2, nullptr}};
Node n2{plusOne(n)};
```
* Theoretically 2 basic constructor calls and 6 copy constructor calls
* However, only 2 constructor calls and 4 copy constructor calls due to *copy/move elision*

```cpp
struct Node {
  Node(); // default constructor
  ~Node(); // destructor
  Node(const Node &); // copy constructor
  Node(Node &&); // move constructor
  Node &operator=(const Node &); // copy assignment
  Node &operator=(const Node &&); // move assignment
};

// copy constructor
Node::Node(const Node &other): data(other.data), next(other.next ? new Node(*other.next) : nullptr) {}

// move constructor
Node::Node(const Node &&other): data(other.data), next(other.next) {
  other.next = nullptr;
}

// copy assignment
Node &Node::operator=(const Node &other) {
  if (this == &other) return *this;
  data = other.data;
  delete next;
  next = other.next ? new Node(*other.next) : nullptr;
  return *this;
}

// move assignment
Node &Node::operator=(Node &&other) {
  swap(data, other.data);
  swap(next, other.next);
  return *this;
}

// OR move assignment pt2:
Node &Node::operator=(Node &&other) {
  data = other.data;
  delete next;
  next = other.next;
  other.next = nullptr;
  return \*this;
}

// destructor
Node::~Node() {
  delete next;
}
```

##### about destructors
  * stack-allocated: called when it goes out of scope
  * heap-allocated: is deleted
  * specifically:
     1. the destructor body runs
     2. the destructor is called on all the fields (in reverse declaration order)
     3. space is deallocated

If you need to customize any one of:
  1. copy constructors
  2. copy assignment
  3. destructors
  4. move ctor
  5. move assignment

Then you *usually* need to customize all 5.

## Invariants/Encapsulation <a name="inv"></a>
```cpp
struct Node {
  int data;
  Node *next;
  Node (int data, Node *next); // fill in urself
  ...
  ~Node() { delete next; }
};

Node n1 {1, new Node {2, nullptr}};
Node n2 {3, nullptr};
Node n3 {4, &n2};
```

##### what happens when n1, n2, n3 go out of scope?
* n1, destructor runs and then deletes the whole list
* n2 and n3, n3's destructor runs and then tries to delete n2, but n2 is on the *stack* not the *heap* (undefined behavior)
* node relies on the assumption that `next` is either a `nullptr` or a valid pointer to the heap. this is an ***invariant***
  * e.g. stack: invariant is that the last item pushed is the first item popped

##### Enforcing invariants - Encapsulation
  * create a wrapper class for the Nodes of a linked list

```cpp
// list.h
class List {
  struct Node; // private nested class
  Node *theList = nullptr;
public:
  void addToFront(int n);
  int ith(int i);
  ~List();
};

// list.cc
struct List::Node {
  int data;
  Node *next;
  Node(int data, Node * next): ....
  ~Node() { delete next; }
};

List::~List(){ delete theList; }
List::addToFront(int n) { theList = new Node(n, theList); }
int List::ith(int i) {
  Node *curr = theList;
  for (int j = 0; j < i && curr; curr = curr->next, ++j);
  return curr->data;
}
```

## Relationships <a name="relationships"></a>

##### Three main types of relationships: <a name="types"></a>
1. composition (***owns-a***)
   * `class A` owns an instance of `class B`
   * `class A` is responsible for deleting the instance of `class B` when `class A` is destroyed
   * car owns 4 wheels; destroy a car, destroy the wheels; copy a car, copy the wheels
2. aggregation (***has-a***)
   * `class A` has an instance of `class B`
   * `class A` is not responsible for deleting the instance of `class B`
3. inheritance (***is-a***)
   * `class B` is an instance of `class A`
   * an instance of `class B` can be used in any situation where an instance of `class A` can be used


If `class A` has a pointer to an instance of `class B`, you don't know if its composition of aggregation without looking at the source code:
```cpp
B b;   // This is composition
B* b2; // This could be composition or aggregation
```

## Inheritance <a name="inherit"></a>
##### e.g. tracking collection of books:

```cpp
class Book {
  string title, author;
  int numPages;
public:
  Book (....);
};

class Text:public Book {
  string topic;
public:
  Text (.....);
};

class Comic:public Book {
  string hero;
public:
  Comic (.....);
};
```

##### How do we initialize `Text`?
* need title, author, numpages (for book part) and topic (specific to text)
* soln: ***invoke Book's constructor in Text's MIL***

```cpp
class Text : public Book {
  string topic;
public:
  Text(string title, string author, int numPages, string topic):
  Book{title, author, numPages}, topic{topic} {}
};
```

##### Good design habits
* not a good idea to give subclasses unlimited access to fields; better - make fields private, but provide protected accessors

```cpp
class Book {
  string author, title;
  int numPages;
protected:
  string getTitle() const;
  void setAuthor(string newAuthor);
public:
  Book(...); // ctor
  bool isItHeavy() const;
};
```

* how to make a comic act like a comic, even when pointed at by a book pointer?
  * soln: declare the certain method ***virtual***

```cpp
class Book {
  // ... fields
protected:
  int numPages;
public:
  Book(...);
  virtual bool isItHeavy() const; // use of virtual here
};

class Comic : public Book {
  // ...
public:
  bool isItHeavy() const override; // override keyword in virtual function
};

// =================
// client
Comic c {"RealisticMathStudent", "UWGo", 40, "Quest God"};
Book *pb = &c;
Book *rb = c;
Comic &pc = &c;
Book b = c;

cout << pb->isItHeavy(); // true, Comic::isItHeavy
cout << rb.isItHeavy(); // true, Comic::isItHeavy
cout << pc->isItHeavy(); // true, Comic::isItHeavy
cout << b.isItHeavy(); // FALSE, Book::isItHeavy
```

#### Arrays and Inheritance
Consider:
```cpp
void foo(A* arr) {
  arr[0] = A{10};
  arr[1] = A{7};
}
B arr[2] = {{1,2}, {3,4}};
foo(arr);
```

what happens with this code?
* compiles perfectly types match
* but `foo` believes the array it receive is an array of `A`'s
* so when we assign a value to `arr[1]`, the value 7 will be actually assigned to the location where 2 is stored
* this means our data is misaligned and this is dangerous

##### Never use array objects polymorphically. If you want a polymorphic array, use an array of pointers

### Copy/Move <a name="copy"></a>
Consider:
```cpp
class Book {
  ...
public:
  // Defines a copy/move constructor and assignment
};

class Text:public Book {
  string topic;
public:
  // Does not define copy/move
};

Text t{"algorithms", "clrs", 50000, "cs"};
Text t2 = t;
```

##### Question: no copy ctor in Text - what happens?
* calls Book's copy ctor
* goes field by field i.e. default behavior for the Text part
* same is true for the other operations

<a name="move" />

```cpp
Text::Text(const Text &other): Book(other), topic(other.topic) {}

Text &Text::operator=(const Text &other) {
  Book::operator=(other);
  topic = other.topic;
  return * this;
}

Text::Text(Text &&other): Book(other), topic(other.topic) {} ❌
// other refers to an rvalue, but IS AN LVALUE
// so book COPY CTOR IS RUNNING not MOVE CTOR

// std::move(other) means treat this like garbage
Text::Text(Text &&other): Book(std::move(other)), topic(std::move(other.topic)) {}
```

***Even though `other` and `other.top` refer to rvalues, they themselves are lvalues***

`std::move(x)` forces an lvalue x to be treated as an rvalue so that the "move" versions of the operations run

```cpp
// same kinda deal for move assignment
Text &Text::operator=(Text &&other) {
  Book::operator=(std::move(other));
  topic = std::move(other.topic);
  return * this;
}
```

Consider:

```cpp
Text t1 {...};
Text t2 {...};
Book *pb1 = &t1, *pb2 = &t2;
// What if we do:
*pb1 = *pb2;
```

* it is Book::operator= that runs - copies only the book part ❌

##### Soln: all superclasses should be abstract:

```cpp
class AbstractBook {
  string title, author;
  int numpages;
protected: // ❗
  AbstractBook &operator=(const AbstractBook &other);
public:
  AbstractBook( ... );
  virtual ~AbstractBook() = 0;
};
```
```cpp
class NormalBook:public AbstractBook {
public:
  NormalBook(....);
  ~NormalBook();
  NormalBook &operator=(const NormalBook &other) {
    AbstractBook::operator=(other); // copy the abstractbook part
    // no other fields, nothing left to do
    return * this;
  }
}; // other classes are similar, except with extra fields
```

*Don't forget to implement the virtual destructor*

## Abstract <a name="abstract"></a>
* a class is an abstract class if it has ***one or more pure virtual methods***
* a class is a concrete class if is has ***no pure virtual methods***
* the purpose of an abstract class is to allow subclasses to inherit from a base class containing the info that is common
* you can not create objects of abstract classes
* a subclass of an abstract class is, by default, also abstract, unless the subclass overrides ***all*** pure virtual methods of the base class


### Destructor <a name="destruct"></a>
1. your destructors should **always be virtual**
   * to ensure the right destructor is called when polymorphism is involved
2. your destructors must always have an implementation, even if they are virtual
   * this is because the destructor of the base class is called even when a derived class is destroyed
   * thus, the destructor of the base class must have some implementation (even if the implementation is empty)


```cpp
class B {
public:
  virtual ~B() = 0;             // these two methods are pure virtual
  virtual string hello() = 0;   // making B an abstract class
};

class A: public B {
  ... // inherits all fields from B
  A* arr;
public:
  A(): arr{new A[5]} {}
  ~A() { delete [] arr; }
  string hello() override { return "hello"; }
};

// In B.cc
B::~B() {} // need this..
```

## Templates <a name="templates"></a>
* important example - ***stack***

```cpp
template <typename T> class Stack {
  int size, cap;
  T * the stack;
public:
  Stack(...);
  void push(T x);
  T top();
  void pop();
};

// Client:
  List <int> l1;
  List <List <int>> l2;
  l1.addToFront(3);
  l2.addToFront(l1);

// Looping:
  for (List<int>:: Iterator it=l1.begin(); it != l1.end(); ++it) {
    cout << * it << endl;
  }

  for (auto n : l1)
    cout << n << endl;
```

Compiler specializes templates at the source code level, before compilation. I.e. the compiler writes the new classes for you.

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
### Observer Pattern <a name="op"></a>
* follows the pubsub model, one publisher/subject generates data, one or more subscriber/observer classes receive and react to it
* subjects should not need to know all the details of the observers
```
*Subject* o-----------> *Observer*
 ________________        __________
 +notifyObservers        +*notify()*
 +attach(Observer)           ^
 +detach(Observer)           |
       ^                     |
       |                     |
       |                     |
 ConcreteSubject <-----o ConcreteObserver
 ________________        ________________
 +getState()             +notify()
 +setState(?)
```

##### Order of calls
1. subject state changes
2. `subject::notifyObservers()` called either by subject itself or external Controller
   * calls each observer's `notify`
3. each observer calls `ConcreteSubject::getState` to get new state and reacts appropriately

##### Example: horse race

```cpp
class Subject {
  vector <Observer*> observers;
public:
  void attach(Observer * ob) {
    observers.emplace_back(ob);
  }
  void detach(Observer * ob) {
    // remove it from the vector
  }
  void notifyObservers() {
    for (auto &ob: observers) ab->notify();
  }
  virtual ~Subject() = 0;// still have to implement it!
};

// somewhere in cc....
Subject::~Subject() {}
// ...................

class Observer {
public:
  virtual void notify() = 0;
  virtual ~Observer();
};

class HorseRace:public Subject {
  ifstream in; // source of winners
  string lastwinner;
public:
  HorseRace(string source): in(source) {}
  bool runRace() {
    return in >> lastwinner ? true : false; // true means there was a race, false means out of races
  }
  string getState() {
    return lastwinner;
  }
};

class Bettor:public Observer {
  HorseRace * subject;
  string name, myHorse;
public:
  Bettor(  ): ..... {
    subject->attach(this);
  }
  ~Bettor() {
    subject->detach(this);
  }
  void notify() {
    string winner = subject->getState();
    if (winner == myHose) {
      cout << "win!" << endl;
    } else {
      cout << "lose :(" << endl;
    }
  }
};

// main.cc
int main() {
  HorseRace hr{"file.txt"};
  Bettor Larry{&hr, "Larry", "RunsLikeACow"};
  ... // other bettors

  while (hr.runRace()) {
    hr.notifyObservers();
  }
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
### Decorator Pattern <a name="dec"></a>
* we want to enhance an object ***at runtime***, i.e. add functionality/features

```
                | PIZZA |<------------
                /       \            |
| CRUST AND SAUCE |     | DECORATOR |o
                            ^
                            |
                --------------------------
                |           |            |
| Stuffed Crust |   | Topping |     | Dipping Sauce |
```

* every decorator is a component, and every decorator has a component ⭐
* eg:

```cpp
class Pizza {
public:
  virtual float price() const = 0;
  virtual string desc() const = 0;
  virtual ~Pizza();
};

class CrustAndSauce:public Pizza {
public:
  float price() const override { return 5.99; }
  string desc() const { return "Pizza"}
};

class Decorator:public Pizza {
protected:
  pizza * component;
public:
  Decorator(Pizza * p): component(p) {}
  ~Decorator() { delete component; }
};

class StuffedCrust:public Decorator {
public:
  StuffedCrust(Pizza * p): Decorator(p) {}
  float price() const override { return component->price() + 2.69; }
  string desc() const override { return component->desc() + " with stuffed crust"; }
};

class Topping:public Decorator {
  string theTopping;
public:
  Topping(string topping, Pizza * p): Decorator(p), topping(topping) {}
  float price() const override { return component->price() + 0.75; }
  string desc() const override { return component_>desc() + " with " + theTopping; }
};
```
```cpp
// USE

Pizza * p1 = new CrustAndSauce;
p1 = new Topping("Cheese", p1);
p1 = new Topping("Mushrooms", p1);
p1 = new StuffedCrust(p1);

cout << p1.desc() << ' ' << p1.price();
delete p1;
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
* instead of using C-style strategies of handling errors, we use exceptions in C++ to control the behaviour of the program when errors arise
* by default, if an exception is raised/thrown, the program will terminate if there is no handler (read: catch block) for it
* the try and catch blocks come with each other. You can not have one without the other.
* we can throw anything, including exception classes, strings, integers, etc.
* unless it’s a primative type, we always catch by reference.
  * if we weren’t passing objects by reference in the catch parameter, slicing could occur; potentially resulting in missing information.
* ***throw by value, catch by reference***


```cpp
class BadInput{};

class BadNumber: public BadInput{
  string what() { return "no number given" }
};

int main(){
  try{
    cin >> x;
    if (x < 50){
      throw BadNumber{};
    }
  }
  catch(BadInput& b){}
  //accessing auxilliary information
  //from object that was caught
  catch(BadNumber& b){ cerr << b.what() << endl; }
}
```


##### Three levels of guarantee that you can provide with respect to exception safety:
1. basic guarantee
   * if an exception is thrown, data will be in a valid state and all class invariants are maintained
   * ex. if we change variables in an assignment operator before allocating heap memory with new
2. strong guarantee
   * if an exception is thrown, the data will appear as if nothing happened
   * ex. the copy and swap idiom for assignment operator
3. no-throw guarantee
   * an exception is never thrown, the function must always succeed

## Casting  <a name="casting"></a>
4 kinds of casting:

`static_cast` - "sensible cast"

```cpp
Book *b = new Text {...};
Text *t = static_cast<Text*>(b);
```

 * you are taking responsibility that b actually points at a `Text`


`reinterpret_cast` - unsafe, "weird" conversions

```cpp
Student s;
Turtle *t = reinterpret_cast<Turtle*>(&s);
```

`const_cast` - for converting between const and non-const, only c++ cast that can "cast away const"

```cpp
void g(int *p); // g doesn't promise not the change *p
// but suppose g doesn't actually change *p (assumption)

void f(const int *p) { // f promises not to change *p
  // ...
  g(p); // WRONG
  g(const_cast<int*>p); // CORRECT
  // casting the constness away
}
```

`dynamic_cast` - is it safe to convert a `Book *` to a `Text *`??

```cpp
Book *pb = ...; // don't know the right side
static_cast<Text*>(*pb)->getTopic(); // safe?
```
Is this safe? Use a tentative cast instead:
```cpp
Book *pb = ...;
Text *pt = dynamic_cast<Text*>(pb);
```

If the cast works(i.e. pb points at a Text or a subclass of Text), then pt points at the object. If the cast fails, pt will be a nullptr. So we can check if the cast is successful by:

```cpp
Book *pb = ...;
Text *pt = dynamic_cast<Text*>(pb);

if (pt) {
  cout << pt->getTopic();
} else {
  cout << "Not a Text";
}
```

Also works with references:
```cpp
Text { ... };
Book &b = t;
Text &t2 = dynamic_cast<Text &>(b);
```
If the cast succeeds, `t2` refers to `t`. If not, raises `bad_cast`.

Can also solve the polymorphic assignment problems.
```cpp
Text &Text::operator=(const Book * other) { // virtual
  const Text &textother = dynamic_cast<const Text &>(other); // throws if other is not a text
  if (this == &textother) return * this;
  Book::operator=(other);
  topic = textother.topic;
  return this;
}
```

*Note: `dynamic_cast`ing only works on classes that have at least one virtual method*

## How Virtual Methods Work  <a name="vmethod"></a>
```cpp
class Vec {
  int x, y;
  void f();
};

class Vec2 {
  int x, y;
  virtual void f();
};

// what's the difference?
Vec v{1, 2};
Vec2 w{1, 2};

// do they look the same in memory
```

For each class with at least one virtual method, the compiler creates a table of f'n pointers (the vtable)

Thus, `w` has 8 extra bytes (pointer to the vtable).
