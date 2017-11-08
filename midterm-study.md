# bash

#### write a script which
  1. accept a program and its args as arguments
  2. if we got no arguments, print a usage message to stderr and exit with code 2
  3. otherwise, run valgrind on the program
    * if valgrind reported memory errors, print a message to stderr and exit with code 1
    * else, exit with code 0

***HINTS***
  1. valgrind with print "0 errors" to stdout if there are no errors
  2. `$@` is all the arguments to your script

```bash
#!/bin/bash

if [ $# -lt 1 ]; then
  echo "Usage $0 [program]" 1>&2
  exit 2
fi

errfile=$(mktmp)
valgrind "$@" 2> "$errfile"
egrep "0 errors" "$errfile" &> /dev/null

if [ $? -ne 0 ]; then
  echo "Memory errors" 1>&2
  rm "$errfile"
  exit 1
fi
rm "$errfile"
```

* why put quotes? because if there are space in filenames, it will treat as one argument with quotes

# the big five

*Steps for object creation*
1. space is allocated (stack/heap)
2. default initialization of fields that are objects
3. constructor body runs

***MIL is used to 'hijack' step 2***
```cpp
struct Student {
  int assns, mt, final;
  const int id; // ya
  Student (int assn, int mt, int final, int id): assn(assn), mt(mt), final(final), id(id) {} // can only initialize constant here
}
```

*Thanks Robert*
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
  if (this == &other) return \*this;
  data = other.data;
  delete next;
  next = other.next ? new Node(*other.next) : nullptr;
  return \*this;
}

// move assignment
Node &Node::operator=(Node &&other) {
  swap(data, other.data);
  swap(next, other.next);
  return \*this;
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

#### destructors
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
