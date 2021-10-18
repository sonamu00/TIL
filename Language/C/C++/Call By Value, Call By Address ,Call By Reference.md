## Call By Value
### 특징
- 값에 의한 전달이다.
- 함수로 값을 전달하면 값이 함수의 매개변수로 복사가 된다.
- 복사한 값이기 때문에 함수 내에서 원본의 값을 변경할 수 없다.
### 예제
```c++
void functionA(int value) {
    value++;
}

int main(void) {
    int a = 10;
    functionA(a);
    printf("%d\n", a);
    return 0;
}
// 결과 : 10
```
- 인자로 전달 받은 수에 1을 더하는 functionA를 실행해도 n의 값은 바뀌지 않는다.
- 함수를 호출할 때 값(a)을 복사해서 매개변수(value)로 피호출 함수(justFunc)에 넘겨준다.
- 주소의 값에 직접 접근해서 변경하는 것이 아니기 때문에 값을 변경할 수 없다.

## Call By Reference
### 특징
- 함수에서 함수 외부 메모리 공간을 참조할 때 사용한다.
- 함수 선언시 매개변수에 &를 사용해 변수의 주소를 받도록 하고 함수 내부에서는 주소를 준 변수를 일반 변수처럼 사용한다.
- call by value처럼 메모리 공간을 할당 받는것이 아닌 원본 변수의 별명으로 존재하는 것이다.
- 즉 변수 자체를 참조하여 값을 변경이 가능 하기 때문에 함수 내에서 값을 변경을 해도 원본 변수의 값 자체가 바뀐다.

 
```c++
void functionB(int& b) {
    b++;
}

int main(void) {
    int a = 10;
    functionB(a);
    printf("%d\n", a);
    return 0;
}
// 결과 : 11
```
-  functionB는 main 함수가 메모리를 어떻게 사용하고 있는지 전혀 모른다.
-  main 함수가 사용하고 있는 메모리 공간에 직접 좌표를 찍어서 강제로 값을 변경하는 것이다.

```c++
void functionC(int& c) {
    (*(&c+1))++;
}

int main(void) {
    int a = 10;
    int b = 1;

    functionC(a);

    printf("%d, %d\n", a, b);
    return 0;
}
// 결과 : 10, 2
```
- main 함수의 a가 가진 주소 값에 1을 더한 위치에 b가 있었고, b의 위치값에 증가 연산(++)을 하니 결국 b의 값이 늘어났다.
- 참조에 의한 호출은 좌표를 찍는 방식이기 때문에 연산을 하면 주소 값 위치 연산이 되게 된다.



## Call By Address
### 특징
- 매개변수에 포인터형 변수를 선언한다.
- 포인터 변수의 공간에 원본의 주소값을 복사하고 변수의 주소를 참조하여 가르키는 곳의 값을 변경하면 원본의 데이터가 바뀐다.
- 함수 인자에 원본의 주소 값을 복사해서 전달하기 때문에 call by value 형태이다.
- 주소 값은 별도의 메모리 공간에 복사된다.



### 예제
```c++
void functionD(int* v) {
    (*v)++;
}

int main(void) {
    int a = 10;
    functionD(&a);
    printf("%d\n", a);
    return 0;
}
// 결과 : 1
```
- functionD 실행하고 main함수의 변수 a의 값이 1로 바뀐다.
- a의 주소를 복사해서 매개변수(v)로 넘겨주고 v(a의 주소)가 가르키는 주소에 1을 더해서 변수의 값을 변경했다.
- functionD의 int* v는 functionD 스택에서 정의된 새로운 포인터 변수이며, int* v의 주소값은 원본 주소값(ex:0x1000)이 아닌, functionD에서 생성된 새로운 주소값(ex:0x1012)이고, 이 변수에 저장된 값이 0x1000인 것이다.

## 요약
- call by value: 원본값 복사
- call by address: 원본 값의 주소 복사
- call by reference: 원본 값의 주소 직접 참조
