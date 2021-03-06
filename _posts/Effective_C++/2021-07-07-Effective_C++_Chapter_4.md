---
title:  "Effective_C++_Chapter 4"
excerpt: "설계 및 선언"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]

 
date: 2021-07-07
last_modified_at: 2021-07-07

published: true
---

## <span style="color:#8F7CEE">1. 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 하자. </span>

- 좋은 인터페이스는 제대로 쓰기엔 쉬우며 엉터리로 쓰기에는 어렵다. 인터페이스를 만들 때 이 특성을 지닐 수 있도록 하자.  
- 새로운 타입 만들기, 타입에 대한 연산을 제한, 객체의 값에 대한 제약, 자원 관리  작업을 사용자 책임으로 놓지 않기  

**Example**

▶엉터리로 쓰기 어렵게 하자  

```c++
class Date {
public:
    Date(int month, int day, int year){}

private:
    int month;
    int day;
    int year;
};

int main() {
	Date(30, 1, 2021); //month와 day를 바꾸어 써버렸네요..
}
```

▶ 타입을 만들어 주면 엉터리로 쓰기 어려워 집니다.  

```c++	
struct Day { //Day, Month, Year 타입을 각각 만들고
    int val;
    explicit Day(int d): val(d) {}
};

struct Month {
    int val;
    explicit Month(int m): val(m){}
};

struct Year {
    int val;
    explicit Year(int y): val(y){}
};

class Date {
private:
    Month _month;
    Day _day;
    Year _year;

public:
    Date(const Month& month, const Day& day, const Year& year)
        :_month(month), _day(day), _year(year){} //타입에 해당하는 값을 넣어줍니다.
};

int main() {
	Date d(Month(1), Day(30), Year(2021)); //이렇게 써야하니 엉터리로 쓰기가 힘들겠죠?
	return 0;
}
```



## <span style="color:#8F7CEE">2. 클래스 설계는 타입 설계와 똑같이 취급하자 </span>

- 새로 정의한 타입의 객체 생성 및 소멸은 어떻게 이루어 져야 할까?  

  생성자, 소멸자, 메모리 할당을 어떻게 설계 했는가?  

- 객체 초기화는 객체 대입과 어떻게 달라야 하는가?  

  초기화와 대입을 헷갈리지 말자.  

- 새로운 타입으로 만든 객체가 값에 의한 전달되는 경우에 어떤 의미를 줄 것인가?  

  복사 생성자는 참조자를 사용해야 하며, 사용하지 않으면 무한 루프에 빠지게 된다.  

- 새로운 타입이 가질 수 있는 적법한 값에 대한 제약으로 무엇으로 잡을 것인가?  

  예외 처리  

- 기존의 클래스 상속 계통망에 맞출 것인가?  

  virtual 함수 사용  

- 어떤 종류의 타입 변환을 허용할 것인가?  

  명시적 변환, 암시적 변환  

- 어떤 연산자와 함수를 두어야 의미가 있을까?  

- 표준 함수들 중 어떤 것을 허용하지 말 것인가?  

  private으로 선언한 함수  

- 새로운 타입의 멤버에  대한 접근 권한을 어느 쪽에 줄 것인가?    

  멤버 접근 권한 설정. (private, public, protected)  

- 선언되지 않은 인터페이스로 무엇을 둘 것인가?    

- 새로 만드는 타입이 얼마나 일반적인가?  

  템플릿 정의  

- 정말로 꼭 필요한 타입인가?  

  기능 몇 개가 아쉬워 파생 클래스를 만드느니, 비멤버 함수 or 템플릿을 몇 개 더 정의하는 것이 더 낫다.  

## <span style="color:#8F7CEE">3. '값에 의한 전달' 보다는 '상수 객체 참조자에 의한 전달' 방식을 택하는 편이 대개 낫다 </span>

