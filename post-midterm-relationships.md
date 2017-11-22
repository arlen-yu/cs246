
# Static fields and methods

What if we want to track the number of times a method is ever called or the number of students created?

#### Static members - associated with the class itself, not with any specific instance (object)

```cpp
struct Student {
  ...
  static int numInstances;
  Student (....): ..... {
    ++numInstances;
  }
}
```

#### Static member functions:
  * don't depend on a specific instance of the class (no `this` param)
  * can only access static fields and call other static functions

```cpp
struct Student {
  ...
  static int NumInstances;
  ...
  static void printNumInstances() {
    cout << numInstances << endl;
  }
};

int Student::numInstances = 0;

Student billy{60, 70, 80};
Student jane{70, 80, 90};
Student::printNumInstances(); // prints 2
```

## System Modeling

*Building an OO system involves planning - identify abstractions and relationships amoung them*

Diagramming standard: UML (unified modeling language)

|name|Vec|
|-----|----|
|fields (optional):| x (Integer), y (Integer) |
|methods (optional):| getX: Integer, getY: Integer |

## Relationship: Composition of classes:
```cpp
class Vec {
  int x, y, x;
public:
  Vec(int x, int y, int z): x(x), y(y), z(z) {}
};

// Two vecs define a plane
class Plane {
  Vec v1, v2;
};

Plane p; // ❌ does not compile
         // can't initialize v1, v2 because theres no default ctor for vec
```

***INSTEAD:***
```cpp
class Plane {
  Vec v1, v2;
public:
  Plane(): v1{1, 0, 0}, v2{0,0,0} {}
}
```

Embedding one obj (Vec) inside another (Plane) is called ***composition***.

The relationship between Plane and Vec is called an "owns a" relationship. A plane *owns* two Vec objects.

##### If A owns a B, then *typically*:
  * B has not identity outside A (no independent existence)
  * If A is destroyed, then B is destroyed
  * If A is copied, then B is copied (deep copy)

##### An analogy:
  - car owns four wheels - a wheel is part of a car
  - destroy a car => destroy the wheels
  - copy a car => copy the wheels

##### Implementation: *usually* as composition of classes

Modelling:
  * `[Plane]*------>[Vec]` means Plane owns some number of Vec's
  * can annotate with multiplicities or field names idc

##### Aggregation

Compare car parts in a car ("owns a") vs car parts in a catalogue.
  * the catalogue contains the parts, but the parts have an independent existence
  * this is a "has a" relationship (***aggregation***)

If A "has a" B, then *typically*:
  * B has an existence apart from its association with A
  * If A is destroyed, B lives on
  * If A is copied, B is not (shallow copy) - copies of A share the same B

e.g. Ducks in a Pond

##### Typical implementation: pointer fields

```cpp
class Pond {
  Duck * ducks[maxDucks];
};
```

# Specialization: (Inheritance)
Suppose you want to track your collection of books:

```cpp
class Book {
  string title, author;
  int numPages;
public:
  Book (....)
};

// For textbooks we also want topic:
class Text {
  string title, author;
  int numPages;
  string topic;
public:
  Text(......)
};

// For comic books we also want hero:
class Comic {
  string title, author;
  int numPages;
  string hero;
public:
  Comic(...)
};
```

This is OK - does not capture the relationships among these classes. How do we create an array (or list) that contains a mix of these?

Could:
1. `union BookTypes { Book *b; Text *t; Comic *c; };`, access with `BookTypes myBooks[20];`
2. Array of `void*`
Neither one of these are type safe

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


******
# missed lecture [dzed](http://dzed.me/notes/2016/05/02/Cs-246.html) june 15/16th ish
******

### Destructor revisited

```cpp
class X {
  int * x;
public:
  X(int n)L x{new int[n]} {}
  ~X() { delete [] x; }
}

class Y:public X {
  int * y;
public:
  Y(int m, int n): X{n}, y{new int [m]} {}
  ~Y() { delete [] y; }
}
```

