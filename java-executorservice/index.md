# Java ExecutorService


Recently, I was working on HDFS-17030 and used ExecutorService for multi-threading
execution. We hit a non-intuitive \`bug\` of how JVM/garbage collector
works. The issue is as following.

We created an executorService with 1 core thread in a class. We did not
set _allowCoreThreadTimeOut_ to true (default is false). So, 
that core thread will be kept running, even when the main thread exits!
Then, the JVM process won't exit, because there is still one thread running!
Even when we shut down the executorService in _.close()_ method of that class,
the same issue persists. The issue is _.close()_ method of a Closable
object is called by Java garbage collector _only_ when it is reclaiming
that object. This does not happen immediately when there is
no reference to an object. After an object becomes orphan (no 
more reference to it), it will be added to a queue. The garbage collector
will look into the queue and determine which objects to reclaim
based on its own algorithm. It is undeterministic/out of control from
programmers on when the garbage collector will reclaim an object. 

The following code is an example.

```java
import java.io.Closeable;
import java.io.IOException;
import java.util.Date;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;


public class ExecutorServiceCoreThreadIdleTimeoutTest implements Closeable {
  ExecutorServiceCoreThreadIdleTimeoutTest() {
    pool =
    new ThreadPoolExecutor(1, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(1024),
        namedThreadFactory);
     
    //pool.allowCoreThreadTimeOut(true);
  }

  ThreadFactory namedThreadFactory = new ThreadFactory() {
    private final AtomicInteger threadNumber = new AtomicInteger(1);

    @Override
    public Thread newThread(Runnable r) {
      String name = "myThread-" + threadNumber.getAndIncrement();
      return new Thread(r, name);
    }
  };

  private final ThreadPoolExecutor pool;

  public void submitTask() {

    pool.submit(() -> {
      System.out.printf("%tc: task is running\n", new Date());
    });
  }

  public static void main(String[] args) throws InterruptedException {
    System.out.printf("%tc: Main thread started\n", new Date());

    ExecutorServiceCoreThreadIdleTimeoutTest test = new ExecutorServiceCoreThreadIdleTimeoutTest();
    test.submitTask();
    Thread.sleep(1000);
    System.out.printf("%tc: Main thread exited\n", new Date());
  }

  @Override
  public void close() throws IOException {
    pool.shutdown();
    System.out.printf("%tc: shutdown thread pool\n", new Date());
  }
}
```
- When we comment out `pool.allowCoreThreadTimeOut(true);`, the core 
thread will be kept running and the JVM process kept running, though the main thread exits.
```bash
Mon Jul 03 15:42:50 PDT 2023: Main thread started
Mon Jul 03 15:42:50 PDT 2023: task is running
Mon Jul 03 15:42:51 PDT 2023: Main thread exited
```
Intellij threaddump showed there is a thread "myThread-1" in `parking` state,
waiting for new tasks to execute.

- After we comment out `pool.allowCoreThreadTimeOut(true);`, the core thread
will be running idle for 10 seconds before terminating. After that,
the JVM process also exited. Please note the `Process finished with exit code 0`
message at the end.
```bash
Mon Jul 03 15:43:43 PDT 2023: Main thread started
Mon Jul 03 15:43:43 PDT 2023: task is running
Mon Jul 03 15:43:44 PDT 2023: Main thread exited

Process finished with exit code 0
```

At first glance, this looks a bit wired, as when the main thread exits, we would expect
the program to shut down. In retrospect, the main thread is just one of the threads in a JVM
process. So, as long as there is one runnable/running thread, the JVM process won't shut down 
itself.

Output from _.close()_ is not redirected as part of stdout.
They don't show up in the output.

