# 12장. 직렬화

> **객체 직렬화**: <br>
JAVA가 **객체를 바이트 스트림으로 인코딩**하고(**직렬화**), 그 **바이트 스트림으로부터 다시 객체를 재구성**하는(**역직렬화**) 메커니즘 <br>
직렬화된 객체는 다른 VM에 전송하거나 디스크에 저장한 후 나중에 역직렬화 가능
> 

<aside>
✏️ 직렬화가 품고 있는 위험과 그 위험을 최소화하는 방법

</aside>

## *아이템 85*. 자바 직렬화의 대안을 찾으라 *(p.450)*

- 직렬화의 **위험성**: **보이지 않는 생성자**, API와 구현 사이의 **모호해진 경계**, 잠재적인 **정확성** 문제, **성능**, **보안**, **유지보수성** 등
- 직렬화의 근본적인 문제: **공격 범위가 너무 넓고** 지속적으로 더 넓어져 **방어하기 어렵다는 점**
- 아주 신중하게 제작한 바이트 스트림만 역직렬화해야 함
    - **역직렬화 과정에서 호출**되어 **잠재적으로 위험한 동작을 수행**하는 메서드(:**가젯**, gadget)이 체인으로 엮이면 큰 피해 발생
    - 역직렬화에 시간이 오래 걸리는 짧은 스트림(:**역직렬화 폭탄**, deserialization bomb)을 역직렬화하는 것만으로도 서비스 거부 공격에 쉽게 노출
    
- **직렬화 위험을 회피**하는 가장 좋은 방법 
    - **아무것도 역직렬화하지 않는 것**
    - 우리가 작성하는 새로운 시스템에서 자바 직렬화를 써야 할 이유는 전혀 없음!
    
- **객체와 바이트 시퀀스를 변환**해주는 다른 메커니즘(:크로스-플랫폼 구조화된 데이터 표현)이 많이 있음
    - (이것 또한 **직렬화 시스템**이라 불리기도 하지만, 명확히 구분하고자 이번 아이템에서 쓰이는 용어)
    - 자바 직렬화의 여러 위험을 회피하면서 풍부한 이점 제공
    - 자바 직렬화보다 훨씬 간단함 - 임의의 객체 그래프를 자동으로 직렬화/역직렬화하지 않고, **속성-값 쌍의 집합**으로 구성된 간단하고 구조화된 **데이터 객체 사용**
    - 대표적 예시
        - **JSON** - **텍스트** 기반, 오직 **데이터** 표현
        - **프로토콜 버퍼**(Protocol Buffers, protobuf) - **이진** 표현, **문서**를 위한 스키마 제공
    
- 레거시 시스템 때문에 자바 **직렬화를 완전히 배제할 수 없을 때**의 차선책
    - **신뢰할 수 없는 데이터**는 절대 **역직렬화하지 않는 것**

- 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없을 때
    - **객체 역직렬화 필터링** (`java.io.ObjectInputFilter`) 사용 - 데이터 스트림이 **역직렬화되기 전에 필터를 설치**하는 기능
    

> **핵심 정리**
직렬화는 위험하니 피해야 한다.
시스템을 새로 설계한다면 JSON이나 프로토콜 버퍼 같은 대안을 사용하자.
신뢰할 수 없는 데이터는 역직렬화하지 말고, 꼭 해야 한다면 객체 역직렬화 필터링을 사용하되, 이마저도 모든 공격을 막아줄 수는 없다.
> 

## *아이템 86*. `Serializable`을 구현할지는 신중히 결정하라 *(p.455)*

- 어떤 클래스의 **인스턴스를 직렬화**할 수 있게 하려면 **클래스 선언**에 `implements Serializable`만 덧붙이면 되므로 간단해 보이지만, 사실 복잡한 문제가 많음

### `Serializable` 구현의 문제점

1. **릴리스한 뒤에는 수정하기 어려움**
    - 직렬화된 바이트 스트림 인코딩**(직렬화 형태)도 하나의 공개 API**가 되어 영원히 지원해야 함
    - 기본 직렬화 형태에서는 private, package-private 인스턴스 필드들마저 공개되어 **캡슐화가 깨짐**
    - ⇒ 직렬화 가능 클래스를 만들고자 한다면, 고품질의 **직렬화 형태도 주의해서 함께 설계**해야 함(*아이템 87, 90*)

1. **버그와 보안 구멍**이 생길 위험이 높아짐(*아이템 85*)
    - **객체는 생성자를 사용해 만드는 것이 기본**이지만, 직렬화는 언어의 **기본 메커니즘을 우회**하는 객체 생성 기법 ⇒ ‘숨은 생성자’
    - ⇒ 기본 역직렬화를 사용하면 **불변식 깨짐**과 **허가되지 않은 접근에 쉽게 노출**됨(*아이템 88*)
    
2. 해당 클래스의 **신버전을 릴리스**할 때 **테스트할 것이 늘어남**
    - 신/구버전의 **양방향 직렬화/역직렬화**가 모두 가능한지 확인해야 함
    - 클래스를 처음 제작할 때 커스텀 직렬화 형태를 잘 설계해놓았다면 테스트 부담을 줄일 수 있음(*아이템 87, 90*)
    

