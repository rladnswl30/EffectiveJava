## Rule15. 변경 가능성을 최소화하라

- 변경 불가능 클래스란, 그 객체를 수정할 수 없는 클래스이다.  
  ex) String, 기본 자료형 클래스(Boolean, Char...), BigInteger 등
- 변경 불가능 클래스는 설계하기 쉽고, 구현하기 쉽고, 사용하기 쉽고, 오류 가능성도 적고, 안전하다.


### 1. 변경 불가능 클래스를 만들 때의 규칙
 1. 객체 상태를 변경하는 메서드(수정자 메서드 <sub>setter</sub> 등)를 제공하지 않는다.  
<br>
 2. 계승할 수 없도록 한다.
  - 잘못 작성되거나 악의적인 하위 클래스가 변경 불가능성을 깨트리는 일을 막아야 한다.  
  ex) 클래스를 final로 선언  
<br>
 3. 모든 필드를 final로 선언한다.
  - 프로그래머의 의도(변경 불가능)가 분명히 드러난다.
  - 새로 생성된 객체에 대한 참조가 동기화 없이 다른 스레드로 전달되어도 안전  
<br>
 4. 모든 필드를 private로 선언한다.
  - 변경 불가능 객체에 대한 참조 필드를 public으로 선언하는 변경 불가능 클래스는 되도록 만들지 말자.  
  -> 나중에 클래스의 내부 표현을 변경할 수 없기 때문.  
<br>
 5. 변경 가능 컴포넌트에 대한 독점적 접근권을 보장한다.
  - 클래스에 포함된 변경 가능 객체에 대한 참조를 클라이언트는 획득할 수 없어야 한다.
  - 생성자나 접근자, readObject 메서드 안에서는 방어적 복사본을 만들어야 한다. ex) [규칙13](/Rule13.md)  

<br>
### 2. 변경 불가능 클래스 규칙을 따른 예시
```JAVA
public final class Complex {
  private final double re;
  private final double im;

  public Complex(double re, double im) {
    this.re = re;
    this.im = im;
  }

  //대응되는 수정자가 없는 접근자들
  public double realPart() { return re; }
  public double imaginaryPart() { return im; }

  public Complex add(Complex c) {
    return new Complex(re + c.re, im + c.im);
  }

  public Complex subtract(Comlex c) {
    return new Complex(re - c.re, im - c.im);
  }

  @Override
  public boolean equals(Object o) {
    if(o == this) return true;
    if(!(o instanceof Complex)) return false;
    Complex c = (Complex) o;

    // == 대신 compare를 쓰는 이유 ? 정적 메서드
    return Double.compare(re, c.re) == 0 && Double.compare(im, c.im) == 0;
  }
}
```

- 복소수(complex number, 실수부와 허수부를 갖는 수)를 표현하는 클래스
- 사칙연산은 this 객체를 변경하는 대신 새로운 Complex 객체를 만들어 반환
 - 변경 불가능 클래스가 따르는 패턴
 - __함수형 접근법__ : 피연산자를 변경하는 대신, 연산을 적용한 결과를 새롭게 반환  
 (cf. 절차적 혹은 명령형 접근법 : 피연산자에 일정한 절차를 적용하여 그 상태를 바꾼다.)

<br>
### 3. 변경 불가능 객체의 장점
- 변경 불가능 객체는 단순하다.
 - 생성될 때 부여된 한 가지 상태만을 갖는다.
 - 생성자가 불변식을 확실히 따르면, 해당 객체는 불변식을 절대로 어기지 않게 된다.


- 스레드에 안전(thread-safe)할 수 밖에 없다.
 - 어떤 동기화도 필요 없으며, 여러 스레드가 동시에 사용해도 상태가 훼손될 일이 없다.
 - 스레드 안전성 보장의 가장 쉬운 방법


- 자유롭게 공유할 수 있다.
 - (방법 1.) private static final 상수로 만들어라.
 ```JAVA
 public static final Complex ZERO = new Complex(0, 0);
 public static final Complex ONE = new Complex(1, 0);
 ```

 - (방법 2. <sub>위 접근법 개선</sub>) 변경 불가능 클래스는 자주 사용하는 객체를 캐시(cache)하여 이미 있는 객체가 거듭 생성되지 않도록 하는 정적 팩터리([규칙1](/Chapter1/Rule1.md))를 제공할 수 있다.
   - 새로운 객체를 만드는 대신 기존 객체들을 공유하여 메모리 요구량과 쓰레기 수집 비용이 줄어든다.
   - 클라이언트 코드를 변경하지 않고도 캐시 기능을 추가할 수 있다.

 - 방어적 복사본을 만들 필요가 없다. -> 객체가 영원히 같은 상태


