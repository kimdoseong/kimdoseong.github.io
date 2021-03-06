---
title:  "Effective_C++_Chapter 8"
excerpt: "new와 delete를 내 맘대로"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]


date: 2021-08-08
last_modified_at: 2021-08-08

published: true
---

##   <span style="color:#9282CD">1. new 처리자의 동작 원리를 제대로 이해하자</span>

- set_new_handler 함수를 사용하여 메모리 할당에 실패하면 동작할 함수를 지정할 수 있다.

- 사용자가 부탁한 만큼 메모리를 할당하지 못하면 operator new는 충분한 메모리를 찾을 때까지 new 처리자를 되풀이해서 호출한다. new 처리자 함수가 다음 동작 중 하나를 꼭 해야 한다.

  - 사용할 수 있는 메모리를 더 많이 확보한다.

    프로그램이 시작할 때 메모리 블록을 크게 하나 할당해 놓았다가 new 처리자가 가장 처음 호출될 때 그 메모리를 쓸 수 있도록 허용하는 방법.

  - 다른 new 처리자를 설치한다.

    현재의 new 처리자 안에서 set_new_handler를 호출

  - new 처리자의 설치를 제거.

    set_new_handler에 널 포인터를 넘긴다. operator new는 메모리 할당이 실패하면 예외를 던지게 된다.

  - 복귀하지 않는다.

    abort 혹은 exit을 호출.

**Example** 

▶다른 new 처리자와 new 처리자의 설치 제거.

```c++
//헤더 
class Widget {
private:
	static std::new_handler currentHandler;
public:
	static std::new_handler set_new_handler(std::new_handler p) throw();
	static void* operator new(std::size_t size) throw(std::bad_alloc);
};

//구현부 start
std::new_handler Widget::currentHandler = nullptr;    //초기화.
std::new_handler Widget::set_new_handler(std::new_handler p) throw() //예외를 던지지 않는다.
{
	std::new_handler oldHandler = currentHandler;
	currentHandler = p; //handler를 저장해 둔다.
	return oldHandler; 
}

void* Widget::operator new(std::size_t size) throw(std::bad_alloc)
{
	NewHandlerHolder   // Widget의 저장해둔 handler 설치
		h(std::set_new_handler(currentHandler));  
	return ::operator new(size); // 메모리를 할당하거나
			// 할당이 실패하면 예외를 던진다.
}
//end

// 이전에 쓰이고 있던 전역 new 처리자
// 복원의 자동화를 위한 객체
class NewHandlerHolder {
private:
	std::new_handler handler;                        //handler
	NewHandlerHolder(const NewHandlerHolder&);        //private 선언으로 복사를 막기 위한 부분
	NewHandlerHolder&
		operator=(const NewHandlerHolder&);
public:
	explicit NewHandlerHolder(std::new_handler nh)   // 현재의 new 처리자를 획득
		: handler(nh) {}                             //operator new에서 호출.

	~NewHandlerHolder(void) {                       
		std::set_new_handler(handler);
	}
};

void outOfMem(){...}

//main
Widget::set_new_handler(outOfMem); // Widget의 new 처리자 함수로서
								// outOfMem을 설치.

Widget* pw1 = new Widget;          // 메모리 할당이 실패하면
								// outOfMem이 호출됩.

std::string* ps = new std::string; // 메모리 할당이 실패하면
								// 전역 new 처리자 함수가
								// (있으면) 호출.

Widget::set_new_handler(nullptr);       // Widget 클래스만을 위한
									// new 처리자 함수가 아무것도 없도록
									// 한다(즉, nullptr로 설정합니다).

Widget* pw2 = new Widget;          // 메모리 할당이 실패하면 이제는
								// 예외를 바로 던진다.(Widget
								// 클래스를 위한 new 처리자 함수가
								// 없다. ->기본으로 설정해둔 throw(std::bad_alloc) 호출 되겠죠?
```



▶믹스인 양식

