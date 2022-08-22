---

# [ Item 57 ] 지역변수의 범위를 최소화하라

---

> **[Item15]클래스와 멤버의 접근 권한을 최소한하라** 와 취지가 비슷하다.
지역번수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.
> 

## 지역변수의 범위를 줄이는 방법

---

### **가장 처음 쓰일 때 선언하기**

1. 미리 선언부터 해두면 코드가 어수선해져 가독성이 떨어진다.
2. 지역변수를 생각 없이 선언하다 보면 변수가 쓰이는 범위보다 너무 앞서 선언하거나, 다 쓴 뒤에도 여전히 살아 있게 되기 쉽다.
3. 지역변수의 범위는 선언된 지점부터 그 지점을 포함한 블록이 끝날 때까지이므로, 실제 사용하는 블록 바깥에 선언된 변수는 그 블록이 끝난 뒤까지 살아있게 된다.

### **선언과 동시에 초기화하기**

1. 초기화에 필요한 정보가 충분하지 않다면 정보가 충분해질 때까지 선언을 미뤄야 한다.
2. try-catch 문은 이 규칙에서 예외다. 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야한다.
3. 반복문에서는 반복 변수 ( loop variable ) 의 범위가 반복문의 몸체, 그리고 for 키워드와 몸체 사이 괄호 안으로 제한된다.
4. 반복자 ( index )를 사용해야 하는 경우에는 for-each 구문보다 전통적인 for문이 낫다.

### **반복문은 while문보다 for문을 사용하는 편이 낫다.**

1. 변수의 값을 반복문이 종료된 이후에도 사용해야 하는 상황이라면 while문을 써야 하겠지만, 아니라면 for문을 쓰는게 낫다. 
2. 복사 붙여넣기 오류 ( 비슷한 형태의 반복문을 반복할 때 반복 변수를 잘못 쓰는 경우 ) 를 컴파일 타임에 잡아준다.
    
    ```java
    Iterator<Element> i = c.iterator();
    while (i.hasNext()) {
        doSomething(i.next());
    }
    
    Iterator<Element> i2 = c2.iterator();
    while (i.hasNext()) {           // 버그 !!
        doSomething(i.next());
    }
    ```
    
    1. 두 번째 while문에는 복사 붙여넣기 오류가 있다. 하지만 이 코드는 컴파일도 잘되고 예외도 던지지 않는다.
    2. 두번째 while문은 c2를 순회하지 않고 곧장 끝나버려 c2가 비었다고 착각하게 한다.
    
    ```java
    for (Iterator<Element> i = c.iterator(); i.hasNext();) {
        Element e = i.next();
        ...// e와 i로 무언가를 한다. 
    }
    
    // 다음 코드는 "i를 찾을 수 없다"는 컴파일 오류를 낸다.
    for (Iterator<Element> i2 = c2.iterator(); i.hasNext();) {
        Element e2 = i2.next();
        ...// e2와 i2로 무언가를 한다. 
    }
    ```
    
    1. for문을 사용하면 이런 복사 붙여넣기 오류를 컴파일 타임에 잡아준다. 
    2. 첫 번째 반복문이 사용한 원소와 반복자의 유효 범위가 반복문 종료와 함께 끝나기 때문이다.
3. 똑같은 이름의 변수를 여러 반복문에서 사용해도 서로 아무 영향을 주지 않는다.
4. while문보다 짧아서 가독성이 좋다.
5. 반복 여부를 결정짓는 변수 i의 한곗값을 변수 n에 저장하여, 반복 때마다 다시 계산해야 하는 비용을 없앴다.

### **메서드를 작게 유지하고 한 가지 기능에 집중한다.**

1. 한 메서드에서 여러 가지 기능을 처리한다면 그중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다.
2. 단순히 메서드를 기능별로 쪼개자.

# [ Item 58 ] 전통적인 for문보다는 for-each문을 사용하라

---

> 전통적인 for문과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다.
성능 저하도 없다. **가능한 모든 곳에서 for문이 아닌 for-each문을 사용하자.**
> 

```java
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
    ... // e로 무언가를 한다. 
}
```

```java
for(int i = 0; i < a.length; i++) {
    ... // a[i]로 무언가를 한다. 
}
```

위의 for문은 while문보다는 낫지만 가장 좋은 방법은 아니다. 

**반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 우리에게 진짜 필요한 건 원소들 뿐이다.**

이러한 문제들은 for-each문을 사용하면 다 해결된다.

---

- for-each문의 정식 이름은 ‘향상된 for문 ( enhanced for statement )’이다.
- 반복자와 인덱스 변수를 사용하지 않으므로 코드가 깔끔해지고 오류가 날 일이 없다.
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 씬경 쓰지 않아도 된다.

