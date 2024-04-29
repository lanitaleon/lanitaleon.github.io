---
layout: post
title: Guava(4) Collection Utilities
---

### 3.Collection Utilities

据说是guava最受欢迎的部分，基于Collections延伸的集合静态处理方法。

| Interface                                                                             | JDK or Guava? | Corresponding Guava utility class                                                                                    |
| ------------------------------------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------- |
| Collection                                                                            | JDK           | [Collections2](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Collections2.html) |
| List                                                                                  | JDK           | [Lists](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html)               |
| Set                                                                                   | JDK           | [Sets](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html)                 |
| SortedSet                                                                             | JDK           | [Sets](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html)                 |
| Map                                                                                   | JDK           | [Maps](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html)                 |
| SortedMap                                                                             | JDK           | [Maps](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html)                 |
| Queue                                                                                 | JDK           | [Queues](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Queues.html)             |
| [Multiset](https://github.com/google/guava/wiki/NewCollectionTypesExplained#Multiset) | Guava         | [Multisets](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html)       |
| [Multimap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#Multimap) | Guava         | [Multimaps](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html)       |
| [BiMap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#BiMap)       | Guava         | [Maps](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html)                 |
| [Table](https://github.com/google/guava/wiki/NewCollectionTypesExplained#Table)       | Guava         | [Tables](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html)             |

JDK中的常用集合以及对应的guava中的工具类如上所示。

#### 3.1Constructors

非常方便的创建集合实例。

```java
Set<Integer> copySet = Sets.newHashSet(1, 2, 3);
List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);
Multiset<String> multiset = HashMultiset.create();
```

#### 3.2Iterables

```java
Iterable<String> strings = Iterables.concat(copySet, theseElements);
int occurrences = Iterables.frequency(strings, "a");
Iterable<List<String>> lists = Iterables.partition(strings, 3);
String first = Iterables.getFirst(strings, "");
String last = Iterables.getLast(strings);
String theElement = Iterables.getOnlyElement(strings);
Iterables.elementsEqual(copySet, copySet2);
```

#### 3.3Lists

```java
List<Integer> countUp = Ints.asList(1, 2, 3, 4, 5);
List<Integer> countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}
List<List<Integer>> parts = Lists.partition(countUp, 2); // { {1, 2}, {3, 4}, {5}}
Longs.max(1, 2, 3);
Ints.max(1, 2, 3);
Collections.max(Arrays.asList(1, 2, 3)); // 不推荐
max(asList(1, 2, 3)); // 推荐
```

像max，asList这些方法都是静态的，直接引用看起来可读性更好，更简洁。

#### 3.4Sets

Sets中引入了很多对区间的操作，就像Range那样，返回值是一个SetView。

```java
Sets.SetView<Integer> set1 = Sets.union(Sets.newHashSet(1, 2, 3), Sets.newHashSet(3, 4));
// [1, 2, 3, 4]
Sets.SetView<Integer> set2 = Sets.intersection(Sets.newHashSet(1, 2, 3), Sets.newHashSet(3, 4));// [3]
Sets.SetView<Integer> set3 = Sets.difference(Sets.newHashSet(1, 2, 3), Sets.newHashSet(3, 4));// [1, 2]
Sets.SetView<Integer> set4 = Sets.symmetricDifference(Sets.newHashSet(1, 2, 3), Sets.newHashSet(3, 4)); // [1, 2, 4]
```

SetView可以直接当做Set使用，其中有两个常用的方法，copyInto可以将视图拷贝到可变集合中，immutableCopy可以将视图拷贝到一个不可变集合。

```java
public abstract static class SetView<E> extends AbstractSet<E> {
    ...
    @CanIgnoreReturnValue
    public <S extends Set<E>> S copyInto(S set) {
      set.addAll(this);
      return set;
    }

    public ImmutableSet<E> immutableCopy() {
      return ImmutableSet.copyOf(this);
    }    
    ...
}
```

Sets还提供了笛卡尔积和子集的实现。

```java
Set<String> animals = ImmutableSet.of("gerbil", "hamster");
Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");
Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
// [[gerbil, apple], [gerbil, orange], [gerbil, banana], [hamster, apple], [hamster, orange], [hamster, banana]]
Set<Set<String>> animalSets = Sets.powerSet(animals);
// [[][gerbil][hamster][gerbil, hamster]]
```

#### 3.5Maps

Maps设计了很多很有特色的方法。

```java
List<String> strings = Lists.newArrayList("alpha", "beta", "3");
ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings, (String string) -> {
    assert string != null;
    return string.length();
});
```

uniqueIndex可以通过自定义的某种规则，将指定集合转成Map，如果不是唯一的，可以使用Multimaps.index。

```java
ImmutableListMultimap<Integer, String> stringsByIndex = Multimaps.index(strings, (String string) -> {
    assert string != null;
    return string.length();
}); // {5=[alpha, gamma], 4=[beta]}
```

difference可以比较Map之间的差异，生成四个视图。

```java
Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
Map<String, Integer> right = ImmutableMap.of("b", 2, "c", 4, "d", 5);
MapDifference<String, Integer> diff = Maps.difference(left, right);

diff.entriesInCommon(); // {"b" => 2}
diff.entriesDiffering(); // {"c" => (3, 4)} 
// Map<String, MapDifference.ValueDifference<Integer>>
diff.entriesOnlyOnLeft(); // {"a" => 1}
diff.entriesOnlyOnRight(); // {"d" => 5}
```

#### 3.6Multisets

针对出现次数的一些常用方法。

```java
Multiset<String> multiset1 = HashMultiset.create();
multiset1.add("a", 2);

Multiset<String> multiset2 = HashMultiset.create();
multiset2.add("a", 5);

multiset1.containsAll(multiset2); // returns true: all unique elements are contained,
// even though multiset1.count("a") == 2 < multiset2.count("a") == 5
Multisets.containsOccurrences(multiset1, multiset2); // returns false

Multisets.removeOccurrences(multiset2, multiset1); // multiset2 now contains 3 occurrences of "a"

multiset2.removeAll(multiset1); // removes all occurrences of "a" from multiset2, even though multiset1.count("a") == 2
multiset2.isEmpty(); // returns true

Multiset<String> multiset = HashMultiset.create();
multiset.add("c", 1);
multiset.add("b", 5);
multiset.add("a", 3);
ImmutableMultiset<String> highestCountFirst = Multisets.copyHighestCountFirst(multiset);
// [b x 5, a x 3, c]
```

#### 3.7Multimaps

index，与Maps.uniqueIndex类似，区别是后者要求元素唯一，否则报错。

```java
// Multimaps.index
ImmutableSet<String> digits = ImmutableSet.of(
        "zero", "one", "two", "three", "four",
        "five", "six", "seven", "eight", "nine");
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
    public Integer apply(String string) {
        return string.length();
    }
};
ImmutableListMultimap<Integer, String> digitsByLength = Multimaps.index(digits, lengthFunction);
// {4=[zero, four, five, nine], 3=[one, two, six], 5=[three, seven, eight]}

// Maps.uniqueIndex
List<String> strings = Lists.newArrayList("beta", "gamma");
ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings, new Function<String, Integer> () {
    public Integer apply(String string) {
        return string.length();
    }
});
// {3=abc, 5=gamma}
```

invertFrom，置换key-value。

```java
ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
multimap.putAll("b", Ints.asList(2, 4, 6));
multimap.putAll("a", Ints.asList(4, 2, 1));
multimap.putAll("c", Ints.asList(2, 5, 3));
TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap.create());
```

forMap，将普通Map转成Multimap。

```java
Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
// multimap maps ["a" => {1}, "b" => {1}, "c" => {2}]
Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap.create());
```

warppers，提供了默认的初始化封装实现。

| Multimap type     | Unmodifiable                                                                                                                                                                                                  | Synchronized                                                                                                                                                                                                  | Custom                                                                                                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Multimap          | [unmodifiableMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableMultimap-com.google.common.collect.Multimap-)                            | [synchronizedMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedMultimap-com.google.common.collect.Multimap-)                            | [newMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap-java.util.Map-com.google.common.base.Supplier-)                   |
| ListMultimap      | [unmodifiableListMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableListMultimap-com.google.common.collect.ListMultimap-)                | [synchronizedListMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedListMultimap-com.google.common.collect.ListMultimap-)                | [newListMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newListMultimap-java.util.Map-com.google.common.base.Supplier-)           |
| SetMultimap       | [unmodifiableSetMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSetMultimap-com.google.common.collect.SetMultimap-)                   | [synchronizedSetMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSetMultimap-com.google.common.collect.SetMultimap-)                   | [newSetMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSetMultimap-java.util.Map-com.google.common.base.Supplier-)             |
| SortedSetMultimap | [unmodifiableSortedSetMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSortedSetMultimap-com.google.common.collect.SortedSetMultimap-) | [synchronizedSortedSetMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSortedSetMultimap-com.google.common.collect.SortedSetMultimap-) | [newSortedSetMultimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSortedSetMultimap-java.util.Map-com.google.common.base.Supplier-) |

一个例子，支持传入Supplier。

```java
ListMultimap<String, Integer> myMultimap = Multimaps.newListMultimap(
        Maps.<String, Collection<Integer>>newTreeMap(),
        new Supplier<LinkedList<Integer>>() {
            public LinkedList<Integer> get() {
                return Lists.newLinkedList();
            }
        });
```

需要注意的是：

第一个参数是源引用，必须是空Map；

第二个参数是Collection的构造器，但是Multimap.get(key)的返回类型并不是Supplier的类型，是一个封装的List。

3.8Tables

4.Extension Utilities

4.1Forwarding

4.2Decorators

4.3PeekingIterator

4.4AbstractIterator
