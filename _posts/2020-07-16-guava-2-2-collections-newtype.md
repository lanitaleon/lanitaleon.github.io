---
layout: post
title: guava(2.2) collections new types
tags:
  - guava
---

### 2.New Collection Types

guava定义了一些JDK中没有的，但是比较常用的集合结构。

Multiset，Multimap，BiMap，Table，ClassToInstanceMap，RangeSet，RangeMap。

#### 2.1Multiset

一个带有计数、元素可重复、无序的set。

假设如下场景，对一篇文章中出现的单词进行计数。

```java
List<String> words = Lists.newArrayList("Potter", "Potter", "Potter");
Map<String, Integer> counts = new HashMap<String, Integer>();
for (String word : words) {
    Integer count = counts.get(word);
    if (count == null) {
        counts.put(word, 1);
    } else {
        counts.put(word, count + 1);
    }
}
```

使用Multiset就可以简化实现。

```java
List<String> words = Lists.newArrayList("Potter", "Potter", "Potter");
Multiset<String> countSet = HashMultiset.create();
for (String word : words) {
    countSet.add(word);
}
```

即使是使用Map.merge简化，Multiset依然更加简洁。

```java
List<String> words = Lists.newArrayList("Potter", "Potter", "Potter");
// map
Map<String, Integer> counts = new HashMap<String, Integer>();
for (String word : words) {
    counts.merge(word, 1, Integer::sum);
}
// multiset
Multiset<String> countSet = HashMultiset.create();
countSet.addAll(words);
```

实际上Multiset具有两面性：

- 一个无序的ArrayList

add()添加元素，可迭代操作每个元素，size()获取的是所有出现次数的总数。

- 一个带有计数的集合Map<E, Integer>

count(object)获取出现次数，entrySet()类似于Map的entrySet，elementSet()类似于Map的keySet()。

一个更具体的例子如下所示。

```java
Multiset<String> bookStore = HashMultiset.create();
bookStore.add("Potter");
bookStore.add("Potter");
bookStore.add("Potter");
System.out.println(bookStore.contains("Potter")); // true
System.out.println(bookStore.count("Potter")); // 3

bookStore.remove("Potter");
System.out.println(bookStore.contains("Potter")); // true
System.out.println(bookStore.count("Potter")); // 2
bookStore.setCount("Potter", 5);
System.out.println(String.join(",", bookStore)); // Potter...[5]

for (Multiset.Entry<String> bookEntry : bookStore.entrySet()) {
    System.out.println(bookEntry.getElement()); // Potter
    System.out.println(bookEntry.getCount()); // 5
}

System.out.println(bookStore.elementSet()); // [Potter]
```

虽然Multiset具有Map的特征，但是它不是一个Map，它继承的是Collection接口。

```java
@GwtCompatible
public interface Multiset<E> extends Collection<E> {
  ...
}
```

除此之外，还有如下区别：

- Multiset的元素count只能大于0，count为0的元素不会出现在elementSet()和entrySet()中。

- Multiset的size()指的是所有元素的count的和，如果要查看distinct数量，使用elementSet().size()。

- Multiset的iterator()迭代的是所有出现，也就是iterator的长度是size()。

- Multiset支持直接设置元素的count，setCount(elem, 0)代表清空该元素的所有出现。

- Multiset的count()对不存在的元素返回值为0。

通过SortedMultiset可以提取Multiset指定范围内的元素。

```java
SortedMultiset<String> words = TreeMultiset.create();
words.add("a");
words.add("b");
words.add("c");
words.add("d");
SortedMultiset<String> queryWords = words.subMultiset("b", BoundType.CLOSED, "d", BoundType.OPEN);
System.out.println(queryWords); // b c
```

guava提供了一些Multiset的实现，大致与JDK中的map实现对应。

| Map               | Corresponding Multiset                                       | Supports null elements |
| ----------------- | ------------------------------------------------------------ | ---------------------- |
| HashMap           | [HashMultiset](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashMultiset.html) | Yes                    |
| TreeMap           | [TreeMultiset](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/TreeMultiset.html) | Yes                    |
| LinkedHashMap     | [LinkedHashMultiset](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/LinkedHashMultiset.html) | Yes                    |
| ConcurrentHashMap | [ConcurrentHashMultiset](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ConcurrentHashMultiset.html) | No                     |
| ImmutableMap      | [ImmutableMultiset](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html) | No                     |

#### 2.2Multimap

使用Multimap可以更简单地处理如下数据结构。

```java
Map<K, List<V>>
Map<K, Set<V>>
```

这种数据结构可能需要以下两种表现形式，Multimap通过asMap()可以支持这两种视图。

