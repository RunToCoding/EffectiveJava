# 2장. 객채 생성과 파괴

<pre>
<b>🔥 이번 장을 통해 배울 수 있는 내용은 아래와 같다.</b>
1. 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법
2. 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법
3. 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령
</pre>

---

## 아이템 01. 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩터리 메서드 장점

#### 1. 이름을 가질 수 있다.
생성자인 **`BigInt(int, int, Random)`** 과 정적 팩터리 메서드인 **`BigInteger.probablePrime`** 중 어느 쪽이 `'값이 소수인 BitInteger를 반환한다'`는 의미를 더 잘 설명하는지 생각해보자.
정적 팩터리 메서드가 더 의미가 잘 설명한다고 생각할 것이다. 이렇듯 정적 팩터리 메서드는 이름을 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.

#### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
정적 팩터리 메서드는 언제 어느 인스턴스를 살아 있게 할질를 철저히 통제할 수 있다. 이런 클래스를 `인터턴스 통제 클래스`라 한다.

<pre>
<b>🔎 그렇다면 인스턴스를 통제하는 이유는 무엇일까?</b>
인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 반들 수도 있다.
또한, 인스턴스가 단 하나뿐임을 보장할 수 있다.
</pre>

#### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
정적 팩터리 메서드는 반환할 객체의 클래스를 자유롭게 선택할 수 있는 `'엄청난 유연성'`을 가졌다.
API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.

#### 4. 입력 매개 변수에 따라 매번 다른 클래스의 반환을 할 수 있다.
반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

대표적으로 `EnumSet` 클래스의 경우 원소가 64개 이하면 long 변수 하나로 관리하는 `ReqularEnumSet`을 반환하고 65개 이상이면 long 배열로 관리하는 `JumboEnumSet`을 반환한다.

#### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
이러한 유연함은 서비스 제공자 프레임워크를 만드는 근간이 되며, **대표적인 서비스 제공자 프레임워크**  로는 **`JDBC`** 가 있다.

<pre>
<b>🔎 서비스 제공자 프레임워크의 핵심 컴포넌트</b>
1. 서비스 인터페이스
2. 제공자 등록 API
3. 서비스 접근 API
+ 4. 서비스 제공자 인터페이스 (종종 쓰임)

<b>🔎 JDBC 핵심 컴포넌트 구성은 어떻게 될까?</b>
1. Connection - 서비스 인터페이스 역할
2. DriverManager.registerDriver - 제공자 등록 API 역할
3. DriverManager.getConnection - 서비스 접근 API 역할
4. Driver - 서비스 제공자 인터페이스 역할
</pre>

### 정적 팩터리 메서드 단점

#### 1. 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
상속을 하려면 `public`이나 `protected` 생성자가 필요하다.
또한, 컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다.

#### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
API 문서를 잘 써놓고 메서드 이름도 널리 알려진 규약에 따라 짓는 식으로 문제를 완화해줘야 한다.

### 정적 팩터리 메서드에서 흔히 사용하는 명명 방식
- `from` : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드<br/>
  ```Date d = Date.from(instant);```
- `of` : 여러 매개변수를 받아 적합한 타입의 인스턴스로 반환하는 집계 메서드<br/>
  ```Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);```
- `valueOf` : from과 of의 더 자세한 버전<br/>
  ```BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);```
- `instance 혹은 getInstance` : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.<br/>
  ```StackWalker luke = StackWalker.getInstance(options);```
- `create 혹은 newInstance` : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.<br/>
  ```Object newArray = Array.newInstance(classObject, arrayLen);```
- `getType` : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.<br/>
  ```FileStore fs = Files.getFileStore(path);```
- `newType` : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.<br/>
  ```BufferedReader br = Files.newBuffereReader(path);```
- `type` : getType과 newType의 간결한 버전<br/>
  ```List<Complaint> litany = Collections.list(legacyLitany);```

<br/>
<pre>
<b>📌 핵심정리</b>
정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.
그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제종하던 습관이 있다면 고치자.
</pre>

---