* should Y's destructor delete X?
  - no you shouldn't or you will double free
  - Y's destructor will call X's destructor

```cpp
X * myX = new Y{5, 10}; // Valid; "is a"
delete myX; // ❌ LEAKS ❌
```

* Y's destructor calls X's destructor, but will X's destructor call Y's destructor?
  - nope X doesn't even know that Y exists
* so `delete myX;` calls X's destructor but *not* Y's destructor, so only X is freed (leak!)
* how can we ensure that deletion through a superclass pointer will call the subclass destructor?
  * ⭐ make the destructor virtual ⭐

```cpp
class X{
  ...
public:
  ...
  virtual ~X() { delete []x };
}
```

### *ALWAYS* make the destructor virtual in classes that are meant to have subclasses, *even if the destructor doesn't do anything*

If a class is *not* meant to have subclasses, declare it `final`.
  * `class Y final:public X { ... }`

## Pure Virtual Methods + Abstract Classes
```cpp
class Student {
public:
  virtual float fees();
};

// 2 kinds of student: regular + co-op
class Regular: public Student {
public:
  float fees() override; // calc regular student fees
};

class Coop: public Student{
public:
  float fees() override; // calc coop student fees
};
```
#### Question: what should be put for student fees??
  * not sure - every student should be regular or coop
  * can explicitly give student no implementation

```cpp
class Student {
  ...
public:
  virtual float fees() = 0; // method has NO implementation
                            // this is called a Pure Virtual Method ⭐
};
```
Rule: A class with a pure virtual method ***cannot be instantiated***
  * `Student s;` ❌ wont compile
  * it's purpose is just to organize subclasses

Subclasses of abstract classes are also abstract, *unless* they implement all pure virtual methods.

Non abstract classes are called *concrete* classes.

### Brief note on UML
  * virtual + pure virtual methods, use ITALICS
  * abstract classes you put the class name in italics
  * static represented with underline

# Inheritance and Copy/Move
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
  * calls `Book`'s copy ctor
  * goes field by field i.e. default behavior for the `Text` part
  * same is true for the other operations

##### To write your own operations:
```cpp
Text::Text(const Text &other): Book(other), topic(other.topic) {}

Text &Text::operator=(const Text &other) {
  Book::operator=(other);
  topic = other.topic;
  return * this;
}

Text::Text(Text &&other): Book(other), topic(other.topic) {} ❌❌❌
// other refers to an rvalue, but IS AN LVALUE
// so book COPY CTOR IS RUNNING not MOVE CTOR

// std::move(other) means treat this like garbage ⭐
Text::Text(Text &&other): Book(std::move(other)), topic(std::move(other.topic)) {}
```

### *Note*: Even though other and other.topic refer to rvalues, they themselves are lvalues
  * `std::move(x)` forces an lvalue `x` to be treated as an rvalue so that the "move" versions of the operations run

```cpp
// same kinda deal for move assignment
Text &Text::operator=(Text &&other) {
  Book::operator=(std::move(other));
  topic = std::move(other.topic);
  return * this;
}
```

* these op'ns are equivalent to built-in
* specialize as needed for Node, etc

Now, consider:
```cpp
Text t1 {...};
Text t2 {...};
Book *pb1 = &t1, *pb2 = &t2;
// What if we do:
*pb1 = *pb2;
```
  * it is `Book::operator=` that runs!!
    * called ***partial assignment*** - copies only the book part
    * so how do we fix this?

#### Inclination: try making operator= virtual??

