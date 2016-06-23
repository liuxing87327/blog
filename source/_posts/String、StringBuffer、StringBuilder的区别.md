title: "String、StringBuffer、StringBuilder的区别"
date: 2016-03-15 03:14:00
category: Java
tags: [String,StringBuffer,StringBuilder]

---

使用java有很长一段时间了，一直都是用的一些框架。
还从未深入思考过一些java基础的东西
写代码时大家常说，字符串拼接不要用String，要用StringBuffer、StringBuilder，
今天写篇文字总结一下String、StringBuffer、StringBuilder的区别。
本文不深入探讨jvm的机制（本人这块比较渣），有建议欢迎指点讨论学习，十分感谢！

## 前戏

- 被final修饰的类是不能被继承的，没有子类
- 被final修饰的对象，其引用不能改变，但是对象中的属性值可以修改(String不行哦，编译都不通过)
- 一个线程访问一个对象中的synchronized(this)同步代码块时，其他试图访问该对象的线程将被阻塞
- 基本类型的变量和引用变量都是在函数的栈内存中分配，堆中存放对象和数组



## 代码结构

### String

#### 源码分析
```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    
    // 数组的被final修饰，所以数据引用变量的值不能变
    private final char value[];
    
    ...
    
    public String replace(char oldChar, char newChar) {
        if (oldChar != newChar) {
            int len = value.length;
            int i = -1;
            char[] val = value; /* avoid getfield opcode */

            while (++i < len) {
                if (val[i] == oldChar) {
                    break;
                }
            }
            if (i < len) {
                char buf[] = new char[len];
                for (int j = 0; j < i; j++) {
                    buf[j] = val[j];
                }
                while (i < len) {
                    char c = val[i];
                    buf[i] = (c == oldChar) ? newChar : c;
                    i++;
                }
                return new String(buf, true);
            }
        }
        return this;
    }
    
    // replaceAll采用正则匹配
    public String replaceAll(String regex, String replacement) {
        return Pattern.compile(regex).matcher(this).replaceAll(replacement);
    }
    
    ...
}

```

String的底层是使用字符数组来实现的，
value是一个被final修饰的数组对象，所以只能说他不能再引用到其他对象而不能说明他所引用的对象的内容不能改变。
但继续看源码就会发现String类没有给这两个成员变量提供任何的方法所以我们也没办法修改所引用对象的内容，
所以String对象一旦被创建，这个变量被初始化后就不能再修改了，所以说String对象是不可变对象。

String的replace方法也并没有修改本身，而是重复创建了新的对象。

#### 内存分配

##### 常量池

Java中字符串对象创建有两种形式，一种为字面量形式

> String str = "lianjia";

另一种就是使用new这种标准的构造对象的方法，

> String str = new String("lianjia");

这两种方式我们在代码时都经常使用，尤其是字面量的方式。
然而这两种实现其实存在着一些性能和内存占用的差别。
这一切都是源于JVM为了减少字符串对象的重复创建，其维护了一个特殊的内存，这段内存被成为字符串常量池或者字符串字面量池。

**工作原理**

当代码中出现字面量形式创建字符串对象时，JVM首先会对这个字面量进行检查，
如果字符串常量池中存在相同内容的字符串对象的引用，则将这个引用返回，
否则新的字符串对象被创建，然后将这个引用放入字符串常量池，并返回该引用。

字面量形式

> String str1 = "lianjia";

JVM检测这个字面量，这里我们认为没有内容为lianjia的对象存在。
JVM通过字符串常量池查找不到内容为lianjia的字符串对象存在，那么会创建这个字符串对象，
然后将刚创建的对象的引用放入到字符串常量池中,并且将引用返回给变量str1。

> String str2 = "lianjia";

同样JVM还是要检测这个字面量，JVM通过查找字符串常量池，
发现内容为"lianjia"字符串对象存在，于是将已经存在的字符串对象的引用返回给变量str2。
注意这里不会重新创建新的字符串对象。

验证是否为str1和str2是否指向同一对象，可以通过这段代码

> System.out.println(str1 == str2); // true

使用new创建

> String str3 = new String("lianjia");

当我们使用了new来构造字符串对象的时候，不管字符串常量池中有没有相同内容的对象的引用，新的字符串对象都会创建。

> System.out.println(str1 == str3); // false，两个变量指向的为不同的对象


关于常量池的更多信息，请谷歌”Java class 文件结构 常量池“等关键字。


![内存分配](/images/uml/Snip20160315_2.png)


### StringBuffer、StringBuilder
#### 源码分析
![String、StringBuffer、StringBuilder类图对比](/images/uml/AbstractStringBuilder.png)

`查看类图方法：选中包或类 - 右键 - Diagrams` 神马，你是eclipse，滚粗...

可以通过“Show Categories - Methods”查看具体的方法，因图片太大，就不展示了 [点我下载](/images/uml/AbstractStringBuilder-Methods.png) 

从类结构可以看出，StringBuffer和StringBuilder是典型的模板模式。
所有的通用方法都在AbstractStringBuilder的模板类中，子类只是进行了差异化处理。

