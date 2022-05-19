---
title:  "Effective_Modern_C++_Chapter 3_2"
excerpt: "현대적 C++에 적응하기_2"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2022-04-21
last_modified_at: 2022-05-08

published: true
---

## <span style="color:#8F7CEE">  1. iterator보다 const_iterator를 선호하라 </span>

iterator가 가리키는 것을 전혀 수정하지 않을 것이라면 const_iterator를 사용 해야한다.

c++11 표준에 없는 cbegin, cend, rend, crbegin, crend 추가 방법
```c++
template <class C>
auto cbegin(const C& container)->decltype(std::begin(container))
{
    return std::begin(container);
}
```

begin 으로 무슨 const iterator 를 반환하나 싶겠지만  
멤버변수 container를 통해서 자료구조에 접근 하므로 그 container에 대한 const 버전에 대한 참조가 된다.  
따라서 begin 함수를 호출하면 const_iterator 형식의 반복자가 반환된다.

*기억해 둘 사항들*
- iterator 보다 const_iterator를 선호하라.
- 최대한 일반적인 코드에서는 begin, end, rbegin 등의 비멤버 버전들을 해당 멤버 함수들보다 선호하라.

## <span style="color:#8F7CEE">  2. 예외를 방출하지 않을 함수는 noexcept로 선언하라 </span>

함수가 어떠한 예외도 발생시키지 않는다면 noexcept 키워드를 통해 명시할 수 있다.

noexcept 키워드는 1. 예외를 던지지 않는다(noexcept) 와 2. 예외를 던진다(noexcept(false)) 의
이분법적 논리이다.  

```c++

void foo() noexcept; // 최적화 여지가 가장 크다

void foo() throw(); // 최적화 여지가 더 작다

void foo();         // 최적화 여지가 더 작다.
```

단순히 인터페이스 명세만을 위한 것이 아니라 컴파일러는 noexcept 키워드가 붙은 함수는  

예외를 발생시키지 않는다고 판단하고 컴파일한다. (예외 처리 하지 않고 프로그램 종료)  


- throw 와의 차이  

throw는 std::unexcepted를 호출하며, noexcept는 std::terminated를 호출한다.  

throw와 달리 noexcept는 stack uwinding이 발생할 수도 있고 발생하지 않을수도 있다.  

별 것 아닌것 같지만 throw는 최적화의 유연성이 없다는 것을 의미하며, 컴파일러 코드 작성에 큰 영향을 끼친다.  


- move semantics

"가능하면 이동하되 필요하면 복사한다" 를 사용한다  

주어진 연산이 noexcept로 선언되어 있지 않으면 copy semantics로 처리한다.  


*기억해 둘 사항들*
- noexcept는 함수의 인터페이스의 일부이다. 이는 호출자가 noexcept 여부에 의존할 수 있음을 뜻한다.  
- noexcept 함수는 비 noexcept 함수보다 최적화의 여지가 크다.  
- noexcept는 이동 연산들과 swap, 메모리 해제 함수들, 그리고 특히 소멸자들에 특히나 유용하다.
- 대부분의 함수는 noexcept가 아니라 예외에 중립적이다.  

## <span style="color:#8F7CEE">  3. 가능하면 항상 constexpr을 사용하라 </span>

constexpr 은 어떠한 값이 단지 **상수** 일 뿐만 아니라 컴파일 시점에서 알려진다는 점을 나타낸다.

```c++

int size; // 비 constexpr 변수

...

constexpr auto arraySize1 = size; // Error size의 값이 컴파일 도중에 알려지지 않음.
std::array<int, size> data1; // Error 위와 같은 사유

constexpr auto arraySize2 = 10; // Ok, 10은 컴파일 시점의 상수.
std::array<int, arraySize2> data2; // Ok arraySize 는 constexpr 변수로 컴파일 시점의 상수이다.

```

생성자 혹은 함수에서 사용

```c++

class Point {
    public:
    constexpr Point(double a) noexcept:{}
}

...

constexpr Point(1.1) // Ok constexpr 생성자가 컴파일 시점에서 실행됨

```

*기억해 둘 사항들*
- constexpr 객체는 const 이며, 컴파일 도중에 알려지는 값들로 초기화 된다.
- constexpr 함수는 그 값이 컴파일 도중에 알려지는 인수들로 호출하는 경우에는 컴파일 시점 결과를 산출한다.
- constexpr 객체나 함수는 비 constexpr 객체나 함수보다 광범위한 문맥에서 사용할 수 있다.
- constexpr은 객체나 함수의 인터페이스의 일부이다.

