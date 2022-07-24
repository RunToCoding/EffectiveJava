# 8장. 메서드

<aside>
✏️ 사용성, 견고성, 유연성을 중심으로 메서드를 설계할 때 주의할 점 <br>
⇒ 상당 부분 생성자에도 적용

</aside>

<br><br>

## *아이템 49.* 매개변수가 유효한지 검사하라 *(p.298)*

- 메서드와 생성자의 입력 **매개변수 값이 만족해야 하는 조건** (e.g. 인덱스 값 ≠ 음수, 객체 참조 ≠ `null`)은 반드시 **문서화**해야 하며, **메서드 몸체가 시작되기 전에 검사**해야 함.

- 매개변수 검사를 제대로 하지 못하면 생길 수 있는 문제?
    - 메서드가 **수행되는 중간**에 모호한 **예외를 던지며 실패**
    - 메서드가 잘 수행되지만 **잘못된 결과를 반환**
    - 메서드는 문제 없이 수행됐지만, 객체를 이상한 상태로 만들어 미래에 **이 메서드와는 관련 없는 오류** 발생 <br>
    ⇒ **실패 원자성**(failure atomicity, *아이템 76*)을 어기는 결과 <br>
    (실패 원자성 : 호출된 **메서드가 실패**하더라도 해당 **객체는 메서드 호출 전 상태 유지**)
    
- `public`과 `protected` 메서드는 매개 변수 값이 잘못됐을 때 던지는 **예외를 문서화**해야 함. (`@throws` 자바독 태그 사용, *아이템 74*) <br>
⇒ 보통 `IllegalArgumentException`, `IndexOutOfBoundsException`, `NullPointerException` 중 하나(*아이템 72*)

- 매개변수의 제약을 문서화한다면 그 **제약을 어겼을 때 발생하는 예외도 함께 기술**해야 함.

- 유용한 자동 검사 메서드들
    - `null` 검사 - JAVA 7 `java.util.Objects.requireNonNull`  <br>
    ⇒ 원하는 예외 메시지 지정, 입력을 그대로 반환하여 **값을 사용하는 동시에** `null` **검사 수행** 가능
        
        ```java
        this.strategy = Objects.requireNonNull(strategy, "전략");
        ```
        
    - 리스트/배열 `Objects` 범위 검사 - JAVA 9 `checkFromIndexSize`, `checkFromToIndex`, `checkIndex`
    
- `public`이 아닌 메서드는 **단언문**(`assert`)을 사용해 매개변수 유효성 검증
    
    ```java
    private static void sort(long a[], int offset, int length) {
    	assert a != null;
    	assert offset >= 0 && offset <= a.length;
    	assert length >= 0 && length <= a.length - offset;
    	...
    }
    ```
    
    - 단언문은 자신이 단언한 **조건이 무조건 참이라고 선언**
    - 일반적인 유효성 검사와 다른 점?
        - 실패하면 `AssertionError`를 던짐
        - 런타임에 아무런 효과도, 성능 저하도 없음
        
- 메서드가 직접 사용하지는 않으나 **나중에 쓰기 위해 저장하는 매개변수**는 더 신경 써서 검사해야 함
    - **생성자 매개변수의 유효성 검사**는 이 원칙의 특수한 사례
    
- 메서드 **몸체 실행 전에 매개변수 유효성을 검사**해야 한다는 규칙의 **예외**?
    - **유효성 검사 비용**이 지나치게 높거나 실용적이지 않을 때
    - **계산 과정**에서 암묵적으로 **검사가 수행**될 때
    e.g. `Collections.sort(List)` 메서드에서 객체를 비교할 때 **상호 비교될 수 없는 타입**의 객체가 있으면 `ClassCastException`을 던지므로 미리 검사할 필요 없음 <br>
        - 계산 과정에서 필요한 유효성 검사가 이루어지지만, 실패했을 때 **잘못된 예외**를 던지는 경우가 있음 (발생한 예외 ≠ API에서 던지기로 한 예외) <br>
        ⇒ **예외 번역**(exception translate, *아이템 73*) **관용구**를 사용하여 **API 문서에 기재된 예외로 번역**해주어야 함
        

