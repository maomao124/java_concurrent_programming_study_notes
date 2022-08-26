<h1 style="color:green;text-align:center">java并发编程学习笔记</h1>







---





# 进程与线程

## 进程

* 程序由指令和数据组成，但这些指令要运行，数据要读写，就必须将指令加载至 CPU，数据加载至内存。在 指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理 IO 的
* 当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程
* 进程就可以视为程序的一个实例。大部分程序可以同时运行多个实例进程（例如记事本、画图、浏览器 等），也有的程序只能启动一个实例进程（例如网易云音乐、360 安全卫士等）





## 线程

* 一个进程之内可以分为一到多个线程
* 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行
* Java 中，线程作为最小调度单位，进程作为资源分配的最小单位。 在 windows 中进程是不活动的，只是作为线程的容器





## 二者对比

* 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集
* 进程拥有共享的资源，如内存空间等，供其内部的线程共享
* 进程间通信较为复杂
  * 同一台计算机的进程通信称为 IPC（Inter-process communication）
  * 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如 HTTP
* 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量
* 线程更轻量，线程上下文切换成本一般上要比进程上下文切换低







# 并行与并发

单核 cpu 下，线程实际还是 串行执行 的。操作系统中有一个组件叫做任务调度器，将 cpu 的时间片（windows 下时间片最小约为 15 毫秒）分给不同的程序使用，只是由于 cpu 在线程间（时间片很短）的切换非常快，人类感 觉是 同时运行的 。总结为一句话就是： 微观串行，宏观并行 

一般会将这种 线程轮流使用 CPU 的做法称为并发

多核 cpu下，每个 核（core） 都可以调度运行线程，这时候线程可以是并行的



* 并发（concurrent）是同一时间应对（dealing with）多件事情的能力
* 并行（parallel）是同一时间动手做（doing）多件事情的能力







# 异步调用

以调用方角度来讲，如果

* 需要等待结果返回，才能继续运行就是同步
* 不需要等待结果返回，就能继续运行就是异步



多线程可以让方法执行变为异步的（即不要巴巴干等着）比如说读取磁盘文件时，假设读取操作花费了 5 秒钟，如 果没有线程调度机制，这 5 秒 cpu 什么都做不了，其它代码都得暂停



同步调用：

```java
package mao;

import java.util.logging.Logger;

/**
 * Project name(项目名称)：java并发编程_异步调用
 * Package(包名): mao
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 19:19
 * Version(版本): 1.0
 * Description(描述)： 同步调用
 */

public class Test
{
    public static void m1()
    {
        System.out.println("开始执行m1方法");
        try
        {
            Thread.sleep(2000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        System.out.println("m1方法执行完毕");
    }

    public static void main(String[] args)
    {
        m1();
        System.out.println("继续执行main方法");
    }
}
```



异步调用：

```java
package mao;

/**
 * Project name(项目名称)：java并发编程_异步调用
 * Package(包名): mao
 * Class(类名): Test_Async
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 19:23
 * Version(版本): 1.0
 * Description(描述)： 异步调用
 */

public class Test_Async
{
    public static void m1()
    {
        System.out.println("开始执行m1方法");
        try
        {
            Thread.sleep(2000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        System.out.println("m1方法执行完毕");
    }

    public static void m1_async()
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                m1();
            }
        }, "m1");
        thread.start();
    }

    public static void main(String[] args)
    {
        m1_async();
        System.out.println("继续执行main方法");
    }
}
```



运行结果：

```sh
继续执行main方法
开始执行m1方法
m1方法执行完毕
```





* tomcat 的异步 servlet 也是类似的目的，让用户线程处理耗时较长的操作，避免阻塞 tomcat 的工作线程
* UI 程序中，开线程进行其他操作，避免阻塞 ui 线程







# 提高效率

充分利用多核 cpu 的优势，提高运行效率。想象下面的场景，执行 3 个计算，最后将计算结果汇总



* 计算 1 花费 10 ms
* 计算 2 花费 11 ms
* 计算 3 花费 9 ms
* 汇总需要 1 ms



