---
author: author2
layout: post
title: Java Concurrent Programming - Atomic
subtitle: About Java Concurrent Programming, Atomic article
category: devlog
tag: java
comments: true
---

## 프롤로그
<p style='text-align: justify;'> 전편에서 다루었던 가시성과 이번에 다루려고 하는 “원자성”이라는 주제가 Multi-Thread 상황에서 Thread간 공유 메모리 이슈를 발생시킨 다는 점에서 공통분모를 가지고 있고 서로간의 상호작용을 잘 염두해두어야 한다는 것은 사실이다. 그렇지만 시스템 관점에서 보면 이 두 개념은 조금 다른 곳에 존재한다. 가시성은 CPU - Cache - Memory관계상의 개념이다. 반면 원자성은 한 줄의 프로그램 statement가 컴파일러에서 기계어로 변경되며 이를 기계가 순차적으로 처리하기 위한 여러 개의 machine instruction이 만들어져 실행되는데 여기서 이 하나의 machine instruction 이 컴퓨터 입장에서 최소 연산 단위인 “원자”적 연산이다. 말을 풀어 놨는데 여전히 어려울 것이다. 오히려 본론으로 들어가 그림과 예제 코드를 보면 이해가 쉬울 것이다.
</p>

## 원자단위의 연산의 이해
<p style='text-align: justify;'>
원자성 즉 연산의 원자 단위를 이해하기 위해 이전 편에서도 다루었던 “i++” 를 원자연산으로 분해 해보도록 하겠다.
</p>

![repsimg_atomic_instruction](/assets/img/blog/java-concurrent/i_plusplus_atomic_instruction.png){: .center-block :}
<p style='text-align: center;'>
[프로그램 언어 문장 -> machine instruction 변환]
</p>

<p style='text-align: justify;'>
 i++이 위 그림과 같이 캐싱을 배제 하더라도 읽고 > 연산하고 > 저장하는 총 3가지의 Instruction이 수행된다. 이 원자 단위의 연산이 수행 중에는 다른 Thread에 의해서 컨트롤 되는 타 CPU의  개입이 있을 수 없는 최소단위의 연산이라고 이해 하면 된다. 여기에 Multi-Thread를 개입 시켜보자. 각 instruction 수행 사이에는 다른 thread의 공유메모리(변수)의 접근이 가능하여 소위 값이 꼬이는 현상이 생기는 것이다. 아주 옛날부터 시스템이 이렇게 설계된 것을 어떻게 하겠는가? 순응 해야지 ... 지금부터 시스템에 순응하는 방법을 다뤄보도록 하겠다.
</p>

## 동기화 처리를 통한 Thread 안정성 확보
<p style='text-align: justify;'>
하나의 “원자”연산이 이루어지는 동안 이 연산은 다른 Thread - CPU에 의해 간섭을 받을 수 없다. 뭐 이건 시스템이 그렇게 구현이 되어있을 것이다. 헌데 우리가 다루고자 하는 연산은 원자 단위로 하기에는 너무 복잡하다. 각종 조건, 반복, 보조 기억장치로부터의 데이터 읽기와 쓰기 등이 복합적으로 이루어지는 연산이 섞여 있을 것이다. 이런 복잡한 연산을 i++도 한번에 못하는 시스템 에게 시킨다는 무리다. (사실 요즘 시스템은 i++를 원자 단위로 할 방법이 있다. 이 내용은 이글의 후반부에 다룬다.)  개발자가 비교적 쉽게 이를 해결할 수 있는 방법이 임계 영역(Critical Section) 지정이다. 동시에 처리되면 문제가 되어 배타적인 영역을 설정하는 것이다. 여기서 말하는 영역은 배타적 경제 수역처럼 공간적인 영역이 아닌 statement의 블록이다.
</p>

### 임계영역 설정 - synchronized 블럭
<p style='text-align: justify;'>
코드의 가 독성 측면에서 봤을 때 가장 좋은 방법이다. 단, 가 독성이 좋다고 성능도 좋은 것은 아니니 오해 없길 바란다. 이 블록을 설정 해놓으면 그 구간에는 Thread하나만 접근한다. 다른 Thread가 접근하려고 하면 기다려야 한다. 이를 Lock이라 한다. 문법은 다음과 같다.
</p>

```java
synchronized(락-객체) { 
    // 임계 영역 (Multi-Thread의 동시 접근이 불가능한 공유자원 복합연산) 
}
```
<p style='text-align: center;'>
[synchronized 블럭을 이용한 임계영역 설정]
</p>

### 임계영역 설정 - synchronized 함수
<p style='text-align: justify;'>
이 방법은 함수가 통체로 임계 영역으로 구성되어야 할 때 사용하는 방법이다. 문법은 다음과 같다.
</p>

```java
public synchronized void method() { 
    // 자원 경합이 일어나는 코드 
}
```
<p style='text-align: center;'>
[synchronized 함수를 이용한 임계영역 설정]
</p>

