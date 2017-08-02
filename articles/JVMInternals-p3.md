# 探秘JVM内部结构，第二部分

[吴晟](https://github.com/wu-sheng)，
[sky-walking APM](https://github.com/wu-sheng/sky-walking)

[回到第二部分...](JVMInternals-p2.md)

### Run Time Constant Pool 续
如果你编译下面这个类：
```java
package org.jvminternals;

public class SimpleClass {

    public void sayHello() {
        System.out.println("Hello");
    }
}
```

会生成如下所示的常量池：
```
Constant pool:
   #1 = Methodref          #6.#17         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #18.#19        //  java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #20            //  "Hello"
   #4 = Methodref          #21.#22        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #23            //  org/jvminternals/SimpleClass
   #6 = Class              #24            //  java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lorg/jvminternals/SimpleClass;
  #14 = Utf8               sayHello
  #15 = Utf8               SourceFile
  #16 = Utf8               SimpleClass.java
  #17 = NameAndType        #7:#8          //  "<init>":()V
  #18 = Class              #25            //  java/lang/System
  #19 = NameAndType        #26:#27        //  out:Ljava/io/PrintStream;
  #20 = Utf8               Hello
  #21 = Class              #28            //  java/io/PrintStream
  #22 = NameAndType        #29:#30        //  println:(Ljava/lang/String;)V
  #23 = Utf8               org/jvminternals/SimpleClass
  #24 = Utf8               java/lang/Object
  #25 = Utf8               java/lang/System
  #26 = Utf8               out
  #27 = Utf8               Ljava/io/PrintStream;
  #28 = Utf8               java/io/PrintStream
  #29 = Utf8               println
  #30 = Utf8               (Ljava/lang/String;)V
```

常量池包含以下类型：
- Integer：4 bytes的整数常量
- Long：8 bytes的长整形常量
- Float：4 bytes浮点
- Double：8 bytes浮点
- String：一个指向另一个Utf8实体的引用
- Utf8：一个使用UTF-8编码的字符串常量
- Class：一个指向另一个Utf8实体的引用，那个实体中存储类的全名，在动态链接时使用（查看Dynamic Linking章节）。
- NameAndType：以冒号分隔的两个值，每一个都指向常量池的一个实体。第一个值指向一个Utf8的实体，代表方法名或字段名；第二个值指向一个Utf8实体，代表类型，包含字段的全名，如果是方法，还会包含完整的方法签名和参数名。
- Fieldref，Methodref，InterfaceMethodref：用点分隔的两个值，每一个都指向常量池的一个实体。第一个指向一个Class实体，第二个指向一个NameAndType实体

### Exception Table


___
[返回吴晟的首页](https://wu-sheng.github.io/me/)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[英文版](http://blog.jamesdbloom.com/JVMInternals.html)
