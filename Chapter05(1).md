# 5장. 제네릭

> 컬렉션(List, Set, Queue, Map)이 담을 수 있는 타입을 컴파일러에 알려주어 컴파일러가 알아서 형변환 코드를 추가할 수 있도록 함 ⇒ 잘못된 타입의 객체를 넣으려는 시도 차단
> 

<aside>
✏️ 제네릭의 이점을 최대화하고 단점을 최소화하는 방법

</aside>

---

## 아이템 26. 로 타입은 사용하지 말라 (p154)

- **제네릭 클래스/인터페이스**: 클래스/인터페이스 선언에 타입 매개변수가 쓰인 경우 
e.g. `List<E>`
- **제네릭 타입**: 제네릭 클래스, 제네릭 인터페이스

각각의 제네릭 타입은 일련의 매개변수화 타입(parameterized type) 정의

e.g. `List<String>` : 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입
              ⇒ String: 정규(formal) 타입 매개변수 E에 해당하는 실제(actual) 타입 매개변수

- **로 타입(raw type)**: 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때
e.g. `List<E>`의 로 타입은 `List`
    - 로 타입을 사용하면 잘못된 타입의 객체를 넣고 꺼낼 때까지 오류 발견 불가
    - 타입 매개변수를 명시하면 잘못된 인스턴스를 넣을 때 컴파일 오류 발생
    

```java
private final Collection stamps = ...;          //컬렉션의 로 타입
private final Collection<Stamp> stamps = ...;   //매개변수화된 컬렉션 타입
```

- **임의의 객체를 허용하는 매개변수화 타입**(e.g. `List<Object>`) 사용은 허용

```java
public static void main(String[] args) {
		List<String> strings = new ArrayList<>();
		unsafeAdd(strings, Integer.valueOf(42));
		String s = strings.get(0);  // 1의 경우 Integer를 String으로 형변환 시도하여 오류
}

// 1
private static void unsafeAdd(**List** list, Object o) {
		list.add(o);
}

// 2 - 컴파일 오류
private static void unsafeAdd(**List<Object>** list, Object o) {
		list.add(o);
}
```

- **비한정적 와일드카드 타입(unbounded wildcard type)**: 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때 사용
e.g. `Set<E>`의 비한정적 와일드 카드 타입: `Set<?>`

- **로 타입을 쓰면 안 되는 규칙의 예외**
    1. **class 리터럴**
        
        class 리터럴: `클래스이름.class`로 class에 접근하는 방법
        e.g. `List.class`, `String[].class`, `int.class` (o)
               `List<String>.class`, `List<?>.class` (x)
        
    2. `**instanceof` 연산자**
        
        런타임에는 제네릭 타입 정보가 지워지므로 `instanceof` 연산자는 매개변수화 타입에는 적용할 수 없음
        
        비한정적 와일드카드 타입은 사용 가능하나, 로 타입과 똑같이 동작하므로 로 타입 사용이 깔끔함
        
        ```java
        if (o instanceof **Set**) {
        		**Set<?>** s = **(Set<?>)** o;
        		...
        }
        ```
        
        ⇒ `o`의 타입이 `Set`임을 확인한 후 와일드카드 타입인 `Set<?>`로 형변환해야 함
        

## 아이템 27. 비검사 경고를 제거하라 (p161)

> `warning : [unchecked]` - casting 할 때 검사를 하지 않았다고 뜨는 경고
런타임에 `ClassCastException`을 일으킬 수 있는 잠재적 가능성
> 
- e.g.
    
    ```java
    Set<Lark> exaltation = new HashSet();
    ```
    
    → javac 명령줄 인수에 `-Xlint:uncheck` 옵션 추가하여 컴파일
    
    ```java
    Venery.java:4: warning: [unchecked] unchecked conversion
    				Set<Lark> exaltation = new HashSet();
                                   ^
    	required: Set<Lark>
    	found:    HashSet
    ```
    
    ⇒ 컴파일러가 알려준 대로 수정하거나 자바 7부터 지원하는 다이아몬드 연산자(`<>`)로 해결
    
    ```java
    Set<Lark> exaltation = new HashSet<>();
    ```
    
