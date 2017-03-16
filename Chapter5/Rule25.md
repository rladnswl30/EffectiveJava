## Rule25. 배열 대신 리스트를 써라

### 배열은 제네릭 자료형과 두 가지 중요한 차이점을 갖고 있다.  

#### 1. 배열은 __공변 자료형(covariant)__ 인 반면, 제네릭은 __불변 자료형(invariant)__ 이다.
- 배열
 - Sub가 Super의 하위 자료형이라면, Sub[]도 Super[]의 하위 자료형이다.

  ```JAVA
  //실행 중 문제를 일으킴
  Object[] objectArray = new Long[1];
  objectArray[0] = "I don't fit in";  //ArrayStoreException 예외 발생
  ```
  Long 객체 컨테이너 안에 String 객체를 넣을 수 없다.

- 제네릭
 - Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 상위 자료형이나 하위 자료형이 될 수 없다.

 ```JAVA
//컴파일이 되지 않는 코드
List<Object> ol = new ArrayList<Long>();  //자료형 불일치
ol.add("I don't fit in");
 ```
 Long 객체 컨테이너 안에 String 객체를 넣을 수 없다.

#### 2. 배열은 __실체화(reification)__ 되는 자료형인 반면, 제네릭은 __삭제(erasure)__ 과정을 통해 구현된다.
- 배열은 각 원소의 자료형은 실행시간(runtime)에 결정되지만,   
제네릭은 자료형에 관계된 조건들은 컴파일 시점에만 적용되고, 각 원소의 자료형 정보는 프로그램이 실행될 때 삭제된다.
- 이러한 기본적인 차이점으로 인해 배열과 제네릭을 섞어쓰기 힘들다.
- 제네릭 배열 생성은 __형 안전성(typesafe)__ 이 보장되지 않는다.

```JAVA
// 제넬기 배열 생성이 허용되지 않는 이유 - 아래 코드는 컴파일되지 않는다!
List<String>[] stringLists = new List<String>[1];
List<Integer> initList = Arrays.asList(42);
Object[] objects = stringLists;
objects[0] = initList;
String s = stringLists[0].get(0);
```
List<String> 객체만을 담는다고 선언한 배열에서 List<Integer>를 저장했다.  
마지막 줄에서 배열의 Integer 원소를 꺼내면서 컴파일러가 자동적으로 String으로 변환하려고 할 것이다. -> ClassCastException 발생!

### 제네릭 배열 생성 오류에 대한 해결책
- 배열 대신 리스트를 쓰자. (E[] -> List<E>)
- 성능이 저하되거나 코드가 길어질 수 있지만, 형 안전성과 호환성이 좋아진다.


- 배열

```JAVA
//제네릭 없이 작성한 reduce 함수, 병행성(concurrency) 문제가 있다!
static Object reduce(List list, Function f, Object initVal) {
  synchronized(list) {
    Object result = initVal;
    for(Object o : List) {
      result = f.apply(result, o);
    }
    return result;
  }
}

interface Function {
  Object apply(Object arg1, Object arg2);
}
```
동기화(synchronized) 영역 안에서 "불가해 메서드(alien method)"를 호출하면 안된다.


동기화가 적용된 영역(synchronized) 안에서는 재정의 가능 메서드나 클라이언트가 제공한 함수 객체 메서드를 호출하지 말라는 의미  
-> __불가해(alien) 메서드이기 때문__ (제어권이 다른 객체로 위임되는데 위임되는 메서드 안에서는 동기화 제어를 할 수 없기 때문이다.)

```JAVA
//제네릭 없이 작성한 reduce 함수, 병행성(concurrency) 문제는 없다.
static Object reduce(List list, Function f, Object initVal) {
  Object[] snapshot = list.toArray(); //리스트에 내부적으로 락(lock)을 건다.
  Object result = initVal;
  for(Object o : snapshot) {
    result = f.apply(result, o);
  }
  return result;
}
```
락(lock)을 건 상태에서 리스트를 복사한 다음, 복사본에 작업하도록 reduce 메서드를 수정. (list.toArray 메서드는 내부적으로 리스트에 락을 건다)

- 제네릭

```JAVA
// reduce의 제네릭 버전 - 컴파일되지 않는다!
static <E> E reduce(List<E> list, Function<E> f, E initVal) {
  E[] snapshot = list.toArray();
  E result = initVal;
  for (E e : snapshot) {
    result = f.apply(result, e);
  }
  return result;
}

interface Function<T> {
  T apply(T arg1, T arg2);
}
```

프로그램 실행 시 E가 무슨 자료형이 될지 알 수 없어 안전하지 않다.

- 리스트를 사용하는 제네릭
```JAVA
//리스트를 사용하는 제네릭 버전 reduce
// reduce의 제네릭 버전 - 컴파일되지 않는다!
static <E> E reduce(List<E> list, Function<E> f, E initVal) {
  List<E> snapshot;
  synchronized(list) {
    snapshot = new ArrayList<E>(list);
  }
  E result = initVal;
  for (E e : snapshot) {
    result = f.apply(result, e);
  }
  return result;
}
```
실행 도중에 ClassCastException이 발생할 일이 없으며, 안전하다.

### 요약
- 제네릭과 배열이 따르는 자료형 규칙은 서로 많이 다르다.
- __배열은__ 공변 자료형이자 실체화 가능 자료형
- __제네릭은__ 불변 자료형(실행 시점에 해당 자료형 정보가 사라져 모든 정보를 이용할 수 없다)
- 배열은 컴파일 시간에 형 안전성을 보장하지 않지만, 제네릭은 안전성을 보장한다.
- 배열과 제네릭을 섞어 쓰다가 컴파일 오류나 경고 메시지를 만나면, __배열을 리스트로 바꾸자!__
