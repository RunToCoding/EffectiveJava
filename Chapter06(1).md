# 6장. 열거 타입과 애너테이션

> 자바의 특수한 목적의 참조 타입
- **열거 타입(enum**; 열거형) : 클래스의 일종, **애너테이션(annotation)** : 인터페이스의 일종
> 

<aside>
✏️ 열거 타입과 애너테이션을 올바르게 사용하는 방법

</aside>

## 아이템 34. `int` 상수 대신 열거 타입을 사용하라 (p208)

- 열거 타입을 지원하기 전에는 **정수 열거 패턴(int enum pattern)** 사용

```java
public static final int APPLE_FUJI         = 0;
public static final int APPLE_PIPPIN       = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD  = 2;
```

⇒ 타입 안전 보장 ❌, 표현력 👎, 상수의 값이 바뀌면 다시 컴파일해야 함

- 정수 상수는 숫자 값이므로 의미 파악 불가
- 정수 대신 문자열 상수를 사용하는 **문자열 열거 패턴(string enum pattern)**은 더 나쁨 (오타로 인한 런타임 버그, 문자열 비교 성능 저하 등)

- **열거 타입**: 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TAMPLE, BLOOD }
```

- **열거 타입 자체는 클래스**
- **상수 하나 당** 자신의 **인스턴스**를 하나씩 만들어 `public static final` ****필드로 공개
- 열거 타입 선언으로 만들어진 인스턴스는 딱 하나씩만 존재함이 보장 (⇒ **인스턴스 통제**됨)

- 열거 타입은 **컴파일타임 타입 안전성** 제공
: `Apple` 열거 타입을 매개변수로 받는 메서드에서 건네받은 참조는 `Apple`의 세 가지 값 중 하나임이 확실하고, 다른 타입의 값을 넘기려 하면 컴파일 오류 발생
- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 됨
- `toString` 메서드는 출력하기에 적합한 문자열을 내어줌

- **임의의 메서드나 필드를 추가**하거나 **임의의 인터페이스 구현** 가능
- 각 상수와 연관된 데이터(e.g. 과일의 색, 과일 이미지 반환 메서드)를 해당 상수 자체에 내장시키고 싶은 경우

```java
/* 데이터와 메서드를 갖는 열거 타입 */

public enum Planet {
	MERCURY(3.302e+23, 2.439e6),
	VENUS(4.869e+24, 6.052e6),
	EARTH(5.97e+24, 6.378e6);

	// 열거 타입은 근본적으로 불변이므로 모든 필드는 final이어야 함
	private final double mass;
	private final double radius;
	private final double surfaceGravity;

	private final double G = 6.67300E-11;

	// 생성자에서 데이터를 받아 인스턴스 필드에 저장
	Planet(double mass, double radius) {
		this.mass = mass;
		this.radius = radius;
		surfaceGravity = G * mass / (radius * radius);
	}

	public double mass() { return mass; }
	public double radius() { return radius; }
	public double surfaceGravity() { return surfaceGravity; }
}
```

- **제거된 상수를** **참조**하는 클라이언트 프로그램에서는 **컴파일 오류/런타임 예외** 발생
**제거된 상수를** **참조하지 않는** 클라이언트에는 **아무 영향 없음**
- 널리 쓰이는 열거 타입은 **톱레벨 클래스**로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 **멤버 클래스**(아이템 24)로 만듦

- **상수별 메서드 구현**(constant-specific method implementation):  열거 타입에 **추상 메서드**를 선언하고 각 상수에서 자신에 맞게 **재정의**하는 방법
- 상수마다 동작이 달라져야 하는 경우

```java
/* 상수별 메서드 구현을 활용한 열거 타입 */

public enum Operation {
	PLUS   { public double apply(double x, double y){ return x + y; } },
	MINUS  { public double apply(double x, double y){ return x - y; } },
	TIMES  { public double apply(double x, double y){ return x * y; } },
	DIVIDE { public double apply(double x, double y){ return x / y; } };

	public abstract double apply(double x, double y);
}
```

- `toString` 재정의

```java

