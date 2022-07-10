# 6장. 열거 타입과 애너테이션

> 자바의 특수한 목적의 참조 타입<br>
\- **열거 타입(enum**; 열거형) : 클래스의 일종, **애너테이션(annotation)** : 인터페이스의 일종
>

<aside>
✏️ 열거 타입과 애너테이션을 올바르게 사용하는 방법

</aside>

<br><br>

## 아이템 38.확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

> 열거 타입은 확장할 수 없으며, 대부분의 상황에서 열거 타입의 확장은 좋지 않은 생각이다. <br/>
> 그런데 확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다. 바로 **연산 코드** 이다.

### 인터페이스 구현을 통한 열거 타입 확장
연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 한다. <br/>
이 때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다. <br/>
코드를 보면 더 쉽게 이해할 수 있을 것이다.

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {public double apply (double x, double y) {return x + y;} },
    MINUS("-") {public double apply (double x, double y) {return x - y;} },
    TIMES("*") {public double apply (double x, double y) {return x * y;} },
    DIVIDE("/") {public double apply (double x, double y) {return x / y;} };
    
    private final String symbol;
    BasicOperation(String symbol) {this.symbol = symbol;}
    @Override public String toString() {return symbol;}
}
```
열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다. <br/>
연산 타입을 확장해 지수 연산과 나머지 연산을 추가해보자.
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {public double apply (double x, double y) {return Math.pow(x, y);} },
    REMAINDER("&") {public double apply (double x, double y) {return x % y;} };

    private final String symbol;
    ExtendedOperation(String symbol) {this.symbol = symbol;}
    @Override public String toString() {return symbol;}
}
```
BasicOperation이 아닌 Operation 인터페이스를 사용하도록 작성되었다면, 새로 작성한 연산은 기존 연산이 쓰이던 곳 어디든 쓸 수 있다.

## 아이템 39. 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴 단점
1. 오타가 나면 안된다.
2. 올바른 프로그램 요소에서만 사용된다고 보증할 방법이 없다.
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.

### 애너테이션
> 명명 패턴에서 발생하는 단점을 해결해준다. <br/>
> 간단한 테스트 프레임워크를 사용해 애너테이션 동작 방식을 알아보자.

#### 마커(marker) 애너테이션 타입 선언
- 자동으로 수행되는 간단한 테스트용 애너테이션으로, 예외가 발생하면 해당 테스트를 실패로 처리한다.
- "아무 매개변수 없이 단순히 대상에 마킹(marking)한다"는 뜻에서 `마커(marker)애너테이션`이라 한다.
```java
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
- `@Rentention(RetentionPolicy.RUNTIME)` 메타애너에티션 : `@Test`가 런타임에도 유지되어야 한다는 표시
- `@Target(ElementType.METHOD)` 메타애너테이션 : `@Test`가 반드시 메서드 선언에서만 사용돼야 한다는 표시

#### 마커 애너테이션을 사용한 프로그램 예
```java
public class Sample {
    @Test public static void m1() {}         // 성공
    public static void m2() {}               // 동작 X
    @Test public static void m3() {          // 실패
        throw new RuntimeException("실패");
    }
    public static void m4() {}               // 동작 X
    @Test public void m5() {}                // 잘못 사용 (정적 메서드가 아님)
    public static void m6() {}               // 동작 X
    @Test public static void m7() {          // 실패
        throw new RuntimeException("실패");
    }
    public static void m8() {}                // 동작 X
}
```
- `@Test`가 붙어있지 않으면 테스트 도구가 무시하여 동작하지 않는다.

#### 마커 애너테이션을 처리하는 프로그램
```java
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclareMethods()) {
            if (m.isAnnotationPresent(Test.Class)) { // 실행할 메서드를 찾아주는 메서드
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) { // @Test 애너테이션을 제대로 사용한 경우 발생하는 예외
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패 : " + exc);
                } catch (Exception exc) { // @Test 애너테이션을 잘못 사용한 경우 발생하는 예외
                    System.out.println("잘못 사용한 @Test : " + m);
                }
            }
        }
        System.out.printf(" 성공 : %d, 실패 : %d%n", passed, tests - passed);
    }
}
```

#### 매개변수 하나를 받는 애너테이션 타입
특정 예외를 던져야만 성공하는 테스트를 만들어보려 한다. 이를 위해서는 새로운 애너테이션 타입이 필요하다.
```java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```
애너테이션을 실제로 활용하는 모습이다.
```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() { // 성공해야 한다.
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() {} // 실패해야 한다. (예외가 발생하지 않음)
}
```
이제 이 애너테이션을 다룰 수 있도록 테스트 도구를 수정해보자. <br/>
위에서 작성했던 마커 애너테이션을 처리하는 프로그램을 통해 살펴보려 한다.
```java
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclareMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.Class)) { // 실행할 메서드를 찾아주는 메서드
                tests++;
                try { // 예외 발생 X
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패 : 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedExc) { // @Test 애너테이션을 제대로 사용한 경우 발생하는 예외
                    Throwable exc = wrappedExc.getCause();
                    Class<? extends Throwable> excType = m.getAnnotion(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) { // 발생하길 기대했던 특정 애너테이션인 경우
                        passed++;
                    } else {
                        System.out.println("테스트 %s 실패 : 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
                    }
                } catch (Exception exc) { // @Test 애너테이션을 잘못 사용한 경우 발생하는 예외
                    System.out.println("잘못 사용한 @ExceptionTest : " + m);
                }
            }
        }
        System.out.printf(" 성공 : %d, 실패 : %d%n", passed, tests - passed);
    }
}
```
하나의 예외가 아닌 여러 개를 명시하고 그 중 하나가 발생하면 성공하게 만들 수도 있다.<br/>
너무 부가적인 설명 같아서 여기서는 정리하지 않았지만, 궁금하다면 책 242p를 읽어보면 된다.

## 아이템 40. @Override 애너테이션을 일관되게 사용하라
- `@Override`는 메서드 선언에만 달 수 있으며, 상위 타입의 메서드를 재정의했음을 뜻한다.
- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 `필수`로 달아야 한다.
- 필수로 달지 않는 예외는 단 한 가지이다. → 구체 클래스에서 상위 클래스의 추상 메서드를 재정의하는 경우
  - 해당 경우에 @Override 애너테이션을 단다고 해서 문제가 되는 것은 아니다.
  - 오히려, IDE에서는 @Override를 일관되게 사용하도록 부추기기도 한다.
- 클래스뿐만 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다.
```
상위 클래스나 인터페이스를 재정의하는 경우, @Override를 다는 습관을 들이자!
```

## 아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라
### 마커 인터페이스 (Marker Interface)
아무 메서드도 담고 있지 않고, 단지 자신을 구현한 클래스가 특정 속성을 가짐을 표시해주는 인터페이스
### 마커 인터페이스 vs. 마커 애너테이션
- 마커 인터페이스는 이를 구현한 클래스 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다.
  - 런타임 마커 인터페이스는 컴파일 시 오류를 검출하지만, 마커 애너테이션은 런타임 시 오류를 검출한다.
- 마커 인터페이스가 마커 애너테이션보다 더 정밀하게 적용 대상을 지정할 수 있다.
  - 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있다면 마커 인터페이스를 사용하자.
- 마커 인터페이스와 달리 마커 애너테이션은 거대한 애너테이션 시스템의 지원을 받는다는 장점이 있다.
  - 애너테이션을 적극 확용하는 프레임워크라면 마커 애너테이션을 사용하는 편이 더 좋다.