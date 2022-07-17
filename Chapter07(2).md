# 7장. 람다와 스트림

## 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라

### 스트림 패러다임의 핵심
- 계산을 일련의 변환(transformation)으로 재구성하는 부분
- 이 때, 각 변환 단계는 가능한 이전 다녜의 결과를 받아 처리하는 순수 함수여야 한다.
  > 순수 함수란? <br/>
  > - 오직 입력만이 결과에 영향을 주는 함수 <br/>
  > - 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.
- 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)이 없어야 한다.
#### 텍스트 파일에서 단어별 수를 세어 빈도표를 만드는 함수
> 스트림 패러다임을 이해하지 못한 채 API만 사용한 경우이므로 따라하지 말자!!
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = ne Scanner(file).tokens()) {
    words.forEach(word -> { // 스트림을 가장한 반복적 코드
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
- 위 코드는 길고, 읽기 어렵고, 유지보수에도 좋지 않다.
- 모든 작업이 종단 연산인 forEach 일어나기 때문에, 외부 상태(빈도표)를 수정하는 람다를 실행하면 문제가 발생한다.
- **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자!**
#### "스트림을 제대로 활용한" 텍스트 파일에서 단어별 수를 세어 빈도표를 만드는 함수
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = ne Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting));
}
```
- 계산을 할 때는 forEach가 아닌 `collector(수집기)`를 사용하자!

### java.util.stream.Collectors 클래스
> 무려 39개의 메서드를 가지고 있으며, 그중에는 타입 매개변수가 5개나 되는 것도 있다.
#### toList(), toSect(), toCollections(collectionFactory)
- 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.
- 차례대로 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환한다.
```java
// 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
List<String> topTen = freq.keySet().stream() // 빈도표에서 키 추출
        .sorted(comparing(freq::get).reversed()) // 빈도를 추출하여 역순으로 정렬
        .limit(10) // 상위 10개 추출
        .collect(toList()); // 결과를 리스트로 반환
