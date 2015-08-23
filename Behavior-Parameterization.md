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
            if(animals.getType() == 'Elephant')
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