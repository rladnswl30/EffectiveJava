## 규칙30. int 상수 대신 enum을 사용하라
__열거 자료형(enumerated type)__ 은 고정 개수의 상수들로 값이 구성되는 자료형이다.  
java 1.5 이전에 자바 enum 자료형이 도입되기 전에는 통상 int형의 상수들을 정의해서 열거 자료형을 흉내냈다.

```JAVA
// int를 사용한 enum 패턴 - 지극히 불만족 스럽다
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

### int enum 패턴의 단점
위 예시처럼 __int enum 패턴__ 으로 구현하면 단점이 많다.  
형 안전성 관점에서나 편의성 관점에서 봐도 그렇다.  
- 오렌지를 기대하는 메서드에 사과를 인자로 넘겨도 컴파일러는 불평하지 않는다.
```JAVA
// 귤 맛 나는 사과주스!
int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
```

- 별도의 이름 공간(namespace)를 제공하지 않는다.  
APPLE\_ 이나 ORANGE\_ 접두어가 붙은 이유는 자바가 int enum 그룹별로 별도의 이름 공간(namespace)를 제공하지 않기 때문이다. 접두어로 상수 이름 충돌을 방지했었다.  
- 컴파일 시점 상수(compile-time constant)이기 때문에 상수를 사용하는 클라이언트 코드와 함께 컴파일된다.  
상수의 int 값이 변경되면 클라이언트도 다시 컴파일해야 한다. 그리고, 컴파일하지 않더라도 실행은 되겠지만 어떤 결과를 야기할지 알 수 없다.
- 인쇄 가능한 문자열로 변환하기 쉽지 않다.  
이를 위한 변종 패턴인 __String enum 패턴__ 이 있는데, 더 나쁜 패턴이다. 상수 이름을 화면에 출력할 수 있디만, 상수 비교 시 문자열을 비교해야 해서 성능이 떨어질 수 있다.  
또한, 하드코딩된 문자열 상수에 오타가 있을 경우 문제가 생길 수 있다.  

ex) [참고 자료](http://stackoverflow.com/questions/5092015/advantages-of-javas-enum-over-the-old-typesafe-enum-pattern)

### int enum 단점의 극복, enum 자료형
java 1.5 이후 int와 String enum 패턴의 문제점을 해소하는 대안으로 __enum 자료형__ 이 도입되었다.
```JAVA
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

- 자바의 enum 자료형은 완전한 기능을 갖춘 클래스로, 다른 언어의 enum보다 강력하다.
- 열거 상수(enumeration contstant)별로 하나의 객체를 public static final로 제공하는 것앋.
- 싱글턴 패턴을 일반화한 것으로([규칙3](/Chapter2/Rule3.md)), 싱글턴 패턴은 본질적으로 보면 열거 상수가 하나뿐인 enum과 같다.
- 형안전 enum 패턴(typesafe enum pattern)([규칙21](/Chapter4/Rule21.md))을 자바 문법에 포함시켰다.
- 컴파일 시점 형 안전성(compile-time type safety)을 제공한다.
- 이름 공간(namespace)이 분리되어서 같은 이름의 상수가 평화롭게 공존할 수 있다.
- 상수를 추가하거나 순서를 변경해도 클라이언트는 다시 컴파일 할 필요가 없다.
- toString 메서드를 호출하면 인쇄 가능한 문자열로 쉽게 변환할 수 있다.
- 임의의 메서드나 필드도 추가할 수 있으며, 임의의 인터페이스를 구현할 수 있다.
- Object에 정의된 고품질 메서드가 포함되어 있으며(3장),  
Comparable 인터페이스(규칙12)와 Serializable 인터페이스(11장)가 구현되어 있다.

```JAVA
//데이터와 연산을 구비한 enum 자료형
public enum Planet {
  MERCURY (3.302e+23, 2.439e6),
  VENUS   (4.869e+24, 6.052e6),
  EARTH   (5.974e+24, 6.378e6)
  ...
  ;

  private final double mass;    //kg단위
  private final double radius;  //m단위
  private final double surfaceGravity;  //m / s^2

  //중력 상수(m^3 / kg s^2)
  private static final double G = 6.67300E-11;

  //Constructor
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }

  public double mass() { return mass; }
  public double radius() { return radius; }
  public double surfaceGravity() { return surfaceGravity; }

  public double surfaceWeight(double mass) {
    return mass * surfaceGravity; // F = ma
  }
}
```
이처럼 enum 상수에 데이터를 넣으려면 객체 필드(instance field)를 선언하고, 생성자를 통해 받은 데이터를 그 필드에 저장하면 된다.
- enum은 원래 변경 불가능하므로(immutable) 모든 필드는 final로 선언되어야 한다([규칙15](/Chapter4/Rule15.md)).
- 필드는 public으로 선언할 수 있지만, privat으로 선언하고 public 접근자(accessor)를 두는 편이 더 낫다.
- 모든 enum 상수를 선언된 순서대로 저장하는 배열을 반환하는 static values 메서드가 기본적으로 정의되어 있다.  
ex) for(Planet p : Planet.values())
- enum에 정의한 메서드를 클라이언트에 공개할 특별한 이유가 없으면, private 혹은 package-private로 선언해라([규칙13](/Chapter4/Rule13.md))
- 일반적으로 유용하게 쓰일 enum이라면, 최상위(top-level) public 클래스로 선언해야 한다.  
ex) java.math.RoundingMode enum

### 요약
- enum은 고정된 상수 집합이 필요할 때 쓴다. (태양계 행성, 요일, 장기판 말과 같은 "원래 열거형인 자료형(natural enumerated type)들"이 포함!)
- enum 자료형은 int 상수에 비해 많은 장점을 가지고 있다.
 - 가독성↑, 안전하며, 더 강력하다.
- 임의의 메서드나 인터페이스를 추가해서 기능을 향상시킨 enum도 많다.
- 동일한 메서드가 상수별로 다르게 동작하도록 만들어야 하는 enum은 switch문 대신 상수별 메서드를 구현하자.
- 여러 enum 상수가 공통 기능을 이용해야 하는 일이 생길 땐, __정책 enum 패턴__ 사용을 고려해라.
