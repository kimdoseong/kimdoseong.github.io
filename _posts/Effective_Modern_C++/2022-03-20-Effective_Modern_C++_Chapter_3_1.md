---
title:  "Effective_Modern_C++_Chapter 3_1"
excerpt: "현대적 C++에 적응하기_1"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2022-03-27
last_modified_at: 2022-04-21

published: true
---

## <span style="color:#8F7CEE">  1. 객체 생성시 괄호()와 중괄호{}를 구분하라 </span>
 
1-1. most vexing parse  

 ```c++
    Widget w1(10, true); //인수 10 과 true를 받는 생성자 호출
    Widget w2(); // !!!생성자 호출이 아닌 함수 선언!!!
    Widget w3{}; // 인수 없이 생성자 호출
 ```

중괄호 리스트를 사용하여 두번째 같은 경우를 피할 수 있다.

1-2. 중괄호 초기화 구문은 std::initializer_list를 강하게 선호한다.  

```c++
class Widget {
public:
    Widget(int i, double d);
    Widget(std::initializer_list<long double> il);
};

...
Widget w1(10, 5.0); // Widget(int i, double d) 호출
Widget w2{10, 5.0}; // Widget(std::initializer_list<long double> il) 호출
```

- 더 이상해지는 경우
 
```c++
class Widget {
public:
    Widget(int i, double d);
    Widget(std::initializer_list<bool> il); //bool 로 변경
};
...
Widget w1{10, 5.0}; // Widget(int i, double d) 생성자가 아닌 Widget(std::initializer_list<bool> il) 생성자를 호출하려고 함.  (error)
```

- 인수가 없는 경우  

표준에 따르면 빈 중괄호 쌍은 빈 std::initializer_list가 아닌 인수 없음을 뜻한다.  

```c++
class Widget {
public:
    Widget();
    Widget(std::initializer_list<int> il);
};
...
Widget w1; // 기본 생성자 호출
Widget w2{}; // 기본 생성자 호출

...
//인수 없는 std::initializer_list 호출 법

Widget w1({});
Widget {{}};
```

1-3. vector  
```c++
std::vector<int> v1(10, 20); //모든 요소 값이 20인 10개짜리 std::vector 생성

std::vector<int> v2{10, 20}; //10, 20 두 요소를 담은 vector 생성
```

*기억해 둘 사항들*
- 중괄호 초기화는 좁히기 변환을 방지하며, C++의 가장 성가신 구문 해석에서 자유롭다.
- 중괄호 초기화는 가능한 한 std::initializer_list 매개변수가 있는 생성자와 부합한다.
- 괄호와 중괄호의 선택이 의미 있는 차이를 만드는 예는 인수 두 개로 std::vector<type>을 생성하는 것이다.
- 템플릿 안에서 객체를 생성할 때 괄호와 중괄호 선택이 어려울 수 있다.

## <span style="color:#8F7CEE">  2. 0과 NULL보다 nullptr를 선호하라 </span>

*기억해 둘 사항들*
- 0과 NULL보다 nullptr 사용
- 정수 형식과 포인터 형식에 대한 중복적재를 피하라


## <span style="color:#8F7CEE">  3. typedef보다 별칭 선언을 선호하라 </span>

```c++
// 같은 의미
typedef void(*FP)(int, const std::string&);

using FP = void(*)(int, const std::string&);
```

둘의 가장 큰 차이는 템플릿화의 가능 유무이다.

```c++
// using - 간단함!
template <typename T>
using uVector = std::vector<T>;

// typedef - struct 안에 넣어줘야 한다.
template<typename T>
struct MyVector {
    typedef std::vector<T> tVector;

};
...

// using vector
    uVector<int> v;

// typedef vector
    MyVector<int>::tVector v;
```

중첩 의존 이름 참조  
```c++
template<typename T>
class A{
private:
    uVector<T> v;
};

template<typename T>
class B{
private:
    //typename 까지 붙여줘야함
    typename MyVector<T>::tVector v;
};
```

*기억해 둘 사항들*
- typedef는 템플릿화를 지원하지 않지만, 별칭 선언은 지원한다.
- 별칭 템플릿에서는 "::~" 접미어를 붙일 필요가 없다. 템플릿 안에서 typedef를 지칭할 떄에는  
"typename" 접두사를 붙여야 하는 경우가 많다.
- C++14는 C++11의 모든 형식 특질 변환에 대한 별칭 템플릿들을 제공한다.

