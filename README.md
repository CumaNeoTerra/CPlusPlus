# CPlusPlus
Based on the site http://cppquiz.org/quiz, extracted questions for easier overview.

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

#### 17. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

####  

</p>
</details>

#### 18. :skull:
```
```
<details><summary><b>Answer</b></summary>
<p>

####  
</p>
</details>