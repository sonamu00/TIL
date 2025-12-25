---
share: "true"
---

## 1. Object 개요
- 자바에서 명시적으로 다른 클래스를 상속하지 않으면 자동으로 `java.lang.Object`를 상속받는다.
- 자바는 이중상속을 받을 수 없지만, 여러 단계로 상속 가능하기 때문에 extends에 없어도 Object 클래스를 사용할 수 있다.
- Object 클래스를 자동으로 상속 받는 이유
	- Obejct 클래스의 메서드를 통해서 클래스의 **기본적인 행동 정의**
	- 클래스라면 이 정도의 메서드는 정의되어 있어야 하고 처리해 주어야 한다는 것을 Object 클래스 상속으로 정의함
		- 모든 객체가 공통으로 가져야 하는 최소 메서드 집합(`equals`, `hashCode`, `toString`, `clone`, `wait`, `notify`, `notifyAll`, `getClass` 등)을 제공
- 모든 참조 타입을 `Object` 타입 하나로 다룰 수 있기 때문에 컬렉션 내부 저장소, 메서드 매개변수, 리턴 타입 등을 공통 타입으로 처리할 수 있고 이것이 다형성(polymorphism)의 기반이 된다.
    
---

## 2. `==` 와 `equals()` 비교
### 2-1. `==` (동일성: identity)
- 기본형(primitive)에서는 실제 값이 같은지 비교한다.
- 참조형(reference)에서는 두 변수가 **같은 객체(같은 주소)를 가리키는지** 비교한다.
- 참조형에서 `==`는 “힙에 있는 같은 인스턴스를 가리키고 있는가?”를 검사하는 **동일성(identity) 비교 연산자**이다.
    

### 2-2. `equals()` (동등성: equality)
- `Object.equals()`의 기본 구현은 내부에서 `this == obj`와 동일하게 동작하여 기본적으로는 동일성을 비교한다.
- 구현 코드
``` java
public boolean equals(Object obj) {  
    return (this == obj);  
}
```
- 의미 있는 “내용 비교”를 하고 싶으면 각 클래스에서 `equals()`를 오버라이드해서 두 객체의 **논리적인 동등성(내용 기준)**을 정의해야 한다.
- `String.equals()`는 문자열 내용을 비교하도록 오버라이드되어 있어서 서로 다른 `new String("hello")` 두 개라도 내용이 같으면 `true`를 반환한다.
- 구현 코드
    ``` java
	public boolean equals(Object anObject) {  
	if (this == anObject) {  
		return true;  
	}    return (anObject instanceof String aString)  
			&& (!COMPACT_STRINGS || this.coder == aString.coder)  
			&& StringLatin1.equals(value, aString.value);  
	}
    ```
- 실무에서는 엔티티(예: User), 값 객체(예: Money, Point) 등에서 “무엇을 같다고 볼지” 기준을 정하고 그 기준에 따라 `equals()`를 오버라이드한다.
    

---

## 3. `hashCode()`와의 관계
- 객체를 식별하기 위한 **int 정수 해시값**을 반환해서 해시 기반 컬렉션(`HashMap`, `HashSet`, `ConcurrentHashMap` 등)에서 어느 버킷에 저장하고 검색할지 결정하는 데 사용된다.
- 핵심 규칙: 두 객체에 대해 `a.equals(b)`가 `true`라면 반드시 `a.hashCode()`와 `b.hashCode()`도 같아야 한다.
	- 반대의 케이스(`a.equals(b)`가 `false`) 는 안 같아도 된다.
	- 해시값이 같다고 해서 항상 `equals()`도 `true`일 필요는 없고 해시 충돌은 허용된다.
