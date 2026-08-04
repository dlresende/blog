---
layout: post
title: "Java: Inner Classes"
date: 2012-07-15 00:00:00 +0000
tags: ["inner classes", "java"]
---

Since Java 1.1, [when inner classes were first introduced](http://www.public.iastate.edu/~java/docs/guide/innerclasses/html/innerclasses.doc.html), they have started a lot of discussion. Some people like them and find them useful. Others hate them. Point of views apart, the fact is that inner classes can be helpful in many situations.

Basically, inner classes give us the ability to define one class within another.

In general, **we divide classes in four groups:**

- **Inner classes**
- **Anonymous inner classes**
- **Method-local inner classes**
- **Static inner classes**

> Examples showed on this post were built using a JDK 6.

## Inner Classes

**Inner classes are generally used when some pieces of the code of different classes are intimately tied.**

Let's use a chat application as an example. When someone is chatting, the application needs to handle a couple of things in the background, like, listening and writing into sockets, reading messages from text fields, sending messages when the "Send" button is pressed, etc. The handlers (in this case the Java components that help us to send and receive messages) are strongly tied to some components of the chat's client. These handlers need to access members of a specific object, like buttons, text fields, etc. Thus, we tend to implement these handlers as inner classes in such a way they share a special relationship with their outer class instance. As the inner class is part of the outer class, it can access the private members of its outer class.

Following a simple example of an inner class.

```java
public class OuterClass {
    private int index = 0;

    class InnerClass {
        public void printIndex() {
            // Because the inner class is a member of the
            // outer class, it can access its private members
            System.out.println("Index: " + index);
        }
    }
}
```

Compiling the above example will produce two class files:

```
OuterClass.class
OuterClass$InnerClass.class
```

**An inner class has its own class file.** Nevertheless, it is not possible to execute the inner class file in the usual way by doing `java OuterClass$InnerClass`.  That's because a simple inner class cannot have `static` declarations, and then it is not possible to put a `main` method on it.

The only way to access inner classes members is through an instance of the outer class. This is only possible at runtime when there is an instance of the inner class tied to its outer class. In other words, **an inner class instance cannot stand alone without an outer class instance.**

Following an example of an outer class using an inner class.

```java
public class MyOuterClass {
    private int index = 0;

    public void usingInnerClass() {
        // The only way to access MyInnerClass instance is
        // through MyOuterClass instance.
        MyInnerClass innerClass = new MyInnerClass();
        innerClass.printIndex();
    }

    class MyInnerClass {
        public void printIndex() {
            // Because the inner class is a member of the
            // outer class, it can access its private members.
            System.out.println("Index: " + index);
        }
    }
}
```

However, it is possible to call methods of inner classes directly from a static context. As every inner class instance is tied to its outer class instance, we need an outer class instance for this. Let's see an example:

```java
public class InnerClassAccessor {
    public static void main(String... args) {
        MyOuterClass.MyInnerClass myInnerClass = new MyOuterClass().new MyInnerClass();
        myInnerClass.printIndex(); // will print Index: 0
    }
}
```

Magic? No, it's only the keyword `new` called from an object! **Instantiating an inner class is the *only* case where we can call `new` on an object instance, rather than invoking `new` to construct an instance.** This is a special way to instantiate inner class from outside the class it lives in.

Finally, since the keyword `this` always refers to the current object instance, the same keyword used within an inner class refers to the instance of the inner class itself. So, when an inner class needs to refer to a member of the outer class, it needs to use the `NameOfTheOuterClass` followed by `.this`.

```java
public class SecondOuterClass {
    private String classAlias = "outer alias";

    class InnerClass {
        String classAlias = "inner alias";

        public void printAlias() {
            System.out.println("Outer: " + SecondOuterClass.this.classAlias);
            System.out.println("Inner: " + this.classAlias);
        }
    }

    public static void main(String... args) {
        InnerClass innerClass = new SecondOuterClass().new InnerClass();
        innerClass.printAlias();    // will print   Outer: outer alias
                                    //              Inner: inner alias
    }
}
```

## Anonymous inner classes

Anonymous inner classes are called "anonymous" because they don't have a class name. Anonymous inner classes can be used in many situations: inside a method, as a class member or even as a method argument.

Bellow an example of anonymous inner class defined inside a method.

```java
abstract class Bird {
    abstract void fly();
}

public class AnonymousInnerClass {
    public static void main(String... args) {
        Bird bird = new Bird() {
            @Override
            void fly() {
                System.out.println("Flying...");
            }
        };  // Semicolon required!

        bird.fly(); // Will print Flying...
    }
}
```

It is important to note that **when using anonymous inner classes as a class member or a local variable inside a method, the semicolon is required at the end of the class instantiation**. If the semicolon is omitted the code does not compile.

Anonymous inner classes are available in two flavours. The first one is called *anonymous subclass.* **Anonymous subclasses are created extending a class**. The second flavour is called ***anonymous implementer*** and **is created when implementing anonymously an interface**.

Another point to be mentioned is that methods defined inside anonymous inner classes other those overridden from the super class or the interface cannot be called from the outside. Since Java is a [strongly typed language](http://en.wikipedia.org/wiki/Strong_typing), only methods defined by the type can be called.

## Method-local inner classes

Simple inner classes are defined inside a class, more specifically inside the curly braces of a class. **Method-local inner classes are inner classes defined inside methods and can only be used within the method they are defined in**. The only place from where we can create an instance of a method-local inner class is inside the method where the same inner class was defined.

```java
public class MethodLocalInnerClass {
    private int x = 10;

    void printFromInner() {
        class InnerClass {
            public void printX() {
                System.out.println("x=" + x);
            }
        }

        InnerClass innerClass = new InnerClass(); // must come after the class
        innerClass.printX();
    }
}
```

Worth noting: **method-local inner classes cannot access the local variables of the method the inner class was defined in** (if you try the compiler will complain). The reason is that, as the local variables of the method live in the stack, they exist only during the method execution lifetime. So, if an instance of a method-local inner class is passed to another class and it is being used there, when the method ends, its local variables won't exist anymore. We can only use method-local variables inside method-local inner classes if they were declared `final`. Once it has been assigned, the value of the `final` variable cannot change. This allows the Java compiler to "capture" the value of the variable at runtime and store a copy as a field in the inner class.

```java
class MethodLocalInnerClass2 {
    private int x = 10;

    public static void main(String... args) {
        MethodLocalInnerClass2 clazz = new MethodLocalInnerClass2();
        clazz.printFromInner(); // will print x=101
    }

    void printFromInner() {
        final int y = 1;
        class InnerClass {
            public void printX() {
                System.out.println("x=" + x + y);
            }
        }

        InnerClass innerClass = new InnerClass(); // must come after the class
        innerClass.printX();
    }
}
```

And only to remember, **local variables rules are also applied to method-local inner classes** and they cannot be declared `public`, `private`, `static`, `protected`, `transient`, etc. Only `abstract` and `final` are allowed.

[This question](http://stackoverflow.com/questions/1183453/whats-the-use-of-a-method-local-inner-class) on stackoverflow contains some good examples regarding the usage of method-local inner classes.

## Static inner classes

**Static inner classes** or static nested classes **are members of the outer class that can be accessed from a static context**. In this case, static inner classes can be accessed like any other static member, without having an instance of the outer class.

Although, unlike other inner classes, static inner classes don't have any kind of special relationship with the outer class, except the namespace. In other words, they cannot access non-static members of the outer class.

```java
class HelperClass {
    static class InnerHelper {
        void print() {
            doPrint();
        }
    }

    static void doPrint() {
        System.out.println("Printing from HelperClass");
    }
}

public class StaticInnerClassExample {
    static class AnotherHelper {
        void print() {
            System.out.println("Printing from AnotherHelper");
        }
    }

    public static void main(String... args) {
        /* Note the way we create an instance of a static inner class
         is not the same as for inner classes */
        HelperClass.InnerHelper h1 = new HelperClass.InnerHelper();
        h1.print(); // will print: Printing from HelperClass
        AnotherHelper h2 = new AnotherHelper();
        h2.print(); // will print: Printing from AnotherHelper
    }
}
```

Note: A non-static nested class (or "inner class") has full access to the members of the class within which it is nested. A static nested class does not have a reference to a nesting instance, so a static nested class cannot invoke non-static methods or access non-static fields of an instance of the class it is nested on.
