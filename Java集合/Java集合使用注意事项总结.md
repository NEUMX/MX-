# Java集合使用注意事项总结

# Java集合使用注意事项总结
这篇文章我根据《阿里巴巴 Java 开发手册》总结了关于集合使用常见的注意事项以及其具体原理。

强烈建议小伙伴们多多阅读几遍，避免自己写代码的时候出现这些低级的问题。

## 集合判空
《阿里巴巴 Java 开发手册》的描述如下：

*_判断所有集合内部的元素是否为空，使用__*__isEmpty()__**方法，而不是**__size()==0__*__的方式。_*

这是因为isEmpty()方法的可读性更好，并且时间复杂度为 O(1)。

绝大部分我们使用的集合的size()方法的时间复杂度也是 O(1)，不过，也有很多复杂度不是 O(1) 的，比如java.util.concurrent包下的某些集合（ConcurrentLinkedQueue、ConcurrentHashMap...）。

下面是ConcurrentHashMap的size()方法和isEmpty()方法的源码。

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
public boolean isEmpty() {
    return sumCount() <= 0L; // ignore transient negative values
}
```

## 集合转 Map
《阿里巴巴 Java 开发手册》的描述如下：

*_在使用__**java.util.stream.Collectors**__类的__*__toMap()__**方法转为**__Map__*__集合时，一定要注意当 value 为 null 时会抛 NPE 异常。_*

```java
class Person {
    private String name;
    private String phoneNumber;
    // getters and setters
}

