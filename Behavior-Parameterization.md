# Behavior Parameterization

![](http://geeknizer.com/wp-content/uploads/2013/07/lambda-java8.jpg)

## Overview

Behavior parameterization is a technique to improve code modularity by allowing the caller to pass custom **behavior** to a method as a **parameter**. Hence the name: behavior parameterization.

In this article, I will show you how it is possible to improve the design using **behavior parameterization** first by using *Functional Interfaces*, then further improve the example using Java 8 lambda expressions.

### The Problem: Implementing Every Possible Behavior is Verbose and Impossible

Suppose we have a class called `AnimalContainer` which contains many types of animals, and this class has a simple method called `filter()` returns all of the animals that are of type `DOG`.

```java
public class AnimalContainer {
    private List<Animal> animals;

    public List<Animal> filter() {
        List<Animal> results = new ArrayList<>();
        for(Animal animal : animals) {
            if(animal.getType() == DOG) // Let's assume that type is an enum
                results.add(animal);
        }
        return results;
    }
}
```

Now, let's suppose the caller of this function would also like to filter out dogs that were below a certain weight; We'd have to write another method with like this:

```java
public List<Animal> filter(int weight) {
    List<Animal> results = new ArrayList<>();
    for(Animal animal : animals) {
        if(animal.getType() == DOG && animal.getWeight < weight )
            results.add(animal);
    }
    return results;
}
```

Now, suppose you wanted to filter if the weight was `>=` than the weight, not less than? And then, later, suppose you needed filter out dogs of a certain weight *AND* a certain color? Oh, and let's filter cats, and then let's also support a handful of additional cat filtering options. Pretty soon, we have pages and pages full of filter methods to test and maintain; do we feel happy? The answer is no.

Since we've seen that we can't possibly think up all the scenarios that a user would need to filter animals, wouldn't it be great if we could just write a single `filter` method, and then let THEM provide the behavior via a parameter? Kind of like, **behavior parameterization**?

## Enter Functional Interfaces as Behavior Parameters

A functional interface is very simple: it's just an interface with one non-default, non-static method. Seems kind of weird right? Like a waste of typing to create an interface with just one method. But it'll soon be clear why this structure is important when we get to the section of lambda expressions.

You may not realize it, but you've already been using functional interfaces. Some examples are: `Runnable` and `Callable`, each containing just one method. Java 8 introduced some very important functional interfaces: `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>` to name just a few.

In order to look for a better definition of our `filter()` method, we're going to focus on the `Predicate<T>` interface. This interface has a single non-default, non-static method defined as:

```java
boolean test(T t);
```

Very simply put, this method says: Given a value of type `T`, check that this value matches some form of a predicate (or filter test), and respond with `true` or `false`. Well, this sounds exactly like what we'd want in our `filter()` method. Given some predicate that the **user** defines (emphasis on user-defined), return all objects that match this predicate. Now, if properly implemented, the user can give us their predicate requirements and we will simply pass out all animals that match it. **Wonderful**!. Let's take a stab at our new method definition:

```java
public List<Animal> filter(Predicate<Animal> pred) {
    List<Animal> results = new ArrayList<>();
    for(Animal animal : animals) {
        if(pred.test(animal))
            results.add(animal);
    }
    return results;
}
```

Wow! Talk about powerful! Now, the user can pass in any type of filtering logic they wish, and our simple, generic and non-complex `filter()` method can accommodate any of it. This is exactly the type of power that behavior parameterization allows. So, what would the use of this filter look like?

```java
AnimalContainer container = ...
List<Animal> filteredAnimals = container.filter(new Predicate<Animal>() {
    @Override
    boolean test(Animal animal) {
        if(animal.getType() == DOG && animal.getWeight >= 100 && animal.getColor() == BROWN)
            return true;
        else
            return false;
    }
});
```

This is how the consumer can create custom predicates on the fly, and in any combination. And, they could also create reusable filters by assigning predicates to variables:

```java
Predicate<Animal> lightCatsFilter = new Predicate<Animal>() {
    @Override
    boolean test(Animal animal) {
        if(animal.getType() == CAT && animal.getWeight < 20)
            return true;
        else
            return false;
    }
});

Predicate<Animal> heavyBrownDogsFilter = new Predicate<Animal>() {
    @Override
    boolean test(Animal animal) {
        if(animal.getType() == DOG && animal.getWeight >= 100 && animal.getColor() == BROWN)
            return true;
        else
            return false;
    }
});

List<Animal> lightCats = container.filter(lightCatsFilter);
```

Now the consumer can create powerful, reusable and composable filters. You might ask: How can filters compose? (Okay, you probably didn't ask that, but I'm going to explain it anyway). In the `Predicate` definition there are a couple default methods called `and` and `or`. This allows one to compose predicates in any combination that they'd like. For example:

```java
Predicate<Animal> heavyDogsFilter = ...
Predicate<Animal> brownDogsFilter = ...
Predicate<Animal> heavyAndBrownDogsFilter = heavyDogsFilter.and(brownDogsFilter);

List<Animal> heavyAndBrownDogs = container.filter(heavyAndBrownDogsFilter);
```

And voila! You can create any powerful combination of filters you'd like.

## Lambda Expressions

<img src="https://keefcode.files.wordpress.com/2013/12/lambda.png" width="340" height="180"/>

We've seen the power of behavior parameterization by deferring the definitions and combinations of business logic to the consumer; however the use of `Predicate<T>` anonymous classes are verbose and clunky. Such has been the curse of Java for years until the advent of Java 8 and it's introduction of lambda expressions.

Lambda expressions allow us to replace the verbose anonymous class definitions with a succinct expression. Let's see our example from above using a lambda expression:

```java
AnimalContainer container = ...
List<Animal> filteredAnimals = container.filter(
    animal -> {
        if(animal.getType() == DOG && animal.getWeight >= 100 && animal.getColor() == BROWN)
            return true;
        else
            return false;
    }
);
```

Umm...wow! That is much more readable.

## Lambda Expressions and Functional Interfaces

One downside of lambda expressions is not all libraries accept them. However, Java will **automagically convert** a lambda expression to generate an anonymous class definition for a functional interface. And, this leads me to my next point: for this conversion to happen, it's imperative that a functional interface only have one method.

Suppose you are the compiler for a moment, and you're given a lambda expression which takes a `T` and returns a `boolean`. Use that lambda expression to implement a functional interface which contains the following methods:

```java
public boolean passes(T t);
public boolean notPasses(T t);
```

Can't decide what to do? Neither can the Java compiler. There is no indication which method the lambda expression was meant to replace. This is why they must be used with functional interfaces which define **only one** non-default, non-static method.

The generic form of a lambda expression for a `Predicate<T>` is: `(T) -> boolean`. Another example that takes a parameter and returns a user-defined type is `Function<T,R>`, or: `(T) -> R`.

### The Correlation Between Behavior Parameters and Type Parameters

Type parameterization (generics) is, in essence, a place holder for types in a class/method/field. The writer is saying that they don't want to define the type here, and allow the consumer to define the type later, vastly improving flexibility as the consumer can use whatever type they like (or, at least within the bound of the type definition):

```java
public <T> T doWork(T obj);
```

Here, `doWork()` can take any user-defined type, because the type has no bearing on the behavior of the method. A good example of this is the `List<T>` type. A list's behavior works the same no matter what type it contains. So it doesn't make sense for the writer of List to hard-code what the list can contain. Let the consumer of `List` define that later.

As we've shown above, behavior parameterization does **exactly** the same thing, but with behavior. The writer is saying that they don't want to define the behavior here, and allow the conumser to define the behavior later, vastly improving flexibility as the consumer can use whatever behavior they like.

## Conclusion

Behavior parameterization improves the modularity of our code by allowing us to write more generic methods and give the consumer the power to provide behavior that we didn't even think of yet. It's benefits are parallel to type parameterization.

I would encourage you to use Behavior Parameterization in your code as it can lead to clean, powerful, reusable, composable code. And in the end, isn't this a goal that all of us "pragmatic" programmers aspire to?

Further reading: [Strategy Pattern](http://c2.com/cgi/wiki?StrategyPattern)
