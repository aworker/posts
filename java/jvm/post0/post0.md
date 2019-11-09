java虚拟机是一个抽象的计算机。对于程序开发人员来说，我们只需要编写好我们的程序，让不同平台上的java虚拟机完成java代码到不同操作系统计算机指令的转化，通过这种方式java语言实现了其夸平台特性。虽然名为java虚拟机，但是它并不是和java语言强绑定的，java虚拟机只能识别有特定语法格式的class文件，而所有能编译为class文件的语言，都可以在java虚拟机上运行，java虚拟机不是“java语言的虚拟机”而是“多语言的虚拟机”。

在java虚拟机的发展历史上，曾经有很多的优秀的虚拟机实现。如随着jdk1.0一起发布的Classic VM，号称速度最快的JRockit VM，以及专精Windows平台的Microsoft JVM等。但我们普通程序员用的最多的也是“血统最纯正”虚拟机实现就是Sun公司发布的HotSpot VM，这个系列也以HotSpot虚拟机为蓝本进行讲述的。


HotSpot虚拟机的组件结构如下：

![运行时栈结构](https://github.com/aworker/posts/raw/master/java/jvm/post0/HotSpot-architecture.jpg) 

HotSpot虚拟机有三大主要模块：

+ 类加载子系统，主要用来加载class文件，从中获取类信息。
+ 虚拟机运行时数据区，主要存储程序运行时的各种对象信息和线程栈帧数据。
+ 执行引擎，主要内存垃圾回收和运行时代码优化。

本系列将会从实际应用、面试需要、饭后吹逼等三个角度分别讲解这三大主要模块。同时给自己立个flag到2020年能完成本系列文章的全部写作。

> 有任何不懂或者质疑的地方，都欢迎大家积极留言讨论，留言必回，一起学习进步。



<!-- 
参考文献 https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html
-->