## String 리터럴과 String 객체 + 연산 차이
### 문자열 변수끼리 더하는 경우
```java
public class Test {
    public static void main(String[] args) {
        String a = "a";
        String b = "b";
        String result = a+b;
    }
}
```
- String 리터럴 값일 경우: 문자열 상수풀에서 문자열 가져옴

```java
public class Test {
    public static void main(String[] args) {
        String a = new String("a");
        String b = new String("b");
        String result = a+b;
    }
}
```
- new String 으로 값 생성한 경우: 인스턴스 생성하고 문자열 가져옴

#### 두 방식의 최적화 공통점
- 컴파일러가 StringBuilder로 변환해서	문자열 append 메서드로 추가 후 toString으로 반환하도록 최적화 되어 있음
	- 이 방식은 JDK 1.5 버전부터 적용
	- JDK 9 버전부터는 StringConcatFactory 클래스의 makeConcatWithConstants 메서드 사용해서 최적화

### 여러 문자열 리터럴끼리 더하는 경우
``` java
String a = "a" + "b" + "C" + "d"; 
```
- 연산 시 컴파일러가 우측항에 있는 리터럴들을 연산과정에서 결합하는 최적화를 함
- "abCd" 로 컴파일러가 문자열 상수풀에 저장함(컴파일 타임 상수 폴딩)



### JDK 9 부터 문자열 연산 최적화 방식
- **컴파일 타임**에 StringBuilder 코드로 바뀌지 않고 대신 여기서 문자열 합쳐야 된다는 **표지판(= invokedynamic 명령)** 만 남겨둠
- **런타임**에 JVM이 그 invokedynamic을 보고 **부트스트랩** 과정을 통해 JDK 내부의 `StringConcatFactory`를 호출해서 그 자리(그 코드 라인)에 딱 맞는 concat 함수를 **한 번 만들어서 연결(link)**
	- 부트스트랩(bootstrap): 처음 코드 지점이 실행될 때 JVM이 뭘 어떻게 호출할지를 한 번 결정해 연결해두고, 이후엔 그 결정을 재사용하는 초기 설정 과정
- 만들어둔 concat 함수는 재사용
- `StringConcatFactory`는 그 자리(그 코드 라인)에 딱 맞는 concat 함수를 **한 번 만들어서 연결(link)** 해주고 나중에 계속 사용


## 요약
- Java 8까지: `javac`가 `new StringBuilder().append(...).toString()`로 **고정 번역**
- Java 9부터: `javac`가 `invokedynamic`을 찍고, JVM이 처음 실행 때 **부트스트랩으로 concat 함수를 만들고 연결**

## 참고 자료
https://strong-park.tistory.com/entry/Java%EC%9D%98-String-%ED%83%80%EC%9E%85%EC%9D%98-%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EC%B5%9C%EC%A0%81%ED%99%94%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90