---
title: "Generics in Java"
date: 2018-07-13T10:52:37+05:30
tags: ["generics"]
categories : ["java"]
---
Have you ever wondered how the HashMap class in Java can any two parameters for key and value. Have you ever wondered how sort method of Collections API can sort a List of Strings, Integers and other primitive data types. Well look no further, the answer is Generics.
 So what is Generics in Java? If you look at the Collections API you can find a lot of code written in the format given below. Lets take the example of method signature of Collections.sort method, you will see something like this
 ```java
 public static <T extends Comparable<? super T>> void sort(List<T> list)
 ```
Woah! What the heck is that thing! T extends blah blah blah. That is a Typed parameter which is an integral part of Generics. In layman terms, generics is nothing but a way to implement type safety in Java.
For example, before generics, the List was implemented as follows.
```java
import java.util.*;
//This is required to supress the warnings
@SuppressWarnings("unchecked")
public class program{
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add(1);
        list.add("String");
        list.add(3);
        int temp  = 0;
        Iterator it = list.iterator();
        while(it.hasNext()){
            temp += (int)it.next(); //Runtime error because string cannot be converted into int
        }
        System.out.println(temp);
    }
}
```
The problem with this code was it can compile fine, but if we do some operation like addition of list items, it will throw runtime error. We can fix this by adding a type parameter to the List.
```java
List<Integer> list = new ArrayList<>();
```
This is basically saying that the list will contain only integers. In a way what we have done is we have introduced a type safety which can help us solving the issues in code and catch bugs at compile time rather than runtime. Now if you try to compile the code it will throw compile error. Another benefit that we can see from generics is that we can avoid unnecessary type-casting. So instead of <code class = "inline_code">temp += (int)it.next();</code> we can write <code class = "inline_code">temp += it.next();</code> .

There are different ways of implementing generics in Java.
## Generics on Class
Lets take an example and see how generics are implements on Class level. There are three types by which generics are implemented on Classes.
### Implementing a Generic Type
In this case we just implement class which has a generic Type parameter. We use this a lot for example when we try to initialize a List of Strings.
```java
List<String> list = new ArrayList<>();
```
Here we are saying that the List will only contain strings. We don't need to specify that in <code class = "inline_code">ArrayList&lt;String&gt; </code> because the diamond <code class = "inline_code">&lt;&gt;</code> operator is implicit in taking the Typed parameter from the left hand side of assignment operator.
### Passing a parameter
In order to understand this section, lets take an example of two classes, <code class = "inline_code">Employee</code> and <code class = "inline_code">Point</code> .
```java
class Employee{
    String name;
    int age;
    double salary;
    public Employee(String name, int age, double salary){
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}
class Point{
    int x;
    int y;
    int z;
    public Point(int x, int y, int z){
        this.x = x;
        this.y = y;
        this.z = z;
    }
}
```
Let's say we want to sort a list of Employees by their age in increasing order. One way to implement this is by writing a 
<code class = "inline_code">Comparator</code>. The code looks like this.
```java
public class AgeComparator<Employee> implements Comparator<Employee>{
    @Override
    public int compare(Employee e1, Employee e2){
        return Integer.compare(e1.age, e2.age);
    }
}
```
Similarly we can implement comparator for each of the members of both classes. Now what if we want to get list of Employees by their age in decreasing order. What we can do is we can write a <code class = "inline_code">ReverseAgeComparator&lt;Employee&gt;</code> and then modify the compare method (multiply by -1, or swap the order) and get the desired result. But we have to unnecessarily implement <code class = "inline_code">ReverseComparator</code> for all the members of class. A better way to use generics is as follows.
```java
public class ReverseComparator<T> implements Comparator<T>{
    private final Comparator<T> delegateComparator;
    public ReverseComparator(Comparator<T> comparator){
        this.delegateComparator = comparator;
    }
    @Override
    public int compare(T t1, T t2){
        return -1*delegateComparator.compare(t1, t2);
    }
}
```
Here we are passing the Type parameter T from ReverseComparator to Comparator. What it means that ReverseComparator of a particular type must implement a Comparator of the same type. For example, <code class = "inline_code">ReverseAgeComparator&lt;Point&gt;</code> must implement <code class = "inline_code">Comparator&lt;Point&gt;</code> .
Now if we want to sort the Employees in any order according to age we can achieve so by using the following code.
```java
List<Employee> employees = new ArrayList<>();

//increasing order
AgeComparator<Employee> ageComparator = new AgeComparator<>();
Collections.sort(employees, ageComparator);

//decreasing order
ReverseComparator<Employee> reverseAgeComparator = new ReverseComparator<>(ageComparator);
Collections.sort(employees, reverseAgeComparator);
```