```c++
class NewHandler Holder {};

template <typename T>                // 클래스별 set_new_handler를 설정.
class NewHandlerSupport {            //Widget에서 정의한 opertor new, set_new_handler를 여기에 정의.
private:
	static std::new_handler currentHandler;
public:                                
	static std::new_handler set_new_handler(std::new_handler p) throw();
	static void* operator new(std::size_t size) throw(std::bad_alloc);
};
 //Widget에서 정의한 opertor new, set_new_handler를 여기에 정의.
template <typename T>
std::new_handler NewHandlerSupport<T>::set_new_handler(std::new_handler p) throw()
{
	std::new_handler oldHandler = currentHandler;
	currentHandler = p;
	return oldHandler;
}
 //Widget에서 정의한 opertor new, set_new_handler를 여기에 정의.
template <typename T>
void* NewHandlerSupport<T>::operator new(std::size_t size) throw(std::bad_alloc)
{
	NewHandlerHolder h(std::set_new_handler(currentHandler));
	return ::operator new(size);
}

// 클래스별로 만들어지는 currentHandler 멤버를 nulptr로 초기화.
template <typename T>
std::new_handler NewHandlerSupport<T>::currentHandler = nullptr;

// NewHandlerSupport<Widget>을 상속만 받으면 끝.
//CRTP(curiously recurring tempate pattern)
class Widget : public NewHandlerSupport<Widget> {
//NewHandlerSupport에서 정의한 set_new_handler, operator new에 대한 선언문이 빠진다.
};

```



## <span style="color:#9282CD">2. new 및 delete를 언제 바꿔야 좋은 소리를 들을지 파악해 두자</span>

- 개발자가 new및 delete를 재정의 하는 데는 여러가지 나름의 타당한 이유가 있다.
- 수행 성능 향상, 힙 에러 디버깅, 힙 사용 정보 수집 목적 등이 있다.

1. 잘못된 힙 사용 탐지

   오버런, 언더런 발생 등 잘못된 힙 사용을 탐지 하기 위한 목적

2. 힙 메모리의 실제 사용에 관한 통계 정보를 수집

3. 할당 및 해제 속력을 높이기 위해

4. 기본 할당자의 바이트 정렬 동작을 보장하기 위해

   - 32bit 아키텍쳐에서 double이 8 바이트 단위로 정렬되어 있을 때 읽기, 쓰기가 가장 빠르다 하지만 시중의 컴파일러에서 new 함수가 double에 대한 동적 할당이 8 바이트 정렬을 보장하지 않는 경우가 있다. 이 때 사용자 정의 버전으로 바꾸어 성능을 올릴 수 있다

5. 원하는 동작을 수행하도록 하기 위해

   컴파일러가 주는 new 및 delete가 하지 못하는 일을 해주었으면 바라는 때가 있기 마련이다.

   이 때 사용자 정의 버전을 만든다.



## <span style="color:#9282CD">3. new 및 delete를 작성할 때 따라야 할 기존의 관례를 잘 알아두자</span>

- operator new 함수는 메모리 할당을 반복해서 시도하는 무한 루프를 가져아 한다.
- 메모리 할당 요구를 만족시킬 수 없을 때  new 처리자를 호출해야 하며 0 바이트에 대한 대책도 있어야 한다.
- 클래스 전용 버전은 자신이 할당하기로 예정된 크기보다 더 큰 메모리 블록에 대한 요구도 처리해야 한다.
- operator delete 함수는 nullptr가 들어왔을 때 아무일도 하지 않아야 한다.
- 마찬가지로 클래스 전용 버전은 예정된 크기보다 더 큰 블록을 처리해야 한다.



## <span style="color:#9282CD">4. 위치지정 new를 작성한다면 위치지정 delete도 같이 준비하자</span>

- operator new 함수의 위치지정 버전을 만들 때는, 이 함수와 짝을 이루는 위치지정 버전의 opertator delete 함수도 꼭 만들자 (메모리 누수 발생)
- new및 delete 위치지정 버전을 선언할 때는 의도한 바도 아닌데 이들의 표준 버전이 가려지는 일이 생기지 않게 하자



**Example** 

▶위치지정 버전 new - delete 

```c++
class Widget {
public:
	static void* operator new(std::size_t size, std::ostream& logStream) throw(std::bad_alloc);
	static void operator delete(void* pMemory, std::ostream& logStream) throw();
};

//위치지정 버전 new와 delete를 세트로 만들어주자.

//위처럼 추가 매개변수를 취하는 opertator new 함수가 있으면 그것과 
//똑같은 추가 매개변수를 받는 opertator delete가 짝으로 존재하지 않으면 
//operator new에서 예외가 발생하면 delete를 호출하지 않는다.
    
```



▶표준 버전이 가려질 때 

```c++
//위처럼 위치지정 버전을 선언하면 표준버전 new와 delete를 사용하지 못하게 된다.

Widget* pw = new Widget; // Error 발생
delete pw; // Error 발생

//기본버전 new와 delete도 선언해 준다.

//세가지 표준
void* operator new(std::size_t) throw(std::bad_alloc); //기본형 new
void* operator new(std::size_t, void*) throw() //위치지정 new
void* operator new(std::size_t, const std::nothrow_t&) throw(); //예외불가 new

//위의 세가지 버전의 new와 delete를 선언, 정의한 클래스를 상속받아 사용하면 깔끔하다.
```



