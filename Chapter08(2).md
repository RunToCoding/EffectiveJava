
## [item53] 가변인수는 신중히 사용하라

---

**가변인수(varargs)** 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.

- 가변인수 메서드를 호출하면, 인수의 개수와 길이가 같은 배열을 만들고 인수들을 배열에 저장하여 가변인수 메서드에 건네줌
- 다음은 입력받은 int 인수들의 합을 계산해주는 가변인수 메서드이다.

**간단한 가변인수 활용**

```java
//sum(1,2,3)은 6을, sum()은 0을 리턴한다.
static int sum(int... args) {
    int sum = 0;
		// 배열에서 하나씩 꺼낸다
    for (int arg : args)
        sum += arg;
    return sum;
}
```

**인수가 1개 이상**이어야할 때 다음과 같이 설계할 수 있다 - 잘못 설계된 예!

```java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

- 인수를 0개만 넣어 호출(컴파일X)하면 런타임에 실패한다 - 가장 심각한 문제
- args 유효성 검사를 명시적으로 해야하고
- min의 초깃값을 Integer.MAX_VALUE로 설정하지 않고는 for-each문도 사용할 수 없다.

이러한 문제를 해결 수 있는 더 나은 방법 → **인수가 1개 이상이어야 할 때 가변 인수를 제대로 사용하는 방법 = 매개변수를 2개 받도록 한다. → (평범한 매개변수, 가변인수)**

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

- 가변인수는 인수 개수가 정해지지 않았을 때 아주 유용
- printf는 가변인수와 한 묶음으로 자바에 도입되었고, 이때 핵심 리플렉션 기능도 재정비 되었다.(item65)

---

- 하지만 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다.
- 이러한 문제를 해결 할 수 있는 방법이 있다. 비록 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때, 선택할 수 있는 다중정의를 이용하면 된다.

```java
//인수 0개
public void foo() { }
//인수 1개
public void foo(int a1) { }
//인수 2개
public void foo(int a1, int a2) { }
//인수 3개
public void foo(int a1, int a2, int a3) { }
//인수 4개 -> 나머지 5%의 배열을 생성한다.
public void foo(int a1, int a2, int a3, int... rest) { }
```

- 예를들어 해당 메서드 호출의 95%가 인수 3개를 사용한다고 하면
    - 인수가 0개인것부터 4개인것 까지 총 5개를 다중정의한다.
        - 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하는 것이다.
- 따라서 메서드 호출 중 단 5%만이 배열을 생성한다.
- EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화하였다.

> **핵심 정리**
- 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다.
- 메서드를 정의할 때  필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.
> 

---

## [item 54] null 이 아닌, 빈 컬렉션이나 배열을 반환하라

---

### 컬렉션이 비었으면 null을 반환한다. - 따라하지 말 것!

- 주변에서 흔히 볼 수 있는 메서드

```java
private final List<Cheese> cheeseInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다. 
 *      단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheeseInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock) ;
}
```

- 사실 재고가 없다고 해서 특별히 취급할 이유는 없다.
- 그럼에도 이 코드처럼 null을 반환해야한다면
    - 클라이언트는 **이 null 상황을 처리하는 코드를 추가로 작성**해야한다.

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("좋았어, 바로 그거야.");
```

- 컬렉션이나 배열같은 컨테이너(Container)가 비었을 때 null을 반환하는 메서드를 사용할 때면 항시 **이와 같은 방어 코드를 넣어줘야 한다.**
- 클라이언트에서 방어코드를 빼먹으면 오류가 발생할 수 있다.
    - 실제로 객체가 0개일 가능성이 거의 없는 상황에서는 수년 뒤에야 오류가 발생하기도 한다.
    - 한편, null을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야 해서 **코드가 더 복잡**해진다.

---

### 컴테이너를 할당하는 데도 비용이 들어 null을 반환하는 쪽이 낫다는 주장이 틀린 점

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한 **이 정도의 성능차이는 신경 쓸 수준이 못된다**. 
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 
- 빈 컬렉션을 반환하는 올바른 예

