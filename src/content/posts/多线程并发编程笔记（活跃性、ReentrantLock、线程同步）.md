---
title: 多线程并发编程笔记（活跃性、ReentrantLock、线程同步）
published: 2025-11-06
updated: 2025-11-06
description: '活跃性、ReentrantLock、线程同步模式'
image: ''
tags: [Java]
category: 'Java'
draft: false 
---

# 多线程并发编程笔记

## 活跃性

### 死锁

t1线程拥有A锁，等待获取B锁

t2线程拥有B锁，等待获取A锁

```java
package com;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.LockSupport;

@Slf4j
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            synchronized ("A".intern()) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.info("占有A，等待B");
                synchronized ("B".intern()) {
                    System.out.println("获取到B");
                }
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            synchronized ("B".intern()) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.info("占有B，等待A");
                synchronized ("A".intern()) {
                    System.out.println("获取到A");
                }
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

死锁定位：jconsole工具，jps定位进程中的活锁

可以通过顺序加锁进行处理



### 活锁

两个线程互相改变对方结束的条件，导致了两个线程都不结束

```java
@Slf4j
public class DeadLock {
    private static int i = 10;
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
           while(i >= 0) {
               i--;
               log.info("i=" + i);
               try {
                   Thread.sleep(300);
               } catch (InterruptedException e) {
                   throw new RuntimeException(e);
               }
           }
        }, "t1");
        Thread t2 = new Thread(() -> {
            while(i <= 20) {
                i++;
                log.info("i=" + i);
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

可以加一些随机的休眠时间解决



### 饥饿

一个线程由于优先级太低，始终得不到CPU调度执行，也不能结束

我们在用顺序获取锁解决的死锁，就有可能导致后面的线程一直抢不到锁，导致饥饿



## ReentrantLock

具备以下特点：可中断，可以设置超时时间，可以设置为公平锁，支持多个变量条件

基本语法：

创建ReentrantLock对象

可重入

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            try {
                log.info("线程对ReentrantLock加锁");
                lock.lock();
                log.info("尝试重入锁");
                lock.lock();
                log.info("再次尝试重入锁");
                lock.lock();
            } finally {
                lock.unlock();
            }
        }, "t1");
    }
}
```

可被打断：使用lockInterruptibly锁可以被打断

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                log.info("线程对ReentrantLock加锁");
                lock.lockInterruptibly();
                Thread.sleep(40000);
            } catch (InterruptedException e) {
                log.info("被打断");
            } finally {
                lock.unlock();
            }
        }, "t1");
        thread.start();
        Thread.sleep(2000);
        thread.interrupt();
    }
}
```

锁超时：立即失败

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                log.info("线程对ReentrantLock尝试加锁");
                boolean b = lock.tryLock();
                if(!b) {
                    log.info("没获取到锁ReentrantLock，立即返回");
                    return;
                }
                log.info("成功获取锁，执行代码");
            }  finally {
                lock.unlock();
            }
        }, "t1");
        lock.lock();
        Thread.sleep(2000);
        thread.start();
        Thread.sleep(2000);
        log.info("主线程结束");
    }
}
```

也可以在尝试获取锁添加时间，在这个时间段上获取到锁也可以正常执行

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                log.info("线程对ReentrantLock尝试加锁");
                boolean b = lock.tryLock(3, TimeUnit.SECONDS);
                if(!b) {
                    log.info("没获取到锁ReentrantLock，立即返回");
                    return;
                }
                log.info("成功获取锁，执行代码");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }, "t1");
        lock.lock();
        Thread.sleep(1000);
        thread.start();
        Thread.sleep(2000);
        log.info("主线程结束");
        lock.unlock();
    }
}
```

公平锁：ReentrantLock默认是不公平的

```java
private static ReentrantLock lock = new ReentrantLock(true);
//设置为公平
```