## 아이템 02. 생성자에 매개변수가 많다면 빌더를 고려하라

### 점층적 생성자 패턴 (Telescoping Constructor Pattern)
필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자, ... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘리는 방식이다.

점층적 생성자 패턴은 매개변수 개수가 많아지면 클라이언트 **`코드를 작성하거나 읽기 어렵다`** 는 단점이 있다.

<details>
<summary>점층적 생성자 패턴 - 확장하기 어렵다!</summary>

```java
public class NutritionFacts { 
    private final int servingSize;  // 필수
    private final int servings;     // 필수 
    private final int calories;     // 선택
    private final int fat;          // 선택
    private final int sodium;       // 선택
    private final int carbohydrate; // 선택
    
    public NutritionFact (int servingSize, int servings) {
        this (servingSize, servings, 0);
    }
    
    public NutritionFacts (int servingSize, int servings, int calories) {
        this (servingSize, servings, calories, 0);
    }
    
    ...
    
}
```
</details>

### 자바빈즈 패턴 (JavaBeans Pattern)
매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.

<details>
<summary>자바빈즈 패턴 - 일관성이 깨지고, 불변으로 만들 수 없다.</summary>

```java
public class NutritionFacts { 
    // 기본값이 있다면, 매개변수들을 기본값으로 초기화된다.
    private final int servingSize  = -1;  // 필수, 기본값 없음
    private final int servings     = -1;  // 필수, 기본값 없음
    private final int calories     = 0;
    private final int fat          = 0;
    private final int sodium       = 0;
    private final int carbohydrate = 0;
    
    public NutritionFact () {}
    
    // 세터 메서드들
    public void setServingSize(int val) { servingSize = val;}
    public void setServings(int val) { servings = val;}
    public void setCalories(int val) { calories = val;}
    public void setFat(int val) { fat = val;}
    public void setSodium(int val) { sodium = val;}
    public void setCarbohydrate(int val) { carbohydrate = val;}
    
}
```

```java
NutritionFacts cocaCola = new NutritionFacts();
// 원하는 매개변수의 값만 설정
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
</details>

#### 자바빈즈 패턴에서는 객체 하나를 만드려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되지 전까지는 일관성이 무너진 상태에 놓이게 된다.
그렇기에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드 안정성을 얻으려면 프로그래머가 추가 작업을 해줘야 한다.


### 빌더 패턴 (Builder Pattern)
클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자 (혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다.
그런 다음 빌더 객체가 제공하는 일종의 메서드들로 원하는 선택 매개변수들을 설정한다.

빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는 게 보통이다.

<details>
<summary>빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다.</summary>

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
  
    public static class Builder { 
        // 필수 매개변수
        private final int servingSize;
        private final int servings;
    
        // 선택 매개변수
        private final int calories;
        private final int fat;
        private final int sodium;
        private final int carbohydrate;
        
        // 필수 매개변수만 사용
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        // 선택 매개변수 메서드
        public Builder calories(int val) {
            calories = val;
            return this;
        }
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
</details>

빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 이러한 방식을 플루언트(fluent) API 혹은 메서드 연쇄라고 한다.

#### 빌더 패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수를 흉내낸 것이다.
그렇기에 아래 클라이언트 코드와 같이 쓰기와 읽기가 쉽다.

<details>
<summary>클라이언트 코드</summary>

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(35).carbohydrate(27).build();
```
</details>


#### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.
추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

<details>
<summary>계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴</summary>

```java
public abstract class Pizza {
  public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
  final Set<Topping> toppings;

  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    // 피자 생성 시 "Topping"은 필수로 들어가야 한다.
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }

    abstract Pizza build();

    // 하위 클래스는 이 메서드를 재정의(overriding)하여
    // "this"를 반환하도록 해야 한다.
    // self 타입이 없는 바자를 위한 우회 방법이며,
    // 시뮬레이트한 셀프 타입 (simulated self-type) 관용구라 한다.
    protected abstract T self();
  }

  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone(); // 아이템 50 참조
  }
}
```
</details>