- '값에 의한 전달' 보다는 '상수 객체 참조자에 의한 전달' 을 선호하자. 대체적으로 효율적이고 복사 손실 문제까지 막아 준다.  

**Example**  

▶ call by reference  

```c++
void print(const Derived& a) { //참조자를 사용합니다.
	...
} //별 다른 생성자를 호출하지 않습니다.
```

▶ call by value 결과 창 (참조자 사용하지 않은 경우)  

```c++
void print(const Derived a) //참조자가 없습니다.

Base 복사 생성자 //값을 복사 해야 하니 
Derived 복사 생성자//이렇게 복사 생성자에
Derived 소멸자 //소멸자 까지 호출하게 됩니다.
Base 소멸자 // 물론 기반 클래스 복사 생성자&소멸자도 호출 됩니다.
```

'복사 손실 문제란' call by value 를 사용하면 데이터 손실이 생길 수 있다.  

▶ call by value 데이터 손실  

```c++
Derived A; //파생 클래스 객체인데

void print(const Base a) { //기반 클래스 복사 생성자만 호출 합니다. 
...   // 파생 클래스 복사 생성자는 호출하지 않아 데이터가 손실됩니다.
}

Base 복사 생성자 //Derived 클래스 데이터가 손실됩니다.
Base 소멸자
```

파생 클래스 객체 임에도 기반 클래스 복사 생성자만 호출되며, 파생 클래스 복사 생성자는 호출되지 않아 데이터가 의도치 않게 손실되게 됩니다.  

## <span style="color:#8F7CEE">4. 함수에서 객체를 반환해야 할 경우 참조자를 반환하려고 들지 말자  </span>

- 지역 스택 객체에 대한 포인터나 참조자를 반환 금지
- 힙에 할당된 객체 참조자 반환 금지
- 올바른 지역 정적 객체에 대한 참조자 반환은 Chapter 1 의 Singleton 예제 참조.  

**Example**  

▶올바르지 않은 참조반환 예제  

```c++
    const Derived& operator*(const Derived& rhs) {
        //첫번째 예제
        Derived *result = new Derived(this->_num * rhs._num);
        return *result; //result 라는 new로 생성한 객체를 반환하네요.. delete 처리가 어렵습니다.
		
        //두번째 예제
        Derived result(this->_num * rhs._num);
        return result; //지역 객체를 참조자로 반환한다니..절대 안됩니다.
                       //result는 함수가 끝나면 소멸하는 객체입니다. 데이터를 정상적으로 반환하지 못합니다.
    }
```

▶올바른 반환 예제  

```c++
    inline const Derived operator*(const Derived& rhs) { //더이상 참조 반환이 아닙니다. 
        return Derived(this->_num * rhs._num); //임시 객체는 생성하지만 위의 예제와 달리 올바른 동작을
    } //하게 됩니다. (생성자와 소멸자 비용이 들지만 말이죠)
```



## <span style="color:#8F7CEE">5. 데이터 멤버가 선언될 곳은 private 영역임을 명심하자 </span>

- 데이터 멤버 변수는  private로 선언하자  

  1. 멤버에 접근하고 싶을 때 괄호를 붙여야 할지 말지 하는 문법적 일관성을 지킬 수 있다.  

  2. 캡슐화로 클래스의 불변속성을 강화한다.  

  3. 클래스에 public 데이터 멤버를 하나 제거한다면 수많은 코드를 수정해야 한다.

     이 점에서는 protected도 마찬가지(데이터 멤버에 직접 접근할 수 있기 때문에) 이므로

     protected와 public 데이터 멤버는 큰 차이가 없다.   

## <span style="color:#8F7CEE">6. 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지자 </span>