如果是串行执行，那么总共花费的时间是 10 + 11 + 9 + 1 = 31ms

但如果是四核 cpu，各个核心分别使用线程 1 执行计算 1，线程 2 执行计算 2，线程 3 执行计算 3，那么 3 个 线程是并行的，花费时间只取决于最长的那个线程运行的时间，即 11ms 最后加上汇总时间只会花费 12ms



> 需要在多核 cpu 才能提高效率，单核仍然时是轮流执行



同步计算：

```java
package mao;

/**
 * Project name(项目名称)：java并发编程_异步调用
 * Package(包名): mao
 * Class(类名): Calculate
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 19:33
 * Version(版本): 1.0
 * Description(描述)： 同步计算
 */

public class Calculate
{
    /**
     * 计算1，模拟计算花费80毫秒
     */
    public static void c1()
    {
        try
        {
            Thread.sleep(80);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 计算2，模拟计算花费120毫秒
     */
    public static void c2()
    {
        try
        {
            Thread.sleep(120);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 计算3，模拟计算花费70毫秒
     */
    public static void c3()
    {
        try
        {
            Thread.sleep(70);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    public static void main(String[] args)
    {
        //------------------------------------------------------
        long startTime = System.currentTimeMillis();    //获取开始时间
        //------------------------------------------------------

        c1();
        c2();
        c3();

        //------------------------------------------------------
        long endTime = System.currentTimeMillis();    //获取结束时间
        System.out.println("程序运行时间：" + (endTime - startTime) + "ms");    //输出程序运行时间
        //------------------------------------------------------
    }

}
```



运行结果：

```sh
程序运行时间：289ms
```





异步计算：

```java
package mao;

/**
 * Project name(项目名称)：java并发编程_异步调用
 * Package(包名): mao
 * Class(类名): Calculate_Async
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 19:39
 * Version(版本): 1.0
 * Description(描述)： 异步调用
 */

public class Calculate_Async
{
    /**
     * 计算1，模拟计算花费80毫秒
     */
    public static void c1()
    {
        try
        {
            Thread.sleep(80);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 计算2，模拟计算花费120毫秒
     */
    public static void c2()
    {
        try
        {
            Thread.sleep(120);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 计算3，模拟计算花费70毫秒
     */
    public static void c3()
    {
        try
        {
            Thread.sleep(70);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 异步调用
     */
    public static Thread c1_async()
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                c1();
            }
        }, "c1");
        thread.start();
        return thread;
    }

    /**
     * 异步调用
     */
    public static Thread c2_async()
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                c2();
            }
        }, "c2");
        thread.start();
        return thread;
    }

    /**
     * 异步调用
     */
    public static Thread c3_async()
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                c3();
            }
        }, "c3");
        thread.start();
        return thread;
    }

    public static void main(String[] args) throws InterruptedException
    {
        //------------------------------------------------------
        long startTime = System.currentTimeMillis();    //获取开始时间
        //------------------------------------------------------

        Thread t1 = c1_async();
        Thread t2 = c2_async();
        Thread t3 = c3_async();
        t1.join();
        t2.join();
        t3.join();

        //------------------------------------------------------
        long endTime = System.currentTimeMillis();    //获取结束时间
        System.out.println("程序运行时间：" + (endTime - startTime) + "ms");    //输出程序运行时间
        //------------------------------------------------------
    }

}
```



运行结果：

```sh
程序运行时间：123ms
```





* 单核 cpu 下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu ，不至于一个线程总占用 cpu，别的线程没法干活
* 多核 cpu 可以并行跑多个线程，但能否提高程序运行效率还是要分情况的
  * 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任 务都能拆分
  * 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
* IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO】，这时相当于线程虽然不用 cpu，但需要一 直等待 IO 结束，没能充分利用线程









# Java 线程

## 创建和运行线程



方法一，直接使用 Thread：

```java
// 创建线程对象
Thread t = new Thread() {
 public void run() {
 // 要执行的任务
 }
};
// 启动线程
t.start();
```



方法二，使用 Runnable 配合 Thread

