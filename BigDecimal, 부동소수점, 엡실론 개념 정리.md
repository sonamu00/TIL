---
share: true
---
## 1. float / double vs BigDecimal
### 1-1. float / double (IEEE 754 부동소수점)
- 저장 방식
    - 값 ≈ `sign × (1.fraction) × 2^exponent`
    - **2진수(2의 거듭제곱) 기준**으로 소수를 표현한다.
- 소수 표현 특징
    - 소수점 이하 자리는 `1/2, 1/4, 1/8, 1/16, ... (2⁻¹, 2⁻², 2⁻³ …)` 조합으로만 표현 가능.
    - 0.5, 0.25, 0.75 등은 정확히 표현 가능.
    - **0.1, 0.01, 0.3 등 대부분의 10진 소수는 2진수에서 무한 반복 소수**가 된다.
        - 예: 0.1(10진) → 0.0001100110011…(2진) (반복)
        - 컴퓨터는 유한 비트만 있으므로 중간에서 잘라서 **근사값**으로 저장
        - 10이나 6처럼 2의 거듭제곱이 아니면 2진수 체계에서 값이 확실이 나눠지지 않음
    - `0.1 + 0.2`를 double로 계산하면 내부적으로 `0.30000000000000004` 같은 값이 나올 수 있음    

---

### 1-2. BigDecimal (10진 소수 전용 타입)
> JDK 17 정의 (원문)
> Immutable, arbitrary-precision signed decimal numbers. A BigDecimal consists of an arbitrary precision integer unscaled value and a 32-bit integer scale.
- 해석
    - 변경 불가능한(immutable), 임의 정밀도의 부호 있는(decimal) 10진수 숫자 타입이다.    
    - BigDecimal은 **임의 정밀도의 정수(unscaled value)** 와 **32비트 정수 scale**로 구성된다.
        - `unscaled value`: 소수점 없는 큰 정수 값 (예: 12345)
        - `scale`: 소수점 아래 자릿수(예: scale=2 → 소수 둘째 자리까지)
        - 실제 값은 `unscaledValue × 10^(-scale)` 로 해석된다. (예: `unscaledValue = 12345`, `scale = 2` → `123.45`)
- 특징
    - **10진법 기준**으로 값을 표현 → 0.1, 0.01 같은 값도 정확히 표현 가능
    - 한 번 만든 BigDecimal은 값이 바뀌지 않고, 연산 시마다 새 객체 생성 (immutable)
- 소수점 없는 정수값과 소수점 아래 자릿수 정보를 가지고 소수를 표현한다.
        
---

### 1-3. 왜 금액 계산에 BigDecimal을 쓰는가?
- 우리가 사용하는 돈은 **10진수 체계**라서 0.1원, 0.01원 같은 값이 10진수로 정확하게 표현되어야 한다.
- double은 2진수로 소수를 표현하기 때문에 10진수 값 중에 2의 거듭제곱이 아닌 수가 있으면 소수를 정확히 표현하지 못해 매 거래마다 작은 오차가 생길 수 있다. 
- BigDecimal은 “정수 + 소수 자리수(10진)” 방식이라, 통화 금액을 **정확하게** 표현하고 소수 몇 자리까지, 어떻게 반올림할지(`setScale`, `RoundingMode`)를 명시적으로 제어 가능하다.
- 금융/정산/세금 같은 영역에서는  
    👉 **double + 엡실론 ≠ 해결책**  
    👉 **처음부터 BigDecimal 또는 정수(long, 센트 단위)** 를 써야 한다.
    
---

## 2. IEEE 754 부동소수점 오차 발생 원리
1. **표현 오차**
    - 표현 가능한 실수는 유한 개 → 대부분의 실수(특히 10진 소수)는 **가장 가까운 값으로 반올림**되어 저장된다.
    - 0.1, 0.01, 0.3 등이 대표적인 예.
2. **연산 오차**
    - 덧셈/곱셈/나눗셈을 할 때마다 결과를 다시 “표현 가능한 값”으로 반올림
    - 이 반올림이 누적되면서 실제 값과 차이가 커진다.
    - `(a + b) + c` 와 `a + (b + c)` 가 IEEE 754에서는 결과가 다를 수도 있다 (비결합성).
        
---

## 3. 엡실론(epsilon) 개념
### 3-1. 엡실론이 필요한 이유
- double 값끼리 `a == b`로 비교하면 수학적으로는 같아야 할 값도, 내부적으로는 미세하게 달라서 `false`가 되는 경우가 많다.
- 미세하게 다른 값을 **허용 오차 범위**를 두고, 이 정도 차이면 같은 값이라고 보자라는 기준이 필요하다.
- 그 기준값이 바로 **엡실론(epsilon)**
    