This <code class = "inline_code">ReverseComparator</code> will work for classes of any type as long as they implement the <code class = "inline_code">Comparator</code> interface.
### Type Bounds
Type Bounds add constraints to the the type parameter that we are using. Lets say we want to restrict the type parameter T to implement certain subclass or superclass of a class or an interface we can do so by using Type bounds. Lets take an example of a class <code class = "inline_code">SortedPair</code> which takes two parameters (first and second) and return the sorted pair where <code class = "inline_code">first &lt;= second</code>. The skeleton code look like this.
```java
public class SortedPair{
    Object first;
    Object second;
    public SortedPair(Object first, Object second){
        if(compare(first, second) <= 0){
            this.first = first;
            this.second = second;
        }else{
            this.first = second;
            this.second = first;
        }
    }
}
```
The problem with this code is again type safety. We can pass an <code class = "inline_code">Integer</code> and a <code class = "inline_code">String</code> as first and second parameter and the code will compile fine but will throw runtime error. Lets try to implement generics to ensure type safety. Since the typed parameter should be comparable, it should implement Comparable interface.
We can specify this by using <code class = "inline_code">SortedPair&lt;T extends Comparable&gt;</code>. This is an example of type bound.
```java
public class SortedPair<T extends Comparable>{
    T first;
    T second;
    public SortedPair(T first, T second){
        if(first.compareTo(second) <= 0){
            this.first = first;
            this.second = second;
        }else{
            this.first = second;
            this.second = first;
        }
    }
}
```
Still there is some problem with code. We can pass an <code class = "inline_code">Object</code> in place of second in the constructor.
```java
public static class SortedPair<T extends Comparable>{
    T first;
    T second;
    public SortedPair(T first, T second){
        if(first.compareTo(new Object()) <= 0){
            this.first = first;
            this.second = second;
        }else{
            this.first = second;
            this.second = first;
        }
    }
}

//main code
SortedPair<Integer> sp = new SortedPair<>(1, 4);
System.out.println(sp.first);
System.out.println(sp.second);
```
If we try to run the above code, it will compile fine but will throw runtime error. Here we are not checking if T is implementing the <code class = "inline_code">Comparable</code> of its own type. To fix this we need to replace <code class = "inline_code">SortedPair&lt;T extends Comparable&gt;</code> with <code class = "inline_code">SortedPair&lt;T extends Comparable&lt;T&gt;&gt;</code>. The final program looks like this.
```java
public static class SortedPair<T extends Comparable<T>>{
    T first;
    T second;
    public SortedPair(T first, T second){
        if(first.compareTo(second) <= 0){
            this.first = first;
            this.second = second;
        }else{
            this.first = second;
            this.second = first;
        }
    }
}

//main code
SortedPair<Integer> sp = new SortedPair<>(1, 4);
System.out.println(sp.first); // 1
System.out.println(sp.second); // 4

SortedPair<Integer> sp2 = new SortedPair<>(4, 1);
System.out.println(sp.first); // 1
System.out.println(sp.second); // 4
```
Thus we can see that using type bounds correctly will help in preventing bugs at compile time rather than debugging them at runtime.
## Generics on Methods
We have seen how to implement generics on classes. Now we will look how to implement generics on methods. We will try to implement a generic function max which will take a List of Object and return the maximum element among them. The method signature looks like this.
```java
Object max(List objects)
```
Now lets try to make it generic. Now you may be wondering that you can write like this.
```java
public static T max(List<T> objects){
    if(objects.size() == 0){
        throw new IllegalStateException("Cannot return max on empty list");
    }
    T res = objects.get(0);
    for(int i = 1; i < objects.size(); i++){
        if(res.compareTo(objects.get(i)) < 0){
            res = objects.get(i);
        }
    }
    return res;
}
```
### Problems in above code:

1. 1. &nbsp;&nbsp;&nbsp;&nbsp;This code will throw compile error because we can define the type parameter on Class but on method the correct way of defining the type parameter is <code class = "inline_code">public static &lt;T&gt; T max(List&lt;T&gt; objects);</code>
2. 2.&nbsp;&nbsp;&nbsp;&nbsp;The second problem is that we are not sure that the Type parameter that we are passing is comparable or not, i.e, it has implemented the Comparable interface or not. You can ensure that by using <code class = "inline_code">List&lt;T extends Comparable&lt;? extends T&gt;&gt; objects</code> as method parameter.
3. 3. &nbsp;&nbsp;&nbsp;&nbsp;If we want to implement comparision on different property of object, we cannot do so in this code.

So after fixing the errors, the above code looks like this
```java
public static <T> T max(List<T> objects, Comparator<T> comparator){
    if(objects.size() == 0){
        throw new IllegalStateException("Cannot return max on empty list");
    }
    T res = objects.get(0);
    for(int i = 1; i < objects.size(); i++){
        if(comparator.compare(res, objects.get(i)) < 0){
            res = objects.get(i);
        }
    }
    return res;
}
```

For your reference you can checkout the [method signature](https://docs.oracle.com/javase/7/docs/api/java/util/Collections.html#max(java.util.Collection)) of Collections.max() API.

This was a quick introduction to Generics in Java. Later we will look into how generics work under the hoods and what is Erasure in Java. Till then you can checkout the [Generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html) and [Generics Extra](https://docs.oracle.com/javase/tutorial/extra/generics/index.html) pages from Oracle on Generics. It has good explanation on Generics with examples.

Next we are going to look at [Wilcards in Generics]( {{< relref "blog/wildcards-in-generics" >}}).
