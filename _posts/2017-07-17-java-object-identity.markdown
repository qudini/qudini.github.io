---
layout: post
title:  "Java object identity, or how to override equals, hashCode and compareTo"
author: christophe_maillard
date:   2017-07-17 16:42:00 +0000
categories: qudini java equals hashcode compareto object identity
---
## Java object identity, or how to override equals, hashCode and compareTo

Two rules when dealing with object identities in Java:

1. Override `hashCode` when you override `equals`
2. Make `compareTo` consistent with `equals`

---

### Implement `hashCode` when you override `equals`

This is already nicely explained by many articles (see [this Stack Overflow answer](https://stackoverflow.com/a/2265637/1225328) for example), so I'll go straight to the point with an example.

Say you have the following `Item` class:

```java
public class Item {

  private final int id;

  public Item(int id) {
    this.id = id;
  }

  @Override
  public String toString() {
    return String.valueOf(id);
  }

}
```

You want to define its identity based on the field `id`, but you forget to override `hashCode`:

```java
@Override
public boolean equals(Object otherObject) {
  if (!(otherObject instanceof Item)) {
    return false;
  } else {
    Item otherItem = (Item) otherObject;
    return Objects.equals(id, otherItem.id);
  }
}
```

Later, you add some instances to a `Set`. As a `Set` removes duplicates, if you add two instances that have the same `id`, you could expect that only one is kept:

```java
Item i1 = new Item(1);
Item i2 = new Item(1);
Set<Item> items = new HashSet<>();
items.add(i1);
items.add(i2);
System.out.println(items); // weird, instances are not considered as duplicates: [1, 1]
```

Indeed, put simple, `HashSet` first compares the `hashCode`s of its elements: if they match, then it compares thanks to `equals`. But if they don't match, **it even doesn't uses `equals`**: elements are not identical.

Without `hashCode` being overridden by `Item`, [the `Object`'s `hashCode` method](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--) is called:

> As much as is reasonably practical, **the hashCode method defined by class `Object` does return distinct integers for distinct objects**. (This is typically implemented by converting the internal address of the object into an integer, but this implementation technique is not required by the Java™ programming language.)

This means that `i1.hashCode() != i2.hashCode()`, as `i1` and `i2` are distinct instances.

If we override the `hashCode` method based on the same `id` field though:

```java
@Override
public int hashCode() {
  return Objects.hash(id);
}
```

We do obtain the desired behavior:

```java
Item i1 = new Item(1);
Item i2 = new Item(1);
Set<Item> items = new HashSet<>();
items.add(i1);
items.add(i2);
System.out.println(items); // great, instances are successfully considered as duplicates: [1]
```

Indeed, in this case, `i1.hashCode() == i2.hashCode()`.

---

One side rule that you should be aware of, is that while two instances that are equal *must* return the same hash code, **two instances that return the same hash code are not necessarily equal**.

Indeed, when I say *"`HashSet` first compares the `hashCode`s of its elements: if they match, then it compares thanks to `equals`"*, one could wonder: why do we actually need to call `equals` once we know the `hashCode`s match?

Well, this tackles *hash collisions*. Take for example the two strings `"FB"` and `"Ea"`. Although they are obviously not equal, **they do produce the same hash code**:

```java
System.out.println("FB".hashCode()); // 2236
System.out.println("Ea".hashCode()); // 2236
```

So if `HashSet` used hash codes only, these two strings would have been considered equal: only one would have been kept.

---

### Make `compareTo` consistent with `equals`

This is less known, but not making `compareTo` consistent with `equals` can lead to bugs that are hard to get. Let's quote [`Comparable`'s documentation](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html):

> It is strongly recommended (though not required) that **natural orderings be consistent with equals**. This is so because sorted sets (and sorted maps) without explicit comparators behave "strangely" when they are used with elements (or keys) whose natural ordering is inconsistent with `equals`. In particular, such a sorted set (or sorted map) violates the general contract for set (or map), which is defined in terms of the `equals` method.

Let's take our example back. Say our `Item`s now have a `name` field, that is used when sorting a collection of `Item`s:

```java
public class Item implements Comparable<Item> {

  private final int id;
  private final String name;

  public Item(int id, String name) {
    this.id = id;
    this.name = name;
  }

  @Override
  public String toString() {
    return id + ":" + name;
  }

  @Override
  public boolean equals(Object otherObject) {
    if (!(otherObject instanceof Item)) {
      return false;
    } else {
      Item otherItem = (Item) otherObject;
      return Objects.equals(id, otherItem.id);
    }
  }

  @Override
  public int hashCode() {
    return Objects.hash(id);
  }

  @Override
  public int compareTo(Item o) {
    return name.compareTo(o.name);
  }

}
```

Now, let's create two instances:

```java
Item i1 = new Item(1, "first");
Item i2 = new Item(2, "first");
```

As you see, instances have different `id`, so thanks to our `equals`/`hashCode` implementations, they should not be considered equal, even if they share the same `name`. Let's add them into a `TreeSet` (a `Set` thats keep its elements ordered):

```java
Set<Item> items = new TreeSet<>();
items.add(i1);
items.add(i2);
System.out.println(items); // weird, instances are considered as duplicates: [1:first]
```

What happened here? Well, our `compareTo` implementation is not consistent with out `equals` implementation. Indeed, from [`compareTo` method's javadoc](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html#compareTo-T-):

> Returns a negative integer, **zero**, or a positive integer as this object is less than, **equal to**, or greater than the specified object.

In the example, both `Item`s share the same `name`: `"first"`. Thus, `compareTo` returns `0`, although our objects are not equal (they are just sharing the same `name`). The issue is that **`TreeSet` doesn't use `equals`/`hashCode` to check for duplicates, but `compareTo`'s result**, as it is implied that `compareTo` returns `0` if instances are equal. So `TreeSet` is actually right: to improve performances, it should not need to call `equals`/`hashCode` if `compareTo` has been already called.

To make our `compareTo` method consistent with our `equal` one, we must compare `id`s when `name`s are equal:

```java
@Override
public int compareTo(Item o) {
  int result = name.compareTo(o.name);
  if (result == 0) {
    result = Integer.valueOf(id).compareTo(o.id);
  }
  return result;
}
```

That way, **`compareTo` will return `0` only if `id`s are equal**.

Let's try again to add two instances sharing the same `name` in a `TreeSet`:

```java
Item i1 = new Item(1, "first");
Item i2 = new Item(2, "first");
Set<Item> items = new TreeSet<>();
items.add(i1);
items.add(i2);
System.out.println(items); // great, instances are not considered as duplicated: [1:first, 2:first]
```
