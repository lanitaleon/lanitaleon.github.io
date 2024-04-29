---
layout: post
title: Guava(2) Immutable Collections
---

### 1.Immutable Collections

不可变集合，特点：

- 被不受信任的库调用是安全的

- 线程安全

- 不需要考虑值的变化从而节省时间和空间

- 可以用作常量

- 不接收null值

创建一个不可变集合的简单例子如下：

```java
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
        "red",
        "orange",
        "yellow",
        "green",
        "blue",
        "purple");

class Foo {
    final ImmutableSet<Bar> bars;

    Foo(Set<Bar> bars) {
        this.bars = ImmutableSet.copyOf(bars); // defensive copy!
    }
}
```

值得注意的是，copyOf是浅拷贝。

```java
Map<Integer, StringBuilder> map = new HashMap<>();
map.put(1, new StringBuilder("1"));
map.put(2, new StringBuilder("2"));

Map<Integer, StringBuilder> unmodifiableMap = Collections.unmodifiableMap(map);
unmodifiableMap.get(2).append("unmodifiable");
System.out.println(unmodifiableMap.get(2)); // 2unmodifiable

Map<Integer, StringBuilder> immutableMap = ImmutableMap.copyOf(map);
immutableMap.get(2).append("immutable");
System.out.println(immutableMap.get(2)); //2unmodifiableimmutable

System.out.println(map.get(2)); // 2unmodifiableimmutable
System.out.println(map.get(2) == immutableMap.get(2)); // true
System.out.println(map.get(2) == unmodifiableMap.get(2)); // true

map.put(3, new StringBuilder("3"));
System.out.println(unmodifiableMap.get(3)); // 3
System.out.println(immutableMap.get(3)); // null
```

上述例子中提到了Collections.unmodifiableXXX，两者的差别：

- 只有外部没有拥有对象引用的时候Collections.unmodifiableXXX才是真正不可变的
- 笨重而冗长

- 低效，返回的数据结构仍然具有可变集合的所有开销

关于低效的解释，可以简单看一下unmodifiableMap的实现。

```java
public class Collections {
    ...
    public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m) {
        return new UnmodifiableMap<>(m);
    }

    private static class UnmodifiableMap<K,V> implements Map<K,V>, Serializable {
        private static final long serialVersionUID = -1034234728574286014L;

        private final Map<? extends K, ? extends V> m;

        UnmodifiableMap(Map<? extends K, ? extends V> m) {
            if (m==null)
                throw new NullPointerException();
            this.m = m;
        }
		...
        public V put(K key, V value) {
            throw new UnsupportedOperationException();
        }
        public V remove(Object key) {
            throw new UnsupportedOperationException();
        }
        public void putAll(Map<? extends K, ? extends V> m) {
            throw new UnsupportedOperationException();
        }
        public void clear() {
            throw new UnsupportedOperationException();
        }
        ...
    }
}        
```

可以看到，UnmodifiableMap仅仅对Map接口中修改值的实现重写为不可用，对源引用并没有再做拷贝，所以对源引用来说，数据结构没有发生变化，本质上仍是可变的，要承担可变的开销（并发修改检查、哈希表中的额外空间等）。

这在一些场景下是适用的，但是guava认为这种不可变不够彻底，可以简单看下ImmutableList的实现，做一个对比。

```java
@GwtCompatible(serializable = true, emulated = true)
@SuppressWarnings("serial")
public abstract class ImmutableList<E> extends ImmutableCollection<E>
    implements List<E>, RandomAccess {
  ...
  public static <E> ImmutableList<E> copyOf(Collection<? extends E> elements) {
    if (elements instanceof ImmutableCollection) {
      @SuppressWarnings("unchecked") // all supported methods are covariant
      ImmutableList<E> list = ((ImmutableCollection<E>) elements).asList();
      return list.isPartialView() ? ImmutableList.<E>asImmutableList(list.toArray()) : list;
    }
    return construct(elements.toArray());
  }

  private static <E> ImmutableList<E> construct(Object... elements) {
    return asImmutableList(checkElementsNotNull(elements));
  }

  static <E> ImmutableList<E> asImmutableList(Object[] elements) {
    return asImmutableList(elements, elements.length);
  }

  static <E> ImmutableList<E> asImmutableList(Object[] elements, int length) {
    switch (length) {
      case 0:
        return of();
      case 1:
        return of((E) elements[0]);
      default:
        if (length < elements.length) {
          elements = Arrays.copyOf(elements, length);
        }
        return new RegularImmutableList<E>(elements);
    }
  }  
  ...
}  

@GwtCompatible(serializable = true, emulated = true)
@SuppressWarnings("serial") 
class RegularImmutableList<E> extends ImmutableList<E> {
  ...
  static final ImmutableList<Object> EMPTY = new RegularImmutableList<>(new Object[0]);

  @VisibleForTesting final transient Object[] array;

  RegularImmutableList(Object[] array) {
    this.array = array;
  }
  ...
} 
```

如果对象本身就是不可变集合，那么直接返回。

如果对象是可变集合，先检查集合元素是否不为null，然后再做一层拷贝，将拷贝结果返回。

尽管如此，guava的不可变集合并不是真正意义上的深拷贝，集合中的元素如果是引用，那么元素依然是可变的。



剩余问题：

https://www.jianshu.com/p/bf2623f18d6a

https://zhangzw.com/posts/20190718.html

https://demonyangyue.github.io/

```
@LazyInit
@VisibleForTesting
@GwtCompatible
```

