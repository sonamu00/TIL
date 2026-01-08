## class file constant pool
- 위치: `.class` 파일 내부(하드디스크에 바이너리로 저장)
- 역할: 클래스가 쓰는 각종 정보를 모아서 클래스 파일 내부에 테이블로 저장함
    - 문자열 텍스트, 클래스명, 메서드명, 타입 서명, 숫자 리터럴 등
- 특징:
    - 아직 메모리에 안 올라가서 실제 객체 주소가 아니라 심볼릭 정보를 사용
	    - 심볼릭 정보: 특정 대상을 지칭하기 위해 정해진 **문자열 기반의 식별 정보**(ex: 택배 송장번호)
    - 다른 곳(바이트코드, 필드/메서드 정보)에서 인덱스로 참조
- 예: 
	- CONSTANT_Utf8: 텍스트 조각을 저장하는 엔트리, 클래스 이름, 메서드 이름, 디스크립터, 문자열 리터럴 텍스트가 여기 들어감
	- CONSTANT_Methodref: 어떤 클래스의 어떤 메서드를 부를지 가리키는 엔트리, 내부적으로 Class 엔트리 + NameAndType 엔트리를 조합해서 표현
- 한 줄 요약: JVM 실행 전에 파일 안에 있는 설계도 테이블

## runtime constant pool
- 위치: JVM 메모리 내 메서드 영역(클래스가 로딩될 때 만들어짐), HotSpot 기준 Metaspace
- 역할: class file constant pool을 바탕으로 JVM이 실행 중 쓰기 좋게 만든 테이블
- 특징:
    - 클래스가 로딩될 때, `.class` 파일 내부에 class file constant pool 정보를 읽어서 실행 중 빠르게 참조할 수 있는 runtime constant pool 구조로 구성함
    - **심볼릭 참조**를 **실제 런타임 엔티티를 가리키는 직접 참조나 캐시 형태**로 바꿔, 이후 호출을 빠르게 만듬(Resolution)
- 예:
    - `ldc`(상수 로드 바이트코드)가 여기서 항목을 찾아 스택에 올림
한 줄 요약: JVM 실행 중에 쓰는 상수/심볼 테이블, 필요하면 해석 결과까지 붙음
---
## string pool
- 위치: 실제 String 객체는 JVM 메모리 내 힙 메모리 영역, HotSpot JVM이 문자열 상수 객체들을 찾아주고 관리하는 테이블은 네이티브 메모리 영역
- 역할: 같은 내용의 문자열을 **대표 하나**로 공유해서 메모리와 문자열 비교 비용을 줄이기 위해 나온 저장소
	- 같은 내용 문자열이 **같은 참조** 로 공유될 수 있어서 `==`(참조 비교: 같은 객체를 가리키는지 비교)로 **참조 비교 1번**으로 끝남 → 아주 빠름
- 특징:
    - 문자열 리터럴은 보통 풀을 통해 대표 객체를 씀
    - `intern()`(인터닝: 어떤 문자열을 풀의 대표로 등록하거나 대표를 가져오는 동작)으로 강제로 풀을 이용할 수 있음
	    - string pool에 강제로 문자열을 넣기 때문에 string pool 영역↑ 메모리도 ↑ => 사용ㄴㄴ
    - HotSpot 기준 실제 String 객체는 힙에 있고, string pool은 JVM 내부의 StringTable 같은 구조가 힙의 String들을 참조하며 관리
	    - StringTable은 네이티브 메모리에 있는 테이블이고, 그 테이블 인덱스가 가리키는 실제 String 객체들은 힙에 있다.
한 줄 요약: 문자열을 공유하기 위한 대표 객체 저장소

### 코드로 각 pool의 흐름 보기

``` java
String a = "hi";
String b = new String("hi");
String c = b.intern();
```

1. 컴파일 결과
	- class file constant pool에 hi 텍스트가 기록됨
2. 클래스 로딩
	- runtime constant pool이 준비되고, hi 항목을 인덱스로 참조할 준비를 함
3. 실행 중 변수 `a` 에 값 대입 시점
	- `ldc`(엘디씨: 런타임 상수 풀의 항목을 스택에 올리는 바이트코드)가 hi를 처음 꺼냄
	- JVM은 string pool에서 hi 대표 String을 찾거나 만들고, 그 참조를 `a`에 넣음
	- 결과: `a`는 풀의 대표 객체를 가리키는 경우가 일반적
4. 실행 중 변수 `b` 에 값 대입 시점
	- `new String`은 힙에 새 String 객체를 하나 더 만듦
	- 결과: `b`는 `a`와 다른 객체일 가능성이 매우 높음(왜 매우 높음으로 썼냐면 컴파일러가 개선할 수도 있음)
5. 실행 중 변수 `c` 에 값 대입 시점
	- intern은 `b`의 내용과 같은 대표 String을 string pool에서 가져옴
	- 결과: `c`는 보통 `a`와 같은 객체를 가리킴
    
	
---
## 문맥에서 헷갈리지 않는 판별법
- `.class` 파일 포맷/구조 얘기(tag, cp_info, CONSTANT_XXX 등)면 → **class file constant pool** (파일 안 테이블)
- 실행 중 바이트코드가 인덱스로 찾고 해석한다(`ldc`, resolve, linking 등)면 → **runtime constant pool** (실행용 테이블)
- 문자열을 대표 1개로 공유한다(intern, 리터럴 재사용, StringTable 등)면 → **string pool** (문자열 공유 저장소)
