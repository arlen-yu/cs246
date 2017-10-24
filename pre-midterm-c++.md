# Module 2: c++

```c
// Hello world in c
# include <stdio.h>
int main () {
  printf("hello, world!\n");
  return 0;
}
```

```c++
// Hello world in c++
# include <iostream>
using namespace std;
int main() {
  cout <<"Hello, world!" <<endl;
  return 0;
}
```
  * main **must** return int in c++
  * stdio.h and printf are still available
    - prefer c++ I/0: header `<iostream>` and output `std::cout << ___ << __ << __ ..` and `std::endl` is the end of line
    - `using namespace std` lets you omit std::

#### Compiling C++ Programs
```bash
g++ -std=c++14 -Wall hello.cc -o hello
# OR use the alias
g++14 hello.cc -o hello
```
#### Input/Output
  * 3 I/O streams
    - `cout` for printing to stdout
    - `cin` for reading from stdin
    - `cerr` for printing to stderr
  * 2 I/O operators
    - `<<` "put to" (output)
    - `>>` "get from" (input)

```c++
// Example: add 2 #'s
# include <iostream>
using namespace std;

int main () {
    int x, y;
    cin >> x >> y; // gets 2 ints from stdin, skipping whitespace
    cout << x + y << endl;
}
```
  * if a read fails, `cin.fail()` will be true
  * if EOF, `cin.eof()` and `cin.fail()` will be true, but not until the attempted read fails!

```cpp
// Example - read all ints from stdin and echo them one per line to stdout
//         - stop on bad input or EOF
int main () {
  int i;
  while (true) {
    cin >> i;
    if (cin.fail()) {
      break;
    }
    cout << i << endl;
  }
}
```
  * there is an implicit conversion from `cin` to `bool`
    - lets cin be used as a condition
    - will act like `true` for success or `false` for failure

```c++
// Same example, V2.0
int main() {
  int i;
  while (true) {
    cin >> i;
    if (!cin) break; // <- the change!
    count << i << endl;
  }
}
```

* operator `>>`
  - inputs
    - `cin` type iostream
    - `date` various types
  - outputs
    - returns `cin` back (iostream)
      - this is why we can write `cin >> x >> y >> z`

```c++
// Same example V3.0
int main () {
  int i;
  while (true) {
    if (!(cin >> i)) break;
    count << i << endl;
  }
}
```

```c++
// Same example V4.0
int main () {
  int i;
  while (cin >> i) {
    cout << i << endl;
  }
}
```

```c++
// Example: - read all ints and echo on stdout until EOF
//          - skip non integer inputs
int main() {
  int i;
  while (true) {
    if (!(cin >> i)) {
      if (cin.eof()) break;
      cin.clear() // reset cin after it fails
      cin.ignore() // ignore the current input (it's garbage!)
    } else { // the read was ok
      cout << i << endl;
    }
  }
}
```

#### Reading Strings
  * type `std::string` you get by `#include <string>` - more details later

```c++
int main() {
  string s;
  cin >> s; // read a string - skips leading whitespace! - stops at whitespace!
  cout << s << endl;
}

// What if we want whitespace??
// We would use getline(cin, s)
```
  * `getline(cin, s)` reads from current position to the newline char in s
  * consider `cout << 95 << endl;`, will print 95. what if we want to print it in hexadecimal?
    - `cout << hex << 95 << endl;` will print 5f
      - ^ says that all subsequent ints will be printed in `hex` forever
      - many, many other manipulators like this
        - `#include <iomanip>`
  * stream abstraction applies to other sources of data
    * reading from a file instead of stdin
      1. std::ifstream
        - reading from a file
      2. std::ofstream
        - writing to a file

```c
// File access in C
int main() {
  char s[246];
  FILE \*f = fopen("suite.txt", "r");
  while (true) {
    fscanf(f, "%255s", s);
    if (feof(f)) break;
    printf("%s\n", s);
  }
  fclose(f)
}
```
```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;
int main () {
  ifstream f { "suite.txt" } // Defining and initializing f is what opens the file
  string s;
  while (f >> s) {
    cout << s << endl;
  }
  // No fclose needed because the file is closed when the ifstream i.e. f goes out of scope
}
```
  * anything you can do with `cin` or `cout` you can do with an `ifstream` or `ofstream`
    - ex: strings
      - you can attach a stream to a string variable and then read from or write to the string
      - `#include <sstream>`, you want `std::istringstream` (reading from string) and `std:ostringstream` (writing to string)

```c++
// Ex converting numbers to strings
int hi = _____, lo = _____;
ostringstream oss;
oss << "Enter a # between" << lo << "and" << hi; // Converting numbers to strings
string s = oss.str();
cout << s << endl;
```

