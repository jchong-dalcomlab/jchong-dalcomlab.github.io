---
author: author2
layout: post
title: Java Concurrent Programming - Synchronizers
subtitle: About Java Concurrent Programming, Synchronizers article
category: devlog
tag: java
comments: true
---

## 프롤로그
<p style='text-align: justify;'>
이번에 다룰 내용은 java.util.concurrent package에서 제공하는 Synchronizer(동기자)들에 대한 내용이다. 동기자라 비 동기화된 각 Thread의 작업을 동기화 시키는데 그 목적이 있다. 하나의 일을 여러 스레드가 나누어 수행을 하고 이를 모아서 하나의 결과를 얻는 집계와 같은 프로세스를 예로 들 수 있다. JAVA 1.5 이전의 전통적인 방법으로 thread를 다룬다면 object.wait()로 프로세스를 멈추게 하고 다른 스레드에서 object.notify() 또는 object.notifyAll()을 호출 했을 때 이 멈춤이 풀리게 된다. 그런데 이 방법은 잘못 사용하면 상당히 위험 해질 수 있으니 조심해야 할 뿐만 아니라 특정 플랫폼에서는 소위 허위 각성 현상이라는 문제가 있어 깨우지도 않았는데 깨는 현상이 발생한다고 한다. 이를 대신하여 사용할 수 있는 몇몇 class를 소개하려 한다.
</p>

## CountDownLatch
<p style='text-align: justify;'>
영어가 짧은 필자는 이 class를 처음 봤을때 Latch라는 단어를 사전을 찾아 봤다. 빗장, 걸쇠라는 뜻이다. 숫자를 정해놓고 await()로 멈춘 스레드는 다른 스레드에 의해 카운트 다운되다 0에 도달하면 빗장이 풀리는 기능을 가지고 있다.  
아래 예는 총 두 개의 Latch를 사용하고 있다. startGate 빗장은 상수 1을 카우터로 가지고 있고 메인을 제외한 모든 스레드들은 이 빗장이 풀리기를 기다린다. 그리고 이 빗장은 메인 스레드에서 모든 스레드를 start하는 작업이 끝나고 나면 카운트다운(startGate.countDown())한다. for loop에 의해 생성되어 대기중인 모든 스레드들은 일제히 출발한다. 애초에 카운터의 크기가 1이였기 때문에 한번에 풀린다.
또 하나의 Latch는 생성되 스레드 수와 같은 Latch Count를 가지고 초기화 된다. 이 래치는 Main thread의 마지막 단계에서 멈춘다. 그리고 각 스레드 작업의 종료시점에서 Count down 된다. 마지막 스레드 작업에 의해 Latch의 Count가 0되는 순간 Main thread의 빗장을 풀리게 된다. 이 프로그램은 빗장이 풀린 시점에서 수행 시간을 측정 했다.     
</p>

``` java
public long timeTasks(int nThreads, final Runnable task)
    throws InterruptedException {
    final CountDownLatch startGate = new CountDownLatch(1);
    final CountDownLatch endGate = new CountDownLatch(nThreads);
    for (int i = 0; i < nThreads; i++) {
        Thread t = new Thread() {
            public void run() {
                try {
                    startGate.await();
                    try {
                        task.run();
                    } finally {
                        endGate.countDown();
                    }
                } catch (InterruptedException ignored) { }
            }
        };
        t.start();
    }
    long start = System.nanoTime();
    startGate.countDown();
    endGate.await();
    long end = System.nanoTime();
    return end-start;
}
```
