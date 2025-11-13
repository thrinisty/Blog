---
title: 多线程并发编程笔记（JUC工具）
published: 2025-11-10
updated: 2025-11-10
description: 'JUC工具'
image: ''
tags: [Java]
category: 'Java'
draft: false 
---

# 多线程并发编程笔记

## AQS

AQS全称为AbstractQueuedSynchronizer，是阻塞式锁的相关的同步器工具框架

特点：

1.state属性表示资源状态（分为独占模式、共享模式），子类需要定义如何维护这个状态，控制如何获取锁

2.提供了基于FIFO的等待队列，类似于Monitor的WaitSet

3.条件变量实现等待，唤醒机制，支持多个条件变量，类似于Monitor的WaitSet



相关方法：

tryAcquire 获取锁

tryRelease 释放锁



自定义锁实现

```java
class MyLock implements Lock {
    //独占锁
    class MySync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int arg) {
            if (compareAndSetState(0, 1)) {
                //加上锁设置线程owner为当前线程
               setExclusiveOwnerThread(Thread.currentThread());
               return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        public Condition newCondition() {
            return new ConditionObject();
        }
    }

    private MySync sync = new MySync();

    //加锁，不成功进入队列等待
    @Override
    public void lock() {
        sync.acquire(1);
    }

    //加可打断锁
    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    //尝试加锁
    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    //尝试加锁（带超时时间）
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    //解锁
    @Override
    public void unlock() {
        sync.release(1);
    }

    //创建条件变量
    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

测试方法

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        MyLock myLock = new MyLock();
        new Thread(() -> {
            try {
                log.info("try_lock");
                myLock.lock();
                sleep(1000);
                log.info("t1");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                myLock.unlock();
            }

        }, "t1").start();

        new Thread(() -> {
            try {
                log.info("try_lock");
                myLock.lock();
                sleep(1000);
                log.info("t2");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                myLock.unlock();
            }

        }, "t2").start();
    }
}
```



## 读写锁

ReentrantReadWriteLock

当读操作远高于写操作的时候，这个时候使用读写锁让RR可以并发，提高性能，WW或WR是互斥的

```java
@Data
@Slf4j
class DataContainer {
    private Object date;
    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    private ReentrantReadWriteLock.ReadLock readLock = lock.readLock();
    private ReentrantReadWriteLock.WriteLock writeLock = lock.writeLock();

    public Object read() {
        log.info("get read lock");
        readLock.lock();
        try {
            log.info("read");
            return date;
        } finally {
            log.info("read end");
            readLock.unlock();
        }
    }

    public void write(Object date) {
        log.info("get write lock");
        writeLock.lock();
        try {
            log.info("write");
            this.date = date;
        } finally {
            log.info("write end");
            writeLock.unlock();
        }
    }
}
```

注意事项：

读锁不支持条件变量，写锁支持条件变量

重入时升级不支持：持有写锁的时候获取读锁会导致写锁的永久等待

重入时降级支持：持有写锁情况下去获取读锁



## StampedLock

JDK8加入，进一步优化读性能，特点是使用读锁，写锁时都必须配合【戳】使用

```java
long stamp = lock.writeLock();
lock.unlockWrite(stamp);
```

乐观读，不会加任何的锁，读完之后需要做戳校验，如果验证通过表示期间没有写操作，结果正确则返回，失败的时候升级为读锁



## Semaphre

信号量，用于限制能够同时访问共享资源的线程上线

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(2);
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    log.info("running");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } finally {
                    log.info("done");
                    semaphore.release();
                }
            }).start();
        }
    }
}
```



## CountDownLatch

用于线程同步协作，等待所有线程完成倒计时，只能用一次

其中构造参数用来初始化等待计数值，await用来等待归零，countDown用来让计数减一

```java
@Slf4j
public class Main {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(3);
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                try {
                    log.info("running");
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } finally {
                    log.info("done");
                    countDownLatch.countDown();
                }
            }).start();
        }
        log.info("main thread start");
        countDownLatch.await();
        log.info("main thread end");
    }
}
```

线程池使用CountDownLatch

```java
@Slf4j
public class Main {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(4);
        CountDownLatch countDownLatch = new CountDownLatch(3);
        service.submit(() -> {
            try {
                log.info("running");
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                log.info("done");
                countDownLatch.countDown();
            }
        });
        service.submit(() -> {
            try {
                log.info("running");
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                log.info("done");
                countDownLatch.countDown();
            }
        });
        service.submit(() -> {
            try {
                log.info("running");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                log.info("done");
                countDownLatch.countDown();
            }
        });
        service.submit(() -> {
            log.info("main thread start");
            try {
                countDownLatch.await();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.info("main thread end");
        });
    }
}
```



## Cycilcbarrier

循环栅栏，计数可以恢复到初始值

计数到0的时候同步继续向下执行，而构造的消费接口方法实现会在为0的时候执行

```java
@Slf4j
public class Main {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(4);
        CyclicBarrier barrier = new CyclicBarrier(3, () -> {
            log.info("Barrier Done");
        });
        service.submit(() -> {
            try {
                log.info("running");
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                log.info("done");
                try {
                    barrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
            }
        });

        service.submit(() -> {
            log.info("main thread start");
            try {
                barrier.await();
            } catch (BrokenBarrierException | InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.info("main thread end");
        });
    }
}
```



## 线程安全集合类

Blocking 大部分基于锁，提供用来阻塞的方法

CopyOnWrite 容器修改开销相对较重

Concurrent 类型很多操作使用cas优化：弱一致性对于便利的时候，求size的时候，读取时弱一致性



### ConcurrentHashMap

computeIfAbsent

```java
public static void main(String[] args) {
    Map<String, LongAdder> map = new ConcurrentHashMap();
    //如果缺少key，则生成计算一个value，将key value放入map
    LongAdder value = map.computeIfAbsent("a", (key) -> new LongAdder());
    //累加执行
    value.increment();
}
```