```c++
// Ex converting strings to numbers
int n;
while (true) {
  cout << "Enter a #" << endl;
  string s;
  cin >> s;
  istringstream iss {s}; // Wrap a sock around the string s
  if (iss >> n) break;
  cout << "I said,";
}
cout << "You entered" << n << endl;
```

Example revisited: Echo all numbers, and skip the non numbers
```c++
int main() {
  string s;
  while (cin << s) {
    istringstream iss {s};
    int n;
    if (iss >> n) cout << n << endl;
  }
}
```

#### More on Strings

* in C: array of char (`char *` or `char []`) terminated by `\0`
  - you have to explicitly manage memory - allocate more memory as strings get larger
  - easy to overwrite `\0` and corrupt memory
* in C++: strings grow as needed
  - no need to manage memory
  - safer to manipulate
  - `string s = "hello";`
    - "hello" is still a c-style string, the meaning does not change
      - still a char array with a null terminator
    - `s` is a C++ string which is created from the C string on initialization

#### String Operations

  * equality/inequality
    - `s1 == s2`, `s1 != s2` works!!
  * comparison
    - `s1 <= s2` ez
  * length
    - `s.length()` O(1) time!!
  * get individual chars
    - `s[0]`, `s[1]`, etc.
  * concatenation
    - `s3 = s1 + s2`, `s3 += s4`

#### Default Function Params
```c++
void printSuiteFile(string name = "suite.txt") { // Gives name a default value
  ifstream f {name};
  while (f >> s) cout << s << endl;
}

printSuiteFile("suite2.txt")
printSuiteFile() // Defaults to suite.txt
```
* Note: Optional arguments **must be last**

#### Overloading
```c
// In C:
int negInt(int n) { return -n; }
bool negBool(bool b) { return !b; }

// In C++: - functions with different parameter lists can share the same name
int neg(int n) { return -n; }
bool neg(bool b) { return !b; } // This is called overloading
```
  * in c++ the compiler uses the number and types of the arguments to decide which `neg()` is being called
    - this process is known as overload resolution
  * overloads must differ in number or types of arguments - may not differ on just the return type
  * we've seen this already
    - input/output operators are overloaded - sometimes they mean in/out, sometimes they mean bitshift
    - `+` sometimes means add two numbers, sometimes means concat two strings

#### Structs
```c++
struct Node {
  int data;
  Node \*next;
}; // NOTE: Don't forget the semicolon!!
```

#### Constants
```c++
const int maxtrade = 100; // Must be initialized
```
  * declare as many things const as you can, catches errors

```c++
Node n1 = {5, nullptr}; // Syntax for Null pointers in C++
const Node n2 = n1; // Immutable copy of n2
const Node *pn = &n1; // Can't change n1 through the pointer pn, but changing n1 directly is o-k.
Node *p2 = &n2; // This is ILLEGAL. You can't have ptr-to-non-const pointing to const
```

### Parameter passing ⭐

#### Recall
```c++
void inc(int n) {
  int x = 5;
  inc(x);
  cout << x; // this will print 5, because you're passing by value
}
```
  * pass-by-value: `inc` gets a copy of `x`, then increments the copy, while the original is unchanged
  * If a function needs to modify a parameter, **pass a pointer**

```cpp
void inc(int *n) { *n = *n + 1 }

...
int x = 5;
inc(&x);
count << x;
```
  * **address** passed by value
  * inc changes the data at that address
  * changes visible to the caller

Q: *Why do we say `cin >> x` and not `cin >> (&x)`?*
A: c++ has another ptr-like type: **references** (⭐ important ⭐)

## REFERENCES

```cpp
int y = 10;
int &z = y; // Called an lvalue reference to int (to y)
```
  * `&z` is a called an lvalue reference to int (to y)
  * it's kind of like a const ptr
  * similar to `int *const z = &y`, i.e. you can't change the address, but you can still change the data it's pointing to

References are like constant pointers with automatic dereferencing
```
z [ * ]             
      \
      y[10]
```
  * `z = 12` (**NOT** `*z = 12`) will result in y[12]
  * `int *p = &z`, where `&z` gives the address of y!!
  * in all cases, `z` behaves exactly like `y`
  * we say that ***z is an alias (another name) for y***

***Things you can't do with lvalue references***
  1. you can't leave them uninitialized - e.g. `int &x;` is invalid ❌
  2. they must be initialized with something that has an address (i.e. an lvalue) since ***refs are ptrs***
    * `int &x = 4;` ❌
    * `int &x = y + z;` ❌
    * `int &x = y;` ✔️
  3. create a pointer to a references
    * `int &*x = ______;` ❌
    * ref to a ptr `int *&x = ______;` ✔️
  4. create a reference to a reference
    * `int &&r = ________;` ❌
  5. create an array of references
    * `int &r[3] = {_, _, _};` ❌

