---
share: "true"
---
## 1. main() 메서드와 프로그램 진입점
- `public static void main(String[] args)` 정의  
	- 자바 애플리케이션에서 **JVM이 가장 처음 호출하는 진입점 메서드**
	- 객체를 만들지 않고도 호출할 수 있고, 반환값 없이 프로그램 실행을 시작한다.

---

## 2. main 시그니처 각 키워드의 의미

### 2.1 public
- `public`
	- 어디에서나 호출 가능한 **접근 제어자**
	- JVM(정확히는 `java` 런처)이 **클래스 바깥에서 이 메서드를 호출**해야 하므로 `public`이어야 한다.
	- `public`이 아니면 JVM이 진입점으로 인식하지 못하고 실행 오류가 발생할 수 있다.

### 2.2 static
- `static`
	- **클래스에 속한 메서드**라는 의미 (인스턴스가 아닌 클래스 소속)
	- **객체 생성 없이** `클래스이름.main(args)` 형태로 호출할 수 있다.
	- JVM은 프로그램 시작 시 아직 객체가 없는 상태에서
		- `클래스이름.main(String[] args)` 를 바로 호출해야 하므로 `static`이어야 한다.
	- static 메서드/필드는 메서드 영역에 하나만 존재하고, 모든 스레드가 이를 공유한다.

### 2.3 void
- `void`
	- **아무 것도 반환하지 않는 반환 타입**을 의미한다.
	- `main` 메서드의 반환 타입이 `void` 인 이유 
		- JVM이 `main`을 호출하고나서 `main`의 **반환값을 받아서 쓰도록 설계되어 있지 않다.**
		- 프로세스 종료 코드는 `System.exit(0)` 같은 방식으로 전달한다.

### 2.4 String[] args
- `String[] args`
	- 프로그램 실행 시 **명령줄에서 전달된 문자열 인자들을 담는 배열**
	- 흐름
		- 사용자가 셸/터미널 또는 IDE Run 설정에서 인자를 지정한다.
		- OS → `java` 런처(JVM) → `main(String[] args)` 로 배열이 전달된다.
	- 예시
		- `java MyMainClass A B C`
			- `args[0] = "A"`
			- `args[1] = "B"`
			- `args[2] = "C"`
	- 스프링 부트 등 프레임워크에서는
		- `SpringApplication.run(MyApp.class, args);` 와 같이 `args`를 넘겨
		- `--spring.profiles.active=dev`, `--server.port=9000` 같은 옵션을 환경 설정으로 사용한다.

---

## 3. JVM과 main 실행 흐름

- `java MyMainClass` 실행 시 (클래스 하나만 실행)
	- OS가 `java` 실행 파일(JVM 런처)을 실행한다.
	- JVM은 `MyMainClass` 라는 **클래스 이름**을 기준으로 클래스를 로드한다.
	- 그 클래스 안에서 `main` 메서드를 찾는다
	- 찾으면 **main 스레드(맨 처음 만들어지는 스레드)에서 main 메서드를 호출**하며 프로그램이 시작된다.
- `java -jar app.jar` 실행 시
	- JAR 내부 `MANIFEST.MF` 의 `Main-Class: com.example.MyMainClass` 설정을 보고
		- 해당 클래스를 엔트리(진입) 클래스로 사용한다.
	- 이후 흐름은 동일하게 `public static void main(String[] args)` 를 호출한다.
- main 실행 시 메모리 관점
	1. `main` 메서드의 **바이트코드(명령어)** 는 메서드 영역(Method Area)에 올라간다.
	2. JVM이 main 스레드를 만든다.
	3. main 스레드의 스택(Stack) 위에 **main 스택 프레임**을 생성한다.
	4. 이 프레임에 `String[] args` 매개변수와 main 내부 지역 변수들이 저장된다.

---

## 4. 패키지와 디렉터리 구조

### 4.1 package
- 클래스를 **논리 단위로 묶는 이름공간(namespace)** 을 정의한다.
- 역할
	- 클래스를 기능/도메인별로 논리적으로 그룹화
	- 서로 다른 패키지에서 같은 이름의 클래스를 구분 (`com.a.User`, `com.b.User`)
	- 기본 접근 제어 범위(package-private)의 기준이 된다.
