## [Item 20] 추상 클래스보다는 인터페이스를 우선하라

> **자바의 다중 구현 메커니즘**
- 인터페이스, 추상클래스
- 자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있게 되었다.
> 

### 추상클래스와 인터페이스의 공통점, 차이점

- 공통점
    - 메서드의 시그니쳐만 만들고 구현을 구현 클래스에 맡긴다.
- 차이점
    - `추상클래스`: 단일 상속만 가능하다. 구현체는 추상클래스의 하위 클래스가 된다.
    - `인터페이스`: 다중 상속이 가능하다. 인터페이스를 구현했다면, 같은 타입으로 취급된다.

### 인터페이스의 장점

1. **믹스인**
    1. `믹스인`: 클래스가 구현할 수 있는 타입
    2. 믹스인을 구현한 클래스에 원래의 ‘주된 타입’ 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
    3. 예를 들어 Comparable은 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스이다.
    4. 추상 클래스 중 이미 다른 클래스를 상속하는 클래스의 경우, 해당 클래스가 두 부모 클래스를 가질 수는 없으므로 믹스인으로 사용될 수 없다.
2. **계층구조가 없는 타입 프레임워크**
    
    타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만 현실에서는 계층을 엄격히 구분하기 어려운 개념이다.
    
    ```java
    public class Item20Test {
        interface Singer {
            void Sing();
        }
    
        interface Songwriter {
            void compose(int chartPosition);
        }
    
        interface SingerSongWriter extends Singer, Songwriter {
            void strum();
        }
    }
    ```
    
    1. `SingerSongWriter` 인터페이스처럼 인터페이스는 다른 안터페이스를 상속할 수 있다.
    2. 인터페이스의 상속은 상속이라는 단어는 사용하지만, 클래스의 상속처럼 부모, 자식 계층이 존재하지 않는다.
    3. 클래스로 이와 같은 구조를 구현하려면, 상하 관계를 따져보며 차례로 다닐 상속을 받아야 한다. 
3. **래퍼(Wrapper) 클래스**
    1. `래퍼 클래스`: 기존에 인터페이스를 구현한 클래스를 주입받아 기존 구현체에 부가기능을 손쉽게 더할 수 있는 클래스
    2. 래퍼 클래스의 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
4. **디폴트 메서드(default method)**
    1. 인터페이스의 메서드 중 구현 방법이 명확한 메서드가 있다면, 디폴트 메서드를 활용할 수 있다.
    2. 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 `@implSpec` 자바독 태그를 뭍여 문서화해야 한다.
    3. 디폴트 메서드의 제약
        1. 많은 인터페이스가 `equals`와 `hashCode`같은 Object의 메서드를 정의하고 있지만, 이들은 디폴트 메서드로 제공해서는 안된다.
        2. 또한 인터페이스는 인스턴스 필드를 가질 수 없고, public이 아닌 정적 멤버도 가질 수 없다.(단, private 정적 메서드는 예외이다.)
5. **골격 구현(skeletal implementation)**
    1. 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다.
        1. 인터페이스로는 타입을 정의하고 필요하면 디폴트 메서드 몇 개도 함께 제공한다.
        2. 골격 구현 클래스는 나머지 메서들까지 구현한다.
        3. 이렇게 하면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다.
        4. 이를 바로 **템플릿 메서드 패턴**이라고 한다.
        5. 관례상 인터페이스 이름이 Interface라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다.
    2. 골격 구현을 사용해 완성한 구체 클래스
        
        ```java
        public class AbstractSkeletalConcreteClass {
            static List<Integer> intArrayAsList(int[] a) {
                Objects.requireNonNull(a);
        
                // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
                // 더 낮은 버전을 사용한다면 <Integer>로 추정하자.
                return new AbstractList<>() {
                    @Override
                    public Integer get(int i) {
                        return a[i]; // 오토박싱
                    }
        
                    @Override
                    public Integer set(int i, Integer val) {
                        int oldVal= a[i];
                        a[i] = val; //오토 언박싱
                        return oldVal; // 오토 박싱
                    }
        
                    @Override
                    public int size() {
                        return a.length;
                    }
                };
            }
        }
        
        ```
        
        - `int`배열을 받아 `Integer` 인스턴스의 리스트 형태로 보여주는 어댑터이다.
        - 박싱과 언박싱 덕에 성능은 그리 좋지 않다.
        - 익명 클래스를 사용하였다.
        - 골격 구현의 클래스는 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에는 자유롭다. 인터페이스 디폴트 메서드가 갖는 한계를 추상클래스를 이용하여 벗어난다.
        - 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하면 된다.
        - 래퍼 클래스와 비슷한 이 방식을 **시뮬레이트한 다중 상속(simulated multiple inheritance)**라 하며, 다중 상속의 많은 장점을 제공하면서 단점은 피하게 해준다.