条件变量

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock(false);
    public static void main(String[] args) throws InterruptedException {
        Condition condition = lock.newCondition();
        Thread thread = new Thread(() -> {
            lock.lock();
            try {
                Thread.sleep(2000);
                condition.signal();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        });
        thread.start();
        lock.lock();
        //进入休息室等待
        condition.await();
        lock.unlock();
    }
}
```



## 线程同步模式

**先后执行**

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock(false);
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
           log.info("1");
        }, "t1");
        Thread t2 = new Thread(() -> {
            log.info("2");
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

```
2025-11-06 20:06:07 [t2] INFO  com.Reentrant - 2
2025-11-06 20:06:07 [t1] INFO  com.Reentrant - 1
```



使用条件变量实现

```java
@Slf4j
public class Reentrant {
    private static ReentrantLock lock = new ReentrantLock(false);
    public static void main(String[] args) throws InterruptedException {
        Condition condition = lock.newCondition();
        Thread t1 = new Thread(() -> {
            try {
                lock.lock();
                condition.await();
                log.info("1");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            try {
                lock.lock();
                log.info("2");
                condition.signal();
            }  finally {
                lock.unlock();
            }
        }, "t2");
        t1.start();
        t2.start();
    }
}
```

unpark/park实现

```java
@Slf4j
public class Reentrant {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            LockSupport.park();
            log.info("t1");

        }, "t1");
        Thread t2 = new Thread(() -> {
            log.info("t2");
            LockSupport.unpark(t1);
        }, "t2");
        t1.start();
        t2.start();
    }
}
```



**交替输出**

自旋实现，忙等待

```java
@Slf4j
public class Reentrant {
    private static volatile int flag = 1;
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while (true) {
                while (flag == 1) {
                    log.info("a");
                    flag = 2;
                }
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            while (true) {
                while (flag == 2) {
                    log.info("b");
                    flag = 3;
                }
            }
        }, "t2");
        Thread t3 = new Thread(() -> {
            while (true) {
                while (flag == 3) {
                    log.info("c");
                    flag = 1;
                }
            }
        }, "t3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

使用wait+notifyAll实现

```java
@Slf4j
public class Reentrant {
    private static int flag = 1;
    private static final Object lock = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    while (flag != 1) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    }
                    log.info("a");
                    flag = 2;
                    lock.notifyAll();
                }
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    while (flag != 2) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    }
                    log.info("b");
                    flag = 3;
                    lock.notifyAll();
                }
            }
        }, "t2");
        Thread t3 = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    while (flag != 3) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    }
                    log.info("c");
                    flag = 1;
                    lock.notifyAll();
                }
            }
        }, "t3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

再封装一下

```java
package com;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Reentrant {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            Notify notify = new Notify();
            notify.printChar("a", 1, 2);
        });
        Thread t2 = new Thread(() -> {
            Notify notify = new Notify();
            notify.printChar("b", 2, 3);
        });
        Thread t3 = new Thread(() -> {
            Notify notify = new Notify();
            notify.printChar("c", 3, 1);
        });
        t1.start();
        t2.start();
        t3.start();
    }
}

@Slf4j
class Notify {
    private static int flag = 1;
    public final int LOOP_NUM = 10;
    public void printChar(String msg, int target, int next) {
        for (int i = 0; i < LOOP_NUM; i++) {
            synchronized (Notify.class) {
                while (flag != target) {
                    try {
                        Notify.class.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                log.info(msg);
                flag = next;
                Notify.class.notifyAll();
            }
        }
    }
}
```

使用park/unpark实现

```java
@Slf4j
public class Reentrant {
    static Thread t1;
    static Thread t2;
    static Thread t3;
    public static void main(String[] args) throws InterruptedException {
        Notify notify = new Notify();
        t1 = new Thread(() -> {
            notify.printChar("a", t2);
        });
        t2 = new Thread(() -> {
            notify.printChar("b", t3);
        });
        t3 = new Thread(() -> {
            notify.printChar("c", t1);
        });
        t1.start();
        t2.start();
        t3.start();
        Thread.sleep(100);
        LockSupport.unpark(t1);
    }
}

@Slf4j
class Notify {
    public final int LOOP_NUM = 10;
    public void printChar(String msg, Thread next) {
        for (int i = 0; i < LOOP_NUM; i++) {
            LockSupport.park();
            log.info(msg);
            LockSupport.unpark(next);
        }
    }
}
```

