## 규칙31. ordinal 대신 객체 필드를 사용하라
상당수 enum 상수는 자연스레 int값 하나에 대응된다.  
ordinal 메서드는 enum 상수의 위치를 나타내는 정수값을 반환한다.

```JAVA
// ordinal을 남용한 사례 - 따라하면 곤란
public enum Ensemble {
  SOLO, DUET, TRIO, QUARTET;

  public int numberOfMusicians() { return ordinal() + 1; }
}
```
동작은 하지만 유지보수 관점에서 끔찍한 코드다.  
상수 순서를 변경하는 순간, numberOfMusicians 메서드는 깨지고 만다. 게다가 이미 사용한 정수 값에 대응되는 새로운 enum 상수 정의가 불가능하다.  

새로운 상수가 나타내는 int값은 순서상 바로 앞에 오는 상수의 int 값보다 정확히 1만큼 커야 한다. 그렇지 않으면 새 상수를 추가할 수 없다.

### enum 상수에 연계되는 값을 ordinal을 사용해 표현하지 말고, 객체 필드(instance field)에 저장해라
```JAVA
public enum Ensemble {
  SOLO(1), DUET(2), TRIO(3), QUARTET(4);

  private final int numberOfMusicians;
  Ensemble(int size) { this.numberOfMusicians = size; }
  public int numberOfMusicians() { return numberOfMusicians; }
}
```
자바 Enum 명세에 따르면, ordinal에 관해...  
```
"대부분의 프로그래머는 이 메서드를 사용할 일이 없을 것이다. EnumSet이나 EnumMap처럼 일반적인 용도의 enum 기반 자료구조에서 사용할 목적으로 설계한 메서드다."
```
라고 명시되어 있다.

따라서, 위 목적이 아니면 ordinal 메서드는 사용하지 말자!