```java
for (Element e : elements) {
    ... // e로 무언가를 한다. 
}
```

- 콜론(:)은 안의 ( in ) 이라고 읽으면 된다.

## 컬렉션을 중첩해 순회해야 한다면 for-each문의 이점이 더욱 커지나.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext();)
        deck.add(new Card(i.next(), j.next()));     //오류 발생!!
```

- 원래대로 하면 숫자 1개당 rank가 여러번 불려야 하는데 저러면 숫자 1개에 rank 1개가 불려 `NoSuchElementException`을 던진다.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE}
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Suit suit : suits) {
    for(Rank rank : ranks) {
        deck.add(new Card(suit, rank));   
    }
}
```

- 위와 같이 for-each를 사용하면 깔끔하고 간단하게 코드를 짤 수 있다.

## for-each를 사용할 수 없는 상황

- **파괴적인 필터링 ( destructive filtering )**
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.
    - Java 8 부터 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하지 않을 수 있다.
- **변형 ( transforming )**
    - 리스트나 배열을 순회하면서 원소 값 일부 혹은 전체를 교체하는 경우에는 인덱스를 사용해야 한다.
- **병렬 반복 ( parallel iteration )**
    - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

```java
**핵심정리**
- for문 대신 for-each를 사용할 수 있는 경우는 무조건 사용하자
- 전통적인 for문과 비교했을 때, for-each문은 명료하고 유연하고 버그를 예방해 준다.
- for-each문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회 가능하다.
```

# [ Item 59 ] 라이브러리를 익히고 사용하라

---

무작위 정수를 하나 생성해본다고 해보자

```java
static Random rnd = new Random();
static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```

- n이 그리 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다.
- n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다.

---

```java
public static void main(String[] args) {
    int n = 2 * (Integer.MAX_VALUE / 3);
    int low = 0;
    for(int i = 0; i < 1000000; i++) {
        if(random(n) < n/2) {
            low++;
        }
        System.out.println(low);
    }
}
```

- 무작위 수를 백만개 생성한 다음, 그중 중간 값보다 작은 게 몇 개인지를 출력한다.
- random 메서드가 이상적으로 동작한다면 약 50만개가 출력되야 하지만, 실제로 돌려보면 666,666에 가까운 값을 얻는다.
- 무작위로 생성된 수 중에서 2/3 가량이 중간값보다 낮은 쪽으로 쏠린 것이다.
- 지정한 범위 바깥의 수가 종종 튀어나올 수 있다. rnd.nextInt()가 반환한 값을 Math.abs를 이용해 음수가 아닌 정수로 매핑하기 때문이다.

---

## 표준 라이브러리를 사용하면 다른 프로그래머들의 경험을 활용할 수 있다.

- 메서드 동작 방식은 몰라도 알고리즘에 능통한 개발자나, 여러 분야의 전문가가 설계와 구현에 시간을 들여 개발한 것이다.
- 버그가 발생되더라도 다음 릴리스에 수정 보완 개선이 될 수 있다.
- 표준 라이브러리를 사용하며 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.
- 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 되고 어플리케이션 기능 개발에 집중할 수 있다.
- 따로 노력하지 않아도 릴리즈가 나올 때 마다 성능이 지속해서 개선된다.
- 기능이 점점 많아진다. 개발자 커뮤니티에서 나온 이야기를 바탕으로 논의 후 다음 릴리즈에 기능이 추가되곤 한다.
- 라이브러리를 사용하면 많은 사람들에게 낯익은 코드가 된다. 다른 개발자들이 유지보수 하기 쉬워지고 재사용성이 높아진다.

### ****메이저 버전 릴리즈 마다 수많은 기능이 추가된다.****

- 자바는 메이저 릴리즈마다 새로운 기능을 설명하는 웹페이지를 공시한다.
- 한 번쯤은 읽어볼만 하다
- 너무 많아서 읽기 힘든 경우에는 java.lang, java.util, java.io와 하위 패키지들에는 익숙해져야 한다.
- 컬렉션 프레임워크나 concurrent 패키지는 알아두면 도움이 된다.

```java
핵심정리
- 바퀴를 다시 발명하지 말자. 아주 특별한 나만의 기능이 아니라면 누군가 이미 라이브러리 형태로 구현해놓았을 가능성이 크다.
- 그런 라이브러리가 있다면, 쓰면 된다. 있는지 모르겠다면 찾아보라
- 일반적으로 라이브러리의 코드는 여러분이 직접 작성한 것보다 품질이 좋고, 점차 개선될 가능성이 크다.
- 코드 품질에도 규모의 경제가 적용된다. 라이브러리 코드는 개발자 각자가 작성하는 것보다 주목을 훨씬 많이 받으므로 코드 품질도 그만큼 높아진다.
```