'public enum Operation {
	PLUS("+")   { public double apply(double x, double y){ return x + y; } },
	MINUS("-")  { public double apply(double x, double y){ return x - y; } },
	TIMES("*")  { public double apply(double x, double y){ return x * y; } },
	DIVIDE("/") { public double apply(double x, double y){ return x / y; } };

	private final String symbol;

	Operation(String symbol) { this.symbol = symbol; }

	@Override public String toString() { return symbol; }
	public abstract double apply(double x, double y);
}

public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	for (Operation op : Operation.values())
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

```java
/* 인수 2 4 실행결과 */

2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
```

- `toString`이 반환하는 **문자열을 해당 열거 타입 상수로 변환**해주는 `fromString` 메서드

```java
// 열거 타입 상수 생성 후 정적 필드가 초기화될 때 Operation 상수가 stringToEnum 맵에 추가
private static final Map<String, Operation> stringToEnum = 
				Stream.of(values()).collect(toMap(Object::toString, e -> e));

// 문자열이 가리키는 연산이 존재한다면 반환
public static Optional<Opearation> fromString(String symbol) { 
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

- 상수별 메서드 구현은 열거 타입 **상수끼리 코드를 공유하기 어렵다**는 단점이 있음

```java
/* e.g. 급여명세서에서 쓸 요일을 표현하는 열거 타입 */

/* 값에 따라 분기하여 코드를 공유하는 경우 */
enum PayrollDay {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

	private static final int MINS_PER_SHIFT = 8 * 60;

	int pay(int minutesWorked, int payRate) {
		int basePay = minutesWorked * payRate;
		
		int overtimePay;
		switch(this) {
			case SATURDAY: case SUNDAY: // 주말에는 무조건 overtimePay
				overtimePay = basePay / 2;
				break;
			default: // 주중에 오버타임이 발생하면 overtimePay
				overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 :
					(minutesWorked - MINS_PER_SHIFT) * payRate / 2;
		}
		return basePay + overtimePay;
	}
}
```

→ 휴가 같은 새로운 값을 열거 타입에 추가하려면 case문을 추가해야 함

- 새로운 상수를 추가할 때 잔업수당 ‘전략’을 선택하도록 하는 방법

```java
/* 전략 열거 타입 패턴 */