```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

- 매번 똑같은 빈 **불변** 컬렉션을 반환하는 것이다.
    - 알다시피 불변객체는 자유롭게 공유해도 안전하다.(아이템 17)

```java
public List<Cheese> getCheeses() {
    return cheeseInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
																			//불변 객체 공유
}
```

- 이와 같이 `Collections.emptyList` 메서드가 그런 예이다.
    - 집합은 `Collections.emptyset` 을, 맵은 `Collections.emptyMap` 을 사용
    - 단, 최적화에 해당하니 꼭 필요할 때만 사용하자.
    - 최적화가 필요하다고 판단되면 수정 전과 후의 성능을 측정하여 실제로 성능이 개선되는지 꼭 확인하라.

---

### 빈 컬렉션을 매번 새로 할당하지 않아도 된다.

```java
public List<Cheese> getCheeses() {
    return cheeseInStock.toArray(new Cheese[0]);
}
```

- 배열을 쓸 때도 마찬가지로 **절대 null을 반환하지 말고 길이가 0인 배열을 반환**하라.
    - 보통은 단순히 정확한 길이의 배열을 반환하기만 하면 된다.
    - 그 길이가 0일수도 있을 뿐이다.

---

### 길이가 0일수도 있는 배열을 반환하는 올바른 방법

```java
public Cheese[] getCheeses() {
    return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

- `toArray` 메서드에 건넨 길이 0까지 배열은 우리가 원하는 반환 타입을 알려주는 역할을 함
- 이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다.
    - 길이가 0인 배열은 모두 불변이기 때문이다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

- 최적화 버전의 getCheeses 는 항상 `EMPTY_CHEESE_ARRAY`를 인수로 넘겨 `toArray`를 호출한다.
    - 따라서 `cheeseInStock`이 비었을 때면 언제나 `EMPTY_CHEESE_ARRAY`를 반환하게 된다.
    
    ```java
    return cheeseInStock.toArray(new Cheese[cheesesInStock.size()]);
    ```
    
    - 하지만 단순히 성능을 개선할 목적이라면 `toArray`에 넘기는 배열을 미리 할당하는건 추천하지 않는다. 오히려 성능이 떨어진다는 연구 결과도 있다.

> 핵심 정리
- null이 아닌, 빈 배열이나 컬렉션을 반환하라.
- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은것도 아니다.
> 

---

## [item 55] 옵셔널 반환은 신중히 하라

---

### 메서드가 특저 조건에서 값을 반환할 수 없을 때의 두 가지 방법의 허점

1. 예외를 던지기의 허점
    1. 예외는 진짜 예외적인 상황에서만 사용해야함
    2. 예외를 생성할 때 스택 추적 전체를 캡쳐하므로 비용이 많이 든다.
2. (반환 타입이 객체 참조라면) null을 반환하기의 허점
    1. null을 반환할 수 있는 메서드를 호출할 때는, 별도의 null 처리 코드를 추가해야한다.
        1. null 처리를 무시하고 반환된 null 값을 어딘가에 저장해두면 언젠가 NullPointerException이 발생할 수 있다.

---

### Optional<T>의 등장

> 자바 8로 버전이 올라가면서 하나의 선택지가 등장하였다. = Optional<T>
> 
- null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.
- 아무것도 담지 않은 옵셔널 - **비었다.**
- 어떤 값을 담은 옵셔널 - **비지 않았다.**
- 옵셔널은 원소를 최대 1개 가질 수 있는 **불변** 컬렉션이다.
- 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다.

---

**컬렉션에서 최대값을 구한다.(컬렉션이 비었으면 예외를 던진다.)**

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
//컬렉션이 비었으면 예외를 던진다
	if(c.isEmpty())
    	throw new IllegalArgumentException("빈 컬렉션");
    
    E result = null;
    for (E e : c)
    	if (result==null || e.compareTo(result) > 0)
        	result = Objects.requireNonNull(e);
            
    return result;
}
```

- 이 메서드에 빈 컬렉션을 건네면 `IllegalArgumentException`을 던진다.
- 아이템 30에서도 Optional<E>를 반환하는 편이 더 낫다고 이야기 했다

---

**컬렉션에서 최댓값을 구해 Optional<E>로 반환한다.**

```java
public static <E extends Comparable<E>> **Optional<E>** max(Collection<E> c) {
   if (c.isEmpty())
			//빈 옵셔널
   		return **Optional.empty()**;
        
   E result = null;
   for (E e : c)
   		if(result==null||e.compareTo(result) > 0)
        	result = Objects.requireNonNull(e);
   //값이 든 옵셔널 -> null을 넣으면 NullPointerException을 던짐
   return **Optional.of(result)**;
}
```

- 빈 옵셔널은 `Optional.Empty()` 로 만들고, 값이 든 옵셔널은 `Optional.of(value)` 로 생성했따.
- `Optional.of(value)`에 null을 넣으면 `NullPointerException`을 던지니 주의하자.
- `null`값도 허용하는 옵셔널을 만들려면 `Optional.ofNullable(value)` 를 사용하면 된다. **옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자.**
    - 옵셔널을 도입한 취지를 완전히 무시하는 행위다.

---

**컬렉션에서 최댓값을 구해 Optional<E>로 반환한다 - 스트림 버전**

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	return c.stream().max(Comparator.naturalOrder());
}
```

