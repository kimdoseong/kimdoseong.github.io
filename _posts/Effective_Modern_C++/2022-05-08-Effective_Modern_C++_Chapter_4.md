---
title:  "Effective_Modern_C++_Chapter 4"
excerpt: "스마트 포인터"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2022-05-08
last_modified_at: 2022-06-10

published: true
---

- raw pointer  

	1. 선언만 봐서는 객체인지 배열인지 구분할 수 없다.
	2. 선언만 봐서는 포인터가 객체를 소유하고 있는지 알 수 없다.
	3. 어떻게 delete 해야할지 구체적인 정보를 얻을 수 없다.
	4. delete 를 사용해야할지 delete[] 를 사용해야 할지 알 수 없는 경우가 있다.
	5. delete 가 정확히 한번 일어남을 보장하기 어렵다.
	6. 포인터가 객체를 잃어버렸는지 알아내는 방법이 없다. (dangling pointer)

- smart pointer

	**raw pointer** 의 단점을 보완하기 위해 사용한다.  
	smart pointer 종류는 네가지로 std::auto_ptr, std::unique_ptr, std::shared_ptr, std::week_ptr 가 있다.  


## <span style="color:#8F7CEE">  1. 소유권 독점 자원의 관리에는 std::unique_ptr을 사용하라 </span>

std::unique_ptr 는 독점적 소유권(exclusive ownership) 의 객체 이다.  

nullptr 이 아닌 std::unique_ptr 는 항상 자기이 가리키는 객체를 소유한다.  

std::unique_ptr 를 이동하면 소유권이 원본 포인터에서 대상 포인터로 옮겨진다.  

이동 전용 형식(move-only type) 이며 복사되지 않는다.  

```c++
	std::unique_ptr<Base> ptr1(new Base); // OK - 생성자로 생성

	std::unique_ptr<Base> ptr2(ptr1); // Error - 복사 금지!!

	std::unique_ptr<Base> ptr2 = ptr1; // Error - 대입 금지!!

	std::unique_ptr<Base> ptr2 = new Base; // Error - raw pointer 대입 금지!!

	std::unique_ptr<Base> ptr2 = std::make_unique<Base>(); // OK - C++14 이상 부터 make_unique 를 통한 생성
```

**커스텀 삭제자 (custom deleter)**

std::unique_ptr 객체를 생성할 때 커스텀 삭제자를 사용하도록 지정할 수 있다.  

기존의 방식과 다른 소멸 방식을 원할때 함수 혹은 람다를 사용할 수 있다.  


```c++
// 커스텀 삭제자 
auto delBase = [](Base* base)
{
  makeLog(base); // 소멸시 로그 생성하는 
  delete base; // 커스텀 삭제자
};

//  반환 형식이 바뀌었음
std::unique_ptr<Base, decltype(delBase)> ptr(new Base, delBase);
```

 *기억해 둘 사항들*
 - std::unique_ptr 는 독점 소유권 의미론을 가진 자원의 관리를 위한, 작고 빠른 이동 전용 스마트 포인터이다.
 - 기본적으로 자원 파괴는 delete를 통해 일어나나, 커스텀 삭제자를 지정할 수 있다. 상태있는 삭제자나 함수 포인터를 사용하면 std::unique_ptr 객체의 크기가 커진다.
 - std::unique_ptr 를 std::shared_ptr로 손쉽게 변환할 수 있다.

## <span style="color:#8F7CEE">  2. 소유권 공유 자원의 관리에는 std::shared_ptr를 사용하라 </span>

 여러 객체에서 소유하는 경우에는 unique_ptr 를 사용할 수 없다. 이때는 shared_ptr 을 사용한다. shared_ptr 은 참조 횟수(reference count)를 사용하여 그 값이 0이 되면 자원을 파괴한다.  

1. shared_ptr 의 크기는 raw pointer의 두배이다.

	내부적으로 자원을 가리키는 raw pointer 와 자원의 참조 횟수를 가리키는 raw pointer 또한 저장해야 하기 때문.

2. 참조 횟수를 담을 메모리를 반드시 동적으로 할당해야 한다. 

	참조 횟수를 가리키는 객체는 공유 되는 자원이다.

3. 참조 횟수의 증가와 감소가 반드시 원자적 연산이어야 한다. 

	멀티 스레딩 방식 프로그램에서도 안전하게 atomic 한 연산을 해야한다.
	이동 연산은 참조 횟수를 증가시키지 않는다. (기존 ptr 이 null 이 되므로)   

**커스텀 삭제자**