- `Object.toString()` 기본 구현
	- 구현 코드
		``` java
		public String toString() {  
		    return getClass().getName() + "@" + Integer.toHexString(hashCode());  
		}
		```
	- 정수형인 해시값을 16진수로 변환하고 `클래스명@Integer.toHexString(hashCode())` 형식으로 반환한다
		- 해시값: 복잡한 객체를 해시 테이블(HashMap, HashSet)에서 관리하기 위해 만든 대표 번호
		- 번호표(= hashCode())를 가지고 `번호표 % 배열길이` 같은 계산으로 “몇 번 칸(버킷)에 넣을지”를 먼저 정하고, 그 칸 안에 들어 있는 값들 중에서 `equals()`로 하나씩 비교해서 “진짜 같은 객체인지” 최종 확인한다.
	- 객체를 toString으로 출력했을 때 로그에서 보는 `MyClass@1a2b3c` 같이 보이는 값은 단지 `hashCode()`를 16진수로 바꿔서 보여주는 “식별용 숫자”일 뿐이고, JVM이 내부에서 객체를 어디에 어떻게 배치하는지는 알 수 없다. `hashCode()` 값이 진짜 물리적인 메모리 주소와 1:1로 같다고 생각하면 안 된다.
    
---

## 4. `toString()` 과 로그·디버깅
- `System.out.println(obj)`를 호출하면 내부에서 `String.valueOf(obj)`가 호출되고 이 메서드는 `obj == null ? "null" : obj.toString()`을 호출하므로 결국 `toString()` 결과를 출력한다.
- `Object.toString()`의 기본 구현은 `클래스명@해시코드(16진수)` 형태라서 객체 내부 상태를 파악하는 데 큰 도움이 되지 않는다. 내부 상태를 파악할 때에는 `Object.toString()` 메서드를 오버라이딩해서 사용한다.
- `toString()`을 오버라이딩해서 사용하는 이유
	- 도메인 객체(엔티티, DTO, VO 등)에서 `toString()`을 오버라이드하면 객체의 주요 필드들을 한 번에 문자열로 확인할 수 있어서 디버깅과 로그 출력이 매우 편해진다.
	- 여러 곳에서 `id`, `name` 등을 일일이 이어붙여 로그를 남기는 대신 `log.debug("user={}", user);`처럼 객체 한 번만 넘기고 포맷은 `toString()` 한 군데에서 관리하면 필드가 늘거나 줄어도 `toString()`만 수정하면 되어 유지보수성이 좋아진다.

    
---

## 5. `clone()`과 얕은 복사
- `Object.clone()` 선언은 `protected native Object clone() throws CloneNotSupportedException;` 이고 기본 구현은 **얕은 복사(shallow copy)**를 수행한다.
- 얕은 복사
	- 새 인스턴스를 만들고 모든 필드를 “대입하듯이” 그대로 복사하는 방식
	- **기본형 필드는 값이 그대로 복사**
	- **참조형 필드는 참조값만 복사**
	- 참조형 필드는 원본과 복제 객체가 같은 하위 객체를 공유하게 된다.
- 깊은 복사
	- 새 인스턴스를 만들고, 그 안에 들어 있는 **참조형 필드가 가리키는 객체들까지 전부 새로 만들어 복사하는 방식**
	- 기본형 필드는 값 그대로 복사 (얕은 복사와 동일)
	- 참조형 필드는 참조값을 그대로 쓰지 않고, **참조가 가리키는 객체를 새로 생성한 뒤 그 값들을 복사**
	- 원본 객체와 복제 객체가 서로 다른 하위 객체를 가지므로, 한쪽에서 내부 객체를 수정해도 다른 쪽에는 전혀 영향을 주지 않는다
	- 구조가 복잡할수록(리스트 안에 리스트, 맵 안에 객체 등) 깊은 복사 구현 비용이 크고, 모든 참조 필드를 일일이 새로 만들어 채워줘야 한다
- `Object.clone()`을 그대로 호출하려면 해당 클래스가 `Cloneable` 인터페이스를 구현하고 있어야 하며, 구현이 없으면 `CloneNotSupportedException`이 발생한다.
- `Cloneable`은 메서드가 없는 마커 인터페이스로 “이 클래스는 clone을 허용한다”라는 의도를 표시하는 역할만 한다.
- 배열은 예외적으로 모든 타입에 대해 자동으로 `Cloneable` 취급되므로 `someArray.clone()`은 바로 사용 가능하다.
- 깊은 복사가 필요할 경우 `clone()`을 오버라이드해서 내부 참조 필드들도 새 객체로 복사하거나 생성자, 팩토리 메서드, 빌더 등 다른 패턴으로 복사 로직을 직접 구현하는 방식을 실무에서 더 많이 사용한다.
    
