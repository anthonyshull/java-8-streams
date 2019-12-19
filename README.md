Java 8 Streams
==============

Java 8 was released in March 2014.

Many organizations have adopted Java 8 as their officially supported version, and most developers have at least been
witness to its new features. But, many developers have yet to fully embrace the new programming techniques it affords.

This talk hopes to serve as a gentle introduction and general motivator for Java 8 feature adoption.

The goal is to move us from simply recognizing a lambda expression to beginning to think about programming differently.
That is, we want to approach our work with an expanded set of tools.

We will build up to and focus on the Streams API.

For a more thorough treatment on all of the new features in Java 8-11 see [Modern Java in Action](modern).

### Motivation

> Indeed, there is a danger that the Java Virtual Machine (JVM) and its bytecode will be seen as more important than
> the Java language itself and that, for certain applications, Java might be replaced by one of its competing languages 
> such as Scala, Groovy, or Kotlin..."
>
>&mdash; Modern Java in Action

Java will forever be an object-oriented language. But, by adding support for functional programming techniques Java
can hope to stay competitive in certain fast-growing domains, e.g., stream processing.

Learning the Java Streams API is a natural first step toward learning reactive programming with [RxJava](rxjava) and/or Java 9,
streaming frameworks like [Beam](beam) or [Flink](flink), or even a fully-fledged functional programming language such as [Clojure](clojure).

In general, I argue that functional programming leads to programs that are:

#### Shorter
> If I had more time, I would have written a shorter letter.
> 
>&mdash; Blaise Pascal

*Programming isn't typing* and we aren't paid per line of code.

The fewer lines of code we write, the fewer bugs we introduce, and generally the fewer computations we process.

#### Easier to understand and more expressive
> Everything must be made as simple as possible, but not simpler.
>
>&mdash; Albert Einstein

By declaring what our program does rather than how it does it, we focus attention more on the problem at hand.

#### More reusable
> To see what is general in what is particular and what is permanent in what is transitory is the aim of scientific thought.
>
>&mdash; Alfred North Whitehead

Our job is to break problems down into their constituent parts (analysis), and build programs that maximize reuse (composition.)

#### Easier to parallelize and test
> Simplicity is a prerequisite for reliability.
>
>&mdash; Edsger Dijkstra

Pure functions, or side-effect-free functions, do not manipulate state.
Rather than mutating data, pure functions return a new *copy* of said data.

Pure functions over immutable data makes testing trivial as nearly everything becomes a unit test.

Because pure functions are thread-safe concurrency/parallelism is much simplified.
We will see that with the Streams API we get concurrency/parallelism almost for free.

<hr />

**Problem Statement**

We want to process a list of persons with contact information that can be either a single email address or single phone number.
Or, they could have no contact information.

Our goal is to end up with a `Map<Processor.ContactType, List<String>>` with contact information grouped by type: either email or phone.
We will send this information downstream to send bulk emails and bulk text messages.

Needless to say, we shouldn't have any null values or null pointer exceptions.
<hr />

### Optional

The `Optional<T>` class adds meaning to our models.
It allows us to wrap a context around a value.
In our case, contact information could be present or absent.

```java
class Person {
  private Optional<String> contact;
 
  Person() {
    this.contact = Optional.empty();
  }

  Person(String contact) {
    this.contact = Optional.of(contact);
  }
}
```

### Behavior Parameterization

In functional languages, functions are first-class citizens.
That is, functions can be passed into functions as arguments as well as be returned from functions.

In Java 8 this takes the form of behavior parameterization.
We can write methods that change behavior based on methods they are given.

This allows us to build more complex, reusable methods by composing behaviors.
```java
class Processor {
    
    static void process(List<Person> contacts) {
        contacts.stream()
                .map(Person::getContact);
    }

}
```
Here, `map` takes only one function.
But, there is already an important point being made.
#### Map

Map is a method for doing some thing to every thing in a stream.
This is an incredibly powerful abstraction.
Notice how we don't say *how* to transform each element in the stream.
We just say *what* we want to do: get the contact information.

This is an example of behavior parameterization because we could have passed **any** function that operated on Persons.

### Lambdas

Some functions are so simple that we don't bother naming them.
These are called lambdas or anonymous functions.

The above could be re-written to use a lambda expression.
```java
class Processor {
    
    static void process(List<Person> contacts) {
        contacts.stream()
                .map(person -> person.getContact());
    }

}
```
But, the method reference is much more expressive and reusable so we generally prefer them to lambdas.

Lambda expressions can be used when a functional interface is exposed; a functional interface is an interface that 
specifies exactly one abstract method.

#### Filter

Our requirements tasked us with removing any contact information that is missing.
The Streams API allows us to filter items from our stream.

And, as you might have guessed, filter takes advantage of a functional interface `Predicate<T>`.
```java
public interface Stream<T> {
  Stream<T> filter(Predicate<? super T> var1);
}
```
As long as we pass it any function that takes `<T>` and returns a boolean, we can use it to filter our stream.
```java
class Processor {
    
    static void process(List<Person> contacts) {
        contacts.stream()
                .map(Person::getContact)
                .filter(Optional::isPresent); // could be .filter(contact -> contact.isPresent())
    }

}
```
### Streams

A *stream* is a sequence of data items that are conceptually produced one at a time.

The Streams API allows us to abstract away common operations.
We've already seen examples of mapping over elements and filtering elements, but we also have powerful tools for
combining or grouping elements.

#### Collect

Our requirements asked that we group contacts by type: either email or phone.

Take a minute to think about how you would do that iteratively.
With the Streams API:
```java
class Processor {
    
    enum ContactType { EMAIL, PHONE }
    
    static ContactType getContactType(String contact) {
        // implement regex, etc.
    }

    static void process(List<Person> contacts) {
        contacts.stream()
                .map(Person::getContact)
                .filter(Optional::isPresent)
                .map(Optional::get)
                .collect(groupingBy(Processor::getContactType));
    }

}
```

#### Parallel

It's pretty obvious that our processing pipeline is shorter, more expressive, and more reusable than any iterative
implementation we could write.

But, any iterative solution would also be processed linearly.
With the Streams API we can parallelize any portions of our computation that can be made parallel.

```java
class Processor {

    static void process(List<Person> contacts) {
        contacts.stream()
                .parallel()
                .map(Person::getContact)
                .filter(Optional::isPresent)
                .map(Optional::get)
                .collect(groupingBy(Processor::getContactType));
    }

}
```
Parallel will use `Runtime.getRuntime().availableProcessors()` by default.
But, you can control the number of threads used with:
```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12");`
```

[beam]: http://beam
[clojure]: http://clojure
[flink]: http://flink
[modern]: https://www.manning.com/books/modern-java-in-action
[rxjava]: http://rxjava