```cpp
class Book {
  ...
public:
  virtual Book &operator=(const Book &other) {...}
};

class Text {
  ...
public:
  Text &operator=(const Text &other) override {...}
};

// ❌ WILL NOT COMPILE ❌
```
*Note*: different return types, but *param* types MUST be the same or its not an override (won't compile)

The above example violates "is a".

  * So why not do `...=(const Book &other)...`
    * this means that assignment from a `Book` to a `Text` is allowed
      - `Text t; t = Book{...};` makes no sense
      - `t = Comic{...};`


  * If `operator=` is non virtual we get partial assignment through base class ptrs
  * if its virtual, compilers allow mixed assignments
  * ❌ both of these are bad

### RECOMMENDATION: All superclasses should be abstract
*******

```
              |abstract book|
              /      |      \
|normal book|     |text|     |comic|
```

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

Need at least one pure virtual method so that the class is abstract. If you don't have one, use the destructor.

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
##### So now:
  * `*pab1 = *pab2` will not compile ⭐
    * `protected` prevents the assignment through base class ptrs from compiling
    * the implementation is still available to subclasses

##### New problem: THE CODE DOESN'T LINK! ⭐

```cpp
AbstractBook::~AbstractBook() {}
```

* a virtual destructor ***must*** be implemented even though it is pure virtual
* step 3: subclass destructor *will* call it. so it must exist.

# Templates

```
          "It's about to crest."
                        - B. Lushman
```

Huge topic - just the highlights here
```cpp
class List {
  struct Node;
  Node * theList;
};

struct List::Node {
  int data;
  Node * next;
};
```

What if we want to use a different type of data? Does that mean we write a whole new class??

##### Instead, write a Template
  * template is a class parameterized by a type

```cpp
template <typename T> class List {
  struct Node;
  Node * theList;
public:
  class Iterator {
    Node * p;
  public:
    ...
    T operator*();
    ...
    T ith (int i);
    void addToFront(T n);
    ...
  };
};

template <typename T> struct List<T>::Node {
  T data;
  Node * next;
};
```

##### Stack example

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

# The Standard Templating Library (STL)

* Large number of useful templates in the c++ library that solve common programming problems

##### E.g. dynamic length arrays : ❗ vectors ❗
```cpp
#include <vector>
using namespace std;
vector <int> v{4,5}; // 4 5
// NOTE: vector <int> v(4,5) is different, will make four 5's i.e. 5 5 5 5
v.emplace_back(6);
v.emplace_back(7); // 4 5 6 7

// Looping:
  for (int i = 0; i < v.size(); ++i) {
    cout << v[i] << endl;
  }

  for (vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
    cout << * it << endl;
  }

  for (auto n: v) {
    cout << n << endl;
  }
  // to iterate in reverse
  for (vector<int>:: reverse_iterator it = v.rbegin(); it != v.rend(); ++it) {
  }

  v.pop_back(); // remove last element

  // use iterators to remove items from inside a vector, for example
  auto it = v.erase(v.begin()); // erase item 0
       it = v.erase(v.begin() + 3); // erase item 3
       it = v.erase(it); // erase the next item
       it = v.erase(v.end() - 1); // erase the last item

  v[i]; // the ith element
        // unchecked, if you go out of bounds that's your fault (undefined behavior)

  v.at(i); // checked version of the ith element
           // checks to make sure you don't go out of bounds - or else.
```

## Problem
* vector's code can detect the error, but doesn't know what to do with it
* client can respond, but can't detect the error

##### C solution
  * function can return a status code or set a global variable `errno`
  * leads to awkward programming
  * encourages programmers to ignore error checks

##### C++ solution
  * in error cases, the function raises an exception
  * by default, execution stops (crash)
  * but we can write handlers to catch exceptions and deal with them
  * `vector<T>::at throws std::out_of_range` when fails

```cpp
#include <stdexcept>
...
try {
  cout << v.at(10000) << endl;
} catch (std::out_of_range theExn) {
  cout << "Range error" << endl;
}
cout << "carry on" << endl;
```

********

##### now consider
```cpp
void f() {
  throw out_of_range("f"); // ctor call
                           // string "f" is what "what" is
}

void g() { f(); }
void h() { g(); }
void main () {
  try { h (); }
  catch (out_of_range) { ... }
}
```

##### what happens?
  * main calls h
  * h calls g
  * g calls f
  * f throws out_of_range
  * g doesn't have a handler for out_of_range
  * control goes back through the call change (unwinding the stack) until a handler is found
    * in this case goes all the way back to main, which handles it
  * if no handler is found in the entire call chain, program terminates

Reminder that `out_of_range` is a class. `throw out_of_range("f")` is the constructor call and `"f"` is the auxiliary info for `what()`

A handler might do part of the recovery job - execute some corrective code and throw another exception `try { ... } catch(awfe) { throw someerror(awef) }` or throw the same exception:

```cpp
try { ... }
catch(someErrorType s) {
  ...
  throw;
}
```

##### throw vs throw s;
  * `throw;` says throw the exact exception that was thrown at you - type information is retained
  * `throw s;` re throws a new exception of type `someErrorType`
    * could by a subclass of `someErrorType`

##### a handler can act as a catch-all
```cpp
try { ---- }
catch(...) { // ⭐ catches all exceptions!!
}
```
* you can throw anything you wan't - you don't even have to throw objects

##### define your own classes for errors
```cpp
class BadInput {};

try {
  int n;
  if (!(cin >> n)) throw BadInput();
} catch(BadInput &) { // catches by reference, reduces copying, avoids slicing
  cerr << "input not well formed";
}
```

##### some standard exceptions
`length_error` - you tried to resize string/vector and it was too big

`bad_alloc` - new fails

***NEVER*** let a destructor throw an exception. If a destructor throws, your program terminates immediately, *unless* the destructor was tagged `noexcept(false)`

If a destructor can and does throw - if that destructor was executed during stack unwinding while dealing with another exception that is still looking for a handler, you now have 2 active unhandled exceptions - your program ***will abort immediately***.

# Design Patterns Continued

##### Guiding principle - program to the interface, not the implementation
  * abstract base classes define the interface
    * work with pointers to abstract base classes and call their methods
    * concrete subclasses can be swapped in and out
      * abstraction over a variety of behavior

```cpp
class List {
public:
  class Iterator:AbstractIterator {

  };
};

class Set {
public:
  class Iterator:AbstractIterator {

  };
};

class AbstractIterator {
public:
  virtual int &operator*() = 0;
  virtual AbstractIterator &operator++() = 0;
  virtual bool operator=(const AbstractIterator &other) const = 0;
  virtual ~AbstractIterator();
};
```

Then you can write code that operates over iterators.

```cpp
void forEach(AbstractIterator &start, AbstractIterator &end, void (*f)(int)) {
  while (start != end) {
    f(*start);
    ++start;
  }
} // works over both lists and sets
```

## Observer Pattern
  * publish-subscribe model
  * one class: publisher/subject (generates data)
  * one or more subscriber / observer classes - receive and react to it

E.g. publisher = spreadsheet cells, observers = graphs. when cells change, graphs update

Can be many different kinds of observe objects - subject should not need to know all the details

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

##### sequence of method calls
1. subject state changes
2. subject::notifyObservers() called (either by the subject itself or by an external controller)
  * calls each observer's notify
3. each observer calls ConcreteSubject::getState to get the new state + reacts appropriately

******

##### Example: horse race
* Subject - publishes winners
* Observers - individual betters
  - declare victory when their horse wins

```javascript
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

## Decorator Pattern

Want to enhance an object at runtime - add functionality/features

##### e.g. windowing system
  * start with a basic window
  * add scrollbar
Want to choose these enhancements at runtime

```
            | Component  | window interface
            | +operator  |
      /                         \
  | ConcreteComponent |       | Decorator|
  | +operation        |             ^
                                    |
                              ---------------
                              |             |
            | ConcreteDecorator A |     | ConcreteDecorator B |
            | +operation          |     | +operation          |

```

* Class Component
  * defines the interface
  * the operations your objects will provide
* ConcreateComponent
  * implements the interface
* Decorators
  * all inherit from decorator, which inherits from component
  * therefore every decorator is a component, and every decorator has a component

##### Eg window w/ scrollbar
  * this is a kind of window, AND it *has* a pointer to the underlying plain window
  * window w/ scrollbar AND menu is a window, and HAS ptr to window w/scrollbar, which has a pointer to the plain window

***All inherit from abstract window, so window methods can be used polymorphically on all of them***

##### E.g. PIZZA
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

## Factory Method Pattern (aka virtual constructor pattern)

##### Write a video game with 2 kinds of enemies: turtle's and bullets.
  * system randomly sends turtles and bullets, but bullets become more frequent in harder levels

```
          | Enemy |
      /             \
| Turtle |         | Bullet |


        | Level |
      /           \
| Easy |         | Hard |

```

* never know which enemy comes next, so can't call turtle/bullet constructors directly
* instead, put a factory method inside level that creates enemies

```cpp
class Level {
public:
  virtual Enemy * createEnemy() = 0; // factory method
};
```

*********

## Template Method Pattern

* Want subclasses to override superclass behavior, but some aspects must stay the same

e.g. there are red turtles and green turtles

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

Subclasses can't change the way a turtle is drawn (head, shell, feet) but can change the way the shell is drawn.

##### Generalization - the non-virtual intervace (NVI) idiom

* a public virtual method is really two things:
  1. public: interface to the client - indicates provided behavior with pre/post conditions
  2. virtual - interface to the subclasses - hook to insert specialized behavior
* hard to separate these ideas if they are tied to the same function
* what if you want to split a virtual method into two without affecting the client interface
* how can you make overriding functions respect pre/post conditions

#### NVI idiom
  * all public methods should be non-virtual
  * all virtual methods should be non-public (private or protected)
    * except the destructor ⭐

```cpp
class DigitalMedia {
public:
  virtual void play() = 0;
}

class DigitalMedia {
public:
  void play() { doPlay(); } // can add before code e.g. check copyright
                            // can add after code e.g. update play count
private:
  virtual void doPlay() = 0;
}
```
Generalize Template Method:
  * puts ***every*** void method inside a template method

******

### (ASIDE) STL map - for creating dictionaries

##### e.g. "arrays" that map strings to int
```cpp
#include <map>
using namespace std;
map<string, int> m;
m["abc"] = 1;
m["def"] = 4;
cout << m["hgi"]; // you get 0 back
                  // if the key is not present, it is inserted...
                  // the value is default constructed, for ints, that is 0

m.erase("abc");

if (m.cout("def")) // 0 not found, 1 found
```

* iterating over a map - results are presented in sorted key order

```cpp
for (auto &p: m) { // p is a std.pair(key, value) from <utility>
  cout << p.first << " " << p.second;
}
```
******

## Visitor Pattern

  * For implementing double dispatch
  * virtual methods - chosen based on the actual type (at runtime) of the object on which they are called
  * what if - you want to choose based on two objects?

```
        | enemy |
      /             \
| turtle |        | bullet |

        | Weapon |
      /           \
    stick         rock
```
  * strike an enemy with a weapon, result depends on both enemy and weapon
  * if strike virtual in Enemy, choose based on Enemy but not Weapon aiya
  * if strike virtual in Weapon, choose based on Weapon but not enemy :/

##### Trickery to get dispatch based on both
  * involves a combination of *overriding* and *overloading*

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
* bullet's `bestruckby` runs (by virtual method lookup)
* calls weapon::strike, `*this` is bullet
  * bullet version chosen at compile time
  * virtual strike method call resolves to `rock::strike` on a bullet

##### Visitor can be used to add functionality to existing classes without changing/recompiling

```cpp
class Book {
public:
  ...
  virtual void accept(BookVisitor &v) { v.visit(* this); }
};
class Text:public Book {
public:
  ...
  void accept(BookVisitor &v) override { v.visit(* this) }
};
// similarly, comic

class BookVisitor {
public:
  virtual void visit(Book &b) = 0;
  virtual void visit(Text &t) = 0;
  virtual void visit(Comic &c) = 0;
};
```
##### Application - track how many of each kind of book I have

* Books - by author
* Text - by topic
* Comic - by hero
* I could add:
  * `virtual void addMeTopMap(...)`
* or write a visitor:

```cpp
class Catalogue:public BookVisitor {
public:
  map<string, int> theCatalogue;
  void visit (Book &b) override { ++theCatalogue[b.getAuthor()]; }
};
```
**********

##### But above won't compile... why??
  * `book.h` includes `BookVisitor.h` includes `text.h` includes `book.h`
  * circular includes dependency
  * `Text` does not know what `Book` is

## Compilation Dependencies
```cpp
// consider
class A {...}; // a.h

class B:public A { ... };
class C {A a;};
class D {A*ap;};
class E{A f(A a);};
```

##### Question: Which of B, C, D, E require an include??
  * B needs `#include a.h`
  * C has an A in it, so to know how big a C object is we need an `#include a.h`
  * pointers are the same size, so all we need in D is `class A;` (forward declaring)
  * E doesn't need include, only `class A;`
  * Only A and C need it, because we need to know how big B and C are

##### If there is no compilation dependency necessitated by the code, don't create one with extra includes.

When class `A` changes, only `A`, `B`, `C` need to recompile.

Different story when you look at .cc implementation files. In the implementations of D and E:
```cpp
#include "a.h"

void D::f() {
  myA->someMethod();
  // need to know about class A here - a true compilation dependency
  // needs to include a.h here, but not in E's header.
}
```

Do the #include in the .cc, not in .h, where possible. Because including .h files in .cc, there would never be a cycle, because you never include .cc files. Reduces compilation cycle errors.

##### Now consider the XWindow class
```cpp
class XWindow {
  Display * d;
  Window w;
  int s;
  GC gc;
  unsigned long int colors[10];
public:
};
```
* this is private data. yet we can look at it. do we know what it all means? do we care? (no)
* what if we add/change a private member? all classes must recompile 😢
* it would be better if those details weren't there...

##### SOLN: the `pimpl idiom` ("pointer to implementation")
Create a second class `XWindowImpl.h`

```cpp
#include <X11/Xlib.h>

struct XWindowImpl {
  Display * d;
  Window w;
  int s;
  GC gc;
  unsigned long int colors[10];
};
```

Now in `window.h`:

```cpp
class XWindowImpl;
class XWindow {
  XWindowImpl * pimpl;
public:
  // no change
};
```

In window.cc:
```cpp
#include "window.h"
#include "XWindowImpl.h"

XWindow::XWindow(...):pimpl(new XWindowImpl) {}
// MIL: need to allocate space for the pointer
// Also need to destroy pImpl in the dtor
// Other methods: replace fields (d, w, s, etc.) with pImpl->d, pImpl->w, pImpl->s, etc.
```

If you confine all private fields to XWindowImpl then only window.cc needs to recompile if XWindow's implementation

##### Generalization - what if there are several possible window implementations, e.g. XWindows and YWindows
  * then make the Impl structure a superclass

```
  window (ptr) *----owns-a---> windowImpl
                                  ^
                                  |
                            ---------------
                            |             |
                      XWindowImpl       YWindowImpl
```
pimpl idiom with a class hierarchy of implementations - called ***bridge pattern***

## Measures of Design Quality
* coupling and cohesion

##### coupling
  * the degree to which distinct program modules depend on eachother
  * ⬇️ low to high ⬆️
    * modules communicating via function calls with basic params/results
    * modules pass arrays/structs back and forth
    * modules affect each other's control flow
    * modules have access to each other's implementation (friends)
  * high coupling implies that the changes to a module require greater changes to other modules

##### cohesion
  * how closely elements of a module are related to each other
  * ⬇️ low to high ⬆️
    * arbitrary grouping of unrelated elements (c++ utility)
    * elements sharing a common theme but otherwise unrelated (perhaps share some base code)
    * elements manipulate state over the lifetime of an object (e.g. open/read/close files)
    * elements pass data to eachother
    * elements cooperate to perform exactly one task 🙏

##### GOAL: LOW COUPLING HIGH COHESION
Your primary program classes should not be printing things!
E.g.
```cpp
class ChessBoard {
  ...
  cout << "Your move" << endl;
};
```
Bad design - inhibits code reuse.
What if you want to reuse Chessboard, but not have it communicate via stdout???

******
### Missed lecture courtesy of dzed

One solution: give the class stream objects, where it can perform input and output (I/O)

```cpp
class Chessboard {
  istream &in;
  ostrema &out;
public:
  ChessBoard(istream &in, ostream &out) :
    in{in}, out{out} {}
  // and now we will have this code instead:
  out << "Your move";
};
```

What if we don’t want to use streams at all?

But ChessBoard shouldn’t be talking or doing any communication at all. Its job is to play chess.

### Single-Responsiblity Principle

“A class should have only one reason to change”

In the above example, game state AND communication are TWO reasons to change.

Better solution: Communication with the ChessBoard via parameters and results, and occasionally via exceptions.

Confine user communication to outside the game class.

Question: Should main do all the communication, and then call ChessBoard methods?

Answer: NO. Hard to reuse if it’s in main. Should have a class to manage interaction that is separate from the game state class

## Pattern - Model - View - Controller (MVC)

* Separate the distinct notions of the data (or state), the presentation of the data, and the controll of the data.

Example: ChessBoard

* Model: the main data you are manipulating (e.g. game state)
* View: how the model is displayed to the user
* Controller: how the model is manipulated
`Model -> Controller <- View`

##### Model:
* Can have multiple views (e.g. text and graphics, or several graphics)
* Doesn’t need to know about their details
* Classic observer pattern (or could communicate through controller)

##### Controller:
* Mediates control flow between the model and view
* Might encapsulate turn-taking, or full game rules
* May communicate with user for input (or this could be the view)
* By decoupling presentation and control, MVC promotes reuse.

### Exception safety

```cpp
void f() {
  MyClass * p = new MyClass;
  MyClass mc;
  g();
  delete p;
}
```
No leaks. But what if g raises an exception?

What is guaranteed?

* During stack unwinding, all stack-allocated data is cleaned up - dtors run, memory reclaimed
* Heap-allocated memory is not freed
Therefore, if g throws, `*p` is leaked, mc is not. So:

```cpp
void f() {
  MyClass * p = new MyClass;
  MyClass mc;
  try {
    g();
  } catch (...) {
    delete p;
    throw;
  }
  delete p;
}
```

Tedious, error-prone, duplication code. How else can we guarantee that something (e.g. delete p) happens no matter f exits now or exits due to an exception?

In some languages, “finally” clauses (in Java) guarantee certain final actions. NOT IN C++.

The only thing you can count on in C++ is that destructors for stack-allocated data will run.

Thus, use stack-alocated data with dtors as much as possible. Use the guarantee to your advantage.

##### C++ Idiom: RAII - Resource Acquisition Is Initialization

Every resource should be wrapped in a stack-allocated object whose dtor destroys it.

e.g. files:
```cpp
void h() {
  ifstream f("file"); // acquiring the resource, ("file")
  // initializing the object
  // ...
}
```

File is guaranteed to be closed when f is popped from the stack (f’s dtor runs).

The same can be done with dynamic memory.

```cpp
class std::unique.ptr<T>; // takes a T* in ctor
// dtor will free the pointer
// In-between - can dereference just like a pointer

#include <memory>
```

fix `f()`:
```cpp
void f() {
  auto p = std::make_unique<class>(); // allocate MyClass on the heap
  MyClass mc;
  g();
}
```

This will not leak and is also safer. Also shorter.

*****
