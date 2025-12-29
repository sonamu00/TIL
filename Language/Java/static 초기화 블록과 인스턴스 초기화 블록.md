## 초기화 블록이란
초기화 블록(initialization block)은 **생성자보다 먼저** 혹은 **클래스 로딩 시점에 실행되는 코드 블록**이다.

## static 초기화 블록
### 개념
- 클래스가 JVM에 의해 초기화될 때 최초 1번만 실행됨
- 클래스 공용 자원들을 클래스가 로딩될 때 초기화하고 인스턴스에서 공유해서 사용할 때 이 블록 사용함
	- 예시: 환경변수/설정 값 캐싱, 정규식
### 생성자 대신 사용하는 이유
- 생성자는 객체 단위로 매번 실행되기 때문에 static을 매번 덮어써서 초기화 로직이 반복됨 -> **static의 생명주기와 인스턴스 생명주기가 맞지 않음** 
### 리스크/사이드 이펙트
- static 블록에 DB나 네트워크 작업 같은 무거운 작업 넣음 -> 클래스 로딩이 느려짐
- 초기화 순서(위에서 아래)가 중요해서 **다른 static 필드와의 의존관계**가 꼬이면 버그남
### 대안
```java
class Whatever {
    public static varType myVar = initializeClassVariable();
        
    private static varType initializeClassVariable() {

        // initialization code goes here
    }
}
```
- prviate static 으로 메서드를 선언하고 public 필드에 static 메서드를 호출하는 식으로 초기화 가능
- static 필드는 클래스가 초기화될 때 최초 1번만 실행되고, 클래스 생성자보다 먼저 실행되어서 static 초기화 블록 대체 가능
- 추후에 다시 초기화가 필요할 때 메서드를 재사용할 수 있는 장점이 있음

## 인스턴스 초기화 블록
### 개념
- 객체를 생성할 때마다 생성자보다 먼저 실행
- 모든 생성자에서 공통으로 실행되어야 하는 인스턴스 초기화 로직이 있을 때 이 블록 사용함
- 생성자 체이닝 방법도 있어서 이 블록은 자주 사용하지 않음
### 리스크/사이드 이펙트
- 위/아래에 선언된 필드 초기화식과 얽히면 예상과 다른 값이 들어감
- 초기화 과정에서 오버라이딩된 메서드 사용 -> 자식 필드가 아직 준비 안 된 상태에서 실행되는 문제 발생
### 대안
- 생성자 체이닝 쓰기 -> 코드가 깔끔하고 재사용성 높음
- final 메서드로 초기화
	``` java
	class Whatever {
	    private varType myVar = initializeInstanceVariable();
	        
	    protected final varType initializeInstanceVariable() {
	
	        // initialization code goes here
	    }
	}
	```
	- 필드에서 인스턴스 초기화 메서드 호출해서 사용 가능
	- final 로 둬서 서브클래스에서 오버라이드하지 못하도록 막아야 됨 -> 안 막으면 하위클래스에서 상위클래스보다 먼저 초기화 로직을 실행하는 문제 발생 가능

---

## 참고 자료
- [오라클 자바 튜토리얼](https://docs.oracle.com/javase/tutorial/java/javaOO/initial.html)