代码片段

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {

    // 没有被final修饰，所以引用变量的值可以改变
    char[] value;
    
    // 字符的个数
    int count;
    
    ...
    
    AbstractStringBuilder(int capacity) {
       // 构造函数，创建数组对象，实现类默认是16个长度
        value = new char[capacity];
    }
        
    // 拼接字符
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        // 获取拼接内容长度
        int len = str.length();
        // 扩展存储数组长度
        ensureCapacityInternal(count + len);
        // 把拼接数据拷贝到源数组
        str.getChars(0, len, value, count);
        // 更新数组长度
        count += len;
        return this;
    }
    
    // 确保数组长度够用
    private void ensureCapacityInternal(int minimumCapacity) {
        if (minimumCapacity - value.length > 0)
            expandCapacity(minimumCapacity);
    }
    
    // 扩展存储数组长度
    void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        if (newCapacity < 0) {
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            newCapacity = Integer.MAX_VALUE;
        }
        value = Arrays.copyOf(value, newCapacity);
    }
    
    ...
}
```

通过扩展数组长度的方法可以看出，当数组长度不够时，每次都是扩展了两倍的长度，
所以一般建议预估一下结果最终的长度，避免做不必要的事。

StringBuffer、StringBuilder的功能大同小异，区别是StringBuffer的方法都加了同步关键字，以保证线程安全

## 线程安全

### 测试代码

```java
import com.dooioo.commons.Randoms;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * 测试String、StringBuffer、StringBuilder的线程安全
 * 500组不重复字符串，100个线程进行拼接操作，看最终结果长度是否是500*32*100
 *
 * @author ：liuxing
 * @since ：2016-03-15 01:55
 */
public class AbstractStringBuilderTest {

    int threadCount = 100;
    ExecutorService executor;
    List<String> testData = new ArrayList<>();

    /**
     * 数据准备
     * 500个不重复的字符串
     */
    @Before
    public void after() {
        executor = Executors.newCachedThreadPool();
        for (int i = 0; i < 500; i++) {
            // 生成32位的随机数，防止使用字符串池
            testData.add(Randoms.getPrimaryKey());
        }
    }

    /**
     * 测试String
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    @Test
    public void testString() throws ExecutionException, InterruptedException {
        // 编译不会通过
//        final String str = "";
//
//        List<Future> futures = new ArrayList<>();
//
//        for (int i = 0; i < threadCount; i++) {
//            futures.add(executor.submit(() -> {
//                for (String s : testData) {
//                    str += s;
//                }
//                return true;
//            }));
//        }
//
//        for (Future future : futures) {
//            future.get();
//        }
//
//        Assert.assertEquals("String线程不安全", 500*32*100, str.length());
    }

    /**
     * 测试StringBuffer
     *
     * @throws ExecutionException
     * @throws InterruptedException
     * @see AbstractStringBuilder#append(String)
     */
    @Test
    public void testStringBuffer() throws ExecutionException, InterruptedException {
        StringBuffer buffer = new StringBuffer();

        List<Future> futures = new ArrayList<>();

        for (int i = 0; i < threadCount; i++) {
            futures.add(executor.submit(() -> {
                testData.forEach(s -> {
                    try {
                        buffer.append(s);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                });

                return true;
            }));
        }

        for (Future future : futures) {
            future.get();
        }

        Assert.assertEquals("StringBuffer线程不安全", 500 * 32 * 100, buffer.toString().length());
    }

    /**
     * 测试StringBuilder
     * 会出现下标越界和字符拼接丢失
     * 查看StringBuilder的源码，它内部自动扩展字符数组时是先确定新数组长度，再拷贝旧数据。
     * 极端情况：当a线程中一次append正在进行时，确定了新长度后，线程切换，另一个线程b写入了较短的字符串，
     * 但还没更新内部count计数，于是就在数组后面留下空白；然后a切回来，拷贝原有的数据（实即b写入的短字符串加上末尾空格），
     * 然后a将新数据append到数组中，实际上就排在了空白后面。
     *
     * @throws ExecutionException
     * @throws InterruptedException
     * @see AbstractStringBuilder#append(String)
     */
    @Test
    public void testStringBuilder() throws ExecutionException, InterruptedException {
        StringBuilder builder = new StringBuilder();

        List<Future> futures = new ArrayList<>();

        for (int i = 0; i < threadCount; i++) {
            futures.add(executor.submit(() -> {
                testData.forEach(s -> {
                    try {
                        builder.append(s);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                });

                return true;
            }));
        }

        for (Future future : futures) {
            future.get();
        }


        Assert.assertEquals("StringBuilder线程不安全", 500 * 32 * 100, builder.toString().length());
    }

    /**
     * 测试同步的StringBuilder
     *
     * @throws ExecutionException
     * @throws InterruptedException
     * @see AbstractStringBuilder#append(String)
     */
    @Test
    public void testSyncStringBuilder() throws ExecutionException, InterruptedException {
        StringBuilder builder = new StringBuilder();

        List<Future> futures = new ArrayList<>();

        for (int i = 0; i < threadCount; i++) {
            futures.add(executor.submit(() -> {
                testData.forEach(s -> {
                    try {
                        synchronized (builder) {
                            builder.append(s);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                });

                return true;
            }));
        }

        for (Future future : futures) {
            future.get();
        }

        Assert.assertEquals("StringBuilder线程不安全，即使加了synchronized", 500 * 32 * 100, builder.toString().length());
    }

    /**
     * 简单比较String、StringBuffer、StringBuilder在赋值后是否对象引用会改变
     */
    @Test
    public void testHashCode() {
        String str = "000";
        System.out.println(str.hashCode());
        str += "123";
        System.out.println(str.hashCode());

        StringBuffer buffer = new StringBuffer("000");
        System.out.println(buffer.hashCode());
        buffer.append("123");
        System.out.println(buffer.hashCode());

        StringBuilder builder = new StringBuilder("000");
        System.out.println(builder.hashCode());
        builder.append("123");
        System.out.println(builder.hashCode());
    }

}

```

### 结果

StringBuffer是线程安全的

StringBuilder多线程会抛出异常，字符串长度不准确，线程不安全

对StringBuilder对象加上synchronized能制造线程安全

```java
java.lang.ArrayIndexOutOfBoundsException
	at java.lang.String.getChars(String.java:814)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:422)
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at com.dooioo.lang.AbstractStringBuilderTest.lambda$null$2(AbstractStringBuilderTest.java:124)
	at com.dooioo.lang.AbstractStringBuilderTest$$Lambda$3/138527898.accept(Unknown Source)
	at java.util.ArrayList.forEach(ArrayList.java:1249)
	at com.dooioo.lang.AbstractStringBuilderTest.lambda$testStringBuilder$3(AbstractStringBuilderTest.java:122)
	at com.dooioo.lang.AbstractStringBuilderTest$$Lambda$1/411631404.call(Unknown Source)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

java.lang.AssertionError: StringBuilder线程不安全 
Expected :1600000
Actual   :1450240
 <Click to see difference>


	at org.junit.Assert.fail(Assert.java:88)
	at org.junit.Assert.failNotEquals(Assert.java:834)
	at org.junit.Assert.assertEquals(Assert.java:645)
	at com.dooioo.lang.AbstractStringBuilderTest.testStringBuilder(AbstractStringBuilderTest.java:139)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:119)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:42)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:234)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:74)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)

