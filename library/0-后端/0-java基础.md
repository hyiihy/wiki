# <center>Java基础</center>

## Integer的缓存
```Java
public static void main(String[] args){
  Integer i1 = 100;
  Integer i2 = 100;
  System.out.println(i1 == i2);//true

  Integer i3 = 1000;
  Integer i4 = 1000;
  System.out.println(i3 == i4);//false
}
```
解释：  
Java在编译时，会对`Integer i1 = 100;`进行自动装箱操作，即相当于`Integer i1 = Integer.valueOf(100);`  
value方法源码为：  
```Java
public static Integer valueOf(int i) {
       if (i >= IntegerCache.low && i <= IntegerCache.high)
           return IntegerCache.cache[i + (-IntegerCache.low)];
       return new Integer(i);
   }
```
可以看到当i在`IntegerCache.low`到`IntegerCache.high`之间时，直接从`IntegerCache.cache`中返回缓存对象。  
其中`IntegerCache`为`Integer`的内部静态类，内有一个Integer数组`Integer[] catch`,在Integer类被加载时执行内部静态类的静态方法，构造值为`IntegerCache.low`到`IntegerCache.high`之间的Integer对象，存入数组中。其中`IntegerCache.low`和`IntegerCache.high`的值默认为-128到127，可以通过修改JVM的配置参数修改它们。
IntegerCache类源码如下：
```Java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
因此，`Integer i1 = 100;`和`Integer i2 = 100;`中，i1和i2均指向缓存中的同一个对象，`i1 == i2`为true;而值为1000的整型对象不在缓存中，`Integer i3 = 1000;`和`Integer i4 = 1000;`会在堆内存中分别新建两个实例，i3和i4的引用指向不同的地址。

## String的缓存
```Java
public static void main(String[] args){
    String s1 = "abc";
    String s2 = "abc";
    System.out.println(s1 == s2);//true

    String s3 = new String("abc");
    String s4 = new String("abc");
    System.out.println(s3 == s4);//false
}
```
解释：  
创建String类型的对象时，如果采用`"abc"`的方式去创建，则JVM会先判断内存中是否有该字符串，有则直接返回，没有则新建该字符串并放置于方法区的常量池中，然后返回。因此s1和s2的引用指向同一个地址。  
而采用`new String()` 的方式去创建时，JVM直接在堆内存中开辟空间新建对象，因此s3和s4指向不同的地址。

*
注意`==`与`equals()`的区别:  
==为比较两对象的地址是否相同。  
而equals()则是一个普通的方法，在Object中的equals默认实现为比较两对象的地址是否相同。  
一般有特殊需求判断两者是否相等则要改写equals()方法。  
附上Integer和String中重写的equals()方法：  
*
```Java
@Override
public boolean equals(Object obj) {
       if (obj instanceof Integer) {
           return value == ((Integer)obj).intValue();
       }
       return false;
   }
```
```Java
@Override
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
*Java规范要求，一旦复写了equals()方法，必须同时复写hashCode()方法，按下不表*

## 自动装箱、拆箱
**8种基本数据类型都有对应的引用类型：**  
- 第一类：整型
  - byte Byte
  - short Short
  - int Integer
  - long Long


- 第二类：浮点型
  - float Float
  - double Double


- 第三类：逻辑型
  - boolean Boolean


- 第四类：字符型
  - char Character

**引入引用类型的好处：**
- 空值可以用null存储
- 为泛型提供支持
- 提供常用的属性和API

**装拆箱方法：**
- 装箱：valueOf()
- 拆箱：xxxValue()

*注意Integer中的parseInt()方法和Long中的parseLong()方法，是传入String类型将字符串转为Integer/Long*

## equals()和hashCode()  

**java语言规范要求equals方法具有下面的特性：**
- 自反性:对于任何非空引用x,x.equals(x)应该返回true;
- 对称性:对于任何引用x,和y,当且仅当,y.equals(x)返回true,x.equals(y)也应该返回true;
- 传递性:对于任何引用x,y,z,如果x.equals(y)返回true,y.equals(z)返回true,那么x.equals(z)也应该返回true;
- 一致性:如果x,y引用的对象没有发生变化,反复调用x.equals(y)应该返回同样的结果;
- 对于任意非空引用x,x.equals(null)返回false;

Object中的equals():
```Java
public boolean equals(Object obj) {
        return (this == obj);
    }
```
要复写equals()方法时，应注意equals传入的参数是Object。  
若传入的参数不是Object类，加@Override注解会报错。（@Override的作用正是在于此）

**java语言规范要求hashCode()方法具有下面的特性：**  
- 重写了equals()方法的对象一定要重写hashCode()方法。
- 如果两个对象的equals()方法返回true,则它们的hashCode()方法返回的值相同。
- 如果两个对象的equals()方法返回false,则他们的hashCode()方法返回的值可能相同。(当然，hash冲突越多，hash集合的效率越低，因此hashCode()方法的设计很关键)

Object的hashCode()方法为native方法，源码略。  
hashCode()方法返回int类型的散列码。  
hashCode()方法是为了更好地支持基于散列的集合类，如Hashtable, HashMap, HashSet 等。  

*疑问：
既然hashCode()是为了满足hash集合，为何不要求传入hash集合的元素必须实现一个hashable接口，这样就能保证传入的元素一定有复写*

在ArrayList中，contains()方法用到了equals()方法，源码如下：
```Java
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
HashMap中，containsKey()方法用到了equals()方法和hashCode()方法，源码如下：
```Java
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
HashMap中，containsValue()方法用到了equals()方法，源码如下：
```Java
public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    if ((tab = table) != null && size > 0) {
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```
HashSet底层实现是HashMap，它的contains()方法调用的是HashMap的containsKey()方法，因此也用到了equals()方法和hashCode()方法,源码如下：
```Java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

## ArrayList
ArrayList底层为数组，对应于数据结构为顺序表。  
构造方法的源码如下：
```Java
//无参
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//有参
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

transient Object[] elementData; // non-private to simplify nested class access

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```
可以看到，ArrayList在初始化时，其内部的数组Object[]，都是指常量池中的一个空数组。

add()方法的源码如下：
```Java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```
其中ensureCapacityInternal()是为了确定数组的大小，以及modCount自增。源码略，modCount记录修改次数，功能按下不表。  
ensureCapacityInternal()内部还使用了`System.arraycopy()`方法，用来复制数组。

*ArrayList与数组的异同：  
ArrayList底层依赖数组实现。  
ArrayList可扩容（1.5倍），数组大小固定。*

## Java集合类
![集合类UML图](assets/0/v2-76c3c04de2e8609c488fa0081fb99c26_hd.jpg)  
