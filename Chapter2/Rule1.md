## Rule1. 생성자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해보라.

### 예시
```JAVA
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```  

===  

### 장점
 1. 생성자와는 달리 정적 팩터리 메서드에는 이름이 있다.
  - 정적 팩터리 메서드는 이름을 잘 짓기만 하면 사용하기 쉽고, 코드의 가독성도 높아진다.
  - API 사용자가 정적 팩터리 메서드의 이름을 보고 각각의 생성자 용도를 파악할 수 있다.  
<br>

 2. 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요는 없다.  
  - 다음 메서드가 좋은 예시다.
  ```JAVA
  Boolean.valueOf(Boolean)
  ```  
  - 어떤 시점에서 어떤 객체가 얼마나 존재할지를 정밀하게 제어할 수 있다(개체 통제 클래스).  
<br>

 3. 생성자와는 달리 반환값 자료형의 하위 자료형 객체를 반환할 수 있다.
  - public으로 선언되지 않은 클래스의 객체를 반환하는 API를 만들 수 있다.
  - 세부사항을 감출 수 있으므로 아주 간결한 API가 가능하다.
  - 인터페이스 기반 프레임워크 구현에 적합
  - 서비스 제공자 프레임워크(ex. JDBC)의 근간을 이루는 것이 바로 유연한 정적 팩터리 메서드이다.
   
   ```JAVA

   //서비스 인터페이스
   public interface Service {
   		//서비스에 고유한 메서드들이 이 자리에 온다.
   }
   
   //서비스 제공자 인터페이스
   public interface Provider {
   		Service newService();
   }
   
   //서비스 등록과 접근에 사용되는 객체 생성 불가능 클래스
   public class Services {
   		private Services() {} //객체 생성 방지

   		//서비스 이름과 서비스 간 대응관계 보관
   		private static final Map<string, Provider> providers = new ConcurrentHashMap<String, Provider>();
   		public static final String DEFAULT_PROVIDER_NAME = "<def>";

   		//제공자 등록 API
   		public static void registerDefaultProvider(Provider p) {
   			registerProvider(DEFAULT_PROVIDER_NAME, p);
   		}
   		public static vlid registerProvider(String name, Provider p) {
   			providers.put(name, p);
   		}

   		//서비스 접근 API
   		public static Service newInstance() {
   			return newInstance(DEFAULT_PROVIDER_NAME);
   		}
   		public static Service newInstance(String name) {
   			Provider p = providers.get(name);
   			if(p == null) {
   				throw new IllegalArgumentException("No provider registered with name: " + name);
   				return p.newService();
   			}
   		}
   }

   ```
<br>

 4. 형인자 자료형 객체를 만들 때 편하다.
   ```JAVA
   public static <K, V> HashMap<K, V> newInstance() {
    	return new HashMap<K, V>();
   }

   Map<String, List<String>> m = HashMap.newInstacne();
   ```  

===  

### 단점
1. 정적 팩터리 메서드만 있는 클래스를 만들면 생기는 가장 큰 문제는, 
public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다.  
<br>

2. 정적 팩터리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다.
따라서 클래스나 인터페이스 주석을 통해 정적 팩터리 메서드임을 널리 알리거나, 정적 팩터리 메서드 이름을 지을 때 조심하자.  
<br>

 - valueOf : 형변환 메서드. 인자로 주어진 값과 같은 값을 갖는 객체 반환
 - of : valueOf를 더 간단하게 쓴 것.
 - getInstance : 인자에 기술된 객체를 반환하지만, 인자와 같은 값을 갖지 않을 수 있다.
 - newInstance : getInstance와 같지만 호출할 때마다 다른 객체를 반환한다.
 - getType : getInstance와 같지만, 반환될 객체의 클래스와 다른 클래스에 팩터리 메서드가 있을 때 사용한다.
 - newType : newInstacne와 같지만, 반환될 객체의 클래스와 다른 클래스에 팩터리 메서드가 있을 때 사용한다.