## <span style="color:#8F7CEE">  4. const 멤버 함수를 스레드에 안전하게 작성하라 </span>

멤버 함수에서 멤버 변수의 값을 수정하지 않을 것이라면 const로 선언하는 것이 자연스럽다.

아래와 같이 개념적으로는 어떠한 값도 변경하지 않지만 필요한 경우에만 계산하여 반환하는 함수는 변수를 "mutable" 로 선언하고 합당한 코드이다. 

 ```c++

class Point {
    public:
        int Pos() const {
            if(isValid){
                ...
                isValid = true;
            }

            return pos;
        }

    private:
        mutable bool isValid{false};
        mutable int pos;
}

```

하지만 const 키워드는 읽기 전용이라는 생각으로 큰 생각없이 여러 스레드로 사용하면 data race를 발생시킨다.  
(같은 메모리를 동기화 없이 읽고 쓰려고 하므로)  

근본적인 문제는 const 함수이지만 스레드가 안전하지 않다는 점이다.  

이에 가장 쉬운 해결 방법은 std::mutex 이다.

 ```c++

class Point {
    public:
        int Pos() const {
            std::lock_guard<std::mutex> g(m) // Add lock_gurad
            if(isValid){
                ...
                isValid = true;
            }

            return pos;
        }

    private:
        mutable std::mutex m;
        mutable bool isValid{false};
        mutable int pos;
}

```


counting 처럼 단순한 연산을 하는 경우는 mutex 가 과한 일일 때도 있다.  

그럴땐 std::atomic 을 사용하여 해결하면 더 간단하게 해결할 수 있다.

 ```c++

class Point {
    public:
        int Pos() const {
            ++;callCount; // 원자적 증가
            return pos;
        }

    private:
        mutable std::atomic<int> callCount{0}; // atomic 을 사용
        mutable bool isValid{false};
        mutable int pos;
}

```

하지만 아래와 같은 경우는 위험을 초래한다.

 ```c++

class Point {
    public:
        int Pos() const {
            if(isValid) return pos;
            else{
                auto val1 = expensiveComputation1();
                auto val2 = expensiveComputation2();

#ifdef CASE1          
                pos = val1 + val2;
                isValid = true;
#else // CASE2
                isValid = true;
                pos = val1 + val2;
#endif
                return pos;
            }
        }

    private:
        mutable std::atomic<int> isValid{false}; // atomic 을 사용
        mutable std::atomic<int> pos;
}

```

**CASE1 - performance 저하**
- 한 스레드가 Point::Pos()를 호출해서 isValid 가 false 라고 관측하고 비용이 큰 계산을 수행한 후 둘의 합을 pos 에 배정한다.
- 그 시점에서 둘째 스레드가 Point::Pos()를 호출해서 isValid를 점검한다. 역시 false 라고 관측한 스레드는 방금 마친것과 동일한 비싼 연산을 수행하고 같은 값을 pos에 배정한다.

**CASE2 - 올바르지 않은 값 반환**
- 한 스레드가 Point::Pos()를 호출해서 isValid 가 true로 설정되는 지점까지 나아간다.
- 그 시점에서 둘째 스레드가 Point::Pos()를 호출해서 isValid를 점검한다. 그것이 true임을 관측한 둘째 스레드는, 첫 스레드가 isValid에 값을 배정하기도 전에 pos 값을 반환한다. 따라서 그 반환값은 정확하지 않다.

위와 같은 경우는 앞서 본 std::mutex 를 사용하여 동기화 하는 것이 바람직하다.

*기억해 둘 사항들*
- 동시성 문맥에서 쓰이지 않을 것이 확실한 경우가 아니라면, const 멤버 함수는 스레드에 안전하게 작성하라.
- std::atomic 변수는 mutex에 비해 성능상의 이점이 있지만, 하나의 변수 또는 메모리 장소를 다룰 때에만 적합하다.

## <span style="color:#8F7CEE">  5. 특수 멤버 함수들의 자동 작성 조건을 숙지하라 </span>

C++의 공식적인 어법에서 특수 멤버 함수(special member function)들은 C++이 스스로 작성하는 멤버 함수들을 가리킨다.

C++98 - 기본 생성자, 소멸자, 복사 생성자, 복사 대입 연산자  

추가 + C++11 - 이동 생성자, 이동 대입 연산자

