这里谈论的是Java虚拟机层面所暴露给我们的状态，与操作系统底层的线程状态是两个不同层面的事。具体而言，这里说的 Java 线程状态均来自于 Thread 类下的 State 这一内部枚举类中所定义的状态：

![image.png](https://upload-images.jianshu.io/upload_images/14079828-00eb8298b7fcb63c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

它与传统的线程状态的对应可以如下来看：

![image.png](https://upload-images.jianshu.io/upload_images/14079828-cb413f3aa98482a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

RUNNABLE 状态对应了传统的 ready， running 以及部分的 waiting 状态。而 BLOCKED 状态是只跟 synchronize 机制有关的一个状态。