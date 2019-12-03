---
title: 说一说Array转List
author: Sophosss
layout: post
---
List 提供了toArray的接口，直接调用转为object型数组，也可以指定类型转换。

```java
List<String> list = new ArrayList<String>();
Object[] array=list.toArray();
String[] array=list.toArray(new String[list.size()]);
```

Array自身也有asList方法。

```java
String[] array = {"java", "c"};
List<String> list = Arrays.asList(array);
```

但该方法存在一定的弊端，返回的list是Arrays里面的一个静态内部类，该类并未实现add,remove方法，因此在使用时存在局限性。

```
public static <T> List<T> asList(T... a) {
//  注意该ArrayList并非java.util.ArrayList
//  java.util.Arrays.ArrayList.ArrayList<T>(T[])
    return new ArrayList<>(a);
}
```

所以我们该怎么做最好呢，这里提供两种方式。

- 运用ArrayList的构造方法，代码简洁，效率高。

```java
List<String> list = new ArrayList<String>(Arrays.asList(array));

//ArrayList构造方法源码
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    size = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

- 使用Collections的addAll方法。

```java
List<String> list = new ArrayList<String>(array.length);
Collections.addAll(list, array);
```

