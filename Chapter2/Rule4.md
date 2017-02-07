## Rule4. 객체 생성을 막을 때는 private 생성자를 사용하라


### 정적 메서드나 필드만 모은 클래스
```JAVA
java.lang.Math
java.util.Arrays
```
- 자바의 기본 자료형 값 또는 배열에 적용되는 메서드를 한군데 모아놓을 때 유용


```JAVA
java.util.Collections
```
- 특정 인터페이스를 구현하는 객체를 만드는 팩터리 메서드 등의 정적 메서드를 모아놓을 때도 사용

<br>
___위의 유틸리티 클래스들은 객체를 만들 목적의 클래스가 아니다. 따라서,___
<br>
### private 생성자를 클래스에 넣어서 객체 생성을 방지하자.

```JAVA
//객체를 만들 수 없는 유틸리티 클래스
public class UtilityClass {
  private UtilityClass() {
    throw new AssertionError();
  }
}
```

- 클래스 안에서 실수로 생성자를 호출하면 바로 알 수 있도록 AssertionError를 던진다.
- 위 코드가 명시적이지 않으므로 주석을 달아두는 것이 바람직.
- 호출 가능한 생성자가 상위 클래스에 없기 때문에 하위 클래스도 만들 수 없다.
