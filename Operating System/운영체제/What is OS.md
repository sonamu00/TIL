# Computer System Overview
## 운영체제(OS)란?
컴퓨터 시스템자원(HW)를 효율적으로 관리해서 사용자, 응용 프로그램에 서비스를 제공하는 소프트웨어이다.

## 컴퓨터 하드웨어
#### 프로세서(Processor)
- 연산을 위해서 사용된다.
- CPU
- 그래픽카드(GPU)
- 응용전용처리장치등

#### 메모리(Memory)
- 저장을 위해서 사용한다.
- 주기억장치
- 보조기억장치등

#### 주변장치
- 키보드/마우스
- 모니터, 프린터
- 네트워크모뎀 등

## 프로세서(Processor)
![1](https://github.com/ChaewonHan/TIL/blob/81250c1e52bb89184e5a1a6794d9cb8eeb6f6a2f/Operating%20System/img/1.PNG)
연산을 수행하고 컴퓨터의 모든 장치의 동작을 제어해서 중앙처리장치(CPU)로 불린다.

## 레지스터(Register)
프로세서 내부에 있는 메모리이다. 컴퓨터에서 가장 빠른 메모리이며 프로세서가 사용할 데이터를 저장한다. 

### 레지스터의 종류
- 용도에 따른 분류
  - 전용 레지스터,범용 레지스터
- 사용자가 정보 변경 가능 여부에 따른 분류
  - 사용자 가시 레지스터 사용자 불가시 레지스터
- 저장하는 정보의 종류에 따른분류
  - 데이터 레지스터, 주소 레지스터, 상태 레지스터

![2](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/2.PNG)
![3](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/3.PNG)
  
  
### 프로세서의 동작
![4](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/4.PNG)
계산하는 연산 장치(ALU) 안에 다양한 레지스터를 통해서 연산이 이루어진다.

## 메모리(Memory)
프로그램(os, 사용자sw 등), 사용자 데이터를 저장하는 장치(기억장치)
#### 메모리의 종류
![5](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/5.PNG)
위로 갈수록 용량이 크고, 속도가 빠르고 가격이 비싸다. 최소한의 비용으로 성능을 내기 위해서 위 같은 계층이 탄생했다.
### 주 기억장치(Main memory)
- 프로세서가 수행할 프로그램과 데이터를 저장하는 역할을 한다. 
- DRAM과 DDR4를 주로 사용한다.
- 디스크 입출력 병목현상(I/O bottleneck)해소 
![6](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/6.PNG)
프로그램과 데이터는 반드시 메인메모리 안에 들어있어야 하는데 이유는 cpu의 속도를 disk가 따라 잡지 못한다. 이를 디스크 입출력 병목현상이라고 한다. 이를 해결하기 위해 메모리를 중간에 두어 cpu가 메모리에 있는 데이터를 사용하게 한다.

## 캐시
프로세서 내부에 있는 메모리이다. 속도가 빠르고 가격이 비싸다는 특징이 있지만 메인 메모리의 입출력 병목 현상을 해결할 수 있다.

### 캐시의 동작
![7](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/7.PNG)
- 일반적으론 HW(하드웨어)적으로 관리된다.
- 캐시 히트 : 필요한 데이터 블록에 캐시 존재하면 프로세서에서 캐시에 있는 데이터 블록을 가져오게 된다. 이럴 경우 메인 메모리에 가는 시간과 가져오는 시간을 단축할 수 있게 되는 것을 의미한다.
- 캐시 미스 : 필요한 데이터 블록이 캐시에 없으면 캐시에 들린 후 메인 메모리에 가야 되기 때문에 시간이 늘어나는 것을 의미한다.

### 지역성
캐시의 크기는 128byte~ 정도 된다. 메모리 입출력 병목 현상을 해결하기엔 크기가 작은데 어떻게 해결할 수 있을까? 지역성을 살펴보도록 하자.

#### 공간적 지역성(Spatial locality)
참조한 주소와 인접한 주소를 참조하는 특성이다. 데이터 블록을 인접한 데이터들과 함께 가져온다.
- ex) 순차적 프로그램

