---
layout: post
title: "Java: Generic Types"
date: 2012-02-15 00:00:00 +0000
tags: ["generics", "java"]
---

> Code samples in this post were compiled with a JDK 6.

Before Java 5, it was not possible to instruct to a collection (like a List, a Map or a Set) to accept only objects of a specific type. At that moment, collections could hold anything that was not a primitive type ([which are boxed when added to the collection](https://diegolemos.net/2011/12/05/java-6-passing-variables-into-methods/)). For that reason, applications before Java 5 were "less type safe" and more error prone when manipulating collections.

Here's an example:

```java
List list = new ArrayList();

// We can add whatever we want...
list.add("string");
list.add(1);
list.add(4.0);
list.add(new Object());
```

This approach can be a little bit dangerous, particularly when working with external APIs.

Suppose that in your code you call a method of an external class (third party code) that manipulates lists of `String`s and gets a `List` as parameter. For any reason, you put an `Integer` inside a `List` object and you pass it to the method. Inside the method, a cast is required to get elements of the list and assign them to variables, like `String value = (String) list.get(0)`. Since you can have limited or even no access to the foreign code, you can inadvertently get a `RuntimeException` when the method will try to cast the `Integer` element into a `String` O_o

Java 5 introduced [Generics](http://docs.oracle.com/javase/1.5.0/docs/guide/language/generics.html). **With generics, we can guarantee which object type a collection can hold.** If we declare a collection with a given type and we try to add to it an object of a different type, we get a compilation error. **The type validation is performed by the compiler. Generics are not present inside the bytecode, so they are not used on runtime.**

Using generics is simple! Just add a type inside angle brackets `<>` immediately following the collection type. This needs to be done in the variable declaration and in the constructor call. Finally, generics can also be used as type parameters and return types for methods.

```java
import java.util.ArrayList;
import java.util.List;

public class GenericTypes {
    public static void main(String... args) {
        GenericTypes gt = new GenericTypes();

        // We define a list of Strings
        List<String> people = gt.getArrayList();
        people.add("Alice");
        people.add("Bob");

        gt.printList(people);
    }

    void printList(List<String> list) { // Now casts are not necessary
        for(String name : list) {
            System.out.println(name);
        }
    }

    List<String> getArrayList() {
        return new ArrayList<String>();
    }
}
```

## Legacy code

As said before, generic types are not present during runtime. The type compliance check is made by the compiler. Once the code is compiled, collections post and pre-Java 5 are the same thing. The reason is simple: retro-compatibility. Sun did not want to break code written before Java 5, so the type verification does not exist in runtime. By doing this, legacy and post-Java 5 code can interoperate.

```java
import java.util.ArrayList;
import java.util.List;

public class LegacyExample {
 public static void main(String... args) {
  List<String> list = new ArrayList<String>();
  list.add("1");
  list.add("2");

  LegacyExample le = new LegacyExample();
  le.addObject(list);
 }

 void addObject(/*Parameters here are non-type safe*/ List list) {
  list.add(1);
 }
}
```

The code above compiles and runs without errors. Even if the object added to `list` object inside the `addObject` method is different from the type accepted by the *List*, it will work. This happens because type-safe protection does NOT exist at runtime. Nevertheless, the compiler will identify that the code is not as safe as it could be and will issue warnings:

```bash
$ javac LegacyExample.java
Note: LegacyExample.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

As pointed out in the warning, it is possible to identify which method is to blame. To find that out, just recompile with `-Xlint:unchecked` option.

```bash
$ javac -Xlint:unchecked LegacyExample.java
LegacyExample.java:18: warning: [unchecked] unchecked call to add(E) as a member of the raw type java.util.List
list.add(1);
        ^ 1 warning
```

At this point, you may have a question. If it is not possible to have type-safe protection at runtime like arrays, what are the advantages in using generics?

**Generic types bring mainly two advantages:**

- **Type-safe verification at compile time**
- **No need for object casting when you get something out of a collection **

## Generics and Polymorphism

It is important to point out that there are a couple of differences between arrays and generic collections. Let's see some examples:

```java
import java.util.ArrayList;
import java.util.List;

class Professional{}
class Engineer extends Professional {}

public class ArraysXCollections {
 public static void main(String... args) {
  Professional[] professionals = new Engineer[5]; // This works fine
  List<Professional> professionalList = new ArrayList<? extends Professional>(); // But this does not compile
 }
}
```

**Generic types do not support natural polymorphism**. The reason is simple. Imagine that we have a method receiving `List list` as parameter. So, we could perfectly pass an `ArrayList list` to that method. Right. The problem is that, by doing this, we allow the code inside the method to add to the list any object that passes an [IS-A](https://diegolemos.net/2011/10/17/java-6-object-orientation/) test with `Professional` that is not an `Engineer`. In the end, there is no guarantee that the collection of engineers given to the method in the past still holds only engineers and this can be dangerous :S

However, it is possible to add `Engineer`s into a collection of `Professional`s:

```java
public class CollectionsExample {
    public static void main(String... args) {

        List<Professional> professionalList = new ArrayList<Professional>(); // No problem here

        professionalList.add(new Engineer());   // This is allowed

        addElements(professionalList);          // Everything is fine
    }

    static void addElements(List<Professional> list) {
        list.add(new Engineer());
    }
}
```

**To solve the problem of manipulating sub and super types in generic collections, Java has a mechanism called "wildcards"**, like ``. By using the type `List` as a method parameter, we can pass any list to that method since its generic type passes an IS-A test for `Professional`. The only restriction is, when using something like `List` (or even simply `List`), it is not possible to insert any object into the list (otherwise, a compiler error will occur):

```java
import java.util.ArrayList;
import java.util.List;

public class Wildcard {

    public static void main(String... args) {
        List<Engineer> engineers = new
        ArrayList<Engineer>(); addElement(engineers);
    }

    static void addElement(List<? extends Professional> list) {
        list.add(new Engineer());   // This line does not compile
    }
}
```

In the example above, if we delete the instruction `list.add(new Engineer())` the code compiles and runs fine.

On the other hand, it is possible to deal with the insert restriction imposed by the keyword `extends` used with the wildcard `?`. By using the keyword `super`, it is possible to add elements into a collection received as parameter.

```java
public class Wildcard2 {
    public static void main(String... args) {
        List<Professional> list = new ArrayList<Professional>();
        addElements(list);
    }

    static void addElements(List<? super Engineer> list) {
        list.add(new Engineer()); // this line compiles fine
    }
}
```

When the keyword `super` is used, `Engineer`s and any super-type of `Engineer` – like `Professional`, and obviously `Object` – will be accepted. Maybe this will not shock you, since upcasts are natural in Java (like putting a `String` into an `Object`).

Summarizing, `List` and `List` are the same thing. Nevertheless, neither of them are the same as `List`, because the later will leave you add elements inside.** Wildcards are used only for reference declarations like arguments, return types and variables. Wildcards cannot be used as generic type to create new typed collections.**

## Parameterised Types

**It is possible to use generic types to create "template classes" that can safely manipulate objects of any type.** In this case, generic types are used as instance variables giving a class the chance to manipulate objects generically:

```java
class Test<T> {
    T t;

    public Test(T t) {
        this.t = t;
    }

    public T getT() {
        return t;
    }
}

public class GenericTypeExample {
    public static void main(String...  args) {
        Test<String> test = new Test<String>("Generics");
        System.out.print(test.getT());  // will print Generics
    }
}
```

In the example above, `T` is a placeholder used to designate a parameter type for the class. We could have used anything else, but `T` is a convention for class parameter types. The placeholder `E`, for example, designate "Element" and is used when the template is a collection (see [List](http://docs.oracle.com/javase/6/docs/api/java/util/List.html) for example). The wildcard notation can be used in many ways to combine types and create templates.

```java
class Converter<T, Y> {}
class Calculator<T extends Number> {}
public class GenericTypeExample2 {
    Converter<Integer, Long> converter = new Converter<Integer, Long>();
    Calculator<Integer> calculator = new Calculator<Integer>();
}
```

However, it is also possible to use parameterised types to ensure type safety only in methods. This is used when only the method needs to be aware of the generic type, not the whole class.Let's suppose we need a generic method to create a parameterised *Map* for cache purposes.

```java
import
java.util.HashMap;import
java.util.Map;class Cache {
    static <K, V>  Map<K,V> getCache() {
        return new HashMap<K, V>();
    }
}

public class CacheTest {
    public static void main(String...  args) {
        Map<Integer, String> cache = Cache.<Integer,String>getCache();
        cache.put(1, "test1");
        cache.put(2, "test2");
    }
}
```

In this example, `K` is replaced by `Integer` and `V` by `String`. In generic methods, it is necessary to declare the parameterised types before using them: `static  Map getCache()`. We must do this in order to define what `K` and `V` are before using them as parameters or return types. If the type is defined in the class, it is not necessary to re-define it in the method signature. Worth noting it is also possible to use boundaries in generic methods: `static <K, V extends Object> Map getCache()`.

Update: From Java 7 on, the [diamond operator](https://docs.oracle.com/javase/7/docs/technotes/guides/language/type-inference-generic-instance-creation.html) was introduced in order to spare developers of repeating the generic type twice, like:

```java
// Instead of doing:
List<Professional> list = new ArrayList<Professional>();

// You can now do:
List<Professional> list = new ArrayList<>();
```
