# Behavior Parameterization

## What it is

### Concept similar to type parameterization

Behavior parameterization is far less scary than it sounds. It espouses the same concepts that type parameterization does. Type parameterization (generics) is, in essence, a place holder for types in a class/method/field. The writer is saying that they don't want to define the type here, and allow the consumer to define the type later. This is the case when the class/method/field behavior doesn't rely on the type, or is not effected by the type. So flexibility is given to the consumer to use whatever type they like, or at least within the bound of the type definition:

```
public <T> T doWork(T obj);
```

### Correlation

To broach the subject of behavior parameterization, let's start with a simple example. Lets say we have a class called `AnimalContainer` which contains many types of animals. We may want to have a simple method called `filter()` that allows us to filter out certain types of animals.

```
public class AnimalContainer {
    private List<Animal> animals;

    public List<Animal> filter() {
        List<Animal> results = new ArrayList<>();
        for(Animal animal : animals) {
            if(animal.getType() == 'Dog')
                results.add(animal);
        }
        return results;
    }

    public List<Animal> getAnimal() {
        return animals;
    }
    public void setAnimal(List<Animal> animals) {
        this.animals = animals;
    }
}
```

The filter method is great, except for the fact that it is completely inflexible.  What if you wanted to filter out dogs that were of a certain weight? Write another method with like this:

```
public List<Animal> filter(int weight);
```

And then later, what if someone wanted to filter out dogs of a certain weight *AND* a certain color? You see where this is going. The combinations can become unwieldy and you can't resort to writing a new method each time you need more options.

Plus, what if we want to filter out animals of type 'Cat'? What will we do? Write a new method called `filterCats`? And then write more methods for cat filters with many filtering options?

What would be nice is to take an approach similar to what we were doing with type parameterization where we put in a place holder and let the user fill in the details later. But in this case, they fill in the behavior. We've seen that we can't possibly think up all the scenarios that a user would need for our methods, so why not just allow them to tell us? Enter in behavior parameterization. 

But how does one do this? Generics make this easy for types, but how do you put a place holder in for behavior? This is facilitated by functional interfaces. 

## Functional Interfaces

A functional interface is very simple: it's just an interface with one non-default and non-static method. Seems kind of weird right? Like a waste of typing to create an interface with just one method. But it'll soon be clear why this structure is important when we get to the section of lambda expressions. 

You may not realize it, but you've already been using functional interfaces without realizing it. Some examples are: `Runnable` and `Callable`. Java 8 introduced some very important functional interfaces: `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>` to name a few.

In order to look for a better definition of our `filter()` method, we're going to focus on the `Predicate<T>` interface. This interface has a single non-default and non-static method defined as: 

`boolean test(T t);`

Very simply put, this method says: Given a value of type `T`, check that this value matches some form of a predicate, and respond with `true` or `false`. Well, this sounds exactly like what we'd want in our `filter()` method. Given some predicate that the **user** defines (emphasis on user-defined), return all objects that match this predicate. Now, if properly implemented, the user can give us their predicate requirements and we will simply pass out all animals that match it. **Wonderful**!. Let's take a stab at our new method definition:

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

Wow! Talk about powerful! Now, the user can pass in any type of filtering logic they wish, and our simple `filter()` method can accommodate any of it. This is exactly the type of power that behavior parameterization allows. So, what would the use of this filter look like?

```
AnimalContainer container = ...
List<Animal> filteredAnimals = container.filter(new Predicate<Animal>() {
    @Override
    boolean test(Animal animal) {
        if(animal.getType() == 'Dog' && animal.getWeight >= 100 && animal.getColor() == 'Brown')
            return true;
        else
            return false;
    }
});
```

This is how the consumer can create custom predicates on the fly, and in any combination. And, if could also create reusable filters by assigning predicates to variables:

```
Predicate<Animal> lightCats = new Predicate<Animal>() {
    @Override
    boolean test(Animal animal) {
        if(animal.getType() == 'Cat' && animal.getWeight < 20)
            return true;
        else
            return false;
    }
});

Predicate<Animal> heavyBrownDogs = new Predicate<Animal>() {
    @Override
    boolean test(Animal animal) {
        if(animal.getType() == 'Dog' && animal.getWeight >= 100 && animal.getColor() == 'Brown')
            return true;
        else
            return false;
    }
});
```

Now the consumer can create powerful, reusable and composable filters.

## Lambdas

## 