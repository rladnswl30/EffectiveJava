## Rule21. 전략을 표현하고 싶을 때는 함수 객체를 사용하라

- 함수 포인터(function pointer), 대리자(delegate), 람다 표현식(lambda expression)처럼, 특정 함수를 호출할 수 있는 능력을 저장하고 전달할 수 있도록 하는 것들이 있다.
- 인자로 전달되는 함수는 호출된 함수의 기능을 변경하는 구실을 함

ex) C 표준 라이브러리의 qsort
```C
#include <stdio.h>
#include <stdlib.h>

int values[] = { 88, 56, 100, 2, 25 };

int cmpfunc (const void * a, const void * b)
{
   return ( *(int*)a - *(int*)b );
}

int main()
{
   int n;

   qsort(values, 5, sizeof(int), cmpfunc);

   printf("\nAfter sorting the list is: \n");
   for( n = 0 ; n < 5; n++ )
   {   
      printf("%d ", values[n]);
   }

   return(0);
}
```
- 어떤 비교자 함수를 인자로 주느냐에 따라 정렬 순서 변경
- __전략 패턴(starategy pattern)__ 의 사례

<br>
- 자바는 함수 포인터를 지원하지 않는 대신, __객체 참조__ 를 통해 전략 패턴을 구현
- __함수 객체(function object)__ : 가지고 있는 메서드가 다른 객체에 작용하는 메서드(인자로 전달된 객체에 무언가를 하는 메서드)가 하나뿐인 객체. 포인터 구실을 한다.

ex) 실행 가능 전략(concrete starategy) 예시
```JAVA
class StringLengthComparator {
  public int compare(String s1, String s2) {
    return s1.length() - s2.length();
  }
}
```
- 문자열을 비교하는데 사용될 수 있는 실행가능 전략
- 무상태(stateless) 클래스
- 필드가 없어, 모든 객체는 기능적으로 동일하다. 따라서 __싱글턴 패턴(singleton pattern)__ 을 따르면 쓸데없는 객체 생성을 피할 수 있다. ([규칙3](/Chapter2/Rule3.md), [규칙5](/Chapter2/Rule5.md))

ex) 싱글턴 패턴을 이용한 실행 가능 전략 예시
```JAVA
class StringLengthComparator {
  private StringLengthComparator() {};
  public static final StringLengthComparator
    INSTANCE = new StringLengthComparator();
  public int compare(String s1, String s2) {
    return s1.length() - s2.length();
  }
}
```

- 위 객체를 메서드에 전달하기 위해서는 인자의 자료형이 맞아야 하며, 위 객체를 인자로 받는 메서드에 다른 전략을 전달할 수 없다.
- 따라서, 실행 가능 전략 클래스에 어울리는 __전략 인터페이스(strategy interface)__ 를 정의해라

ex) 전략 인터페이스
```JAVA
import java.util.Comparator;

// 전략 인터페이스
public interface Comparator<T> {
  public int compare(T t1, T t2);
}
```
- java.util 패키지에 포함된 제네릭(generic) 인터페이스
- T는 자료형 인자(formal type parameter)

ex) Comparator<String> 인터페이스를 구현하는 StringLengthComparator 클래스
```JAVA
class StringLengthComparator implements Comparator<String> {
  ...
}
```

- 실행 가능 전략 클래스는 __익명 클래스(anonymouse class)__ 로 정의하는 경우도 많다.

ex) 익명 클래스로 정의한 실행 가능 전략 클래스
```JAVA
Arrays.sort(stringArray, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return s1.length() - s2.length();
  }
})
```
- 위처럼 익명 클래스를 사용하면 sort 호출 시마다 새로운 객체가 만들어지므로 함수 객체를 private static final 필드에 저장하여 재사용 하는 것 고려!
- 전략 인터페이스는 실행 가능 전략 객체들의 자료형 구실을 한다. 따라서 실행 가능 전략 클래스는 굳이 public으로 공개할 필요가 없다.
- 대신, 전략 인터페이스가 자료형인 public static 필드를 갖는 __호스트 클래스(host class)__ 를 정의해보자

```JAVA
//실행 가능 전략들을 외부에 공개하는 클래스
class Host {
  //실행 가능 전략 클래스 - private 중첩 클래스(nested class)
  private static class StrLenCmp implements Comparator<String>, Serializable {
    public int compare(String s1, String s2) {
      return s1.length() - s2.length();
    }
  }

  //외부에 공개
  //이 비교자는 직렬화 가능
  public static final Comparator<String>
    STRING_LENGTH_COMPARATOR = new StrLenCmp();

  ...
}
```
[직렬화?](http://flowarc.tistory.com/entry/Java-%EA%B0%9D%EC%B2%B4-%EC%A7%81%EB%A0%AC%ED%99%94Serialization-%EC%99%80-%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94Deserialization)

- String 클래스에도 위와 같은 패턴 사용(CASE_INSENSITIVE_ORDER)

### 요약
- 함수 객체의 주된 용도는 __전략 패턴__ 을 구현하는 것
- 자바에서는 전략을 표현하는 인터페이스를 선언하고, 실행 가능 전략 클래스로 해당 인터페이스 구현
- 실행 가능 전략이 한 번만 사용되는 경우 -> 익명 클래스 객체로 구현
- 실행 가능 전략이 여러번 사용되는 경우 -> private static 멤버 클래스로 전략을 표현하고, public satic final 필드를 통해 외부에 공개하자
