# 11장. 동시성

<aside>
✏️ 동시성 프로그램(멀티 스레드 프로그램)을 명확하고 정확하게 만들고 잘 문서화하는 방법

</aside>

## *아이템 78.* 공유 중인 가변 데이터는 동기화해 사용하라 *(p.414)*

- `synchronized` 키워드는 해당 메서드나 블록을 한 번에 **한 스레드씩 수행**하도록 보장

- **동기화의 기능**
    1. **일관성이 깨진 상태를 볼 수 없게** **함** (변경 중인 객체를 다른 스레드가 보지 못하도록)
    2. 동기화된 메서드나 블록에 들어간 스레드가 **같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해줌**
    - 자바 명세 상 `long`과 `double` 외의 변수를 읽고 쓰는 동작은 원자적
        - ⇒ 스레드가 필드를 읽을 때 항상 ‘**수정이 완전히 반영된**’ 값을 얻는다고 보장
        - 한 스레드가 저장한 값이 다른 스레드에게 ‘**보이는가**’는 보장하지 않음
    - ⇒ 동기화는 배타적 실행뿐 아니라 **스레드 사이의 안정적인 통신**에 꼭 필요
    
- **다른 스레드를 멈추는 작업**
    - `Thread.stop` : 안전하지 않아 오래 전에 deprecated API로 지정
    - `boolean` 필드 이용
        
        ```java
        /* 잘못된 코드 */
        public class StopThread {
        	private static boolean stopRequested;
        	
        	public static void main(String[] args) throws InterruptedException {
        		Thread backgroundThread = new Thread(() => {
        			int i = 0;
        			while (!stopRequested)  // stopRequested가 false인 동안 실행되는 스레드
        				i++;
        		});
        
        		backgroundThread.start();
        		TimeUnit.SECONDS.sleep(1);
        		stopRequested = true;
        	}
        }
        ```
        
        - ⇒ 멈추지 않고 계속 수행
        - 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 보게 될지 보증할 수 없음
        
        ```java
        /* 적절한 코드 */
        public class StopThread {
        	private static boolean stopRequested;
        
        	private static synchronized void requestStop() {
        		stopRequested = true;
        	}
        
        	private static synchronized boolean stopRequested() {
        		return stopRequested;
        	}
        	
        	public static void main(String[] args) throws InterruptedException {
        		Thread backgroundThread = new Thread(() => {
        			int i = 0;
        			while (!stopRequested())
        				i++;
        		});
        		backgroundThread.start();
        
        		TimeUnit.SECONDS.sleep(1);
        		requestStop();
        	}
        }
        ```
        
        - **쓰기와 읽기 모두가 동기화**되지 않으면 동작을 보장하지 않음
        
        - 속도가 더 빠른 대안 - `stopRequested` 필드를 `volatile`로 선언하면 동기화 생략 가능
            - **가장 최근에 기록된 값을 읽게 됨**을 보장
            - **원자성(배타적 실행)은 지원하지 않으므로** 주의해서 사용!
        
    - `java.util.concurrent.atomic` 패키지에는 락 없이도(lock-free) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있음
        
        ```java
        private static final AtomicLong nextSerialNum = new AtomicLong();
        public static long generateSerialNumber() {
        	return nextSerialNum.getAndIncrement();
        }
        ```
        
    
- 동기화 문제를 피하는 가장 좋은 방법은 **가변 데이터는 단일 스레드에서만 사용**하는 것
- 한 스레드가 데이터를 다 **수정한 후 다른 스레드에 공유**할 때는, **공유하는 부분만 동기화**해도 됨
    - ⇒ 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽을 수 있음
    - ⇒ **사실상 불변(effectively immutable)**
    - 이러한 객체를 건네는 행위: **안전 발행(safe publication)**
    - 객체를 안전하게 발행하는 방법 - 클래스 초기화 과정에서 객체를 **정적** 필드, `volatile` 필드, `final` 필드, **lock을 통해 접근**하는 필드에 저장하거나 **동시성 컬렉션** (*아이템 81*)에 저장
    

> **핵심 정리**
여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다.
배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면 `volatile` 한정자만으로 동기화할 수 있지만 올바로 사용하기가 까다롭다.
> 

## *아이템 79*. 과도한 동기화는 피하라 *(p.420)*

