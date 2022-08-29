# [Item 81] wait와 notify보다는 동시성 유틸리티를 애용하라.

---

> wait과 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자
> 

지금은 wait과 notify를 사용해야할 이유가 많이 줄었다. 자바 5에서 도입된 고수준의 동시성 유틸리티 덕분이다.

java.util.concurrent의 유틸리티는 세 가지로 나눌 수 있다. ( 여기서는 2,3번만 주목 )

1. 실행자 프레임워크
2. 동시성 컬렉션 ( concurrent collection )
3. 동기화 장치 ( synchronizer )

## 동시성 컬렉션 ( Concurrent Collection )

- 동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 고려하여 구현한 고성능 컬렉션
- 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행함
- **동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.**
- 동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일은 불가능하다. 그래서 여러 여러 기본 동작을 하나의 원자적 동작으로 묶는 ‘**상태 의존적 수정**’ 메서드들이 추가되었습니다.

## 동기화 장치 ( Synchronizer )

- 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.
- 가장 자주 쓰이는 동기화 장치는 `Count DownLatch`와 `Semaphore`이고 `CyclicBarrie`와 `Exchanger`는 그보다 덜 쓰인다.
- 가장 강력한 동기화 장치는 `Phaser` 다.

### CountDownLatch 예제 코드

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.Executor;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchExam {
    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비를 마쳤음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    System.out.println("Start at " + Thread.currentThread().getName());
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();
                }
            });
        }

        ready.await();  // 모든 작업자가 준비될 때까지 기다린다.
        System.out.println("Ready!");
        long startNanos = System.nanoTime();
        start.countDown();  // 작업자들을 깨운다.
        done.await();   // 모든 작업자가 일을 끝마치기를 기다린다.
        System.out.println("Done!");
        return System.nanoTime() - startNanos;
    }

    public static void main(String[] args) throws Exception {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
            ExecutorService executorService = Executors.newFixedThreadPool(10);

            while (true) {
                String str = br.readLine();
                if (str.equals("exit")) break;

                Runnable action = () -> System.out.println(Thread.currentThread().getName() + " : " + str);
                System.out.println(time(executorService, 10, action));
            }
        }
    }
}
```

- CountDownLatch ( Latch: 걸쇠 )는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
- ready latch: 스레드들이 준비가 완료됨을 타미어 스레드에 통지할 때 사용
- 통지를 끝낸 작업자 스레드들은 start latch가 열리기를 기다림
- 마지막 작업자 스레드가 ready.countDown() 메서드를 호출하면, 잠들어 있던 모든 작업자 스레드 깨어나서 start.countDown() 메서드가 호출됨
- 이후에 action.run()이 호출되어 기능이 수행됨
- 모든 작업자 스레드가 작업을 끝낼 때까지 done 래치에 의해 대기
- done 래치가 열리면 마지막 남은 로직들이 수행

Time 메서드에 넘겨진 **실행자(executor)는 concurrent 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다.** 그렇지 못하면 이 메서드는 끝나지 못하게 되고, 이러한 상태를 스레드 기아 교착상태(thread starvation deadlock)라 합니다.

## wait과 notify의 주의사항

- 새로운 코드를 작성하는 경우라면 언제나 동시성 유틸리티를 쓰는 것이 좋다. 하지만 기존의 레거시 코드를 다뤄야 하는 경우에는 wait()과 notify() 코드를 다뤄야 할 때도 있다.
- wait() 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용함.
    - 락 객체의 wait() 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야한다.
    - wait() 메서드를 사용하는 표준 방식
    
    ```java
    synchronized (obj) {
        while (<조건이 충족되지 않았다>)
            obj.wait();	// 락을 놓고 대기 상태로 들어간다.
            
        // 조건이 충족됐을 때의 동작을 수행한다.
    }
    ```
    
    - wait() 메서드를 사용할 때는 반드시 대기 반복문 ( wait loop ) 관용구를 사용해야 한다.
    - 그리고 반복문 밖에서는 절대로 호출해서는 안된다.
    - 이 반복문은 wait() 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.
- notify와 notifyAll 중에서 선택해야 하는 경우
    - notifyAll을 사용하는게 안전함
    - 깨어나야 하는 모든 스레드가 깨어남을 보장할 수 있으니 항상 정확한 결과를 얻을 수 있기 때문입니다.
    - 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify를 사용해 최적화하라.

# [item 82] ****스레드 안전성 수준을 문서화하라****

---

> 한 메서드를 여러 스레드가 동시에 호출할 때 그 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약과 같다.
> 

## API 문서에 synchronized 한정자가 보이는 메서드

- API 문서에 synchronized 한정자가 보이는 메소드는 스레드 안전하다고 100% 확신할 수 없다.
- 자바독이 기본 옵션에서 생성한 API 문서에는 synchronized 한정자가 포함되지 않는다.
- 메소드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않기 떄문이다.
- synchronized 유무로 스레드 안전성을 알수있다는 것은 모 아니면 도라는 위험한 발상이다.

**멀티스레드 환경에서도 API를 안전하게 사용하려면 클래스가 지원하는 스레드 안정성 수준을 정확히 명시하자.**

## 스레드 안전성 수준

---

- 스레드 안전성에도 수준이 나뉜다.
- **멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.**

### 스레드 안전성 높은 순서

1. **불변**
    1. 이 클래스의 인스턴스는 마치 상수같아서 외부 동기화도 필요 없다
2. **무조건적인 스레드 안전**
    1. 이 클래스의 인스턴스는 수정될 수 있으나 내부에서 잘 동기화하여 외부 동기화 없이 동시에 사용해도 안전하다.
3. **조건부 스레드 안전**
    1. • 무조건적 스레드 안전과 같으나 일부 메서드는 동시 사용을 위해 외부 동기화가 필요하다.
4. **스레드 안전하지 않음**
    1. 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의 메서드 호출을 외부 동기화로 감싸야 한다.
5. **스레드 적대적**
    1. 모든 메서드 호출을 외부 동기화로 감싸도 멀티스레드 환경에서 안전하지 않다. 이 수준의 클래스는 보통 정적 데이터를 아무 동기화 없이 수정한다.

## 문서화

- 조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다.
- 클래스의 스레드 안전성은 보통 클래스의 문서화 주석에 기재하지만, 독특한 특성의 메소드면 해당 메소드의 주석에 기재하자
- 열거타입은 굳이 불변이라고 쓰지 않아도 된다.
- 반환 타입만으로 명확히 알 수 없는 정적 팩토리라면 자신이 반환하는 객체의 스레드 안전성을 반드시 문서화하자

## 비공개 Lock 객체 사용

---

- 클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다.
- 하지만 이러한 유연성은 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없는 단점이 있다.
- 그래서  `ConcurrentHashMap` 같은 동시성 컬렉션과는 함께 사용하지 못하고, 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 **서비스 거부 공격**(denial-of-service attack)을 수행할 수도 있다.
- **서비스 거부 공격을 막으려면 공개된 락과 마찬가지인 synchronized 메소드 대신 비공개 락 객체를 사용하자.** 즉, 락 객체를 동기화 대상 객체 안으로 캡슐화하는 것.

```java
private final Object lock = new Object();

