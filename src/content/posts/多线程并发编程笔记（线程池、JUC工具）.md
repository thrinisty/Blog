---
title: 多线程并发编程笔记（线程池、JUC工具）
published: 2025-11-09
updated: 2025-11-09
description: '线程池、JUC工具'
image: ''
tags: [Java]
category: 'Java'
draft: false 
---

# 多线程并发编程笔记

## 线程池

阻塞队列实现

```java
class BlockingQueue<T> {
    //任务队列
    private Deque<T> deque = new ArrayDeque<>();
    //锁：防止线程竞争获取线程
    ReentrantLock lock = new ReentrantLock();
    //生产者条件变量
    private Condition fullWaitSet = lock.newCondition();
    //消费者条件变量
    private Condition emptyWaitSet = lock.newCondition();
    //容量
    private int capacity;

    public T take() {
        lock.lock();
        try {
            while (deque.isEmpty()) {
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            fullWaitSet.signal();
            return deque.removeFirst();
        } finally {
            lock.unlock();
        }
    }

    public void put(T t) {
        lock.lock();
        try {
            while (capacity == deque.size()) {
                try {
                    fullWaitSet.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            deque.addLast(t);
            emptyWaitSet.signal();
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return deque.size();
        } finally {
            lock.unlock();
        }
    }
}
```