# [ Item 60 ] 정확한 답이 필요하다면 Float와 Double은 피하라.

---

> float와 double 타입은 과학과 공학 계산용으로 설계되어있다.
이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정말한 ‘근사치’로 계산하도록 세심하게 설계되었다.
따라서 정확한 결과가 필요할 때는 사용하면 안된다.
> 

### 예시 - 금융 계산에 부동 소수 타입을 사용

```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

- 프로그램의 실행결과는 사탕 3개를 구입한 후 잔돈은 0.3999999999999달러가 남는다고 나온다.
- 잘못된 결과이며, 올바른 결과를 위해서는 `BigDecimal`, `int`, `long`을 사용해야 한다.

---

### `BigDecimal`을 사용한 코드 - 속도가 느리고 쓰기 불편하다.

```java
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");
    
    int itemBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for(BigDecimal price = TEN_CENT; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

- 이 프로그램의 실행결과는 사탕 4개 구입 후 잔돈이 0달러가 남는다. ( 올바른 답 )
- `BigDecimal`에는 2가지 단점이 있다.
    - 기본 타입보다 쓰기가 훨씬 불편하고, 느리다. 단발성 계산이라면 문제는 아니지만 쓰기 불편하다.
    - `BigDecimal`의 대안으로 `int`, `long`을 사용해도 된다. 하지만 소수점을 직접 관리해야 한다.

---

### `int`를 사용한 코드 - `Cent`로 문제 해결

```java
public static void main(String[] args) {
    int itemBought = 0;
    int funds = 100;
    for(int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

- `int`로 사용하면 `BigDecimal` 보다는 깔끔하고 정확한 답을 얻을 수 있다.

```java
**핵심 요약**
- 정확한 답이 필요한 계산에는 float난 double을 피하라.
- 소수점 추적은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라.
- BigDecimal이 제공하는 여덟 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다.
- 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서 아주 편리한 기능이다.
- 반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라.
- 숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용
- 열여덟 자리 십진수로 표현할 수 있다면 long을 사용
- 열여덟 자리를 넘어가면 BigDecimal을 사용해야 한다.
```

# [ item 61 ] 박싱된 기본 타입보다는 기본 타입을 사용하라.

---

> 자바의 데이터 타입은 크게 두 가지로 나눌 수 있다.
**기본 타입 ( Primitive Type ) vs 참조 타입 ( Reference Type )**으로 구분할 수 있다.
> 

## 기본 타입 ( Primitive Type )

- int
- long
- short
- double
- char
- boolean

## 참조 타입 ( Reference Type )

- String
- Integer
- Long
- Double
- Boolean

각각 기본 타입에는 대응하는 참조타입이 하나씩 있으며 이를 박싱된 기본 타입이라고 한다.

## 기본타입과 참조타입의 차이

1. **기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성 ( identity ) 란 속성을 갖는다.**
    1. 기본 타입은 흔히 말하는 리터럴 ( Literal ) 이다.
    2. 기본 타입의 값은 JVM 내의 Stack 메모리에 저장된다.
    3. 참조 타입의 값은 객체 내의 상수에 저장된다. 따라서 JVM 내의 Heap 메모리에 저장된다.
    4. 따라서 박싱된 타입의 객체는 같은 값이라 하더라도 다른 객체일 경우에는 다르다고 식별이 가능하다.
2. **기본 타입의 값은 언제나 유효한 값을 가지고 있으나 박싱된 기본 타입은 유효하지 않을 수 있다.**
    1. 기본 타입의 값은 Java의 경우 초기화되지 않으면 0으로 초기화된다.
    2. 박싱된 기본 타입의 경우에는 초기화되지 않으면 null이 될 수 있다.
3. **기본 타입이 박싱된 기본 타입보다 시간과 메로리 사용면에서 더 효율적이다.**
    1. 기본 타입은 변수에 값이 있는 반면, 박싱된 기본 타입은 변수의 객체참조 정보를 바탕으로 heap에서 찾으므로 시간적인 측면에서 기본 타입보다 값에 접근하는 시간이 더 들게 된다.
    2. 박싱된 타입은 heap에 객ㅊ에를 생성하기 때문에 메모리 사용에서 안좋다.

### 잘못 구현된 비교자 예제

```java
Comparator<Integer> naturalOrder = 
    (i, j) -> (i < j) ? -1 : (i==j ? 0 : 1);
```

- 같은 객체를 비교하는게 아니라면 박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어난다.
- 실무에서 기본 타입을 다루는 비교자가 필요하다면 Comparator.naturalOrder()를 사용하자.

### 기이하게 동작하는 프로그램

```java
public class Unbelievable {
    static Integer i;
    
    public static void main(String[] args) {
        if (i == 42)
            System.out.println("믿을 수 없군!");
    }
}
```

- "믿을 수 없군"을 출력하지만, i == 42를 검사할 때 NullPointerException을 던지는 것이다.
    - 원인: i가 int가 아닌 Integer, i의 초깃값도 null
- 거의 예외 없이 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다. 그리고 null 참조를 언박싱하면 NullPointerException이 발생한다.

---

### 박싱된 기본 타입을 사용하는 경우

1. 컬렉션의 원소, 키, 값으로 쓴다.
    1. 컬렉션은 기본 타입을 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 써야만 한다.
2. 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다.

```java
핵심 정리
- 기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 기본 타입을 사용하라.
- 기본 타입은 간단하고 빠르다.
- 박싱된 기본 타입을 써야 한다면 주의를 기울이자.
- 오토박싱이 박싱된 기본 타입을 상요할 때의 번거로움을 줄여주니만, 그 위험까지 없애주지는 않는다.
- 두 박싱된 기본 타입을 == 연산자로 비교한다면 식별성 비교가 이뤄지는데, 이는 여러분이 원한 게 아닐 가능성이 크다.
- 같은 연산에서 기본 타입과 박싱된 기본 타입을 혼용하면 언박싱이 이뤄지며, 언박싱 과정에서 NullPointerException을 던질 수 있다.
- 마지막으로, 기본 타입을 박싱하는 작업은 필요 없는 객체를 생성하는 부작용을 나을 수 있다.
```

# [ Item 62 ] 다른 타입이 적절하다면 문자열 사용을 피하라

> 문자열 ( String ) 은 텍스트를 표현하도록 설계되었다.
문자열은 워낙 흔하고 자바가 또 잘 지원해주어 원래 의도하지 않은 용도로도 쓰이는 경향이 있다.
이번 아이템에서는 문자열을 쓰지 않아야 할 사례를 다룬다.
> 

## 문자열을 다른 값 타입을 대신하기에 적합하지 않다.

- 받은 데이터가 수치형이라면 int, float BigInteger 등 적당한 수치 타입으로 변환해야 한다.
- 예/아니오 질문의 답이라면 절절한 열거타입이나 boolean으로 변환해야 한다.
- 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 하나 작성하라.

## 문자열은 열거 타입을 대신하기에 적합하지 않다.

- 아이템 34에서 이야기했듯, 상수를 열거할 때는 문자열보다는 열거 타입이 월등히 낮다.

## 문자열은 혼합 타입을 대신하기 적합하지 않다.

- 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 대체로 좋지 않은 생각이다.
- 혼합 타입을 문자열로 처리한 부적절한 예

```java
String compoundKey = className + "#" + i.next();
```

## 문자열은 권한을 표현하기에 적합하지 않다.

- 권한 ( capacity )을 문자열로 표현하는 경우가 종종 있다.
- 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별함

```java
public class ThreadLocal {
    private ThreadLocal() { } //객체 생성 불가
     
    // 현 스레드의 값을 키로 구분해 저장한다. 
    public static void set(String key, Object value);
    
    // (키가 가리키는 ) 현 스레드의 값을 반환한다. 
    public static Object get(String key);
}
```

- 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다는 점이 문제이다.
- 보안도 취약하다.

```java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    public static class Key {
        key() {}
    }
    
    //위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		return new Key();
    }
    
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

- String으로 권한을 구분하는 것이 아니라 별도의 타입을 만들어 해결해야 한다.
- 이 방법은 앞선 문자열 기반 API의 문제점을 해결해주지만 개선해야할 부분이 있다.
- set/get 메서드는 이제 static 메서드일 이유가 없다. 따라서 key의 인스턴스 메서드로 변경하는것이 좋다.
- 그렇게 하면 key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라 그 자체가 스레드 지역변수가 된다.

```java
public final class Key {
    public Key();
    public void set(Object value);
    public Object get();
}
```

- 이 API는 get으로 얻은 Object를 실제 타입으로 타입 캐스팅 해야 해서 타입안전하지 않다.
- 하지만 제네릭을 사용한다면 조금 더 타입 안전하게 만들 수 있다.

```java
public final class Key<T> {
    public Key();
    public void set(T value);
    public T get();
}
```

- 이제 자바의 java.lang.ThreadLocal과 흡사해졌다. 문자열 기반 API의 문제를 해결해주며, 키 기반 API보다 빠르고 우아하다.

```java
**핵심 정리**
- 더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열을 쓰고 싶은 유혹을 뿌리쳐라.
- 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다.
- 문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.
```