public void foo() {
    synchronized(lock) {
        ...
    }
}
```

- 비공개 락 객체는 클래스 외부에서 못 보니 클라이언트가 그 객체의 동기화에 관여할 수 없다.
- 리고 **lock 필드는 final로 선언하자.** 우연히라도 락 객체가 교체되는 일을 예방한다.
- 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용할 수 있고, 조건부 스레드 안전 클래스에서는 특정 호출 순서에 필요한 락이 무엇인지 클라이언트에게 알려줘야 해서 이 관용구를 사용할 수 없다.
- 부모 클래스에서 synchronized 메소드나 블록을 사용하고 오래동안 lock을 쥐고있다고 가정해보자.
    - 자식 클래스에서 이 메소드를 super를 통해 호출한다면 자식 클래스에 lock이 생긴다.
    - 자식 클래스에서 다른 synchronized 메소드나 블록을 사용하려 하면 lock이 걸려있기 때문에 당장 사용할 수 없는 문제가 발생한다.
    - 하지만 비공개 락 객체 관용구를 사용한다면 이 문제에서 자유롭다.

---

### 핵심 정리

- 모든 클래스를 자신의 스레드 안전성 정보를 정확한 언어로 명확히 설명하거나 스레드 안전성 애노테이션을 사용해 문서화 하자.
- synchronized 한정자는 문서화와 관련이 없다.
- 조건부 스레드 안전 클래스는 메소드를 어떤 순서로 호출할 때 외부 동기화가 요구되고, 어떤 락을 얻어야 하는지도 알려줘야 한다.
- 무조건적 스레드 안전 클래스 작성시 synchronized 메소드 대신 비공개 락 객체를 사용하자. 그래야 클라이언트나 하위 클래스에서 동기화 메커니즘을 깨뜨리는 걸 예방할 수 있고 필요시 다음에 더 정교한 동시성을 제어 메커니즘으로 재구현할 여지가 생긴다.

# [Item 83] 지연 초기화는 신중히 사용하라

---

> 지연 초기화(lazy initialization)는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다.
> 
- 이 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다.
- 이 기법은 정적 필드와 인스턴스 필드 모두에 사용할 수 있다.
- 지연 초기화는 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

## 지연 초기화의 단점

- 다른 모든 최적화와 마찬가지로 지연 초기화에 대해 해줄 최선의 조언은 "**필요할 때까지는 하지 말라**"다.
- 지연초기화는 양날의 검이다. 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 지연 초기화는 필드에 접근하는 비용은 커진다.
- 실제 초기화에 드는 비용에 따라, 초기화된 각 필드를 얼마나 빈번히 호출하느냐에 따라 지연초기화가 (다른 수많은 최적화와 마찬가지로) 실제로는 성능을 느려지게 할 수도 있다.

## 지연 초기화가 필요할 때

- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다.
- 하지만 안타깝게도 정말 그런지를 알 수 있는 유일한 방법은 지연 초기화 적용 전후의 성능을 측정해보는 것이다.
- 멀티스레드 환경에서는 지연 초기화를 하기가 까다롭다. 지연 초기화는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다.

**대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.**

코드 83-1 인스턴스 필드를 초기화하는 일반적인 방법

```java
private final FiledType field = computeFieldValue();
```

- 지연 초기화가 초기화 순환성(initialization circularity)을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자.

코드 83-2 인스턴스 필드의 지연 초기화 - synchronized 접근자 방식

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null)
        filed = computeFieldValue();
    return field;
}
```

