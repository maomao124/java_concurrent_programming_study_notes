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

### sleep

* 调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态（阻塞）
* 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException
* 睡眠结束后的线程未必会立刻得到执行
* 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性





```java
package mao.sleep;

/**
 * Project name(项目名称)：java并发编程_sleep和yield
 * Package(包名): mao.sleep
 * Class(类名): Test1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 20:28
 * Version(版本): 1.0
 * Description(描述)： 调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态（阻塞）
 */

public class Test1
{
    public static void main(String[] args) throws InterruptedException
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(3000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        });
        System.out.println(thread.getState());
        thread.start();
        System.out.println(thread.getState());
        Thread.sleep(30);
        System.out.println(thread.getState());
    }
}
```



运行结果：

```sh
NEW
RUNNABLE
TIMED_WAITING
```





```java
package mao.sleep;

/**
 * Project name(项目名称)：java并发编程_sleep和yield
 * Package(包名): mao.sleep
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 20:31
 * Version(版本): 1.0
 * Description(描述)： 睡眠结束后的线程未必会立刻得到执行
 */

public class Test2
{
    public static void main(String[] args) throws InterruptedException
    {
        //------------------------------------------------------
        long startTime = System.currentTimeMillis();    //获取开始时间
        //------------------------------------------------------

        Thread.sleep(100);
        //时间消耗大于100毫秒

        //------------------------------------------------------
        long endTime = System.currentTimeMillis();    //获取结束时间
        System.out.println("程序运行时间：" + (endTime - startTime) + "ms");    //输出程序运行时间
        //------------------------------------------------------
    }
}
```



运行结果：

```sh
程序运行时间：107ms
```



```java
package mao.sleep;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_sleep和yield
 * Package(包名): mao.sleep
 * Class(类名): Test3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 20:38
 * Version(版本): 1.0
 * Description(描述)： 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性
 */

public class Test3
{
    public static void main(String[] args) throws InterruptedException
    {
        //------------------------------------------------------
        long startTime = System.currentTimeMillis();    //获取开始时间
        //------------------------------------------------------

        TimeUnit.SECONDS.sleep(2);
        //两秒
        //TimeUnit.MINUTES.sleep(1);
        //一分钟

        //------------------------------------------------------
        long endTime = System.currentTimeMillis();    //获取结束时间
        System.out.println("程序运行时间：" + (endTime - startTime) + "ms");    //输出程序运行时间
        //------------------------------------------------------
    }
}
```



运行结果：

```sh
程序运行时间：2002ms
```





### yield

* 调用 yield 会让当前线程从 Running 进入 Runnable 就绪状态，然后调度执行其它线程
* 具体的实现依赖于操作系统的任务调度器



```java
public class Thread implements Runnable 
{
/**
 * A hint to the scheduler that the current thread is willing to yield
 * its current use of a processor. The scheduler is free to ignore this
 * hint.
 *
 * <p> Yield is a heuristic attempt to improve relative progression
 * between threads that would otherwise over-utilise a CPU. Its use
 * should be combined with detailed profiling and benchmarking to
 * ensure that it actually has the desired effect.
 *
 * <p> It is rarely appropriate to use this method. It may be useful
 * for debugging or testing purposes, where it may help to reproduce
 * bugs due to race conditions. It may also be useful when designing
 * concurrency control constructs such as the ones in the
 * {@link java.util.concurrent.locks} package.
 */
public static native void yield();
}
```







## 线程优先级

* 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
* 如果 cpu 比较忙，那么优先级高的线程会获得更多的时间片，但 cpu 闲时，优先级几乎没作用



```java
package mao;

/**
 * Project name(项目名称)：java并发编程_线程优先级
 * Package(包名): mao
 * Class(类名): Test1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 20:53
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test1
{
    private static int a = 0;
    private static int b = 0;

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    a++;
                }
            }
        }, "t1");

        Thread thread2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    b++;
                }
            }
        }, "t2");

        //启动线程
        thread1.start();
        thread2.start();

        //主线程休眠
        Thread.sleep(2000);

        //强制打断线程
        thread1.stop();
        thread2.stop();

        //查看a和b的值
        System.out.println("a=" + a);
        System.out.println("b=" + b);
        //优先级一样，a和b的值也差不多
    }
}
```



运行结果：

```sh
a=172403497
b=153380774
```

```sh
a=172840004
b=229767254
```

```sh
a=160737324
b=167375710
```





```java
package mao;

/**
 * Project name(项目名称)：java并发编程_线程优先级
 * Package(包名): mao
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 21:02
 * Version(版本): 1.0
 * Description(描述)： yield方法让出CPU使用权
 */

public class Test2
{
    private static int a = 0;
    private static int b = 0;

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    a++;
                }
            }
        }, "t1");

        Thread thread2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    b++;
                    //自增后调用yield方法
                    Thread.yield();
                }
            }
        }, "t2");

        //启动线程
        thread1.start();
        thread2.start();

        //主线程休眠
        Thread.sleep(2000);

        //强制打断线程
        thread1.stop();
        thread2.stop();

        //查看a和b的值
        System.out.println("a=" + a);
        System.out.println("b=" + b);
        //线程2每次自增后调用yield方法，a远大于b
    }
}
```



运行结果：

```sh
a=238488195
b=11278176
```

```sh
a=247029331
b=11722908
```

```sh
a=264029438
b=12110354
```



```java
package mao;

/**
 * Project name(项目名称)：java并发编程_线程优先级
 * Package(包名): mao
 * Class(类名): Test3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 21:06
 * Version(版本): 1.0
 * Description(描述)： 更改线程的优先级
 */

public class Test3
{
    private static int a = 0;
    private static int b = 0;

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    a++;
                }
            }
        }, "t1");

        Thread thread2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    b++;
                }
            }
        }, "t2");

        //更改优先级
        thread1.setPriority(1);
        thread2.setPriority(10);

        //启动线程
        thread1.start();
        thread2.start();

        //主线程休眠
        Thread.sleep(2000);

        //强制打断线程
        thread1.stop();
        thread2.stop();

        //查看a和b的值
        System.out.println("a=" + a);
        System.out.println("b=" + b);
        
    }
}
```



如果 cpu 比较忙，那么优先级高的线程会获得更多的时间片，但 cpu 闲时，优先级几乎没作用



```java
package mao;

import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_线程优先级
 * Package(包名): mao
 * Class(类名): Test3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 21:06
 * Version(版本): 1.0
 * Description(描述)： 更改线程的优先级
 */

public class Test3
{
    private static int a = 0;
    private static int b = 0;

    /**
     * 使CPU使用率达到100%
     */
    public static void t()
    {
        int threadCount = 15;
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < threadCount; i++)
        {
            Thread thread = new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    //死循环
                    while (true)
                    {

                    }
                }
            });
            //设置为守护线程
            thread.setDaemon(true);
            threads.add(thread);
        }
        //启动线程
        for (Thread thread : threads)
        {
            thread.start();
        }
    }

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    a++;
                }
            }
        }, "t1");

        Thread thread2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    b++;
                }
            }
        }, "t2");

        //更改优先级
        thread1.setPriority(1);
        thread2.setPriority(10);

        //启动线程
        t();
        thread1.start();
        thread2.start();

        //主线程休眠
        Thread.sleep(2000);

        //强制打断线程
        thread1.stop();
        thread2.stop();

        //查看a和b的值
        System.out.println("a=" + a);
        System.out.println("b=" + b);

    }
}
```



