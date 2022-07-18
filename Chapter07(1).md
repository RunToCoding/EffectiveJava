# 7장. 람다와 스트림


<aside>
✏️ Java 8에서 추가된 "함수형 인터페이스, 람다, 메서드 참조, 스트림 API" 기능을 효과적으로 사용하는 방법

</aside>

<br><br>


## *아이템 42*. 익명 클래스보다는 람다를 사용하라 (p254)

### 함수 객체(function object)

- 추상 메서드 하나만 담은 인터페이스 **(⇒함수형 인터페이스)** 의 인스턴스

- 이전에는 주로 **익명 클래스** *(아이템24)* 를 이용해 만들었음
    
    ```java
    // 문자열을 길이순으로 정렬하는 코드
    Collections.sort(words, new Comparator<String>() {
    	public int compare(String s1, String s2) {
    		return Integer.compare(s1.length(), s2.length());
    	}
    });
    ```
    

- 지금은 **람다식**(lambda expression, **람다**)을 사용해서 만들 수 있음
    
    ```java
    Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
    ```
    
    → 람다, 매개변수, 반환값의 **타입은 컴파일러가 추론**
    

- 람다 자리에 **비교자 생성 메서드** 사용 *(아이템 14, 43)*
    
    ```java
    Collections.sort(words, comparingInt(String::length));
    ```
    
    - `::`**(더블콜론)**: `인스턴스 :: 메서드(또는 new)` 형태로 람다식에서만 사용 가능

- Java 8에서 `List` 인터페이스에 추가된 `sort` 메서드 사용
    
    ```java
    words.sort(comparingInt(String::length));
    ```
    
- 함수 객체의 실용적 사용 가능
    - 상수별 클래스 몸체 구현 방식 *(아이템34)*
        
        ```java
        public enum Operation {
        	PLUS("+")   { public double apply(double x, double y){ return x + y; } },
        	MINUS("-")  { public double apply(double x, double y){ return x - y; } },
        	TIMES("*")  { public double apply(double x, double y){ return x * y; } },
        	DIVIDE("/") { public double apply(double x, double y){ return x / y; } };
        
        	private final String symbol;
        
        	Operation(String symbol) { this.symbol = symbol; }
        
        	@Override public String toString() { return symbol; }
        	public abstract double apply(double x, double y);
        }
        ```
        
    
    - 각 열거 타입 **상수의 동작을 람다로 구현**해 생성자에 넘기고, 생성자는 **람다를 인스턴스 필드로 저장** → **메서드에서 람다 호출**
        
        ```java
        public enum Operation {
        	PLUS("+", (x, y) -> x + y),
        	MINUS("-", (x, y) -> x - y),
        	TIMES("*", (x, y) -> x * y),
        	DIVIDE("/", (x, y) -> x / y);
        
        	private final String symbol;
        	private final DoubleBinaryOperator op;
        
        	Operation(String symbol, DoubleBinaryOperator op) {
        		this.symbol = symbol;
        		this.op = op;
        	}
        
        	@Override public String toString() { return symbol; }
        
        	public double apply(double x, double y) {
        		return op.applyAsDouble(x, y);
        	}
        }
        ```
        
- 람다를 사용할 수 없는 경우?
    - 람다는 **함수형 인터페이스**에서만 사용 가능
    ⇒ **추상 클래스의 인터페이스** or **추상 메서드가 여러 개인 인터페이스**의 인스턴스를 만들 때는 **익명클래스** 사용
    - 자기 자신을 참조할 때
        - **람다**의 `this`는 **바깥 인스턴스**, **익명 클래스**의 `this`는 익명 클래스의 **인스턴스 자신**
        
<br>
## *아이템 43*. 람다보다는 메서드 참조를 사용하라 (p259)

### 메서드 참조(method reference)

- 함수 객체를 람다보다도 더 간결하게 만드는 방법

- 람다를 사용한 경우
    
    ```java
    map.merge(key, 1, (count, incr) -> count + incr);
    ```
    
    - `merge`: `[키, 값, 함수]`를 인수로 받아 키가 맵에 **없으면** 주어진 `{키, 값}` 쌍을 **그대로** 저장, 키가 맵에 **있으면** **함수를 현재 값과 주어진 값에 적용**하여 `{키, 함수의 결과}` 쌍으로 **덮어씀**
    
- Java 8부터 모든 **기본 타입의 박싱 타입** 클래스(`Integer`, `Long`, …)는 정적 메서드 `sum` 제공
    
    ```java
    map.merge(key, 1, Integer::sum);
    ```
    