- 일반적으로 패키지 이름과 디렉터리 구조는 1:1로 매핑된다.
	- `package com.example.demo;`
	- → 클래스패스 기준 `com/example/demo` 디렉터리 안에 소스/클래스 파일이 위치한다.
	- 예: `com/example/demo/Sample.java`

---

### 4.2 소스 파일과 public 클래스 규칙
- 하나의 `.java` 파일에서의 규칙
	- **top-level `public` 클래스는 최대 1개만 존재할 수 있다.**
	- 그 `public` 클래스의 이름과 **파일명은 반드시 같아야 한다.**
		- `Sample.java` → `public class Sample { ... }`
- `public` 이 아닌 top-level 클래스(package-private)는
	- 같은 `.java` 파일 안에 추가로 둘 수 있다.
	- 예:
		```java
		// Sample.java
		public class Sample { }

		class Helper { }
		```
	- 이 경우
		- `Sample` 은 `public` 이고, 파일명과 일치해야 한다.
		- `Helper` 는 package-private 이며, 별도의 `Helper.java` 파일이 없어도 된다.

---

## 5. 접근 제어와 package-private
- top-level 클래스에서 가능한 접근 제어
	- `public class A`
		- 어디에서나 접근 가능
	- `class A`
		- 접근 제어자를 생략하면 **package-private**
- **package-private** 의 의미
	- 같은 패키지 안에서는 접근 가능
	- 다른 패키지에서는 **아예 보이지 않는 클래스/멤버** 취급
	- 예:
		```java
		// com/example/a/SampleA.java
		package com.example.a;

		class SampleA { }  // package-private
		```
		```java
		// com/example/b/SampleB.java
		package com.example.b;

		class SampleB {
			SampleA a;  // ❌ 다른 패키지라서 접근 불가
		}
		```
	- `import`를 사용해도
		- 접근 제어자가 package-private 이면 **여전히 접근할 수 없다.**

---

## 6. import 의 역할
- `import com.example.a.SampleA;`
	- 다른 패키지에 있는 클래스를 사용할 때
		- 매번 `com.example.a.SampleA` 라고 쓰지 않고
		- **패키지 이름 없이 `SampleA`만으로 사용할 수 있게 해주는 문법**
- 중요한 점
	- `import`는 **이름을 짧게 쓰게 해주는 것일 뿐, 접근 권한을 열어주지 않는다.**
	- 실제로 접근 가능/불가능은 `public`, `protected`, `private` 같은 **접근 제어자**로 결정된다.
	- 완전한 이름을 쓰면 import 없이도 사용 가능하다.
		```java
		com.example.a.SampleA a = new com.example.a.SampleA();
		```

---

## 7. System.out.println 과 스트림

### 7.1 System.out.println 의 구조
- `System.out.println("Hello");` 한 줄의 의미
	- `System` 클래스에 있는
	- `public static final PrintStream out` 필드를 통해
	- `PrintStream` 객체의 `println("Hello")` 메서드를 호출하여
	- **표준 출력(콘솔/IDE Run 콘솔)에 "Hello"와 줄바꿈을 출력하는 코드**
- 구성 요소
	- `System`
		- `java.lang.System` 클래스
	- `out`
		- `System` 클래스의 `public static final PrintStream` 필드
		- **표준 출력(standard output)에 연결된 스트림**
	- `println(...)`
		- `PrintStream` 클래스의 메서드
		- 문자열을 출력하고, 줄바꿈 문자를 추가한다.

### 7.2 스트림(Stream)의 개념
- 스트림(stream)
	- 자바에서 스트림은 **데이터를 한 방향으로 흘려보내거나 읽어들이는 통로 역할을 하는 객체**
	- 주로 I/O(입출력)에서 사용된다.
	- 종류 예시
		- `InputStream` : 외부 → 프로그램 방향으로 데이터가 들어오는 통로
		- `OutputStream` : 프로그램 → 외부 방향으로 데이터가 나가는 통로
		- `PrintStream`  : 사람이 읽기 쉬운 형태로 출력하는 `OutputStream`의 래퍼
- `System.out`
	- 표준 출력에 연결된 `PrintStream` 객체
	- `println(...)` 호출 시
		- 문자열을 바이트로 변환해 내부 `OutputStream`에 쓰고
		- 줄바꿈 문자를 추가하여 **콘솔에 한 줄 출력**한다.
