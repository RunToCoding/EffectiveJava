# **Chapter04**

# 4장 클래스와 인터페이스

## [Item 15] 클래스와 멤버의 접근 권한을 최소화하라 - page 96

---

> **정보은닉(캡슐화)**
잘 설계된 컴포넌트는 클래스 내부 데이터를 **외부 컴포넌트로부터 얼마나 잘 숨겼는지**에 따라 달라진다. 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리하고, 이 API를 통해서만 다른컴포넌트와 소통하며 서로의 내부 동작 방식에 전혀 개의치 않는다.
> 

### **정보은닉(캡슐화)의 장점**

1. 시스템 개발 속도를 높인다.
2. 시스템 관리 비용을 낮춘다.
3. 성능 최적화에 도움을 준다.
4. 소프트웨어 재사용성을 높인다.
5. 큰 시스템을 제작하는 난이도를 낮춰준다.

### 정보 은닉의 다양한 장치

- 접근 제어 매커니즘: 클래스, 인터페이스, 멤버의 접근성
- 각 요소의 접근성: 요소가 선언된 위치와 접근 제한자(priavate, protected, public)으로 정해진다.

### 정보은닉의 **기본 원칙**

**모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.** 다시 말하면 항상 ****가장 낮은 접근 수준을 부여해야한다.

- 톱레벨(가장 바깥) 클래스나 인터페이스를 `public`으로 선언하면 공개 API가 되며 하위호환을 위해 영원히 관리해줘야 한다.
- `package-private` 으로 선언하면 해당 패키지 안에서만 사용 가능하다.
    
    > 패키지 외부에서 쓸 이유가 없다면 `package-private`으로 선언하자. 그러면 이들은 API가 아닌 내부 구현이 되어 언제든 수정할 수 있다.
    > 

### 멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준

- `**private**`: 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
- `**package-private`:** 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. 접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준이다. (단, 인터페이스의 멤버는 기본적으로 public이 적용된다.)
- `**protected**`: `package-private`의 접근범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
- `**public**`: 모든 곳에서 접근할 수 있다.

### 클래스의 접근범위 설정

1. 클래스의 공개 API를 세심히 설계한 후 그 외의 모든 멤버는 `private`으로 만든다.
2. 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한하여(`private` 제한자를 제거하여) `package-private`으로 풀어준다.
    
    > 권한을 풀어주는 일을 자주 하게 된다면 시스템에서 컴포넌트를 더 분해해야 하는 것은 아닌지 다시 고민해보자.
    > 
    
    > `private`과 `package-private` 멤버는 모두 해당 클래스의 구현에 해당하므로 보통은 API에 영향을 주지 않는다.
    > 
    

### Public 클래스와 protected

- `public` 클래스에서 멤버의 접근 수준을 `package-private`에서 `protected`로 바꾸는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청나게 넓어진다.
- `public` 클래스의 `protected` 멤버는 공개 API이므로 영원히 지원돼야 한다.
- 내부 동작 방식을 API 문서에 적어 사용자에게 공개해야 할 수도 있다.(Item 19) 따라서 `protected`의 수는 적을수록 좋다.

### 멤버 접근성의 제약

- 상위 클래스의 메서드를 재정의할 때 접근 수준을 상위 클래스보다 좁게 설정할 수 없다.
    
    > 리스코프 치환 원칙: 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야한다.
    > 
- 이 규칙을 어기면 하위 클래스를 컴파일 할 때 컴파일 오류가 난다.
- 이때 클래스는 인터페이스가 정의한 모든 메서드를 public으로 선언해야한다.

### 테스트의 접근범위

- 코드 테스트 목적으로 클래스, 인터페이스, 멤버의 접근 범위를 넓히려고 할 때 적당 수준까지는 넓혀도 괜찮다.
    
    > public 클래스의 private 멤버를 package-private까지 풀어주는 것은 허용
    > 
- 하지만 테스트만을 위해 클래스, 인터페이스, 멤버를 공개 API로 만들어서는 안된다.

### `public` 클래스와 인스턴스 필드

