---
title: Java Generics&#58; Type Erasure & Wildcards
author: Hamza Belmellouki
categories: [Java]
tags: [core-java]
comments: true
---  


Specifying the generic type allows Java to perform type checking at compile-time. But when using generics in your code, due to “*Type Erasure”* that occurs at compile-time, *generic type parameters* are converted to the `Object` type. This makes *generic* *type parameters* unable to call other methods except for the `Object` ones.

What if we want to invoke methods other than that those in `Object` class? This article explains what “*Wildcards*”, “*bounds*” and “*Type Erasure*” are. And how to use Wildcards to increase flexibility when using generics.

## Type Erasure

Wikipedia defines Type erasure as:
> **Type erasure** is the load-time process by which explicit type annotations are removed from a program, before it is executed at run-time.

This example uses generics so we can use it with multiple types.

```java
class Container<T> {
    private T contents;

    public Container(T contents) {
        this.contents = contents;
    }

    public T getContents() {
        return contents;
    }

    public static void main(String[] args) {
        Container<String> container = new Container<>("Hello!");
        String contents = container.getContents();
        System.out.println(contents);// Hello!
    }
}
```

You may think when typing `Container<String> container = new Container<>(“Hello!”);` you’re replacing the type `T` with `String`. But this is not the case, at compile-time, the Java compiler removes the generic type parameter `T` from the code and replaces it with the `Object` type:
```java
class Container {
    private Object contents;

    public Container(Object contents) {
        this.contents = contents;
    }

    public Object getContents() {
        return contents;
    }

    public static void main(String[] args) {
        Container container = new Container("Hello!");
        String contents = (String) container.getContents();
        System.out.println(contents);
    }
}
```

Notice how `String contents = container.getContents();` is transformed to `String contents = (String) container.getContents();` As you can see the compiler inserted a cast when invoking `getContents()` method.

## Wildcard generic type

A wildcard, represented by `?` is useful when you want to define a “whatever type.”

### Unbounded wildcard

One of the generics constraints is you cannot, for example, pass `List<Integer>` argument to `List<Object>` parameter; doing so would result in a compile-time error. It is actually a good thing because Java is protecting you from you!

```java
public static void processList(List<Object> list) {
    for (Object o : list)
    System.out.print(o);
}

public static void main(String[] args) {
    List<Integer> numbers = List.of(1, 2);
    processList(numbers); // compile-time error
}
```
This example isn’t functioning because `List<Integer>` isn’t a subtype of `List<Object>`. As you might expect the `List<Object>` type in the `processList` parameter needs to be replaced with `List<?>` (read: *list of unknown type*)

Now I can pass any kind of List to `processList` method:

```java
public static void processList(List<?> list) {
    for (Object o : list)
        System.out.print(o);
}

public static void main(String[] args) {
    List<Integer> list = List.of(1, 2);
    processList(list);// 12

    System.out.println();
    List<String> strings = List.of("Hello", "World!");
    processList(strings);// HelloWorld!
}
```

### bounded wildcard

Using *bounded* *wildcards* allows the compiler to enforce a restriction on the type. This example demonstrates the power and safety benefits of the bounded wildcard:

```java
public static void transformList(List<? extends CharSequence> list) {
    List<Integer> charsCount = list.stream()
            .map(CharSequence::length)
            .collect(Collectors.toList());
    System.out.println(charsCount);
}

public static void main(String[] args) {
    List<String> stringList = List.of("Hello", "World", "!");
    transformList(stringList);// [5, 5, 1]
    List<StringBuilder> stringBuilders = List.of(new StringBuilder("Java"), new StringBuilder("Rocks"));
    transformList(stringBuilders);// [4, 5]
}
```

What `transformList` does is it takes a `List` and transform it into a `List<Integer>`, expressing each element’s length.

Notice the parameter of `transformList` is `List<? extends CharSequence>` which means that this method will accept any list which is upper-bounded to `CharSequence` like `List<String>`, `List<StringBuilder>`, any list that has a subtype of `CharSequence` can be passed as an argument to this method.

## Wrap up

In this post, I attempted to clarify the concept of Type Erasure and introduced you to wildcards and bounds. Still, there is much more to learn from generics. If you’re curious and want to know more, I highly recommend this comprehensive [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html).

If you found this post useful, you know what to do now. Hit that clap button and follow me to get more articles and tutorials on your feed.