<p style='text-align: justify;'>
처음 보는 분들은 Lock-객체의 부재에 의문점을 갖는 독자들도 있으리라 본다. 적어도 필자는 그랬다. 답은 의외로 간단하다. this가 생략 되었다고 보면 된다. 해당 class의 모든 synchronized함수 그리고 this를 사용하는 블록의 lock이 공유되는 것이다. 하나의 임계 영역만 Thread가 점유해도 모두 못 들어 간다. 
지금까지 설명한 두 가지 동기화 블록 설정을 통한 Thread안정성 확보 방법 외에도 명시적으로 임계 영역의 시작을 lock하고 끝나면 Unlock하는 방법도 있다. 뭐가 되었던 지금까지 설명한 동기화 처리 방법은 여러 Thread를 그야 말로 동기 처리 하는 것이다. 이는 마치 개수가 한정되어 있는 또는 하나 밖에 없는 도구를 사용 해야 할 때 다른 사람의 사용이 끝날 때 까지 기다려야 하는 이치와 같다. 이를 책이나 여러 자료에서는 “Blocking” 또는 “동기화” 라 부른다
</p> 


## 단일연산(atomic) 변수를 이용한 None-Blocking 동기화
<p style='text-align: justify;'>
앞에서 다룬 Blocking/동기화는 여러가지 단점이 존재한다. 그 중에서도 손꼽는 문제가 성능이슈이다. 어떤 Thread는 Lock을 확보하느라 또 다른 Thread는 Lock을 확보하지 못해 Blocking상태에 들어가느라 그리고 이 상태가 변경이 되는 동안 많은 시스템 자원이 쓰인다고 한다. 결국 이 문제는 성능 문제로 이어진다. 차를 운전 할 때도 시내에서는 방향전환을 위해서는 해당 차선에 일단 서야한다. 나보다 앞에 와서 서 있는 차량이 먼저 지나가는 것도 기다려야 한다. 차가 서고 다시 출발하는 많은 에너지가 소비되는것과 비슷한 이치이다. 
 
최근의 CPU는 이러한 문제를 해결하기 위해 atomic hardware primitives를 제공한다. 예를 들어 i++을 단일 연산으로 처리 할 수 있는 방법을 제시하는 것이다. 이 instruction의 동작원리는 다음과 같다. 
</p>

* 인자로 기존 값과 변경할 값을 전달한다.
* 기존 값으로 던진 값이 현재 시스템이 가지고 있는 값과 같다면 변경할 값을 반영해준다. 반환 값으로 true 리턴 한다.
* 반대로 기존 값으로 던진 값이 현재 시스템이 가지고 있는 값과 다르다면 값을 반영 하지 않고 false를 리턴 한다.

<p style='text-align: justify;'>
이게 다야 하는 독자도 있겠지만 이게 어딘가? 근데 기존 값과 다른 경우는 뭐지 그 사이에 다른 Thread가 들어가서 바꿔 놨다는 말이다. 그러니 이런 경우에는 false를 반환한다. 그 이후는 개발자 보고 알아서 하라는 말이다. 일반적으로는 loop를 구성하여 다시 기존 값을 읽고 같은 시도를 한다. 만약 뭐 바쁜 일이 있으면 다른 일을 해도 된다. 개발자 마음이다. 다른 사람이 하나밖에 없는 도구를 쓰고 있다고 계속 기다리는 것도 미련한 짓 아닌가? 다른 일이 없어서 Loop를 돌면서 계속 “내 값을 반영 해주겠냐”를 물어본다고 해도 Blocking이 일어나는 것 보다는 성능 적으로 우수하다. 이와 같은 연산 방식을 CAS(Compare And Swap)이라고 한다. 모든 단일 연산 변수의 핵심은 이 부분이다. 이를 이용하여 자료구조를 안전하게 구현하는 것을 lock-free알고리즘 이라고 부른다.
 
다음 코드는 단일 연산 변수 중 하나인 AtomicIntege를 이용하여 Thread에 안전한 카운터를 구현하여 Test한 예제이다. 두 개의 Thread는 동시에 시작하여 0 ~ 10만회 카운트를 증가시킨다. 결과는 20만이 나온다. 만약 Thread 안정성에 문제가 있다면 20만 이하의 숫자가 반환 될 것이고, Lock을 이용한다면 Thread간의 경합으로 인해 상대적으로 성능이 저하 될 것이다.
</p>


