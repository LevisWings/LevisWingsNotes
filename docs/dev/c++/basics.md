# Basics

## Getting started with C++

### Hello World

This program prints **Hello World!** to the standard output stream:

```cpp
#include <iostream>

int main()
{
  std::cout << "Hello World" << std::endl;
}
```

<details>

<summary>Analysis</summary>

`#include <iostream>` is a preprocessor directive that includes the content of the standard C++ header file iostream.&#x20;

**iostream** is a **standard library header file** that contains definitions of the standard input and output streams. These definitions are included in the std namespace, explained below.

The **standard input/output (I/O) streams** provide ways for programs to get input from and output to an external system -- usually the terminal.

`int main() { ... }` defines a new function named main. By convention, the main function is called upon execution of the program. There must be only one main function in a C++ program, and it must always return a number of the <mark style="color:purple;">int</mark> type.

Here, the **int** is what is called the function's return type. The value returned by the main function is an **exit code**.

By convention, a program exit code of 0 or **EXIT\_SUCCESS** is interpreted as success by a system that executes the program. Any other return code is associated with an error.

If no return statement is present, the main function (and thus, the program itself) returns 0 by default. In this example, we don't need to explicitly write `return 0;`.

All other functions, except those that return the **void** type, must explicitly return a value according to their return type, or else must not return at all.

`std::cout << "Hello World!" << std::endl;` prints "Hello World!" to the standard output stream:

`std` is a namespace, and `::` is the **scope resolution operator** that allows look-ups for objects by name within a namespace.

There are many namespaces. Here, we use `::` to show we want to use **cout** from the std namespace. For more information refer to Scope Resolution Operator - Microsoft Documentation.

`std::cout` is the **standard output stream** object, defined in iostream, and it prints to the standard output (stdout).

`<<` is, in this context, the **stream insertion operator**, so called because it inserts an object into the stream object.

The standard library defines the `<<` operator to perform data insertion for certain data types into output streams. stream `<<` content inserts content into the stream and returns the same, but updated stream. This allows stream insertions to be chained: `std::cout << "Foo" << " Bar";` prints "FooBar" to the console.

`"Hello World!"` is a **character string literal**, or a "text literal." The stream insertion operator for character string literals is defined in file iostream.

`std::endl` is a special **I/O stream manipulator** object, also defined in file iostream. Inserting a manipulator into a stream changes the state of the stream.

The stream manipulator `std::endl` does two things: first it inserts the end-of-line character and then it flushes the stream buffer to force the text to show up on the console. This ensures that the data inserted into the stream actually appear on your console. (Stream data is usually stored in a buffer and then "flushed" in batches unless you force a flush immediately.)

An alternate method that avoids the flush is:

```cpp
std::cout << "Hello World!\n";
```

where `\n` is the character escape sequence for the newline character.

The semicolon (**;**) notifies the compiler that a statement has ended. All C++ statements and class definitions require an ending/terminating semicolon.

</details>

### Comments

A **comment** is a way to put arbitrary text inside source code without having the C++ compiler interpret it with any functional meaning. Comments are used to give insight into the design or method of a program.

There are two types of comments in C++:

```c
int main()
{
 // This is a comment.
 int a;
 /* A block comment with the symbol /*
 Note that the compiler is not affected by the second /*
 however, once the end-block-comment symbol is reached,
 the comment ends.
 */
 int b;
}
```

### Functions

A **function** is a unit of code that represents a sequence of statements.

Functions can accept **arguments** or values and **return** a single value (or not). To use a function, a **function call** is used on argument values and the use of the function call itself is replaced with its return value.

Every function has a **type signature** -- the types of its arguments and the type of its return type.

Functions are inspired by the concepts of the procedure and the mathematical function.

{% hint style="info" %}
**Note**: C++ functions are essentially procedures and do not follow the exact definition or rules of mathematical functions.
{% endhint %}

Functions are often meant to perform a specific task. and can be called from other parts of a program. A function must be declared and defined before it is called elsewhere in a program.

{% hint style="info" %}
**Note**: popular function definitions may be hidden in other included files (often for convenience and reuse across many files). This is a common use of header files.
{% endhint %}

