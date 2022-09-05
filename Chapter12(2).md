# 12장. 직렬화

## 아이템 88. readObject 메서드는 방어적으로 작성하라
- readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다.
- 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안된다.
### 안전한 readObject 메서드를 작성하는 지침
- private 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로복사하라.
- 모든 불변식을 검사하여 어긋나는 게 발견되면 `InvalidObjectException`을 던진다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 `ObjectInputValidation` 인터페이스를 사용하라.
- 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.

## 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라
- 싱글턴 패턴에 `implements Serializable`을 추가하는 순간 더 이상 싱글턴이 아니게 된다.
- readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 trasiend로 선언해야 한다.
- readResolve 메서드의 접근성은 매우 중요하다. final 클래스에서라면 readResolve 메서드는 private 여야 한다.
- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하는 것이 더 좋다.
```java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = {"Hound Dog", "Hearbreak Hotel"};
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

## 아이템 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
### 직렬화 프록시 패턴 (Serialization proxy pattern)
- Serializable 구현 시 생성자 이외의 방법으로 인스턴스를 생성할 수 있어 발생하는 버그와 보안 문제에 대한 위험을 크게 줄여준다.
- 직렬화 프록시 패턴 구현 방법
  1. 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static 으로 선언한다.
  2. 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개 변수로 받아야 한다
  3. 바깥 클래스와 직렬화 프록시 모두 Serializable 을 구현한다고 선언해야 한다.
  4. 바깥 클래스에 writeReplace 메서드 추가하여, 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환한다.
  5. 불변식을 훼손하려는 공격을 막기 위해 readObject 메서드에 바깥 클래스를 추가한다.
  6. 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가한다.
```java
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;
    
    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }
    
    private static final long serialVersionUID = 234089894760437; // 아무 값이나 상관없다.
    
    // 직렬화 프록시 패턴용 wirteReplace
    private Object wirteReplace() {
        return new SerializationProxy(this);
    }
    
    // 불변식을 훼손하려는 공격을 막기 위한 직렬화 프록시 패턴용 readObject 메서드
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
    
    // Period.SerializationProxy 용 readResolve 메서드
    private Object readResolve() {
        return new Period(start, end);
    }
}
```
### 직렬화 프록시 패턴 한계
1. 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다.
2. 객체 그래프에 순환이 잇는 클래스에도 적용할 수 없다.