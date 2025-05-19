# Java Concurrency Interview Questions


## 1. How to create a new thread?

**Answer:**

1. **实现 `Runnable` 接口：**

   ```java
   Runnable task = () -> {
       // 任务逻辑
       System.out.println("Hello from a new thread");
   };
   Thread thread = new Thread(task);
   thread.start();
   ```
2. **继承 `Thread` 类：**

   ```java
   class MyThread extends Thread {
       @Override
       public void run() {
           System.out.println("Hello from MyThread");
       }
   }

   // 使用：
   Thread t = new MyThread();
   t.start();
   ```

## 2. Difference between `Runnable` and `Callable`?

| 特性          | `Runnable`      | `Callable<V>`    |
| ----------- | --------------- | ---------------- |
| 方法签名        | `void run()`    | `V call()`       |
| 返回值         | 无               | 返回类型 `V`         |
| 异常处理        | 不能抛出 checked 异常 | 可以抛出 `Exception` |
| 提交到执行器后获取结果 | 不支持             | 支持 `Future<V>`   |

```java
// Callable 示例：
Callable<Integer> computation = () -> {
    // 计算并返回结果
    return 42;
};
Future<Integer> future = executor.submit(computation);
Integer result = future.get();
```
## 3. How do threads communicate with each other?

* **共享内存**：多个线程访问同一个对象的字段。需要同步（`synchronized`、`volatile`）以保证可见性。
* **`wait()` / `notify()` / `notifyAll()`**：在 `synchronized` 块内调用，实现线程间的条件等待与唤醒。
* **高阶并发工具**：如 `BlockingQueue`、`Exchanger`、`SynchronousQueue`、`PipedInputStream`/`PipedOutputStream` 等。
* **`Lock` / `Condition`**：`ReentrantLock` 搭配 `Condition` 提供类似 `wait/notify` 的功能，但更灵活。


## 4. What is `synchronized`?

* Java 关键字，用于修饰方法或代码块，获取对象监视器（monitor）。
* **对象锁**：`synchronized(this)` 或 修饰实例方法。
* **类锁**：`synchronized(SomeClass.class)` 或 修饰静态方法。
* **作用**：保证同一时刻只有一个线程执行被同步的代码段，并保证内存可见性（进入前先刷新主存，退出后写回）。


## 5. What is `volatile`?

* 修饰 **变量**，保证对该变量的写操作对其他线程 **立即可见**。
* 禁止指令重排序，保证对该变量的读写顺序不被编译器或 CPU 重排。
* **不保证原子性**，仅确保可见性和有序性。


## 6. `sleep()` vs. `wait()`?

| 特性        | `Thread.sleep(long ms)` | `Object.wait()`                                         |
| --------- | ----------------------- | ------------------------------------------------------- |
| 属于类       | `Thread`                | `Object`                                                |
| 是否释放锁     | 不释放                     | 释放当前对象监视器                                               |
| 必须在同步块中调用 | 否                       | 是（必须在持有对象锁时）                                            |
| 抛出异常      | `InterruptedException`  | `IllegalMonitorStateException` 或 `InterruptedException` |


## 7. What is the difference between `t.start()` and `t.run()`?

* `t.start()`：创建并启动一个新线程，由 JVM 调度，在新线程中执行 `run()` 方法。
* `t.run()`：只是普通方法调用，在当前线程中执行 `run()` 内部逻辑，不会启动新线程。


\## 8. Class lock vs. Object lock

* **对象锁**（Instance Lock）：通过 `synchronized(this)` 或 同步实例方法获得，锁的粒度为该对象实例。
* **类锁**（Class Lock）：通过 `synchronized(SomeClass.class)` 或 同步静态方法获得，锁的粒度为该类的全局，作用于所有实例。


## 9. What is the `join()` method?

* `Thread.join()` 会 **阻塞当前线程**，直到调用 `join()` 的线程终止（自然结束或抛出异常）。
* 用于实现 “等待线程完成” 的场景。



## 10. What is the `yield()` method?

* `Thread.yield()` 给调度器一个提示，当前线程愿意让出 CPU，但不保证立即切换或指定切换到哪一个线程。
* 只是一个 **hint**，可能对性能调优有用，但不应用于强制同步或性能保证。



## 11. What is a ThreadPool? How many types are there? What is the TaskQueue?

* **ThreadPool**：线程池，复用一组固定数量或可变数量的线程来执行任务，避免频繁创建/销毁线程。
* **常见实现**：`ThreadPoolExecutor`。
* **预定义类型（通过 `Executors` 工厂方法）**：

  * `newFixedThreadPool(int nThreads)`
  * `newCachedThreadPool()`
  * `newSingleThreadExecutor()`
  * `newScheduledThreadPool(int corePoolSize)`
* **TaskQueue**：`BlockingQueue<Runnable>`，用于在所有线程都处于忙碌时保存待执行任务，如 `LinkedBlockingQueue`、`ArrayBlockingQueue`、`SynchronousQueue`。



## 12. Difference between `shutdown()` and `shutdownNow()` of Executor

* `shutdown()`：平滑关闭，停止接受新任务，执行已提交任务后结束。
* `shutdownNow()`：尝试停止所有正在执行的任务，返回未执行的任务列表；会中断正在运行的线程，需要任务响应中断。



## 13. What are concurrent collections? List some thread-safe data structures.

* **Concurrent Collections**：为了在多线程环境中安全访问而设计的集合类，内部采用分段锁、CAS 等机制。
* **常见实现**：

  * `ConcurrentHashMap`
  * `CopyOnWriteArrayList` / `CopyOnWriteArraySet`
  * `ConcurrentLinkedQueue` / `ConcurrentLinkedDeque`
  * `BlockingQueue` 及其实现：`LinkedBlockingQueue`、`ArrayBlockingQueue`、`PriorityBlockingQueue`
  * `ConcurrentSkipListMap` / `ConcurrentSkipListSet`

---