unique_ptr 과 마찬가지로 커스텀 삭제자를 구현할 수 있다.

```c++
auto logDel = [](Base* ptr){
	makeLog(ptr);
	delete ptr;
}

// 삭제자가 포인터 형식의 일부이다.
std::unique_ptr<Base, decltype(logDel)> up(new Base, logDel);

// 그렇지 않음!
std::shared_ptr<Base> sp(new Base, logDel);
```

포인터 형식에 커스텀 삭제자를 두지 않음에 따라 유연하게 설계가 가능하다.

```c++
auto del1 = [](Base* ptr){...}
auto del2 = [](Base* ptr){...}

// 각각 다른 커스텀 삭제자를 사용하여 생성
std::shared_ptr<Base> sp1(new Base, del1);
std::shared_ptr<Base> sp2(new Base, del2);


// 포인터 타입이 동일하므로 가능하다
std::vector<std::shared_ptr<Base>> vsp{sp1, sp2};
```

**Control Block**

커스텀 삭제자를 지정해도 std::shared_ptr 객체의 크기가 변하지 않는다. std::shared_ptr 객체의 크기는 항상 포인터 두 개 분량이다.

객체를 지칭하는 포인터 하나와 control block 을 지칭하는 포인터 하나로 이루어져 있다.

생 포인터 하나로 여러개의 std::shared_ptr 를 생성하면 안된다.

```c++
auto ptr = new Base;

std::shared_ptr<Base> p1(ptr, logDel);

std::shared_ptr<Base> p2(ptr, logDel);
```

p1 과 p2 가 각각 ptr 에 대한 제어 블록을 갖고 있어서 ptr 에 대한 파괴가 두번 시도된다.

std::make_shared 로는 커스텀 삭제자를 지정할 수 없다.


 *기억해 둘 사항들*
 - std::shared_ptr 는 임의의 공유 자원의 수명을 편리하게(쓰레기 수거에 맡길 때만큼이나) 관리할 수 있는 수단을 제공한다.
 - 대체로 std::shared_ptr 객체는 그 크기가 std::unique_ptr 객체의 두 배이며, 제어 블록에 관련한 추가 부담을 유발하며, 원자적 참조 횟수 조작을 요구한다.
 - 자원은 기본적으로 delete 를 통해 파괴되나, 커스텀 삭제자도 지원된다. 삭제자 형식은 std::shared_ptr 의 형식에 아무런 영향도 미치지 않는다.
 - 생 포인터 형식의 변수로부터 std::shared_ptr 를 생성하는 일은 피해야 한다.

## <span style="color:#8F7CEE">  3. std::shared_ptr 처럼 작동하되 대상을 잃을 수도 있는 포인터가 필요하면 std::weak_ptr 를 사용하라 </span>

std::weak_ptr 은 std::shared_ptr 와 비슷하되 참조 횟수에는 영향을 미치지 않는다.

```c++
auto spw = std::make_shared<Base>();
// shared_ptr 를 생성하고 spw 객체의 참조 횟수는 1이다.

std::weak_ptr<Base> wpw(spw);
// wpw 는 spw 와 같은 Base 를 가르키며 참조 횟수는 여전히 1이다.

spw = nullptr; 
// 참조 횟수가 0이 되고 wpw 는 대상을 잃은 상태이다.
```

std::weak_ptr 가 객체를 가리키고 있어도 다른 std::shared_ptr 들이 가리키지 않는다면 함께 소멸된다.

weak_ptr 자체로는 원래 객체를 참조할 수 없고, 반드시 shared_ptr 로 변환해서 사용해야 한다.  

```c++
std::shared_ptr<Base> spw1 = wpw.lock();
// std::weak_ptr 이 가리키는 객체를 std::shared_ptr 로 반환한다. 
// 이제 spw1 객체로 사용하면 된다.

auto spw2 - wpw.lock();
// 같은 내용이지만 auto 사용


if(spw2)
	std::cout << " 참조 " << std::endl;
else
	std::cout << " nullptr " << std::endl; 
```

*기억해 둘 사항들*
 - std::shared_ptr 처럼 작동하되 대상을 잃을 수도 있는 포인터가 필요하면 std::wark_ptr 를 사용하라.
 - std::weak_ptr 의 잠재적인 용도로는 캐싱, 관찰자 목록, std::shared_ptr 순환 고리 방지가 있다.

## <span style="color:#8F7CEE">  4. new 를 직접 사용하는 것보다 std::make_unique 와 std::make_shared 를 선호하라 </span>

