---
title: JAVA并发之CountDownLatch、CyclicBarrier和Semaphor
tags: 并发
sidebar: 
  nav: docs-zh
---

### CountDownLatch 

一个线程或N个线程一直等待，直到其他线程执行的操作完成后才唤醒。通过调用await()方法让线程进入阻塞状态等待倒计时0时唤醒。 它通过线程调用countDown()让倒计时中的计数器减去1，当计数器为0时，会唤醒因为调用了await()而阻塞的线程。

```
public class CountDownLatchDemo {
    
    public static void main(String[] args) throws InterruptedException {
        //初始化
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"START");
                //计数器减一
                countDownLatch.countDown();
            }, String.valueOf(i)).start();

        }
        //阻塞等待计数器归零
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"END");
    }

}
```

### CyclicBarrier

当某个线程调用await方法时，该线程进入等待状态，且计数器加1，当计数器的值达到设置的初始值时，所有因调用await进入等待状态的线程被唤醒，继续执行后续操作。

```
public class CyclicBarrierDemo {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, new Runnable() {
            @Override
            public void run() {
                System.out.println("神龙召唤成功");
            }
        });

        for (int i = 1; i <= 7; i++) {
            final int temp = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集了 第"+temp+"颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

### Semaphore

多个共享资源互斥使用。例如信号灯，停车场抢位置

```
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(1);
        for (int i = 1; i <= 3; i++) {
            new Thread(()->{
                try {
                    //获取权限
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"号灯开启");
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName()+"号灯关闭");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    //释放权限
                    semaphore.release();
                }

            },String.valueOf(i)).start();
        }

    }
}
```