- 람다가 메서드 참조보다 간결한 경우? : **메서드와 람다가 같은 클래스**에 있을 때
    
    ```java
    // 메서드 참조
    public class GoshThisClassNameIsHomogeneous {
    	...
    	service.execute(GoshThisClassNameIsHomogeneous::action);
    	...
    }
    
    // 람다
    public class GoshThisClassNameIsHomogeneous {
    	...
    	service.execute(() -> action());
    	...
    ]
    ```
    
- 메서드 참조의 유형
    1. **정적 메서드 참조**
    2. **한정적(bound) 인스턴스 참조**: 수신 객체(receiving object; 참조 대상 인스턴스)를 특정
    3. **비한정적(unbound) 인스턴스 참조**: 수신 객체를 특정하지 않음
    4. **클래스 생성자**를 가리키는 메서드 참조
    5. **배열 생성자**를 가리키는 메서드 참조
    <br>
    | 메서드 참조 유형 | 예 | 같은 기능을 하는 람다 |
    | --- | --- | --- |
    | 정적 | Integer::parseInt | str -> Integer.parseInt(str) |
    | 한정적(인스턴스) | Instant.now()::isAfter | Instant then = Instant.now();
    t → then.isAfter(t) |
    | 비한정적(인스턴스) | String::toLowerCase | str -> str.toLowerCase() |
    | 클래스 생성자 | TreeMap<K,V>::new | () -> new TreeMap<K,V>() |
    | 배열 생성자 | int[]::new | len -> new int[len] |
<br>
## *아이템 44*. 표준 함수형 인터페이스를 사용하라 (p263)

- **상위 클래스의 기본 메서드를 재정의**해 원하는 동작을 구현하는 템플릿 메서드 패턴 
→ **람다**를 지원하면서 같은 효과의 **함수 객체를 매개변수로 받는 정적 팩터리나 생성자를 제공**하도록 대체
    - `LinkedHashMap`의 메서드 `removeEldestEntry` 재정의
        - `put` 메서드에서 `removeEldestEntry`를 호출하여 `true`가 반환되면 가장 오래된 원소 제거
        
        ```java
        protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        	return size() > 100;
        }
        ```
        
    
    - **함수형 인터페이스** 선언 (불필요한 인터페이스의 예시)
        
        ```java
        @FunctionalInterface interface EldestEntryRemoveFunction<K,V> {
        	boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
        }
        ```
        
    
    - **표준 인터페이스**인 `BiPredicate<Map<K,V>, Map.Entry<K,V>>` 사용 가능
        
         
        
<br>
### 기본 함수형 인터페이스

| 인터페이스 | 함수 시그니처 | 예 |
| --- | --- | --- |
| UnaryOperator<T> | T apply(T t) | String::toLowerCase |
| BinaryOperator<T> | T apply T t1, T t2) | BigInteger::add |
| Predicate<T> | boolean test(T t) | Collection::isEmpty |
| Function<T,R> | R apply(T t) | Arrays::asList |
| Supplier<T> | T get() | Instant::now |
| Consumer<T> | void accept(T t) | System.out::print |

⇒ 변형은 p265 참고!
<br>
- 표준 함수형 인터페이스를 사용하지 않고 **직접 작성**해야 하는 경우?
    1. 필요한 용도에 맞는 게 없는 경우
    2. 같은 구조가 있더라도 다음과 같은 경우
        - 자주 쓰이며, **이름** 자체가 **용도를 명확히 설명**
        - 반드시 따라야 하는 **규약** 존재
        - 유용한 **디폴트 메서드** 제공

- 직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용할 것
    - **람다용으로 설계**된 것임을 명시
    - **추상 메서드를 오직 하나만** 가지고 있어야 컴파일됨
    - 유지보수 과정에서 실수로 **메서드를 추가하지 못하게** 막아줌
    
- ※ 함수형 인터페이스를 API에서 사용할 때 서로 **다른 함수형 인터페이스**를 **같은 위치의 인수**로 받는 메서드들을 **다중 정의**해서는 안 됨


## *아이템 45*. 스트림은 주의해서 사용하라 (p268)
## 아이템 45. 스트림은 주의해서 사용하라 (p268)
### 스트림 API

- **다량의 데이터 처리** 작업을 돕고자 추가됨
- 핵심 개념
    - **스트림**(stream): **데이터 원소의** 유한 혹은 무한 **시퀀스**
    - **스트림 파이프라인**(stream pipeline): 이 원소들로 수행하는 **연산 단계**
    
- 스트림 안의 데이터 원소들은 **객체 참조**나 **기본 타입** 값

