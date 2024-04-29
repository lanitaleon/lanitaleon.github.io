---
layout: post
title: Guava(1) Basic Utils
---

1.null

开销小，速度快，在对象数组中不可避免。

另一方面，歧义、奇怪的bug、混乱也不可避免，比如说在Map.get()中，null可以意味着key不存在，也可以意味着存在且值为null。

guava的很多工具包都不允许null；

在set以及map的key中，不要使用null；

在map的value中，不推荐将key不存在和value为null混在一起；

在list中，推荐使用Map<Integer,E>替代；

在枚举类中，使用指定值表达null的含义；

如果实在需要使用null，那就用，但是用不了一些禁用null的实现了，如：

使用Collections.unmodifiableList(Lists.newArrayList())替代ImmutableList.

2.Optional

```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```

表达一种模棱两可的状态，可能有值，也可能没有。

静态方法：

```java
Optional.of(T); // not null
Optional.absent(); // 不存在的对象
Optional.fromNullable(T); // 将一个可能为null的值转为Optional对象
```

非静态方法：

```java
boolean isPresent(); 
T get();
T or(T); // orElse(T)
T orNull(); // orElse(null)
Set<T> asSet(); // Optional.of(1) >> [1]
```

使用Optional的意义在于什么？

它强迫开发者正视且必须正视null的处理，null会让人轻易地回避潜在的问题。

3.Strings

```java
emptyToNull(String);
isNullOrEmpty(String);
nullToEmpty(String);
```

便捷地处理string中的null。

4.preconditions

```java
// IllegalArgumentException
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
checkArgument(i < j, "Expected i < j, but %s >= %s", i, j);
// NullPointerException
checkNotNull(T);
// IllegalStateException
checkState(boolean);
// IndexOutOfBoundsException
checkElementIndex(int index, int size); // < size
checkPositionIndex(int index, int size); // <= size
checkPositionIndexes(int start, int end, int size); // start < 0 || end < start || end > size
```

直白实用。

5.Object common methods

```java
Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true
```

顺利逃避令人痛苦的null检查。jdk7已实装。

```java
Objects.hash(Object...);
```

简洁地hashCode构建方式。jdk7已实装。

```java
   // Returns "ClassName{x=1}"
   MoreObjects.toStringHelper(this)
       .add("x", 1)
       .toString();

   // Returns "MyObject{x=1}"
   MoreObjects.toStringHelper("MyObject")
       .add("x", 1)
       .toString();
```

这个方法还挺迷惑的，主要用于debug log，比map稍微好写点。

```java
class Person implements Comparable<Person> {
  private String lastName;
  private String firstName;
  private int zipCode;

  public int compareTo(Person other) {
    int cmp = lastName.compareTo(other.lastName);
    if (cmp != 0) {
      return cmp;
    }
    cmp = firstName.compareTo(other.firstName);
    if (cmp != 0) {
      return cmp;
    }
    return Integer.compare(zipCode, other.zipCode);
  }
}
// guava
   public int compareTo(Foo that) {
     return ComparisonChain.start()
         .compare(this.aString, that.aString)
         .compare(this.anInt, that.anInt)
         .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
         .result();
   }
```

简化compareTo，可读性好些。

6.Ordering

https://github.com/google/guava/wiki/OrderingExplained

一个特殊的Comparator实例，与原生Comparator相性良好。

```java
Ordering.from(Comparator);
```

虽然如此，直接通过Ordering构建更为容易。

```java
class Foo {
  @Nullable String sortedBy;
  int notSortedBy;
}
Ordering<Foo> ordering = Ordering.natural().nullsFirst().onResultOf(new Function<Foo, String>() {
  public String apply(Foo foo) {
    return foo.sortedBy;
  }
});
```

链的顺序是从右往左。从右往左。从右往左。

Ordering.compound()从左往右，所以不要混用。

```java
greatestOf(Iterable iterable, int k);
isOrdered(Iterable);
sortedCopy(Iterable);
min(E...);
min(Iterable)
```

7.Throwables

https://github.com/google/guava/wiki/ThrowablesExplained

https://github.com/google/guava/wiki/Why-we-deprecated-Throwables.propagate

有时候可能需要捕获某种异常，然后抛出到上一层try/catch。

```java
try {
    someMethod();
} catch (Throwable t) {
    Throwables.throwIfInstanceOf(t, IOException.class);
    Throwables.throwIfInstanceOf(t, SQLException.class);
    Throwables.throwIfUnchecked(t);
    throw new RuntimeException(t);
}

@GwtIncompatible // Class.cast, Class.isInstance
public static <X extends Throwable> void throwIfInstanceOf(
  Throwable throwable, Class<X> declaredType) throws X {
checkNotNull(throwable);
if (declaredType.isInstance(throwable)) {
  throw declaredType.cast(throwable);
}
}

public static void throwIfUnchecked(Throwable throwable) {
checkNotNull(throwable);
if (throwable instanceof RuntimeException) {
  throw (RuntimeException) throwable;
}
if (throwable instanceof Error) {
  throw (Error) throwable;
}
}
```