c++11 에 std::make_unique 가 없음 따라서 기본적인 구현을 해야한다.

```c++
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params){
	return std::unique<T>(new T(std::forward<Ts>(param)...));
}
```

new 보다 make 함수들을 사용해야 하는 이유

1. 중복을 피한다.

```c++
auto upw1(std::make_unique<Base>()); // make

std::unique_ptr<Base> upw2(new Base); // 사용하지 않음


auto spw1(std::make_shared<Base>()); // make 

std::shared_ptr<Base> spw(new Base); // 사용하지 않음
```

new 버전은 객체의 형식이 중복되어 나오지만 make 함수 버전은 그렇지 않다.

소스 코드에 중복이 있으면 컴파일 시간이 늘어나고 object 코드의 덩치가 커지고 code base 를 다루기가 좀 더 어려워 진다.


2. 예외 안정성

```c++
function(std::shared_ptr<Base>(new Base), Priority());
```

그냥 보기에는 큰 문제가 없어 보이는 코드이지만 코드가 실행되는 순서는 보장되지 않는다.

(1) "new Base" 실행
(2) Priority() 실행
(3) std::shared_ptr 생성자 실행

이와 같이 코드를 실행 한다면 2단계 Priority 함수에서 예외를 던지면 1단계  Base 객체의 누수가 생기게 된다.  

Base 객체를 관리할 std::shared_ptr 생성자는 3단계에 실행되므로 2단계에서 예외를 던질때는 아직 생 포인터가 가리키는  

동적할당 객체일 뿐이다.


- 아래와 같이 바꾸어 주면 된다.  

```c++
function(std::make_shared<Base>(), Priority()); // 누수의 위험이 없음
```

또한 방금 본 바와 같이 new 를 사용하게 되면 두번의 메모리 할당이 일어난다.  

(1) new 를 통한 메모리 할당
(2) std::shared 의 제어 블록을 위한 메모리 할당

그러나 make 를 사용하면 한번의 할당만 일어난다.

Base 객체와 제어 블록을 담을수 있는 크기의 메모리 조각이 한번에 할당되기 때문이다.

3. std::initializer_list 사용

```c++
auto upv = std::make_unique<std::vector<int>>(10, 20);
// 모든 값이 20인 요소 10개 짜리 std::vector<int>를 생성한다.

```


객체를 중괄호 초기치로 생성하려면 반드시 new를 직접 사용해야 한다. 

하지만 여기에는 std::initializer_list를 사용한 우회책이 있다.

```c++
// std::initializer_list 객체 생성
auto initList = {10, 20};

// std::initializer_list 객체 이용해서 std::vector 생성
auto spv = std::make_shared<std::vector<int>>(initList);
```

4. custom operator new, delete

custom operator 로 new, delete 가 있다면 make 함수로 생성하는 것은 대체로 바람직 하지 않다.  

std::allocate_shared가 요구하는 메모리 조각의 크기는 동적으로 할당되는 객체의 크기가 아니라 그 크기에 제어 블록의 크기를 더한것 이기 때문이다.  

*기억해 둘 사항들*
- new의 직접 사용에 비해, make 함수를 사용하면 소스 코드 중복의 여지가 없어지고, 예외 안전성이 향상되고, std::make_shared와 std::allocate_shared의 경우 더 작고 빠른 코드가 산출된다.
- make 함수의 사용이 불가능 또는 부적합한 경우로는 커스텀 삭제자를 지정해야 하는 경우와 중괄호 초기치를 전달해야 하는 경우가 있다.
- std::shared_ptr 에서는 make 함수가 부적합한 경우가 더 있는데, 두가지 예를 들자면 (1) 커스텀 메모리 관리 기능을 가진 클래스를 다루는 경우와 (2) 메모리가 넉넉하지 않은 시스템에서 큰 객체를 자주 다루어야 하고 std::weak_ptr 들이 해당 std::shared_ptr 들 보다 더 오래 살아남는 경우이다.

## <span style="color:#8F7CEE">  5. Pimpl 관용구를 사용할 때에는 특수 멤버 함수들을 구현 파일에서 정의하라 </span>

컴파일 시간을 줄이기 위해 Pimpl (Pointer to implementation idiom) 을 사용한다.

```c++
class Base{
:
	...
private:
	std::string name;
	std::vector<int> data;
	Gadget g1;
}
```

std::string, std::vector, Gadget 형식을 가지므로 헤더 파일또한 포함해야 한다. 

따라서 컴파일 시간이 증가하며 헤더들의 내용에 의존하게 된다.