- 변경 불가능한 객체는 그 내부도 공유할 수 있다.
  - BigInteger 클래스의 값과 부호
  ```JAVA
  final int signum;  //부호
  final int[] mag;   //크기
  ```
   값의 부호와 크기를 각각 int 변수와 int 배열로 표현

  - negate 메서드
  ```JAVA
  public BigInteger negate() {
	   return new BigInteger(this.mag, -this.signum);
    }
  ```
   같은 크기의 값을 부호만 바꾸어서 새로운 BigInteger 객체로 반환.  
   그러나 배열은 복사하지 않는다. 새로운 BigInteger 객체도 원래 객체와 같은 내부 배열을 참조한다.


- 변경 불가능 객체는 다른 객체의 구성요소로도 훌륭하다.
 - 구성요소들의 상태가 변경되지 않는 객체는 설사 복잡하다 해도 훨씬 쉽게 불변식을 준수할 수 있다.
 - 맵(map), 집합(set) : 맵의 키나 집합의 원소로 활용하기 좋다. 한번 집어놓고 나면, 그 값이 변경되어 맵이나 집합의 불변식이 깨질 걱정은 하지 않아도 된다.

<br>
### 4. 변경 불가능 객체의 단점
- 값마다 별도의 객체를 만들어야 한다.  
-> 객체 생성 비용이 높을 가능성이 있다.

  ```JAVA
  BigInteger moby = ...;
  moby = moby.flipBit(0);
  ```

  flipBit 메서드는 새로운 BigInteger 객체를 생성할 것인데, 그 비트 개수도 역시 백만 개다.
  flipBit 연산에는 BigInteger 객체 크기에 비례하는 시간과 공간이 필요!

<br>
### 5. 변경 불가능 객체의 단점 극복 방법
- 다단계 연산 가운데 자주 요구되는 것을 기본연산으로 제공
 - 변경 불가능 클래스를 단계별로 만들 필요가 없다.
 - 다단계 연산이 하나의 기본 연산으로 축약되면 내부적으로 일 처리 가능
 - ex) String 클래스 : 변경 가능 동료 클래스(companion class)로 StringBuilder, ~~StringBuffer~~  
 BigInteger 클래스 : BitSet


<br>
### 6. 변경 불가능 클래스의 대안적 설계법
- 모든 생성자를 private이나 package-private으로 선언하고, public 생성자 대신 public 정적 팩터리를 제공하라.([규칙1](/Chapter1/Rule1.md))

  ```JAVA
  ///생성자 대신 정적 팩터리 메서드를 제공하는 변경 불가능 클래스
  public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public static Complex valueOf(double re, double im) {
      return new Complex(re, im);
    }
  }
  ```

  - 여러개의 package-private 구현 클래스를 활용할 수 있게 되어 유연성이 가장 좋다.
  - 해당 패키지 외부에서는 상속이 불가능하고, public이나 protected로 선언된 생성자가 없어 final로 선언된 것이나 마찬가지다.
  - 다양한 구현 클래스(implementation class)를 사용할 수 있다.
  - 정적 팩터리 메서드에 객체 캐싱 기능을 추가해서 성능 향상


<br>
### 7. 요약
- 변경 불가능 클래스 구현 규칙은, 어떤 메서드도 객체를 수정해서는 안되며, 모든 필드는 final로 선언되어야 한다.
- 시간이 많이 걸리는 계산 결과를 캐시(cache) 해두는 비-final필드를 가지고 있다. 따라서 캐시 값 반환을 통해 재계산 비용을 줄인다.
- 객체 상태를 변경하는 메서드(수정자 메서드 <sub>setter</sub> 등)를 제공하지 않는다.
- 변경 불가능 클래스의 public 변경 가능 동료 클래스 제공은 만족스러운 성능을 얻을 수 있다는 확신이 들 때만 써라.(String - StringBuilder/StringBuffer)
- 변경 불가능한 클래스로 만들 수 없다면, 변경 가능성을 최대한 제한하라. 되도록 모든 필드는 __final__ 로 선언하라.
- 생성자 이외의 public 초기화 메서드나 정적 팩터리 메서드를 제공하지 마라.
- 재초기화(reinitialize) 메서드도 제공하지 마라. ex) TimerTask class
