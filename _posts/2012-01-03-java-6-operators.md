---
layout: post
title: "Java: Operators"
date: 2012-01-03 00:00:00 +0000
tags: ["java", "operators"]
---

As in mathematics, operators in Java are used to produce new values from operands. Choosing the right operator at the right time is important in order to write clear code in a fast way.

> Code samples in this post were compiled with a JDK 6.

## Assignment Operators

The most common assignment operator in Java is the `=` operator. This operator simply assign values to variables. Nevertheless, the `=` operator can be composed with other operators like `+`, `-`, `*`, `/`, `>>`, `<<`, `&` and `|` forming the compound operators: `+=`, `-=`, `*=`, `/=`, `>>=`, `<<=`, `&=` and `|=`.

```java
int i = 0;
int j = 0;

i += 2;         // Which is the same that i = i + 2;
j *= 10 + 4;    // Which is the same that j = j * (10 + 4)
```

## Relational Operators

Relational operators are often used on `if` tests and result in a `boolean` value (`true` or `false`). Java has six relational operators which are: `>`, `>=`, `<`, `<=`, `==`, and `!=`. All of them can perform comparisons on integers, floating-points or characters. Only the last two (`==` and `!=`) are used to perform comaprisons on `boolean` values, or object reference variables.

```java
char sex = 'f';

if(sex
Relational r1 = new Relational();
Relational r2 = new Relational();

if(r1 == r2) {} // false

r1 = r2;        // now they hold the same reference

if(r1 == r2) {} // true
```

## *instanceof* Operator

The *instanceof* operator verifies if an object is of a particular type. To evaluate the type of an object (on the left), the operator uses a IS-A test for a class or interface (on the right). The IS-A test returns `true` if the object inherits from the class or implements the interface used in the test. Otherwise, the result is `false`.

```java
class A {}
class B extends A {}

public class InstanceOf {
    public static void main(String... args) {
        A a = new B();
        B b;

        if(a instanceof B) {    // the IS-A test passes
            b = (B)a;   // we can then downcast
        }
    }
}
```

## Arithmetic Operators

Beyond the basic arithmetic operators `+`, `-`, `*`, and `/`, there are three other operators also widely used which are:

- the Remainder operator (`%`);
- the String Concatenation operator (`+`);
- the Increment and Decrement operators (`++` and `--`, respectively).

The remainder operator `%` returns the remainder of a division operation.

```java
int x = 10;
int y = 4;

int rem = x % y;    // which will be 2
```

Observation: **in Java expressions are evaluated from left to right**. The operators `*`, `/`, and `%` have higher priority then `+` and `-` operators. Parentheses can be used to change the evaluation order.

The operator `+` can act as addition operator or string concatenation operator, which depends on the operands. If there is a string among the operands, the `+` becomes a string concatenation operator. If the operands are numbers, the `+` will perform the addition.

```java
int i = 5;
int j = 5;

System.out.println("String" + i);   // will print String5

System.out.println("String" + (i + j)); // will print String10
                                        // because the parentheses
```

Finally, the increment and decrement operators will increment or decrement variables by one. The operator can prefix or postfix the operand. When the prefix approach is used, the operator is placed before the variable, and the operation is performed before the variable is used. When the postfix approach is employed, the operator is placed after the variable, and the increment or decrement operation takes place after the variable is used.

```java
public class IncrementDecrement {
    public static void main(String... args) {

        int i = 0;

        if(++i == 1) {
            System.out.println("++i is performed before the test");
        }

        System.out.println("i = " + i--); // output : i = 1

    }
}
```

## Conditional Operators

The ternary conditional operator will decide which value to assign or return after evaluating a `boolean` expression. Here is the structure of the conditional operator:

`   (boolean expression) ? value if true : value if false`

The first value is assigned or returned if the result of the expression is `true`. Otherwise, the second value is used. The parentheses are optionals and it is possible to nest conditional operators.

```java
public class ConditionalTest {
    public static void main(String... args) {
        int i = 4;
        int j = 5;

        String result = ++i == 4 ? "i=4" :
                (i + j++ > 10) ? "i+j>10" : "i+j    }
}
```

## Logical Operators

Logical operators in Java basically evaluate an expression and return a `boolean` value.

### Short-Circuit Logical Operators

Short-circuit operators evaluate boolean expressions and return boolean values. Short-circuit operators are represented as follow:

- `||` short-circuit OR
- `&&` short-circuit AND

A short-circuit AND (*&&*) will evaluate the left side of an expression first. If the result is `false`, the operator will not waste time evaluating the right side and will return `false`. The right side will be evaluated only if the result of the left side is `true`. If the result of both sides is `true`, the hole expression is evaluated as `true`. If one of them results false, the expression is evaluated as `false`.