```cpp
// Function Declaration:
int add2(int i); // The function is of the type (int) -> (int)
// ------------------
// Function Definition:
int add2(int i) // Data that is passed into (int i) will be referred to by the
{               // name i while in the function's curly brackets or "scope."

 int j = i + 2; // Definition of a variable j as the value of i+2.
 return j;      // Returning or, in essence, substitution of j for a function
                // call to add2.
}
// ------------------
// Function Overloading:

int add2(int i) // Code contained in this definition will be evaluated
{               // when add2() is called with one parameter.
 int j = i + 2;
 return j;
}
int add2(int i, int j) // However, when add2() is called with two parameters, the
{ // code from the initial declaration will be overloaded,
 int k = i + j + 2 ; // and the code in this declaration will be evaluated
 return k; // instead.
}
// Both functions are called by the same name add2, but the actual function that
// is called depends directly on the amount and type of the parameters in the call
// ------------------
// Default Parameters:
int multiply(int a, int b = 7); // b has default value of 7.
int multiply(int a, int b)
{
 return a * b; // If multiply() is called with one parameter, the
               // value will be multiplied by the default, 7.
}

int multiply(int a = 10, int b = 20); // This is legal
int multiply(int a = 10, int b); // This is illegal since int a is in the former
// Default arguments must be placed in the latter arguments of the function
```

### Visibility of function prototypes and declarations

In C++, code must be declared or defined before usage. For example, the following produces a compile time error:

```cpp
int main()
{
 foo(2); // error: foo is called, but has not yet been declared
}
void foo(int x) // this later definition is not known in main
{
}
```

There are two ways to resolve this: putting either the definition or declaration of foo() before its usage in `main()`. Here is one example:

```cpp
void foo(int x) {} //Declare the foo function and body first
int main()
{
 foo(2); // OK: foo is completely defined beforehand, so it can be called here.
}
```

However it is also possible to "forward-declare" the function by putting only a "prototype" declaration before its usage and then defining the function body later:

```cpp
void foo(int); // Prototype declaration of foo, seen by main
 // Must specify return type, name, and argument list types
int main()
{
 foo(2); // OK: foo is known, called even though its body is not yet defined
}
void foo(int x) //Must match the prototype
{
 // Define body of foo here
}
```

### Integer literal

An integer literal is a primary expression of the form:

* decimal-literal: It is a non-zero decimal digit (1, 2, 3, 4, 5, 6, 7, 8, 9), followed by zero or more decimal digits (0, 1, 2, 3, 4, 5, 6, 7, 8, 9).
* octal-literal: It is the digit zero (0) followed by zero or more octal digits (0, 1, 2, 3, 4, 5, 6, 7).
* hex-literal: It is the character sequence 0x or the character sequence 0X followed by one or more hexadecimal digits (0, 1, 2, 3, 4, 5, 6, 7, 8, 9, a, A, b, B, c, C, d, D, e, E, f, F).
* binary-literal (since C++14): It is the character sequence 0b or the character sequence 0B followed by one or more binary digits (0, 1)

```cpp
int d = 42; // dicimal-literal
int o = 052 // octal-literal
int x = 0x2a; int X = 0X2A; // hex-literal
int b = 0b101010; // binary-literal (C++14)
```

### Data types

```cpp
#include <iostream>
#include <limits.h>
#include <float.h>

int main()
{
  // Numerical
  std::cout << "short: " << sizeof(short) << " bytes - Range: " << SHRT_MIN << "/+" << SHRT_MAX << std::endl;
  std::cout << "int: " << sizeof(int) << " bytes - Range: " << INT_MIN << "/+" << INT_MAX << std::endl;
  std::cout << "long: " << sizeof(long) << " bytes - Range: " << LONG_MIN << "/+" << LONG_MAX << std::endl;
  std::cout << "float: " << sizeof(float) << " bytes - Range: " << FLT_MIN << "/+" << FLT_MAX << std::endl;
  std::cout << "double: " << sizeof(double) << " bytes - Range: " << DBL_MIN << "/+" << DBL_MAX << std::endl;
  // Textual
  char c = 'a'; // Example
  std::cout << sizeof(char) << std::endl;
  char a = 80; // ASCII code
  // The ascii code has all UTF-8 characters, which is 255,
  // the amount that the char value has in C++ (1 byte = 8 bits = 255 decimal).
  std::cout << a << std::endl;

  std::string s = "Hello World!";
  std::cout << s << std::endl;
  std::cout << sizeof(std::string) << std::endl;

  // Boolean
  bool t = true; bool t1 = 1;
  std::cout << t << std::endl; // Return 1 (true)
  bool f = false; bool f1 = 0;
  std::cout << f << std::endl; // Return 0 (false)
  return 0;
}

```

