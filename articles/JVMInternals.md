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

- **VM thread**: 这个线程专门用于处理那些需要等待JVM满足safe-point条件的操作。safe-point代表现在没有修改heap的操作发生。这种类型的操作包括："stop-the-world"类型的GC，thread stack dump，线程挂起，或撤销对象偏向锁(biased locking revocation)|
- **Periodic task thread**: 用于处理周期性事件（如：中断）的线程|
- **GC threads**: JVM中，用于支持不同阶段的GC操作的线程|
- **Compiler threads**: 用于在运行时，将字节码编译为本地代码的线程|
- **Signal dispatcher thread**: 接受发送给JVM处理的信号，并调用对应的JVM方法|

## Per Thread



___
[返回吴晟的首页](https://wu-sheng.github.io/me/)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[英文版](http://blog.jamesdbloom.com/JVMInternals.html)