- 그렇다면 `null`을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준은 무엇일까?
- **옵셔널은 검사 예외와 취지가 비슷하다.**(아이템 71) 즉,반환값이 없을 수도 있음을 API 사용자에게 명확하게 알려주어야 한다.
    - 비검사 예외를 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 끔찍한 결과가 나온다.

---

### 메서드가 옵셔널을 반환한다면 클라이언트가 값을 받지 못했을 때 취할 행동

1. **기본값 설정**
    
    옵셔널 활용 1 - 기본값을 정해둘 수 있다.
    
    ```java
    String lastWordInLexicon = max(words).orElse("단어 없음...");
    ```
    

---

1. **상황에 맞는 예외 던지기**
    
    옵셔널 활용 2 - 원하는 예외를 던질 수 있다.
    

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

- 실제 예외가 아니라 예외 팩터리를 건넨 것에 주목하자.
    - 이렇게 하면 예외가 실제로 발생하지 않는 한 예외 생성 비용은 발생하지 않는다.

---

1. **항상 값이 채워져 있다고 가정한다.**
    
    옵셔널 활용 3 - 항상 값이 채워져 있다고 가정
    

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

- 옵셔널에 항상 값이 없다면  NoSuchElementException이 발생

---

- 이것 이외에 filter, map, flatMap, ifPresent 메서드가 있다.

**isPresent**

- 안전 밸브 역할의 메서드로, 옵셔널이 채워져 있으면  true, 비어있으면 false 를 반환한다.
- 신중히 사용해야 하는데, 실제로 isPresent를 쓴 코드 중 상당수는 앞서 언급한 메서드들로 대체할 수 있으며, 대체하는 것이 더 짧고 명확하고 용법에 맞는 코드가 된다.

```java
//부모 프로세스의 프로세스 ID를 출력하거나, 부모가 없다면 "N/A"를 출력
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID:" + (parentProcess.**isPresent**() ?
	String.valueOf(parentProcess.get().pid()) : "N/A"));
```

- 이 코드는 optional의 **map**을 사용하여 다음처럼 다듬을 수 있다.

**map**

```java
System.out.println("부모 PID:" +
	ph.parent()**.map(h-> String.valueOf(h.pid)))**.orElse("N/A");
```

스트림을 사용한다면 옵셔널들을 Stream<Optional>로 받아서, 그 중 채워진 옵셔널들에서 값을 뽑아 Stream에 건네 담아 처리하는 경우가 종종 있다.

**filter**

```java
streamOfOptionals.filter(Optional::isPresnet).map(Optional::get)
```

optional에 값이 있다면, 그 값을 꺼내서 스트림에 매핑한다.

**flatMap**

```java
streamOfOptionals.flatMap(Optional::stream)
```

