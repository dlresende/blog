---
layout: post
title: "Java: Passing Variables into Methods"
date: 2011-12-05 00:00:00 +0000
tags: ["autoboxing", "java", "overloading", "wrappers"]
---

**Java use pass-by-value semantics**, instead of pass-by-reference, when passing variables to methods.

During a method call for example, Java will copy values given by the caller and will pass them to the callee method. If the variable has a primitive type, the value of the variable is copied and passed through. If the method changes the value received as argument, the variable's value on the caller side will remain unchanged.

However, when the argument is an object, since Java copies the variable's value and, since in this case the value is a reference (the address of the object on the heap), operations performed on that object will take effect on the caller (assuming the object has mutable state).

> Code samples in this post were compiled with a JDK 6.

## Wrappers

Wrappers are Java objects that encapsulate primitive types. Wrappers have two main purposes:

- provide utility functions for primitive types
- allow primitive types to be used where only objects are accepted

Every primitive type has a wrapper class associated.

| Data Type | Wrapper Class |
| --- | --- |
| byte | java.lang.Byte |
| short | java.lang.Short |
| int | java.lang.Integer |
| long | java.lang.Long |
| float | java.lang.Float |
| double | java.lang.Double |
| char | java.lang.Character |
| boolean | java.lang.Boolean |

Since Java 5, we can use a feature called autoboxing. **Autoboxing allows us to perform conversions between primitive types and their wrappers automatically.** With autoboxing it is possible to assign wrappers to primitive types and vice-versa without the necessity of calling wrapper methods to unwrap and re-wrap. These operations are usually called boxing and unboxing.

```java
public class Wrappers {

    public static void main(String... args) {

        // Before Java 5
        Integer i = new Integer(5); // create
        int x = i.intValue();       // unwrap
        x += 10;                    // use
        i = new Integer(x);         // re-wrap

        // From Java 5 onwards
        Integer j = new Integer(5); // create
        j += 10;                    // use
    }
}
```

Observation: two autoboxed instances of types `Boolean`, `Byte`, `Character` from `\u000` to `\u007f` (`7f` is `127` in decimal), `Short` and `Integer` (from `-128` to `127`); always pass a `==` test when their primitive values are the same. More about this can be found [here](http://docs.oracle.com/javase/specs/jls/se6/html/conversions.html#5.1.7).

Quiz: what is the output of the code below?

```java
public class Quiz {
    public static void main(String... args) {
        Integer i1 = 1000;
        Integer i2 = 1000;

        if(i1 != i2) {
            System.out.print("1 ");
        }

        if(i1.equals(i2)) {
            System.out.print("2 ");
        }

        Integer i3 = 10;
        Integer i4 = 10;

        if(i3 == i4) {
            System.out.print("3 ");
        }

        if(i3.equals(i4)) {
            System.out.print("4");
        }
    }
}
```

## Overloading

Overloading methods in Java can be a little bit difficult when many methods are eligible to match to a certain call. When this kind of situation happens, **the Java compiler will use the following order of factors to determine which method will be invoked when an overloading happens:**

- **Widening**
- **Autoboxing**
- **Var-args**

When there is a dispute, widening is the winner over autoboxing and var-args. When there is a dispute between autoboxing and var-args, autoboxing wins.

```java
public class Overloading {

    static void doSomething(long x) {
        System.out.println("long");
    }

    static void doSomething(Integer i) {
        System.out.println("Integer");
    }

    static void doSomething(Integer i, Integer j) {
        System.out.println("Integer Integer");
    }

    static void doSomething(int... ints) {
        System.out.println("ints...");
    }

    public static void main(String[] args) {
        int i = 10;
        int j = 20;

        doSomething(i);
        doSomething(i, j);
    }
}
```

In the example above, we first call the method `doSomething` passing an `int` as argument. As we have a widening and an autoboxing as options, widening wins and the argument is casted from `int` to `long`. In the second call, the compiler needs to decide between autoboxing and var-args. Since the first one has higher priority, autoboxing will be used. Executing the code above gives:

```
long
Integer Integer
```

Widening reference variables is also possible in Java. In this case, widening depends on inheritance and the IS-A test is used to perform the operation. Check the code below.

```java
class Vehicle {}

class Car extends Vehicle {}

public class OverloadingReferences {

    static void doSomething(Vehicle vehicle) {}

    public static void main(String[] args) {
        Car car = new Car();
        doSomething(car);
    }
}
```

Observation: type castings using references are called *upcast* when the object is casted to one of the classes or interfaces it inherits from or implements, respectively. When the cast is made the other way round (towards a more specific object), it is called *downcast*.
