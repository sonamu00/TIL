## Call By Value
### Ư¡
- ���� ���� �����̴�.
- �Լ��� ���� �����ϸ� ���� �Լ��� �Ű������� ���簡 �ȴ�.
- ������ ���̱� ������ �Լ� ������ ������ ���� ������ �� ����.
### ����
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
// ��� : 10
```
- ���ڷ� ���� ���� ���� 1�� ���ϴ� functionA�� �����ص� n�� ���� �ٲ��� �ʴ´�.
- �Լ��� ȣ���� �� ��(a)�� �����ؼ� �Ű�����(value)�� ��ȣ�� �Լ�(justFunc)�� �Ѱ��ش�.
- �ּ��� ���� ���� �����ؼ� �����ϴ� ���� �ƴϱ� ������ ���� ������ �� ����.

## Call By Reference
### Ư¡
- �Լ����� �Լ� �ܺ� �޸� ������ ������ �� ����Ѵ�.
- �Լ� ����� �Ű������� &�� ����� ������ �ּҸ� �޵��� �ϰ� �Լ� ���ο����� �ּҸ� �� ������ �Ϲ� ����ó�� ����Ѵ�.
- call by valueó�� �޸� ������ �Ҵ� �޴°��� �ƴ� ���� ������ �������� �����ϴ� ���̴�.
- �� ���� ��ü�� �����Ͽ� ���� ������ ���� �ϱ� ������ �Լ� ������ ���� ������ �ص� ���� ������ �� ��ü�� �ٲ��.

 
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
// ��� : 11
```
-  functionB�� main �Լ��� �޸𸮸� ��� ����ϰ� �ִ��� ���� �𸥴�.
-  main �Լ��� ����ϰ� �ִ� �޸� ������ ���� ��ǥ�� �� ������ ���� �����ϴ� ���̴�.

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
// ��� : 10, 2
```
- main �Լ��� a�� ���� �ּ� ���� 1�� ���� ��ġ�� b�� �־���, b�� ��ġ���� ���� ����(++)�� �ϴ� �ᱹ b�� ���� �þ��.
- ������ ���� ȣ���� ��ǥ�� ��� ����̱� ������ ������ �ϸ� �ּ� �� ��ġ ������ �ǰ� �ȴ�.



## Call By Address
### Ư¡
- �Ű������� �������� ������ �����Ѵ�.
- ������ ������ ������ ������ �ּҰ��� �����ϰ� ������ �ּҸ� �����Ͽ� ����Ű�� ���� ���� �����ϸ� ������ �����Ͱ� �ٲ��.
- �Լ� ���ڿ� ������ �ּ� ���� �����ؼ� �����ϱ� ������ call by value �����̴�.
- �ּ� ���� ������ �޸� ������ ����ȴ�.



### ����
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
// ��� : 1
```
- functionD �����ϰ� main�Լ��� ���� a�� ���� 1�� �ٲ��.
- a�� �ּҸ� �����ؼ� �Ű�����(v)�� �Ѱ��ְ� v(a�� �ּ�)�� ����Ű�� �ּҿ� 1�� ���ؼ� ������ ���� �����ߴ�.
- functionD�� int* v�� functionD ���ÿ��� ���ǵ� ���ο� ������ �����̸�, int* v�� �ּҰ��� ���� �ּҰ�(ex:0x1000)�� �ƴ�, functionD���� ������ ���ο� �ּҰ�(ex:0x1012)�̰�, �� ������ ����� ���� 0x1000�� ���̴�.

## ���
- call by value: ������ ����
- call by address: ���� ���� �ּ� ����
- call by reference: ���� ���� �ּ� ���� ����