***Things you can do with lvalue references***
  1. pass them as function parameters ⭐

```c++
void inc(int &n) { // const ptr to the arg (x), changes to n affect x
  n = n + 1; // no ptr deref
}
int x = 5;
inc(x)
count << x; // prints 6
```
*So, why does `cin << x` work? Because it takes in `x` by reference!*
  * `istream &operator >> (istream &in, int &x);`
    - you can't copy a stream, so you need to pass in it by reference, and give it back as reference

Pass-by-value, e.g. `int f(int x) { ... }` copies the argument
  * if the argument is really big, this is *expensive*

```c++
struct ReallBig {...1 trillion elements...};
int f(ReallyBig rb) {...} // SLOW AF
int g(ReallyBig &rb) {...} // Alias - fast, but you lose the guarantee that rb won't change

// Is there a compromise? (Pass rb quickly, but guarantee it wont change)
int h(const ReallyBig &rb) {...} // FAST, NO COPY, PARAMETER CAN NOT CHANGE
```
  * *Advice*: prefer pass-by-const-ref over pass-by-value for anything larger than a pointer, *unless* the function needs to make a copy anyways (in that case, use pass-by-value)

Suppose:
```cpp
int f(int &n) { ... }
int g(const int &n) { ... }
f(5); // you can't do this, because 5 doesn't have an address ❌
g(5); // you CAN do this, since n can never change the compiler allows it ✅
```
  * for g(5), the compiler makes a temporary location in memory to hold the 5, so the ref n has something to point to

#### Dynamic Memory Allocation
In C: `malloc(....), free(...)` - don't use this in C++!!
Instead: `new/delete` - a lot better (type aware, better integrated, error-prone)

e.g. `Node *np = new Node; ... delete np;`
  * Stack
    * all local variables reside on the stack
    * vars are deallocated when they go out of scope (stack is popped)
  * Heap
    * allocated memory resides on the heap
    * remains allocated until delete is called
    * if you don't delete all allocated memory, you get a *memory leak*
      * the program will eventually fail
      * incorrect behaviour

|heap|stack|
|-----|----|
|Node|np|

e.g.
```cpp
Node getMeANode() {
  Node n;
  return n;
}
```
  * this is potentially very expensive - n is copied to the caller's stack frame on return
  * should be return a reference/ptr instead?

```cpp
❌ ❌ ❌
Node & getMeANode() {
  Node n;
  return n;
}
```
  * ❌ "this is one of the worst things you could ever do" - Brad Lushman
  * you're pointing at something in an old frame, the frame that the var is in will be overwritten!!

```cpp
Node *getMeANode() {
  return new Node;
}
```
  * this works, is ok because we're returning a pointer to the heap (will be alive)
  * but you have to remember to delete the data when you're done with it

#### Operator Overloading
Give meanings to c++ operators or new types!
e.g.
```cpp
struct Vec {
  int x, y;
};

Vec operator +(const Vec &v1, const Vec &v2) {
  Vec v { v1.x + v2.x, v1.y + v2.y };
  return v;
}

Vec operator *(const int k, const Vec &v1) { // Only works when the scalar is on the left
  return { k*v1.x, k*v1.y };
}

Vec operator *(const Vec &v1, const int k) {
  return k*v1;
}

Vec v, w, x;
v = w + x;
```

***Overloading << and >>***
```cpp
// Example
struct Grade {
  int theGrade;
};

ostream &operator<<(ostream &out, const Grade &g) {
  return out << g.theGrade << '%';
}

istream &operator>>(istream &in, Grade &g) {
  in >> g.theGrade;
  if (g.theGrade < 0) g.theGrade = 0;
  if (g.theGrade > 100) g.theGrade = 100;
  return in;
}
```

#### The Preprocessor
  * Transforms the program before the compiler sees it
  * \#_________ - preprocessor directive, e.g. `#include`
  * Including old C headers - new naming convention
    - instead of `#include <stdio.h>`, you do `#include <csdio>`
  * `#define VAR VALUE` sets a preprocessor variable
    - then all occurrences of `VAR` in src file are replaced with `VALUE`

```cpp
// OBSOLETE ❌
#define MAX 10
int x[MAX]; // transformed by the preprocessor to int x[10];

// it's whack, look at this:
#define ever ;;
...
for(ever) {

}
```
  * nowadays you would use const definitions instead
  * but, defined constants are useful for conditional compilations

