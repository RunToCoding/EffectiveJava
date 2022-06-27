# 3장 모든 객체의 공통 method

## **[Item 10] equals는 일반 규약을 지켜 재정의하라_(page52)**

equals 메서드는 재정의 하지 않으면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

equals를 재정의하면 인스턴스는 값을 비교할 수 있고, Map의 키와 Set의 원소로 사용할 수 있게 된다.

### 1. equals를 재정의하지 않는 것이 최선일 때

- 각 인스턴스가 본질적으로 고유하다
- 인스턴스의 ‘논리적 동치성(logical equlity)’을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

```java
@Override
public boolean equals(Object o) {
	throw new AssertionError(); //호출 금지!
}
```

### 2. equals를 재정의해야 할 때

- 객체 식별성(Object identity)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않을 때
    
    : 주로 값 클래스(Integer, String)들이 해당 됨
    

### 3. eq**uals메서드**를 재정의시 **일반 규약**

1. **반사성(reflexivity)**
: null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true

2. **대칭성(symmetry)**
: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true이다
    
    ```java
    public final class CaseInsensitiveString {
        private final String s;
    
        public CaseInsensitiveString(String s) {
            this.s = Objects.requireNonNull(s);
        }
    
        **// 대칭성 위배!**
        @Override public boolean equals(Object o) {
            if (o instanceof CaseInsensitiveString)
                return s.equalsIgnoreCase(
                        ((CaseInsensitiveString) o).s);
            if (o instanceof String)  // 한 방향으로만 작동한다!
                return s.equalsIgnoreCase((String) o);
            return false;
        }
    ...
    }
    
    // 확인 1
    String s = "test";
    CaseInsensitiveString cis = new CaseInsensitiveString(”test”);
    
    // 확인 2
    List<CaseInsensitiveString> list = new ArrayList<>();
    list.add(cis);
    ```
    
    - 확인 1
        
        위 코드에서 보면 `cis.equals(”test”)` 는 `true`를 반환할 수 있지만 `s.equals(cis)` CaseInsensitiveString 클래스를 모르기 때문에 `false`를 반환하여 대칭성에 위반된다.
        
    - 확인 2
        
        `list.contains(s)`할 경우 `true`가 나올지 `false`가 나올지 알 수 없다(JDK 버전마다 다름)
        
        equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할 지 알 수 없다.
        
