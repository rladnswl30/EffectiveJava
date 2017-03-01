## Rule20. 태그 달린 클래스 대신 클래스 계층을 활용하라
 
### 태그(tag)가 달린 클래스
- 두 가지 이상의 기능을 가지고 있으며, 그 중 어떤 기능을 제공하는지 표시하는 태그(tag)가 달린 클래스를 만날 때가 있다.

ex)
```JAVA
// 태그 달린 클래스 - 클래스 계층을 만드는 쪽이 더 낫다!
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    //어떤 모양인지 나타내는 태그 필드
    final Shape shape;

    //태그가 RECTANGLE일 때만 사용되는 필드들
    double length;
    double width;

    //태그가 CIRCLE일 때만 사용되는 필드들
    double radius;

    //원을 만드는 생성자
    Figure(double radius) {
      shape = Shape.CIRCLE;
      this.radius = radius;
    }

    //사각형을 만드는 생성자
    Figure(double length, double width) {
      shape = Shape.RECTANGLE;
      this.length = length;
      this.width = width;
    }

    double area() {
    switch(shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius)
      default:
        throw new AssertionError();
    }
  }
}
```

### 태그 달린 클래스의 문제점
- enum 선언, 태그 필드, switch문 등 상투적 코드가 반복되는 클래스가 만들어진다.
- 서로 다른 기능을 위한 코드가 한 클래스에 모여있어 가독성이 떨어진다.
- 객체를 만들 때 필요 없는 기능을 위한 필드도 함께 생성되어 메모리 요구량이 늘어난다.
- 생성자에서 관련 없는 필드를 초기화하지 않는 한, 필드들을 final로 선언할 수도 없으므로 상투적 코드가 늘어난다.
- 잘못된 필드 초기화 시, 프로그램이 실행 중 뻗어버릴 수 있다.
- 새로운 기능 추가 시, 소스 파일을 반드시 수정해야하고, switch-case문 추가적으로 수정해야 한다.
- 객체의 자료형만 봐서 객체가 제공하는 기능을 알기 힘들다.

즉, 태그 기반 클래스(tagged class)는 너저분하고, 오류 발생 가능성이 높으며, 효율적이지도 않다.

### 태그 기반 클래스 대신, 클래스 계층인 하위 자료형(subtyning)과 하위 클래스(concrete subeclass)로 정의하자
- 태그 기반 클래스는 클래스 계층을 얼기설기 흉내낸 것
- 자바는 다양한 기능의 객체들을 하나의 자료형으로 표현할 좋은 방법을 갖고 있다. -> __하위 자료형 정의(subtyping)__
- 태그 기반 클래스를 클래스 계층으로 변환하려면, 태그 값에 따라 달리 동작하는 메서드를 추상 메서드로 선언하는 __추상 클래스(abstract class)__ 정의
- 태그 기반 클래스가 제공하던 각각의 기능을 최상위 클래스의 __객체 생성 가능 하위 클래스__ 로 정의

ex) 하위 자료형과 하위 클래스로 정의한 클래스 계층의 예시
```JAVA
//태그 기반 클래스를 클래스 계층으로 변환한 결과
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius }

    double area() {
      return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
  final double length;
  final double width;

  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }

  double area() { return length * width; }
}
```

- 단순하고 명료하며, 원래 클래스에 있던 난삽하고 상투적인 코드도 없다.
- 각각의 기능이 별도 클래스로 구현되어 있고, 각 클래스 안에는 관련 없는 필드도 존재하지 않는다.
- 모든 필드는 final로 선언되어 있어서 생성자가 모든 데이터 필드를 적절히 초기화 할 것이다.
- 클래스 계층 확장 시 최상위 클래스의 소스 코드를 보지 않고도 독립적으로 협업 가능
- 자료형 간의 자연스러운 계층 관계를 반영할 수 있어서 유연성이 높아지고, 컴파일 시 형 검사(type checking) 용이

ex)
```JAVA
class Squre extends Rectangle {
  Squre(double side) {
    super(side, side);
  }
}
```

### 요약
- 태그 기반 클래스 사용은 피하자
- 태그 필드가 있는 클래스를 보면, 클래스 계층으로 변환하는 것을 고려해보자