- 할 수 있는 한 모든 비검사 경고를 제거하라
- 모두 제거하면 타입 안정성이 보장됨
- 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarning(”unchecked”)` 애너테이션을 달아 경고를 숨길 수 있음 
(가능한 한 좁은 범위에 적용하자)
    - 애너테이션은 선언에만 달 수 있기 때문에 범위가 큰 경우(메서드 전체) 지역변수를 선언하고 그 변수에 달아주는 것이 좋음
    - 경고를 무시해도 안전한 이유를 주석으로 남겨야 함
    

## 아이템 28. 배열보다는 리스트를 사용하라 (p164)

- **배열과 제네릭 타입의 차이점**
    1. **공변/불공변**
        - 배열 - 공변(convariant): 함께 변함
            
            `Sub`가 `Super`의 하위 타입이면 배열 `Sub[]`는 배열 `Super[]`의 하위 타입
            
        - 제네릭 - 불공변(invariant)
            
            서로 다른 타입 `Type1`과 `Type2`가 있을 때, `List<Type1>`은 `List<Type2>`의 하위 타입도 아니고 상위 타입도 아님
            
        
        ```java
        // 배열 사용 - 런타임 실패
        Object[] objectArray = new Long[1];
        objectArray[0] = "타입이 달라 넣을 수 없다.";
        
        // 제네릭 사용 - 컴파일 오류
        List<Object> ol = new ArrayList<Long>();
        ol.add("타입이 달라 넣을 수 없다.");
        ```
        
    2. **실체화(reify)**
    
    배열: 런타임에도 담기로 한 원소의 타입을 인지하고 확인
    
    제네릭: 타입 정보가 런타임에 소거
    
    ⇒ 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용 불가
         e.g. `new List<E>[]`, `new List<String>[]`, `new E[]` → 컴파일 오류
    
- 구현 예시
    
    ```java
    // 1. 제네릭을 쓰지 않고 구현
    public class Chooser {
    		private final Object[] choiceArray;
    		
    		public Chooser(Collection choices) {  // 타입이 다른 원소가 들어 있었다면
    				choiceArray = choices.toArray();
    		}
    
    		public Object choose() {  // 반환된 Object를 형변환할 때 오류 발생
    				Random rnd = ThreadLocalRandom.current();
    				return choiceArray[rnd.nextInt(choiceArray.length)];
    		}
    }
    
    // 2-1. 제네릭 적용
    public class Chooser**<T>** {
    		private final **T**[] choiceArray;
    		
    		public Chooser(Collection**<T>** choices) {
    				choiceArray = choices.toArray();   // T[] = Object[] => 컴파일 오류
    		}
    
    		...
    }
    
    // 2-2. 제네릭 적용 - 오류 수정
    public class Chooser**<T>** {
    		private final **T**[] choiceArray;
    		
    		public Chooser(Collection**<T>** choices) {
    				choiceArray = **(T[])** choices.toArray();   // 형변환 
    		}   // => T가 무슨 타입인지 알 수 없어 안전을 보장할 수 없다는 경고
    
    		...
    }
    
    // 3. 리스트 기반 - 타입 안전성 확보
    public class Chooser**<T>** {
    		private final **List<T>** choiceList;
    		
    		public Chooser(Collection**<T>** choices) {
    				choiceArray = **new ArrayList<>(choices)**;
    		}
    
    		public T choose() {
    				Random rnd = ThreadLocalRandom.current();
    				return **choiceList.get(rnd.nextInt(choiceList.size()))**;
    		}
    }
    ```
    

## 아이템 29. 이왕이면 제네릭 타입으로 만들라 (p170)

 

```java
// Object 기반 스택
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) throw new EmptyStackException();
				Object result = elements[--size];
				elements[size] = null;
        return result;
    }
    
    public boolean isEmpty() {...}
	  private void ensureCapacity() {...}
}
```

- 클래스 선언에 타입 매개변수 추가 (타입 이름은 보통 `E` 사용)

```java
// 제네릭 스택으로 가는 첫 단계 - 컴파일되지 않는다.
public class Stack**<E>** {
    private **E**[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new **E**[DEFAULT_INITIAL_CAPACITY];  
    }   // E와 같은 실체화 불가 타입으로는 배열을 만들 수 없음 (아이템 28)
    
    public void push(**E** e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public **E** pop() {
        if (size == 0) throw new EmptyStackException();
				**E** result = elements[--size];
				elements[size] = null;
        return result;
    }
    
    public boolean isEmpty() {...}
	  private void ensureCapacity() {...}
}
```

```java
// 방법 1-1 - 비검사 형변환 경고 발생
elements = **(E[])** new Object[DEFAULT_INITIAL_CAPACITY];

// 방법 1-2
**@SuppressWarnings("unchecked")**
public Stack() {
		elements = **(E[])** new Object[DEFAULT_INITIAL_CAPACITY];
}

// 방법 2-1 - **E** result = elements[--size]; 에서 형변환 오류
private **Object[]** elements;

// 방법 2-2 - 비검사 형변환 경고 발생
**E** result = **(E)** elements[--size];

// 방법 2-3
public **E** pop() {
        if (size == 0) throw new EmptyStackException();
				**@SuppressWarnings("unchecked") E** result = elements[--size];
				elements[size] = null;
        return result;
}
```

- **방법 1**: 코드가 짧고 가독성 좋음. 배열 생성 시 **단 한 번만 형변환**
- **방법 2**: 배열에서 **원소를 읽을 때마다 형변환**

⇒ 현업에서는 방법 1을 더 선호하지만, (`E`가 `Object`가 아닌 한) 배열의 **런타임 타입**이 **컴파일타임 타입**과 달라 **힙 오염(heap pollution**; 아이템 32**)**를 일으켜 방법 2를 고수하기도 함

<aside>
💭 위 Stack 예시가 “배열보다는 리스트를 우선하라”는 아이템 28과 모순되어 보임

사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아님
ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 함
HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 함

</aside>

- 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않음
e.g. `Stack<Object>`, `Stack<int[]>`, `Stack<List<String>>`, `Stack` 등 가능
기본 타입을 사용한 `Stack<int>`, `Stack<double>` 등은 사용할 수 없지만 **박싱된 기본 타입**(아이템 61)을 사용해 우회 가능
- 타입 매개변수에 제약을 두는 제네릭 타입도 있음
e.g. `java.util.concurrent.DelayQueue`는 `java.util.concurrent.Delayed`의 하위 타입만 받음
⇒ `DelayQueue` 자신과 `DelayQueue`를 사용하는 클라이언트는 `DelayQueue`의 원소에서 형변환 없이 곧바로 `Delayed` 클래스의 메서드 호출 가능
⇒ 이러한 타입 매개변수 `E`를 **한정적 타입 매개변수(bounded type parameter)**라 함
+) 모든 타입은 자기 자신의 하위 타입이므로 `DelayQueue<Delayed>`로 사용 가능