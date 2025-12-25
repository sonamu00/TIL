---
share: "true"
---
## 1. 인터페이스(interface)
### 1-1. 한 줄 정의
- 인터페이스는 **이 타입이 어떤 메서드를 가져야 하는지 정의만 해두는 설계도이자, 구현을 뒤로 미루고 동작만 약속하는 계약** 이다.
    
### 1-2. 문법·특징
- 선언: `public interface MyInterface { void doWork(); }`
- 인터페이스에 선언된 메서드는 기본적으로 **몸통이 없는 추상 메서드**
- 구현하는 클래스는 `implements` 사용
    - `public class MyService implements MyInterface { ... }`
- JDK 8 이후에는 `default` 메서드를 사용해 인터페이스 안에 **기본 구현**도 둘 수 있음
        
### 1-3. JDK 17 API 관점 – `Runnable` 예시
> OpenJDK 17의 `Runnable` 문서
 > The `Runnable` interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called `run`. 
- 스레드에서 실행할 작업을 추상화한 **공통 프로토콜**
- OS 레벨에서 “새 실행 흐름(스레드)”을 만들고, 자바는 그 안에서 어떤 코드를 돌릴지 `Runnable.run()`에 담도록 강제

```java
public interface Runnable {
    void run();
}
```

### 1-4. CS/운영체제 관점
- 운영체제는 여러 스레드/프로세스를 스케줄링하며 실행한다.
- 자바는 **스레드에서 수행할 작업을 인터페이스로 추상화**해서, `Thread`, `ExecutorService` 등에서 **동일한 타입(Runnable)으로 처리**할 수 있게 만든다.
- 즉, 인터페이스는 **실행 단위의 행동(메서드 시그니처)를 정의하는 계약** 역할.
    
### 1-5. 실무 관점(백엔드/보안 솔루션)
- 서비스/리포지토리 인터페이스 설계
    - `UserRepository`, `AssetScanService` 같은 인터페이스 정의 후 여러 구현체(`JdbcUserRepository`, `JpaUserRepository`, `MockUserRepository`)로 갈아끼우기 쉽다.
    - DB 종류 변경, API 연동 방식 변경, 테스트용 Mock 도입 등을 **구현 변경만으로 처리**할 수 있어 다형성과 확장성이 좋아진다.
        
---

## 2. 추상 클래스(abstract class)
### 2-1. 한 줄 정의
- 추상 클래스는 **공통 상태와 일부 동작은 구현해 두고, 나머지 핵심 동작만 하위 클래스가 구현하도록 강제하는 미완성 클래스** 이다.
    

### 2-2. 문법·특징
- 선언: `public abstract class AbstractWorker { ... }`
- 특징
    - **추상 메서드**(몸통 없음)와 **일반 메서드**(몸통 있음)를 같이 가질 수 있다.
    - **필드(상태)** 를 자유롭게 가질 수 있다.
    - 직접 인스턴스를 만들 수 없고, `extends`로 상속받은 하위 클래스만 인스턴스 생성 가능

### 2-3. JDK 17 API 관점 – `AbstractList`
- `AbstractList<E>`는 `List` 인터페이스를 구현하기 위한 **스켈레톤 구현(skeleton implementation)** 을 제공하는 추상 클래스다.
- 서브클래스는 `get`, `size` 같은 핵심 메서드만 구현하면, 나머지 공통 메서드는 기본 구현을 재사용 가능
        
### 2-4. CS/설계 관점
- 인터페이스: **무슨 기능을 해야 하는가(행동 규약)** 만 정의.
- 추상 클래스: **이 계열 클래스들 사이의 공통 구현** 을 모아서 중복을 줄이는 상위 타입
        

### 2-5. 실무 관점
- 공통 로깅/검증/템플릿 메서드 패턴
    - 예: `AbstractReportGenerator` 안에
        - `generate()`는 `final`로 흐름 전체 고정
        - `readData()`, `writeExcel()` 같은 단계 메서드는 `abstract`로 열어두고 하위 클래스에서 구현
- 여러 리포트 타입, 여러 스캔 타입이 있을 때 공통 흐름은 추상 클래스에 묶고, 세부 차이만 하위 클래스에서 구현.
    
---

## 3. 인터페이스 vs 추상 클래스
### 3-1. 공통점
- 둘 다 직접 인스턴스 생성 불가 (인터페이스는 구현체 필요, 추상 클래스는 하위 클래스 필요)
- 둘 다 하위 타입에게 메서드 구현을 강제해서 **다형성** 구현에 쓰인다.
    