- **`public` 클래스의 인스턴스 필드는 되도록 `public`이 아니어야 한다.(Item 16)**
- 필드가 가변 객체를 참조하거나, `final`이 아닌 인스턴스 필드를 `public`으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃게 된다.
- 그 필드와 관련된 모든 것은 불변식을 보장할 수 없게 된다.
- 필드가 수정될 때, 다른 작업을 할 수 없게 되므로 **`public` 가변 필드를 갖는 클래슨느 일반적으로 스레드 안전하지 않다.**
- 이러한 문제는 정적 필드에서도 마찬가지지만 예외가 있다. 해당 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 `public static final` 필드로 공개해도 좋다.
    - 이런 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 한다.(Item 17)
    - 가변 객체를 참조하면 `final`이 아닌 필드에 적용되는 모든 불이익이 그대로 적용된다.
    - 다른 객체를 참조하지는 못하지만, 참조된 객체 자체는 수정될 수 있으니 주의해야한다.

### 배열

- 길이가 0이 아닌 배열은 모두 변경 가능하니 주의해야한다. 따라서 **클래스에서 `public static final` 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다.**
- 이런 필드나 접근자를 제공한다면 클라이언트에서 그 배열의 내용을 수정할 수 있게 된다. 어떤 IDE가 생성하는 접근자는 private 배열 필드의 참조를 반환하여 이 같은 문제를 똑같이 일으키니 주의하자.
    
    ```java
    public static final Thing[] VALUES = { ... };
    ```
    

이에 대한 해결책은 두가지다.

- 앞 코드의 public 배열을 private으로 만들고 public 불변 리스트를 추가하는 것이다.
    
    ```java
    private static final Thing[] PRIVATE_VALUES = { ... };
    public static final List<Thing> VALUES = 
    	Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
    ```
    
- 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 추가하는 방법이다.(방어적 복사)
    
    ```java
    private static final Thing[] PRIVATE_VALUES = { ... };
    public static final Thing[] values() {
    	return PRIVATE_VALUES.clone();
    }
    ```
     

### 자바 9에 추가된 모듈 시스템과 접근 수준

- 모듈 시스템이라는 개념이 도입되면서 두 가지 암묵적 접근 수준이 추가되었다.
- 패키지가 클래스들의 묶음이듯, 모듈은 패키지들의 묶음이다. 모듈은 자신에 속하는 패키지 중 공개(export)할 것들을(관례상 module-info.java 파일에) 선언한다.
- protected 또는 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에는 접근할 수 없다. 물론 모듈 안에서는 export 선언에 대한 영향을 받지 않는다.
- 모듈 시스템을 활용하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다.
- 두 번째 접근 수준은 모듈의 JAR 파일을 자신의 모듈 경로가 아닌 애플리케이션의 클래스패스에 두면 모듈 안의 모든 패키지는 마치 모듈이 없는 것처럼 행동한다.
- 모듈이 공개했는지 여부와 상관없이, public 클래스가 선언한 모든 public 혹은 protected 멤버를 모듈 밖에서도 접근할 수 있게 된다.

```java
**핵심 정리**
프로그램 요소의 접근성은 가능한 한 최소한으로 하라. 
꼭 필요한 것만 골라 최소한의 public API를 설계하자.
그 외에는 클랫, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야한다.
public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다.
public static final 필드가 참조하는 객체가 불변인지 확인하라.
```

$\large\color{black}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━$

## [Item 16] public 클래스에서는 public 필드가 아닌 접근자 매서드를 사용하라

---

- 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있다.
    
    ```java
    class Point {
        public double x;
        public double y;
    }
    ```
    

### 캡슐화의 이점

- API를 수정하지 않고 재부 표현을 바꿀 수 있다.
- 불변식을 보장할 수 있다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수 있다.
- **따라서 필드들을 모두 private으로 바꾸고 public 접근자(getter)를 추가한다.**
    
    ```java
    class Point {
        private double x; //public을 private로 바꿈
        private double y;
    
        public Point(double x, double y) {
            this.x = x;
            this.y = y;
        }
    
        public double getX() { return x; } //public 접근자를 추가
        public double getY() { return y; }
    
        public void setX(double x) { this.x = x; }
        public void setY(double y) { this.y = y; }
    }
    ```
    

### 접근자와 변경자의 활용

- **class가 public일 때: 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공**함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
    - `public` 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 된다.
- **`package-private` 클래스 혹은 `private` 중첩 클래스일 때: 데이터 필드를 노출한다 해도 문제가 없다.** 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.
    - 이 경우 클라이언트가 이 클래스를 포함하는 패키지 안에서만 동작한다는 보장이 있기 때문에 `public`으로 공개해도 이 클래스를 사용하는 다른 클래스가 없기 때문에 상관이 없다.

