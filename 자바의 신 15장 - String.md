---
share: "true"
---
## 1. String 클래스 선언부와 특징
### 1-1. 선언부
OpenJDK 17 기준 `String` 선언
```java
public final class String
    extends Object
    implements Serializable, Comparable<String>, CharSequence
```
문서 원문:
> “The `String` class represents character strings.”

→ **문자열을 표현하는 클래스**라는 뜻.

### 1-2. 불변(Immutable) 특성

문서 원문:

> “Strings are constant; their values cannot be changed after they are created.”

→ 한 번 생성된 `String`의 **값은 절대 바뀌지 않는다.**

추가 문장:

> “String buffers support mutable strings. Because String objects are immutable they can be shared.”

→ `String`은 불변이라서 **여러 곳에서 공유해도 안전**하고,  
변경이 필요한 문자열은 `StringBuffer`/`StringBuilder` 같은 “mutable string”이 담당한다.

**정리**
- `final class` + 불변
    - 내용 변경 X
    - 여러 스레드에서 동시에 읽기만 하는 사용에는 **동기화 없이도 Thread-safe**
	    - **동기화**: 공유 데이터를 여러 스레드가 접근할 때 **한 번에 하나씩만 수정하게 하고(락)**, 서로의 **변경사항이 보이도록 메모리 가시성을 보장**하는 것
    - 상수 풀에서 한 객체를 공유하기 쉽다.
        
---

## 2. 문자열 리터럴과 String 상수 풀(constant pool)
### 2-1. 문자열 리터럴
- `"hello"`, `"java"` 처럼 **소스 코드에 값 그대로 적혀 있는 문자열**을 “문자열 리터럴”이라고 한다.
- `String s = "hello";` 에서 리터럴은 `"hello"`, 변수는 `s`.

### 2-2. 상수 풀에 저장되는 리터럴
- **컴파일 타임 상수식**인 문자열(리터럴 + 상수 결합식)은 String 상수 풀에 저장된다.
- `"hello"` 라는 리터럴이 여러 번 등장해도, 보통 **풀에 있는 하나의 객체를 재사용**한다.
    
예:
```java
String s1 = "hello";
String s2 = "hello";

s1 == s2      // true (같은 풀 객체)
s1.equals(s2) // true
```

### 2-3. `new String("...")` 와 힙 객체
```java
String s1 = "hello";             // 상수 풀의 "hello" 참조
String s2 = new String("hello"); // 힙에 새 String 생성
```
- `"hello"` 리터럴 자체는 여전히 풀에 있다.
- `new String("hello")` 는 **풀에 있는 값을 기반으로 힙에 새 객체를 하나 더 만든다.**
- `s1 == s2` → `false` (서로 다른 객체)
- `s1.equals(s2)` → `true` (내용은 같음)
        

### 2-4. 변수 섞인 문자열과 컴파일 타임 상수식
```java
String a = "he";
String b = "hello";
String c = a + "llo";

b == c      // 보통 false
b.equals(c) // true
```
- `"he" + "llo"` 처럼 **전부 리터럴만** 있으면 컴파일러가 `"hello"`로 합쳐서 상수 풀에 넣는다.
- `a + "llo"` 처럼 **변수가 섞이면** 컴파일 타임에 값이 확정되지 않으므로, 런타임에 새 `String` 객체를 만든다.
- 이 경우 상수 풀 재사용이 **보장되지 않기 때문에**, 같은 내용이라도 `==`는 믿으면 안 되고 **항상 `equals()`로 비교**한다.
    

---

## 3. String이 불변인 이유 – 장단점
### 3-1. 장점
1. **스레드 안전성**
    - 한 번 만들어진 문자열 값이 절대 바뀌지 않으므로,  여러 스레드가 동시에 같은 `String` 인스턴스를 읽어도 상태가 꼬일 일이 없다.
    - 공유 가능한(read-only) 데이터로 쓰기 좋다.
2. **상수 풀 재사용 가능**
    - 불변이기 때문에 상수 풀에서 `"hello"` 객체를 여러 곳에서 공유해도  “누가 중간에 문자열을 바꾸어 다른 코드에 영향을 줌” 같은 문제가 없다.
3. **보안/안정성**
    - 클래스 이름, 파일 경로, 환경 변수, 키 값 같은 중요한 설정/식별자 정보를  `String`으로 들고 있어도 **중간에 값이 바뀌지 않는다.**
    - 예측 가능한 동작과 디버깅에 도움이 된다.
        
