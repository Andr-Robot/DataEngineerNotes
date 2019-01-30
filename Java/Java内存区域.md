* [方法区](#方法区)
    * [运行时常量池](#运行时常量池)
* [JVM堆](#jvm堆)
* [程序计数器](#程序计数器)
* [虚拟机栈](#虚拟机栈)
    * [栈帧(Stack Frame)结构](#栈帧stack-frame结构)
* [本地方法栈](#本地方法栈)
* [JDK1.8的一点变化](#jdk18的一点变化)
* [参考文献](#参考文献)

Java虚拟机在运行程序时会把其自动管理的内存划分为以下几个区域，每个区域都有的用途以及创建销毁的时机。接下来会分别介绍各个区域的功能。    
![jvm运行时数据区](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/java-memory.jpg)   
# 方法区
方法区属于线程共享的内存区域，又称Non-Heap（非堆），主要用于存储已被虚拟机加载的**类信息、常量、静态变量、即时编译器编译后的代码**等数据，根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出**OutOfMemoryError**异常。   
> **HotSpot**虚拟机处理方法区的时候选择把**GC分代收集**扩展至方法区，或者说使用**永久代**来实现方法区而已，这样HotSpot的垃圾收集器可以像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。可是其他的虚拟机并不会这样处理。

值得注意的是在方法区中存在一个叫**运行时常量池**(Runtime Constant Pool）的区域。
## 运行时常量池
它主要用于存放编译器生成的各种**字面量**和**符号引用**，这些内容将在类加载后存放到运行时常量池中，以便后续使用。运行时常量池除了编译期产生的Class文件的常量池，还可以**在运行期间**，将新的常量加入常量池，比较常见的是String类的intern()方法。
- 字面量：与Java语言层面的常量概念相近，包含文本字符串、声明为final的常量值等。
- 符号引用：编译语言层面的概念，包括以下3类：
    - 类和接口的全限定名
    - 字段的名称和描述符
    - 方法的名称和描述符

![字面量与符号引用](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/runtime_constant_pool.png)

接下来区分一下几个“常量池”的概念。
- **常量池（Constant Pool）**：常量池在数据编译期被确定，是Class文件中的一部分。存储了类、方法、接口等中的常量，当然也包括字符串常量。
- **字符串池/字符串常量池（String Pool/String Constant Pool）**：**是常量池中的一部分**，存储编译期类中产生的字符串类型数据。
- **运行时常量池（Runtime Constant Pool）**：方法区的一部分，所有线程共享。虚拟机加载Class后把**常量池**中的数据放入到运行时常量池。

**注意**：JDK1.6之前字符串常量池位于方法区之中，JDK1.7字符串常量池已经被挪到堆之中，JDK1.8方法区放在元空间里面。   

# JVM堆
Java堆也是属于线程共享的内存区域，它在**虚拟机启动时创建**，是Java虚拟机所管理的内存中**最大的一块**，主要用于**存放对象实例**，几乎所有的对象实例都在这里分配内存。   
注意**Java堆是垃圾收集器管理的主要区域**，因此很多时候也被称做GC堆。
- 从内存回收的角度来看。由于现在的收集器基本上采用的都是**分代收集算法**，所有Java堆可以细分为：**新生代**和**老年代**。在细致分就是把新生代分为：**Eden空间**、**From Survivor空间**、**To Survivor空间**。   
![java堆](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/javaheap.png)
- 从内存分配的角度来看，线程共享的Java堆可以划分出**线程私有的分配缓冲区(Thread Local Allocation Buffer,TLAB)**；这样划分的好处是为了更快的分配内存。

如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出**OutOfMemoryError** 异常。   
**注意**：关于在堆上内存分配是**并发**进行的，虚拟机采用**CAS加失败重试**保证原子操作，或者是采用**每个线程预先分配线程私有的分配缓冲区(Thread Local Allocation Buffer,TLAB)**。

# 程序计数器
属于线程私有的数据区域，是一小块内存空间，主要代表当前线程所执行的**字节码行号指示器**。字节码解释器工作时，通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。   
**注意**：如果线程在执行**Java方法**，这个计数器记录的是正在执行的虚拟机字节码指令地址；如果执行的是**Native方法**，这个计数器的值为**空（Undefined）**

# 虚拟机栈
属于线程私有的数据区域，与线程同时创建，总数与线程关联，代表Java方法执行的内存模型。每个方法（不包含native方法）执行时都会创建一个**栈桢**来存储**局部变量表（函数内部的变量）、操作数栈、动态链接、方法出口**等信息。每个方法的执行过程对应一个栈桢在虚拟机栈中的入栈和出栈过程。   

## 栈帧(Stack Frame)结构
![栈帧结构](https://raw.githubusercontent.com/Andr-Robot/iMarkdownPhotos/master/Res/jvmstack.jpg)   
- **局部变量表**：局部变量表所需的内存空间在编译期完成分配，用于存放方法参数和方法内部定义的局部变量。可能是**基本数据类型**（int、long、byte、short、float、double、char和boolean）、**对象引用**（reference类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）和**returnAddress类型**（指向了一条字节码指令的地址）,容量以**Slot**为最小单位。（**注意**：64位长度的long和double类型会占用2个Slot）
- **操作数栈**：操作数栈也常被称为操作栈，栈中的每一个元素可以是任意Java数据类型。当一个方法刚刚执行的时候，这个方法的操作数栈是空的，在方法执行的过程中，会有各种字节码指向操作数栈中写入和提取值，也就是入栈与出栈操作。
- **动态链接**：每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接。
- **方法出口**：
    - 正常退出，执行引擎遇到方法返回的字节码，将返回值传递给调用者
    - 异常退出，遇到Exception,并且方法未捕捉异常，那么不会有任何返回值。

Java虚拟机规范规定该区域有两种异常：
- **StackOverFlowError**：当线程请求栈深度超出虚拟机栈所允许的深度时抛出
- **OutOfMemoryError**：当Java虚拟机栈动态扩展到无法申请足够内存时抛出

# 本地方法栈
本地方法栈属于线程私有的数据区域，这部分主要与虚拟机用到的Native方法相关。Java虚拟机规范规定该区域可抛出**StackOverFlowError**和**OutOfMemoryError**。

# JDK1.8的一点变化
JDK 1.8同1.7比，最大的差别就是：**元数据区取代了永久代**。元空间的本质和永久代类似，都是对JVM规范中方法区的实现。主要用于存储类的信息、常量池、方法数据、方法代码等，由所有线程共享。不过元空间与永久代之间最大的区别在于：**元数据空间并不在虚拟机中，而是使用本地内存**。因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：    
- **-XX:MetaspaceSize**：初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
- **-XX:MaxMetaspaceSize**：最大空间，默认是没有限制的。   

**类的元数据存放在 MetaSpace，字符串常量移至 Java Heap。**

# 参考文献
[全面理解Java内存模型(JMM)及volatile关键字](https://blog.csdn.net/javazejian/article/details/72772461)    
[Java虚拟机-----方法区和运行时常量池](http://www.voidcn.com/article/p-caxakkds-bnw.html)     
[jvm 内存溢出 - 方法区及运行时常量池溢出](https://niuhp.github.io/java/jvm-oom-pg.html)   
[Java 内存之方法区和运行时常量池](https://mritd.me/2016/03/22/Java-%E5%86%85%E5%AD%98%E4%B9%8B%E6%96%B9%E6%B3%95%E5%8C%BA%E5%92%8C%E8%BF%90%E8%A1%8C%E6%97%B6%E5%B8%B8%E9%87%8F%E6%B1%A0/)    
[JVM理解其实并不难！](https://blog.csdn.net/huachao1001/article/details/51533132)   
[Jvm内存模型](http://gityuan.com/2016/01/09/java-memory/)   
[JDK1.8 JVM内存模型](https://blog.csdn.net/bruce128/article/details/79357870)   
[深入探究JVM | 探秘 Metaspace](https://www.sczyh30.com/posts/Java/jvm-metaspace/)   
[JVM内存模型](http://throwable.coding.me/2017/10/22/jvm-memory/)