### 접근자와 변경자를 활용하지 않을 때의 단점

- API를 변경하지 않고는 표현 방식을 바꿀 수 없다.
- 필드를 읽을 때 부수 작업을 수행할 수 없다.
- 단, 불변식은 보장할 수 있게 된다.

```markdown
**핵심 정리**
* public class는 절대 가변 필드를 직접 노출해서는 안된다.
* 불변 필드라면 노출해도 덜 위험하지만 안심할 수 는 없다.
* 종종 package-private 클래스나 private 중첩 틀래스에서는 
  필드를 노출하는 편이 나을 때도 있다.

```

$\large\color{black}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━$

## [Item 17] 변경 가능성을 최소화하라

---

### 불변 클래스

> 불변 클래스란?
인스턴스 내부 값을 수정할 수 없는 클래스이다. 불변 인스턴스에 간직된 정보는 고정되어서 객체가 파괴되는 순간까지 절대 달라지지 않는다.
`String`, `BigInteger`, `BigDecimal`이 여기에 속한다.
> 
- 불변 클래스는가변 클래스보다 설계하고 구현하고 사용하기가 쉽다.
- 오류가 생길 여지도 적고 훨씬 안전하다.

### 불변 클래스의 다섯가지 규칙

1. **객체의 상태를 변경하는 메서드를 제공하지 않는다.**
2. **클래스를 확장할 수 없도록 한다.**
    1. 상속이 불가능하게 만든다.
3. **모든 필드를 final로 선언한다.**
4. **모든 필드를 private로 설언한다.**
    1. 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막는다.
    2. `public final`은 다음 릴리스에서 내부 표현을 바꾸지 못하므로 주의해서 사용해야한다. 
5. **자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도로 한다.**
    1. 만약 내부에 가변 참조 필드가 있다면 클라이언트에서 이 필드를 절대 참조할 수 없도록 해야한다. 접근자 매서드로도 접근할 수 없도록 하여 불변성을 지켜야 한다.
    2. 아예 접근조차 하지 못하게 하여 수정할 여지를 주지 않도록 한다.

### 불변 클래스의 예시

- 불변 복소수 클래스(예제)
    
    ```java
    public **final** class Complex {
        **private fina**l double re;
        **private final** double im;
    
        public Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }
    
        public double realPart() {
            return re;
        }
    
        public double imaginaryPart() {
            return im;
        }
    		
    		//add 대신 plus와 같은 전치사 이름 사용
        public Complex plus(Complex c) {
            return new Complex(re + c.re, im + c.im);
        }
    
        public Complex minus(Complex c) {
            return new Complex(re * c.re - im * c.im, 
    														re * c.im + im*c.re);
        }
    
        public Complex dividedBy(Complex c) {
            double tmp = c.re * c.re + c.im * c.im;
            return new Complex((re * c.re + im * c.im) / tmp, 
                                (im * c.re - re * c.im) / tmp); 
        }
    
        @Override public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof Complex))
                return false;
            Complex c = (Complex) o;
    
            // == 대신 compare를 사용하는 이유는 63 페이지를 확인하라
            return Double.compare(c.re, re) == 0 
                    && Double.compare(c.im,  im) == 0;
        }
    
        @Override public int hashCode() {
            return 31 * Double.hashCode(re) + Double.hashCode(im);
        }
    
        @Override public String toString() {
            return "(" + re + " + " + im + "i)";
        }
    
    }
    ```
    
    - 접근 메서드와 사칙연산 메서드가 있다.
    - 사칙연산 메서드는 피연산자인 클래스의 멤버를 건들지 않고, 매번 새로운 메서드를 반환한다.
        - 그래서 메서드 이름으로 `add` 같은 동사 메서드 대신 `plus` 와 같은 전치사를 사용하였다.
        - 이를 함수형 프로그래밍이라 한다.
        - 이렇게 활용하면 코드에서 불변이 되는 영역이 늘어나는 장점을 누릴 수 있다.

### 불변 객체의 특징

- **불변 객체는 단순하다**
    - 생성된 시점의 상태가 파괴될 때까지 상태를 그대로 간직한다.
