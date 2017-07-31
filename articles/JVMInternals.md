# JVMInternals

这篇文章将解释JVM的内部架构。根据JAVA7 JVM规范，下图展现了一个典型JVM的的内部关键组件。

<img src="http://blog.jamesdbloom.com/images_2013_11_17_17_56/JVM_Internal_Architecture.png"/>

图上的这些组件将通过下面的两个章节注意解释。[章节一](#Thread)涵盖每一个线程独立创建的组件，[章节二]()包含独立在线程之外的组件。

- Threads
  - JVM System Threads
  - Per Thread
  - program Counter (PC)
  - Stack
  - Native Stack
  - Stack Restrictions
  - Frame
  - Local Variables Array
  - Operand Stack
  - Dynamic Linking
- Shared Between Threads
  - Heap
  - Memory Management
  - Non-Heap Memory
  - Just In Time (JIT) Compilation
  - Method Area
  - Class File Structure
  - Classloader
  - Faster Class Loading
  - Where Is The Method Area
  - Classloader Reference
  - Run Time Constant Pool
  - Exception Table
  - Symbol Table
  - Interned Strings (String Table)

## Thread
线程是程序中的一个执行主线。JVM允许应用程序并行执行多个主线，也就是多线程并行执行。在Hostspt JVM中，存在一个Java线程和本地操作系统线程的一一映射。Java线程启动时，需要分配thread-local存储空间、buffer、synchronization 对象、堆栈和计数器，之后本地线程被创建。当Java线程终止时，本地线程会被回收复用。因此，操作系统负责调度所有的线程，为这些线程分配CPU时间片。一旦本地线程初始化完成，Java线程类中声明的run()方法将被执行。当run()方法返回时，JVM会处理未捕获的异常，之后由本地线程根据情况决定，是否此线程结束后（如：最后一个非守护线程退出），JVM也需要停止并退出。当线程结束后，会释放所有的本地和Java线程资源。

### JVM System Threads
如果你使用jconsole或者其他任何debug工具，有可能你会发现有大量的线程在后台运行。这些后台线程随着main线程的启动而启动，即，在执行**public static void main(String[])**后，或其他main线程创建的其他线程，被启动后台执行。

Hotspot JVM 主要的后台线程包括：

1. **VM thread**: 这个线程专门用于处理那些需要等待JVM满足safe-point条件的操作。safe-point代表现在没有修改heap的操作发生。这种类型的操作包括："stop-the-world"类型的GC，thread stack dump，线程挂起，或撤销对象偏向锁(biased locking revocation)
1. **Periodic task thread**: 用于处理周期性事件（如：中断）的线程
1. **GC threads**: JVM中，用于支持不同阶段的GC操作的线程
1. **Compiler threads**: 用于在运行时，将字节码编译为本地代码的线程
1. **Signal dispatcher thread**: 接受发送给JVM处理的信号，并调用对应的JVM方法

## Per Thread
每个执行线程包含以下组件：

### Program Counter (PC)
当前操作指令或opcode的地址指针，如果当前方法是本地方法，则PC值为undefined。每个CPU都有一个PC，一般来说，每一次指令之后，PC值会增加，指向下一个操作指令的地址。JVM使用PC保持操作指令的执行顺序，PC值实际上就是指向方法区(Method Area)中的内存地址。

### Stack
每一个线程都拥有自己的栈（Stack），用于在本线程中正在执行的方法。栈是一个先进后出（LIFO）的数据结构，所以当前的执行方法位于栈顶。每一个方法开始执行时，一个新的帧（Frame)被创建（压栈），并添加到栈顶。当方法正常执行返回，或方法执行时抛出一个未捕获的异常，则此帧被移除（弹栈）。栈，除了压栈和弹栈操作外，不会被执行操作，因此，帧对象可以被分配在堆（Heap）内存中，并且不需要分配连续内存。

### Native Stack
不是所有的JVM都支持本地方法，然而，基本上都会为每个线程，创建本地方法栈。如果JVM使用C-Linkage模型，实现了JNI（Java Native Invocation），那么本地栈就会是一个C语言的栈。在这种情况下，本地栈中的方法参数和返回值顺序将和C语言程序完全一致。一个本地的方法一般可以回调JVM中的Java方法（依据具体JVM实现而定）。这样的本地方法调用Java方法一般会使用Java栈实现，当前线程将从本地栈中退出，在Java栈中创建一个新的帧。

### Stack Restrictions
栈可以使一个固定大小或动态大小。如果一个线程请求超过允许的栈空间，允许抛出StackOverflowError。如果一个线程请求创建一个帧，而没有足够内存时，则抛出OutOfMemoryError。

### Frame
每个方法开始执行时，一个帧被创建，压到栈顶。当方法正常执行返回，或方法执行时抛出一个未捕获的异常，此帧被删除，弹栈操作。更多细节，查看Exception Table章节。

每一帧包含以下信息：
- 本地变量数组, Local variable array
- 返回值
- 操作对象栈, Operand stack
- 当前方法所属类的运行时常量池

### Local Variables Array
本地变量数组包含所有方法执行过程中的所有变量，包括this引用，方法参数和其他定义的本地变量。对于类方法（静态方法），方法参数从0开始，然后对于实例方法，参数数据的第0个元素是this引用。

本地变量包括：
- boolean
- byte
- char
- long
- short
- int
- float
- double
- reference
- returnAddress

所有类型都占用一个数据元素，除了long和double，他们占用两个连续数组元素。（这两个类型是64位的，其他是32位的）

### Operand Stack
在执行字节代码指令过程中，使用操作对象栈的方式，与在本机CPU中使用通用寄存器相似。大多数JVM的字节码通过压栈、弹栈、复制、交换、操作执行这些方式来改变操作对象栈中的值。因此，在本地变量数组中和操作栈中移动复制数据，是高频操作。下面举例说明，通过操作对象栈，将一个简单的变量赋值为0.

```
int i;
```

编译后得到以下字节码
```
 0:	iconst_0	// 将0压到操作对象栈的栈顶
 1:	istore_1	// 从操作对象栈中弹栈，并将值存储到本地变量1中
```

更多的关于本地变量、对象操作栈和运行时常量池间的交互操作，请查看Class File Structure章节

### Dyanmic Linking
每个帧都包含一个引用指针，指向运行时常量池。这个引用指针指向当前被执行方法所属对象的常量池。这个引用帮助进行动态链接。

C/C++代码可以讲一个或者多个对象文件进行链接编译成一个可执行单元或dll。在链接过程中，每个对象文件中的引用变量，根据最终的可执行单元，被替换到实际的内存地址。在Java中，这个链接过程在运行时动态执行。

当Java Class被编译后，所有的变量和方法引用都利用一个引用标识存储在class的常量池中。一个引用标识是一个逻辑引用，而不是指向物理内存的实际指针。JVM实现可以选择何时替换引用标识，例如：class文件验证阶段、class文件加载后、高频调用发生时、静态编译链接、首次使用时。然后，如果在首次链接解析过程中出错，JVM不得不在后续的调用中，一直上报相同的错误。使用直接引用地址，替换属性字段、方法、类的引用标识被称作绑定（Binding）,这个操作只会被执行一次，因为引用标识都被完全替换掉，无法进行二次操作。如果引用标识指向的类没有被加载（resolved），则JVM会优先加载（load）它。每一个直接引用，就是方法和变量的运行时所存储的相对位置，也就是对应的内存偏移量。

## Shared Between Threads

### Heap

___
[返回吴晟的首页](https://wu-sheng.github.io/me/)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[英文版](http://blog.jamesdbloom.com/JVMInternals.html)