运行结果：

```sh
a=120955355
b=487451855
```

```sh
a=117420720
b=542875659
```

```sh
a=194952269
b=272184251
```







## join 方法



```java
static int r = 0;
public static void main(String[] args) throws InterruptedException {
 test1();
}
private static void test1() throws InterruptedException {
 log.debug("开始");
 Thread t1 = new Thread(() -> {
 log.debug("开始");
 Thread.sleep(1000);
 log.debug("结束");
 r = 10;
 });
 t1.start();
 log.debug("结果为:{}", r);
 log.debug("结束");
}
```



* 因为主线程和线程 t1 是并行执行的，t1 线程需要 1 秒之后才能算出 r=10
* 而主线程一开始就要打印 r 的结果，所以只能打印出 r=0



想要在t1线程运行结束后立马打印结果，就需要用到join方法



```java
static int r = 0;
public static void main(String[] args) throws InterruptedException {
 test1();
}
private static void test1() throws InterruptedException {
 log.debug("开始");
 Thread t1 = new Thread(() -> {
 log.debug("开始");
 Thread.sleep(1000);
 log.debug("结束");
 r = 10;
 });
 t1.start();
 t1.join();
 log.debug("结果为:{}", r);
 log.debug("结束");
}
```





```java
static int r1 = 0;
static int r2 = 0;
public static void main(String[] args) throws InterruptedException {
 test2();
}
private static void test2() throws InterruptedException {
 Thread t1 = new Thread(() -> {
 sleep(1000);
 r1 = 10;
 });
 Thread t2 = new Thread(() -> {
 sleep(2000);
 r2 = 20;
 });
 long start = System.currentTimeMillis();
 t1.start();
 t2.start();
 t1.join();
 t2.join();
 long end = System.currentTimeMillis();
 log.debug("r1: {} r2: {} cost: {}", r1, r2, end - start);
}
```



* 第一个 join：等待 t1 时, t2 并没有停止, 而在运行
* 第二个 join：1s 后, 执行到此, t2 也运行了 1s, 因此也只需再等待 1s



所以，时间花费为两秒







## interrupt 方法

**打断 sleep，wait，join 的线程**

这几个方法都会让线程进入阻塞状态

打断 sleep 的线程, 会清空打断状态



```java
package mao;

/**
 * Project name(项目名称)：java并发编程_interrupt
 * Package(包名): mao
 * Class(类名): Test1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 22:05
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test1
{
    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(2000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        });
        t1.start();
        Thread.sleep(1000);
        t1.interrupt();
        System.out.println(t1.isInterrupted());
    }
}
```



运行结果：

```sh
false
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.Test1$1.run(Test1.java:27)
	at java.base/java.lang.Thread.run(Thread.java:831)
```





**打断正常运行的线程**

打断正常运行的线程, 不会清空打断状态



```java
package mao;

/**
 * Project name(项目名称)：java并发编程_interrupt
 * Package(包名): mao
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/26
 * Time(创建时间)： 22:09
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    Thread thread = Thread.currentThread();
                    boolean interrupted = thread.isInterrupted();
                    if (interrupted)
                    {
                        System.out.println("当前线程已经被打断");
                        break;
                    }
                }
            }
        });
        t1.start();
        Thread.sleep(1000);
        t1.interrupt();
        System.out.println(t1.isInterrupted());
    }
}
```



运行结果：

```sh
true
当前线程已经被打断
```







## 模式之两阶段终止



![image-20220826222422781](img/java并发编程学习笔记/image-20220826222422781.png)





```java
package mao;

/**
 * Project name(项目名称)：java并发编程_两阶段终止
 * Package(包名): mao
 * Class(类名): Test1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 20:02
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test1
{
    public static void service()
    {
        System.out.println("----------------------------------");
        Runtime r = Runtime.getRuntime();
        float memory;
        memory = r.totalMemory();
        memory = memory / 1024 / 1024;
        System.out.printf("JVM总内存：%.3fMB\n", memory);
        memory = r.freeMemory();
        memory = memory / 1024 / 1024;
        System.out.printf(" 空闲内存：%.3fMB\n", memory);
        memory = r.totalMemory() - r.freeMemory();
        memory = memory / 1024 / 1024;
        System.out.printf("已使用的内存：%.4fMB\n", memory);
        System.out.println("----------------------------------");
    }

    public static void release()
    {
        System.out.println("释放资源");
    }

    public static Thread t1()
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    Thread currentThread = Thread.currentThread();
                    if (currentThread.isInterrupted())
                    {
                        //被打断，料理后事，比如释放资源
                        release();
                        //结束循环
                        break;
                    }
                    //没有被打断
                    try
                    {
                        //睡眠
                        Thread.sleep(1000);
                        //无异常，执行业务
                        service();
                    }
                    catch (InterruptedException e)
                    {
                        //有异常，设置打断标记
                        currentThread.interrupt();
                    }

                }
            }
        });
        thread.start();
        return thread;
    }

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread = t1();
        Thread.sleep(3000);
        thread.interrupt();
    }
}
```



运行结果：

```sh
----------------------------------
JVM总内存：256.000MB
 空闲内存：251.118MB
已使用的内存：4.8820MB
----------------------------------
----------------------------------
JVM总内存：256.000MB
 空闲内存：251.118MB
已使用的内存：4.8820MB
----------------------------------
释放资源
```





**封装**



```java
package mao;

/**
 * Project name(项目名称)：java并发编程_两阶段终止
 * Package(包名): mao
 * Interface(接口名): ThreadService
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 20:20
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public interface ThreadService
{
    /**
     * 主业务
     *
     * @throws Exception 异常
     */
    void service() throws Exception;

    /**
     * 被打断后释放资源的业务
     */
    void release();
}
```