- **불변 객체는 근본적으로 스레드 안전하여 따로 동기화가 필요 없다.**
    - 불변 객체는 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
- **불변 객체는 안심하게 공유될 수 있다.**
    - 불변 클래스일 경우 한번 만든 인스턴스를 재활용하여 사용하는 것을 권장한다.
    - 재활용 방법으로 자주 쓰이는 값들을 상수로 제공한다. `public static final`
    - 방어적 복사(item 50)도 필요 없다. 아무리 복사해도 원본과 같으므로 복사 생성자(item 13)를 제공하지 않는 것이 좋다.
- 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 적정 팩터리를 제공한다.
    - 정적 팩터리를 사용하면 여러 클라이언트가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어들게 된다.
- **불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**
    - 값이 변하지 않기 때문에 내부 데이터를 공유하더라도 문제가 생기지 않는다.
- **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.**
    - 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 수월하다.
- **불변 객체는 그 자체로 실패 원자성을 제공한다.**
    - 실패원자성: 베서드에서 예외가 발생한 후에도 그 객체의 값 자체에는 훼손이 없다는 것을 의미한다.

### 불변 객체의 단점

- 값이 다르면 반드시 독립된 객체로 만들어야 한다.
    - 값이 모두 다르면 값이 다를 때마다 객체를 새로 만들어야 하므로 메모리 낭비가 생긴다.
    - 값의 가지수가 많다면 이를 만드는 데 모두 큰 비용이 생긴다.

### 불변 객체의 단점 해소

1. 다단계 연산들을 예측하여 기본 기능으로 제공한다.
    1. 클라이언트가 원하는 복잡한 연산을 미리 예측하여 제공한다.

### 불변 클래스의 또 다른 설계 방법

1. 클래스 자기 자신을 상속하지 못하게 한다.
    1. FINAL 클래스로 선언한다.
    2. 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공한다.
    - 생성자 대신 정적 팩터리를 사용한 불변 클라스
        
        ```java
        public class Complex {
            private final double re;
            private final double im;
            
            private Complex(double re, double im){
            	this.re = re;
        			this.im = im;
            }
        		//정적 팩터리로 호춣
            public static Complex valueOf(double re, double im){
            	return new Complex(re, im);
            }
        }
        ```
        
        - 생성자를 private로 막아서 상속 자체를 방지할 수 있고 정적 팩터리로 호출 시, 항상 새로운 객체를 생성해주기 때문에 불변을 유지한다.
        - 정적 팩터리로 제공한다면 다수의 구현 클래스를 만들 수 있고 자주 사용하는 값에 객체 캐싱 기능을 사용하여 성능을 끌어올릴 수도 있다.
    
    > `BigInteger`의 경우 `final`이어야 한다는 아이디어가 퍼지기 전에 만들어진 것이라 `public` 생성자를 제공하는데, 하휘 호환성에 의한 제약이 걸리고 지금까지 이 문제를 고치지 못했다.
    > 
    
    > 불변 클래스의 규칙에 따라 `모든 필드가 final이고 어떤 메서드도 그 객체를 수정할 수 없어야한다`는 과한 느낌이 있어 `어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다` 와 같이 완화할 수 있다.
    > 
    
    ### 정리
    
    1. **클래스는 꼭 필요한 경우가 아니면 불변인 것이 좋다.**
        1. 단점이라곤 특정 상황에서의 잠재적 성능 저하 뿐이다.
    2. **불변으로 만들 수 없는 클래스는 변경할 수 있는 부분을 최대한 줄이는 것이 좋다.**
        1. 객체가 가질 수 있는 상태의 수를 줄이는 것이 객체를 예측하기 쉬워서 오류가 생길 가능성이 줄어든다.
        2. 꼭 변경해야 할 필드를 뺀 나머지 모두를 `final`로 선언하자.
    3. **다른 합당한 이유가 없다면 모든 필드는 `private final` 접근 제어자를 가지는 것이 좋다.**
    4. **생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.**
    
    $\large\color{black}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━$
    
    ## [Item 18] 상속보다는 컴포지션을 사용하라
    
    ---
    
    > 상속은 코드를 재사용하는 강력한 수단이지만 항상 최선은 아니다. 구체 클래스를 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 것은 위함하다.
    이때 `상속`은 다른 패키지의 구체 클래스를 확장하는 `구현 상속`을 말하며, 인터페이스의 상속과는 무관하다.
    > 
    
    ### 상속에서 일어나기 쉬운 문제 1: 자기사용
    
    - 매서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
        - 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스으 ㅣ동작에 이상이 생길 수 있다.
        - 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다.
        - 예시
            
            ```java
            public class InstrumentedHashSetUseExtends<E> extends HashSet {
                private int addCount = 0; // 추가된 원소의 수
            
                public InstrumentedHashSetUseExtends() {
                }
            
                public InstrumentedHashSetUseExtends(int initCap, float loadFactor) {
                    super(initCap, loadFactor);
                }
            
                @Override
                public boolean add(Object o) {
                    return super.add(o);
                }
            
            		//이 메서드를 사용하면 제대로 작동되지 않는다.
                @Override
                public boolean addAll(Collection c) {
                    return super.addAll(c);
                }
            
                public int getAddCount() {
                    return addCount;
                }
            
            }
            ```
            
            - 위 코드는 원소가 추가될 때마다 `addCount`의 수를 1씩 증가시키려는 코드이다.
                - 하지만 3개의 원소가 추가되었는데 `addCount`가 6이 되었다.
            - 사실 하위 클래스에서 `addAll()`을 재정의하지 않으면 일어나지 않을 문제
                - 그러나 `addAll()`은 결국 `add()`를 이용하여 구현하여 `addCount`의 값이 중복으로 더해지게 된다.
                - 이렇게 자신의 다른 부분을 사용하는 방식을 `**자기사용**(self-use)`이라고 한다.
    
    ### 상속에서 일어나기 쉬운 문제 2: 하위 클래스의 캡슐화가 깨져버리는 경우
    
    - 원소를 추가하는데 제약조건이 있는 컬렉션이 있을 때 상위 클래스에서 새롭게 원소를 추가하는 메서드가 생긴다면
        - 보안상의 구멍이 생길 수 있다.
        - 이전에 `HashTable`과 `Vector`를 컬렉션에 추가하자 심각한 보안 구멍이 생긴 예시가 있다.
    - 운 없게도 하필 하위 클래스에 추가한 매서드와 시그니처가 상위 클래스에 추가한 매서드와 같고 반환 타입만 다르다면
        - 컴파일 에러가 뜨게 된다.
        - 반환 타입마저 같다고 하면 상위 클래서의 새 메서드를 재정의한 꼴이 된다.

