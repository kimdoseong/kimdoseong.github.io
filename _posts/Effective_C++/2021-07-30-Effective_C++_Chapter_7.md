---
title:  "Effective_C++_Chapter 7"
excerpt: "템플릿과 일반화 프로그래밍"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]


date: 2021-07-30
last_modified_at: 2021-07-30

published: true
---
##  <span style="color:#9282CD">1. 템플릿 프로그래밍의 천릿길도 암시적 인터페이스와 컴파일 타입 다형성 부터</span>

- 클래스 및 템플릿은 모두 인터페이스와 다형성을 지원한다.
- 클래스의 경우, 인터페이스는 명시적이고 함수의 시그니처 중심 구성, 다형성은 프로그램 실행 중에 가상 함수를 통해 나타난다.
- 템플릿 매개변수의 경우 인터페이스는 암시적이며, 유효 표현식에 기반을 두어 구성, 다형성은 컴파일 중에 템플릿 인스턴스화와 함수 오버로딩 모호성 해결을 통해 나타난다.

**Example** 

▶명시적 ­ - 암시적 인터페이스와 컴파일 타임 다형성

```c++
class Widget {
public:
	Widget() {};
	virtual ~Widget() {};
	virtual std::size_t size(void) const; //가상 함수는 w의 동적 타입 기반으로
	virtual void normalize(void); //런타임에 결정된다.
	void swap(Widget& other);
};

void doProcessing(Widget& w) //w가 Widget type으로 Widget 인터페이스를 지원해야한다.
{//명시적 인터페이스
    if (w.size() > 10 && w != someNastyWidget) {
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}

template <typename T>
void doProcessing(T& w) //복사 생성자, size(), swap등 멤버 함수를 T에서 지원 해야한다.
{//암시적 인터페이스.
	if (w.size() > 10 && w != someNastyWidget) { 
		t temp(w); 
		temp.normalize();
		temp.swap(w);
	}
}//w가 수반되는 함수 호출이 일어날 때 인스턴스화가 일어나기 때문에 
//컴파일 타임 다형성이라 한다.
```



## <span style="color:#9282CD">2. typename의 두가지 의미를 제대로 파악하자</span>

- 템플릿 매개변수를 선언할 때 class와 typename은 서로 바꾸어 써도 된다.
- 중첩 의존 타입 이름을 식별하는 용도는 typename을 반드시 사용해야 한다.
- 중첩 의존 이름이 기본 클래스 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로 있는 경우에는 예외.

**Example**

▶중첩 의존 이름에 typename을 붙여야 하는 경우

```c++
template <typename T>
void print(const T& container)
{
	T::const_iterator citer(container.begin);
} //T 클래스 데이터 멤버 const_iterator가 된다..
//중첩 의존 이름이라 한다.

typename T::const_iterator citer(container.begin);
//typename을 꼭 붙여주자.
```

▶typedef와 typename 같이 쓰는 경우

```c++
template <typename T>
void print(const T& container)
{   //typename 앞에 typedef를 사용합니다.
	typedef typename std::iterator_traits<T>::value_type value_type; //멤버 이름과 똑같이 짓는다.
	value_type temp(*container);
}

```



## <span style="color:#9282CD">3.템플릿으로 만들어진 기본 클래스 안의 이름에 접근하는 방법을 알아 두자</span>

- 파생 클래스 템플릿에서 기반 클래스 템플릿의 이름을 참조할 때, "this->"를 접두사로 붙이거나, 기반 클래스 한정문을 명시적으로 써주자.

**Example**

▶상속 받은 기반 클래스가 템플릿인 경우 함수 사용법

```c++
class CompanyA {
public:
	void send(std::string msg) {
		cout << "CompanyA Send Msg: " << msg <<endl;
	}
}; 

class CompanyB {
public:
	void send(std::string msg) {
		cout << "CompanyB Send Msg: " << msg << endl;
	}
}; //Company 클래스 입니다.

template <typename Company>
class MsgSender {
public:
	void MsgSend(std::string msg) {
		Company c;
		c.send(msg); //사용하는 type이 send(std::string)을 지원해야 합니다.
	}
};

template <typename Company>
class MsgLogger :public MsgSender<Company> {
public:
	//using MsgSender<Company>::MsgSend; //using 을 사용하거나,
	void MsgSendLog(std::string msg) {
		cout << "Send Msg Start" << endl;
		this->MsgSend(msg); //this-> 를 사용합니다.
		//MsgSend(msg); //using으로 사용할 때 방법
		cout << "Send Msg Success" << endl;
	}
};

MsgLogger<CompanyA> mlca;
mlca.MsgSendLog("Hello");

MsgLogger<CompanyB> mlcb;
mlcb.MsgSendLog("Hello");
```