```java
package mao;

/**
 * Project name(项目名称)：java并发编程_两阶段终止
 * Package(包名): mao
 * Class(类名): T
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 20:20
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class T
{
    /**
     * 启动一个支持两阶段终止的线程，不需要调用start方法
     *
     * @param threadName    线程的名称
     * @param intervals     调用service方法的间隔时间，单位为毫秒
     * @param threadService 业务
     * @return Thread对象
     */
    public static Thread startThread(String threadName, long intervals, ThreadService threadService)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    Thread currentThread = Thread.currentThread();
                    if (currentThread.isInterrupted())
                    {
                        //被打断，料理后事，比如释放资源
                        threadService.release();
                        //结束循环
                        break;
                    }
                    //没有被打断
                    try
                    {
                        //睡眠
                        Thread.sleep(intervals);
                        //无异常，执行业务
                        threadService.service();
                    }
                    catch (Exception e)
                    {
                        //有异常，设置打断标记
                        currentThread.interrupt();
                    }
                }
            }
        }, threadName);
        thread.start();
        return thread;
    }

    /**
     * 启动一个支持两阶段终止的线程，不需要调用start方法
     *
     * @param intervals     调用service方法的间隔时间，单位为毫秒
     * @param threadService 业务
     * @return Thread对象
     */
    public static Thread startThread(long intervals, ThreadService threadService)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    Thread currentThread = Thread.currentThread();
                    if (currentThread.isInterrupted())
                    {
                        //被打断，料理后事，比如释放资源
                        threadService.release();
                        //结束循环
                        break;
                    }
                    //没有被打断
                    try
                    {
                        //睡眠
                        Thread.sleep(intervals);
                        //无异常，执行业务
                        threadService.service();
                    }
                    catch (Exception e)
                    {
                        //有异常，设置打断标记
                        currentThread.interrupt();
                    }
                }
            }
        });
        thread.start();
        return thread;
    }

    /**
     * 启动一个支持两阶段终止的线程，不需要调用start方法。间隔时间默认为1秒
     *
     * @param threadService 业务
     * @return Thread对象
     */
    public static Thread startThread(ThreadService threadService)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    Thread currentThread = Thread.currentThread();
                    if (currentThread.isInterrupted())
                    {
                        //被打断，料理后事，比如释放资源
                        threadService.release();
                        //结束循环
                        break;
                    }
                    //没有被打断
                    try
                    {
                        //睡眠
                        Thread.sleep(1000);
                        //无异常，执行业务
                        threadService.service();
                    }
                    catch (Exception e)
                    {
                        //有异常，设置打断标记
                        currentThread.interrupt();
                    }
                }
            }
        });
        thread.start();
        return thread;
    }
}
```



```java
package mao;


/**
 * Project name(项目名称)：java并发编程_两阶段终止
 * Package(包名): mao
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 20:30
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    public static int getIntRandom(int min, int max)
    {
        if (min > max)
        {
            min = max;
        }
        return min + (int) (Math.random() * (max - min + 1));
    }


    public static void main(String[] args) throws InterruptedException
    {
        Thread thread = T.startThread(500, new ThreadService()
        {
            @Override
            public void service() throws Exception
            {
                System.out.println("----------------------------------");
                Runtime r = Runtime.getRuntime();
                float memory;
                memory = r.totalMemory();
                memory = memory / 1024 / 1024;
                System.out.printf("JVM总内存：%.3fMB\n", memory);
                memory = r.freeMemory();
                memory = memory / 1024 / 1024;
                System.out.printf(" 空闲内存：%.3fMB\n", memory);
                memory = r.totalMemory() - r.freeMemory();
                memory = memory / 1024 / 1024;
                System.out.printf("已使用的内存：%.4fMB\n", memory);
                System.out.println("----------------------------------");
            }

            @Override
            public void release()
            {
                System.out.println("释放资源");
            }
        });

        Thread.sleep(getIntRandom(2000, 3000));
        thread.interrupt();
    }
}
```



运行结果：

```sh
----------------------------------
JVM总内存：256.000MB
 空闲内存：252.000MB
已使用的内存：4.0000MB
----------------------------------
----------------------------------
JVM总内存：256.000MB
 空闲内存：251.118MB
已使用的内存：4.8821MB
----------------------------------
----------------------------------
JVM总内存：256.000MB
 空闲内存：251.118MB
已使用的内存：4.8821MB
----------------------------------
----------------------------------
JVM总内存：256.000MB
 空闲内存：251.118MB
已使用的内存：4.8821MB
----------------------------------
----------------------------------
JVM总内存：256.000MB
 空闲内存：251.118MB
已使用的内存：4.8821MB
----------------------------------
释放资源
```





## 守护线程

默认情况下，Java 进程需要等待所有线程都运行结束才会结束。有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束



如果需要设置成守护线程，则需要调用线程的setDaemon方法，设置成true





## 五种状态

![image-20220827211631119](img/java并发编程学习笔记/image-20220827211631119.png)



**这是从操作系统层面来描述的**



* 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联
* 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），可以由 CPU 调度执行
* 【运行状态】指获取了 CPU 时间片运行中的状态
  * 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
* 【阻塞状态】
  * 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入【阻塞状态】
  * 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
  * 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑调度它们
* 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态





## 六种状态



![image-20220827211854901](img/java并发编程学习笔记/image-20220827211854901.png)





**这是从 Java API 层面来描述的**



* NEW： 线程刚被创建，但是还没有调用 start() 方法
* RUNNABLE ：当调用了 start() 方法之后，注意，Java API 层面的 RUNNABLE 状态涵盖了 操作系统 层面的 【可运行状态】、【运行状态】和【阻塞状态】（由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为是可运行）
* BLOCKED ， WAITING ， TIMED_WAITING 都是 Java API 层面对【阻塞状态】的细分
* TERMINATED 当线程代码运行结束





```java
/**
 * A thread state.  A thread can be in one of the following states:
 * <ul>
 * <li>{@link #NEW}<br>
 *     A thread that has not yet started is in this state.
 *     </li>
 * <li>{@link #RUNNABLE}<br>
 *     A thread executing in the Java virtual machine is in this state.
 *     </li>
 * <li>{@link #BLOCKED}<br>
 *     A thread that is blocked waiting for a monitor lock
 *     is in this state.
 *     </li>
 * <li>{@link #WAITING}<br>
 *     A thread that is waiting indefinitely for another thread to
 *     perform a particular action is in this state.
 *     </li>
 * <li>{@link #TIMED_WAITING}<br>
 *     A thread that is waiting for another thread to perform an action
 *     for up to a specified waiting time is in this state.
 *     </li>
 * <li>{@link #TERMINATED}<br>
 *     A thread that has exited is in this state.
 *     </li>
 * </ul>
 *
 * <p>
 * A thread can be in only one state at a given point in time.
 * These states are virtual machine states which do not reflect
 * any operating system thread states.
 *
 * @since   1.5
 * @see #getState
 */
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     */
    NEW,

    /**
     * Thread state for a runnable thread.  A thread in the runnable
     * state is executing in the Java virtual machine but it may
     * be waiting for other resources from the operating system
     * such as processor.
     */
    RUNNABLE,

    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */
    BLOCKED,

    /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
     * <p>A thread in the waiting state is waiting for another thread to
     * perform a particular action.
     *
     * For example, a thread that has called {@code Object.wait()}
     * on an object is waiting for another thread to call
     * {@code Object.notify()} or {@code Object.notifyAll()} on
     * that object. A thread that has called {@code Thread.join()}
     * is waiting for a specified thread to terminate.
     */
    WAITING,

    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```









# 共享模型之管程

## 共享问题

故事