- Java 9에서 Optional에 stram()이 추가되었는데, 이 메서드는 Optional을 Stream으로 변환해주는 어댑터이다.
- 옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환한다.

### Optional를 사용하면 안되는 이유

- 컨테이너 타입(컬렉션, 스트림, 배열, 옵셔널)은 옵셔널로 감싸면 안된다.
- 빈 Optional<List>를 반환하기 보다는 빈 List를 반환하는게 좋다. (아이템 54번)
    - 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다.

### Optional을 사용해야하는 경우

- **결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면, Optional<T>를 반환한다.**

**성능 이슈 발생**

- Optional도 엄연히 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야 하니 한 단계를 더 거치는 셈이다.
    - 그래서 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다.
- 박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수 밖에 없다. 값을 두 겹이나 감싸기 때문이다.
    - 그래서 자바 API 설계자들은 int, long, double 전용 옵셔널 클래스들을 준비해놨다.
    - `OptionalInt`, `OptionalLong`, `OptionalDouble`
    - 이 옵셔널들도 기본 Optional이 제공하는 메서드들을 거의 다 제공한다.
    - **대체제가 있는 상황에서 박싱된 기본 타입을 담은 옵셔널을 반환하게하지 마라.**
    - 상대적으로 `Boolean`, `Byte`, `Character`, `Short`, `Float`은 예외일 수 있다.

### 절대 하지 말아야 할 것

- 옵셔널을 맵의 값으로 사용하면 절대 안된다. 만약 이렇게 한다면 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 된다.
    - 키 자체가 없는 경우
    - 키는 있지만 그 키가 속이 빈 옵셔널인 경우
    - 쓸데없이 복잡성만 높여서 혼란과 오류 가능성을 키운다.
- **옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.**

> **핵심정리**
- 값을 반환하지 못할 가능성이 있는 메서드라면 옵셔널을 반환해야 하는 상황일 수 있다.
- 하지만 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면, null을 반환하거나 예외를 던지는 편이 나을 수 있다.
- 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.
> 

---

## [Item 56] 공개된 API 요소에는 항상 문서화 주석을 작성하라.

> 자바독은 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.
> 

**API를 올바로 문서화하려면 공개된 모든 클래서, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.**

- 문서가 잘 갖춰지지 않은 API는 쓰기 헷갈려서 오류의 원인이 되기 쉽다.
- 문서화 주석이 없다면 자바독도 그저 공개 API 요소들의 ‘선언’만 나열해주는게 전부이다.

**메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야한다.**

- 상속용으로 설계된 클래스의 메서드가 아니라면 그 메서드가 어떻게 동작하는지가 아니라 무엇을 하는지를 기술해야 한다.
- 문서화 주석에는 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야 한다.
- 또한 메서드가 성공적으로 수행된 후에 만족해야 하는 사후 조건도 모두 나열해야한다.

**전제 조건과 사후 조건 뿐만 아니라 부작용도 문서화해야한다.**

**관례상 @param 태그와 @return 태그의 설명은 해당 매개변수가 뜻하는 값이나 반환값을 설명하는 명사구를 쓴다.**

`**@throws` 절에 사용한 `{@code}` 태그는 두 가지 효과를 가진다.**

1. 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
2. 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다.

**영문 문서화 주석에서 쓴 `this list` 라는 단어에 주목하자**

- 관례상, 인스턴스 메서드의 문서화 주석에 쓰인 `this` 는 호출된 메서드가 자리하는 객체를 가리킨다.

**자바독 유틸리티는 문서화 주석을 HTML로 변환하므로 문서화 주석 안의 HTML 요소들이 최종 HTML 문서에 반영된다.**

- 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
- 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.
- 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든 스데 안전 수준을 반드시 API 설명에 포함해야 한다.

> **핵심정리**
-문서화 주석은 여러분들의 API를 문서화하는 가장 훌륭하고 효과적인 방법이다.
- 공개 API라면 빠짐없이 설명을 달아야 한다.
- 문서화 주석에 임의의 HTML 태그를 사용할 수 있다.
단, HTML 메타문자는 특별하게 취급해야 한다.
>
