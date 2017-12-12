# Yuchen Martin Ma
Review session questions taken from `question.pdf`
## q1 a)
##### Visitor Pattern
* double dispatch

```
  A        B
  |        |                <- specific code for both hierarchies
A1, A2..  B1, B2..
```

```cpp
class A1: public A {
  public:
    void accept(B &b) override { b.visit(* this); }
};

class B1: public B {
public:
  void visit(A1 &b) override {
    // A1 B1 code
  }
  ... // one overload for each subclass
};
```
```
    |A|           |B|
  /     \       /     \
a1      a2     b1     b2
methods
etc
```
* visitor loosens coupling
  * easier to extend hierarchies, add subclasses, etc
* look at the old code, any changes requires you to go through and change everything

## q2
```cpp
#include "a.h"
#include "b.h"

#include <iostream>

class C;
class D;
class E;
class F;
class G;
class H;
class I;
class J;

class K: public A {
  B b;
  C* c;
public:
  D& d;
  void foo(E);
  void foo(F*);
  void foo(G&);
  H func();
  I* func2();
  J& func3();
};

std::ostream& operator<<(std::ostream&, const K&);
```

## q3
```cpp
struct Node{
  int data;
  Node * next;
  // Assume the following methods are properly defined.
  Node();
  Node(int x); // No-throw guarantee
  Node(const Node &); // basic guarantee
  Node &operator=(const Node &); // basic guarantee
};

Node giveMeANode(int a){
  Node n;
  try{
    n = Node(a); // assignment operator is basic guarantee
  } catch (...){
  }
  return n; // calls copy ctor, basic guarantee
}
```
* basic guarantee

```cpp
class C {
  struct CImpl{
    A a;
    B b;
    // A, B copy ctor has strong guarantee (assumption)
  };
  unique_ptr<CImpl> pImpl;
  void f();
};

void C::f() {
  auto temp = make_unique<CImpl>(* pImpl);
  temp->a.method1(); // No-throw
  temp->b.method2(); // No-throw
  std::swap(pImpl, temp); // No-throw
}
```
* strong guarantee

```cpp
void g(){
  int * x = nullptr;
  * x = 3;
}
```
* no throw

```cpp
class C {
  int a, b, c;
  D d;
public:
  // helper function
  void swap(C &other){
    std::swap(a, other.a);
    std::swap(b, other.b);
    std::swap(c, other.c);
    std::swap(d, other.d); // if this swap throws, you've already swapped a,b,c
  }
  C &operator=(const C &other){
    unique_ptr<C> temp;
    try{
      temp = make_unique<C>(other);
    } catch (...) {return * this;}
    this->swap(* temp);
    return * this;
  }
};
```
* basic guarantee

## q5
```cpp
template <typename T>
class BinTree {
  struct BinNode {
    T data;
    BinTree<T> * left, * right, * parent;
  }
  unique_ptr<BinNode> root;
public:
  // omitted
};

// part c)
template <typename T>
class BinTree<T>::Iterator {
  BinTree<T> * curr;
  Iterator(BinTree * b): curr(b);
public:
  bool operator!=(const Iterator &other) {
    return curr != other.curr;
  }
  T &operator*() {
    return curr->node->data;
  }
  Iterator &operator++() {
    if (curr->node->right) {
      curr = curr->node->right;
      while (curr->node->left) {
        curr = curr->node->left;
        return * this;
      }
    } else {
      while (curr == curr->node->parent->node->right) {
        curr = curr->node->parent;
      }
    }
  }

  friend class BinTree<T>;
}

// In bintree class
Iterator begin() {
  BinTree * curr = * this;
  while (curr->node->left) {
    curr = curr->node->left;
  }
  return Iterator(curr);
}

Iterator end() {
  return nullptr;
}
```

## q6
```cpp
template <typename T, typename Fn, typename Xiter>
T foldl(Fn f, T base, Xiter begin, Xiter end) {
  while (begin != end) {
    base = f(*begin, base);
    ++begin;
  }
  return base;
}

template <typename T, typename Fn, typename Xiter>
T folr(Fn f, T base, Xiter begin, Xiter end) {
  if (!(begin != end)) return base;

  Xiter next = begin;
  ++next;
  return f(*begin, foldr(f, base, next, end));
}
```