<aside>
📢 “매개변수에 제약을 두는 게 좋다” 가 아님! <br>
오히려 메서드는 최대한 범용적으로 설계해야 함. <br>
그러나 메서드나 생성자를 작성할 때 매개변수들에 어떤 제약이 있을지 생각하고, 제약들을 문서화하고 명시적으로 검사하는 습관을 기르자. <br>

</aside>

<br><br>

## *아이템 50.* 적시에 방어적 복사본을 만들라 *(p.302)*

JAVA는 **메모리 충돌 오류에서 안전**하고 시스템의 다른 부분의 동작에 대해 **클래스 불변식**이 지켜지는 언어지만, 방어적으로 프로그래밍 해야 함.

- 어떤 객체든 그 객체의 허락 없이는 **외부에서 내부를 수정하는 일은 불가능**하지만, 주의를 기울이지 않으면 **내부를 수정하도록 허락**하는 경우가 생김
    
    ```java
    /* 기간을 표현하는 클래스 - 불변식을 지키지 못한 예시 */
    public final class Period {
    	private final Date start;
    	private final Date end;
    
    	public Period(Date start, Date end) {
    		if (start.compareTo(end) > 0)
    			throw new IllegalArgumentException(start + "가" + end + "보다 늦다.");
    		this.start = start;
    		this.end = end;
    	}
    	
    	public Date start() {
    		return start;
    	}
    
    	public Date end() {
    		return end;
    	}
    }
    ```
    
    ⇒ `Date`가 가변이라는 사실을 이용하면 불변식을 깨뜨릴 수 있음
    
    ```java
    /* 공격 1 */
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    end.setYear(78);  // 1978년으로 설정
    ```
    
    ⇒ `Date` 대신 JAVA 8  불변(*아이템 17*)인 `Instant` 사용 (혹은 `LocalDateTime` or `ZonedDateTime`)
    
    `Date`는 낡은 API이므로 더 이상 사용하면 안 됨! 
    
    <aside>
    ✏️ 예전에 작성된 낡은 코드들을 대처하기 위한 방법
    
    </aside>
    
- 인스턴스 내부를 보호하려면 **생성자에서 받은 가변 매개변수 각각을 방어적으로 복사**(defensive copy)하고, **인스턴스 내부에서는** 원본이 아닌 **복사본을 사용**해야 함.
    
    ```java
    /* 공격 1에 대한 방어 */
    public Period(Date start, Date end) {
    	this.start = new Date(start.getTime()); // 복사본 생성
    	this.end = new Date(end.getTime());
    	
    	if (this.start.compareTo(end) > 0)      // 복사본으로 유효성 검사
    			throw new IllegalArgumentException(start + "가" + end + "보다 늦다.");
    }
    ```
    
    - 방어적 복사를 매개변수 유효성 검사 전에 수행
    - 방어적 복사에 `Date`의 `clone` 메서드를 사용하지 않음 <br>
    : `Date`는 `final`이 아니므로 `clone`이 `Date`가 정의한 게 아닐 수 있기 때문 <br>
    ⇒ 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 `clone`을 사용해서는 안 됨
    
- 생성자를 수정하면 앞선 공격은 막을 수 있지만, **접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문**에 인스턴스는 아직도 변경 가능
    
    ```java
    /* 공격 2 */
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    p.end().setYear(78);
    ```
    
    ⇒ 접근자가 가변 필드의 방어적 **복사본을 반환**하면 됨
    
    ```java
    /* 공격 2에 대한 방어 */
    public Date start {
    	return new Date(start.getTime());
    }
    public Date end {
    	return new Date(end.getTime());
    }
    ```
    
    ⇒ `Period`의 모든 필드가 객체 안에 완벽하게 캡슐화되어 완벽한 불변 !
    
    - `Period`가 가지고 있는 `Date` 객체는 `java.util.Date`임이 확실하므로 접근자 메서드에서는 방어적 복사에 `clone`을 사용해도 되지만, **인스턴스를 복사**하는 데는 **생성자나 정적 팩터리**를 쓰는 것이 좋음(*아이템 13*).
    