- 스트림 파이프라인은 **소스 스트림**에서 시작해 **종단 연산**(terminal operation)으로 끝나며, 그 사이에 하나 이상의 **중간 연산**(intermediate operation)이 있을 수 있음
- 스트림 파이프라인은 **지연 평가**(lazy evoluation)됨: **평가는 종단 연산이 호출될 때** 이루어지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않음

- 스트림 API는 메서드 연쇄를 지원하는 **플루언트 API(fluent API)** 
⇒ 모든 호출을 연결해 단 **하나의 표현식**으로 완성 가능
- 스트림 파이프라인은 **순차적**으로 수행됨

- 사전 파일에서 단어를 읽어 특정값보다 원소 수가 많은 아나그램(anagram) 그룹 (e.g. {”aelpst”: “petals”, “staple”,…}…)을 출력하는 예시
    - 스트림을 사용하지 않은 경우
        
        ```java
        public class Anagrams {
        	public static void main(String[] args) throws IOExceptions {
        		File dictionary = new File(args[0]);
        		int minGroupSize = Integer.parseInt(args[1]);
        
        		Map<String, Set<String>> groups = new HashMap<>();
        		try (Scanner s = new Scanner(dictionary)) {
        			while (s.hasNext()) {
        				String word = s.next();
        				groups.computeIfAbsent(alphabetize(word), 
        							(unused) -> new TreeSet<>()).add(word);
        			}
        		}
        		for (Set<String> group : groups.values()) {
        			if (group.size() >= minGroupSize) 
        				System.out.println(group.size() + ": " + group);
        		}
        	}
        
        	private static String alphabetize(String s) {
        		char[] a = s.toCharArray();
        		Arrays.sort(a);
        		return new String(a);
        	}
        }
        ```
        
    - 스트림을 과하게 사용한 경우 ⇒ 읽거나 유지보수하기 어려움
        
        ```java
        public class Anagrams {
        	public static void main(String[] args) throws IOExceptions {
        		File dictionary = new File(args[0]);
        		int minGroupSize = Integer.parseInt(args[1]);
        
        		try (Stream<String> words = Files.lines(dictionary)) {
        			words.collect(
        				groupingBy(word -> word.chars().sorted()
        							.collect(StringBuilder::new),
        								(sb, c) -> sb.append((char) c),
        								StringBuilder::append).toString()))
        			.values().stream()
        			.filter(group -> group.size() >= minGroupSize)
        			.map(group -> group.size() + ": " + group)
        			.forEach(System.out::println)
        		}
        	}
        	private static String alphabetize(String s) { ...	}
        }
        ```
        
    - 스트림을 적절히 활용한 경우
        
        ```java
        public class Anagrams {
        	public static void main(String[] args) throws IOException {
        		Path dictionary = Paths.get(args[0]);
        		int minGroupSize = Integer.parseInt(args[1]);
        		
        		try (Stream<String> words = Files.lines(dictionary)) {
        			words.collect(groupingBy(word -> alphabetize(word)))
        					.values().stream()
        					.filter(group -> group.size() >= minGroupSize)
        					.forEach(g -> System.out.println(g.size() + ": " + g));
        		}
        	}
        	private static String alphabetize(String s) { ...	}
        }
        ```
        
    
- `char`용 스트림은 지원하지 않기 때문에, 명시적 형변환을 이용해 사용할 수는 있지만 이때는 스트림을 삼가는 편이 낫다.

- **스트림** vs **반복 코드**
    - **스트림**은 계산을 **함수 객체**(주로 람다나 메서드 참조)로 표현하고, **반복 코드**에서는 **코드 블록**을 사용해 표현
    
    - 함수 객체로는 할 수 없지만 **코드 블록으로는 할 수 있는 일** ⇒ **반복 코드** 사용!
        - 범위 안의 지역변수를 읽고 수정
        - `return`, `break`, `continue` 사용
        - 메서드 선언에 명시된 검사 예외 던지기
        
    - **스트림**에 적합한 일
        - 원소들의 시퀀스를 **일관되게 변환**
        - 원소드의 시퀀스를 **필터링**
        - 원소들의 시퀀스를 **하나의 연산을 사용해 결합**
        - 원소들의 시퀀스를 **컬렉션에 모으기**
        - 원소들의 시퀀스에서 **특정 조건을 만족하는 원소 찾기**
        
    - 스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 **원래 값을 잃는 구조**
    ⇒ 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 **각 단계에서의 값들에 동시에 접근하기는 어려운 경우**에는 스트림으로 처리하기 어려움
    
    - 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라
