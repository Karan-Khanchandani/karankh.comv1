---
title: "Wildcards in Generics"
date: 2018-07-15T10:52:37+05:30
tags: ["generics", "wilcards"]
categories : ["java"]
---
In Generics, the <code class = "inline_code">?</code> called the wildcard, represents an Unknown type. We can see this wildcard being used in a various places like the type of parameter, field or local varidble, and sometimes as return type. The wildcard should never be used as a type argument for a generic method invocation, a generic class instance creation or a supertype.

A wildcard provides flexibility or restriction to the type parameter <code class = "inline_code">&lt;T&gt;</code> in terms of what classes can be used in place of <code class = "inline_code">&lt;T&gt;</code>. There are different types of wildcards.

## Upper Bounded Wildcard
You can use an upper bounded wildcard to relax the restrictions on a variable. For example, say you want to write a method that works on <code class = "inline_code">List&lt;Integer&gt;</code>, <code class = "inline_code">List&lt;Double&gt;</code>, and <code class = "inline_code">List&lt;Number&gt;</code> ; you can achieve this by using an upper bounded wildcard.

To declare an upper-bounded wildcard, use the wildcard character <code class = "inline_code">?</code>, followed by the <code class = "inline_code">extends</code> keyword, followed by its upper bound. Note that, in this context, extends is used in a general sense to mean either "extends" (as in classes) or "implements" (as in interfaces).

To write the method that works on lists of <code class = "inline_code">Number</code> and the subtypes of <code class = "inline_code">Number</code>, such as <code class = "inline_code">Integer</code>, <code class = "inline_code">Double</code>, and <code class = "inline_code">Float</code>, you would specify <code class = "inline_code">List&lt;? extends Number&gt;</code>. The term <code class = "inline_code">List&lt;Number&gt;</code> is more restrictive than <code class = "inline_code">List&lt;? extends Number&gt;</code> because the former matches a list of type <code class = "inline_code">Number</code> only, whereas the latter matches a list of type <code class = "inline_code">Number</code> or ***any of its subclasses***.

The method for the above case will look like this
```java
public static double sumOfList(List<? extends Number> list) {
    double s = 0.0;
    for (Number n : list)
        s += n.doubleValue();
    return s;
}
```
## Lower Bounded Wildcard
As we saw above, an upper bounded wildcard restricts the unknown type to be a specific type or a subtype of that type and is represented using the extends keyword. In a similar way, a lower bounded wildcard restricts the unknown type to be a specific type or a super type of that type.

A lower bounded wildcard is expressed using the wildcard character <code class = "inline_code">?</code>, following by the <code class = "inline_code">super</code> keyword, followed by its lower bound.

Say you want to write a method that puts Integer objects into a list. To maximize flexibility, you would like the method to work on <code class = "inline_code">List&lt;Integer&gt;</code>, <code class = "inline_code">List&lt;Number&gt;</code>, and <code class = "inline_code">List&lt;Object&gt;</code> , anything that can hold <code class = "inline_code">Integer</code> values.

To write the method that works on lists of <code class = "inline_code">Integer</code> and the supertypes of <code class = "inline_code">Integer</code>, such as <code class = "inline_code">Integer</code>, <code class = "inline_code">Number</code>, and <code class = "inline_code">Object</code>, you would specify <code class = "inline_code">List&lt;? super Integer&gt;</code>. The term <code class = "inline_code">List&lt;Integer&gt;</code> is more restrictive than <code class = "inline_code">List&lt;? super Integer&gt;</code> because the former matches a list of type <code class = "inline_code">Integer</code> only, whereas the latter matches a list of any type that is a supertype of <code class = "inline_code">Integer</code>.

A method that uses a lower bound may look like this.
```java
public static void addNumbers(List<? super Integer> list) {
    for (int i = 1; i <= 10; i++) {
        list.add(i);
    }
}
```
<span class = "imp_note"><i class="fas fa-exclamation-triangle"></i> Note: You can specify an upper bound for a wildcard, or you can specify a lower bound, but you cannot specify both.</span>

## Unbounded Wildcard
The unbounded wildcard type is specified using the wildcard character <code class = "inline_code">?</code>, for example, <code class = "inline_code">List&lt;?&gt;</code>. This is called a list of unknown type. There are two scenarios where an unbounded wildcard is a useful approach:

If you are writing a method that can be implemented using functionality provided in the <code class = "inline_code">Object</code> class.

When the code is using methods in the generic class that don't depend on the type parameter. For example, List.size or List.clear. In fact, <code class = "inline_code">Class&lt;?&gt;</code> is so often used because most of the methods in <code class = "inline_code">Class&lt;T&gt;</code> do not depend on T.

Consider the following method, printList:
```java
public static void printList(List<Object> list) {
    for (Object elem : list)
        System.out.println(elem + " ");
    System.out.println();
}
```

