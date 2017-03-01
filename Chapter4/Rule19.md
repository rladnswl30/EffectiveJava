## Rule19. 인터페이스는 자료형을 정의할 때만 사용하라.
 
- 인터페이스를 구현하는 클래스를 만들게 되면, 그 인터페이스는 해당 클래스의 객체를 참조할 수 있는 __자료형(type)__ 역할을 한다.

### 상수 인터페이스 패턴
- 다른 목적으로 인터페이스를 정의하고 사용하는 것은 적절치 못한데, 예시로 __상수 인터페이스(constant interface) 패턴__ 이 있다.

```JAVA
// 상수 인터페이스 안티 패턴 (사용하지 말 것 )
public interface PhysicalConstants {
  static final double AVOGADROS_NUMBER = 6.02214199e23;
  static final double BOLTZMANN_CONSTANT = 1.38060503e-23;
}
```

- 클래스가 어떤 상수를 어떻게 사용하는지는 구현 세부사항인데, 상수 정의를 인터페이스에 포함시켜 세부사항이 공개 API에 스며들게 된다.
- 다음번 리릴즈에는 위의 상수를 사용하지 않도록 클래스를 변경한다고 하면, 이진 호환성(binary compatibility) 보장을 위해 인터페이스를 계속 구현해야 함
- 비-final 클래스가 상수 인터페이스를 구현할 경우, 그 모든 하위 클래스의 이름 공간(namespace)이 해당 인터페이스 상수들로 오염
- 자바 플랫폼 라이브러리에 포함된 상수 인터페이스 java.io.ObjetStreamConstants 는 잘못된 것이므로 따라하지 말자
- 상수를 API 일부로 공개하고 싶을 때  
-> 해당 상수가 이미 존재하는 클래스나 인터페이스에 강하게 연결되어 있을 때는 그 상수들을 해당 클래스나 인터페이스에 추가해야 한다.

ex) 상수를 API에 공개하는 Integer class 예시
```JAVA
public final class Integer extends Number implements Comparable<Integer> {
    /**
     * A constant holding the minimum value an <code>int</code> can
     * have, -2<sup>31</sup>.
     */
    public static final int   MIN_VALUE = 0x80000000;

    /**
     * A constant holding the maximum value an <code>int</code> can
     * have, 2<sup>31</sup>-1.
     */
    public static final int   MAX_VALUE = 0x7fffffff;
```
- 기본 자료형의 객체 표현형들(Integer, Double)에는 MIN_VALUE와 MAX_VALUE 상수가 공개되어 있다.
- 이런 상수들이 enum 자료형의 멤버가 되어야 바람직할 때는 enum 자료형(규칙30)과 함께 공개해야 한다. 그렇지 않을 땐, 해당 상수들을 객체 생성 불가능한 유틸리티 클래스__([규칙4](/Chapter2/Rule4.md))__ 에 넣어 공개해야 한다.

ex) 상수 유틸리티 클래스 예시
```JAVA
package com.effectivejava;

//상수 유틸리티 클래스
public class PhysicalConstants {
  private PhysicalConstants() { } //객체 생성 막음

  public static final double AVOGADROS_NUMBER = 6.02214199e23;
  public static final double BOLTZMANN_CONSTANT = 1.38060503e-23;
}
```

- 보통 이런 유틸리티 클래스 사용 시, 상수 앞에 PhysicalConstants.AVOGADROS_NUMBER 처럼 클래스 이름을 붙여야 한다.
- 유틸리티 클래스에 포함된 상수를 이용할 일이 많다면, __정적 임포트(static import)__ 기능을 사용하면 클래스 이름 제거 가능(jdk 1.5)

ex) 정적 임포트 기능을 활용한 유틸리티 클래스 예시
```JAVA
//정적 임포트 기능을 활용해 상수 이름 앞의 클래스명 제거
import static com.effectivejava.PhysicalConstants.*;

public class Test {
    double atoms(double mols) {
      return AVOGADROS_NUMBER * mols;
    }
    ...
    //PhysicalConstants를 사용할 일이 많다면 정적 임포트가 적절!
}
```

### 요약
- 인터페이스는 자료형을 정의할 때만 사용하자.  
특정 상수를 API의 일부로 공개할 목적으로는 적절치 않다!
