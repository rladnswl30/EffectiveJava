## Rule23. 새 코드에는 무인자 제네릭 자료형을 사용하지 마라

- 선언부에 형인자(type parameter)가 포함된 클래스는 제네릭(generic) 클래스나 인터페이스라고 부른다. 이는 __제네릭 자료형이다(generic type).__
- 제네릭 자료형은 __형인자 자료형(parameterized type) 집합__ 을 정의한다.
- 형인자 자료형 집합 : < 와 > 로 감싼 실 형인자(actual type parameter) 목록이 붙은 클래스나 인터페이스 들로 구성  
-> 제네릭 자료형의 형식 형인자(formal type parameter) 각각에 대응  
ex) List<String>은 원소 자료형이 String인 리스트를 나태내는 자료형
- 각 제네릭 자료형은 새로운 __무인자 자료형(raw type)__ 을 정의.
이는 실 형인자 없이 사용되는 제네릭 자료형이다.  
ex) List<E>

java 1.5 이전
```JAVA
//무인자 컬렉션 자료형. 앞으론 이렇게하면 안됨
private final Collection stamps = ... ;

// 실수로 Coin 객체를 넣음
stamps.add(new Coin( ... ));

//무인자 반복자 자료형. 앞으론 이렇게 하면 안됨
for(Iterator i = stamps.Iterator(); i.hasNext();) {
  //ClassCastException 예외 발생
  Stamp s = (Stamp) i.next();
}
```

제네릭을 써서, 컴파일러에게 컬렉션에 담길 객체의 자료형이 무엇인지 선언하자!

java 1.5 이후
```JAVA
//형인자 컬렉션 자료형 - 형 안전성(typesafe) 확보
private final Collection<Stamp> stamps = ...;
```
stamps 컬렉션에 Stamp 객체만 넣을 수 있다는 것이 명시적이며, 실제로 Stamp 객체만 삽입되도록 함.
다른 자료형의 객체를 넣어 컴파일하면 오류 메시지가 출력된다.

- 형인자 자료형을 쓰면, 컬렉션에서 원소를 꺼낼 때 형변환하지 않아도 된다.

```JAVA
//형인자 컬렉션 자료형에 대한 for-each문 : 형 안전성 보장
for(Stamp s : stamps) { //형변환 불필요
  ...
}

//형인자 반복자 자료형을 이용한 for문 : 형 안전성 보장
for(Iterator<Stamp> i = stamps.iterator(); i.hasNext();) {
  Stamp s = i.next(); //형변환 불필요
  ...
}
```

- 제네릭 자료형을 형인자 없이 사용할 수 있지만, 그래서는 안된다.  
-> 무인자 자료형을 쓰면 형 안전성이 사라지고, 제네릭의 장점 중 하나인 표현력(expressiveness) 측면에서 손해를 보게 됨!

- 왜 자바는 아직 무인자 자료형을 지원하는가 ?  
-> 호환성 때문. 제네릭이 도입되기 전인 java 1.5 이전의 코드들은 제네릭 없이 구현되어 있어서 무인자 자료형을 지원한다.
(이전 호환성(migration compatibility))

- List<Object>는 아무 객체나 넣을 수 있지만, List와 같은 무인자 자료형은 아무 객체나 넣을 수 없다.
-> List와 같은 무인자 자료형을 사용하면 형 안전성을 잃게 되지만, List<Object>와 같은 형인자 자료형을 쓰면 그렇지 않다.

```JAVA
//실행 도중에 오류를 일으키는 무인자 자료형(List) 사용 예시
public static void main(String[] args) {
  List<String> strings = new ArrayList<String>();
  unsafeAdd(strings, new Integer(42));
  String s = strings.get(0);
}

private static void unsavfeAdd(List list, Object o) { //무인자 자료형 list 사용
  list.add(o);
}
```
위 예시는 무인자 자료형 List을 쓰면서 실행 결과를 String으로 반환하려 했기에 ClassCastException 예외가 발생한다.
-> List<Object>로 바꾸면 예외 발생 안함!

- 컬렉션에 들어갈 원소들의 자료형을 모르고 상관할 필요가 없다면 __비한정적 와일드카드 자료형(unbounded wildcard type)__ 을 써라.

```JAVA
//비한정적 와일드카드 자료형 - 형 안전성과 유연성 만족
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
  int result = 0;
  for(Object o1, s1) {
    if(s2.contains(o1)) result ++;
    return result;
  }
}
```
무인자 자료형 Set을 쓰면 안전하지 않으므로 비한정적 와일드카드 Set<?>울 쓰자.

- 그러나, Collection<?>에는 null 이외에 어떤 원소도 넣을 수 없다.  
-> 제네릭 메서드를 사용하거나 한정적 와일드카드 자료형을 써라.

- 무인자 자료형을 쓰면 안된다 했지만, 그 규칙에도 사소한 예외 두가지가 있다.
 1. 클래스 리터럴(class literal)에는 반드시 무인자 자료형을 사용해야 한다.  
 List.class, String[].class -> OK
 List<String>.class List<?>.class -> X
 2. 제네릭 자료형에 instanceof 연산자를 적용할 때는 다음과 같이 하는 것이 좋다.

 ```JAVA
if(o instanceof Set) {  //무인자 자료형
  Set<?> m = (Set<?>) o;  //와일드카드 자료형
  ...
}
 ```
제네릭 자료형 정보가 프로그램이 실행될 때는 지워지기 때문에, instanceof 연산자는 비한정적 와일드카드 자료형 이외의 형인자 자료형에 적용할 수 없다. 따라서 무인자 자료형을 써도 OK다.

### 요약
- 무인자 자료형을 쓰면 실행 도중 예외 발생의 위험이 있어, 이젠 쓰지 말자.
- Set<Object>는 어떤 자료형의 객체도 담을 수 있는 집합의 형인자 자료형이며, Set<?> 는 모종의 자료형 객체만을 담을 수 있는 집합을 표현하는 와일드카드 자료형이다.
- 무인자 자료형은 안전하지 못하다.
