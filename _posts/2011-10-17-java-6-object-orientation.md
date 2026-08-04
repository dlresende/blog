---
layout: post
title: "Java: Object Orientation"
date: 2011-10-17 00:00:00 +0000
tags: ["java", "OOP"]
---

Object-oriented programming (OOP) is a programming paradigm which uses objects to design applications. **OOP has 3 big principles: Encapsulation, Inheritance, Polymorphism**. When correctly used, these principles plus Overriding/Overloading can help developers to design flexible, extensible, and maintainable applications. When not used correctly, maintainability becomes expensive and flexibility compromised.

Nowadays, OPP seems to be a basic subject, something everyone should be comfortable with. However, having worked in many projects until now, I think this is not completely true. There are uncountable applications making poor usage of OOP concepts.

This article brings a quick review of OOP principles and some best practices. The goal here is not to detail every aspect of OOP, but to present the main principles, advantages of good usage and some examples.

> Code samples in this post were compiled with a JDK 6.

## Encapsulation

**Encapsulation consists in not exposing implementation details of a class**. It helps programmers to make changes without breaking other parts of the code. But how to do that?

- Creating *public* methods to access class' instance variables
- Protecting instance variables from exposure setting them *private*
- Using the [JavaBeans](http://download.oracle.com/javase/tutorial/javabeans/) convention

Here is an example that does not use Encapsulation:

```java
package net.diegolemos.oop;

class NotEncapsulatedClass {
    public int attribute;
}

public class NotEncapsulated {
    public static void main(String... args) {
        NotEncapsulatedClass obj = new NotEncapsulatedClass();
        obj.attribute = -300;
    }
}
```

In the previous example, the instance variable `attribute` from class `NotEncapsulated` is visible for everyone, since it is declared `public`. There are many drawbacks when designing like that. First of all, validation becomes difficult to implement when exposing a public variable. So other classes can assign any value to that variable, even a value that might be unexpected. Secondly, cohesion and loose coupling are not respected, since other components will know details from this object.

Next, we are going to refactor the above example to apply the encapsulation principles.

```java
package net.diegolemos.oop;

class EncapsulatedClass {
    private int attribute;

    public int getAttribute() {
        return attribute;
    }

    public void setAttribute(int attribute) {
        if(attribute < 0) {   // We don't want attribute smaller then zero
            this.attribute = 0;
        }
        else {
            this.attribute = attribute;
        }
    }
}

public class Encapsulated {
    public static void main(String... args) {
        EncapsulatedClass obj = new EncapsulatedClass();
        obj.setAttribute(-300);
    }
}
```

## Inheritance

I**nheritance is the mechanism by which a Java object can specialize another Java object members (instance variables and methods)**. In Java, we can create inheritance relationships by extending a class (using keyword `extends`).

**The common reasons to use inheritance are:**

- **to take advantage of polymorphism**
- **to be able to reuse code**

Reusing code is an essential aspect of OOP. Objects can inherit members of less-specialized objects avoiding code redundancy. Here is an example:

```java
package net.diegolemos.oop;

class Person {
    void talk(){}
}

class Cop extends Person {
    void arrest(Person person){}
}

public class Inheritance {
    public static void main(String... args) {
        Person bob = new Person();
        Cop cop = new Cop();

        bob.talk();
        cop.talk();
        cop.arrest(bob);
    }
}
```

**Polymorphism (which means many forms) allows specialized classes to be accessed without being necessarily known at compile time**. When used correctly, the code becomes clear and maintenance becomes easier. We will talk about polymorphism later on this post.

Note: In order to avoid the ["Deadly Diamond of Death" problem](http://en.wikipedia.org/wiki/Diamond_problem), Java does not support multiple inheritance. Each Java class can use the keyword `extends` over only one other class.

### IS-A and HAS-A relationships

**IS-A is an object relationship based on inheritance and implementation of interfaces**. If we have a class `Plane` and a class `Boeing` which inherits from `Plane`, we can say that `Boeing` IS-A `Plane`. In Java, we express IS-A relationships using the keywords `implements` to implement an interface and `extends` inherit from a class.

**A HAS-A relationship is characterized when an object uses another object as a member**. Imagine that a class `Computer` has, as an instance variable, an object called `Processor`. So we know that `Computer` HAS-A `Processor`. HAS-A relationships are also known as composition.

Even if these two relationships have each one their advantages and drawbacks, whenever you face a situation where you need to chose one of them, **favour composition over inheritance**. Composition tends to make the design more flexible and easily testable, compared to inheritance.

## Polymorphism

In Java, **Polymorphism is the ability Java objects have to assume many forms. Any Java object that pass more than one IS-A test can be considered polymorphic.** As all Java objects inherit from class [Object](http://docs.oracle.com/javase/6/docs/api/java/lang/Object.html), they are all polymorphic since they pass the IS-A test for class Object and for their own type. Logically, this rule is not applicable for class `Object` itself.

To better understand Polymorphism, let's remember a few concepts regarding reference variables:

- Since a reference to an object is stored in a variable, the variable can be reassigned to reference any other object – when the variable is not declared `final`
- We can declare a reference variable as a class type or as an interface type
- A reference variable can have only one type (once declared, the type cannot be changed)
- The methods that can be invoked on an object are determined by the reference variable's type
- A reference variable can refer to any object of the same type and to any subtype of the declared type

Since Java does not support multiple inheritance, implementing interfaces is a good way to ensure behaviour when the object has already a super class other than `Object`. Let's see an example:

```java
package net.diegolemos.oop;

import java.util.ArrayList;
import java.util.List;

class Bird {}

class Gull extends Bird implements Flyer {
    public void fly(){
        System.out.println("I can fly");
    }
}

class Chicken extends Bird {}

interface Flyer {
    void fly();
}

public class Polymorphism {
    public static void main(String... args) {
        Bird gull = new Gull();
        Bird chicken = new Chicken();

        List<Bird> birds = new ArrayList<Bird>();
        birds.add(gull);
        birds.add(chicken);

        for(Bird bird : birds){
            if(bird instanceof Flyer) {
                ((Flyer) bird).fly();
            }
        }
    }
}
```

As shown above, the classes `Gull` and `Chicken` inherit from `Bird` class. Therefore, we used an interface called `Flyer` to give some birds the ability to fly. Thanks to `instanceof` operator we can identify during runtime which birds can fly. In this example, the object `Gull` takes the form of the `Flyer` interface.

## Overloading and Overriding

**An overloaded method is a method that reuses the name of another existing one with different arguments and/or return type**. Overloading a method is generally used when developers need to redefine the signature of an existing method.

Some points are worth noting regarding methods overloading:

- A method can be overloaded in a class or in a subclass
- Overloaded methods must change the argument list
- Overloaded methods can change the return type
- Overloaded methods can change the access modifier
- Overloaded methods can change the declared checked exceptions

Bellow an example of overloading.

```java
package net.diegolemos.oop;

public class Overloading {
    int multiply(int a, int b) {
        return a * b;
    }

    double multiply(double a, double b) {
        return a * b;
    }

    public static void main(String... args) {
        Overloading calculator = new Overloading();

        calculator.multiply(5, 4);      // Will call the first method
        calculator.multiply(2.2, 4.4);  // Will call the second
    }
}
```

On the other hand, **overriding occurs when we have in a subclass a method with the same signature of a method inherited from a superclass** (unless the superclass method is marked `final`). The main advantage in overriding is that we can redefine a behaviour of a particular subclass.

When overriding methods some rules apply:

- Methods marked `final `or` static` cannot be overridden
- The parameter list must be the same compared to the overridden method (if it is not the case overloading will occur)
- The return type must be of the same type or a subtype of the return type of the original method
- The overriding method cannot be more restrictive in its access modifiers than the method being overridden (if a method have `public` access the overriding method cannot be `private`)
- A method can be overridden only if it is inherited from the super class (`private` and `final` methods are not overridden)
- Overriding methods can throw unchecked (runtime) exceptions
- The overriding method cannot throw checked exceptions that aren't the same or a subtype of the exception of the overridden method
- You must override abstract methods from abstract classes, unless the subclass is also abstract

Here's an overriding example:

```java
package net.diegolemos.oop;

class Fish {
    void swim() {
        System.out.print("Let's swim");
    }
}

class Sailfish extends Fish {
    @Override
    void swim() {
        super.swim();
        System.out.println(" fast");
    }
}

public class Overriding {
    public static void main(String... args) {
        Fish aFish = new Fish();
        Fish sailfish = new Sailfish();

        aFish.swim();
        sailfish.swim();
    }
}
```

Since Java 5, we have the [*@Override* annotation](http://download.oracle.com/javase/6/docs/api/java/lang/Override.html) to help us to override and not overload a method. If a method is annotated with the keyword *@Override* and it does not override any method, compilers will inform you ;

As you could see, this post is a quick-overview of OOP principles. To go deep in OOP, I recommend the book [SCJP Sun Certified Programmer for Java 6 Exam](https://www.goodreads.com/book/show/2593099-scjp-sun-certified-programmer-for-java-6-study-guide) (or a more recent edition) from Katherine Sierra and Bert Bates.
