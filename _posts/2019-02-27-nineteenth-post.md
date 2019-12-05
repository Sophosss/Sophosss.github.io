---
title: 总结下Java线程中断
author: Sophosss
layout: post
---
开门见山，直接列出来相关方法。

- `void thread.stop()`
  这个方法能中断正在运行的线程，但是已经不推荐使用了，在将来的版本或许弃用，因为强行中断运行中的线程，是不安全的。
- `void thread.interrupt()`
  如果正在运行wait()，sleep()，join()这三个方法阻塞了线程，那么将会使得线程抛出InterruptedException异常，这是一个中断阻塞的过程。如果是其它的正在运行的状态，那么将不会有任何影响，也不会中断线程，或者抛出异常，只会会打上一个中断线程的标志，是否中断线程，将由程序控制。
- `boolean thread.isInterrupted()`
  它会获取当前线程的标志，如果之前调用过`thread.interrupt()`，那么它的返回值是true。它的作用就是返回该线程是否有中断标志。多次调用这个方法的结果是一样的。
- `void Thread.interrupted()`
  与前面的方法不一样的是，这是一个静态方法，代表着不需要拿到线程对象就可以直接执行，所以它的作用是返回当前线程是否有中断标志。但是它的区别是，当调用这个方法之后，会清除程序的中断标志，就是如果当前线程已中断，第一次调用这个方法的返回值是true，第二次调用这个方法的返回值为false，因为调用方法时，会清除它的中断标志。

1. 阻塞的退出线程

   这里主要注意几个点就可以了：

   - 只要是在运行wait()，sleep()，join()的方法，它就会声明一个InterruptedException异常，也就是意味着这些方法并不是一定能执行完成，因为当调用线程的interrupt()方法时，就会中断这个阻塞的办法，从而进入到异常中。
   - 只要进入了InterruptedException异常，中断标记就会被清除掉。
   - 另外for循环中是无法中止的，所以线程会在进入sleep()的时候，瞬间就抛出异常。

2. for循环标记退出

   这种也可以使用while(true)轮询方式，安全。结合代码和上述方法食用更佳。

   ```java
   	@Override
       public void run() {
           for(int i = 0 ; i < 10000 ; i ++){
               boolean interruped = Thread.currentThread().isInterrupted();
               
               //有中断标记，中断
               if(interruped) break;
               
               System.out.println(i);
           }
           System.out.println("over");
       }
   ```

3. stop()的方式不安全所以很不推荐使用，就不介绍了。