```
Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히들 이렇게 사용한다.

#### toMap
- `toMap(keyMapper, valueMapper)`는 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.
  ```java
  // toMap 수집기를 사용하여 fromString 구현
  private static final Map<String, Operation> stringToEnum =
      Stream.of(values()).collect(
              toMap(Object::toString, e -> e));
  ```
- toMap에 키 매퍼와 값 매퍼는 물론 병합(merge) 함수까지 제공할 수 있다.
  ```java 
  // 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기
  Map<Artist, Album> toHits = albums.collect(
    toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));
  ```
  - 여기서 maxBy는 Comparator<T>를 입력받아 BinaryOperator<T>를 돌려준다.
  - 키 추출 함수로는 Album::sales를 받았다.
  - 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다.
- 인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는 수집기를 만들 때도 유용하다.
  ```java
  toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal);
  ```

#### groupBy
- 입력 : 분류 함수 (입력받은 원소가 속하는 카테고리를 반환하는 함수)
- 출력 : 원소들을 카테고리별로 모아 놓은 맵
- 대표적인 예 : 아나그램 프로그램 (아이템 45) <br/>
  알파벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵
  ```java
  words.collect(groupingBy(word -> alphabetize(word)))
  ```
- groupBy 리스트 외의 값을 갖는 맵을 생성하게 할 때, 분류 함수와 함께 다운스트림 수집기도 명시해야 한다.

##### 다운스트림(downstream) 수집기
- 해당 카테고리의 모든 우너소를 담은 스트림으로부터 값을 생성한다.
- to Set()
  - 가장 간단한 방법
  - 집합(Set)을 갖는 맵을 만들어낸다.
- toCollection(collectionFactory)
  - 컬렉션을 값으로 갖는 맵을 생성한다.
- counting()
  - 각 카테고리(키)를 해당 카테고리에 속하는 원소의 개수(값)와 매핑한 맵을 얻는다.
  ```java
  Map<String, Long> freq = words
      .collect(groupingBy(String::toLowerCase, counting()));
  ```

#### partitioningBy
- groupingBy의 사촌격
- 분류 함수 자리에 프레디키트(predicate)를 받고 키가 Boolean인 맵을 반환
- 프레디키트에 더해 다운스트림 수집기까지 입력받는 버전도 다중정의되어 있다.

#### counting
- 다운스트림 수집기 전용
- Stream의 count 메서드를 직접 사용하여 같은 기능을 수행할 수 있으니 **collect(counting()) 형태로 사용할 일은 전혀 없다.
- Collections에는 counting 같은 속성의 메서드가 16개나 더 있다.
  - 그 중 9개 : summing, averaging, summarizing 으로 시작하며, 각각 int, long, double 스트림용으로 하나씩 존재한다.
  - 다중정의된 reducing 메서드들, filtering, mapping, flatMapping, collectingAndThen 메서드가 있는데, 대부분 프로그래머는 이들의 존재를 모르고 있어도 상관없다.

#### minBy, maxBy
- 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환

#### joining
- CharSequence 인스턴스의 스트림에만 적용할 수 있다.
- 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다.
- 인수 하나짜리 joining은 CharSequence 타입의 구분문자를 매개변수로 받는다.
- 인수 3개짜리 joining은 구분문자에 더해 접두문자와 접미문자도 받는다.

## 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낮다
### 스트림은 반복(iteration)을 지원하지 않는다.
사실 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, Iterable 인터페이스가 정의한 방식대로 동작한다. <br/>
그럼에도 for-each로 스트림을 반복할 수 없는 까닭은 `Stream이 Iterable을 확장하지 않아서`다. <br/>
**Stream의 iterator 메서드에 메서드 참조를 건넨다면 어떻게 될까?** <br/>
메서드 참조를 매개변수화된 Iterable로 적절히 형변환해서 코드를 작성해야 하며, 이 방법은 스트림 반복을 위한 `끔찍한 우회 방법`이다.
```java
for (ProcessHandle ph : (Iterable<ProcessHandle>) // 형변환 필요
        ProcessHandle.allProcesses()::iterator) {
    // 프로세스 처리
}
```
**어댑터 메서드를 사용하면 더 좋은 코드를 작성할 수 있다.**
```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    // 프로세스 처리
}
```
반대로, API가 Iterable만 반환할 때 스트림 파이프라인에서 처리하기 위해서도 어댑터 메서드를 사용하면 좋다.
```java
// Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterato(), false);
}
```

객체 시퀀스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주자!

반대로 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자!

### Collection 인터페이스
- Iterable의 하위 타입이고 stream 메서드도 제공하니 `반복과 스트림을 동시에 지원`한다.
- 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.


```
컬렉션을 반환할 수 있다면 그렇게 하고, 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라!!
```

## 아이템 48. 스트림 병렬화는 주의해서 적용하라
- 데이터 소스가 Stream.iterate 거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.
- 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다.
  - 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기 좋다.
  - 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다.
    - 참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분의 시간을 멍하니 보내게 된다.
    - 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다.
    - 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.
- 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.
- 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다.
  - Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족하고, 간섭받지 않고, 상태를 갖지 않아야 한다.
  - 데이터 소스 스트림이 효율적으로 나눠지고, 병렬화하거나 빨리 끝나는 종단 연산을 사용하고, 함수 객체들도 간섭하지 않더라도, 파이프라인이 수행하는 진짜 작업이 병렬화에 드는 추가 비용을 상쇄하지 못한다면 성능 향상은 미미할 수 있다.
```java
// n보다 작거나 같은 소수의 개수를 계산하는 함수

// 병렬화하기 전에는 31초 정도 걸린다.
static long pi (long n) {
    return LongStream.rangeClosed(2, n)
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}

// 병렬화 후에는 9.2초로 단축된다.
static long pi (long n) {
    return LongStream.rangeClosed(2, n)
        .parallel()
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```
