# CPlusPlus-Quiz
Based on the site http://cppquiz.org/, extracted questions for easier overview.

The more :skull: you see, the higher the difficulty!

According to the C++17 standard, what is the output of the following programs? 
 
#### 1. :skull::skull:   
```
#include <iostream>

int main() {
  int a = 10;
  int b = 20;
  int x;
  x = (a, b);
  std::cout << x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 20

The comma operator is applied on two expressions: a and b.

According to [expr.comma](https://timsong-cpp.github.io/cppwp/n4659/expr.comma#1)§8.19¶1 in the standard: "A pair of expressions separated by a comma is evaluated left-to-right; the left expression is a discarded-value expression (...) The type and value of the result are the type and value of the right operand".

The right operand here being b, with the value 20. This is then the resulting value of the expression (a, b), and 20 is assigned to x.
</p>
</details>

#### 2. :skull: 

```
#include <iostream>

class A {
public:
  A() { std::cout << 'a'; }
  ~A() { std::cout << 'A'; }
};

class B : public A {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
};

int main() { B b; }
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: abBA

The derived class constructor B() first calls the base class constructor A(), and then executes the compound-statement (the part inside {}) of its own body.

The derived class destructor ~B() first executes its own body, and then calls the base class destructor ~A().

Let's have a look at the standard:

