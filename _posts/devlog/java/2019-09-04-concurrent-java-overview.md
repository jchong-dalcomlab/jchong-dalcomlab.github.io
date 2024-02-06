---
author: author2
layout: post
title: Java Concurrent Programming - Overview
subtitle: About Java Concurrent Programming, Overview page
category: devlog
tag: java
comments: true
---

# 프롤로그
<p style='text-align: justify;'>
프로그래밍이라는 것을 시작한 이래로 늘 무겁게 다가오는 주제가 Multi-Thread였다. 어떤 언어로 하던간에 어렵고 난해    했다. 그렇기에 늘 내가 알고 하고있구나 하는 느낌 보다는 어디서 주섬 주섬 주워온 코드로 현실에 Issue들을 해결하기급급 했다. 헌데 지금은 양상이 다르다. 서버 프로그램밍을 하고 있는 시점에 "아 됐어, 돌아가!" 그리고 다음진도를    빼기에는 너무 불안하다. 사실 이 글을 쓰기전에 atomicReference와 atomicStampedReference등의    자바 Class를 이용한 기술 단편을 정리해서 글을 작성 했는데 내가 이거 뭐 알고 쓰고 있는거야 하는 자책감에 차마    공개하지 못하고 모든 진도와 계획을 멈추고 책과 자료를 찾아 뒤졌다. 자료를 하루 이틀 찾다 보니 그간 내가 정말 개념 없이 volitile, atomic을 운운 했구나 하는 생각이 들었다. 약 일주일의 시간이 긴 터널 같이 느껴질 정도로 간만에 정말 열심히 공부 했다. 더 나가기 전에 이 글을 이해하려면 적어도 Thread가 뭔지는 알아야 한다. 그렇지 못한 독자가 있다면 잠시 다른 자료를 통해 개념이라도 이해 하고 다시 돌아오시기 바란다.
</p>
<p style='text-align: justify;'>
이번 글에서는 Multi-Thread와 이를 다루는 과정의 기초가 되는 가시성과 원자성을 정의해 보도록 하겠다. 사실 가시성과 원자성이라고 하는 단어는 문제를 해결하기 위한 원칙(?)이다. 바꿔 말해 Multi-Thread를 구성하다 보니 비 가시성 비 원자성 문제가 발생 했고 Muti-Thread를 문제 없이 사용하려면 가시성과 원자성을 확보해야 한다. 이렇게 정의 할 수 있다. 필자가 이야기 하는 "~해야 한다."가 비단 개발자의 노력만으로는 이루어 질수 없음을 먼저 밝혀둔다. (CPU, OS, JVM 등이 지원을 해야한다는 말이다.) 참 노파심에서 집고 넘어가는데 가시성과 원자성은 여러 Thread가 동시에 접근이 가능한 공유변수에 대한 이야기다. Thread을 많이 작성해봤지만 이런 내용는 금시초문이다 하는 분들은 아마도 각각의 Thread가 각각의 변수만을 다루는 격리된 설계상에서 구현된 프로그램이였을 것이라 상상해본다.
</p>

![repsimg_20190905](/assets/img/blog/java-concurrent/multi-thread-shared-resource.png){: .img-center}
<p style='text-align: center;'>
[공유 변수에 대한 스레드간 경합]
</p>

![repsimg_multi-thread-isolation](/assets/img/blog/java-concurrent/multi-thread-isolation.png){: .img-center}
<p style='text-align: center;'>
[격리된 멤버변수 접근(비 경합)]
</p>

# 가시성(Visibility)
<p style='text-align: justify;'>
먼저 가시성을 이해 해보자. 가시성 그러니까 잘 보임의 대상은 무엇일까? 위에서 이미 거론 했지만 우리는 공유하는 변수에 대해 다루고 있다. 이 변수가 H/W에서는 Main-Memory에 적재 된다고 알고 있다. 맞다. 헌데 int i = 10; 이렇게 하면 그냥 메모리에 딱 하고 들어가 끝나면 좋겠는데 성능이니 뭐니 하면서 시스템은 아주 복잡한 설계를 해놨다. Thread는 동작하는 시점에 하나의 CPU(요즘에는 core라고 해야 맞게다.)를 점유하고 동작을 한다. 선언한 변수의 값이 Memory에만 존재하는 것이 아니라 CPU cache라고 하는 영역에도 존재한다. 학창시절에 풀이과정 까지 필요한 문제를 풀때 바로 답안지에 적는 것이 아니고 문제지에 풀어서 옮기는 것을 상상해보면 된다. CPU는 Memory까지 왔다 갔다 하는 시간을 아끼려 했나보다. 더 큰 문제는 CPU cache에 값이 Memory에 언제 옮겨 갈지도 모른다는 것이다. 이를 해결하는 것을 가시성이라고 한다. 이 해법에 대해서는 다음 이어질 글들에서 다루기로 한다.
</p>

