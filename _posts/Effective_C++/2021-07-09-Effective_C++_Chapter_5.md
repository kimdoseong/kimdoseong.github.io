---
title:  "Effective_C++_Chapter 5"
excerpt: "구현"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]


date: 2021-07-09
last_modified_at: 2021-07-09

published: true
---

## <span style="color:#8F7CEE">1. 변수 정의는 늦출 수 있는 데까지 늦추는 근성을 발휘하자 </span>

- 변수 정의는 늦출 수 있을 때까지 늦추자. 프로그램이 더 깔끔해지고 효율도 좋아진다.

**Example**

▶최대한 늦추는 변수 정의

```c++
string changeString(string num1,string num2){
    ...//여러가지 계산을 합니다.
    string returnString(result); //변수가 정말 필요해질 때 정의함과 동시에 초기화 합니다.
    					  //복사 생성자가 사용됩니다.
}
```

변수를 일찍 선언하게 된다면 함수에서 변수를 사용하기도 전에 return 되거나, exception으로 함수가 종료될 수도 있는데 이러한 경우 변수를 선언하는 비용은 들었지만 사용은 하지 못하게 된다. 따라서 최대한 변수 정의를 늦춰  효율성을 높인다.   

## <span style="color:#8F7CEE">2. 캐스팅은 절약, 또 절약! 잊지 말자 </span>

- 다른 방법이 가능하다면 캐스팅은 피하자. 
- 캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길 수 있도록 해 보세요. 최소한 사용자는 자신의 코드에 캐스팅을 넣지 않고 함수를 호출할 수 있게 됩니다.
- 구형 스타일 캐스트를 쓰지 말고 c++ 스타일 캐스트를 선호하자. 발견하기 쉽고 어떤 역할을 의도했는지 더 자세히 드러난다.

const_cast: 객체의 상수성을 없애는 용도.  

dynamic_cast: 안전한 다운 캐스팅을 할 수 있게 해 준다.  

reinterpret_cast: 포인터를 int로 바꾸는 등의 하부 수준 캐스팅을 위해 만들어진 연산자.  

static_cast: 암시적 변환을 강제로 진행할 때 사용.  

**Example**

▶dynamic_cast를 쓰지 않는 방법으로 변경

```c++
//dynamic_cast Example
typedef vector<shared_ptr<Base>> VPB;
VPB basePtr;

for (VPB::iterator iter = basePtr.begin(); iter != basePtr.end(); iter++) {
	Derived* pdr = dynamic_cast<Derived*>(iter->get());
	//dynamic_cast로 iter를 다운 캐스팅 했습니다.(Base -> Derived)
}

//unused dynamic_cast Example
typedef vector<shared_ptr<Derived> > VPD;
VPD derivedPtr;

for (VPD::iterator iter = derivedPtr.begin(); iter != derivedPtr.end(); iter++) {
	Derived* pdr = iter->get();
	//dynamic_cast를 쓰지말고 typedef를 하나 더 만드는 편이 안정적입니다.
}
```

## <span style="color:#8F7CEE">3. 내부에서 사용하는 객체에 대한 '핸들'을 반환하는 코드는 되도록 피하자 </span>

- 어떤 객체의 내부 요소에 대한 핸들(참조자, 포인터, 반복자)을 반환하는 것은 되도록 피하자.

-Chapter 4의 4 항목과 비슷합니다.  

**Example**

▶반환에 '핸들'을 사용한 좋지 않은 예제  

```c++
class Rectangle{
    public:
    ...
    const Point& upperLeft() const { return pData->left; }
    //참조를 반환합니다.. const 키워드로 쓰는것은 제한됩니다.
    //const도 없다면 캡슐화 정도가 낮아집니다.
}
```

그나마 const 키워드를 사용하여 멤버 데이터가 변경 되는 것은  제한하였습니다. 그러나 "무효참조 핸들"이 발생할 수 있습니다.  만약 참조 받는 객체가 임시 객체라면 그 임시 객체를 가리키는 포인터는 주소값만 남기고 모두 사라져 버립니다. 이러한 이유로 객체 내부에 대한 핸들을 반환하는 것은 위험합니다.  

## <span style="color:#8F7CEE">4. 예외 안전성이 확보되는 그 날 위해 싸우고 또 싸우자! </span>

예외 안정성을 가지기 위한 함수  

- 자원이 새지 않도록 한다.
- 자료구조가 더럽혀지는 것을 허용하지 않습니다.

아래 세가지 보장 중 하나를 제공하자.  

- 기본적인 보장: 함수 동작 중에 예외가 발생하면, 실행 중인 프로그램에 관련된 모든 것들을 유효한 상태로 유지.
- 강력한 보장: 함수 동작 중에 예외가 발생하면, 프로그램 상태를  절대로 변경하지 않겠다는 보장(예외 발생하면 함수 호출이 없었던 것처럼 프로그램의 상태가 되돌아간다.)
- 예외 불가 보장: 예외를 절대로 던지지 않겠다는 보장. 

## <span style="color:#8F7CEE">5. 인라인 함수는 미주알고주알 따져서 이해해 두자 </span>

- 함수 인라인은 작고, 자주 호출되는 함수에 대해서만 하는 것으로 묶어두자. 이렇게 하면 디버깅 및 업그레이드가 용이, 자칫 생길 수 있는 코드 부풀림 현상이 최소화되며, 프로그램의 속력이 더 빨라질 수 있는 여지가 많아진다.
- 함수 템플릿과 인라인 함수는 대체적으로 헤더 파일에 들어가지만 템플릿을 모두 인라인 함수라고 생각하면 안된다.