- **과도한 동기화**는 **성능**을 떨어뜨리고, **교착상태**에 빠뜨리고, **예측할 수 없는 동작**을 낳기도 함
- **응답 불가(교착 상태)**와 **안전 실패(데이터 훼손)**를 피하려면 **동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 됨**
    - 동기화된 영역 안에서 **재정의할 수 있는 메서드**는 호출하면 안 되며, **클라이언트가 넘겨준 함수 객체**(*아이템 24*)를 호출해서도 안 됨 :**외계인 메서드(alien method)**
        
        ```java
        public class ObservableSet<E> extends ForwardingSet<E> {
        	public ObservableSet(Set<E> set) { super(set); }
        	
        	private final List<SetObserver<E>> observers = new ArrayList<>();
        
        	...
        
        	private void notifyElementAdded(E element) {
        		synchronized(observers) {
        			for (SetObserver<E> observer : observers)
        				observer.added(this, element); // 외계인 메서드 호출
        		}
        	}
        	...
        }
        
        @FunctionalInterface public interface SetObserver<E> {
        	void added(ObservableSet<E> set, E element);
        }
        ```
        
    
    - 방법 1. **외계인 메서드 호출**을 **동기화 블록 바깥**으로 옮김
        
        ```java
        private void notifyElementAdded(E element) {
        	List<SetObserver<E>> snapshot = null;
        	synchronized(observers) {
        		snapshot = new ArrayList<>(observers);
        	}
        	for (SetObserver<E> observer : snapshot)
        		observer.added(this, element);
        }
        ```
        
    - 방법 2. JAVA의 동시성 컬렉션 라이브러리의 `CopyOnWriteArrayList` 사용
        - 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현된 `ArrayList`
        
        ```java
        private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();
        
        public void addObserver(SetObserver<E> observer) {
        	observers.add(observer);
        }
        
        public boolean removeObserver(SetObserver<E> observer) {
        	return observers.remove(observer);
        }
        
        private void notifyElementAdded(E element) {
        	for (SetOberver<E> observer : observers)
        		observer.added(this, element);
        }
        ```
        
        ⇒ 명시적 동기화 없이 구현 가능
        
- **성능 측면**에서 **가변 클래스**를 작성할 때 선택지
    1. 동기화를 전혀 사용하지 말고, **그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화**하게 함
    2. **동기화를 내부에서 수행**해 스레드 안전한 클래스로 만듦(*아이템 82*)
        - 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 **동시성을 월등히 개선**할 수 있을 때에만 선택
        - 락 분할(lock splitting), 락 스트라이핑(lock striping), 비차단 동시성 제어(nonblocking concurrency control) 등 기법을 동원해 동시성 향상 가능

> **핵심 정리**
교착 상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자.
가변 클래스를 설계할 때는 스스로 동기화해야 할지 고민하자.
합당한 이유가 있을 때에만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자(*아이템 82*).
> 

## *아이템 80*. 스레드보다는 실행자, 태스크, 스트림을 애용하라 *(p.428)*

- `java.util.concurrent` 패키지에는 **실행자 프레임워크(Executor Framework)** 인터페이스 기반의 유연한 **태스크 실행 기능**이 있음
    
    ```java
    /* 작업 큐 생성 */
    ExecutorService exec = Executors.newSingleThreadExecutor();
    
    /* 이 실행자에 실행할 태스크(task; 작업)를 넘김 */
    exec.execute(runnable);
    
    /* 실행자 종료 */
    exec.shutdown();
    ```
    
    - **특정 태스크가 완료**되기를 기다림 (`get` 메서드)
    - 태스크 모음 중 **아무것 하나** (`invokeAny` 메서드) 혹은 **모든 태스크** (`invokeAll` 메서드)가 **완료**되기를 기다림
    - **실행자 서비스가 종료**하기를 기다림 (`awaitTermination` 메서드)
    - **완료된 태스크들의 결과**를 차례로 받음 (`executorCompletionService` 이용)
    - 태스크를 **특정 시간에 혹은 주기적으로 실행**하게 함 (`ScheduledThreadPoolExecutor` 이용)
    
- **큐를 둘 이상의 스레드가 처리**하게 하고 싶다면 다른 정적 팩터리를 이용하여 **다른 종류의 실행자 서비스(스레드 풀)을 생성**하면 됨
    - 대부분 `java.util.concurrent.Executors` 의 정적 팩터리들을 이용해 생성 가능
    - **작은 프로그램**이나 **가벼운 서버**라면 `Executors.newCachedThreadPool` 사용
    - **무거운 프로덕션 서버**에서는 스레드 개수를 고정한 `Executors.newFixedThreadPool` 또는 완전히 통제할 수 있는 `ThreadPoolExecutor` 사용
    
- **스레드를 직접 다루는 것은 일반적으로 삼가야 함**
    - 스레드를 직접 다루면 `Thread`가 **작업 단위와 수행 메커니즘 역할을 모두 수행**
    - **실행자 프레임워크**에서는 **작업 단위와 실행 메커니즘이 분리**
    
    - **작업 단위**: **태스크** - `Runnable`, `Callable` (`Callable`은 **값을 반환**하고 **임의의 예외**를 던질 수 있음)
    - **태스크를 수행**하는 일반적인 메커니즘: **실행자 서비스**
        - ⇒ 실행자 프레임워크를 사용하면 태스크 수행을 실행자 서비스에 맡길 수 있음
    
- JAVA 7부터 실행자 프레임워크는 **포크-조인(fork-join) 태스크**를 지원하도록 확장
    - `ForkJoinTask`는 `ForkJoinPool`이라는 실행자 서비스가 실행
    - `ForkJoinTask` 인스턴스는 **하위 태스크**로 나뉠 수 있고, `ForkJoinPool`을 구성하는 **스레드들이 이 태스크들을 처리**, 일을 먼저 끝낸 스레드는 **다른 스레드의 남은 태스크를 대신 처리**
    - ⇒ **높은 처리량**과 **낮은 지연시간** 달성