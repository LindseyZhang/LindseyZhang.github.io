layout: post
title: Java Concurrency
categories: [Tech]
tags: [Java]
description:  Java 并发相关知识总结

---

1. ExecutorService ： JDK 提供的用于简化 Java 中异步任务执行的框架。

   * Executors 提供了创建 ExecutorService 的一系列工厂方法。

   Executors.newSingleThreadExecutor(); // 单线程执行器

   Executors.newFixedThreadPool(); // 可以指定线程池的大小

   Executors.newCachedThreadPool(); // 可以重用执行完任务的空闲线程。

   Executors.newScheduledThreadPool(); // 支持在指定时间后执行命令

   

   * 接收 Runnable 或 Callable 任务。

   execute(); // 无法获得执行结果及查看任务状态

   submit(); // 返回 Future 类型

   invokeAny(); // 接收一个任务集合，返回其中一个执行成功的任务的结果

   invokeAll(); // 接收一个任务集合，返回一个List Future

   * 停止

     ExecutorService 不会自动停止，需要调用方法：

     shutdown(); // 停止接收新任务，并在当前执行任务全部执行完成后停止。

     shutdownNow(); // 立即停止所有正在执行的任务，返回待处理任务的集合。

     ```
     // 推荐的 shut down 写法
     executorService.shutdown();
     try {
         if (!executorService.awaitTermination(800, TimeUnit.MILLISECONDS)) {
             executorService.shutdownNow();
         } 
     } catch (InterruptedException e) {
         executorService.shutdownNow();
     }
     ```

     

   2. Fork/ Join Framework -- Java 7 引入

   ForkJoinPool -- ExecutorService 的抽象类，比 ExecutorService 多了 work-stealing

   Java 8 中最方便的使用方法是使用: commonPool() 静态方法。

   Java 7 中： 自行创建并赋值给静态变量。

   

   ForkJoinTask  -- Future 的抽象实现类， 是 ForkJoinPool 中的运行基类。特点是任务要采用分而治之的方式完成。 

   实现ForkJoinTask 的抽象类有两个：

   RecursiveAction: 无返回值

   RecursiveTask: 有返回值

   

   ​        fork(): 向 ForkJoinPool 提交 task.

   ​        join(): 用于开始任务。

   

   ```
   customRecursiveTaskFirst.fork();
   result = customRecursiveTaskLast.join();
   ```

   

   提交：

   submit()/execute(): 需要调用join()

   ```
   forkJoinPool.execute(customRecursiveTask);
   int result = customRecursiveTask.join();
   ```

   使用invoke() 可以直接  fork 任务并等待返回结果。

   ```
   int result = forkJoinPool.invoke(customRecursiveTask);
   ```

   