virtual 함수: "어떤 함수를 호출 할지 결정하는 작업을 실행 중에 한다."  

inline 함수: " 함수 호출 위치에 호출된 함수를 끼워넣는 작업을 프로그램 실행 전에 한다."  

이처럼 virtual 함수가 당연히 inline 함수로 사용하면 안되는 것처럼 루프가 들어가 있거나, 재귀 함수인 경우 등 복잡한 함수는 inline 함수로 사용하면 안됩니다.(생성자, 소멸자도 간단한 함수 같지만 전혀 그렇지 않습니다.)   

## <span style="color:#8F7CEE">6. 파일 사이의 컴파일 의존성을 최대로 줄이자 </span>

- 컴파일 의존성을 최소화 하자. '정의' 대신 '선언' 에 의존하게 하자.
- 핸들 클래스는 동적 메모리 할당에 따르는 연산 오버헤드와 bad_alloc(메모리 고갈) 예외가 발생할 수 있다.
- 인터페이스 클래스는 함수 호출 마다 가상 테이블 점프에 따르는 비용이 소모된다.
- 하지만 사용자에게 미칠 효과를 최소화로 하기 위해 위 방법을 사용하며, 프로그램 실행 속력, 파일 크기에서 손해가 많다면 그 때 바꾸면 된다.

class선언과 include의 차이  

```c++
//class 선언
class A;
class B{
    A *pA; //가능
    A a; //불가능 - include 사용해야 합니다.
} //A 클래스의 포인터만 이용할때 전방선언이 가능해 진다.
  //포인터는 4byte or 8byte든 컴파일러가 할당할 수 있지만 
  //A a는 객체 하나의 크기가 얼마인지는 알 수 없다.
```

**Example**

▶ 1. 핸들 클래스 사용

```c++
/*Person.h File*/ 
class PersonImpl; //구현 클래스

class Person { //선언(핸들) 클래스 별 다른 기능을 구현하지 않습니다.
private:
	std::shared_ptr<PersonImpl> pImpl; //구현 클래스에 대한 포인터를 가집니다.
public:
	Person(const std::string name, const int age);
	void printPerson();
};

/*Person.cpp File*/
#include "Person.h"
#include "PersonImpl.h" //include와 class선언의 차이 참조.

Person::Person(const std::string name, int age) 
	: pImpl(new PersonImpl(name, age)){} //포인터로 객체를 생성합니다.(include 사용)

void Person::printPerson() {
	pImpl->printPerson();//함수를 호출하기만 합니다.
}

```

```c++
/*PersonImpl.h File*/
class PersonImpl { //Person 구현 클래스
private:
	std::string _name;
	int _age;
public:
	PersonImpl(std::string, int);
	void printPerson();
};

/*PersonImpl.cpp File*/
#include "PersonImpl.h"
#include <iostream>
//기능 구현
PersonImpl::PersonImpl(std::string name, int age)
    :_name(name), _age(age){}

void PersonImpl::printPerson() const{
	std::cout << "name: " << _name << " " <<
		"age: " << _age << std::endl;
}
```

```c++
#include "Person.h" //Person.h만 include하여 사용합니다.

int main() {
	Person ps("kds", 28); //선언부로 사용합니다.
	ps.printPerson();
}
```

▶ 2. 인터페이스 클래스 사용

```c++
/*Person.h File*/ 
class Person { //인터페이스 클래스 (구현하지 않습니다)
//파생이 목적이기 때문에 데이터 멤버, 생성자가 없습니다.
//하나의 가상 소멸자와 순수 가상 함수만 들어갑니다.
public:
	virtual ~Person() {} //{}를 꼭 붙여주세요.
	virtual void printPerson() const =0;

	static std::tr1::shared_ptr<Person> 
        create(std::string name, int age);//객체 생성 수단이 있어야 한다.
    //주어진 매개 변수로 Person 객체를 반환합니다.
    //(팩토리 함수, 가상 생성자라 불린다)
    //사용자가 원하는 대로 다른 타입의 파생 클래스 객체를 생성할 수 있게 만들면 된다.
};.
    
/*Person.cpp File*/
#include "Person.h"
#include "PersonImpl.h"

std::tr1::shared_ptr<Person> Person::create(const std::string name, int age) {
	return std::tr1::shared_ptr<Person>(new PersonImpl(name, age));
}//객체 생성
```

```c++
/*PersonImpl.h File*/
#include "Person.h"

class PersonImpl :public Person { //Person 구현 클래스
private:
	std::string _name;
	int _age;
public:
	PersonImpl(std::string name, int age);
	~PersonImpl() {};
	void printPerson() const;
};

/*PersonImpl.cpp File*/
//기능 구현
PersonImpl::PersonImpl(std::string name, int age)
    :_name(name), _age(age){}

void PersonImpl::printPerson() const{
	std::cout << "name: " << _name << " " <<
		"age: " << _age << std::endl;
}
```

```c++
#include "Person.h" //Person.h만 include하여 사용합니다.

int main() {
	std::tr1::shared_ptr<Person> ps = Person::create("kds", 28);
    //인터페이스 클래스 객체로 사용합니다.
	ps->printPerson();
}
```

