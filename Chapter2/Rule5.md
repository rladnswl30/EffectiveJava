## Rule5. 불필요한 객체는 만들지 말라
### 1. 변경 불가능 객체는 언제나 재사용할 수 있다.
- 피해야 하는 예
```JAVA
String s = new String("stringette");
```
- 바람직한 예
```JAVA
String s = "stringette";
```
이렇게 실행하면 실행할 때마다 객체를 만드는 대신, 동일한 String 객체를 사용한다.
그리고, 같은 가상머신에서 실행되는 모든 코드가 해당 객체를 재사용한다.

<br>

- 생성자 대신 정적 팩터리 메서드를 이용하면 불필요한 객체 생성을 피할 수 있다.
 - 피해야 하는 예 (생성자)
 ```JAVA
 new Boolean(String)
 ```
 - 바람직한 예 (정적 팩터리 메서드 이용)
 ```JAVA
 Boolean.valueOf(String)
 ```

### 2. 변경 가능한 객체도 재사용할 수 있다.
- 피해야 하는 예
```JAVA
public class Person {
  private final Date birthDate;

  //이렇게 하면 안된다!
  public boolean isBabyBoomer() {
    //생성 비용이 높은 객체를 쓸데없이 생성한다.
    Calendar gmtCal =
      Calendar.getInstance(TimeZone.getTimeZone("GMT"));

    gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomStart = gmtCal.getTime();
    gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomEnd = gmtCal.getTime();

    return birthDate.compareTo(boomStart) >= 0 &&
      birthDate.compareTo(boomEnd) < 0;
  }
}
```
해당 메서드가 호출될 때마다 Calendar 객체, TimeZone 객체, Date 객체를 쓸데없이 만들어낸다.


- 바람직한 예 (정적 초기화 블록)
```JAVA
public class Person {
  private final Date birthDate;

  /**
   * 베이비 붐 시대의 시작과 끝
   */
   private static final Date BOOM_START;
   private static final Date BOOM_END;

   static {
     Calendar gmtCal =
       Calendar.getInstance(TimeZone.getTimeZone("GMT"));

     gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
     BOOM_START = gmtCal.getTime();
     gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
     BOOM_END = gmtCal.getTime();
   }

  public boolean isBabyBoomer() {
    return birthDate.compareTo(BOOM_START) >= 0 &&
      birthDate.compareTo(BOOM_END) < 0;
  }
}
```
이렇게 하면 Calendar, TimeZone, Date 객체를 클래스가 초기화 될 때 한 번만 만든다.
 - 정적 초기화 블록을 통해 성능 개선, 코드 간결화
 - boomStart와 boomEnd를 지역변수 -> static final로 바꾸어 읽기 좋은 코드로 개선
 - 초기화 지연 기법을 통해 BOOM_START, BOOM_END를 isBabyBoomer 메서드 호출 시 초기화 할 수 있지만, 위 코드에서는 구현이 복잡해지고, 성능 개선이 어려워 추천하지 않는다.

### 3. 객체 표현형 대신 기본 자료형을 사용하고, 생각지도 못한 자동 객체화(autoboxing)가 발생되지 않도록 유의하자.
- 피해야 하는 예
```JAVA
public static void main(String[] args) {
  Long sum = 0L; // long이 아니라 Long으로 선언됨.
  for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
  }
  System.out.println(sum);
}
```
 - Long sum이 더해질 때마다 long i가 하나씩 생성된다.
 - 객체를 만들어서 코드의 명확성과 단순성을 높이고, 프로그램 능력을 향상시킬 수 있다면, 위 방식처럼 만들어도 좋다.

### 4. 직접 관리하는 객체 풀(object pool)을 만들어 객체 생성을 피하는 기법
 - 객체 생성 비용이 극단적으로 높지 않다면 사용하지 말자.
 - 좋은 예 : 데이터베이스 연결(DB 접속 비용이 충분히 높으므로 재사용하는 것이 좋음)
 - 최산 JVM은 고도화된 쓰레기 수집기를 갖고 있기 때문에, 가벼운 객체라면 객체 풀 생성을 피해라.

### 5. 방어적 복사
 - 새로운 객체를 만들어야 한다면 기존 객체는 재사용하지 말라.
 - 방어적 복사가 요구되는 상황일 때, 객체 재사용 비용은 쓸데없이 같은 객체를 여러번 생성하는 비용보다 훨씬 높다.
 - 방어적 복사를 하지 않는다면, 버그나 보안 결함이 생길 수 있고, 성능에 영향을 미친다.
