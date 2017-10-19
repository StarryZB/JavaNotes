# 关于 String 类

## String 直接赋值和 new String 的区别

```java
String str1 = "str";
String str2 = new String("str");
```

第一句，栈中开辟一块空间存放引用 str1 ,然后在 String 池开辟一个空间存放 String 常量"str"， str1 指向这个常量。值得注意的是，如果字符串已经存在池中，就返回池中的实例引用。如果字符串不在池中，就会实例化一个字符串并放到池中，而且，常量池指的是在编译期间就已经被确定了的。第二句，栈中开辟一块空间存放引用 str2 ，然后在堆中开辟一个空间存放 String 对象 "str"，str2 指向堆里面的这个对象，同时，如果常量池里没有这个对象，也会把 "str" 字符串加进常量池，如果有就不加，所以单独第二句的话是产生了两个对象，一个在堆中，一个在常量池。

## String 常量池

两种用法：

1. 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
2. 如果不是用双引号声明的 String 对象，可以使用 String 提供的 intern 方法。 intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。

## 字符串相加问题

```java
public class StringTest {
	public static void main(String[] args) {
		String str1 = "string";
		String str2 = "str";
		String str3 = "ing";
		String str4 = "str" + "ing";
		String str5 = str2 + str3;
		String str6 = str2 + "ing";	
		System.out.println("str1 和 str4 使用 == 比较，返回" + (str1 == str4));	
		System.out.println("str1 和 str5 使用 == 比较，返回" + (str1 == str5));
		System.out.println("str1 和 str6 使用 == 比较，返回" + (str1 == str6));	
	}
}
```

输出结果：

```java
str1 和 str4 使用 == 比较，返回true
str1 和 str5 使用 == 比较，返回false
str1 和 str6 使用 == 比较，返回false
```

分析： 前三句很简单，str1 , str2 和 str3 的值都在 String 池里面，然后各自指向对应的值。然后第四句，就涉及到 + 号了，如果 + 两边都是字面字符常量来说，在编译期就将 + 两边直接拼接合并了，也就是说 str1 和 str4 都是指向常量池中的 "string"，所以 == 返回 true 。然后 str5 和 str6 一起说，其实都是一个原理， + 号两边，只要有一边是变量或者两边都是变量，那么会先 new 出一个  StringBuilder （在 JDK 6.0 之前是 StringBuffer ），然后调用 append 方法来将 + 号两边的字符串拼接起来（底层就是 char 数组的拼接），然后 toString 之后返回给 = 号左边的变量，也就是说，最后得到的是一个 new 出来的字符串，那么这个字符串必然是在堆中了，所以结果就会是 false 了。

## 关于 intern 方法 

此节参考文章：[深入解析String#intern](https://tech.meituan.com/in_depth_understanding_string_intern.html) —— [美团点评团队](https://tech.meituan.com/)

JDK7 之后，常量池发生了许多改变，很多都和之前的 JDK6 不一样，常量池搬到堆区里面了。

源代码

```java
/** 
 * Returns a canonical representation for the string object. 
 * <p> 
 * A pool of strings, initially empty, is maintained privately by the 
 * class <code>String</code>. 
 * <p> 
 * When the intern method is invoked, if the pool already contains a 
 * string equal to this <code>String</code> object as determined by 
 * the {@link #equals(Object)} method, then the string from the pool is 
 * returned. Otherwise, this <code>String</code> object is added to the 
 * pool and a reference to this <code>String</code> object is returned. 
 * <p> 
 * It follows that for any two strings <code>s</code> and <code>t</code>, 
 * <code>s.intern()&nbsp;==&nbsp;t.intern()</code> is <code>true</code> 
 * if and only if <code>s.equals(t)</code> is <code>true</code>. 
 * <p> 
 * All literal strings and string-valued constant expressions are 
 * interned. String literals are defined in section 3.10.5 of the 
 * <cite>The Java&trade; Language Specification</cite>. 
 * 
 * @return  a string that has the same contents as this string, but is 
 *          guaranteed to be from a pool of unique strings. 
 */  
public native String intern();
```

直白翻译：如果常量池已经存在当前字符串，会返回池中的字符串。否则（就是说常量池不存在当前字符串），这个字符串会加入到池中并且一个对这个字符串的引用会被返回。

美团翻译：如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回。

本人理解：如果常量池已经存在当前字符串，会**返回池中的字符串的内容引用，所以地址引用没有发生改变，就是说地址引用仍然指向堆中的对象**。否则（就是说常量池不存在当前字符串），这个字符串会加入到池中并且，**地址引用发生了改变，地址引用直接指向了常量池中的这个字符串**，最后返回这个地址引用。

**以下代码指在 JDK7 的环境下运行**

代码一：

```java
public static void main(String[] args) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
}
```

输出结果：

```java
false true
```

解析：

![](https://tech.meituan.com/img/in_depth_understanding_string_intern/jdk7_1.png)

注：图中绿色线条代表 string 对象的内容指向。 黑色线条代表地址指向。

- 在第一段代码中，先看 s3 和 s4 字符串。`String s3 = new String("1") + new String("1");`，这句代码中现在生成了 2 最终个对象，是字符串常量池中的"11"和 JAVA Heap 中的 s3 引用指向的对象。中间还有2个匿名的`new String("1")`我们不去讨论它们。此时 s3 引用对象内容是"11"，但此时常量池中是没有"11"对象的。
- 接下来`s3.intern();`这一句代码，是将 s3 中的"11"字符串放入 String 常量池中，因为此时常量池中不存在"11"字符串，因此常规做法是跟 jdk6 图中表示的那样，在常量池中生成一个 "11" 的对象，关键点是 jdk7 中常量池不在 Perm 区域了，这块做了调整。常量池中不需要再存储一份对象了，可以直接存储堆中的引用。这份引用指向 s3 引用的对象。 也就是说引用地址是相同的。
- 最后`String s4 = "11";` 这句代码中"11"是显示声明的，因此会直接去常量池中创建，创建的时候发现已经有这个对象了，此时也就是指向 s3 引用对象的一个引用。所以 s4 引用就指向和 s3 一样了。因此最后的比较 `s3 == s4` 是 true。
- 再看 s 和 s2 对象。 `String s = new String("1");` 第一句代码，生成了 2 个对象。常量池中的“1” 和 JAVA Heap 中的字符串对象。`s.intern();` 这一句是 s 对象去常量池中寻找后发现"1" 已经在常量池里了。
- 接下来`String s2 = "1";` 这句代码是生成一个 s2 的引用指向常量池中的“1”对象。 结果就是 s 和 s2 的引用地址明显不同。图中画的很清晰。

代码二：

```java
public static void main(String[] args) {
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();
    System.out.println(s3 == s4);
}
```

输出结果：

```java
false false
```

解析：

![](https://tech.meituan.com/img/in_depth_understanding_string_intern/jdk7_2.png)

- 来看第二段代码，从上边第二幅图中观察。第一段代码和第二段代码的改变就是 `s3.intern();` 的顺序是放在`String s4 = "11";`后了。这样，首先执行`String s4 = "11";`声明 s4 的时候常量池中是不存在"11"对象的，执行完毕后，"11"对象是 s4 声明产生的新对象。然后再执行`s3.intern();`时，常量池中“11”对象已经存在了，因此 s3 和 s4 的引用是不同的。
- 第二段代码中的 s 和 s2 代码中，`s.intern();`，这一句往后放也不会有什么影响了，因为对象池中在执行第一句代码`String s = new String("1");`的时候已经生成"1"对象了。下边的 s2 声明都是直接从常量池中取地址引用的。 s 和 s2 的引用地址是不会相等的。