![repsimg_cache-by-cpu](/assets/img/blog/java-concurrent/cache-by-cpu.png){: .center-block :}
<p style='text-align: center;'>
[각 Thread가 각기 바라보는 CPU의 cashe]
</p>

![repsimg_cache-use-visibility](/assets/img/blog/java-concurrent/cache-use-visibility.png){: .center-block :}
<p style='text-align: center;'>
[CUP1은 7까지 증가했으나 CPU2의 cache는 아직도 0으로 알고 있다.]
</p>

# 원자성(Atomicity)
<p style='text-align: justify;'>
가시성이라는 용어도 그렇지만 원자성이라는 이 단어 역시 상당히 은유적이다. 가시성 역시 메모리 가시성이라고 하면 좀더 쉽게 와닿을것이고 원자성 역시 연산의 원자성이라고 하면 좀더 이해가 쉽다. 관련 도서나 자료에서는 이를 연산의 단일성 혹은 단일 연산이라고 부르기도 한다. 공유되는 변수를 변경 할 때 기존의 값을 기반으로 하여 새로운 값이 결정되는 과정에서 여러 Thread가 이를 동시에 수행 할 때 생기는 이슈를 부르는 명칭이기도 하다. 다수의 책이나 자료에서 다루는 예가 i++; 연산이다. 자연어 입장에서는 하나의 문장이지만 이를 CPU가 수행 하기 위해서는 총 3가지의 instruction이 동작한다. i의 기존 값을 읽는다. i에 1을 더한다. i의 값을 변수에 할당한다. 이를 2개의 Thread가 동시에 100회 실시 한다고 했을때 만약 이 i++이 원자성을 가지고 있는 연한이라고 하면 결과가 200이어야 하겠지만 이를 프로그램으로 수행해보면 200보다 작은 값이 도출된다. 원인은 다시한번 이야기 하지만 i++이 3개의 instruction으로 구성되어 있기 때문에 Thread-1이 값을 읽어 i+1을 하기 직전에 Thread-2가 i을 읽어 i+1을 수행하고 반영하는 동작을 수행 한다면 후자의 연산은 무효가 되는 현상이 발생하는 것이다. 
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
<p style='text-align: justify;'>
상기 코드는 원자성을 확보하지 못하여 기대 했던 200보다 작은 값이 도출된다. 첫 행의count변수 선언시 volatile을 지정하면 기대 했던 결과를 얻을 수도 있지만 속으면 않된다. 횟수를 늘리거나 성능이 떨어지는 PC에서 테스트 하면 기대치 보다 작은 숫자를 보게된다.
</p> 

![repsimg_cache-use-visibility](/assets/img/blog/java-concurrent/cache-use-visibility_problem.png){: .center-block :}
<p style='text-align: center;'>
[가시성 문제를 해결해도 원자성이 확보되지 못하면 스레드는 여전히 불안]
</p>

# 애필로그
<p style='text-align: justify;'>
거창하게 이름 붙여진 JAVA concurrent programmming이라는 제목에서 느낄수 있겠지만 복잡하고 상당한 분량을 예고하고 있다. 이번 편에서는 병렬 프로그램의 뿌리가 되는 2가지의 용어의 개념을 이해 하자는 차원에서 그림과 간단한 코드로만 설명했다. 문제를 확실히 이해 했으면 이제 풀어나가면 된다. 모든 과학이 그러 하듯이....
</p>

# 참고자료
[Java Volatile Keyword](http://tutorials.jenkov.com/java-concurrency/volatile.html)