- 멤버 함수 보다는 비멤버 비프렌드 함수를 자주 쓰자.  

  1. 캡슐화 정도가 높아진다. (어떤 데이터를 접근하는 함수가 많으면 그 데이터의 캡슐화 정도가 낮다)  

     1-1. 비멤버 비프렌드 함수는 private 데이터 멤버에 접근하는 함수 수를 늘리지 않는다.   

  2. 패키징 유연성이 커진다.  

     2-1. 반드시 사용해야 하는 핵심 기능들은 멤버 함수에 두어 하나의 헤더 파일로 선언하고, 나머지 부가 기능들은 같은
          namespace 내의 다른 헤더에 기능 별로 분류하여 비멤버 비프렌드 함수로 선언해 두면 사용자는
          필수 헤더 + 사용하고자 하는 기능이 포함된 헤더만 include하여 라이브러리를 사용할 수 있다.  

  3. 기능적인 확장성이 늘어난다.  

     3-1. 기능 확장을 원하는 클래스와 동일한 namespace에 편의를 위한 비멤버 비프렌드 함수를 추가로 정의해 넣을 수 있다.

**Example**

▶패키지 유연성과 기능적 확장성

```c++
"Number.h" file
namespace NumberStuff { //NumberStuff namespace 안에 
    class number { //number에 관련된 핵심 기능들 선언
    private:
        int _num;
    public:
        number() : _num() {}
        ~number() {}
        number(int num) : _num(num) {}

        int getNum() {
            return this->_num;
        }
    }; //거의 모든 사용자가 사용해야 하는 비멤버 함수들도 들어갑니다. 
    ...
}
```

```c++
"NumCalculation.h" file //다른 헤더 파일입니다.
 namespace NumberStuff { //마찬가지로 NumberStuff namespace 안에 정의합니다.
	int plusNumber(number& lhs, number& rhs) {
		int result = 0; 
		result = lhs.getNum() + rhs.getNum();
		return result;
	} //number에 관련된 계산 기능들이 들어갑니다.
    ...
}
```

```c++
"NumPrint.h" file //다른 헤더 파일입니다.
namespace NumberStuff { //위와 동일 합니다.
	void printNumber(number& lhs, number& rhs) {
		cout << "lhs num :" << lhs.getNum() << endl;
		cout << "rhs num :" << rhs.getNum() << endl;
	} //number에 관련된 출력 기능들이 들어갑니다.
    ...
}
```

```c++
#include <iostream>
#include "Test.h"
#include "Number.h" //핵심기능 헤더는 가져오고
#include "NumCalculation.h" //필요한 비멤버 비프렌드 함수가 정의된 헤더 파일을 가져오면 됩니다. 
//#include "NumPrint.h" //출력 기능은 사용하지 않으면 "NumPrint.h" 파일을 가져오지 않으면 됩니다.
//패키징 유연성과 기능적 확장성이 늘어난다.

using namespace std;
using namespace numberStuff;

int main() {
	number a(2);
	number b(3);
	
	//printNumber(a, b);
	cout << plusNumber(a, b) << endl;
}
```



## <span style="color:#8F7CEE">7. 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자  </span>

- 어떤 함수에 들어가는 모든 매개변수(this도 포함)에 대해 타입 변환을 해줄 필요가 있으면 그 함수는 비멤버 여야 한다.  

**Example**

▶멤버 함수로 만든 잘못된 예시  

```c++
 const number operator*(const number& lhs) {
     return number(this->_num * lhs._num);
 }//멤버 함수로 만든다면

number result;
result = a * 2; //컴파일 됩니다.
//a.operator(number(2)) // 암시적 변환이 됩니다.

number result;
result = 2 * a; //Error가 발생합니다. 
//2.operator(a)로 말이 안됩니다. 2가 객체가 아닙니다.
```

▶비멤버 함수로 만든 예시  

```c++
const number operator*(const number& lhs, const number& rhs) {
	return number(lhs.getNum() * rhs.getNum());
}//이렇게 비 멤버 함수로 만들어 줍니다.

number result;
result = 2 * a; //이제 잘 됩니다.
//number(2).operator(a)로 잘 됩니다.
```

