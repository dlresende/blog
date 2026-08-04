---
layout: post
title: "Java: Garbage Collector"
date: 2011-12-09 00:00:00 +0000
tags: ["garbage collector", "java"]
---

> Code samples in this post were compiled with a JDK 6.

Garbage collection is one of the big advantages of Java. Many languages like C and C++ do not offer an automatic mechanism to perform memory management. Manually releasing every object which is not being used anymore could become a fastidious and a really difficult task.

When memory is not correctly managed, objects can stay allocated indefinitely, making applications to consume more memory than what they actually need. This can lead to performance and memory management issues, and eventually make the application crash. Such problem is known as [memory leak](https://en.wikipedia.org/wiki/Memory_leak).

**The garbage collector (GC) removes from the [heap (memory area where Java objects live)](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html#wp1085825) objects that were discarded and won't be used anymore by a Java program.**

It is the JVM that decides when to run the garbage collector. Throughout a Java program, it's possible to ask the JVM to run the garbage collector using `System.gc()`. Nevertheless, there are no guarantees that the request will be attended. So, **it's not a good practice to design an application relying on the execution of the garbage collection**.

In Java, objects are allocated on the heap space. The mission of the garbage collector is to make sure that the heap has as much free space as possible. It ensures that the available memory will be managed efficiently, but it cannot guarantee that there will be always enough space.

It is important to identify when an object is eligible for garbage collection. In Java, **an object is eligible for deletion when it cannot be accessed by any live thread**. In other words, if all the references to an object cannot be reached by a live thread the object is marked for collection. Obviously, if there are no reachable references to an object, we don't care about that object anymore.

**There are basically three ways to mark an object for garbage collection:**

- **Nullifying a reference**
- **Reassigning another value to the reference**
- **Isolating a reference**

```java
public class GC {
    GC gc;

    public static void main(String... args) {
        // Nulling the object
        StringBuilder nulling = new StringBuilder("nulling");
        nulling = null;  // nulling is now eligible for garbage collection

        // Reassigning a new value to the object
        StringBuilder first = new StringBuilder("first string");
        StringBuilder second = new StringBuilder("second string");

        first = second; // now second is marked to be collected

        // Isolated island
        GC gc1 = new GC();
        GC gc2 = new GC();
        GC gc3 = new GC();

        // creating a circular reference
        gc1.gc = gc2;
        gc2.gc = gc3;
        gc3.gc = gc1;

        // now there is an island that cannot be reached
        gc1 = gc2 = gc3 = null;
    }
}
```

Finally, it is also possible to execute some operations just before an object is deleted by the garbage collector. In order to do that we need to override the method `finalize()` inherited from class `Object`. But be careful, as we have no guarantees about when the GC will run, the code inside the method `finalize()` can wait a very long time to be executed. For that reason, it is recommended to not use the `finalize()` method to release resources allocated by an object. Use an explicit method call for that instead.