### Operators

```cpp
#include <iostream>

int main()
{
  // Arithmetic operators
  int sum = 10 + 10;
  int subtraction = 10 - 5;
  int multiplication = 5 * 5;
  int division = 4 / 2;
  int module = 10 % 3;
  std::cout << module << std::endl;

  // Assigment operators
  int a = 20; // =
  a = a + 10; // +
  a += 10; // Simplification
  // a <OPERATOR>+ = 10
  // Prints the current value of "a" and then increments it by 1:
  std::cout << a++ << std::endl;
  std::cout << a << std::endl;
  // There is also subtraction: a--, --a

  // Comparison operators (Relational)
  // ==
  // !=
  // <
  // >
  // <=
  // >=
  // All return a boolean value.

  // Logical operators
  int g = 10, p = 20;
  std::cout << ((g < p) && (g > 5)) << std::endl;
  std::cout << (!(g < p)) << std::endl; // The ! operator changes the boolean value
  return 0;
}
```

#### Bitwise Operators

1. The **& (bitwise AND)** in C or C++ takes two numbers as operands and does AND on every bit of two numbers. The result of AND is 1 only if both bits are 1. \
   &#x20;
2. The **| (bitwise OR)** in C or C++ takes two numbers as operands and does OR on every bit of two numbers. The result of OR is 1 if any of the two bits is 1. \
   &#x20;
3. The **^ (bitwise XOR)** in C or C++ takes two numbers as operands and does XOR on every bit of two numbers. The result of XOR is 1 if the two bits are different. \
   &#x20;
4. The **<< (left shift)** in C or C++ takes two numbers, left shifts the bits of the first operand, the second operand decides the number of places to shift. \
   &#x20;
5. The **>> (right shift)** in C or C++ takes two numbers, right shifts the bits of the first operand, the second operand decides the number of places to shift. \
   &#x20;
6. The **\~ (bitwise NOT)** in C or C++ takes one number and inverts all bits of it.

```cpp
#include <iostream>
using namespace std;

int main() {
	// a = 5(00000101), b = 9(00001001)
	int a = 5, b = 9;

	// The result is 00000001
	cout<<"a = " << a <<","<< " b = " << b <<endl;
	cout << "a & b = " << (a & b) << endl;

	// The result is 00001101
	cout << "a | b = " << (a | b) << endl;

	// The result is 00001100
	cout << "a ^ b = " << (a ^ b) << endl;

	// The result is 11111010
	cout << "~(" << a << ") = " << (~a) << endl;

	// The result is 00010010
	cout<<"b << 1" <<" = "<< (b << 1) <<endl;

	// The result is 00000100
	cout<<"b >> 1 "<<"= " << (b >> 1 )<<endl;

	return 0;
}

// This code is contributed by sathiyamoorthics19
```

### Typecasting

Typecasting is making a variable of one type, such as an int, act like another type, a char, for one single operation.

```cpp
#include <iostream>

using std::cout;
using std::endl;
using std::string;

int main()
{
  // Float to int:
  float a = 10.89;
  cout << (int) a << endl;
  // Int to string:
  int b = 9;
  string s1 = "This number is: ";
  string s2 = std::to_string(b);
  cout << s1 + s2 << endl;
  return 0;
}
```

### Conditions

```cpp
#include <iostream>

using namespace std;

int main()
{
  int a;
  cin < a;

  if (a > 100) {
    cout << "Your number is greater than 100!" << endl;
  } else if (a > 50) {
    cout << "Greater than 50!" << endl;
  } else {
    cout << "Less or equal to 50!" << endl;
  }
}

```

### Loops

```cpp
#include <iostream>

using namespace std;

int main()
{
  // While:
  int i;
  std::cin >> i;

  while (i > 0) {
    std::cout << i-- << std::endl;
  }
  std::cout << i << std::endl;

  // for:
  for (int a = 0; a < 10; a++){
    std::cout << "Hello" << std::endl;
  }
  return 0;
}

```

