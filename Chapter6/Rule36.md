## 규칙36. Override Annotation은 일관되게 사용하라.
java 1.5에 Annotation이 도입되었을 때, 자바 라이브러리에 몇 가지 Annotation 자료형이 추가되었다. 그 중, 가장 중요하게 쓰이는 것이 __Override__ 일 것이다.  
__@Override__ 는 메서드 선언부에만 사용할 수 있고, 상위 자료형(supertype)에 선언된 메서드를 재정의한다는 것을 표현한다.

```JAVA
public class Bigram {
  private final char first;
  private final char second;

  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }

  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }

  public int hashCode() {
    return 31 * first + second;
  }

  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<Bigram>();
    for (int i = 0; i < 10; i++) {
      for (char ch = 'a'; ch <= 'z'; ch++) {
        s.add(new Bigram(ch, ch));
      }
    }
    System.out.println(s.size());
  }
}
```

위 프로그램을 실행하면 26이 아닌 260이 찍힌다.  
equals 메소드는 재정의(overriding) 대신 오버로딩(overloading) 되었다.  
equlas 메소드를 재정의 하려면 자료형을 Object로 해야 한다.
따라서,

```java
@Override
public boolean equals(Object o) {
  if (!(o instanceof Bigram))
    return false;
  Bigram b = (Bigram) o;
  return b.first == first && b.second == second;
}
```

이처럼, 상위 클래스에 선언된 메서드를 재정의할 때는 반드시 선언부에 Override Annotation을 붙여야 한다.  

예외) 비-abstrat 클래스에서 abstract 메서드 재정의 시, Override Annotation을 붙이지 않아도 된다. 어차피 컴파일 시 오류를 내주기 때문.

### 요약
- 상위 자료형에 선언된 메서드를 제정의하는 모든 메서드에 @Override를 붙이면 많은 오류를 막을 수 있다.