```cpp
// Controlling what code the compiler is seeing
#if SECURITYLEVEL == 1
  short int
#elif SECURITYLEVEL == 2
  long long int
#endif
  publickey;
```
```cpp
// SPECIAL CASE - heavy duty commenting out
#if 0 // will never be true
  ...
#endif
```
  * heavy duty commenting out is better because `/* */` doesn't nest, can't comment within comments

Can also define symbols via compile args:
```cpp
int main () {
  cout << X << endl;
}
```
and then run with
```bash
g++14 -DX=15 test.cc -o define // Tells the processor to define X as 15
```

another case:
```cpp
// true if name has/has not been defined

#ifdef NAME
#ifndef NAME
```

```cpp
int main() {
  #ifdef DEBUG
    cout << "setting x = 1" << endl;
  #endif
    int x = 1;
    while (x < 10) {
      x += 1
      #ifdef DEBUG
        cout << "x is now" << x << endl;
      #endif
    }
    cout << x << endl;
}
```

## Separate Compilation
Split programs into composable modules which each provide:
  * interface
    - type definitions, function headers, `.h` file
  * implementation
    - full definition for every provided function, will be in your `.cc` file

Recall: *declaration vs definition*
  * declaration asserts **existence**
  * definition gives the "full details" and also allocates space (in the case of variables and fn's)

```cpp
// INTERFACE (vec.h)
struct Vec { // DEFINE the structure
  int x, y;
};

Vec operator+(const Vec &v1, const Vec &v2); // DECLARE the function

// MAIN
#include vec.h

int main () {
  Vec v{1, 2};
  v = v + v;
}

// IMPLEMENTATION
#include vec.h
Vec operator+(const Vec &v1, const Vec &v2) {
  return {v1.x + v2.x, v1.y + v2.y};
}
```

  * Recall: an entity can be declared many times but defined at most once

Compiling Separately ⭐:
```
g++14 -c vector.cc // -c means compile only... do NOT link
// this creates vgector.o (object file)

g++14 -c main.cc // creates main.o

g++14 main.o vector.o -o main // this puts the pieces together, runs the linker
```

*What if we want a module to provide a global variable?*
  * abc.h: `int globalNum;`, this is a declaration **AND** a definition
    - every file that includes abc.h defines a separate globalNum - ***program will not link***
  * abc.cc: `int globalNum;` this is a defn, ❌
  * abc.h: `extern int globalNum;` this is a declaration but **NOT** a definition ✅

Suppose we write a linear algebra module:
  * linalg.h: `#include vector.h`
  * linalg.cc: `#include "linalg.h"` and `#include "vector.h"`
  * this ***wont compile*** because `linalg.h` includes `vector.h` as does `linalg.cc`
  * at the end, `vector.h` is included twice
  * therefore `struct Vec` defined twice ❌

Thus, we need to prevent files from being included more than once
  * sol'n: `#include guard`

```
// vector,h
#ifndef VECTOR_H
#define VECTOR_H
..
..
..
#endif
```
  * the first time vector.h is included, `VECTOR_H` becomes defined
  * the next time its included, the shit in the middle ***doesn't run***

***ALWAYS*** put #include guards in .h files ⭐
***NEVER EVER*** compile `.h` files or include `.cc` files
***NEVER EVER*** put `using namespace std` in headers - it forces the client to open the namespace, should be the client's choice not the library's choice
***ALWAYS*** use `std::` prefix in headers


# Classes

Recall:
``` c++
Node *np = new Node,
...
delete np;
```

**Arrays**:
``` c++
int *p = new int [40];
...
delete [] p; // The form of delete must match the form of new
```

### Classes
* Can put function inside of structs

Example:
``` c++
struct Student {
  int assns, mt, final;
  float grade(){
    return assns * 0.4 + mt * 0.2 + final * 0.4;
  }
};

Student s{60, 70, 80};
cout << s.grade() << endl;
```

**Class**:
 * A structure type that can contain functions.
 * C++ has a *class* keyword - we will use it later.

**Object**:
 * An instance of a class

**Member Function**:
 * Also known as a method
 * Is the function contained in the class (like grade() in Student)

What do assns, mt, final mean inside of `grade(){...}`?
 * They are fields of the current object: the object upon which the method was called.
 ``` c++
 Student billy {...};
 billy.grade();   // Uses Billy's assns, mt, final
 ```
 * **Formally**: Methods take a hidden extra parameter called *this* - A pointer to the object upon which the method was called.
 ``` c++
 billy.grade();       // this == &billy in the function.
 ```
  * We can write:
  ``` c++
  struct Student{
    int assns, mt, final;
    float grade(){
      return this->assns * 0.4 + this->mt * 0.2 + this->final * 0.4;
    }
  };
  ```

### Initializing Objects
* `Student billy{60, 70, 80};` is okay, but limited
* Better: A method that does initialization: a **constructor**.
  ``` c++
  Struct Student {
    int assns, mt, final;
    Student (int assns, int mt, int final){
      this->assns = assns;
      this->mt = mt;
      this->final = final;
    }
  };

  Student billy {60, 70, 80};
  ```
  * If a constructor has been defined, these arguments are passed to the constructor
  * If no constructor has been defined, then these arguments initialize the individual fields of Student

##### Initializing variables:
  * `int x=5; // Works`
  * `int x(5); // Works`
  * `string s = "hello"; // Works`
  * `string s("hello"); // Works`
  * `ifstream f = "hello.txt"; // DOESN"T WORK`
  * `ifstream f("hello.txt"); // Works`

In 2010:
 * `Student billy(60, 70, 80); // Constructor call`
 * `Student billy {60, 70, 80}; // C-style direct initialization`
 * `int a[5] = {1, 2, 3, 4, 5};`

From 2011:
 * `{}` is set to the default initialization method.
  * `int x{5}; // Works`
  * `string s{"hello"}; // Works`
  * `ifstream f{"hello.txt"}; // Works`

##### Advantages of Constructors:
 * Default parameters, overloading, sanity checks
 ``` c++
 struct Student{
   Student (int assns = 0, int mt = 0, int final = 0){
     this->assns = assns;
     this->mt = mt;
     this->final = final;
   }
 };

 Student jane{70, 80};  // 70, 80, 8
 Student newKid;        // 0, 0, 0
 ```
 * **Note**: Every class comes with a built-in default constructor (i.e. takes no args).
  * This just default constructs all fields that are objects.
  * For non-objects, the default constructor does nothing.
   ``` c++
   Vec V; // Does nothing
   ```
  * The built in constructor goes away if you provide any constructor.
   ``` c++
   struct Vec {
     int x, y;
     Vec (int x, int y){
       this->x = x;
       this->y = y;
     }
   };
   Vec v;       // Error! no default constructor.
   Vec v{1, 2}; // Works
   ```
 * What if struct contains constants or references?
   ``` c++
   struct MyStruct {
     const int myConst;     // Must be Initialized
     int &myRef;            // Must be Initialized
   };
   ```
  * So initialize it:
    ``` c++
    int z;
    struct MyStruct {
      const int myConst = 5;     // Must be Initialized
      int &myRef = z;            // Must be Initialized
    };
    ```
 * But does every instance of MyStruct need to have the same value of MyConst?
   ``` c++
   Struct Student {
     const int id;  // constant (i.e. doesn't change), but not the same for every student
   };
   ```
 * Then where do we initialize?
  * Constructor Body - Too late as fields must be fully constructed by then.
 * When an object is created:
  1. Space is allocated
  2. Fields are constructed in declaration order
  3. Constructor body runs

##### Member Initialization List (MIL):
```c++
struct Student{
 const int id;
 int assns, mt, final;
 Student (int id, int assns, int mt, int final):
  id{id}, assns{assns}, mt{mt}, final{final} {}
 // The outer id, assns, etc. have to be fields by the rules so it doesn't use the internal scope
 // The inner id, etc. is the parameter from the internal scope.
 // We may initialize any field this way. Not just Consts and Refs
};
```

**Example**: Not using the MIL
``` c++
struct Student{
  string name;
  Student(string name){
    this->name = name;
  }
};
```
 * String is default constructed to `""` in step 2. it is then assigned the value in step 3.
 * Using the MIL makes the initialization only occur once (step 2 it is assigned the parameter value).

**Note**: Fields are initialized in *the order in which they are declared in class*. Even if the MIL orders them differently.

**Embrace the MIL!**
 * What if a field is defined inline AND in the MIL?
   ``` c++
   struct Vec {
     int x = 0, y = 0;    // Only happens if the field is not mentioned in the MIL
     Vec (int x): x{x} {}
   }
   ```
  * The MIL takes precedence.

**Example**: Now consider this
``` c++
Student billy {60, 70, 80};
Student bobby = billy;      // How does this initialize?
```
 * The **Copy Constructor** is used for constructing on e object as a copy of another.

**Note**: Every class comes with:
 1. Default constructor (default constructs all objects)
  * Lost if you write any constructors
 2. Copy constructor (just copies all fields)
 3. Copy assignment operator
 4. Destructor
 5. Move constructor
 6. Move assignment operator

Consider:
```cpp
Node *n = new Node{1, new Node{2, new Node{3, nullptr}}}
Node m = *n
Node *p = new Node{*n}
```
  *  pointer n is on the stack, points to a linked list on the heap
  * pointer m is on the stack, points to what n points to
  * pointer p is on the stack, points to ***a copy of the first node*** (its a shallow copy)
  * if you want a *deep* copy, i.e. copies the whole list, then you must write your own copy constructor:

```cpp
struct Node {
  Node (const Node &other): data { other.data }, // must pass other by ref, by value would call the copy constructor again and be infinite recursion
    next { other.next ? new Node { *other.next } : nullptr } // we use *other.next because other.next is not a node, its a ptr
};
```

The copy constructor is called:
  1. when an object is initialized with another object
  2. when an object is passed by value
  3. when an object is returned by a function

*Note: all of these are only sometimes true, there are exceptions!!!!*

**Beware!** of constructors that can take *one* argument.
e.g:
```
struct Node {
  Node (int data): data { data { next { nullptr }}}
}
```
Single-arg constructors create implicit conversions
e.g. Node n{4};
but also Node n = 4 <- implicit conversion from int to Node
string s = "hello" <- implicit conversion from char* to string

So what's the problem:
```
int f(Node n) { ... }
f(4); // works - 4 implicitly converted to a Node
```
  * danger: accidentally pass an int to a function expecting a Node
  * silent conversion - no error message
  * potential errors not caught
  * disable it by making the constructor explicit

```
Node n {4}; // OK!
Node n = 4; // NOT OK!
f(4); // NOT OK!
```

# Destructors
When an object is destroyed:
  * stack allocated: goes out of scope
  * heap allocated: is destroyed

A method calls the *destructor* runs
  1. destructor body runs
  2. fields' destructors (if they are objects) are called *in reverse declaration order*
  3. space is deallocated

As before, classes come with a destructor (just calls destructors for all fields that are objects)
When do we need to write a destructor???
  * i.e. llists. if the pointer goes out of scope all the stuff on the heap is LEAKED (but the pointer isn't)

```cpp
struct Node {
  ~Node() { delete next; } // deletes next, recursively calls *next destructor therefore the whole list is deallocated
}

```

## Copy Assignment Operator
```cpp
Student bill { 60, 70, 80 }
Student jane = billy; // copy constructor
Student joey; // Default constructor 0, 0, 0
joey = bill; // Copy, but not a constructor - uses copy assignment operator (default one is SHALLOW)
```
May need to write our own:
```cpp
❌❌
struct Node {
  Node &operator = (const Node &other) {
    data = other.data
    next = other.next ? new Node{*other.next} : nullptr;
    return *this;
  }
}
```
WHY IS THIS BAD?
```cpp
Node n {1, newNode{2, newNode{3, nullptr}}}
n = n // deletes n, then tries to copy n to n, undefined behaviour

*p = *q;
a[i] = a[j];
```

```cpp
struct Node {
  Node &operator (const Node &other) {
    data = other.data;
    delete next;
    next = other.next ? new Node{*other.next} : nullptr;
    return *this;
  } // Dangerous?? n = n
}
```

```cpp
struct Node {
  ...
  Node &operator  = (const Node &other) {
    if (this == &other) return * this;
    data = other.data;
    delete next;
    next = other.next ? new Node {* other.next} : nullptr;
    return * this;
  }
}
```
* what if new fails? i.e. there's no more memory
* then next would be pointed to deleted memory, i.e. you have a corrupted data structure

#### better
```cpp
Node &operator = (const Node &other) {
  if (this == other) return * this;
  Node &tmp = next;
  data = other.data;
  next = other.next ? new Node{* other.next} : nullptr;
  delete tmp;
  return * this;
}
```
* if new fails -- operator aborts
* next points at undeleted old nodes - data structure is not corrupted

#### alternative - copy and swap idiom
```cpp
#include <utility>
struct Node {
  ...
  void swap(Node &other) {
    using std::swap;
    swap(data, other.data);
    swap(next, other.next);
  }

  Node &operator=(const Node &other) {
    Node tmp = other; // Calls the copy constructor
    swap(tmp); // "this" holds a deep copy of other
    return * this; // tmp goes out of scope here, destructor runs on tmp and frees the old data
  }
}
```

#### Notice: operator= is a member fn, not a standalone fn
* when an operator is declared as a member fn, it has one less argument
* \*this plays the role of the first operand, eg:

```cpp
struct Vec {
  int x, y;
  ...
  Vec operator+(const Vec &other) {
    return {x + other.x, y + other.y};
  }
  Vec operator*(const int k) {
    return { x * k, y * k }
  }
}
```
* note: this is the scalar operand for v \* k NOT k \* v
* so how would I implement k \* v?? **you can't as a member fn** because the first arg is not a `Vec`. Must be standalone:

```cpp
Vec operator*(const int k, const Vec &v) {
  return v * k;
}
```

#### What about I/O operators?
```cpp
// What's wrong with this ? ❌
// - will it compile? ya
struct Vec {
  ostream &operator<<(ostream &out) {
    return out << x << ' ' << y;
  }
}
```
* it makes Vec the LHS operand, not the RHS
  * use as `v << cout` ❌ confusing
* MORAL: define operators input and output as stand alones

#### certain operators MUST be members
* `operator=`
* `operator[]`
* `operator->`
* `operator()`
* `operator T` where T is a type

## Separate Compilation For Classes

```cpp
// Node.h
#ifndef NODE_H
#define NODE_H
struct Node {
  int data;
  Node *next;
  explicit Node(int data, Node *next = nullptr);
  bool isSingleton();
}
```

```cpp
// Node.cc
#include "Node.h"
  Node::Node (int data, Node *next): data{data}, next{next}{}
  bool Node::isSingleton() { return next == nullptr }
```
* `::` is called the scope resolution operator
* `Node::_______` means `______` in the context of struct node

## Const objects

```cpp
int f(const Node &n) { ... }
```
* const objects arise often, especially as Params
* what is a const object?
  * fields cannot be modified

* can we call methods on a const object?

*Issue: the method may modify fields, which would violate the const*
* Answer: yes, you can call methods on a const object. (strings attached)
  * we can call methods that promise not to modify fields
  * eg:

```cpp
struct Student {
  int assns, mt, final;
  float grade() const; // doesn't modify fields, so DECALRE IT CONST
};
float Student::grade() const {-----}
// Compiler checks that const methods don't modify fields
```
* only const methods may be called on const objects ⭐

## Rvalues and Rvalue references
Recall:
* an lvalue is anything with an address
* an lvalue reference is like a const pointer with automatic dereferencing
  * always initialized to an lvalue

Now consider:
```cpp
Node n {l, new Node{2, nullptr}};
Node m = n; // calls the copy constructor
m2 = n; // calls the copy assignment operator

Node plusOne(Node n) { // copy constructor on n on the way in ⭐
  for (Node * p = &n; p; p = p->next) {
    ++p-> data;
    return n; // copy constructor on n on the way out ⭐
  }
}
Node m3 = plusOne(n); // This is also the copy constructor
```
* in the cpy constructor, "other" is the plusOne(n) part.
* the compiler create a *temporary object* to hold the result of plusOne(n)
* so other is a reference to this temporary object
* copy constructor deep copies the data from this temporary

But - the temporary is just gonna be discarded anyways
* wasteful to deep copy, why not just steal the data?
* but how can you tell whether 'other' is a reference to a temporary or a standalone object

**RVALUE REFERENCES**
* `Node &&` is a reference to a temporary object (rvalue) of type Node
* So, we should write a version of the constructor that takes an rvalue reference (Node &&)

```cpp
struct Node {
  ...
  Node(Node && other): data{other.data}, next{other.next}{}// called a move constructor
}
```

# New lecture

*Recall: move ctor*
```cpp
struct Node {
  Node (Node &&other): data(other.data), next(other.next) {
    other.next = nullptr;
  }
}
```

Similarly:
```cpp
Node m;
m = plusOne(n);

// Move assignment operator
Struct Node {
  ...
  // Needs to steal others data
  // Needs to destroy my old data
  Node &operator=(Node &&other) { // steal other's data
      using std::swap;
      swap(data, other.data);
      swap(next, other.next);
      return * this; // temp will be destroyed, takes the old data with it
  }
}
```

* if you don't write a move constructor and a move assignment, the copy version will be used
* if the move constructor/assignment is defined, it will replace all calls to the copy constructor / assignment operator when the argument is a temporary

### Copy/Move Elision (to omit)
```cpp
Vec makeAVec() {
  return {0, 0}; // invokes basic constructor
}

Vec v = makeAVec(); // What runs??
                    // Copy ctor? (if Vec has no move ctor)
                    // Move ctor? (if there exists one)
```
* g++ - basic ctor only - no copy, no move
* in some circumstances, the compiler is allowed to skip calling copy and move ctors, but doesn't have to
* in the example above, `makeAVec` writes its result ({ 0, 0 }) directly into the sapce occupied by v in the caller, rather than copy it later

```cpp
void doSomething(Vec v) { ... } // pass by value - copy or move ctor

doSomething(makeAVec());
// Result of makeAVec() is written directly into the param - no copy or move
```

* this is allowed, even if dropping constructor calls would change the behavior of the program
* good news: you are nOt eXpEcTeD to KNow eXActlY wHen tHe ElisION iS aLLowEd
  * just know that's its possible
* if you need all ctors to run:
  * g++14 -fno-elide-constructors file.cc
  * this can slow your program, will cause thousands of extra fnction calls

#### ***In summary: Rule of 5 (big 5)*** ⭐ ⭐ ⭐

If you need to customize any one of:

  1. copy ctors
  2. copy assignment
  3. dtor
  4. move ctor
  5. move assignment

Then you *usually* need to customize all 5.

### Arrays of objects
```cpp
struct Vec {
  int x, y;
  Vec (int x, int y): x{x}, y{y} {};
}

Vec *vp = new Vec[15]; // On the heap
Vec moreVecs[10]; // On the stack
// THESE WILL NOT ❌ COMPILE, UNCONSTRUCTED OBJECTS
```
* c++ needs to call constructor on each item, so it looks for default constructor, but there is none for Vec and therefore theres an error
* Options to fix:
  1. provide a default constructor, then the program compiles
  2. For stack arrays:
    * `Vec moreVecs[] = {Vec{0, 0}, Vec{1, 1}, Vec{2, 3}}`
  3. For heap arrays, create an array of pointers:
```cpp
Vec **vp = new Vec*[3];
v[0] = new Vec{0, 0};
v[1] = new Vec{1, 2};
 ....
// WHEN YOUR DONE, MUST DELETE
for (int i = 0; i < 3; i++) {
  delete vp[i];
}
delete [] vp;
```

# Invariants & encapsulation
```cpp
struct Node {
  int data;
  Node *next;
  Node (int data, Node *next); // fill in urself
  ...
  ~Node() { delete next; }
};*

Node n1 {1, new Node {2, nullptr}};
Node n2 {3, nullptr};
Node n3 {4, &n2};
```
* question: what happens when n1,n2,n3 go out of scope?
  * n1: dtor runs, deletes the whole list
  * n2 and n3: n3's destructor tries to delete n2, but n2 is on the stack, not the heap. ❌
    * if you call delete on a ptr on the stack, then undefined behavior happens

Node relies on an assumption for its proper operation: that next is either a`nullptr` or a valid ptr to the heap, allocated by `new`. ***This is an invariant***.

An invariant is a statement that always holds true. (the invariant is that next is either `nullptr` or allocated by `new` - upon which node relies).

But we can't guarantee the invariant - can't trust the user to use node properly.

Right now, can't enforce any invariants - the user can interfere with our data.
  * e.g. stack: invariant is that the last item pushed is the first item popped
    * but not if the client can rearrange the underlying data~!
    * i.e. it is very hard to reason about programs without invariants ⭐

### Enforcing invariants: encapsulation!

We want to treat our objects as black boxes - capsules.

* implies details sealed away
* can only interact via provided methods
* creates an abstraction - regains control over our objects

E.g.
```cpp
struct Vec{
  Vec(int x, int y);
private: // can't be accessed outside the struct
  int x, y;
public:
  Vec operator+(const Vec &other);
}`
```
* if you don't say anything, it's public
* in general - fields should be private; only methods should be public.
* better to have default visibility = private
  * switch from struct to class:

```cpp
class Vec {
  int x, y;
public:
  Vec(int x, int y);
  Vec operator+(const Vec &v);
}`
```

***The difference between `class` and `struct` is default visibility: public in struct, private in class***

********

# Mandatory CS Tutorial
```bash
makefile v2
          CXX=g++
          CXXFLAGS=-std=c++14 -Wall -g -Werror
        /
targets
      \
      book.o: book.cc book,h
      _ ${CXX} ${CXXFLAGS} -c book.cc

EXEC=main
OBJECTS=main.o book.o comic.o text.o
DEPENDS=${OBJECTS:.o=.d}
${EXEC}: ${OBJECTS}
  ${CXX} ${CXXFLAGS} ${OBJECTS} -o ${EXEC}
-include ${DEPENDS}
clean:
  rm ${OBJECTS} ${DEPENDS} ${EXEC}
```
#### ⭐see tutorial5.pdf⭐

Lvalues
  * permenant
  * can be anywhere
  * have an address
  ```cpp
  int n = 10;
  5 = n; ❌
  n = 5;
  int m = n;
  ```
Rvalues
  * temps
  * can only be on the RIGHT of expressions

********
# missed lect on iterator pattern. see [dzed](http://dzed.me/notes/2016/05/02/Cs-246.html)
********

*Recall*: accessors + mutators
```cpp
class Vec {
  int x, y;
public:
  int getX() const { return x } // accessor
  void setT(int x) { y = x; } // mutator
}
```

What about `<<`?
  * needs x + y, but can't be a member
  * if getX, getY defined, then you're ok.

```cpp
class Vec {
  ...
  friend ostream &operator<<(ostream &out, const vec &v);
};

ostream &operator<<(ostream &out, const vec &v) {
  return out << v.x << ' ' << v.y;
}
```