### 3-2. 단점 – 성능/GC 관점
```java
String s = "";
for (int i = 0; i < 100000; i++) {
    s = s + i;
}
```
- `s = s + i`는 내부적으로 **새 String 객체를 만들고**, 기존 `s`는 GC 대상이 된다.
- 반복이 많으면 **쓰레기 객체가 많이 생겨 GC 부담 + 성능 저하**가 생긴다.
→ 이 문제를 해결하려고 **가변 버퍼**인 `StringBuilder` / `StringBuffer`가 등장한다.

---

## 4. StringBuilder / StringBuffer (가변 문자열 버퍼)

### 4-1. StringBuilder – 단일 스레드용
OpenJDK 문서:
> “A mutable sequence of characters. This class provides an API compatible with `StringBuffer`, but with no guarantee of synchronization.”

→ **가변(mutable) 문자 시퀀스**이고,  
`StringBuffer`와 API는 비슷하지만 **동기화(synchronization)를 제공하지 않는다**는 뜻.

추가 설명:
> “… designed for use as a drop-in replacement for `StringBuffer` in places where the string buffer was being used by a single thread … it will be faster under most implementations.”

→ **단일 스레드에서 `StringBuffer` 대신 쓰라고 설계**되었고, 대부분의 구현에서 더 빠르다.

**특징 정리**
- 내부에 `char[]` 버퍼를 가지고, 그 위에 문자를 덧붙이거나(insert, append) 수정한다.
- **동기화를 하지 않으므로** 락 비용이 없고, 단일 스레드 환경에서 **가장 빠르다.**
- 문자열 많이 붙일 때: `String` `+` 반복 → ❌, `StringBuilder` → ✅
    
---

### 4-2. StringBuffer – 동기화된(mutable + synchronized) 버퍼
OpenJDK `java.lang` 패키지 설명:
> “StringBuffer. A thread-safe, mutable sequence of characters.”

→ **스레드 안전한, 가변 문자 시퀀스.**
- `append`, `insert` 같은 메서드에 `synchronized` 키워드가 붙어 있다.
- 한 쓰레드가 `append()`를 실행하는 동안, 다른 쓰레드는 같은 인스턴스의 `append()`에 진입할 수 없다.
- 그래서 **여러 스레드가 같은 버퍼를 공유해도 데이터가 꼬이지 않는다.**
- 대신 락/언락 비용 때문에 `StringBuilder`보다 느릴 수 있다.
    

**정리**
- 단일 스레드에서 문자열 조립 → **`StringBuilder` 기본 선택**
- 정말로 여러 스레드가 **같은 버퍼를 공유해야 하는 구조**에서만 → `StringBuffer` 고려 (요즘은 애초에 공유하지 않게 설계하는 편이 더 일반적)
    

---

## 5. String이 구현한 인터페이스들
### 5-1. Serializable
- `java.io.Serializable` 은 **마커 인터페이스** (메서드 없음).
- “이 클래스 인스턴스는 직렬화(serialize)해도 된다”는 표시.
- 객체를 파일에 저장하거나 네트워크로 전송하거나 캐시에 바이트 스트림으로 넣을 때  `String` 을 다른 직렬화 가능한 DTO 필드와 함께 자연스럽게 묶어 쓸 수 있다.
- 주의: `Serializable` 자체는 스레드 안정성과는 무관하고,  **오직 직렬화/역직렬화 가능 여부만 표시**하는 역할이다.
 
---

### 5-2. `Comparable<String>`
- `Comparable<String>` 을 구현해서, `compareTo(String another)` 메서드를 제공한다.
- 역할
    - 두 문자열을 **사전순(lexicographical)** 으로 비교.
    - 반환값
        - 0 → 내용이 같다 (`equals()`와 일치)
        - 음수 → 호출한 쪽이 앞
        - 양수 → 호출한 쪽이 뒤
- 실무
    - `Collections.sort(List<String>)`, `Arrays.sort(String[])` 사용 시 정렬 기준으로 `compareTo`가 사용된다.
    - 이름/아이디 정렬, 이진 탐색 등에서 활용
        
---

### 5-3. CharSequence
OpenJDK 17 문서:
> “A `CharSequence` is a readable sequence of `char` values. This interface provides uniform, read-only access to many different kinds of char sequences.”

**의미**
- “읽기 가능한 문자 시퀀스”를 추상화하는 인터페이스.
- `length()`, `charAt(int index)`, `subSequence(int, int)`, `toString()` 같은 메서드 제공.
- 구현체 예시
    - `String`
    - `StringBuilder`
    - `StringBuffer`
    - `CharBuffer` 등
        