하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.
이렇게 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌,
그 하위 타입을 반환하는 기능을 **`공변환 타이핑`** 이라 한다.

<details>
<summary>하위 클래스인 뉴욕 피자</summary>

```java
public class NyPizza extends Pizza {
  public enum Size { SMALL, MEDIUM, LARGE }
  private final Size size;

  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;

    // 피자 크기는 필수 매개변수
    public Builder(Size size) {
      this.size = Objects.requireNonNull(size);
    }

    @Override public NyPizza build() {
      return new NyPizza(this);
    }

    @Override protected Builder self() { return this; }
  }

  private NyPizza(Builder builder) {
    super(builder);
    size = builder.size;
  }

  @Override public String toString() {
    return toppings + "로 토핑한 뉴욕 피자";
  }
}
```
</details>

빌더 패턴은 상당히 유연하다. 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.
특정 필드는 빌더가 알아서 채우도록 할 수도 있다. 빌더 생성 비용이 크지는 않지만 성능이 민감한 상황에서는 문제가 될 수 있다.
그렇기에 매개변수가 4개 이상 되어야 값어치를 한다.
하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있으므로 **빌더 패턴을 사용하는 습관을 들이도록 하자.**

<br/>
<pre>
<b>📌 핵심정리</b>
생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.
매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.
빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
</pre>

---

## 아이템 03. private 생성자나 열거 타입으로 싱글턴입을 보증하라

### 싱글턴 (Singleton)
싱글턴은 인스턴스를 오직 하나만 생성할 수 있는 클래스이다.

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.

### public static final 필드 방식의 싱글턴

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {...}
  public void leaveTheBuilding() {...}
}
```

private 생성자는 public static 필드인 `Elvis.INSTANCE`를 초기화할 때 딱 한 번만 호출된다.
그렇기에 인스턴스가 전체 시스템에서 하나뿐임이 보장되며, 클라이언트는 손 쓸 방법이 없다.

#### 장점
- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 간결하다.

### 정적 팩터리 방식의 싱글턴

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {...}
  public static Elvis getInstance() { return INSTANCE; }
  
  public void leaveTheBuilding() {...}
}
```

`Elvis.getInstance`는 항상 같은 객체의 참조를 반환하므로 인스턴스가 전체 시스템에서 하나뿐임을 보장한다.

#### 장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

#### pubilc static final 필드 방식과 정적 팩터리 방식의 공통적인 단점
- 권한이 있는 클라이언트는 `리플렉션 API`인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.
이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 해야 한다.

- 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readReslove 메서드를 제공해야 한다.
이렇게 하지 않으면 **직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.**

### 열거 타입 방식의 싱글턴

```java
public enum Elvis {
  INSTANCE;
  
  public void leaveTheBuilding() {...}
}
```

public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있고,
심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽하게 막아준다.

**대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

단, 만드려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

---

## 아이템 04. 인스턴스화를 막으려거든 private 생성자를 사용하라
정적 멤버만 담은 유틸리티 클래스는 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어 준다.
실제로 공개된 API들에서도 이처럼 의도치 않게 인스턴스화할 수 있게 된 클래스가 종종 목격되곤 한다.

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없으며, **private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**

```java
public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다. (인스턴스화 방지용)
    private UtilityClass() {
        throw new AssertionError();
    }
    ...
}
```

이 방식은 상속을 불가능하게 하는 효과도 있다. 
모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데,
이를 `private`으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버린다.

---

## 아이템 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

대부분의 클래스가 하나 이상의 자원에 의존한다. 이런 클래스를 정적 유틸리티 클래스로 구현하면, 유연하지 않고 테스트하기 어렵다.
그렇기에 사용하는 자원에 따라 동작이 달라지는 클래스에는 적합하지 않다.

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChekcer() {} // 객체 생성 방지
    
    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String type) {...}
}
```

싱글톤으로 구현한 경우도 마찬가지이다.

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChekcer(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String type) {...}
}
```
그렇다면 어떠한 방식을 사용해야 할까?

**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식** 을 사용하면 된다.