### 문제에 대한 해결 방안: 컴포지션

- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고, `private` 필드로 기존 클래스의 인스턴스를 참조하게 하면 된다.
    - 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 `컴포지션(composition)` 이라고 한다.
- 새 클래스의 인스턴스 메서드들은 기존 클래스의 메서드들을 `전달 메서드(forwarding method)`라고 부른다.
    - 이 방식을 전달(forwarding)이라고 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다.
    - 그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향을 받지 않는다.
    - 예시-래퍼 클래스(추가)
        
        ```java
        public class InstrumentedHashSetUseComposition<E> extends ForwardingSet<E> {
            private int addCount = 0;
        
            public InstrumentedHashSetUseComposition(Set<E> s) {
                super(s);
            }
        
            @Override
            public boolean add(E e) {
                addCount++;
                return super.add(e);
            }
        
            @Override
            public boolean addAll(Collection<? extends E> c) {
                addCount += c.size();
                return super.addAll(c);
            }
        
            public int getAddCount() {
                return addCount;
            }
        }
        ```
        
        ```java
        //재사용할 수 있는 전달 클래스
        public class ForwardingSet<E> implements Set<E> {
            private final Set<E> s;
        
            public ForwardingSet(Set<E> s) {
                this.s = s;
            }
        
            public int size() {
                return 0;
            }
        
            public boolean isEmpty() {
                return s.isEmpty();
            }
        
            public boolean contains(Object o) {
                return s.contains(o);
            }
        
            public Iterator<E> iterator() {
                return s.iterator();
            }
        
            public Object[] toArray() {
                return s.toArray();
            }
        
            public <T> T[] toArray(T[] a) {
                return s.toArray(a);
            }
        
            public boolean add(E e) {
                return s.add(e);
            }
        
            public boolean remove(Object o) {
                return s.remove(o);
            }
        
            public boolean containsAll(Collection<?> c) {
                return s.containsAll(c);
            }
        
            public boolean addAll(Collection<? extends E> c) {
                return s.addAll(c);
            }
        
            public boolean retainAll(Collection<?> c) {
                return s.retainAll(c);
            }
        
            public boolean removeAll(Collection<?> c) {
                return s.removeAll(c);
            }
        
            public void clear() {
                s.clear();
            }
        
            @Override
            public boolean equals(Object o) {
                return s.equals(o);
            }
        
            @Override
            public int hashCode() {
                return s.hashCode();
            }
        
            @Override
            public String toString() {
                return s.toString();
            }
        }
        ```
        