### 3-2. 엡실론 비교 패턴 (절대 오차)
```java
final double EPS = 1e-9; // 허용할 오차 범위

boolean almostEqual(double a, double b) {
    return Math.abs(a - b) < EPS;
}
```
- `Math.abs(a - b)`가 `EPS`보다 작으면 → “같다”고 본다.
- 그래픽, 좌표 계산, 시뮬레이션, 수학적 계산에서 자주 사용하는 패턴.

### 3-3. 실무에서의 사용 구분
- **엡실론 사용에 적합**
    - 게임 물리, 그래픽 좌표, 과학 시뮬레이션, 통계 → 약간의 오차 허용 가능.
- **엡실론 사용하면 안 되는 영역**
    - 금융, 회계, 정산, 세금, 포인트 누적 → 1원 단위 오차도 치명적.
    - 이 영역에서는 **애초에 double을 쓰지 않고** BigDecimal/정수로 설계.
        
---

## 4. BigDecimal 실무 사용 패턴
### 4-1. 생성 시 주의점
- ❌ 피해야 할 것
    ```java
    new BigDecimal(0.1);  // double의 오차를 그대로 안고 들어감
    ```
- ✅ 권장
    ```java
    new BigDecimal("0.1");   // 문자열(10진) 기반
    BigDecimal.valueOf(10L); // 정수 기반
    ```
    
### 4-2. 상수 캐싱
```java
public static final BigDecimal TAX_RATE = new BigDecimal("0.1");

BigDecimal zero = BigDecimal.ZERO;
BigDecimal one  = BigDecimal.ONE;
BigDecimal ten  = BigDecimal.TEN;
```
- 자주 쓰는 값은 `static final`로 캐싱해서 객체 생성 & GC 비용을 줄인다.
    
---

## 5. 소수 자릿수와 반올림: setScale + RoundingMode
### 5-1. setScale 예시
```java
BigDecimal v = new BigDecimal("1.005");

// 소수 둘째 자리까지, HALF_UP 반올림
BigDecimal r1 = v.setScale(2, RoundingMode.HALF_UP); // 1.01

// 소수 둘째 자리까지, DOWN(버림)
BigDecimal r2 = v.setScale(2, RoundingMode.DOWN);    // 1.00
```

- `setScale(2, RoundingMode.HALF_UP)`  → “소수 둘째 자리까지만 남기고, 셋째 자리에서 5 이상이면 올린다.”
- 돈 단위에서 **“표시/저장할 자릿수 + 반올림 규칙”**을 명시할 때 핵심.
    
---

## 6. MathContext로 정밀도(precision) + 반올림 정책 관리
### 6-1. MathContext 정의 (요약)
> JDK 17 정의 (원문)
> “Immutable objects which encapsulate the context settings which describe certain rules for numerical operators, such as those implemented by the BigDecimal class.”
- 해석: BigDecimal 연산에서  **유효 자릿수(precision)** 와 **반올림 모드(rounding mode)**를  한 번에 들고 다니는 설정 객체.

### 6-2. 간단 예시
```java
MathContext mc = new MathContext(10, RoundingMode.HALF_UP); // 유효 숫자 10자리

BigDecimal a = new BigDecimal("1.23456789012345");
BigDecimal b = new BigDecimal("3");

BigDecimal c = a.divide(b, mc); // precision=10, HALF_UP 규칙으로 나눗셈
```
- `precision = 10` → 결과의 유효 숫자를 10자리로 제한, 그 이상은 반올림.
- 내부 계산에서는 `MathContext`로 “전체 정밀도”를 관리하고
- 최종 출력/저장에서는 `setScale`로 “표시할 자리수”를 맞춘다.
        
---

## 7. BigDecimal 성능 이슈 & 완화 전략

### 7-1. 왜 느린가?
- `double`
    - CPU의 FPU(부동소수점 연산 장치)에서 **하드웨어 명령어**로 바로 계산 → 빠름.
- `BigDecimal`
    - 내부에 `BigInteger(큰 정수)`를 들고,  덧셈/곱셈/나눗셈을 **자바 코드로 구현**.
    - immutable이라 연산할 때마다 새 객체 생성.
    - BigDecimal 연산을 많이 하면 **연산 + 객체 생성 비용**이 커진다.  
### 7-2. 완화 방법
1. **상수 BigDecimal 캐싱 (`static final`)**
    - 세율, 고정 금액 등 자주 쓰는 값은 한 번만 생성해 재사용.
2. **핵심 연산은 정수/long으로 하고 경계에서만 BigDecimal**
    ```java
    long totalCents = 0L;
    for (Transaction t : txList) {
        totalCents += t.getAmountCents(); // long 연산
    }
    
    BigDecimal total = BigDecimal.valueOf(totalCents, 2); // scale=2 → 소수 둘째 자리
    ```
3. **`new BigDecimal(double)` 지양, 문자열/정수 기반 사용**
	- 정확한 값 유지 + 예측 가능성 향상.
        