## <span style="color:#8F7CEE">  4. 범위 없는 enum보다 범위 있는 enum을 선호하라 </span>
 
4-1. 일반적으로 중괄호 쌍 내부에 이름을 선언하면 그 이름은 해당 중괄호 쌍 내에서만 유효하다.  
하지만 enum 의 경우 그렇지 않다.  

 ```c++
    enum Color {black, white, red}; // 중괄호 내에 선언되어 있지만,
    auto white = false; // error - 위에 white 가 선언되어 있음.
 ```

이 경우 enum 을 공식적으로 unscoped(범위 없는) enum 이라고 부른다.  

하여 C++11의 새로운 열거형인 scoped(범위 있는) enum 에서는 그러한 누수가 발생하지 않게한다.
 ```c++
    //enum class
    enum class Color {black, white, red};
    auto white = false; // 이제 가능하다.
    
    /*
    Color c = white; // Error - white 가 없음
    */ 

    // 이렇게 써줘야함.
    Color c = Color::white;
 ```

4-2. enum class 의 형식이 훨씬 강력하게 적용된다.  

 범위 없는 enum의 열거자들은 암묵적으로 정수 형식으로 변환된다.

 ```c++
    // enum class
    enum class Color {black, white, red};

    auto c = Color::red;
    auto i = c + 1; // Error - enum class와 정수를 계산하려고함.
    
    //  enum
    enum Color {black, white, red};

    auto c = Color::red;
    auto i = c + 1; // enum 의 경우 계산을 해준다.
 ```

4-3. 전방 선언  

 enum class는 전방 선언이 가능하다.

 ```c++
enum Color; // Error  전방 선언 불가
enum class Color // OK
 ```

 enum 이 전방선언 되지 않는 이유  
```c++
enum Status {
    good = 0,
    failed = 1,
    indeterminate = 0xffffffff
}
```

enum 이 나타내는 값의 범위는 0 ~ 0xffffffff 까지 이다. 32비트 까지 표현하기 위해  
메모리 효율을 위해 컴파일러들은 enum의 열거자 값들의 범위를 표현할 수 있는 가장 작은 **바탕 형식**으로  
선택하는 경향이 있으나 경우에 따라 크기 대신 속도를 위한 최적화를 적용하여 가장 작은 바탕 형식을  
선택하지 않을 수도 있다.  

따라서 바탕형식이 어떤것일지 명확하지 않기에 enum의 전방 선언을 지원하지 않는다.  
(모든 열거자가 나열(정의)된 경우에만 지원한다.)

 그렇다고 enum 을 전방 선언하지 못한다면 컴파일 의존 관계가 늘어난다.
 ```c++
    enum class Color {black, white, red
    , blue // 추가
    };
 ```
 이렇게 하나만 추가되어도 시스템 전체가 다시 컴파일 되어야 한다.  

 enum class가 전방선언이 되는 이유는 바탕형식을 컴파일러가 알기 때문이다.  
 ```c++
enum class Status // 바탕형식은 기본적으로 int 이다.

enum class Status: std::uint8_t; // 명식적으로 지정 가능.
 ```

enum 전방선언 하는법

```c++
enum Status: std::uint8_t; // 바탕형식을 명시적으로 지정해주면 된다.

enum Status: std::uint8_t{
    good = 0,
    failed = 255, // 8bit로 255 까지 가능
}
```

*기억해 둘 사항들*
- enum class의 열거자들은 그 안에서만 보인다. 오직 캐스팅을 통해서 다른 형식으로 변환된다.
- enum, enum class 모두 바탕 형식 지정을 지원한다.
- enum 기본 바탕 형식은 없다.
- enum class 바탕 형식은 int 이다.
- enum class 는 항상 전방 선언이 가능하다.
- enum 은 해당 선언에 바탕형식을 지정하는 경우에만 전방 선언이 가능하다.