enum PayrollDay {
	MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), 
  THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

	private final PayType payType;

	PayrollDay(PayType payType) { this.payType = payType; }

	int pay(int minutesWorked, int payRate) {
		return payType.pay(minutesWorked, payRate);
	}

	// 중첩 열거 타입
	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				return minutesWorked <= MINS_PER_SHIFT ? 0 :
					(minsWorked - MINS_PER_SHIFT) * payRate / 2;
				}
		},
		WEEKEND {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked * payRate / 2;
		};

		abstract int overtimePay(int mins, int payRate);
		private static final int MINS_PER_SHIFT = 8 * 60;

		int pay(int minsWorked, int payRate) {
			int basePay = minsWorked * payRate;
			return basePay + overtimePay(minsWorked, payRate);
		}
	}
}
```

- 기존 열거 타입에 **상수별 동작을 혼합**해 넣을 때는 `switch`문이 좋은 선택이 될 수 있음
e.g. 각 연산의 반대 연산 반환
- 열거 타입을 쓰는 것이 좋은 경우 : 필요한 원소를 **컴파일타임에 다 알 수 있는 상수 집합**일 때
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없음

## 아이템 35. `ordinal` 메서드 대신 인스턴스 필드를 사용하라 (p221)

- `ordinal` 메서드 : 열거 타입에서 **해당 상수가 몇 번째 위치**인지를 반환하는 메서드

```java
/* ordinal을 잘못 사용한 예 */
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
	public int numberOfMusicians() { return ordinal() + 1; }
}
```

⇒ 상수 선언 순서를 바꾸면 오동작.
이미 사용 중인 정수와 값이 같은 상수를 추가하거나 값을 중간에 비워둘 수 없음
`ordinal`은 이런 용도로 설계된 게 아님!

```java
/* 인스턴스 필드에 저장하는 방법 */
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), 
	SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
	NONET(9), DECTET(10), TRIPLE_QUARTET(12);

	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; }
	public int numberOfMusicians() { return numberOfMusicians; }
}
```

## 아이템 36. 비트 필드 대신 `EnumSet`을 사용하라 (p223)

- **비트 필드(bit field)** : 비트별 OR을 사용해 여러 상수를 하나로 모아 만든 집합
- 상수 집합을 주고받아야 할 때 주로 사용해 옴
    
    ```java
    public class Text {
    	public static final int STYLE_BOLD          = 1 << 0; // 1
    	public static final int STYLE_ITALIC        = 1 << 1; // 2
    	public static final int STYLE_UNDERLINE     = 1 << 2; // 4
    	public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
    
    	public void applyStyles(int styles) { ... }
    }
    ```
    
    - 비트별 연산을 사용해 집합 연산 수행 가능
    e.g. `text.applyStyle(STYLE_BOLD | STYLE_ITALIC);`
    - 비트 필드 값이 그대로 출력되면 해석하기 훨씬 어렵고, 필요한 최대 비트를 API 작성 시 미리 예측해야 함

- `java.util` 패키지의 `EnumSet` 클래스
: `Set` 인터페이스 구현한 클래스. 타입 안전, 다른 어떤 `Set` 구현체와도 함께 사용 가능
    
    ```java
    public class Text {
    	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    	public void applyStyles(Set<Style> styles) { ... }
    }
    ```
    
    → `text.applyStyle(EnumSet.of(Style.BOLD, Style.ITALIC));` 수행
    
    - `EnumSet<Style>`이 아닌 `Set<Style>`을 받은 이유?
    : 인터페이스로 받는 게 좋은 습관(아이템 64)

## 아이템 37. `ordinal` 인덱싱 대신 `EnumMap`을 사용하라 (p226)

- `ordianl` 인덱싱
    
    ```java
    class Plant {
    	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    	
    	final String name;
    	final LifeCycle lifeCycle;
    
    	Plant(String name, LifeCycle lifeCycle) {
    		this.name = name;
    		this.lifeCycle = lifeCycle;
    	}
    
    	@Override public String toString() {
    		return name;
    	}
    }
    
    /* ordinal 인덱싱 */
    Set<Plant>[] plantsByLifeCycle = 
    	(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
    
    for (int i = 0; i < plantsByLifeCycle.length; i++)
    	plantsByLifeCycle[i] = new HashSet<>();
    
    for (Plant p : garden)
    	plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
    
    for (int i = 0; i < plantsByLifeCycle.length; i++) {
    	System.out.printf("%s: %s%n, Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
    ```
    
    - 배열과 제네릭이 호환되지 않으므로 **비검사 형변환** 수행해야 함(아이템28)
    - 배열은 각 인덱스의 의미를 모르므로 출력 결과에 **직접 레이블**을 달아야 함
    - **정확한 정숫값을 사용**한다는 것을 직접 보증해야 함

- `EnumMap` : **열거 타입을 키로 사용**하도록 설계한 `Map` 구현체
    
    ```java
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
    	new EnumMap<>(Plant.LifeCycle.class);
    
    for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    	plantsByLifeCycle.put(lc, new HashSet<>());
    
    for (Plant p : garden)
    	plantsByLifeCycle.get(p.lifeCycle).add(p);
    
    System.out.println(plantsByLifeCycle);
    ```
    
    - 안전하지 않은 형변환은 쓰지 않음
    - 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하므로 직접 레이블을 달지 않아도 됨
    - 배열 인덱스 계산 과정에서 오류 발생 가능성 없음
    
- **스트림**(아이템 45)
    
    ```java
     System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle,
    	() -> new EnumMap<>(LifeCycle.class), toSet())));
    ```
    
    - `EnumMap`만 사용하면 언제나 상수 당 하나씩의 중첩 맵을 만들지만, 스트림을 사용하면 해당 상수에 속하는 객체가 있을 때만 만듦
