# java虚拟机实用、面试、装x指南之jvm内存模型

有别与C/C++语言，java虚拟机解放了java程序员，使我们在开发过程中不必负责每个对象的出生和死亡，我们只管用new关键字创建对象，使用对象，对于不在使用的对象所占的内存空间的回收，完全交给java虚拟机来负责。在程序内存管理领域，java虚拟机即让我们避免成为
分配对象内存，回收对象内存这样单调工作的奴隶；也让我们丧失了成为掌握对象内存空间管理“生杀大权”的皇帝，内存管理对我们越发的陌生和透明，一旦有内存泄露或者溢出以及涉及到虚拟机方面的性能调优，都会让我们手足无措。而我们的《java虚拟机实用、面试、装x指南》系列文章，力求用
简单，生动，具体的讲述风格讲解java虚拟机的运作机制，让广大java开发人员得以管窥一二，在实际工作中、在面试找工作时、在饭后和同事吹哔时，都能得心应手，从容应对。鉴于java虚拟机知识体系的庞杂和深邃以及本员能力的限制，系列文章中难免有错误和不足，希望大家多多包涵。


记得有位大佬曾经说过这样一句话：
> 如欲征服java，必须征服java虚拟机，如欲征服java虚拟机，需先征服java虚拟机内存模型。

java虚拟机内存，是java虚拟机进行对象内存空间分配、垃圾回收的活动室，只有先了解java虚拟机内存才能在此基础上进一步了解对象内存分配、垃圾回收等活动。有别于真实物理机硬盘、主存、缓存、寄存器的存储模型，java虚拟机内存模型按照其存储模块负责的数据类型将其划分为如下图所示的模型：

![java虚拟机内存模型](https://github.com/aworker/posts/raw/master/java/jvm/post1/jvm_memory_model.png)

## 堆
堆是各个线程共享的内存区域，是java对象内存分配和垃圾回收的主战场，几乎所有的对象都是在堆中创建的。根据*Java虚拟机规范（Java Virtual Machine Specification）* 的规则，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可。如果在堆中没有内存空间完成Java对象的内存分配时，将会抛出OutOfMemoryError（一下简称OOM）。

关于堆的虚拟机参数最常见的两个是：
+ -Xms ：表示虚拟机堆的最小值，如 -Xms10M 表示堆的最小值为10MB
+ -Xmx ：表示虚拟机堆的最大值，如果 -Xmx100M 表示堆的最大值为100MB

```
/**
 * 设置虚拟机参数为：-Xms5M -Xmx5M
 */
public class HeapOOM {
    public static void main(String[] args) {
        ArrayList<Byte[]> bytes = new ArrayList<>();
        for (; ; ) {
            Byte[] _1M = new Byte[1024 * 1024];
            bytes.add(_1M);
        }

    }
}
```
执行结果:
```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at jvm.post1.HeapOOM.main(HeapOOM.java:15)
```
“Java heap space”类型的OOM表示堆中没有可用的内存空间，具体到本例子中就是在大小为5M的堆中没有可用空间分配给大小为1M的数组对象。再来看一个例子：
```aidl
/**
 * @createtime 2019/11/2
 * 虚拟机参数 -Xms5M -Xmx5M 
*/ 
public class HeapOOM1 {
    public static void main(String[] args) {
        ArrayList<Object> heapOOM1s = new ArrayList<>();
        for (; ; ) {
            heapOOM1s.add(new Object());
        }
    }
}
```
执行结果:
```aidl
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at jvm.post1.HeapOOM1.main(HeapOOM1.java:14)
```
“GC overhead limit exceeded” 类型的OOM是在jdk6后引入的一种新的错误类型。发生错误的原因是虚拟机用了大量的时间进行GC但是只释放了较小的空间，这是虚拟机的一种保护机制。具体到本例子中就是虚拟机在GC时没有能回收内存空间，浪费了时间却没有收获，所以就抛出了这个错误。可以用 *-XX:-UseGCOverheadLimit*参数禁用这个检查，但解决不了内存问题，只是把错误的信息延后，替换成 java.lang.OutOfMemoryError: Java heap space错误。

## 方法区
方法区和堆一样，也是各个线程共享的内存区域，它用来存储已经被虚拟机加载的类信息、常量池、静态变量等。方法区是jdk5到jdk8变化较大的java虚拟机内存区域。在jdk5和jdk6时，常量池是存在方法区的：

![jdk5和jdk6](https://github.com/aworker/posts/raw/master/java/jvm/post1/jvm_memory_model_1.5.png)

而从jdk7及其以后的版本，常量池被放到了堆里面：

![jdk7](https://github.com/aworker/posts/raw/master/java/jvm/post1/jvm_memory_model_1.7.png)

常量池就是java语言系统级别的缓存，目的是让程序在运行过程中速度更快，更节省内存空间，java的8种基本数据类型外加String类型，共9种类型都有对应的常量池。这些类型的对象不可能全都放到常量池中存储，因此不同的类型有不同的存储策略，具体到String类型的对象来说，有如下三条规则：

+ 用双引号创建的对象放在常量池中，如 "Hello"，"Jvm"这种。
+ 用双引号创建的对象相加产生的对象放在常量池，如 *String s = "Hello" + "Jvm";*，这里的s对象就是放在常量池中的。
+ 调用String对象的intern方法会返回一个存放在常量池中的String对象,且两个对象内容相同。

再回到本篇的主题上，因为常量池位置的变化，在不同的jdk版本下，下面代码的执行结果是不一样的：
```aidl
/**
 * @description java常量池
 */
public class ConstantsPool {
    public static void main(String[] args) {

        String s = new String("Hello") + new String("Jvm"); //1
        String s1 = s.intern();  //2
        System.out.println(s == s1); //jdk5和jdk6中返回false，jdk7及其以上版本返回true。
    }
}
``` 

在jdk7之前，程序在执行//2处代码之前常量池中没有"HelloJvm"这个字符串常量，//2处代码执行时，程序会在常量池中创建一个"HelloJvm"的字符串对象s1并返回，而常量池是在方法区的。那一个在堆中的s对象和方法区中的s1对象比较地址是否相同，当然会得到false。
在jdk7及其以后的版本，程序在执行//2出代码时，发现常量池中同样没有"HelloJvm"这个对象，但因为常量池已经迁移到堆中，常量池不需要存储一个对象了，程序只是简单的把s这个对象的引用在常量池中存储了，此时s和s1指向的是同一个对象，结果当然是true。












> 有任何不懂或者质疑的地方，都欢迎大家积极留言讨论，留言必会，一起学习进步。


程序计数器，唯一不会OOM