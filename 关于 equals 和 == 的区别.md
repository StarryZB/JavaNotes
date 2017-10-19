# 关于 equals 和 == 的区别

## 关于 equals  

###Object 中的 equals 是比较是否相等，也就是比较对象的地址是否相同

equals 是根类 Object 中的一个方法。源码如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

可以看出，当参数 obj 引用的对象与当前对象为同一个对象（同一个地址）时，就返回 true ，否则返回 false 。所以，一般来说， equals 方法都需要我们去重写。

```java
public class EqualsTest {
	public static void main(String[] args) {
		Object obj1 = new Object();
		Object obj2 = new Object();
		Object obj3 = new Object();
		Object obj4 = new Object();
		obj3 = obj4;	
		System.out.println("obj1 和 obj2 使用 equals 比较，返回" + obj1.equals(obj2));
		System.out.println("obj3 和 obj4 使用 equals 比较，返回" + obj3.equals(obj4));
	}
}
```

输出结果：

```java
obj1 和 obj2 使用 equals 比较，返回false
obj3 和 obj4 使用 equals 比较，返回true
```

### String 类就重写了 equals 方法

源码如下：

```java
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

可以看出

1. String 类中首先是比较地址，如果引用对象相同，返回 true 。

2. 如果引用对象不相同， equals 一个一个去比较两个字符串对象里面的字符，只有完全相同，才会返回 true ，否则返回 false 。

   ```java
   public class StringEqualsTest {
   	public static void main(String[] args) {
   		String str1 = new String("str");
   		String str2 = new String("str");
   		String str3 = "str";	
   		System.out.println("str1 和 str2 使用 equals 比较，返回" + str1.equals(str2));
   		System.out.println("str1 和 str3 使用 equals 比较，返回" + str1.equals(str3));		
   	}
   }
   ```

   输出结果：

   ```java
   str1 和 str2 使用 equals 比较，返回true
   str1 和 str3 使用 equals 比较，返回true
   ```

JDK 中有许多类都已经重写好 equals 方法，例如 Integer 类等等，但是如果是其他没重写过 equals 方法的，比如我们自己建的类 ，需要实现比较功能，这时候我们就要自己去重写这个方法。

## 关于 ==

在Java里，除了 boolean , byte , char , short , int , long , float , double 这8种基本类型外，其它类型都是引用类型。引用类型的值是指向对象的引用。

###对于基本类型来说，就是单纯地比较的是两个值是否相同

```java
public class CompareTest {
	public static void main(String[] args) {
		int a = 1;
		int b = 1;
		int c = 2;		
		System.out.println("a 和 b 使用 == 比较，返回" + (a == b));
		System.out.println("a 和 c 使用 == 比较，返回" + (a == c));
	}
}
```

输出结果：

```java
a 和 b 使用 == 比较，返回true
a 和 c 使用 == 比较，返回false
```

### 对于引用类型来说，是比较两者的地址是否相同 

这点跟在 Object 中的 equals 方法作用是相同的。

```java
public class CompareTest {
	public static void main(String[] args) {
		Object obj1 = new Object();
		Object obj2 = new Object();
		Object obj3 = new Object();
		Object obj4 = new Object();
		obj3 = obj4;	
		System.out.println("obj1 和 obj2 使用 == 比较，返回" + (obj1 == obj2));
		System.out.println("obj3 和 obj4 使用 == 比较，返回" + (obj3 == obj4));
	}
}
```

输出结果：

```java
obj1 和 obj2 使用 == 比较，返回false
obj3 和 obj4 使用 == 比较，返回true
```

## 用 String 举个例子

两者的基本功能都讲清楚后，用经典的 String 类来举个例子，也是很多人头晕的地方。

```java
public class StringTest {
	public static void main(String[] args) {
		String str1 = new String("str");
		String str2 = new String("str");
		String str3 = "str";	
		System.out.println("str1 和 str2 使用 equals 比较，返回" + str1.equals(str2));
		System.out.println("str1 和 str3 使用 equals 比较，返回" + str1.equals(str3));
		System.out.println("str1 和 str2 使用 == 比较，返回" + (str1 == str2));
		System.out.println("str1 和 str3 使用 == 比较，返回" + (str1 == str3));		
	}
}
```

输出结果：

```java
str1 和 str2 使用 equals 比较，返回true
str1 和 str3 使用 equals 比较，返回true
str1 和 str2 使用 == 比较，返回false
str1 和 str3 使用 == 比较，返回false
```

分析：首先， String 是一个类，属于引用类型，它的 equals 方法已经重写过的了，如果两者内容相同，会返回 true ，所以不管地址如何，只要里面内容相同，就返回 true 。因此，前两句，三者内容都是"str"，用 equals 比较，都会返回 true 。然后接下来两句， str1 和 str2 的 String 对象是分别存放在堆中的两块地址，尽管里面存放的内容相同，但是地址不相同，所以用 == 比较会返回 false 。str3 则是存放在 String 池里面的，所以和堆中的 str1 地址也不相同，所以用 == 比较也会返回 false 。所以，内容相同，地址不一定相同， == 比较可能会返回 false 。但是反过来，地址相同，内容一定是相同的。