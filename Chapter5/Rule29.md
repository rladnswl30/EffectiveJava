## Rule29. 형 안전 다형성 컨테이너를 쓰면 어떨지 따져보라
- 제네릭은 Set이나 Map 같은 컬렉션(Collection)과 ThreadLocal, AtomicReference처럼 하나의 원소만을 담는 __컨테이너(container)__ 에 가장 많이 쓰인다.
- 컨테이너별로 형인자의 수는 고정된다.  
ex) Set에는 집합에 담길 원소의 자료형을 나타내는 형인자 1개,  
Map은 키와 value 각각에 대해 형인자 1개

### 제네릭 자료형 시스템을 통해 유연성을 보장하자
- 컨테이너 대신 키(key)에 형인자를 지정해라
- 컨테이너에 값을 넣거나 뺄 때마다 키를 제공
- 참고로, 자바 1.5부터 Class가 제네릭 클래스가 되었다.  
class의 리터럴 자료형은 Class< T >

```JAVA
//형 안전 다형성 컨테이너 패턴 - API
public class Favorites {
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}

//형 안전 다형성 컨테이너 패턴 - 클라이언트
public static void main(String[] args) {
  Favorites f = new Favorites();
  f.putFavorite(String.class, "n.d");
  f.putFavorite(Integer.class, 0xlikes);
  f.putFavorite(Class.class, Wst.class);
  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Integer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);
  System.out.printf("%s %x %s\n",
      favoriteString, favoriteInteger, favoriteClass.getName());
}
```

```JAVA
n.d likes Wst
```

Favorites 객체는 형 안전성을 보장하고, 다형성을 갖고 있다.
따라서, Favorites 같은 클래스를 __형 안전 다형성 컨테이너(typesafe heterogeneous container)__ 라고 부른다.

```JAVA
//형 안전 다형성 컨테이너 패턴의 구현
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();

  public <T> void putFavorite(Class<T> type, T instance) {
    if(type = null)
        throw new NullPointerException("Type is null");
    favorites.put(type, instance);
  }

  public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```
위 코드에는 두가지 이슈가 있다.  
1. Favorites 객체는 내부적으로 private Map<Class<?>, Object> 형의 favorites 필드를 이용한다.  
그런데, 비한정적 와일드카드를 사용했으므로 맵에 아무것도 넣을 수 없는 것 아닌가?  
-> 와일드카드 자료형이 중첩되어 있다.  
와일드카드 자료형이 쓰인 곳은 맵이 아닌 키! 모든 키가 상이한 형인자 자료형을 가질 수 있다.  
ex) Class< String >인 키도 있고, Class< Integer >인 키도 있을 수 있다.

2. Map이 키와 값 사이의 자료형 관계가 일치되도록, 자료형이 키가 나타내는 자료형이 되도록 보장하지 않는다.  
-> 맵에 저장된 객체를 꺼낼 때 그 사실을 이용할 수 있다.  
ex) getFavorite 메서드 구현 시 cast 메서드를 통해 __동적 형변환(dynamic cast)__ 을 한다.

위 코드에서 중요한 문제점 두가지가 있다.  
1. 클라이언트가 Favorite 객체의 형 안전성을 쉽게 깨트릴 수 있다.  
-> 무점검 경고 메시지가 뜸  
-> 해결법 : __동적 형변환__ 을 통해 안전성을 보장해라
```JAVA
//동적 형변환으로 실행시간 형 안전성 확보
public <T> void putFavorite(Class<T> type, T instance) {
  favorites.put(type, type.cast(instance));
}
```

2. 실체화 불가능 자료형(non-reifiable type)에는 쓰일 수 없다.  
String이나 String[]는 저장할 수 있으나, List< String >은 저장할 수 없다.(List< String >.class X)  
-> 해결법 : __한정적 자료형 토큰(bounded type token)__ 을 써라  
한정적 형인자나 한정적 와일드카드를 사용해서 표현 가능한 자료형을 제한
```JAVA
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```
annotationType이 __한정적 자료형 토큰__ 이고 리플렉션 객체인 T는 __형 안전 다형성 컨테이너__ 이다.  
이 메서드는 리플렉션 객체에 붙은 Annotation 가운데서 주어진 자료형에 해당하는 Annotation을 반환한다.

여기서, 무점검 형변환의 문제가 있다.  
-> Class에 asSubclass라는 형변환 안전성을 보장해주는 메서드가 존재한다.

```JAVA
//한정적 자료형 토큰으로 안전하게 형변환하기 위해 asSubClass를 이용한 사례
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
  Class<?> annotationType = null; //비한정적 자료형 토큰
  try {
    annotationType = Class.forName(annotationTypeName);
  } catch(Exception ex) {
    throw new IllegalArgumentExceptioin(ex);
  }
  return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```
```JAVA
public <U> Class<? extends U> asSubclass(Class<U> clazz) {
    if (clazz.isAssignableFrom(this))
        return (Class<? extends U>) this;
    else
        throw new ClassCastException(this.toString());
}
```

### 요약
- 컬렉션 API에서 컨테이너 대신 키를 제네릭으로 만들면 형인자 개수에 제약이 없는 __형 안전 다형성 컨테이너__ 를 만들 수 있다.
- 컨테이너는 Class 객체를 키로 쓰는데, 그런 Class객체를 __자료형 토큰__ 이라고 부른다.
- 키 자료형을 직접 구현할 수 있다.