- 방어적 복사의 또 다른 목적
    - **잠재적으로 변경될 수 있는 객체**의 참조를 내부의 자료구조에 보관해야 할 때, 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 문제 없이 동작할지 확신할 수 없다면 **복사본을 만들어 저장**
    - **가변인 객체를 클라이언트에 반환**할 때 안심할 수 없다면 **원본을 노출하지 말고** 방어적 **복사본 반환**
        - **길이가 1 이상인 배열은 무조건 가변**이므로, 내부에서 사용하는 **배열을 클라이언트에 반환할 때**는 항상 **방어적 복사 수행**(혹은 배열의 불변 뷰 반환, *아이템 15*)

<aside>
💡 방어적 복사에는 성능 저하가 따르고, 항상 쓸 수 있는 것도 아님. <br>
   호출자가 컴포넌트 내부를 수정하지 않을 것임을 확신하면 생략 가능 <br>
   - 이러한 상황에도 호출자가 수정하지 말아야 함을 명확히 문서화해야 함

</aside>

<br><br>

## *아이템 51.* 메서드 시그니처를 신중히 설계하라 *(p.308)*

<aside>
✏️ 배우기 쉽고, 쓰기 쉬우며, 오류 가능성이 적은 API 설계 요령

</aside>

- **메서드 이름을 신중히 짓자**
    - 표준 명명 규칙(*아이템 68*)을 따라야 함
    - **이해**할 수 있고, 같은 패키지에 속한 **다른 이름들과 일관**되게 짓는 것이 최우선
    - 개발자 커뮤니티에서 **널리 받아들여지는 이름** 사용
    - **긴 이름은 피하자.**
    
- **편의 메서드를 너무 많이 만들지 말자**
    - 메서드가 너무 많은 클래스나 인터페이스는 좋지 않고, **자신의 각 기능을 완벽히 수행하는 메서드**로 제공해야 함
    - 아주 자주 쓰일 경우에만 별도의 약칭 메서드를 두고, 확신이 서지 않으면 만들지 말자.
    
- **매개변수 목록은 짧게 유지하자**
    - **4개 이하**가 좋음
    - **같은 타입의 매개변수** 여러 개가 연달아 나오는 경우는 **특히 좋지 않음**
    - 매개변수 목록을 줄이는 방법?
        - **여러 메서드**로 쪼갬
        - 매개변수 여러 개를 묶어주는 **도우미 클래스**를 만들어 하나의 매개변수로 주고 받음 <br>
        : 매개변수 몇 개를 독립된 하나의 개념으로 볼 수 있을 때
        - 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용 <br>
        : 매개변수가 많을 때, 특히 일부는 생략해도 괜찮을 때
        - **모든 매개변수를 하나로 추상화한 객체**를 정의하고, 클라이언트에서 `setter` 메서드를 호출해 **일부 매개변수 값만 설정**
    
- **매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다**(*아이템 64*)
    - 클래스를 사용하면 **특정 구현체만 사용하도록 제한**하는 것
    - 입력 데이터가 다른 형태로 존재하면 **객체 복사** 비용 소모
    
- `boolean`**보다는 원소 2개짜리 열거 타입이 낫다**
    - 메서드 이름상 `boolean`을 받아야 의미가 더 명확할 때는 예외
    
<br><br>

## *아이템 52.* 다중정의는 신중히 사용하라 *(p.312)*

> **다중정의** : overloading, **오버로딩**
> 

```java
/* 컬렉션 분류기 */
public class CollectionClassifier {
	public static String classify(Set<?> s) {
		return "집합";
	}
	public static String classify(List<?> lst) {
		return "리스트";
	}
	public static String classify(Collection<?> c) {
		return "그 외";
	}

	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>(),
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		};

		for (Collection<?> c : collections)
			System.out.println(classify(c));
	}
}
```