```
// 1-1
a -> 1
a -> 2
a -> 4
b -> 3
c -> 5
// 1-N
a -> [1, 2, 4]
b -> [3]
c -> [5]
```

构建一个Multimap如下，tree map和hash map的简单区别就是元素的有序和无序。

```java
// tree map
ListMultimap<String, Integer> treeListMultimap = MultimapBuilder.treeKeys().arrayListValues().build();
treeListMultimap.put("euclid", 4); // [4]
List<Integer> valueList = Lists.newArrayList(1, 2, 3, 2);
treeListMultimap.putAll("euclid", valueList); // [4, 1, 2, 3, 2]
List<Integer> list = treeListMultimap.get("euclid");
list.add(5); // [4, 1, 2, 3, 2, 5]
list.remove(0); // [1, 2, 3, 2, 5]
treeListMultimap.remove("euclid", 2); // [1, 3, 2, 5]
List<Integer> replaceList = Lists.newArrayList(8, 9, 10);
treeListMultimap.replaceValues("euclid", replaceList); // [7, 8, 9]
treeListMultimap.removeAll("euclid"); 
List<Integer> emptyList = treeListMultimap.get("euclid"); // []
boolean exists = treeListMultimap.containsKey("euclid"); //false
// hash map
SetMultimap<Integer, MyEnum> hashEnumMultimap =
        MultimapBuilder.hashKeys().enumSetValues(MyEnum.class).build();
hashEnumMultimap.put(0, MyEnum.DONE);
```

与Multiset一样，如果集合必须包含至少一个元素，否则视为不存在于Multimap中。

同样的，Multimap虽然具备Map的一些特征，比如entries、keySet、keys、values等，但是它也不是一个Map，它并没有继承Map接口。

```java
@GwtCompatible
public interface Multimap<K, V> {} 
```

更具体的差异描述如下：

Multimap.containsKey仅在集合对象中至少包含一个元素的情况下为true，当value为null或者空集合时视为false；

Multimap.entries的返回值是1-1的键值对，如下所示；

```java
ListMultimap<String, Integer> treeListMultimap = MultimapBuilder.treeKeys().arrayListValues().build();
treeListMultimap.put("euclid", 1);
treeListMultimap.put("euclid", 2);
treeListMultimap.put("tulip", 3);
for (Map.Entry<String, Integer> entry : treeListMultimap.entries()) {
    System.out.println(entry.getKey());
    System.out.println(entry.getValue());
} 
// euclid 1 euclid 2 tulip 3
```

Multimap.size的返回值是entries的数量，并非keySet().size。

#### 2.3BiMap

key和value相互索引，都是唯一的。

```java
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("euclid", 1);
BiMap<String, Integer> map = HashBiMap.create(hashMap);

BiMap<String, Integer> map = HashBiMap.create();
map.put("euclid", 1);
map.inverse().get(1); // euclid
map.forcePut("euclid", 2);
map.inverse().get(1); // null
```

常用的其他实现如下所示：

| Key-Value Map Impl | Value-Key Map Impl | Corresponding BiMap                                          |
| ------------------ | ------------------ | ------------------------------------------------------------ |
| HashMap            | HashMap            | [HashBiMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashBiMap.html) |
| ImmutableMap       | ImmutableMap       | [ImmutableBiMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html) |
| EnumMap            | EnumMap            | [EnumBiMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/EnumBiMap.html) |
| EnumMap            | HashMap            | [EnumHashBiMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/EnumHashBiMap.html) |

EnumBiMap与EnumHashBiMap的区别是，EnumBiMap要求key和value都是Enum。

```java
EnumMap<MyEnum, String> enumStringEnumMap = new EnumMap<>(MyEnum.class);
enumStringEnumMap.put(MyEnum.TODO, "待办");
enumStringEnumMap.put(MyEnum.DONE, "完成");
EnumHashBiMap<MyEnum, String> enumBiMap = EnumHashBiMap.create(enumStringEnumMap);

EnumMap<MyEnum, Foo> enumStringEnumMap = new EnumMap<>(MyEnum.class);
enumStringEnumMap.put(MyEnum.TODO, Foo.EUCLID);
enumStringEnumMap.put(MyEnum.DONE, Foo.POISSION);
EnumBiMap<MyEnum, Foo> enumBiMap = EnumBiMap.create(enumStringEnumMap);
```

#### 2.4Table

需要使用两个主键索引元素时，可以替换掉Map实现，比较简洁。