### 상속 주의점

1. 상속은 반드시 하위 클래스가 상위 클래스의 ‘진짜’ 하위 타입인 상황에서만 쓰여야 한다.
    1. 클래스 B가 클래서 A와 is-a 관계일 때만 클래스 A를 상속해야 한다.
    2. 맞다고 확신할 수 없다면 상속 관계를 사용하지 않는 것이 좋다.
    3. 예를 들어, 자바 기본 API의 `Stack`은 `Vector`를 상속받아서는 안됐다.
        1. 스택은 벡터가 아님(is-a 관계가 성립하지 않는다.)
    4. 또 다른 예시로 `Properties`도 `HashTable`을 상속받으면 안된다.
        1. 두 사례 모두 컴포지션을 사용한다면 더 좋다.
2. 확장하려는 클래스의 API에 아무런 결함이 없는지 확인해야한다.
    1. 상속받게 되면 상위 클래스 API의 결함까지도 그대로 상속받으므로 주의해야 한다.

```java
**핵심 정리**
* 상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
* 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야한다.
* 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용한다. 
```

$\large\color{black}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━$

## [Item 19] 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

> item 18에서 상속할 때 주의점도 문서화해놓지 않은 ‘외부’ 클래스를 상속할 때 위험을 경고했다. 여기서 ‘외부’란 프로그래머의 통제권 밖에 있어서 언제 어떻게 변경될지 모른다는 뜻이다.
**따라서 메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서화로 남겨야 한다.**
> 

### Implementation Requirements

- API 문서의 메서드 설명 끝에 종종 `implementation Requirements`로 시작하는 절을 볼 수 있는데 그 메서드의 내부 동작 방식을 설명하는 곳이다.
- 이 절은 메서드 주석에 @implSpec 태그를 붙여주면 자바독 도구가 생성해준다.

### 좋은 API 문서란 ‘어떻게’가 아닌 ‘무엇’을 하는지를 설명해야 한다.

- 하지만 특정 클래스에서 메서드를 재정의하면 내부의 다른 메서드의 동작에 주는 영향도 정확하게 설명하고 있는 API들이 있다.
- 이는 상속이 캡슐화를 해치기 때문에 클래스를 안전하게 상속할 수 있도록 하려면 내부 구현 방식을 설명해야 했다.

### 추가로 고려해야 할 상황

1. **내부 구현에서 사용되는 메서드를 잘 선별하여 protected 메서드 형태로 공개해야 할 수 있다.**
    1. protected 메서드 하나하나가 모두 내부 구현에 해당하므로 그 수는 가능한 적어야 한다. 
    2. 다만, 너무 적게 노출해서 상속으로 얻는 이점을 없애지 않도록 주의해야 한다.
2. **상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 “유일’하다.**
    1. 하위 클래스 3개 정도가 적당하다.
3. **상속용 클래서의 생성자는 직접적이든 간접적이든 재정의 가능 매서드를 호출해서는 안된다.**
    1. 이를 어기면 프로그램이 오작동 된다.
4. **`Cloneable`과 `Serializable` 인터페이스는 상속용으로 설계하는 것은 좋지 않다.**
5. `**clone`과 `readObject` 모두 직접적, 간접적으로 재정의 가능 메서드를 호출해서는 안된다.**
6. `**Serializable`을 구현한 상속용 클래스가 `readResolve`나 `writeReplace` 메서드를 갖는다면 이 메서드들은 `protected`로 선언해야 한다.**

상속 문제를 해결하는 가장 좋은 방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이다.

1. 클래스를 `final`로 선언하는 것
2. 모든 생성자를 `private`이나 `default`로 선언하고 정적 팩토리 메서드를 제공한다.
