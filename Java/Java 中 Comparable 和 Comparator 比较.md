[toc]

# Comparable
**Comparable 是排序接口。**

若一个类实现了Comparable接口，就意味着“**该类支持排序**”。  即然实现Comparable接口的类支持排序，假设现在存在“实现Comparable接口的类的对象的List列表(或数组)”，则该List列表(或数组)可以通过 Collections.sort（或 Arrays.sort）进行排序。

此外，“实现Comparable接口的类的对象”可以用作“有序映射(如TreeMap)”中的键或“有序集合(TreeSet)”中的元素，而不需要指定比较器。

## Comparable 定义
Comparable 接口仅仅只包括一个函数，它的定义如下：

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

假设我们通过 `x.compareTo(y)` 来“比较x和y的大小”。若返回“负数”，意味着“x比y小”；返回“零”，意味着“x等于y”；返回“正数”，意味着“x大于y”。

# Comparator
**Comparator 是比较器接口。**

我们若需要控制某个类的次序，而该类本身不支持排序(即没有实现Comparable接口)；那么，我们可以建立一个“该类的比较器”来进行排序。这个“比较器”只需要实现Comparator接口即可。

也就是说，我们可以通过“**实现Comparator类来新建一个比较器**”，然后通过该比较器对类进行排序。

## Comparator 定义
Comparator 接口仅仅只包括两个个函数，它的定义如下：

```java
public interface Comparator<T> {

    int compare(T o1, T o2);

    boolean equals(Object obj);
}
```

- 若一个类要实现Comparator接口：它一定要实现compare(T o1, T o2) 函数，但可以不实现 equals(Object obj) 函数。
- int compare(T o1, T o2) 是“比较o1和o2的大小”。返回“负数”，意味着“o1比o2小”；返回“零”，意味着“o1等于o2”；返回“正数”，意味着“o1大于o2”。

**为什么可以不实现 equals(Object obj) 函数呢？**    
**因为任何类，默认都是已经实现了equals(Object obj)的。** Java中的一切类都是继承于java.lang.Object，在Object.java中实现了equals(Object obj)函数；所以，其它所有的类也相当于都实现了该函数。

# Comparator 和 Comparable 相同的地方
**他们都是java的一个接口, 并且是用来对自定义的class比较大小的**。

什么是自定义class，如：

```
public class Person{ 
    String name; 
    int age; 
}
```

当我们有这么一个`personList`，里面包含了`person1, person2, persion3.....`， 我们用`Collections.sort( personList )`，是得不到预期的结果的。

这时肯定有人要问, 那为什么可以排序一个字符串list呢？如`StringList{"hello1" , "hello3" , "hello2"}`, `Collections.sort( stringList )` 能够得到正确的排序, 那是因为**String 这个对象已经帮我们实现了 Comparable接口**，所以我们的 Person 如果想排序, 也要实现一个比较器。

# Comparator 和 Comparable 不同的地方
## Comparable
**Comparable 定义在 Person类的内部**:

```java
public class Persion implements Comparable<T> {
    ..比较Person的大小..
}
```

因为已经实现了比较器，那么我们的Person现在是一个可以比较大小的对象了，它的比较功能和String完全一样，可以随时随地的拿来
比较大小，因为Person现在自身就是有大小之分的。`Collections.sort(personList)`可以得到正确的结果。

## Comparator
**Comparator 是定义在Person的外部的**，此时我们的Person类的结构不需要有任何变化，如：

```java
public class Person{ 
    String name; 
    int age;
}

public PersonComparator implements Comparator<T> {
    ..比较Person的大小..
}
```

在PersonComparator里面实现了怎么比较两个Person的大小。所以，用这种方法，当我们要对一个 personList进行排序的时候，我们除了要传递personList过去，还需要把PersonComparator传递过去，因为怎么比较Person的大小是在PersonComparator
里面实现的, 如:

```java
Collections.sort(personList, new PersonComparator());
```

# Comparator 和 Comparable 的实例
## Comparable
实现Comparable接口要覆盖compareTo方法, 在compareTo方法里面实现比较：

```java
public class Person implements Comparable {
     String name;
     int age
     public int compareTo(Person another) {
          int i = 0;
          i = name.compareTo(another.name); // 使用字符串的比较
          if(i == 0) { // 如果名字一样,比较年龄, 返回比较年龄结果
               return age - another.age;
          } else {
               return i; // 名字不一样, 返回比较名字的结果.
          }
     }
}
```

这时我们可以直接用 `Collections.sort( personList )` 对其排序了。

## Comparator
实现Comparator需要覆盖 compare 方法：

```java
public class Person{
     String name;
     int age
}

class PersonComparator implements Comparator { 
     public int compare(Person one, Person another) {
          int i = 0;
          i = one.name.compareTo(another.name); // 使用字符串的比较
          if(i == 0) { // 如果名字一样,比较年龄,返回比较年龄结果
               return one.age - another.age;
          } else {
               return i; // 名字不一样, 返回比较名字的结果.
          }
     }
}
```

`Collections.sort( personList , new PersonComparator())` 可以对其排序。

# 总结
- Comparable是排序接口；若一个类实现了Comparable接口，就意味着“该类支持排序”。
- Comparator是比较器；我们若需要控制某个类的次序，可以建立一个“该类的比较器”来进行排序。

**Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”。**

## 参考文献
[Java 中 Comparable 和 Comparator 比较](http://www.cnblogs.com/skywang12345/p/3324788.html)