```java
// hash map
Map<Integer, Map<Integer, String>> map = new HashMap<>();
// hash table
Table<Integer, Integer, String> table = HashBasedTable.create();
table.put(1, 1, "1行1列");
table.put(1, 2, "1行2列");
table.put(2, 1, "2行1列");
table.put(2, 2, "2行2列");
Map<Integer, String> row = table.row(1);
Map<Integer, Map<Integer, String>> rowMap = table.rowMap();
Map<Integer, String> column = table.column(1);
for (Table.Cell<Integer, Integer, String> cell : table.cellSet()) {
    cell.getRowKey();
    cell.getColumnKey();
    cell.getValue();
}
// tree table
Table<Integer, Integer, String> table = TreeBasedTable.create();
// immutable table
Table<Integer, Integer, String> table = ImmutableTable.<Integer, Integer, String>builder()
        .put(1, 1, "1行1列")
        .build();
// array table
Table<Integer, Integer, String> arrayTable = ArrayTable.create(table);
```

其中ArrayTable比较特殊，先看一下HashBasedTable的实现。

```java
@GwtCompatible(serializable = true)
public class HashBasedTable<R, C, V> extends StandardTable<R, C, V> {
  ...
  public static <R, C, V> HashBasedTable<R, C, V> create() {
    return new HashBasedTable<>(new LinkedHashMap<R, Map<C, V>>(), new Factory<C, V>(0));
  }

  HashBasedTable(Map<R, Map<C, V>> backingMap, Factory<C, V> factory) {
    super(backingMap, factory);
  }  
  ...    
}

@GwtCompatible
class StandardTable<R, C, V> extends AbstractTable<R, C, V> implements Serializable {
  @GwtTransient final Map<R, Map<C, V>> backingMap;
  @GwtTransient final Supplier<? extends Map<C, V>> factory;

  StandardTable(Map<R, Map<C, V>> backingMap, Supplier<? extends Map<C, V>> factory) {
    this.backingMap = backingMap;
    this.factory = factory;
  }
  ...
} 
```

可以看到HashBasedTable最终的结构依然是一个Map，包括TreeBasedTable、ImmutableTable也是类似的实现，接下来看一下但是ArrayTable。

```java
@Beta
@GwtCompatible(emulated = true)
public final class ArrayTable<R, C, V> extends AbstractTable<R, C, V> implements Serializable {
  ...
  public static <R, C, V> ArrayTable<R, C, V> create(Table<R, C, V> table) {
    return (table instanceof ArrayTable<?, ?, ?>)
        ? new ArrayTable<R, C, V>((ArrayTable<R, C, V>) table)
        : new ArrayTable<R, C, V>(table);
  }

  private ArrayTable(Table<R, C, V> table) {
    this(table.rowKeySet(), table.columnKeySet());
    putAll(table);
  }   

  private final ImmutableList<R> rowList;
  private final ImmutableList<C> columnList;

  private final ImmutableMap<R, Integer> rowKeyToIndex;
  private final ImmutableMap<C, Integer> columnKeyToIndex;
  private final V[][] array;  

  private ArrayTable(Iterable<? extends R> rowKeys, Iterable<? extends C> columnKeys) {
    this.rowList = ImmutableList.copyOf(rowKeys);
    this.columnList = ImmutableList.copyOf(columnKeys);
    checkArgument(rowList.isEmpty() == columnList.isEmpty());

    rowKeyToIndex = Maps.indexMap(rowList);
    columnKeyToIndex = Maps.indexMap(columnList);

    @SuppressWarnings("unchecked")
    V[][] tmpArray = (V[][]) new Object[rowList.size()][columnList.size()];
    array = tmpArray;

    eraseAll();
  }  

  @Override
  public void putAll(Table<? extends R, ? extends C, ? extends V> table) {
    super.putAll(table);
  }  

  @CanIgnoreReturnValue
  @Override
  public V put(R rowKey, C columnKey, @Nullable V value) {
    checkNotNull(rowKey);
    checkNotNull(columnKey);
    Integer rowIndex = rowKeyToIndex.get(rowKey);
    checkArgument(rowIndex != null, "Row %s not in %s", rowKey, rowList);
    Integer columnIndex = columnKeyToIndex.get(columnKey);
    checkArgument(columnIndex != null, "Column %s not in %s", columnKey, columnList);
    return set(rowIndex, columnIndex, value);
  }

  @CanIgnoreReturnValue
  public V set(int rowIndex, int columnIndex, @Nullable V value) {
    // In GWT array access never throws IndexOutOfBoundsException.
    checkElementIndex(rowIndex, rowList.size());
    checkElementIndex(columnIndex, columnList.size());
    V oldValue = array[rowIndex][columnIndex];
    array[rowIndex][columnIndex] = value;
    return oldValue;
  }     
}

@GwtCompatible
abstract class AbstractTable<R, C, V> implements Table<R, C, V> {
  ...
  @Override
  public void putAll(Table<? extends R, ? extends C, ? extends V> table) {
    for (Table.Cell<? extends R, ? extends C, ? extends V> cell : table.cellSet()) {
      put(cell.getRowKey(), cell.getColumnKey(), cell.getValue());
    }
  }
  ...
}  
```