## <span style="color:#8F7CEE">  5. 정의되지 않은 비공개 함수보다 삭제된 함수를 선호하라 </span>
 
 c++98 비공개 함수 선언
 ```c++
...
    private:
        basic_ios(const basic_ios&);
        basic_ios& operator=(const basic_ios&);

    // private 에 선언함으로 호출할 수 없게한다.
    // 호출 되더라도 정의가 없어서 실패한다.
}
 ```

 c++11 
 ```c++
...
    public:
        basic_ios(const basic_ios&) = delete;
        basic_ios& operator=(const basic_ios&) = delete;

    // delete 사용
 ```

 가독성 차이만 있는것이 아니라 delete 는 어떠한 방법으로도 호출할 수 없다.  
 
 멤버 함수나 friend 함수에서 객체를 복사하려 하면 컴파일 실패하는데,  

 링크 시점에 가서야 발견되는 c++98 방식에 비해 개선된 것이다.


 *삭제 함수가 private 가 아닌 public으로 선언하는 것이 관례인 이유.  
 
 c++은 먼저 함수 접근성을 점검한 후 삭제 여부(delete)를 점검한다.  
 그런데 private에 선언되어 있으면 삭제가 아닌 private(접근성) 문제로 문제 삼는  
 컴파일러가 있다. 따라서 오해를 낳을수 있는 오류 메시지를 보기 싫다면 public 으로 선언해야 한다.

 *삭제 함수의 중요한 장점은 그 어떤 함수도 삭제할 수 있다.  

 private는 멤버 함수만 적용 가능

 ```c++
 //비 멤버 함수
bool isLucky(int number);


// int가 아닌 값들도 컴파일 가능
isLucky('a');
isLucky(true);
isLucky(3.5);

// 반드시 정수여야 하는 경우 삭제시켜 준다.
bool isLucky(char) = delete;
bool isLucky(bool) = delete;
bool isLucky(double) = delete;

//char bool double, float 형 배제
 ```

 *템플릿 인스턴스화 방지


어떠한 자료형에 대해 호출하지 못하게 하려면 (템플릿 특수화)  
delete 하면 된다.  

 ```c++
 ...
public:
    template<typename T>
    void test(T* ptr){...}
}


// 클래스 밖에서 템플릿 특수화 삭제
template<>
void A::test<void>(void*) = delete;
 ```

c++98 에서는 적용할 수 없다.

```c++
public:
    template<typename T>
    void test(T* ptr){...}


private:
// private에 템플릿 특수화 선언
template<>
void A::test<void>(void*); // <---- Error!!
}
```

그 이유는 위의 c++11 예제와 같이 반드시 클래스 범위 밖에 작성해야 한다.   
하여 private 에 선언하여 delete를 흉내낼 수 없다.

*기억해 둘 사항들*
- 정의되지 않은 비공개 함수보다 삭제된 함수를 선호하라.
- 비멤버 함수와 템플릿 인스턴스를 비롯한 그 어떤 함수도 삭제할 수 있다.

## <span style="color:#8F7CEE">  6. 재정의 함수들을 override로 선언하라 </span>

override 필수조건
1. 기반클래스 함수가 반드시 가상함수 이어야 한다.
2. 기반 함수와 파생 함수의 이름이 반드시 동일해야 한다.(소멸자 예외)
3. 기반 함수와 파생 함수의 매개변수 형식들이 반드시 동일해야 한다.
4. 기반 함수와 파생 함수의 const성이 반드시 동일해야 한다.
5. 기반 함수와 파생 함수의 반환 형식과 예외 명세가 반드시 호환되어야 한다.

+ c++11 추가 조건
6. 멤버 함수들의 참조 한정사들이 반드시 동일해야 한다.

참조 한정사(reference qualifier)
```c++
class A{
    public:
        void doWork() &; // *this가 왼값일때 적용 가능
        void doWork() &&; // *this가 오른값일때 적용 가능
}
```

이러한 요구조건들이 뜻하는 것은 작은 실수가 큰 차이를 빚을 수 있다.  

```c++
class Base{
public:
    virtual void mf1() const;
    virtual void mf2(int x);
    virtual void mf3() &;
    void mf4() const;
};

class Derived : public Base{
public:
    virtual void mf1(); // const 없음
    virtual void mf2(unsigned int x); // int -> unsigned int
    virtual void mf3() &&; // & -> &&
    void mf4() const; // Base 에서 virtual 로 선언되지 않음
};

// 이렇게 잘못된 코드도 컴파일이 되는게 문제
```

```c++
class Derived : public Base{
public:
    virtual void mf1() const override;
    virtual void mf2(int x) override;
    virtual void mf3() & override;
    virtual void mf4() const override;

    //virtual 을 제거해도 된다.
```

*기억해 둘 사항들*
- 재정의 함수는 override로 선언하라.
- 멤버 함수 참조 한정사를 이용하면 멤버 함수가 호출되는 객체(*this)의 왼값 버전과 오른값 버전을 다른 방식으로 처리할 수 있다.