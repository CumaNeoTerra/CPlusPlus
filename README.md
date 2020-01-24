# CPlusPlus
Based on the site http://cppquiz.org/quiz, extraced questions for easier overview.

The more :skull: you see, the higher the difficulty!

According to the C++17 standard, what is the output of the following programs? 
 
#### 1. :skull:   
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

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>

```
```
<details><summary><b>Answer</b></summary>
<p>

#### 

</p>
</details>
