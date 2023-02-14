### 什么是虚拟线程

JDK19中的虚拟线程就是业界的协程。

因为协程是用户态的，线程是操作系统内核态的，所以协程仍然基于的是线程，一个线程可以承载多个协程。但如果所有协程都是基于一个线程，那样效率肯定不高，所以JDK19中协程会基于ForkJoinPool线程沲。利用多个线程来支持协程的运行，并且利用ForkJoinPool而不是普通的的ThreadPoolExecutor，可以支持大任务的拆分。

JDK19中的协程底层是基于ForkJoinPool的，相当于我们在利用协程执行Runnable时，底层会把Runnable提交到一个ForkJoinPool中去执行，我们可以通过下面的参数进行设置:

```bash
-Djdk.virtualThreadScheduler.parallelism=1
-Djdk.virtualThreadScheduler.maxPoolSize=1
```

这两个参数来设置ForkJoinPool的核心线程数和最大线程数：

- parallelism默认为Runtime.getRuntime().avaliableProcessors()
- maxPoolSize默认为256

ForkJoinPool中的线程在执行任务过程中，一旦线程阻塞了，比如:sleep、lock、io操作时，那么这个线程就会去执行ForkJoinPool中的其它任务，从而可以做到在一个线程在执行过程中，也能并发的执行多个任务，达到协程并发执行任务的效果。

### 普通线程

对于普通的线程来说，它的执行是串行的，底层的实现原理是基于队列的，所以需要一个一个的执行任务，代码示例如下：

```java
public class NormalThread {
  
  public static void main(String[] args) throws InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(1);
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
  }
  
  static class MyTask implements Runnable {
    
    @Override
    public void run() {
      System.out.println(Thread.currentThread());
      sleep();
      System.out.println(LocalDateTime.now());
    }
    
    void sleep() {
      try {
        Thread.sleep(Duration.ofSeconds(1));
      } catch(InterruptedException e) {
        e.printlnStackTrace();
      }
    } 
  }  
}
```

### 虚拟线程

为了提升效率，JDK19提供的虚拟线程可以很好在在执行任务发生阻塞时去执行其它的任务，代码示例如下：

```java
public class VirtualThread {
  
  public static void main(String[] args) throws InterruptedException {
    ExecutorService executorService = Executors.newVirtualThreadPerTaskExecutor();
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
    executorService.execute(new MyTask());
  }
  
  static class MyTask implements Runnable {
    
    @Override
    public void run() {
      System.out.println(Thread.currentThread());
      sleep();
      System.out.println(LocalDateTime.now());
    }
    
    void sleep() {
      try {
        Thread.sleep(Duration.ofSeconds(1));
      } catch(InterruptedException e) {
        e.printlnStackTrace();
      }
    } 
  }  
}
```



