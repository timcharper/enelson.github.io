# Behavior Parameterization

## What it is

### Concept similar to type parameterization

Behavior parameterization is far less scary than it sounds. It espouses the same concepts that type parameterization does. Type parameterization (generics) is, in essence, a place holder for types in a class/method/field. The writer is saying that they don't want to define the type here, and allow the consumer to define the type later. This is the case when the class/method/field behavior doesn't rely on the type to define its behavior. So flexibility is given to the consumer to use whatever type they like, or at least within the bound of the type definition:

```
public <T> T doWork(T obj);
```

Here, `doWork()` can take any user-defined type, because the type has no bearing on the behavior of the method. A good example of this is the `List<T>` type. A list's behavior works the same no matter what type it contains. So it doesn't make sense for the writer of List to hard-code what the list can contain. Let the consumer of `List` define that later.

### Correlation

To broach the subject of behavior parameterization, let's start with a simple example. Lets say we have a class called `AnimalContainer` which contains many types of animals. We may want to have a simple method called `filter()` that allows us to filter out certain types of animals.

```
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

The filter method is great, except for the fact that it is completely inflexible.  What if you wanted to filter out dogs that were below a certain weight? Write another method with like this:

```
public List<Animal> filter(int weight) {
    List<Animal> results = new ArrayList<>();
    for(Animal animal : animals) {
        if(animal.getType() == DOG && animal.getWeight < weight )
            results.add(animal);
    }
    return results;
}
```

But what if someone wanted to filter if the weight was `>=` than the weight, not less than? And then later, what if someone wanted to filter out dogs of a certain weight *AND* a certain color? You see where this is going. The combinations can become unwieldy and you can't resort to writing a new method each time you need more options.

Plus, what if we want to filter out animals of type 'Cat'? What will we do? Write a new method called `filterCats`? And then write more methods for cat filters with many filtering options?

What would be nice is to take an approach similar to what we were doing with type parameterization where we put in a place holder for the filter behavior and let the user fill in the details later. We've seen that we can't possibly think up all the scenarios that a user would need for our methods, so why not just allow them to tell us? Enter in behavior parameterization. 

But how does one do this? Generics make this easy for types, but how do you put a place holder in for behavior? This is facilitated by functional interfaces. 

## Functional Interfaces

A functional interface is very simple: it's just an interface with one non-default and non-static method. Seems kind of weird right? Like a waste of typing to create an interface with just one method. But it'll soon be clear why this structure is important when we get to the section of lambda expressions. 

You may not realize it, but you've already been using functional interfaces. Some examples are: `Runnable` and `Callable`, each containing just one method. Java 8 introduced some very important functional interfaces: `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>` to name just a few.

In order to look for a better definition of our `filter()` method, we're going to focus on the `Predicate<T>` interface. This interface has a single non-default and non-static method defined as: 

`boolean test(T t);`

Very simply put, this method says: Given a value of type `T`, check that this value matches some form of a predicate (or filter test), and respond with `true` or `false`. Well, this sounds exactly like what we'd want in our `filter()` method. Given some predicate that the **user** defines (emphasis on user-defined), return all objects that match this predicate. Now, if properly implemented, the user can give us their predicate requirements and we will simply pass out all animals that match it. **Wonderful**!. Let's take a stab at our new method definition:

```
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

```
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

```
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

```
Predicate<Animal> heavyDogsFilter = ...
Predicate<Animal> brownDogsFilter = ...
Predicate<Animal> heavyAndBrownDogsFilter = heavyDogsFilter.and(brownDogsFilter);

List<Animal> heavyAndBrownDogs = container.filter(heavyAndBrownDogsFilter);
```

And voila! You can create any powerful combination of filters you'd like.

## Lambdas

This is all well and good. We've started to see the power of behavior parameterization by differing the definitions and combinations of business logic to the consumer. Not only is our code cleaner by providing more generic methods, but we give the consumer the power to use it however they like. But lets go ahead and admit that the use `Predicate<T>` anonymous classes is pretty verbose and clunky. Such has been the curse of Java for years until the advent of Java 8 and it's introduction of lambda expressions. 

Lambda expressions allow you to replace the verbose anonymous class definitions with a succinct expression. Back to why functional interfaces are important: lambda expressions can't be used just anywhere in Java. They can be used in place of an anonymous class definition for a functional interface. The reason that they need to be used in conjunction with functional interfaces is that the compiler wouldn't know what method the lambda expression was replacing if there were several methods with the same signature. What if an interface had two methods that looked like this:

```
public boolean passes(T t);
public boolean notPasses(T t);
```

The compiler would have no indication which method the lambda expression was meant to replace. This is why they must be used with functional interfaces, defining only one non-default and non-static method. 

Okay, enough talking! What do lambdas look like? Here is what our new use of the filter method could look like:

```
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

Umm...wow! So much better! So much more succinct. So much more readable. So the generic form of a lambda expression for a `Predicate<T>` is: `(x) -> boolean`. Another example that takes a parameter and returns a user-defined type is `Function<T,R>`, or: `(x) -> R`.

I recently used this type of approach in some code I wrote pertaining to wrapping code in asynchronous blocks. Here is a link to the code if you are interested. It worked out nicely provide simple and reusable code blocks for a consumer to wrap around any code that needed to be run in the background within a Jersey controller: [Async code link](https://github.com/enelson/enelson.github.io/wiki/Asynchronous-code-blocks)

## Conclusion

Sorry if this was a TL;DR discussion of behavioral parameterization, but I felt that the concepts written about here were important to build up to first, understanding why it's important, and then second, how you can implement it. This concept can be used in innumerable cases and in many different circumstances. And each functional interface provides different types of behavior. I would encourage you to read any articles relating to this subject as it can lead to clean, powerful, reusable, composable code. And in the end, isn't this a goal that all of us "pragmatic" programmers aspire to? 

Thanks so much for taking the time to read this article, and please provide any feedback for me relating to anything that is unclear in my explanations, or anything that is wrong.
