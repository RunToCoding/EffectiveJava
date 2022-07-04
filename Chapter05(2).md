# 5장. 제네릭

> 컬렉션(List, Set, Queue, Map)이 담을 수 있는 타입을 컴파일러에 알려주어 컴파일러가 알아서 형변환 코드를 추가할 수 있도록 함 ⇒ 잘못된 타입의 객체를 넣으려는 시도 차단
> 

<aside>
✏️ 제네릭의 이점을 최대화하고 단점을 최소화하는 방법

</aside>

---

## 아이템 30. 이왕이면 제네릭 메서드로 만들라
- 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다.
- 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.
- e.g. Collections의 '알고리즘' 메서드 (binarySearch, sort ...)
- 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환 해야하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기 쉽다.

### 제네릭 싱글턴 팩터리
- 요청한 타입 매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리이다.
- e.g. Collections.reverseOrder, Collections.emptySet
```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN''
}
```
- T가 어떤 타입이든 UnaryOperator<T>를 사용해도 타입 안전하다.<br/>
  → 비검사 형변환 경고를 숨겨도 되므로 @SuppressWarnings 애너테이션을 추가하자!

### 재귀적 타입 한정
- 자기 자신이 들어간 표현식을 사용하여 탕비 매개변수의 허용 범위를 한정한다. (드물게 사용)
- Comparable 인터페이스와 함께 쓰인다.
```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

## 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높여라
- 일련의 원소를 스택에 넣는 메서드를 추가해야 한다고 해보자.
```java
public void pushAll(Iterable<E> src) {
    for (E e : src) push(e);
}
```
그 다음 아래와 같이 클라이언트 코드를 호출하면 어떻게 될까?
```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
```
매개변수화 타입이 불공변이기 때문에 오류 메시지가 뜨게 된다.
```java
error : incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
    numberStack.pushAll(integers);