```java
Runnable runnable = new Runnable() {
 public void run(){
 // 要执行的任务
 }
};
// 创建线程对象
Thread t = new Thread( runnable );
// 启动线程
t.start(); 
```



方法三，FutureTask 配合 Thread：

```java
// 创建任务对象
FutureTask<Integer> task3 = new FutureTask<>(() -> {
 log.debug("hello");
 return 100;
});
// 参数1 是任务对象; 参数2 是线程名字，推荐
new Thread(task3, "t3").start();
// 主线程阻塞，同步等待 task 执行完毕的结果
Integer result = task3.get();
log.debug("结果是:{}", result);
```





## 栈与栈帧

Java Virtual Machine Stacks （Java 虚拟机栈）

我们都知道 JVM 中由堆、栈、方法区所组成，其中栈内存是给谁用的呢？其实就是线程，每个线程启动后，虚拟 机就会为其分配一块栈内存。

* 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存
* 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法



比如：main方法调用t方法



当函数开始执行的时候，会先将main方法压入栈中

![image-20220826200809004](img/java并发编程学习笔记/image-20220826200809004.png)



main方法调用t方法



![image-20220826200859030](img/java并发编程学习笔记/image-20220826200859030.png)



 **每一个栈帧都包含**

​    **1.局部变量表**

​    **2.操作数栈**

​    **3.返回地址**

​    **4.动态链接（指向运行时常量池的引用）**



![image-20220826200931983](img/java并发编程学习笔记/image-20220826200931983.png)









## 线程上下文切换

因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

* 线程的 cpu 时间片用完
* 垃圾回收
* 有更高优先级的线程需要运行
* 线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法



当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念 就是程序计数器（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，是线程私有的

* 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
* Context Switch 频繁发生会影响性能





## 常见方法



*  **start()：**启动一个新线 程，在新的线程 运行 run 方法 中的代码。start 方法只是让线程进入就绪，里面代码不一定立刻 运行（CPU 的时间片还没分给它）。每个线程对象的 start方法只能调用一次，如果调用了多次会出现 IllegalThreadStateException
* **run()：**新线程启动后会调用的方法。如果在构造 Thread 对象时传递了 Runnable 参数，则 线程启动后会调用 Runnable 中的 run 方法，否则默 认不执行任何操作。但可以创建 Thread 的子类对象， 来覆盖默认行为
* **join()：**等待线程运行结 束
* **join(long n)：**等待线程运行结 束,最多等待 n 毫秒
* **getId() ：**获取线程长整型 的 id 。id 唯一
* **getName()：**获取线程名
* **setName(String)：**修改线程名
* **getPriority()：**获取线程优先级
* **setPriority(int)：**修改线程优先级。java中规定线程优先级是1~10 的整数，较大的优先级 能提高该线程被 CPU 调度的机率
* **getState() ：**获取线程状态。Java 中线程状态是用 6 个 enum 表示，分别为： NEW, RUNNABLE, BLOCKED, WAITING,  TIMED_WAITING, TERMINATED
* **isInterrupted()：**判断是否被打断。不会清除 打断标记
* **isAlive()：**线程是否存活 （还没有运行完毕）
* **interrupt()：**打断线程。如果被打断线程正在 sleep，wait，join 会导致被打断 的线程抛出 InterruptedException，并清除打断标 记 ；如果打断的正在运行的线程，则会设置打断标 记 ；park 的线程被打断，也会设置打断标记
* **interrupted()：**判断当前线程是否被打断。会清除打断标记。静态方法
* **currentThread()：**获取当前正在执 行的线程。静态方法
* **sleep(long n)：**让当前执行的线 程休眠n毫秒， 休眠时让出 cpu 的时间片给其它线程。静态方法
* **yield() ：**提示线程调度器 让出当前线程对 CPU的使用。主要是为了测试和调试。静态方法





## start 与 run

* 直接调用 run 是在主线程中执行了 run，没有启动新的线程
* 使用 start 是启动新的线程，通过新的线程间接执行 run 中的代码



## sleep 与 yield





