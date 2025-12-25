---
share: "true"
---

## 1. 이 장에서 꼭 기억할 키워드
* 참조 자료형(reference type) vs 기본 자료형(primitive type)
* JVM 메모리 구조: 스택(Stack) / 힙(Heap) / 메서드 영역(Method Area)
* 호출 방식: 자바는 전부 call by value(값에 의한 호출)
* 생성자(constructor), this, this()
* 메서드 오버로딩(overloading)
* static 필드/메서드, 인스턴스 필드/메서드
* 가변 인자(varargs, `int... nums`)

---

## 2. 기본 자료형 vs 참조 자료형
### 2.1 기본 자료형
* 예: `int`, `long`, `double`, `boolean`, `char` 등.
* 변수 자체가 실제 값을 스택에 직접 가진다.
* 대입이나 매개변수 전달 시 값이 그대로 복사된다.
* CPU 관점에서는 레지스터와 스택만으로 바로 연산하기 좋기 때문에 빠르고 단순하다.
- 예시:
```java
int a = 10;
int b = a; // a의 값 10이 그대로 b로 복사됨
b = 20;    // b만 변경되고 a는 그대로 10
```

### 2.2 참조 자료형
* 예: `String`, 배열(`int[]`), 사용자 정의 클래스(`User`, `Account`) 등.
* 변수에는 “객체가 있는 힙 영역의 주소값(참조)”이 저장된다.
* 대입이나 매개변수 전달 시 이 참조값이 복사된다.
* OS 관점에서 보면 힙은 동적 메모리 영역이고, JVM이 여기서 객체를 할당하고 가비지 컬렉션으로 회수한다.
- 예시:
```java
class User {
    String name;
}

User u1 = new User();
u1.name = "Alice";

User u2 = u1;  // u1이 가지고 있는 참조값이 그대로 u2에 복사됨
u2.name = "Bob"; // 같은 객체를 보고 있으므로 u1.name도 "Bob"이 됨
```

---

## 3. 자바의 호출 방식: 항상 call by value
### 3.1 개념
* 자바는 **항상** “값을 복사해서” 전달한다.
* 기본 타입: 실제 값이 복사된다.
* 참조 타입: “참조값(객체의 주소)”이 복사된다.
* C++의 call by reference처럼 “원본 변수 자체를 넘기는 것”은 없다.

### 3.2 참조 타입 매개변수 예시
```java
class User {
    String name;
    User(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        User user = new User("Alice");
        change(user);
        System.out.println(user.name); // ?
    }

    private static void change(User u) {
        u = new User("Bob");
    }
}
```
* `main`에서 `user`는 힙에 있는 `User("Alice")` 객체의 참조값을 가진다.
* `change(user)`를 호출하면 참조값이 `u`로 복사된다.
* `u = new User("Bob");`은 **u가 가리키는 대상만** 새 객체로 바꾸고, `main`의 `user`에는 영향을 주지 않는다.
* 결과적으로 콘솔에는 `"Alice"`가 출력된다.
- 필드 변경 예시:
```java
private static void change(User u) {
    u.name = "Bob";
}
```
* 이 경우 `u`와 `user`가 같은 객체를 바라보므로 힙에 있는 동일한 인스턴스의 필드가 변경된다.
* 결과는 `"Bob"`이 출력된다.
* “참조값을 복사해서 넘긴다”는 말은 “같은 객체를 여러 이름이 공유할 수 있다”는 뜻이고, 이 때문에 멀티스레드 환경에서 상태 공유로 인한 동시성 버그(concurrent modification, race condition)가 생기기도 한다.

---

## 4. JVM 메모리와 메서드 호출 흐름
### 4.1 메서드 호출 시 스택/힙
* 메서드가 호출될 때마다 스택에 “스택 프레임(Stack Frame)”이 하나씩 쌓인다.
* 스택 프레임에는 지역 변수, 매개변수, 리턴 주소 등이 저장된다.
* `new`로 생성한 객체는 힙에 저장되고, 스택에는 이 객체를 가리키는 참조값만 저장된다.

흐름 예시:
1. `main`이 호출되어 `main`의 스택 프레임이 올라감.
2. `User user = new User("Alice");` 실행 시 힙에 `User` 객체가 생성되고, 그 주소값이 `user` 지역 변수에 저장됨.
3. `change(user);` 호출 시 `main`의 스택 프레임 위에 `change`의 스택 프레임이 생성되고, `user`의 참조값이 `u`에 복사됨.
4. `change`가 끝나면 `change` 스택 프레임은 사라지고, 다시 `main`으로 돌아감.
5. 어느 시점에 더 이상 참조되지 않는 힙 객체는 GC가 회수한다.