6. **단순 구현(simple implementation)**
    1. 단순 구현은 골격 구현의 작은 변종이다.
    2. 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만 추상클래스가 아니란 점이 다르다.
    3. 단순구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.
    4. 쉽게 말해 동작하는 가장 단순한 구현이다.
    
    ```java
    핵심 정리
    * 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
    * 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 활용해보자
    * 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 
    모든 곳에서 활용하도록 하자
    *  사실 인터페이스의 구현상 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더
    흔하다.
    
    ```
    

---

## [Item 21] 인터페이스는 구현하는 쪽을 생각해 설계하라

---

## 디폴트 메서드

- 자바 8 이전에는 인터페이스는 한번 정의되면 절대 새로운 메서드가 추가되거나 기존 메서드가 사라지지 않는다는 저네 하에 코드를 작성했다.
- 자바 8 이후부터 디폴트 메서드가 등장하며 새로운 메서드를 추가할 수 있게 되었다.
    - 주로 람다를 활용하기 위해서이다.
    - 하지만 위험이 사라진 건 아니다.
    - 자바 라이브러리의 디폴트 메스드는 코드 품질이 높고 범용적이라 대부분의 상황에서는 잘 작동하지만, **생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어렵다.**

### 자바 8의ㅣ `collection` 인터페이스에 추가된 디폴트 메서드

```java
default boolean removeIf(Predicate<? super E> filter) {
  Objects.requireNonNull(filter);
  boolean result = false;
  for(Iterator<E> it = iterator(); it.hasNext();) {
    if(filter.test(it.next())) {
      it.remove();
      result = true;
    }
  }

  return result;
}
```

- 자바 8의 `Collection` 인터페이스에 추가된 `removeIf` 메서드로, 주어진 boolean 함수가 true를 반환하는 모든 원소를 제거한다.
- 범용적으로 잘 구현되어 일반적으로 재정의가 필요 없지만
- 동기화가 필요한 `SynchronizedCollection` 클래스의 경우 이를 재정의해야한다.

### 디폴트 메서드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다.

- 디폴트 메서드가 기존 구현체와 충돌할 수 있다.
    - 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피애햐 한다.
- **새로운 인터페이스를 설계할 때 여전히 세심한 주의를 기울여햐 한다.**
- **인터페이스를 릴리스한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안 된다.**

---

## [Item 22] 인터페이스는 타입을 정의하는 용도로만 사용하라

---

### 인터페이스

- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
- 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 이야기해주는 것이다.

### 인터페이스의 잘못된 예: 안티패턴

1. 상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예이다.
2. 상수 인터페이스란 메서드 없이 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스이다.
3. 이 상수들을 사용하려는 클래스에서는 정규화된 이름(qualified name)을 쓰는 걸 피하고자 그 인터페이스를 구현하곤 한다.

```java
public interface PhysicalConstants {
    // 인터페이스 내부의 필드는 `static final`이 자동으로 붙는다.
    // 아보가드로 수 (1/몰)
    double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수 (J/K)
    double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량 (kg)
    double ELECTRON_MASS = 9.109_383_56e-31;
}

@Test
public void test() {
    System.out.println(PhysicalConstants.AVOGADROS_NUMBER);
}
```

- 상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예이다.
- 상수 인터페이스를 구현하는 것은 내부 구현 클래스의 API로 노출하는 행위이다.
- 클래스가 어떤 상수 인터페이스를 사용하든 사용자에게는 아무런 의미가 없다.
- 사용자에게 혼란을 주며, 클라이언트 코드가 이 상수들에게 종속되게 한다.
    - 만약 다음 릴리스에서 이 상수들을 사용하지 않게 되더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현하고 있어야 한다.