* 老王（操作系统）有一个功能强大的算盘（CPU），现在想把它租出去，赚一点外快
* 小南、小女（线程）来使用这个算盘来进行一些计算，并按照时间给老王支付费用
* 但小南不能一天24小时使用算盘，他经常要小憩一会（sleep），又或是去吃饭上厕所（阻塞 io 操作），有 时还需要一根烟，没烟时思路全无（wait）这些情况统称为（阻塞）
* 在这些时候，算盘没利用起来（不能收钱了），老王觉得有点不划算
* 另外，小女也想用用算盘，如果总是小南占着算盘，让小女觉得不公平
* 于是，老王灵机一动，想了个办法 [ 让他们每人用一会，轮流使用算盘 ]
* 这样，当小南阻塞的时候，算盘可以分给小女使用，不会浪费，反之亦然
* 最近执行的计算比较复杂，需要存储一些中间结果，而学生们的脑容量（工作内存）不够，所以老王申请了 一个笔记本（主存），把一些中间结果先记在本上
* 计算流程是这样的



![image-20220827213020070](img/java并发编程学习笔记/image-20220827213020070.png)



*  但是由于分时系统，有一天还是发生了事故
* 小南刚读取了初始值 0 做了个 +1 运算，还没来得及写回结果
* 老王说 [ 小南，你的时间到了，该别人了，记住结果走吧 ]，于是小南念叨着 [ 结果是1，结果是1...] 不甘心地 到一边待着去了（上下文切换）
* 老王说 [ 小女，该你了 ]，小女看到了笔记本上还写着 0 做了一个 -1 运算，将结果 -1 写入笔记本
* 这时小女的时间也用完了，老王又叫醒了小南：[小南，把你上次的题目算完吧]，小南将他脑海中的结果 1 写 入了笔记本



![image-20220827213134035](img/java并发编程学习笔记/image-20220827213134035.png)



* 小南和小女都觉得自己没做错，但笔记本里的结果是 1 而不是 0





## 共享问题的java实现



```java
/**
 * Project name(项目名称)：java并发编程_共享问题
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 21:33
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test1
{
    public static int count = 0;

    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 10000; i++)
                {
                    count++;
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 10000; i++)
                {
                    count--;
                }
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        //输出
        System.out.println(count);
    }
}
```



运行结果：

```sh
-82
```

```sh
-2122
```

```sh
334
```





## 问题分析

以上的结果可能是正数、负数、零。为什么呢？因为 Java 中对静态变量的自增，自减并不是原子操作，要彻底理 解，必须从字节码来进行分析 

例如对于 i++ 而言（i 为静态变量），实际会产生如下的 JVM 字节码指令：



```java
getstatic i   // 获取静态变量i的值
iconst_1      // 准备常量1
iadd           // 自增
putstatic i   // 将修改后的值存入静态变量i
```



而对应 i-- 也是类似

```java
getstatic i    // 获取静态变量i的值
iconst_1       // 准备常量1
isub          // 自减
putstatic i   // 将修改后的值存入静态变量i
```



```java
// class version 60.0 (60)
// access flags 0x20
class Test1$1 implements java/lang/Runnable {

  // compiled from: Test1.java
  NESTHOST Test1
  OUTERCLASS Test1 main ([Ljava/lang/String;)V
  // access flags 0x0
  INNERCLASS Test1$1 null null

  // access flags 0x0
  <init>()V
   L0
    LINENUMBER 21 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this LTest1$1; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x1
  public run()V
   L0
    LINENUMBER 25 L0
    ICONST_0
    ISTORE 1
   L1
   FRAME APPEND [I]
    ILOAD 1
    SIPUSH 10000
    IF_ICMPGE L2
   L3
    LINENUMBER 27 L3
    GETSTATIC Test1.count : I
    ICONST_1
    IADD
    PUTSTATIC Test1.count : I
   L4
    LINENUMBER 25 L4
    IINC 1 1
    GOTO L1
   L2
    LINENUMBER 29 L2
   FRAME CHOP 1
    RETURN
   L5
    LOCALVARIABLE i I L1 L2 1
    LOCALVARIABLE this LTest1$1; L0 L5 0
    MAXSTACK = 2
    MAXLOCALS = 2
}
```



而 Java 的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换



![image-20220827214622005](img/java并发编程学习笔记/image-20220827214622005.png)



如果是单线程，代码是顺序执行，不会交错，没有问题



![image-20220827214723506](img/java并发编程学习笔记/image-20220827214723506.png)





但多线程下，会出现问题



出现负数的情况：

![image-20220827214809053](img/java并发编程学习笔记/image-20220827214809053.png)





出现正数的情况：

![image-20220827215019209](img/java并发编程学习笔记/image-20220827215019209.png)







## 临界区

即Critical Section

* 一个程序运行多个线程本身是没有问题的
* 问题出在多个线程访问共享资源
  * 多个线程读共享资源其实也没有问题
  * 在多个线程对共享资源读写操作时发生指令交错，就会出现问题
* 一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为临界区



```java
/**
 * Project name(项目名称)：java并发编程_共享问题
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test1
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 21:33
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test1
{
    public static int count = 0;

    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 10000; i++)
                {
                    //临界区
                    count++;
                    //临界区结束
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 10000; i++)
                {
                    //临界区
                    count--;
                    //临界区结束
                }
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        //输出
        System.out.println(count);
    }
}

```





## 竞态条件

即Race Condition

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件





## 解决

为了避免临界区的竞态条件发生，有多种手段可以达到目的

* 阻塞式的解决方案：synchronized，Lock
* 非阻塞式的解决方案：原子变量





```java
/**
 * Project name(项目名称)：java并发编程_共享问题
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 21:55
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    public static int count = 0;

    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 10000; i++)
                {
                    synchronized (Test2.class)
                    {
                        //临界区
                        count++;
                        //临界区结束
                    }
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 10000; i++)
                {
                    synchronized (Test2.class)
                    {
                        //临界区
                        count--;
                        //临界区结束
                    }
                }
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        //输出
        System.out.println(count);
    }
}
```



运行结果：

```sh
0
```

```sh
0
```

```sh
0
```





## synchronized

语法：

```java
synchronized(对象)
{
   //临界区
}
```



* synchronized(对象) 中的对象，可以想象为一个房间（room），有唯一入口（门）房间只能一次进入一人 进行计算，线程 t1，t2 想象成两个人
* 当线程 t1 执行到 synchronized(room) 时就好比 t1 进入了这个房间，并锁住了门拿走了钥匙，在门内执行 count++ 代码
* 这时候如果 t2 也运行到了 synchronized(room) 时，它发现门被锁住了，只能在门外等待，发生了上下文切换，阻塞住了
* 这中间即使 t1 的 cpu 时间片不幸用完，被踢出了门外（不要错误理解为锁住了对象就能一直执行下去）， 这时门还是锁住的，t1 仍拿着钥匙，t2 线程还在阻塞状态进不来，只有下次轮到 t1 自己再次获得时间片时才能开门进入
* 当 t1 执行完 synchronized{} 块内的代码，这时候才会从 obj 房间出来并解开门上的锁，唤醒 t2 线程把钥 匙给他。t2 线程这时才可以进入 obj 房间，锁住了门拿上钥匙，执行它的 count-- 代码





![image-20220827220604717](img/java并发编程学习笔记/image-20220827220604717.png)