---

## 5. 생성자와 this, this()
### 5.1 생성자 기본 개념
* 생성자 이름은 클래스 이름과 같고 리턴 타입이 없다.
* 객체가 생성될 때 초기 상태를 설정하는 역할을 한다.
* 아무 생성자도 만들지 않으면 컴파일러가 파라미터 없는 기본 생성자(디폴트 생성자)를 자동으로 만들어 준다.
* 생성자를 하나라도 정의하면 더 이상 기본 생성자를 자동 생성해 주지 않는다.

```java
class Account {
    String owner;
    int balance;

    // 생성자
    Account(String owner, int balance) {
        this.owner = owner;
        this.balance = balance;
    }
}
```

### 5.2 this 키워드
* `this`는 “현재 인스턴스 자신”을 가리키는 참조다.
* 인스턴스 메서드와 생성자 안에서만 사용할 수 있다.
* `this.field` 형태로 쓰면 현재 객체의 필드를 명시적으로 가리킨다.
* 매개변수 이름과 필드 이름이 같을 때 주로 사용한다.

```java
class User {
    String name;
    User(String name) {
        this.name = name; // this.name은 필드, name은 매개변수
    }
}
```

### 5.3 this() – 다른 생성자 호출
* 같은 클래스 안의 다른 생성자를 호출할 때 사용한다.
* 생성자 안에서만 사용 가능하며, 반드시 첫 줄에 와야 한다.
* 자바는 기본 인자(default parameter)가 없기 때문에 this(...)를 이용해 “기본값이 있는 생성자 패턴”을 만든다.

```java
class Account {
    String owner;
    int balance;

    Account(String owner) {
        this(owner, 0); // 다른 생성자 호출, 기본 잔액 0으로 초기화
    }

    Account(String owner, int balance) {
        this.owner = owner;
        this.balance = balance;
    }
}
```

* `new Account("Alice")` → `Account(String)` 생성자 호출 → 내부에서 `this("Alice", 0)` 실행 → 최종적으로 `owner="Alice", balance=0`.
* `new Account("Bob", 1000)` → 바로 두 번째 생성자 실행.
- 실무에서의 활용 예:
	* DB 연결 설정 객체에서 URL만 받으면 나머지는 기본값으로 채우고 싶을 때.
	* HTTP 요청 옵션 객체에서 타임아웃, 재시도 횟수의 디폴트를 한 곳에서 관리할 때.
	* 게임 캐릭터, 설정 값 같은 곳에서 공통 초기화 로직을 한 생성자에 몰고, 나머지는 this(...)로 재사용할 때.

---

## 6. static 필드와 static 메서드
### 6.1 static의 의미
* JVM이 클래스를 로드할 때 메서드 영역에 만들고 클래스의 인스턴스가 여러 개 생성되어도 하나만 존재한다.
* 인스턴스를 만들지 않아도 클래스 이름으로 바로 접근할 수 있다.
* JVM에서 클래스가 로딩될 때 메서드 영역(Method Area)에 static 필드와 static 메서드 정보가 올라간다.
* 인스턴스 필드는 각 객체마다 힙에 따로 존재하지만 static 필드는 클래스당 하나만 존재한다.

```java
class Counter {
    static int totalCount = 0; // 모든 인스턴스가 공유
    int id;                    // 각 인스턴스마다 개별 값

    Counter() {
        totalCount++;
        id = totalCount;
    }
}
```

### 6.2 static과 인스턴스의 차이
```java
Counter c1 = new Counter(); // totalCount = 1, c1.id = 1
Counter c2 = new Counter(); // totalCount = 2, c2.id = 2
Counter c3 = new Counter(); // totalCount = 3, c3.id = 3

System.out.println(Counter.totalCount); // 3
```
* `totalCount`는 세 인스턴스가 모두 공유하는 값이다.
* `id`는 각 인스턴스마다 다른 값을 갖는다.
* OS / 시스템 관점에서 static은 “프로세스 내 클래스 로더 단위로 한 번만 로드되는 전역 데이터”처럼 동작한다.

### 6.3 static 메서드의 제약
```java
class Util {
    int value = 10;

    static void printValue() {
        // System.out.println(value);      // 컴파일 에러: non-static variable
        // System.out.println(this.value); // 컴파일 에러: this 사용 불가
    }

    static void printValue(Util u) {
        System.out.println(u.value); // 이런 식으로는 가능
    }
}
```
* static 메서드는 인스턴스 없이도 호출할 수 있기 때문에 `this`가 없다.
* 인스턴스 필드는 사실상 `this.value`로 접근하는 것인데, static 컨텍스트에는 `this`가 없으므로 직접 접근할 수 없다.
* 인스턴스 필드에 접근하려면 반드시 명시적으로 인스턴스를 매개변수로 받아서 `u.value`처럼 사용해야 한다.
* 반대로 static 필드/메서드는 인스턴스와 상관없이 공통으로 쓰는 값, 유틸리티 메서드, 상수 정의 등에 사용된다.