```java
@Test
public void atomicIntegerTest() {
    final int REPEAT_INCREMENT = 100000;
    AtomicInteger forAtomicIntegerTest = new AtomicInteger();
    forAtomicIntegerTest.set(0);
    ExecutorService es = Executors.newFixedThreadPool(2);
    ExecutorService[] ess = new ExecutorService[] {es};
    es.execute(new Runnable() {
        @Override
        public void run() {
            int cnt = 0;
            while (!Thread.currentThread().isInterrupted()) {
                int current = -1;
                cnt++;
                if (cnt > REPEAT_INCREMENT)
                    break;
                do {
                    // 현재 값을 읽어 비교 대상으로 해야 한다.
                    // 그래야 CAS 연산시 변경이 발생 했는지 알 수 있다.
                    current = forAtomicIntegerTest.get();
                } while (!forAtomicIntegerTest.compareAndSet(current, current + 1));
            }
            System.out.println(String.format("CHECK POINT >>>> End of %s thread."
                , Thread.currentThread().getName()));
            synchronized (ess) {
                ess.notify();
            }
        }
    });
    es.execute(new Runnable() {
        @Override
        public void run() {
            int cnt = 0;
            while (!Thread.currentThread().isInterrupted()) {
                cnt++;
                if (cnt > REPEAT_INCREMENT)
                    break;
                // 단순히 읽기만 할꺼라면 현재 값을 읽는(get()) 함수를 사용할 필요가 없다.
                forAtomicIntegerTest.incrementAndGet();
            }
            System.out.println(String.format("CHECK POINT >>>> End of %s thread."
                , Thread.currentThread().getName()));
            synchronized (ess) {
                ess.notify();
            }
        }
    });
    es.shutdown();
    try {
        es.awaitTermination(10, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    int result = forAtomicIntegerTest.get();
    System.out.println( String.format( "RESULT OF 2 Threads >>>> %s", result));
    Assert.assertEquals(REPEAT_INCREMENT * 2, result);
}
```
<p style='text-align: center;'>
[AtomicInteger 사용예]
</p>

```java
@Test
public void atomicReferenceTest_Stack() {
    final int REPEAT_INCREMENT = 100000;
    ConcurrentStack<Integer> stack = new ConcurrentStack<Integer>();
    ExecutorService es = Executors.newFixedThreadPool(2);
    ExecutorService[] ess = new ExecutorService[] {es};
    es.execute(new Runnable() {
        @Override
        public void run() {
            int cnt = 0;
            while (!Thread.currentThread().isInterrupted()) {
                cnt++;
                if (cnt > REPEAT_INCREMENT)
                    break;
                stack.push(cnt);
            }
            System.out.println(String.format("CHECK POINT >>>> End of %s thread."
                , Thread.currentThread().getName()));
            synchronized (ess) {
                ess.notify();
            }
        }
    });
    es.execute(new Runnable() {
        @Override
        public void run() {
            int cnt = 0;
            while (!Thread.currentThread().isInterrupted()) {
                cnt++;
                if (cnt > REPEAT_INCREMENT)
                    break;
                stack.push(cnt);
            }
            System.out.println(String.format("CHECK POINT >>>> End of %s thread."
                , Thread.currentThread().getName()));
            synchronized (ess) {
                ess.notify();
            }
        }
    });
    es.shutdown();
    try {
        es.awaitTermination(10, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    int result = stack.size();
    System.out.println( String.format( "RESULT OF 2 Threads >>>> %s", result));
    Assert.assertEquals(REPEAT_INCREMENT * 2, result);
    int topOfStack = stack.pop();
    System.out.println( String.format( "TOP OF STACK VALUE >>>> %s", topOfStack));
    Assert.assertEquals(REPEAT_INCREMENT, topOfStack);
}
```

```java
class ConcurrentStack<E> {

    AtomicInteger size = new AtomicInteger(0);
    AtomicReference<Node<E>> head = new AtomicReference<Node<E>>();

    public void push(E item) {
        Node<E> newHead = new Node<E>(item);
        Node<E> oldHead;
        do {
            oldHead = head.get();
            newHead.next = oldHead;
        } while (!head.compareAndSet(oldHead, newHead));
        size.incrementAndGet();
    }

    public E pop() {
        Node<E> oldHead;
        Node<E> newHead;
        do {
            oldHead = head.get();
            if (oldHead == null) {
                return null;
            }
            newHead = oldHead.next;
        } while (!head.compareAndSet(oldHead, newHead));
        size.decrementAndGet();
        return oldHead.item;
    }

    public int size() {
        return size.get();
    }

    static class Node<E> {
        final E item;
        Node<E> next;

        public Node(E item) {
            this.item = item;
        }
    }
}
```

<p style='text-align: center;'>
[AtomicReference 사용예]
</p>

## 애필로그
<p style='text-align: justify;'>
가시성과 더불어 원자성을 이해하고 이로 인해 발생할 수 있는 Thread안정성 문제와 이를 해결하기 위한 기법에 대해서 간략히 정리 해봤다. 실제 병렬 처리를 수행하는 프로그램을 작성해보면 정말 고민해야 할 부분이 많다. 그러다 보면 코드가 복잡해진다. 더욱이 성능 문제 때문에 Lock-Free를 구현하고자 한다면 코드는 한층 더 복잡해질 것이다. 필자도 앞으로 작성해야 하는 병렬처리 코드 때문에 머리가 아파온다.
</p>

> 참고자료
> https://www.ibm.com/developerworks/java/library/j-jtp04186/