As the short-circuit AND, the short-circuit OR (*`||`*``) will start evaluating the left side of the expression. Nevertheless, if the result is `true` the right side will not be analysed and the result of the whole expression will be `true`. The right side will be evaluated only if the result of the expression on left side  is `false`. So, if there is one `true`, the global result is `true`. If both sides are `false`, the final result is `false`.

```java
public class ShortCircuitLogicalOperator {
    public static void main(String... args) {
        if(check(9) && check(11)) {
            System.out.println("First IF");
        }

        if(false || check(13)) {
            System.out.println("Second IF");
        }
    }

    private static boolean check(int value) {
        if(value % 2 < 2) {
            return true;
        }

        return false;
    }
}
```

The output of the code above is:

```
First IF
Second IF
```

### Not Short-Circuit Logical Operators

Not short-circuit operators are represented as follows:

- `|` non-short-circuit OR
- `&` non-short-circuit AND

As not short-circuit operators will evaluate logical expressions like the operators `&&` and `||`. On the other hand, not short-circuit operators will evaluated ALWAYS both sides of the expression. Non-short-circuit OR (`|`) will return `false `if both sides of the logical expression are `false`. It returns `true `if one side is `true`. Non-short-circuit AND (&) returns `true` only if both sides are evaluated as `true`. Otherwise, it returns `false`.

```java
public class NotShortCircuitLogicalOperator {
    public static void main(String... args) {
        int i = 0;

        if((i++ < 1 & ++i > 1) | i == 2) {
            System.out.println("Little puzzle");
        }
    }
}
```

### Boolean Invert Logical Operator

The unary `boolean` invert (`!`) operator evaluates only `boolean` expression and returns its opposite value. So, if the result is `true`, it gets `false` and vice-versa.

### Exclusive-OR (XOR) Logical Operator

The exclusive-OR (`^`) operator also used with boolean expressions and it evaluates BOTH sides of an expression. For an exclusive-OR (`^`) expression be `true`, EXACTLY one operand must be `true`.

```java
public class ExclusiveOR {

    public static void main(String... args) {
        byte size1 = -127;
        byte size2 = 127;

        if(checkSize(size1) ^ checkSize(size2)) {
            System.out.println("There is a size that passes the" +
                    " check AND a size that does not match. ");
        }
    }

    private static boolean checkSize(byte i) {
        if(i < 127) {
            return true;
        }

        return false;
    }
}
```

## Bitwise Operators

Bitwise operators are operators that evaluate the bit value of each operand. They are used in expressions with integer values. If the operand is smaller than `int`, the primitive value will be converted to an `int`.

Following a list with the bitwise operators in Java.

- `~` unary bitwise complement
- `&` bitwise AND
- `|` bitwise OR
- `^` bitwise XOR
- Shift operators: <<, >>, and >>>

The unary bitwise complement simply invert the bit value. The bitwise operators AND, OR and XOR evaluate bits using simple logic. The shift operator `<<` shifts bits of to left putting zeros on the right side (low order position). The shift operator `>>` shifts bits to the right side saving the number sign. In other words, a negative number remains negative after the operation. Finally, the shift operator `>>>` move bits to right without saving the number sign. In this case, zeros are inserted as most significant bits (on the left). All the operators can be also used with the assignment (`=`) operator.

Bitwise operators are generally used to pack a lot of information into a single variable, like masks, flags, etc.

```java
public class BitwiseOperators {
    public static void main(String... args) {
        short i = 127;
        long j = 0;
        byte k = 85; // binary: 0101 0101
        int l = 99; // binary: 0000 0000 0000 0000 0000 0000 0110 0011
        int m = -99; // binary: 1111 1111 1111 1111 1111 1111 1001 1101

        System.out.println("~i  = " + ~i);
        System.out.println("i&j = " + (i&j));
        System.out.println("i|j = " + (i|j));
        System.out.println("i^j = " + (i^k));

        j |= i;

        System.out.println("j (after j |= i) = " + j);

        System.out.println("m>>4 = " + (m>>4)); // binary: 1111 1111 1111 1111 1111 1111 1111 1001
        System.out.println("m>>>12 = " + (m>>>12)); // binary: 0000 0000 0000 1111 1111 1111 1111 1111
    }
}
```

The output of the code above will be:

```
~i  = -128
i&j = 0
i|j = 127
i^j = 42
j (after j |= i) = 127
m>>4 = -7
m>>>2 = 1048575
```

Observation: Java uses two's-complement mechanism to sign integral types. More on this [here](http://java.sun.com/docs/books/jvms/second_edition/html/Overview.doc.html#22239).