![image-20220827220628234](img/java并发编程学习笔记/image-20220827220628234.png)





synchronized 实际是用对象锁保证了临界区内代码的原子性，临界区内的代码对外是不可分割的，不会被线程切换所打断。





## 改进

把需要保护的共享变量放入一个类



```java
/**
 * Project name(项目名称)：java并发编程_共享问题
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 22:09
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test3
{

    public static void main(String[] args) throws InterruptedException
    {
        Room room = new Room();

        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 20000; i++)
                {
                    room.increment();
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 20000; i++)
                {
                    room.decrement();
                }
            }
        }, "t2");

        t1.start();
        t2.start();

        t1.join();
        t2.join();
        //打印
        System.out.println(room.get());
    }
}


class Room
{
    private int value = 0;

    /**
     * 加1
     */
    public void increment()
    {
        synchronized (this)
        {
            value++;
        }
    }

    /**
     * 减1
     */
    public void decrement()
    {
        synchronized (this)
        {
            value--;
        }
    }

    /**
     * 取值
     *
     * @return value
     */
    public int get()
    {
        synchronized (this)
        {
            return value;
        }
    }
}
```



运行结果：

```sh
0
```

```sh
0
```

```sh
0
```





## synchronized加在方法上

```java
class Test
{
    public synchronized void test()
    {

    }
}
```

等价于

```java
class Test
{
    public void test()
    {
        synchronized (this)
        {

        }
    }
}
```





```java
class Test
{
    public synchronized static void test()
    {
        
    }
}
```

等价于

```java
class Test
{
    public static void test()
    {
        synchronized (Test.class)
        {

        }
    }
}
```





```java
/**
 * Project name(项目名称)：java并发编程_共享问题
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test4
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 22:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test4
{
    public static void main(String[] args) throws InterruptedException
    {
        Room2 room = new Room2();

        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 20000; i++)
                {
                    room.increment();
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 20000; i++)
                {
                    room.decrement();
                }
            }
        }, "t2");

        t1.start();
        t2.start();

        t1.join();
        t2.join();
        //打印
        System.out.println(room.get());
    }
}


class Room2
{
    private int value = 0;

    /**
     * 加1
     */
    public synchronized void increment()
    {
        value++;
    }

    /**
     * 减1
     */
    public synchronized void decrement()
    {
        value--;
    }

    /**
     * 取值
     *
     * @return value
     */
    public synchronized int get()
    {
        return value;
    }
}

```



运行结果：

```sh
0
```

```sh
0
```

```sh
0
```







## 不加 synchronized 的方法

不加 synchronzied 的方法就好比不遵守规则的人，不去老实排队（好比翻窗户进去的）



```java
/**
 * Project name(项目名称)：java并发编程_共享问题
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test5
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/27
 * Time(创建时间)： 22:24
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test5
{
    public static void main(String[] args) throws InterruptedException
    {
        Room3 room = new Room3();

        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 20000; i++)
                {
                    room.increment();
                }
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                for (int i = 0; i < 20000; i++)
                {
                    room.decrement();
                }
            }
        }, "t2");

        t1.start();
        t2.start();

        t1.join();
        t2.join();
        //打印
        System.out.println(room.get());
    }
}

class Room3
{
    private int value = 0;

    /**
     * 加1
     */
    public synchronized void increment()
    {
        value++;
    }

    /**
     * 减1，未加同步锁
     */
    public void decrement()
    {
        value--;
    }

    /**
     * 取值
     *
     * @return value
     */
    public synchronized int get()
    {
        return value;
    }
}
```



运行结果：

```sh
436
```

```sh
-7257
```

```sh
-10274
```

```sh
-6992
```







## 线程八锁

考察 synchronized 锁住的是哪个对象



### 情况一

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 11:41
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        Number n = new Number();
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.a();
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.b();
            }
        },"t2").start();

    }

}


class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public synchronized void a()
    {
        log.debug("1");
    }

    public synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:01:16.732  [t1] DEBUG mao.t1.Number:  1
2022-08-28  12:01:16.734  [t2] DEBUG mao.t1.Number:  2
```



* 1和2几乎同时运行，先1后2，或者先2后1





### 情况二

```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:02
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
        Number n = new Number();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.b();
            }
        }, "t2");

        
        t1.start();
        t2.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:06:08.625  [main] INFO  mao.t2.Test:  开始
2022-08-28  12:06:09.642  [t1] DEBUG mao.t2.Number:  1
2022-08-28  12:06:09.642  [t2] DEBUG mao.t2.Number:  2
```

```sh
2022-08-28  12:08:05.119  [main] INFO  mao.t2.Test:  开始
2022-08-28  12:08:05.123  [t2] DEBUG mao.t2.Number:  2
2022-08-28  12:08:06.130  [t1] DEBUG mao.t2.Number:  1
```



* 1秒后打印1然后再打印2
* 开始立马打印2，1秒后再打印1





### 情况三

```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:10
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
        Number n = new Number();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.b();
            }
        }, "t2");

        Thread t3 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.c();
            }
        }, "t3");


        t1.start();
        t2.start();
        t3.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public synchronized void b()
    {
        log.debug("2");
    }

    public void c()
    {
        log.debug("3");
    }
}
```



运行结果：

````sh
2022-08-28  12:12:31.557  [main] INFO  mao.t3.Test:  开始
2022-08-28  12:12:31.560  [t3] DEBUG mao.t3.Number:  3
2022-08-28  12:12:32.569  [t1] DEBUG mao.t3.Number:  1
2022-08-28  12:12:32.569  [t2] DEBUG mao.t3.Number:  2
````

```sh
2022-08-28  12:14:14.919  [main] INFO  mao.t3.Test:  开始
2022-08-28  12:14:14.923  [t3] DEBUG mao.t3.Number:  3
2022-08-28  12:14:14.923  [t2] DEBUG mao.t3.Number:  2
2022-08-28  12:14:15.938  [t1] DEBUG mao.t3.Number:  1
```

```sh
2022-08-28  12:14:35.940  [main] INFO  mao.t3.Test:  开始
2022-08-28  12:14:35.943  [t2] DEBUG mao.t3.Number:  2
2022-08-28  12:14:35.944  [t3] DEBUG mao.t3.Number:  3
2022-08-28  12:14:36.950  [t1] DEBUG mao.t3.Number:  1
```



* 开始后打印3，1秒后打印1和2
* 开始后打印3，然后打印2,1秒后打印1
* 开始后打印2，然后打印3，1秒后打印1





### 情况四

```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(mao.t2.Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
        Number n1 = new Number();
        Number n2 = new Number();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n1.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n2.b();
            }
        }, "t2");


        t1.start();
        t2.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:20:09.809  [main] INFO  mao.t2.Test:  开始
2022-08-28  12:20:09.812  [t2] DEBUG mao.t4.Number:  2
2022-08-28  12:20:10.816  [t1] DEBUG mao.t4.Number:  1
```



* 开始后打印2，1秒后再打印1







### 情况五

```java
package mao.t5;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:29
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
       Number n = new Number();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                Number.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n.b();
            }
        }, "t2");


        t1.start();
        t2.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public static synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:31:57.157  [main] INFO  mao.t5.Test:  开始