- **상속용으로 설계된 클래스** (*아이템 19*)는 대부분 `Serializable`을 구현하면 안 되며, **인터페이스**도 대부분 `Serializable`을 확장해서는 안 됨

- 우리가 작성하는 클래스의 인스턴스 필드가 **직렬화와 확장이 모두 가능**하다면 주의할 점
    - 인스턴스 필드 값 중 **불변식을 보장**해야 할 게 있다면 반드시 **하위 클래스에서** `finalize` 메서드를 **재정의하지 못하게** 해야 함 (⇒ `finalize` 메서드를 자신이 재정의하면서 `final`로 선언)
    - 인스턴스 필드 중 **기본값** (`0`, `false`, `null`) **으로 초기화되면 위배되는 불변식**이 있다면 클래스에 다음의 메서드를 반드시 추가해야 함
        
        ```java
        private void readObjectNoData() throws InvalidObjectException {
        	throw new InvalidObjectException("스트림 데이터가 필요합니다");
        }
        ```
        

- `Serializable`을 구현하지 않을 때 주의할 점
    - 역직렬화를 위해서는 상위 클래스에서 **매개변수가 없는 생성자를 제공**해야 함

- **내부 클래스** (*아이템 24*)는 **직렬화를 구현하지 말아야 함**
- **정적 멤버 클래스**는 구현해도 됨

## *아이템 87*. 커스텀 직렬화 형태를 고려해보라 *(p.459)*

- 먼저 고민해보고 **괜찮다고 판단될 때만 기본 직렬화 형태** 사용

- 객체의 **물리적 표현과 논리적 내용이 같다면** 기본 직렬화 형태라도 무방
    
    ```java
    /* 기본 직렬화 형태에 적합한 클래스 - 사람의 성명을 표현한 예 */
    public class Name implements Serializable {
    	/**
    	* 성. null이 아니어야 함.
    	* @serial
    	*/
    	private final String lastName;
    
    	/**
    	* 이름. null이 아니어야 함.
    	* @serial
    	*/
    	private final String firstName;
    
    	/**
    	* 중간이름. 중간이름이 없다면 null.
    	* @serial
    	*/
    	private final String middleName;
    }
    ```
    
    ⇒ 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성되고 **논리적 구성요소를 정확히 반영**
    
    ```java
    /* 기본 직렬화 형태에 적합하지 않은 클래스 - 문자열 리스트를 표현한 예 */
    public final class StringList implements Serializable {
    	private int size = 0;
    	private Entry head = null;
    
    	private static class Entry implements Serializable {
    		String data;
    		Entry next;
    		Entry previous;
    	}
    }
    ```
    
    ⇒ **논리적**으로는 **일련의 문자열** 표현, **물리적**으로는 문자열들을 **이중 연결 리스트**로 연결
    
    - 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 생기는 문제
        1. **공개 API가 현재의 내부 표현 방식에 영구히 묶임**
        2. 너무 많은 **공간**을 차지할 수 있음 (e.g. 엔트리와 연결 정보는 내부 구현에 해당하므로 직렬화 형태에 포함할 가치가 없음)
        3. **시간**이 너무 많이 걸릴 수 있음 (객체 그래프 직접 순회)
        4. 스택 오버플로를 일으킬 수 있음 (객체 그래프 재귀 순회)
        
    - **해시테이블**은 물리적으로는 **키-값 엔트리들을 담은 해시 버킷을 차례로 나열**한 형태
        - 어떤 엔트리를 어떤 버킷에 담을지는 키에서 구한 해시코드가 결정
        - 계산 방식은 구현에 따라 달라질 수 있고 계산할 때마다 달라지기도 함
        - ⇒ 해시테이블에 기본 직렬화를 사용하면 심각한 버그로 이어질 수 있음
        
- `transient` 한정자는 해당 인스턴스 필드가 **기본 직렬화 형태에 포함되지 않는다는 표시**
    - ⇒ `transient`로 선언하지 않은 모든 인스턴스가 직렬화됨
    - **해당 객체의 논리적 상태와 무관한 필드**라고 확신할 때만 `transient` 한정자를 생략하고, 대부분의 인스턴스 필드를 `transient`로 선언해야 함
    - 기본 직렬화를 사용하면 `transient` 필드들은 **역직렬화될 때 기본값으로 초기화**됨
    - 기본값을 그대로 사용해서는 안 된다면 `readObject` 메서드에서 `defaultReadObject`를 호출한 다음, **해당 필드를 원하는 값으로 복원**(*아이템 88*)하거나 그 값을 **처음 사용할 때 초기화**(*아이템 83*)
    
- 객체의 전체 상태를 읽는 메서드에 적용해야 하는 **동기화 메커니즘을 직렬화에도 적용**해야 함
    - e.g. 모든 메서드를 `synchronized`로 선언하여 스레드 안전하게 만든 객체(*아이템 82*)에서 기본 직렬화를 사용하려면 `writeObject`도 `synchronized`로 선언해야 함
    
- 직렬화 가능 클래스 모두에 **직렬 버전 UID를 명시적으로 부여**해야 함
    
    ```java
    private static final long serialVersionUID = <무작위로 고른 long 값>;
    ```
    
    - **구버전으로 직렬화된 인스턴스와의 호환성을 끊으려는 경우**를 제외하고는 직렬 버전 **UID를 절대 수정하면 안 됨**
