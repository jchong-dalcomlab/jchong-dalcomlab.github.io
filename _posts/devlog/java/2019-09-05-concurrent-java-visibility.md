---
author: author2
layout: post
title: Java Concurrent Programming - Visibility
subtitle: About Java Concurrent Programming, Visibility article
category: devlog
tag: java
comments: true
---

# 프롤로그
<p style='text-align: justify;'> 가시성과 원자성 이 두가지를 이해 하면 Multi-thread프로로그램을 작성할 때 무엇을 주의해하는지 명확해진다. 또 다른 표현으로 설명 하자면 단일 Thread 프로그램에서는 가시성과 원자성을 괘념치 않아도 프로그램 작성하는데 문제가 없다.  그렇다고 해서 가시성과 원자성이 Multi-thread를 할 때만 뚜둥 나타나는 개념은 아니라는점 명확히 밝혀둔다. 하드웨어를 설계한 사람들에 의해 만들어진 원래 컴퓨터 내의 구조에 관한 이야기다. 이번 편에서는 지난편에서 설명한 이 두 가지 개념중에 가시성을 좀더 깊게 파고들어보려 한다.
</p>

# 비 가시성(가시성 이슈)
<p style='text-align: justify;'>
다른 자료나 책을 통해 가시성에 대한 개념을 이해하고 있는 독자라면 필자가 외 비 가시성이라는 반대가 되는 단어를 만들어 썼는지 감이 왔을것이다. 지난 편에도 아래 그림이 등장했는데 다시한번 고찰 해보자. 
</p>

