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
