# What’s New in Java 8
> Introduction to Lambda Expression in Java 8

## Agenda
* Java 8 lambda expressions and Interfaces
* Stream API and Collectors
* Date and Time API
* Strings, I/O and other bits and pieces

## Targeted Audience
* Basic knowledge of the main APIs
* Generics
* Collection API

## Introduction to the **Lambda expressions**
* The lambda syntax
* Functional interfaces
* Method references
* Constructor references
* How to process data from the Collection API?

### What Is a Lambda Expression for?
* A simple example
```java
public interface FileFilter {
    boolean accept(File file);
}
```

* Let’s implement this interface
```java
FileFilter filter = new FileFilter() {
    @Override
    public boolean accept(File pathname) {
        return pathname.getName().endsWith(".java");
    }
};
```

* Use it
```java
JavaFileFilter fileFilter = new JavaFileFilter(); 
File dir = new File("d:/tmp");
File[] javaFiles = dir.listFiles(fileFilter);
```

* Let’s use an anonymous class
```java
FileFilter fileFilter = new FileFilter() {
    @Override
    public boolean accept(File file) {
        return file.getName().endsWith(".java");
    }
}
File dir = new File("d:/tmp");
File[] javaFiles = dir.listFiles(fileFilter);
```

* The first answer is :
> **To make instances of anonymous classes easier to write and read**

* This is a Java 8 lambda expression:
```java
FileFilter filter = (File file) ‐> file.getName().endsWith(".java");
```

## Streams & Collectors
> New APIs for map / filter / reduce

### Streams & Collectors Outline
* Introduction: map / filter / reduce
* What is a « Stream »?
* Patterns to build a Stream
* Operations on a Stream

### What Is a Stream?
* Technical answer: a typed interface
```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    //...
}
```

* What does Stream do?
> It gives ways to efficiently process large amounts of data... and also smaller ones

<!--
What does efficiently mean?
- Two things:
- In parallel, to leverage the computing power of multicore CPUs
- Pipelined, to avoid unnecessary intermediary computations
-->

* So what is a Stream?
  * An object on which one can define operations
  * An object that does not hold any data
  * An object that should not change the data it processes
  * An object able to process data in « one pass »
  * An object optimized from the algorithm point of view, and able to process data in parallel

* Why can’t a Collection be a Stream?
> Because Stream is a new concept, and we dont want to change the way the Collection API works


* How Can We Build a Stream?
```java
List<Person> people = new ArrayList<>() ;
Stream<Person> stream = people.stream();
```

### Map / Filter / Reduce
* Let’s take a list a Person
```java
List<Person> people = new ArrayList<>() ;
```
> Suppose we want to compute the « average of the age of the people older than 20 »

* 1st step: mapping
<!--
- The mapping step takes a List<Person> and returns a List<Integer>
- The size of both lists is the same
-->

* 2nd step: filtering
<!--
- The filtering step takes a List<Integer> and returns a List<Integer> - But there some elements have been filtered out in the process
-->

* 3rd step: reduce
<!--
- This is the reduction step, equivalent to the SQL aggregation
-->

### Consumming
* Operation: forEach()
  * Prints all the elements of the list
  * It takes an instance of Consumer as an argument
```java
stream.forEach(p ‐> System.out.println(p));
```
* Interface Consumer<T>
  * Consumer<T> is a functional interface
  * Can be implemented by a lambda expression
```java
Consumer<T> c = p ‐> System.out.println(p);
```
```java
@FunctionalInterface
public interface Consumer<T> { 
    void accept(T t);
}
```
* Method reference
```java
Consumer<T> c = System.out::println;
```

* In fact Consumer<T> is a bit more complex
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    //One can chain consumers!
    default Consumer<T> andThen(Consumer<? super T> after) {            
        Objects.requireNonNull(after);
        return (T t) ‐> { accept(t); after.accept(t); 
    };
}
```
* Let’s chain consumers
```java
List<String> list = new ArrayList<>();
A First Operation
Consumer<String> c1 = list::add; 
Consumer<String> c2 = System.out::println;
Consumer<String> c3 = c1.andThen(c2);
```

### Filtering Operation
* Example
```java
List<Person> list = ...; 
Stream<Person> stream = list.stream(); 
Stream<Person> filtered = stream.filter(person ‐> person.getAge() > 20);
```
* Takes a predicate as a parameter:
```java
Predicate<Person> p = person ‐> person.getAge() > 20;