2022-08-28  12:31:57.161  [t2] DEBUG mao.t5.Number:  2
2022-08-28  12:31:58.163  [t1] DEBUG mao.t5.Number:  1
```



* 开始打印2，1秒后打印1





### 情况六

```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:33
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                Number.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                Number.b();
            }
        }, "t2");


        t1.start();
        t2.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public static synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public static synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:34:55.050  [main] INFO  mao.t6.Test:  开始
2022-08-28  12:34:56.065  [t1] DEBUG mao.t6.Number:  1
2022-08-28  12:34:56.065  [t2] DEBUG mao.t6.Number:  2
```

```sh
2022-08-28  12:35:31.238  [main] INFO  mao.t6.Test:  开始
2022-08-28  12:35:31.242  [t2] DEBUG mao.t6.Number:  2
2022-08-28  12:35:32.247  [t1] DEBUG mao.t6.Number:  1
```



* 开始后，过1秒，打印1和2
* 开始后先打印2，过1秒再打印1





### 情况七

```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t7
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:37
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
        Number n1 = new Number();
        Number n2 = new Number();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n1.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n2.b();
            }
        }, "t2");


        t1.start();
        t2.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public static synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:39:07.774  [main] INFO  mao.t7.Test:  开始
2022-08-28  12:39:07.778  [t2] DEBUG mao.t7.Number:  2
2022-08-28  12:39:08.781  [t1] DEBUG mao.t7.Number:  1
```



* 开始后先打印2，1秒后再打印1





### 情况八

```java
package mao.t8;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_线程八锁
 * Package(包名): mao.t8
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 12:40
 * Version(版本): 1.0
 * Description(描述)： 无
 */


@SuppressWarnings("all")
public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.info("开始");
        Number n1 = new Number();
        Number n2 = new Number();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n1.a();
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                n2.b();
            }
        }, "t2");


        t1.start();
        t2.start();
    }
}

class Number
{
    private static final Logger log = LoggerFactory.getLogger(Number.class);

    public static synchronized void a()
    {
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        log.debug("1");
    }

    public static synchronized void b()
    {
        log.debug("2");
    }
}
```



运行结果：

```sh
2022-08-28  12:43:04.928  [main] INFO  mao.t8.Test:  开始
2022-08-28  12:43:05.933  [t1] DEBUG mao.t8.Number:  1
2022-08-28  12:43:05.933  [t2] DEBUG mao.t8.Number:  2
```

```sh
2022-08-28  12:43:40.942  [main] INFO  mao.t8.Test:  开始
2022-08-28  12:43:40.946  [t2] DEBUG mao.t8.Number:  2
2022-08-28  12:43:41.952  [t1] DEBUG mao.t8.Number:  1
```





* 1秒后打印1然后再打印2
* 开始立马打印2，1秒后再打印1







## 变量的线程安全



**成员变量和静态变量是否线程安全？**

* 如果它们没有共享，则线程安全
* 如果它们被共享了，根据它们的状态是否能够改变，又分两种情况
  * 如果只有读操作，则线程安全
  * 如果有读写操作，则这段代码是临界区，需要考虑线程安全



**局部变量是否线程安全？**

* 局部变量是线程安全的
* 但局部变量引用的对象则未必
  * 如果该对象没有逃离方法的作用范围，它是线程安全的
  * 如果该对象逃离方法的作用范围，需要考虑线程安全



```java
public static void test1() 
{
    int i = 10;
    i++;
}
```



每个线程调用 test1() 方法时局部变量 i，会在每个线程的栈帧内存中被创建多份，因此不存在共享



**局部变量引用的对象：**

```java
package mao.t1;

import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 13:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private final List<Integer> list = new ArrayList<>();

    public void method1(int loopNumber)
    {
        for (int i = 0; i < loopNumber; i++)
        {
            add();
            sub();
        }
    }

    private void add()
    {
        list.add(1);
    }

    private void sub()
    {
        list.remove(0);
    }

    public static void main(String[] args)
    {
        Test t = new Test();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```



运行结果：

```sh
Exception in thread "t2" java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
	at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64)
	at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70)
	at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266)
	at java.base/java.util.Objects.checkIndex(Objects.java:359)
	at java.base/java.util.ArrayList.remove(ArrayList.java:504)
	at mao.t1.Test.sub(Test.java:39)
	at mao.t1.Test.method1(Test.java:28)
	at mao.t1.Test$2.run(Test.java:59)
	at java.base/java.lang.Thread.run(Thread.java:831)
```





![image-20220828133730847](img/java并发编程学习笔记/image-20220828133730847.png)





ThreadUnsafe则为Test类



**将 list 修改为局部变量：**

```java
package mao.t2;

import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 13:40
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public void method1(int loopNumber)
    {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < loopNumber; i++)
        {
            add(list);
            sub(list);
        }
    }

    private void add(List<Integer> list)
    {
        list.add(1);
    }

    private void sub(List<Integer> list)
    {
        list.remove(0);
    }

    public static void main(String[] args)
    {
        Test t = new Test();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```



* list 是局部变量，每个线程调用时会创建其不同实例，没有共享
* 而 add的参数是从 method1 中传递过来的，与 method1 中引用同一个对象
* sub的参数分析与 method2 相同



![image-20220828134351257](img/java并发编程学习笔记/image-20220828134351257.png)







如果把 method2 和 method3 的方法修改为 public 会不会代理线程安全问题？

* 情况1：有其它线程调用 add和 sub
* 情况2：在 情况1 的基础上，为 Test类添加子类，子类覆盖 add或 sub方法



```java
package mao.t3;

import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/28
 * Time(创建时间)： 13:46
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public void method1(int loopNumber)
    {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < loopNumber; i++)
        {
            add(list);
            sub(list);
        }
    }

    public void add(List<Integer> list)
    {
        list.add(1);
    }

    public void sub(List<Integer> list)
    {
        list.remove(0);
    }

    public static void main(String[] args)
    {
        Test t = new Test();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}

class Test2 extends Test
{
    @Override
    public void sub(List<Integer> list)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                list.remove(0);
            }
        }).start();
    }

    public static void main(String[] args)
    {
        Test2 t = new Test2();
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.method1(500);
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```



运行结果：

```sh
Exception in thread "Thread-951" java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
	at java.base/jdk.internal.util.Preconditions.outOfBounds(Preconditions.java:64)
	at java.base/jdk.internal.util.Preconditions.outOfBoundsCheckIndex(Preconditions.java:70)
	at java.base/jdk.internal.util.Preconditions.checkIndex(Preconditions.java:266)
	at java.base/java.util.Objects.checkIndex(Objects.java:359)
	at java.base/java.util.ArrayList.remove(ArrayList.java:504)
	at mao.t3.Test2$1.run(Test.java:77)
	at java.base/java.lang.Thread.run(Thread.java:831)
```







## 常见线程安全类

* String
* Integer
* StringBuffer
* Random
* Vector
* Hashtable
* java.util.concurrent 包下的类



![image-20220828135912826](img/java并发编程学习笔记/image-20220828135912826.png)





这里说它们是线程安全的是指，多个线程调用它们同一个实例的某个方法时，是线程安全的

* 它们的每个方法是原子的
* 但是它们多个方法的组合不是原子的



```java
package mao.t4;

import java.util.Hashtable;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 19:54
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Hashtable<String, String> hashtable = new Hashtable<>();

    public static void m(String value)
    {
        if (hashtable.get("key") == null)
        {
            hashtable.put("key", value);
            System.out.println("put "+value);
        }
    }

    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                m("v1");
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                m("v2");
            }
        }, "t2").start();
    }
}