**장점**
- 메서드 파라미터를 `String` 이 아니라 **`CharSequence`로 받으면**,  `String`, `StringBuilder`, `StringBuffer` 를 모두 동일하게 받을 수 있다.
예:
```java
void log(CharSequence message) {
    System.out.println(message);
}
```
- 위 메서드는 `log("text")`, `log(new StringBuilder("text"))` 모두 호출 가능
- 문자열 비슷한 타입들을 **다형성 있게 처리**할 수 있다.
    
---

## 6. 자주 쓰는 String 메서드와 활용

### 6-1. equals() vs compareTo()
- `equals(Object o)`
    - 두 문자열의 **동등성(내용이 완전히 같은지)** 를 `boolean` 으로 리턴.
- `compareTo(String other)`
    - 두 문자열의 **사전순 순서**를 `int` 로 리턴.
    - 정렬/검색에서 사용

→ 동등성 체크 → `equals()`  
→ 정렬/순서 비교 → `compareTo()`

---

### 6-2. length(), isEmpty()
- `length()` : 문자열의 길이(문자 개수) 반환.
- `isEmpty()` : 길이가 0인지 여부 반환 (`length() == 0`).
    
입력 검증, 공백 체크 등에서 기본적으로 사용.

---

### 6-3. indexOf() + substring() – 문자열 파싱 핵심 조합
```java
String log = "user=alice ip=192.168.0.10 result=SUCCESS";

int ipPos = log.indexOf("ip=");
if (ipPos != -1) {
    int start = ipPos + "ip=".length();
    int end = log.indexOf(' ', start); // 다음 공백 인덱스
    String ip = (end == -1)
        ? log.substring(start)
        : log.substring(start, end);
}

```
- `indexOf("ip=")` : 키워드 시작 위치 찾기.
- `indexOf(' ', start)` : 구분자(공백, `&` 등) 위치 찾기.
- `substring(start, end)` : 해당 구간 잘라서 **새 String** 생성.
- 실무 예시
	- 웹 서버 로그에서 IP, 유저명, 에러 코드 등 추출
	- 쿼리스트링 `?user=alice&lang=ko` 파싱
	- 자체 정의 텍스트 프로토콜/헤더 파싱
- **정규식 vs indexOf 조합**
	- `matches()` / `replaceAll()` 처럼 정규 표현식 기반 메서드는  패턴을 해석하고 상태 기계를 돌리는 비용이 있어서 **단순 패턴에 과한 선택**이 될 수 있다.
	- 단순히 `"ip="와 다음 공백 사이 값만 잘라내기` 같은 패턴은  `indexOf` + `substring` 조합이 **더 단순하고 빠르다.**
    

---

## 7. 요약 – 15장 핵심 한 눈에 보기

1. **String은 불변**

    - “Strings are constant; their values cannot be changed after they are created.”
        
    - 불변 덕분에 스레드 안전, 상수 풀 공유, 보안·예측 가능성 측면에서 유리하다.
        
2. **상수 풀과 리터럴**
    
    - 문자열 리터럴/상수식은 상수 풀에 저장되고, 같은 리터럴은 재사용된다.
        
    - `==` 는 **참조(객체)** 비교, 문자열 내용 비교는 항상 `equals()`를 사용한다.
        
3. **문자열 조립은 StringBuilder / StringBuffer**
    
    - `StringBuilder` : “A mutable sequence of characters … with no guarantee of synchronization.”  
        → 단일 스레드에서 빠르게 문자열을 만들 때 사용.
        
    - `StringBuffer` : “A thread-safe, mutable sequence of characters.”  
        → 동기화된 가변 버퍼, 멀티스레드 공유 시 안전하지만 느릴 수 있음.
        
4. **구현 인터페이스 3종**
    
    - `Serializable` : 직렬화/역직렬화 가능
        
    - `Comparable<String>` : `compareTo()`로 정렬/검색 기준 제공
        
    - `CharSequence` : String, StringBuilder, StringBuffer를 공통 타입으로 묶어서 다형적으로 다룸.
        
5. **문자열 메서드 활용**
    
    - `equals()` vs `compareTo()` : 동등성 vs 순서
        
    - `length()`, `isEmpty()` : 기본 검증
        
    - `indexOf()` + `substring()` : 로그/쿼리스트링 파싱, 단순 패턴 추출에 효율적