- 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자.

코드 83-3 정적 필드용 지연 초기화 홀더 클래스 관용구

```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

- getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, 비로소 FieldHolder 클래스 초기화를 촉발한다.
- 이 관용구의 멋진 점은 getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는 것이다.
- 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사(double-check) 관용구를 사용하라.

코드 83-4 인스턴스 필드 지연 초기화용 이중검사 관용구

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null) { // 첫 번째 검사 (락 사용 안 함)
        return result;
    
    synchronized(this) {
        if(filed == null)  // 두 번째 검사( 락 사용)
            field = computeFieldValue();
        return field;
    }
}
```

- 이 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
- 필드의 값을 두 번 검사하는 방식으로, 한 번은 동기화 없이 검사하고, (필드가 아직 초기화되지 않았다면) 두 번째는 동기화하여 검사한다.

코드 83-5 단일검사 관용구 - 초기화가 중복해서 일어날 수 있다!

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```

- 단일 검사 관용구류는 이중검사의 변종이다.
- 반복해서 초기화해도 상관없는 인스턴스 필드를 지연초기화할 때 이중검사에서 두 번째 검사를 생략할 수 있다.

### 핵심정리

---

- 설명한 모든 초기화 기법은 기본 타입, 객체 참조 필드 모두에 적용할 수 있고 이중 검사와 단일 검사 관용구를 수치 기본 타입 필드에 적용하면 필드의 값 null 대신 0과 비교하면 된다.
- 대부분의 필드는 지연시키지 말고 곧바로 초기화하자.
- 성능이나 위험한 초기화 순환을 막기위해 써야한다면 올바른 지연 초기화 기법을 사용하자.
- 인스턴스 필드에는 이중검사 관용구를, 정적 필드에는 홀더 클래스 관용구를 사용하자
- 반복해 초기화해도 괜찮은 인스턴스 필드는 단일검사 관용구도 고려 대상이다.

# [Item 84] ****프로그램의 동작을 스레드 스케줄러에 기대지 말라****

> 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다.
> 
- 여러 스레드가 실행 중이면 운영체제의 스레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할지 정한다.
- 정상적인 운영체제라면 이 작업을 공정하게 수행하지만 구체적인 스케줄링 정책은 운영체제마다 다를 수 있다.

## **견고하고 빠릿하고 이식성 좋은 프로그램을 작성하는 가장 좋은 방법**

- **실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다.**
- 그래야 스레드 스케줄러가 고민할 거리가 줄어든다.
- 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만들자.
- 이런 프로그램이라면 스레드 스케줄링 정책이 아주 상이한 시스템에서도 동작이 크게 달라지지 않는다.
- 여기서 실행 가능한 스레드의 수와 전체 스레드 수는 구분해야 한다.

## **실행 가능한 스레드 수를 적게 유지하는 주요 기법**

- **각 스레드가 무언가 유용한 작업을 완료한 후에는 다음 일거리가 생길 때까지 대기하도록 하는 것**
- **스레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.**
- 실행자 프레임워크를 예로 들면, 스레드 풀 크기를 적절히 설정하고 작업은 짧게 유지하면 된다.
- 단, 너무 짧으면 작업을 분배하는 부담이 오히려 성능을 떨어뜨릴 수도 있다.
- **스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다.**
- 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사해서는 안 된다는 뜻이다.
- 바쁜 대기는 스레드 스케줄러의 변덕에 취약할 뿐 아니라, 프로세서에 큰 부담을 주어 다른 유용한 작업이 실행될 기회를 박탈한다.

## 이식성이 나쁜 특성

1. **Thread.yield 사용**
    1. 특정 스레드가 다른 스레드들과 비교해 CPU 시간을 충분히 얻지 못해 간신히 돌아가더라도, **Thread.yield를 써서 문제를 해결하지 말자.**
    2. 증상이 호전될 순 있지만 이식성은 그렇지 않다.JVM 버전이나 종류에 따라서 성능의 차이가 존재해 오히려 느려질 수도 있다.
    3. **Thread.yield는 테스트할 수단도 없으니**
     차라리 애플리케이션 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 조치하자.
2. **스레드 우선순위 조절**
    1. 스레드 우선순위를 조절할 수도 있지만, yield 사용 때와 비슷한 위험이 있다. **스레드 우선순위는 자바에서 이식성이 가장 나쁜 특성이다.**
    2. 스레드 몇 개 우선순위 조율해서 애플리케이션 반응 속도를 높일수도 있지만 그래야할 상황은 드물고 이식성도 떨어진다.
    
    ### 핵심 정리
    
    ---
    
    - 프로그램의 동작을 스레드 스케줄러에 기대지 말자.
    - 견고성과 이식성을 모두 해치는 행위다.
    - 같은 이유로, Thread, yield와 스레드 우선순위에 의존해서도 안 된다.
    - 이 기능들은 스레드 스케줄러에 제공하는 힌트일 뿐이다.
    - 스레드 우선순위는 이미 잘 동작하는 프로그램의 서비스 품질을 높이기 위해 드물게 쓰일 수는 있지만, 간신히 동작하는 프로그램을 '고치는 용도'로 사용해서는 절대 안 된다.