// Predicate interface
@FunctionalInterface
A Second Operation: Filter
public interface Predicate<T> { 
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) { ... }
    default Predicate<T> or(Predicate<? super T> other) { ... }
    default Predicate<T> negate() { ... }
}
```

<!-- a stream does not hold any data -->
* Question: what do we have in *filtered* stream
  * ~~The filtered data~~
  * nothing, since a Stream does not hold any data
> The call to the filter method is ==lazy==. And all the methods of Stream that return another Stream are lazy
<!--Another way of saying it:
an operation on a Stream that returns a Stream is called an intermediary operation-->

* What does this code do?
```java
List<String> result = new ArrayList<>();
List<Person> people = ... ;
people.stream().peek(System.out::println).filter(person ‐> person.getAge() > 20).peek(result::add);

// Answer: nothing!
// This code does not print anything 
// The list « result » is empty
```

* We saw the Stream API defines operations
  * forEach(Consumer) (not lazy)
  * peek(Consumer) (lazy)
  * filter(Predicate) (lazy)

### Mapping Operation
* Example:
```java
List<Person> list = ... ; 
Stream<Person> stream = list.stream(); 
Stream<String> names = stream.map(person ‐> person.getName());
```

* map() returns a Stream, so it is an intermediary operation

```java
@FunctionalInterface
public interface Function<T, R> { 
    R apply(T t);
}
```

### Flatmapping Operation
* signature
```java
//The flatMapper takes an element of type T, and returns an element of type Stream<R>
<R> Stream<R> flatMap(Function<T, Stream<R>> flatMapper);
//If the flatMap was a regular map, it would return a Stream<Stream<R>>
<R> Stream<R> map(Function<T, R> mapper);
//Thus the « stream of streams » is flattened, and becomes a stream
```

* Summary Mapping
  * forEach() and peek()
  * filter()
  * map() and flatMap()


### Reducing Operation

* And what about the reduction step?
  * Two kinds of reduction in the Stream API 
  * 1st: aggregation = min, max, sum, etc...

* How does it work?
```java
List<Integer> ages = ... ; 
Stream<Integer> stream = ages.stream(); 
Integer sum = stream.reduce(0, (age1, age2) ‐> age1 + age2);

// 1st argument: identity element of the reduction operation
// 2nd argument: reduction operation, of type BinaryOperator<T>
```

* BinaryOperator

```java
// A BinaryOperator is a special case of BiFunction
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}

@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    // T apply(T t1, T t2);
}
```
### Aggregations
```java 
BinaryOperation<Integer> sum = (i1, i2) ‐> i1 + i2;
Stream<Integer> stream = Stream.of(1, 2, 3, 4);
int red = stream.reduce(id, sum); 
System.out.println(red); //10
```

### Optionals
* Optional means « there might be no result »
```java
List<Integer> ages = ... ; 
Stream<Integer> stream = ages.stream(); 
Optional<Integer> max = stream.max(Comparator.naturalOrder());
```

* How to use an Optional?
```java
Optional<String> opt = ... ;
if (opt.isPresent()) {
    String s = opt.get() ;
} else {
    ///... 
}
```

* The method orElse() encapsulates both calls
```java
String s = opt.orElse("") ;
```

### Reductions
* Available reductions: 
  * max(), min()
  * count()
* Boolean reductions
  * allMatch(), noneMatch(), anyMatch()
* Reductions that return an optional 
  * findFirst(), findAny()

* Reductions are terminal operations
* They trigger the processing of the data
* Example
```java
List<Person> people = ...;
Terminal Operation
Optional<Integer> minAge = 
    people.map(person ‐> person.getAge()) // Stream<Integer>
        .filter(age ‐> age > 20) // Stream<Integer> 
        .min(Comparator.naturalOrder()); // terminal operation
```
### Collectors
* There is another type of reduction
* Called « mutable » reduction
* Instead of aggregating elements, this reduction put them in a « container »
* Example: Collecting in a String
```java
List<Person> people = ... ;
String result = people.stream().filter(person ‐> person.getAge() > 20) .map(Person::getLastName).collect(Collectors.joining(", ") );
//Result is a String with all the names of the people in people, older than 20, separated by a comma

// Another example is: Collectors.toList() returns a list
```

* Collecting in a Map
```java
List<Person> people = ... ;
Map<Integer, List<Person>> result = people.stream().filter(person ‐> person.getAge() > 20) .collect(Collectors.groupingBy(Person::getAge) );
```
* Result is a Map containing the people of people, older than 20
  * The keys are the ages of the people
  * The values are the lists of the people of that age

* So What Is a Stream?
  * An object that allows one to define processings on data
  * There is no limit on the amount of data that can be processed
  * Those processings are typically map / filter / reduce operations
  * Those processings are optimized : 
    * First, we define all the operations 
    * Then, the operations are triggered

### Summary
* Quick explanation of the map / filter / reduce
* The difference between intermediary and final operations 
* The « Consuming » operations: forEach() and peek()
* The « mapping » operations: map() and flatMap()
* The « filter » operation: filter()
* The « reduction » operations:
  * Aggregations: reduce(), max(), min(), ...
  * Mutable reductions: collect, Collectors


