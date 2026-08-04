---
layout: post
title: "Java: Assignments"
date: 2011-11-30 00:00:00 +0000
tags: ["assignments", "java", "literals", "type casting"]
---

Before we start talking about assignments I think it is important to quick review some concepts like heap and stack.

Simply put, **the heap is a memory section where the JVM keeps the objects it creates.** **The stack is a memory section that stores method calls and local variables.**

**Based on that, we can say that:**

- **local variables live on the stack**
- **objects and instance variables live on the heap**

Back to assignments, we can say **assignments in Java are all about putting a value into a variable** using the operator `=`, something like `x = 2`.

Variables are bit holders of a specific type. Values can take the form a literal or a reference to an object. A value is a literal when it has a primitive type, like `int` or `char`; and `Stings`. A value is a reference when it holds a memory reference to something that inherits from `Object`.

> Code samples in this post were compiled with a JDK 6.

## Literals

In Java, literals can take the form of numbers, booleans, characters or strings.

## Integer/Long Literals

Integer literals can be represented as:

- octals (base 8)
- decimals (base 10)
- and hexadecimals (base 16)

Decimal literals are the most widely used and they are pretty simple.

Octal integers use digits from 0 to 7. We represent octal integers in Java by placing a 0 in front of the number. Removing the digit 0, octal numbers can have up to 21 digits.

Hexadecimals are formed by 16 symbols which are `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, a, b, c, d, e, f`. To represent hexadecimals in Java, we need to prefix the literal by *0x*. Hexadecimals can hold up to 16 digits, excluding the prefix.

Finally, in order to declare literals of type *long*, it must have the suffixes *l* or *L*.

```java
int two = 2;    // The literal 2 is a decimal value
int seven = 07; // The literal 07 is an octal value
int x = 0x007;  // The literal 0x007 is an hexadecimal
int y = 0XCAFE; // 0XCAFE is a valid value (capital and lowercase letters are accepted)
long l = 0x123l;    // An hexadecimal of type long
```

## Floating-Point Literals

Floating-point literals are defined as numbers, decimal symbols, or fractions, like: 123.456. By default, floating-points are defined as doubles (64 bits). To assign a floating-point literal to a float variable, it is necessary to use the suffixes *f* or *F*. Otherwise, the compiler will return an error in order to prevent a big number (double by default) being assigned to a small holder (float). It's also possible to suffix double literals with *D* or *d*.

```java
double d = 123.456; // 123.345 is a floating-point literal
double s = 12.34d;  // 12.34d is a floating-point suffixed with d
float  f = 12.50;   // Compiler error: possible loss of precision
float  f2 = 34.1F;  // A valid floating-point
```

## Boolean Literals

A boolean literal can only be `true` or `false`. There is no secret, if you try to put something other than these two values, the compiler will let you know you're doing something bad.

```java
boolean TRUE = true;    // Literal true
boolean FALSE = false;  // Literal false
```

## Character Literals

A literal of type *char* is represented by single quotes `''`. If you want to use an [Unicode](http://www.unicode.org/standard/standard.html) value, you need to use the unicode notation `\u` as a prefix.

In Java, the `char` type can hold up to 16 bits and can also accommodate unsigned integers, since they have 65535 as maximum value.

Finally, to use special characters you need to use the escape code `\`.

```java
char a = 'a';           // 'a' is a literal
char n = '\u004E';      // Unicode representation for the letter N
char h = 0x8;           // Hexadecimal literal
char o = (char) 88888;  // A cast is required when the literal is out of range
char l = '\n';          // New line
```

## Literals for Strings

Even if strings are not primitive types, like in many other languages we can represent strings as literals.

```java
String s = "Learning literals =]";
```

## Casting

Casting or type casting is a mechanism by which values or object references can be converted from one type to another.

**Casts can be either implicit or explicit.** Implicit casts happen in widening conversions, when putting a small value (like an `int`) into a big container (like a `long`). Implicit casts do automatic conversions, and do not require a cast operator. However, explicit casts happen when putting bit values into small containers (like putting an `int` into a `short`). In explicit casts, a cast operator is required because the compiler needs to be informed that "you know what you are doing". That's because there's a risk of losing data in the conversion. If we don't do that, we are going to get a message like "possible lost of precision".

```java
long l = 5;         // Implicit casting: int to long
float f = 100.0F;   // Explicit cast
short s = (short) f;    // Explicit cast
```

When a value is narrowed on a type casting operation, Java simply truncates the higher-order bits that won't fit. In another words, if we cast an `int` (32-bit signed) into a `short` (16-bit signed), the 16 bits on the left will just disappear when the cast is done. More details about the capacity of Java primitive data types can be found [here](http://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html).

Reference variable assignments follow the same logic. We can assign a subclass of a type, but not a superclass.

```java
class Vehicle {}

class Car extends Vehicle {}

public class CastingObjects {
    public static void main(String... args) {

        Vehicle car = new Car();    // No problem,
                                    // the Car is a Vehicle

        Car vehicle = new Vehicle();    // Compiler error :
                                        // Incompatible types
    }
}
```

## Variable Scope

Variable scope is another important element when it comes to assignments. **It is good practice to keep the scope as small as possible.** Variables with a large scope can turn the code vulnerable to bugs, and difficult to read and debug.

In Java, there are four possible scopes to variables (on ascending life time):

- Variables in a { block of code } live only during the execution of the block
- Local variables live as long as their method remains on the stack
- Instance variables are created when an instance is created and they live as long as the instance lives
- Static variables are created when the class is loaded and they stay alive as long as the class remains in the JVM

## Initialization

**Instance variables **(also called member variables)** are initialized with default values**, even if there isn't any explicit initialization. It is good practice to initialize all instance variables explicitly, so the code becomes clearer to other programmers 😉

Here are the default values used to initialize instance variables:

| Data Type | Default value |
| --- | --- |
| byte | 0 |
| short | 0 |
| int | 0 |
| long | 0L |
| float | 0.0F |
| double | 0.0 |
| char | '\\u0000' |
| boolean | false |

And what happens when the instance variable is an array? Well, since an array is an object, it will be assigned to `null`. Nevertheless, when an array is explicit initialized, default values are assigned to its elements. An array of `int`s, for instance, will have all its elements initialized to `0`. An array of Objects will have all its elements initialized t0 `null`, and so on.

On the other hand, **local variables must always be initialized**. A default value is not given to local variables. Even objects and arrays need to be given a value. When you try to use a variable that has not been initialized, you will get a compilation error.
