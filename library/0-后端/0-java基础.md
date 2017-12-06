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
public boolean equals(Object obj) {
       if (obj instanceof Integer) {
           return value == ((Integer)obj).intValue();
       }
       return false;
   }
```
```Java
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