3. **추이성(transitivity)**
: null이 아닌 모든 참조 값 x, y, z에 대해 , x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
    <details>
    <summary> Point 클래스</summary>

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        @Override public boolean equals(Object o) {
            if (!(o instanceof Point))
                return false;
            Point p = (Point)o;
            return p.x == x && p.y == y;
        }
    ...
    }
    ```
    </details>

    </br>

    <details>
    <summary> ColorPoint 클래스 </summary>

    ```java
    public class ColorPoint extends Point {
        private final Color color;
    
        public ColorPoint(int x, int y, Color color) {
            super(x, y);
            this.color = color;
        }
    
    		// 대칭성 위배 코드
    		@Override public boolean equals(Object o) {
            if (!(o instanceof ColorPoint))
                return false;
            return super.equals(o) && ((ColorPoint) o).color == color;
        }
    
    		// 추이성 위배 코드
        @Override public boolean equals(Object o) {
            if (!(o instanceof Point))
                return false;
    
            // o가 일반 Point면 색상을 무시하고 비교한다.
            if (!(o instanceof ColorPoint))
                return o.equals(this);
    
            // o가 ColorPoint면 색상까지 비교한다.
            return super.equals(o) && ((ColorPoint) o).color == color;
        }
    
    		// 리스코프 치환 원칙 위배
        @Override public boolean equals(Object o) {
            if (o == null || o.getClass() != getClass())
                return false;
            Point p = (Point) o;
            return p.x == x && p.y == y;
        }
    ...
    }
    ```
    </details>   
    
    </br>
    
    위와 같이 두 클래스가 있을 때
    
    ```java
    // 확인1
    ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
    Point p2 = new Point(1,2);
    ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
    ```
    
    - **대칭성 위배** 코드
    `p2.equals(p1)` 은 `true` 를 반환하지만 `p1.equals(p2)` 는 `false`를 반환한다.
    - **추이성 위배** 코드
     `p1` 과 `p2` 두개를 비교할 때는 대칭성은 지켜주지만 `p3` 까지 같이 비교하게 되면 `p1.equals(p3)`는 `false`를 반환하게 되어 추이성 위배하게 된다.
    - **리스코프 치환 원칙 위배**
        
        > **리스코프 치환 원칙**
        어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 항위 타입에서도 똑같이 잘 작동해야 한다.
        > 
        
        `ColorPoint`는 `Point`의 하위 클래스이지만 각각 개별적으로 `equals`가 동작하기에 옳지 않은 코드이다.
        
    - **추이성 위해 해결 방법**
        
        **추이성 위배가 발생 근본적 이유** : 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
        **해결 방법(우회 방법) :** 상속 대신 컴포지션을 사용하라
        
        `ColorPoint` 클래스를 `Point`클래스의 상속을 받는 것이 아닌 `ColorPoint` 클래스에 `Point`를 private필드로 두고 뷰 메서드를 추가하는 방식
        
    
    > **java.sql.Timestamp와 java,util.Date 관계**
    실제 자바라이브러리의 Timestamp와 Date클래스의 경우가 대표적인 예이다.
    Timestamp는 Data를 확장한 후 nanoseconds 필드를 추가하였다. 때문에 대칭성을 위배한다(라이브러리 사용시 유의사항에 명시되어있음)
    > 
    
4. **일관성(consistency)**
: null이 아닌 모든 참조 값 x, y,에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
    - 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
    
    > **java.net.URL**
    해당 클래스의 equals는 url의 ip주소를 이용해 비교하지만, URL과 매핑된 호스트의 IP 주사가 항상 같다고 보장할 수 없어 일관성을 위배한다.(잘못 구현된 사례)
    > 
    
5. **null-아님**
: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
    
    `o.equals(null)`이 `true`를 반환하는 상황은 거의 발생하지 않지만 `NullPointerException` 을 던지는 코드는 항상 주의하여야 한다.
    
    ```java
    // 명시적 검사는  필요 없다
    @Override public boolean equals(Object o) {
    		if (o == null)
    				return false;
    		...
    }
    
    // 묵시적 null 검사 - 권장
    @Override public boolean equals(Object o) {
    		if (!(o instanceof MyType))
    				return false;
    		MyType mt = (MyType) o;
    		...
    }
    ```
    

### 4. 양질의 equals 메서드 구현 방법

1. ==연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.

### 5. 구현시 주의 및 참고사항

- float, double 를 제외한 기본 타입 필드는 `==` 연산자로 비교하고, 참조 타입 필드는 각각의 `equals` 메서드로, float double 필드는 `[Float.compare](http://Float.compare)` , `[Double.compare](http://Double.compare)` 로  비교를 한다
    
    → Float.NaN, -0.0f, 특수한 부동소수 값 등을 다뤄야 하기 때문이다.
    
- equals를 재정의할 땐 hashCode도 반드시 재정의하자
- 너무 복잡하게 해결하려 들지 말자.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
- equals 메서드 테스트해주는 오픈소스 → AutoValue 프레임워크

## **[Item 11] equals를 재정의하려거든 hashCode도 재정의하라**

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야한다.
선언하지 않는다면 HashMap 이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.

**Object 명세서에서 발췌한 규약**

- equals에서 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### 1. hashCode 구현 방법 및 주의 사항

**[구현 방법]**

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다(여기서 핵심필드란 equals 비교에 사용되는 필드를 말한다.)
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
        2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형(canonical representation)을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다(다른 상수도 괜찮지만 전통적으로 0이 적절)
        3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
    2. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다. → result = 31 * result + c;
3. result를 반환한다.
- 전형적인 hashCode 메서드 구현 예
    
    ```java
    @Override public int hashCode() {
    		int result = Short.hashCode(areaCode);
    		result = 31 * result + Short.hashCode(prefix);
    		result = 31 * result + Short.hashCode(lineNum);
    		return result
    }
    ```
    
- 간단한 방법(성능이 낮아 - 성능에 민감하지 않을 때 사용 권장)
    
    ```java
    @Override public int hashCode() {
    		return Object.hash(lineNum, prefix, areaCode);
    }
    ```
    

**[주의사항]**

- 아래 코드는 충분히 구현 가능한 코드지만 모든 값을 같은 값을 반환하기 때문에 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트처럼 동작한다. → O(1)의 수행시간이 O(n)으로 느려진다.
    
    ```java
    @Override public int hashCode() { return 42;}
    ```
    
- 이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.
- 성능 향상을 위해 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

## **[Item 12] toString을 항상 재정의하라**

Object의 기본 toString 메서드는 **클래스_이름@16진수로_표시한_해시코드**를 반환한다

1. **toString의 일반 규약**
    - 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보
    - 모든 하위 클래스에서 이 메서드를 재정의하라
    
2. **toString 참고사항**
    - toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
    - 스스로를 완벽히 설명하거나 의도하는 문자열이 반환되어야 한다. → [참고 코드](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter3/item12/PhoneNumber.java)
    - toString을 구현할 때면 반환값의 **포맷을 문서화**할지 정해야 한다.
        - 장점
            - 문서화하는 것을 권장한다.
            - 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게된다.
        - 단점
            - 포맷을 한번 명시하면 변경이 어렵다
    - 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자
    - AutoValue 프레임워크는 toString메소드도 생성해준다. → 잘 생성은 해주지만 클래스의 의미까지는 파악 못하여 확인은 필요

## **[Item 13] clone 재정의는 주의해서 진행하라**

### 1. **Cloneable vs clone**

- **Cloneable**
: 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(marker interface)
- **clone method**
: java.util.Object에 구현된 protected 메서드로 Object를 똑같이 복사하는 메서드

⇒ Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다
(clone 메서드가 선언된 곳은 Cloneable이 아닌 **Object**이기 때문)

### 2. Cloneable과 clone의 특징

- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하고, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다. → 인스턴스 사용의 특이 케이스로 따라하지 않는 것을 추천
- Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.(실무 사용자 입장)

### 3. 구현시 주의 및 참고사항

- 클래스의 필드가 기본 타입 or 불변 객체의 경우
    
    clone 메서드 사용 시 클래스에 정의된 모든 필드는 **원본 필드와 똑같은 값**을 갖는다 → 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이상적인 clone 메소드이다(단, 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메서드를 제공하지 않는게 좋다)
    
    ```java
    @Override public PhoneNumber clone() {
    		try {
    				return (PhoneNumber) super.clone();
    		} catch (CloneNotSupportedException e) {
    				throw new AssertionError(); // 일어날 수 없는 일이다.
    		}
    }		
    ```
    
    위 코드가 동작 하기 위해서는 PhoneNumber클래스에 Cloneable을 구현한다고 추가해야 한다. → `implements Cloneable` 
    
    PhoneNumber는 Object의 하위 클래스로 clone결과가 같은 PhoneNumber를 반환하기 위해 반환 형식을 PhoneNumber로 형변환해주어야 한다.
    
    PhoneNumber 클래스는 모든 필드가 기본 타입이기 때문에 위 코드는 옳은 코드이다.
    
- 클래스의 필드가 배열인 경우
    
    하지만 Stack 클래스와 같이 가지고 있는 필드가 배열과 같은 참조 타입일 경우 참조 값을 복사하지 배열의 각각의 값을 복사하지 않는다. → 배열 각각의 값을 복사할 수 있도록 구현 필요(clone을 재귀적으로 호출)
    
    > clone 메서드는 사실상 **생성자와 같은 효과**를 낸다. 즉, clone은 **원본 객체에 아무런 해를 끼치지 않는 동시**에 **복제된 객체의 불변식을 보장**해야 한다.
    > 
    
    > 배열의 clone 메서드는 **런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환**하기 때문에 배열의 clone을 사용할 경우 형변환을 해줄 필요는 없다
    > 
    - Stack 클래스의 elements 필드가 final이었다면 앞선 방식은 작동하지 않는다(final 필드에는 새로운 값을 할당할 수 없기 때문)
        
        **Cloneable 아키텍처는 ‘가변 객체를 참조하는 필드는 final로 선언하라’는 일반 용법과 충돌**하기 때문 → 일부 클래스에서는 clone을 구현하기 위해 final 한정자를 제거해야 함
        
- clone을 재귀적으로 호출하는 것만으로 부족한 경우 : HashTable
    
    HashTable의 버킷(bucket) 배열의 **각 원소는 연결리스트로 이루어져 있어 clone할 경우 참조값을 복사**하여 완벽히 독립적으로 사용할 수 없다.
    
    버킷 배열을 순회하여 **각 Entry를 deepcopy를 수행하**여 복사하는 것으로 해결 가능하다.(참조 코드 82p) → 연결리스트를 복제하는 방법은 복제하려는 배열의 길이가 길수록 StackOverFlow를 일으킬 위험이 있어 권장하지 않는다. → deepcopy를 순회하는 방법으로 코드 구현으로 해결 가능
    
- 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 한다
    
    만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 원본과 복제본의 상태가 달라질 가능성이 크다 → deepcopy 또는 put(key, value)와 같은 메서드는 final이거나 private이어야 한다.
    
- 상속용 클래스는 Cloneable을 구현해서는 안된다, But 상속해서 쓰기 위한 2가지 클래스 설계 방식
    1. Object의 방식처럼 잘 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException도 던질 수 있다고 선언하는 것
    2. clone을 동작하지 않게 구현하고 하위 클래스에서 재정의하지 못하게 하는 방법
- Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화 해주어야 한다.
    
    : Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야한다. 이때, 접근 제한자는 public, 반환 타입은 클래스 자신으로 변경
    
- CLoneable을 이무 구현한 클래스를 확장할 경우
    1. clone을 잘 구현해야한다.
    2. 복사 생성자  or 복사 팩터리를 활용한다.
        1. 복사 생성자 : `public Yum(Yum Yum) {...}` 
        2. 복사 팩터리 : `public static Yum newInstance(Yum Yum) {...}`
        - 생성자를 쓰지않고 인스턴스 생성, 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요하지 않는 이상적인 방법
        - 해당 클래스에서 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
            
            - 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다
            - 인터페이스 기반 복사 생성자와 복사 팩터리의 더 정확한 이름은 ‘변환 생성자’ ‘변환 팩터리’다.
            - 이들을 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.\
            → HashSet 객체 s를 TreeSet타입으로 복제할 수 있다. clone으로는 불가능한 이 기능을 변환 생성자로는 간단히 `new TreeSet<>(s)`로 처리할 수 있다.

## **[Item 14] Comparable을 구현할지 고려하라**
### **1. CompareTo의 특징**
- compareTo는 Object의 메서드가 아니다(2가지 특징만 빼면 Object의 equals와 같다)\
즉, compareTo를 구현했다는 것은 클래스의 인스턴스들에는 자연적인 순서가 있다는 뜻
    1. compareTo는 단순 동치성 비교 + 순서까지 비교 가능
    2. 제너릭하다

- 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현했다.

### **2. compareTo 메스드의 일반 규약**
- compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다.
    - Comparable을 구현한 클래스는 모든 x, y에 대해 `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`여야 한다(따라서 `x.compareTo(y)는 y.compareTo(x)`가 예외를 던질 때에 한해 예외를 던져야 한다)\
    : 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
    - Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, `(x.compareTo(y) > 0 && y.compareTo(z) > 0)`이면 `x.compareTo(z) > 0`이다\
    : 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면 첫 번째가 세 번째보다 크다
    - Comparable을 구현한 클래스는 모든 z에 대해 `x.compareTo(y) == 0` 이면 `sgn(x.compareTo(z) == sgn(y.compareTo(z))`다)\
    : 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
    - 이번 권고가 필수는 아니지만 꼭 지키는 게 좋다. `(x.compareTo(y) == 0) == (x.equals(y))`여야 한다. Compareable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다.\
    "주의 : 이 클래스의 순서는 equals 메서드와 일관되지 않다."\
    : (필수는 아니지만 꼭 지켜지길 권장)compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.

### **3. compareTo구현시 주의 사항**
- equals메서드와 유사하여 주의사항도 똑같다.
- 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가 했다면  compareTo 규약을 지킬 방법이 없다.(객체 지향적 추상화의 이점을 포기할 생각이 아니라면..)
    - 우회방법 : Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두는 법.

### **4. compareTo 메서드 작성 요령**
- equals 메서드와 유사, 몇가지 차이점만 주의 필요
- Comparable은 타입을 인수로 받는 제네릭 인터페이스로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다(입력 인수 타입을 형변환할 필요 x)
- compareTo 메서드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교
- 클래스의 핵심 필드가 여러개 일 경우, 가장 핵심 필드부터 비교해 나가야 한다. 비교 결과가 0이 아니라면, 즉 순서가 결정되면 바로 반환한다.