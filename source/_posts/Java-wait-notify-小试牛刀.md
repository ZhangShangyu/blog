---
title: Java wait/notify 小试牛刀
date: 2019-02-18 15:38:43
categories:
- 软件开发
tags:
- Java
---

![](https://upload.wikimedia.org/wikipedia/en/0/0d/NBA_All-Star_2019_logo.png)

<!--more-->

两个线程交替打印奇数偶数，使用wait notify进行线程协调通信

```java

public class WaitNotify {

    private final Object lock = new Object();

    public void run() {
    	
    	//线程1打印奇数
    	new Thread(new Runnable() {
            public void run() {
                for (int i = 1; i <= 100; i += 2) {
                		// 获得锁
                    synchronized (lock) {
                    	 // 打印数字
                        System.out.println(i);
                        // 打印完成后通知线程2继续操作
                        lock.notify();
                        // 阻塞，释放锁 等待线程2完成操作后唤醒
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }).start();
		 
		 //线程2打印偶数
        new Thread(new Runnable() {
            public void run() {
                for (int i = 2; i <= 100; i += 2) {
                    synchronized (lock) {
                    	 // 若获得锁 则通知线程1进行操作
                        lock.notify();
                        // 阻塞 等待线程1先打印奇数后通知
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        // 打印偶数
                        System.out.println(i);
                    }
                }
            }
        }).start();
    }

    public static void main(String[] args) {
        WaitNotify waitNotify = new WaitNotify();
        waitNotify.run();
    }
}

```
