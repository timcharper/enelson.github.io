# Behavior Parameterization

## What it is

Behavior parameterization is far less scary than it sounds. It espouses the same concepts that type parameterization does. Type parameterization (generics) is, in essence, a place holder for types in a class/method/field. The writer is saying that they don't want to define the type here, and allow the consumer to define the type later. This is the case when the class/method/field behavior doesn't rely on the type, or is not effected by the type. So flexibility is given to the consumer to use whatever type they like, or at least within the bound of the type definition.

```
public <T> T doWork(T obj);
```