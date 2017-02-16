## Rule14. public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라.

### 1. 선언된 패키지 밖에서도 사용 가능한 클래스에는 접근자 메서드를 제공하라.

```JAVA
// 이런 저급한 클래스는 절대로 public으로 선언하지 말 것
class Point {
  public double x;
  public double y;
}
```

- 데이터 필드 조작이 가능하여 __캡슐화의 이점을 누릴 수 없다.__

```JAVA
class POINT {
  private double x;
  private double y;

  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```
- private 필드와 public 접근자 메서드(getter)로 바꿔야 좋다.
- 변경 가능 클래스라면 수정자(mutator) 메서드(setter)도 제공하자.
-> 클래스 내부의 표현을 자유로이 수정할 수 있게 된다.

### 2. package-private 클래스나 private 중첩 클래스(nested class)는 데이터 필드를 공개해도 잘못이라 할 수 없다.
- 대신 추상화 하려는 내용을 제대로 기술해라.
- 클래스 정의로 보나 클라이언트 코드로 보나, 접근자 메서드보다는 시각적으로 깔끔해보인다.
- 클래스 내부 표현을 변경해도 외부 코드는 변경되지 않을 것이다.
- package 중첩 클래스는 외부 클래스에 어떠한 영향도 받지 않는다.


### 요약
- public 클래스는 변경 가능 필드를 외부로 공개하면 안된다.
- 변경 불가능 필드는 외부에 공개해도 되지만, 그럴 필요는 없다.
- package-private 또는 private 중첩 클래스의 데이터 필드는 외부로 공개하는 것이 좋을 때도 있다.