---

## 7. 메서드 오버로딩 (Overloading)
* 같은 이름의 메서드를 여러 개 정의하되, 매개변수 타입 또는 개수로 구분하는 것.
* 리턴 타입만 다르게 해서 오버로딩할 수는 없다.
* API 사용성을 높이고, “비슷한 행동을 하는 메서드 묶음”을 한 이름 아래 모을 수 있다.
```java
void print(String s) {}
void print(int n) {}
void print(String s, int n) {}
```
* 운영체제나 라이브러리 수준에서 보면, 결국 컴파일된 바이트코드는 시그니처(메서드 이름 + 파라미터 타입 조합)별로 구분되기 때문에 메서드 테이블에서 서로 다른 엔트리로 관리된다.

---

## 8. 가변 인자(Varargs)
### 8.1 기본 개념
```java
static int sum(int... nums) {
    int result = 0;
    for (int n : nums) {
        result += n;
    }
    return result;
}
```
* `int... nums`는 내부적으로 `int[]` 배열로 취급된다.
* 호출 시 `sum(1, 2)`, `sum(1, 2, 3, 4)`처럼 여러 개를 자유롭게 넘길 수 있고, 이미 만들어진 배열(`int[]`)도 그대로 넘길 수 있다.
* 아무 것도 안 넘기고 `sum()`을 호출하면 길이 0인 배열이 넘어온다.

```java
sum();            // nums.length == 0
sum(1);           // nums = new int[]{1}
sum(1, 2, 3, 4);  // nums = new int[]{1,2,3,4}
```
* 자바 컴파일러가 호출 시점에 자동으로 배열을 생성해 주기 때문에, 호출하는 쪽 코드가 간결해지고 API 설계가 유연해진다.
* 실무에서 로그 출력, 통계 집계, 권한 목록 처리 등 “0개 이상”의 인자를 받을 때 자주 사용된다.

### 8.2 제약
* 가변 인자는 메서드 정의에서 **항상 마지막 파라미터**여야 한다.
* 하나의 메서드에 가변 인자를 두 개 이상 둘 수 없다.
* 오버로딩과 섞어서 사용할 때는 호출 모호성(ambiguity)이 생기지 않도록 신중하게 설계해야 한다.

---

## 9. 정리 – 8장 핵심 한 줄 요약
* 자바는 기본 타입이든 참조 타입이든 **항상 값(참조값 포함)을 복사해서** 전달한다.
* 참조 타입은 “객체의 주소값”을 공유하기 때문에, 필드를 변경하면 같은 객체를 보는 모든 코드에 영향이 간다.
* 생성자와 this, this()는 객체의 초기화를 깔끔하게 정리하고, 기본값 패턴을 설계하는 핵심 도구다.
* static은 “클래스 단위” 개념으로, 인스턴스 없이 동작하는 공통 상태와 기능을 표현할 때 쓰이고, 인스턴스와 생명주기와 메모리 위치가 다르다.
* 가변 인자는 결국 배열이지만, 호출 시 문법을 단순하게 만들어 주는 문법적 설탕(syntactic sugar)이다.

---

## 10. 자기 점검용 질문
정답을 바로 보지 말고, 스스로 60초 안에 설명해 볼 수 있는지 체크해 보기.

1. 자바가 “항상 call by value”라는 말의 의미를, 참조 타입 예시를 들어 설명해 보세요.
2. 아래 코드에서 콘솔에 어떤 값이 찍히는지, JVM 메모리 구조를 떠올리며 이유까지 말해 보세요.
   ```java
   class User {
       String name;
       User(String name) { this.name = name; }
   }

   void change(User u) {
       u.name = "Bob";
   }
   ```
3. 생성자에서 `this()`를 사용하는 이유를, 실무에서 자주 있는 “기본값을 한 곳에서 관리하는 패턴” 관점에서 설명해 보세요.
4. static 메서드 안에서 인스턴스 필드를 바로 사용할 수 없는 이유를 `this`의 존재 여부와 인스턴스 생성 시점 기준으로 설명해 보세요.
5. `int sum(int... nums)`를 배열 관점에서 풀어 설명하고, `sum()`, `sum(1)`, `sum(1,2,3,4)` 호출 시 내부에서 어떤 일이 일어나는지 말해 보세요.

