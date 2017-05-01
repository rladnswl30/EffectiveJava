## 규칙35. 작명 패턴 대신 어노테이션을 사용하라.
java 1.5 이전에는 도구나 프레임워크가 특별히 취급해야 하는 프로그램 요소를 구별하기 위해 __작명 패턴__ 을 썼다.  
ex) JUnit이라는 test 프레임워크를 사용하려면, 테스트 메서드의 이름을 test로 시작해야 했다.  
하지만 작명 패턴에는 몇 가지 큰 문제점이 있다.  
1. 철자를 틀리면 알아채기 힘들다.
2. 특정한 프로그램 요소에만 적용되도록 만들 수 없다.
3. 프로그램 요소에 인자를 전달할 마땅한 방법이 없다.

### annotation은 작명패턴의 모든 문제를 해결한다.
어노테이션 자료형(annotation type)을 이용하여 테스트 시에 자동으로 실행될 테스트 메서드를 지정할 뿐 아니라, 해당 메서드 안에서 예외가 발생하면 테스트가 실패한 것으로 보겠다는 사실을 명시할 수 있다.

```JAVA
//표식 어노테이션 자료형 선언
import java.lang.annotation.*;

/**
* 어노테이션이 붙은 메서드가 테스트 메서드임을 표시.
* 무인자(parameterless) 정적 메서드에만 사용 가능.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{
  ...
}
```

- @Retention(RetentionPolicy.RUNTIME) : Test가 실행시간(runtime)에도 유지되어야 한다.
- @Target(ElementType.METHOD) : Test가 메서드 선언부에만 적용할 수 있다. 클래스나 필드 등 다른 프로그램 요소에는 적용할 수 없다.

```JAVA
// 기존 Test 어노테이션의 예(표식 어노테이션)
public class Sample {
  @Test
  public static void m1() {} // 이 테스트는 성공해야 함.

  public static void m2() {}

  @Test
  public static void m3() { // 이 테스트는 실패해야 함.
    throw new RuntimeException("Boom");
  }

  @Test
  public void m5() {} // 잘못된 사용. static 메서드가 아님.
}
```
- @Test는 표식 어노테이션이라고 부른다.
- 아무 인자도 받지 않으며, "표식"의 구실만 한다.
- 철자가 틀리거나, Test를 메서드 선언부가 아닌 다른 곳에 붙이면 이 프로그램은 컴파일되지 않을 것이다.
- Test 어노테이션은 Sample 클래스가 동작하는 데에 직접적 영향을 미치지는 않는다.

### @Test annotation은 단점이 많으므로, 다음과 같이 사용해라.
```JAVA
import java.lang.reflect.*;
public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class testClass = Class.forName(args[0]);
    for(Method m : testClass.getDeclaredMethods()) {
      if(m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
          m.invoke(null);
          System.out.printf("Test %s failed : no exception%n", m);
        } catch (Throwable wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          Class<? extends Exception>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
          int oldPassed = passed;
          for (Class<? extends Exception> excType : excTypes) {
            if (excType.isInstance(exc)) {
              passed++;
              break;
            }
          }
          if (passed == oldPassed)
            System.out.printf("Test %s failed : %s %n", m, exc);
        } catch (Exception exc) {
          System.out.println("INVALID @Test : " + m);
        }
      }
    }

  }
}
```
- ExceptionTest 어노테이션이 붙은 메서드를 전부 찾아내서 reflection 기능을 활용해 실행한다.(Method.invoke 호출)

```JAVA
// ExceptionTest
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Exception>[] value();
}
```
- Exception을 계승한 클래스에 대한 Class 객체
- 어노테이션 사용자가 예외 자료형을 명시할 수 있도록 한다.
- 한정적 자료형 토큰

```JAVA
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class})
public static void doubleBad() {
  List<String> list = new ArrayList<String>();

  //자바 명세에는 다음과 같이 addAll을 호출하면
  //IndexOutOfBoundsException이나 NullPointerException이 발생한다고 명시되어 있다.
  list.addAll(5, null);
}
```

### 요약
- 프로그래머가 소스 파일에 정보를 추가할 수 있도록 하는 도구를 만들고 싶다면, 어떤 어노테이션 자료형이 필요한지 찾아서 알아서 만들어라.
- @Test는 단점이 많으니, 위 어노테이션을 만들어 사용하면 좋다.
- 어노테이션이 있으므로 더 이상 작명 패턴에 기대면 안된다.
- 위와 같은 어노테이션들은 표준화 된 것이 아니므로, 도구를 바꾸거나 새 표준이 재정될 경우 수정해야 한다.