```c++

class Base{
    public:
    Base(Base&& rhs); // 이동 생성자
    Base&& operator=(Base&& rhs); // 이동 대입 연산자
}

```

3의 법칙(Rule of Three) - 만일 복사 생성자와 복사 대입 연산자, 소멸자 중 하나라도 선언 했다면 나머지 둘도(즉 셋 모두) 선언해야 한다.

1. 한 복사 연산이 수행하는 자원 관리를 다른 복사 연산에서도 수행해야 한다
2. 클래스의 소멸자 역시 그 자원의 관리에 참여한다 (보통 자원의 해제)

따라서 셋 중 하나라도 선언을 하면 나머지 둘 모두 선언해 주어야 한다.  

```c++
class Base{
    public:
    ~Base(); // 사용자 선언 소멸자

    Base(const Base&) = default; // 기본 복사 생성자 사용 선언
    Base& operator=(const Base&) = default; // 기본 복사 연산자 사용 선언

    // 혹은 사용자 선언 복사 연산자들 선언
}

```

기반 클래스는 가상 소멸자를 선언해 주어야 하므로 복사 연산자들도 따라서 선언해 주어야 한다.  

복사 연산자들을 선언하면 이동 연산자들은 자동으로 삭제되므로 만약 기반 클래스 혹은 파생 클래스에서 이동 연산자를 사용할 것이라면 이동 연산자들도 선언해주면 된다.

```c++
class Base{
    public:
    virtual ~Base() = default; // 기반 클래스로 virtual 사용자 선언 소멸자

// 소멸자 선언에 따른 복사 연산자들 선언
    Base(const Base&) = default; // 기본 복사 생성자 사용 선언
    Base& operator=(const Base&) = default; // 기본 복사 연산자 사용 선언

// 자동으로 삭제된 이동 연산자들을 지원하기 위한 선언
    Base(Base&& rhs) = default; // 기본 이동 생성자 사용 선언
    Base&& operator=(Base&& rhs) = default; // 기본 이동 대입 연산자 사용 선언
}

```

특수 멤버 함수 규칙

- 기본 생성자 - 클래스에 사용자 선언 생성자가 없는 경우에만 자동으로 작성된다.
- 소멸자 - 기본적으로 noexcept 이다.(C++11 이상) 기본적으로 작성되는 소멸자는 오직 기반 클래스 소멸자가 가상일 때에만 가상이다.
- 복사 생성자
    - 클래스에 사용자 선언 복사 생성자가 없을 때에만 자동으로 작성된다  
    - 클래스에 이동 연산이 하나라도 선언되어 있으면 삭제(비활성화) 된다.
    - 사용자 선언 복사 대입 연산자나 소멸자가 있는 클래스에서 이 함수가 자동 작성되는 기능은 비권장이다.
- 복사 대입 연산자
    - 클래스에 사용자 선언 복사 대입 생성자가 없을 때에만 자동으로 작성된다  
    - 클래스에 이동 연산이 하나라도 선언되어 있으면 삭제(비활성화) 된다.
    - 사용자 선언 복사 생성자나 소멸자가 있는 클래스에서 이 함수가 자동 작성되는 기능은 비권장이다.
- 이동 생성자와 이동 대입 연산자 - 클래스에 사용자 선언 복사 연산들과 이동 연산들, 소멸자가 없을 때에만 자동으로 작성된다.

멤버 함수 템플릿이 존재하면 특수 멤버 함수의 작성이 비활성되는 규칙은 없다.

 *기억해 둘 사항들*
- 컴파일러가 스스로 작성할 수 있는 멤버 함수들, 즉 기본 생성자와 소멸자, 복사 연산자들, 이동 연산자들을 가리켜 특수 멤버 함수라고 부른다.
- 이동 연산들은 이동 연산들이나 복사 연산들, 소멸자가 명시적으로 선언되어 있지 않은 클래스에 대해서만 자동으로 작성된다.
- 복사 연산자들(복사 생성자,복사 대입 연산자)은 해당하는 복사 연산자가 명시적으로 선언되어 있지 않은 클래스에 대해서만 자동으로 작성되며, 만일 이동 연산이 하나라도 선언되어 있으면 삭제된다. 소멸자가 명시적으로 선언된 클래스에서 복사 연산들이 자동으로 작성되는 기능은 비권장이다.
- 멤버 함수 템플릿 때문에 특수 멤버 함수의 자동 작성이 금지되는 경우는 전혀없다.