결과창

```c++
Send Msg Start//CompanyA로 호출한 경우
CompanyA Send Msg: Hello
Send Msg Success
    
Send Msg Start //CompanyB로 호출한 경우
CompanyB Send Msg: Hello
Send Msg Success
```

▶완전 템플릿 특수화

```c++
template <typename Company>
class MsgSender {
public:
	void MsgSend(std::string msg) {
		Company c;
		c.send(msg);
	}
}; //일반적인 템플릿

template<> //void 상태 Company 템플릿 뒤에 선언해줍니다.
class MsgSender<CompanyA> { //여기에 CompanyA 클래스에 대해 특수화 해줍니다.
public:
	void MsgSend(std::string msg) {
		CompanyA c;
		c.send(msg);
		c.send(msg);
	}
};

MsgLogger<CompanyA> mlca;
mlca.MsgSendLog("Hello");
//만약 MsgSendLog 함수에서 호출하는 MsgSend가 MsgSender<CompanyA> 클래스에 없으면
//컴파일 되지 않습니다. 컴파일러가 해당 함수가 없다는 것을 알고 있습니다.
MsgLogger<CompanyB> mlcb;
mlcb.MsgSendLog("Hello");
```



결과창

```c++
Send Msg Start //Company 객체는 특수화된 템플릿에 타게 됩니다.
CompanyA Send Msg: Hello //물론 MsgSender<CompanyA> 클래스에 
CompanyA Send Msg: Hello //MsgSend 함수가 있어야만 합니다.
Send Msg Success
    
Send Msg Start //CompanyB 객체로 기본 템플릿 클래스 MsgSend 호출 됩니다.
CompanyB Send Msg: Hello //특수화 되지 않았습니다.
Send Msg Success
```



## <span style="color:#9282CD">4. 매개변수에 독립적인 코드는 템플릿으로부터 분리시키자</span>

- 템플릿을 사용하면 비슷비슷한 클래스와 함수가 여러 벌 만들어 진다. 따라서 템플릿 매개변수에 종속되지 않은 템플릿 코드는 비대화의 원인이 된다.
- 비 타입 템플릿 매개변수로 생기는 코드 비대화의 경우 템플릿 매개변수를 함수 매개변수 혹은 클래스 데이터 멤버로 대체함으로 비대화를 없앨 수 있다.
- 타입 매개변수로 생기는 코드 비대화의 경우 동일한 이진 표현 구조를 가지고 인스턴스화 되는 타입들이 한 가지 함수 구현을 공유하게 만듦으로써 비대화를 감소시킬 수 있다.

**Example**

▶ 비 타입 템플릿 매개변수 ->매개변수 or 클래스 데이터 멤버

변수 'n'과 관련된 함수들만 복사되어 inline 으로 퍼포먼스 향상, 코드 비대화가 줄어든다.  

```c++
template <typename T>
class Calculate {
protected:
	void Mp(std::size_t size) {
		cout << "size*size: " << size * size << endl;
	}
};

template <typename T, std::size_t n> 
class Rectangle :private Calculate<T> {
public:
	using Calculate<T>::Mp; //using을 통한 구현
	void Mp() { Mp(n); }//Calculate Mp() 호출
};

//같은 템플릿 호출로 코드 비대화 나타나지 않음.
Rectangle<int, 3> r1; //Rectangle<int, std::size_t>::Mp 를 호출한다.
r1.Mp(); 

Rectangle<int, 2> r2; //Rectangle<int, std::size_t>::Mp 를 호출한다.
r2.Mp();

//Calculate 클래스가 없다면
template <typename T, std::size_t n> 
class Rectangle {
public:
	void Mp(std::size_t size) {
		cout << "size*size: " << size * size << endl;
	}
}; //이렇게 만 구현한 경우..

//Rectangle<int,3>::Mp
//Rectangle<int,2>::Mp
//호출로 코드 비대화의 원인이 된다.
```

▶ 타입 매개변수로 생기는 코드 비대화 방지

```c++
template <typename T>
class RectangleBase {
private:
	std::size_t _size;
	T* _pData;
protected:
	RectangleBase(T* pData, std::size_t size) :
		_pData(pData), _size(size) {}
};

template <typename T, std::size_t n> 
class Rectangle :private RectangleBase<T> {
private:
	T* _pData;
public:
	Rectangle() :RectangleBase<T>(_pData, n) {}
};
```

## <span style="color:#9282CD">5. "호환되는 모든 타입"을 받아들이는 데는 멤버 함수 템플릿이 직방!</span>