---

## 6. native 메서드와 JVM
``` java
@IntrinsicCandidate
public final native void notify();
```
- Object 클래스의 구현 소스를 보면 선언부에 `native` 와 `@IntrinsicCandidate` 어노테이션을 사용하는 메서드가 있다.
- `native` 키워드가 붙어 있는 메서드는 “자바 코드 안에 구현부가 없고 실제 동작은 JVM이 로드한 C/C++ 같은 native 코드에서 수행되는 메서드”를 의미한다.
- `Object` 안에서도 `hashCode()`, `getClass()`, `clone()` 등은 `native`로 선언되어 있고 자바 쪽에는 시그니처와 규약만 있으며 실제 구현은 JVM에 있다.
- `@IntrinsicCandidate` 어노테이션은 “이 메서드는 JVM이 더 빠른 intrinsic 코드로 치환해 최적화할 수 있다”는 힌트일 뿐이고 네이티브 여부를 결정하는 것은 오직 `native` 키워드이다. (OpenJDK 17버전부터 추가됨)
- OpenJDK 를 보면 시간 조회(`System.currentTimeMillis()`), 메모리 복사(`System.arraycopy()`), 스레드 생성 및 관리, 파일·소켓 I/O처럼 OS와 직접 맞닿는 기능들은 대부분 native 메서드를 통해 구현되어 있다.
    
---

## 7. `wait()` / `notify()` / `notifyAll()`
### 7-1. 개념 (쉽게)
- `wait()`는 “나 지금 할 일이 없으니까 자리(락)를 반납하고 밖에서 기다릴게, 준비되면 나를 깨워줘”에 해당한다.
- `notify()`는 “이 객체를 기다리던 스레드 중 한 명, 이제 들어와도 돼”라는 신호이다.
- `notifyAll()`은 “이 객체를 기다리던 스레드 전부, 한 번씩 들어올 준비해”라는 신호이다.

### 7-2. 사용 규칙과 동작
- `wait()`, `notify()`, `notifyAll()`은 반드시 해당 객체에 대한 `synchronized(obj) { ... }` 블록 안에서 `obj.wait()`, `obj.notify()`처럼 호출해야 하며 그렇지 않으면 `IllegalMonitorStateException`이 발생한다.
- 스레드가 `obj.wait()`를 호출하면 현재 가지고 있던 그 객체의 모니터 락을 반납하고 그 객체를 기다리는 대기열에 들어가 `WAITING` 또는 `TIMED_WAITING` 상태로 잠든다.
- 다른 스레드가 같은 객체에 대해 `synchronized(obj) { obj.notify(); }` 또는 `notifyAll();`을 호출하면 기다리던 스레드들이 깨워지지만 깨운 시점에도 락은 여전히 호출한 스레드가 쥐고 있고 호출한 스레드가 `synchronized` 블록을 빠져나와 락을 반납해야 깨워진 스레드들이 다시 락을 얻고 `wait()` 다음 줄부터 실행을 재개할 수 있다.

### 7-3. `sleep()`과의 차이
- `Thread.sleep(ms)`는 단순히 해당 스레드를 일정 시간 동안 재우는 것으로 락을 반납하지 않기 때문에 `synchronized` 블록 안에서 `sleep()`을 호출하면 그 동안에도 락은 계속 쥔 상태로 유지된다.
- `obj.wait()`는 락을 내려놓고 조건이 만족될 때까지 기다리기 때문에 다른 스레드가 들어와 상태를 변경한 뒤 `notify`/`notifyAll`로 깨워줄 수 있다.
- 정리하면 `sleep()`은 “그냥 나 혼자 쉰다”이고 `wait()`는 “자리(락)를 양보하고 조건이 만족될 때까지 기다린 뒤 다시 들어온다”이다.
    

---

## 참고
- [이펙티브 자바의 clone() 메서드 설명에 대한 질문](https://stackoverflow.com/questions/11346759/clone-implementation-in-object-class?utm_source=chatgpt.com)