```



![image-20220829200214428](img/java并发编程学习笔记/image-20220829200214428.png)







## 不可变类线程安全性

String、Integer 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的



```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {

    /**
     * The value is used for character storage.
     *
     * @implNote This field is trusted by the VM, and is a subject to
     * constant folding if String instance is constant. Overwriting this
     * field after construction will cause problems.
     *
     * Additionally, it is marked with {@link Stable} to trust the contents
     * of the array. No other facility in JDK provides this functionality (yet).
     * {@link Stable} is safe here, because value is never null.
     */
    @Stable
    private final byte[] value;

    /**
     * The identifier of the encoding used to encode the bytes in
     * {@code value}. The supported values in this implementation are
     *
     * LATIN1
     * UTF16
     *
     * @implNote This field is trusted by the VM, and is a subject to
     * constant folding if String instance is constant. Overwriting this
     * field after construction will cause problems.
     */
    private final byte coder;

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /**
     * Cache if the hash has been calculated as actually being zero, enabling
     * us to avoid recalculating this.
     */
    private boolean hashIsZero; // Default to false;
                   
...

    /**
     * Returns a string that is a substring of this string. The
     * substring begins at the specified {@code beginIndex} and
     * extends to the character at index {@code endIndex - 1}.
     * Thus the length of the substring is {@code endIndex-beginIndex}.
     * <p>
     * Examples:
     * <blockquote><pre>
     * "hamburger".substring(4, 8) returns "urge"
     * "smiles".substring(1, 5) returns "mile"
     * </pre></blockquote>
     *
     * @param      beginIndex   the beginning index, inclusive.
     * @param      endIndex     the ending index, exclusive.
     * @return     the specified substring.
     * @throws     IndexOutOfBoundsException  if the
     *             {@code beginIndex} is negative, or
     *             {@code endIndex} is larger than the length of
     *             this {@code String} object, or
     *             {@code beginIndex} is larger than
     *             {@code endIndex}.
     */
    public String substring(int beginIndex, int endIndex) {
        int length = length();
        checkBoundsBeginEnd(beginIndex, endIndex, length);
        if (beginIndex == 0 && endIndex == length) {
            return this;
        }
        int subLen = endIndex - beginIndex;
        return isLatin1() ? StringLatin1.newString(value, beginIndex, subLen)
                          : StringUTF16.newString(value, beginIndex, subLen);
    }
}
```





例子：

```java
package mao.t5;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t5
 * Class(类名): Int
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:11
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Int
{
    private int value = 0;

    public Int(int value)
    {
        this.value = value;
    }

    public int getValue()
    {
        return this.value;
    }

    @Override
    public String toString()
    {
        return String.valueOf(value);
    }

    public Int add(int v)
    {
        return new Int(value + v);
    }

    public Int sub(int v)
    {
        return new Int(value - v);
    }
}

```



```java
package mao.t5;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:14
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        System.out.println(new Int(2));
        System.out.println(new Int(6).add(5));
    }
}
```





## 卖票问题



```java
package mao.t6;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t6
 * Class(类名): TicketWindow
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class TicketWindow
{
    private int count;

    public TicketWindow(int count)
    {
        this.count = count;
    }

    public int getCount()
    {
        return count;
    }

    public int sell(int amount)
    {
        if (this.count >= amount)
        {
            this.count -= amount;
            return amount;
        }
        else
        {
            return 0;
        }
    }
}
```



```java
package mao.t6;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.Vector;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t6
 * Class(类名): ExerciseSell
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:19
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ExerciseSell
{
    public static void main(String[] args)
    {
        TicketWindow ticketWindow = new TicketWindow(20000);
        List<Thread> list = new ArrayList<>();
        List<Integer> sellCount = new Vector<>();
        for (int i = 0; i < 10000; i++)
        {
            Thread t = new Thread(() ->
            {
                int count = ticketWindow.sell(randomAmount());
                sellCount.add(count);
            });
            list.add(t);
        }

        for (Thread thread : list)
        {
            thread.start();
        }

        list.forEach((t) ->
        {
            try
            {
                t.join();
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        });
        System.out.println("卖出去的票总数：" + sellCount.stream().mapToInt(c -> c).sum());
        System.out.println("剩余票数：" + ticketWindow.getCount());
    }

    // Random 为线程安全
    static Random random = new Random();

    // 随机 1~5
    public static int randomAmount()
    {
        return random.nextInt(5) + 1;
    }

}
```



```java
package mao.t6;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:28
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        for (int i = 0; i < 10; i++)
        {
            ExerciseSell.main(null);
        }
    }
}
```



运行结果：

```sh
卖出去的票总数：20006
剩余票数：0
```



解决：

加锁

```java
package mao.t6;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t6
 * Class(类名): TicketWindow
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class TicketWindow
{
    private int count;

    public TicketWindow(int count)
    {
        this.count = count;
    }

    public int getCount()
    {
        return count;
    }

    public synchronized int sell(int amount)
    {
        if (this.count >= amount)
        {
            this.count -= amount;
            return amount;
        }
        else
        {
            return 0;
        }
    }
}
```





## 转账问题

```java
package mao.t7;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t7
 * Class(类名): Account
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:36
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Account
{
    private int money;

    public Account(int money)
    {
        this.money = money;
    }

    public int getMoney()
    {
        return money;
    }

    public void setMoney(int money)
    {
        this.money = money;
    }

    /**
     * 转账
     *
     * @param target 对方的账户
     * @param amount 要转账的金额
     */
    public void transfer(Account target, int amount)
    {
        if (this.money > amount)
        {
            this.setMoney(this.getMoney() - amount);
            target.setMoney(target.getMoney() + amount);
        }
    }

}
```



```java
package mao.t7;

