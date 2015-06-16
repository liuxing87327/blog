title: "Java7 Try-with-resources (TWR)"
date: 2014-9-25 13:45:49
category: 书摘
tags: [TWR, Java7]

---

转自《Java程序员修炼之道》

使用Java7的Try-with-resources来自动关闭资源

这个修改说起来容易，但其实暗藏玄机，最终证明做起来比最初预想的要难。其基本设想是把资源（比如文件或类似的东西）的作用域限定在代码块内，当程序离开这个代码块时，资源会被自动关闭。

这是一项非常重要的改进，因为没人能在手动关闭资源时做到100%正确，甚至不久前Sun提供的操作指南都是错的。在向Coin项目提交这一提案时，提交者宣称JDK中有三分之二的close()用法都有bug，真是不可思议！

好在编译器可以生成这种学究化、公式化且手工编写易犯错的代码，所以Java 7借助了编译器来实现这项改进。

这可以减少我们编写错误代码的几率。相比之下，想想你用Java 6写段代码，要从一个URL（url）中读取字节流，并把读取到的内容写入到文件（out）中，这么做很容易产生错误。以下代码是可行方案之一。


**Java 6中的资源管理语法**

```java
InputStream is = null;
try {
  is = url.openStream();
  OutputStream out = new FileOutputStream(file);
  try {
    byte[] buf = new byte[4096];
    int len;
    while ((len = is.read(buf)) >= 0)
      out.write(buf, 0, len);
   } catch (IOException iox) {               // 处理异常（能读或写）
   } finally {
     try {
       out.close();
      } catch (IOException closeOutx) {      // 遇到异常也做不了什么
      }
   }
 } catch (FileNotFoundException fnfx) {      // 处理异常
 } catch (IOException openx) {               // 处理异常
 } finally {
    try {
      if (is != null) is.close();
    } catch (IOException closeInx) {         // 遇到异常也做不了什么
    }
 }
```

看明白了吗？重点是在处理外部资源时，墨菲定律（任何事都可能出错）一定会生效，比如：

- URL中的InputStream无法打开，不能读取或无法正常关闭；
- OutputStream对应的File无法打开，无法写入或不能正常关闭；
- 上面的问题同时出现。


最后一种情况是最让人头疼的——异常的各种组合拳打出来令人难以招架。

新语法能大大减少错误发生的可能性，这正是它受欢迎的主要原因。编译器不会犯开发人员编写代码时易犯的错误。

让我们看看代码清单1-3中的代码用Java 7写出来什么样。和前面一样，url是一个指向下载目标文件的URL对象，file是一个保存下载数据的File对象。


**Java 7中的资源管理语法**

```java
try (OutputStream out = new FileOutputStream(file);
     InputStream is = url.openStream() ) {
  byte[] buf = new byte[4096];
  int  len;
  while ((len = is.read(buf)) > 0) {
    out.write(buf, 0, len);
  }
}
```

这是资源自动化管理代码块的基本形式——把资源放在try的圆括号内。C#程序员看到这个也许会觉得有点眼熟，是的，它的确很像C#中的从句，带着这种理解使用这个新特性是个不错的起点。在这段代码块中使用的资源在处理完成后会自动关闭。

但使用try-with-resources特性时还是要小心，因为在某些情况下资源可能无法关闭。比如在下面的代码中，如果从文件（someFile.bin）创建ObjectInputStream时出错，FileInputStream可能就无法正确关闭。

```java
try ( ObjectInputStream in = new ObjectInputStream(new
      FileInputStream("someFile.bin")) ) { 
      ...
}
```

假定文件（someFile.bin）存在，但可能不是ObjectInput类型的文件，所以文件无法正确打开。因此不能构建ObjectInputStream，所以FileInputStream也没办法关闭。

要确保try-with-resources生效，正确的用法是为各个资源声明独立变量。

```java
try ( FileInputStream fin = new FileInputStream("someFile.bin");
          ObjectInputStream in = new ObjectInputStream(fin) ) {
    ...
}
```

TWR的另一个好处是改善了错误跟踪的能力，能够更准确地跟踪堆栈中的异常。在Java 7之前，处理资源时抛出的异常信息经常会被覆盖。TWR中可能也会出现这种情况，因此Java 7对跟踪堆栈进行了改进，现在开发人员能看到可能会丢失的异常类型信息。


比如在下面这段代码中，有一个返回InputStream的值为null的方法：

```java
 try(InputStream i = getNullStream()) {
   i.available();
}
```

在改进后的跟踪堆栈中能看到提示，注意其中被抑制的NullPointerException（简称NPE）：

```java
Exception in thread "main" java.lang.NullPointerException 
  at wgjd.ch01.ScratchSuprExcep.run(ScratchSuprExcep.java:23)
  at wgjd.ch01.ScratchSuprExcep.main(ScratchSuprExcep.java:39)
  Suppressed:java.lang.NullPointerException 
  at wgjd.ch01.ScratchSuprExcep.run(ScratchSuprExcep.java:24)   
    1 more
```

**TWR与AutoCloseable**

目前TWR特性依靠一个新定义的接口实现AutoCloseable。TWR的try从句中出现的资源类都必须实现这个接口。Java 7平台中的大多数资源类都被修改过，已经实现了AutoCloseable（Java 7中还定义了其父接口Closeable），但并不是全部资源相关的类都采用了这项新技术。不过，JDBC 4.1已经具备了这个特性。


然而在你自己的代码里，在需要处理资源时一定要用TWR，从而避免在异常处理时出现bug。

希望你能尽快使用try-with-resources，把那些多余的bug从代码库中赶走。


![xiulianzhidao](/images/xiulianzhidao.png)