- Pimpl 관용구 

*Header file*

```c++
// .h
class Base{
	...

private:
	// 선언만 하고 cpp 파일에서 정의함
	struct Impl
#if 0
	Impl* pImpl; // c++98
#else
	std::unique_ptr<Impl> pImpl; // modern c++
#endif
}

```

*cpp file*

```c++
// .cpp
#include "Base.h"
#include "Gadget.h"
#include <vector>

struct Base::Impl{
	std::string name;
	std::vector<int> data;
	Gadget g1;
}; // .cpp file 에서 구현

#if 0 // c++98
Base::Base(){
	: pImpl(new Impl) // new 로 생성
}

Base::~Base(){
	delete pImpl; // new 할당에 대한 소멸
}

#else // modern c++ 소멸자 또한 제거 되었다.
Base::Base(){
	: pImpl(std::make_unique<Impl>()) // make
}
#endif
```

이 코드 자체는 컴파일이 잘 되지만 다음과 같은 코드는 error 를 발생시킨다.

```c++
#include "Base.h"

Base b; // Error !!
```

*Error Message*

```text

: 컴파일되는 클래스 템플릿 인스턴스화 'std::default_delete<Base::Impl>'에 대한 참조를 확인하세요.
: 컴파일되는 클래스 템플릿 인스턴스화 'std::unique_ptr<Base::Impl,std::default_delete<Base::Impl>>'에 대한 참조를 확인하세요.
: error C2338: static_assert failed: 'can't delete an incomplete type'
: warning C4150: 불완전한 형식 'Base::Impl'에 대한 포인터를 삭제했습니다. 소멸자가 호출되지 않습니다.
: message : 'Base::Impl' 선언을 참조하십시오.

```

Error 발생 이유

1. 소멸자가 호출되는데  std::unique_ptr 를 사용하는 Base 클래스는 따로 소멸자를 선언하지 않았다. (할 필요가 없으므로)
2. 하여 special member functions 자동 생성 규칙에 의해 소멸자가 작성된다.
3. 기본 삭제자를 사용하였고, 기본 삭제자는 std::unique_ptr 내부의 raw pointer 에 대해 delete 를 적용한다.
4. 그런데 대부분 C++ 표준 라이브러리 delete 전에 raw pointer가 불완전한 형식을 가리키는지 C++11의 static assert 로 점검한다.
5. special member functions 는 inline 으로 작성되며, 구현부의 정의는 확인하지 않으므로 Impl 이 불완전한 형식이라고 간주한다.

따라서 컴파일 에러가 발생하고, 빌드가 안된다.


해결 방법

std::unique_ptr 파괴되는 지점에서 Base::Impl 이 완전한 형식이 완전하게 하면 된다.

따라서 소멸자 (파괴되는 지점) 를 구현부에 두면 컴파일러가 cpp file 을 확인 하게 되며, Impl 의 정의 또한 확인하여 불완전한 형식이 아님을 확인한다.

*Header file*
```c++
...

~Base();

...
```

*cpp file*

```c++
Base::~Base(){}

//or

Base::~Base() = default;
```

이동 연산들 또한 마찬가지이다. 

std::shared_ptr 는 이 항목이 적용되지 않으며 따라서 소멸자를 선언해줄 필요가 없다.

이 차이는 커스텀 삭제자를 지원하는 방식의 차이에서 비롯된다.

std::unique_ptr 는 삭제자가 포인터 형식에 일부이며, 이 덕분에 컴파일러는 더 작은 실행시점 자료구조와 더 빠른 실행시점 코드를 만들어 낼 수 있다.  

std::shared_ptr 는 삭제자가 포인터 형식이 아니며, 실행 코드도 느리고 구조의 크기도 커지지만, 컴파일러가 작성한 special memember functions 가 쓰이는 시점에서 피지칭 형식들이 완전한 형식이어야 한다는 요구조건이 사라진다.  


*기억해 둘 사항들*
- Pimpl 관용구는 클래스 구현과 클래스 클라이언트 사이의 컴파일 의존성을 줄임으로써 빌드 시간을 감소한다.
- std::unique_ptr 형식의 pImpl 포인터를 사용할 때에는 특수 멤버 함수들을 클래스 헤더에 선언하고 구현 파일에서 구현해야 한다. 컴파일러가 기본으로 작성하는 함수 구현들이 사용하기에 적합한 경우에도 그렇게 해야 한다. 
- 위의 조건의 std::unique_ptr 에 적용될 뿐 std::shared_ptr 에는 적용되지 않는다.