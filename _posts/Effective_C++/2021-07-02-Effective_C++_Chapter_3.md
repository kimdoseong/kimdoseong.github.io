---
title:  "Effective_C++_Chapter 3"
excerpt: "자원 관리"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]


date: 2021-07-02
last_modified_at: 2021-07-02

published: true
---

## <span style="color:#8F7CEE">1. 자원 관리에는 객체가 그만!  </span>

- 자원 해제는 자원을 객체에 넣고 그 자원 해제를 소멸자가 맡도록 한다.  
- 소멸자가 자동으로 delete를 불러주도록 설계된 스마트 포인터를 사용하면 된다.  
- auto_ptr과 shared_ptr은 소멸자 내부에서 delete 연산을 한다. (delete[]가 아님을 주의) 따라서 동적 배열에 사용하지 말자.  
- 배열에 쓸 수 있는 auto_ptr이나 shared_ptr은 boost에 있다.  

첫째, 자원을 획득한 후에 자원 관리 객체에 넘기자.  

둘때, 자원 관리 객체는 자신의 소멸자를 사용해서 자원이 확실히 해제되도록 한다.  

 **Example**

▶ auto_ptr 객체를 복사하면 원본 객체는 null을 만든다. (auto_ptr은 개수가 둘 이상이면 절대로 안 된다)  

```c++
auto_ptr<Base> ptr1(new Base("ptr1"));
auto_ptr<Base> ptr2(new Base("ptr2"));

cout << "ptr1_text: " << ptr1->getText() << endl;
cout << "ptr2_text: " << ptr2->getText() << endl;

ptr1 = ptr2; //ptr2가 null이 됩니다.

cout << "ptr1_text: " << ptr1->getText() << endl;
cout << "ptr2_text: " << ptr2->getText() << endl; //error 발생.
```

**결과창**  

```c++
    ptr1_text: ptr1  
    ptr2_text: ptr2  
    ptr1_text: ptr2
```



## <span style="color:#8F7CEE">2. 자원 관리 클래스의 복사 동작에 대해 진지하게 고찰하자 </span>

- RAII 객체의 복사는 그 객체가 관리하는 자원의 복사 문제를 안고 가기 때문에, 그 자원을 어떻게 복사하느냐에 따라 RAII 객체의 복사 동작이 결정된다.  
- 복사 동작을 금지하거나, 참조 카운팅하는 방법을 사용하자.  

## <span style="color:#8F7CEE">3. 자원 관리 클래스에서 관리되는 자원은 외부에서 접근할 수 있도록 하자  </span>

- RAII 클래스를 만들 때는 그 클래스가 관리하는 자원을 얻을 방법을 열어 주어야 합니다.  
- 안정성 -> 명시적 변환 , 고객 편의성 -> 암시적 변환  

RAII 클래스를 실제 자원으로 얻어낼 필요가 있다.(매개변수, 등) RAII 클래스 자체로 매개 변수에  넣는 것은 불가능 하므로 명시적 변환 혹은 암시적 변환을 사용하면 된다.  

▶ RAII 클래스에 정의  

```c++
    Customer RAII::get() const { return _raii; } //명시적 변환 함수
    RAII::operator Customer() const { return _raii; } //암시적 변환 함수
```

## <span style="color:#8F7CEE">4. new 및 delete를 사용할 때는 형태를 반드시 맞추자 </span>

- new -> delete
- new [] -> delete []

## <span style="color:#8F7CEE">5. new로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자  </span>

- new로 생성한 객체를 스마트 포인터로 넣는 코드는 별도의 한 문장으로 만들자  
- 예외가 발생할 때 디버깅하기 힘든 자원 누출이 초래될 수 있다  

매개변수의 순서는 컴파일러 제작사 마다 다르다. 따라서 미정 동작으로 빠지지 않게 설계하자.  

▶ 메모리 누수가 생길 수 있으니  new로 객체를 생성하는 한 문장을 만들자.  

```c++
	shared_ptr<Base> ptr1(new Base()); //한문장으로 만들고
	printBase(ptr1);// 매개변수로 사용

	printBase(shared_ptr<Base>(new Base())); //컴파일은 되지만 미정의 동작으로 메모리 누수가 생길 수 있다.
```