import java.util.Random;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t7
 * Class(类名): ExerciseTransfer
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:37
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ExerciseTransfer
{
    public static void main(String[] args) throws InterruptedException
    {
        Account a = new Account(1000);
        Account b = new Account(1000);

        Thread t1 = new Thread(() ->
        {
            for (int i = 0; i < 1000; i++)
            {
                a.transfer(b, randomAmount());
            }
        }, "t1");

        Thread t2 = new Thread(() ->
        {
            for (int i = 0; i < 1000; i++)
            {
                b.transfer(a, randomAmount());
            }
        }, "t2");

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        // 查看转账2000次后的总金额
        System.out.println("转账后双方的总金额:" + (a.getMoney() + b.getMoney()));
    }

    // Random 为线程安全
    static Random random = new Random();

    // 随机 1~100
    public static int randomAmount()
    {
        return random.nextInt(100) + 1;
    }
}
```



```java
package mao.t7;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t7
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:39
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args) throws InterruptedException
    {
        for (int i = 0; i < 10; i++)
        {
            ExerciseTransfer.main(null);
        }
    }
}
```



运行结果：

```sh
转账后双方的总金额:5271
转账后双方的总金额:4157
转账后双方的总金额:147
转账后双方的总金额:4125
转账后双方的总金额:6143
转账后双方的总金额:1883
转账后双方的总金额:48
转账后双方的总金额:1507
转账后双方的总金额:2000
转账后双方的总金额:2000
```



存在线程安全问题



加上锁住this对象：

```java
package mao.t7;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t7
 * Class(类名): Account
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:36
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Account
{
    private int money;

    public Account(int money)
    {
        this.money = money;
    }

    public int getMoney()
    {
        return money;
    }

    public void setMoney(int money)
    {
        this.money = money;
    }

    /**
     * 转账
     *
     * @param target 对方的账户
     * @param amount 要转账的金额
     */
    public synchronized void transfer(Account target, int amount)
    {
        if (this.money > amount)
        {
            this.setMoney(this.getMoney() - amount);
            target.setMoney(target.getMoney() + amount);
        }
    }

}
```



运行结果：

```sh
转账后双方的总金额:4434
转账后双方的总金额:4963
转账后双方的总金额:2239
转账后双方的总金额:2375
转账后双方的总金额:2350
转账后双方的总金额:771
转账后双方的总金额:1547
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2289
```



也有线程安全问题

因为当前方法操作了this和target两个Account类的对象



需要锁住类的字节码



```java
package mao.t7;

/**
 * Project name(项目名称)：java并发编程_线程安全
 * Package(包名): mao.t7
 * Class(类名): Account
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/29
 * Time(创建时间)： 20:36
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Account
{
    private int money;

    public Account(int money)
    {
        this.money = money;
    }

    public int getMoney()
    {
        return money;
    }

    public void setMoney(int money)
    {
        this.money = money;
    }

    /**
     * 转账
     *
     * @param target 对方的账户
     * @param amount 要转账的金额
     */
    public void transfer(Account target, int amount)
    {
        synchronized (Account.class)
        {
            if (this.money > amount)
            {
                this.setMoney(this.getMoney() - amount);
                target.setMoney(target.getMoney() + amount);
            }
        }
    }

}
```



运行结果：

```sh
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
转账后双方的总金额:2000
```







## Monitor

Java 对象头



普通对象：

```
|--------------------------------------------------------------|
|                    Object Header (64 bits)                   |
|------------------------------------|-------------------------|
|                Mark Word (32 bits) | Klass Word (32 bits)    |
|------------------------------------|-------------------------|
```



数组对象：

```
|---------------------------------------------------------------------------------|
|                         Object Header (96 bits)                                 |
|--------------------------------|-----------------------|------------------------|
|              Mark Word(32bits) | Klass Word(32bits)    | array length(32bits)   |
|--------------------------------|-----------------------|------------------------|
```





其中 Mark Word 结构为：

```
|-------------------------------------------------------|--------------------|
|                                   Mark Word (32 bits) |              State |
|-------------------------------------------------------|--------------------|
| hashcode:25         | age:4 | biased_lock:0      | 01 |             Normal |
|-------------------------------------------------------|--------------------|
| thread:23 | epoch:2 | age:4 | biased_lock:1      | 01 |             Biased |
|-------------------------------------------------------|--------------------|
| ptr_to_lock_record:30                            | 00 | Lightweight Locked |
|-------------------------------------------------------|--------------------|
| ptr_to_heavyweight_monitor:30                    | 10 | Heavyweight Locked |
|-------------------------------------------------------|--------------------|
|                                                  | 11 |      Marked for GC |
|-------------------------------------------------------|--------------------|
```





## Monitor 原理

Monitor 被翻译为监视器或管程

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象的指针



Monitor 结构：

![image-20220830210226716](img/java并发编程学习笔记/image-20220830210226716.png)





* 刚开始 Monitor 中 Owner 为 null
* 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一 个 Owner
* 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入 EntryList ， BLOCKED状态
* Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争的时是非公平的
* WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程







## synchronized 原理

### 轻量级锁

轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以 使用轻量级锁来优化。

轻量级锁对使用者是透明的，即语法仍然是 synchronized



假设有两个方法同步块，利用同一个对象加锁：

```java
static final Object obj = new Object();
public static void method1() 
{
 synchronized( obj ) 
 {
 // 同步块 A
 method2();
 }
}
public static void method2() 
{
 synchronized( obj ) 
 {
 // 同步块 B
 }
}
```



创建锁记录（Lock Record）对象，每个线程都的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的 Mark Word



![image-20220830211536104](img/java并发编程学习笔记/image-20220830211536104.png)



让锁记录中 Object reference 指向锁对象，并尝试用 cas 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录



![image-20220830211623427](img/java并发编程学习笔记/image-20220830211623427.png)



如果 cas 替换成功，对象头中存储了锁记录地址和状态 00 ，表示由该线程给对象加锁



![image-20220830211811180](img/java并发编程学习笔记/image-20220830211811180.png)



如果 cas 失败，有两种情况

* 如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程
* 如果是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数



![image-20220830211913699](img/java并发编程学习笔记/image-20220830211913699.png)



当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重入计数减一



![image-20220830211941686](img/java并发编程学习笔记/image-20220830211941686.png)



当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 cas 将 Mark Word 的值恢复给对象头

* 成功，则解锁成功
* 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程





### 锁膨胀

如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁



当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁



![image-20220831204351507](img/java并发编程学习笔记/image-20220831204351507.png)





这时 Thread-1 加轻量级锁失败，进入锁膨胀流程

* 即为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址
* 然后自己进入 Monitor 的 EntryList BLOCKED



![image-20220831204443346](img/java并发编程学习笔记/image-20220831204443346.png)



当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程





### 自旋优化

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞



* 自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势
* 在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会 高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能
* Java 7 之后不能控制是否开启自旋功能





### 偏向锁

轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作

Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现 这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有



```java
static final Object obj = new Object();
public static void m1() 
{
 synchronized( obj ) 
 {
 // 同步块 A
 m2();
 }
}
public static void m2() 
{
 synchronized( obj ) 
 {
 // 同步块 B
 m3();
 }
}
public static void m3() 
{
 synchronized( obj ) 
 {
    // 同步块 C
 }
}
```





![image-20220831205511909](img/java并发编程学习笔记/image-20220831205511909.png)



![image-20220831205524485](img/java并发编程学习笔记/image-20220831205524485.png)



一个对象创建时：

* 如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的 thread、epoch、age 都为 0
* 偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数 -XX:BiasedLockingStartupDelay=0 来禁用延迟
* 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 hashcode、 age 都为 0，第一次用到 hashcode 时才会赋值