```java
public class SpellChecker {
    private static final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requreNonNull(dictionary);
    }
    
    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String type) {...}
}
```
- 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 작동한다.
- 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.
- 의존 객체 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용할 수 있다.

더 나아가 `생성자에 자원 팩터리를 넘겨주는 방식`으로 변형할 수도 있다.

팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.
즉, `팩터리 메서드 패턴`을 구현한 것이다.
자바 8에서 등장한 **`Supplier<T> 인터페이스`** 가 팩터리를 표현한 완벽한 예이다.

<br/>
<pre>
<b>📌 핵심정리</b>
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면
싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
이 자원들은 클래스가 직접 만들게 해서도 안된다.
대신 필요한 자원 혹은 그 자원을 만들어주는 팩터리를 생성자 혹은 정적 팩터리나 빌더에 넘겨주자.
<b>의존 객체 주입</b>이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.
</pre>

---

## 아이템 06. 불필요한 객체 생성을 피하라
반복문이나 빈번히 호출되는 메서드 안에서 생성자로 문자열을 만들어내는 경우 쓸데없는 String 인스턴스가 수백만 개 만들어질 수도 있다. 
하나의 String 인스턴스를 사용하기 위해서는 **`문자열 리터럴`** 을 사용하면 된다.

```java
// AS-IS : String 생성자 사용
String s = new String("banana");
// TO-BE : String 리터럴 사용
String s = "banana";
```

생성자 대신 **정적 팩터리 메서드를 제공하는 불변 클래스** 에서는 **`정적 팩터리 메서드를 사용`** 해 불필요한 객체 생성을 피할 수 있다.
예를 들면, Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩터리 메서드를 사용하는 것이 좋다.
생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않기 때문이다.

생성 비용이 아주 비싼 객체인 경우, **`캐싱하여 재사용`** 하는 것이 좋다.
```java
// AS-IS
// 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서
// 곧바로 가비지 컬렉션 대상이 된다.
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
// TO-BE
// Pattern 인스턴스를 클래스 초기화 (정적 초기화) 과정에서
// 직접 생성해 캐싱해두고, 메서드가 호출될 때마다 재사용한다.
private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
```

불필요한 객체를 만들어내는 또 다른 예로는 `오토박싱(auto boxing)`이 있다.

오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.

```java
// AS-IS
// 반복문 수행 과정에서 불필요한 Long 인스턴스가 약 231개나 만들어진다.
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    
    return sum;
}
// TO-BE
private static long sum() {
    long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i;

    return sum;
}
```

위 코드에서 타입을 Long에서 long으로 변경하면 6.3초에서 0.59초로 빨라진다. (책 34p)
그러므로 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

---

## 아이템 07. 다 쓴 객체 참조를 해제하라

### 메모리 누수 주범 1 - 자기가 메모리를 직접 관리하는 클래스
**`자기가 메모리를 직접 관리하는 클래스`** 라면 항시 메모리 누수에 주의해야 한다.
그 예로 `Stack` 클래스를 들 수 있다. 아래 코드에서 **`메모리 누수가 일어나는 위치`** 는 어디인가?
```java
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
        return elements[--size];
    }
    
    // 배열 크기를 늘리는 메서드
    private void ensureCapacity() {...}
}
```
이 코드에서는 스택이 줄어들 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.
즉, 다 쓴 참조를 여전히 가지고 있으며 이러한 경우 가비지 컬렉터는 그 객체뿐만 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못한다.