```

## 效率
结合上面的分析，可以看出String使用”+“拼接字符，每次都会开辟新的内存空间，然后修改变量的指向，同时也会频繁触发GC。
那么我们来测试一下具体性能差异有多少。

```java
import org.junit.Test;

/**
 * StringContactTest
 *
 * @author ：liuxing
 * @since ：2016-03-15 06:14
 */
public class StringContactTest {

    int COUNT = 10;

    @Test
    public void testContact() {
        long start = System.nanoTime();

        String s  = "";

        for (int i = 0; i < COUNT; i++) {
            s += i;
        }

        long end = System.nanoTime();

        System.out.println("String耗时: " + (end - start));
    }

    @Test
    public void testBufferContact() {
        long start = System.nanoTime();

        StringBuffer buffer = new StringBuffer();

        for (int i = 0; i < COUNT; i++) {
            buffer.append(i);
        }

        // 需要注意，StringBuffer的toString和StringBuilder的toString有点不一样哦
        buffer.toString();

        long end = System.nanoTime();

        System.out.println("StringBuffer耗时: " + (end - start));
    }

    @Test
    public void testBuilderContact() {
        long start = System.nanoTime();

        StringBuilder builder = new StringBuilder();

        for (int i = 0; i < COUNT; i++) {
            builder.append(i);
        }

        builder.toString();

        long end = System.nanoTime();

        System.out.println("StringBuilder耗时: " + (end - start));
    }

}

```

分5、50、500、5000、50000次拼接测试

5
```bash
StringBuilder耗时: 53216
String耗时: 10565
StringBuffer耗时: 37100

```

50
```bash
StringBuilder耗时: 77982
String耗时: 65830
StringBuffer耗时: 80627
```

500
```bash
StringBuilder耗时: 1672791
String耗时: 5102420
StringBuffer耗时: 1405787
```

5000
```bash
StringBuilder耗时: 17866567
String耗时: 166399935
StringBuffer耗时: 9257570
```

50000
```bash
StringBuilder耗时: 33779794
String耗时: 8160426817
StringBuffer耗时: 3683921
```


小数据量用哪种都无所谓，String甚至更快，数据量大的时候就不要用String了，本来还要测试500000+，但是太慢了，就没测试了

## 延伸知识
分析上面东东时，使用的一些还未深入测试的知识点

### 查看java字节码
1.javac –verbose查看运行类加载的jar
    
    javac –verbose Test.java
2.javap查看字节码
    
    javap –c Test
    javap –verbose Test
    

查看字节码对比String的字面量形式和new创建的差异，待定！
    
### 查看jvm的图形工具
常用的jvm图形分析工具，待定！



    
