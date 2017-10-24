
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
}
```
