# Final Study

Trivia and a lot of examples, mostly from tutorials

# Table of Contents
1. [Abstract](#abstract)
2. [Design Patterns](#design)
   - [Abstract Iterator Pattern](#aip)
   - [Factory Design Pattern](#fdp)
   - [Template Method Pattern](#tmp)
   - [Visitor Pattern](#vp)

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
#include "window.h"
#include <ctime>

class Turtle{
	protected:
		int x;
		int y;
		Xwindow *w;
	private:
		void drawHead(){
			w->fillCircle(x,y,15,3);
		}
		void drawFeet(){
			w->fillCircle(x + 17,y+14 ,6,3);
			w->fillCircle(x + 37,y+14,6,3);
		}
		virtual void drawShell() = 0;
	public:
		Turtle(int x, int y, Xwindow * w)
			:x(x), y(y), w(w){}
		void drawTurtle(){
			drawHead();
			drawFeet();
			drawShell();
		}
};

class RedTurtle: public Turtle{
		void drawShell() override{
			w->fillArc(x + 10,y-5,40,40,180,-180,2);
		}
	public:
		RedTurtle(int x, int y, Xwindow *w): Turtle(x,y,w){}
};

class GreenTurtle: public Turtle{
		void drawShell() override{
			w->fillArc(x + 10,y-5,40,40,180,-180,10);
		}
	public:
		GreenTurtle(int x, int y, Xwindow *w): Turtle(x,y,w){}
};

class NinjaTurtle: public GreenTurtle{
		void drawShell() override{
			w->fillArc(x + 10,y-5,40,40,180,-180,10);
			drawSash();
		}
		virtual void drawSash() = 0;
	public:
		NinjaTurtle(int x, int y, Xwindow *w): GreenTurtle(x,y,w){}
};

class Leo: public NinjaTurtle{
		void drawSash() override {
			w->fillRectangle(x + 27, y-5, 5, 20,5);
		}
	public:
		Leo(int x, int y, Xwindow *w): NinjaTurtle(x,y,w) {}
};

class Don: public NinjaTurtle{
		void drawSash() override {
			w->fillRectangle(x + 27, y-5, 5, 20, 7);
		}
	public:
		Don(int x, int y, Xwindow *w): NinjaTurtle(x,y,w) {}

};

class Raph: public NinjaTurtle{
		void drawSash() override {
			w->fillRectangle(x + 27, y-5, 5, 20, 2);
		}
	public:
		Raph(int x, int y, Xwindow *w): NinjaTurtle(x,y,w) {}

};

class Mike: public NinjaTurtle{
		void drawSash() override {
			w->fillRectangle(x + 27, y-5, 5, 20, 8);
		}
	public:
		Mike(int x, int y, Xwindow *w): NinjaTurtle(x,y,w) {}

};

int main(){
	Xwindow win;
	RedTurtle rt(100, 50, &win);
	GreenTurtle gt(100, 100, &win);
	Leo leo(200, 200, &win);
	Don don(250, 200, &win);
	Raph raph(300, 200, &win);
	Mike mike(350, 200, &win);

    rt.drawTurtle();
	gt.drawTurtle();
	leo.drawTurtle();
	don.drawTurtle();
	raph.drawTurtle();
	mike.drawTurtle();

    int x;
	std::cin >> x;
}
```

### Visitor Pattern <a name="vp"></a>