⇒ “`그 외`” 만 세 번 출력됨

⇒ 다중정의된 3개의 `classify`중 **어느 메서드를 호출할지 컴파일타임에 정해지기 때문**

- 컴파일타임에는 `for`문 안의 `c`는 항상 `Collection<?>` 타입이고, **런타임에 타입이 달라짐**
- **재정의(오버라이딩)한 메서드는 동적으로 선택**되고, **다중정의(오버로딩)한 메서드는 정적으로 선택**됨

- 위 예시의 해결 방법
: `CollectionClassifier`의 `classify` 메서드를 하나로 합친 후 `instanceof`로 명시적으로 검사
    
    ```java
    public static String classify(Collection<?> c) {
    	return c instanceof Set ? "집합" :
    				 c instanceof List ? "리스트" : "그 외";
    }
    ```
    

- **매개변수 수가 같은 다중정의**는 피하는 게 좋고, 대신 **메서드 이름을 다르게** 짓는 방법 고려
    - **생성자**는 이름을 다르게 지을 수 없으므로 두 번째 생성자부터는 무조건 다중정의가 됨 (⇒ 대안으로 **정적 팩터리** 활용 가능, *아이템 1*)
    - **여러 생성자**가 **같은 수의 매개변수**를 받아야 하는 경우의 대책?
    - 매개변수 중 하나 이상이 **두 타입의 값을** 서로 어느 쪽으로든 **형변환할 수 없으면** 됨 (=근본적으로 다름) <br>
    ⇒ **어느 다중정의 메서드를 호출**할지 매개변수들의 **런타임 타입만으로 결정**되므로 혼동 X

- 다중정의 시 주의해야 할 근거 예시 - 1. **제네릭**과 **오토박싱**
    
    ```java
    public class SetList {
    	public static void main(String[] args) {
    		Set<Integer> set = new TreeSet<>();
    		List<Integer> list = new ArrayList<>();
    		
    		for (int i = -3; i < 3; i++) {
    			set.add(i);
    			list.add(i);
    		} // -3, -2, -1, 0, 1, 2 추가
    
    		for (int i = 0; i < 3; i++) {
    			set.remove(i);   // => (-3, -2, -1)
    			list.remove(i);  // => [-2,  0,  2]
    		}
    	}
    }
    ```
    
    - `set`의 `remove`는 `remove(Object)`, 다중정의된 메서드 X
    - `list`는 다중정의된 `remove(Object)`,  `remove(int)` 중 후자를 선택 → **지정한 위치의 원소** 제거 <br>
    ⇒ `list.remove`의 인수로 **박싱**된 타입 `Integer`을 전달하여 해결
        
        ```java
        for (int i = 0; i < 3; i++) {
        	set.remove(i);             // => (-3, -2, -1)
        	list.remove((Integer) i);  // => [-3, -2, -1]
        	// 또는 list.remove(Integer.valueOf(i));
        }
        ```
        
    
    - JAVA 5에서 **제네릭**과 **오토박싱**의 도입으로 `Object`와 `int`가 근본적으로 다르지 않게 되어 발생한 문제
    
- 다중정의 시 주의해야 할 근거 예시 - 2. **람다**와 **메서드 참조**
    
    ```java
    // Thread의 생성자 호출
    new Thread(System.out::println).start();
    
    // ExecutorService의 submit 메서드 호출 => 컴파일 오류!
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.submit(System.out::println);
    ```
    
    - `submit`의 다중정의 메서드 중에는 `Runnable`을 받는 메서드도, `Callable<T>`를 받는 메서드도 있음
    - 다중정의된 메서드(혹은 생성자)들이 **함수형 인터페이스를 인수로 받을 때**, 서로 다른 함수형 인터페이스라도 **인수 위치가 같으면 혼란**이 생김 <br>
    ⇒ 서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않음