#### 시간적 지역성(Temporal locality)
한 번 참조한 주소를 다시 참조하는 특성이다.
- ex) while, for문 등의 순한문

지역성은 캐시 적중률에 많은 도움을 주어서 캐시 히트가 발생할 가능성을 높혀준다. 다음 예를 살펴보자.

#### 지역성의 동작 방법
![8](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/8.PNG)

위 for문에서 캐시 히트가 발생할 가능성이 높은 것은 A일까? B일까?
답은 A이다.
- a[0][n]으로 반복하면 a[0][15]까지 이니까 총 15번의 캐시 히트가 발생한다.
- a[n][0]으로 반복하면 a[1][0]부터 미스가 난다.

이 처럼 캐시의 지역성의 특징을 이해하면 효율적인 프로그램을 작성할 수 있다.

## 메모리의 종류
### 보조기억 장치
- Auxiliary memory / secondary memory / storage
- 프로그램과 데이터를 저장한다.
- 프로세서가 직접 접근 할 수 없다(주변장치)
  - 주기억장치를 거쳐서 접근한다.
  - 프로그램,데이터가 주기억장치보다 데이터 큰 경우는 가상 메모리를 사용한다.
- 용량이 크고 가격이 저렴하다.  

## 시스템 버스(System Bus) 
하드웨어들이 데이터 및 신호를 주고 받는 물리적인 통로이다.
![9](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/9.PNG)
![10](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/10.PNG)

### 시스템 버스 동작 방법
![11](https://github.com/ChaewonHan/TIL/blob/77a0a11271285bd62fc3e370154cac189823f9be/Operating%20System/img/11.PNG)

## 주변장치
프로세서와 메모리를 제외한 하드웨어을 의미한다.
- 입력 장치
- 출력 장치
- 저장 장치

### 주변장치와 운영체제
- 장치 드라이버 관리
  - 주변 장치 사용을 위한 인터페이스를 제공한다.
- 인터럽트 처리
  - 주변 장치의 요청을 처리한다.
- 파일 및 디스크 관리
  - 파일 생성 및 삭제
  - 디스크 공간 관리 등
## 운영체제의 구조
![18](https://github.com/ChaewonHan/TIL/blob/ff5e937fb7bf806157326b3674b13006fead7302/Operating%20System/img/18.PNG)
### 커널(kernel)
- OS의 핵심 부분이여서 메모리에 상주해 있다.
- 가장 빈번하게 사용되는 기능을 담당한다.
  - 시스템 관리(프로세서, 메모리, etc 등)
- 동의어
  - 핵(neucleus), 관리자(supervisor) 프로그램, 상주프로그램(resident program), 제어프로그램(control program) 등
### 유틸리티(Utility)
- 비상주 프로그램
- UI 등 서비스 프로그램이 해당된다.
### 단일 구조
![19](https://github.com/ChaewonHan/TIL/blob/ff5e937fb7bf806157326b3674b13006fead7302/Operating%20System/img/19.PNG)
#### 장점
- 커널 내 모듈 사이에서 직접 통신을 해서 효율적으로 자원을 관리하고 사용할 수 있다.
#### 단점
- 커널이 거대해져서 오류를 고치거나 추가 기능을 구현하는 등, 유지보수가 어렵다.
- 동일 메모리에 모든 기능이 있어서 한 모듈의 문제가 전체 시스템에 영향이 간다.
### 계층 구조
![20](https://github.com/ChaewonHan/TIL/blob/ff5e937fb7bf806157326b3674b13006fead7302/Operating%20System/img/20.PNG)
#### 장점
- 모듈화가 되어 있어서 계층간 검증과 수정이 용이하다.
- 설계와 구현이 단순하다.
#### 단점
- 단일 구조라서 성능이 저하된다.
- 원하는 기능 수행을 위해 여러 계층을 거쳐야한다.
### 마이크로 커널 구조
![21](https://github.com/ChaewonHan/TIL/blob/ff5e937fb7bf806157326b3674b13006fead7302/Operating%20System/img/21.PNG)
- 커널의 크기를 최소화 한다.
- 필수 기능만 포함되어 있다.
- 기타 기능은 사용자 영역에서 수행할 수 있도록 설계되었다.