### 3-2. 차이점 정리
- **상태(필드)**
    - 인터페이스: 상수(`public static final`)만 가질 수 있고, 일반 인스턴스 필드를 두지 않는다.
    - 추상 클래스: 인스턴스 필드, protected 필드 등 상태를 자유롭게 가질 수 있다.
- **상속 개수**
    - 인터페이스: 다중 구현 가능(`implements A, B, C`)
    - 추상 클래스: 단일 상속만 가능(`extends`는 하나만)
- **역할 vs 계층**
    - 인터페이스: “이 역할을 할 수 있다”는 **능력/프로토콜** 중심 타입 (예: `Runnable`, `Comparable`)
    - 추상 클래스: 공통 구현을 모은 **상속 계층의 상위 타입** (예: `AbstractList`, `AbstractMap`)
        

### 3-3. 공개 API vs 내부 구현
- 외부에 “우리 모듈은 이런 기능을 제공합니다” 라고 **약속할 때** → **인터페이스**로 공개
- 내부에서 공통 코드와 흐름을 재사용할 때 → **추상 클래스**로 뼈대 구현
    
---

## 4. final

### 4-1. 공통 의미
- `final`은 어디에 쓰든 **“이 지점 이후로는 더 이상 변경/확장할 수 없다”** 라는 뜻이다.
    
### 4-2. final class
- 예: JDK 17 `String` 선언
    
```java
public final class String extends Object implements Serializable, Comparable<String>, CharSequence, Constable, ConstantDesc
```

- 의미: `String`을 상속받은 서브클래스를 만들 수 없다.
- 이유:
    - `String`의 **불변성(immutability)** 과 여러 최적화/보안 가정을 깨는 것을 막는다.
    - 전역적으로 많이 쓰이는 핵심 타입이므로, 서브클래스로 이상한 동작을 끼워 넣는 것을 차단.
        
### 4-3. final method
- 메서드에 `final`을 붙이면 하위 클래스에서 **오버라이딩 금지**
- 템플릿 메서드 패턴 등에서 전체 흐름을 고정하고, 세부 단계 메서드만 오버라이드 허용할 때 사용
    

### 4-4. final 변수
- 기본형:
```java
final int port = 8080;
```
- `port` 값은 이후에 변경할 수 없다.
    
- 참조형:
```java
final List<String> list = new ArrayList<>();
list.add("A");      // OK
// list = new ArrayList<>(); // ❌ 다른 객체를 가리키게 변경 불가
```
- 참조가 가리키는 **객체 자체를 바꾸는 것은 불가능**
- 하지만 그 객체의 **내부 상태(필드 값)는 변경 가능**

---

## 5. enum
### 5-1. 한 줄 정의
- enum은 **한정된 개수의 상수/상태를 이름 있는 값으로 묶은 타입 안전한 상수 집합이자, `java.lang.Enum`을 상속받는 특수한 클래스** 이다.
    

### 5-2. 문법·특징
```java
public enum Severity {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
}
```
- enum으로 선언한 클래스는 java.lang.Enum 클래스의 상속을 자동으로 받는다.
- enum 클래스에 선언되어 있지는 않지만 컴파일 시 자동으로 추가되는 상수의 목록을 배열로 리턴하는 메소드는 values()이다.
        

### 5-3. JDK 17 API – Enum 클래스
- `Enum` 선언 (문서):
```java
public abstract class Enum<E extends Enum<E>>
        extends Object
        implements Constable, Comparable<E>, Serializable
```
- 모든 enum 타입은 자동으로 이 클래스를 상속한다.
- `Enum`에서 제공하는 대표적인 메서드들: `name()`, `ordinal()`, `compareTo()` 등.
- 각 enum 타입에는 **컴파일러가 자동으로** `static T[] values()`, `static T valueOf(String name)`  를 만들어 준다.
        
### 5-4. CS/운영체제 관점
- OS의 프로세스 상태처럼 **유한 개의 상태 집합**을 표현하기에 적합함
    - `NEW`, `READY`, `RUNNING`, `BLOCKED`, `TERMINATED` 등을 enum으로 표현하면, 컴파일 시점에 잘못된 상태를 잡을 수 있고, 상태 전이 로직이 더 읽기 쉬운 상태 머신이 된다.
        

### 5-5. 실무 관점
- 숫자 코드/문자열 상수 대신 enum을 사용 시
    - 타입 안전성 (오타/잘못된 코드 값 방지)
    - IDE 지원(자동완성, 사용처 찾기)
    - 리팩토링 시 변경 범위 추적이 쉽다.  
- 예: 자산 탐색 상태, 스캔 결과 상태, 취약점 심각도, 사용자 권한 레벨 등을 enum으로 관리.
        