### 상수 유틸리티 클래스

```java
public class PhysicalConstantsClass {
    private PhysicalConstantsClass() { // 인스턴스화 방지
    }

    // 아보가드로 수 (1/몰)
    public static final double AVOGADRO_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

- 상수를 공개할 합당한 방법 이 여러 개 있다.
- 열거 타입으로 나타내기 적합한 상수라면 열거타입으로 만들어 공개하면된다.
- 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개하는 방법도 있다.

### 정적 임포트 사용

```java
import static chapter4.item22.PhysicalConstantsClass.AVOGADRO_NUMBER;

public class Test {
    double atoms(double mols) {
        return AVOGADRO_NUMBER * mols;
    }
    // ... PhysicalConstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다.
}
```

- 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트 (static import)하여 클래스 이름은 생략할 수 있다.

```java
핵심 정리
* 인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자
```

---

## [Item 23] 태그 달린 클래스보다는 클래스 계층구조를 활용하라

---

### 태그 달린 클래스

태그 달린 클래스는 두 가지 이상의 의미를 표현할 수 있으며, 그 중 현재 표현하는 의미를 태그 값으로 알려주는 클래스가 있다.

```java
public class FigureWithTag {
    enum Shape {RECTANGLE, CIRCLE}

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    public FigureWithTag(double radius) {
        this.shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    public FigureWithTag(double length, double width) {
        this.shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

### 태그 달린 클래스의 단점

- 태그 달린 클래스에는 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합되어 있어서 가독성도 나쁘다
- 다른 의미를 위한 코드도 언제나 함께 하므로 메모리도 많이 사욯한다.
- 필드들을 `final`로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야한다. 즉, 불필요한 코드가 늘어난다.
- 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화 하는 데 컴파일러가 도와줄 수 있는 게 별로 없다. 엉뚱한 필드를 초기화해도 런타임에서 문제가 드러날 뿐이다.
- 또 다른 의미를 추가하려면 코드를 수정해야한다.
- 인스턴스의 타입만으로는 현재 나타내는 의미를 알 방법이 없다.

> **태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.
태그 달린 클래스는 계층구조를 어설프게 흉내낸 아류일 뿐이다.**
> 

### 태그 달린 클래스를 계층 구조로 변환

```java
public abstract class Figure {
    abstract double area();
}

public class Circle extends Figure {
    final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

public class Rectangle extends Figure {
    final double length;
    final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

### 클래스 계층 구조의 장점

- 쓸데없는 코드가 모두 사라진다.
- 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거할 수 있으며, 살아남은 필드들은 모두 final이다.
- 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.
- 실수로 빼먹은 case ㅁ누 때문에 런타임 오류가 발생할 일이 없다.
- 루트 클래스의 코드를 건드리지 않고도 독립적으로 계층구조를 확장하고 사용할 수 있다.
- 타입이 의미별로 따로 존재하기 때문에 변수의 의미를 명시하거나 제한할 수 있고,  특정 의미만 매개변수로 받을 수 있다.
- 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다.

```java
핵심 정리
* 새 클래스를 작성하는데 태그 필드가 등장하면, 태그를 없애고 계층구조로 대체하는 방법을
생각해보자
* 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는걸 고민해보자
```

---

## [Item 24] 멤버 클래스는 되도록 static으로 만들라

---

### 중첩 클래스란?

- 중첩 클래스(nested class)란, 다른 클래스 안에 정의된 클래스를 말한다.
- 오직 자신을 감싼 바깥 클래스에서만 쓰일 용도로 만들어진 클래스이다.
    - 그 외의 용도로 쓰일 클래스는 톱레벨 클래스로 만들어야 한다.
- 중첩 클래스의 종류는 4가지가 있다. 정적 멤버 클래스를 제외한 나머지는 내부 클래스(inner class)에 해당한다.
    - 정적 멤버 클래스
    - 비정적 멤버 클래스
    - 익명 클래스
    - 지역 클래스

### 정적 멤버 클래스

- 정적 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다.
- 정적 멤버 클래스는 보통 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 사용된다.

### 비정적 멤버 클래스

- 정적 멤버 클래스와의 구문상 차이는 `static` 유무이다.
- 하지만 의미상 차이는 바깥 클래스의 인스턴스와 연결되는지 아닌지 여부이다.
    - 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
    - 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.
- 비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다.

```java
public class MySet<E> extends AbstractSet<E> {
    // ...

    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        // ...
    }
}
```

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 `static`을 붙여서 정적 멤버 클래스로 만들자.**

- `static` 을 생략하면 바깥 인스턴스로의 숨은 외부참조를 갖게 되기 때문에 시간과 공간이 소비되고, 메모리 누수가 생길 수도 있다.

### 익명 클래스

- 익명 클래스에는 이름이 없다. 그리고 바깥 클래스의 멤버도 아니다.
- 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어지고, 코드의 어디서든 만들 수 있다.
- 또, 오직 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 정적 문맥에서라도 상수 변수 이외의 정적 멤버는 가질 수 없다.
- 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 이외에는 호출할 수 없다.
- 익명 클래스는 로직의 중간에 등장하므로, 짧지 않으면 가독성이 떨어진다.
- 익명 클래스의 또 다른 주 쓰임은 정적 팩터리 메서드를 구현할 때다.

### 지역 클래스

- 네 가지 중첩 클래스 중 가장 드물게 사용된다.
- 지역변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있고, 유효범위도 지역변수와 같다.
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버를 가질 수 없고, 가독성을 위해 짧게 작성해야 한다.

```java
핵심 정리
- 중첩 클래스에는 네가지 종류가 있고 각각의 쓰임이 다르다.
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로 만들자.
- 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조하지 않는다면 정적(static)으로 만들자.
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고, 
해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들자.
- 그렇지 않고 재사용해야 하면 지역 클래스로 만들자.
```

---

## [Item 25] 톱레벨 클래스는 한 파일에 하나만 담으라

---

> - 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다.
- 따라서 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그 중 어는 것을 사용할지는 어는 소스파일을 먼저 컴파일하냐에 따라 달라진다.
> 

### 두 클래스가 한 파일에 정의되었다. - 따라하지 말 것!

---

> `Utensil.java`
> 

```java
class Utensil {
  static final String NAME = "pan";
}

class Dessert {
  static final String NAME = "cake";
}
```

> `Dessert.java`
> 

```java
class Utensil {
  static final String NAME = "pot";
}

class Dessert {
  static final String NAME = "pie";
}
```

> `Main.java`
> 

```java
public class Main {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Desert.NAME);
  }
}
```

- 위는 `Utensil.java`와 `Dessert.java`라는 두 개의 파일에 중복으로 정의된 2개의 클래스의 예이다.
- 위와 같은 경우 `javac` 명령어에 들어가는 인수에 따라 실행결과가 달라진다.
    - `javac Main.java Desser.java` : 에러, `Utensil` 과 `Dessert` 클래스가 중복 정의되었습니다.
    - `javac Main.java, javac Main.java Utensil.java` : `"pancake"` 출력
    - `javac Dessert.java Main.java` : `"potpie"` 출력
    - **동작 원리**
        - `Main.java` 가 먼저 인수에 들어왔을 때, 자바는 `Main.java`를 실행시키며, `Utensil.NAME` 을 만나고, `Utensil.java` 파일을 찾아서 클래스를 로드하려한다.
            - 그래서 `javac Main.java`의 경우 `"pancake'` 가 출력된다.
            - 그래서 `javac Main.java Dessert.java` 의 경우 클래스가 중복으로 선언되었다고 알린다.
        - `Dessert.java` 가 먼저 인수에 들어왔을 때는, 자바는 `Utensil` 클래스와 `Dessert` 클래스의 정의를 불러와 놓는다.
            - 그래서 `javac Dessert.java Main.java`의 경우 `"potpie"` 가 출력된다.

**파일을 나누면 위와 같은 복잡한 동작원리도 알 필요 없고, 잠재적 에러도 없으므로 `Utensil.java`와 `Dessert.java` 각 파일별로 클래스는 1개씩만 선언하는 것이 좋다.**

### 굳이 한 파일에 여러 클래스를 정의하고 싶다면?

```java
public class Test {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }

  private static class Utensil {
    final String NAME = "pan";
  }

  private static class Dessert {
    final String NAME = "cake";
  }
}
```

```java
핵심 정리
- 소스파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자.
```