```
이를 해결 할 수 있는 방법은 **`한정적 와일드카드 타입`**을 사용하는 것이다. <br/>
한정적 와일드카드 타입을 사용하여 코드를 아래와 같이 수정해보았다.
```java
public void pushAll(Iterable<? extends  E> src) {
    for (E e : src) push(e);
}
```
와일드카드 타입 `Iterable<? extends  E>`는 `'E의 하위 타입의 Iterable'`이라는 뜻이며,
이전의 클라이언트 코드가 정상적으로 수행되는 것을 확인할 수 있다.

이번에는 pushAll와 짝을 이루는 popAll 메서드를 작성할 차례이다. 여기서도 와일드카드 타입을 사용해보자.
```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) dst.add(pop());
}
```
와일드카드 타입 `Collection<? super E>`는 `E의 상위 타입의 Collection`이라는 뜻이다.
이렇게 **와일드카드 타입을 적용하면 API 유연성이 극대화됨**을 알 수 있다.

### PECS 공식
- 펙스(PECS) : producer-extends, consumer-super
- i.e. 겟풋 원칙 : Get and Put Principle
- 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용하라.
- **와일드카드 타입을 써야할 때를 기억하는 데 도움이 되는 공식**이다.
- 생산자 매개변수에 와일드카드 타입 적용해보자.
  ```java
  public Chooser(Collection<? extends T> choices)
  ```
  - choices 컬렉션은 T 타입의 값을 **생산**하기만 하니, T를 **확장**하는 와일드카드 타입을 사용해 선언해야 한다.
- Comparable 및 Comparator은 언제나 소비자이므로, 일반적으로는 Comparable<? super E>, Comparator<? super E>를 사용하는 편이 낫다.

## 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라
- 가변인수(varargs) 메서드 (아이템 53)와 제네릭은 자바 5 때 함께 추가되었으나 함께 사용할 때 이슈가 있다.
  - 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어지는 데, 내부로 감춰야 했을 이 배열이 클라이언트에 노출하는 문제가 발생했다.<br/>
    → 그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.
- 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 타입 안정성이 깨지기 때문에 안전하지 않다.
  ```java
  static void dangerous(List<String>... StringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList;              // 힙 오염 발생
    String s = stringLists[0].get(0);  // ClassCastException
  }
  ```
  - 위 코드는 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면서 힙 오염이 발생했다.
  - 가변인수 메서들ㄹ 호출할 때 varargs 매개변수가 실체화 불가 타입으로 추론되면, 그 호출에 대해서도 경고를 낸다
    ```java
    warning : [unchecked] Possible heap pollution from parameterized varag type List<String>
    ```
- varargs 매개변수는 오류가 아닌 경고로 끝내는 이유는 무엇일까?
  - 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문
  - 그러나 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기는 방법은 비추천함.
    - 가독성을 떨어뜨리고, 진자 문제를 알려주는 경고마저 숨기기 때문이다.
  - 자바 7에서 추가된 `@Safewarargs` 애너테이션을 사용하여 경고를 숨기자!
    - 해당 애너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다.
- 안전한 제네릭 varargs 메서드는 어떤 것일까?
  - varargs 매개변수 배열에 아무것도 저장하지 않는다.
  - 그 배열 (혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

## 아이템 33. 타입 안전 이종 컨테이너를 고려하라
- 제네릭은 `ThreadLocal<T>, AtomicReference<T>` 등의 **단일 원소 컨테이너**에도 흔히 쓰인다.<br/>
  매개변수화되는 대상은 원소가 아닌 컨테이너 자신이며, 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수는 제한된다.
- 타입 수 제한을 해결하는 방법은? **`타입 안전 이종 컨테이너 패턴`**을 사용하자.

### 타입 안전 이종 컨테이너 패턴 (Type Safe Heterogeneous Container Pattern)
- 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다.
  <br/> → 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장한다.
- 각 타입의 Class 객체를 매개변수화한 키 역할로 사용하며, 이런 식으로 쓰이는 Class 객체를 **`타입 토큰`**이라 한다.
- 타입 안전 이종 컨테이너 패턴 - API
    ```java
    public class Favorites {
        public <T> void putFavorite(Class<T> type, T instance);
        public <T> T getFavorite(Class<T> type);
    }
    ```
- 타입 안전 이종 컨테이너 패턴 - 클라이언트
    ```java
    public static void main(String[] args) {
        Favorites f = new Favorites();
        
        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);
        
        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);
        
        System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName()); // 기대한 출력이 나옴. Java cafebabe Favorites
    }
    ```
- 타입 안전 이종 컨테이너 패턴 - 구현
    ```java
    public class Favorites {
        private Map<Class<?>, Object> favorites = new HashMap<>();
        
        public <T> void putFavorite(Class<T> type, T instance) {
            favorites.put(Objects.requireNonNull(type), instance);
        }
        
        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
        }
    }
    ```
    - 와일드카드 타입이 중첩되어 있어, 다양한 타입을 지원함
- Favorites 클래스 제약
  1. 악의적인 클라이언트가 Class 객체를 (제네릭이 아닌) 로타입으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다.
  2. 실체화 불가 타입에는 사용할 수 없다.

### 한정적 타입 토큰
- Favorites가 사용하는 타입 토큰은 비한정적이기에 어떤 Classr 객체든 받아들인다.<br/>
  만약, 허용하는 타입을 제한하고 싶다면 `한정적 타입 토큰`을 활용하자.
- `한정적 타입 토큰` : 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰
- e.g. 애너테이션 API (아이템 39)
  ```java
  static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) { // 인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰
    Class<?> annotationType = null; // 비한정적 타입 토큰
    try {
      annotationType = Class.forName(annotationTypeName); // 명시한 타입의 애너테이션이 대상 요소에 달려있다면 그 애너테이션 반환, 없다면 null 반환
    } catch (Exception ex) {
      throw new IllegalArgumentException(ex);
    }
  
    return element.getAnnotation(
      annotationType.asSubclass(Annotation.class); // asSubclass를 사용해 한정적 타입 토큰을 안전하게 형변환
    )
  }
  ```

<pre>
컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.
Clss를 키로 쓰며, Class 객체를 타입 토큰이라 한다.
또한, 직접 구현한 키 타입도 쓸 수 있다.
</pre>
