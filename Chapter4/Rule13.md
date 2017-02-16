## Rule13. 클래스와 멤버의 접근 권한은 최소화하라.
### 1. 정보은닉(캡슐화)
- 구현 세부사항을 전부 API 뒤쪽에 감추는 것.
- 모듈 사이의 의존성을 낮춰 개별적으로 개발
- 병렬적으로 개발하여 개발 속도가 올라가고, 유지보수에 좋다.
- 효과적인 성능 튜닝을 가능하게 한다.
- 재사용을 가능하게 한다.
- 대규모 시스템 개발 과정의 risk를 낮춘다.

<br>
### 2. 정보 은닉의 원칙 실현 : 각 클래스와 멤버는 가능한 한 접근 불가능하도록 만들라.
- 정상 동작을 보증하는 한도 내에서 __가장 낮은 접근 권한__ 을 설정하라.
- 최상위 레벨 클래스나 인터페이스는 package-private로 선언
- 최상위 레벨 클래스나 인터페이스가 하나뿐이라면, __private 중첩 클래스__ 로 만들자.

 - 중첩 클래스란    
  최상위 클래스(패키지 멤버 클래스, 중첩된 최상위 클래스) <br> + inner class(멤버 클래스, 지역 클래스, 익명 클래스)를 합쳐 중첩 클래스 라고 한다.


  ```JAVA
   public class OuterClass {
   // fields, methods, etc
       private class NestedClass{
       // ...
       };
   }
  ```

  ```
  멤버의 접근 권한 종류
  - private : 최상위 레벨 클래스의 내부에서만 접근 가능
  - package-private : 기본 접근 권한. 같은 패키지 내의 아무 클래스나 사용 가능
  - protected : 선언된 클래스 및 그 하위 클래스만 사용 가능
  - public : 어디서든 사용 가능
  ```

- public API 설계 시 모든 멤버는 최대한 private으로 선언하되,  
같은 패키지 내의 다른 클래스가 반드시 사용해야 하는 멤버인 경우 package-private으로 만들자.  
-> 의존성을 최대한 끊어내기 위해
- package-private에서 protected로 변경하면 멤버를 사용할 수 있는 범위가 넓어지기 때문에 사용을 자제하자.
- 특정한 인터페이스를 구현하는 클래스를 만들 때는 인터페이스에 속한 모든 메서드를 해당 클래스의 public 메서드로 선언해야 한다.  
-> 상위 클래스 객체를 사용할 수 있는 곳에서는 하위 클래스 객체도 사용 가능해야 하기 때문

<br>
### 3. 객체 필드는 절대로 public으로 선언하면 안된다.
- 비-final 필드나 변경 가능한 객체에 대한 final 참조 필드를 public으로 선언하면, 필드에 저장될 값을 제한할 수 없게 된다.
- 변경 가능 public 필드를 가진 클래스는 다중 스레드에 안전하지 않다.
- 변경 불가능 객체를 참조하는 final 필드라 해도 public으로 선언하면 클래스 내부 데이터 표현 형태를 유연하게 바꿀 수 없다.

<br>
### 4. public static final 배열 필드를 두거나, 배열 필드를 반환하는 접근자를 정의하면 안된다.
- 클라이언트가 변경할 수 있어 보안에 문제가 생긴다.

```JAVA
// 보안 문제를 초래할 수 있는 코드
public static final Thing[] VALUE = {...};
```

<br>
__해결 방법 1)__
```JAVA
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
public으로 선언된 배열을 private으로 바꾸고, 변경 불가능한 public 리스트를 만든다.

<br>
__해결 방법 2)__
```JAVA
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
  return PRIVATE_VALUES.clone();
}
```
배열은 private로 선언하고, 해당 배열을 복사해서 반환하는 public 메서드를 추가한다.

<br>
### 요약
- 접근 권한은 가능한 낮춰라
- 최소한의 public API를 설계한 후, 모든 클래스, 인터페이스, 멤버는 API에서 제외하라.
- public static final 필드를 제외한 어느 필드도 public으로 선언하지 말아라.
- public static final 필드가 참조하는 객체는 변경 불가능 객체로 만들라.