![repsimg_cache-use-visibility](https://jchong00.github.io/img/about-concurrent/cache-use-visibility.png){: .center-block :}
<p style='text-align: center;'>
[가시성 이슈]
</p>

<p style='text-align: justify;'>
 각기 다른  Thread 2개는 CPU 1과 CPU2를 할당 받아 공유자원에 해당하는 변수를 연산한다. 이 때 메인 메모리에서 값을 읽어 연산에 사용하는 것이 아니라 가 CPU에 존재하는 Cache에 옮겨놓고 연산을 한다. 그 사이에 다른 쓰레드에서 같은 변수를 대상으로 연산을 한다. 이 때 Thread는 타 Thread가 사용하고 있는 CPU의 Cache상의 값을 알지 못한다. 언제 Cache의 값이 메인 메모리로 쓰여질지도 모른다. 그래서 아래코드를 돌려보면 각각의 Thread가 100회씩 변수를 증가연산을 실시 했는데 200에 휠씬 못 미치는 103 ~ 105쯤이 나타난다. (이 실험 결과는 하드웨어의 성능에 따라 상이 할 수 있으니 참고 바란다.) 만약 동시에 시작하는 Thread가 아닌 순차 처리라면 200이 나와야 하는 현상이다. Cache에 담아서 연산을 하더라도 바로 바로 메모리에 적용을 했으면 200은 아니더라도 180~190정도는 날올것 같은데 너무나도 100가까운 결과에 놀라는 독자도 있을 것이다. 다음 단계에서 이 문제를 조금(?) 해결 해보자.
</p>

```java
private int count = 0; 

@Test 
public void Test_AtomicIssue() { 
    ExecutorService es = Executors.newFixedThreadPool(2); 
    es.execute(new ForThreadTest()); 
    es.execute(new ForThreadTest()); 
    es.shutdown(); 
    try { 
        es.awaitTermination(10, TimeUnit.MINUTES); 
    } catch (InterruptedException e) { 
        e.printStackTrace(); 
    } 
    System.out.println("TEST result >>> " + count); 
} 

class ForThreadTest implements Runnable { 
    @Override public void run() { 
        for(int i = 0 ; i < 100; i++) { 
            AtomicStampedRefTest.this.count++; 
        } 
    } 
}
```
<p style='text-align: center;'>
[공유자원 문제가 심각한 코드]
</p>

# volatile을 이용한 가시성 확보
<p style='text-align: justify;'>
 visibility(가시성)에 이어 volatile(변덕 스러운) 용어 더럽게 맘에 안든다. 가시성은 이제 좀 와 닫는데 아직도 외 volatile이라는 단어를 사용했는지 이해가 가지 않는다. 나중에 이해가 갈 때가 오면 이 글에 첨언을 남기겠다. 암튼 JAVA에선 위에서 설명한 비 가시성 이슈를 해결하기 위해 JAVA 1.4 부터 volatile을 지원하기 시작 했다. 변수를 선언 할 때 해당 단어를 앞에 써주기만 하면 되는데 이렇게만 해도 위 테스트 코드가 거의 200을 반환한다. (사실 이 결과는 미신같은 이야기다. 정확히 이야기 하면 200이거나 200보다 작은 값을 반환한다.) 원리는 이렇다. volatile로 선언된 변수를 CPU에서 연산을 하면 바로 메모리에 쓴다. (Cache에서 메모리로 값이 이동하는 것을 다른 책이나 문서에서는 flush라고 표현한다.) 그러니 운이 좋게 Thread 두 개가 주거니 받거니 하면서 증가를 시키면 200에 가까운 결과를 얻어내는 것이다. 
</p>

```java
private volatile int count = 0;

@Test
public void Test_AtomicIssue() {
    ExecutorService es = Executors.newFixedThreadPool(2);
    es.execute(new ForThreadTest());
    es.execute(new ForThreadTest());
    es.shutdown();
    try {
        es.awaitTermination(10, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("TEST result >>> " + count);
}

class ForThreadTest implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            AtomicStampedRefTest.this.count++;
        }
    }
}
```
<p style='text-align: center;'>
[가시성 부여로 조금 해결된 코드]
</p>

<p style='text-align: justify;'>
하지만 개발자라면 이 미신같은 결과에 흡족해 하면 안된다. 필자의 PC를 기준으로 각 Thread당 100회가 아닌 1000회정도 연산을 시키면 2000이 아닌 1998같은 결과를 얻어낸다. 이 이야기인 즉 가시성이 확보된다 하더라도 원자성 문제(동시에 같은 값을 읽어다 증가시키고 flush하는...)로 인해 이와 같은 문제가 생기는 것이다. 이 문제는 원자성 다루면서 해결 해보자.
</p> 

![repsimg_cache-use-visibility](https://jchong00.github.io/img/about-concurrent/cache-use-visibility_problem.png){: .center-block :}
<p style='text-align: center;'>
[가시성은 확보하였다. 그러나 원자성 문제로 완벽하지 못한 공유자원]
</p>

# volatile로도 충분한 경우
<p style='text-align: justify;'>
필자가 가시성과 원자성을 구분하여 이렇게 글을 쓰는 이유이기도 하다. 나중에 다룰 이야기이긴 하지만 Multi-Thread의 안정성(데이터 무결성)을 확보 한다고 여기저기 syschronized 혹은 lock을 남발한다면 Multi-Thread처리 로직으로 인해 코드 복잡도만 높아지고 실제 성능에 대한 효과는 크게 누리지 못 할 것이다. 필자도 lock, unlock과정이 내부적으로 어떻게 구현되었는지는 모르겠지만 이 과정속에 프로그램 statement context의 switch가 일어나면서 성능이 떨어진다고 한다. 해서 어떤 형태던 lock을 최소화 하는 것이 좋은데 volatile도 그 방법중 하나다. 단 제약 조건이 깔린다. 하나의 Thread만이 연산을 해야 한다. 만약 이 전제가 확실한 경우라면 변수에 volatile로만 lock 없이도 문제 없는 데이터를 사용할 수 있다. 아래 코드는 이런 상황을 가정하여 하나의 Thread가 Count를 생산하고 한쪽 Thread는 반대로 읽기만 하는 구조를 갖는 예시이다.
</p>

```java

private volatile boolean done = true;
private volatile int count = 0;
private final int TEST_REPETITION = 1000;
@Test
public void Test_AtomicIssue() {
    ExecutorService es = Executors.newFixedThreadPool(2);
    es.execute(new ForThreadTestReadOnly());
    es.execute(new ForThreadTest());
    es.shutdown();
    try {
        es.awaitTermination(10, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("TEST result >>> " + count);
}
class ForThreadTest implements Runnable {
    @Override
    public void run() {
        while (true) {
            if (AtomicStampedRefTest.this.done) {
                AtomicStampedRefTest.this.count++;
                AtomicStampedRefTest.this.done = false;
            }
            if (AtomicStampedRefTest.this.count == TEST_REPETITION) {
                break;
            }
        }
    }
}
class ForThreadTestReadOnly implements Runnable {
    @Override
    public void run() {
        while (true) {
            if (!AtomicStampedRefTest.this.done) {
                System.out.println("Current count >>>" + AtomicStampedRefTest.this.count);
                AtomicStampedRefTest.this.done = true;
            }
            if (AtomicStampedRefTest.this.count == TEST_REPETITION) {
                break;
            }
        }
    }
}

```

# 애필로그
<p style='text-align: justify;'>
가시성에 대한 정리는 이쯤에서 정리하고 다음편은 원자성에 대한 내용을 다루려고 한다.
</p>
