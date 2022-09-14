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





添加依赖：

```xml
        <dependency>
            <groupId>org.openjdk.jol</groupId>
            <artifactId>jol-core</artifactId>
            <version>0.16</version>
        </dependency>
```





 测试偏向锁：

```java
package mao.t1;


import org.openjdk.jol.info.ClassLayout;

/**
 * Project name(项目名称)：java并发编程_synchronized原理
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/31
 * Time(创建时间)： 21:27
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    public static void main(String[] args) throws InterruptedException
    {
        Lock lock = new Lock();
        ClassLayout classLayout = ClassLayout.parseInstance(lock);

        System.out.println(classLayout.toPrintable());

        new Thread(() ->
        {
            System.out.println("synchronized 前");
            System.out.println(classLayout.toPrintable());
            synchronized (lock)
            {
                System.out.println("synchronized 中");
                System.out.println(classLayout.toPrintable());
            }
            System.out.println("synchronized 后");
            System.out.println(classLayout.toPrintable());
        }, "t1").start();
    }

}

class Lock
{

}
```



运行结果：

```sh
mao.t1.Lock object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4        (object header: class)    0x00180240
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

synchronized 前
mao.t1.Lock object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4        (object header: class)    0x00180240
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

synchronized 中
mao.t1.Lock object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x000000f734bff150 (thin lock: 0x000000f734bff150)
  8   4        (object header: class)    0x00180240
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

synchronized 后
mao.t1.Lock object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000001 (non-biasable; age: 0)
  8   4        (object header: class)    0x00180240
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```





### 撤销 - 调用对象 hashCode

调用了对象的 hashCode，但偏向锁的对象 MarkWord 中存储的是线程 id，如果调用 hashCode 会导致偏向锁被撤销

* 轻量级锁会在锁记录中记录 hashCode
* 重量级锁会在 Monitor 中记录 hashCode





### 撤销 - 其它线程使用对象

当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁





### 批量重偏向

如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象 的 Thread ID

当撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至加锁线程





### 批量撤销

当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象 都会变为不可偏向的，新建的对象也是不可偏向的







## wait notify

* 由于条件不满足，小南不能继续进行计算
* 但小南如果一直占用着锁，其它人就得一直阻塞，效率太低
* 于是老王单开了一间休息室（调用 wait 方法），让小南到休息室（WaitSet）等着去了，但这时锁释放开，其它人可以由老王随机安排进屋
* 直到小女进来，调用 notify 方法
* 小南于是可以离开休息室，重新进入竞争锁的队列



### API

* obj.wait() 让进入 object 监视器的线程到 waitSet 等待
* obj.notify() 在 object 上正在 waitSet 等待的线程中挑一个唤醒
* obj.notifyAll() 让 object 上正在 waitSet 等待的线程全部唤醒





```java
public class Object {

    /**
     * Constructs a new object.
     */
    @IntrinsicCandidate
    public Object() {}

    /**
     * Wakes up a single thread that is waiting on this object's
     * monitor. If any threads are waiting on this object, one of them
     * is chosen to be awakened. The choice is arbitrary and occurs at
     * the discretion of the implementation. A thread waits on an object's
     * monitor by calling one of the {@code wait} methods.
     * <p>
     * The awakened thread will not be able to proceed until the current
     * thread relinquishes the lock on this object. The awakened thread will
     * compete in the usual manner with any other threads that might be
     * actively competing to synchronize on this object; for example, the
     * awakened thread enjoys no reliable privilege or disadvantage in being
     * the next thread to lock this object.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. A thread becomes the owner of the
     * object's monitor in one of three ways:
     * <ul>
     * <li>By executing a synchronized instance method of that object.
     * <li>By executing the body of a {@code synchronized} statement
     *     that synchronizes on the object.
     * <li>For objects of type {@code Class,} by executing a
     *     synchronized static method of that class.
     * </ul>
     * <p>
     * Only one thread at a time can own an object's monitor.
     *
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @see        java.lang.Object#notifyAll()
     * @see        java.lang.Object#wait()
     */
    @IntrinsicCandidate
    public final native void notify();

    /**
     * Wakes up all threads that are waiting on this object's monitor. A
     * thread waits on an object's monitor by calling one of the
     * {@code wait} methods.
     * <p>
     * The awakened threads will not be able to proceed until the current
     * thread relinquishes the lock on this object. The awakened threads
     * will compete in the usual manner with any other threads that might
     * be actively competing to synchronize on this object; for example,
     * the awakened threads enjoy no reliable privilege or disadvantage in
     * being the next thread to lock this object.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#wait()
     */
    @IntrinsicCandidate
    public final native void notifyAll();

    /**
     * Causes the current thread to wait until it is awakened, typically
     * by being <em>notified</em> or <em>interrupted</em>.
     * <p>
     * In all respects, this method behaves as if {@code wait(0L, 0)}
     * had been called. See the specification of the {@link #wait(long, int)} method
     * for details.
     *
     * @throws IllegalMonitorStateException if the current thread is not
     *         the owner of the object's monitor
     * @throws InterruptedException if any thread interrupted the current thread before or
     *         while the current thread was waiting. The <em>interrupted status</em> of the
     *         current thread is cleared when this exception is thrown.
     * @see    #notify()
     * @see    #notifyAll()
     * @see    #wait(long)
     * @see    #wait(long, int)
     */
    public final void wait() throws InterruptedException {
        wait(0L);
    }

    /**
     * Causes the current thread to wait until it is awakened, typically
     * by being <em>notified</em> or <em>interrupted</em>, or until a
     * certain amount of real time has elapsed.
     * <p>
     * In all respects, this method behaves as if {@code wait(timeoutMillis, 0)}
     * had been called. See the specification of the {@link #wait(long, int)} method
     * for details.
     *
     * @param  timeoutMillis the maximum time to wait, in milliseconds
     * @throws IllegalArgumentException if {@code timeoutMillis} is negative
     * @throws IllegalMonitorStateException if the current thread is not
     *         the owner of the object's monitor
     * @throws InterruptedException if any thread interrupted the current thread before or
     *         while the current thread was waiting. The <em>interrupted status</em> of the
     *         current thread is cleared when this exception is thrown.
     * @see    #notify()
     * @see    #notifyAll()
     * @see    #wait()
     * @see    #wait(long, int)
     */
    public final native void wait(long timeoutMillis) throws InterruptedException;

    /**
     * Causes the current thread to wait until it is awakened, typically
     * by being <em>notified</em> or <em>interrupted</em>, or until a
     * certain amount of real time has elapsed.
     * <p>
     * The current thread must own this object's monitor lock. See the
     * {@link #notify notify} method for a description of the ways in which
     * a thread can become the owner of a monitor lock.
     * <p>
     * This method causes the current thread (referred to here as <var>T</var>) to
     * place itself in the wait set for this object and then to relinquish any
     * and all synchronization claims on this object. Note that only the locks
     * on this object are relinquished; any other objects on which the current
     * thread may be synchronized remain locked while the thread waits.
     * <p>
     * Thread <var>T</var> then becomes disabled for thread scheduling purposes
     * and lies dormant until one of the following occurs:
     * <ul>
     * <li>Some other thread invokes the {@code notify} method for this
     * object and thread <var>T</var> happens to be arbitrarily chosen as
     * the thread to be awakened.
     * <li>Some other thread invokes the {@code notifyAll} method for this
     * object.
     * <li>Some other thread {@linkplain Thread#interrupt() interrupts}
     * thread <var>T</var>.
     * <li>The specified amount of real time has elapsed, more or less.
     * The amount of real time, in nanoseconds, is given by the expression
     * {@code 1000000 * timeoutMillis + nanos}. If {@code timeoutMillis} and {@code nanos}
     * are both zero, then real time is not taken into consideration and the
     * thread waits until awakened by one of the other causes.
     * <li>Thread <var>T</var> is awakened spuriously. (See below.)
     * </ul>
     * <p>
     * The thread <var>T</var> is then removed from the wait set for this
     * object and re-enabled for thread scheduling. It competes in the
     * usual manner with other threads for the right to synchronize on the
     * object; once it has regained control of the object, all its
     * synchronization claims on the object are restored to the status quo
     * ante - that is, to the situation as of the time that the {@code wait}
     * method was invoked. Thread <var>T</var> then returns from the
     * invocation of the {@code wait} method. Thus, on return from the
     * {@code wait} method, the synchronization state of the object and of
     * thread {@code T} is exactly as it was when the {@code wait} method
     * was invoked.
     * <p>
     * A thread can wake up without being notified, interrupted, or timing out, a
     * so-called <em>spurious wakeup</em>.  While this will rarely occur in practice,
     * applications must guard against it by testing for the condition that should
     * have caused the thread to be awakened, and continuing to wait if the condition
     * is not satisfied. See the example below.
     * <p>
     * For more information on this topic, see section 14.2,
     * "Condition Queues," in Brian Goetz and others' <em>Java Concurrency
     * in Practice</em> (Addison-Wesley, 2006) or Item 69 in Joshua
     * Bloch's <em>Effective Java, Second Edition</em> (Addison-Wesley,
     * 2008).
     * <p>
     * If the current thread is {@linkplain java.lang.Thread#interrupt() interrupted}
     * by any thread before or while it is waiting, then an {@code InterruptedException}
     * is thrown.  The <em>interrupted status</em> of the current thread is cleared when
     * this exception is thrown. This exception is not thrown until the lock status of
     * this object has been restored as described above.
     *
     * @apiNote
     * The recommended approach to waiting is to check the condition being awaited in
     * a {@code while} loop around the call to {@code wait}, as shown in the example
     * below. Among other things, this approach avoids problems that can be caused
     * by spurious wakeups.
     *
     * <pre>{@code
     *     synchronized (obj) {
     *         while (<condition does not hold> and <timeout not exceeded>) {
     *             long timeoutMillis = ... ; // recompute timeout values
     *             int nanos = ... ;
     *             obj.wait(timeoutMillis, nanos);
     *         }
     *         ... // Perform action appropriate to condition or timeout
     *     }
     * }</pre>
     *
     * @param  timeoutMillis the maximum time to wait, in milliseconds
     * @param  nanos   additional time, in nanoseconds, in the range 0-999999 inclusive
     * @throws IllegalArgumentException if {@code timeoutMillis} is negative,
     *         or if the value of {@code nanos} is out of range
     * @throws IllegalMonitorStateException if the current thread is not
     *         the owner of the object's monitor
     * @throws InterruptedException if any thread interrupted the current thread before or
     *         while the current thread was waiting. The <em>interrupted status</em> of the
     *         current thread is cleared when this exception is thrown.
     * @see    #notify()
     * @see    #notifyAll()
     * @see    #wait()
     * @see    #wait(long)
     */
    public final void wait(long timeoutMillis, int nanos) throws InterruptedException {
        if (timeoutMillis < 0) {
            throw new IllegalArgumentException("timeoutMillis value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0 && timeoutMillis < Long.MAX_VALUE) {
            timeoutMillis++;
        }

        wait(timeoutMillis);
    }
}
```





### sleep和 wait的区别

* sleep 是 Thread 方法，而 wait 是 Object 的方法
* sleep 不需要强制和 synchronized 配合使用，但 wait 需要 和 synchronized 一起用
* sleep 在睡眠的同时，不会释放对象锁，但 wait 在等待的时候会释放对象锁
* 它们的状态为 TIMED_WAITING





### 使用

notify唤醒一个

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private final static Object lock = new Object();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);
    
    
    public static void main(String[] args) throws InterruptedException
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t1....");
                    try
                    {
                        //等待
                        lock.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t1被唤醒....");
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t2....");
                    try
                    {
                        lock.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t2被唤醒....");
                }
            }
        }, "t2").start();


        Thread.sleep(300);
        log.debug("唤醒 obj 上其它线程");
        synchronized (lock)
        {
            //唤醒
            lock.notify();
        }
    }
}
```



运行结果：

```sh
2022-09-01  20:26:59.527  [t1] DEBUG mao.t1.Test:  执行t1....
2022-09-01  20:26:59.529  [t2] DEBUG mao.t1.Test:  执行t2....
2022-09-01  20:26:59.837  [main] DEBUG mao.t1.Test:  唤醒 obj 上其它线程
2022-09-01  20:26:59.837  [t1] DEBUG mao.t1.Test:  t1被唤醒....
```





notifyAll唤醒全部：

```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:27
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private final static Object lock = new Object();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t1....");
                    try
                    {
                        //等待
                        lock.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t1被唤醒....");
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t2....");
                    try
                    {
                        lock.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t2被唤醒....");
                }
            }
        }, "t2").start();


        Thread.sleep(300);
        log.debug("唤醒 obj 上其它线程");
        synchronized (lock)
        {
            //唤醒全部
            lock.notifyAll();
        }
    }
}
```



运行结果：

```sh
2022-09-01  20:28:32.332  [t1] DEBUG mao.t2.Test:  执行t1....
2022-09-01  20:28:32.334  [t2] DEBUG mao.t2.Test:  执行t2....
2022-09-01  20:28:32.637  [main] DEBUG mao.t2.Test:  唤醒 obj 上其它线程
2022-09-01  20:28:32.637  [t1] DEBUG mao.t2.Test:  t1被唤醒....
2022-09-01  20:28:32.637  [t2] DEBUG mao.t2.Test:  t2被唤醒....
```



wait() 方法会释放对象的锁，进入 WaitSet 等待区，从而让其他线程就机会获取对象的锁。无限制等待，直到 notify 为止

wait(long n) 有时限的等待, 到 n 毫秒后结束等待，或是被 notify



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:30
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private final static Object lock = new Object();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t1....");
                    try
                    {
                        //等待
                        lock.wait(500);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t1被唤醒....");
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t2....");
                    try
                    {
                        lock.wait(200);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t2被唤醒....");
                }
            }
        }, "t2").start();


        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("执行t3....");
                    try
                    {
                        lock.wait(2000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t3被唤醒....");
                }
            }
        }, "t3").start();

        Thread.sleep(1000);
        synchronized (lock)
        {
            lock.notify();
        }
    }
}
```



运行结果：

```sh
2022-09-01  20:33:48.912  [t1] DEBUG mao.t3.Test:  执行t1....
2022-09-01  20:33:48.914  [t3] DEBUG mao.t3.Test:  执行t3....
2022-09-01  20:33:48.914  [t2] DEBUG mao.t3.Test:  执行t2....
2022-09-01  20:33:49.128  [t2] DEBUG mao.t3.Test:  t2被唤醒....
2022-09-01  20:33:49.414  [t1] DEBUG mao.t3.Test:  t1被唤醒....
2022-09-01  20:33:49.911  [t3] DEBUG mao.t3.Test:  t3被唤醒....
```







### 示例

```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:36
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 房间
     */
    private static final Object room = new Object();

    /**
     * 是否有香烟
     */
    private static boolean hasCigarette = false;

    /**
     * 是否有外卖
     */
    private static boolean hasTakeout = false;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette)
                {
                    log.debug("没烟，先歇会！");
                    try
                    {
                        Thread.sleep(2000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette)
                {
                    log.debug("可以开始干活了");
                }
            }
        }, "小南").start();

        for (int i = 0; i < 5; i++)
        {
            new Thread(() ->
            {
                synchronized (room)
                {
                    log.debug("可以开始干活了");
                }
            }, "其它人").start();
        }

        Thread.sleep(1000);

        new Thread(() ->
        {
            hasCigarette = true;
            log.debug("烟到了噢！");
        }, "送烟的").start();
    }

}
```



运行结果：

```sh
2022-09-01  20:41:29.374  [小南] DEBUG mao.t4.Test:  有烟没？[false]
2022-09-01  20:41:29.376  [小南] DEBUG mao.t4.Test:  没烟，先歇会！
2022-09-01  20:41:30.373  [送烟的] DEBUG mao.t4.Test:  烟到了噢！
2022-09-01  20:41:31.379  [小南] DEBUG mao.t4.Test:  有烟没？[true]
2022-09-01  20:41:31.379  [小南] DEBUG mao.t4.Test:  可以开始干活了
2022-09-01  20:41:31.379  [其它人] DEBUG mao.t4.Test:  可以开始干活了
2022-09-01  20:41:31.380  [其它人] DEBUG mao.t4.Test:  可以开始干活了
2022-09-01  20:41:31.380  [其它人] DEBUG mao.t4.Test:  可以开始干活了
2022-09-01  20:41:31.381  [其它人] DEBUG mao.t4.Test:  可以开始干活了
2022-09-01  20:41:31.381  [其它人] DEBUG mao.t4.Test:  可以开始干活了
```





问题：

*  其它干活的线程，都要一直阻塞，效率太低
* 小南线程必须睡足 2s 后才能醒来，就算烟提前送到，也无法立刻醒来
* 加了 synchronized (room) 后，就好比小南在里面反锁了门睡觉，烟根本没法送进门，main 没加 synchronized 就好像 main 线程是翻窗户进来的
* 解决方法，使用 wait - notify 机制





```java
package mao.t5;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:43
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 房间
     */
    private static final Object room = new Object();

    /**
     * 是否有香烟
     */
    private static boolean hasCigarette = false;

    /**
     * 是否有外卖
     */
    private static boolean hasTakeout = false;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette)
                {
                    log.debug("没烟，先歇会！");
                    try
                    {
                        room.wait(2000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette)
                {
                    log.debug("可以开始干活了");
                }
            }
        }, "小南").start();

        for (int i = 0; i < 5; i++)
        {
            new Thread(() ->
            {
                synchronized (room)
                {
                    log.debug("可以开始干活了");
                }
            }, "其它人").start();
        }


        Thread.sleep(1000);

        new Thread(() ->
        {
            synchronized (room)
            {
                hasCigarette = true;
                log.debug("烟到了噢！");
                room.notify();
            }
        }, "送烟的").start();

    }
}
```



运行结果：

```sh
2022-09-01  20:45:47.213  [小南] DEBUG mao.t5.Test:  有烟没？[false]
2022-09-01  20:45:47.216  [小南] DEBUG mao.t5.Test:  没烟，先歇会！
2022-09-01  20:45:47.216  [其它人] DEBUG mao.t5.Test:  可以开始干活了
2022-09-01  20:45:47.217  [其它人] DEBUG mao.t5.Test:  可以开始干活了
2022-09-01  20:45:47.217  [其它人] DEBUG mao.t5.Test:  可以开始干活了
2022-09-01  20:45:47.217  [其它人] DEBUG mao.t5.Test:  可以开始干活了
2022-09-01  20:45:47.217  [其它人] DEBUG mao.t5.Test:  可以开始干活了
2022-09-01  20:45:48.210  [送烟的] DEBUG mao.t5.Test:  烟到了噢！
2022-09-01  20:45:48.211  [小南] DEBUG mao.t5.Test:  有烟没？[true]
2022-09-01  20:45:48.211  [小南] DEBUG mao.t5.Test:  可以开始干活了
```



问题：

*  解决了其它干活的线程阻塞的问题
* 但如果有其它线程也在等待条件呢？还需要进一步解决





```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:47
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 房间
     */
    private static final Object room = new Object();

    /**
     * 是否有香烟
     */
    private static boolean hasCigarette = false;

    /**
     * 是否有外卖
     */
    private static boolean hasTakeout = false;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette)
                {
                    log.debug("没烟，先歇会！");
                    try
                    {
                        room.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette)
                {
                    log.debug("可以开始干活了");
                }
                else
                {
                    log.debug("没干成活...");
                }
            }
        }, "小南").start();

        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (!hasTakeout)
                {
                    log.debug("没外卖，先歇会！");
                    try
                    {
                        room.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (hasTakeout)
                {
                    log.debug("可以开始干活了");
                }
                else
                {
                    log.debug("没干成活...");
                }
            }
        }, "小女").start();

        Thread.sleep(1000);

        new Thread(() ->
        {
            synchronized (room)
            {
                hasTakeout = true;
                log.debug("外卖到了噢！");
                room.notify();
            }
        }, "送外卖的").start();
    }
}
```



错误结果：

```sh
2022-09-01  20:50:18.165  [小南] DEBUG mao.t6.Test:  有烟没？[false]
2022-09-01  20:50:18.168  [小南] DEBUG mao.t6.Test:  没烟，先歇会！
2022-09-01  20:50:18.168  [小女] DEBUG mao.t6.Test:  外卖送到没？[false]
2022-09-01  20:50:18.168  [小女] DEBUG mao.t6.Test:  没外卖，先歇会！
2022-09-01  20:50:19.177  [送外卖的] DEBUG mao.t6.Test:  外卖到了噢！
2022-09-01  20:50:19.177  [小南] DEBUG mao.t6.Test:  有烟没？[false]
2022-09-01  20:50:19.177  [小南] DEBUG mao.t6.Test:  没干成活...
```



问题：

* notify 只能随机唤醒一个 WaitSet 中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线程，称之为【虚假唤醒】
* 解决方法，改为 notifyAll





```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t7
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:53
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 房间
     */
    private static final Object room = new Object();

    /**
     * 是否有香烟
     */
    private static boolean hasCigarette = false;

    /**
     * 是否有外卖
     */
    private static boolean hasTakeout = false;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("有烟没？[{}]", hasCigarette);
                if (!hasCigarette)
                {
                    log.debug("没烟，先歇会！");
                    try
                    {
                        room.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette)
                {
                    log.debug("可以开始干活了");
                }
                else
                {
                    log.debug("没干成活...");
                }
            }
        }, "小南").start();

        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (!hasTakeout)
                {
                    log.debug("没外卖，先歇会！");
                    try
                    {
                        room.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (hasTakeout)
                {
                    log.debug("可以开始干活了");
                }
                else
                {
                    log.debug("没干成活...");
                }
            }
        }, "小女").start();

        Thread.sleep(1000);

        new Thread(() ->
        {
            synchronized (room)
            {
                hasTakeout = true;
                log.debug("外卖到了噢！");
                room.notifyAll();
            }
        }, "送外卖的").start();
    }
}
```



运行结果：

```sh
2022-09-01  20:55:15.165  [小南] DEBUG mao.t7.Test:  有烟没？[false]
2022-09-01  20:55:15.167  [小南] DEBUG mao.t7.Test:  没烟，先歇会！
2022-09-01  20:55:15.167  [小女] DEBUG mao.t7.Test:  外卖送到没？[false]
2022-09-01  20:55:15.168  [小女] DEBUG mao.t7.Test:  没外卖，先歇会！
2022-09-01  20:55:16.164  [送外卖的] DEBUG mao.t7.Test:  外卖到了噢！
2022-09-01  20:55:16.165  [小南] DEBUG mao.t7.Test:  有烟没？[false]
2022-09-01  20:55:16.165  [小南] DEBUG mao.t7.Test:  没干成活...
2022-09-01  20:55:16.165  [小女] DEBUG mao.t7.Test:  外卖送到没？[true]
2022-09-01  20:55:16.165  [小女] DEBUG mao.t7.Test:  可以开始干活了
```



问题：

* 用 notifyAll 仅解决某个线程的唤醒问题，但使用 if + wait 判断仅有一次机会，一旦条件不成立，就没有重新 判断的机会了
* 解决方法，用 while + wait，当条件不成立，再次 wait



```java
package mao.t8;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_wait_notify
 * Package(包名): mao.t8
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 20:56
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 房间
     */
    private static final Object room = new Object();

    /**
     * 是否有香烟
     */
    private static boolean hasCigarette = false;

    /**
     * 是否有外卖
     */
    private static boolean hasTakeout = false;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("有烟没？[{}]", hasCigarette);
                while (!hasCigarette)
                {
                    log.debug("没烟，先歇会！");
                    try
                    {
                        room.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("有烟没？[{}]", hasCigarette);
                }
                if (hasCigarette)
                {
                    log.debug("可以开始干活了");
                }
                else
                {
                    log.debug("没干成活...");
                }
            }
        }, "小南").start();

        new Thread(() ->
        {
            synchronized (room)
            {
                log.debug("外卖送到没？[{}]", hasTakeout);
                while (!hasTakeout)
                {
                    log.debug("没外卖，先歇会！");
                    try
                    {
                        room.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("外卖送到没？[{}]", hasTakeout);
                }
                if (hasTakeout)
                {
                    log.debug("可以开始干活了");
                }
                else
                {
                    log.debug("没干成活...");
                }
            }
        }, "小女").start();

        Thread.sleep(1000);

        new Thread(() ->
        {
            synchronized (room)
            {
                hasTakeout = true;
                log.debug("外卖到了噢！");
                room.notifyAll();
            }
        }, "送外卖的").start();

        Thread.sleep(2000);

        new Thread(() ->
        {
            synchronized (room)
            {
                hasCigarette = true;
                log.debug("烟到了噢！");
                room.notifyAll();
            }
        }, "送烟的").start();
    }
}
```



运行结果：

```sh
2022-09-01  21:00:25.420  [小南] DEBUG mao.t8.Test:  有烟没？[false]
2022-09-01  21:00:25.423  [小南] DEBUG mao.t8.Test:  没烟，先歇会！
2022-09-01  21:00:25.423  [小女] DEBUG mao.t8.Test:  外卖送到没？[false]
2022-09-01  21:00:25.423  [小女] DEBUG mao.t8.Test:  没外卖，先歇会！
2022-09-01  21:00:26.429  [送外卖的] DEBUG mao.t8.Test:  外卖到了噢！
2022-09-01  21:00:26.429  [小南] DEBUG mao.t8.Test:  有烟没？[false]
2022-09-01  21:00:26.429  [小南] DEBUG mao.t8.Test:  没烟，先歇会！
2022-09-01  21:00:26.429  [小女] DEBUG mao.t8.Test:  外卖送到没？[true]
2022-09-01  21:00:26.429  [小女] DEBUG mao.t8.Test:  可以开始干活了
2022-09-01  21:00:28.440  [送烟的] DEBUG mao.t8.Test:  烟到了噢！
2022-09-01  21:00:28.440  [小南] DEBUG mao.t8.Test:  有烟没？[true]
2022-09-01  21:00:28.441  [小南] DEBUG mao.t8.Test:  可以开始干活了
```







## 模式之保护性暂停

### 定义

即 Guarded Suspension，用在一个线程等待另一个线程的执行结果

* 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个 GuardedObject
* 如果有结果不断从一个线程到另一个线程那么可以使用消息队列
* JDK 中，join 的实现、Future 的实现，采用的就是此模式
* 因为要等待另一方的结果，因此归类到同步模式





### 实现

```java
package mao.t1;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t1
 * Class(类名): GuardedObject
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 21:39
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class GuardedObject
{
    /**
     * 响应
     */
    private Object response;
    /**
     * 锁
     */

    private final Object lock = new Object();

    /**
     * 得到响应
     *
     * @return {@link Object}
     */
    public Object getResponse()
    {
        synchronized (lock)
        {
            while (response == null)
            {
                try
                {
                    lock.wait();
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
            return response;
        }
    }

    /**
     * 完成
     *
     * @param response 响应
     */
    public void complete(Object response)
    {
        synchronized (lock)
        {
            this.response = response;
            lock.notifyAll();
        }
    }
}
```



```java
package mao.t1;

import java.util.Date;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/1
 * Time(创建时间)： 21:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        GuardedObject guardedObject = new GuardedObject();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                //休眠2秒
                try
                {
                    Thread.sleep(2000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                //返回结果
                guardedObject.complete(Math.random());

            }
        }, "t1").start();

        //主线程等待
        System.out.println("主线程开始获取t1线程的运行结果");
        System.out.println(new Date());
        Object response = guardedObject.getResponse();
        System.out.println("结果为" + response);
        System.out.println(new Date());
    }
}
```



运行结果：

```sh
主线程开始获取t1线程的运行结果
Thu Sep 01 21:52:28 CST 2022
结果为0.8617855786578636
Thu Sep 01 21:52:30 CST 2022
```







### 支持超时时间



```java
package mao.t2;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t2
 * Class(类名): GuardedObject
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 22:48
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class GuardedObject
{
    /**
     * 响应
     */
    private Object response;

    /**
     * 锁
     */
    private final Object lock = new Object();


    /**
     * 得到响应
     *
     * @param millis 毫秒值
     * @return {@link Object}
     */
    public Object getResponse(long millis)
    {
        synchronized (lock)
        {
            //记录最初时间
            long begin = System.currentTimeMillis();
            //已经经历的时间
            long timePassed = 0;
            while (response == null)
            {
                long waitTime = millis - timePassed;
                if (waitTime <= 0)
                {
                    break;
                }
                try
                {
                    lock.wait(waitTime);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                timePassed = System.currentTimeMillis() - begin;
            }
            return response;
        }

    }

    /**
     * 完成
     *
     * @param response 响应
     */
    public void complete(Object response)
    {
        synchronized (lock)
        {
            this.response = response;
            System.out.println("notify...");
            lock.notifyAll();
        }
    }

}
```



```java
package mao.t2;

import java.util.Date;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 22:53
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        GuardedObject guardedObject = new GuardedObject();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                //休眠2秒
                try
                {
                    Thread.sleep(2000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                //返回结果
                guardedObject.complete(Math.random());

            }
        }, "t1").start();

        //主线程等待
        System.out.println("主线程开始获取t1线程的运行结果");
        System.out.println(new Date());
        Object response = guardedObject.getResponse(1000);
        System.out.println("结果为" + response);
        System.out.println(new Date());
    }
}
```



运行结果：

```sh
主线程开始获取t1线程的运行结果
Fri Sep 02 22:54:41 CST 2022
结果为null
Fri Sep 02 22:54:42 CST 2022
notify...
```





线程的join方法采用此模式

```java
public class Thread implements Runnable 
{
    /**
     * Waits at most {@code millis} milliseconds for this thread to
     * die. A timeout of {@code 0} means to wait forever.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public final synchronized void join(final long millis)
    throws InterruptedException {
        if (millis > 0) {
            if (isAlive()) {
                final long startTime = System.nanoTime();
                long delay = millis;
                do {
                    wait(delay);
                } while (isAlive() && (delay = millis -
                        TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime)) > 0);
            }
        } else if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            throw new IllegalArgumentException("timeout value is negative");
        }
    }

    /**
     * Waits at most {@code millis} milliseconds plus
     * {@code nanos} nanoseconds for this thread to die.
     * If both arguments are {@code 0}, it means to wait forever.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @param  nanos
     *         {@code 0-999999} additional nanoseconds to wait
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative, or the value
     *          of {@code nanos} is not in the range {@code 0-999999}
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public final synchronized void join(long millis, int nanos)
    throws InterruptedException {

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0 && millis < Long.MAX_VALUE) {
            millis++;
        }

        join(millis);
    }

    /**
     * Waits for this thread to die.
     *
     * <p> An invocation of this method behaves in exactly the same
     * way as the invocation
     *
     * <blockquote>
     * {@linkplain #join(long) join}{@code (0)}
     * </blockquote>
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public final void join() throws InterruptedException {
        join(0);
    }
}
```







### 多任务

```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t3
 * Class(类名): GuardedObject
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 23:12
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class GuardedObject
{
    // 标识 Guarded Object
    private int id;

    /**
     * 响应
     */
    private Object response;

    /**
     * 保护对象
     *
     * @param id id
     */
    public GuardedObject(int id)
    {
        this.id = id;
    }

    /**
     * 得到id
     *
     * @return int
     */
    public int getId()
    {
        return id;
    }

    /**
     * 得到响应
     *
     * @param timeout 超时时间
     * @return {@link Object}
     */
    public Object getResponse(long timeout)
    {
        synchronized (this)
        {
            long begin = System.currentTimeMillis();
            long passedTime = 0;
            while (response == null)
            {
                long waitTime = timeout - passedTime;
                if (timeout - passedTime <= 0)
                {
                    break;
                }
                try
                {
                    this.wait(waitTime);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                passedTime = System.currentTimeMillis() - begin;
            }
            return response;
        }
    }

    /**
     * 完成
     *
     * @param response 响应
     */
    public void complete(Object response)
    {
        synchronized (this)
        {
            // 给结果成员变量赋值
            this.response = response;
            this.notifyAll();
        }
    }


}
```





```java
package mao.t3;

import java.util.Hashtable;
import java.util.Map;
import java.util.Set;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t3
 * Class(类名): Mailboxes
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 23:15
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Mailboxes
{
    /**
     * 盒子
     */
    private static final Map<Integer, GuardedObject> boxes = new Hashtable<>();

    private static int id = 1;

    /**
     * 生成id
     *
     * @return int
     */
    private static synchronized int generateId()
    {
        return id++;
    }

    /**
     * 获得保护对象
     *
     * @param id id
     * @return {@link GuardedObject}
     */
    public static GuardedObject getGuardedObject(int id)
    {
        return boxes.remove(id);
    }

    /**
     * 创建对象
     *
     * @return {@link GuardedObject}
     */
    public static GuardedObject createGuardedObject()
    {
        GuardedObject guardedObject = new GuardedObject(generateId());
        boxes.put(guardedObject.getId(), guardedObject);
        return guardedObject;
    }

    /**
     * 得到id
     *
     * @return {@link Set}<{@link Integer}>
     */
    public static Set<Integer> getIds()
    {
        return boxes.keySet();
    }
}
```





```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t3
 * Class(类名): Consumer
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 23:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Consumer extends Thread
{
    @Override
    public void run()
    {
        GuardedObject guardedObject = Mailboxes.createGuardedObject();
        System.out.println("开始收信 id:" + guardedObject.getId());
        Object mail = guardedObject.getResponse(5000);
        System.out.println("收到信 id:" + guardedObject.getId() + ", 内容:" + mail);
    }
}
```





```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t3
 * Class(类名): Producer
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 23:21
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Producer extends Thread
{
    /**
     * id
     */
    private final int id;

    /**
     * 邮件
     */
    private final String mail;

    /**
     * 生产商
     *
     * @param id   id
     * @param mail 邮件
     */
    public Producer(int id, String mail)
    {
        this.id = id;
        this.mail = mail;
    }

    /**
     * 运行
     */
    @Override
    public void run()
    {
        GuardedObject guardedObject = Mailboxes.getGuardedObject(id);
        System.out.println("送信 id:" + id + ", 内容:" + mail);
        guardedObject.complete(mail);
    }
}
```





```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_保护性暂停
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/2
 * Time(创建时间)： 23:23
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args) throws InterruptedException
    {
        for (int i = 0; i < 3; i++)
        {
            new Consumer().start();
        }

        Thread.sleep(2000);

        for (Integer id : Mailboxes.getIds())
        {
            new Producer(id, "内容" + id).start();
        }

    }
}
```



运行结果：

```sh
开始收信 id:2
开始收信 id:3
开始收信 id:1
送信 id:3, 内容:内容3
送信 id:1, 内容:内容1
收到信 id:3, 内容:内容3
送信 id:2, 内容:内容2
收到信 id:1, 内容:内容1
收到信 id:2, 内容:内容2
```







## Park和Unpark



### API

它们是 LockSupport 类中的方法



* LockSupport.park() ：暂停当前线程
* LockSupport.unpark(暂停线程对象) ：恢复某个线程的运行



```java
public class LockSupport {
    private LockSupport() {} // Cannot be instantiated.

    private static void setBlocker(Thread t, Object arg) {
        U.putReferenceOpaque(t, PARKBLOCKER, arg);
    }

    /**
     * Sets the object to be returned by invocations of {@link
     * #getBlocker getBlocker} for the current thread. This method may
     * be used before invoking the no-argument version of {@link
     * LockSupport#park() park()} from non-public objects, allowing
     * more helpful diagnostics, or retaining compatibility with
     * previous implementations of blocking methods.  Previous values
     * of the blocker are not automatically restored after blocking.
     * To obtain the effects of {@code park(b}}, use {@code
     * setCurrentBlocker(b); park(); setCurrentBlocker(null);}
     *
     * @param blocker the blocker object
     * @since 14
     */
    public static void setCurrentBlocker(Object blocker) {
        U.putReferenceOpaque(Thread.currentThread(), PARKBLOCKER, blocker);
    }

    /**
     * Makes available the permit for the given thread, if it
     * was not already available.  If the thread was blocked on
     * {@code park} then it will unblock.  Otherwise, its next call
     * to {@code park} is guaranteed not to block. This operation
     * is not guaranteed to have any effect at all if the given
     * thread has not been started.
     *
     * @param thread the thread to unpark, or {@code null}, in which case
     *        this operation has no effect
     */
    public static void unpark(Thread thread) {
        if (thread != null)
            U.unpark(thread);
    }

    /**
     * Disables the current thread for thread scheduling purposes unless the
     * permit is available.
     *
     * <p>If the permit is available then it is consumed and the call returns
     * immediately; otherwise
     * the current thread becomes disabled for thread scheduling
     * purposes and lies dormant until one of three things happens:
     *
     * <ul>
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread upon return.
     *
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @since 1.6
     */
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        U.park(false, 0L);
        setBlocker(t, null);
    }

    /**
     * Disables the current thread for thread scheduling purposes, for up to
     * the specified waiting time, unless the permit is available.
     *
     * <p>If the specified waiting time is zero or negative, the
     * method does nothing. Otherwise, if the permit is available then
     * it is consumed and the call returns immediately; otherwise the
     * current thread becomes disabled for thread scheduling purposes
     * and lies dormant until one of four things happens:
     *
     * <ul>
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The specified waiting time elapses; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread, or the elapsed time
     * upon return.
     *
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @param nanos the maximum number of nanoseconds to wait
     * @since 1.6
     */
    public static void parkNanos(Object blocker, long nanos) {
        if (nanos > 0) {
            Thread t = Thread.currentThread();
            setBlocker(t, blocker);
            U.park(false, nanos);
            setBlocker(t, null);
        }
    }

    /**
     * Disables the current thread for thread scheduling purposes, until
     * the specified deadline, unless the permit is available.
     *
     * <p>If the permit is available then it is consumed and the call
     * returns immediately; otherwise the current thread becomes disabled
     * for thread scheduling purposes and lies dormant until one of four
     * things happens:
     *
     * <ul>
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts} the
     * current thread; or
     *
     * <li>The specified deadline passes; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread, or the current time
     * upon return.
     *
     * @param blocker the synchronization object responsible for this
     *        thread parking
     * @param deadline the absolute time, in milliseconds from the Epoch,
     *        to wait until
     * @since 1.6
     */
    public static void parkUntil(Object blocker, long deadline) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        U.park(true, deadline);
        setBlocker(t, null);
    }

    /**
     * Returns the blocker object supplied to the most recent
     * invocation of a park method that has not yet unblocked, or null
     * if not blocked.  The value returned is just a momentary
     * snapshot -- the thread may have since unblocked or blocked on a
     * different blocker object.
     *
     * @param t the thread
     * @return the blocker
     * @throws NullPointerException if argument is null
     * @since 1.6
     */
    public static Object getBlocker(Thread t) {
        if (t == null)
            throw new NullPointerException();
        return U.getReferenceOpaque(t, PARKBLOCKER);
    }

    /**
     * Disables the current thread for thread scheduling purposes unless the
     * permit is available.
     *
     * <p>If the permit is available then it is consumed and the call
     * returns immediately; otherwise the current thread becomes disabled
     * for thread scheduling purposes and lies dormant until one of three
     * things happens:
     *
     * <ul>
     *
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread upon return.
     */
    public static void park() {
        U.park(false, 0L);
    }

    /**
     * Disables the current thread for thread scheduling purposes, for up to
     * the specified waiting time, unless the permit is available.
     *
     * <p>If the specified waiting time is zero or negative, the
     * method does nothing. Otherwise, if the permit is available then
     * it is consumed and the call returns immediately; otherwise the
     * current thread becomes disabled for thread scheduling purposes
     * and lies dormant until one of four things happens:
     *
     * <ul>
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The specified waiting time elapses; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread, or the elapsed time
     * upon return.
     *
     * @param nanos the maximum number of nanoseconds to wait
     */
    public static void parkNanos(long nanos) {
        if (nanos > 0)
            U.park(false, nanos);
    }

    /**
     * Disables the current thread for thread scheduling purposes, until
     * the specified deadline, unless the permit is available.
     *
     * <p>If the permit is available then it is consumed and the call
     * returns immediately; otherwise the current thread becomes disabled
     * for thread scheduling purposes and lies dormant until one of four
     * things happens:
     *
     * <ul>
     * <li>Some other thread invokes {@link #unpark unpark} with the
     * current thread as the target; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The specified deadline passes; or
     *
     * <li>The call spuriously (that is, for no reason) returns.
     * </ul>
     *
     * <p>This method does <em>not</em> report which of these caused the
     * method to return. Callers should re-check the conditions which caused
     * the thread to park in the first place. Callers may also determine,
     * for example, the interrupt status of the thread, or the current time
     * upon return.
     *
     * @param deadline the absolute time, in milliseconds from the Epoch,
     *        to wait until
     */
    public static void parkUntil(long deadline) {
        U.park(true, deadline);
    }

    /**
     * Returns the thread id for the given thread.  We must access
     * this directly rather than via method Thread.getId() because
     * getId() has been known to be overridden in ways that do not
     * preserve unique mappings.
     */
    static final long getThreadId(Thread thread) {
        return U.getLong(thread, TID);
    }

    // Hotspot implementation via intrinsics API
    private static final Unsafe U = Unsafe.getUnsafe();
    private static final long PARKBLOCKER
        = U.objectFieldOffset(Thread.class, "parkBlocker");
    private static final long TID
        = U.objectFieldOffset(Thread.class, "tid");

}
```





### 使用

先 park 再 unpark：

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.LockSupport;

/**
 * Project name(项目名称)：java并发编程_Park和Unpark
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/3
 * Time(创建时间)： 18:37
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("开始");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                log.debug("park");
                LockSupport.park();
                log.debug("释放");
            }
        }, "t1");

        thread.start();

        Thread.sleep(2000);
        log.debug("开始释放");
        LockSupport.unpark(thread);

    }
}
```



运行结果：

```sh
2022-09-03  18:52:05.080  [t1] DEBUG mao.t1.Test:  开始
2022-09-03  18:52:06.086  [t1] DEBUG mao.t1.Test:  park
2022-09-03  18:52:07.086  [main] DEBUG mao.t1.Test:  开始释放
2022-09-03  18:52:07.086  [t1] DEBUG mao.t1.Test:  释放
```



先 unpark 再 park：

```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.LockSupport;

/**
 * Project name(项目名称)：java并发编程_Park和Unpark
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/3
 * Time(创建时间)： 18:53
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws InterruptedException
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("开始");
                try
                {
                    Thread.sleep(2000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                log.debug("park");
                LockSupport.park();
                log.debug("释放");
            }
        }, "t1");

        thread.start();

        Thread.sleep(1000);
        log.debug("开始释放");
        LockSupport.unpark(thread);

    }
}
```



运行结果：

```sh
2022-09-03  18:54:26.733  [t1] DEBUG mao.t2.Test:  开始
2022-09-03  18:54:27.741  [main] DEBUG mao.t2.Test:  开始释放
2022-09-03  18:54:28.745  [t1] DEBUG mao.t2.Test:  park
2022-09-03  18:54:28.745  [t1] DEBUG mao.t2.Test:  释放
```





###  wait和notify

* wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必
* park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll  是唤醒所有等待线程，就不那么【精确】
* park & unpark 可以先 unpark，而 wait & notify 不能先 notify





### park unpark 原理

每个线程都有自己的一个 Parker 对象，由三部分组成 _counter ， _cond 和 _mutex

* 线程就像一个旅人，Parker 就像他随身携带的背包，条件变量就好比背包中的帐篷。_counter 就好比背包中 的备用干粮（0 为耗尽，1 为充足）
* 调用 park 就是要看需不需要停下来歇息
  * 如果备用干粮耗尽，那么钻进帐篷歇息
  * 如果备用干粮充足，那么不需停留，继续前进
* 调用 unpark，就好比令干粮充足
  * 如果这时线程还在帐篷，就唤醒让他继续前进
  * 如果这时线程还在运行，那么下次他调用 park 时，仅是消耗掉备用干粮，不需停留继续前进。因为背包空间有限，多次调用 unpark 仅会补充一份备用干粮





### 线程状态转换



![image-20220827211854901](img/java并发编程学习笔记/image-20220827211854901.png)





### 情况1 NEW --> RUNNABLE

* 当调用 t.start() 方法时，由 NEW --> RUNNABLE



### 情况2 RUNNABLE <--> WAITING

t 线程用 synchronized(obj) 获取了对象锁后

* 调用 obj.wait() 方法时，t 线程从 RUNNABLE --> WAITING
* 其它线程调用 obj.notify() ， obj.notifyAll() ， t.interrupt() 时
  * 竞争锁成功，t 线程从 WAITING --> RUNNABLE
  * 竞争锁失败，t 线程从 WAITING --> BLOCKED





### 情况 3 RUNNABLE <--> WAITING

* 当前线程调用 t.join() 方法时，当前线程从 RUNNABLE --> WAITING。注意是当前线程在t 线程对象的监视器上等待
* t 线程运行结束，或调用了当前线程的 interrupt() 时，当前线程从 WAITING --> RUNNABLE





### 情况4 RUNNABLE <--> WAITING

* 当前线程调用 LockSupport.park() 方法会让当前线程从 RUNNABLE --> WAITING
* 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，会让目标线程从 WAITING -->  RUNNABLE





### 情况5 RUNNABLE <--> TIMED_WAITING

t 线程用 synchronized(obj) 获取了对象锁后

* 调用 obj.wait(long n) 方法时，t 线程从 RUNNABLE --> TIMED_WAITING
* t 线程等待时间超过了 n 毫秒，或调用 obj.notify() ， obj.notifyAll() ， t.interrupt() 时
  * 竞争锁成功，t 线程从 TIMED_WAITING --> RUNNABLE
  * 竞争锁失败，t 线程从 TIMED_WAITING --> BLOCKED





### 情况6 RUNNABLE <--> TIMED_WAITING

* 当前线程调用 t.join(long n) 方法时，当前线程从 RUNNABLE --> TIMED_WAITING。注意是当前线程在t 线程对象的监视器上等待
* 当前线程等待时间超过了 n 毫秒，或t 线程运行结束，或调用了当前线程的 interrupt() 时，当前线程从 TIMED_WAITING --> RUNNABLE





### 情况7 RUNNABLE <--> TIMED_WAITING

* 当前线程调用 Thread.sleep(long n) ，当前线程从 RUNNABLE --> TIMED_WAITING
* 当前线程等待时间超过了 n 毫秒，当前线程从 TIMED_WAITING --> RUNNABLE





### 情况8 RUNNABLE <--> TIMED_WAITING

* 当前线程调用 LockSupport.parkNanos(long nanos) 或 LockSupport.parkUntil(long millis) 时，当前线 程从 RUNNABLE --> TIMED_WAITING
* 调用 LockSupport.unpark(目标线程) 或调用了线程 的 interrupt() ，或是等待超时，会让目标线程从 TIMED_WAITING--> RUNNABLE





### 情况 9 RUNNABLE <--> BLOCKED

* t 线程用 synchronized(obj) 获取了对象锁时如果竞争失败，从 RUNNABLE --> BLOCKED
* 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED 的线程重新竞争，如果其中 t 线程竞争 成功，从 BLOCKED --> RUNNABLE ，其它失败的线程仍然 BLOCKED





### 情况10 RUNNABLE <--> TERMINATED

当前线程所有代码运行完毕，进入 TERMINATED







## 多把锁

一间大屋子有两个功能：睡觉、学习，互不相干

现在小南要学习，小女要睡觉，但如果只用一间屋子（一个对象锁）的话，那么并发度很低 解决方法是准备多个房间（多个对象锁）



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_多把锁
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 10:58
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private static final Object lock = new Object();

    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("线程t1获取到锁");
                    log.debug("t1休眠3秒");
                    try
                    {
                        Thread.sleep(3000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t1运行");
                    log.debug("t1释放锁");
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock)
                {
                    log.debug("线程t2获取到锁");
                    log.debug("t2休眠2秒");
                    try
                    {
                        Thread.sleep(2000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t2运行");
                    log.debug("t2释放锁");
                }
            }
        }, "t2").start();
    }
}
```



运行结果：

```sh
2022-09-05  11:05:20.793  [t1] DEBUG mao.t1.Test:  线程t1获取到锁
2022-09-05  11:05:20.795  [t1] DEBUG mao.t1.Test:  t1休眠3秒
2022-09-05  11:05:23.796  [t1] DEBUG mao.t1.Test:  t1运行
2022-09-05  11:05:23.796  [t1] DEBUG mao.t1.Test:  t1释放锁
2022-09-05  11:05:23.796  [t2] DEBUG mao.t1.Test:  线程t2获取到锁
2022-09-05  11:05:23.796  [t2] DEBUG mao.t1.Test:  t2休眠2秒
2022-09-05  11:05:25.809  [t2] DEBUG mao.t1.Test:  t2运行
2022-09-05  11:05:25.809  [t2] DEBUG mao.t1.Test:  t2释放锁
```





改进：

```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_多把锁
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 11:06
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    /**
     * lock1
     */
    private static final Object lock1 = new Object();

    /**
     * lock2
     */
    private static final Object lock2 = new Object();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock1)
                {
                    log.debug("线程t1获取到锁1");
                    log.debug("t1休眠3秒");
                    try
                    {
                        Thread.sleep(3000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t1运行");
                    log.debug("t1释放锁1");
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock2)
                {
                    log.debug("线程t2获取到锁2");
                    log.debug("t2休眠2秒");
                    try
                    {
                        Thread.sleep(2000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug("t2运行");
                    log.debug("t2释放锁2");
                }
            }
        }, "t2").start();
    }
}
```



运行结果：

```sh
2022-09-05  11:07:45.315  [t1] DEBUG mao.t2.Test:  线程t1获取到锁1
2022-09-05  11:07:45.315  [t2] DEBUG mao.t2.Test:  线程t2获取到锁2
2022-09-05  11:07:45.318  [t1] DEBUG mao.t2.Test:  t1休眠3秒
2022-09-05  11:07:45.318  [t2] DEBUG mao.t2.Test:  t2休眠2秒
2022-09-05  11:07:47.320  [t2] DEBUG mao.t2.Test:  t2运行
2022-09-05  11:07:47.320  [t2] DEBUG mao.t2.Test:  t2释放锁2
2022-09-05  11:07:48.333  [t1] DEBUG mao.t2.Test:  t1运行
2022-09-05  11:07:48.333  [t1] DEBUG mao.t2.Test:  t1释放锁1
```



将锁的粒度细分

* 好处，是可以增强并发度
* 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁







## 活跃性

### 死锁

有这样的情况：一个线程需要同时获取多把锁，这时就容易发生死锁



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_活跃性
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 11:22
 * Version(版本): 1.0
 * Description(描述)： 死锁
 */

public class Test
{
    /**
     * lock1
     */
    private static final Object lock1 = new Object();

    /**
     * lock2
     */
    private static final Object lock2 = new Object();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock1)
                {
                    log.debug("t1获得锁1");
                    log.debug("t1尝试获取锁2");
                    synchronized (lock2)
                    {
                        log.debug("t1获得锁2");
                        log.debug("todo...");
                    }
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (lock2)
                {
                    log.debug("t2获得锁2");
                    log.debug("t2尝试获取锁1");
                    synchronized (lock1)
                    {
                        log.debug("t2获得锁1");
                        log.debug("todo...");
                    }
                }
            }
        }, "t2").start();


        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.info("程序退出");
            }
        },"ShutdownHook"));
    }

}

```



运行结果：

```sh
2022-09-05  11:34:05.530  [t2] DEBUG mao.t1.Test:  t2获得锁2
2022-09-05  11:34:05.530  [t1] DEBUG mao.t1.Test:  t1获得锁1
2022-09-05  11:34:05.532  [t2] DEBUG mao.t1.Test:  t2尝试获取锁1
2022-09-05  11:34:05.533  [t1] DEBUG mao.t1.Test:  t1尝试获取锁2
2022-09-05  11:34:17.715  [ShutdownHook] INFO  mao.t1.Test:  程序退出
```



已经产生死锁





### 定位死锁

检测死锁可以使用 jconsole工具，或者使用 jps 定位进程 id，再用 jstack 定位死锁



**jps+jstack**



```sh
PS C:\Users\mao\Desktop> jps
10712 Test
17512 Launcher
11644 RemoteMavenServer36
12812 RemoteMavenServer36
2844
2892 Jps
PS C:\Users\mao\Desktop>
```



```sh
PS C:\Users\mao\Desktop> jstack 10712
2022-09-05 11:35:59
Full thread dump OpenJDK 64-Bit Server VM (16.0.2+7-67 mixed mode, sharing):

Threads class SMR info:
_java_thread_list=0x00000207e648f3d0, length=15, elements={
0x00000207e5115710, 0x00000207e51165d0, 0x00000207e512c7c0, 0x00000207e512f5f0,
0x00000207e512fff0, 0x00000207e51309f0, 0x00000207e5136c30, 0x00000207e5149070,
0x00000207e5c08070, 0x00000207e5cede20, 0x00000207e5e48520, 0x00000207e5e4d110,
0x00000207e664bf80, 0x00000207e664c920, 0x00000207c19cf9d0
}

"Reference Handler" #2 daemon prio=10 os_prio=2 cpu=0.00ms elapsed=91.73s tid=0x00000207e5115710 nid=0x458 waiting on condition  [0x00000071f3bfe000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.ref.Reference.waitForReferencePendingList(java.base@16.0.2/Native Method)
        at java.lang.ref.Reference.processPendingReferences(java.base@16.0.2/Reference.java:243)
        at java.lang.ref.Reference$ReferenceHandler.run(java.base@16.0.2/Reference.java:215)

"Finalizer" #3 daemon prio=8 os_prio=1 cpu=0.00ms elapsed=91.73s tid=0x00000207e51165d0 nid=0x2e70 in Object.wait()  [0x00000071f3cfe000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(java.base@16.0.2/Native Method)
        - waiting on <0x000000070f602c18> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@16.0.2/ReferenceQueue.java:155)
        - locked <0x000000070f602c18> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@16.0.2/ReferenceQueue.java:176)
        at java.lang.ref.Finalizer$FinalizerThread.run(java.base@16.0.2/Finalizer.java:171)

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 cpu=0.00ms elapsed=91.72s tid=0x00000207e512c7c0 nid=0xbd8 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 cpu=15.62ms elapsed=91.72s tid=0x00000207e512f5f0 nid=0x37a0 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #6 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=91.72s tid=0x00000207e512fff0 nid=0x98c runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Deflation Thread" #7 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=91.72s tid=0x00000207e51309f0 nid=0x930 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #8 daemon prio=9 os_prio=2 cpu=187.50ms elapsed=91.72s tid=0x00000207e5136c30 nid=0x2344 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread0" #16 daemon prio=9 os_prio=2 cpu=78.12ms elapsed=91.72s tid=0x00000207e5149070 nid=0x4058 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #20 daemon prio=9 os_prio=2 cpu=0.00ms elapsed=91.72s tid=0x00000207e5c08070 nid=0x2bd4 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #21 daemon prio=8 os_prio=1 cpu=0.00ms elapsed=91.70s tid=0x00000207e5cede20 nid=0x7c0 in Object.wait()  [0x00000071f44fe000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
        at java.lang.Object.wait(java.base@16.0.2/Native Method)
        - waiting on <0x000000070f6041c0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@16.0.2/ReferenceQueue.java:155)
        - locked <0x000000070f6041c0> (a java.lang.ref.ReferenceQueue$Lock)
        at jdk.internal.ref.CleanerImpl.run(java.base@16.0.2/CleanerImpl.java:140)
        at java.lang.Thread.run(java.base@16.0.2/Thread.java:831)
        at jdk.internal.misc.InnocuousThread.run(java.base@16.0.2/InnocuousThread.java:134)

"Monitor Ctrl-Break" #22 daemon prio=5 os_prio=0 cpu=15.62ms elapsed=91.66s tid=0x00000207e5e48520 nid=0x213c runnable  [0x00000071f46fe000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.SocketDispatcher.read0(java.base@16.0.2/Native Method)
        at sun.nio.ch.SocketDispatcher.read(java.base@16.0.2/SocketDispatcher.java:46)
        at sun.nio.ch.NioSocketImpl.tryRead(java.base@16.0.2/NioSocketImpl.java:261)
        at sun.nio.ch.NioSocketImpl.implRead(java.base@16.0.2/NioSocketImpl.java:312)
        at sun.nio.ch.NioSocketImpl.read(java.base@16.0.2/NioSocketImpl.java:350)
        at sun.nio.ch.NioSocketImpl$1.read(java.base@16.0.2/NioSocketImpl.java:803)
        at java.net.Socket$SocketInputStream.read(java.base@16.0.2/Socket.java:976)
        at sun.nio.cs.StreamDecoder.readBytes(java.base@16.0.2/StreamDecoder.java:297)
        at sun.nio.cs.StreamDecoder.implRead(java.base@16.0.2/StreamDecoder.java:339)
        at sun.nio.cs.StreamDecoder.read(java.base@16.0.2/StreamDecoder.java:188)
        - locked <0x000000070f601ec8> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(java.base@16.0.2/InputStreamReader.java:178)
        at java.io.BufferedReader.fill(java.base@16.0.2/BufferedReader.java:161)
        at java.io.BufferedReader.readLine(java.base@16.0.2/BufferedReader.java:329)
        - locked <0x000000070f601ec8> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(java.base@16.0.2/BufferedReader.java:396)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)

"Notification Thread" #23 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=91.66s tid=0x00000207e5e4d110 nid=0x3348 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"t1" #27 prio=5 os_prio=0 cpu=0.00ms elapsed=91.17s tid=0x00000207e664bf80 nid=0x2a54 waiting for monitor entry  [0x00000071f51ff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t1.Test$1.run(Test.java:50)
        - waiting to lock <0x000000070f711788> (a java.lang.Object)
        - locked <0x000000070f711798> (a java.lang.Object)
        at java.lang.Thread.run(java.base@16.0.2/Thread.java:831)

"t2" #28 prio=5 os_prio=0 cpu=0.00ms elapsed=91.17s tid=0x00000207e664c920 nid=0xad4 waiting for monitor entry  [0x00000071f52fe000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t1.Test$2.run(Test.java:68)
        - waiting to lock <0x000000070f711798> (a java.lang.Object)
        - locked <0x000000070f711788> (a java.lang.Object)
        at java.lang.Thread.run(java.base@16.0.2/Thread.java:831)

"DestroyJavaVM" #30 prio=5 os_prio=0 cpu=546.88ms elapsed=91.17s tid=0x00000207c19cf9d0 nid=0x2084 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=2 cpu=0.00ms elapsed=91.74s tid=0x00000207e51106c0 nid=0x2924 runnable

"GC Thread#0" os_prio=2 cpu=15.62ms elapsed=91.74s tid=0x00000207c1a228a0 nid=0x37ec runnable

"GC Thread#1" os_prio=2 cpu=15.62ms elapsed=91.43s tid=0x00000207e6416f10 nid=0xde8 runnable

"GC Thread#2" os_prio=2 cpu=15.62ms elapsed=91.43s tid=0x00000207e60f2000 nid=0x226c runnable

"GC Thread#3" os_prio=2 cpu=15.62ms elapsed=91.43s tid=0x00000207e60f2310 nid=0x3ebc runnable

"GC Thread#4" os_prio=2 cpu=15.62ms elapsed=91.43s tid=0x00000207e60f2620 nid=0x19c8 runnable

"GC Thread#5" os_prio=2 cpu=15.62ms elapsed=91.43s tid=0x00000207e60f31c0 nid=0x2808 runnable

"G1 Main Marker" os_prio=2 cpu=0.00ms elapsed=91.74s tid=0x00000207c1a337f0 nid=0x3994 runnable

"G1 Conc#0" os_prio=2 cpu=0.00ms elapsed=91.74s tid=0x00000207c1a34a00 nid=0x2f44 runnable

"G1 Refine#0" os_prio=2 cpu=0.00ms elapsed=91.74s tid=0x00000207e4fd44e0 nid=0x2a94 runnable

"G1 Service" os_prio=2 cpu=0.00ms elapsed=91.74s tid=0x00000207c1a9f080 nid=0x3e30 runnable

"VM Periodic Task Thread" os_prio=2 cpu=0.00ms elapsed=91.66s tid=0x00000207e5e4e2c0 nid=0x1268 waiting on condition

JNI global refs: 23, weak refs: 0


Found one Java-level deadlock:
=============================
"t1":
  waiting to lock monitor 0x00000207e66a1340 (object 0x000000070f711788, a java.lang.Object),
  which is held by "t2"

"t2":
  waiting to lock monitor 0x00000207e66a2bc0 (object 0x000000070f711798, a java.lang.Object),
  which is held by "t1"

Java stack information for the threads listed above:
===================================================
"t1":
        at mao.t1.Test$1.run(Test.java:50)
        - waiting to lock <0x000000070f711788> (a java.lang.Object)
        - locked <0x000000070f711798> (a java.lang.Object)
        at java.lang.Thread.run(java.base@16.0.2/Thread.java:831)
"t2":
        at mao.t1.Test$2.run(Test.java:68)
        - waiting to lock <0x000000070f711798> (a java.lang.Object)
        - locked <0x000000070f711788> (a java.lang.Object)
        at java.lang.Thread.run(java.base@16.0.2/Thread.java:831)

Found 1 deadlock.

PS C:\Users\mao\Desktop>
```





**jconsole**



![image-20220905113904130](img/java并发编程学习笔记/image-20220905113904130.png)



![image-20220905113922440](img/java并发编程学习笔记/image-20220905113922440.png)



点击线程选项卡



![image-20220905113949824](img/java并发编程学习笔记/image-20220905113949824.png)



点击检测死锁



![image-20220905114026348](img/java并发编程学习笔记/image-20220905114026348.png)



![image-20220905114032967](img/java并发编程学习笔记/image-20220905114032967.png)







### 哲学家就餐问题

有五位哲学家，围坐在圆桌旁

* 他们只做两件事，思考和吃饭，思考一会吃口饭，吃完饭后接着思考
* 吃饭时要用两根筷子吃，桌上共有 5 根筷子，每位哲学家左右手边各有一根筷子
* 如果筷子被身边的人拿着，自己就得等待





```java
package mao.t2;

/**
 * Project name(项目名称)：java并发编程_活跃性
 * Package(包名): mao.t2
 * Class(类名): Chopstick
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 11:42
 * Version(版本): 1.0
 * Description(描述)： 筷子类
 */

public class Chopstick
{
    /**
     * 筷子名字
     */
    String name;

    /**
     * 筷子
     *
     * @param name 名字
     */
    public Chopstick(String name)
    {
        this.name = name;
    }

    /**
     * 字符串
     *
     * @return {@link String}
     */
    @Override
    public String toString()
    {
        return "筷子" + name;
    }
}
```



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_活跃性
 * Package(包名): mao.t2
 * Class(类名): Philosopher
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 11:44
 * Version(版本): 1.0
 * Description(描述)：哲学家
 */

public class Philosopher extends Thread
{
    private static final Logger log = LoggerFactory.getLogger(Philosopher.class);

    /**
     * 左边筷子
     */
    final Chopstick leftChopstick;

    /**
     * 右边筷子
     */
    final Chopstick rightChopstick;

    /**
     * 哲学家
     *
     * @param name           线程名字，也就是哲学家名字
     * @param leftChopstick  左边筷子
     * @param rightChopstick 右边筷子
     */
    public Philosopher(String name, Chopstick leftChopstick, Chopstick rightChopstick)
    {
        super(name);
        this.leftChopstick = leftChopstick;
        this.rightChopstick = rightChopstick;
    }

    /**
     * 吃饭
     */
    private void eat()
    {
        log.debug("eating...");
        try
        {
            Thread.sleep(1000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    @Override
    public void run()
    {
        while (true)
        {
            // 获得左手筷子
            synchronized (leftChopstick)
            {
                // 获得右手筷子
                synchronized (rightChopstick)
                {
                    // 吃饭
                    eat();
                }
                // 放下右手筷子
            }
            // 放下左手筷子
        }
    }
}
```





```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_活跃性
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 11:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        Chopstick c1 = new Chopstick("1");
        Chopstick c2 = new Chopstick("2");
        Chopstick c3 = new Chopstick("3");
        Chopstick c4 = new Chopstick("4");
        Chopstick c5 = new Chopstick("5");
        new Philosopher("苏格拉底", c1, c2).start();
        new Philosopher("柏拉图", c2, c3).start();
        new Philosopher("亚里士多德", c3, c4).start();
        new Philosopher("赫拉克利特", c4, c5).start();
        new Philosopher("阿基米德", c5, c1).start();

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.info("程序退出");
            }
        }, "ShutdownHook"));
    }
}
```



运行结果：

```sh
2022-09-05  11:55:37.333  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:37.333  [苏格拉底] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:38.337  [苏格拉底] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:38.337  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:39.339  [苏格拉底] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:39.339  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:40.341  [苏格拉底] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:40.341  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:41.349  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:42.350  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:43.359  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:44.372  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:45.374  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:46.384  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:47.399  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:48.400  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:49.413  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
2022-09-05  11:55:50.427  [亚里士多德] DEBUG mao.t2.Philosopher:  eating...
```



没有向下运行了



检测死锁



```sh
PS C:\Users\mao\Desktop> jps
14704 Launcher
4116 Jps
12812 RemoteMavenServer36
2844
3132 Test
PS C:\Users\mao\Desktop> jstack.exe 3132
2022-09-05 11:57:54
Full thread dump OpenJDK 64-Bit Server VM (16.0.2+7-67 mixed mode, sharing):

Threads class SMR info:
_java_thread_list=0x0000023eb9e5aca0, length=18, elements={
0x0000023eb88f4c40, 0x0000023eb88f5b00, 0x0000023eb890cc50, 0x0000023eb8911980,
0x0000023eb8912290, 0x0000023eb8912ba0, 0x0000023eb8918e90, 0x0000023eb8919980,
0x0000023eb9410060, 0x0000023eb88d5090, 0x0000023eb965a8e0, 0x0000023eb965ae00,
0x0000023eb9ddab50, 0x0000023eb9ddb070, 0x0000023eb9ded010, 0x0000023eb9ded530,
0x0000023eb9df1a60, 0x0000023e931ac200
}

"Reference Handler" #2 daemon prio=10 os_prio=2 cpu=0.00ms elapsed=138.10s tid=0x0000023eb88f4c40 nid=0x3514 waiting on condition  [0x000000d9f41ff000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.ref.Reference.waitForReferencePendingList(java.base@16.0.2/Native Method)
        at java.lang.ref.Reference.processPendingReferences(java.base@16.0.2/Reference.java:243)
        at java.lang.ref.Reference$ReferenceHandler.run(java.base@16.0.2/Reference.java:215)

"Finalizer" #3 daemon prio=8 os_prio=1 cpu=0.00ms elapsed=138.10s tid=0x0000023eb88f5b00 nid=0xc10 in Object.wait()  [0x000000d9f42ff000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(java.base@16.0.2/Native Method)
        - waiting on <0x000000070f602c18> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@16.0.2/ReferenceQueue.java:155)
        - locked <0x000000070f602c18> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@16.0.2/ReferenceQueue.java:176)
        at java.lang.ref.Finalizer$FinalizerThread.run(java.base@16.0.2/Finalizer.java:171)

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 cpu=0.00ms elapsed=138.09s tid=0x0000023eb890cc50 nid=0x4290 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 cpu=15.62ms elapsed=138.09s tid=0x0000023eb8911980 nid=0x4398 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #6 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=138.09s tid=0x0000023eb8912290 nid=0x2aec runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Deflation Thread" #7 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=138.09s tid=0x0000023eb8912ba0 nid=0x37d0 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #8 daemon prio=9 os_prio=2 cpu=296.88ms elapsed=138.09s tid=0x0000023eb8918e90 nid=0x1350 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"C1 CompilerThread0" #16 daemon prio=9 os_prio=2 cpu=109.38ms elapsed=138.09s tid=0x0000023eb8919980 nid=0x3f68 waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
   No compile task

"Sweeper thread" #20 daemon prio=9 os_prio=2 cpu=0.00ms elapsed=138.09s tid=0x0000023eb9410060 nid=0x2758 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Common-Cleaner" #21 daemon prio=8 os_prio=1 cpu=0.00ms elapsed=138.07s tid=0x0000023eb88d5090 nid=0x2a30 in Object.wait()  [0x000000d9f4afe000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
        at java.lang.Object.wait(java.base@16.0.2/Native Method)
        - waiting on <0x000000070f6041c0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@16.0.2/ReferenceQueue.java:155)
        - locked <0x000000070f6041c0> (a java.lang.ref.ReferenceQueue$Lock)
        at jdk.internal.ref.CleanerImpl.run(java.base@16.0.2/CleanerImpl.java:140)
        at java.lang.Thread.run(java.base@16.0.2/Thread.java:831)
        at jdk.internal.misc.InnocuousThread.run(java.base@16.0.2/InnocuousThread.java:134)

"Monitor Ctrl-Break" #22 daemon prio=5 os_prio=0 cpu=15.62ms elapsed=138.03s tid=0x0000023eb965a8e0 nid=0x3c3c runnable  [0x000000d9f4efe000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.SocketDispatcher.read0(java.base@16.0.2/Native Method)
        at sun.nio.ch.SocketDispatcher.read(java.base@16.0.2/SocketDispatcher.java:46)
        at sun.nio.ch.NioSocketImpl.tryRead(java.base@16.0.2/NioSocketImpl.java:261)
        at sun.nio.ch.NioSocketImpl.implRead(java.base@16.0.2/NioSocketImpl.java:312)
        at sun.nio.ch.NioSocketImpl.read(java.base@16.0.2/NioSocketImpl.java:350)
        at sun.nio.ch.NioSocketImpl$1.read(java.base@16.0.2/NioSocketImpl.java:803)
        at java.net.Socket$SocketInputStream.read(java.base@16.0.2/Socket.java:976)
        at sun.nio.cs.StreamDecoder.readBytes(java.base@16.0.2/StreamDecoder.java:297)
        at sun.nio.cs.StreamDecoder.implRead(java.base@16.0.2/StreamDecoder.java:339)
        at sun.nio.cs.StreamDecoder.read(java.base@16.0.2/StreamDecoder.java:188)
        - locked <0x000000070f601d08> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(java.base@16.0.2/InputStreamReader.java:178)
        at java.io.BufferedReader.fill(java.base@16.0.2/BufferedReader.java:161)
        at java.io.BufferedReader.readLine(java.base@16.0.2/BufferedReader.java:329)
        - locked <0x000000070f601d08> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(java.base@16.0.2/BufferedReader.java:396)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:49)

"Notification Thread" #23 daemon prio=9 os_prio=0 cpu=0.00ms elapsed=138.03s tid=0x0000023eb965ae00 nid=0x2740 runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"苏格拉底" #27 prio=5 os_prio=0 cpu=15.62ms elapsed=137.56s tid=0x0000023eb9ddab50 nid=0x1024 waiting for monitor entry  [0x000000d9f57ff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a28> (a mao.t2.Chopstick)
        - locked <0x0000000710249a18> (a mao.t2.Chopstick)

"柏拉图" #28 prio=5 os_prio=0 cpu=0.00ms elapsed=137.56s tid=0x0000023eb9ddb070 nid=0x47c8 waiting for monitor entry  [0x000000d9f58ff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a38> (a mao.t2.Chopstick)
        - locked <0x0000000710249a28> (a mao.t2.Chopstick)

"亚里士多德" #29 prio=5 os_prio=0 cpu=0.00ms elapsed=137.56s tid=0x0000023eb9ded010 nid=0x310c waiting for monitor entry  [0x000000d9f59ff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a48> (a mao.t2.Chopstick)
        - locked <0x0000000710249a38> (a mao.t2.Chopstick)

"赫拉克利特" #30 prio=5 os_prio=0 cpu=0.00ms elapsed=137.56s tid=0x0000023eb9ded530 nid=0x1078 waiting for monitor entry  [0x000000d9f5aff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a58> (a mao.t2.Chopstick)
        - locked <0x0000000710249a48> (a mao.t2.Chopstick)

"阿基米德" #31 prio=5 os_prio=0 cpu=0.00ms elapsed=137.56s tid=0x0000023eb9df1a60 nid=0xfdc waiting for monitor entry  [0x000000d9f5bfe000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a18> (a mao.t2.Chopstick)
        - locked <0x0000000710249a58> (a mao.t2.Chopstick)

"DestroyJavaVM" #33 prio=5 os_prio=0 cpu=515.62ms elapsed=137.56s tid=0x0000023e931ac200 nid=0x50c waiting on condition  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"VM Thread" os_prio=2 cpu=0.00ms elapsed=138.10s tid=0x0000023eb88efff0 nid=0x294c runnable

"GC Thread#0" os_prio=2 cpu=0.00ms elapsed=138.11s tid=0x0000023e931ff040 nid=0xfe4 runnable

"GC Thread#1" os_prio=2 cpu=0.00ms elapsed=137.80s tid=0x0000023eb9c958e0 nid=0xc4 runnable

"GC Thread#2" os_prio=2 cpu=0.00ms elapsed=137.80s tid=0x0000023eb9c95bf0 nid=0x30e8 runnable

"GC Thread#3" os_prio=2 cpu=0.00ms elapsed=137.80s tid=0x0000023eb9c95f00 nid=0x2500 runnable

"GC Thread#4" os_prio=2 cpu=0.00ms elapsed=137.80s tid=0x0000023eb9c96620 nid=0x46d0 runnable

"GC Thread#5" os_prio=2 cpu=0.00ms elapsed=137.80s tid=0x0000023eb9c975c0 nid=0x470 runnable

"G1 Main Marker" os_prio=2 cpu=0.00ms elapsed=138.11s tid=0x0000023e9320ff90 nid=0x22e4 runnable

"G1 Conc#0" os_prio=2 cpu=0.00ms elapsed=138.11s tid=0x0000023e932111a0 nid=0x24d8 runnable

"G1 Refine#0" os_prio=2 cpu=0.00ms elapsed=138.10s tid=0x0000023e9327c5d0 nid=0x4d8 runnable

"G1 Service" os_prio=2 cpu=0.00ms elapsed=138.10s tid=0x0000023e9327d050 nid=0x3760 runnable

"VM Periodic Task Thread" os_prio=2 cpu=0.00ms elapsed=138.03s tid=0x0000023eb965b7a0 nid=0x3190 waiting on condition

JNI global refs: 23, weak refs: 0


Found one Java-level deadlock:
=============================
"苏格拉底":
  waiting to lock monitor 0x0000023eb9dc11b0 (object 0x0000000710249a28, a mao.t2.Chopstick),
  which is held by "柏拉图"

"柏拉图":
  waiting to lock monitor 0x0000023eb9dc10d0 (object 0x0000000710249a38, a mao.t2.Chopstick),
  which is held by "亚里士多德"

"亚里士多德":
  waiting to lock monitor 0x0000023eb9dc2a30 (object 0x0000000710249a48, a mao.t2.Chopstick),
  which is held by "赫拉克利特"

"赫拉克利特":
  waiting to lock monitor 0x0000023eb9dc1a70 (object 0x0000000710249a58, a mao.t2.Chopstick),
  which is held by "阿基米德"

"阿基米德":
  waiting to lock monitor 0x0000023eb9dc16f0 (object 0x0000000710249a18, a mao.t2.Chopstick),
  which is held by "苏格拉底"

Java stack information for the threads listed above:
===================================================
"苏格拉底":
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a28> (a mao.t2.Chopstick)
        - locked <0x0000000710249a18> (a mao.t2.Chopstick)
"柏拉图":
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a38> (a mao.t2.Chopstick)
        - locked <0x0000000710249a28> (a mao.t2.Chopstick)
"亚里士多德":
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a48> (a mao.t2.Chopstick)
        - locked <0x0000000710249a38> (a mao.t2.Chopstick)
"赫拉克利特":
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a58> (a mao.t2.Chopstick)
        - locked <0x0000000710249a48> (a mao.t2.Chopstick)
"阿基米德":
        at mao.t2.Philosopher.run(Philosopher.java:75)
        - waiting to lock <0x0000000710249a18> (a mao.t2.Chopstick)
        - locked <0x0000000710249a58> (a mao.t2.Chopstick)

Found 1 deadlock.

PS C:\Users\mao\Desktop>
```



全部都产生了死锁





### 活锁

活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_活跃性
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 12:00
 * Version(版本): 1.0
 * Description(描述)： 活锁
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * volatile保证可见性，保证获取的是最新的数据，但不能保证原子性
     */
    static volatile int count = 100;

    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (count > 0)
                {
                    try
                    {
                        Thread.sleep(30);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    count--;
                    log.debug("t1 count:" + count);
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (count < 200)
                {
                    try
                    {
                        Thread.sleep(30);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    count++;
                    log.debug("t2 count:" + count);
                }
            }
        }, "t2").start();
    }
}

```



运行结果：

```sh
2022-09-05  12:35:08.357  [t2] DEBUG mao.t3.Test:  t2 count:100
2022-09-05  12:35:08.357  [t1] DEBUG mao.t3.Test:  t1 count:100
2022-09-05  12:35:08.401  [t1] DEBUG mao.t3.Test:  t1 count:99
2022-09-05  12:35:08.401  [t2] DEBUG mao.t3.Test:  t2 count:100
2022-09-05  12:35:08.432  [t1] DEBUG mao.t3.Test:  t1 count:99
2022-09-05  12:35:08.432  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:08.463  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.463  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.495  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.495  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.527  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:08.527  [t1] DEBUG mao.t3.Test:  t1 count:100
2022-09-05  12:35:08.558  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.558  [t1] DEBUG mao.t3.Test:  t1 count:102
2022-09-05  12:35:08.589  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.589  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.621  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.621  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:08.653  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.653  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.698  [t1] DEBUG mao.t3.Test:  t1 count:102
2022-09-05  12:35:08.698  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.729  [t1] DEBUG mao.t3.Test:  t1 count:102
2022-09-05  12:35:08.729  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.761  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.761  [t1] DEBUG mao.t3.Test:  t1 count:102
2022-09-05  12:35:08.793  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:08.793  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.825  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:08.825  [t1] DEBUG mao.t3.Test:  t1 count:100
2022-09-05  12:35:08.856  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.857  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.889  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:08.889  [t1] DEBUG mao.t3.Test:  t1 count:100
2022-09-05  12:35:08.921  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.921  [t1] DEBUG mao.t3.Test:  t1 count:100
2022-09-05  12:35:08.952  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.952  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:08.983  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:08.983  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:09.014  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:09.014  [t2] DEBUG mao.t3.Test:  t2 count:102
2022-09-05  12:35:09.046  [t2] DEBUG mao.t3.Test:  t2 count:103
2022-09-05  12:35:09.046  [t1] DEBUG mao.t3.Test:  t1 count:102
2022-09-05  12:35:09.078  [t1] DEBUG mao.t3.Test:  t1 count:101
2022-09-05  12:35:09.078  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:09.109  [t2] DEBUG mao.t3.Test:  t2 count:101
2022-09-05  12:35:09.109  [t1] DEBUG mao.t3.Test:  t1 count:100
2022-09-05  12:35:09.140  [t2] DEBUG mao.t3.Test:  t2 count:101
...
```





### 饥饿

一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束



![image-20220905124020764](img/java并发编程学习笔记/image-20220905124020764.png)



顺序加锁



![image-20220905124057947](img/java并发编程学习笔记/image-20220905124057947.png)









## ReentrantLock

相对于 synchronized 它具备如下特点

* 可中断
* 可以设置超时时间
* 可以设置为公平锁
* 支持多个条件变量



与 synchronized 一样，都支持可重入





### 语法

```java
		//获取锁
        log.debug("尝试获取锁");
        reentrantLock.lock();
        try
        {
            log.debug("获得到锁");
            //临界区
        }
        finally
        {
            //释放锁
            reentrantLock.unlock();
            log.debug("释放锁");
        }
```







### 可重入

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁 

如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:05
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 可重入锁
     */
    private static final ReentrantLock REENTRANT_LOCK = new ReentrantLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * method1
     */
    public static void method1()
    {
        REENTRANT_LOCK.lock();
        try
        {
            log.debug("execute method1");
            method2();
        }
        finally
        {
            REENTRANT_LOCK.unlock();
        }
    }

    /**
     * method2
     */
    public static void method2()
    {
        REENTRANT_LOCK.lock();
        try
        {
            log.debug("execute method2");
            method3();
        }
        finally
        {
            REENTRANT_LOCK.unlock();
        }
    }

    /**
     * method3
     */
    public static void method3()
    {
        REENTRANT_LOCK.lock();
        try
        {
            log.debug("execute method3");
        }
        finally
        {
            REENTRANT_LOCK.unlock();
        }

    }

    public static void main(String[] args)
    {
        method1();
    }
}
```



运行结果：

```sh
2022-09-05  16:09:45.185  [main] DEBUG mao.t2.Test:  execute method1
2022-09-05  16:09:45.187  [main] DEBUG mao.t2.Test:  execute method2
2022-09-05  16:09:45.187  [main] DEBUG mao.t2.Test:  execute method3
```





### 可打断



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:11
 * Version(版本): 1.0
 * Description(描述)： 可打断
 */

public class Test
{
    /**
     * 可重入锁
     */
    private static final ReentrantLock REENTRANT_LOCK = new ReentrantLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("启动...");
                try
                {
                    REENTRANT_LOCK.lockInterruptibly();
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                    log.debug("等锁的过程中被打断");
                    return;
                }
                try
                {
                    log.debug("获得了锁");
                }
                finally
                {
                    log.debug("释放锁");
                    REENTRANT_LOCK.unlock();
                }
            }
        }, "t1");

        //主线程获取锁
        REENTRANT_LOCK.lock();
        log.debug("main线程获得了锁");
        thread.start();
        try
        {
            Thread.sleep(1000);
            //打断
            log.debug("执行打断");
            thread.interrupt();
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            REENTRANT_LOCK.unlock();
        }
    }
}
```



运行结果：

```sh
2022-09-05  16:18:42.110  [main] DEBUG mao.t3.Test:  main线程获得了锁
2022-09-05  16:18:42.112  [t1] DEBUG mao.t3.Test:  启动...
2022-09-05  16:18:43.113  [main] DEBUG mao.t3.Test:  执行打断
2022-09-05  16:18:43.114  [t1] DEBUG mao.t3.Test:  等锁的过程中被打断
java.lang.InterruptedException
	at java.base/java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:958)
	at java.base/java.util.concurrent.locks.ReentrantLock$Sync.lockInterruptibly(ReentrantLock.java:161)
	at java.base/java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:372)
	at mao.t3.Test$1.run(Test.java:44)
	at java.base/java.lang.Thread.run(Thread.java:831)
```





如果是不可中断模式，那么即使使用了 interrupt 也不会让等待中断



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:19
 * Version(版本): 1.0
 * Description(描述)： 如果是不可中断模式，那么即使使用了 interrupt 也不会让等待中断
 */

public class Test
{
    /**
     * 可重入锁
     */
    private static final ReentrantLock REENTRANT_LOCK = new ReentrantLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("启动...");
                REENTRANT_LOCK.lock();
                try
                {
                    log.debug("获得了锁");
                }
                finally
                {
                    log.debug("释放锁");
                    REENTRANT_LOCK.unlock();
                }
            }
        }, "t1");

        //主线程获取锁
        REENTRANT_LOCK.lock();
        log.debug("main线程获得了锁");
        thread.start();
        try
        {
            Thread.sleep(1000);
            //打断
            log.debug("执行打断");
            thread.interrupt();
            Thread.sleep(2000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            log.debug("释放锁");
            REENTRANT_LOCK.unlock();
        }
    }
}
```



运行结果：

```sh
2022-09-05  16:24:00.274  [main] DEBUG mao.t4.Test:  main线程获得了锁
2022-09-05  16:24:00.276  [t1] DEBUG mao.t4.Test:  启动...
2022-09-05  16:24:01.277  [main] DEBUG mao.t4.Test:  执行打断
2022-09-05  16:24:03.289  [main] DEBUG mao.t4.Test:  释放锁
2022-09-05  16:24:03.289  [t1] DEBUG mao.t4.Test:  获得了锁
2022-09-05  16:24:03.289  [t1] DEBUG mao.t4.Test:  释放锁
```







### 锁超时



**立刻失败：**



```java
package mao.t5;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:25
 * Version(版本): 1.0
 * Description(描述)： 锁超时 , 立刻失败
 */

public class Test
{
    /**
     * 可重入锁
     */
    private static final ReentrantLock REENTRANT_LOCK = new ReentrantLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(500);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                boolean lock = REENTRANT_LOCK.tryLock();
                if (!lock)
                {
                    log.debug("获取锁失败，直接返回");
                    return;
                }
                try
                {
                    log.debug("获取锁成功");
                }
                finally
                {
                    log.debug("释放锁");
                    REENTRANT_LOCK.unlock();
                }
            }
        }, "t1");

        REENTRANT_LOCK.lock();
        log.debug("获得锁");
        thread.start();
        try
        {
            Thread.sleep(2000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            log.debug("释放锁");
            REENTRANT_LOCK.unlock();
        }
    }
}
```



运行结果：

```sh
2022-09-05  16:31:22.420  [main] DEBUG mao.t5.Test:  获得锁
2022-09-05  16:31:22.930  [t1] DEBUG mao.t5.Test:  获取锁失败，直接返回
2022-09-05  16:31:24.437  [main] DEBUG mao.t5.Test:  释放锁
```



**超时失败：**



```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:35
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 可重入锁
     */
    private static final ReentrantLock REENTRANT_LOCK = new ReentrantLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(500);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                boolean lock = false;
                try
                {
                    log.debug("开始尝试获取锁，超时时间为500毫秒");
                    lock = REENTRANT_LOCK.tryLock(500, TimeUnit.MILLISECONDS);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                if (!lock)
                {
                    log.debug("获取锁失败，直接返回");
                    return;
                }
                try
                {
                    log.debug("获取锁成功");
                }
                finally
                {
                    log.debug("释放锁");
                    REENTRANT_LOCK.unlock();
                }
            }
        }, "t1");

        REENTRANT_LOCK.lock();
        log.debug("获得锁");
        thread.start();
        try
        {
            Thread.sleep(2000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            log.debug("释放锁");
            REENTRANT_LOCK.unlock();
        }
    }
}
```



运行结果：

```sh
2022-09-05  16:40:38.206  [main] DEBUG mao.t6.Test:  获得锁
2022-09-05  16:40:38.709  [t1] DEBUG mao.t6.Test:  开始尝试获取锁，超时时间为500毫秒
2022-09-05  16:40:39.210  [t1] DEBUG mao.t6.Test:  获取锁失败，直接返回
2022-09-05  16:40:40.222  [main] DEBUG mao.t6.Test:  释放锁
```





### 哲学家就餐问题

可以用ReentrantLock的tryLock方法解决死锁问题



```java
package mao.t7;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t7
 * Class(类名): Chopstick
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Chopstick extends ReentrantLock
{
    /**
     * 筷子名字
     */
    String name;

    /**
     * 筷子
     *
     * @param name 名字
     */
    public Chopstick(String name)
    {
        this.name = name;
    }

    /**
     * 字符串
     *
     * @return {@link String}
     */
    @Override
    public String toString()
    {
        return "筷子" + name;
    }
}
```





```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t7
 * Class(类名): Philosopher
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Philosopher extends Thread
{
    private static final Logger log = LoggerFactory.getLogger(Philosopher.class);

    /**
     * 左边筷子
     */
    final Chopstick leftChopstick;

    /**
     * 右边筷子
     */
    final Chopstick rightChopstick;

    /**
     * 哲学家
     *
     * @param name           线程名字，也就是哲学家名字
     * @param leftChopstick  左边筷子
     * @param rightChopstick 右边筷子
     */
    public Philosopher(String name, Chopstick leftChopstick, Chopstick rightChopstick)
    {
        super(name);
        this.leftChopstick = leftChopstick;
        this.rightChopstick = rightChopstick;
    }

    /**
     * 吃饭
     */
    private void eat()
    {
        log.debug("eating...");
        try
        {
            Thread.sleep(200);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    @Override
    public void run()
    {
        while (true)
        {
            //尝试获得左手筷子
            if (leftChopstick.tryLock())
            {
                try
                {
                    //尝试获得右手筷子
                    if (rightChopstick.tryLock())
                    {
                        try
                        {
                            eat();
                        }
                        finally
                        {
                            //放下右手筷子
                            rightChopstick.unlock();
                        }
                    }
                }
                finally
                {
                    //放下左手筷子
                    leftChopstick.unlock();
                }
            }
        }
    }
}
```





```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t7
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        Chopstick c1 = new Chopstick("1");
        Chopstick c2 = new Chopstick("2");
        Chopstick c3 = new Chopstick("3");
        Chopstick c4 = new Chopstick("4");
        Chopstick c5 = new Chopstick("5");
        new Philosopher("苏格拉底", c1, c2).start();
        new Philosopher("柏拉图", c2, c3).start();
        new Philosopher("亚里士多德", c3, c4).start();
        new Philosopher("赫拉克利特", c4, c5).start();
        new Philosopher("阿基米德", c5, c1).start();

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.info("程序退出");
            }
        }, "ShutdownHook"));
    }
}
```



运行结果：

```sh
2022-09-05  16:50:10.430  [亚里士多德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:10.430  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:10.637  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:10.637  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:10.841  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:10.841  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.047  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.047  [亚里士多德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.255  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.255  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.461  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.461  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.667  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.667  [亚里士多德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.874  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:11.874  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.077  [亚里士多德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.077  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.283  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.283  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.489  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.489  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.697  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.697  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.903  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:12.903  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.107  [亚里士多德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.107  [苏格拉底] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.312  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.312  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.519  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.519  [亚里士多德] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.725  [赫拉克利特] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.725  [柏拉图] DEBUG mao.t7.Philosopher:  eating...
2022-09-05  16:50:13.930  [阿基米德] DEBUG mao.t7.Philosopher:  eating...
...
```







### 公平锁

ReentrantLock 默认是不公平的



```java
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```



```java
/**
 * Sync object for fair locks
 */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    /**
     * Acquires only if reentrant or queue is empty.
     */
    final boolean initialTryLock() {
        Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedThreads() && compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        } else if (getExclusiveOwnerThread() == current) {
            if (++c < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(c);
            return true;
        }
        return false;
    }

    /**
     * Acquires only if thread is first waiter or empty
     */
    protected final boolean tryAcquire(int acquires) {
        if (getState() == 0 && !hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
}
```



```java
/**
 * Sync object for non-fair locks
 */
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    final boolean initialTryLock() {
        Thread current = Thread.currentThread();
        if (compareAndSetState(0, 1)) { // first attempt is unguarded
            setExclusiveOwnerThread(current);
            return true;
        } else if (getExclusiveOwnerThread() == current) {
            int c = getState() + 1;
            if (c < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(c);
            return true;
        } else
            return false;
    }

    /**
     * Acquire for non-reentrant cases after initialTryLock prescreen
     */
    protected final boolean tryAcquire(int acquires) {
        if (getState() == 0 && compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
}
```







非公平锁：

```java
package mao.t8;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t8
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 16:52
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 非公平锁
     */
    private static final ReentrantLock lock = new ReentrantLock(false);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        lock.lock();
        for (int i = 0; i < 50; i++)
        {
            new Thread(() ->
            {
                lock.lock();
                try
                {
                    log.debug("运行");
                }
                finally
                {
                    lock.unlock();
                }
            }, "t" + i).start();
        }
        //1s之后去争抢锁
        Thread.sleep(1000);
        new Thread(() ->
        {
            log.debug("--------------->开始运行");
            lock.lock();
            try
            {
                log.debug("---------------->运行");
            }
            finally
            {
                lock.unlock();
            }
        }, "强行插入").start();
        lock.unlock();
    }

}

```



强行插入，有机会在中间输出

```sh
2022-09-05  17:03:54.751  [t0] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.751  [强行插入] DEBUG mao.t8.Test:  --------------->开始运行
2022-09-05  17:03:54.752  [强行插入] DEBUG mao.t8.Test:  ---------------->运行
2022-09-05  17:03:54.753  [t1] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.753  [t2] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.753  [t3] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.753  [t4] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.754  [t5] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.754  [t6] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.754  [t7] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.754  [t8] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.754  [t9] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.755  [t10] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.755  [t11] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.755  [t12] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.755  [t13] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.755  [t14] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t15] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t16] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t17] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t18] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t19] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t20] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.756  [t21] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.757  [t22] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.757  [t23] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.757  [t24] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.757  [t25] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.758  [t26] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.758  [t27] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.758  [t28] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.758  [t29] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.758  [t30] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.759  [t31] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.759  [t32] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.759  [t33] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.759  [t34] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.759  [t35] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.759  [t36] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.760  [t37] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.760  [t38] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.760  [t39] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.760  [t40] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.760  [t41] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.761  [t42] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.761  [t43] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.761  [t44] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.761  [t45] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.761  [t46] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.761  [t47] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.762  [t48] DEBUG mao.t8.Test:  运行
2022-09-05  17:03:54.762  [t49] DEBUG mao.t8.Test:  运行
```





公平锁实现：

```java
package mao.t9;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t9
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 17:01
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 非公平锁
     */
    private static final ReentrantLock lock = new ReentrantLock(true);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        lock.lock();
        for (int i = 0; i < 50; i++)
        {
            new Thread(() ->
            {
                lock.lock();
                try
                {
                    log.debug("运行");
                }
                finally
                {
                    lock.unlock();
                }
            }, "t" + i).start();
        }
        //1s之后去争抢锁
        Thread.sleep(1000);
        new Thread(() ->
        {
            log.debug("--------------->开始运行");
            lock.lock();
            try
            {
                log.debug("---------------->运行");
            }
            finally
            {
                lock.unlock();
            }
        }, "强行插入").start();
        lock.unlock();
    }

}

```



强行插入，总是在最后输出

```sh
2022-09-05  17:04:51.359  [强行插入] DEBUG mao.t9.Test:  --------------->开始运行
2022-09-05  17:04:51.359  [t1] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.361  [t2] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.361  [t3] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.362  [t0] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.362  [t4] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.362  [t5] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.362  [t6] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.362  [t7] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.362  [t8] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.363  [t9] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.363  [t10] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.363  [t11] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.363  [t12] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.363  [t13] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.364  [t14] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.364  [t15] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.364  [t16] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.364  [t17] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.364  [t18] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.364  [t19] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.365  [t20] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.365  [t21] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.365  [t22] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.365  [t23] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.365  [t24] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.366  [t25] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.366  [t26] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.366  [t27] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.366  [t28] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.366  [t29] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.366  [t30] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.367  [t31] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.367  [t32] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.367  [t33] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.367  [t34] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.367  [t35] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t36] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t37] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t38] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t39] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t40] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t41] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.368  [t42] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.369  [t43] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.369  [t44] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.369  [t45] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.369  [t46] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.369  [t47] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.369  [t48] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.370  [t49] DEBUG mao.t9.Test:  运行
2022-09-05  17:04:51.370  [强行插入] DEBUG mao.t9.Test:  ---------------->运行
```



**公平锁一般没有必要，因为会降低并发度**





### 条件变量

ReentrantLock 的条件变量比 synchronized 强大之处在于，它是支持多个条件变量的，这就好比

* synchronized 是那些不满足条件的线程都在一间休息室等消息
* 而 ReentrantLock 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒



使用要点：

* await 前需要获得锁
* await 执行后，会释放锁，进入 conditionObject 等待
* await 的线程被唤醒（或打断、或超时）重新竞争 lock 锁
* 竞争 lock 锁成功后，从 await 后继续执行





```java
package mao.t10;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t10
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 17:13
 * Version(版本): 1.0
 * Description(描述)： 条件变量
 */

public class Test
{
    /**
     * 锁
     */
    private static final ReentrantLock LOCK = new ReentrantLock();

    /**
     * 条件
     */
    private static final Condition condition = LOCK.newCondition();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args)
    {
        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("获取锁");
                LOCK.lock();
                try
                {
                    log.debug("获取到锁");
                    Thread.sleep(2000);
                    log.debug("唤醒");
                    condition.signal();
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放锁");
                    LOCK.unlock();
                }
            }
        }, "t1");

        log.debug("获取锁");
        LOCK.lock();
        log.debug("获取到锁");
        thread.start();
        try
        {
            Thread.sleep(500);
            log.debug("await");
            condition.await();
            log.debug("被唤醒");
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        finally
        {
            log.debug("释放锁");
            LOCK.unlock();
        }
    }
}

```



运行结果：

```sh
2022-09-05  17:22:25.566  [main] DEBUG mao.t10.Test:  获取锁
2022-09-05  17:22:25.568  [main] DEBUG mao.t10.Test:  获取到锁
2022-09-05  17:22:25.569  [t1] DEBUG mao.t10.Test:  获取锁
2022-09-05  17:22:26.071  [main] DEBUG mao.t10.Test:  await
2022-09-05  17:22:26.071  [t1] DEBUG mao.t10.Test:  获取到锁
2022-09-05  17:22:28.086  [t1] DEBUG mao.t10.Test:  唤醒
2022-09-05  17:22:28.086  [t1] DEBUG mao.t10.Test:  释放锁
2022-09-05  17:22:28.086  [main] DEBUG mao.t10.Test:  被唤醒
2022-09-05  17:22:28.087  [main] DEBUG mao.t10.Test:  释放锁
```







```java
package mao.t11;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantLock
 * Package(包名): mao.t11
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/5
 * Time(创建时间)： 17:24
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    private static final ReentrantLock lock = new ReentrantLock();

    private static final Condition waitCigaretteQueue = lock.newCondition();

    private static final Condition waitbreakfastQueue = lock.newCondition();

    private static boolean hasCigrette = false;

    private static boolean hasBreakfast = false;

    private static final Logger log = LoggerFactory.getLogger(Test.class);


    private static void sendCigarette()
    {
        lock.lock();
        try
        {
            log.debug("送烟来了");
            hasCigrette = true;
            waitCigaretteQueue.signal();
        }
        finally
        {
            lock.unlock();
        }
    }

    private static void sendBreakfast()
    {
        lock.lock();
        try
        {
            log.debug("送早餐来了");
            hasBreakfast = true;
            waitbreakfastQueue.signal();
        }
        finally
        {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    lock.lock();
                    while (!hasCigrette)
                    {
                        try
                        {
                            log.debug("开始等待它的烟");
                            waitCigaretteQueue.await();
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                    }
                    log.debug("等到了它的烟");
                }
                finally
                {
                    lock.unlock();
                }
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    lock.lock();
                    while (!hasBreakfast)
                    {
                        try
                        {
                            log.debug("开始等待它的早餐");
                            waitbreakfastQueue.await();
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                    }
                    log.debug("等到了它的早餐");
                }
                finally
                {
                    lock.unlock();
                }
            }
        }, "t2").start();

        Thread.sleep(1000);
        sendBreakfast();
        Thread.sleep(1000);
        sendCigarette();
    }

}
```



运行结果：

```sh
2022-09-05  17:34:26.745  [t1] DEBUG mao.t11.Test:  开始等待它的烟
2022-09-05  17:34:26.747  [t2] DEBUG mao.t11.Test:  开始等待它的早餐
2022-09-05  17:34:27.746  [main] DEBUG mao.t11.Test:  送早餐来了
2022-09-05  17:34:27.746  [t2] DEBUG mao.t11.Test:  等到了它的早餐
2022-09-05  17:34:28.748  [main] DEBUG mao.t11.Test:  送烟来了
2022-09-05  17:34:28.748  [t1] DEBUG mao.t11.Test:  等到了它的烟
```









## 同步模式之顺序控制

### 固定运行顺序

必须先 t2先打印，然后 t1 打印



####  wait notify



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 11:04
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * 锁
     */
    private static final Object LOCK = new Object();

    //t2运行标记
    private static boolean t2isRun = false;

    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (LOCK)
                {
                    while (!t2isRun)
                    {
                        try
                        {
                            LOCK.wait();
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                    }
                }
                log.debug("t1");
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                synchronized (LOCK)
                {
                    log.debug("t2");
                    //修改运行标记
                    t2isRun = true;
                    LOCK.notifyAll();
                }
            }
        }, "t2");

        t1.start();
        Thread.sleep(20);
        t2.start();

    }

}
```



运行结果：

```sh
2022-09-06  11:09:56.782  [t2] DEBUG mao.t1.Test:  t2
2022-09-06  11:09:56.785  [t1] DEBUG mao.t1.Test:  t1
```





#### Park Unpark

使用wait notify存在的问题：

* 首先，需要保证先 wait 再 notify，否则 wait 线程永远得不到唤醒。因此使用了『运行标记』来判断该不该 wait
* 第二，如果有些干扰线程错误地 notify 了 wait 线程，条件不满足时还要重新等待，使用了 while 循环来解决此问题
* 最后，唤醒对象上的 wait 线程需要使用 notifyAll，因为『同步对象』上的等待线程可能不止一个



可以使用 LockSupport 类的 park 和 unpark来实现



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.LockSupport;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 11:13
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);


    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                LockSupport.park();
                log.debug("t1");
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("t2");
                LockSupport.unpark(t1);
            }
        }, "t2");

        t1.start();
        Thread.sleep(20);
        t2.start();

    }
}
```



```sh
2022-09-06  11:15:36.834  [t2] DEBUG mao.t2.Test:  t2
2022-09-06  11:15:36.836  [t1] DEBUG mao.t2.Test:  t1
```



park 和 unpark 方法比较灵活，他俩谁先调用，谁后调用无所谓。并且是以线程为单位进行『暂停』和『恢复』， 不需要『同步对象』和『运行标记』







### 交替输出

线程 1 输出 a 5 次，线程 2 输出 b 5 次，线程 3 输出 c 5 次。现在要求输出 abcabcabcabcabc



#### wait notify



```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t3
 * Class(类名): WaitNotify
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 11:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class WaitNotify
{
    /**
     * 运行标记
     */
    private int flag;

    /**
     * 循环次数
     */
    private final int loopNumber;


    /**
     * Instantiates a new Wait notify.
     *
     * @param flag       the flag
     * @param loopNumber the loop number
     */
    public WaitNotify(int flag, int loopNumber)
    {
        this.flag = flag;
        this.loopNumber = loopNumber;
    }

    /**
     * 打印
     *
     * @param waitFlag 等待标记
     * @param nextFlag 下一个标记
     * @param str      字符串
     */
    public void print(int waitFlag, int nextFlag, String str)
    {
        //循环
        for (int i = 0; i < loopNumber; i++)
        {
            synchronized (this)
            {
                //不是就等待
                while (this.flag != waitFlag)
                {
                    try
                    {
                        this.wait();
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                //是
                //输出
                System.out.print(str);
                this.flag = nextFlag;
                this.notifyAll();
            }
        }
    }
}
```





```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 11:17
 * Version(版本): 1.0
 * Description(描述)： 交替输出，wait notify
 */

public class Test
{
    public static void main(String[] args)
    {
        WaitNotify waitNotify = new WaitNotify(1, 5);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                waitNotify.print(1, 2, "a");
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                waitNotify.print(2, 3, "b");

            }
        }, "t2").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                waitNotify.print(3, 1, "c");
            }
        }, "t3").start();
    }
}
```



运行结果：

```sh
abcabcabcabcabc
```





#### Lock 条件变量



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t4
 * Class(类名): AwaitSignal
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 12:10
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class AwaitSignal extends ReentrantLock
{
    private static final Logger log = LoggerFactory.getLogger(AwaitSignal.class);

    /**
     * 循环数
     */
    private final int loopNumber;

    /**
     * 等待信号
     *
     * @param loopNumber 循环数
     */
    public AwaitSignal(int loopNumber)
    {
        this.loopNumber = loopNumber;
    }

    /**
     * 开始
     *
     * @param first 第一个
     */
    public void start(Condition first)
    {
        this.lock();
        try
        {
            log.debug("开始");
            first.signal();
        }
        finally
        {
            this.unlock();
        }
    }

    /**
     * 打印
     *
     * @param str     str
     * @param current 当前
     * @param next    下一个
     */
    public void print(String str, Condition current, Condition next)
    {
        for (int i = 0; i < loopNumber; i++)
        {
            this.lock();
            try
            {
                current.await();
                System.out.print(str);
                next.signal();
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
            finally
            {
                this.unlock();
            }
        }
    }

}
```



```java
package mao.t4;

import java.util.concurrent.locks.Condition;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 12:14
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        AwaitSignal awaitSignal = new AwaitSignal(5);
        Condition condition1 = awaitSignal.newCondition();
        Condition condition2 = awaitSignal.newCondition();
        Condition condition3 = awaitSignal.newCondition();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                awaitSignal.print("a", condition1, condition2);
            }
        }, "t1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                awaitSignal.print("b", condition2, condition3);
            }
        }, "t2").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                awaitSignal.print("c", condition3, condition1);
            }
        }, "t3").start();

        awaitSignal.start(condition1);

    }
}
```



运行结果：

```sh
2022-09-06  12:17:33.399  [main] DEBUG mao.t4.AwaitSignal:  开始
abcabcabcabcabc
```





####  Park Unpark



```java
package mao.t5;

import java.util.concurrent.locks.LockSupport;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t5
 * Class(类名): ParkUnpark
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 12:20
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ParkUnpark
{
    /**
     * 循环数
     */
    private final int loopNumber;
    /**
     * 线程数组
     */
    private Thread[] threads;

    public ParkUnpark(int loopNumber)
    {
        this.loopNumber = loopNumber;
    }

    /**
     * 设置线程
     *
     * @param threads 线程
     */
    public void setThreads(Thread... threads)
    {
        this.threads = threads;
    }

    /**
     * 打印
     *
     * @param str str
     */
    public void print(String str)
    {
        for (int i = 0; i < loopNumber; i++)
        {
            LockSupport.park();
            System.out.print(str);
            LockSupport.unpark(nextThread());
        }
    }

    /**
     * 下一个线程
     *
     * @return {@link Thread}
     */
    private Thread nextThread()
    {
        Thread current = Thread.currentThread();
        int index = 0;

        for (int i = 0; i < threads.length; i++)
        {
            if (threads[i] == current)
            {
                index = i;
                break;
            }
        }
        if (index < threads.length - 1)
        {
            return threads[index + 1];
        }
        else
        {
            return threads[0];
        }
    }

    /**
     * 开始
     */
    public void start()
    {
        for (Thread thread : threads)
        {
            thread.start();
        }
        LockSupport.unpark(threads[0]);
    }
}
```



```java
package mao.t5;

/**
 * Project name(项目名称)：java并发编程_顺序控制
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 12:20
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        ParkUnpark parkUnpark = new ParkUnpark(5);

        Thread thread1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                parkUnpark.print("a");
            }
        }, "t1");

        Thread thread2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                parkUnpark.print("b");
            }
        }, "t2");

        Thread thread3 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                parkUnpark.print("c");
            }
        }, "t3");

        parkUnpark.setThreads(thread1, thread2, thread3);

        parkUnpark.start();
    }
}
```



运行结果：

```sh
abcabcabcabcabc
```















# 共享模型之内存

## Java 内存模型

JMM 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、 CPU 指令优化等

JMM 体现在以下几个方面：

* 原子性 - 保证指令不会受到线程上下文切换的影响
* 可见性 - 保证指令不会受 cpu 缓存的影响
* 有序性 - 保证指令不会受 cpu 指令并行优化的影响





## 可见性

### **退不出的循环**

main 线程对 run 变量的修改对于 t1 线程不可见，导致了 t1 线程无法停止



```java
package mao.t1;

import java.util.Date;

/**
 * Project name(项目名称)：java并发编程_Java内存模型_可见性
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 12:43
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static boolean run = true;

    public static void main(String[] args) throws InterruptedException
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (run)
                {
                    //运行
                }
                System.out.println("运行结束");
            }
        }, "t1").start();

        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    System.out.println("------>" + new Date());
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
            }
        }, "t2");
        thread.setDaemon(true);
        thread.start();

        Thread.sleep(1500);
        System.out.println("开始停止");
        run = false; //并不会停止运行
    }
}
```



运行结果：

```sh
------>Tue Sep 06 12:49:01 CST 2022
------>Tue Sep 06 12:49:02 CST 2022
开始停止
------>Tue Sep 06 12:49:03 CST 2022
------>Tue Sep 06 12:49:04 CST 2022
------>Tue Sep 06 12:49:05 CST 2022
------>Tue Sep 06 12:49:06 CST 2022
------>Tue Sep 06 12:49:07 CST 2022
------>Tue Sep 06 12:49:08 CST 2022
------>Tue Sep 06 12:49:09 CST 2022
...
```





**分析：**



* 初始状态， t1 线程刚开始从主内存读取了 run 的值到工作内存



![image-20220906125127149](img/java并发编程学习笔记/image-20220906125127149.png)





*  因为 t1 线程要频繁从主内存中读取 run 的值，JIT 编译器会将 run 的值缓存至自己工作内存中的高速缓存中， 减少对主存中 run 的访问，提高效率



![image-20220906125358425](img/java并发编程学习笔记/image-20220906125358425.png)





* 1.5秒之后，main 线程修改了 run 的值，并同步至主存，而 t1 是从自己工作内存中的高速缓存中读取这个变量的值，结果永远是旧值



![image-20220906125253893](img/java并发编程学习笔记/image-20220906125253893.png)







### 解决

**在字段添加volatile关键字**

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存





```java
package mao.t2;

import java.util.Date;

/**
 * Project name(项目名称)：java并发编程_Java内存模型_可见性
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 12:55
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static volatile boolean run = true;

    public static void main(String[] args) throws InterruptedException
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (run)
                {
                    //运行
                }
                System.out.println("运行结束");
            }
        }, "t1").start();

        Thread thread = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                while (true)
                {
                    System.out.println("------>" + new Date());
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
            }
        }, "t2");
        thread.setDaemon(true);
        thread.start();

        Thread.sleep(1500);
        System.out.println("开始停止");
        run = false;
    }
}
```



运行结果：

```sh
------>Tue Sep 06 12:56:34 CST 2022
------>Tue Sep 06 12:56:35 CST 2022
开始停止
运行结束
```





### 可见性 vs 原子性

可见性，它保证的是在多个线程之间，一个线程对 volatile 变量的修改对另一个线程可见， 不能保证原子性，仅用在一个写线程，多个读线程的情况

比如两个线程一个 **i++**操作 ，一个 **i--**操作，只能保证看到最新值，不能解决指令交错



synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是 synchronized 是属于重量级操作，性能相对更低





### 案例

单例模式

使用双重检查锁

对于 `getInstance()` 方法来说，绝大部分的操作都是读操作，读操作是线程安全的，所以我们没必让每个线程必须持有锁才能调用该方法，我们需要调整加锁的时机。由此也产生了一种新的实现模式：双重检查锁模式





```java
package mao.m5;

/**
 * Project name(项目名称)：java设计模式_单例模式
 * Package(包名): mao.m5
 * Class(类名): Singleton
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/11
 * Time(创建时间)： 22:28
 * Version(版本): 1.0
 * Description(描述)： 懒汉式-方式3（双重检查锁）
 */

public class Singleton
{
    public String str = "hello world";

    public String show()
    {
        return "show";
    }

    /**
     * 私有化构造方法
     */
    private Singleton()
    {
        System.out.println("实例私有化构造方法");
    }

    private static Singleton instance;

    /**
     * 对外提供方法获取该对象
     * 线程安全
     *
     * @return Singleton对象
     */
    public static Singleton getInstance()
    {
        //第一次判断，如果instance不为null，不进入抢锁阶段，直接返回实例
        if (instance == null)
        {
            synchronized (Singleton.class)
            {
                //抢到锁之后再次判断是否为null
                if (instance == null)
                {
                    System.out.println("创建对象实例");
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```



```java
package mao.m5;



/**
 * Project name(项目名称)：java设计模式_单例模式
 * Package(包名): mao.m5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/11
 * Time(创建时间)： 22:34
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args) throws ClassNotFoundException, InterruptedException
    {
        Class.forName("mao.m5.Singleton");
        Thread.sleep(1000);
        System.out.println(Singleton.getInstance().str);
        System.out.println(Singleton.getInstance().show());
        //打印的内存地址都一样，单例
        System.out.println(Singleton.getInstance());
        System.out.println(Singleton.getInstance());
        System.out.println(Singleton.getInstance());
    }
}
```





```java
package mao.m5;


/**
 * Project name(项目名称)：java设计模式_单例模式
 * Package(包名): mao.m5
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/11
 * Time(创建时间)： 22:35
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    public static void main(String[] args)
    {
        Thread[] threads = new Thread[100];
        for (int i = 0; i < 100; i++)
        {
            //可简写为：new Thread(Singleton::getInstance);
            threads[i] = new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    System.out.println(Singleton.getInstance());
                }
            });
        }
        for (int i = 0; i < 100; i++)
        {
            threads[i].start();
        }
    }
}
```



双重检查锁模式是一种非常好的单例实现模式，解决了单例、性能、线程安全问题，上面的双重检测锁模式看上去完美无缺，其实是存在问题，在多线程的情况下，可能会出现空指针问题，出现问题的原因是JVM在实例化对象的时候会进行优化和指令重排序操作。

要解决双重检查锁模式带来空指针异常的问题，只需要使用 `volatile` 关键字, `volatile` 关键字可以保证可见性和有序性。





```java
package mao.m5;

/**
 * Project name(项目名称)：java设计模式_单例模式
 * Package(包名): mao.m5
 * Class(类名): Singleton
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/8/11
 * Time(创建时间)： 22:28
 * Version(版本): 1.0
 * Description(描述)： 懒汉式-方式3（双重检查锁）
 * <p>
 * 在多线程的情况下，可能会出现空指针问题，出现问题的原因是JVM在实例化对象的时候会进行优化和指令重排序操作。
 * 要解决双重检查锁模式带来空指针异常的问题，只需要使用 `volatile` 关键字, `volatile` 关键字可以保证可见性和有序性。
 */

public class Singleton
{
    public String str = "hello world";

    public String show()
    {
        return "show";
    }

    /**
     * 私有化构造方法
     */
    private Singleton()
    {
        System.out.println("实例私有化构造方法");
    }

    private static volatile Singleton instance;

    /**
     * 对外提供方法获取该对象
     * 线程安全
     *
     * @return Singleton对象
     */
    public static Singleton getInstance()
    {
        //第一次判断，如果instance不为null，不进入抢锁阶段，直接返回实例
        if (instance == null)
        {
            synchronized (Singleton.class)
            {
                //抢到锁之后再次判断是否为null
                if (instance == null)
                {
                    System.out.println("创建对象实例");
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```







## 有序性

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序



```
static int i;
static int j;
// 在某个线程内执行如下赋值操作
i = ...; 
j = ...; 
```



至于是先执行 i 还是 先执行 j ，对最终的结果不会产生影响。所以，上面代码真正执行时，既可以是

```
i = ...; 
j = ...;
```

也可以是

```
j = ...;
i = ...;
```







### 指令级并行

#### Clock Cycle Time

CPU 的 Clock Cycle Time（时钟周期时间），等于主频的倒数，意思是 CPU 能 够识别的最小时间单位，比如说 4G 主频的 CPU 的 Clock Cycle Time 就是 0.25 ns，作为对比，我们墙上挂钟的 Cycle Time 是 1s 例如，运行一条加法指令一般需要一个时钟周期时间



#### CPI

有的指令需要更多的时钟周期时间，所以引出了 CPI （Cycles Per Instruction）指令平均时钟周期数



#### IPC

IPC（Instruction Per Clock Cycle） 即 CPI 的倒数，表示每个时钟周期能够运行的指令数



#### CPU 执行时间

程序的 CPU 执行时间，可以用下面的公式来表示

```
程序 CPU 执行时间 = 指令数 * CPI * Clock Cycle Time 
```





#### 故事

加工一条鱼需要 50 分钟，只能一条鱼、一条鱼顺序加工



![image-20220906191918021](img/java并发编程学习笔记/image-20220906191918021.png)





可以将每个鱼罐头的加工流程细分为 5 个步骤：

* 去鳞清洗 10分钟 
* 蒸煮沥水 10分钟 
* 加注汤料 10分钟 
* 杀菌出锅 10分钟 
* 真空封罐 10分钟



![image-20220906192027066](img/java并发编程学习笔记/image-20220906192027066.png)





即使只有一个工人，最理想的情况是：他能够在 10 分钟内同时做好这 5 件事，因为对第一条鱼的真空装罐，不会 影响对第二条鱼的杀菌出锅





#### 指令重排序优化

事实上，现代处理器会设计为一个时钟周期完成一条执行时间最长的 CPU 指令。为什么这么做呢？可以想到指令 还可以再划分成一个个更小的阶段，例如，每条指令都可以分为： **取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回** 这 5 个阶段



![image-20220906192156857](img/java并发编程学习笔记/image-20220906192156857.png)



* instruction fetch (IF) 
* instruction decode (ID) 
* execute (EX) 
* memory access (MEM) 
* register write back (WB)



在不改变程序结果的前提下，这些指令的各个阶段可以通过重排序和组合来实现指令级并行



指令重排的前提是，重排指令不能影响结果，例如

```java
// 可以重排的例子
int a = 10; // 指令1
int b = 20; // 指令2
System.out.println( a + b );
```

```java
// 不能重排的例子
int a = 10; // 指令1
int b = a - 5; // 指令2
```







#### 支持流水线的处理器

现代 CPU 支持多级指令流水线，例如支持同时执行 **取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回** 的处理 器，就可以称之为五级指令流水线。这时 CPU 可以在一个时钟周期内，同时运行五条指令的不同阶段（相当于一 条执行时间最长的复杂指令），IPC = 1，本质上，流水线技术并不能缩短单条指令的执行时间，但它变相地提高了 指令地吞吐率。



![image-20220906192456527](img/java并发编程学习笔记/image-20220906192456527.png)





#### SuperScalar 处理器

大多数处理器包含多个执行单元，并不是所有计算功能都集中在一起，可以再细分为整数运算单元、浮点数运算单 元等，这样可以把多条指令也可以做到并行获取、译码等，CPU 可以在一个时钟周期内，执行多于一条指令，IPC > 1







### 防止指令重排

volatile 修饰的变量，可以禁用指令重排





##  volatile 原理

volatile 的底层实现原理是内存屏障，Memory Barrier（Memory Fence） 

* 对 volatile 变量的写指令后会加入写屏障 
* 对 volatile 变量的读指令前会加入读屏障





### 如何保证可见性

写屏障（sfence）保证在该屏障之前的，对共享变量的改动，都同步到主存当中



```java
public void actor2(I_Result r) {
 num = 2;
 ready = true; // ready 是 volatile 赋值带写屏障
 // 写屏障
}
```



而读屏障（lfence）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据

```java
public void actor1(I_Result r) {
 // 读屏障
 // ready 是 volatile 读取值带读屏障
 if(ready) {
 r.r1 = num + num;
 } else {
 r.r1 = 1;
 }
}
```





![image-20220906193601805](img/java并发编程学习笔记/image-20220906193601805.png)







### 如何保证有序性

* 写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后

* 读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前



* 写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面去

* 而有序性的保证也只是保证了本线程内相关代码不被重排序



![image-20220906194154526](img/java并发编程学习笔记/image-20220906194154526.png)









## happens-before

happens-before 规定了对共享变量的写操作对其它线程的读操作可见，它是可见性与有序性的一套规则总结，抛开以下 happens-before 规则，JMM 并不能保证一个线程对共享变量的写，对于其它线程对该共享变量的读可见



### 规则一

线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见



```java
package mao.t1;

/**
 * Project name(项目名称)：java并发编程_happens_before
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 19:46
 * Version(版本): 1.0
 * Description(描述)： 线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见
 */

public class Test
{
    private static int x;
    private static final Object m = new Object();

    public static void main(String[] args)
    {
        new Thread(() ->
        {
            synchronized (m)
            {
                x = 10;
            }
        }, "t1").start();

        new Thread(() ->
        {
            synchronized (m)
            {
                System.out.println(x);
            }
        }, "t2").start();
    }
}
```



运行结果：

```sh
10
```





### 规则二

线程对 volatile 变量的写，对接下来其它线程对该变量的读可见



```java
package mao.t2;

/**
 * Project name(项目名称)：java并发编程_happens_before
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 19:49
 * Version(版本): 1.0
 * Description(描述)： 线程对 volatile 变量的写，对接下来其它线程对该变量的读可见
 */

public class Test
{
    private volatile static int x;

    public static void main(String[] args)
    {

        new Thread(() ->
        {
            x = 10;
        }, "t1").start();


        new Thread(() ->
        {
            System.out.println(x);
        }, "t2").start();
    }
}
```



运行结果：

```sh
10
```





### 规则三

线程 start 前对变量的写，对该线程开始后对该变量的读可见



```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_happens_before
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 19:52
 * Version(版本): 1.0
 * Description(描述)： 线程 start 前对变量的写，对该线程开始后对该变量的读可见
 */

public class Test
{
    static int x;

    public static void main(String[] args)
    {
        x = 10;

        new Thread(() ->
        {
            System.out.println(x);
        }, "t2").start();

    }
}
```



运行结果：

```sh
10
```





### 规则四

线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束）



```java
package mao.t4;

/**
 * Project name(项目名称)：java并发编程_happens_before
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 19:59
 * Version(版本): 1.0
 * Description(描述)： 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束）
 */

public class Test
{
    private static int x;

    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(() ->
        {
            x = 10;

        }, "t1");
        t1.start();

        t1.join();
        System.out.println(x);

    }

}
```



运行结果：

```sh
10
```





### 规则五

线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过 t2.interrupted 或 t2.isInterrupted）



```java
package mao.t5;

/**
 * Project name(项目名称)：java并发编程_happens_before
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 20:05
 * Version(版本): 1.0
 * Description(描述)： 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见
 * （通过t2.interrupted 或 t2.isInterrupted）
 */

public class Test
{
    private static int x;

    public static void main(String[] args)
    {
        Thread t2 = new Thread(() ->
        {
            while (true)
            {
                if (Thread.currentThread().isInterrupted())
                {
                    System.out.println(x);
                    break;
                }
            }
        }, "t2");


        t2.start();

        new Thread(() ->
        {
            try
            {
                Thread.sleep(1000);
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
            x = 10;
            t2.interrupt();
        }, "t1").start();


        while (!t2.isInterrupted())
        {
            Thread.yield();
        }
        System.out.println(x);
    }

}
```



运行结果：

```sh
10
10
```





### 规则六

对变量默认值（0，false，null）的写，对其它线程对该变量的读可见





### 规则七

具有传递性，如果 x hb-> y 并且 y hb-> z 那么有 x hb-> z 



```java
package mao.t6;

/**
 * Project name(项目名称)：java并发编程_happens_before
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/6
 * Time(创建时间)： 20:10
 * Version(版本): 1.0
 * Description(描述)： 具有传递性
 */

public class Test
{
    volatile static int x;
    static int y;

    public static void main(String[] args)
    {
        new Thread(() ->
        {
            y = 10;
            x = 20;
            //写屏障，写屏障保证在该屏障之前的，对共享变量的改动，都同步到主存当中
            //y在写屏障之前
        }, "t1").start();


        new Thread(() ->
        {
            // x=20 对 t2 可见, 同时 y=10 也对 t2 可见
            System.out.println(x);
        }, "t2").start();
    }

}
```



运行结果：

```sh
20
```



















# 共享模型之无锁

## 取款问题



### 线程不安全实现

```java
package mao;

import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao
 * Interface(接口名): Account
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:27
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public interface Account
{
    /**
     * 获取余额
     *
     * @return {@link Integer}
     */
    Integer getBalance();

    /**
     * 取款
     *
     * @param amount amount
     */
    void withdraw(Integer amount);

    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void start(Account account)
    {
        List<Thread> threads = new ArrayList<>();
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++)
        {
            threads.add(new Thread(() ->
            {
                account.withdraw(10);
            }));
        }
        threads.forEach(Thread::start);
        threads.forEach(t ->
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
        long end = System.nanoTime();
        System.out.println("剩余金额：" + account.getBalance());
        System.out.println("花费时间: " + (end - start) / 1000_000 + " ms");
    }
}
```



```java
package mao.t1;

import mao.Account;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao.t1
 * Class(类名): AccountUnsafe
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:29
 * Version(版本): 1.0
 * Description(描述)： 无
 */
public class AccountUnsafe implements Account
{
    private Integer balance;


    /**
     * Instantiates a new Account unsafe.
     *
     * @param balance the balance
     */
    public AccountUnsafe(Integer balance)
    {
        this.balance = balance;
    }


    @Override
    public Integer getBalance()
    {
        return balance;
    }

    @Override
    public void withdraw(Integer amount)
    {
        balance -= amount;
    }
}
```



```java
package mao.t1;

import mao.Account;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:25
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * @param args 参数
     */
    public static void main(String[] args)
    {
        Account.start(new AccountUnsafe(10000));
    }
}
```



运行结果：

```sh
剩余金额：150
花费时间: 113 ms
```

```sh
剩余金额：650
花费时间: 105 ms
```

```sh
剩余金额：280
花费时间: 98 ms
```





线程不安全



### 加锁方式实现



```java
package mao.t2;

import mao.Account;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao.t2
 * Class(类名): AccountSynchronized
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:46
 * Version(版本): 1.0
 * Description(描述)： 无
 */


public class AccountSynchronized implements Account
{
    private Integer balance;

    /**
     * Instantiates a new Account synchronized.
     *
     * @param balance the balance
     */
    public AccountSynchronized(Integer balance)
    {
        this.balance = balance;
    }

    @Override
    public synchronized Integer getBalance()
    {
        return balance;
    }

    @Override
    public synchronized void withdraw(Integer amount)
    {
        balance -= amount;
    }
}
```



```java
package mao.t2;

import mao.Account;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:47
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        Account.start(new AccountSynchronized(10000));
    }
}
```



运行结果：

```sh
剩余金额：0
花费时间: 98 ms
```

```sh
剩余金额：0
花费时间: 95 ms
```

```sh
剩余金额：0
花费时间: 103 ms
```







### 无锁实现



```java
package mao.t3;

import mao.Account;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao.t3
 * Class(类名): AccountSafe
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:49
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class AccountSafe implements Account
{
    private final AtomicInteger balance;

    public AccountSafe(Integer balance)
    {
        this.balance = new AtomicInteger(balance);
    }


    @Override
    public Integer getBalance()
    {
        return balance.get();
    }

    @Override
    public void withdraw(Integer amount)
    {
        while (true)
        {
            int prev = balance.get();
            int next = prev - amount;
            if (balance.compareAndSet(prev, next))
            {
                break;
            }
        }
        //或者
        //balance.addAndGet(-1 * amount);
    }
}
```



```java
package mao.t3;

import mao.Account;

/**
 * Project name(项目名称)：java并发编程_取款问题无锁实现
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 10:54
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        Account.start(new AccountSafe(10000));
    }
}
```



运行结果：

```sh
剩余金额：0
花费时间: 93 ms
```

```sh
剩余金额：0
花费时间: 92 ms
```

```sh
剩余金额：0
花费时间: 92 ms
```







## CAS 与 volatile

AtomicInteger 的解决方法，内部并没有用锁来保护共享变量的线程安全。那么它是如何实现的呢？

compareAndSet 正是做这个检查，在 set 前，先比较 prev 与当前值 

不一致了，next 作废，返回 false 表示失败 

比如，别的线程已经做了减法，当前值已经被减成了 990 那么本线程的这次 990 就作废了，进入 while 下次循环重试 

一致，以 next 设置为新值，返回 true 表示成功

需要不断尝试，直到成功为止



 compareAndSet，它的简称就是 CAS （也有 Compare And Swap 的说法），它必须是原子操作

![image-20220907110139090](img/java并发编程学习笔记/image-20220907110139090.png)



 CAS 的底层是 lock cmpxchg 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交 换】的原子性

在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的





获取共享变量时，为了保证该变量的可见性，需要使用 volatile 修饰

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取 它的值，线程操作 volatile 变量都是直接操作主存。即一个线程对 volatile 变量的修改，对另一个线程可见

CAS 必须借助 volatile 才能读取到共享变量的最新值来实现【比较并交换】的效果





## 为什么无锁效率高

* 无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 synchronized 会让线程在没有获得锁的时候，发生上下文切换，进入阻塞。打个比喻
* 线程就好像高速跑道上的赛车，高速运行时，速度超快，一旦发生上下文切换，就好比赛车要减速、熄火， 等被唤醒又得重新打火、启动、加速... 恢复到高速运行，代价比较大
* 但无锁情况下，因为线程要保持运行，需要额外 CPU 的支持，CPU 在这里就好比高速跑道，没有额外的跑 道，线程想高速运行也无从谈起，虽然不会进入阻塞，但由于没有分到时间片，仍然会进入可运行状态，还 是会导致上下文切换。





## CAS 的特点

结合 CAS 和 volatile 可以实现无锁并发，适用于线程数少、多核 CPU 的场景下

* CAS 是基于乐观锁的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再 重试呗
* synchronized 是基于悲观锁的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想 改，我改完了解开锁，你们才有机会
* CAS 体现的是无锁并发、无阻塞并发
  * 因为没有使用 synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一
  * 但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响







## 原子整数

J.U.C 并发包提供了： 

* AtomicBoolean 
* AtomicInteger 
* AtomicLong



![image-20220907111526908](img/java并发编程学习笔记/image-20220907111526908.png)





### AtomicInteger 



```java
package mao.t1;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.IntBinaryOperator;
import java.util.function.IntUnaryOperator;

/**
 * Project name(项目名称)：java并发编程_原子整数
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:14
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        AtomicInteger i = new AtomicInteger(0);
        // 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
        System.out.println(i.getAndIncrement());
        // 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
        System.out.println(i.incrementAndGet());
        // 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
        System.out.println(i.decrementAndGet());
        // 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
        System.out.println(i.getAndDecrement());
        // 获取并加值（i = 0, 结果 i = 5, 返回 0）
        System.out.println(i.getAndAdd(5));
        // 加值并获取（i = 5, 结果 i = 0, 返回 0）
        System.out.println(i.addAndGet(-5));
        // 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        //System.out.println(i.getAndUpdate(p -> p - 2));
        System.out.println(i.getAndUpdate(new IntUnaryOperator()
        {
            @Override
            public int applyAsInt(int operand)
            {
                return operand - 2;
            }
        }));
        // 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        //System.out.println(i.updateAndGet(p -> p + 2));
        System.out.println(i.updateAndGet(new IntUnaryOperator()
        {
            @Override
            public int applyAsInt(int operand)
            {
                return operand + 2;
            }
        }));
        // 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        // getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
        // getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
        //System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));
        System.out.println(i.getAndAccumulate(10, new IntBinaryOperator()
        {
            @Override
            public int applyAsInt(int left, int right)
            {
                return left + right;
            }
        }));
        // 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        //System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
        System.out.println(i.accumulateAndGet(-10, new IntBinaryOperator()
        {
            @Override
            public int applyAsInt(int left, int right)
            {
                return left + right;
            }
        }));
    }
}
```





### AtomicLong



```java
package mao.t2;

import java.util.concurrent.atomic.AtomicLong;
import java.util.function.LongBinaryOperator;
import java.util.function.LongUnaryOperator;

/**
 * Project name(项目名称)：java并发编程_原子整数
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:28
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        AtomicLong i = new AtomicLong(0);
        // 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
        System.out.println(i.getAndIncrement());
        // 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
        System.out.println(i.incrementAndGet());
        // 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
        System.out.println(i.decrementAndGet());
        // 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
        System.out.println(i.getAndDecrement());
        // 获取并加值（i = 0, 结果 i = 5, 返回 0）
        System.out.println(i.getAndAdd(5));
        // 加值并获取（i = 5, 结果 i = 0, 返回 0）
        System.out.println(i.addAndGet(-5));
        // 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        //System.out.println(i.getAndUpdate(p -> p - 2));
        System.out.println(i.getAndUpdate(new LongUnaryOperator()
        {
            @Override
            public long applyAsLong(long operand)
            {
                return operand - 2;
            }
        }));
        // 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        //System.out.println(i.updateAndGet(p -> p + 2));
        System.out.println(i.updateAndGet(new LongUnaryOperator()
        {
            @Override
            public long applyAsLong(long operand)
            {
                return operand + 2;
            }
        }));
        // 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        // getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
        // getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
        //System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));
        System.out.println(i.getAndAccumulate(10, new LongBinaryOperator()
        {
            @Override
            public long applyAsLong(long left, long right)
            {
                return left + right;
            }
        }));
        // 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
        // 其中函数中的操作能保证原子，但函数需要无副作用
        //System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
        System.out.println(i.accumulateAndGet(-10, new LongBinaryOperator()
        {
            @Override
            public long applyAsLong(long left, long right)
            {
                return left + right;
            }
        }));
    }
}
```





### AtomicBoolean 



```java
package mao.t3;

import java.util.concurrent.atomic.AtomicBoolean;

/**
 * Project name(项目名称)：java并发编程_原子整数
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:32
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        AtomicBoolean atomicBoolean = new AtomicBoolean(true);
        
        System.out.println(atomicBoolean.getAndSet(false));
        System.out.println(atomicBoolean.get());
    }
}
```







## 原子引用

* AtomicReference 
* AtomicMarkableReference 
* AtomicStampedReference





### 取款问题

```java
package mao.t1;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t1
 * Interface(接口名): DecimalAccount
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:50
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public interface DecimalAccount
{
    /**
     * 获取余额
     *
     * @return {@link BigDecimal}
     */
    BigDecimal getBalance();

    /**
     * 取款
     *
     * @param amount BigDecimal
     */
    void withdraw(BigDecimal amount);

    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void start(DecimalAccount account)
    {
        List<Thread> threads = new ArrayList<>();
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000; i++)
        {
            threads.add(new Thread(() ->
            {
                account.withdraw(BigDecimal.TEN);
            }));
        }
        threads.forEach(Thread::start);
        threads.forEach(t ->
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
        long end = System.currentTimeMillis();
        System.out.println("剩余余额：" + account.getBalance());
        System.out.println("运行时间：" + (end - start) + "ms");
    }
}
```



#### 不安全实现

```java
package mao.t1;

import java.math.BigDecimal;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t1
 * Class(类名): DecimalAccountUnsafe
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:51
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class DecimalAccountUnsafe implements DecimalAccount
{

    private BigDecimal balance;

    public DecimalAccountUnsafe(BigDecimal balance)
    {
        this.balance = balance;
    }

    @Override
    public BigDecimal getBalance()
    {
        return balance;
    }

    @Override
    public void withdraw(BigDecimal amount)
    {
        BigDecimal balance = this.getBalance();
        this.balance = balance.subtract(amount);
    }
}
```





#### 使用锁

```java
package mao.t1;

import java.math.BigDecimal;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t1
 * Class(类名): DecimalAccountSafeLock
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:53
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class DecimalAccountSafeLock implements DecimalAccount
{
    /**
     * 锁
     */
    private final Object lock = new Object();
    private BigDecimal balance;

    public DecimalAccountSafeLock(BigDecimal balance)
    {
        this.balance = balance;
    }

    @Override
    public BigDecimal getBalance()
    {
        return balance;
    }

    @Override
    public void withdraw(BigDecimal amount)
    {
        synchronized (lock)
        {
            BigDecimal balance = this.getBalance();
            this.balance = balance.subtract(amount);
        }
    }

}
```



#### 使用 CAS



```java
package mao.t1;

import java.math.BigDecimal;
import java.util.concurrent.atomic.AtomicReference;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t1
 * Class(类名): DecimalAccountSafeCas
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 12:40
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class DecimalAccountSafeCas implements DecimalAccount
{
    AtomicReference<BigDecimal> ref;

    public DecimalAccountSafeCas(BigDecimal balance)
    {
        ref = new AtomicReference<>(balance);
    }

    @Override
    public BigDecimal getBalance()
    {
        return ref.get();
    }

    @Override
    public void withdraw(BigDecimal amount)
    {
        while (true)
        {
            BigDecimal prev = ref.get();
            BigDecimal next = prev.subtract(amount);
            if (ref.compareAndSet(prev, next))
            {
                break;
            }
        }
    }
}
```





#### 测试

```java
package mao.t1;

import java.math.BigDecimal;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 11:47
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    public static void main(String[] args)
    {
        DecimalAccount.start(new DecimalAccountUnsafe(new BigDecimal("10000")));
        DecimalAccount.start(new DecimalAccountSafeLock(new BigDecimal("10000")));
        DecimalAccount.start(new DecimalAccountSafeCas(new BigDecimal("10000")));
    }
}
```



运行结果：

```sh
剩余余额：690
运行时间：102ms
剩余余额：0
运行时间：90ms
剩余余额：0
运行时间：94ms
```

```sh
剩余余额：440
运行时间：101ms
剩余余额：0
运行时间：92ms
剩余余额：0
运行时间：95ms
```

```sh
剩余余额：170
运行时间：107ms
剩余余额：0
运行时间：100ms
剩余余额：0
运行时间：93ms
```







### ABA 问题

```java
package mao.t2;

import java.util.concurrent.atomic.AtomicReference;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 12:54
 * Version(版本): 1.0
 * Description(描述)： ABA 问题
 */

public class Test
{
    private static final AtomicReference<String> ref = new AtomicReference<>("A");

    /**
     * 睡眠
     *
     * @param time 时间
     */
    private static void sleep(long time)
    {
        try
        {
            Thread.sleep(time);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 改变
     */
    private static void change()
    {
        new Thread(() ->
        {
            System.out.println("t1线程A改B：" + ref.compareAndSet(ref.get(), "B"));
        }, "t1").start();
        sleep(500);
        new Thread(() ->
        {
            System.out.println("t2线程B改A：" + ref.compareAndSet(ref.get(), "A"));
        }, "t2").start();

    }

    public static void main(String[] args)
    {
        String prev = ref.get();
        change();
        sleep(1000);
        System.out.println("main线程A改为C：" + ref.compareAndSet(prev, "C"));
    }
}
```



运行结果：

```sh
t1线程A改B：true
t2线程B改A：true
main线程A改为C：true
```



主线程仅能判断出共享变量的值与最初值 A 是否相同，不能感知到这种从 A 改为 B 又 改回 A 的情况，如果主线程希望只要有其它线程**动过了**共享变量，那么自己的 cas 就算失败，这时，仅比较值是不够的，需要再加一个版本号，这就需要使用到**AtomicStampedReference**





### AtomicStampedReference

```java
package mao.t3;


import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 13:03
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

    /**
     * 睡眠
     *
     * @param time 时间
     */
    private static void sleep(long time)
    {
        try
        {
            Thread.sleep(time);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 改变
     */
    private static void change()
    {
        new Thread(() ->
        {
            System.out.println("t1线程A改B：" + ref.compareAndSet(ref.getReference(),
                    "B", ref.getStamp(), ref.getStamp() + 1));
        }, "t1").start();
        sleep(500);
        new Thread(() ->
        {
            System.out.println("t2线程B改A：" + ref.compareAndSet(ref.getReference(),
                    "A", ref.getStamp(), ref.getStamp() + 1));
        }, "t2").start();

    }

    public static void main(String[] args)
    {
        String prev = ref.getReference();
        //版本号
        int stamp = ref.getStamp();
        change();
        sleep(1000);
        System.out.println("main线程A改为C：" + ref.compareAndSet(prev, "C", stamp, stamp + 1));
    }
}
```



运行结果：

```sh
t1线程A改B：true
t2线程B改A：true
main线程A改为C：false
```



AtomicStampedReference 可以给原子引用加上版本号，追踪原子引用整个的变化过程，如： A -> B -> A -> C ，通过AtomicStampedReference，我们可以知道，引用变量中途被更改了几次





### AtomicMarkableReference

但是有时候，并不关心引用变量更改了几次，只是单纯的关心是否更改过，所以就有了 AtomicMarkableReference



```java
package mao.t4;

import java.util.concurrent.atomic.AtomicMarkableReference;

/**
 * Project name(项目名称)：java并发编程_原子引用
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 13:12
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final AtomicMarkableReference<String> ref = new AtomicMarkableReference<>("A", true);

    /**
     * 睡眠
     *
     * @param time 时间
     */
    private static void sleep(long time)
    {
        try
        {
            Thread.sleep(time);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * 改变
     */
    private static void change()
    {
        new Thread(() ->
        {
            System.out.println("t1线程A改B：" + ref.compareAndSet(ref.getReference(), "B", ref.isMarked(), false));
        }, "t1").start();
        sleep(500);
        new Thread(() ->
        {
            System.out.println("t2线程B改A：" + ref.compareAndSet(ref.getReference(), "A", ref.isMarked(), false));
        }, "t2").start();

    }

    public static void main(String[] args)
    {
        String prev = ref.getReference();
        boolean marked = ref.isMarked();
        change();
        sleep(1000);
        System.out.println("main线程A改为C：" + ref.compareAndSet(prev, "C", marked, false));
    }
}
```



运行结果：

```sh
t1线程A改B：true
t2线程B改A：true
main线程A改为C：false
```







## 原子数组

* AtomicIntegerArray 
* AtomicLongArray 
* AtomicReferenceArray



不安全的实现：

```java
package mao.t1;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_原子数组
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 15:11
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        long start = System.currentTimeMillis();
        int[] integers = new int[10];
        int total = 10000;
        List<Thread> threads = new ArrayList<>(total);
        for (int i = 0; i < total; i++)
        {
            Thread thread = new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    for (int k = 0; k < 10; k++)
                    {
                        for (int j = 0; j < integers.length; j++)
                        {
                            integers[j] = integers[j] + 1;
                        }
                    }
                }
            }, "t" + (i + 1));
            threads.add(thread);
        }

        threads.forEach(Thread::start);

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                long end = System.currentTimeMillis();
                System.out.println("运行时间：" + (end - start) + "ms");
                System.out.println(Arrays.toString(integers));
            }
        }));
    }
}
```



运行结果：

```sh
运行时间：913ms
[99995, 99994, 99994, 99994, 99994, 99993, 99993, 99994, 99992, 99993]
```

```sh
运行时间：921ms
[99990, 99990, 99990, 99990, 99990, 99990, 99990, 99990, 99990, 99990]
```

```sh
运行时间：963ms
[99987, 99987, 99988, 99987, 99985, 99985, 99985, 99985, 99987, 99987]
```





### AtomicIntegerArray 



```java
package mao.t2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.atomic.AtomicIntegerArray;

/**
 * Project name(项目名称)：java并发编程_原子数组
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 15:21
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        long start = System.currentTimeMillis();
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(10);
        int total = 10000;
        List<Thread> threads = new ArrayList<>(total);
        for (int i = 0; i < total; i++)
        {
            Thread thread = new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    for (int k = 0; k < 10; k++)
                    {
                        for (int j = 0; j < atomicIntegerArray.length(); j++)
                        {
                            atomicIntegerArray.incrementAndGet(j);
                        }
                    }
                }
            }, "t" + (i + 1));
            threads.add(thread);
        }

        threads.forEach(Thread::start);

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                long end = System.currentTimeMillis();
                System.out.println("运行时间：" + (end - start) + "ms");
                System.out.println(atomicIntegerArray);
            }
        }));
    }
}
```



运行结果：

```sh
运行时间：898ms
[100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000]
```

```sh
运行时间：955ms
[100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000]
```

```sh
运行时间：929ms
[100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000]
```





### AtomicLongArray 

```java
package mao.t3;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLongArray;

/**
 * Project name(项目名称)：java并发编程_原子数组
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 15:44
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        long start = System.currentTimeMillis();
        AtomicLongArray atomicLongArray = new AtomicLongArray(10);
        int total = 10000;
        List<Thread> threads = new ArrayList<>(total);
        for (int i = 0; i < total; i++)
        {
            Thread thread = new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    for (int k = 0; k < 10; k++)
                    {
                        for (int j = 0; j < atomicLongArray.length(); j++)
                        {
                            atomicLongArray.incrementAndGet(j);
                        }
                    }
                }
            }, "t" + (i + 1));
            threads.add(thread);
        }

        threads.forEach(Thread::start);

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                long end = System.currentTimeMillis();
                System.out.println("运行时间：" + (end - start) + "ms");
                System.out.println(atomicLongArray);
            }
        }));
    }
}
```



运行结果：

```sh
运行时间：917ms
[100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000]
```

```sh
运行时间：949ms
[100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000]
```

```sh
运行时间：910ms
[100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000]
```







## 字段更新器

* AtomicReferenceFieldUpdater 
* AtomicIntegerFieldUpdater
* AtomicLongFieldUpdater



利用字段更新器，可以针对对象的某个域（Field）进行原子操作，只能配合 volatile 修饰的字段使用，否则会出现异常





学生类：

```java
package mao;

/**
 * Project name(项目名称)：java并发编程_字段更新器
 * Package(包名): mao
 * Class(类名): Student
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 15:56
 * Version(版本): 1.0
 * Description(描述)： 字段不设置成public不然无法使用
 */

public class Student
{
    /**
     * id
     */
    public volatile long id;

    /**
     * 名字
     */
    public volatile String name;

    /**
     * 年龄
     */
    public volatile int age;

    public Student(long id, String name, int age)
    {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Student()
    {
    }

    public long getId()
    {
        return id;
    }

    public Student setId(long id)
    {
        this.id = id;
        return this;
    }

    public String getName()
    {
        return name;
    }

    public Student setName(String name)
    {
        this.name = name;
        return this;
    }

    public int getAge()
    {
        return age;
    }

    public Student setAge(int age)
    {
        this.age = age;
        return this;
    }

    @Override
    @SuppressWarnings("all")
    public String toString()
    {
        final StringBuilder stringbuilder = new StringBuilder();
        stringbuilder.append("id：").append(id).append('\n');
        stringbuilder.append("name：").append(name).append('\n');
        stringbuilder.append("age：").append(age).append('\n');
        return stringbuilder.toString();
    }
}
```





### AtomicIntegerFieldUpdater

```java
package mao.t1;

import mao.Student;

import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;

/**
 * Project name(项目名称)：java并发编程_字段更新器
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 15:59
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        Student student = new Student();
        student.setId(101L).setName("张三").setAge(18);

        AtomicIntegerFieldUpdater<Student> atomicIntegerFieldUpdater
                = AtomicIntegerFieldUpdater.newUpdater(Student.class, "age");

        System.out.println(atomicIntegerFieldUpdater.getAndSet(student, 21));
        int age = atomicIntegerFieldUpdater.get(student);
        System.out.println(age);
    }
}
```



运行结果：

```sh
18
21
```





### AtomicLongFieldUpdater



```java
package mao.t2;

import mao.Student;

import java.util.concurrent.atomic.AtomicLongFieldUpdater;

/**
 * Project name(项目名称)：java并发编程_字段更新器
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:11
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        Student student = new Student();
        student.setId(101L).setName("张三").setAge(18);

        AtomicLongFieldUpdater<Student> atomicLongFieldUpdater
                = AtomicLongFieldUpdater.newUpdater(Student.class, "id");

        long id = atomicLongFieldUpdater.getAndSet(student, 103);
        System.out.println(id);
        System.out.println(atomicLongFieldUpdater.get(student));
    }
}
```



运行结果：

```sh
101
103
```





### AtomicReferenceFieldUpdater 



```java
package mao.t3;

import mao.Student;

import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;

/**
 * Project name(项目名称)：java并发编程_字段更新器
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:13
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        Student student = new Student();
        student.setId(101L).setName("张三").setAge(18);

        AtomicReferenceFieldUpdater<Student, String> atomicReferenceFieldUpdater
                = AtomicReferenceFieldUpdater.newUpdater(Student.class, String.class, "name");

        String name = atomicReferenceFieldUpdater.getAndSet(student, "李四");
        System.out.println(name);
        System.out.println(atomicReferenceFieldUpdater.get(student));
    }
}
```



运行结果：

```sh
张三
李四
```









## 原子累加器



### 基本使用

```java
package mao.t1;

import java.util.concurrent.atomic.LongAdder;

/**
 * Project name(项目名称)：java并发编程_原子累加器
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:22
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args)
    {
        LongAdder longAdder = new LongAdder();
        for (int i = 0; i < 100; i++)
        {
            longAdder.increment();
        }
        System.out.println(longAdder.longValue());
    }
}
```



```sh
100
```





### 累加器性能比较



```java
package mao.t2;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAdder;
import java.util.function.Consumer;
import java.util.function.Supplier;

/**
 * Project name(项目名称)：java并发编程_原子累加器
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:27
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static <T> void demo(Supplier<T> adderSupplier, Consumer<T> action)
    {
        T adder = adderSupplier.get();
        long start = System.nanoTime();
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 50; i++)
        {
            threads.add(new Thread(() ->
            {
                for (int j = 0; j < 1000000; j++)
                {
                    action.accept(adder);
                }
            }));
        }
        threads.forEach(Thread::start);
        threads.forEach(t ->
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
        long end = System.nanoTime();
        System.out.println(adder + " 花费的时间:" + (end - start) / 1000_000 + "ms");
    }

    public static void main(String[] args)
    {
        int total = 10;
        System.out.println("AtomicLong");
        for (int i = 0; i < total; i++)
        {
            demo(AtomicLong::new, AtomicLong::getAndIncrement);
        }
        System.out.println("---------------");
        System.out.println("LongAdder");
        for (int i = 0; i < total; i++)
        {
            demo(LongAdder::new, LongAdder::increment);
        }

    }

}
```



运行结果：

```sh
AtomicLong
50000000 花费的时间:802ms
50000000 花费的时间:795ms
50000000 花费的时间:780ms
50000000 花费的时间:784ms
50000000 花费的时间:783ms
50000000 花费的时间:792ms
50000000 花费的时间:762ms
50000000 花费的时间:785ms
50000000 花费的时间:783ms
50000000 花费的时间:778ms
---------------
LongAdder
50000000 花费的时间:56ms
50000000 花费的时间:35ms
50000000 花费的时间:32ms
50000000 花费的时间:36ms
50000000 花费的时间:31ms
50000000 花费的时间:34ms
50000000 花费的时间:34ms
50000000 花费的时间:28ms
50000000 花费的时间:29ms
50000000 花费的时间:29ms
```

```sh
AtomicLong
50000000 花费的时间:799ms
50000000 花费的时间:788ms
50000000 花费的时间:756ms
50000000 花费的时间:781ms
50000000 花费的时间:775ms
50000000 花费的时间:779ms
50000000 花费的时间:775ms
50000000 花费的时间:770ms
50000000 花费的时间:781ms
50000000 花费的时间:774ms
---------------
LongAdder
50000000 花费的时间:51ms
50000000 花费的时间:31ms
50000000 花费的时间:29ms
50000000 花费的时间:50ms
50000000 花费的时间:34ms
50000000 花费的时间:32ms
50000000 花费的时间:32ms
50000000 花费的时间:31ms
50000000 花费的时间:32ms
50000000 花费的时间:30ms
```

```sh
AtomicLong
50000000 花费的时间:795ms
50000000 花费的时间:809ms
50000000 花费的时间:765ms
50000000 花费的时间:784ms
50000000 花费的时间:792ms
50000000 花费的时间:785ms
50000000 花费的时间:789ms
50000000 花费的时间:789ms
50000000 花费的时间:776ms
50000000 花费的时间:770ms
---------------
LongAdder
50000000 花费的时间:43ms
50000000 花费的时间:28ms
50000000 花费的时间:31ms
50000000 花费的时间:37ms
50000000 花费的时间:27ms
50000000 花费的时间:27ms
50000000 花费的时间:28ms
50000000 花费的时间:27ms
50000000 花费的时间:29ms
50000000 花费的时间:28ms
```



性能提升的原因很简单，就是在有竞争时，设置多个累加单元，Therad-0 累加 Cell[0]，而 Thread-1 累加 Cell[1]... 最后将结果汇总。这样它们在累加时操作的不同的 Cell 变量，因此减少了 CAS 重试失败，从而提高性能





### cas 锁



```java
package mao.t3;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * Project name(项目名称)：java并发编程_原子累加器
 * Package(包名): mao.t3
 * Class(类名): LockCas
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:40
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class LockCas
{
    private final AtomicInteger state = new AtomicInteger(0);

    public void lock()
    {
        while (true)
        {
            if (state.compareAndSet(0, 1))
            {
                break;
            }
        }
        System.out.println("lock");
    }

    public void unlock()
    {
        System.out.println("unlock");
        state.set(0);
    }

}
```



```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_原子累加器
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args) throws InterruptedException
    {
        LockCas lockCas = new LockCas();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                lockCas.lock();
                try
                {
                    Thread.sleep(5000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    System.out.println("t1线程释放锁");
                    lockCas.unlock();
                }
            }
        }, "t1").start();

        Thread.sleep(100);
        System.out.println("主线程尝试获取锁");
        lockCas.lock();
        System.out.println("主线程获取锁成功");
    }
}
```



运行结果：

```sh
lock
主线程尝试获取锁
t1线程释放锁
unlock
lock
主线程获取锁成功
```





注意：**不能用于实践，因为获取锁失败的时候不是进入阻塞，而是一直使用CPU做无用功**



```java
package mao.t3;

/**
 * Project name(项目名称)：java并发编程_原子累加器
 * Package(包名): mao.t3
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 16:46
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    public static void main(String[] args)
    {
        for (int i = 0; i < 15; i++)
        {
            new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    try
                    {
                        Test.main(null);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}
```



![image-20220907165256696](img/java并发编程学习笔记/image-20220907165256696.png)











## Unsafe

Unsafe 对象提供了非常底层的，操作内存、线程的方法，Unsafe 对象不能直接调用，只能通过反射获得



![image-20220907170901374](img/java并发编程学习笔记/image-20220907170901374.png)







```java
package mao.t1;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

/**
 * Project name(项目名称)：java并发编程_Unsafe
 * Package(包名): mao.t1
 * Class(类名): UnsafeAccessor
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 17:05
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class UnsafeAccessor
{
    private static final Unsafe unsafe;

    static
    {
        try
        {
            Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafe.setAccessible(true);
            unsafe = (Unsafe) theUnsafe.get(null);
        }
        catch (NoSuchFieldException | IllegalAccessException e)
        {
            throw new Error(e);
        }
    }

    static Unsafe getUnsafe()
    {
        return unsafe;
    }
}
```



```java
package mao.t1;

/**
 * Project name(项目名称)：java并发编程_Unsafe
 * Package(包名): mao.t1
 * Class(类名): Student
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 17:09
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Student
{
    /**
     * id
     */
    public volatile int id;
    /**
     * 名字
     */
    public volatile String name;

    @Override
    @SuppressWarnings("all")
    public String toString()
    {
        final StringBuilder stringbuilder = new StringBuilder();
        stringbuilder.append("id：").append(id).append('\n');
        stringbuilder.append("name：").append(name).append('\n');
        return stringbuilder.toString();
    }
}
```



```java
package mao.t1;


import sun.misc.Unsafe;

import java.lang.reflect.Field;

/**
 * Project name(项目名称)：java并发编程_Unsafe
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/7
 * Time(创建时间)： 17:04
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args) throws NoSuchFieldException
    {
        Unsafe unsafe = UnsafeAccessor.getUnsafe();
        System.out.println(unsafe);

        Field id = Student.class.getDeclaredField("id");
        Field name = Student.class.getDeclaredField("name");
        // 获得成员变量的偏移量
        long idOffset = unsafe.objectFieldOffset(id);
        long nameOffset = unsafe.objectFieldOffset(name);

        System.out.println(idOffset);
        System.out.println(nameOffset);

        Student student = new Student();
        System.out.println(student);

        // 使用 cas 方法替换成员变量的值
        unsafe.compareAndSwapInt(student, idOffset, 0, 18);
        unsafe.compareAndSwapObject(student, nameOffset, null, "张三");

        System.out.println(student);
    }
}
```



运行结果：

```sh
sun.misc.Unsafe@682a0b20
12
16
id：0
name：null

id：18
name：张三
```



















# 共享模型之不可变

## 日期转换的问题

SimpleDateFormat 不是线程安全的



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.text.SimpleDateFormat;

/**
 * Project name(项目名称)：java并发编程_不可变
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 12:56
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        for (int i = 0; i < 20; i++)
        {
            new Thread(() ->
            {
                try
                {
                    log.debug(simpleDateFormat.parse("2022-07-30").toString());
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
            }).start();
        }

    }
}
```



运行结果：

```sh
java.lang.NumberFormatException: For input string: ""
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Long.parseLong(Long.java:724)
	at java.base/java.lang.Long.parseLong(Long.java:839)
	at java.base/java.text.DigitList.getLong(DigitList.java:195)
	at java.base/java.text.DecimalFormat.parse(DecimalFormat.java:2197)
	at java.base/java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1934)
	at java.base/java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1542)
	at java.base/java.text.DateFormat.parse(DateFormat.java:394)
	at mao.t1.Test.lambda$main$0(Test.java:34)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.NumberFormatException: For input string: ".330E"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Long.parseLong(Long.java:700)
	at java.base/java.lang.Long.parseLong(Long.java:839)
	at java.base/java.text.DigitList.getLong(DigitList.java:195)
	at java.base/java.text.DecimalFormat.parse(DecimalFormat.java:2197)
	at java.base/java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:2241)
	at java.base/java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1542)
	at java.base/java.text.DateFormat.parse(DateFormat.java:394)
	at mao.t1.Test.lambda$main$0(Test.java:34)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.NumberFormatException: For input string: ""
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Long.parseLong(Long.java:724)
	at java.base/java.lang.Long.parseLong(Long.java:839)
	at java.base/java.text.DigitList.getLong(DigitList.java:195)
	at java.base/java.text.DecimalFormat.parse(DecimalFormat.java:2197)
	at java.base/java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1934)
	at java.base/java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1542)
	at java.base/java.text.DateFormat.parse(DateFormat.java:394)
	at mao.t1.Test.lambda$main$0(Test.java:34)
	at java.base/java.lang.Thread.run(Thread.java:831)
2022-09-08  13:02:37.265  [Thread-17] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-18] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-5] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-16] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-13] DEBUG mao.t1.Test:  Wed Jul 30 00:00:00 CST 77
2022-09-08  13:02:37.265  [Thread-8] DEBUG mao.t1.Test:  Thu Jun 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-10] DEBUG mao.t1.Test:  Tue Aug 02 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-6] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-14] DEBUG mao.t1.Test:  Wed Jul 30 00:00:00 CST 77
2022-09-08  13:02:37.265  [Thread-7] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-4] DEBUG mao.t1.Test:  Sun Jul 30 00:00:00 CST 2220
2022-09-08  13:02:37.265  [Thread-12] DEBUG mao.t1.Test:  Tue May 30 00:00:00 CST 2028
2022-09-08  13:02:37.265  [Thread-0] DEBUG mao.t1.Test:  Thu Jun 30 00:00:00 CST 2472
2022-09-08  13:02:37.265  [Thread-2] DEBUG mao.t1.Test:  Sun Jul 30 00:00:00 CST 2220
2022-09-08  13:02:37.265  [Thread-11] DEBUG mao.t1.Test:  Wed Jul 30 00:00:00 CST 77
2022-09-08  13:02:37.265  [Thread-19] DEBUG mao.t1.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:02:37.265  [Thread-3] DEBUG mao.t1.Test:  Sun Jul 30 00:00:00 CST 2220
```



有很大几率出现 java.lang.NumberFormatException 或者出现不正确的日期解析结果



可以使用同步锁，这样虽能解决问题，但带来的是性能上的损失，并不算很好



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.text.SimpleDateFormat;

/**
 * Project name(项目名称)：java并发编程_不可变
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:04
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");

        for (int i = 0; i < 20; i++)
        {
            new Thread(() ->
            {
                synchronized (simpleDateFormat)
                {
                    try
                    {
                        log.debug(simpleDateFormat.parse("2022-07-30").toString());
                    }
                    catch (Exception e)
                    {
                        e.printStackTrace();
                    }
                }
            }).start();
        }

    }
}
```



运行结果：

```sh
2022-09-08  13:05:16.804  [Thread-0] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.806  [Thread-19] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.806  [Thread-18] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.807  [Thread-17] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.807  [Thread-16] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.808  [Thread-15] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.808  [Thread-14] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.808  [Thread-13] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.808  [Thread-12] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.809  [Thread-11] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.809  [Thread-10] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.809  [Thread-9] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.809  [Thread-8] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.810  [Thread-7] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.810  [Thread-6] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.810  [Thread-5] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.810  [Thread-4] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.811  [Thread-3] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.811  [Thread-2] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
2022-09-08  13:05:16.811  [Thread-1] DEBUG mao.t2.Test:  Sat Jul 30 00:00:00 CST 2022
```





不可变设计

在 Java 8 后，提供了一个新的日期格式化类



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.text.SimpleDateFormat;
import java.time.format.DateTimeFormatter;

/**
 * Project name(项目名称)：java并发编程_不可变
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:07
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        for (int i = 0; i < 20; i++)
        {
            new Thread(() ->
            {
                try
                {
                    log.debug(dateTimeFormatter.parse("2022-07-30").toString());
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
            }).start();
        }

    }
}
```



运行结果：

```sh
2022-09-08  13:10:00.812  [Thread-9] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-1] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-3] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-19] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-18] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-16] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-13] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-8] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-12] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-15] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-2] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-11] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-4] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-14] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-7] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-6] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-0] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-10] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-5] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
2022-09-08  13:10:00.812  [Thread-17] DEBUG mao.t3.Test:  {},ISO resolved to 2022-07-30
```





## 不可变设计

 String 类也是不可变的



```java
public final class String
 implements java.io.Serializable, Comparable<String>, CharSequence {
 /** The value is used for character storage. */
 private final char value[];
 /** Cache the hash code for the string */
 private int hash; // Default to 0
 
 // ...
 
}
```



## 保护性拷贝

```java
public String substring(int beginIndex) {
 if (beginIndex < 0) {
 throw new StringIndexOutOfBoundsException(beginIndex);
 }
 int subLen = value.length - beginIndex;
 if (subLen < 0) {
 throw new StringIndexOutOfBoundsException(subLen);
 }
 return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```



发现其内部是调用 String 的构造方法创建了一个新字符串，再进入这个构造看看，是否对 final char[] value 做出了修改



```java
public String(char value[], int offset, int count) {
 if (offset < 0) {
 throw new StringIndexOutOfBoundsException(offset);
 }
 if (count <= 0) {
 if (count < 0) {
 throw new StringIndexOutOfBoundsException(count);
 }
 if (offset <= value.length) {
 this.value = "".value;
 return;
 }
 }
 if (offset > value.length - count) {
 throw new StringIndexOutOfBoundsException(offset + count);
 }
 this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```



结果发现也没有，构造新字符串对象时，会生成新的 char[] value，对内容进行复制 。这种通过创建副本对象来避 免共享的手段称之为**保护性拷贝**





## 享元模式

### 包装类

在JDK中 Boolean，Byte，Short，Integer，Long，Character 等包装类提供了 valueOf 方法，例如 Long 的 valueOf 会缓存 -128~127 之间的 Long 对象，在这个范围之间会重用对象，大于这个范围，才会新建 Long 对 象

```java
public static Long valueOf(long l) {
 final int offset = 128;
 if (l >= -128 && l <= 127) { // will cache
 return LongCache.cache[(int)l + offset];
 }
 return new Long(l);
}
```



* Byte, Short, Long 缓存的范围都是 -128~127
* Character 缓存的范围是 0~127
* Integer的默认范围是 -128~127
* Boolean 缓存了 TRUE 和 FALSE





### 自定义简单连接池



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.Connection;
import java.util.concurrent.atomic.AtomicIntegerArray;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t1
 * Class(类名): Pool
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:25
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Pool
{
    /**
     * 池大小
     */
    private final int poolSize;

    /**
     * 连接池
     */
    private final Connection[] connections;

    private final AtomicIntegerArray states;

    private static final Logger log = LoggerFactory.getLogger(Pool.class);

    public Pool(int poolSize)
    {
        this.poolSize = poolSize;
        this.connections = new Connection[poolSize];
        this.states = new AtomicIntegerArray(new int[poolSize]);
        for (int i = 0; i < poolSize; i++)
        {
            connections[i] = new MockConnection("连接" + (i + 1));
        }
    }

    /**
     * 借连接
     *
     * @return {@link Connection}
     */
    public Connection borrow()
    {
        while (true)
        {
            for (int i = 0; i < poolSize; i++)
            {
                // 获取空闲连接
                if (states.get(i) == 0)
                {
                    if (states.compareAndSet(i, 0, 1))
                    {
                        log.debug("borrow {}", connections[i]);
                        return connections[i];
                    }
                }
            }
            // 如果没有空闲连接，当前线程进入等待
            synchronized (this)
            {
                try
                {
                    log.debug("wait...");
                    this.wait();
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        }


    }

    /**
     * 归还连接
     *
     * @param conn Connection
     */
    public void free(Connection conn)
    {
        for (int i = 0; i < poolSize; i++)
        {
            if (connections[i] == conn)
            {
                states.set(i, 0);
                synchronized (this)
                {
                    log.debug("free {}", conn);
                    this.notifyAll();
                }
                break;
            }
        }
    }



}

class MockConnection implements Connection {
    // 实现略
}
```







### 自定义连接池



```java
package mao.t2;

import javax.sql.DataSource;
import java.awt.*;
import java.io.PrintWriter;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.logging.Logger;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t2
 * Class(类名): ConnectionPool
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:39
 * Version(版本): 1.0
 * Description(描述)： 无
 */


public class ConnectionPool implements DataSource
{
    //连接池对象的数量
    private static final int ConnectionPoolSize = 10;

    //连接池，线程安全
    private static final List<Connection> pool = Collections.synchronizedList(new ArrayList<>());

    static
    {
        try
        {
            for (int i = 0; i < ConnectionPoolSize; i++)
            {
                Connection connection = JDBC.getConnection();
                pool.add(connection);
            }
        }
        catch (Exception e)
        {
            System.err.println("连接池初始化失败！");
            Toolkit.getDefaultToolkit().beep();
            e.printStackTrace();
        }
    }

    /**
     * 返回连接池剩余的连接数量
     *
     * @return int型 连接池剩余的连接数量
     */
    public int getPoolSize()
    {
        return pool.size();
    }

    /**
     * 获取连接池最大连接的数量，此数量固定
     *
     * @return int型 返回最大连接的数量
     */
    public static int getConnectionPoolSize()
    {
        return ConnectionPoolSize;
    }


    @SuppressWarnings("all")
    /**
     * 获得连接 动态代理方式
     *
     * @return 加强的连接对象，此对象调用close方法归还连接，而不是关闭连接
     * @throws SQLException 数据库异常
     */
    /*
    @Override
    public Connection getConnection() throws SQLException
    {
        if (pool.size() > 0)
        {
            Connection connection = pool.remove(0);
            @SuppressWarnings("all")
            Connection proxy = (Connection) Proxy.newProxyInstance(connection.getClass().getClassLoader(), new Class[]{Connection.class}, new InvocationHandler()
            {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
                {
                    if (method.getName().equals("close"))
                    {
                        //归还连接
                        connection.setAutoCommit(true);     //重新将连接设置为自动提交
                        pool.add(connection);               //归还连接
                        return null;
                    }
                    else
                    {
                        return method.invoke(connection, args);
                    }
                }
            });
            return proxy;
        }
        else
        {
            throw new RuntimeException("连接池连接数量已经用尽！");
        }
    }
     */

    /**
     * 获得连接 装饰设计模式或者适配器设计模式
     *
     * @return 加强的连接对象，此对象调用close方法归还连接，而不是关闭连接
     * @throws SQLException 数据库异常
     */
    @Override
    public mao.t2.Connection getConnection() throws SQLException
    {
        if (pool.size() > 0)
        {
            Connection connection = pool.remove(0);
            @SuppressWarnings("all")
            mao.t2.Connection con = new mao.t2.Connection(connection, pool);
            return con;
        }
        else
        {
            throw new RuntimeException("连接池连接数量已经用尽！");
        }
    }


    @Override
    public Connection getConnection(String username, String password) throws SQLException
    {
        return null;
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException
    {
        return null;
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException
    {

    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException
    {

    }

    @Override
    public int getLoginTimeout() throws SQLException
    {
        return 0;
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException
    {
        return null;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException
    {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException
    {
        return false;
    }
}
```





```java
package mao.t2;

import java.sql.*;
import java.sql.Connection;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.Executor;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t2
 * Class(类名): ConnectionAdapter
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */



public abstract class ConnectionAdapter implements Connection
{
    //连接对象
    private final Connection connection;

    public ConnectionAdapter(Connection connection)
    {
        this.connection = connection;
    }

    @Override
    public Statement createStatement() throws SQLException
    {
        return connection.createStatement();
    }

    @Override
    public PreparedStatement prepareStatement(String sql) throws SQLException
    {
        return connection.prepareStatement(sql);
    }

    @Override
    public CallableStatement prepareCall(String sql) throws SQLException
    {
        return connection.prepareCall(sql);
    }

    @Override
    public String nativeSQL(String sql) throws SQLException
    {
        return connection.nativeSQL(sql);
    }

    @Override
    public void setAutoCommit(boolean autoCommit) throws SQLException
    {
        connection.setAutoCommit(autoCommit);
    }

    @Override
    public boolean getAutoCommit() throws SQLException
    {
        return connection.getAutoCommit();
    }

    @Override
    public void commit() throws SQLException
    {
        connection.commit();
    }

    @Override
    public void rollback() throws SQLException
    {
        connection.rollback();
    }


    @Override
    public boolean isClosed() throws SQLException
    {
        return connection.isClosed();
    }

    @Override
    public DatabaseMetaData getMetaData() throws SQLException
    {
        return connection.getMetaData();
    }

    @Override
    public void setReadOnly(boolean readOnly) throws SQLException
    {
        connection.setReadOnly(readOnly);
    }

    @Override
    public boolean isReadOnly() throws SQLException
    {
        return connection.isReadOnly();
    }

    @Override
    public void setCatalog(String catalog) throws SQLException
    {
        connection.setCatalog(catalog);
    }

    @Override
    public String getCatalog() throws SQLException
    {
        return connection.getCatalog();
    }

    @Override
    public void setTransactionIsolation(int level) throws SQLException
    {
        connection.setTransactionIsolation(level);
    }

    @Override
    public int getTransactionIsolation() throws SQLException
    {
        return connection.getTransactionIsolation();
    }

    @Override
    public SQLWarning getWarnings() throws SQLException
    {
        return connection.getWarnings();
    }

    @Override
    public void clearWarnings() throws SQLException
    {
        connection.clearWarnings();
    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency) throws SQLException
    {
        return connection.createStatement(resultSetType, resultSetConcurrency);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency)
            throws SQLException
    {
        return connection.prepareStatement(sql, resultSetType, resultSetConcurrency);
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency) throws SQLException
    {
        return connection.prepareCall(sql, resultSetType, resultSetConcurrency);
    }

    @Override
    public Map<String, Class<?>> getTypeMap() throws SQLException
    {
        return connection.getTypeMap();
    }

    @Override
    public void setTypeMap(Map<String, Class<?>> map) throws SQLException
    {
        connection.setTypeMap(map);
    }

    @Override
    public void setHoldability(int holdability) throws SQLException
    {
        connection.setHoldability(holdability);
    }

    @Override
    public int getHoldability() throws SQLException
    {
        return connection.getHoldability();
    }

    @Override
    public Savepoint setSavepoint() throws SQLException
    {
        return connection.setSavepoint();
    }

    @Override
    public Savepoint setSavepoint(String name) throws SQLException
    {
        return connection.setSavepoint(name);
    }

    @Override
    public void rollback(Savepoint savepoint) throws SQLException
    {
        connection.rollback();
    }

    @Override
    public void releaseSavepoint(Savepoint savepoint) throws SQLException
    {
        connection.releaseSavepoint(savepoint);
    }

    @Override
    public Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability)
            throws SQLException
    {
        return connection.createStatement(resultSetType, resultSetConcurrency, resultSetConcurrency);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability)
            throws SQLException
    {
        return connection.prepareStatement(sql, resultSetType, resultSetConcurrency, resultSetHoldability);
    }

    @Override
    public CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability)
            throws SQLException
    {
        return connection.prepareCall(sql, resultSetType, resultSetConcurrency, resultSetHoldability);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int autoGeneratedKeys) throws SQLException
    {
        return connection.prepareStatement(sql, autoGeneratedKeys);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, int[] columnIndexes) throws SQLException
    {
        return connection.prepareStatement(sql, columnIndexes);
    }

    @Override
    public PreparedStatement prepareStatement(String sql, String[] columnNames) throws SQLException
    {
        return connection.prepareStatement(sql, columnNames);
    }

    @Override
    public Clob createClob() throws SQLException
    {
        return connection.createClob();
    }

    @Override
    public Blob createBlob() throws SQLException
    {
        return connection.createBlob();
    }

    @Override
    public NClob createNClob() throws SQLException
    {
        return connection.createNClob();
    }

    @Override
    public SQLXML createSQLXML() throws SQLException
    {
        return connection.createSQLXML();
    }

    @Override
    public boolean isValid(int timeout) throws SQLException
    {
        return connection.isValid(timeout);
    }

    @Override
    public void setClientInfo(String name, String value) throws SQLClientInfoException
    {
        connection.setClientInfo(name, value);
    }

    @Override
    public void setClientInfo(Properties properties) throws SQLClientInfoException
    {
        connection.setClientInfo(properties);
    }

    @Override
    public String getClientInfo(String name) throws SQLException
    {
        return connection.getClientInfo(name);
    }

    @Override
    public Properties getClientInfo() throws SQLException
    {
        return connection.getClientInfo();
    }

    @Override
    public Array createArrayOf(String typeName, Object[] elements) throws SQLException
    {
        return connection.createArrayOf(typeName, elements);
    }

    @Override
    public Struct createStruct(String typeName, Object[] attributes) throws SQLException
    {
        return connection.createStruct(typeName, attributes);
    }

    @Override
    public void setSchema(String schema) throws SQLException
    {
        connection.setSchema(schema);
    }

    @Override
    public String getSchema() throws SQLException
    {
        return connection.getSchema();
    }

    @Override
    public void abort(Executor executor) throws SQLException
    {
        connection.abort(executor);
    }

    @Override
    public void setNetworkTimeout(Executor executor, int milliseconds) throws SQLException
    {
        connection.setNetworkTimeout(executor, milliseconds);
    }

    @Override
    public int getNetworkTimeout() throws SQLException
    {
        return connection.getNetworkTimeout();
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException
    {
        return connection.unwrap(iface);
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException
    {
        return connection.isWrapperFor(iface);
    }
}
```





```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.SQLException;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t2
 * Class(类名): Connection
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:37
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Connection extends ConnectionAdapter
{
    //连接对象
    private final java.sql.Connection connection;
    //连接池
    private final List<java.sql.Connection> pool;

    private static final Logger log = LoggerFactory.getLogger(Connection.class);

    /**
     * 构造方法对成员变量赋值
     *
     * @param connection 连接对象
     * @param pool       连接池
     */
    public Connection(java.sql.Connection connection, List<java.sql.Connection> pool)
    {
        super(connection);
        this.connection = connection;
        this.pool = pool;
    }

    //重写close方法

    /**
     * 重写close方法 完成归还连接操作
     *
     * @throws SQLException 抛出异常
     */
    @Override
    public void close() throws SQLException
    {
        connection.setAutoCommit(true);     //重新将连接设置为自动提交
        pool.add(connection);               //归还连接
        log.debug("归还连接：" + connection);
    }
}
```



```java
package mao.t2;

import java.awt.*;
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t2
 * Class(类名): JDBC
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:39
 * Version(版本): 1.0
 * Description(描述)： 无
 */


public class JDBC
{

    /*
    config.properties文件内容：

    driverClass=com.mysql.cj.jdbc.Driver
    url=jdbc:mysql://localhost:3306/student
    username=root
    password=20010713

     */

    @SuppressWarnings("all")
    private static String driverClass;              //驱动程序类
    private static String url;                      //url
    private static String username;                 //用户名
    private static String password;                 //密码

    private JDBC()
    {

    }

    static
    {
        InputStream inputStream;
        Properties properties;
        try
        {
            //src目录下,maven 资源路径下
            inputStream = JDBC.class.getClassLoader().getResourceAsStream("config.properties");
            properties = new Properties();
            properties.load(inputStream);
            driverClass = properties.getProperty("driverClass");      //读取驱动程序类
            url = properties.getProperty("url");                       //读取url
            username = properties.getProperty("username");           //读取用户名
            password = properties.getProperty("password");          //读取密码

            //System.out.println(driverClass);
            //System.out.println(url);
            //System.out.println(username);
            //System.out.println(password);

            //注册驱动
            Class.forName(driverClass);                             //新版
            //旧版：com.mysql.jdbc.Driver
        }
        catch (IOException e)
        {
            Toolkit.getDefaultToolkit().beep();
            e.printStackTrace();
        }
        catch (ClassNotFoundException e)
        {
            Toolkit.getDefaultToolkit().beep();
            System.err.println("驱动程序类加载失败!");
            System.err.println(e.getMessage());
        }
    }

    //连接方法

    /**
     * 获取数据库连接对象
     *
     * @return Connection对象
     * @throws Exception 获取对象失败时抛出
     */
    public static java.sql.Connection getConnection() throws Exception
    {
        java.sql.Connection connection = null;

        //Loading class `com.mysql.jdbc.Driver'. This is deprecated.
        // The new driver class is `com.mysql.cj.jdbc.Driver'.
        // The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
        connection = DriverManager.getConnection(url, username, password);
        return connection;
    }

    //关闭方法

    /**
     * 关闭数据库连接  3个对象，用于关闭查询的连接 123
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     * @param resultSet  结果集对象
     */
    public static void close(Connection connection, Statement statement, ResultSet resultSet)
    {
        if (resultSet != null)
        {
            try
            {
                resultSet.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
        if (statement != null)
        {
            try
            {
                statement.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
        if (connection != null)
        {
            try
            {
                connection.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭查询的连接 132
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     * @param resultSet  结果集对象
     */
    public static void close(Connection connection, ResultSet resultSet, Statement statement)
    {
        close(connection, statement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭查询的连接 213
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     * @param resultSet  结果集对象
     */
    public static void close(Statement statement, Connection connection, ResultSet resultSet)
    {
        close(connection, statement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭查询的连接 231
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     * @param resultSet  结果集对象
     */
    public static void close(Statement statement, ResultSet resultSet, Connection connection)
    {
        close(connection, statement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭查询的连接 312
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     * @param resultSet  结果集对象
     */
    public static void close(ResultSet resultSet, Connection connection, Statement statement)
    {
        close(connection, statement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭查询的连接 321
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     * @param resultSet  结果集对象
     */
    public static void close(ResultSet resultSet, Statement statement, Connection connection)
    {
        close(connection, statement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭预编译的查询的连接 123
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     * @param resultSet         结果集对象
     */
    public static void close(Connection connection, PreparedStatement preparedStatement, ResultSet resultSet)
    {
        if (resultSet != null)
        {
            try
            {
                resultSet.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
        if (preparedStatement != null)
        {
            try
            {
                preparedStatement.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
        if (connection != null)
        {
            try
            {
                connection.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭预编译的查询的连接 132
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     * @param resultSet         结果集对象
     */
    public static void close(Connection connection, ResultSet resultSet, PreparedStatement preparedStatement)
    {
        close(connection, preparedStatement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭预编译的查询的连接 213
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     * @param resultSet         结果集对象
     */
    public static void close(PreparedStatement preparedStatement, Connection connection, ResultSet resultSet)
    {
        close(connection, preparedStatement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭预编译的查询的连接 231
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     * @param resultSet         结果集对象
     */
    public static void close(PreparedStatement preparedStatement, ResultSet resultSet, Connection connection)
    {
        close(connection, preparedStatement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭预编译的查询的连接 312
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     * @param resultSet         结果集对象
     */
    public static void close(ResultSet resultSet, Connection connection, PreparedStatement preparedStatement)
    {
        close(connection, preparedStatement, resultSet);
    }

    /**
     * 关闭数据库连接  3个对象，用于关闭预编译的查询的连接 321
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     * @param resultSet         结果集对象
     */
    public static void close(ResultSet resultSet, PreparedStatement preparedStatement, Connection connection)
    {
        close(connection, preparedStatement, resultSet);
    }


    /**
     * 关闭数据库连接  2个对象，用于关闭更新的连接 12
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     */
    public static void close(Connection connection, Statement statement)
    {
        if (statement != null)
        {
            try
            {
                statement.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
        if (connection != null)
        {
            try
            {
                connection.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭数据库连接  2个对象，用于关闭更新的连接 21
     *
     * @param connection 数据库连接对象
     * @param statement  Statement对象
     */
    public static void close(Statement statement, Connection connection)
    {
        close(connection, statement);
    }

    /**
     * 关闭数据库连接  2个对象，用于关闭预编译的更新的连接 12
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     */
    public static void close(Connection connection, PreparedStatement preparedStatement)
    {
        if (preparedStatement != null)
        {
            try
            {
                preparedStatement.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
        if (connection != null)
        {
            try
            {
                connection.close();
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭数据库连接  2个对象，用于关闭预编译的更新的连接 21
     *
     * @param connection        数据库连接对象
     * @param preparedStatement PreparedStatement对象
     */
    public static void close(PreparedStatement preparedStatement, Connection connection)
    {
        close(connection, preparedStatement);
    }
}
```



```java
package mao.t2;


import java.sql.*;
import java.sql.Connection;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t2
 * Class(类名): JDBCTemplate2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:03
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class JDBCTemplate2
{
    private static final ConnectionPool connectionPool = new ConnectionPool();

    /**
     * 查询方法，此方法用于将一条或者多条记录封装成一个list集合并返回，适用于多表查询
     * 此方法不推荐使用，因为在map集合中，key是列名，重复。但是集合重写了toString方法，可以直接输出字符串。
     *
     * @param sql  预编译的sql查询语句，经常使用多表查询，sql语句实例：查询某号学生选修的课程名称和对应课程的分数：
     *             SELECT course.`name`, grade.grade
     *             FROM grade, course WHERE grade.cno = course.cno AND grade.`no`=?
     * @param objs sql中的问号占位符，数量和顺序一定要一致，在上一个例子中，问号的数量为1，数组个数为一个。
     *             例如，查询2号学生的学生选修的课程名称和对应课程的分数：可变数组的值为一个，值为2，
     * @return 返回一个ArrayList集合，list集合的泛型为HashMap集合，map集合中的键为列名，值为此行的列名对应的值
     * 此方法不推荐使用，因为在map集合中，key是列名，重复。
     */
    public static List<HashMap<String, Object>> queryForListMap(String sql, Object... objs)
    {
        //定义一个List集合
        List<HashMap<String, Object>> list = new ArrayList<>();
        //连接对象
        Connection connection = null;
        //预编译执行者对象
        PreparedStatement preparedStatement = null;
        //结果集对象
        ResultSet resultSet = null;
        try
        {
            //获取连接对象(Druid连接池)
            //connection = Druid.getConnection();
            //或者(自定义数据库连接池)：
            connection = connectionPool.getConnection();
            //或者直接获取(自定义JDBC工具类)：
            //connection = JDBC.getConnection();

            //预编译sql，返回执行者对象
            preparedStatement = connection.prepareStatement(sql, ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE);
            //获取参数的源信息对象
            ParameterMetaData parameterMetaData = preparedStatement.getParameterMetaData();
            //获取参数个数
            int count = parameterMetaData.getParameterCount();
            //判断参数是否一致，如果不一致，异常抛出
            if (objs.length != count)
            {
                throw new RuntimeException("queryForArray方法中参数个数不一致!");
            }
            //为问号占位符赋值
            for (int i = 0; i < count; i++)
            {
                preparedStatement.setObject(i + 1, objs[i]);
            }
            //执行sql语句,返回结果集
            resultSet = preparedStatement.executeQuery();
            //通过结果集的对象获取结果源信息对象
            ResultSetMetaData metaData = resultSet.getMetaData();
            //通过源信息对象获取列数
            int column = metaData.getColumnCount();
            //定义HashMap集合对象
            HashMap<String, Object> map = null;
            //填充数据
            while (resultSet.next())
            {
                //开辟对象
                map = new HashMap<>();
                //循环遍历列数
                for (int i = 0; i < column; i++)
                {
                    //获取列名
                    String columnName = metaData.getColumnName(i + 1);
                    //通过列名获取数据
                    Object object = resultSet.getObject(columnName);
                    //System.err.print(object+" ");
                    //键值对加入到map集合中
                    map.put(columnName, object);
                }
                //加入到list集合中
                list.add(map);
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        finally
        {
            //释放资源
            Druid.close(connection, preparedStatement, resultSet);
            //或者：
            //JDBC.close(connection, preparedStatement,resultSet);
        }
        //返回结果
        return list;
    }
}
```



```java
package mao.t2;

import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;

/**
 * Project name(项目名称)：java并发编程_享元模式
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 13:46
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final ConnectionPool connectionPool = new ConnectionPool();

    public static void main(String[] args) throws SQLException
    {

        for (int i = 0; i < 20; i++)
        {
            Connection connection = connectionPool.getConnection();
            System.out.println(connection);
            connection.close();
        }

//        List<HashMap<String, Object>> students = JDBCTemplate2.queryForListMap("select * from student");
//        for (HashMap<String, Object> student : students)
//        {
//            System.out.println(student);
//        }
    }
}
```



运行结果：

```sh
mao.t2.Connection@435fb7b5
2022-09-08  14:23:31.529  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@4e70a728
mao.t2.Connection@4ba302e0
2022-09-08  14:23:31.531  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@e98770d
mao.t2.Connection@1ae67cad
2022-09-08  14:23:31.532  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@2f6e28bc
mao.t2.Connection@7c098bb3
2022-09-08  14:23:31.532  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@31e4bb20
mao.t2.Connection@18cebaa5
2022-09-08  14:23:31.533  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@463b4ac8
mao.t2.Connection@765f05af
2022-09-08  14:23:31.533  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@62f68dff
mao.t2.Connection@f001896
2022-09-08  14:23:31.534  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@13f17eb4
mao.t2.Connection@1d0d6318
2022-09-08  14:23:31.534  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@4bc28c33
mao.t2.Connection@4409e975
2022-09-08  14:23:31.534  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@5c153b9e
mao.t2.Connection@2a7686a7
2022-09-08  14:23:31.535  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@758a34ce
mao.t2.Connection@7ec3394b
2022-09-08  14:23:31.535  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@4e70a728
mao.t2.Connection@bff34c6
2022-09-08  14:23:31.535  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@e98770d
mao.t2.Connection@1522d8a0
2022-09-08  14:23:31.536  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@2f6e28bc
mao.t2.Connection@312ab28e
2022-09-08  14:23:31.536  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@31e4bb20
mao.t2.Connection@5644dc81
2022-09-08  14:23:31.536  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@463b4ac8
mao.t2.Connection@246f8b8b
2022-09-08  14:23:31.537  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@62f68dff
mao.t2.Connection@278bb07e
2022-09-08  14:23:31.537  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@13f17eb4
mao.t2.Connection@4351c8c3
2022-09-08  14:23:31.537  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@4bc28c33
mao.t2.Connection@3381b4fc
2022-09-08  14:23:31.538  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@5c153b9e
mao.t2.Connection@6bea52d4
2022-09-08  14:23:31.538  [main] DEBUG mao.t2.Connection:  归还连接：com.mysql.cj.jdbc.ConnectionImpl@758a34ce
```





















# 共享模型之工具

## 线程池

### 自定义线程池



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayDeque;
import java.util.Deque;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): BlockingQueue
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:40
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class BlockingQueue<T>
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(BlockingQueue.class);

    /**
     * 任务队列
     */
    private final Deque<T> queue = new ArrayDeque<>();

    /**
     * 锁
     */
    private final ReentrantLock lock = new ReentrantLock();

    /**
     * 生产者条件变量
     */
    private final Condition fullWaitSet = lock.newCondition();

    /**
     * 消费者条件变量
     */
    private final Condition emptyWaitSet = lock.newCondition();

    /**
     * 队列的容量
     */
    private int capacity;

    /**
     * 阻塞队列
     *
     * @param capacity 队列的容量
     */
    public BlockingQueue(int capacity)
    {
        this.capacity = capacity;
    }

    /**
     * 阻塞获取
     *
     * @return {@link T}
     */
    public T take()
    {
        lock.lock();
        try
        {
            //队列为空，则一直等待
            while (queue.isEmpty())
            {
                try
                {
                    //消费者条件变量
                    emptyWaitSet.await();
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
            //不为空
            T t = queue.removeFirst();
            //唤醒生产者条件变量里的线程
            fullWaitSet.signal();
            return t;
        }
        finally
        {
            lock.unlock();
        }
    }

    /**
     * 阻塞添加
     *
     * @param task 任务
     */
    public void put(T task)
    {
        lock.lock();
        try
        {
            //队列已满，则一直等待
            while (queue.size() == capacity)
            {
                log.debug("等待加入任务队列：" + task);
                try
                {
                    fullWaitSet.await();
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
            //队列不为满
            log.debug("加入任务队列：" + task);
            queue.addLast(task);
            emptyWaitSet.signal();
        }
        finally
        {
            lock.unlock();
        }
    }

    /**
     * 带超时阻塞获取
     *
     * @param timeout 超时
     * @param unit    单位
     * @return {@link T}
     */
    public T poll(long timeout, TimeUnit unit)
    {
        lock.lock();
        try
        {
            long nanos = unit.toNanos(timeout);
            //队列为空，则一直等待
            while (queue.isEmpty())
            {
                try
                {
                    if (nanos <= 0)
                    {
                        return null;
                    }
                    nanos = emptyWaitSet.awaitNanos(nanos);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
            //不为空
            T t = queue.removeFirst();
            //唤醒生产者条件变量里的线程
            fullWaitSet.signal();
            return t;
        }
        finally
        {
            lock.unlock();
        }
    }

    /**
     * 带超时时间阻塞添加
     *
     * @param task     任务
     * @param timeout  超时
     * @param timeUnit 时间单位
     * @return boolean
     */
    public boolean offer(T task, long timeout, TimeUnit timeUnit)
    {
        lock.lock();
        try
        {
            long nanos = timeUnit.toNanos(timeout);
            //队列已满，则一直等待
            while (queue.size() == capacity)
            {
                if (nanos <= 0)
                {
                    return false;
                }
                log.debug("等待加入任务队列：" + task);
                try
                {
                    nanos = fullWaitSet.awaitNanos(nanos);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
            //队列不为满
            log.debug("加入任务队列：" + task);
            queue.addLast(task);
            emptyWaitSet.signal();
            return true;
        }
        finally
        {
            lock.unlock();
        }
    }

    /**
     * 获取队列大小
     *
     * @return int
     */
    public int size()
    {
        lock.lock();
        try
        {
            return queue.size();
        }
        finally
        {
            lock.unlock();
        }
    }

    /**
     * @param rejectPolicy 拒绝策略
     * @param task         任务
     */
    public void tryPut(RejectPolicy<T> rejectPolicy, T task)
    {
        lock.lock();
        try
        {
            // 判断队列是否满
            if (queue.size() == capacity)
            {
                rejectPolicy.reject(this, task);
            }
            else
            {
                log.debug("加入任务队列：" + task);
                queue.addLast(task);
                emptyWaitSet.signal();
            }
        }
        finally
        {
            lock.unlock();
        }
    }
}
```





```java
package mao.t1;


/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Interface(接口名): RejectPolicy
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 自定义拒绝策略接口
 */

@FunctionalInterface
interface RejectPolicy<T>
{
    void reject(BlockingQueue<T> queue, T task);
}
```





```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashSet;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): ThreadPool
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 15:03
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class ThreadPool
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(ThreadPool.class);

    /**
     * 任务队列
     */
    private final BlockingQueue<Runnable> taskQueue;

    /**
     * 线程集合
     */
    private final HashSet<Worker> workers = new HashSet<>();

    /**
     * 核心线程数大小
     */
    private final int coreSize;

    /**
     * 获取任务时的超时时间
     */
    private final long timeout;

    /**
     * 时间单位
     */
    private final TimeUnit timeUnit;

    /**
     * 拒绝策略
     */
    private final RejectPolicy<Runnable> rejectPolicy;

    /**
     * 构造方法，线程池
     *
     * @param coreSize     核心线程数大小
     * @param timeout      超时时间
     * @param timeUnit     时间单位
     * @param queueSize    队列大小
     * @param rejectPolicy 拒绝策略
     */
    public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueSize,
                      RejectPolicy<Runnable> rejectPolicy)
    {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueSize);
        this.rejectPolicy = rejectPolicy;
    }


    /**
     * 执行任务
     *
     * @param task 任务
     */
    public void execute(Runnable task)
    {
        synchronized (workers)
        {
            //// 当任务数没有超过coreSize时，直接交给worker对象执行
            if (workers.size() < coreSize)
            {
                Worker worker = new Worker(task);
                log.debug("task加入到队列 workers  task:" + task + "    worker:" + worker);
                workers.add(worker);
                worker.start();
            }
            //任务数超过 coreSize
            else
            {
                //选择策略：

                //1.一直等待
                //taskQueue.put(task);
                //2.超时等待
                //taskQueue.offer(task, timeout, timeUnit);
                //3.抛出异常，不加入等待队列
                //throw new RuntimeException("workers队列已满! task无法加入：" + task);
                //4.放弃任务执行，不加入等待队列
                //log.warn("workers队列已满！task被丢弃：" + task);
                //5.由调用者决定，队列满时...
                taskQueue.tryPut(rejectPolicy, task);
            }
        }
    }


    class Worker extends Thread
    {
        /**
         * 任务
         */
        private Runnable task;

        /**
         * 构造方法
         *
         * @param task 任务
         */
        public Worker(Runnable task)
        {
            this.task = task;
        }


        @Override
        public void run()
        {
            // 当 task 不为空，执行任务
            //当 task 执行完毕，再接着从任务队列获取任务并执行
            while (task != null || (task = taskQueue.poll(timeout, timeUnit)) != null)
            {
                try
                {
                    log.debug("正在执行任务：" + task);
                    task.run();
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    //设置为空，不然会影响下一次的判断
                    task = null;
                }
            }
            //队列里也没有任务了
            //移除
            synchronized (workers)
            {
                log.debug("worker 被移除" + this);
                workers.remove(this);
            }
        }
    }

}
```





```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ThreadPool threadPool = new ThreadPool(2, 3, TimeUnit.SECONDS, 10, new RejectPolicy<Runnable>()
        {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task)
            {
                queue.put(task);
            }
        });

        for (int i = 0; i < 5; i++)
        {
            threadPool.execute(new Runnable()
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
        }
    }
}
```



运行结果：

```sh
2022-09-08  20:03:44.199  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@56cdfb3b    worker:Thread[Thread-0,5,main]
2022-09-08  20:03:44.201  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@306e95ec    worker:Thread[Thread-1,5,main]
2022-09-08  20:03:44.202  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@6fd83fc1
2022-09-08  20:03:44.202  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@56cdfb3b
2022-09-08  20:03:44.202  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@306e95ec
2022-09-08  20:03:44.202  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@4f2b503c
2022-09-08  20:03:44.202  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@bae7dc0
2022-09-08  20:03:46.213  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6fd83fc1
2022-09-08  20:03:46.213  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@4f2b503c
2022-09-08  20:03:48.227  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@bae7dc0
2022-09-08  20:03:51.235  [Thread-1] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-1,5,main]
2022-09-08  20:03:53.241  [Thread-0] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-0,5,main]
```





更改线程池大小

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ThreadPool threadPool = new ThreadPool(3, 3, TimeUnit.SECONDS, 10, new RejectPolicy<Runnable>()
        {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task)
            {
                queue.put(task);
            }
        });

        for (int i = 0; i < 5; i++)
        {
            threadPool.execute(new Runnable()
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
        }
    }
}
```



运行结果：

```sh
2022-09-08  20:10:18.656  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@56cdfb3b    worker:Thread[Thread-0,5,main]
2022-09-08  20:10:18.658  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@306e95ec    worker:Thread[Thread-1,5,main]
2022-09-08  20:10:18.659  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@6fd83fc1    worker:Thread[Thread-2,5,main]
2022-09-08  20:10:18.659  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@306e95ec
2022-09-08  20:10:18.659  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@56cdfb3b
2022-09-08  20:10:18.659  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@bae7dc0
2022-09-08  20:10:18.659  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6fd83fc1
2022-09-08  20:10:18.659  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@209da20d
2022-09-08  20:10:20.661  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@bae7dc0
2022-09-08  20:10:20.675  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@209da20d
2022-09-08  20:10:23.675  [Thread-2] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-2,5,main]
2022-09-08  20:10:25.671  [Thread-1] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-1,5,main]
2022-09-08  20:10:25.687  [Thread-0] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-0,5,main]
```





更改加入的线程数

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ThreadPool threadPool = new ThreadPool(3, 3, TimeUnit.SECONDS, 10, new RejectPolicy<Runnable>()
        {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task)
            {
                queue.put(task);
            }
        });

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.execute(new Runnable()
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
                    log.debug("" + finalI);
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-08  20:18:06.494  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@56cdfb3b    worker:Thread[Thread-0,5,main]
2022-09-08  20:18:06.496  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@306e95ec    worker:Thread[Thread-1,5,main]
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@6fd83fc1    worker:Thread[Thread-2,5,main]
2022-09-08  20:18:06.497  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@306e95ec
2022-09-08  20:18:06.497  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@56cdfb3b
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@bae7dc0
2022-09-08  20:18:06.497  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6fd83fc1
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@209da20d
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@e15b7e8
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@1b2abca6
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@6392827e
2022-09-08  20:18:06.497  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:18:06.498  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@d5b810e
2022-09-08  20:18:06.498  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@43dac38f
2022-09-08  20:18:06.498  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@342c38f8
2022-09-08  20:18:06.498  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@c88a337
2022-09-08  20:18:06.498  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@5d0a1059
2022-09-08  20:18:08.501  [Thread-1] DEBUG mao.t1.Test:  1
2022-09-08  20:18:08.501  [Thread-2] DEBUG mao.t1.Test:  2
2022-09-08  20:18:08.501  [Thread-0] DEBUG mao.t1.Test:  0
2022-09-08  20:18:08.502  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@bae7dc0
2022-09-08  20:18:08.502  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@209da20d
2022-09-08  20:18:08.502  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@5d0a1059
2022-09-08  20:18:08.502  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@593aaf41
2022-09-08  20:18:08.502  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@5a56cdac
2022-09-08  20:18:08.502  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@e15b7e8
2022-09-08  20:18:08.502  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@5a56cdac
2022-09-08  20:18:08.503  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@7c711375
2022-09-08  20:18:10.502  [Thread-1] DEBUG mao.t1.Test:  3
2022-09-08  20:18:10.502  [Thread-2] DEBUG mao.t1.Test:  4
2022-09-08  20:18:10.502  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@1b2abca6
2022-09-08  20:18:10.502  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6392827e
2022-09-08  20:18:10.502  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@7c711375
2022-09-08  20:18:10.502  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@57cf54e1
2022-09-08  20:18:10.502  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@5b03b9fe
2022-09-08  20:18:10.517  [Thread-0] DEBUG mao.t1.Test:  5
2022-09-08  20:18:10.517  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:18:10.517  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@5b03b9fe
2022-09-08  20:18:10.517  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@37d4349f
2022-09-08  20:18:12.514  [Thread-1] DEBUG mao.t1.Test:  6
2022-09-08  20:18:12.514  [Thread-2] DEBUG mao.t1.Test:  7
2022-09-08  20:18:12.514  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@d5b810e
2022-09-08  20:18:12.514  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@37d4349f
2022-09-08  20:18:12.514  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@43dac38f
2022-09-08  20:18:12.530  [Thread-0] DEBUG mao.t1.Test:  8
2022-09-08  20:18:12.530  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@342c38f8
2022-09-08  20:18:14.525  [Thread-1] DEBUG mao.t1.Test:  9
2022-09-08  20:18:14.525  [Thread-2] DEBUG mao.t1.Test:  10
2022-09-08  20:18:14.525  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@c88a337
2022-09-08  20:18:14.525  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@5d0a1059
2022-09-08  20:18:14.541  [Thread-0] DEBUG mao.t1.Test:  11
2022-09-08  20:18:14.541  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@593aaf41
2022-09-08  20:18:16.530  [Thread-1] DEBUG mao.t1.Test:  13
2022-09-08  20:18:16.530  [Thread-2] DEBUG mao.t1.Test:  12
2022-09-08  20:18:16.530  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@5a56cdac
2022-09-08  20:18:16.530  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@7c711375
2022-09-08  20:18:16.546  [Thread-0] DEBUG mao.t1.Test:  14
2022-09-08  20:18:16.546  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@57cf54e1
2022-09-08  20:18:18.533  [Thread-1] DEBUG mao.t1.Test:  15
2022-09-08  20:18:18.533  [Thread-2] DEBUG mao.t1.Test:  16
2022-09-08  20:18:18.533  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@5b03b9fe
2022-09-08  20:18:18.533  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@37d4349f
2022-09-08  20:18:18.549  [Thread-0] DEBUG mao.t1.Test:  17
2022-09-08  20:18:20.540  [Thread-2] DEBUG mao.t1.Test:  19
2022-09-08  20:18:20.540  [Thread-1] DEBUG mao.t1.Test:  18
2022-09-08  20:18:21.559  [Thread-0] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-0,5,main]
2022-09-08  20:18:23.543  [Thread-1] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-1,5,main]
2022-09-08  20:18:23.543  [Thread-2] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-2,5,main]
```





更改拒绝策略为超时失败

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ThreadPool threadPool = new ThreadPool(3, 3, TimeUnit.SECONDS, 5, new RejectPolicy<Runnable>()
        {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task)
            {
                boolean b = queue.offer(task, 1, TimeUnit.SECONDS);
                if (!b)
                {
                    log.warn("任务" + task + "添加失败");
                }
            }
        });

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.execute(new Runnable()
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
                    log.debug("" + finalI);
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-08  20:25:22.642  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@56cdfb3b    worker:Thread[Thread-0,5,main]
2022-09-08  20:25:22.645  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@306e95ec    worker:Thread[Thread-1,5,main]
2022-09-08  20:25:22.645  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@6fd83fc1    worker:Thread[Thread-2,5,main]
2022-09-08  20:25:22.645  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@56cdfb3b
2022-09-08  20:25:22.645  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@bae7dc0
2022-09-08  20:25:22.645  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6fd83fc1
2022-09-08  20:25:22.645  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@306e95ec
2022-09-08  20:25:22.645  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@209da20d
2022-09-08  20:25:22.645  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@e15b7e8
2022-09-08  20:25:22.645  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@1b2abca6
2022-09-08  20:25:22.646  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@6392827e
2022-09-08  20:25:22.646  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:25:23.646  [main] WARN  mao.t1.Test:  任务mao.t1.Test$2@2ed2d9cb添加失败
2022-09-08  20:25:23.646  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@c88a337
2022-09-08  20:25:24.657  [main] WARN  mao.t1.Test:  任务mao.t1.Test$2@c88a337添加失败
2022-09-08  20:25:24.657  [Thread-2] DEBUG mao.t1.Test:  2
2022-09-08  20:25:24.657  [Thread-0] DEBUG mao.t1.Test:  0
2022-09-08  20:25:24.657  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@5d0a1059
2022-09-08  20:25:24.657  [Thread-1] DEBUG mao.t1.Test:  1
2022-09-08  20:25:24.657  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@bae7dc0
2022-09-08  20:25:24.657  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@209da20d
2022-09-08  20:25:24.657  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@e15b7e8
2022-09-08  20:25:24.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@5d0a1059
2022-09-08  20:25:24.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@485966cc
2022-09-08  20:25:24.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@1de76cc7
2022-09-08  20:25:24.658  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@54bff557
2022-09-08  20:25:25.670  [main] WARN  mao.t1.Test:  任务mao.t1.Test$2@54bff557添加失败
2022-09-08  20:25:25.670  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@593aaf41
2022-09-08  20:25:26.672  [main] WARN  mao.t1.Test:  任务mao.t1.Test$2@593aaf41添加失败
2022-09-08  20:25:26.672  [Thread-2] DEBUG mao.t1.Test:  3
2022-09-08  20:25:26.672  [Thread-1] DEBUG mao.t1.Test:  4
2022-09-08  20:25:26.672  [Thread-0] DEBUG mao.t1.Test:  5
2022-09-08  20:25:26.672  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@5a56cdac
2022-09-08  20:25:26.672  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@1b2abca6
2022-09-08  20:25:26.672  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@5a56cdac
2022-09-08  20:25:26.672  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@7c711375
2022-09-08  20:25:26.672  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@5d0a1059
2022-09-08  20:25:26.672  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6392827e
2022-09-08  20:25:26.673  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@57cf54e1
2022-09-08  20:25:26.673  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@5b03b9fe
2022-09-08  20:25:27.677  [main] WARN  mao.t1.Test:  任务mao.t1.Test$2@5b03b9fe添加失败
2022-09-08  20:25:27.677  [main] DEBUG mao.t1.BlockingQueue:  等待加入任务队列：mao.t1.Test$2@37d4349f
2022-09-08  20:25:28.675  [Thread-1] DEBUG mao.t1.Test:  6
2022-09-08  20:25:28.675  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@485966cc
2022-09-08  20:25:28.675  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@37d4349f
2022-09-08  20:25:28.687  [Thread-2] DEBUG mao.t1.Test:  7
2022-09-08  20:25:28.687  [Thread-0] DEBUG mao.t1.Test:  10
2022-09-08  20:25:28.687  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@1de76cc7
2022-09-08  20:25:28.687  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@5a56cdac
2022-09-08  20:25:30.679  [Thread-1] DEBUG mao.t1.Test:  11
2022-09-08  20:25:30.679  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@7c711375
2022-09-08  20:25:30.694  [Thread-2] DEBUG mao.t1.Test:  12
2022-09-08  20:25:30.694  [Thread-0] DEBUG mao.t1.Test:  15
2022-09-08  20:25:30.694  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@57cf54e1
2022-09-08  20:25:30.694  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@37d4349f
2022-09-08  20:25:32.687  [Thread-1] DEBUG mao.t1.Test:  16
2022-09-08  20:25:32.703  [Thread-0] DEBUG mao.t1.Test:  19
2022-09-08  20:25:32.703  [Thread-2] DEBUG mao.t1.Test:  17
2022-09-08  20:25:35.689  [Thread-1] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-1,5,main]
2022-09-08  20:25:35.706  [Thread-0] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-0,5,main]
2022-09-08  20:25:35.706  [Thread-2] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-2,5,main]
```





更改拒绝策略为直接丢弃

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ThreadPool threadPool = new ThreadPool(3, 3, TimeUnit.SECONDS, 10, new RejectPolicy<Runnable>()
        {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task)
            {
                //queue.put(task);

                /*boolean b = queue.offer(task, 1, TimeUnit.SECONDS);
                if (!b)
                {
                    log.warn("任务" + task + "添加失败");
                }*/

                log.warn("丢弃任务：" + task);
            }
        });

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.execute(new Runnable()
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
                    log.debug("" + finalI);
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-08  20:28:12.655  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@56cdfb3b    worker:Thread[Thread-0,5,main]
2022-09-08  20:28:12.657  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@306e95ec    worker:Thread[Thread-1,5,main]
2022-09-08  20:28:12.657  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@6fd83fc1    worker:Thread[Thread-2,5,main]
2022-09-08  20:28:12.657  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@56cdfb3b
2022-09-08  20:28:12.657  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@bae7dc0
2022-09-08  20:28:12.657  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@306e95ec
2022-09-08  20:28:12.657  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6fd83fc1
2022-09-08  20:28:12.657  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@209da20d
2022-09-08  20:28:12.657  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@e15b7e8
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@1b2abca6
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@6392827e
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@d5b810e
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@43dac38f
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@342c38f8
2022-09-08  20:28:12.658  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@c88a337
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@5d0a1059
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@485966cc
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@1de76cc7
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@54bff557
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@593aaf41
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@5a56cdac
2022-09-08  20:28:12.658  [main] WARN  mao.t1.Test:  丢弃任务：mao.t1.Test$2@7c711375
2022-09-08  20:28:14.657  [Thread-1] DEBUG mao.t1.Test:  1
2022-09-08  20:28:14.658  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@bae7dc0
2022-09-08  20:28:14.659  [Thread-0] DEBUG mao.t1.Test:  0
2022-09-08  20:28:14.659  [Thread-2] DEBUG mao.t1.Test:  2
2022-09-08  20:28:14.659  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@209da20d
2022-09-08  20:28:14.659  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@e15b7e8
2022-09-08  20:28:16.670  [Thread-0] DEBUG mao.t1.Test:  4
2022-09-08  20:28:16.670  [Thread-1] DEBUG mao.t1.Test:  3
2022-09-08  20:28:16.670  [Thread-2] DEBUG mao.t1.Test:  5
2022-09-08  20:28:16.670  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@1b2abca6
2022-09-08  20:28:16.670  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6392827e
2022-09-08  20:28:16.670  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:28:18.681  [Thread-1] DEBUG mao.t1.Test:  7
2022-09-08  20:28:18.681  [Thread-0] DEBUG mao.t1.Test:  6
2022-09-08  20:28:18.681  [Thread-2] DEBUG mao.t1.Test:  8
2022-09-08  20:28:18.681  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@d5b810e
2022-09-08  20:28:18.681  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@43dac38f
2022-09-08  20:28:18.681  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@342c38f8
2022-09-08  20:28:20.683  [Thread-2] DEBUG mao.t1.Test:  11
2022-09-08  20:28:20.683  [Thread-0] DEBUG mao.t1.Test:  10
2022-09-08  20:28:20.683  [Thread-1] DEBUG mao.t1.Test:  9
2022-09-08  20:28:20.683  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@c88a337
2022-09-08  20:28:22.691  [Thread-2] DEBUG mao.t1.Test:  12
2022-09-08  20:28:23.690  [Thread-0] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-0,5,main]
2022-09-08  20:28:23.690  [Thread-1] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-1,5,main]
2022-09-08  20:28:25.706  [Thread-2] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-2,5,main]
```



更改拒绝策略为抛出异常

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_自定义线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/8
 * Time(创建时间)： 14:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ThreadPool threadPool = new ThreadPool(3, 3, TimeUnit.SECONDS, 10, new RejectPolicy<Runnable>()
        {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task)
            {
                //queue.put(task);

                /*boolean b = queue.offer(task, 1, TimeUnit.SECONDS);
                if (!b)
                {
                    log.warn("任务" + task + "添加失败");
                }*/

                //log.warn("丢弃任务：" + task);

                throw new RuntimeException("线程池任务队列已满！task:" + task + "已被丢弃");
            }
        });

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            try
            {
                threadPool.execute(new Runnable()
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
                        log.debug("" + finalI);
                    }
                });
            }
            catch (RuntimeException e)
            {
                //处理异常
                e.printStackTrace();
            }
        }
    }
}
```



运行结果：

```sh
2022-09-08  20:33:09.977  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@56cdfb3b    worker:Thread[Thread-0,5,main]
2022-09-08  20:33:09.979  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@306e95ec    worker:Thread[Thread-1,5,main]
2022-09-08  20:33:09.979  [main] DEBUG mao.t1.ThreadPool:  task加入到队列 workers  task:mao.t1.Test$2@6fd83fc1    worker:Thread[Thread-2,5,main]
2022-09-08  20:33:09.979  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@56cdfb3b
2022-09-08  20:33:09.979  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@306e95ec
2022-09-08  20:33:09.979  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@bae7dc0
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@209da20d
2022-09-08  20:33:09.980  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6fd83fc1
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@e15b7e8
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@1b2abca6
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@6392827e
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@d5b810e
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@43dac38f
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@342c38f8
2022-09-08  20:33:09.980  [main] DEBUG mao.t1.BlockingQueue:  加入任务队列：mao.t1.Test$2@c88a337
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@5d0a1059已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@593aaf41已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@7c711375已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@5b03b9fe已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@434a63ab已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@2805d709已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
java.lang.RuntimeException: 线程池任务队列已满！task:mao.t1.Test$2@2ea41516已被丢弃
	at mao.t1.Test$1.reject(Test.java:42)
	at mao.t1.Test$1.reject(Test.java:28)
	at mao.t1.BlockingQueue.tryPut(BlockingQueue.java:250)
	at mao.t1.ThreadPool.execute(ThreadPool.java:110)
	at mao.t1.Test.main(Test.java:51)
2022-09-08  20:33:11.988  [Thread-1] DEBUG mao.t1.Test:  1
2022-09-08  20:33:11.988  [Thread-0] DEBUG mao.t1.Test:  0
2022-09-08  20:33:11.988  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@bae7dc0
2022-09-08  20:33:11.988  [Thread-2] DEBUG mao.t1.Test:  2
2022-09-08  20:33:11.988  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@209da20d
2022-09-08  20:33:11.989  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@e15b7e8
2022-09-08  20:33:13.991  [Thread-2] DEBUG mao.t1.Test:  5
2022-09-08  20:33:13.991  [Thread-0] DEBUG mao.t1.Test:  4
2022-09-08  20:33:13.991  [Thread-1] DEBUG mao.t1.Test:  3
2022-09-08  20:33:13.991  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@1b2abca6
2022-09-08  20:33:13.991  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@6392827e
2022-09-08  20:33:13.991  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@2ed2d9cb
2022-09-08  20:33:15.998  [Thread-1] DEBUG mao.t1.Test:  8
2022-09-08  20:33:15.998  [Thread-0] DEBUG mao.t1.Test:  7
2022-09-08  20:33:15.998  [Thread-2] DEBUG mao.t1.Test:  6
2022-09-08  20:33:15.998  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@d5b810e
2022-09-08  20:33:15.998  [Thread-2] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@43dac38f
2022-09-08  20:33:15.998  [Thread-0] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@342c38f8
2022-09-08  20:33:18.007  [Thread-1] DEBUG mao.t1.Test:  9
2022-09-08  20:33:18.007  [Thread-0] DEBUG mao.t1.Test:  11
2022-09-08  20:33:18.007  [Thread-2] DEBUG mao.t1.Test:  10
2022-09-08  20:33:18.007  [Thread-1] DEBUG mao.t1.ThreadPool:  正在执行任务：mao.t1.Test$2@c88a337
2022-09-08  20:33:20.021  [Thread-1] DEBUG mao.t1.Test:  12
2022-09-08  20:33:21.011  [Thread-2] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-2,5,main]
2022-09-08  20:33:21.011  [Thread-0] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-0,5,main]
2022-09-08  20:33:23.026  [Thread-1] DEBUG mao.t1.ThreadPool:  worker 被移除Thread[Thread-1,5,main]
```







### ThreadPoolExecutor

ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量



![image-20220908211748762](img/java并发编程学习笔记/image-20220908211748762.png)



![image-20220908211859666](img/java并发编程学习笔记/image-20220908211859666.png)



这些信息存储在一个原子变量 ctl 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 cas 原子操作进行赋值





**构造方法：**



```java
public class ThreadPoolExecutor extends AbstractExecutorService 
{
    ...
    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
    ...
}
```





* corePoolSize： 核心线程数目 (最多保留的线程数) 
* maximumPoolSize： 最大线程数目 
* keepAliveTime： 生存时间 - 针对救急线程 
* unit ：时间单位 - 针对救急线程 
* workQueue： 阻塞队列 
* threadFactory： 线程工厂 - 可以为线程创建时起个好名字 
* handler： 拒绝策略





**工作方式：**



* 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务
* 当线程数达到 corePoolSize 并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue 队列排 队，直到有空闲的线程
* 如果队列选择了有界队列，那么任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize 数目的线 程来救急
* 如果线程到达 maximumPoolSize 仍然有新任务这时会执行拒绝策略。拒绝策略 jdk 提供了 4 种实现，其它 著名框架也提供了实现
  * AbortPolicy 让调用者抛出 RejectedExecutionException 异常，这是默认策略
  * CallerRunsPolicy 让调用者运行任务
  * DiscardPolicy 放弃本次任务
  * DiscardOldestPolicy 放弃队列中最早的任务，本任务取而代之
  * Dubbo 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方便定位问题
  * Netty 的实现，是创建一个新线程来执行任务
  * ActiveMQ 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略
  * PinPoint 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略
* 当高峰过去后，超过corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由 keepAliveTime 和 unit 来控制



![image-20220908213146748](img/java并发编程学习笔记/image-20220908213146748.png)





根据这个构造方法，JDK Executors 类中提供了众多工厂方法来创建各种用途的线程池

* newFixedThreadPool
* newCachedThreadPool
* newSingleThreadExecutor





### Executors

#### newFixedThreadPool

```java

    /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```



* 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间
* 阻塞队列是无界的，可以放任意数量的任务



适用于任务量已知，相对耗时的任务



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 10:23
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    //log.debug(finalI + "结束运行");
                }
            });
        }


    }
}
```



运行结果：

```sh
2022-09-09  11:02:24.327  [pool-2-thread-1] DEBUG mao.t1.Test:  0开始运行
2022-09-09  11:02:24.327  [pool-2-thread-4] DEBUG mao.t1.Test:  3开始运行
2022-09-09  11:02:24.327  [pool-2-thread-3] DEBUG mao.t1.Test:  2开始运行
2022-09-09  11:02:24.327  [pool-2-thread-2] DEBUG mao.t1.Test:  1开始运行
2022-09-09  11:02:25.344  [pool-2-thread-2] DEBUG mao.t1.Test:  4开始运行
2022-09-09  11:02:25.344  [pool-2-thread-3] DEBUG mao.t1.Test:  5开始运行
2022-09-09  11:02:25.344  [pool-2-thread-4] DEBUG mao.t1.Test:  6开始运行
2022-09-09  11:02:25.344  [pool-2-thread-1] DEBUG mao.t1.Test:  7开始运行
2022-09-09  11:02:26.356  [pool-2-thread-3] DEBUG mao.t1.Test:  10开始运行
2022-09-09  11:02:26.356  [pool-2-thread-2] DEBUG mao.t1.Test:  8开始运行
2022-09-09  11:02:26.356  [pool-2-thread-4] DEBUG mao.t1.Test:  9开始运行
2022-09-09  11:02:26.356  [pool-2-thread-1] DEBUG mao.t1.Test:  11开始运行
2022-09-09  11:02:27.363  [pool-2-thread-2] DEBUG mao.t1.Test:  12开始运行
2022-09-09  11:02:27.363  [pool-2-thread-1] DEBUG mao.t1.Test:  13开始运行
2022-09-09  11:02:27.363  [pool-2-thread-4] DEBUG mao.t1.Test:  14开始运行
2022-09-09  11:02:27.363  [pool-2-thread-3] DEBUG mao.t1.Test:  15开始运行
2022-09-09  11:02:28.364  [pool-2-thread-1] DEBUG mao.t1.Test:  16开始运行
2022-09-09  11:02:28.364  [pool-2-thread-4] DEBUG mao.t1.Test:  18开始运行
2022-09-09  11:02:28.364  [pool-2-thread-3] DEBUG mao.t1.Test:  17开始运行
2022-09-09  11:02:28.364  [pool-2-thread-2] DEBUG mao.t1.Test:  19开始运行
```



更改线程数

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 10:23
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(8);

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    //log.debug(finalI + "结束运行");
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-09  11:04:08.120  [pool-2-thread-6] DEBUG mao.t1.Test:  5开始运行
2022-09-09  11:04:08.120  [pool-2-thread-8] DEBUG mao.t1.Test:  7开始运行
2022-09-09  11:04:08.120  [pool-2-thread-5] DEBUG mao.t1.Test:  4开始运行
2022-09-09  11:04:08.120  [pool-2-thread-4] DEBUG mao.t1.Test:  3开始运行
2022-09-09  11:04:08.120  [pool-2-thread-1] DEBUG mao.t1.Test:  0开始运行
2022-09-09  11:04:08.120  [pool-2-thread-3] DEBUG mao.t1.Test:  2开始运行
2022-09-09  11:04:08.120  [pool-2-thread-2] DEBUG mao.t1.Test:  1开始运行
2022-09-09  11:04:08.120  [pool-2-thread-7] DEBUG mao.t1.Test:  6开始运行
2022-09-09  11:04:09.123  [pool-2-thread-5] DEBUG mao.t1.Test:  9开始运行
2022-09-09  11:04:09.123  [pool-2-thread-3] DEBUG mao.t1.Test:  8开始运行
2022-09-09  11:04:09.123  [pool-2-thread-1] DEBUG mao.t1.Test:  11开始运行
2022-09-09  11:04:09.123  [pool-2-thread-4] DEBUG mao.t1.Test:  10开始运行
2022-09-09  11:04:09.124  [pool-2-thread-8] DEBUG mao.t1.Test:  12开始运行
2022-09-09  11:04:09.127  [pool-2-thread-2] DEBUG mao.t1.Test:  13开始运行
2022-09-09  11:04:09.127  [pool-2-thread-7] DEBUG mao.t1.Test:  14开始运行
2022-09-09  11:04:09.127  [pool-2-thread-6] DEBUG mao.t1.Test:  15开始运行
2022-09-09  11:04:10.124  [pool-2-thread-1] DEBUG mao.t1.Test:  16开始运行
2022-09-09  11:04:10.124  [pool-2-thread-5] DEBUG mao.t1.Test:  17开始运行
2022-09-09  11:04:10.126  [pool-2-thread-4] DEBUG mao.t1.Test:  18开始运行
2022-09-09  11:04:10.126  [pool-2-thread-3] DEBUG mao.t1.Test:  19开始运行
```





####  newCachedThreadPool

```java

    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```



* 核心线程数是 0， 最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着
  * 全部都是救急线程（60s 后可以回收）
  * 救急线程可以无限创建
* 队列采用了 SynchronousQueue 实现特点是，它没有容量，没有线程来取是放不进去的



整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线程。 适合任务数比较密集，但每个任务执行时间较短的情况





```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:06
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newCachedThreadPool();

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束运行");
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-09  11:07:33.511  [pool-2-thread-11] DEBUG mao.t2.Test:  10开始运行
2022-09-09  11:07:33.511  [pool-2-thread-1] DEBUG mao.t2.Test:  0开始运行
2022-09-09  11:07:33.511  [pool-2-thread-2] DEBUG mao.t2.Test:  1开始运行
2022-09-09  11:07:33.511  [pool-2-thread-13] DEBUG mao.t2.Test:  12开始运行
2022-09-09  11:07:33.511  [pool-2-thread-19] DEBUG mao.t2.Test:  18开始运行
2022-09-09  11:07:33.511  [pool-2-thread-7] DEBUG mao.t2.Test:  6开始运行
2022-09-09  11:07:33.511  [pool-2-thread-20] DEBUG mao.t2.Test:  19开始运行
2022-09-09  11:07:33.511  [pool-2-thread-17] DEBUG mao.t2.Test:  16开始运行
2022-09-09  11:07:33.511  [pool-2-thread-8] DEBUG mao.t2.Test:  7开始运行
2022-09-09  11:07:33.511  [pool-2-thread-9] DEBUG mao.t2.Test:  8开始运行
2022-09-09  11:07:33.511  [pool-2-thread-16] DEBUG mao.t2.Test:  15开始运行
2022-09-09  11:07:33.511  [pool-2-thread-12] DEBUG mao.t2.Test:  11开始运行
2022-09-09  11:07:33.511  [pool-2-thread-15] DEBUG mao.t2.Test:  14开始运行
2022-09-09  11:07:33.511  [pool-2-thread-5] DEBUG mao.t2.Test:  4开始运行
2022-09-09  11:07:33.511  [pool-2-thread-6] DEBUG mao.t2.Test:  5开始运行
2022-09-09  11:07:33.511  [pool-2-thread-4] DEBUG mao.t2.Test:  3开始运行
2022-09-09  11:07:33.511  [pool-2-thread-3] DEBUG mao.t2.Test:  2开始运行
2022-09-09  11:07:33.511  [pool-2-thread-14] DEBUG mao.t2.Test:  13开始运行
2022-09-09  11:07:33.511  [pool-2-thread-10] DEBUG mao.t2.Test:  9开始运行
2022-09-09  11:07:33.511  [pool-2-thread-18] DEBUG mao.t2.Test:  17开始运行
2022-09-09  11:07:34.515  [pool-2-thread-1] DEBUG mao.t2.Test:  0结束运行
2022-09-09  11:07:34.515  [pool-2-thread-8] DEBUG mao.t2.Test:  7结束运行
2022-09-09  11:07:34.515  [pool-2-thread-6] DEBUG mao.t2.Test:  5结束运行
2022-09-09  11:07:34.515  [pool-2-thread-7] DEBUG mao.t2.Test:  6结束运行
2022-09-09  11:07:34.515  [pool-2-thread-14] DEBUG mao.t2.Test:  13结束运行
2022-09-09  11:07:34.515  [pool-2-thread-2] DEBUG mao.t2.Test:  1结束运行
2022-09-09  11:07:34.515  [pool-2-thread-13] DEBUG mao.t2.Test:  12结束运行
2022-09-09  11:07:34.515  [pool-2-thread-17] DEBUG mao.t2.Test:  16结束运行
2022-09-09  11:07:34.515  [pool-2-thread-16] DEBUG mao.t2.Test:  15结束运行
2022-09-09  11:07:34.515  [pool-2-thread-9] DEBUG mao.t2.Test:  8结束运行
2022-09-09  11:07:34.515  [pool-2-thread-19] DEBUG mao.t2.Test:  18结束运行
2022-09-09  11:07:34.515  [pool-2-thread-20] DEBUG mao.t2.Test:  19结束运行
2022-09-09  11:07:34.515  [pool-2-thread-5] DEBUG mao.t2.Test:  4结束运行
2022-09-09  11:07:34.517  [pool-2-thread-4] DEBUG mao.t2.Test:  3结束运行
2022-09-09  11:07:34.517  [pool-2-thread-15] DEBUG mao.t2.Test:  14结束运行
2022-09-09  11:07:34.517  [pool-2-thread-12] DEBUG mao.t2.Test:  11结束运行
2022-09-09  11:07:34.517  [pool-2-thread-3] DEBUG mao.t2.Test:  2结束运行
2022-09-09  11:07:34.524  [pool-2-thread-10] DEBUG mao.t2.Test:  9结束运行
2022-09-09  11:07:34.524  [pool-2-thread-11] DEBUG mao.t2.Test:  10结束运行
2022-09-09  11:07:34.524  [pool-2-thread-18] DEBUG mao.t2.Test:  17结束运行
```



更改提交的线程数

```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:06
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newCachedThreadPool();

        for (int i = 0; i < 30; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束运行");
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-09  11:10:41.429  [pool-2-thread-1] DEBUG mao.t2.Test:  0开始运行
2022-09-09  11:10:41.429  [pool-2-thread-14] DEBUG mao.t2.Test:  13开始运行
2022-09-09  11:10:41.429  [pool-2-thread-22] DEBUG mao.t2.Test:  21开始运行
2022-09-09  11:10:41.429  [pool-2-thread-9] DEBUG mao.t2.Test:  8开始运行
2022-09-09  11:10:41.429  [pool-2-thread-21] DEBUG mao.t2.Test:  20开始运行
2022-09-09  11:10:41.429  [pool-2-thread-15] DEBUG mao.t2.Test:  14开始运行
2022-09-09  11:10:41.429  [pool-2-thread-19] DEBUG mao.t2.Test:  18开始运行
2022-09-09  11:10:41.429  [pool-2-thread-2] DEBUG mao.t2.Test:  1开始运行
2022-09-09  11:10:41.429  [pool-2-thread-11] DEBUG mao.t2.Test:  10开始运行
2022-09-09  11:10:41.429  [pool-2-thread-7] DEBUG mao.t2.Test:  6开始运行
2022-09-09  11:10:41.429  [pool-2-thread-3] DEBUG mao.t2.Test:  2开始运行
2022-09-09  11:10:41.429  [pool-2-thread-13] DEBUG mao.t2.Test:  12开始运行
2022-09-09  11:10:41.429  [pool-2-thread-23] DEBUG mao.t2.Test:  22开始运行
2022-09-09  11:10:41.429  [pool-2-thread-25] DEBUG mao.t2.Test:  24开始运行
2022-09-09  11:10:41.429  [pool-2-thread-10] DEBUG mao.t2.Test:  9开始运行
2022-09-09  11:10:41.429  [pool-2-thread-12] DEBUG mao.t2.Test:  11开始运行
2022-09-09  11:10:41.429  [pool-2-thread-17] DEBUG mao.t2.Test:  16开始运行
2022-09-09  11:10:41.429  [pool-2-thread-29] DEBUG mao.t2.Test:  28开始运行
2022-09-09  11:10:41.429  [pool-2-thread-24] DEBUG mao.t2.Test:  23开始运行
2022-09-09  11:10:41.429  [pool-2-thread-20] DEBUG mao.t2.Test:  19开始运行
2022-09-09  11:10:41.429  [pool-2-thread-30] DEBUG mao.t2.Test:  29开始运行
2022-09-09  11:10:41.429  [pool-2-thread-28] DEBUG mao.t2.Test:  27开始运行
2022-09-09  11:10:41.429  [pool-2-thread-5] DEBUG mao.t2.Test:  4开始运行
2022-09-09  11:10:41.429  [pool-2-thread-6] DEBUG mao.t2.Test:  5开始运行
2022-09-09  11:10:41.429  [pool-2-thread-8] DEBUG mao.t2.Test:  7开始运行
2022-09-09  11:10:41.429  [pool-2-thread-16] DEBUG mao.t2.Test:  15开始运行
2022-09-09  11:10:41.429  [pool-2-thread-27] DEBUG mao.t2.Test:  26开始运行
2022-09-09  11:10:41.429  [pool-2-thread-4] DEBUG mao.t2.Test:  3开始运行
2022-09-09  11:10:41.429  [pool-2-thread-18] DEBUG mao.t2.Test:  17开始运行
2022-09-09  11:10:41.429  [pool-2-thread-26] DEBUG mao.t2.Test:  25开始运行
2022-09-09  11:10:42.433  [pool-2-thread-13] DEBUG mao.t2.Test:  12结束运行
2022-09-09  11:10:42.434  [pool-2-thread-14] DEBUG mao.t2.Test:  13结束运行
2022-09-09  11:10:42.434  [pool-2-thread-3] DEBUG mao.t2.Test:  2结束运行
2022-09-09  11:10:42.434  [pool-2-thread-9] DEBUG mao.t2.Test:  8结束运行
2022-09-09  11:10:42.434  [pool-2-thread-15] DEBUG mao.t2.Test:  14结束运行
2022-09-09  11:10:42.434  [pool-2-thread-19] DEBUG mao.t2.Test:  18结束运行
2022-09-09  11:10:42.434  [pool-2-thread-21] DEBUG mao.t2.Test:  20结束运行
2022-09-09  11:10:42.434  [pool-2-thread-11] DEBUG mao.t2.Test:  10结束运行
2022-09-09  11:10:42.434  [pool-2-thread-7] DEBUG mao.t2.Test:  6结束运行
2022-09-09  11:10:42.434  [pool-2-thread-2] DEBUG mao.t2.Test:  1结束运行
2022-09-09  11:10:42.434  [pool-2-thread-22] DEBUG mao.t2.Test:  21结束运行
2022-09-09  11:10:42.448  [pool-2-thread-1] DEBUG mao.t2.Test:  0结束运行
2022-09-09  11:10:42.448  [pool-2-thread-20] DEBUG mao.t2.Test:  19结束运行
2022-09-09  11:10:42.448  [pool-2-thread-6] DEBUG mao.t2.Test:  5结束运行
2022-09-09  11:10:42.448  [pool-2-thread-27] DEBUG mao.t2.Test:  26结束运行
2022-09-09  11:10:42.448  [pool-2-thread-8] DEBUG mao.t2.Test:  7结束运行
2022-09-09  11:10:42.448  [pool-2-thread-23] DEBUG mao.t2.Test:  22结束运行
2022-09-09  11:10:42.448  [pool-2-thread-4] DEBUG mao.t2.Test:  3结束运行
2022-09-09  11:10:42.448  [pool-2-thread-30] DEBUG mao.t2.Test:  29结束运行
2022-09-09  11:10:42.448  [pool-2-thread-29] DEBUG mao.t2.Test:  28结束运行
2022-09-09  11:10:42.448  [pool-2-thread-12] DEBUG mao.t2.Test:  11结束运行
2022-09-09  11:10:42.448  [pool-2-thread-17] DEBUG mao.t2.Test:  16结束运行
2022-09-09  11:10:42.448  [pool-2-thread-25] DEBUG mao.t2.Test:  24结束运行
2022-09-09  11:10:42.448  [pool-2-thread-24] DEBUG mao.t2.Test:  23结束运行
2022-09-09  11:10:42.448  [pool-2-thread-18] DEBUG mao.t2.Test:  17结束运行
2022-09-09  11:10:42.448  [pool-2-thread-28] DEBUG mao.t2.Test:  27结束运行
2022-09-09  11:10:42.448  [pool-2-thread-16] DEBUG mao.t2.Test:  15结束运行
2022-09-09  11:10:42.448  [pool-2-thread-26] DEBUG mao.t2.Test:  25结束运行
2022-09-09  11:10:42.448  [pool-2-thread-10] DEBUG mao.t2.Test:  9结束运行
2022-09-09  11:10:42.448  [pool-2-thread-5] DEBUG mao.t2.Test:  4结束运行
```





####  newSingleThreadExecutor

```java

    /**
     * Creates an Executor that uses a single worker thread operating
     * off an unbounded queue. (Note however that if this single
     * thread terminates due to a failure during execution prior to
     * shutdown, a new one will take its place if needed to execute
     * subsequent tasks.)  Tasks are guaranteed to execute
     * sequentially, and no more than one task will be active at any
     * given time. Unlike the otherwise equivalent
     * {@code newFixedThreadPool(1)} the returned executor is
     * guaranteed not to be reconfigurable to use additional threads.
     *
     * @return the newly created single-threaded Executor
     */
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```



希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程 也不会被释放

* 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新建一个线程，保证池的正常工作
* Executors.newSingleThreadExecutor() 线程个数始终为1，不能修改
  * FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 ExecutorService 接口，因此不能调用 ThreadPoolExecutor 中特有的方法
* Executors.newFixedThreadPool(1) 初始时为1，以后还可以修改
  * 对外暴露的是 ThreadPoolExecutor 对象，可以强转后调用 setCorePoolSize 等方法进行修改



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:11
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束运行");
                }
            });
        }
    }
}
```



运行结果：

```sh
2022-09-09  11:16:22.399  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始运行
2022-09-09  11:16:23.414  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束运行
2022-09-09  11:16:23.414  [pool-2-thread-1] DEBUG mao.t3.Test:  1开始运行
2022-09-09  11:16:24.419  [pool-2-thread-1] DEBUG mao.t3.Test:  1结束运行
2022-09-09  11:16:24.419  [pool-2-thread-1] DEBUG mao.t3.Test:  2开始运行
2022-09-09  11:16:25.430  [pool-2-thread-1] DEBUG mao.t3.Test:  2结束运行
2022-09-09  11:16:25.430  [pool-2-thread-1] DEBUG mao.t3.Test:  3开始运行
2022-09-09  11:16:26.434  [pool-2-thread-1] DEBUG mao.t3.Test:  3结束运行
2022-09-09  11:16:26.434  [pool-2-thread-1] DEBUG mao.t3.Test:  4开始运行
2022-09-09  11:16:27.439  [pool-2-thread-1] DEBUG mao.t3.Test:  4结束运行
2022-09-09  11:16:27.439  [pool-2-thread-1] DEBUG mao.t3.Test:  5开始运行
2022-09-09  11:16:28.448  [pool-2-thread-1] DEBUG mao.t3.Test:  5结束运行
2022-09-09  11:16:28.448  [pool-2-thread-1] DEBUG mao.t3.Test:  6开始运行
2022-09-09  11:16:29.453  [pool-2-thread-1] DEBUG mao.t3.Test:  6结束运行
2022-09-09  11:16:29.453  [pool-2-thread-1] DEBUG mao.t3.Test:  7开始运行
2022-09-09  11:16:30.466  [pool-2-thread-1] DEBUG mao.t3.Test:  7结束运行
2022-09-09  11:16:30.466  [pool-2-thread-1] DEBUG mao.t3.Test:  8开始运行
2022-09-09  11:16:31.468  [pool-2-thread-1] DEBUG mao.t3.Test:  8结束运行
2022-09-09  11:16:31.468  [pool-2-thread-1] DEBUG mao.t3.Test:  9开始运行
2022-09-09  11:16:32.483  [pool-2-thread-1] DEBUG mao.t3.Test:  9结束运行
2022-09-09  11:16:32.483  [pool-2-thread-1] DEBUG mao.t3.Test:  10开始运行
2022-09-09  11:16:33.488  [pool-2-thread-1] DEBUG mao.t3.Test:  10结束运行
2022-09-09  11:16:33.488  [pool-2-thread-1] DEBUG mao.t3.Test:  11开始运行
2022-09-09  11:16:34.496  [pool-2-thread-1] DEBUG mao.t3.Test:  11结束运行
2022-09-09  11:16:34.496  [pool-2-thread-1] DEBUG mao.t3.Test:  12开始运行
2022-09-09  11:16:35.501  [pool-2-thread-1] DEBUG mao.t3.Test:  12结束运行
2022-09-09  11:16:35.501  [pool-2-thread-1] DEBUG mao.t3.Test:  13开始运行
2022-09-09  11:16:36.510  [pool-2-thread-1] DEBUG mao.t3.Test:  13结束运行
2022-09-09  11:16:36.510  [pool-2-thread-1] DEBUG mao.t3.Test:  14开始运行
2022-09-09  11:16:37.511  [pool-2-thread-1] DEBUG mao.t3.Test:  14结束运行
2022-09-09  11:16:37.511  [pool-2-thread-1] DEBUG mao.t3.Test:  15开始运行
2022-09-09  11:16:38.512  [pool-2-thread-1] DEBUG mao.t3.Test:  15结束运行
2022-09-09  11:16:38.512  [pool-2-thread-1] DEBUG mao.t3.Test:  16开始运行
2022-09-09  11:16:39.521  [pool-2-thread-1] DEBUG mao.t3.Test:  16结束运行
2022-09-09  11:16:39.521  [pool-2-thread-1] DEBUG mao.t3.Test:  17开始运行
2022-09-09  11:16:40.527  [pool-2-thread-1] DEBUG mao.t3.Test:  17结束运行
2022-09-09  11:16:40.527  [pool-2-thread-1] DEBUG mao.t3.Test:  18开始运行
2022-09-09  11:16:41.541  [pool-2-thread-1] DEBUG mao.t3.Test:  18结束运行
2022-09-09  11:16:41.541  [pool-2-thread-1] DEBUG mao.t3.Test:  19开始运行
2022-09-09  11:16:42.547  [pool-2-thread-1] DEBUG mao.t3.Test:  19结束运行
```







### 提交任务

#### execute

```java
//在将来的某个时间执行给定的命令。根据Executor实现的判断，该命令可以在新线程、池线程或调用线程中执行
void execute(Runnable command);
```



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.execute(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```



```sh
2022-09-09  11:24:05.065  [pool-2-thread-4] DEBUG mao.t4.Test:  3开始运行
2022-09-09  11:24:05.065  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始运行
2022-09-09  11:24:05.065  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始运行
2022-09-09  11:24:05.065  [pool-2-thread-3] DEBUG mao.t4.Test:  2开始运行
2022-09-09  11:24:06.080  [pool-2-thread-1] DEBUG mao.t4.Test:  4开始运行
2022-09-09  11:24:06.080  [pool-2-thread-3] DEBUG mao.t4.Test:  5开始运行
2022-09-09  11:24:06.080  [pool-2-thread-4] DEBUG mao.t4.Test:  7开始运行
2022-09-09  11:24:06.080  [pool-2-thread-2] DEBUG mao.t4.Test:  6开始运行
2022-09-09  11:24:07.081  [pool-2-thread-3] DEBUG mao.t4.Test:  8开始运行
2022-09-09  11:24:07.094  [pool-2-thread-4] DEBUG mao.t4.Test:  11开始运行
2022-09-09  11:24:07.094  [pool-2-thread-2] DEBUG mao.t4.Test:  9开始运行
2022-09-09  11:24:07.094  [pool-2-thread-1] DEBUG mao.t4.Test:  10开始运行
2022-09-09  11:24:08.088  [pool-2-thread-3] DEBUG mao.t4.Test:  12开始运行
2022-09-09  11:24:08.104  [pool-2-thread-4] DEBUG mao.t4.Test:  13开始运行
2022-09-09  11:24:08.104  [pool-2-thread-1] DEBUG mao.t4.Test:  15开始运行
2022-09-09  11:24:08.104  [pool-2-thread-2] DEBUG mao.t4.Test:  14开始运行
2022-09-09  11:24:09.098  [pool-2-thread-3] DEBUG mao.t4.Test:  16开始运行
2022-09-09  11:24:09.114  [pool-2-thread-2] DEBUG mao.t4.Test:  18开始运行
2022-09-09  11:24:09.114  [pool-2-thread-1] DEBUG mao.t4.Test:  17开始运行
2022-09-09  11:24:09.114  [pool-2-thread-4] DEBUG mao.t4.Test:  19开始运行
```





#### submit



```java
//提交一个返回值的任务以供执行，并返回一个表示该任务待处理结果的 Future。 Future 的get方法将在成功完成后返回任务的结果。
//如果您想立即阻止等待任务，可以使用result = exec.submit(aCallable).get();形式的结构。
//注意： Executors类包含一组方法，可以将一些其他常见的类似闭包的对象（例如java.security.PrivilegedAction ）转换为Callable形式，
//以便可以提交它们
<T> Future<T> submit(Callable<T> task);
```



```java
package mao.t5;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:25
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);
        List<Future<Integer>> futures = new ArrayList<>();

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            //提交一个返回值的任务以供执行，并返回一个表示该任务待处理结果的 Future。 Future 的get方法将在成功完成后返回任务的结果。
            //如果您想立即阻止等待任务，可以使用result = exec.submit(aCallable).get();形式的结构。
            //注意： Executors类包含一组方法，可以将一些其他常见的类似闭包的对象（例如java.security.PrivilegedAction ）转换为Callable形式，
            //以便可以提交它们
            Future<Integer> future = threadPool.submit(new Callable<Integer>()
            {
                @Override
                public Integer call() throws Exception
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    return finalI;
                }
            });
            futures.add(future);
        }

        for (Future<Integer> future : futures)
        {
            Integer integer = future.get();
            log.debug("结果：" + integer);
        }
    }
}
```



```sh
2022-09-09  11:35:40.113  [pool-2-thread-4] DEBUG mao.t5.Test:  3开始运行
2022-09-09  11:35:40.113  [pool-2-thread-2] DEBUG mao.t5.Test:  1开始运行
2022-09-09  11:35:40.113  [pool-2-thread-3] DEBUG mao.t5.Test:  2开始运行
2022-09-09  11:35:40.113  [pool-2-thread-1] DEBUG mao.t5.Test:  0开始运行
2022-09-09  11:35:41.125  [pool-2-thread-1] DEBUG mao.t5.Test:  7开始运行
2022-09-09  11:35:41.125  [pool-2-thread-3] DEBUG mao.t5.Test:  6开始运行
2022-09-09  11:35:41.125  [pool-2-thread-2] DEBUG mao.t5.Test:  5开始运行
2022-09-09  11:35:41.125  [pool-2-thread-4] DEBUG mao.t5.Test:  4开始运行
2022-09-09  11:35:41.125  [main] DEBUG mao.t5.Test:  结果：0
2022-09-09  11:35:41.126  [main] DEBUG mao.t5.Test:  结果：1
2022-09-09  11:35:41.126  [main] DEBUG mao.t5.Test:  结果：2
2022-09-09  11:35:41.126  [main] DEBUG mao.t5.Test:  结果：3
2022-09-09  11:35:42.126  [pool-2-thread-1] DEBUG mao.t5.Test:  8开始运行
2022-09-09  11:35:42.140  [pool-2-thread-2] DEBUG mao.t5.Test:  9开始运行
2022-09-09  11:35:42.140  [main] DEBUG mao.t5.Test:  结果：4
2022-09-09  11:35:42.140  [pool-2-thread-4] DEBUG mao.t5.Test:  10开始运行
2022-09-09  11:35:42.140  [pool-2-thread-3] DEBUG mao.t5.Test:  11开始运行
2022-09-09  11:35:42.140  [main] DEBUG mao.t5.Test:  结果：5
2022-09-09  11:35:42.140  [main] DEBUG mao.t5.Test:  结果：6
2022-09-09  11:35:42.140  [main] DEBUG mao.t5.Test:  结果：7
2022-09-09  11:35:43.136  [pool-2-thread-1] DEBUG mao.t5.Test:  12开始运行
2022-09-09  11:35:43.136  [main] DEBUG mao.t5.Test:  结果：8
2022-09-09  11:35:43.152  [pool-2-thread-3] DEBUG mao.t5.Test:  15开始运行
2022-09-09  11:35:43.152  [main] DEBUG mao.t5.Test:  结果：9
2022-09-09  11:35:43.152  [pool-2-thread-2] DEBUG mao.t5.Test:  14开始运行
2022-09-09  11:35:43.152  [pool-2-thread-4] DEBUG mao.t5.Test:  13开始运行
2022-09-09  11:35:43.152  [main] DEBUG mao.t5.Test:  结果：10
2022-09-09  11:35:43.152  [main] DEBUG mao.t5.Test:  结果：11
2022-09-09  11:35:44.148  [main] DEBUG mao.t5.Test:  结果：12
2022-09-09  11:35:44.148  [pool-2-thread-1] DEBUG mao.t5.Test:  16开始运行
2022-09-09  11:35:44.163  [pool-2-thread-3] DEBUG mao.t5.Test:  17开始运行
2022-09-09  11:35:44.163  [main] DEBUG mao.t5.Test:  结果：13
2022-09-09  11:35:44.163  [pool-2-thread-2] DEBUG mao.t5.Test:  18开始运行
2022-09-09  11:35:44.163  [pool-2-thread-4] DEBUG mao.t5.Test:  19开始运行
2022-09-09  11:35:44.163  [main] DEBUG mao.t5.Test:  结果：14
2022-09-09  11:35:44.163  [main] DEBUG mao.t5.Test:  结果：15
2022-09-09  11:35:45.156  [main] DEBUG mao.t5.Test:  结果：16
2022-09-09  11:35:45.163  [main] DEBUG mao.t5.Test:  结果：17
2022-09-09  11:35:45.172  [main] DEBUG mao.t5.Test:  结果：18
2022-09-09  11:35:45.172  [main] DEBUG mao.t5.Test:  结果：19
```





#### invokeAll



```java
//执行给定的任务，返回一个 Futures 列表，在所有完成时保存它们的状态和结果。 
// Future.isDone对于返回列表的每个元素都是true 。
// 请注意，已完成的任务可能已经正常终止，也可能通过引发异常终止。
// 如果在此操作进行时修改了给定的集合，则此方法的结果是不确定的
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
 throws InterruptedException;
// 提交 tasks 中所有任务，带超时时间
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
 long timeout, TimeUnit unit)
 throws InterruptedException;
```



```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:36
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        List<Callable<Integer>> tasks = new ArrayList<>();

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;

            tasks.add(new Callable<Integer>()
            {
                @Override
                public Integer call() throws Exception
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    return finalI;
                }
            });

        }

        List<Future<Integer>> futures = threadPool.invokeAll(tasks);
        for (Future<Integer> future : futures)
        {
            Integer integer = future.get();
            log.debug("结果：" + integer);
        }
    }
}
```



```sh
2022-09-09  11:39:45.288  [pool-2-thread-1] DEBUG mao.t6.Test:  0开始运行
2022-09-09  11:39:45.288  [pool-2-thread-3] DEBUG mao.t6.Test:  2开始运行
2022-09-09  11:39:45.288  [pool-2-thread-2] DEBUG mao.t6.Test:  1开始运行
2022-09-09  11:39:45.288  [pool-2-thread-4] DEBUG mao.t6.Test:  3开始运行
2022-09-09  11:39:46.294  [pool-2-thread-1] DEBUG mao.t6.Test:  6开始运行
2022-09-09  11:39:46.294  [pool-2-thread-3] DEBUG mao.t6.Test:  7开始运行
2022-09-09  11:39:46.294  [pool-2-thread-2] DEBUG mao.t6.Test:  5开始运行
2022-09-09  11:39:46.294  [pool-2-thread-4] DEBUG mao.t6.Test:  4开始运行
2022-09-09  11:39:47.295  [pool-2-thread-1] DEBUG mao.t6.Test:  8开始运行
2022-09-09  11:39:47.297  [pool-2-thread-4] DEBUG mao.t6.Test:  9开始运行
2022-09-09  11:39:47.297  [pool-2-thread-3] DEBUG mao.t6.Test:  11开始运行
2022-09-09  11:39:47.297  [pool-2-thread-2] DEBUG mao.t6.Test:  10开始运行
2022-09-09  11:39:48.309  [pool-2-thread-1] DEBUG mao.t6.Test:  13开始运行
2022-09-09  11:39:48.309  [pool-2-thread-2] DEBUG mao.t6.Test:  12开始运行
2022-09-09  11:39:48.309  [pool-2-thread-3] DEBUG mao.t6.Test:  15开始运行
2022-09-09  11:39:48.309  [pool-2-thread-4] DEBUG mao.t6.Test:  14开始运行
2022-09-09  11:39:49.310  [pool-2-thread-1] DEBUG mao.t6.Test:  16开始运行
2022-09-09  11:39:49.313  [pool-2-thread-4] DEBUG mao.t6.Test:  17开始运行
2022-09-09  11:39:49.313  [pool-2-thread-2] DEBUG mao.t6.Test:  19开始运行
2022-09-09  11:39:49.313  [pool-2-thread-3] DEBUG mao.t6.Test:  18开始运行
2022-09-09  11:39:50.315  [main] DEBUG mao.t6.Test:  结果：0
2022-09-09  11:39:50.315  [main] DEBUG mao.t6.Test:  结果：1
2022-09-09  11:39:50.315  [main] DEBUG mao.t6.Test:  结果：2
2022-09-09  11:39:50.315  [main] DEBUG mao.t6.Test:  结果：3
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：4
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：5
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：6
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：7
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：8
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：9
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：10
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：11
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：12
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：13
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：14
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：15
2022-09-09  11:39:50.316  [main] DEBUG mao.t6.Test:  结果：16
2022-09-09  11:39:50.317  [main] DEBUG mao.t6.Test:  结果：17
2022-09-09  11:39:50.317  [main] DEBUG mao.t6.Test:  结果：18
2022-09-09  11:39:50.317  [main] DEBUG mao.t6.Test:  结果：19
```





带超时时间

```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t7
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:41
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        List<Callable<Integer>> tasks = new ArrayList<>();

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;

            tasks.add(new Callable<Integer>()
            {
                @Override
                public Integer call() throws Exception
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    return finalI;
                }
            });

        }


        List<Future<Integer>> futures = threadPool.invokeAll(tasks, 3, TimeUnit.SECONDS);
        for (Future<Integer> future : futures)
        {
            Integer integer = future.get();
            log.debug("结果：" + integer);
        }
    }
}
```



```sh
2022-09-09  11:42:57.676  [pool-2-thread-4] DEBUG mao.t7.Test:  3开始运行
2022-09-09  11:42:57.676  [pool-2-thread-1] DEBUG mao.t7.Test:  0开始运行
2022-09-09  11:42:57.676  [pool-2-thread-3] DEBUG mao.t7.Test:  2开始运行
2022-09-09  11:42:57.676  [pool-2-thread-2] DEBUG mao.t7.Test:  1开始运行
2022-09-09  11:42:58.682  [pool-2-thread-4] DEBUG mao.t7.Test:  4开始运行
2022-09-09  11:42:58.682  [pool-2-thread-2] DEBUG mao.t7.Test:  5开始运行
2022-09-09  11:42:58.682  [pool-2-thread-3] DEBUG mao.t7.Test:  6开始运行
2022-09-09  11:42:58.682  [pool-2-thread-1] DEBUG mao.t7.Test:  7开始运行
2022-09-09  11:42:59.685  [pool-2-thread-2] DEBUG mao.t7.Test:  8开始运行
2022-09-09  11:42:59.690  [pool-2-thread-1] DEBUG mao.t7.Test:  9开始运行
2022-09-09  11:42:59.690  [pool-2-thread-3] DEBUG mao.t7.Test:  11开始运行
2022-09-09  11:42:59.690  [pool-2-thread-4] DEBUG mao.t7.Test:  10开始运行
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t7.Test$1.call(Test.java:48)
	at mao.t7.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t7.Test$1.call(Test.java:48)
	at mao.t7.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t7.Test$1.call(Test.java:48)
	at mao.t7.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t7.Test$1.call(Test.java:48)
	at mao.t7.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
Exception in thread "main" java.util.concurrent.CancellationException
	at java.base/java.util.concurrent.FutureTask.report(FutureTask.java:121)
	at java.base/java.util.concurrent.FutureTask.get(FutureTask.java:191)
	at mao.t7.Test.main(Test.java:64)
2022-09-09  11:43:00.680  [main] DEBUG mao.t7.Test:  结果：0
2022-09-09  11:43:00.680  [main] DEBUG mao.t7.Test:  结果：1
2022-09-09  11:43:00.680  [main] DEBUG mao.t7.Test:  结果：2
2022-09-09  11:43:00.681  [main] DEBUG mao.t7.Test:  结果：3
2022-09-09  11:43:00.681  [main] DEBUG mao.t7.Test:  结果：4
2022-09-09  11:43:00.681  [main] DEBUG mao.t7.Test:  结果：5
2022-09-09  11:43:00.681  [main] DEBUG mao.t7.Test:  结果：6
2022-09-09  11:43:00.681  [main] DEBUG mao.t7.Test:  结果：7
```







#### invokeAny



```java
// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
 throws InterruptedException, ExecutionException;

//提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消，带超时时间
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
 long timeout, TimeUnit unit)
 throws InterruptedException, ExecutionException, TimeoutException;
```





```java
package mao.t8;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t8
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 11:44
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        List<Callable<Integer>> tasks = new ArrayList<>();

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;

            tasks.add(new Callable<Integer>()
            {
                @Override
                public Integer call() throws Exception
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    return finalI;
                }
            });

        }

        //执行给定任务，返回已成功完成的任务的结果（即，不抛出异常），如果有的话。
        //正常或异常返回时，取消未完成的任务。如果在此操作进行时修改了给定的集合，则此方法的结果是不确定的
        Integer integer = threadPool.invokeAny(tasks);
        log.debug("结果：" + integer);
    }
}
```



```sh
2022-09-09  11:48:11.396  [pool-2-thread-3] DEBUG mao.t8.Test:  2开始运行
2022-09-09  11:48:11.396  [pool-2-thread-1] DEBUG mao.t8.Test:  0开始运行
2022-09-09  11:48:11.396  [pool-2-thread-2] DEBUG mao.t8.Test:  1开始运行
2022-09-09  11:48:11.396  [pool-2-thread-4] DEBUG mao.t8.Test:  3开始运行
2022-09-09  11:48:12.410  [pool-2-thread-4] DEBUG mao.t8.Test:  4开始运行
2022-09-09  11:48:12.410  [pool-2-thread-3] DEBUG mao.t8.Test:  6开始运行
2022-09-09  11:48:12.410  [pool-2-thread-1] DEBUG mao.t8.Test:  5开始运行
2022-09-09  11:48:12.410  [pool-2-thread-2] DEBUG mao.t8.Test:  7开始运行
2022-09-09  11:48:12.411  [main] DEBUG mao.t8.Test:  结果：2
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t8.Test$1.call(Test.java:48)
	at mao.t8.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t8.Test$1.call(Test.java:48)
	at mao.t8.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t8.Test$1.call(Test.java:48)
	at mao.t8.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t8.Test$1.call(Test.java:48)
	at mao.t8.Test$1.call(Test.java:41)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
```







### 关闭线程池

#### shutdown



```java
//线程池状态变为 SHUTDOWN
//不会接收新任务
//但已提交任务会执行完
//此方法不会阻塞调用线程的执行
void shutdown();
```



源码：

```java

    /**
     * Initiates an orderly shutdown in which previously submitted
     * tasks are executed, but no new tasks will be accepted.
     * Invocation has no additional effect if already shut down.
     *
     * <p>This method does not wait for previously submitted tasks to
     * complete execution.  Use {@link #awaitTermination awaitTermination}
     * to do that.
     *
     * @throws SecurityException {@inheritDoc}
     */
    public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(SHUTDOWN);
            interruptIdleWorkers();
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }
```





```java
package mao.t10;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t10
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 12:27
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束运行");
                }
            });
        }

        Thread.sleep(2500);
        log.debug("开始关闭线程池");
        //启动有序关闭，其中执行先前提交的任务，但不会接受新任务。如果已经关闭，调用没有额外的效果。
        //此方法不等待先前提交的任务完成执行。使用awaitTermination来做到这一点
        threadPool.shutdown();

    }
}
```



```sh
2022-09-09  12:38:58.158  [pool-2-thread-1] DEBUG mao.t10.Test:  0开始运行
2022-09-09  12:38:58.158  [pool-2-thread-4] DEBUG mao.t10.Test:  3开始运行
2022-09-09  12:38:58.158  [pool-2-thread-2] DEBUG mao.t10.Test:  1开始运行
2022-09-09  12:38:58.158  [pool-2-thread-3] DEBUG mao.t10.Test:  2开始运行
2022-09-09  12:38:59.162  [pool-2-thread-1] DEBUG mao.t10.Test:  0结束运行
2022-09-09  12:38:59.162  [pool-2-thread-4] DEBUG mao.t10.Test:  3结束运行
2022-09-09  12:38:59.162  [pool-2-thread-2] DEBUG mao.t10.Test:  1结束运行
2022-09-09  12:38:59.162  [pool-2-thread-3] DEBUG mao.t10.Test:  2结束运行
2022-09-09  12:38:59.162  [pool-2-thread-1] DEBUG mao.t10.Test:  4开始运行
2022-09-09  12:38:59.162  [pool-2-thread-2] DEBUG mao.t10.Test:  6开始运行
2022-09-09  12:38:59.162  [pool-2-thread-4] DEBUG mao.t10.Test:  5开始运行
2022-09-09  12:38:59.162  [pool-2-thread-3] DEBUG mao.t10.Test:  7开始运行
2022-09-09  12:39:00.167  [pool-2-thread-2] DEBUG mao.t10.Test:  6结束运行
2022-09-09  12:39:00.167  [pool-2-thread-3] DEBUG mao.t10.Test:  7结束运行
2022-09-09  12:39:00.167  [pool-2-thread-1] DEBUG mao.t10.Test:  4结束运行
2022-09-09  12:39:00.167  [pool-2-thread-4] DEBUG mao.t10.Test:  5结束运行
2022-09-09  12:39:00.167  [pool-2-thread-2] DEBUG mao.t10.Test:  8开始运行
2022-09-09  12:39:00.167  [pool-2-thread-3] DEBUG mao.t10.Test:  9开始运行
2022-09-09  12:39:00.167  [pool-2-thread-4] DEBUG mao.t10.Test:  11开始运行
2022-09-09  12:39:00.167  [pool-2-thread-1] DEBUG mao.t10.Test:  10开始运行
2022-09-09  12:39:00.666  [main] DEBUG mao.t10.Test:  开始关闭线程池
2022-09-09  12:39:01.182  [pool-2-thread-3] DEBUG mao.t10.Test:  9结束运行
2022-09-09  12:39:01.182  [pool-2-thread-4] DEBUG mao.t10.Test:  11结束运行
2022-09-09  12:39:01.182  [pool-2-thread-1] DEBUG mao.t10.Test:  10结束运行
2022-09-09  12:39:01.182  [pool-2-thread-2] DEBUG mao.t10.Test:  8结束运行
2022-09-09  12:39:01.182  [pool-2-thread-2] DEBUG mao.t10.Test:  13开始运行
2022-09-09  12:39:01.182  [pool-2-thread-1] DEBUG mao.t10.Test:  14开始运行
2022-09-09  12:39:01.182  [pool-2-thread-4] DEBUG mao.t10.Test:  12开始运行
2022-09-09  12:39:01.182  [pool-2-thread-3] DEBUG mao.t10.Test:  15开始运行
2022-09-09  12:39:02.190  [pool-2-thread-1] DEBUG mao.t10.Test:  14结束运行
2022-09-09  12:39:02.190  [pool-2-thread-2] DEBUG mao.t10.Test:  13结束运行
2022-09-09  12:39:02.190  [pool-2-thread-3] DEBUG mao.t10.Test:  15结束运行
2022-09-09  12:39:02.190  [pool-2-thread-4] DEBUG mao.t10.Test:  12结束运行
2022-09-09  12:39:02.190  [pool-2-thread-4] DEBUG mao.t10.Test:  19开始运行
2022-09-09  12:39:02.190  [pool-2-thread-1] DEBUG mao.t10.Test:  18开始运行
2022-09-09  12:39:02.190  [pool-2-thread-2] DEBUG mao.t10.Test:  16开始运行
2022-09-09  12:39:02.190  [pool-2-thread-3] DEBUG mao.t10.Test:  17开始运行
2022-09-09  12:39:03.203  [pool-2-thread-1] DEBUG mao.t10.Test:  18结束运行
2022-09-09  12:39:03.203  [pool-2-thread-2] DEBUG mao.t10.Test:  16结束运行
2022-09-09  12:39:03.203  [pool-2-thread-3] DEBUG mao.t10.Test:  17结束运行
2022-09-09  12:39:03.203  [pool-2-thread-4] DEBUG mao.t10.Test:  19结束运行
```





```java
package mao.t11;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.RejectedExecutionException;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t11
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 12:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束运行");
                }
            });
        }

        Thread.sleep(2500);
        log.debug("开始关闭线程池");
        //启动有序关闭，其中执行先前提交的任务，但不会接受新任务。如果已经关闭，调用没有额外的效果。
        //此方法不等待先前提交的任务完成执行。使用awaitTermination来做到这一点
        threadPool.shutdown();


        for (int i = 20; i < 23; i++)
        {
            try
            {
                int finalI = i;
                threadPool.submit(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        log.debug(finalI + "开始运行");
                        try
                        {
                            Thread.sleep(1000);
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                        log.debug(finalI + "结束运行");
                    }
                });
            }
            catch (RejectedExecutionException e)
            {
                e.printStackTrace();
            }
        }
    }
}
```



```sh
2022-09-09  12:45:02.096  [pool-2-thread-2] DEBUG mao.t11.Test:  1开始运行
2022-09-09  12:45:02.096  [pool-2-thread-4] DEBUG mao.t11.Test:  3开始运行
2022-09-09  12:45:02.096  [pool-2-thread-3] DEBUG mao.t11.Test:  2开始运行
2022-09-09  12:45:02.096  [pool-2-thread-1] DEBUG mao.t11.Test:  0开始运行
2022-09-09  12:45:03.108  [pool-2-thread-4] DEBUG mao.t11.Test:  3结束运行
2022-09-09  12:45:03.108  [pool-2-thread-3] DEBUG mao.t11.Test:  2结束运行
2022-09-09  12:45:03.108  [pool-2-thread-2] DEBUG mao.t11.Test:  1结束运行
2022-09-09  12:45:03.108  [pool-2-thread-1] DEBUG mao.t11.Test:  0结束运行
2022-09-09  12:45:03.108  [pool-2-thread-4] DEBUG mao.t11.Test:  4开始运行
2022-09-09  12:45:03.108  [pool-2-thread-1] DEBUG mao.t11.Test:  6开始运行
2022-09-09  12:45:03.108  [pool-2-thread-2] DEBUG mao.t11.Test:  5开始运行
2022-09-09  12:45:03.108  [pool-2-thread-3] DEBUG mao.t11.Test:  7开始运行
2022-09-09  12:45:04.111  [pool-2-thread-3] DEBUG mao.t11.Test:  7结束运行
2022-09-09  12:45:04.111  [pool-2-thread-4] DEBUG mao.t11.Test:  4结束运行
2022-09-09  12:45:04.111  [pool-2-thread-2] DEBUG mao.t11.Test:  5结束运行
2022-09-09  12:45:04.111  [pool-2-thread-1] DEBUG mao.t11.Test:  6结束运行
2022-09-09  12:45:04.111  [pool-2-thread-2] DEBUG mao.t11.Test:  9开始运行
2022-09-09  12:45:04.111  [pool-2-thread-3] DEBUG mao.t11.Test:  8开始运行
2022-09-09  12:45:04.111  [pool-2-thread-1] DEBUG mao.t11.Test:  11开始运行
2022-09-09  12:45:04.111  [pool-2-thread-4] DEBUG mao.t11.Test:  10开始运行
2022-09-09  12:45:04.596  [main] DEBUG mao.t11.Test:  开始关闭线程池
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@47987356[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@35aea049[Wrapped task = mao.t11.Test$2@7205765b]] rejected from java.util.concurrent.ThreadPoolExecutor@22ef9844[Shutting down, pool size = 4, active threads = 4, queued tasks = 8, completed tasks = 8]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2057)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:827)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1357)
	at java.base/java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:123)
	at mao.t11.Test.main(Test.java:68)
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@2f217633[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@1da2cb77[Wrapped task = mao.t11.Test$2@48f278eb]] rejected from java.util.concurrent.ThreadPoolExecutor@22ef9844[Shutting down, pool size = 4, active threads = 4, queued tasks = 8, completed tasks = 8]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2057)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:827)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1357)
	at java.base/java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:123)
	at mao.t11.Test.main(Test.java:68)
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@7e7be63f[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@1a18644[Wrapped task = mao.t11.Test$2@5acf93bb]] rejected from java.util.concurrent.ThreadPoolExecutor@22ef9844[Shutting down, pool size = 4, active threads = 4, queued tasks = 8, completed tasks = 8]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2057)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:827)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1357)
	at java.base/java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:123)
	at mao.t11.Test.main(Test.java:68)
2022-09-09  12:45:05.125  [pool-2-thread-4] DEBUG mao.t11.Test:  10结束运行
2022-09-09  12:45:05.125  [pool-2-thread-3] DEBUG mao.t11.Test:  8结束运行
2022-09-09  12:45:05.125  [pool-2-thread-1] DEBUG mao.t11.Test:  11结束运行
2022-09-09  12:45:05.125  [pool-2-thread-2] DEBUG mao.t11.Test:  9结束运行
2022-09-09  12:45:05.125  [pool-2-thread-3] DEBUG mao.t11.Test:  12开始运行
2022-09-09  12:45:05.125  [pool-2-thread-1] DEBUG mao.t11.Test:  13开始运行
2022-09-09  12:45:05.125  [pool-2-thread-2] DEBUG mao.t11.Test:  14开始运行
2022-09-09  12:45:05.125  [pool-2-thread-4] DEBUG mao.t11.Test:  15开始运行
2022-09-09  12:45:06.127  [pool-2-thread-2] DEBUG mao.t11.Test:  14结束运行
2022-09-09  12:45:06.127  [pool-2-thread-1] DEBUG mao.t11.Test:  13结束运行
2022-09-09  12:45:06.127  [pool-2-thread-4] DEBUG mao.t11.Test:  15结束运行
2022-09-09  12:45:06.127  [pool-2-thread-3] DEBUG mao.t11.Test:  12结束运行
2022-09-09  12:45:06.127  [pool-2-thread-2] DEBUG mao.t11.Test:  16开始运行
2022-09-09  12:45:06.127  [pool-2-thread-1] DEBUG mao.t11.Test:  17开始运行
2022-09-09  12:45:06.127  [pool-2-thread-4] DEBUG mao.t11.Test:  18开始运行
2022-09-09  12:45:06.127  [pool-2-thread-3] DEBUG mao.t11.Test:  19开始运行
2022-09-09  12:45:07.130  [pool-2-thread-3] DEBUG mao.t11.Test:  19结束运行
2022-09-09  12:45:07.130  [pool-2-thread-2] DEBUG mao.t11.Test:  16结束运行
2022-09-09  12:45:07.130  [pool-2-thread-4] DEBUG mao.t11.Test:  18结束运行
2022-09-09  12:45:07.130  [pool-2-thread-1] DEBUG mao.t11.Test:  17结束运行
```







#### shutdownNow



```java
//线程池状态变为 STOP
//不会接收新任务
//会将队列中的任务返回
//并用 interrupt 的方式中断正在执行的任务
List<Runnable> shutdownNow();
```



源码：

```java

    /**
     * Attempts to stop all actively executing tasks, halts the
     * processing of waiting tasks, and returns a list of the tasks
     * that were awaiting execution. These tasks are drained (removed)
     * from the task queue upon return from this method.
     *
     * <p>This method does not wait for actively executing tasks to
     * terminate.  Use {@link #awaitTermination awaitTermination} to
     * do that.
     *
     * <p>There are no guarantees beyond best-effort attempts to stop
     * processing actively executing tasks.  This implementation
     * interrupts tasks via {@link Thread#interrupt}; any task that
     * fails to respond to interrupts may never terminate.
     *
     * @throws SecurityException {@inheritDoc}
     */
    public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            advanceRunState(STOP);
            interruptWorkers();
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
    }
```





```java
package mao.t12;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.RejectedExecutionException;

/**
 * Project name(项目名称)：java并发编程_线程池
 * Package(包名): mao.t12
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 12:47
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws InterruptedException
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(4);

        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始运行");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束运行");
                }
            });
        }

        Thread.sleep(2500);
        log.debug("开始关闭线程池");
        //尝试停止所有正在执行的任务，停止等待任务的处理，并返回等待执行的任务列表。
        //此方法不等待主动执行的任务终止。使用awaitTermination来做到这一点。
        //除了尽最大努力停止处理正在执行的任务之外，没有任何保证。
        //例如，典型的实现将通过Thread.interrupt取消，因此任何未能响应中断的任务可能永远不会终止。
        List<Runnable> runnableList = threadPool.shutdownNow();

        for (int i = 20; i < 23; i++)
        {
            try
            {
                int finalI = i;
                threadPool.submit(new Runnable()
                {
                    @Override
                    public void run()
                    {
                        log.debug(finalI + "开始运行");
                        try
                        {
                            Thread.sleep(1000);
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                        log.debug(finalI + "结束运行");
                    }
                });
            }
            catch (RejectedExecutionException e)
            {
                e.printStackTrace();
            }
        }
        
        Thread.sleep(500);

        for (Runnable runnable : runnableList)
        {
            log.warn("未执行完成的任务：" + runnable);
        }
    }
}
```



```sh
2022-09-09  12:52:47.300  [pool-2-thread-4] DEBUG mao.t12.Test:  3开始运行
2022-09-09  12:52:47.300  [pool-2-thread-3] DEBUG mao.t12.Test:  2开始运行
2022-09-09  12:52:47.300  [pool-2-thread-1] DEBUG mao.t12.Test:  0开始运行
2022-09-09  12:52:47.300  [pool-2-thread-2] DEBUG mao.t12.Test:  1开始运行
2022-09-09  12:52:48.306  [pool-2-thread-1] DEBUG mao.t12.Test:  0结束运行
2022-09-09  12:52:48.306  [pool-2-thread-4] DEBUG mao.t12.Test:  3结束运行
2022-09-09  12:52:48.306  [pool-2-thread-3] DEBUG mao.t12.Test:  2结束运行
2022-09-09  12:52:48.306  [pool-2-thread-2] DEBUG mao.t12.Test:  1结束运行
2022-09-09  12:52:48.306  [pool-2-thread-1] DEBUG mao.t12.Test:  5开始运行
2022-09-09  12:52:48.306  [pool-2-thread-2] DEBUG mao.t12.Test:  4开始运行
2022-09-09  12:52:48.306  [pool-2-thread-3] DEBUG mao.t12.Test:  6开始运行
2022-09-09  12:52:48.307  [pool-2-thread-4] DEBUG mao.t12.Test:  7开始运行
2022-09-09  12:52:49.307  [pool-2-thread-1] DEBUG mao.t12.Test:  5结束运行
2022-09-09  12:52:49.307  [pool-2-thread-1] DEBUG mao.t12.Test:  8开始运行
2022-09-09  12:52:49.311  [pool-2-thread-3] DEBUG mao.t12.Test:  6结束运行
2022-09-09  12:52:49.311  [pool-2-thread-2] DEBUG mao.t12.Test:  4结束运行
2022-09-09  12:52:49.311  [pool-2-thread-3] DEBUG mao.t12.Test:  9开始运行
2022-09-09  12:52:49.311  [pool-2-thread-4] DEBUG mao.t12.Test:  7结束运行
2022-09-09  12:52:49.311  [pool-2-thread-2] DEBUG mao.t12.Test:  10开始运行
2022-09-09  12:52:49.311  [pool-2-thread-4] DEBUG mao.t12.Test:  11开始运行
2022-09-09  12:52:49.798  [main] DEBUG mao.t12.Test:  开始关闭线程池
2022-09-09  12:52:49.799  [pool-2-thread-1] DEBUG mao.t12.Test:  8结束运行
2022-09-09  12:52:49.799  [pool-2-thread-4] DEBUG mao.t12.Test:  11结束运行
2022-09-09  12:52:49.799  [pool-2-thread-2] DEBUG mao.t12.Test:  10结束运行
2022-09-09  12:52:49.799  [pool-2-thread-3] DEBUG mao.t12.Test:  9结束运行
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t12.Test$1.run(Test.java:46)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@47987356[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@35aea049[Wrapped task = mao.t12.Test$2@7205765b]] rejected from java.util.concurrent.ThreadPoolExecutor@22ef9844[Shutting down, pool size = 4, active threads = 4, queued tasks = 0, completed tasks = 8]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2057)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:827)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1357)
	at java.base/java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:123)
	at mao.t12.Test.main(Test.java:70)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t12.Test$1.run(Test.java:46)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t12.Test$1.run(Test.java:46)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at mao.t12.Test$1.run(Test.java:46)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@611889f4[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@3b6ddd1d[Wrapped task = mao.t12.Test$2@3f6b0be5]] rejected from java.util.concurrent.ThreadPoolExecutor@22ef9844[Shutting down, pool size = 3, active threads = 3, queued tasks = 0, completed tasks = 9]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2057)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:827)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1357)
	at java.base/java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:123)
	at mao.t12.Test.main(Test.java:70)
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.FutureTask@a530d0a[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@48f278eb[Wrapped task = mao.t12.Test$2@2f217633]] rejected from java.util.concurrent.ThreadPoolExecutor@22ef9844[Terminated, pool size = 0, active threads = 0, queued tasks = 0, completed tasks = 12]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2057)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:827)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1357)
	at java.base/java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:123)
	at mao.t12.Test.main(Test.java:70)
2022-09-09  12:52:50.301  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@6cd28fa7[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@5acf93bb[Wrapped task = mao.t12.Test$1@7e7be63f]]
2022-09-09  12:52:50.301  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@66d3eec0[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@614ca7df[Wrapped task = mao.t12.Test$1@4738a206]]
2022-09-09  12:52:50.301  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@18d87d80[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@1e04fa0a[Wrapped task = mao.t12.Test$1@1af2d44a]]
2022-09-09  12:52:50.301  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@543588e6[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@618425b5[Wrapped task = mao.t12.Test$1@58695725]]
2022-09-09  12:52:50.301  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@5d7148e2[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@f5acb9d[Wrapped task = mao.t12.Test$1@4fb3ee4e]]
2022-09-09  12:52:50.301  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@2c35e847[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@25fb8912[Wrapped task = mao.t12.Test$1@7c24b813]]
2022-09-09  12:52:50.302  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@5ba3f27a[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@7bd4937b[Wrapped task = mao.t12.Test$1@21e360a]]
2022-09-09  12:52:50.302  [main] WARN  mao.t12.Test:  未执行完成的任务：java.util.concurrent.FutureTask@741a8937[Not completed, task = java.util.concurrent.Executors$RunnableAdapter@58d75e99[Wrapped task = mao.t12.Test$1@74751b3]]
```





### 其它方法



```java
// 不在 RUNNING 状态的线程池，此方法就返回 true
boolean isShutdown();

// 线程池状态是否是 TERMINATED
boolean isTerminated();

// 调用 shutdown 后，由于调用线程并不会等待所有任务运行结束，因此如果它想在线程池 TERMINATED 后做些事情，可以利用此方法等待
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
```









### 异步模式之工作线程

#### 定义

让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现 就是线程池，也体现了经典设计模式中的享元模式



例如，海底捞的服务员（线程），轮流处理每位客人的点餐（任务），如果为每位客人都配一名专属的服务员，那 么成本就太高了（对比另一种多线程设计模式：Thread-Per-Message） 

注意，不同任务类型应该使用不同的线程池，这样能够避免饥饿，并能提升效率 

例如，如果一个餐馆的工人既要招呼客人（任务类型A），又要到后厨做菜（任务类型B）显然效率不咋地，分成 服务员（线程池A）与厨师（线程池B）更为合理，当然你能想到更细致的分工





#### 饥饿

固定大小线程池会有饥饿现象

* 两个工人是同一个线程池中的两个线程
* 他们要做的事情是：为客人点餐和到后厨做菜，这是两个阶段的工作
  * 客人点餐：必须先点完餐，等菜做好，上菜，在此期间处理点餐的工人必须等待
  * 后厨做菜
* 比如工人A 处理了点餐任务，接下来它要等着 工人B 把菜做好，然后上菜，他俩也配合的很好
* 但现在同时来了两个客人，这个时候工人A 和工人B 都去处理点餐了，这时没人做饭了，饥饿





```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_异步模式之工作线程
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 13:38
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        extracted(threadPool, "西红柿炒番茄");
    }

    /**
     * 提取
     *
     * @param threadPool 线程池
     * @param order      订单，菜名
     */
    private static void extracted(ExecutorService threadPool, String order)
    {
        threadPool.execute(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("点餐：" + order);
                Future<String> future = threadPool.submit(new Callable<String>()
                {
                    @Override
                    public String call() throws Exception
                    {
                        log.debug("做菜：" + order);
                        return order;
                    }
                });
                try
                {
                    log.debug("上菜：" + future.get());
                }
                catch (InterruptedException | ExecutionException e)
                {
                    e.printStackTrace();
                }
            }
        });
    }
}
```



运行结果：

```sh
2022-09-09  13:49:31.580  [pool-2-thread-1] DEBUG mao.t1.Test:  点餐：西红柿炒番茄
2022-09-09  13:49:31.584  [pool-2-thread-2] DEBUG mao.t1.Test:  做菜：西红柿炒番茄
2022-09-09  13:49:31.584  [pool-2-thread-1] DEBUG mao.t1.Test:  上菜：西红柿炒番茄
```





再添加一位客人

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_异步模式之工作线程
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 13:38
 * Version(版本): 1.0
 * Description(描述)： 饥饿现象
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        extracted(threadPool, "西红柿炒番茄");
        extracted(threadPool, "土豆炒马铃薯");
    }

    /**
     * 提取
     *
     * @param threadPool 线程池
     * @param order      订单，菜名
     */
    private static void extracted(ExecutorService threadPool, String order)
    {
        threadPool.execute(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("点餐：" + order);
                Future<String> future = threadPool.submit(new Callable<String>()
                {
                    @Override
                    public String call() throws Exception
                    {
                        log.debug("做菜：" + order);
                        return order;
                    }
                });
                try
                {
                    log.debug("上菜：" + future.get());
                }
                catch (InterruptedException | ExecutionException e)
                {
                    e.printStackTrace();
                }
            }
        });
    }
}
```



运行结果：

```sh
2022-09-09  13:51:15.229  [pool-2-thread-2] DEBUG mao.t1.Test:  点餐：土豆炒马铃薯
2022-09-09  13:51:15.229  [pool-2-thread-1] DEBUG mao.t1.Test:  点餐：西红柿炒番茄
//在这里卡着...
```



产生了饥饿现象





#### 解决饥饿现象

解决方法可以增加线程池的大小，不过不是根本解决方案。可以根据不同的任务类型，采用不同的线程池



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_异步模式之工作线程
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 13:54
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ExecutorService threadPool1 = Executors.newFixedThreadPool(1);
        ExecutorService threadPool2 = Executors.newFixedThreadPool(1);

        extracted(threadPool1, threadPool2, "西红柿炒番茄");
        extracted(threadPool1, threadPool2, "土豆炒马铃薯");
    }


    /**
     * 提取
     *
     * @param threadPool1 线程pool1
     * @param threadPool2 线程pool2
     * @param order       订单
     */
    private static void extracted(ExecutorService threadPool1, ExecutorService threadPool2, String order)
    {
        threadPool1.execute(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("点餐：" + order);
                Future<String> future = threadPool2.submit(new Callable<String>()
                {
                    @Override
                    public String call() throws Exception
                    {
                        log.debug("做菜：" + order);
                        return order;
                    }
                });
                try
                {
                    log.debug("上菜：" + future.get());
                }
                catch (InterruptedException | ExecutionException e)
                {
                    e.printStackTrace();
                }
            }
        });
    }
}
```



运行结果：

```sh
2022-09-09  13:57:59.529  [pool-2-thread-1] DEBUG mao.t2.Test:  点餐：西红柿炒番茄
2022-09-09  13:57:59.533  [pool-3-thread-1] DEBUG mao.t2.Test:  做菜：西红柿炒番茄
2022-09-09  13:57:59.534  [pool-2-thread-1] DEBUG mao.t2.Test:  上菜：西红柿炒番茄
2022-09-09  13:57:59.534  [pool-2-thread-1] DEBUG mao.t2.Test:  点餐：土豆炒马铃薯
2022-09-09  13:57:59.535  [pool-3-thread-1] DEBUG mao.t2.Test:  做菜：土豆炒马铃薯
2022-09-09  13:57:59.535  [pool-2-thread-1] DEBUG mao.t2.Test:  上菜：土豆炒马铃薯
```







### 创建多少线程池合适

* 过小会导致程序不能充分地利用系统资源、容易导致饥饿
* 过大会导致更多的线程上下文切换，占用更多内存



####  CPU 密集型运算

通常采用 cpu 核数 + 1 能够实现最优的 CPU 利用率，+1 是保证当线程由于页缺失故障（操作系统）或其它原因 导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费



#### I/O 密集型运算

CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程 RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率



**公式如下：**

**线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间**



例如 4 核 CPU 计算时间是 50% ，其它等待时间是 50%，期望 cpu 被 100% 利用，套用公式

**4 * 100% * 100% / 50% = 8**

例如 4 核 CPU 计算时间是 10% ，其它等待时间是 90%，期望 cpu 被 100% 利用，套用公式

**4 * 100% * 100% / 10% = 40**







### 任务调度线程池

#### Timer

可以使用 java.util.Timer 来实现定时功能，Timer 的优点在于简单易用，但 由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个 任务的延迟或异常都将会影响到之后的任务



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Timer;
import java.util.TimerTask;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:18
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        Timer timer = new Timer();

        TimerTask timerTask1 = new TimerTask()
        {
            @Override
            public void run()
            {
                log.debug("timerTask1开始");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                log.debug("timerTask1结束");
            }
        };

        TimerTask timerTask2 = new TimerTask()
        {
            @Override
            public void run()
            {
                log.debug("timerTask2开始");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                log.debug("timerTask2结束");
            }
        };

        timer.schedule(timerTask1,1000);
        timer.schedule(timerTask2,1000);
    }
}
```



运行结果：

```sh
2022-09-09  17:21:34.799  [Timer-0] DEBUG mao.t1.Test:  timerTask1开始
2022-09-09  17:21:35.813  [Timer-0] DEBUG mao.t1.Test:  timerTask1结束
2022-09-09  17:21:35.813  [Timer-0] DEBUG mao.t1.Test:  timerTask2开始
2022-09-09  17:21:36.827  [Timer-0] DEBUG mao.t1.Test:  timerTask2结束
```





#### ScheduledExecutorService

使用 ScheduledExecutorService



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:23
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            threadPool.schedule(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束");
                }
            }, 1, TimeUnit.SECONDS);
        }

    }
}
```



运行结果：

```sh
2022-09-09  17:28:21.926  [main] DEBUG mao.t2.Test:  开始运行
2022-09-09  17:28:22.939  [pool-2-thread-1] DEBUG mao.t2.Test:  0开始
2022-09-09  17:28:22.939  [pool-2-thread-2] DEBUG mao.t2.Test:  1开始
2022-09-09  17:28:23.952  [pool-2-thread-1] DEBUG mao.t2.Test:  0结束
2022-09-09  17:28:23.952  [pool-2-thread-2] DEBUG mao.t2.Test:  1结束
```



任务数量更改为3

```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:23
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 3; i++)
        {
            int finalI = i;
            threadPool.schedule(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束");
                }
            }, 1, TimeUnit.SECONDS);
        }

    }
}
```



运行结果：

```sh
2022-09-09  17:29:07.307  [main] DEBUG mao.t2.Test:  开始运行
2022-09-09  17:29:08.320  [pool-2-thread-1] DEBUG mao.t2.Test:  0开始
2022-09-09  17:29:08.320  [pool-2-thread-2] DEBUG mao.t2.Test:  1开始
2022-09-09  17:29:09.334  [pool-2-thread-2] DEBUG mao.t2.Test:  1结束
2022-09-09  17:29:09.334  [pool-2-thread-1] DEBUG mao.t2.Test:  0结束
2022-09-09  17:29:09.334  [pool-2-thread-2] DEBUG mao.t2.Test:  2开始
2022-09-09  17:29:10.340  [pool-2-thread-2] DEBUG mao.t2.Test:  2结束
```





**scheduleAtFixedRate方法**



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:32
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            //提交在给定初始延迟后首先启用的周期性操作，随后在给定周期内启用；
            //也就是说，执行将在initialDelay之后开始，
            //然后是initialDelay + period ，然后是initialDelay + 2 * period ，依此类推。
            //任务执行的顺序无限期地继续，直到发生以下异常完成之一：
            //该任务通过返回的未来显式取消。
            //执行器终止，也导致任务取消。
            //任务的执行会引发异常。在这种情况下，在返回的 future 上调用get将抛出ExecutionException ，并将异常作为其原因。
            //随后的执行被禁止。在返回的未来上对isDone()的后续调用将返回true 。
            //如果此任务的任何执行时间超过其周期，则后续执行可能会延迟开始，但不会同时执行
            //参数：
            //command - 要执行的任务
            //initialDelay – 延迟首次执行的时间
            //period – 连续执行之间的时间段
            //unit – initialDelay 和 period 参数的时间单位
            threadPool.scheduleAtFixedRate(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束");
                }
            }, 1, 2, TimeUnit.SECONDS);
        }
    }
}
```



运行结果：

```sh
2022-09-09  17:35:12.231  [main] DEBUG mao.t3.Test:  开始运行
2022-09-09  17:35:13.242  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:13.242  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:14.252  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:14.252  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:15.247  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:15.247  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:16.255  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:16.255  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:17.248  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:17.248  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:18.254  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:18.254  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:19.240  [pool-2-thread-2] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:19.240  [pool-2-thread-1] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:20.255  [pool-2-thread-2] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:20.255  [pool-2-thread-1] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:21.235  [pool-2-thread-1] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:21.235  [pool-2-thread-2] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:22.245  [pool-2-thread-2] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:22.245  [pool-2-thread-1] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:23.242  [pool-2-thread-1] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:23.242  [pool-2-thread-2] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:24.243  [pool-2-thread-1] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:24.249  [pool-2-thread-2] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:25.239  [pool-2-thread-1] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:25.239  [pool-2-thread-2] DEBUG mao.t3.Test:  0开始
2022-09-09  17:35:26.247  [pool-2-thread-1] DEBUG mao.t3.Test:  1结束
2022-09-09  17:35:26.247  [pool-2-thread-2] DEBUG mao.t3.Test:  0结束
2022-09-09  17:35:27.242  [pool-2-thread-1] DEBUG mao.t3.Test:  1开始
2022-09-09  17:35:27.242  [pool-2-thread-2] DEBUG mao.t3.Test:  0开始
```



更改休眠时间，任务执行时间超过了间隔时间

```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:32
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            //提交在给定初始延迟后首先启用的周期性操作，随后在给定周期内启用；
            //也就是说，执行将在initialDelay之后开始，
            //然后是initialDelay + period ，然后是initialDelay + 2 * period ，依此类推。
            //任务执行的顺序无限期地继续，直到发生以下异常完成之一：
            //该任务通过返回的未来显式取消。
            //执行器终止，也导致任务取消。
            //任务的执行会引发异常。在这种情况下，在返回的 future 上调用get将抛出ExecutionException ，并将异常作为其原因。
            //随后的执行被禁止。在返回的未来上对isDone()的后续调用将返回true 。
            //如果此任务的任何执行时间超过其周期，则后续执行可能会延迟开始，但不会同时执行
            //参数：
            //command - 要执行的任务
            //initialDelay – 延迟首次执行的时间
            //period – 连续执行之间的时间段
            //unit – initialDelay 和 period 参数的时间单位
            threadPool.scheduleAtFixedRate(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始");
                    try
                    {
                        Thread.sleep(3000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束");
                }
            }, 1, 2, TimeUnit.SECONDS);
        }
    }
}
```



运行结果：

```sh
2022-09-09  17:36:55.138  [main] DEBUG mao.t3.Test:  开始运行
2022-09-09  17:36:56.146  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:36:56.146  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:36:59.152  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:36:59.152  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:36:59.152  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:36:59.152  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:37:02.158  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:37:02.158  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:37:02.158  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:37:02.158  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:37:05.172  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:37:05.172  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:37:05.172  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:37:05.172  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:37:08.185  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:37:08.185  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:37:08.185  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
2022-09-09  17:37:08.185  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:37:11.200  [pool-2-thread-2] DEBUG mao.t3.Test:  1结束
2022-09-09  17:37:11.200  [pool-2-thread-1] DEBUG mao.t3.Test:  0结束
2022-09-09  17:37:11.200  [pool-2-thread-2] DEBUG mao.t3.Test:  1开始
2022-09-09  17:37:11.200  [pool-2-thread-1] DEBUG mao.t3.Test:  0开始
```



间隔被撑到了3s，并且两次之间无时间间隙



**scheduleWithFixedDelay方法**



```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:41
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            //提交一个周期性操作，该操作首先在给定的初始延迟之后启用，随后在一个执行的终止和下一个执行的开始之间具有给定的延迟。
            //任务执行的顺序无限期地继续，直到发生以下异常完成之一：
            //该任务通过返回的未来显式取消。
            //执行器终止，也导致任务取消。
            //任务的执行会引发异常。在这种情况下，在返回的 future 上调用get将抛出ExecutionException ，并将异常作为其原因。
            //随后的执行被禁止。在返回的未来上对isDone()的后续调用将返回true
            //参数：
            //command - 要执行的任务
            //initialDelay – 延迟首次执行的时间
            //延迟 - 一个执行的终止和下一个执行的开始之间的延迟
            //unit – initialDelay 和 delay 参数的时间单位
            threadPool.scheduleWithFixedDelay(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束");
                }
            }, 1, 2, TimeUnit.SECONDS);
        }
    }
}
```



运行结果：

```sh
2022-09-09  17:43:42.208  [main] DEBUG mao.t4.Test:  开始运行
2022-09-09  17:43:43.213  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:43:43.213  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:43:44.219  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:43:44.219  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:43:46.232  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:43:46.232  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:43:47.238  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:43:47.238  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:43:49.251  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:43:49.251  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:43:50.265  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:43:50.265  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
```



更改休眠时间

```java
package mao.t4;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t4
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 17:41
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            //提交一个周期性操作，该操作首先在给定的初始延迟之后启用，随后在一个执行的终止和下一个执行的开始之间具有给定的延迟。
            //任务执行的顺序无限期地继续，直到发生以下异常完成之一：
            //该任务通过返回的未来显式取消。
            //执行器终止，也导致任务取消。
            //任务的执行会引发异常。在这种情况下，在返回的 future 上调用get将抛出ExecutionException ，并将异常作为其原因。
            //随后的执行被禁止。在返回的未来上对isDone()的后续调用将返回true
            //参数：
            //command - 要执行的任务
            //initialDelay – 延迟首次执行的时间
            //延迟 - 一个执行的终止和下一个执行的开始之间的延迟
            //unit – initialDelay 和 delay 参数的时间单位
            threadPool.scheduleWithFixedDelay(new Runnable()
            {
                @Override
                public void run()
                {
                    log.debug(finalI + "开始");
                    try
                    {
                        Thread.sleep(3000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    log.debug(finalI + "结束");
                }
            }, 1, 2, TimeUnit.SECONDS);
        }
    }
}
```



运行结果：

```sh
2022-09-09  17:45:11.100  [main] DEBUG mao.t4.Test:  开始运行
2022-09-09  17:45:12.115  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:45:12.115  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:45:15.125  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:45:15.125  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:45:17.136  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:45:17.136  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:45:20.142  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:45:20.142  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:45:22.147  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:45:22.147  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:45:25.157  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:45:25.157  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:45:27.164  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:45:27.164  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:45:30.166  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:45:30.166  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:45:32.174  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:45:32.174  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
2022-09-09  17:45:35.175  [pool-2-thread-1] DEBUG mao.t4.Test:  0结束
2022-09-09  17:45:35.175  [pool-2-thread-2] DEBUG mao.t4.Test:  1结束
2022-09-09  17:45:37.179  [pool-2-thread-2] DEBUG mao.t4.Test:  1开始
2022-09-09  17:45:37.179  [pool-2-thread-1] DEBUG mao.t4.Test:  0开始
```







### 处理执行任务异常



```java
package mao.t5;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t5
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 20:04
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            threadPool.scheduleWithFixedDelay(new Runnable()
            {
                @Override
                @SuppressWarnings("all")
                public void run()
                {
                    log.debug(finalI + "开始");
                    int j = 1 / 0;
                    log.debug(finalI + "结束");
                }
            }, 1, 2, TimeUnit.SECONDS);
        }
    }
}
```



运行结果：

```sh
2022-09-09  20:05:35.608  [main] DEBUG mao.t5.Test:  开始运行
2022-09-09  20:05:36.620  [pool-2-thread-1] DEBUG mao.t5.Test:  0开始
2022-09-09  20:05:36.620  [pool-2-thread-2] DEBUG mao.t5.Test:  1开始
```



并没有抛出异常



#### 方法1 主动捉异常



```java
package mao.t6;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t6
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 20:07
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        log.debug("开始运行");
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            threadPool.scheduleWithFixedDelay(new Runnable()
            {
                @Override
                @SuppressWarnings("all")
                public void run()
                {
                    try
                    {
                        log.debug(finalI + "开始");
                        int j = 1 / 0;
                        log.debug(finalI + "结束");
                    }
                    catch (Exception e)
                    {
                        log.error("产生异常！异常内容：", e);
                    }
                }
            }, 1, 2, TimeUnit.SECONDS);
        }
    }
}
```



运行结果：

```sh
2022-09-09  20:08:59.814  [main] DEBUG mao.t6.Test:  开始运行
2022-09-09  20:09:00.821  [pool-2-thread-2] DEBUG mao.t6.Test:  1开始
2022-09-09  20:09:00.821  [pool-2-thread-1] DEBUG mao.t6.Test:  0开始
2022-09-09  20:09:00.822  [pool-2-thread-1] ERROR mao.t6.Test:  产生异常！异常内容：
java.lang.ArithmeticException: / by zero
	at mao.t6.Test$1.run(Test.java:47) [classes/:?]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) [?:?]
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:305) [?:?]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:305) [?:?]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130) [?:?]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630) [?:?]
	at java.lang.Thread.run(Thread.java:831) [?:?]
2022-09-09  20:09:00.822  [pool-2-thread-2] ERROR mao.t6.Test:  产生异常！异常内容：
java.lang.ArithmeticException: / by zero
	at mao.t6.Test$1.run(Test.java:47) [classes/:?]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) [?:?]
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:305) [?:?]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:305) [?:?]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130) [?:?]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630) [?:?]
	at java.lang.Thread.run(Thread.java:831) [?:?]
2022-09-09  20:09:02.838  [pool-2-thread-1] DEBUG mao.t6.Test:  0开始
2022-09-09  20:09:02.838  [pool-2-thread-2] DEBUG mao.t6.Test:  1开始
2022-09-09  20:09:02.838  [pool-2-thread-2] ERROR mao.t6.Test:  产生异常！异常内容：
java.lang.ArithmeticException: / by zero
	at mao.t6.Test$1.run(Test.java:47) [classes/:?]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) [?:?]
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:305) [?:?]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:305) [?:?]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130) [?:?]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630) [?:?]
	at java.lang.Thread.run(Thread.java:831) [?:?]
2022-09-09  20:09:02.838  [pool-2-thread-1] ERROR mao.t6.Test:  产生异常！异常内容：
java.lang.ArithmeticException: / by zero
	at mao.t6.Test$1.run(Test.java:47) [classes/:?]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515) [?:?]
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:305) [?:?]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:305) [?:?]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130) [?:?]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630) [?:?]
	at java.lang.Thread.run(Thread.java:831) [?:?]
```





#### 方法2 使用 Future



```java
package mao.t7;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.*;

/**
 * Project name(项目名称)：java并发编程_任务调度线程池
 * Package(包名): mao.t7
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/9
 * Time(创建时间)： 20:10
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(mao.t5.Test.class);

    public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        log.debug("开始运行");
        ExecutorService threadPool = Executors.newFixedThreadPool(1);

        for (int i = 0; i < 2; i++)
        {
            int finalI = i;
            Future<?> future = threadPool.submit(new Runnable()
            {
                @Override
                @SuppressWarnings("all")
                public void run()
                {
                    log.debug(finalI + "开始");
                    int j = 1 / 0;
                    log.debug(finalI + "结束");
                }
            });
            future.get();
        }
    }
}
```



运行结果：

```sh
2022-09-09  20:17:24.921  [main] DEBUG mao.t5.Test:  开始运行
2022-09-09  20:17:24.925  [pool-2-thread-1] DEBUG mao.t5.Test:  0开始
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
	at java.base/java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.base/java.util.concurrent.FutureTask.get(FutureTask.java:191)
	at mao.t7.Test.main(Test.java:47)
Caused by: java.lang.ArithmeticException: / by zero
	at mao.t7.Test$1.run(Test.java:43)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:630)
	at java.base/java.lang.Thread.run(Thread.java:831)
```







### 定时任务

```java
package mao.t1;

import java.time.DayOfWeek;
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Project name(项目名称)：java并发编程_定时任务
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/10
 * Time(创建时间)： 12:29
 * Version(版本): 1.0
 * Description(描述)： 何让每周四 18:00:00 定时执行任务
 */

public class Test
{
    public static void main(String[] args)
    {
        //当前时间
        LocalDateTime now = LocalDateTime.now();
        //这周周四 18:00:00的时间
        LocalDateTime thursday = now.with(DayOfWeek.THURSDAY).withHour(18).withMinute(0).withSecond(0).withNano(0);
        //判断当前时间是否已经超过了本周的星期四的18:00:00
        if (now.isAfter(thursday))
        {
            //下一周
            thursday = thursday.plusWeeks(1);
        }
        //计算时间差，即延时执行时间
        long initialDelay = Duration.between(now, thursday).toMillis();
        //一周的时间间隔
        long oneWeek = 7 * 24 * 3600 * 1000;
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(2);
        threadPool.scheduleAtFixedRate(new Runnable()
        {
            @Override
            public void run()
            {
                System.out.println("运行");
            }
        }, initialDelay, oneWeek, TimeUnit.MILLISECONDS);
    }
}
```







###  Tomcat 线程池

* LimitLatch 用来限流，可以控制最大连接个数
* Acceptor 只负责接收新的 socket 连接
* Poller 只负责监听 socket channel 是否有可读的 I/O 事件
* 一旦可读，封装一个任务对象（socketProcessor），提交给 Executor 线程池处理
* Executor 线程池中的工作线程最终负责处理请求



Tomcat 线程池扩展了 ThreadPoolExecutor，行为稍有不同

* 如果总线程数达到 maximumPoolSize
  * 这时不会立刻抛 RejectedExecutionException 异常
  * 而是再次尝试将任务放入队列，如果还失败，才抛出 RejectedExecutionException 异常





```java
/**
 * 执行
 *
 * @param command 命令
 * @param timeout 超时
 * @param unit    单位
 */
public void execute(Runnable command, long timeout, TimeUnit unit)
{
    submittedCount.incrementAndGet();
    try
    {
        super.execute(command);
    }
    catch (RejectedExecutionException rx)
    {
        if (super.getQueue() instanceof TaskQueue)
        {
            final TaskQueue queue = (TaskQueue) super.getQueue();
            try
            {
                if (!queue.force(command, timeout, unit))
                {
                    submittedCount.decrementAndGet();
                    throw new RejectedExecutionException("Queue capacity is full.");
                }
            }
            catch (InterruptedException x)
            {
                submittedCount.decrementAndGet();
                Thread.interrupted();
                throw new RejectedExecutionException(x);
            }
        }
        else
        {
            submittedCount.decrementAndGet();
            throw rx;
        }
    }


}
```





```java
 public boolean force(Runnable o, long timeout, TimeUnit unit) throws InterruptedException
    {
        if (parent.isShutdown())
        {
            throw new RejectedExecutionException(
                    "Executor not running, can't force a command into the queue"
            );
        }
        return super.offer(o, timeout, unit); //forces the item onto the queue, to be used if the task is rejected
    }
```









### Fork/Join

#### 概念

Fork/Join 是 JDK 1.7 加入的新的线程池实现，它体现的是一种分治思想，适用于能够进行任务拆分的 cpu 密集型运算 

所谓的任务拆分，是将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解。跟递归相关的一些计 算，如归并排序、斐波那契数列、都可以用分治思想进行求解

Fork/Join 在分治的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升了运算效率

Fork/Join 默认会创建与 cpu 核心数大小相同的线程池





```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.RecursiveTask;

/**
 * Project name(项目名称)：java并发编程_Fork_Join
 * Package(包名): mao.t1
 * Class(类名): AddTask
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/10
 * Time(创建时间)： 13:07
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class AddTask extends RecursiveTask<Integer>
{

    private final int n;

    private static final Logger log = LoggerFactory.getLogger(AddTask.class);

    public AddTask(int n)
    {
        this.n = n;
    }

    @Override
    public String toString()
    {
        return "{" + n + '}';
    }

    @Override
    protected Integer compute()
    {
        if (n == 1)
        {
            log.debug("join " + n);
            return n;
        }
        //不为1
        //拆分
        AddTask t1 = new AddTask(n - 1);
        t1.fork();
        log.debug("fork " + n + " " + t1);

        //合并
        Integer result = t1.join() + n;
        log.debug("join " + n + " " + t1 + " " + result);
        //返回
        return result;
    }
}
```



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ForkJoinPool;

/**
 * Project name(项目名称)：java并发编程_Fork_Join
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/10
 * Time(创建时间)： 13:00
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Integer result = forkJoinPool.invoke(new AddTask(10));
        log.info("结果：" + result);
    }
}
```



运行结果：

```sh
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-3] DEBUG mao.t1.AddTask:  fork 2 {1}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-17] DEBUG mao.t1.AddTask:  fork 3 {2}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-19] DEBUG mao.t1.AddTask:  fork 10 {9}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-9] DEBUG mao.t1.AddTask:  fork 7 {6}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-23] DEBUG mao.t1.AddTask:  fork 8 {7}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-5] DEBUG mao.t1.AddTask:  fork 9 {8}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-27] DEBUG mao.t1.AddTask:  fork 6 {5}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-31] DEBUG mao.t1.AddTask:  fork 4 {3}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-13] DEBUG mao.t1.AddTask:  fork 5 {4}
2022-09-10  13:19:17.887  [ForkJoinPool-1-worker-21] DEBUG mao.t1.AddTask:  join 1
2022-09-10  13:19:17.893  [ForkJoinPool-1-worker-3] DEBUG mao.t1.AddTask:  join 2 {1} 3
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-17] DEBUG mao.t1.AddTask:  join 3 {2} 6
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-31] DEBUG mao.t1.AddTask:  join 4 {3} 10
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-13] DEBUG mao.t1.AddTask:  join 5 {4} 15
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-27] DEBUG mao.t1.AddTask:  join 6 {5} 21
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-9] DEBUG mao.t1.AddTask:  join 7 {6} 28
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-23] DEBUG mao.t1.AddTask:  join 8 {7} 36
2022-09-10  13:19:17.894  [ForkJoinPool-1-worker-5] DEBUG mao.t1.AddTask:  join 9 {8} 45
2022-09-10  13:19:17.895  [ForkJoinPool-1-worker-19] DEBUG mao.t1.AddTask:  join 10 {9} 55
2022-09-10  13:19:17.895  [main] INFO  mao.t1.Test:  结果：55
```





![image-20220910132122035](img/java并发编程学习笔记/image-20220910132122035.png)







**改进**



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.RecursiveTask;

/**
 * Project name(项目名称)：java并发编程_Fork_Join
 * Package(包名): mao.t2
 * Class(类名): AddTask
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/10
 * Time(创建时间)： 13:22
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class AddTask extends RecursiveTask<Integer>
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(AddTask.class);

    /**
     * 开始
     */
    int begin;
    /**
     * 结束
     */
    int end;


    /**
     * 添加任务
     *
     * @param begin 开始
     * @param end   结束
     */
    public AddTask(int begin, int end)
    {
        this.begin = begin;
        this.end = end;
    }

    @Override
    public String toString()
    {
        return "{" + begin + "," + end + '}';
    }


    /**
     * 计算
     *
     * @return {@link Integer}
     */
    @Override
    protected Integer compute()
    {
        if (begin == end)
        {
            log.debug("join " + begin);
            return begin;
        }
        if (begin == end - 1)
        {
            int result = begin + end;
            log.debug("join " + begin + " " + end + " " + result);
            return result;
        }

        //中间
        int m = (begin + end) / 2;
        AddTask addTaskLeft = new AddTask(begin, m);
        AddTask addTaskRight = new AddTask(m + 1, end);
        addTaskLeft.fork();
        addTaskRight.fork();
        log.debug("fork " + addTaskLeft + "    " + addTaskRight);
        int result = addTaskLeft.join() + addTaskRight.join();
        log.debug("join  " + addTaskLeft + "    " + addTaskRight + "    " + result);
        return result;
    }
}
```



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ForkJoinPool;

/**
 * Project name(项目名称)：java并发编程_Fork_Join
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/10
 * Time(创建时间)： 13:30
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Integer result = forkJoinPool.invoke(new AddTask(1, 15));
        log.info("结果：" + result);
    }
}
```



运行结果：

```sh
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-29] DEBUG mao.t2.AddTask:  join 5 6 11
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-13] DEBUG mao.t2.AddTask:  fork {13,14}    {15,15}
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-5] DEBUG mao.t2.AddTask:  fork {1,4}    {5,8}
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-19] DEBUG mao.t2.AddTask:  fork {1,8}    {9,15}
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-27] DEBUG mao.t2.AddTask:  fork {1,2}    {3,4}
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-25] DEBUG mao.t2.AddTask:  join 1 2 3
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-15] DEBUG mao.t2.AddTask:  join 7 8 15
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-17] DEBUG mao.t2.AddTask:  join 9 10 19
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-11] DEBUG mao.t2.AddTask:  join 3 4 7
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-3] DEBUG mao.t2.AddTask:  join 11 12 23
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-9] DEBUG mao.t2.AddTask:  fork {9,10}    {11,12}
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-7] DEBUG mao.t2.AddTask:  join 15
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-21] DEBUG mao.t2.AddTask:  join 13 14 27
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-31] DEBUG mao.t2.AddTask:  fork {5,6}    {7,8}
2022-09-10  13:37:25.782  [ForkJoinPool-1-worker-23] DEBUG mao.t2.AddTask:  fork {9,12}    {13,15}
2022-09-10  13:37:25.790  [ForkJoinPool-1-worker-13] DEBUG mao.t2.AddTask:  join  {13,14}    {15,15}    42
2022-09-10  13:37:25.790  [ForkJoinPool-1-worker-31] DEBUG mao.t2.AddTask:  join  {5,6}    {7,8}    26
2022-09-10  13:37:25.790  [ForkJoinPool-1-worker-9] DEBUG mao.t2.AddTask:  join  {9,10}    {11,12}    42
2022-09-10  13:37:25.790  [ForkJoinPool-1-worker-27] DEBUG mao.t2.AddTask:  join  {1,2}    {3,4}    10
2022-09-10  13:37:25.790  [ForkJoinPool-1-worker-5] DEBUG mao.t2.AddTask:  join  {1,4}    {5,8}    36
2022-09-10  13:37:25.790  [ForkJoinPool-1-worker-23] DEBUG mao.t2.AddTask:  join  {9,12}    {13,15}    84
2022-09-10  13:37:25.791  [ForkJoinPool-1-worker-19] DEBUG mao.t2.AddTask:  join  {1,8}    {9,15}    120
2022-09-10  13:37:25.791  [main] INFO  mao.t2.Test:  结果：120
```





![image-20220910133934048](img/java并发编程学习笔记/image-20220910133934048.png)





```java
package mao.t3;

import mao.t2.AddTask;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.ForkJoinPool;

/**
 * Project name(项目名称)：java并发编程_Fork_Join
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/10
 * Time(创建时间)： 13:40
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 大小
     */
    private static final int size = 50000;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * test1
     *
     * @param forkJoinPool 连接池
     */
    private static void test1(ForkJoinPool forkJoinPool)
    {
        long startAll = System.currentTimeMillis();
        for (int i = 0; i < 10; i++)
        {
            long start = System.currentTimeMillis();
            Integer result = forkJoinPool.invoke(new AddTask(1, size));
            long end = System.currentTimeMillis();
            log.info("结果：" + result + ", 花费时间：" + (end - start) + "ms");
        }
        long endAll = System.currentTimeMillis();
        log.info("花费总时间：" + (endAll - startAll) + "ms");
    }

    /**
     * test2
     */
    private static void test2()
    {
        long startAll = System.currentTimeMillis();
        for (int j = 0; j < 10; j++)
        {
            long start = System.currentTimeMillis();
            long total = 0;
            for (int i = 1; i <= size; i++)
            {
                total = total + i;
            }
            long end = System.currentTimeMillis();
            log.info("结果：" + total + ", 花费时间：" + (end - start) + "ms");
        }
        long endAll = System.currentTimeMillis();
        log.info("花费总时间：" + (endAll - startAll) + "ms");
    }
    
    public static void main(String[] args)
    {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        test1(forkJoinPool);

        test2();
    }

}
```









## JUC

### AQS

#### 概述

全称是 AbstractQueuedSynchronizer，是阻塞式锁和相关的同步器工具的框架



用 state 属性来表示资源的状态（分独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取锁和释放锁

* getState - 获取 state 状态
* setState - 设置 state 状态
* compareAndSetState - cas 机制设置 state 状态
* 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源

提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList

条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet



子类主要实现这样一些方法

* tryAcquire 
* tryRelease 
* tryAcquireShared 
* tryReleaseShared 
* isHeldExclusively



获取锁：

```java
// 如果获取锁失败
if (!tryAcquire(arg)) 
{
 // 入队, 可以选择阻塞当前线程 park unpark
}
```



释放锁：

```java
// 如果释放锁成功
if (tryRelease(arg)) 
{
 // 让阻塞线程恢复运行
}
```





#### 实现不可重入锁



```java
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;

/**
 * Project name(项目名称)：java并发编程_AQS
 * Package(包名): PACKAGE_NAME
 * Class(类名): MySync
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/11
 * Time(创建时间)： 13:06
 * Version(版本): 1.0
 * Description(描述)： 自定义同步器
 */

public final class MySync extends AbstractQueuedSynchronizer
{
    @Override
    protected boolean tryAcquire(int arg)
    {
        if (arg == 1)
        {
            if (compareAndSetState(0, 1))
            {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
        }
        return false;
    }

    @Override
    protected boolean tryRelease(int arg)
    {
        if (arg == 1)
        {
            if (getState() == 0)
            {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
        return false;
    }

    @Override
    protected boolean isHeldExclusively()
    {
        return getState() == 1;
    }

    Condition newCondition()
    {
        return new ConditionObject();
    }
}
```



```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/**
 * Project name(项目名称)：java并发编程_AQS
 * Package(包名): PACKAGE_NAME
 * Class(类名): MyLock
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/11
 * Time(创建时间)： 13:13
 * Version(版本): 1.0
 * Description(描述)： 自定义锁
 */

public class MyLock implements Lock
{
    private static final MySync sync = new MySync();

    @Override
    public void lock()
    {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException
    {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock()
    {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException
    {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock()
    {
        sync.release(1);
    }

    @Override
    public Condition newCondition()
    {
        return sync.newCondition();
    }
}
```



```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_AQS
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/11
 * Time(创建时间)： 13:19
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);
    /**
     * 锁
     */
    private static final MyLock lock = new MyLock();

    public static void main(String[] args)
    {
        Thread thread1 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取锁");
                lock.lock();
                try
                {
                    log.debug("获取到锁");
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    lock.unlock();
                    log.debug("释放锁");
                }
            }
        }, "t1");

        Thread thread2 = new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取锁");
                lock.lock();
                try
                {
                    log.debug("获取到锁");
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    lock.unlock();
                    log.debug("释放锁");
                }
            }
        }, "t2");

        thread1.start();
        thread2.start();
    }
}
```



运行结果：

```sh
2022-09-11  13:28:38.247  [t1] DEBUG Test:  尝试获取锁
2022-09-11  13:28:38.247  [t2] DEBUG Test:  尝试获取锁
2022-09-11  13:28:38.249  [t1] DEBUG Test:  获取到锁
2022-09-11  13:28:39.252  [t1] DEBUG Test:  释放锁
2022-09-11  13:28:39.252  [t2] DEBUG Test:  获取到锁
2022-09-11  13:28:40.254  [t2] DEBUG Test:  释放锁
```





**不可重入测试：**



```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Project name(项目名称)：java并发编程_AQS
 * Package(包名): PACKAGE_NAME
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/11
 * Time(创建时间)： 13:31
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test2.class);

    /**
     * 不可重入锁
     */
    private static final MyLock lock = new MyLock();

    /**
     * 可重入锁
     */
    //private static final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args)
    {
        log.debug("尝试获取锁");
        lock.lock();
        try
        {
            log.debug("获取到锁");
            m1();
        }
        finally
        {
            lock.unlock();
            log.debug("释放锁");
        }

    }

    private static void m1()
    {
        log.debug("尝试获取锁");
        lock.lock();
        try
        {
            log.debug("获取到锁");
            m2();
        }
        finally
        {
            lock.unlock();
            log.debug("释放锁");
        }
    }

    private static void m2()
    {
        log.debug("尝试获取锁");
        lock.lock();
        try
        {
            log.debug("获取到锁");
            m3();
        }
        finally
        {
            lock.unlock();
            log.debug("释放锁");
        }
    }

    private static void m3()
    {
        log.debug("尝试获取锁");
        lock.lock();
        try
        {
            log.debug("获取到锁");
        }
        finally
        {
            lock.unlock();
            log.debug("释放锁");
        }
    }
}
```



运行结果：

```sh
2022-09-11  13:36:16.697  [main] DEBUG Test2:  尝试获取锁
2022-09-11  13:36:16.699  [main] DEBUG Test2:  获取到锁
2022-09-11  13:36:16.699  [main] DEBUG Test2:  尝试获取锁
```





#### 设计

**获取锁的逻辑：**

```
while(state 状态不允许获取) 
{
 if(队列中还没有此线程) 
 {
 入队并阻塞
 }
}
当前线程出队
```



**释放锁的逻辑：**

```
if(state 状态允许了) 
{
 恢复阻塞的线程(s)
}

```



**state 设计：**

* state 使用 volatile 配合 cas 保证其修改时的原子性
* state 使用了 32bit int 来维护同步状态，因为当时使用 long 在很多平台下测试的结果并不理想



**阻塞恢复设计：**

* 早期的控制线程暂停和恢复的 api 有 suspend 和 resume，但它们是不可用的，因为如果先调用的 resume  那么 suspend 将感知不到
* 解决方法是使用 park & unpark 来实现线程的暂停和恢复
* park & unpark 是针对线程的，而不是针对同步器的，因此控制粒度更为精细
* park 线程还可以通过 interrupt 打断



**队列设计：**

* 使用了 FIFO 先入先出队列，并不支持优先级队列
* 设计时借鉴了 CLH 队列，它是一种单向无锁队列



**CLH 好处：**

* 无锁，使用自旋
* 快速，无阻塞







### ReentrantLock 原理



![image-20220911135704057](img/java并发编程学习笔记/image-20220911135704057.png)





#### 非公平锁实现原理



默认为非公平锁实现

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}
```



可以选择是否为公平锁

```java
/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```



NonfairSync 继承自 AQS



没有竞争时

![image-20220911140024111](img/java并发编程学习笔记/image-20220911140024111.png)



第一个竞争出现时



![image-20220911140140123](img/java并发编程学习笔记/image-20220911140140123.png)





Thread-1 执行：

*  CAS 尝试将 state 由 0 改为 1，结果失败
* 进入 tryAcquire 逻辑，这时 state 已经是1，结果仍然失败
* 接下来进入 addWaiter 逻辑，构造 Node 队列
  * 图中黄色三角表示该 Node 的 waitStatus 状态，其中 0 为默认正常状态
  * Node 的创建是懒惰的
  * 其中第一个 Node 称为 Dummy（哑元）或哨兵，用来占位，并不关联线程



![image-20220911140631546](img/java并发编程学习笔记/image-20220911140631546.png)



当前线程进入 acquireQueued 逻辑

* acquireQueued 会在一个死循环中不断尝试获得锁，失败后进入 park 阻塞
* 如果自己是紧邻着 head（排第二位），那么再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败
* 进入 shouldParkAfterFailedAcquire 逻辑，将前驱 node，即 head 的 waitStatus 改为 -1，这次返回 false



![image-20220911140856981](img/java并发编程学习笔记/image-20220911140856981.png)



* shouldParkAfterFailedAcquire 执行完毕回到 acquireQueued ，再次 tryAcquire 尝试获取锁，当然这时 state 仍为 1，失败
* 当再次进入 shouldParkAfterFailedAcquire 时，这时因为其前驱 node 的 waitStatus 已经是 -1，这次返回 true
* 进入 parkAndCheckInterrupt， Thread-1 park（灰色表示）



![image-20220911185320036](img/java并发编程学习笔记/image-20220911185320036.png)



![image-20220911185406626](img/java并发编程学习笔记/image-20220911185406626.png)





Thread-0 释放锁，进入 tryRelease 流程，如果成功

* 设置 exclusiveOwnerThread 为 null
* state = 0



![image-20220911185943234](img/java并发编程学习笔记/image-20220911185943234.png)





* 当前队列不为 null，并且 head 的 waitStatus = -1，进入 unparkSuccessor 流程
* 找到队列中离 head 最近的一个 Node（没取消的），unpark 恢复其运行，即为 Thread-1
* 回到 Thread-1 的 acquireQueued 流程



![image-20220911190103210](img/java并发编程学习笔记/image-20220911190103210.png)



如果加锁成功（没有竞争），会设置

* exclusiveOwnerThread 为 Thread-1，state = 1
* head 指向刚刚 Thread-1 所在的 Node，该 Node 清空 Thread
* 原本的 head 因为从链表断开，而可被垃圾回收



如果这时候有其它线程来竞争（非公平的体现），例如这时有 Thread-4 来了



如果不巧又被 Thread-4 占了先

* Thread-4 被设置为 exclusiveOwnerThread，state = 1
* Thread-1 再次进入 acquireQueued 流程，获取锁失败，重新进入 park 阻塞



![image-20220911190315957](img/java并发编程学习笔记/image-20220911190315957.png)







部分源码：

```java


public class NonfairSync extends Sync
{
    private static final long serialVersionUID = 7316153563782823691L;

    // 加锁实现
    final void lock()
    {
        // 首先用 cas 尝试（仅尝试一次）将 state 从 0 改为 1, 如果成功表示获得了独占锁
        if (compareAndSetState(0, 1))
        {
            setExclusiveOwnerThread(Thread.currentThread());
        }
        else
        // 如果尝试失败，进入 ㈠
        {
            acquire(1);
        }
    }

    // ㈠ AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquire(int arg)
    {
        // ㈡ tryAcquire
        if (
                !tryAcquire(arg) &&
                        // 当 tryAcquire 返回为 false 时, 先调用 addWaiter ㈣, 接着 acquireQueued ㈤
                        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        )
        {
            selfInterrupt();
        }
    }

    // ㈡ 进入 ㈢
    protected final boolean tryAcquire(int acquires)
    {
        return nonfairTryAcquire(acquires);
    }

    // ㈢ Sync 继承过来的方法, 方便阅读, 放在此处
    final boolean nonfairTryAcquire(int acquires)
    {
        final Thread current = Thread.currentThread();
        int c = getState();
        // 如果还没有获得锁
        if (c == 0)
        {
            // 尝试用 cas 获得, 这里体现了非公平性: 不去检查 AQS 队列
            if (compareAndSetState(0, acquires))
            {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果已经获得了锁, 线程还是当前线程, 表示发生了锁重入
        else if (current == getExclusiveOwnerThread())
        {
            // state++
            int nextc = c + acquires;
            if (nextc < 0) // overflow
            {
                throw new Error("Maximum lock count exceeded");
            }
            setState(nextc);
            return true;
        }
        // 获取失败, 回到调用处
        return false;
    }

    // ㈣ AQS 继承过来的方法, 方便阅读, 放在此处
    private Node addWaiter(Node mode)
    {// 将当前线程关联到一个 Node 对象上, 模式为独占模式
        Node node = new Node(Thread.currentThread(), mode);
        // 如果 tail 不为 null, cas 尝试将 Node 对象加入 AQS 队列尾部
        Node pred = tail;
        if (pred != null)
        {
            node.prev = pred;
            if (compareAndSetTail(pred, node))
            {
                // 双向链表
                pred.next = node;
                return node;
            }
        }
        // 尝试将 Node 加入 AQS, 进入 ㈥
        enq(node);
        return node;
    }

    // ㈥ AQS 继承过来的方法, 方便阅读, 放在此处
    private Node enq(final Node node)
    {
        for (; ; )
        {
            Node t = tail;
            if (t == null)
            {
                // 还没有, 设置 head 为哨兵节点（不对应线程，状态为 0）
                if (compareAndSetHead(new Node()))
                {
                    tail = head;
                }
            }
            else
            {
                // cas 尝试将 Node 对象加入 AQS 队列尾部
                node.prev = t;
                if (compareAndSetTail(t, node))
                {
                    t.next = node;
                    return t;
                }
            }
        }
    }

    // ㈤ AQS 继承过来的方法, 方便阅读, 放在此处
    final boolean acquireQueued(final Node node, int arg)
    {
        boolean failed = true;
        try
        {
            boolean interrupted = false;
            for (; ; )
            {
                final Node p = node.predecessor();
                // 上一个节点是 head, 表示轮到自己（当前线程对应的 node）了, 尝试获取
                if (p == head && tryAcquire(arg))
                {
                    // 获取成功, 设置自己（当前线程对应的 node）为 head
                    setHead(node);
                    // 上一个节点 help GC
                    p.next = null;
                    failed = false;
                    // 返回中断标记 false
                    return interrupted;
                }
                if (
                    // 判断是否应当 park, 进入 ㈦
                        shouldParkAfterFailedAcquire(p, node) &&
                                // park 等待, 此时 Node 的状态被置为 Node.SIGNAL ㈧
                                parkAndCheckInterrupt()
                )
                {
                    interrupted = true;
                }
            }
        }
        finally
        {
            if (failed)
            {
                cancelAcquire(node);
            }
        }
    }

    // ㈦ AQS 继承过来的方法, 方便阅读, 放在此处
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node)
    {
        // 获取上一个节点的状态
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
        {
            // 上一个节点都在阻塞, 那么自己也阻塞好了
            return true;
        }
        // > 0 表示取消状态
        if (ws > 0)
        {
            // 上一个节点取消, 那么重构删除前面所有取消的节点, 返回到外层循环重试
            do
            {
                node.prev = pred = pred.prev;
            }
            while (pred.waitStatus > 0);
            pred.next = node;
        }
        else
        {
            // 这次还没有阻塞
            // 但下次如果重试不成功, 则需要阻塞，这时需要设置上一个节点状态为 Node.SIGNAL
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

    // ㈧ 阻塞当前线程
    private final boolean parkAndCheckInterrupt()
    {
        LockSupport.park(this);
        return Thread.interrupted();
    }


    //*************************************************************

    // 解锁实现
    public void unlock()
    {
        sync.release(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean release(int arg)
    {
        // 尝试释放锁, 进入 ㈠
        if (tryRelease(arg))
        {
            // 队列头节点 unpark
            Node h = head;
            if (
                // 队列不为 null
                    h != null &&
                            // waitStatus == Node.SIGNAL 才需要 unpark
                            h.waitStatus != 0
            )
            {
                // unpark AQS 中等待的线程, 进入 ㈡
                unparkSuccessor(h);
            }
            return true;
        }
        return false;
    }

    // ㈠ Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryRelease(int releases)
    {
        // state--
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
        {
            throw new IllegalMonitorStateException();
        }
        boolean free = false;
        // 支持锁重入, 只有 state 减为 0, 才释放成功
        if (c == 0)
        {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    // ㈡ AQS 继承过来的方法, 方便阅读, 放在此处
    private void unparkSuccessor(Node node)
    {
        // 如果状态为 Node.SIGNAL 尝试重置状态为 0
        // 不成功也可以
        int ws = node.waitStatus;
        if (ws < 0)
        {
            compareAndSetWaitStatus(node, ws, 0);
        }
        // 找到需要 unpark 的节点, 但本节点从 AQS 队列中脱离, 是由唤醒节点完成的
        Node s = node.next;
        // 不考虑已取消的节点, 从 AQS 队列从后至前找到队列最前面需要 unpark 的节点
        if (s == null || s.waitStatus > 0)
        {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
            {
                if (t.waitStatus <= 0)
                {
                    s = t;
                }
            }
        }
        if (s != null)
        {
            LockSupport.unpark(s.thread);
        }
    }

}

```







#### 公平锁实现原理



```java


public class FairSync extends Sync
{
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock()
    {
        acquire(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquire(int arg)
    {
        if (
                !tryAcquire(arg) &&
                        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        )
        {
            selfInterrupt();
        }
    }

    // 与非公平锁主要区别在于 tryAcquire 方法的实现
    protected final boolean tryAcquire(int acquires)
    {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0)
        {
            // 先检查 AQS 队列中是否有前驱节点, 没有才去竞争
            if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires))
            {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread())
        {
            int nextc = c + acquires;
            if (nextc < 0)
            {
                throw new Error("Maximum lock count exceeded");
            }
            setState(nextc);
            return true;
        }
        return false;
    }

    // ㈠ AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean hasQueuedPredecessors()
    {
        Node t = tail;
        Node h = head;
        Node s;
        // h != t 时表示队列中有 Node
        return h != t &&
                (
                        // (s = h.next) == null 表示队列中还有没有老二
                        (s = h.next) == null ||
                                // 或者队列中老二线程不是此线程
                                s.thread != Thread.currentThread()
                );
    }
}

```





#### 条件变量实现原理

每个条件变量其实就对应着一个等待队列，其实现类是 ConditionObject



**await 流程**

开始 Thread-0 持有锁，调用 await，进入 ConditionObject 的 addConditionWaiter 流程

创建新的 Node 状态为 -2（Node.CONDITION），关联 Thread-0，加入等待队列尾部



![image-20220911192828100](img/java并发编程学习笔记/image-20220911192828100.png)



接下来进入 AQS 的 fullyRelease 流程，释放同步器上的锁



![image-20220911192927835](img/java并发编程学习笔记/image-20220911192927835.png)



unpark AQS 队列中的下一个节点，竞争锁，假设没有其他竞争线程，那么 Thread-1 竞争成功



![image-20220911193045121](img/java并发编程学习笔记/image-20220911193045121.png)



park 阻塞 Thread-0



![image-20220911193158175](img/java并发编程学习笔记/image-20220911193158175.png)





**signal 流程**

假设 Thread-1 要来唤醒 Thread-0



![image-20220911193432215](img/java并发编程学习笔记/image-20220911193432215.png)



进入 ConditionObject 的 doSignal 流程，取得等待队列中第一个 Node，即 Thread-0 所在 Node



![image-20220911193544995](img/java并发编程学习笔记/image-20220911193544995.png)



执行 transferForSignal 流程，将该 Node 加入 AQS 队列尾部，将 Thread-0 的 waitStatus 改为 0，Thread-3 的 waitStatus 改为 -1



![image-20220911193646549](img/java并发编程学习笔记/image-20220911193646549.png)





**源码**



```java
import java.util.concurrent.locks.Condition;


public class ConditionObject implements Condition, java.io.Serializable
{
    private static final long serialVersionUID = 1173984872572414699L;

    // 第一个等待节点
    private transient Node firstWaiter;

    // 最后一个等待节点
    private transient Node lastWaiter;

    public ConditionObject()
    {
    }

    // ㈠ 添加一个 Node 至等待队列
    private Node addConditionWaiter()
    {
        Node t = lastWaiter;
        // 所有已取消的 Node 从队列链表删除
        if (t != null && t.waitStatus != Node.CONDITION)
        {
            unlinkCancelledWaiters();
            t = lastWaiter;
        }
        // 创建一个关联当前线程的新 Node, 添加至队列尾部
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        if (t == null)
        {
            firstWaiter = node;
        }
        else
        {
            t.nextWaiter = node;
        }
        lastWaiter = node;
        return node;
    }

    // 唤醒 - 将没取消的第一个节点转移至 AQS 队列
    private void doSignal(Node first)
    {
        do
        {
            // 已经是尾节点了
            if ((firstWaiter = first.nextWaiter) == null)
            {
                lastWaiter = null;
            }
            first.nextWaiter = null;
        }
        while (
            // 将等待队列中的 Node 转移至 AQS 队列, 不成功且还有节点则继续循环 ㈢
                !transferForSignal(first) &&
                        // 队列还有节点
                        (first = firstWaiter) != null
        );
    }

    // 外部类方法, 方便阅读, 放在此处
    // ㈢ 如果节点状态是取消, 返回 false 表示转移失败, 否则转移成功
    final boolean transferForSignal(Node node)
    {
        // 如果状态已经不是 Node.CONDITION, 说明被取消了
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        {
            return false;
        }
        // 加入 AQS 队列尾部
        Node p = enq(node);
        int ws = p.waitStatus;
        if (
            // 上一个节点被取消
                ws > 0 ||
                        // 上一个节点不能设置状态为 Node.SIGNAL
                        !compareAndSetWaitStatus(p, ws, Node.SIGNAL)
        )
        {
            // unpark 取消阻塞, 让线程重新同步状态
            LockSupport.unpark(node.thread);
        }
        return true;
    }

    // 全部唤醒 - 等待队列的所有节点转移至 AQS 队列
    private void doSignalAll(Node first)
    {
        lastWaiter = firstWaiter = null;
        do
        {
            Node next = first.nextWaiter;
            first.nextWaiter = null;
            transferForSignal(first);
            first = next;
        }
        while (first != null);
    }

    // ㈡
    private void unlinkCancelledWaiters()
    {
        // ...
    }

    // 唤醒 - 必须持有锁才能唤醒, 因此 doSignal 内无需考虑加锁
    public final void signal()
    {
        if (!isHeldExclusively())
        {
            throw new IllegalMonitorStateException();
        }
        Node first = firstWaiter;
        if (first != null)
        {
            doSignal(first);
        }
    }

    // 全部唤醒 - 必须持有锁才能唤醒, 因此 doSignalAll 内无需考虑加锁
    public final void signalAll()
    {
        if (!isHeldExclusively())
        {
            throw new IllegalMonitorStateException();
        }
        Node first = firstWaiter;
        if (first != null)
        {
            doSignalAll(first);
        }
    }

    // 不可打断等待 - 直到被唤醒
    public final void awaitUninterruptibly()
    {
        // 添加一个 Node 至等待队列, 见 ㈠
        Node node = addConditionWaiter();
        // 释放节点持有的锁, 见 ㈣
        int savedState = fullyRelease(node);
        boolean interrupted = false;
        // 如果该节点还没有转移至 AQS 队列, 阻塞
        while (!isOnSyncQueue(node))
        {
            // park 阻塞
            LockSupport.park(this);
            // 如果被打断, 仅设置打断状态
            if (Thread.interrupted())
            {
                interrupted = true;
            }
        }
        // 唤醒后, 尝试竞争锁, 如果失败进入 AQS 队列
        if (acquireQueued(node, savedState) || interrupted)
        {
            selfInterrupt();
        }
    }

    // 外部类方法, 方便阅读, 放在此处
    // ㈣ 因为某线程可能重入，需要将 state 全部释放
    final int fullyRelease(Node node)
    {
        boolean failed = true;
        try
        {
            int savedState = getState();
            if (release(savedState))
            {
                failed = false;
                return savedState;
            }
            else
            {
                throw new IllegalMonitorStateException();
            }
        }
        finally
        {
            if (failed)
            {
                node.waitStatus = Node.CANCELLED;
            }
        }
    }

    // 打断模式 - 在退出等待时重新设置打断状态
    private static final int REINTERRUPT = 1;
    // 打断模式 - 在退出等待时抛出异常
    private static final int THROW_IE = -1;

    // 判断打断模式
    private int checkInterruptWhileWaiting(Node node)
    {
        return Thread.interrupted() ?
                (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
                0;
    }

    // ㈤ 应用打断模式
    private void reportInterruptAfterWait(int interruptMode)
            throws InterruptedException
    {
        if (interruptMode == THROW_IE)
        {
            throw new InterruptedException();
        }
        else if (interruptMode == REINTERRUPT)
        {
            selfInterrupt();
        }
    }

    // 等待 - 直到被唤醒或打断
    public final void await() throws InterruptedException
    {
        if (Thread.interrupted())
        {
            throw new InterruptedException();
        }
        // 添加一个 Node 至等待队列, 见 ㈠
        Node node = addConditionWaiter();
        // 释放节点持有的锁
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 如果该节点还没有转移至 AQS 队列, 阻塞
        while (!isOnSyncQueue(node))
        {
            // park 阻塞
            LockSupport.park(this);// 如果被打断, 退出等待队列
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            {
                break;
            }
        }
        // 退出等待队列后, 还需要获得 AQS 队列的锁
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        {
            interruptMode = REINTERRUPT;
        }
        // 所有已取消的 Node 从队列链表删除, 见 ㈡
        if (node.nextWaiter != null)
        {
            unlinkCancelledWaiters();
        }
        // 应用打断模式, 见 ㈤
        if (interruptMode != 0)
        {
            reportInterruptAfterWait(interruptMode);
        }
    }

    // 等待 - 直到被唤醒或打断或超时
    public final long awaitNanos(long nanosTimeout) throws InterruptedException
    {
        if (Thread.interrupted())
        {
            throw new InterruptedException();
        }
        // 添加一个 Node 至等待队列, 见 ㈠
        Node node = addConditionWaiter();
        // 释放节点持有的锁
        int savedState = fullyRelease(node);
        // 获得最后期限
        final long deadline = System.nanoTime() + nanosTimeout;
        int interruptMode = 0;
        // 如果该节点还没有转移至 AQS 队列, 阻塞
        while (!isOnSyncQueue(node))
        {
            // 已超时, 退出等待队列
            if (nanosTimeout <= 0L)
            {
                transferAfterCancelledWait(node);
                break;
            }
            // park 阻塞一定时间, spinForTimeoutThreshold 为 1000 ns
            if (nanosTimeout >= spinForTimeoutThreshold)
            {
                LockSupport.parkNanos(this, nanosTimeout);
            }
            // 如果被打断, 退出等待队列
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            {
                break;
            }
            nanosTimeout = deadline - System.nanoTime();
        }
        // 退出等待队列后, 还需要获得 AQS 队列的锁
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        {
            interruptMode = REINTERRUPT;
        }
        // 所有已取消的 Node 从队列链表删除, 见 ㈡
        if (node.nextWaiter != null)
        {
            unlinkCancelledWaiters();
        }
        // 应用打断模式, 见 ㈤
        if (interruptMode != 0)
        {
            reportInterruptAfterWait(interruptMode);
        }
        return deadline - System.nanoTime();
    }
    // 等待 - 直到被唤醒或打断或超时, 逻辑类似于 awaitNanos

    public final boolean awaitUntil(Date deadline) throws InterruptedException
    {
        // ...
    }

    // 等待 - 直到被唤醒或打断或超时, 逻辑类似于 awaitNanos
    public final boolean await(long time, TimeUnit unit) throws InterruptedException
    {
        // ...
    }
    // 工具方法
}
```









### 读写锁

#### ReentrantReadWriteLock

当读操作远远高于写操作时，这时候使用 读写锁 让 读-读 可以并发，提高性能。



读-读 可以并发

```java
package mao.t1;

import org.apache.logging.log4j.core.util.UuidUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.UUID;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantReadWriteLock
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/12
 * Time(创建时间)： 13:08
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * 读
     */
    public static void read()
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取读锁");
                lock.readLock().lock();
                log.debug("获取到读锁");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放读锁");
                    lock.readLock().unlock();
                }
            }
        }, "read:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

    /**
     * 写
     */
    public static void write()
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取写锁");
                lock.writeLock().lock();
                log.debug("获取到写锁");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放写锁");
                    lock.writeLock().unlock();
                }
            }
        }, "write:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

    public static void main(String[] args)
    {
        read();
        read();
    }
}
```



运行结果：

```sh
2022-09-12  13:21:00.535  [read:e4ceae] DEBUG mao.t1.Test:  尝试获取读锁
2022-09-12  13:21:00.535  [read:a37d09] DEBUG mao.t1.Test:  尝试获取读锁
2022-09-12  13:21:00.537  [read:a37d09] DEBUG mao.t1.Test:  获取到读锁
2022-09-12  13:21:00.537  [read:e4ceae] DEBUG mao.t1.Test:  获取到读锁
2022-09-12  13:21:01.545  [read:a37d09] DEBUG mao.t1.Test:  释放读锁
2022-09-12  13:21:01.545  [read:e4ceae] DEBUG mao.t1.Test:  释放读锁
```



读-写 不可以并发



```java
package mao.t1;

import org.apache.logging.log4j.core.util.UuidUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.UUID;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantReadWriteLock
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/12
 * Time(创建时间)： 13:08
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * 读
     */
    public static void read()
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取读锁");
                lock.readLock().lock();
                log.debug("获取到读锁");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放读锁");
                    lock.readLock().unlock();
                }
            }
        }, "read:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

    /**
     * 写
     */
    public static void write()
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取写锁");
                lock.writeLock().lock();
                log.debug("获取到写锁");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放写锁");
                    lock.writeLock().unlock();
                }
            }
        }, "write:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

    public static void main(String[] args)
    {
        read();
        write();
    }
}
```



运行结果：

```sh
2022-09-12  13:22:22.602  [write:fd3175] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-12  13:22:22.602  [read:9213c5] DEBUG mao.t1.Test:  尝试获取读锁
2022-09-12  13:22:22.605  [write:fd3175] DEBUG mao.t1.Test:  获取到写锁
2022-09-12  13:22:23.618  [write:fd3175] DEBUG mao.t1.Test:  释放写锁
2022-09-12  13:22:23.618  [read:9213c5] DEBUG mao.t1.Test:  获取到读锁
2022-09-12  13:22:24.618  [read:9213c5] DEBUG mao.t1.Test:  释放读锁
```

```sh
2022-09-12  13:23:46.964  [read:48e81e] DEBUG mao.t1.Test:  尝试获取读锁
2022-09-12  13:23:46.964  [write:9ee7ba] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-12  13:23:46.966  [read:48e81e] DEBUG mao.t1.Test:  获取到读锁
2022-09-12  13:23:47.971  [read:48e81e] DEBUG mao.t1.Test:  释放读锁
2022-09-12  13:23:47.971  [write:9ee7ba] DEBUG mao.t1.Test:  获取到写锁
2022-09-12  13:23:48.973  [write:9ee7ba] DEBUG mao.t1.Test:  释放写锁
```





写-写 不可以并发



```java
package mao.t1;

import org.apache.logging.log4j.core.util.UuidUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.UUID;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantReadWriteLock
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/12
 * Time(创建时间)： 13:08
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * 读
     */
    public static void read()
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取读锁");
                lock.readLock().lock();
                log.debug("获取到读锁");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放读锁");
                    lock.readLock().unlock();
                }
            }
        }, "read:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

    /**
     * 写
     */
    public static void write()
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取写锁");
                lock.writeLock().lock();
                log.debug("获取到写锁");
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally
                {
                    log.debug("释放写锁");
                    lock.writeLock().unlock();
                }
            }
        }, "write:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

    public static void main(String[] args)
    {
        write();
        write();
    }
}
```



运行结果：

```sh
2022-09-12  13:25:02.357  [write:238465] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-12  13:25:02.357  [write:e37f08] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-12  13:25:02.359  [write:e37f08] DEBUG mao.t1.Test:  获取到写锁
2022-09-12  13:25:03.373  [write:e37f08] DEBUG mao.t1.Test:  释放写锁
2022-09-12  13:25:03.373  [write:238465] DEBUG mao.t1.Test:  获取到写锁
2022-09-12  13:25:04.382  [write:238465] DEBUG mao.t1.Test:  释放写锁
```





* 读锁不支持条件变量
* 重入时升级不支持：即持有读锁的情况下去获取写锁，会导致获取写锁永久等待



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.UUID;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantReadWriteLock
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/12
 * Time(创建时间)： 13:28
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取读锁");
                lock.readLock().lock();
                log.debug("获取到读锁");
                try
                {
                    log.debug("尝试获取写锁");
                    lock.writeLock().lock();
                    log.debug("获取到写锁");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    finally
                    {
                        log.debug("释放写锁");
                        lock.writeLock().unlock();
                    }
                }

                finally
                {
                    log.debug("释放读锁");
                    lock.readLock().unlock();
                }
            }
        }, "read:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }

}
```



运行结果：

```sh
2022-09-12  13:29:47.871  [read:669706] DEBUG mao.t2.Test:  尝试获取读锁
2022-09-12  13:29:47.873  [read:669706] DEBUG mao.t2.Test:  获取到读锁
2022-09-12  13:29:47.873  [read:669706] DEBUG mao.t2.Test:  尝试获取写锁
...
```





* 重入时降级支持：即持有写锁的情况下去获取读锁



```java
package mao.t3;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.UUID;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Project name(项目名称)：java并发编程_ReentrantReadWriteLock
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/12
 * Time(创建时间)： 13:31
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 锁
     */
    private static final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                log.debug("尝试获取写锁");
                lock.writeLock().lock();
                log.debug("获取到写锁");
                try
                {
                    log.debug("尝试获取读锁");
                    lock.readLock().lock();
                    log.debug("获取到读锁");
                    try
                    {
                        Thread.sleep(1000);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    finally
                    {
                        log.debug("释放读锁");
                        lock.readLock().unlock();
                    }
                }
                finally
                {
                    log.debug("释放写锁");
                    lock.writeLock().unlock();
                }
            }
        }, "write:" + UUID.randomUUID().toString().substring(0, 6)).start();
    }
}
```



运行结果：

```sh
2022-09-12  13:33:23.559  [write:85f1ab] DEBUG mao.t3.Test:  尝试获取写锁
2022-09-12  13:33:23.561  [write:85f1ab] DEBUG mao.t3.Test:  获取到写锁
2022-09-12  13:33:23.562  [write:85f1ab] DEBUG mao.t3.Test:  尝试获取读锁
2022-09-12  13:33:23.562  [write:85f1ab] DEBUG mao.t3.Test:  获取到读锁
2022-09-12  13:33:24.564  [write:85f1ab] DEBUG mao.t3.Test:  释放读锁
2022-09-12  13:33:24.564  [write:85f1ab] DEBUG mao.t3.Test:  释放写锁
```





#### 读写锁原理

读写锁用的是同一个 Sycn 同步器，因此等待队列、state 等也是同一个

 t1执行 w.lock，t1 成功上锁，流程与 ReentrantLock 加锁相比没有特殊之处，不同是写锁状态占了 state 的低 16 位，而读锁 使用的是 state 的高 16 位



![image-20220912134758424](img/java并发编程学习笔记/image-20220912134758424.png)



t2 执行 r.lock，这时进入读锁的 sync.acquireShared(1) 流程，首先会进入 tryAcquireShared 流程。如果有写锁占据，那么 tryAcquireShared 返回 -1 表示失败

* -1 表示失败
* 0 表示成功，但后继节点不会继续唤醒
* 正数表示成功，而且数值是还有几个后继节点需要唤醒，读写锁返回 1



![image-20220912135025879](img/java并发编程学习笔记/image-20220912135025879.png)



这时会进入 sync.doAcquireShared(1) 流程，首先也是调用 addWaiter 添加节点，不同之处在于节点被设置为 Node.SHARED 模式而非 Node.EXCLUSIVE 模式，注意此时 t2 仍处于活跃状态



![image-20220912135131364](img/java并发编程学习笔记/image-20220912135131364.png)



t2 会看看自己的节点是不是第二个，如果是，还会再次调用 tryAcquireShared(1) 来尝试获取锁



如果没有成功，在 doAcquireShared 内 for (;;) 循环一次，把前驱节点的 waitStatus 改为 -1，再 for (;;) 循环一 次尝试 tryAcquireShared(1) 如果还不成功，那么在 parkAndCheckInterrupt() 处 park



![image-20220912135304562](img/java并发编程学习笔记/image-20220912135304562.png)





假设又有 t3 加读锁和 t4 加写锁，这期间 t1 仍然持有锁，就变成了下面的样子



![image-20220912135411707](img/java并发编程学习笔记/image-20220912135411707.png)



t1 w.unlock，这时会走到写锁的 sync.release(1) 流程，调用 sync.tryRelease(1) 成功



![image-20220912135532893](img/java并发编程学习笔记/image-20220912135532893.png)



接下来执行唤醒流程 sync.unparkSuccessor，即让第二个节点恢复运行，这时 t2 在 doAcquireShared 内 parkAndCheckInterrupt() 处恢复运行

这回再来一次 for (;;) 执行 tryAcquireShared 成功则让读锁计数加一



![image-20220912135707664](img/java并发编程学习笔记/image-20220912135707664.png)





这时 t2 已经恢复运行，接下来 t2 调用 setHeadAndPropagate(node, 1)，它原本所在节点被置为头节点



![image-20220912135858992](img/java并发编程学习笔记/image-20220912135858992.png)



在 setHeadAndPropagate 方法内还会检查下一个节点是否是 shared，如果是则调用 doReleaseShared() 将 head 的状态从 -1 改为 0 并唤醒第二个节点，这时 t3 在 doAcquireShared 内 parkAndCheckInterrupt() 处恢复运行



![image-20220912140258831](img/java并发编程学习笔记/image-20220912140258831.png)



这回再来一次 for (;;) 执行 tryAcquireShared 成功则让读锁计数加一



![image-20220912140450813](img/java并发编程学习笔记/image-20220912140450813.png)



这时 t3 已经恢复运行，接下来 t3 调用 setHeadAndPropagate(node, 1)，它原本所在节点被置为头节点



![image-20220912140543309](img/java并发编程学习笔记/image-20220912140543309.png)



下一个节点不是 shared 了，因此不会继续唤醒 t4 所在节点



t2 进入 sync.releaseShared(1) 中，调用 tryReleaseShared(1) 让计数减一，但由于计数还不为零



![image-20220912140700856](img/java并发编程学习笔记/image-20220912140700856.png)



t3 进入 sync.releaseShared(1) 中，调用 tryReleaseShared(1) 让计数减一，这回计数为零了，进入 doReleaseShared() 将头节点从 -1 改为 0 并唤醒第二个节点



![image-20220912140804436](img/java并发编程学习笔记/image-20220912140804436.png)





之后 t4 在 acquireQueued 中 parkAndCheckInterrupt 处恢复运行，再次 for (;;) 这次自己是第二个节点，并且没有其他竞争，tryAcquire(1) 成功，修改头结点，流程结束



![image-20220912140922349](img/java并发编程学习笔记/image-20220912140922349.png)





部分源码



```java
public class NonfairSync extends Sync
{
    // ... 省略无关代码

    // 外部类 WriteLock 方法, 方便阅读, 放在此处
    public void lock()
    {
        sync.acquire(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquire(int arg)
    {
        if (
            // 尝试获得写锁失败
                !tryAcquire(arg) &&
                        // 将当前线程关联到一个 Node 对象上, 模式为独占模式
                        // 进入 AQS 队列阻塞
                        acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        )
        {
            selfInterrupt();
        }
    }

    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryAcquire(int acquires)
    {
        // 获得低 16 位, 代表写锁的 state 计数
        Thread current = Thread.currentThread();
        int c = getState();
        int w = exclusiveCount(c);

        if (c != 0)
        {
            if (
                // c != 0 and w == 0 表示有读锁, 或者
                    w == 0 ||
                            // 如果 exclusiveOwnerThread 不是自己
                            current != getExclusiveOwnerThread()
            )
            {
                // 获得锁失败
                return false;
            }
            // 写锁计数超过低 16 位, 报异常
            if (w + exclusiveCount(acquires) > MAX_COUNT)
            {
                throw new Error("Maximum lock count exceeded");
            }
            // 写锁重入, 获得锁成功
            setState(c + acquires);
            return true;
        }
        if (
            // 判断写锁是否该阻塞, 或者
                writerShouldBlock() ||
                        // 尝试更改计数失败
                        !compareAndSetState(c, c + acquires)
        )
        {
            // 获得锁失败
            return false;
        }
        // 获得锁成功
        setExclusiveOwnerThread(current);
        return true;
    }

    // 非公平锁 writerShouldBlock 总是返回 false, 无需阻塞
    final boolean writerShouldBlock()
    {
        return false;
    }


    //写锁上锁
    //**********************************************************************
    //写锁释放


    // ... 省略无关代码

    // WriteLock 方法, 方便阅读, 放在此处
    public void unlock()
    {
        sync.release(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean release(int arg)
    {
        // 尝试释放写锁成功
        if (tryRelease(arg))
        {
            // unpark AQS 中等待的线程
            Node h = head;
            if (h != null && h.waitStatus != 0)
            {
                unparkSuccessor(h);
            }
            return true;
        }
        return false;
    }

    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryRelease(int releases)
    {
        if (!isHeldExclusively())
        {
            throw new IllegalMonitorStateException();
        }
        int nextc = getState() - releases;
        // 因为可重入的原因, 写锁计数为 0, 才算释放成功
        boolean free = exclusiveCount(nextc) == 0;
        if (free)
        {
            setExclusiveOwnerThread(null);
        }
        setState(nextc);
        return free;
    }


    //写锁释放
    //***********************************************************
    //读锁上锁


    // ReadLock 方法, 方便阅读, 放在此处
    public void lock()
    {
        sync.acquireShared(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final void acquireShared(int arg)
    {
        // tryAcquireShared 返回负数, 表示获取读锁失败
        if (tryAcquireShared(arg) < 0)
        {
            doAcquireShared(arg);
        }
    }

    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final int tryAcquireShared(int unused)
    {
        Thread current = Thread.currentThread();
        int c = getState();
        // 如果是其它线程持有写锁, 获取读锁失败
        if (
                exclusiveCount(c) != 0 &&
                        getExclusiveOwnerThread() != current
        )
        {
            return -1;
        }
        int r = sharedCount(c);
        if (
            // 读锁不该阻塞(如果老二是写锁，读锁该阻塞), 并且
                !readerShouldBlock() &&
                        // 小于读锁计数, 并且
                        r < MAX_COUNT &&
                        // 尝试增加计数成功
                        compareAndSetState(c, c + SHARED_UNIT)
        )
        {
            // ... 省略不重要的代码
            return 1;
        }
        return fullTryAcquireShared(current);
    }

    // 非公平锁 readerShouldBlock 看 AQS 队列中第一个节点是否是写锁
    // true 则该阻塞, false 则不阻塞
    final boolean readerShouldBlock()
    {
        return apparentlyFirstQueuedIsExclusive();
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    // 与 tryAcquireShared 功能类似, 但会不断尝试 for (;;) 获取读锁, 执行过程中无阻塞
    final int fullTryAcquireShared(Thread current)
    {
        HoldCounter rh = null;
        for (; ; )
        {
            int c = getState();
            if (exclusiveCount(c) != 0)
            {
                if (getExclusiveOwnerThread() != current)
                {
                    return -1;
                }
            }
            else if (readerShouldBlock())
            {
                // ... 省略不重要的代码
            }
            if (sharedCount(c) == MAX_COUNT)
            {
                throw new Error("Maximum lock count exceeded");
            }
            if (compareAndSetState(c, c + SHARED_UNIT))
            {
                // ... 省略不重要的代码
                return 1;
            }
        }
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    private void doAcquireShared(int arg)
    {
        // 将当前线程关联到一个 Node 对象上, 模式为共享模式
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try
        {
            boolean interrupted = false;
            for (; ; )
            {
                final Node p = node.predecessor();
                if (p == head)
                {
                    // 再一次尝试获取读锁
                    int r = tryAcquireShared(arg);
                    // 成功
                    if (r >= 0)
                    {
                        // ㈠
                        // r 表示可用资源数, 在这里总是 1 允许传播
                        //（唤醒 AQS 中下一个 Share 节点）
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                        {
                            selfInterrupt();
                        }
                        failed = false;
                        return;
                    }
                }
                if (
                    // 是否在获取读锁失败时阻塞（前一个阶段 waitStatus == Node.SIGNAL）
                        shouldParkAfterFailedAcquire(p, node) &&
                                // park 当前线程
                                parkAndCheckInterrupt()
                )
                {
                    interrupted = true;
                }
            }
        }
        finally
        {
            if (failed)
            {
                cancelAcquire(node);
            }
        }
    }

    // ㈠ AQS 继承过来的方法, 方便阅读, 放在此处
    private void setHeadAndPropagate(Node node, int propagate)
    {
        Node h = head; // Record old head for check below
        // 设置自己为 head
        setHead(node);

        // propagate 表示有共享资源（例如共享读锁或信号量）
        // 原 head waitStatus == Node.SIGNAL 或 Node.PROPAGATE
        // 现在 head waitStatus == Node.SIGNAL 或 Node.PROPAGATE
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
                (h = head) == null || h.waitStatus < 0)
        {
            Node s = node.next;
            // 如果是最后一个节点或者是等待共享读锁的节点
            if (s == null || s.isShared())
            {
                // 进入 ㈡
                doReleaseShared();
            }
        }
    }

    // ㈡ AQS 继承过来的方法, 方便阅读, 放在此处
    private void doReleaseShared()
    {
        // 如果 head.waitStatus == Node.SIGNAL ==> 0 成功, 下一个节点 unpark
        // 如果 head.waitStatus == 0 ==> Node.PROPAGATE, 为了解决 bug, 见后面分析
        for (; ; )
        {
            Node h = head;
            // 队列还有节点
            if (h != null && h != tail)
            {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL)
                {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    {
                        continue; // loop to recheck cases
                    }
                    // 下一个节点 unpark 如果成功获取读锁
                    // 并且下下个节点还是 shared, 继续 doReleaseShared
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                {
                    continue; // loop on failed CAS
                }
            }
            if (h == head) // loop if head changed
            {
                break;
            }
        }
    }


    //读锁上锁
    //***************************************************************
    //读锁释放


    // ReadLock 方法, 方便阅读, 放在此处
    public void unlock()
    {
        sync.releaseShared(1);
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    public final boolean releaseShared(int arg)
    {
        if (tryReleaseShared(arg))
        {
            doReleaseShared();
            return true;
        }
        return false;
    }

    // Sync 继承过来的方法, 方便阅读, 放在此处
    protected final boolean tryReleaseShared(int unused)
    {
        // ... 省略不重要的代码
        for (; ; )
        {
            int c = getState();
            int nextc = c - SHARED_UNIT;
            if (compareAndSetState(c, nextc))
            {
                // 读锁的计数不会影响其它获取读锁线程, 但会影响其它获取写锁线程
                // 计数为 0 才是真正释放
                return nextc == 0;
            }
        }
    }

    // AQS 继承过来的方法, 方便阅读, 放在此处
    private void doReleaseShared()
    {
        // 如果 head.waitStatus == Node.SIGNAL ==> 0 成功, 下一个节点 unpark
        // 如果 head.waitStatus == 0 ==> Node.PROPAGATE
        for (; ; )
        {
            Node h = head;
            if (h != null && h != tail)
            {
                int ws = h.waitStatus;
                // 如果有其它线程也在释放读锁，那么需要将 waitStatus 先改为 0
                // 防止 unparkSuccessor 被多次执行
                if (ws == Node.SIGNAL)
                {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    {
                        continue; // loop to recheck cases
                    }
                    unparkSuccessor(h);
                }
                // 如果已经是 0 了，改为 -3，用来解决传播性，见后文信号量 bug 分析
                else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                {
                    continue; // loop on failed CAS
                }
            }
            if (h == head) // loop if head changed
            {
                break;
            }
        }
    }

}
```









#### StampedLock

该类自 JDK 8 加入，是为了进一步优化读性能，它的特点是在使用读锁、写锁时都必须配合戳使用



加解读锁：

```
long stamp = lock.readLock();
lock.unlockRead(stamp);
```



加解写锁：

```
long stamp = lock.writeLock();
lock.unlockWrite(stamp);
```



乐观读，StampedLock 支持 tryOptimisticRead() 方法（乐观读），读取完毕后需要做一次 戳校验 如果校验通 过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全

```
long stamp = lock.tryOptimisticRead();
// 验戳
if(!lock.validate(stamp))
{
 // 锁升级
}

```





测试两个读

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.StampedLock;

/**
 * Project name(项目名称)：java并发编程_StampedLock
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 11:00
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 数据
     */
    private int data;

    /**
     * StampedLock
     */
    private final StampedLock stampedLock = new StampedLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * Instantiates a new Test.
     *
     * @param data the data
     */
    public Test(int data)
    {
        this.data = data;
    }

    /**
     * 睡眠
     *
     * @param time 时间
     */
    private static void sleep(long time)
    {
        try
        {
            Thread.sleep(time);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }


    /**
     * 读
     *
     * @param sleepTime 睡眠时间
     * @return int
     */
    public int read(int sleepTime)
    {
        long read = stampedLock.tryOptimisticRead();
        sleep(sleepTime);
        //验证
        if (stampedLock.validate(read))
        {
            log.debug("直接返回数据，" + read);
            return data;
        }
        //验证失败
        //锁升级
        log.debug("锁升级为读锁，尝试获取读锁");
        long readLock = stampedLock.readLock();
        try
        {
            log.debug("读锁获取成功，" + readLock);
            sleep(sleepTime);
            return data;
        }
        finally
        {
            log.debug("释放读锁");
            stampedLock.unlockRead(readLock);
        }
    }


    /**
     * 写
     *
     * @param newData   新数据
     * @param sleepTime 睡眠时间
     */
    public void write(int newData, int sleepTime)
    {
        //获取写锁
        log.debug("尝试获取写锁");
        long writeLock = stampedLock.writeLock();
        try
        {
            log.debug("获取写锁成功");
            sleep(sleepTime);
            this.data = newData;
        }
        finally
        {
            log.debug("释放写锁");
            stampedLock.unlockWrite(writeLock);
        }
    }


    /**
     * main方法
     *
     * @param args 参数
     */
    public static void main(String[] args)
    {
        Test t = new Test(10);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                int read = t.read(30);
                log.debug("结果：" + read);
            }
        }, "read1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                int read = t.read(30);
                log.debug("结果：" + read);
            }
        }, "read2").start();
    }
}
```



运行结果：

```sh
2022-09-13  11:21:52.181  [read1] DEBUG mao.t1.Test:  直接返回数据，256
2022-09-13  11:21:52.183  [read1] DEBUG mao.t1.Test:  结果：10
2022-09-13  11:21:52.192  [read2] DEBUG mao.t1.Test:  直接返回数据，256
2022-09-13  11:21:52.193  [read2] DEBUG mao.t1.Test:  结果：10
```





测试读写



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_StampedLock
 * Package(包名): mao.t1
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 11:22
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    private static final Logger log = LoggerFactory.getLogger(Test2.class);

    public static void main(String[] args)
    {
        Test t = new Test(10);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.write(20,500);
            }
        }, "write").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                int read = t.read(500);
                log.debug("结果：" + read);
            }
        }, "read").start();
    }
}
```



运行结果：

```sh
2022-09-13  12:50:54.500  [write] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-13  12:50:54.502  [write] DEBUG mao.t1.Test:  获取写锁成功
2022-09-13  12:50:55.002  [read] DEBUG mao.t1.Test:  锁升级为读锁，尝试获取读锁
2022-09-13  12:50:55.004  [write] DEBUG mao.t1.Test:  释放写锁
2022-09-13  12:50:55.006  [read] DEBUG mao.t1.Test:  读锁获取成功，513
2022-09-13  12:50:55.509  [read] DEBUG mao.t1.Test:  释放读锁
2022-09-13  12:50:55.509  [read] DEBUG mao.t1.Test2:  结果：20
```

```sh
2022-09-13  12:52:25.510  [write] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-13  12:52:25.512  [write] DEBUG mao.t1.Test:  获取写锁成功
2022-09-13  12:52:26.016  [write] DEBUG mao.t1.Test:  释放写锁
2022-09-13  12:52:26.016  [read] DEBUG mao.t1.Test:  锁升级为读锁，尝试获取读锁
2022-09-13  12:52:26.019  [read] DEBUG mao.t1.Test:  读锁获取成功，513
2022-09-13  12:52:26.524  [read] DEBUG mao.t1.Test:  释放读锁
2022-09-13  12:52:26.524  [read] DEBUG mao.t1.Test2:  结果：20
```



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_StampedLock
 * Package(包名): mao.t1
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 11:22
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    private static final Logger log = LoggerFactory.getLogger(Test2.class);

    public static void main(String[] args) throws InterruptedException
    {
        Test t = new Test(10);


        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                int read = t.read(500);
                log.debug("结果：" + read);
            }
        }, "read").start();

        Thread.sleep(400);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.write(20,500);
            }
        }, "write").start();
    }
}
```



```sh
2022-09-13  12:56:09.946  [write] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-13  12:56:09.948  [write] DEBUG mao.t1.Test:  获取写锁成功
2022-09-13  12:56:10.035  [read] DEBUG mao.t1.Test:  锁升级为读锁，尝试获取读锁
2022-09-13  12:56:10.451  [write] DEBUG mao.t1.Test:  释放写锁
2022-09-13  12:56:10.453  [read] DEBUG mao.t1.Test:  读锁获取成功，513
2022-09-13  12:56:10.957  [read] DEBUG mao.t1.Test:  释放读锁
2022-09-13  12:56:10.957  [read] DEBUG mao.t1.Test2:  结果：20
```



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_StampedLock
 * Package(包名): mao.t1
 * Class(类名): Test2
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 11:22
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test2
{
    private static final Logger log = LoggerFactory.getLogger(Test2.class);

    public static void main(String[] args) throws InterruptedException
    {
        Test t = new Test(10);


        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                int read = t.read(500);
                log.debug("结果：" + read);
            }
        }, "read").start();

        Thread.sleep(600);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.write(20,500);
            }
        }, "write").start();
    }
}
```



```sh
2022-09-13  12:57:27.570  [read] DEBUG mao.t1.Test:  直接返回数据，256
2022-09-13  12:57:27.572  [read] DEBUG mao.t1.Test2:  结果：10
2022-09-13  12:57:27.658  [write] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-13  12:57:27.658  [write] DEBUG mao.t1.Test:  获取写锁成功
2022-09-13  12:57:28.164  [write] DEBUG mao.t1.Test:  释放写锁
```





测试两个写

```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：java并发编程_StampedLock
 * Package(包名): mao.t1
 * Class(类名): Test3
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 12:59
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test3
{
    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test3.class);

    public static void main(String[] args)
    {
        Test t = new Test(10);

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.write(20, 500);
            }
        }, "write1").start();

        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                t.write(30, 500);
            }
        }, "write2").start();

        log.debug("结果：" + t.read(10));
    }
}
```



运行结果：

```sh
2022-09-13  13:04:09.981  [write1] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-13  13:04:09.981  [write2] DEBUG mao.t1.Test:  尝试获取写锁
2022-09-13  13:04:09.983  [write1] DEBUG mao.t1.Test:  获取写锁成功
2022-09-13  13:04:09.999  [main] DEBUG mao.t1.Test:  锁升级为读锁，尝试获取读锁
2022-09-13  13:04:10.490  [write1] DEBUG mao.t1.Test:  释放写锁
2022-09-13  13:04:10.490  [write2] DEBUG mao.t1.Test:  获取写锁成功
2022-09-13  13:04:10.998  [write2] DEBUG mao.t1.Test:  释放写锁
2022-09-13  13:04:11.001  [main] DEBUG mao.t1.Test:  读锁获取成功，769
2022-09-13  13:04:11.013  [main] DEBUG mao.t1.Test:  释放读锁
2022-09-13  13:04:11.013  [main] DEBUG mao.t1.Test3:  结果：30
```





* StampedLock 不支持条件变量
* StampedLock 不支持可重入





测试写锁重入



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.locks.StampedLock;

/**
 * Project name(项目名称)：java并发编程_StampedLock
 * Package(包名): mao.t1
 * Class(类名): Test4
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 13:06
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test4
{

    /**
     * 锁
     */
    private static final StampedLock stampedLock = new StampedLock();

    private static final Logger log = LoggerFactory.getLogger(Test4.class);

    public static void main(String[] args)
    {
        m1();
    }

    private static void m1()
    {
        log.debug("尝试获取写锁");
        long writeLock = stampedLock.writeLock();
        try
        {
            log.debug("获取写锁成功");
            try
            {
                Thread.sleep(200);
                m2();
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
        finally
        {
            log.debug("释放写锁");
            stampedLock.unlockWrite(writeLock);
        }
    }

    private static void m2()
    {
        log.debug("尝试获取写锁");
        long writeLock = stampedLock.writeLock();
        try
        {
            log.debug("获取写锁成功");
            try
            {
                Thread.sleep(200);
                m3();
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
        finally
        {
            log.debug("释放写锁");
            stampedLock.unlockWrite(writeLock);
        }
    }

    private static void m3()
    {
        log.debug("尝试获取写锁");
        long writeLock = stampedLock.writeLock();
        try
        {
            log.debug("获取写锁成功");
            try
            {
                Thread.sleep(200);
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
        finally
        {
            log.debug("释放写锁");
            stampedLock.unlockWrite(writeLock);
        }
    }
}
```



运行结果：

```sh
2022-09-13  13:13:02.555  [main] DEBUG mao.t1.Test4:  尝试获取写锁
2022-09-13  13:13:02.557  [main] DEBUG mao.t1.Test4:  获取写锁成功
2022-09-13  13:13:02.764  [main] DEBUG mao.t1.Test4:  尝试获取写锁
...
```









### Semaphore

信号量，用来限制能同时访问共享资源的线程上限



#### 使用



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.Semaphore;

/**
 * Project name(项目名称)：java并发编程_Semaphore
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 18:42
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    /**
     * 信号量
     */
    private static final Semaphore semaphore = new Semaphore(3);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        for (int i = 0; i < 20; i++)
        {
            int finalI = i;
            new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    try
                    {
                        //获取一个许可
                        semaphore.acquire();
                        log.debug("线程" + (finalI + 1) + "开始运行");
                        Thread.sleep(1000);
                        log.debug("线程" + (finalI + 1) + "运行结束");
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                    finally
                    {
                        //释放许可
                        semaphore.release();
                    }
                }
            }, "t" + (i + 1)).start();
        }
    }
}
```



运行结果：

```sh
2022-09-13  18:49:31.106  [t3] DEBUG mao.t1.Test:  线程3开始运行
2022-09-13  18:49:31.106  [t2] DEBUG mao.t1.Test:  线程2开始运行
2022-09-13  18:49:31.106  [t1] DEBUG mao.t1.Test:  线程1开始运行
2022-09-13  18:49:32.116  [t2] DEBUG mao.t1.Test:  线程2运行结束
2022-09-13  18:49:32.116  [t3] DEBUG mao.t1.Test:  线程3运行结束
2022-09-13  18:49:32.116  [t1] DEBUG mao.t1.Test:  线程1运行结束
2022-09-13  18:49:32.116  [t6] DEBUG mao.t1.Test:  线程6开始运行
2022-09-13  18:49:32.116  [t5] DEBUG mao.t1.Test:  线程5开始运行
2022-09-13  18:49:32.116  [t7] DEBUG mao.t1.Test:  线程7开始运行
2022-09-13  18:49:33.124  [t7] DEBUG mao.t1.Test:  线程7运行结束
2022-09-13  18:49:33.124  [t5] DEBUG mao.t1.Test:  线程5运行结束
2022-09-13  18:49:33.124  [t6] DEBUG mao.t1.Test:  线程6运行结束
2022-09-13  18:49:33.124  [t4] DEBUG mao.t1.Test:  线程4开始运行
2022-09-13  18:49:33.124  [t8] DEBUG mao.t1.Test:  线程8开始运行
2022-09-13  18:49:33.124  [t9] DEBUG mao.t1.Test:  线程9开始运行
2022-09-13  18:49:34.136  [t8] DEBUG mao.t1.Test:  线程8运行结束
2022-09-13  18:49:34.136  [t4] DEBUG mao.t1.Test:  线程4运行结束
2022-09-13  18:49:34.136  [t9] DEBUG mao.t1.Test:  线程9运行结束
2022-09-13  18:49:34.136  [t10] DEBUG mao.t1.Test:  线程10开始运行
2022-09-13  18:49:34.136  [t11] DEBUG mao.t1.Test:  线程11开始运行
2022-09-13  18:49:34.136  [t12] DEBUG mao.t1.Test:  线程12开始运行
2022-09-13  18:49:35.144  [t10] DEBUG mao.t1.Test:  线程10运行结束
2022-09-13  18:49:35.144  [t11] DEBUG mao.t1.Test:  线程11运行结束
2022-09-13  18:49:35.144  [t12] DEBUG mao.t1.Test:  线程12运行结束
2022-09-13  18:49:35.144  [t13] DEBUG mao.t1.Test:  线程13开始运行
2022-09-13  18:49:35.144  [t14] DEBUG mao.t1.Test:  线程14开始运行
2022-09-13  18:49:35.144  [t15] DEBUG mao.t1.Test:  线程15开始运行
2022-09-13  18:49:36.150  [t13] DEBUG mao.t1.Test:  线程13运行结束
2022-09-13  18:49:36.150  [t15] DEBUG mao.t1.Test:  线程15运行结束
2022-09-13  18:49:36.150  [t14] DEBUG mao.t1.Test:  线程14运行结束
2022-09-13  18:49:36.150  [t16] DEBUG mao.t1.Test:  线程16开始运行
2022-09-13  18:49:36.150  [t19] DEBUG mao.t1.Test:  线程19开始运行
2022-09-13  18:49:36.150  [t18] DEBUG mao.t1.Test:  线程18开始运行
2022-09-13  18:49:37.159  [t18] DEBUG mao.t1.Test:  线程18运行结束
2022-09-13  18:49:37.159  [t16] DEBUG mao.t1.Test:  线程16运行结束
2022-09-13  18:49:37.159  [t19] DEBUG mao.t1.Test:  线程19运行结束
2022-09-13  18:49:37.159  [t20] DEBUG mao.t1.Test:  线程20开始运行
2022-09-13  18:49:37.159  [t17] DEBUG mao.t1.Test:  线程17开始运行
2022-09-13  18:49:38.170  [t20] DEBUG mao.t1.Test:  线程20运行结束
2022-09-13  18:49:38.170  [t17] DEBUG mao.t1.Test:  线程17运行结束
```





#### 简单连接池Semaphore实现



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.Connection;
import java.util.concurrent.Semaphore;
import java.util.concurrent.atomic.AtomicIntegerArray;

/**
 * Project name(项目名称)：java并发编程_Semaphore
 * Package(包名): mao.t2
 * Class(类名): Pool
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/13
 * Time(创建时间)： 18:53
 * Version(版本): 1.0
 * Description(描述)： 用 Semaphore 实现简单连接池。线程数和数据库连接数是相等的
 */

public class Pool
{
    /**
     * 池大小
     */
    private final int poolSize;

    /**
     * 连接对象数组
     */
    private final Connection[] connections;

    /**
     * 连接状态数组 0 表示空闲， 1 表示繁忙
     */
    private final AtomicIntegerArray states;

    /**
     * 信号量
     */
    private final Semaphore semaphore;

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Pool.class);

    /**
     * 构造方法
     *
     * @param poolSize 池大小
     */
    public Pool(int poolSize)
    {
        if (poolSize <= 0)
        {
            throw new IllegalArgumentException("poolSize must >0");
        }
        this.poolSize = poolSize;
        this.semaphore = new Semaphore(poolSize);
        this.connections = new Connection[poolSize];
        this.states = new AtomicIntegerArray(new int[poolSize]);
        for (int i = 0; i < poolSize; i++)
        {
            connections[i] = new MockConnection("连接" + (i + 1));
        }
    }

    /**
     * 借连接
     *
     * @return {@link Connection}
     */
    public Connection borrow()
    {
        try
        {
            //获取许可
            semaphore.acquire();
            for (int i = 0; i < poolSize; i++)
            {
                //遍历是否有空闲的连接
                if (states.get(i) == 0)
                {
                    while (true)
                    {
                        if (states.compareAndSet(i, 0, 1))
                        {
                            log.debug("获取连接：" + connections[i]);
                            return connections[i];
                        }
                        Thread.yield();
                    }
                }
            }
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
            return null;
        }
        return null;
    }


    /**
     * 归还连接
     *
     * @param connection 连接
     */
    public void free(Connection connection)
    {
        try
        {
            for (int i = 0; i < poolSize; i++)
            {
                if (connections[i] == connection)
                {
                    states.set(i, 0);
                    log.debug("归还连接：" + connection);
                    break;
                }
            }
        }
        finally
        {
            //释放许可
            semaphore.release();
        }


    }

}

class MockConnection implements Connection
{
    //todo
}
```





####  Semaphore 原理

Semaphore 有点像一个停车场，permits 就好像停车位数量，当线程获得了 permits 就像是获得了停车位，然后 停车场显示空余车位减一



刚开始，permits（state）为 3，这时 5 个线程来获取资源：



![image-20220913191455751](img/java并发编程学习笔记/image-20220913191455751.png)



假设其中 Thread-1，Thread-2，Thread-4 cas 竞争成功，而 Thread-0 和 Thread-3 竞争失败，进入 AQS 队列 park 阻塞



![image-20220913191625081](img/java并发编程学习笔记/image-20220913191625081.png)



这时 Thread-4 释放了 permits，状态如下



![image-20220913191707772](img/java并发编程学习笔记/image-20220913191707772.png)



接下来 Thread-0 竞争成功，permits 再次设置为 0，设置自己为 head 节点，断开原来的 head 节点，unpark 接下来的 Thread-3 节点，但由于 permits 是 0，因此 Thread-3 在尝试不成功后再次进入 park 状态



![image-20220913191916405](img/java并发编程学习笔记/image-20220913191916405.png)







### CountdownLatch

用来进行线程同步协作，等待所有线程完成倒计时

其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一



#### 使用



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.CountDownLatch;

/**
 * Project name(项目名称)：java并发编程_CountdownLatch
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 12:10
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    private static final CountDownLatch countDownLatch = new CountDownLatch(4);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * 运行
     *
     * @param sleepTime 睡眠时间
     */
    public static void run(long sleepTime)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    log.debug("开始运行");
                    try
                    {
                        Thread.sleep(sleepTime);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                finally
                {
                    log.debug("结束运行");
                    countDownLatch.countDown();
                }
            }
        }).start();
    }

    public static void main(String[] args) throws InterruptedException
    {
        run(1000);
        run(3000);
        run(2000);
        run(500);
        countDownLatch.await();
        log.debug("主线程开始运行");
    }
}
```



运行结果：

```sh
2022-09-14  12:17:01.289  [Thread-3] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:01.289  [Thread-0] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:01.289  [Thread-1] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:01.289  [Thread-2] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:01.798  [Thread-3] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:02.304  [Thread-0] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:03.302  [Thread-2] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:04.301  [Thread-1] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:04.301  [main] DEBUG mao.t1.Test:  主线程开始运行
```





```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.CountDownLatch;

/**
 * Project name(项目名称)：java并发编程_CountdownLatch
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 12:10
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    private static final CountDownLatch countDownLatch = new CountDownLatch(3);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    /**
     * 运行
     *
     * @param sleepTime 睡眠时间
     */
    public static void run(long sleepTime)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    log.debug("开始运行");
                    try
                    {
                        Thread.sleep(sleepTime);
                    }
                    catch (InterruptedException e)
                    {
                        e.printStackTrace();
                    }
                }
                finally
                {
                    log.debug("结束运行");
                    countDownLatch.countDown();
                }
            }
        }).start();
    }

    public static void main(String[] args) throws InterruptedException
    {
        run(1000);
        run(3000);
        run(2000);
        run(500);
        countDownLatch.await();
        log.debug("主线程开始运行");
    }
}
```



```sh
2022-09-14  12:17:55.513  [Thread-2] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:55.513  [Thread-1] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:55.513  [Thread-0] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:55.513  [Thread-3] DEBUG mao.t1.Test:  开始运行
2022-09-14  12:17:56.031  [Thread-3] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:56.524  [Thread-0] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:57.524  [Thread-2] DEBUG mao.t1.Test:  结束运行
2022-09-14  12:17:57.524  [main] DEBUG mao.t1.Test:  主线程开始运行
2022-09-14  12:17:58.529  [Thread-1] DEBUG mao.t1.Test:  结束运行
```





可以配合线程池使用



```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Project name(项目名称)：java并发编程_CountdownLatch
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 12:22
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final CountDownLatch countDownLatch = new CountDownLatch(5);

    /**
     * 线程池
     */
    private static final ExecutorService threadPool = Executors.newFixedThreadPool(2);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args) throws InterruptedException
    {
        for (int i = 0; i < 5; i++)
        {
            threadPool.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    try
                    {
                        log.debug("开始运行");
                        try
                        {
                            Thread.sleep(1000);
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                    }
                    finally
                    {
                        log.debug("结束运行");
                        countDownLatch.countDown();
                    }
                }
            });
        }
        countDownLatch.await();
        log.debug("主线程开始运行");
    }
}
```



运行结果：

```sh
2022-09-14  12:26:37.211  [pool-1-thread-1] DEBUG mao.t2.Test:  开始运行
2022-09-14  12:26:37.211  [pool-1-thread-2] DEBUG mao.t2.Test:  开始运行
2022-09-14  12:26:38.227  [pool-1-thread-1] DEBUG mao.t2.Test:  结束运行
2022-09-14  12:26:38.227  [pool-1-thread-2] DEBUG mao.t2.Test:  结束运行
2022-09-14  12:26:38.227  [pool-1-thread-2] DEBUG mao.t2.Test:  开始运行
2022-09-14  12:26:38.227  [pool-1-thread-1] DEBUG mao.t2.Test:  开始运行
2022-09-14  12:26:39.239  [pool-1-thread-2] DEBUG mao.t2.Test:  结束运行
2022-09-14  12:26:39.239  [pool-1-thread-1] DEBUG mao.t2.Test:  结束运行
2022-09-14  12:26:39.239  [pool-1-thread-2] DEBUG mao.t2.Test:  开始运行
2022-09-14  12:26:40.252  [pool-1-thread-2] DEBUG mao.t2.Test:  结束运行
2022-09-14  12:26:40.252  [main] DEBUG mao.t2.Test:  主线程开始运行
```







#### 游戏加载



```java
package mao.t3;

import java.awt.*;
import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * Project name(项目名称)：java并发编程_CountdownLatch
 * Package(包名): mao.t3
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 12:28
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    public static void main(String[] args) throws InterruptedException
    {
        AtomicInteger num = new AtomicInteger(0);

        ExecutorService service = Executors.newFixedThreadPool(10);

        CountDownLatch countDownLatch = new CountDownLatch(10);

        String[] all = new String[10];
        Random random = new Random();

        for (int j = 0; j < 10; j++)
        {
            int x = j;
            service.submit(new Runnable()
            {
                @Override
                public void run()
                {
                    for (int i = 0; i <= 100; i++)
                    {
                        try
                        {
                            Thread.sleep(random.nextInt(500));
                        }
                        catch (InterruptedException ignored)
                        {
                        }
                        all[x] = Thread.currentThread().getName() + "(" + (i + "%") + ")";
                        System.out.print("\r" + Arrays.toString(all));
                    }
                    countDownLatch.countDown();
                }
            });
        }

        countDownLatch.await();
        System.out.println();
        Toolkit.getDefaultToolkit().beep();
        System.out.println("加载完毕！！！");
        service.shutdown();
    }
}
```







### CyclicBarrier

循环栅栏，用来进行线程协作，等待线程满足某个计数。构造时设置**计数个数**，每个线程执 行到某个需要“同步”的时刻调用 await() 方法进行等待，当等待的线程数满足**计数个数**时，继续执行



```java
package mao.t1;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * Project name(项目名称)：java并发编程_CyclicBarrier
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 12:43
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{

    private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    log.debug("线程0开始运行");
                    cyclicBarrier.await();
                    log.debug("------------> 满足条件，开始运行");
                }
                catch (InterruptedException | BrokenBarrierException e)
                {
                    e.printStackTrace();
                }
            }
        }).start();


        for (int i = 0; i < 15; i++)
        {
            int finalI = i;
            new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    try
                    {
                        log.debug("线程" + (finalI + 1) + "开始运行");
                        cyclicBarrier.await();
                        log.debug("线程" + (finalI + 1) + "结束运行");
                    }
                    catch (InterruptedException | BrokenBarrierException e)
                    {
                        e.printStackTrace();
                    }
                }
            }).start();

            try
            {
                Thread.sleep(500);
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
    }
}
```



运行结果：

```sh
2022-09-14  13:00:54.829  [Thread-0] DEBUG mao.t1.Test:  线程0开始运行
2022-09-14  13:00:54.829  [Thread-1] DEBUG mao.t1.Test:  线程1开始运行
2022-09-14  13:00:55.334  [Thread-2] DEBUG mao.t1.Test:  线程2开始运行
2022-09-14  13:00:55.843  [Thread-3] DEBUG mao.t1.Test:  线程3开始运行
2022-09-14  13:00:56.348  [Thread-4] DEBUG mao.t1.Test:  线程4开始运行
2022-09-14  13:00:56.857  [Thread-5] DEBUG mao.t1.Test:  线程5开始运行
2022-09-14  13:00:57.363  [Thread-6] DEBUG mao.t1.Test:  线程6开始运行
2022-09-14  13:00:57.870  [Thread-7] DEBUG mao.t1.Test:  线程7开始运行
2022-09-14  13:00:58.380  [Thread-8] DEBUG mao.t1.Test:  线程8开始运行
2022-09-14  13:00:58.887  [Thread-9] DEBUG mao.t1.Test:  线程9开始运行
2022-09-14  13:00:58.888  [Thread-0] DEBUG mao.t1.Test:  ------------> 满足条件，开始运行
2022-09-14  13:00:58.888  [Thread-3] DEBUG mao.t1.Test:  线程3结束运行
2022-09-14  13:00:58.888  [Thread-7] DEBUG mao.t1.Test:  线程7结束运行
2022-09-14  13:00:58.888  [Thread-9] DEBUG mao.t1.Test:  线程9结束运行
2022-09-14  13:00:58.888  [Thread-2] DEBUG mao.t1.Test:  线程2结束运行
2022-09-14  13:00:58.888  [Thread-1] DEBUG mao.t1.Test:  线程1结束运行
2022-09-14  13:00:58.888  [Thread-8] DEBUG mao.t1.Test:  线程8结束运行
2022-09-14  13:00:58.888  [Thread-6] DEBUG mao.t1.Test:  线程6结束运行
2022-09-14  13:00:58.888  [Thread-5] DEBUG mao.t1.Test:  线程5结束运行
2022-09-14  13:00:58.888  [Thread-4] DEBUG mao.t1.Test:  线程4结束运行
2022-09-14  13:00:59.393  [Thread-10] DEBUG mao.t1.Test:  线程10开始运行
2022-09-14  13:00:59.898  [Thread-11] DEBUG mao.t1.Test:  线程11开始运行
2022-09-14  13:01:00.406  [Thread-12] DEBUG mao.t1.Test:  线程12开始运行
2022-09-14  13:01:00.912  [Thread-13] DEBUG mao.t1.Test:  线程13开始运行
2022-09-14  13:01:01.419  [Thread-14] DEBUG mao.t1.Test:  线程14开始运行
2022-09-14  13:01:01.923  [Thread-15] DEBUG mao.t1.Test:  线程15开始运行
```







```java
package mao.t2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * Project name(项目名称)：java并发编程_CyclicBarrier
 * Package(包名): mao.t2
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 13:06
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class Test
{
    private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(10);

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(Test.class);

    public static void main(String[] args)
    {
        for (int i = 0; i < 300; i++)
        {
            int finalI = i;
            new Thread(new Runnable()
            {
                @Override
                public void run()
                {
                    try
                    {
                        log.debug("线程" + (finalI + 1) + "开始运行");
                        cyclicBarrier.await();
                        if (finalI % 10 == 0)
                        {
                            log.info("----------------->满足条件，开始运行");
                        }
                        else
                        {
                            log.debug("线程" + (finalI + 1) + "结束运行");
                        }
                    }
                    catch (InterruptedException | BrokenBarrierException e)
                    {
                        e.printStackTrace();
                    }
                }
            }).start();

            try
            {
                Thread.sleep(500);
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
    }
}
```



运行结果：

```sh
2022-09-14  13:08:12.608  [Thread-0] DEBUG mao.t2.Test:  线程1开始运行
2022-09-14  13:08:13.117  [Thread-1] DEBUG mao.t2.Test:  线程2开始运行
2022-09-14  13:08:13.621  [Thread-2] DEBUG mao.t2.Test:  线程3开始运行
2022-09-14  13:08:14.128  [Thread-3] DEBUG mao.t2.Test:  线程4开始运行
2022-09-14  13:08:14.633  [Thread-4] DEBUG mao.t2.Test:  线程5开始运行
2022-09-14  13:08:15.141  [Thread-5] DEBUG mao.t2.Test:  线程6开始运行
2022-09-14  13:08:15.646  [Thread-6] DEBUG mao.t2.Test:  线程7开始运行
2022-09-14  13:08:16.152  [Thread-7] DEBUG mao.t2.Test:  线程8开始运行
2022-09-14  13:08:16.658  [Thread-8] DEBUG mao.t2.Test:  线程9开始运行
2022-09-14  13:08:17.164  [Thread-9] DEBUG mao.t2.Test:  线程10开始运行
2022-09-14  13:08:17.165  [Thread-0] INFO  mao.t2.Test:  ----------------->满足条件，开始运行
2022-09-14  13:08:17.165  [Thread-9] DEBUG mao.t2.Test:  线程10结束运行
2022-09-14  13:08:17.165  [Thread-2] DEBUG mao.t2.Test:  线程3结束运行
2022-09-14  13:08:17.165  [Thread-7] DEBUG mao.t2.Test:  线程8结束运行
2022-09-14  13:08:17.165  [Thread-3] DEBUG mao.t2.Test:  线程4结束运行
2022-09-14  13:08:17.165  [Thread-8] DEBUG mao.t2.Test:  线程9结束运行
2022-09-14  13:08:17.165  [Thread-5] DEBUG mao.t2.Test:  线程6结束运行
2022-09-14  13:08:17.165  [Thread-6] DEBUG mao.t2.Test:  线程7结束运行
2022-09-14  13:08:17.165  [Thread-1] DEBUG mao.t2.Test:  线程2结束运行
2022-09-14  13:08:17.165  [Thread-4] DEBUG mao.t2.Test:  线程5结束运行
2022-09-14  13:08:17.670  [Thread-10] DEBUG mao.t2.Test:  线程11开始运行
2022-09-14  13:08:18.176  [Thread-11] DEBUG mao.t2.Test:  线程12开始运行
2022-09-14  13:08:18.684  [Thread-12] DEBUG mao.t2.Test:  线程13开始运行
2022-09-14  13:08:19.194  [Thread-13] DEBUG mao.t2.Test:  线程14开始运行
2022-09-14  13:08:19.701  [Thread-14] DEBUG mao.t2.Test:  线程15开始运行
2022-09-14  13:08:20.211  [Thread-15] DEBUG mao.t2.Test:  线程16开始运行
2022-09-14  13:08:20.720  [Thread-16] DEBUG mao.t2.Test:  线程17开始运行
2022-09-14  13:08:21.226  [Thread-17] DEBUG mao.t2.Test:  线程18开始运行
2022-09-14  13:08:21.733  [Thread-18] DEBUG mao.t2.Test:  线程19开始运行
2022-09-14  13:08:22.242  [Thread-19] DEBUG mao.t2.Test:  线程20开始运行
2022-09-14  13:08:22.242  [Thread-19] DEBUG mao.t2.Test:  线程20结束运行
2022-09-14  13:08:22.242  [Thread-11] DEBUG mao.t2.Test:  线程12结束运行
2022-09-14  13:08:22.242  [Thread-10] INFO  mao.t2.Test:  ----------------->满足条件，开始运行
2022-09-14  13:08:22.242  [Thread-15] DEBUG mao.t2.Test:  线程16结束运行
2022-09-14  13:08:22.242  [Thread-14] DEBUG mao.t2.Test:  线程15结束运行
2022-09-14  13:08:22.242  [Thread-16] DEBUG mao.t2.Test:  线程17结束运行
2022-09-14  13:08:22.242  [Thread-12] DEBUG mao.t2.Test:  线程13结束运行
2022-09-14  13:08:22.242  [Thread-18] DEBUG mao.t2.Test:  线程19结束运行
2022-09-14  13:08:22.242  [Thread-13] DEBUG mao.t2.Test:  线程14结束运行
2022-09-14  13:08:22.242  [Thread-17] DEBUG mao.t2.Test:  线程18结束运行
2022-09-14  13:08:22.749  [Thread-20] DEBUG mao.t2.Test:  线程21开始运行
2022-09-14  13:08:23.254  [Thread-21] DEBUG mao.t2.Test:  线程22开始运行
2022-09-14  13:08:23.763  [Thread-22] DEBUG mao.t2.Test:  线程23开始运行
2022-09-14  13:08:24.269  [Thread-23] DEBUG mao.t2.Test:  线程24开始运行
2022-09-14  13:08:24.778  [Thread-24] DEBUG mao.t2.Test:  线程25开始运行
2022-09-14  13:08:25.284  [Thread-25] DEBUG mao.t2.Test:  线程26开始运行
2022-09-14  13:08:25.792  [Thread-26] DEBUG mao.t2.Test:  线程27开始运行
2022-09-14  13:08:26.299  [Thread-27] DEBUG mao.t2.Test:  线程28开始运行
2022-09-14  13:08:26.810  [Thread-28] DEBUG mao.t2.Test:  线程29开始运行
2022-09-14  13:08:27.315  [Thread-29] DEBUG mao.t2.Test:  线程30开始运行
2022-09-14  13:08:27.316  [Thread-29] DEBUG mao.t2.Test:  线程30结束运行
2022-09-14  13:08:27.316  [Thread-20] INFO  mao.t2.Test:  ----------------->满足条件，开始运行
2022-09-14  13:08:27.316  [Thread-24] DEBUG mao.t2.Test:  线程25结束运行
2022-09-14  13:08:27.316  [Thread-21] DEBUG mao.t2.Test:  线程22结束运行
2022-09-14  13:08:27.316  [Thread-27] DEBUG mao.t2.Test:  线程28结束运行
2022-09-14  13:08:27.316  [Thread-22] DEBUG mao.t2.Test:  线程23结束运行
2022-09-14  13:08:27.316  [Thread-23] DEBUG mao.t2.Test:  线程24结束运行
2022-09-14  13:08:27.316  [Thread-25] DEBUG mao.t2.Test:  线程26结束运行
2022-09-14  13:08:27.316  [Thread-26] DEBUG mao.t2.Test:  线程27结束运行
2022-09-14  13:08:27.316  [Thread-28] DEBUG mao.t2.Test:  线程29结束运行
2022-09-14  13:08:27.822  [Thread-30] DEBUG mao.t2.Test:  线程31开始运行
2022-09-14  13:08:28.329  [Thread-31] DEBUG mao.t2.Test:  线程32开始运行
2022-09-14  13:08:28.837  [Thread-32] DEBUG mao.t2.Test:  线程33开始运行
2022-09-14  13:08:29.346  [Thread-33] DEBUG mao.t2.Test:  线程34开始运行
2022-09-14  13:08:29.854  [Thread-34] DEBUG mao.t2.Test:  线程35开始运行
2022-09-14  13:08:30.361  [Thread-35] DEBUG mao.t2.Test:  线程36开始运行
2022-09-14  13:08:30.867  [Thread-36] DEBUG mao.t2.Test:  线程37开始运行
2022-09-14  13:08:31.371  [Thread-37] DEBUG mao.t2.Test:  线程38开始运行
2022-09-14  13:08:31.878  [Thread-38] DEBUG mao.t2.Test:  线程39开始运行
2022-09-14  13:08:32.387  [Thread-39] DEBUG mao.t2.Test:  线程40开始运行
2022-09-14  13:08:32.387  [Thread-39] DEBUG mao.t2.Test:  线程40结束运行
2022-09-14  13:08:32.387  [Thread-31] DEBUG mao.t2.Test:  线程32结束运行
2022-09-14  13:08:32.387  [Thread-30] INFO  mao.t2.Test:  ----------------->满足条件，开始运行
2022-09-14  13:08:32.387  [Thread-34] DEBUG mao.t2.Test:  线程35结束运行
2022-09-14  13:08:32.387  [Thread-32] DEBUG mao.t2.Test:  线程33结束运行
2022-09-14  13:08:32.387  [Thread-35] DEBUG mao.t2.Test:  线程36结束运行
2022-09-14  13:08:32.387  [Thread-37] DEBUG mao.t2.Test:  线程38结束运行
2022-09-14  13:08:32.387  [Thread-38] DEBUG mao.t2.Test:  线程39结束运行
2022-09-14  13:08:32.387  [Thread-36] DEBUG mao.t2.Test:  线程37结束运行
2022-09-14  13:08:32.387  [Thread-33] DEBUG mao.t2.Test:  线程34结束运行
2022-09-14  13:08:32.896  [Thread-40] DEBUG mao.t2.Test:  线程41开始运行
...
```













### 线程安全集合类

线程安全集合类可以分为三大类：

* 遗留的线程安全集合如 Hashtable ， Vector
* 使用 Collections 装饰的线程安全集合，如：
  * Collections.synchronizedCollection
  * Collections.synchronizedList
  * Collections.synchronizedMap
  * Collections.synchronizedSet 
  * Collections.synchronizedNavigableMap 
  * Collections.synchronizedNavigableSet  
  * Collections.synchronizedSortedMap 
  * Collections.synchronizedSortedSet
* java.util.concurrent.*



 java.util.concurrent.*里面包含三类关键词：

* Blocking
* CopyOnWrite
* Concurrent



说明：

* Blocking 大部分实现基于锁，并提供用来阻塞的方法
* CopyOnWrite 之类容器修改开销相对较重
* Concurrent 类型的容器
  * 内部很多操作使用 cas 优化，一般可以提供较高吞吐量
  * 弱一致性
    * 遍历时弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍 历，这时内容是旧的
    * 求大小弱一致性，size 操作未必是 100% 准确
    * 读取弱一致性



遍历时如果发生了修改，对于非安全容器来讲，使用 fail-fast 机制也就是让遍历立刻失败，抛出 ConcurrentModificationException，不再继续遍历







#### ConcurrentHashMap

##### 单词计数



```java
package mao.t1;

import java.io.*;
import java.util.*;
import java.util.function.BiConsumer;
import java.util.function.Supplier;

/**
 * Project name(项目名称)：java并发编程_ConcurrentHashMap
 * Package(包名): mao.t1
 * Class(类名): Test
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/9/14
 * Time(创建时间)： 13:24
 * Version(版本): 1.0
 * Description(描述)： 单词计数
 */

public class Test
{
    private static final String word = "abcedfghijklmnopqrstuvwxyz";

    /**
     * 写
     */
    @SuppressWarnings("all")
    private static void write()
    {
        int length = word.length();
        int count = 200;
        List<String> list = new ArrayList<>(length * count);
        //加入到集合中
        for (int i = 0; i < length; i++)
        {
            char ch = word.charAt(i);
            for (int j = 0; j < count; j++)
            {
                list.add(String.valueOf(ch));
            }
        }
        //打乱
        Collections.shuffle(list);

        //写入到文件
        for (int i = 0; i < 26; i++)
        {
            new File("./file/").mkdir();

            File file = new File(".\\file\\" + (i + 1) + ".txt");
            if (!file.exists())
            {
                try
                {
                    file.createNewFile();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
            }
            try (PrintWriter out = new PrintWriter(
                    new OutputStreamWriter(
                            new FileOutputStream(file))))
            {
                String collect = String.join("\n", list.subList(i * count, (i + 1) * count));
                out.print(collect);
            }
            catch (IOException ignored)
            {
                //ignored.printStackTrace();
            }
        }
    }


    /**
     * 读
     *
     * @param supplier 供应商
     * @param consumer 消费者
     */
    private static <V> void read(Supplier<Map<String, V>> supplier,
                                 BiConsumer<Map<String, V>, List<String>> consumer)
    {
        Map<String, V> counterMap = supplier.get();
        List<Thread> threads = new ArrayList<>();
        for (int i = 1; i <= 26; i++)
        {
            int idx = i;
            Thread thread = new Thread(() ->
            {
                List<String> words = readFromFile(idx);
                //计数
                consumer.accept(counterMap, words);
            });
            threads.add(thread);
        }
        threads.forEach(Thread::start);
        threads.forEach(t ->
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

        //打印
        Set<String> words = counterMap.keySet();
        int i = 0;
        for (String word : words)
        {
            System.out.print(word + "=" + counterMap.get(word) + "\t\t");
            i++;
            if (i % 3 == 0)
            {
                System.out.println();
            }
        }
    }

    /**
     * 从文件读取
     *
     * @param i 文件序号
     * @return {@link List}<{@link String}>
     */
    public static List<String> readFromFile(int i)
    {
        ArrayList<String> words = new ArrayList<>();
        try (BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(new FileInputStream("./file/"
                + i + ".txt"))))
        {
            while (true)
            {
                String word = bufferedReader.readLine();
                if (word == null)
                {
                    break;
                }
                words.add(word);
            }
            return words;
        }
        catch (IOException e)
        {
            throw new RuntimeException(e);
        }
    }


    public static void main(String[] args)
    {
        write();

        long start = System.currentTimeMillis();

        read(new Supplier<Map<String, Integer>>()
        {
            @Override
            public Map<String, Integer> get()
            {
                return new HashMap<>();
            }
        }, new BiConsumer<Map<String, Integer>, List<String>>()
        {
            @Override
            public void accept(Map<String, Integer> stringIntegerMap, List<String> strings)
            {
                for (String word : strings)
                {
                    Integer counter = stringIntegerMap.get(word);
                    int newValue = counter == null ? 1 : counter + 1;
                    stringIntegerMap.put(word, newValue);
                }
            }
        });

        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                System.out.println("\n");
                long end = System.currentTimeMillis();
                System.out.println("运行时间：" + (end - start) + "ms");
            }
        }));
    }
    
}

```



参数：

* 一是提供一个 map 集合，用来存放每个单词的计数结果，key 为单词，value 为计数
* 二是提供一组操作，保证计数的安全性，会传递 map 集合以及单词 List



运行结果：

```sh
a=190		b=194		c=194		
d=194		e=194		f=193		
g=195		h=191		i=192		
j=180		k=196		l=192		
m=192		n=190		o=191		
p=193		q=195		r=197		
s=195		t=194		u=195		
v=195		w=195		x=191		
y=190		z=197		

运行时间：18ms
```

```sh
a=197		b=195		c=194		
d=197		e=194		f=195		
g=194		h=195		i=196		
j=199		k=196		l=194		
m=197		n=198		o=193		
p=196		q=184		r=187		
s=191		t=194		u=197		
v=189		w=192		x=144		
y=194		z=192		

运行时间：17ms
```

```sh
a=192		b=195		c=193		
d=194		e=189		f=192		
g=190		h=192		i=194		
j=191		k=195		l=191		
m=195		n=196		o=193		
p=195		q=196		r=194		
s=194		t=192		u=175		
v=196		w=198		x=194		
y=197		z=191		

运行时间：15ms
```



都小于200，正确结果应该都是200





可以使用同步锁，但是会影响并发



```java
public static void main(String[] args)
{
    write();

    long start = System.currentTimeMillis();

    read(new Supplier<Map<String, Integer>>()
    {
        @Override
        public Map<String, Integer> get()
        {
            return new HashMap<>();
        }
    }, new BiConsumer<Map<String, Integer>, List<String>>()
    {
        @Override
        public void accept(Map<String, Integer> stringIntegerMap, List<String> strings)
        {
            synchronized (this)
            {
                for (String word : strings)
                {
                    Integer counter = stringIntegerMap.get(word);
                    int newValue = counter == null ? 1 : counter + 1;
                    stringIntegerMap.put(word, newValue);
                }
            }
        }
    });

    Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
    {
        @Override
        public void run()
        {
            System.out.println("\n");
            long end = System.currentTimeMillis();
            System.out.println("运行时间：" + (end - start) + "ms");
        }
    }));
}
```



运行结果：

```sh
a=200		b=200		c=200		
d=200		e=200		f=200		
g=200		h=200		i=200		
j=200		k=200		l=200		
m=200		n=200		o=200		
p=200		q=200		r=200		
s=200		t=200		u=200		
v=200		w=200		x=200		
y=200		z=200		

运行时间：17ms
```





解决方案二：使用原子变量



```java
public static void main(String[] args)
{
    write();

    long start = System.currentTimeMillis();

    read(new Supplier<Map<String, LongAdder>>()
    {
        @Override
        public Map<String, LongAdder> get()
        {
            return new ConcurrentHashMap<>();
        }
    }, new BiConsumer<Map<String, LongAdder>, List<String>>()
    {
        @Override
        public void accept(Map<String, LongAdder> stringIntegerMap, List<String> strings)
        {
            for (String word : strings)
            {
                stringIntegerMap.computeIfAbsent(word, (key) -> new LongAdder()).increment();
            }
        }
    });

    Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
    {
        @Override
        public void run()
        {
            System.out.println("\n");
            long end = System.currentTimeMillis();
            System.out.println("运行时间：" + (end - start) + "ms");
        }
    }));
}
```



运行结果：

```sh
a=200		b=200		c=200		
d=200		e=200		f=200		
g=200		h=200		i=200		
j=200		k=200		l=200		
m=200		n=200		o=200		
p=200		q=200		r=200		
s=200		t=200		u=200		
v=200		w=200		x=200		
y=200		z=200		

运行时间：19ms
```





解决方案三：

```java
public static void main(String[] args)
{
    write();

    long start = System.currentTimeMillis();

    read(new Supplier<Map<String, Integer>>()
    {
        @Override
        public Map<String, Integer> get()
        {
            return new ConcurrentHashMap<>();
        }
    }, new BiConsumer<Map<String, Integer>, List<String>>()
    {
        @Override
        public void accept(Map<String, Integer> stringIntegerMap, List<String> strings)
        {
            for (String word : strings)
            {
                //stringIntegerMap.merge(word, 1, Integer::sum);
                stringIntegerMap.merge(word, 1, new BinaryOperator<Integer>()
                {
                    @Override
                    public Integer apply(Integer integer, Integer integer2)
                    {
                        //return integer+integer2;
                        return Integer.sum(integer, integer2);
                    }
                });
            }
        }
    });

    Runtime.getRuntime().addShutdownHook(new Thread(new Runnable()
    {
        @Override
        public void run()
        {
            System.out.println("\n");
            long end = System.currentTimeMillis();
            System.out.println("运行时间：" + (end - start) + "ms");
        }
    }));
}
```



运行结果：

```sh
a=200		b=200		c=200		
d=200		e=200		f=200		
g=200		h=200		i=200		
j=200		k=200		l=200		
m=200		n=200		o=200		
p=200		q=200		r=200		
s=200		t=200		u=200		
v=200		w=200		x=200		
y=200		z=200		

运行时间：17ms
```







##### 原理