List<Person> bookList = new ArrayList<>();
bookList.add(new Person("jack","18163138123"));
bookList.add(new Person("martin",null));
// 空指针异常
bookList.stream().collect(Collectors.toMap(Person::getName, Person::getPhoneNumber));
```

下面我们来解释一下原因。

首先，我们来看java.util.stream.Collectors类的toMap()方法 ，可以看到其内部调用了Map接口的merge()方法。

```java
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                         Function<? super T, ? extends U> valueMapper,
                         BinaryOperator<U> mergeFunction,
                         Supplier<M> mapSupplier) {
    BiConsumer<M, T> accumulator
    = (map, element) -> map.merge(keyMapper.apply(element),
                                  valueMapper.apply(element), mergeFunction);
    return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
}
```

Map接口的merge()方法如下，这个方法是接口中的默认实现。

如果你还不了解 Java 8 新特性的话，请看这篇文章：[《Java8 新特性总结》](https://mp.weixin.qq.com/s/ojyl7B6PiHaTWADqmUq2rw)。

```java
default V merge(K key, V value,
                BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
    remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

merge()方法会先调用Objects.requireNonNull()方法判断 value 是否为空。

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

## 集合遍历
《阿里巴巴 Java 开发手册》的描述如下：

**不要在 foreach 循环里进行元素的****remove/add****操作。remove 元素请使用****Iterator****方式，如果并发操作，需要对****Iterator****对象加锁。**

通过反编译你会发现 foreach 语法底层其实还是依赖Iterator。不过，remove/add操作直接调用的是集合自己的方法，而不是Iterator的remove/add方法

这就导致Iterator莫名其妙地发现自己有元素被remove/add，然后，它就会抛出一个ConcurrentModificationException来提示用户发生了并发修改异常。这就是单线程状态下产生的**fail-fast 机制**。

**fail-fast 机制**：多个线程对 fail-fast 集合进行修改的时候，可能会抛出ConcurrentModificationException。 即使是单线程下也有可能会出现这种情况，上面已经提到过。相关阅读：[什么是 fail-fast](https://www.cnblogs.com/54chensongxia/p/12470446.html)。

Java8 开始，可以使用Collection#removeIf()方法删除满足特定条件的元素,如

```java
List<Integer> list = new ArrayList<>();
for (int i = 1; i <= 10; ++i) {
    list.add(i);
}
list.removeIf(filter -> filter % 2 == 0); /* 删除list中的所有偶数 */
System.out.println(list); /* [1, 3, 5, 7, 9] */
```

除了上面介绍的直接使用Iterator进行遍历操作之外，你还可以：

+ 使用普通的 for 循环
+ 使用 fail-safe 的集合类。java.util包下面的所有的集合类都是 fail-fast 的，而java.util.concurrent包下面的所有的类都是 fail-safe 的。
+ ……

## 集合去重
《阿里巴巴 Java 开发手册》的描述如下：

*_可以利用__**Set**__元素唯一的特性，可以快速对一个集合进行去重操作，避免使用__**List**__的_***contains()****进行遍历去重或者判断包含操作。**

这里我们以HashSet和ArrayList为例说明。

```java
// Set 去重代码示例
public static <T> Set<T> removeDuplicateBySet(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new HashSet<>();
    }
    return new HashSet<>(data);
}

// List 去重代码示例
public static <T> List<T> removeDuplicateByList(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new ArrayList<>();

    }
    List<T> result = new ArrayList<>(data.size());
    for (T current : data) {
        if (!result.contains(current)) {
            result.add(current);
        }
    }
    return result;
}
```

两者的核心差别在于contains()方法的实现。

HashSet的contains()方法底部依赖的HashMap的containsKey()方法，时间复杂度接近于 O（1）（没有出现哈希冲突的时候为 O（1））。

```java
private transient HashMap<E,Object> map;
public boolean contains(Object o) {
return map.containsKey(o);
}
```

我们有 N 个元素插入进 Set 中，那时间复杂度就接近是 O (n)。

ArrayList的contains()方法是通过遍历所有元素的方法来做的，时间复杂度接近是 O(n)。

```java
public boolean contains(Object o) {
return indexOf(o) >= 0;
}
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
        if (elementData[i]==null)
            return i;
    } else {
        for (int i = 0; i < size; i++)
        if (o.equals(elementData[i]))
            return i;
    }
    return -1;
}
```

## 集合转数组
《阿里巴巴 Java 开发手册》的描述如下：

**使用集合转数组的方法，必须使用集合的****toArray(T[] array)****，传入的是类型完全一致、长度为 0 的空数组。**

toArray(T[] array)方法的参数是一个泛型数组，如果toArray方法中没有传递任何参数的话返回的是Object类 型数组。

```java
String [] s= new String[]{
    "dog", "lazy", "a", "over", "jumps", "fox", "brown", "quick", "A"
};
List<String> list = Arrays.asList(s);
Collections.reverse(list);
//没有指定类型的话会报错
s=list.toArray(new String[0]);
```

由于 JVM 优化，new String[0]作为Collection.toArray()方法的参数现在使用更好，new String[0]就是起一个模板的作用，指定了返回数组的类型，0 是为了节省空间，因为它只是为了说明返回的类型。详见：[https://shipilev.net/blog/2016/arrays-wisdom-ancients](https://shipilev.net/blog/2016/arrays-wisdom-ancients/)

## 数组转集合
《阿里巴巴 Java 开发手册》的描述如下：

*_使用工具类__*__Arrays.asList()__**把数组转换成集合时，不能使用其修改集合相关的方法， 它的**__add/remove/clear__*__方法会抛出__**UnsupportedOperationException**__异常。_*

我在之前的一个项目中就遇到一个类似的坑。

Arrays.asList()在平时开发中还是比较常见的，我们可以使用它将一个数组转换为一个List集合。

```java
String[] myArray = {"Apple", "Banana", "Orange"};
List<String> myList = Arrays.asList(myArray);
//上面两个语句等价于下面一条语句
List<String> myList = Arrays.asList("Apple","Banana", "Orange");
```

JDK 源码对于这个方法的说明：

```java
/**
  *返回由指定数组支持的固定大小的列表。此方法作为基于数组和基于集合的API之间的桥梁，
  * 与 Collection.toArray()结合使用。返回的List是可序列化并实现RandomAccess接口。
  */
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

下面我们来总结一下使用注意事项。

**1、****Arrays.asList()****是泛型方法，传递的数组必须是对象数组，而不是基本类型。**

```java
int[] myArray = {1, 2, 3};
List myList = Arrays.asList(myArray);
System.out.println(myList.size());//1
System.out.println(myList.get(0));//数组地址值
System.out.println(myList.get(1));//报错：ArrayIndexOutOfBoundsException
int[] array = (int[]) myList.get(0);
System.out.println(array[0]);//1
```

当传入一个原生数据类型数组时，Arrays.asList()的真正得到的参数就不是数组中的元素，而是数组对象本身！此时List的唯一元素就是这个数组，这也就解释了上面的代码。

我们使用包装类型数组就可以解决这个问题。

```java
Integer[] myArray = {1, 2, 3};
```

**2、使用集合的修改方法:**add()**、**remove()**、****clear()****会抛出异常。**

```java
List myList = Arrays.asList(1, 2, 3);
myList.add(4);//运行时报错：UnsupportedOperationException
myList.remove(1);//运行时报错：UnsupportedOperationException
myList.clear();//运行时报错：UnsupportedOperationException
```

Arrays.asList()方法返回的并不是java.util.ArrayList，而是java.util.Arrays的一个内部类,这个内部类并没有实现集合的修改方法或者说并没有重写这些方法。

```java
List myList = Arrays.asList(1, 2, 3);
System.out.println(myList.getClass());//class java.util.Arrays$ArrayList
```

下图是java.util.Arrays$ArrayList的简易源码，我们可以看到这个类重写的方法有哪些。

```java
private static class ArrayList<E> extends AbstractList<E>
implements RandomAccess, java.io.Serializable
{
    ...

    @Override
    public E get(int index) {
        ...
    }

    @Override
    public E set(int index, E element) {
        ...
    }

    @Override
    public int indexOf(Object o) {
        ...
    }

    @Override
    public boolean contains(Object o) {
        ...
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        ...
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        ...
    }

    @Override
    public void sort(Comparator<? super E> c) {
        ...
    }
}
```

我们再看一下java.util.AbstractList的add/remove/clear方法就知道为什么会抛出UnsupportedOperationException了。

```java
public E remove(int index) {
throw new UnsupportedOperationException();
}
public boolean add(E e) {
    add(size(), e);
    return true;
}
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}

public void clear() {
    removeRange(0, size());
}
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

**那我们如何正确的将数组转换为******ArrayList**?**

1、手动实现工具类

```java
//JDK1.5+
static <T> List<T> arrayToList(final T[] array) {
    final List<T> l = new ArrayList<T>(array.length);

    for (final T s : array) {
        l.add(s);
    }
    return l;
}


Integer [] myArray = { 1, 2, 3 };
System.out.println(arrayToList(myArray).getClass());//class java.util.ArrayList
```

2、最简便的方法

```java
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```

3、使用 Java8 的Stream(推荐)

```java
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
//基本类型也可以实现转换（依赖boxed的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

4、使用 Guava

对于不可变集合，你可以使用[ImmutableList](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java)类及其[of()](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L101)与[copyOf()](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L225)工厂方法：（参数不能为空）

```java
List<String> il = ImmutableList.of("string", "elements");  // from varargs
List<String> il = ImmutableList.copyOf(aStringArray);      // from array
```

对于可变集合，你可以使用[Lists](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java)类及其[newArrayList()](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java#L87)工厂方法：

```java
List<String> l1 = Lists.newArrayList(anotherListOrCollection);    // from collection
List<String> l2 = Lists.newArrayList(aStringArray);               // from array
List<String> l3 = Lists.newArrayList("or", "string", "elements"); // from varargs
```

5、使用 Apache Commons Collections

```java
List<String> list = new ArrayList<String>();
CollectionUtils.addAll(list, str);
```

6、 使用 Java9 的List.of()方法

```java
Integer[] array = {1, 2, 3};
List<Integer> list = List.of(array);
```



> 更新: 2024-01-15 01:37:51  
原文: [https://www.yuque.com/vip6688/neho4x/aras7gazp8489vl6](https://www.yuque.com/vip6688/neho4x/aras7gazp8489vl6)
>



> 更新: 2024-11-25 10:06:26  
> 原文: <https://www.yuque.com/neumx/laxg2e/9b0f37e143c8cddda5e2ee1df3ba5b98>