可以看到ArrayTable的最终的结构是一个二维数组，这也是ArrayTable要求在初始化时就指定完整数据的原因。

ArrayTable使用二维数组是为了在处理大量数据时提高运算效率。

另外在类的注解上可以看到@Beta的注解，以及该类中多处标注了todo字样，是一个还不稳定的实现。

#### 2.5ClassToInstanceMap

有时候会需要这样一个Map，它的key就是value的类型，ClassToInstanceMap就是这样的Map。

```java
// mutable
ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
numberDefaults.putInstance(Integer.class, 1);
numberDefaults.putInstance(Double.class, 1d);
numberDefaults.putInstance(BigDecimal.class, new BigDecimal("0.00001"));
Number n1 = numberDefaults.get(Integer.class);
Double n2 = numberDefaults.getInstance(Double.class);
BigDecimal n3 = numberDefaults.getInstance(BigDecimal.class);
// immutable
ClassToInstanceMap<Number> map = ImmutableClassToInstanceMap.<Number>builder()
        .put(Integer.class, 1)
        .build();
```

#### 2.6RangeSet

比较方便的处理连续或不连续的区间，直接看例子。

```java
RangeSet<Integer> rangeSet = TreeRangeSet.create();
rangeSet.add(Range.closed(1, 10)); // {[1, 10]}
rangeSet.add(Range.closedOpen(11, 15)); // disconnected range: {[1, 10], [11, 15)}
rangeSet.add(Range.closedOpen(15, 20)); // connected range; {[1, 10], [11, 20)}
rangeSet.add(Range.openClosed(0, 0)); // empty range; {[1, 10], [11, 20)}
rangeSet.remove(Range.open(5, 10)); // splits [1, 10]; {[1, 5], [10, 10], [11, 20)}
```

注意到Range.closed(1, 10)和Range.closedOpen(11, 15)并没有被合并为Range.closedOpen(1, 15)。

这里需要通过canonical方法先处理成连续区间后才会自动合并。

```java
Range.closed(1, 10).canonical(DiscreteDomain.integers()); // [1, 11)
```

处理区间，必然要处理排序、合并、切割，这就要求操作对象是可比较的，也就是必须实现Comparable接口。

```java
@Beta
@DoNotMock("Use ImmutableRangeSet or TreeRangeSet")
@GwtIncompatible
public interface RangeSet<C extends Comparable> {}
```

一些常用方法的例子。

```java
RangeSet<Integer> c = rangeSet.complement(); // 补集
RangeSet<Integer> s = rangeSet.subRangeSet(Range.closed(6, 15)); // 交集
Set<Range<Integer>> ranges = rangeSet.asRanges();
rangeSet.span(); // 能包含set中所有range的最小range
rangeSet.encloses(Range.closed(8, 11)); // 是否包含指定集
rangeSet.contains(1); // 是否包含指定元素
rangeSet.rangeContaining(1); // 包含指定元素的集
// 不可变限定 asSet
ImmutableRangeSet<Integer> rangeSet = ImmutableRangeSet
    .<Integer>builder().add(Range.closed(1, 10)).build();
ImmutableSortedSet<Integer> set = rangeSet.asSet(DiscreteDomain.integers());
```

#### 2.7RangeMap

以Range为key的Map，一些常用方法。

```java
RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
rangeMap.put(Range.closed(1, 10), "foo"); // {[1, 10] => "foo"}
rangeMap.put(Range.open(3, 6), "bar"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo"}
rangeMap.put(Range.open(10, 20), "euclid"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo", (10, 20) => "foo"}
rangeMap.remove(Range.closed(5, 11)); // {[1, 3] => "foo", (3, 5) => "bar", (11, 20) => "foo"}
rangeMap.get(2);
rangeMap.putCoalescing(Range.closed(20, 30), "euclid");
rangeMap.subRangeMap(Range.closed(22, 25));
Map<Range<Integer>, String> map = rangeMap.asMapOfRanges();
```

可以通过元素查找到值。

put和putCoalescing的区别在于，后者会查询value相同的Range，尝试将Range连接。

subRangeMap通过Range切割出部分RangeMap。