해법은 간단하다. 해당 참조를 다 썼을 때 null 처리 (참조 해제)하면 된다.
```java
public Object pop() {
    if (size == 0) throw new EmptyStackException();
    
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

null 처리를 통해 다 쓴 참조 해제하는 방법은 예외적인 경우여야 한다.
가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.
변수의 범위를 최소가 되게 정의했다면 이 일은 자연스럽게 이뤄진다. (아이템 57)

### 메모리 누수 주범 2 - 캐시
**`캐시`** 역시 메모리 누수를 일으키는 주범이다.
객체 참조를 캐시에 넣고 나서, 이 사실을 잊은 경우 누수가 발생한다.

캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 `WeakHashMap`을 사용해 캐시를 만들면 된다.

보통의 경우는 캐시를 만들 때 캐시 엔트리의 유효 기간을 정확히 정의하기 어려워서
시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용한다.

### 메모리 누수 주범 3 - 리스너(Listener) 혹은 콜백(Callback)
메모리 누수의 세 번째 주범은 바로 **`리스너(lisetner) 혹은 콜백(callback)`** 이다.

클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치를 해주지 않는 한 콜백은 계속 쌓여갈 것이다.
이럴 때 콜백을 약한 참조(Weak Reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.

<br/>
<pre>
<b>📌 핵심정리</b>
메모리 누수는 겉으로는 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다.
이러한 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다.
그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
</pre>

---

## 아이템 08. finalizer와 cleaner 사용을 피하라
자바는 두 가지 객체 소멸자를 제공한다.

첫 번째로 **`finailzer`** 는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
또한, 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다.

두번째로 **`cleaner`** 는 finalizer 보다는 덜 위험하지만, 여전히 예측할 수 없고 느리고 일반적으로 불필요하다.

finalizer와 cleaner는 즉시 수행된다는 보장이 없다. 그러므로 제때 실행되어야 하는 작업은 절대 할 수 없다.
따라서 프로그램 생애주기와 상관없는 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.

또한, 가비지 컬렉터의 효율을 떨어뜨려 심각한 성능 문제도 동반한다.

finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지도 않다.
final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자.

<pre>
<b>🔎 finalizer나 cleaner를 대체할 수 있는 것은 어떤 것일까?</b>
"AutoCloseable"을 구현해주고,
클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.
</pre>

### cleaner와 finalizer 사용하는 경우 1 - 안전망 역할이 필요할 때
자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다.
대표적으로는 FileInputStream, FileOutputStream, ThreadPoolExecutor가 있다.


### cleaner와 finalizer 사용하는 경우 2 - 네이티브 피어 (native peer)와 연결된 객체일 때
네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.
네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못한다.
그렇기에 cleaner나 finalizer가 나서서 처리하기에 적당한 작업이다
(즉시 회수가 필요하다면 close를 사용하라.)



<br/>
<pre>
<b>📌 핵심정리</b>
cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.
물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다. (그냥 사용하지 않는 게 좋을지도..)
</pre>

---

## 아이템 09. try-finally 보다는 try-with-resources를 사용하라.

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 **`try-finally`** 가 쓰였다.

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader (new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데,
예를 들어 readLine 메서드가 예외를 던지고, 그러면 close 메서드도 실패할 것이다.
이런 상황에서는 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 디버깅을 몹시 어렵게 한다.

여기에 자원을 하나 더 사용하면 어떻게 될까? 코드가 몹시 지저분해진다.
추가될 자원이 많으면 많을수록 코드는 점점 지저분해질 것이다.

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader in = new BufferedReader (new FileReader(path));
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

이러한 문제들은 자바 7이 투척한 **`try-with-resources`** 덕에 모두 해결되었다.
이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.
단순히 void를 반환하는 close 메서드 하나만 덩그러니 정의한 인터페이스이다.

```java
static void copy(String src, String dst) throw IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

try-with-resource 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다.
아까의 상황처럼 readLine과 (코드에서는 나타나지 않는) close 호출 양쪽에서 예외가 발생한다면,
close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다.
이렇게 숨겨진 예외들도 그냥 버려지지 않고, 스택 추적 내역에 **'숨겨졌다(suppressed)'** 는 꼬리표를 달고 출력된다.

`try-finally`에서처럼 `try-with-resources`에서도 `catch 절`을 쓸 수 있다.
catch 절 덕분에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader
        (new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

<br/>
<pre>
<b>📌 핵심정리</b>
꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자.
예외는 없다. 코드는 더 짧고 부명해지고, 만들어지는 예외 정보도 훨씬 유용하다.
try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도,
try-with-resouces로는 정확하고 쉽게 자원을 회수할 수 있다.
</pre>