- 호환되는 모든 타입을 받아들이는 멤버 함수를 만들려면 멤버 함수 템플릿을 사용하자
- 일반화된 복사 생성 연산과 일반화된 대입 연산을 위해 멤버 템플릿을 선언했다 하더라도, 보통의 복사 생성자와 복사 대입 연산자는 여전히 직접 선언해야 한다.

**Example**

▶ 일반화 복사 생성자

```c++
template <typename T>
class SmartPtr {
public:
	SmartPtr(T* ptr) :_Tptr(ptr) { std::cout << "create 1" << std::endl; }; 
    //일반화 복사 생성자
	template <typename U>
	SmartPtr(const SmartPtr<U> &Uptr) : _Tptr(Uptr.get()) { std::cout << "create 2" << std::endl; }
    
	//U->T 포인터
	T* get() const { return _Tptr;}
    
    //operator -> 가능
    T* operator->() {
		return _Tptr;
	} 
private:                               
	T* _Tptr;                            //포인터
};

SmartPtr<Base> sp1 = SmartPtr<Base>(new Base);

SmartPtr<Base> sp2 = SmartPtr<Derived>(new Derived);

//SmartPtr<Derived> sp3 = SmartPtr<Base>(new Base); //Base -> Derived 로 변경하지 못합니다.
```

결과창

```c++
create 1 //Base -> Base

create 1  //Derived -> Base
create 2 
```



## <span style="color:#9282CD"> 6.  타입 변환이 바람직할 경우 비멤버 함수를 클래스 템플릿 안에 정의해 두자</span>

- 모든 매개변수에 암시적인 타입 변환을 지원하는 템플릿과 관계가 있는 함수를 제공하는 클래스 템플릿은 이러한 함수는 클래스 템플릿 안에 프렌드 함수로서 정의하자.

**Example**

▶ freind 함수

```c++
template<typename T>
class Rational {
public:
	Rational(const T& num1 = 0,const T& num2 = 1) : _num1(num1), _num2(num2) {}

    //템플릿 안에 friend 함수 정의
	friend const Rational operator*(const Rational& lhs, const Rational& rhs) {
		return Rational(lhs._num1 * rhs._num1, lhs._num2 * rhs._num2);
	} 
private:
	T _num1;
	T _num2;
};

Rational<int> lhs(3, 2);
Rational<int> result = 2 * lhs;
```

결과창

```c++
6 2
```



## <span style="color:#9282CD"> 7. 타입에 대한 정보가 필요하다면 특성정보 클래스를 사용하자</span>

- 특성정보 클래스는 컴파일 도중에 사용할 수 있는 타입 관련 정보를 만들어 낸다.
- 특수정보 클래스는 템플릿 및 템플릿 특수 버전을 사용하여 구현한다.
- 함수 오버로딩 기법과 결합하여 특성정보 클래스를 사용하면 컴파일 타임에 결정되는 타입별 if..else 점검문을 구사할 수 있다.

**Example**

▶ 특수정보 템플릿

```c++
template <typename IterT, typename DistT>                // 임의 접근 반복자
void doAdvance(IterT& iter, DistT d,                  
	std::random_access_iterator_tag)       
{
	iter += d;
}

template <typename IterT, typename DistT>                // 양방향 반복자
void doAdvance(IterT& iter, DistT d,                    
	std::bidirectional_iterator_tag)     
{
	if (d >= 0) { while (d--) ++iter; }
	else { while (d++) --iter; }
}

template <typename IterT, typename DistT>                // 입력 반복자
void doAdvance(IterT& iter, DistT d,                  
	std::input_iterator_tag)
{
	if (d < 0) {
		throw std::out_of_range("Negative distance");
	}
	while (d--) ++iter;
}

// 오버로딩된 doAdvance를 호출
template <typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
	doAdvance(iter, d,                                            
		typename                                           
		std::iterator_traits<IterT>::iterator_category()// 오버로딩 호출
	);                                                       
}

```



## <span style="color:#9282CD"> 8. 템플릿 메타 프로그래밍</span>

- TMP는 기존 작업을 런타임에서 컴파일 타임으로 전환하는 효과를 낸다.

**Example**

▶ TMP

```c++
template<unsigned n>
struct Factorial { //n * Factorial<n - 1>의 value * ...
	enum { value = n * Factorial<n - 1>::value };
};//나열자 둔갑술을 통한 value 변수 선언

template<>
struct Factorial<1> { //올바른 종료를 위한 특수화
	enum { value = 1 };
};
```

결과창

```c++
120
3628800
//런타임 계산 없이 출력.
```