Initialization order is defined in [class.base.init](https://timsong-cpp.github.io/cppwp/n4659/class.base.init#13)§15.6.2¶13:

"In a non-delegating constructor, initialization proceeds in the following order:
- (...)  
- Then direct base classes are initialized in declaration order as they appear in the base-specifier-list (regardless of the order of the mem-initializers)  
- (...)  
- Finally, the compound-statement of the constructor body is executed."  

Destruction order is defined in [class.dtor](https://timsong-cpp.github.io/cppwp/n4659/class.dtor#9)§15.4¶9:

"After executing the body of the destructor and destroying any automatic objects allocated within the body, a destructor for class X calls (...) the destructors for X’s non-virtual direct base classes"  

</p>
</details>

#### 3. :skull:
```
#include <iostream>
struct X {
  virtual void f() const { std::cout << "X"; }
};

struct Y : public X {
  void f() const { std::cout << "Y"; }
};

void print(const X &x) { x.f(); }

int main() {
  X arr[1];
  Y y1;
  arr[0] = y1;
  print(y1);
  print(arr[0]);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: YX

arr is an array of X, not of pointers to X. When an object of type Y is stored in it, it is converted to X, and we lose the "Y part" of the object. This is commonly known as "slicing".
</p>
</details>

#### 4. :skull::skull:   
```
#include <iostream>

using namespace std;

template<typename T>
void f(T) {
    cout << 1;
}

template<>
void f(int) {
    cout << 2;
}

void f(int) {
    cout << 3;
}

int main() {
    f(0.0);
    f(0);
    f<>(0);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 132

Here we have three calls to f. Which function is selected for each call? [temp.over](https://timsong-cpp.github.io/cppwp/n4659/temp.over#1)§17.8.3¶1 is surprisingly readable:

    A function template can be overloaded either by (non-template) functions of its name or by (other) function templates of the same name. 

The function template f is in our case overloaded by the non-template void f(int). It goes on:

    When a call to that name is written [...], template argument deduction [is] performed for each function template to find the template argument values (if any) that can be used with that function template to instantiate a function template specialization that can be invoked with the call arguments

So in the tree cases above, we first do template argument deduction, giving double, int and int, respectively.

Then:

    For each function template, if the argument deduction and checking succeeds, the template-arguments [...] are used to synthesize the declaration of a single function template specialization which is added to the candidate functions set to be used in overload resolution

So in all the three cases above, a specialization of the function template f is added to the candidates to overload resolution, with T equal to double, int and int, respectively.

Now look at each call:

For f(0.0), T is deduced as double, and a specialization void f(double) is added as a candidate for overload resolution. The other candidate is the non-template function void f(int). The one taking a double is a better match, and 1 is printed.

For f(0), T is deduced as int, and a specialization void f(int) is added as a candidate for overload resolution. The other candidate is the non-template function void f(int). Both are equally good matches, so is the program ill-formed?

[over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)§16.3.3¶1>

    F1 is defined to be a better function than another viable function F2 if [...]
    F1 is not a function template specialization and F2 is a function template specialization

void f(int) is not a function template specialization, and template<> void f(int) is, so the former is selected, and 3 is printed.

Finally we get to f<>(0). <> is an (empty) template argument list, which can be specified for template functions, but not for non-template functions. So in this case, the non-template function is not an option, we explicitly ask for the function template. This is also described in a note in [temp.arg.explicit](https://timsong-cpp.github.io/cppwp/n4659/temp.arg.explicit#4)§17.8.1¶4:

    An empty template argument list can be used to indicate that a given use refers to a specialization of a function template even when a non-template function is visible that would otherwise be used.

T is deduced to int, the specialization for int is the only candidate, and 2 is printed.
</p>
</details>

#### 5. :skull::skull::skull: 

```
#include <iostream>

int a = 1;

int main() {
    auto f = [](int b) { return a + b; };

    std::cout << f(4);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 5  

It is clear that a is not captured explicitly, so the question is whether a should be captured implicitly (and if yes, then a default capture has to be specified). [expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#7)§8.1.5.2¶7 says:

    A lambda-expression with an associated capture-default that does not explicitly capture *this or a variable with automatic storage duration [...], is said to implicitly capture the entity (i.e., *this or a variable) if the compound-statement:  
    - odr-uses the entity (in the case of a variable),  
    - [...]  

The use of a constitutes an odr-use. But since a has static storage duration rather than automatic storage duration, it is not implicitly captured.

Since a is neither explicitly nor implicitly captured, a in the lambda expression simply refers to the global variable a.

(Note: It is also disallowed to capture a explicitly, because of [expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#4)§8.1.5.2¶4):

    The identifier in a simple-capture is looked up using the usual rules for unqualified name lookup; each such lookup shall find an entity. An entity that is designated by a simple-capture is said to be explicitly captured, and shall be *this (when the simple-capture is “this” or “* this”) or a variable with automatic storage duration declared in the reaching scope of the local lambda expression.
</p>
</details>

#### 6. :skull::skull: 

```   
#include <iostream>

int main() {
    int i = '3' - '2';
    std::cout << i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
'3' and '2' are both ordinary character literals, with type char. But we don't know which values they have! That's up to the implementation:

[lex.ccon](https://timsong-cpp.github.io/cppwp/n4659/lex.ccon#2)§5.13.3¶2:

    An ordinary character literal that contains a single c-char representable in the execution character set has type char, with value equal to the numerical value of the encoding of the c-char in the execution character set.

operator- subtracts the value of the second operand from the value of the first operand. But how can we know what the result is, when we don't know the values of the operands?

[lex.charset](https://timsong-cpp.github.io/cppwp/n4659/lex.charset#3)§5.3¶3:

    In both the source and execution basic character sets, the value of each character after 0 in the above list of decimal digits shall be one greater than the value of the previous.

(The "above list of decimal digits" is simply 0123456789.)

So we know that the difference between '3' and '2' is 1, even if we don't know their actual values. Importantly, this is only true for decimal digits, and not for instance for regular letters. There is no guarantee that 'b' - 'a' is 1.
</p>
</details>

#### 7. :skull::skull:

```
#include <iostream>

struct A {
  A() { foo(); }
  virtual ~A() { foo(); }
  virtual void foo() { std::cout << "1"; }
  void bar() { foo(); }
};

struct B : public A {
  virtual void foo() { std::cout << "2"; }
};

int main() {
  B b;
  b.bar();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 121

Even though foo() is virtual, it is not considered to be so during the execution of constructors and destructors.

Rationale:

If an object of type B is being constructed, first the constructor of A is called, then the constructor of B. Thus, during A's constructor, the "B part" of the object has not been constructed yet, and should not be used. One could easily imagine that B::foo() would use the "B part" of the object, so it would be dangerous for A's constructor to call it.

When the object is destroyed, B's destructor is called first, then A's destructor, leading to the same problem.
</p>
</details>

#### 8. :skull:

```
#include <iostream>
using namespace std;

int foo() {
  cout << 1;
  return 1;
}

void bar(int i = foo()) {}

int main() {
  bar();
  bar();
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 11

Is foo called both times or just once? The C++ standard says this in [dcl.fct.default](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#9)§11.3.6¶9: "A default argument is evaluated each time the function is called with no argument for the corresponding parameter."

Thus, foo is called twice.

</p>
</details>

#### 9. :skull::skull:
```
#include <iostream>

int main() {
  int a = 0;
  decltype((a)) b = a;
  b++;
  std::cout << a << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 11

According to [dcl.type.simple](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.simple#4)§10.1.7.2¶4 in the C++ standard:
"The type denoted by decltype(e) is deﬁned as follows:
- if e is an unparenthesized id-expression naming a structured binding ([dcl.struct.bind](https://timsong-cpp.github.io/cppwp/n4659/dcl.struct.bind)§11.5), decltype(e) is the referenced type as given in the specification of the structured binding declaration;  
— if e is an unparenthesized id-expression or an unparenthesized class member access ([expr.ref](https://timsong-cpp.github.io/cppwp/n4659/expr.ref)§8.2.5), decltype(e) is the type of the entity named by e. If there is no such entity, or if e names a set of overloaded functions, the program is ill-formed;  
— otherwise, if e is an xvalue, decltype(e) is T&&, where T is the type of e;  
— otherwise, if e is an lvalue, decltype(e) is T&, where T is the type of e;  
— otherwise, decltype(e) is the type of e."  

Because a is encapsulated in parentheses, it doesn't qualify for the first case, it is treated as an lvalue, therefore b's type is int&, not int.

</p>
</details>

#### 10. :skull:

```
#include <iostream>
     
int main() {
    char* str = "X";
    std::cout << str;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 

According to [lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#8)§5.13.5¶8 in the standard: "A narrow string literal has type “array of n const char”"

An array of n const char converts to a pointer to const char. A note in [conv.qual](https://timsong-cpp.github.io/cppwp/n4659/conv.qual#4)§7.5¶4 extrapolates from the preceeding normative passages that "a prvalue of type “pointer to cv1 T” can be converted to a prvalue of type “pointer to cv2 T” if “cv2 T” is more cv-qualified than “cv1 T”." In this case however, char* is less cv-qualified than const char *, and the conversion is not allowed.

Note: While most compilers still allow char const[] to char* conversion with just a warning, this is not a legal conversion since C++11.

See also http://dev.krzaq.cc/stop-assigning-string-literals-to-char-star-already/
</p>
</details>

#### 11. :skull::skull::skull:

```
#include <iostream>

struct X {
  X() { std::cout << "X"; }
};

struct Y {
  Y(const X &x) { std::cout << "Y"; }
  void f() { std::cout << "f"; }
};

int main() {
  Y y(X());
  y.f();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 

The compilation error is on the line y.f(), but the source of the problem is Y y(X());

This could be interpreted as a a variable definition (which was the intention of the programmer in this example), or as a definition of a function y, returning an object of type Y, taking a function (with no arguments, returning an object of type X) as its argument.

The compiler is required by the standard to choose the second interpretation, which means that y.f() does not compile (since y is now a function, not an object of type Y).

Wikipedia has a concise explanation: http://en.wikipedia.org/wiki/Most_vexing_parse, and the standard has more in [stmt.ambig](https://timsong-cpp.github.io/cppwp/n4659/stmt.ambig)§6.8.

To fix the problem, change Y y(X()) to either Y y{X{}} (modern C++) or Y y((X())) (pre-C++11)
</p>
</details>

#### 12. :skull::skull:

```
#include <iostream>
#include <utility>

int y(int &) { return 1; }
int y(int &&) { return 2; }

template <class T> int f(T &&x) { return y(x); }
template <class T> int g(T &&x) { return y(std::move(x)); }
template <class T> int h(T &&x) { return y(std::forward<T>(x)); }

int main() {
  int i = 10;
  std::cout << f(i) << f(20);
  std::cout << g(i) << g(20);
  std::cout << h(i) << h(20);
  return 0;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 112212

The T&& in the templated functions do not necessarily denote an rvalue reference, it depends on the type that is used to instantiate the template. If instantiated with an lvalue, it collapses to an lvalue reference, if instantiated with an rvalue, it collapses to an rvalue reference. See note [1].

Scott Meyers has written a very good article about this, where he introduces the concept of "universal references" (the official term is "forwarding reference") http://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers

In this example, all three functions are called once with an lvalue and once with an rvalue. In all cases, calling with an lvalue (i) collapses T&& x to T& x (an lvalue reference), and calling with an rvalue (20) collapses T&& x to T&& x (an rvalue reference). Inside the functions, x itself is always an lvalue, no matter if its type is an rvalue reference or an lvalue reference.

-For the first example, y(int&) is called for both cases. Output: 11.  
-For the second example, move(x) obtains an rvalue reference, and y(int&&)is called for both cases. Output: 22.  
-For the third example, forward<T>(x) obtains an lvalue reference when x is an lvalue reference, and an rvalue reference when x is an rvalue reference, resulting in first a call to y(int&)and then a call to y(int&&). Output: 12.  

Note [1]: [dcl.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.ref#6)§11.3.2¶6 in the standard: "If a typedef-name (§10.1.3, §17.1) or a decltype-specifier (§10.1.7.2) denotes a type TR that is a reference to a type T, an attempt to create the type “lvalue reference to cv TR” creates the type “lvalue reference to T”, while an attempt to create the type “rvalue reference to cv TR” creates the type TR." The example at the end of that paragraph is worth a look.

Note from the contributor: This demonstrates Scott Meyers's advice to use std::forward for forwarding references, and std::move for rvalue references.

</p>
</details>

#### 13. :skull::skull:

```
#include <iostream>

struct E
{
  E() { std::cout << "1"; }
  E(const E&) { std::cout << "2"; }
  ~E() { std::cout << "3"; }
};

E f()
{ 
  return E();
}

int main()
{
  f();
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 13

In f(), an E object is constructed, and 1 is printed. This object is then returned to main(), and one could expect the copy constructor to be called, printing 2.

However, E() is a prvalue and as such does not constitute an object just yet by [basic.lval](https://timsong-cpp.github.io/cppwp/n4659/basic.lval#1)§6.10¶1

    A prvalue is an expression whose evaluation initializes an object or a bit-field, or computes the value of the operand of an operator, as specified by the context in which it appears.

A prvalue only creates a temporary when needed, for instance to create an xvalue. In those cases, a temporary materialization conversion happens ([conv.rval](https://timsong-cpp.github.io/cppwp/n4659/conv.rval#1)§7.4¶1). In this case however, no temporary is needed, and none is created.
A pr
[stmt.return](https://timsong-cpp.github.io/cppwp/n4659/stmt.return#2)§9.6.3¶2 says:

    (...) the return statement initializes the glvalue result or prvalue result object of the (explicit or implicit) function call by copy-initialization from the operand.

And copy-initialization for a class-type by [dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#17)§11.6¶17 goes through:

    If the initializer expression is a prvalue and the cv-unqualified version of the source type is the same class as the class of the destination, the initializer expression is used to initialize the destination object.

Which means that no copy or move constructor is called at all. This implies that the copy and move constructor could very well be deleted, and the code would still compile just fine.

The output is thus 13 because of the constructor followed by the destructor call.
</p>
</details>

#### 14. :skull::skull:
```
#include <iostream>

void f(unsigned int) { std::cout << "u"; }
void f(int)          { std::cout << "i"; }
void f(char)         { std::cout << "c"; }

int main() {
    char x = 1;
    char y = 2;
    f(x + y);
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is unspecified / implementation defined 

The type of the sum of two chars is actually not uniquely specified. We do however know it's not char.

Before being passed to operator +, the operands (x and y) go through a conversion. [expr.add](https://timsong-cpp.github.io/cppwp/n4659/expr.add#1)§8.7¶1:

    The additive operators + and - group left-to-right. The usual arithmetic conversions are performed for operands of arithmetic or enumeration type.

What are "the usual arithmetic conversions"?

[expr](https://timsong-cpp.github.io/cppwp/n4659/expr#11)§8¶11:

    Many binary operators that expect operands of arithmetic or enumeration type cause conversions and yield result types in a similar way. The purpose is to yield a common type, which is also the type of the result. This pattern is called the usual arithmetic conversions, which are defined as follows:
    - [a bunch of rules for floats, enums etc]  
    - Otherwise, the integral promotions (7.6) shall be performed on both operands  

So both chars go through integral promotions. Those are defined in [conv.prom](https://timsong-cpp.github.io/cppwp/n4659/conv.prom#1)§7.6¶1:

    A prvalue of an integer type other than bool, char16_t, char32_t, or wchar_t whose integer conversion rank (7.15) is less than the rank of int can be converted to a prvalue of type int if int can represent all the values of the source type; otherwise, the source prvalue can be converted to a prvalue of type unsigned int.

So a char gets converted to an int if int can fit all possible values of a char. But that's not necessarily the case!

First, int could actually be the same size as char. [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#2)§6.9.1¶2:

    There are five standard signed integer types : “signed char”, “short int”, “int”, “long int”, and “long long int”. In this list, each type provides at least as much storage as those preceding it in the list.

Note that it says "at least as much storage", it doesn't have to be more. So for instance you could have an sixteen bit system where both char and int are sixteen bits.

Second, char can be either signed or unsigned, it's up to the implementation: [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#1)§6.9.1¶1:

    It is implementation-defined whether a char object can hold negative values.

int is signed, so if char is also signed, all possible values of char will fit in an int. However, if char is unsigned, and int and char is the same size, char can actually hold larger values than int! In the former case, chars get promoted to ints, but in the latter case, chars get promoted to unsigned int before being summed.

So in practice, most systems will call f(int) and print i, but some might call f(unsigned int) and print u, and they would both be confirming to the standard.
</p>
</details>

#### 15. :skull:
```
#include <iostream>

struct GeneralException {
  virtual void print() { std::cout << "G"; }
};

struct SpecialException : public GeneralException {
  void print() override { std::cout << "S"; }
};

void f() { throw SpecialException(); }

int main() {
  try {
    f();
  }
  catch (GeneralException e) {
    e.print();
  }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: G
We throw a SpecialException. It is derived from GeneralException, but is caught by value, so e will have the dynamic type GeneralException, not SpecialException. This is known as slicing.

Instead, we should have caught it by reference catch (GeneralException& e), then its dynamic type would be SpecialException, and the program would output S.

</p>
</details>

#### 16. :skull:
```
#include <iostream>

int a;

int main () {
    std::cout << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
Since a has static storage duration and no initializer, it is guaranteed to be zero-initialized. Had a been defined as a local non-static variable inside main(), this would not have happened.

Note: int a has static storage duration because it is declared at namespace scope. It does not need to have static in front of it, that would only denote internal linkage.
</p>
</details>

#### 17. :skull::skull:
```
#include <iostream>
using namespace std;

class C {
public:
  explicit C(int) {
    std::cout << "i";
  };
  C(double) {
    std::cout << "d";
  };
};

int main() {
  C c1(7);
  C c2 = 7;
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: id
These are two examples of initialization. The first form, C c1(7), is called direct-initialization, the second, C c2 = 7, is called copy-initialization. In most cases they are equivalent, but in this example they are not, since the int constructor is explicit.

The key is in [over.match.copy](https://timsong-cpp.github.io/cppwp/n4659/over.match.copy#1)§16.3.1.4¶1 :
"[...] as part of a copy-initialization of an object of class type, a user-defined conversion can be invoked to convert an initializer expression to the type of the object being initialized. [...] the candidate functions are selected as follows:  
- [...]  
- When the type of the initializer expression is a class type "cv S", the non-explicit conversion functions of S and its base classes are considered. [...]  
(emphasis added)

And how is direct-initialization defined?

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)§11.6¶16: "The initialization that occurs in the forms
T x(a);
T x{a};
(...) is called direct-initialization."

So the int constructor is not even considered for initialization in the second case. Instead, a standard conversion sequence is used to convert the integer literal to a double, and the double constructor (the only candidate) is used.

</p>
</details>

#### 18. :skull:
```
#include <iostream>

class A {
public:
  A() { std::cout << 'a'; }
  ~A() { std::cout << 'A'; }
};

class B {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
  A a;
};

int main() { B b; }
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: abBA
Member variables are initialized before the constructor is called. The destructor is called before member variables are destroyed.
</p>
</details>

#### 19. :skull:
```
#include <iostream>

int j = 1;

int main() {
  int& i = j, j;
  j = 2;
  std::cout << i << j;
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 12
There are two variables named j here. The first is the global one, the second is local to main. A variable is in scope from the point of its declaration until the end of the region in which it was declared, except for when another variable with the same name is in scope.

It's easiest to first look at the scope of the local j declared inside main, which extends from the , to the }. The scope of the global j is from when it was declared until the end of the program, except for when the local j is in scope.

So the reference i refers to the global j, since the local j is not yet in scope. When we set j=2, we modify the local j, and i is not affected.

This example is almost identical to [basic.scope.declarative](https://timsong-cpp.github.io/cppwp/n4659/basic.scope.declarative#2)§6.3.1¶2 in the C++ standard, which has the following explanation:

    the identifier j is declared twice as a name (and used twice). The declarative region of the first j includes the entire example. The potential scope of the first j begins immediately after that j and extends to the end of the program, but its (actual) scope excludes the text between the , and the }. The declarative region of the second declaration of j (the j immediately before the semicolon) includes all the text between { and }, but its potential scope excludes the declaration of i. The scope of the second declaration of j is the same as its potential scope.
</p>
</details>

#### 20. :skull::skull:
```
#include <variant>
#include <iostream>
 
using namespace std;

int main() {
   variant<int, double, char> v;
   cout << v.index();
}
```
<details><summary><b>Answer</b></summary>
<p>

####  The program is guaranteed to output: 0
std::variant can hold a value of any one of its alternative types, or no value. To refer to the alternative types, it uses an index i.
[variant.ctor](https://timsong-cpp.github.io/cppwp/n4659/variant.ctor#1)§23.7.3.1¶1 in the standard:

    let i be in the range [0, sizeof...(Types)), and Ti be the ith type in Types....

In our case, T0 means int, T1 means double, and T2 means char.

Now what happens if you define a variant without initializing it with a certain type? The default constructor will pick the type T0, in our case int, and value-initialize it.
[variant.ctor](https://timsong-cpp.github.io/cppwp/n4659/variant.ctor#2)§23.7.3.1¶2:

    constexpr variant() noexcept
    Constructs a variant holding a value-initialized value of type T0 

Finally, we call index() and print the result. index() returns the index of the type of the contained value. The contained value is an int, aka T0, so 0 is returned.
[variant.status](https://timsong-cpp.github.io/cppwp/n4659/variant.status#3)§23.7.3.5¶3:

    constexpr size_t index() const noexcept;
    _Effects+: If [it doesn't contain a value], returns variant_npos. Otherwise, returns the zero- based index of the alternative of the contained value.
</p>
</details>

#### 21. :skull::skull::skull:
```
#include <iostream>

int main() {
 int a = 5,b = 2;
 std::cout << a+++++b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
Some might expect the lexer to parse the series of characters a+++++b as follows:

a++ + ++b

but according to the [maximal munch principle](https://en.wikipedia.org/wiki/Maximal_munch), the lexer will take the longest sequence of characters to produce the next token(with a few exceptions).

[lex.pptoken](https://timsong-cpp.github.io/cppwp/n4659/lex.pptoken#3)§5.4¶3 in the C++ standard:
"the next preprocessing token is the longest sequence of characters that could constitute a preprocessing token, even if that would cause further lexical analysis to fail (...)."

So after parsing a++, it is not allowed to just parse +, it has to parse ++. The sequence is thus parsed as:

a ++ ++ + b

which is ill-formed since post-increment requires a modifiable lvalue but the first post-increment will produce a prvalue, as per [expr.post.incr](https://timsong-cpp.github.io/cppwp/n4659/expr.post.incr#1)§8.2.6¶1 in the C++ standard:
"The value of a postfix ++ expression is the value of its operand. (...) The result is a prvalue."
</p>
</details>

#### 22. :skull::skull:
```    
#include <iostream>

template <class T> void f(T &i) { std::cout << 1; }

template <> void f(const int &i) { std::cout << 2; }

int main() {
  int i = 42;
  f(i);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
For the call f(i), since the type of i is int, template argument deduction deduces T = int.

The explicit specialization template <> void f(const int &i) has T = const int, which is not the same type as int, so it doesn't match.

Instead, template <class T> void f(T &i) with T = int is used to create the implicit instantiation void f<int>(int&).
</p>
</details>

#### 23. :skull:
```
#include <iostream>

struct A {
  A() { std::cout << "A"; }
};
struct B {
  B() { std::cout << "B"; }
};

class C {
public:
  C() : a(), b() {}

private:
  B b;
  A a;
};

int main()
{
    C();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: BA
The initialization order of member variables is determined by their order of declaration, not their order in the initialization list.
</p>
</details>

#### 24. :skull::skull:
```
#include <iostream>
#include <string>

void f(const std::string &) { std::cout << 1; }

void f(const void *) { std::cout << 2; }

int main() {
  f("foo");
  const char *bar = "bar";
  f(bar);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 22
A string literal is not a std::string, but a const char[] . If the compiler was to choose f(const std::string&), it would have to go through a user defined conversion and create a temporary std::string. Instead, it prefers f(const void*), which requires no user defined conversion.
</p>
</details>

#### 25. :skull:
```
#include <iostream>

void f(int) { std::cout << 1; }
void f(unsigned) { std::cout << 2; }

int main() {
  f(-2.5);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
This overload is ambiguous. Why?

There are two viable functions for the call f(-2.5). For the compiler to select one, one of them needs to be better than the other, or the program is ill-formed. In our case, they are equally good, making the program ill-formed.

According to [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best)§16.3.3 in the standard, a viable one-argument function is better than another if the conversion sequence for the argument is better. So why isn't the int conversion sequence better than the unsigned conversion sequence, given that the double is signed?

All conversions are given a rank, and both "double => int" and "double => unsigned int" are of type "floating-integral conversion", which has rank "conversion". See Table 13 in the standard and [conv.fpint](https://timsong-cpp.github.io/cppwp/n4659/conv.fpint)§7.10. Since they have the same rank, no conversion is better than the other, and the program is ill-formed.
</p>
</details>

#### 26. :skull:
```
#include <iostream>

void f(float) { std::cout << 1; }
void f(double) { std::cout << 2; }

int main() {
  f(2.5);
  f(2.5f);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 21
The type of a floating point literal is double.
</p>
</details>

#### 27. :skull:
```
#include <iostream>

int main() {
  for (int i = 0; i < 3; i++)
    std::cout << i;
  for (int i = 0; i < 3; ++i)
    std::cout << i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 012012
Whether you post-increment or pre-increment i, its value does not change until after the loop body has executed.

</p>
</details>

#### 28. :skull:
```
#include <iostream>

class A {
public:
  void f() { std::cout << "A"; }
};

class B : public A {
public:
  void f() { std::cout << "B"; }
};

void g(A &a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: A
As long as A::f() is not virtual, A::f() will always be called, even if the reference or pointer is actually referring to an object of type B.
</p>
</details>

#### 29. :skull:
``` 
#include <iostream>

class A {
public:
  virtual void f() { std::cout << "A"; }
};

class B : public A {
public:
  void f() { std::cout << "B"; }
};

void g(A a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: A
g(A a) takes an object of type A by value, not by reference or pointer. This means that A's copy constructor is called on the object passed to g() (no matter if the object we passed was of type B), and we get a brand new object of type A inside g(). This is commonly referred to as slicing.
</p>
</details>

#### 30. :skull:
```
#include <iostream>

int f(int &a, int &b) {
  a = 3;
  b = 4;
  return a + b;
}

int main() {
  int a = 1;
  int b = 2;
  int c = f(a, a);
  std::cout << a << b << c;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 428
When f() is called with a as both parameters, both arguments refer to the same variable. This is known as aliasing. First, a is set to 3, then a is set to 4, then 4+4 is returned. b is never modified.
</p>
</details>

#### 31. :skull:
```
#include <iostream>

int main() {
  static int a;
  std::cout << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
Since a is a static local variable, it is automatically zero-initialized. This would not have happened if we removed the keyword static, making it a non-static local variable.

[basic.start.static](https://timsong-cpp.github.io/cppwp/n4659/basic.start.static#2)§6.6.2¶2 in the standard:

    If constant initialization is not performed, a variable with static storage duration (6.7.1) or thread storage duration (6.7.2) is zero-initialized (11.6)

a has static storage duration and is not constant initialized , so it gets zero-initialized.

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#6)§11.6¶6:

    To zero-initialize an object or reference of type T means:
    — if T is a scalar type (6.9), the object is initialized to the value obtained by converting the integer literal 0 (zero) to T;

So a gets initialized to 0.
</p>
</details>

#### 32. :skull:
```
#include <iostream>

class A {
public:
  A() { std::cout << "a"; }
  ~A() { std::cout << "A"; }
};

class B {
public:
  B() { std::cout << "b"; }
  ~B() { std::cout << "B"; }
};

class C {
public:
  C() { std::cout << "c"; }
  ~C() { std::cout << "C"; }
};

A a;
int main() {
  C c;
  B b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: acbBCA
[basic.start.dynamic](https://timsong-cpp.github.io/cppwp/n4659/basic.start.dynamic#5)§6.6.3¶5 in the standard:
"It is implementation-defined whether the dynamic initialization of a non-local inline variable with static storage duration is sequenced before the first statement of main or is deferred. If it is deferred, it strongly happens before any non-initialization odr-use of that variable."

Since A() is not constexpr, the initialization of a is dynamic. There are two possibilities:
- a is initialized before main() is called, i.e. before b or c are initialized.  
- a is not initialized before main(). It is however guaranteed to be initialized before the the use of any function defined in the same translation unit, i.e. before the constructors of b and c are called.  

Then, b and c are initialized in order.

Before main() exits, b and c are destructed in the reverse order of their construction. Then, when main() returns, a is destructed as per §6.6.4 in the standard:
"Destructors for initialized objects (...) with static storage duration are called as a result of returning from main."
</p>
</details>

#### 33. :skull::skull:
```
#include <iostream>

class A {
public:
  A() { std::cout << "a"; }
  ~A() { std::cout << "A"; }
};

class B {
public:
  B() { std::cout << "b"; }
  ~B() { std::cout << "B"; }
};

class C {
public:
  C() { std::cout << "c"; }
  ~C() { std::cout << "C"; }
};

A a;

void foo() { static C c; }
int main() {
  B b;
  foo();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: abcBCA
[basic.start.dynamic](https://timsong-cpp.github.io/cppwp/n4659/basic.start.dynamic#4)§6.6.3¶4 in the standard:
"It is implementation-defined whether the dynamic initialization of a non-local non-inline variable with static storage duration is sequenced before the first statement of main or is deferred. If it is deferred, it strongly happens before any non-initialization odr-use of any non-inline function or non-inline variable defined in the same translation unit as the variable to be initialized."

Since A() is not constexpr, the initialization of a is dynamic. There are two possibilities:
- a is initialized before main() is called, i.e. before b is initialized.  
- a is not initialized before main(). It is however guaranteed to be initialized before the the use of any function defined in the same translation unit, i.e. before the constructor of b is called.  

When execution reaches B b, it is initialized as normal. Static local variables are initialized the first time control passes through their declaration, so c is initialized next. As main() is exited, its local variable b goes out of scope, and is destroyed. Finally, all static variables are destroyed in reverse order of their initialization, first c, then a.
</p>
</details>

#### 34. :skull::skull::skull:
```
#include <iostream>
#include <exception>

int x = 0;

class A {
public:
  A() {
    std::cout << 'a';
    if (x++ == 0) {
      throw std::exception();
    }
  }
  ~A() { std::cout << 'A'; }
};

class B {
public:
  B() { std::cout << 'b'; }
  ~B() { std::cout << 'B'; }
  A a;
};

void foo() { static B b; }

int main() {
  try {
    foo();
  }
  catch (std::exception &) {
    std::cout << 'c';
    foo();
  }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: acabBA
Static local variables are initialized the first time control passes through their declaration. The first time foo() is called, b is attempted initialized. Its constructor is called, which first constructs all member variables. This means A::A() is called, printing a. A::A() then throws an exception, the constructor is aborted, and neither b or B::a are actually considered constructed. In the catch-block, c is printed, and then foo() is called again. Since b was never initialized the first time, it tries again, this time succeeding, printing ab. When main() exits, the static variable b is destroyed, first calling the destructor printing B, and then destroying member variables, printing A. 
</p>
</details>

#### 35. :skull:
```   
#include <iostream>

class A {
public:
  virtual void f() { std::cout << "A"; }
};

class B : public A {
private:
  void f() { std::cout << "B"; }
};

void g(A &a) { a.f(); }

int main() {
  B b;
  g(b);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B
The "trick" here is that B::f() is called even though it is private.

As [class.access.virt](https://timsong-cpp.github.io/cppwp/n4659/class.access.virt#2)§14.5¶2 in the standard puts it: "Access is checked at the call point using the type of the expression used to denote the object for which the member function is called". The call point here being a.f(), and the type of the expression is A&.
</p>
</details>

#### 36. :skull:
```
#include <iostream>
#include <limits>

int main() {
  unsigned int i = std::numeric_limits<unsigned int>::max();
  std::cout << ++i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
Unsigned integers have well defined behaviour when they overflow. When you go one above the largest representable unsigned int, you end up back at zero.

According to [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#4)§6.9.1¶4 in the C++ standard: "Unsigned integers, declared unsigned, shall obey the laws of arithmetic modulo 2^n where n is the number of bits in the value representation of that particular size of integer."
</p>
</details>

#### 37. :skull::skull:
```
#include <iostream>
#include <limits>

int main() {
  int i = std::numeric_limits<int>::max();
  std::cout << ++i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
Signed integer overflow is undefined behaviour according to the standard [expr](https://timsong-cpp.github.io/cppwp/n4659/expr#4)§8¶4: "If during the evaluation of an expression, the result is not mathematically defined or not in the range of representable values for its type, the behavior is undefined."

Most implementations will just wrap around, so if you try it out on your machine, you will probably see the same as if you had done
std::cout << std::numeric_limits<int>::min();

Relying on such undefined behaviour is however not safe. For an interesting example, see http://stackoverflow.com/questions/7682477/why-does-integer-overflow-on-x86-with-gcc-cause-an-infinite-loop
</p>
</details>

#### 38. :skull:
```
#include <iostream>

int main() {
  int i = 42;
  int j = 1;
  std::cout << i / --j;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
Integer division by zero is undefined behaviour. According to [expr.mul](https://timsong-cpp.github.io/cppwp/n4659/expr.mul#4)§8.6¶4 in the standard: "If the second operand of / or % is zero the behavior is undefined."
</p>
</details>

#### 39. :skull:
```
#include <iostream>

struct A {
  virtual std::ostream &put(std::ostream &o) const {
    return o << 'A';
  }
};

struct B : A {
  virtual std::ostream &put(std::ostream &o) const {
    return o << 'B';
  }
};

std::ostream &operator<<(std::ostream &o, const A &a) {
  return a.put(o);
}

int main() {
  B b;
  std::cout << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B
This is a way to get polymorphic behaviour for operator <<.
</p>
</details>

#### 40. :skull:
```
#include <iostream>

struct A {
  A() { std::cout << "A"; }
  A(const A &a) { std::cout << "B"; }
  virtual void f() { std::cout << "C"; }
};

int main() {
  A a[2];
  for (auto x : a) {
    x.f();
  }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: AABCBC
When the array is initialized, the default constructor is called once for each of the two objects in it.

Then we iterate over the array using auto, which in our case is deduced to be A. This means the copy constructor will be called before f() for each iteration, printing BCBC. (Just as if we had written for (A x: a).

If we want to avoid the copy constructor, we can write for (auto& x : a) instead. Then the loop would print CC. (Just as if we had written for (A& x: a).
</p>
</details>

#### 41. :skull:
```
#include <iostream>
struct X {
  X() { std::cout << "X"; }
};

int main() { X x(); }
```
<details><summary><b>Answer</b></summary>
<p>

#### This program has no output.
X x(); is a function prototype, not a variable definition. Remove the parentheses (or since C++11, replace them with {}), and the program will output X. 
</p>
</details>

#### 42. :skull::skull:
```
#include <iostream>

struct X {
  X() { std::cout << "a"; }
  X(const X &x) { std::cout << "b"; }
  const X &operator=(const X &x) {
    std::cout << "c";
    return *this;
  }
};

int main() {
  X x;
  X y(x);
  X z = y;
  z = x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: abbc
The first line in main(), X x; is straightforward, it calls the default constructor.

The next two lines is the heart of the question: The difference between X y(x) and X z = y is not that the first calls the copy constructor, and the second calls the copy assignment operator. The difference is that the first is direct initialization ([dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)§11.6¶16 in the standard) and the second is copy initialization ([dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#15)§11.6¶15).

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#17)§11.6¶17 says: "If the initialization is direct-initialization, or if it is copy-initialization where the (...) source type is the same class as (...) the class of the destination, constructors are considered." So both our cases use the copy constructor.

Not until z = x; do we have an actual assignment that uses the assignment operator.

See http://stackoverflow.com/questions/1051379/is-there-a-difference-in-c-between-copy-initialization-and-direct-initializati/1051468#1051468 for a more detailed discussion of direct vs. copy initialization.
</p>
</details>

#### 43. :skull:
```
#include <iostream>
#include <vector>

int main() {
  std::vector<int> v1(1, 2);
  std::vector<int> v2{ 1, 2 };
  std::cout << v1.size() << v2.size();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 12
To answer this we need to look at overload resolution of vector's constructors:

[vector.cons](https://timsong-cpp.github.io/cppwp/n4659/vector.cons#6)§26.3.11.2¶6 says (somewhat redacted):
vector(size_type n, const T& value);
Effects: Constructs a vector with n copies of value, using the specified allocator

So v1 contains one "2".

[over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list)§16.3.1.7 says (in summary) that when non-aggregate classes (such as vector) are list-initialized† and have an initializer list constructor (again, like vector), that constructor is chosen, and the argument list consists of the initializer list as a single argument.
(†: [dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#1)§11.6.4¶1: List-initialization is initialization of an object or reference from a braced-init-list.)

So v2 is initialized from the elements (aka initializer-clauses) in the braced-init-list, and contains the elements "1" and "2".
</p>
</details>

#### 44. :skull:
```
#include <iostream>

int main() {
  int a = 0;
  decltype(a) b = a;
  b++;
  std::cout << a << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 01
According to [dcl.type.simple](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.simple#4)§10.1.7.2¶4 in the C++ standard:
"The type denoted by decltype(e) is deﬁned as follows:
— if e is an unparenthesized id-expression [...], decltype(e) is the type of the entity named by e."

The type of a is int, so the type of b is also int.
</p>
</details>

#### 45. :skull::skull:
```
#include <iostream>
int main() {
  std::cout << 1["ABC"];
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B
[expr.sub](https://timsong-cpp.github.io/cppwp/n4659/expr.sub#1)§8.2.1¶1 in the standard says "The expression E1[E2] is identical (by definition) to *((E1)+(E2))".

In our case 1["ABC"] is identical to *(1+"ABC"). Since the plus operator is commutative, this is identical to *("ABC"+1), which is identical to the more familiar "ABC"[1].
</p>
</details>

#### 46. :skull::skull:
```
#include <initializer_list>
#include <iostream>

struct A {
  A() { std::cout << "1"; }

  A(int) { std::cout << "2"; }

  A(std::initializer_list<int>) { std::cout << "3"; }
};

int main(int argc, char *argv[]) {
  A a1;
  A a2{};
  A a3{ 1 };
  A a4{ 1, 2 };
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1133
a1 is default initialized, as described in [dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#12)§11.6¶12.

a2 doesn't actually use the initializer_list constructor with a list of zero elements, but the default constructor:
[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#3)§11.6¶3:
List-initialization of an object or reference of type T is defined as follows:  
- (...)  
- Otherwise, if the initializer list has no elements and T is a class type with a default constructor, the object is value-initialized.  
- Otherwise, if T is a specialization of std::initializer_list, the object is constructed as described below.  

a3's and a4's constructor is chosen in overload resolution, as described in [over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list)§16.3.1.7:

"When objects of non-aggregate class type T are list-initialized (...), overload resolution selects the constructor in two phases:
— Initially, the candidate functions are the initializer-list constructors (§11.6.4) of the class T and the argument list consists of the initializer list as a single argument.
— If no viable initializer-list constructor is found, overload resolution is performed again, where the candidate functions are all the constructors of the class T and the argument list consists of the elements of the initializer list."

Initializer list constructors are greedy, so even though A(int) constructor is available, the standard mandates that initializer_list<int> is prioritized, and if - and only if - it's not available, the compiler is allowed to look for other constructors. (This is why it is not recommended to provide a constructor that ambiguously overloads with an initializer_list constructor. See the answer to #4 in http://herbsutter.com/2013/05/09/gotw-1-solution/ )
</p>
</details>

#### 47. :skull::skull:
```
#include <iostream>
#include <string>
#include <future>

int main() {
  std::string x = "x";

  std::async(std::launch::async, [&x]() {
    x = "y";
  });
  std::async(std::launch::async, [&x]() {
    x = "z";
  });

  std::cout << x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: z
The destructor of a future returned from async is required to block until the async task has finished (see elaboration below). Since we don't assign the futures that are returned from async() to anything, they are destroyed at the end of the full expression (at the end of the line in this case). [class.temporary](https://timsong-cpp.github.io/cppwp/n4659/class.temporary#4)§15.2¶4 in the standard: "Temporary objects are destroyed as the last step in evaluating the full-expression (§4.k) that (lexically) contains the point where they were created."

This means that the first async call is guaranteed to finish execution before async() is called the second time, so, while the assignments themselves may happen in different threads, they are synchronized.

Elaboration on synchronization:
According to [futures.async](https://timsong-cpp.github.io/cppwp/n4659/futures.async#5)§33.6.9¶5 of the standard:
Synchronization: Regardless of the policy argument,
[...]
If the implementation chooses the launch::async policy,
— the associated thread completion synchronizes with (§4.1) the return from the first function that successfully detects the ready status of the shared state or with the return from the last function that releases the shared state, whichever happens first.

In this case, the destructor of std::future<> returned by the async() call is "the last function that releases the shared state", therefore it synchronizes with (waits for) the thread completion.

Scott Meyers writes more about this http://scottmeyers.blogspot.com/2013/03/stdfutures-from-stdasync-arent-special.html

</p>
</details>

#### 48. :skull::skull:
```
#include <iostream>

class C {
public:
  C(int i) : i(i) { std::cout << i; }
  ~C() { std::cout << i + 5; }

private:
  int i;
};

int main() {
  const C &c = C(1);
  C(2);
  C(3);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 127386
[class.temporary](https://timsong-cpp.github.io/cppwp/n4659/class.temporary#4)§15.2¶4 in the standard: "Temporary objects are destroyed as the last step in evaluating the full-expression (...) that (lexically) contains the point where they were created." This means that normally the temporaries returned from C(1), C(2), and C(3) should be destroyed at the end of the line.

However: [class.temporary](https://timsong-cpp.github.io/cppwp/n4659/class.temporary#6)§15.2¶6 states: "(...)when a reference is bound to a temporary. The temporary to which the reference is bound (...) persists for the lifetime of the reference", so the lifetime of the temporary returned by C(1) is extended for the lifetime of c, to the end of main(). The temporaries returned by C(2) and C(3) are still destroyed at the end of their lines of creation, so they get destroyed before the one returned by C(1).
</p>
</details>

#### 49. :skull:
```
#include <iostream>

class A;

class B {
public:
  B() { std::cout << "B"; }
  friend B A::createB();
};

class A {
public:
  A() { std::cout << "A"; }

  B createB() { return B(); }
};

int main() {
  A a;
  B b = a.createB();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
There is a compilation error when attempting to declare A::createB() a friend of B. To declare A::createB() a friend of B, the compiler needs to know that that function exists. Since it has only seen the declaration of A so far, not the full definition, it cannot know this.
</p>
</details>

#### 50. :skull::skull:
```
#include <iostream>

using namespace std;

class A {
public:
  A() { cout << "a"; }
  ~A() { cout << "A"; }
};

int i = 1;

int main() {
label:
  A a;
  if (i--)
    goto label;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: aAaA
The Standard says this about jump statements:

[stmt.jump](https://timsong-cpp.github.io/cppwp/n4659/stmt.jump#2)§9.6¶2 Transfer [...] back past an initialized variable with automatic storage duration involves the destruction of objects with automatic storage duration that are in scope at the point transferred from but not at the point transferred to.
</p>
</details>

#### 51. :skull::skull::skull:
```
#include <iostream>

extern "C" int x;
extern "C" { int y; }

int main() {

	std::cout << x << y;

	return 0;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined
According to [dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#7)§10.5¶7 in the standard : A declaration directly contained in a linkage-specification is treated as if it contains the extern specifier (§10.1.1) for the purpose of determining the linkage of the declared name and whether it is a definition.
extern "C" int x; //is just a declaration
extern "C" { int y; } //is a definition

And according to [basic.def.odr](https://timsong-cpp.github.io/cppwp/n4659/basic.def.odr#4)§6.2¶4: "Every program shall contain exactly one definition of every non-inline function or variable that is odr-used in that program outside of a discarded statement; no diagnostic required."

The result: x is never defined but it is optional for the compiler to print an error. The behaviour of this program is undefined.
</p>
</details>

#### 52. :skull::skull:
```
#include <iostream>
#include <vector>

int f() { std::cout << "f"; return 0;}
int g() { std::cout << "g"; return 0;}

void h(std::vector<int> v) {}

int main() {
    h({f(), g()});
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: fg
The goal of this question is to demonstrate that the evaluation order of elements in an initializer list is specified (as opposed to the arguments to a function call).

[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list#4)§11.6.4¶4: Within the initializer-list of a braced-init-list, the initializer-clauses, including any that result from pack expansions (§17.5.3), are evaluated in the order in which they appear.

If h took two ints instead of a vector<int>, and was called like this:
h(f(), g());
the program would be unspecified, and could either print fg or gf.
</p>
</details>

#### 53. :skull::skull::skull:
```
#include <functional>
#include <iostream>

template <typename T>
void call_with(std::function<void(T)> f, T val)
{
	f(val);
}

int main()
{
	auto print = [] (int x) { std::cout << x; };
	call_with(print, 42);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
The compiler tries to deduce T for every parameter and checks if the deduced types match. Because a lambda is of completely different type, it cannot be matched against std::function<void(T)> and the deduction process fails.
This problem can be fixed by turning the first parameter into a so-called nondeduced context.

[temp.deduct.type](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct.type#5)§17.8.2.5¶5 in the standard:

    The non-deduced contexts are:


        The nested-name-specifier of a type that was specified using a qualified-id.

        (...)

    When a type name is specified in a way that includes a nondeduced context, all of the types that comprise that type name are also nondeduced. However, a compound type can include both deduced and nondeduced types. [Example: If a type is specified as A<T>::B<T2>, both T and T2 are nondeduced. Likewise, if a type is specified as A<I+J>::X<T>, I, J, and T are nondeduced. If a type is specified as void f(typename A<T>::B, A<T>), the T in A<T>::B is nondeduced but the T in A<T> is deduced. ]

In particular, a helper struct template that typedefs the template parameter can be used:

template <typename T>
struct type_identity
{
    typedef T type;
};

This helper struct can then turn std::function<void(T)> into a nondeduced context as shown in the standard:

template <typename T>
void call_with(typename type_identity<std::function<void(T)>>::type f, T val)
{
    f(val);
}

std::type_identity is in the C++20 standard, but not C++17.

The problem can also be solved in a less general way (at each call site) by explicitly specifying the template argument:

call_with<int>(print, 42);


</p>
</details>

#### 54. :skull:
```
#include <iostream>

int main() {
    int i=1;
    do {
        std::cout << i;
        i++;
        if(i < 3) continue;
    } while(false);
    return 0;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
[stmt.cont](https://timsong-cpp.github.io/cppwp/n4659/stmt.cont#1)§9.6.2¶1 in the standard: "The continue statement (...) causes control to pass to the loop-continuation portion of the smallest enclosing iteration-statement, that is, to the end of the loop." (Not to the beginning.)
</p>
</details>

#### 55. :skull::skull:
```
    
#include <iostream>
#include <utility>

struct A
{
	A() { std::cout << "1"; }
	A(const A&) { std::cout << "2"; }
	A(A&&) { std::cout << "3"; }
};

struct B
{
	A a;
	B() { std::cout << "4"; }
	B(const B& b) : a(b.a) { std::cout << "5"; }
	B(B&& b) : a(b.a) { std::cout << "6"; }
};

int main()
{
	B b1;
	B b2 = std::move(b1);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1426
First, b1 is default initialized. All members are initialized before the body of the constructor, so b1.a is default initialized first, and we get the output 14.

[class.base.init](https://timsong-cpp.github.io/cppwp/n4659/class.base.init#9)§15.6.2¶9 in the standard: "In a non-delegating constructor, if a given potentially constructed subobject designated by a
mem-initializer-id (...) then if the entity is a non-static data member that has a default member initializer (§12.2), (...) the entity is initialized as specified in §11.6 (...) otherwise, the entity is default-initialized."

Then, b2 is initialized with the move construcor (since std::move(b1)converts the reference to b1 to an xvalue, allowing it to be moved from.) In B's move constructor, a is initialized in the member initializer list. Even though b is an rvalue reference (and bound to an rvalue), b itself is an lvalue, and cannot be moved from. b2.a is then copy initialized, printing 2, and finally the body of B's move constructor prints 6.

(If the concept of rvalue references being lvalues is confusing, read http://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers . Search for "In widget".)
</p>
</details>

#### 56. :skull:
```
#include <iostream>
#include <memory>
#include <vector>

class C {
public:
  void foo()       { std::cout << "A"; }
  void foo() const { std::cout << "B"; }
};

struct S {
  std::vector<C> v;
  std::unique_ptr<C> u;
  C *const p;

  S() 
    : v(1) 
    , u(new C())
    , p(u.get())
  {}
};

int main() {
  S s;
  const S &r = s;
 
  s.v[0].foo();
  s.u->foo();
  s.p->foo();

  r.v[0].foo();
  r.u->foo();
  r.p->foo();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: AAABAA
According to [dcl.ptr](https://timsong-cpp.github.io/cppwp/n4659/dcl.ptr#1)§11.3.1¶1 in the C++ Standard, "The cv-qualifiers [e.g., const] apply to the pointer and not to the object pointed to."

That is, const-ness is shallow with regards to raw pointers and references (and standard types that seek to emulate them, like std::unique_ptr) but not with regard to standard containers such as std::vector.

In the code above, the object s is non-const, and so its members all retain their default const-ness and all calls through them invoke the non-const version of C::foo().

However, r refers to its object as a const instance of S. That const-ness changes the behavior of its member v, an std::vector which is "const-correct" in the sense that its operator[] returns const C& (see [sequence.reqmts](https://timsong-cpp.github.io/cppwp/n4659/sequence.reqmts#14)§26.2.3¶14) and therefore invokes the const version of C::foo().

The const-ness of r's referent is also propagated to its members u and p (meaning one could not perform a mutating operation on u, e.g., calling r.u.reset()), but this has no effect on the instance of C that they both point to. That is, the pointers themselves become const, but the pointed-to objects remain non-const. Hence, they both still call the non-const version of C::foo().

The const-ness of the member S::p is the same for both s and r. Because it is declared as a const pointer, it does not change const-ness to follow the const-ness of its instance of S but remains a const pointer to a non-const object.
</p>
</details>

#### 57. :skull:
```
#include <iostream>

void f(int) { std::cout << "i"; }
void f(double) { std::cout << "d"; }
void f(float) { std::cout << "f"; }

int main() {
  f(1.0);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: d
According to [lex.fcon](https://timsong-cpp.github.io/cppwp/n4659/lex.fcon#1)§5.13.4¶1 in the standard: "The type of a floating literal is double unless explicitly specified by a suffix."
The best overload is therefore void f(double).
</p>
</details>

#### 58. :skull:
```
#include <iostream>

void print(char const *str) { std::cout << str; }
void print(short num) { std::cout << num; }

int main() {
  print("abc");
  print(0);
  print('A');
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
Sneaky ambiguous function call.

The statement print(0); is ambiguous due to overload resolution rules. Both print functions are viable, but for the compiler to pick one, one of them has to have a better conversion sequence than the other. [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#2)§16.3.3¶2: "If there is exactly one viable function that is a better function than all other viable functions, then it is the one selected by overload resolution; otherwise the call is ill-formed".

(a) Because 0 is a null pointer constant[1], it can be converted implicitly into any pointer type with a single conversion.

(b) Because 0 is of type int, it can be converted implicitly to a short with a single conversion too.

In our case, both are standard conversion sequences with a single conversion of "conversion rank". Since no function is better than the other, the call is ill-formed.

[1] [conv.ptr](https://timsong-cpp.github.io/cppwp/n4659/conv.ptr#1)§7.11¶1 A null pointer constant is an integer literal (§5.13.2) with value zero or a prvalue of type std::nullptr_t. A null pointer constant can be converted to a pointer type.
</p>
</details>

#### 59. :skull::skull:
```
#include <iostream>

int main() {
  void * p = &p;
  std::cout << bool(p);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
As defined in [basic.scope.pdecl](https://timsong-cpp.github.io/cppwp/n4659/basic.scope.pdecl#1)§6.3.2¶1, the point of name declaration is after its
complete declarator and before its initialisation. This
means that line 4 is valid C++, because it's possible
to initialise the variable p with the address of an existing
variable, even if it is its own address.

The value of p is unknown, but can not be a null pointer value. The
cast must thus evaluate to 1 and initialise the temporary
bool as true.
</p>
</details>

#### 60. :skull:
```
int main() {
  int a = 10;
  int b = 20;
  int x;
  x = a, b;
  std::cout << x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 10
The comma operator has the lowest precedence of all C++ operators (specifically lower than =).
In this example it separates the two expressions x = a and b.

First x = a is evaluated, setting x to 10.
Then, b is evaluated, which does nothing.
</p>
</details>

#### 61. :skull::skull::skull:
```
#include <iostream> 

typedef long long ll;

void foo(unsigned ll) {
    std::cout << "1";
}

void foo(unsigned long long) {
    std::cout << "2";
}

int main() {
    foo(2ull);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 2
[dcl.spec](https://timsong-cpp.github.io/cppwp/n4659/dcl.spec#3)§10.1¶3 in the C++ standard states, "If a type-name is encountered while parsing a decl-speciﬁer-seq, it is interpreted as part of the decl-speciﬁer-seq if and only if there is no previous defining-type-speciﬁer other than a cv-qualiﬁer in the decl-speciﬁer-seq."

[dcl.spec](https://timsong-cpp.github.io/cppwp/n4659/dcl.spec#4)§10.1¶4 also has a note: "Since signed, unsigned, long, and short by default imply int, a type-name appearing after one of those speciﬁers is treated as the name being (re)declared."

In void foo(unsigned ll), since unsigned implies int, ll is being redeclared as a parameter name.
</p>
</details>

#### 62. :skull::skull::skull:
```
#include <iostream>

using namespace std;

struct A {};
struct B {};

template<typename T = A>
struct X;

template<>
struct X<A> {
   static void f() { cout << 1 << endl; }
};

template<>
struct X<B> {
   static void f() { cout << 2 << endl; }
};

template< template<typename T = B> class C>
void g() {
   C<>::f();
}

int main() {
   g<X>();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 2
[temp.param](https://timsong-cpp.github.io/cppwp/n4659/temp.param#14)§17.1¶14 in the C++ standard says: "A template-parameter of a template template-parameter is permitted to have a default template-argument.
When such default arguments are specified, they apply to the template template-parameter in the scope of
the template template-parameter."

In this case, the template template-parameter is C, and the scope of C is the function g(), so the default arguments of C (i.e. T = B) are applied and C::f() is called inside g().
</p>
</details>

#### 63. :skull::skull:
```
#include <iostream>

using namespace std;

template <class T> void f(T) {
  static int i = 0;
  cout << ++i;
}

int main() {
  f(1);
  f(1.0);
  f(1);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 112
[temp.fct.spec](https://timsong-cpp.github.io/cppwp/n4659/temp.fct.spec#2)§17.8¶2: Each function template specialization instantiated from a template has its own copy of any static variable.

This means we get two instantiations of f, one for T=int, and one for T=double. Thus, i is shared between the two int calls, but not with the double call.
</p>
</details>

#### 64. :skull::skull:
```
#include<iostream>

int foo()
{
  return 10;
}

struct foobar
{
  static int x;
  static int foo()
  {
    return 11;
  }
};

int foobar::x = foo();

int main()
{
    std::cout << foobar::x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 11
[basic.lookup.unqual](https://timsong-cpp.github.io/cppwp/n4659/basic.lookup.unqual#13)§6.4.1¶13 states "A name used in the definition of a static data member of class X (...) is looked up as if the name was used in a member function of X."

Even though the call foo() occurs outside the class, since foo is used in the definition of the static data member foobar::x, it is looked up as if foo() was called in a member function of foobar. If foo() was called in a member function of foobar, foobar::foo() would be called, not the global foo().
</p>
</details>

#### 65. :skull::skull::skull:
```
#include <iostream>
#include <type_traits>

using namespace std;

int main()
{
  int i, &j = i;
  [=]
  {
    cout << is_same<decltype    ((j)),     int         >::value
         << is_same<decltype   (((j))),    int      &  >::value
         << is_same<decltype  ((((j)))),   int const&  >::value
         << is_same<decltype (((((j))))),  int      && >::value
         << is_same<decltype((((((j)))))), int const&& >::value;
  }();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 00100
[expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#14)§8.1.5.2¶14 says
Every occurrence of decltype((x)) where x is a possibly parenthesized id-expression that names an entity of automatic storage duration is treated as if x were transformed into an access to a corresponding data member of the closure type that would have been declared if x were an odr-use of the denoted entity.

So additional parentheses, as the in the code snippet above, are ignored.

The member of the closure type corresponding to the as-if-captured j will be not a reference, but will have the referenced type of the reference, since it is captured by copy ([expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#10)§8.1.5.2¶10).

Since the lambda is not declared mutable, the overloaded operator() of the closure type will be a const member function. [expr.prim.lambda.closure](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.closure#4)§8.1.5.1¶4: "The function call operator or operator template is declared const if and only if the lambda-expression's parameter-declaration-clause is not followed by mutable."

Since the expression for decltype is a parenthesized lvalue expression, [dcl.type.simple](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.simple#4)§10.1.7.2¶4 has this to say: "The type denoted by decltype(e) is (...) T&, where T is the type of e;" As the expression occurs inside a const member function, the expression is const, and decltype((j)) denotes int const&. See also the example in [expr.prim.lambda.capture](https://timsong-cpp.github.io/cppwp/n4659/expr.prim.lambda.capture#14)§8.1.5.2¶14.
</p>
</details>

#### 66. :skull::skull:
```
#include <vector>
#include <iostream>

using namespace std;

int main() {
  vector<char> delimiters = { ",", ";" };  
  cout << delimiters[0];
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
Here we are trying to initialize a vector<char> using two string literals, not two chars.

The initializer-list constructor for template <class T>vector is defined as vector(initializer_list<T>) by [vector.overview](https://timsong-cpp.github.io/cppwp/n4659/vector.overview)§26.3.11.1 in the standard. In our case, vector(initializer_list<char>).

The type of a string literal is "array of n const char" ([lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#8)§5.13.5¶8), so clearly the initializer-list constructor is not a match.

This problem does however not result in a compiler error, since the compiler is able to find another constructor that matches!

[over.match.list](https://timsong-cpp.github.io/cppwp/n4659/over.match.list#1)§16.3.1.7¶1 explains the rules very clearly:
"When objects of non-aggregate class type T are list-initialized (...), overload resolution selects the constructor in two phases:  
— Initially, the candidate functions are the initializer-list constructors of the class T and the argument list consists of the initializer list as a single argument [which we have seen didn't match].  
— If no viable initializer-list constructor is found, overload resolution is performed again, where the candidate functions are all the constructors of the class T and the argument list consists of the elements of the initializer list [in our case, the two string literals "," and ";" ]".

Going back to [vector.overview](https://timsong-cpp.github.io/cppwp/n4659/vector.overview)§26.3.11.1, we find this candidate:

template <class InputIterator> vector(InputIterator first, InputIterator last)

Note that the type of InputIterator has no link to the type of T in the vector<T>. So even if we are initializing a vector<char>, the two arguments can be of arbitrary type. The only requirement is that they confirm to the concept of InputIterator, which const char[] happens to do.

Now the constructor believes it has been passed two iterators to the same sequence, but it has actually been passed iterators to two completely different sequences, "," and ";". [forward.iterators](https://timsong-cpp.github.io/cppwp/n4659/forward.iterators#2)§27.2.5¶2 says: "The domain of == for forward iterators is that of iterators over the same underlying sequence.". So the result of this program is undefined.
</p>
</details>

#### 67. :skull::skull::skull:
```
#include <iostream>
using namespace std;

template<typename T>
void adl(T)
{
  cout << "T";
}

struct S
{
};

template<typename T>
void call_adl(T t)
{
  adl(S());
  adl(t);
}

void adl(S)
{
  cout << "S";
}

int main ()
{
  call_adl(S());
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: TS
[temp.res](https://timsong-cpp.github.io/cppwp/n4659/temp.res#9)§17.6¶9 states: "When looking for the declaration of a name used in a template definition, the usual lookup rules (§6.4.1, §6.4.2) are used for non-dependent names. The lookup of names dependent on the template parameters is postponed until the actual template argument is known (§17.6.2)."

The first call to adl is a non-dependent call, so it is looked up at the time of definition of the function template. The resolution of the second call is deferred until the template is instantiated because it depends on a template parameter.

template<typename T> void call_adl_function(T t)
{
    adl(S()); // Independent, looks up adl now.
    adl(t); // Dependent, looks up adl later.
}

When adl is being looked up at the time of definition of the function template, the only version of adl that exists is the templated adl(T). Specifically, adl(S) does not exist yet, and is not a candidate.

Note: At the time of writing, this program does not confirm to the standard in some recent versions of Visual Studio's C++ compiler.
</p>
</details>

#### 68. :skull::skull:
```
#include <iostream>
using namespace std;

class A
{
public:
    A() { cout << "A"; }
    A(const A &) { cout << "a"; }
};

class B: public virtual A
{
public:
    B() { cout << "B"; }
    B(const B &) { cout<< "b"; }
};

class C: public virtual A
{
public:
    C() { cout<< "C"; }
    C(const C &) { cout << "c"; }
};

class D:B,C
{
public:
    D() { cout<< "D"; }
    D(const D &) { cout << "d"; }
};

int main()
{
    D d1;
    D d2(d1);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: ABCDABCd
On the first line of main(), d1 is initialized, in the order A, B, C, D. That order is defined by [class.base.init](https://timsong-cpp.github.io/cppwp/n4659/class.base.init#13)§15.6.2¶13:
"
— First, and only for the constructor of the most derived class (§4.5), virtual base classes are initialized in the order they appear on a depth-first left-to-right traversal of the directed acyclic graph of base classes, where “left-to-right” is the order of appearance of the base classes in the derived class base-specifier-list.  
— Then, direct base classes are initialized in declaration order as they appear in the base-specifier-list
(...)  
— Finally, the compound-statement of the constructor body is executed.  
"
So the output is ABCD.

On the second line, d2 is initialized. But why are the constructors (as opposed to the copy constructors) for the base classes, called? Why do we see ABCd instead of abcd?

As it turns out, an implicitly-defined copy constructor would have called the copy constructor of its bases ([class.copy.ctor](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14)§15.8.1¶14: "The implicitly-defined copy/move constructor for a non-union class X performs a memberwise copy/move of its bases and members."). But when you provide a user-defined copy constructor, this is something you have to do explicitly.
</p>
</details>

#### 69. :skull:
```   
#include <iostream>
#include <map>
using namespace std;

int main()
{
  map<bool,int> mb = {{1,2},{3,4},{5,0}};
  cout << mb.size(); 
  map<int,int> mi = {{1,2},{3,4},{5,0}};
  cout << mi.size();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 13
std::map stores values based on a unique key. The keys for mb are boolean, and 1, 3 and 5 all evaluate to the same key, true.

[map.overview](https://timsong-cpp.github.io/cppwp/n4659/map.overview#1)§26.4.4.1¶1 in the standard:
"A map is an associative container that supports unique keys (contains at most one of each key value)."

The type of mb is map<bool,int>. The key is bool, so the integers 1, 3 and 5 used for initialization are first converted to bool, and they all evaluate to true.

[conv.bool](https://timsong-cpp.github.io/cppwp/n4659/conv.bool#1)§7.14¶1 in the standard:
"A prvalue of arithmetic, unscoped enumeration, pointer, or pointer to member type can be converted to a prvalue of type bool. A zero value, null pointer value, or null member pointer value is converted to false; any other value is converted to true."
</p>
</details>

#### 70. :skull::skull:
```
#include <iostream>
using namespace std;

size_t get_size_1(int* arr)
{
  return sizeof arr;
}

size_t get_size_2(int arr[])
{
  return sizeof arr;
}

size_t get_size_3(int (&arr)[10])
{
  return sizeof arr;
}

int main()
{
  int array[10];
  //Assume sizeof(int*) != sizeof(int[10])
  cout << (sizeof(array) == get_size_1(array));
  cout << (sizeof(array) == get_size_2(array));
  cout << (sizeof(array) == get_size_3(array));
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 001
This question compares three ways for a function to take an array as parameter, while two of them are actually the same.

In main, the array is of array type, therefore the sizeof operator returns the size of the array in terms of bytes. ([expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#2)§8.3.3¶2 in the standard: "When applied to an array, the result [of the sizeof operator] is the total number of bytes in the array. This implies that the size of an array of n elements is n times the size of an element.")

In get_size_3, the parameter is a reference to an array of size 10, therefore the sizeof operator returns the size of the array in terms of bytes. ([expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#2)§8.3.3¶2 in the standard: When applied to a reference or a reference type, the result is the size of the referenced type. )

In get_size_1 and get_size_2, the parameter is a pointer, therefore the sizeof operator returns the size of the pointer. Although the parameter of get_size_2 is an array, it is adjusted into a pointer. ([dcl.fct](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct#5)§11.3.5¶5 in the standard: "any parameter of type “array of T” (...) is adjusted to be “pointer to T”")
</p>
</details>

#### 71. :skull::skull::skull:
```
#include <iostream>
#include <limits>

int main()
{
  int N[] = {0,0,0};

  if ( std::numeric_limits<long int>::digits==63 &&
    std::numeric_limits<int>::digits==31 &&
    std::numeric_limits<unsigned int>::digits==32 )
  {
    for (long int i = -0xffffffff; i ; --i)
    {
      N[i] = 1;
    }
  }
  else
  {  
    N[1]=1;
  }

  std::cout << N[0] <<N [1] << N[2];
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 010
As the else part of the branch is obvious, we concentrate on the if part and make the assumptions present in the condition.

[lex.icon](https://timsong-cpp.github.io/cppwp/n4659/lex.icon)§5.13.2 in the standard: "The type of an integer literal is the first of the corresponding list in Table 7 in which its value can be represented." [Table 7: int, unsigned int, long int, unsigned long int ... for hexadecimal literals --end Table]"

Since the literal 0xffffffff needs 32 digits, it can be represented as an unsigned int but not as a signed int, and is of type unsigned int. But what happens with the negative of an unsigned integer?

[expr.unary.op](https://timsong-cpp.github.io/cppwp/n4659/expr.unary.op#8)§8.3.1¶8 in the standard: "The negative of an unsigned quantity is computed by subtracting its value from 2^n, where n is the number of bits in the promoted operand." Here n is 32, and we get:

2^32 - 0xffffffff = 4294967296 - 4294967295 = 1

So i is initialised to 1, and N[1] is the only element accessed in the loop. (The second time around the loop, i is 0, which evaluates to false, and the loop terminates.)
</p>
</details>

#### 72. :skull::skull::skull:
```
#include<iostream>

int main(){
  int x=0; //What is wrong here??/
  x=1;
  std::cout<<x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
??/ is a trigraph which doesn't exist anymore in C++17, and as it is in a comment, it is ignored (as anything else).

So the output is 1.
</p>
</details>

#### 73. :skull::skull::skull:
```
#include <iostream>

volatile int a;

int main() {
  std::cout << (a + a);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
The issue here is not the missing initializer of the variable a - it will implicitly be initialized to 0 here. But the issue is the access to a twice without sequencing between the accesses. According to [intro.execution](https://timsong-cpp.github.io/cppwp/n4659/intro.execution#14)§4.6¶14, accesses of volatile glvalues are side-effects and according to [intro.execution](https://timsong-cpp.github.io/cppwp/n4659/intro.execution#17)§4.6¶17 these two unsequenced side-effects on the same memory location results in undefined behavior.
</p>
</details>

#### 74. :skull::skull:
```   
#include <iostream>
#include <type_traits>

int main()
{
    std::cout << std::is_signed<char>::value;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is unspecified / implementation defined 
[basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#1)§6.9.1¶1

    It is implementation-defined whether a char object can hold negative values.
</p>
</details>

#### 75. :skull::skull:
```    
#include <iostream>
#include <type_traits>
 
int main() {
    if(std::is_signed<char>::value){
        std::cout << std::is_same<char, signed char>::value;
    }else{
        std::cout << std::is_same<char, unsigned char>::value;
    }
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
[basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#1)§6.9.1¶1
Plain char, signed char, and unsigned char are three distinct types (...).
</p>
</details>

#### 76. :skull::skull:
```   
#include <iostream>
#include <typeinfo>

struct A {};

int main() 
{
    std::cout<< (&typeid(A) == &typeid(A));
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is unspecified / implementation defined 
[expr.typeid](https://timsong-cpp.github.io/cppwp/n4659/expr.typeid#1)§8.2.8¶1: "The result of a typeid expression is an lvalue of static type const std::type_info",
and
[expr.unary.op](https://timsong-cpp.github.io/cppwp/n4659/expr.unary.op#3)§8.3.1¶3: "The result of the unary & operator is a pointer to its operand",
so we're comparing two pointers to const std::type_info.

There is no guarantee that the same std::type_info instance will be referred to by all evaluations of the typeid expression on the same type, although std::type_info::hash_code of those type_info objects would be identical, as would be their std::type_index.

(For more info on hash_code(), see [type.info](https://timsong-cpp.github.io/cppwp/n4659/type.info#7)§21.7.2¶7 : "hash_code() (...)shall return the same value for any two type_info objects which compare equal")

(For more info on type_index equality, see [type.index.members](https://timsong-cpp.github.io/cppwp/n4659/type.index.members)§23.18.3 and [type.info](https://timsong-cpp.github.io/cppwp/n4659/type.info)§21.7.2)
</p>
</details>

#### 77. :skull:
```
#include <iostream>
#include <vector>

struct Foo
{
    Foo() { std::cout<<"a"; }
    Foo(const Foo&) { std::cout<<"b"; }
};

int main()
{
    std::vector<Foo> bar(5);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: aaaaa
Since C++11, std::vector has a one parameter constructor ( + allocator). [vector.cons](https://timsong-cpp.github.io/cppwp/n4659/vector.cons#3)§26.3.11.2¶3 in the standard):

explicit vector(size_type n, const Allocator& = Allocator())

which constructs a vector with n value-initialized elements. Each value-initialization calls the default Foo constructor, resulting in the output aaaaa .

The "trick" is, that before C++11, std::vector had a 2 parameter constructor ( + allocator ), which constructed the container with n copies of the second parameter, which is defaulted to T().

So this code before C++11 would output abbbbb, because the call would be equivalent to std::vector<Foo> bar(5,T()).
</p>
</details>

#### 78. :skull::skull:
```
#include <iostream>

int i;

void f(int x) {
    std::cout << x << i;
}

int main() {
    i = 3;
    f(i++);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 34
According to [intro.execution](https://timsong-cpp.github.io/cppwp/n4659/intro.execution#18)§4.6¶18 in the standard, when calling a function, every value computation and side effect associated with any argument expression, is sequenced before the function is entered. Hence, in the expression f(i++), f is called with a parameter of the original value of i, but i is incremented before entering the body of f.
</p>
</details>

#### 79. :skull::skull:
```
#include <iostream>

struct A {
    virtual void foo (int a = 1) {
        std::cout << "A" << a;
    }
};

struct B : A {
    virtual void foo (int a = 2) {
        std::cout << "B" << a;
    }
};

int main () {
    A *b = new B;
    b->foo();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: B1
n the first line of main, we create a new B object, with an A pointer a pointing to it.

On the next line, we call b->foo(), where b has the static type A, and the dynamic type B. Since foo() is virtual, the dynamic type of b is used to ensure B::foo() gets called rather than A::foo().

However, which default argument is used for the int a parameter to foo()? [dcl.fct.default](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#10)§11.3.6¶10 in the standard:

    A virtual function call (§13.3) uses the default arguments in the declaration of the virtual function determined by the static type of the pointer or reference denoting the object. An overriding function in a derived class does not acquire default arguments from the function it overrides.

So B::foo() is called, but with the default argument from A::foo(), and the output is B1.
</p>
</details>

#### 80. :skull::skull::skull:
```
#include <iostream>

int main() {
    int n = 3;
    int i = 0;

    switch (n % 2) {
    case 0: 
    do {
    ++i;
    case 1: ++i;
    } while (--n > 0);
    }

    std::cout << i;
}
```
<details><summary><b>Answer</b></summary>
<p>

### The program is guaranteed to output: 5
Though this piece of code might look strange at first sight, it is still valid C++ code and is known as [Duff's device](https://en.wikipedia.org/wiki/Duff%27s_device).

According to the standard [stmt.select](https://timsong-cpp.github.io/cppwp/n4659/stmt.select#1)§9.4¶1, a switch statement can be followed (likewise, e.g., an if statement) by any valid statement including a compound (block) statement. The only difference from a general compound statement is that case and default labels can appear inside it. Jumping into the nested do-while block is legal ([stmt.dcl](https://timsong-cpp.github.io/cppwp/n4659/stmt.dcl#3)§9.7¶3), and its execution is not altered by the case labels ([stmt.switch](https://timsong-cpp.github.io/cppwp/n4659/stmt.switch#6)§9.4.2¶6).
</p>
</details>

#### 81. :skull::skull:
```
#include <iostream>

void f()
{
    std::cout << "1";
}

template<typename T>
struct B
{
    void f()
    {
        std::cout << "2";
    }
};

template<typename T>
struct D : B<T>
{
    void g()
    {
        f();
    }
};

int main()
{
    D<int> d;
    d.g();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
According to the standard, [temp.dep](https://timsong-cpp.github.io/cppwp/n4659/temp.dep#3)§17.6.2¶3, "in the definition of a class or class template, if a base class depends on a template-parameter, the base class scope is not examined during unqualified name lookup either at the point of definition of the class template or member or during an instantiation of the class template or member". So when the compiler sees a call to a function f() inside g() it should choose the one from the global scope, and the program should output "1". See discussions in [the C++ FAQ](https://isocpp.org/wiki/faq/templates#nondependent-name-lookup-members) and on [Stack Overflow](https://stackoverflow.com/questions/4643074/why-do-i-have-to-access-template-base-class-members-through-the-this-pointer).

In the real life, not all compilers obey this section of the standard, and some of them produce (erroneously) the code that outputs "2". For example, GCC 4.9.3 - "1", Clang 3.8.1 - "1", Microsoft C++ 19.00.24210 - "2", Intel C++ 15.0.7.287 (On Windows) - "2". Even if the implementation you use calls B::f() for the unqualified f(), you should not rely on this behaviour.
</p>
</details>

#### 82. :skull:
```
#include <iostream>

class A {
  int foo = 0;

public:
  int& getFoo() { return foo; }
  void printFoo() { std::cout << foo; }
};

int main() {
  A a;

  auto bar = a.getFoo();
  ++bar;

  a.printFoo();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
To get to the bottom of this, we need to take a detour via function templates.

According to [dcl.spec.auto](https://timsong-cpp.github.io/cppwp/n4659/dcl.spec.auto#6)§10.1.7.4¶6 (auto specifier) in the standard:
"The type of each declared variable is determined by placeholder type deduction (§10.1.7.4.1)."

Placeholder type deduction works as per [dcl.type.auto.deduct](https://timsong-cpp.github.io/cppwp/n4659/dcl.type.auto.deduct#4)§10.1.7.4.1¶4 :

    If the placeholder is the auto type-specifier, the deduced type T' replacing T is determined using the rules for template argument deduction. (...) Deduce a value for U [the template parameter for the invented function] using the rules of template argument deduction from a function call (§17.8.2.1) (...).

So the rules for auto bar = a.getFoo(); are the same as if we did

template<typename T> void f(T t);
f(a.getFoo());

Let's have a look at [temp.deduct.call](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct.call#4)§17.8.2.1¶4:
"In general, the deduction process attempts to find template argument values that will make the deduced A identical to A" (where A is the type of the argument of the call.)

Now what is the type of the argument of the call? getFoo() returns int&, but let's see what [expr](https://timsong-cpp.github.io/cppwp/n4659/expr#5)§5¶5 has to say about the type of expressions:
"If an expression initially has the type “reference to T”, the type is adjusted to T prior to any further analysis."

So the type of the expression a.getFoo() that is used for deduction is int, not int&. Now int is being used for deducing T in our imaginary void f(T t), so T is int. Since auto uses the same rule, the type of bar is also int.

Since bar is not a reference, incrementing it does not increment foo, and the output of the program is 0.
</p>
</details>

#### 83. :skull:
```
#include<iostream>

void f(int& a, const int& b) {
  std::cout << b;
  a = 1;
  std::cout << b;
}

int main(){
  int x = 0;
  f(x,x);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 01
Both a and b refer to the same variable x.

The fact that b is a const reference does not guarantee that b isn't modified from elsewhere, only that we can't modify it.
</p>
</details>

#### 84. :skull::skull:
```
#include <limits>
#include <iostream>

int main() {
    std::cout << std::numeric_limits<unsigned char>::digits;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is unspecified / implementation defined 
Although this program probably outputs 8 on your computer, it's not guaranteed to do so by the standard. All we can be sure of is what's specified in [basic.fundamental](https://timsong-cpp.github.io/cppwp/n4659/basic.fundamental#1)§6.9.1¶1:

"Objects declared as characters (char) shall be large enough to store any member of the implementation’s basic character set. (...) A char, a signed char, and an unsigned char occupy the same amount of storage."

So unsigned char is the same size as char, which is large enough to store any member of the implementation’s basic character set - but not necessarily 8.
</p>
</details>

#### 85. :skull::skull:
```
#include <iostream>

struct Base {
  void f(int) { std::cout << "i"; }
};

struct Derived : Base {
  void f(double) { std::cout << "d"; }
};

int main() {
  Derived d;
  int i = 0;
  d.f(i);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: d
Why does overload resolution not pick the obviously better void f(int), but instead void f(double) which requires a floating-integral conversion from int to double?

It turns out that void f(int) isn't even in scope, and is not considered for overload resolution at all. When the name f is introduced in Derived, it hides the name f that was introduced in Base.

[basic.scope](https://timsong-cpp.github.io/cppwp/n4659/basic.scope)§6.3 in the C++ standard is dedicated to scope. [basic.scope.hiding](https://timsong-cpp.github.io/cppwp/n4659/basic.scope.hiding#1)§6.3.10¶1 and ¶3 has some hints about what's going on in our case:

¶1: "A name can be hidden by an explicit declaration of that same name in a nested declarative region or derived class (§13.2)"
¶3: "The declaration of a member in a derived class (Clause 13) hides the declaration of a member of a base class of the same name; see §13.2"

Both of these refer to [class.member.lookup](https://timsong-cpp.github.io/cppwp/n4659/class.member.lookup)§13.2, so let's explore the details about how the name f is looked up in a class scope C:

[class.member.lookup](https://timsong-cpp.github.io/cppwp/n4659/class.member.lookup#4)§13.2¶4:
"If C contains a declaration of the name f, the declaration set contains every declaration of f declared in C that satisfies the requirements of the language construct in which the lookup occurs.(...) If the resulting declaration set is not empty, the subobject set contains C itself, and calculation is complete."

So when looking for a declaration of f and finding Derived::f, the calculation is complete, and Base is not examined at all. If no f was found in Derived, we would continue with [class.member.lookup](https://timsong-cpp.github.io/cppwp/n4659/class.member.lookup#5)§13.2¶5:

"Otherwise (i.e., C does not contain a declaration of f or the resulting declaration set is empty), S(f,C) is initially empty. If C has base classes, calculate the lookup set for f in each direct base class subobject Bi, and merge each such lookup set S(f, Bi) in turn into S(f, C).

But since Derived does indeed contain a declaration of f, we never get around to looking at Base.
</p>
</details>

#### 86. :skull::skull:
```
#include <iostream>

template <typename T> void f() {
  static int stat = 0;
  std::cout << stat++;
}

int main() {
  f<int>();
  f<int>();
  f<const int>();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 010
The functions called by f<int>() and f<const int>() actually have the same function type, but they are still distinct functions, so each get their own distinct static int stat.

When we call f<int>() for the first time, stat is 0, and 0 is printed. stat is then incremented to 1.
When we call f<int>() again, stat is 1, and 1 is printed. stat is then incremented to 2.
We then call f<const int>(), which is a distinct function with its own stat, which is 0, and 0 is printed.

This example from [temp.deduct](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct#3)§17.8.2¶3 in the standard is is relevant:

template <class T> void f(T t);
(...)
// #1: function type is f(int), t is non const
f<int>(1);
// #2: function type is f(int), t is const
f<const int>(1);

So we can see that all calls to f() in the question are to a function of the same type f(int). However, [temp.deduct](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct#4)§17.8.2¶4 says:
"f<int>(1) and f<const int>(1) call distinct functions even though both of the functions called have the same function type."

So now we know that the functions are distinct. And [temp.fct.spec](https://timsong-cpp.github.io/cppwp/n4659/temp.fct.spec#2)§17.8¶2 says:
"Each function template specialization instantiated from a template has its own copy of any static variable."

So now we know that the stat variables will also be distinct.
</p>
</details>

#### 87. :skull::skull:
```   
#include <iostream>
#include <typeinfo>

void takes_pointer(int* pointer) {
  if (typeid(pointer) == typeid(int[])) std::cout << 'a';
  if (typeid(pointer) == typeid(int*)) std::cout << 'p';
}

void takes_array(int array[]) {
  if (typeid(array) == typeid(int[])) std::cout << 'a';
  if (typeid(array) == typeid(int*)) std::cout << 'p';
}

int main() {
  int* pointer = nullptr;
  int array[1];

  takes_pointer(array);
  takes_array(pointer);

  std::cout << (typeid(int*) == typeid(int[]));
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: pp0
Functions taking pointers can also be called with arrays, and vice versa. So are arrays and pointers the same? No.

If they aren't the same, why can both functions be called with both arguments?

First let's look at takes_pointer(array);. What happens here is usually referred to as the array "decaying" to a pointer. To be a bit more precise, let's have a look at [conv.array](https://timsong-cpp.github.io/cppwp/n4659/conv.array)§7.2 in the C++ standard:

"An lvalue or rvalue of type “array of N T” or “array of unknown bound of T” can be converted to a prvalue of type “pointer to T”."

array is of type "array of 1 int", which converts to a prvalue (temporary) of type "pointer to int".

So what happens with takes_array(pointer);, does the pointer convert to an array? No, it's actually the other way around. Let's look at [dcl.fct](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct#5)§11.3.5¶5 about function parameters:

"After determining the type of each parameter, any parameter of type “array of T” (...) is adjusted to be “pointer to T”".

So in void takes_array(int array[]), the type of array is adjusted to be pointer to int.
</p>
</details>

#### 88. :skull:
```
#include <iostream>

struct C {
  C() { std::cout << "1"; }
  C(const C& other) { std::cout << "2"; }
  C& operator=(const C& other) { std::cout << "3"; return *this;}
};

int main() {
  C c1;
  C c2 = c1;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 12
On the first line, C c1;, the object c1 is default initialized, so the default constructor is called, printing 1.

On the second line, C c2 = c1;, the object c2 is copy initialized, so the copy constructor is called, printing 2. (Note that no assignment is taking place, even though there's a = involved.)

Let's have a look at the C++ standard:

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#15)§11.6¶15:
"The initialization that occurs in the form of a brace-or-equal-initializer (...) is called copy-initialization."

The initialization of C c2 = c1; is the "equal" part of a braced-or-equal-initializer, so we have copy-initialization.

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#16)§11.6¶16:
"If the destination type is a (possibly cv-qualified) class type (...) If the initialization is direct-initialization, or if it is copy-initialization where the cv-unqualified version of the source type is the same class as, or a derived class of, the class of the destination, constructors are considered. The applicable constructors are enumerated (§16.3.1.3), and the best one is chosen through overload resolution (§16.3). The constructor so selected is called to initialize the object, with the initializer expression or expression-list as its argument(s)."

The destination type here is C, and the initialization is copy-initialization. The source type is the same class as the class of the destination, so constructors are considered. The only constructor taking a C as an argument is our copy constructor, which prints 2.  
</p>
</details>

#### 89. :skull::skull:
```
#include <iostream>

int main() {
  char* a = const_cast<char*>("Hello");
  a[4] = '\0';
  std::cout << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined 
Modifying string literals is undefined behavior. In practice, the implementation can for instance store the string literal in read-only memory, such as the code segment. Two string literals might even be stored in overlapping memory. So allowing them to be modified is clearly not a good idea.

Let's look at [lex.literal](https://timsong-cpp.github.io/cppwp/n4659/lex.literal)§5.13 in the C++ standard:

According to [lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#1)§5.13.5¶1 "Hello" is a string literal:

"A string literal is a sequence of characters (as defined in §5.13.3) surrounded by double quotes, optionally
prefixed by R, u8, u8R, u, uR, U, UR, L, or LR, as in "...", R"(...)", u8"...", u8R"(...)", u"...",
uR"˜(...)˜", U"...", UR"zzz(...)zzz", L"...", or LR"(...)", respectively."

The particularity of storing of string literals is unspecified. Thus the result of modification of a string literal is undefined [1]:

[lex.string](https://timsong-cpp.github.io/cppwp/n4659/lex.string#16)§5.13.5¶16:

"Whether all string literals are distinct (that is, are stored in nonoverlapping objects) and (...) is unspecified. [Note: The effect of attempting to modify a string literal is undefined. - end note]"

[1]: Also consider that the type of the string literal is array of const char, and thus not modifiable.
</p>
</details>

#### 90. :skull::skull:
```
#include <iostream>

struct A {
  virtual int fn(int i = 5) const { return i + 1; }
};

struct B : A {
  virtual int fn(int i = 7) const override { return i + 2; }
};


void run(const A& a) {
  std::cout << a.fn(1) << a.fn();
}

int main() {
  run(B());
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 37
In both calls A::fn is overridden, so B::fn will be called. The first call a.fn(1) is trivial, the argument 1 is used, and 3 is printed.

The second call a.fn() is less trivial. The implementation doesn't care about the "virtuality" of a function when determining the default argument, but instead uses the static type of the the referenced object. Thus, the default argument 5 from A::fn will be used even if B::fn is called, and 7 is printed.

[dcl.fct.default](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#10)§11.3.6¶10 in the C++ standard:

"A virtual function call (§13.3) uses the default arguments in the declaration of the virtual function determined by the static type of the pointer or reference denoting the object. An overriding function in a derived class does not acquire default arguments from the function it overrides."
</p>
</details>

#### 91. :skull::skull:
```
#include <iostream>

struct A {
  A(int i) : m_i(i) {}
  operator bool() const { return m_i > 0; }
  int m_i;
};

int main() {
  A a1(1), a2(2);
  std::cout << a1 + a2 << (a1 == a2);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 21
For both a1 + a2 and a1 == a2, the built-in operators operator+ and operator== for int is used. The two objects a1 and a2 are implicitly converted first to bool and then to int before calling the operator. In both cases, they convert first to true, then to 1.

1 + 1 equals 2, and 1 == 1 is true, so 21 is printed.
Detailed explanation:

We'll go through the full reasoning for operator+ first:

struct A does not overload operator+, so we need to use a built-in operator+. Can we somehow convert a1 and a2 to a type compatible with operator+?

[conv](https://timsong-cpp.github.io/cppwp/n4659/conv#2)§4¶2:

    expressions with a given type will be implicitly converted to other types in several contexts:
    — When used as operands of operators
    (...)

So we are allowed to use an implicit conversion sequence to convert a1 and a2 into something we can use with a built in operator+. The specific type of implicit conversion sequence we need in this case is a user-defined conversion sequence, which is defined in [over.ics.user](https://timsong-cpp.github.io/cppwp/n4659/over.ics.user#1)§13.3.3.1.2¶1:

    A user-defined conversion sequence consists of an initial standard conversion sequence followed by a user-defined conversion (12.3) followed by a second standard conversion sequence.

As a first try, can we use the user-defined conversion to bool (our operator bool() const) and then use an operator+ for bool? [over.built](https://timsong-cpp.github.io/cppwp/n4659/over.built#9)§16.6¶9:

    For every promoted arithmetic type T , there exist candidate operator functions of the form
    T operator+(T);

And what's a promoted arithmetic type? [over.built](https://timsong-cpp.github.io/cppwp/n4659/over.built#2)§16.6¶2:

    the term promoted integral type is used to refer to those integral types which are preserved by integral promotion [...]. Similarly, the term promoted arithmetic type refers to floating types plus promoted integral types.

So is bool a promoted arithmetic type? It's certainly not a floating type, but is it a promoted integral type? No, [conv.prom](https://timsong-cpp.github.io/cppwp/n4659/conv.prom#6)§7.6¶6:

    A prvalue of type bool can be converted to a prvalue of type int, with false becoming zero and true becoming one.

So operator+ does not exist for bool, and we have to continue with our implicit conversion sequence. After the user-defined conversion from A to bool, we can use a standard conversion sequence, in particular the integral promotion we just discussed, converting true to 1. We then call operator+ for int, with the operands 1 and 1, resulting in 2.

For operator==, the story is exactly the same, as [over.built](https://timsong-cpp.github.io/cppwp/n4659/over.built#13)§16.6¶13 says:

    For every pair of promoted arithmetic types L and R , there exist candidate operator functions of the form (...)
    bool operator==(L, R);

So we use the same conversion sequence from A to bool to int, ending up comparing 1 to 1, which results in true.
</p>
</details>

#### 92. :skull::skull:
```
#include<iostream>

namespace A{
  extern "C" int x;
};

namespace B{
  extern "C" int x;
};

int A::x = 0;

int main(){
  std::cout << B::x;
  A::x=1;
  std::cout << B::x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 01
Due to the extern "C" specifications, A::x and B::x actually refer to the same variable.

x is first initialized to 0, then main() starts, 0 is printed, x is incremented to 1, and finally 1 is printed.

[dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#6)§10.5¶6 in the C++ standard:
"Two declarations for a variable with C language linkage with the same name (ignoring the namespace names that qualify it) that appear in different namespace scopes refer to the same variable."
</p>
</details>

#### 93. :skull::skull:
```
#include <vector>
#include <iostream>

std::vector<int> v;

int f1() {
  v.push_back(1);
  return 0;
}

int f2() {
  v.push_back(2);
  return 0;
}

void g(int, int) {}

void h() {
 g(f1(), f2()); 
}

int main() {
  h();
  h();
  std::cout << (v[0] == v[2]);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is unspecified / implementation defined 
The evaluation order of function argument expressions is unspecified, all we know is that they will all happen before ("be sequenced before") the contents of the called function.

In particular, in the expression g(f1(), f2()), we don't know whether f1 or f2 will be sequenced first, we only know that they will both be sequenced before the body of g.

Also: There isn't even a requirement on the implementation that f1 and f2 gets evaluated in the same order each time. So after calling h twice, v can contain 1212, 2121, 1221 or 2112.

[expr.call](https://timsong-cpp.github.io/cppwp/n4659/expr.call#5)§8.2.2¶5: in the C++ standard:

"The initialization of a parameter, including every associated value computation and side effect, is indeterminately sequenced with respect to that of any other parameter."

And the helpful note in intro.executation§4.6¶17:

"In an expression that is evaluated more than once during the execution of a program, unsequenced and indeterminately sequenced evaluations of its subexpressions need not be performed consistently in different evaluations."
</p>
</details>

#### 94. :skull::skull::skull:
```
#include <iostream>

int main() {
    int a[] = <%1%>;
    std::cout << a<:0:>;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
C++ provides alternative tokens for some punctuators. The two weird lines are exactly equivalent to

int a[] = {1};
std::cout << a[0];

[lex.digraph](https://timsong-cpp.github.io/cppwp/n4659/lex.digraph#1)§5.5¶1-2 in the C++ standard explains this:

"Alternative token representations are provided for some operators and punctuators."

"In all respects of the language, each alternative token behaves the same, respectively, as its primary token, except for its spelling."

Then, a table of alternative tokens is provided, which includes
- <% and %> for { and }
- <: and :> for [ and ]
</p>
</details>

#### 95. :skull::skull:
```
#include <iostream>
#include <cstddef>
#include <type_traits>

int main() {
  std::cout << std::is_pointer_v<decltype(nullptr)>;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 0
nullptr is a prvalue of type std::nullptr_t, which is not actually a pointer type. Instead, nullptr is a "null pointer constant" which can be converted to a pointer.

[lex.nullptr](https://timsong-cpp.github.io/cppwp/n4659/lex.nullptr#1)§5.13.7¶1 in the C++ standard:

    The pointer literal is the keyword nullptr. It is a prvalue of type std​::​nullptr_­t. [ Note: std​::​nullptr_­t is a distinct type that is neither a pointer type nor a pointer to member type; rather, a prvalue of this type is a null pointer constant and can be converted to a null pointer value or null member pointer value. See [conv.ptr] and [conv.mem].  — end note ]
</p>
</details>

#### 96. :skull::skull:
```
#include <iostream>

namespace x {
  class C {};
  void f(const C& i) {
    std::cout << "1";
  }
}

namespace y {
  void f(const x::C& i) {
    std::cout << "2";
  }
}

int main() {
  f(x::C());
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
Since the functions f are declared inside namespaces, and the call f(x::C()) is unqualified (not preceded by x:: or y::), this would normally not compile.

However, due to [argument-dependent name lookup](https://en.wikipedia.org/wiki/Argument-dependent_name_lookup) a.k.a. "Koenig lookup", the behavior is well defined.

With argument-dependent name lookup, the namespaces of the arguments to a function is added to the set of namespaces to be searched for that function. Since we're passing an x::C to f, the namespace x is also searched, and the function x::f is found.

[basic.lookup.argdep](https://timsong-cpp.github.io/cppwp/n4659/basic.lookup.argdep#1)§6.4.2¶1 in the C++ standard:
"When the postfix-expression in a function call (§8.2.2) is an unqualified-id, other namespaces not considered during the usual unqualified lookup (§6.4.1) may be searched(...). These modifications to the search depend on the types of the arguments (...)."

f is an unqualified-id, so "other namespaces" may be searched.

[basic.lookup.argdep](https://timsong-cpp.github.io/cppwp/n4659/basic.lookup.argdep#2)§6.4.2¶2 has more:
"For each argument type T in the function call, there is a set of zero or more associated namespaces and a set of zero or more associated classes to be considered. The sets of namespaces and classes are determined entirely by the types of the function arguments (...). The sets of namespaces and classes are determined in the following way:
(...)
- If T is a class type (including unions), its associated classes are: the class itself; the class of which it is a member, if any; and its direct and indirect base classes. Its associated namespaces are the namespaces of which its associated classes are members."

T in our case is C, so it's associated classes is just C. It's associated namespaces is then just x, which is searched to find a function f.
</p>
</details>

#### 97. :skull::skull:
```
#include <iostream>

namespace A {
  extern "C" { int x; }
};

namespace B {
  extern "C" { int x; }
};

int A::x = 0;

int main() {
  std::cout << B::x;
  A::x=1;
  std::cout << B::x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
A::x and B::x actually refer to the same variable, and int x; is a definition, not a declaration. Defining the same variable twice results in a compilation error.

Why do A::x and B::x refer to the same variable?

[dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#6)§10.5¶6 in the C++ standard:
"Two declarations for a variable with C language linkage with the same name (ignoring the namespace names that qualify it) that appear in different namespace scopes refer to the same variable."

Now, why is int x; a definition, and not an extern declaration? The standard declares that "a declaration directly contained in a linkage-specification is treated as if it contains the extern specifier". x is not directly contained in the linkage specification, and thus doesn't have the implicit extern. Therefore it's a definition and repeated definition of x causes a compilation error.

[dcl.link](https://timsong-cpp.github.io/cppwp/n4659/dcl.link#7)§10.5¶7 in the standard has the explanation and a relevant example:
"A declaration directly contained in a linkage-specification is treated as if it contains the extern specifier (10.1.1) for the purpose of determining the linkage of the declared name and whether it is a definition. (...) [ Example:

(...)
extern "C" int i;                   // declaration
extern "C" {
  int i;                            // definition
}
(...)

Note: This question is identical to 92. except for the {} surrounding int x;.
</p>
</details>

#### 98. :skull::skull:
```
#include <iostream>

int main() {
   constexpr unsigned int id = 100;
   unsigned char array[] = { id % 3, id % 5 };
   std::cout << static_cast<unsigned int>(array[0]) << static_cast<unsigned int>(array[1]) ;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 10
array is an array of unsigned char, but is initialized with unsigned ints. The unsigned ints have to be converted to unsigned chars, so we might expect a narrowing diagnostic. [dcl.init.aggr](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.aggr#3)§11.6.1¶3 in the standard says:

    (...) If the initializer-clause is an expression and a narrowing conversion (§11.6.4) is required to convert the expression, the program is ill-formed. (...)

[dcl.init.list](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.list)§11.k.4¶7 tells us what a narrowing conversion is:

    A narrowing conversion is an implicit conversion (...) from an integer type (...) to an integer type that cannot represent all the values of the original type

An unsigned char cannot represent all the values of an unsigned int, so we would normally expect a diagnostic. However, the above clause continues with an exception for the case where the source is a constant expression whose value after promotion fits in the target type:

    except where the source is a constant expression whose value after integral promotions will fit into the target type.

So since id is constexpr, and both id % 3 and id % 5 fits in an unsigned char, the program is well-formed.
</p>
</details>

#### 99. :skull::skull::skull:
```
#include <iostream>

int main() {
   int n = sizeof(0)["abcdefghij"]; 
   std::cout << n;   
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
We have several pieces of the puzzle, so let's peel away the layers.

The grammar for sizeof is in [expr.unary](https://timsong-cpp.github.io/cppwp/n4659/expr.unary#1)§8.3¶1:

    unary-expression:
    ...
    sizeof unary-expression
    sizeof ( type-id )
    sizeof ... ( identifier )
    ...

We have three cases and the one that applies here is sizeof unary-expression. The unary expression is (0)["abcdefghij"], which looks odd but is just array indexing of string literal which is a const char array.

We can see that (0)["abcdefghij"] is identical to ("abcdefghij")[0] from [expr.sub](https://timsong-cpp.github.io/cppwp/n4659/expr.sub#1)§8.2.1¶1 which says:

    ... The expression E1[E2] is identical (by definition) to *((E1)+(E2)) ...

So we end up with 0th element of "abcdefghij", which is a, which is a char. And the result of sizeof('a') will be 1 since [expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#1)§8.3.3¶1 says:

    ... sizeof(char), sizeof(signed char) and sizeof(unsigned char) are 1 ...
</p>
</details>

#### 100. :skull:
```
#include <iostream>
#include <map>
using namespace std;

bool default_constructed = false;
bool constructed = false;
bool assigned = false;

class C {
public:
    C() { default_constructed = true; }
    C(int) { constructed = true; }
    C& operator=(const C&) { assigned = true; return *this;}
};

int main() {
    map<int, C> m;
    m[7] = C(1);

    cout << default_constructed << constructed << assigned;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 111
The [] operator inserts an element if the key is not present. In the case of a class, the element is default constructed. So doing m[7] calls the default constructor of C (no matter if we assign to it right after), setting default_constructed to true.

The expression C(1) constructs an instance of C using the constructor taking an int, setting constructed to true.

The = in m[7] = C(1) calls the copy assignment operator to copy assign the newly created C(1) to the previously default constructed C inside the map, setting assigned to true.

The fact that an object is first default constructed is covered by [map.access](https://timsong-cpp.github.io/cppwp/n4659/map.access#2)§26.4.4.3¶2 in the standard:

    T& operator[](key_type&& x);

    Effects: Equivalent to: return try_emplace(move(x)).first->second;

where try_emplace is defined by [map.modifiers](https://timsong-cpp.github.io/cppwp/n4659/map.modifiers#8)§26.4.4.4¶8:

    template <class... Args> pair<iterator, bool> try_emplace(key_type&& k, Args&&... args);

    Effects: If the map already contains an element whose key is equivalent to k, there is no effect. Otherwise inserts an object of type value_type constructed with piecewise_construct, forward_as_tuple(std::move(k)), forward_as_tuple(std::forward<Args>(args)...).

value_type is just a typedef for pair<const Key, T>, which in our case is pair<const int, C>. So it inserts a pair(7, C()), which calls the default constructor for C.

To avoid the default construction followed by a copy assignment, you can use one of the following:

m.insert(pair<int, C>(7, C(1)));
m.emplace(7, C(1));
</p>
</details>

#### 101. :skull::skull::skull:
```
#include <iostream>

int main() {
    int i = 1;
    int const& a = i > 0 ? i : 1;
    i = 2;
    std::cout << i << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 21
The question here is whether a is a reference to i or not. If it is, the value of a changes when we do i=2. If not, a is not modified by that line.

The type and value category of the conditional expression depends on the types and value categories of the second and third expressions, in this case i and 1. For instance, if both were lvalue ints, the result would be an lvalue int. In this case however, i is an lvalue, and 1 is a prvalue, so things are a bit more complicated.

[expr.cond](https://timsong-cpp.github.io/cppwp/n4659/expr.cond)§8.16 in the standard has the details. After ruling out a bunch of options in ¶2-¶5 (none of them are void, none of them are of class type, they don't have the same value category, they aren't both a glvalue) [expr.cond](https://timsong-cpp.github.io/cppwp/n4659/expr.cond#6)§8.16¶6 says:
"Otherwise, the result is a prvalue"

So the expression i > 0 ? i : 1; is a prvalue, in other words a temporary. The reference a is bound to that temporary, not to i itself, and i = 2 ends up not modifying a. After i = 2, i is now 2, but a is still 1, and the output of the program is 21.
</p>
</details>

#### 102. :skull::skull:
```
#include <iostream>

template<typename T>
T sum(T arg) {
    return arg;   
}

template<typename T, typename ...Args>
T sum(T arg, Args... args) {    
    return arg + sum<T>(args...);
}

int main() {
    auto n1 = sum(0.5, 1, 0.5, 1);
    auto n2 = sum(1, 0.5, 1, 0.5);
    std::cout << n1 << n2;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 32
T sum(T arg, Args... args) is a function template which calls itself recursively until it hits the base case T sum(T arg). For instance, sum(a₁, a₂, a₃, a₄) is implemented as a₁ + sum(a₂, a₃, a₄).

When no T is specified, T is deduced to be the type of the first argument to sum. In the case of n1, T is deduced to double (the type of 0.5), and in the case of n2, T is deduced to int (the type of 1).

However, when sum calls itself, it explicitly specifies T (it does sum<T>(args...)). So T is only deduced on the initial call to sum, and this T is used for all the subsequent recursive calls, no matter the type of the arguments along the way.

For n1, T is always double, so sum always takes a double as the first parameter, and returns a double. The sum is (0.5 + (1 + (0.5 + (1)))) == 3. (Each set of parentheses denotes a recursive call to sum).

For n2 however, T is always int, so sum always takes an int as the first parameter, and returns a int, even when we pass 0.5. What happens when we pass a double argument to an int parameter? According to [conv.fpint](https://timsong-cpp.github.io/cppwp/n4659/conv.fpint#1)§7.10¶1 in the C++ standard:

    A prvalue of a floating point type can be converted to a prvalue of an integer type. The conversion truncates; that is, the fractional part is discarded.

So in the case of n2, each time we pass 0.5, it gets truncated to 0, and the sum is (0 + (1 + (0 + (1)))) == 2.
</p>
</details>

#### 103. :skull:
```
#include <iostream>

bool f() { std::cout << 'f'; return false; }
char g() { std::cout << 'g'; return 'g'; }
char h() { std::cout << 'h'; return 'h'; }

int main() {
    char result = f() ? g() : h();
    std::cout << result;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: fhh
f() evaluates to false, so the value of the whole conditional expression is the value of the third expression h(), that is the character h. But which functions are evaluated, and in which order?

Let's have a look at [expr.cond](https://timsong-cpp.github.io/cppwp/n4659/expr.cond#1)§8.16¶1 in the standard:
"Every value computation and side effect associated with the first expression is sequenced before every value computation and side effect associated with the second or third expression."

This means that f() completes before g() or h(), and f is the first thing to get printed.

The second question is whether both g() and h() are called before the resulting value is selected, or if only one of them are. Again, [expr.cond](https://timsong-cpp.github.io/cppwp/n4659/expr.cond#1)§8.16¶1 has the answer:
"Only one of the second and third expressions is evaluated."
Since f() evaluated to false, h() is selected, h(), and h is printed. g() is never called.

Finally, on the last line, the value of result, h is printed.
</p>
</details>

#### 104. :skull::skull:
```
#include <iostream>

struct Base {
    virtual int f() = 0; 
};

int Base::f() { return 1; }

struct Derived : Base {
    int f() override;
};

int Derived::f() { return 2; }

int main() {
    Derived object;
    std::cout << object.f();
    std::cout << ((Base&)object).f();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 22
Providing a definition for a pure virtual function is valid, as long as it is not defined directly in the function declaration. So defining Base::f() outside of Base is ok.

[class.abstract](https://timsong-cpp.github.io/cppwp/n4659/class.abstract#2)§13.4¶2 in the C++ standard:

    A pure virtual function need be defined only if called with, or as if with (§15.4), the qualified-id syntax (§8.1).

This implicitly says that a pure virtual function can in fact be defined. Furthermore:

    A function declaration cannot provide both a pure-specifier and a definition

Which is why we have to define it outside the declaration[1]. Now to the output of the program:

object.f() calls Derived::f(), returning 2.

((Base&)object).f() casts object to a Base& before calling f(), but since f() is a virtual function, Derived::f() is still called, returning 2.

[1]: This is a non-normative note; the actual normative wording that disallows it is the grammar for a function-definition does not include a pure-specifier.
</p>
</details>

#### 105. :skull::skull:
```
#include <iostream>

struct X {
    X() { std::cout << "1"; }
    X(const X &) { std::cout << "3"; }
    ~X() { std::cout << "2"; }

    void f() { std::cout << "4"; }

} object;

int main() {
    X(object);
    object.f();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 11422
First, a global X object is created, printing 1.

Then main is called, and we get to the line X(object);. This can be interpreted in two ways:
1. As creating a temporary unnamed copy of object.
2. As creating a new variable of type X named object. This is easier to see if you remove the parentheses, and it becomes just X object.

[stmt.ambig](https://timsong-cpp.github.io/cppwp/n4659/stmt.ambig#1)§9.8¶1 in the C++ standard says:

    An expression-statement with a function-style explicit type conversion (§8.2.3) as its leftmost subexpression can be indistinguishable from a declaration where the first declarator starts with a (. In those cases the statement is a declaration.

So X(object) is in fact a declaration of a new variable object (which shadows the global object). The constructor is called, and 1 is printed again.

We then call f(), and 4 is printed.

Next, main exits, and the local object is destroyed, printing 2. Finally, the global object is destroyed, again printing 2.
</p>
</details>

#### 106. :skull::skull:
```
    
#include <iostream>
#include <utility>

struct X {
    X() { std::cout << "1"; }
    X(X &) { std::cout << "2"; }
    X(const X &) { std::cout << "3"; }
    X(X &&) { std::cout << "4"; }
    ~X() { std::cout << "5"; }
};

struct Y {
    mutable X x;
    Y() = default;
    Y(const Y &) = default;
};

int main() {
    Y y1;
    Y y2 = std::move(y1);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1255
First, the line

Y y1;

creates an instance of Y. Y has an X data member, which is default constructed, printing 1.

Then,

Y y2 = std::move(y1);

copy-initializes another Y. The std::move turns y1 into an rvalue, but since Y has no move constructor, its copy constructor is called. The implicitly-defined copy-constructor performs a copy of x, as specified in [class.copy.ctor](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14)§15.8.1¶14:

    The implicitly-defined copy/move constructor for a non-union class X performs a memberwise copy/move of its bases and members.

Now the question is which copy constructor is used to initialize the copy of x. Will it pick X(X &) or X(const X &)? The Y inside the Y(const Y &) copy constructor is const, but x is marked mutable, and [dcl.stc](https://timsong-cpp.github.io/cppwp/n4659/dcl.stc#9)§10.1.1¶9 says:

    The mutable specifier on a class data member nullifies a const specifier applied to the containing class object and permits modification of the mutable class member even though the rest of the object is const

So x is considered non-const, and X(X &) is a better match to overload resolution than X(const X &), because in the latter case, a const conversion has to happen. So X(X &) is called, printing 2.

Finally, both y1 and y2 are destroyed at the end of main, printing 55.
</p>
</details>

#### 107. :skull:
```
#include <iostream>

using Func = int();

struct S {
    Func f;
};

int S::f() { return 1; }

int main() {
    S s;
    std::cout << s.f();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
using Func = int() is an alias-declaration introducing the typedef-name Func for a function taking no parameters and returning an int.

This typedef-name can be used both for free functions and, as in this question, for member functions. [class.mfct](https://timsong-cpp.github.io/cppwp/n4659/class.mfct#8)§12.2.1¶8 in the C++ standard:

    [Note: A member function can be declared (but not defined) using a typedef for a function type. The resulting member function has exactly the same type as it would have if the function declarator were provided explicitly

The use of Func f in struct S is therefore equivalent to int f().
</p>
</details>

#### 108. :skull::skull::skull:
```
template <typename ...Ts>
struct X {
  X(Ts ...args) : Var(0, args...) {}
  int Var;
};

int main() {
  X<> x;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is undefined
First, let's have a look at the initialization of X::Var. Var(0, args...) is valid if there are no template arguments, since then it's just Var(0). If there however are any template arguments, they expand to an invalid initializer for int, such as Var(0, arg1, arg2). So the only valid specialization of X is the one with no arguments.

X<> x in fact does just that, it instantiates the template with no arguments, which would be valid.

However, [temp.res](https://timsong-cpp.github.io/cppwp/n4659/temp.res#8.2)§17.6¶8.2 says:

    [if] every valid specialization of a variadic template requires an empty template parameter pack
    [the program is ill-formed, no diagnostic required.]

As we've seen, the only valid specialization requires an empty template parameter pack, so the program is ill-formed, no diagnostic required. When no diagnostic is required, the compiler is not required to diagnose the error, but execution of the program is undefined.

</p>
</details>

#### 109. :skull::skull:
```
#include <iostream>

struct X {
    int var1 : 3;
    int var2;
};

int main() {
    X x;
    std::cout << (&x.var1 < &x.var2);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
int var1 : 3; declares a bit-field, and you can not apply operator& to a bit-field. [class.bit](https://timsong-cpp.github.io/cppwp/n4659/class.bit#3)§12.2.4¶3 in the C++ standard:

    The address-of operator & shall not be applied to a bit-field, so there are no pointers to bit-fields.
</p>
</details>

#### 110. :skull:
```
#include <iostream>

struct override {};

struct Base {
    virtual override f() = 0;
};

struct Derived : Base {
    virtual auto f() -> override override{
        std::cout << "1";
        return override();
    }
};

int main() {
    Derived().f();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
Can you use override as a type name, or is it a reserved keyword?

[lex.key](https://timsong-cpp.github.io/cppwp/n4659/lex.key)§5.11 in the standard lists reserved keywords. override is not one of those. However, [lex.name](https://timsong-cpp.github.io/cppwp/n4659/lex.name#2)§5.10¶2 says about override and final:

    [override and final] have a special meaning when appearing in a certain context. [...] Unless otherwise specified, any ambiguity as to whether a given identifier has a special meaning is resolved to interpret the token as a regular identifier.

[class.mem](https://timsong-cpp.github.io/cppwp/n4659/class.mem)§12.2 lists the grammar for a class declaration, where override only has a special meaning when appearing after the declarator. It's a bit too long to reproduce here, but in all cases except for the last one in the declaration of Derived::f, override is a normal identifier, used for the type override.

If we replace the type name override with type, the definition of Derived::f becomes easier to read:

virtual auto f() -> type override{
    std::cout << "1";
    return type();
}

In main, we create an object of type Derived, call f() on it, and 1 is printed.
</p>
</details>

#### 111. :skull::skull:
```
#include <type_traits>
#include <iostream>

using namespace std;

struct X {
    int f() const&&{
        return 0;
    }
};

int main() {
    auto ptr = &X::f;
    cout << is_same_v<decltype(ptr), int()>
         << is_same_v<decltype(ptr), int(X::*)()>;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 00
[dcl.fct](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct#8)§11.3.5¶8 says:

    The return type, the parameter-type-list, the ref-qualifier, the cv-qualifier-seq, and the exception specification, but not the default arguments, are part of the function type.

This means that the ref-qualifier (in this case const&&) is part of the type and thus the type of ptr is int(X::*)() const&&.
</p>
</details>

#### 112. :skull::skull:
```
#include <iostream>

struct Foo {
  operator auto() {
    std::cout << "A";
    return 1;
  }
};

int main() {
  int a = Foo();
  std::cout << a;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: A1
A normal conversion function can have a deduced return type (only conversion function templates are not allowed so due to [class.conv.fct](https://timsong-cpp.github.io/cppwp/n4659/class.conv.fct#6)§15.3.2¶6):

    A conversion function template shall not have a deduced return type (§10.1.7.4).

And even if conversion functions don't have a return type specified in the same way as normal functions, they do have a return type: [class.conv.fct](https://timsong-cpp.github.io/cppwp/n4659/class.conv.fct#1)§15.3.2¶1:

    The type of the conversion function (§11.3.5) is “function taking no parameter returning conversion-type-id”.

Where conversion-type-id is the T in operator T() {...}. So we're allowed to deduce this type just like we deduce return types from normal functions.
</p>
</details>

#### 113. :skull::skull:
```
#include <algorithm>
#include <iostream>

int main() {
    int x = 10;
    int y = 10;

    const int &max = std::max(x, y);
    const int &min = std::min(x, y);

    x = 11;
    y = 9;

    std::cout << max << min;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1111
std::min() and std::max() return references to their smallest/largest arguments, respectively. But what if they're equal?

The C++17 standard says both for std::min() in [alg.min.max](https://timsong-cpp.github.io/cppwp/n4659/alg.min.max#3)§28.7.8¶3 and for std::max() in [alg.min.max](https://timsong-cpp.github.io/cppwp/n4659/alg.min.max#11)§28.7.8¶11:

    Returns the first argument when the arguments are equivalent.

So both the evaluation of std::max(x,y) and std::min(x,y) return x, and both references max and min are bound to x.

We then set x to 11, and print max and min, which are now both 11.

Some say it would be better if min() returned its first element, but max() returned its last element. Sean Parent explains his rationale for this in his BoostCon 2016 Keynote [Better Code](https://www.youtube.com/watch?v=giNtMitSdfQ&t=1448s).
</p>
</details>

#### 114. :skull::skull::skull:
```
#include <iostream>

using namespace std;

int main() {
    int a = '0';
    char const &b = a;
    cout << b;
    a++;
    cout << b;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 00
'0' is a character literal, with type char. (The value of '0' is actually implementation defined, but will typically be 48.) This value is then promoted to an int, and stored in a.

We then take a reference b to a. But b is a char reference, not an int reference, which means they are not reference related. [dcl.init.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.ref#4)§11.6.3¶4:

    Given types “cv1 T1” and “cv2 T2”, “cv1 T1” is reference-related to “cv2 T2” if T1 is the same type as T2, or T1 is a base class of T2.

"cv1 T1" here being char const, and "cv2 T2" being int , so they're not related. Since they're not, [dcl.init.ref](https://timsong-cpp.github.io/cppwp/n4659/dcl.init.ref#5.2.2.2)§11.6.3¶5.2.2.2 applies:

    Otherwise, the initializer expression is implicitly converted to a prvalue of type “cv1 T1”. The temporary materialization conversion is applied and the reference is bound to the result.

So the initializer expression a is converted to a temporary char const, which b refers to.

We then print b, which refers to our temporary char with the value '0'.

We then increment the original a, which importantly does not modify the temporary that b refers to.

We finally print b again, which still has the value '0'.
</p>
</details>

#### 115. :skull::skull::skull:
```
#include<iostream>

template<typename T>
void foo(T...) {std::cout << 'A';}

template<typename... T>
void foo(T...) {std::cout << 'B';}

int main(){  
   foo(1); 
   foo(1,2);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: AB
The first foo (let's call it foo₁) is a template for an old-school (pre variadic templates) variadic function. It takes a first argument of type T, followed by a varying number of arguments.

The second foo (let's call it foo₂) is a variadic function template (notice the ellipsis in typename...). It takes a parameter pack.

Which overload is selected for each of the calls too foo?

When a function is overloaded, we first need to find which of the functions are viable for a given function call. Then, we need to find out which of those viable functions is the best one. For function templates, we first perform template argument deduction to generate candidate specializations.
foo(1):

First let's look at the call foo(1). The explanation is lengthy, but rest assured we re-use most of it in the much shorter explanation of foo(1,2)!

We start by deducing the template arguments for both overloads. For foo₁, the argument 1 is used to deduce T to be int. It's signature is then foo₁(int, ...). However, for overload resolution we ignore the ... since it doesn't have a matching argument: [over.match.viable](https://timsong-cpp.github.io/cppwp/n4659/over.match.viable#2.3)§16.3.2¶2.3:

    For the purposes of overload resolution, the parameter list is truncated on the right, so that there are exactly m parameters.

So we end up with the viable function foo₁(int).

For foo₂, T... is a function parameter pack. The argument 1 is used to deduce this as one int, and it's signature is foo₂(int).

So, both foos are viable. Which one is best? [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)§16.3.3¶1:

    Given these definitions, a viable function F1 is defined to be a better function than another viable function F2 if for all arguments i, ICSi(F1) is not a worse conversion sequence than ICSi(F2), and then

    for some argument j, ICSj(F1) is a better conversion sequence than ICSj(F2)

ICSi(F1) just means "the Implicit Conversion Sequence for argument i. Our sole argument 1 is of type int, so no conversion sequence is needed for neither foo₁ nor foo₂, as they both take an int as their first argument. (i.e. neither is a better conversion sequence than the other.)

[over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)§16.3.3¶1 continues:

    [or, if not that] F1 and F2 are function template specializations, and the function template for F1 is more specialized than the template for F2 according to the partial ordering rules

I won't go into all the details for partial ordering here, but skip to the interesting parts. In short, we transform each specialization by substituting each template parameter with a unique, made up type and get foo₁(X, ...) and foo₂(Y). Then we do deduction from each transformed function to the other, original template.

Again, the ellipsis is ignored since it doesn't have an argument at the call site: [temp.deduct.partial](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct.partial#3)§17.8.2.4¶3:

    The types used to determine the ordering depend on the context in which the partial ordering is done:
    In the context of a function call, the types used are those function parameter types for which the function call has arguments.

We now do deduction of foo₂(T...) from foo₁(X), which deduces T=X. We then do deduction of foo₁(T) from foo₂(Y). It looks like we would get T=Y, but [temp.deduct.type](https://timsong-cpp.github.io/cppwp/n4659/temp.deduct.type#10)§17.8.2.5¶10 says:

    During partial ordering, if Ai was originally a function parameter pack [and] Pi is not a function parameter pack, template argument deduction fails.

Ai in this case is Y, which was originally a function parameter pack. Pi is T, which is not a function parameter pack, so deduction fails.

Since we can deduce foo₂ from foo₁, but not foo₁, from foo₂, foo₁ is more specialized and a better function. Overload resolution picks it and prints A.
foo(1,2):

Now let's look at the call foo(1,2).

Again we start by deducing the template arguments. For foo₁, the first argument 1 is used to deduce T to be int. It's signature is then foo₁(int, ...).

For foo₂, T... is a function parameter pack. The arguments 1 and 2 are used to deduce this as two ints, and it's signature is foo₂(int, int).

Which of these is the best match? We turn again to [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)§16.3.3¶1 which checks whether

    for some argument j, ICSj(F1) is a better conversion sequence than ICSj(F2)

For both templates, the first argument is an exact match. For foo₁, the second argument 2 requires an ellipsis conversion sequence to match the second parameter ...: [over.ics.ellipsis](https://timsong-cpp.github.io/cppwp/n4659/over.ics.ellipsis#1)§16.3.3.1.3¶1:

    An ellipsis conversion sequence occurs when an argument in a function call is matched with the ellipsis parameter specification of the function called

For foo₂, no conversion is required, so it's a better match. The second foo is called, and B is printed.

</p>
</details>

#### 116. :skull::skull:
```
#include <iostream>

template<class T>
void f(T) { std::cout << 1; }

template<>
void f<>(int*) { std::cout << 2; }

template<class T>
void f(T*) { std::cout << 3; }

int main() {
    int *p = nullptr; 
    f( p );
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 3
The name f is overloaded by the two function templates void f(T) and void f(T*). Note that overload resolution only considers the function templates, not the explicit specialisation void f<>(int*)! For overload resolution, first we deduce the template arguments for each function template, and get T = int * for the first one, and T = int for the second.

Both function templates are viable, but which one is best? According to [over.match.best](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)§16.3.3¶1, a function template is a better match than another function template if it's more specialised:

    a viable function F1 is defined to be a better function than another viable function F2 if (...) the function template for F1 is more specialized than the template for F2 according to the partial ordering rules described in 17.5.6.2

The process of partial ordering is a bit long to quote here, and not key to this question. But in summary, anything accepted by f(T*) would also be accepted by f(T), but not the other way around. So f(T*) is more specialised.

Now we know that the function template void f(T*) is selected by overload resolution, and we can start thinking about specialisations. Which of the function templates is void f<>(int*) a specialisation of? [temp.expl.spec](https://timsong-cpp.github.io/cppwp/n4659/temp.expl.spec#3)§17.7.3¶3:

    A declaration of a function template (...) being explicitly specialized shall precede the declaration of the explicit specialization.

So the explicit specialisation void f<>(int*) is a specialisation of void f(T), since that's the only function template declaration that precedes it. Overload resolution however selected the other function template, void f(T*), and we instead call an implicitly instantiated a specialisation of that, printing 3.

Note: This example is taken from [Why Not Specialize Function Templates?](http://www.gotw.ca/publications/mill17.htm) by Herb Sutter.
</p>
</details>

#### 117. :skull::skull:
```
#include <iostream>
#include <type_traits>

int main() {
    std::cout << std::is_same_v<
        void(int),
        void(const int)>;

    std::cout << std::is_same_v<
        void(int*),
        void(const int*)>;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 10
The constness of parameters are not part of the function type.

[dcl.fct](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct#5)§11.3.5¶5 in the standard:

    The type of a function is determined using the following rules. The type of each parameter (including function parameter packs) is determined from its own decl-specifier-seq and declarator. (...) After producing the list of parameter types, any top-level cv-qualifiers modifying a parameter type are deleted when forming the function type.

So in the first case, the type of void(const int) is actually void(int). The types are the same, and 1 is printed.

What about the second case? Here the parameter types are "pointer to int" and "pointer to const int", respectively. The pointers themselves are not const, so there's no const to remove. The types of the functions are different, and 0 is printed.

Why is is the constness of parameters not part of the function type? When an argument is passed by value, a copy is made, and the original argument is never modified anyway. So whether the parameter is const or not does not matter to the caller, it's only relevant inside the function.
</p>
</details>

#### 118. :skull:
```
#include <iostream>
#include <sstream>

int main() {
  std::stringstream ss("a");
  std::cout << ss.str();
  ss << "b";
  std::cout << ss.str();
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: ab
stringstream is backed by a character buffer. Initializing it with a string initializes the buffer with that string, in this case a. Calling str() returns the contents of the buffer, and we print a.

operator<< writes to the next position in the buffer, in this case the beginning of the buffer, overwriting whatever is already there. A b is written, overwriting the a which was previously in the buffer.

Finally str() is called again, this time printing b.

We can change this behaviour by initializing stringstream like this:

std::stringstream ss("a", std::ios_base::out|std::ios_base::ate);

Then the a doesn't get overwritten, and the program instead outputs aab, where a comes from the first call to str(), and ab comes from the second call to str().
Detail

A stringstream is an alias for basic_stringstream<char>. basic_stringstream<char> uses a basic_stringbuf<char> as storage for the characters in the stream:

using stringstream = basic_stringstream<char>

 ------------------         ---------------
|basic_stringstream|<>-----|basic_stringbuf|
 ------------------         ---------------

Let's have a look at the basic_stringstream constructor: §30.8.5.2¶2:

    explicit basic_stringstream(const basic_string<charT, traits, Allocator>& str, ios_base::openmode which = ios_base::out | ios_base::in);
    Effects: Constructs an object of class basic_stringstream<charT, traits>, initializing the base class with basic_iostream(&sb) and initializing sb with basic_stringbuf<charT, traits, Alloca-
    tor>(str, which).

sb here refers to the basic_stringbuf which backs the basic_stringstream. So this constructor initializes the buffer with "a", and sets its mode to ios_base::out | ios_base::in.

Then we call str() on the basic_stringstream, which again calls str() on the basic_stringbuf, which according to 30.8.2.3¶1:

    Returns: A basic_string object whose content is equal to the basic_stringbuf underlying character sequence.

So we get out a copy of that, and a is printed. (Note that str() does not modify the buffer.)

Then we stream "b" into the basic_stringstream. This operation is defined in terms of some of the basic_stringstream base classes, so here's (parts of) its inheritance diagram, for reference:

           --------- 
          |basic_ios|
           --------- 
          /         \
 -------------   -------------
|basic_istream| |basic_ostream|
 -------------   -------------
          \         /
         -------------- 
        |basic_iostream|
         -------------- 
               |
        ------------------ 
       |basic_stringstream|
        ------------------

basic_stringstream inherits operator<< from ostream. which according to §30.7.5.1¶2:

    generate[s] (or insert[s]) output characters by actions equivalent to calling rdbuf()->sputc(int_type).

rdbuf() here refers to our basic_stringbuf. basic_stringbuf inherits sputc from basic_streambuf. §30.6.3.2.5¶1:

    int_type sputc(char_type c);
    (...) stores c at the next pointer for the output sequence, increments the pointer

So when we stream into our stringstream, the characters get stored at the next pointer in the buffer, and the next pointer is incremented.

Since the basic_stringbuf was constructed with ios_base::out | ios_base::in, the next pointer points to the beginning of the stream, and the a already in the stream gets overwritten. If we add the flag io_base::ate, we get a different result, [ios::openmode]§30.5.3.1.4:

    ate open and seek to end immediately after opening

This will seek to the end of the stream, i.e. after the a which is already in the buffer, and the b will be appended rather than overwriting the a. So if we construct the stringstream like this:

std::stringstream ss("a", std::ios_base::out|std::ios_base::ate);

The output changes to aab, where a comes from the first call to str(), and ab comes from the second call to str().
</p>
</details>

#### 119. :skull::skull:
```
#include <iostream>

struct C {
    C() = default;
    int i;
};

int main() {
    const C c;
    std::cout << c.i;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program has a compilation error 
We're trying to default-initialize c. This is not allowed since it is const and C has a defaulted (not user-provided) constructor.

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#7)§11.6¶7:

    If a program calls for the default-initialization of an object of a const-qualified type T, T shall be a const-default-constructible class type or array thereof.

Is C const-default-constructible?

[dcl.init](https://timsong-cpp.github.io/cppwp/n4659/dcl.init#7)§11.6¶7 again:

    A class type T is const-default-constructible if default-initialization of T would invoke a user-provided constructor of T (not inherited from a base class) or if  
    - each direct non-variant non-static data member M of T has a default member initializer or, if M is of class type X (or array thereof), X is const-default-constructible,  
    - if T is a union with at least one non-static data member, exactly one variant member has a default member initializer,  
    - if T is not a union, for each anonymous union member with at least one non-static data member (if any), exactly one non-static data member has a default member initializer, and  
    - each potentially constructed base class of T is const-default-constructible.  

C does not have a user-provided constructor, and the points below don't apply either.

There are several ways we could make C const-default-constructible:  
- Give int i a default member initializer: int i{0}.    
- Remove = default from the constructor, and instead do C::C() = default; separately outside the class definition. This constructor now counts as user-provided.   
- Manually provide a constructor: C() {}  
</p>
</details>

#### 120. :skull::skull:
```
#include <iostream>
#include <string>

auto main() -> int {
  std::string out{"Hello world"};
  std::cout << (out[out.size()] == '\0');
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
Perhaps surprisingly, this program has no undefined behaviour. std::string's operator[](size_type pos) must return a reference to the null character when pos equals the length of the string:

[string.access](https://timsong-cpp.github.io/cppwp/n4659/string.access#1)§24.3.2.5¶1:

    operator[](size_type pos)
    Returns: *(begin() + pos) if pos < size(). Otherwise, returns a reference to an object of type charT with value charT(), where modifying the object to any value other than charT() leads to undefined behavior.

charT() is char() in this case. The char() expression value-initializes a char, which initializes it to 0.

So out[out.size()] == '\0' compares 0 to \0. Are they the same? Yes:

[lex.charset](https://timsong-cpp.github.io/cppwp/n4659/lex.charset#3)§5.3¶3:

    The basic execution character set and the basic execution wide-character set shall each contain all the members of the basic source character set, plus control characters representing alert, backspace, and carriage return, plus a null character (respectively, null wide character), whose value is 0

So the value of \0 is indeed 0. The comparison is true, and 1 is printed.
</p>
</details>

#### 121. :skull:
```
#include <iostream>

void f(char*&&) { std::cout << 1; }
void f(char*&) { std::cout << 2; }

int main() {
   char c = 'a';
   f(&c);
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 1
c is an lvalue char. &c returns a pointer to the lvalue c, but that pointer itself is an rvalue, since it's just a nameless temporary returned from operator&.

The first overload of f takes an rvalue reference to char *, the second takes an lvalue reference to char*. Since the pointer is an rvalue, the first overload is selected, and 1 is printed.
</p>
</details>

#### 122. :skull::skull:
```
#include <iostream>

struct A {
    A() { std::cout << "A"; }
    ~A() { std::cout << "a"; }
};

int main() {
    std::cout << "main";
    return sizeof new A;
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: main
First, the program unsurprisingly prints main. But what happens on the next line?

[expr.sizeof](https://timsong-cpp.github.io/cppwp/n4659/expr.sizeof#1)§8.3.3¶1 in the standard:

    The sizeof operator yields the number of bytes in the object representation of its operand. The operand is either an expression, which is an unevaluated operand (Clause 8) or a parenthesized type-id. 

The most common form of sizeof is probably the latter, sizeof(type-id) (e.g. sizeof(int)). But there's also the sizeof expression form (e.g. sizeof new A) which is used in this question.

The operand here is the expression new A. The type of that expression is "pointer to A". The size of a pointer varies between platforms, but that doesn't really matter in this question since we never print it. The question is whether new A constructs a new A, and if so, whether it gets destructed.

The quote above says that the expression is "an unevaluated operand", but what does that mean?

[expr](https://timsong-cpp.github.io/cppwp/n4659/expr#8)§8¶8:

    In some contexts, unevaluated operands appear (8.2.8, 8.3.3, 8.3.7, 10.1.7.2). An unevaluated operand is not evaluated. 

So the expression new A is not evaluated, and A is never constructed. The expression is only used for sizeof.

(And since there is no delete, A is never destructed either.)
</p>
</details>

#### 123. :skull::skull:
```
#include <iostream>

class show_id
{
public:
    ~show_id() { std::cout << id; }
    int id;
};

int main()
{
    delete[] new show_id[3]{ {0}, {1}, {2} };
}
```
<details><summary><b>Answer</b></summary>
<p>

#### The program is guaranteed to output: 210
The core of the question is "in what order are the objects destroyed?". In C++, objects are generally destroyed in the reverse order that they were constructed. delete[] is no exception.

[expr.delete](https://timsong-cpp.github.io/cppwp/n4659/expr.delete#6)§7.6.2.8¶6:

    In the case of an array, the elements will be destroyed in order of decreasing address (that is, in reverse order of the completion of their constructor (...).

So the objects are destroyed in the order 2, 1, 0, and 210 is printed.
</p>
</details>