The goal of printList is to print a list of any type, but it fails to achieve that goal — it prints only a list of <code class = "inline_code">Object</code> instances; it cannot print <code class = "inline_code">List&lt;Integer&gt;</code>, <code class = "inline_code">List&lt;String&gt;</code>, <code class = "inline_code">List&lt;Double&gt;</code>, and so on, because they are not subtypes of <code class = "inline_code">List&lt;Object&gt;</code>. To write a generic printList method, use <code class = "inline_code">List&lt;?&gt;</code>:
```java
public static void printList(List<?> list) {
    for (Object elem: list)
        System.out.print(elem + " ");
    System.out.println();
}
```
Because for any concrete type A, <code class = "inline_code">List&lt;A&gt;</code> is a subtype of <code class = "inline_code">List&lt;?&gt;</code>, you can use printList to print a list of any type:
```java
List<Integer> li = Arrays.asList(1, 2, 3);
List<String>  ls = Arrays.asList("one", "two", "three");
printList(li);
printList(ls);
```
It's important to note that <code class = "inline_code">List&lt;Object&gt;</code> and <code class = "inline_code">List&lt;?&gt;</code> are not the same. You can insert an <code class = "inline_code">Object</code>, or any subtype of <code class = "inline_code">Object</code>, into a <code class = "inline_code">List&lt;Object&gt;</code>. But you can only insert <code class = "inline_code">null</code> into a <code class = "inline_code">List&lt;?&gt;</code>. 

## Guildelines for using Wildcard

One of the more confusing aspects when learning to program with generics is determining when to use an upper bounded wildcard and when to use a lower bounded wildcard. This page provides some guidelines to follow when designing your code.

For purposes of this discussion, it is helpful to think of variables as providing one of two functions:

1. 1. &nbsp;&nbsp;&nbsp;&nbsp;An "In" Variable
An "in" variable serves up data to the code. Imagine a copy method with two arguments: <code class = "inline_code">copy(src, dest)</code>. The src argument provides the data to be copied, so it is the "in" parameter.
2. 2. &nbsp;&nbsp;&nbsp;&nbsp;An "Out" Variable
An "out" variable holds data for use elsewhere. In the copy example, <code class = "inline_code">copy(src, dest)</code>, the dest argument accepts data, so it is the "out" parameter.

### Rules for using wildcard
1. 1. &nbsp;&nbsp;&nbsp;&nbsp;An "in" variable is defined with an upper bounded wildcard, using the extends keyword.
2. 2. &nbsp;&nbsp;&nbsp;&nbsp;An "out" variable is defined with a lower bounded wildcard, using the super keyword.
3. 3. &nbsp;&nbsp;&nbsp;&nbsp;In the case where the "in" variable can be accessed using methods defined in the Object class, use an unbounded wildcard.
4. 4. &nbsp;&nbsp;&nbsp;&nbsp;In the case where the code needs to access the variable as both an "in" and an "out" variable, do not use a wildcard.

These guidelines do not apply to a method's return type. Using a wildcard as a return type should be avoided because it forces programmers using the code to deal with wildcards.

A list defined by <code class = "inline_code">List&lt;? extends ...&gt;</code> can be informally thought of as read-only, but that is not a strict guarantee. Suppose you have the following two classes:

```java
class NaturalNumber {

    private int i;

    public NaturalNumber(int i) { this.i = i; }
    // ...
}

class EvenNumber extends NaturalNumber {

    public EvenNumber(int i) { super(i); }
    // ...
}
```

Consider the following code:
```java
List<EvenNumber> le = new ArrayList<>();
List<? extends NaturalNumber> ln = le;
ln.add(new NaturalNumber(35));  // compile-time error
```

Because <code class = "inline_code">List&lt;EvenNumber&gt;</code> is a subtype of <code class = "inline_code">List&lt;? extends NaturalNumber&gt;</code>, you can assign le to ln. But you cannot use ln to add a natural number to a list of even numbers. The following operations on the list are possible:

* * &nbsp;&nbsp;&nbsp;&nbsp;You can add <code class = "inline_code">null</code>.
* * &nbsp;&nbsp;&nbsp;&nbsp;You can invoke clear.
* * &nbsp;&nbsp;&nbsp;&nbsp;You can get the iterator and invoke remove.
* * &nbsp;&nbsp;&nbsp;&nbsp;You can capture the wildcard and write elements that you've read from the list.
* * &nbsp;&nbsp;&nbsp;&nbsp;You can see that the list defined by <code class = "inline_code">List&lt;? extends NaturalNumber&gt;</code> is not read-only in the strictest sense of the word, but you might think of it that way because you cannot store a new element or change an existing element in the list.

I hope you can now use wildcards with ease in your code. I have used the examples shown in [Wildcards Oracle Documentation](https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html) and [Fun with Wildcards] (https://docs.oracle.com/javase/tutorial/extra/generics/morefun.html) links. You can checkout the above links for more description on the topic.