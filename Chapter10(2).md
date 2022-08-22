# 10장. 예외

## 아이템 73. 추상화 수준에 맞는 예외를 던져라
- 예외 번역
  - 상위 계층에서 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던지는 방식
    ```java
    try {
        ... // 저수준 추상화 이용
    } catch (LowerLevelException e) {
        throw new HighterLevelException(...); // 추상화 수준에 맞게 번역
    }
    ```
    - 실제 사용 예
    ```java
    /**
    * 이 리스트 안의 지정한 위치의 원소를 반환한다.
    * @throws IndexOutOfBoundsException index가 범위 밖이라면, 즉 ({@code index < 0 || index >= size()})이면 발생한다.
    */
    public E get (int index) {
        ListIterator<E> i = listIterator(index);
        try {
            return i.next();
        } catch (NoSuchElementException e) {
            throw new IndexOutOfBoundsException("인덱스: " + index)
        };
    }
    ```
- 예외 연쇄
  - 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식
  - 저수준 예외가 디버깅에 도움이 될 때 사용
    ```java
    try {
        ... // 저수준 추상화 이용
    } catch (LowerLevelException cause) { 
    throw new HigherLevelException(cause);
        }
    ```
- 고수준 예외의 생성자는 상위 클래스의 생성자에 '원인'을 건네주어, 최종적으로 Throwable(Throwable) 생성자까지 건네지게 한다
    ```java
    class HigherLevelException extends Exception {
        HigherLevelException(Throwable cause) {
            super(cause);
        }
    }
    ```
- 무턱대고 예외를 전파하는 것보다야 예외 번역이 우수한 방법이지만, 그렇다고 남용해서는 곤란하다.
- 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.
  - 상위 계층에서 그 예외를 조용히 처리하도록 하자!!
  - java.util.logging 같은 적절한 로깅을 활용하자!!
- 클라이언트 코드와 사용자에게 문제를 전파하지 않으면서도 프로그래머가 로그를 분석해 추가 조치를 취할 수 있게 해준다.

## 아이템 74. 메서드가 던지는 모든 예외를 문서화하라
- 검사 예외는 항상 따로 따로 선언하고, 각 예외가 발생하는 상황을 자바독의 @throws 태그를 사용하여 정확히 문서화하자.
- 메서드가 던질 수 있는 예외를 각각 @throws 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.
  - 비검사 예외는 일반적으로 프로그래밍 오류를 뜻한다.
- 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그 예외를 클래스 설명에 추가하는 방법도 있다.

## 아이템 75. 예외의 상세 메시지에 실패 관련 정보를 담으라
- 예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적 정보를 자동으로 출력한다.
  - 스택 추적 :  예외 객체의 toString 메서드를 호출해 얻는 문자열로, 보통 예외의 클래스 이름 뒤에 상세 메시지가 붙는 형태이다.
- 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.
- 관련 데이터를 모두 담아야 하지만 장황할 필ㅇ는 없다.
- 예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안된다.
- 실패를 적절히 포착하려면 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 괜찮다.
  ```java
  /**
  * IndexOutOfBoundsException을 생성한다.
  * 
  * @param lowerBound 인덱스 최솟값
  * @param upperBound 인덱스 최댓값 + 1
  * @param index 인덱스 실젯값
  */
  pubilc IndexOfBoundsException(int lowerBound, int upperBound, int index) {
    // 실패를 포착하는 상세 메시지를 생성한다.
    super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
  
    // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
  }
  ```
- 포착한 실패 정보는 예외 상황을 복구하는 데 유용할 수 있으므로 접근자 메서드는 비검사 예외보다는 검사 예외에서 더 빛을 발한다.
- 비검사 예외라도 상세 정보를 알려주는 접근자 메서드를 제공하는 것을 권한다.

## 아이템 76. 가능한 한 실패 원자적으로 만들라
### 실패 원자적
- 호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 하는 특성
### 메서드를 실패 원자적으로 만드는 방법
- 불변 객체로 설계하는 것
   - 불변 객체는 태생적으로 실패 원자적
   - 생성 시점에 고정되어 절대 변하지 않음
- 가변 객체 메서드인 경우 작업 수행에 앞서 매개변수의 유효성을 검사하는 것
   - 객체의 내부 상태를 변경하기 전에 잠재적 예외의 가능성을 대부분 걸러낼 수 있음
     ```java
     public Object pop() {
         if (size = 0)
             throw new EmptyStackException();
         Object result = elements[--size];
         elements[size] = null;
         return result;
     }
     ```
   - 실패할 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드보다 앞에 배치하여 구현할 수도 있음
- 객체의 임시 복사본에서 작업을 수행한 다음, 작업이 성공적으로 완료되면 원래 객체와 교체하는 것
   - 데이터를 임시 자료구조에 저장해 작업하는 게 더 빠를 때 적용하기 좋은 방식
- 작업 도중 발생하는 실패를 가로채는 복구 코드를 작성하여 작업 전 상태로 되돌리는 것
  - 주로 내구성을 보장하는 자료구조에서 쓰이는데, 자주 사용하는 방법은 아님
### 반드시 실패 원자적으로 만들어야 할까?
실패 원자적으로 만들 수 있더라도 항상 그리해야 하는 것은 아니다! <br/>
실패 원자성을 달성하기 위한 비용이나 복잡도가 아주 큰 연산도 있기 때문이다.

## 아이템 77. 예외를 무시하지 말라
- API 설계자가 메서드 선언에 예외를 명시하는 까닭은, 그 메서드를 사용할 때 적절한 조치를 취해달라고 말하는 것
- try-catch 문으로 예외를 처리할 때, catch 블록을 비워두면 예외가 존재할 이유가 없어진다.
### 예외를 무시해야 할 때는 언제일까?
- FileInputStream을 닫을 때 예외를 무시해야 한다.
  - 파일 상태를 변경하지 않았으니 복구할 것이 없음
  - 필요한 정보를 다 읽었다는 뜻이므로 남은 작업을 중단할 이유도 없음
  - 혹시나 같은 예외가 자주 발생한다면 조사해보는 것이 좋으니 로그로 남기는 것도 좋다.
- 예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓자
```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // 기본값. 어떤 지ㅗ라도 이 값이면 충분하다.
try {
        numColors=f.get(1L,TimeUnit.SECONDS);
} catch (TimeoutException | ExcutionException ignored) {
    // 기본 값을 사용한다 (색상 수를 최소화하면 좋지만, 필수는 아님)
}
```