throwIfInstanceOf抛出指定类型异常的实例。

throwIfUnchecked仅抛出Error或者RuntimeException，Error是编程无法解决的错误，而RuntimeException往往是编程错误。

guava在throwable这一节特别提到了一个过时的方法：Throwables.propagate(Throwable)，并详细解释了为什么弃用了这个封装。

```java
@GwtIncompatible
@Deprecated
public static RuntimeException propagate(Throwable throwable) {
  throwIfUnchecked(throwable);
  throw new RuntimeException(throwable);
}  
```

一般来说，弃用一个方法，都会定义一个等效的新方法替代它，或者至少会定义成单行的，毕竟它本身就只有一行，但是这个方法没有新方法替换它，它的等效实现是：

```java
throwIfUnchecked(e);
throw new RuntimeException(e);
```

这个封装好的propagate方法在Google内部，有超过10000次调用。

尽管作用有限，调用得足够多的话，也足以发挥作用。

但是实际上有75%的调用都用它来抛出一个已检查异常，这是违背方法初衷的。

```java
try {
  return something(...);
} catch (IOException e) {
  throw Throwables.propagate(e);
}
```

因为它就相当于。

```java
try {
  return something(...);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

又或者拿它来抛出一个未检查异常。

```java
try {
  return something(...);
} catch (CancellationException e) {
  throw Throwables.propagate(e);
}
```

这也相当于。

```java
try {
  return something(...);
} catch (CancellationException e) {
  throw e;
}
```

也就是说，75%的调用反而造成了冗余。

而剩下的25%中，也有很多情况可以通过直接throw new RuntimeException(e)解决，不管它是不是已检查的，用RuntimeException包一层抛出都是可以接受的。

还有一个不好的点是，在多层嵌套中捕获异常再抛出。

在不使用propagate的情况下，有了如下简化。

```java
try {
  return something(...);
} catch (CancellationException e) {
  throw e;
}
```

但是很明显，它是一个未检查异常，完全可以不做捕获。

```java
return something(...);
```

实际上推荐的propagate用法如下。

```java
throw Throwables.propagate(e);
```

这么写的话，编译器马上就知道这行代码后面的内容不会被执行。大部分人都是这么用的，但是还是有一部分人写成了这样。

```java
Throwables.propagate(e);
return new MyType<Something>(
    ImmutableList.<Foo>of(),
    ImmutableList.<Bar>of(),
    Something.empty()); // unreachable
```

只要使用throw就可以规避这种写法，它的起因就是propagate本身。

考虑如下代码。

```java
try {
  return callable.call();
} catch (AssertionError e) {
  int delay = retryStrategy.getDelayMillis(tries);
  if (delay >= 0) {
    try {
      Thread.sleep(delay);
    } catch (InterruptedException ie) {
      throw Throwables.propagate(e);
    }
  }
}
```

注意到propagate抛出的是e而不是ie，这种实现很容易让人误解又迷惑。

又或者是这样的代码。

```java
try {
  return something(...);
} catch (SomeException e) {
  Throwables.propagate(e);
  log.log(SEVERE, "error", e);
  return default;
}
```

一些人会认为propagate做了一些比throw更多的事情，实际上并没有。

为什么guava要定义一个跟throw差不多的方法，又为什么它的命名偏偏不是throwXXX而是propagate。

同样令人迷惑的还有这样的代码。

```java
private Something foo() throws IOException {
  try {
    return something(...);
  } catch (IOException e) {
    log.log(SEVERE, "error", e);
    Throwables.propagate(e);
  }
}
```

很明显，方法体上压根就不需要抛出IOException。

但是写这段代码的人似乎期望propagate原样抛出IOException，ta可能以为propagate是这样的。

```java
<X extends Throwable> void propagate(X x) throws X;
```

又或者ta是初学者，并不知道这种写法是否可行。

又或者ta本意如此，只是忘记删掉方法体上抛出的IOException。

这些意图都会因为propagate的存在变得不可捉摸，让人迷惑。

又或者是。

```java
logAndRethrow(Throwables.propagate(e))
```

实际上logAndRethrow永远不会被调用。

另一方面，propagate变相鼓励了"不关心异常处理"，这不是guava希望看到的。

如果不使用propagate，可以怎么做呢？

- 使用throw e / RuntimeException(e)完美替换，基本不影响原实现；

- 使用限定场景下更高级更合适的API，比如Futures.getUnchecked等；

- 自定义相似实现，继续使用。

从这篇弃用说明可以看到，guava的开发者并非制定规范，更多地是通过行为观察、大量的代码分析，尝试让使用者觉得天生就该有这么个工具类/包/功能。在命名上尤其苛刻，力求准确、无歧义，哪怕像propagate这样就两行代码的方法也能被注意到并详细分析。

