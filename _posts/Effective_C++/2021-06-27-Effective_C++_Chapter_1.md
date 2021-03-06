---
title:  "Effective_C++_Chapter 1"
excerpt: "C++에 왔으면 C++의 법을 따릅시다"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]

date: 2021-06-27
last_modified_at: 2021-06-27

published: true
---

## <span style="color:#8F7CEE"> 1. #define대신 const, enum, inline을 사용하자</span>
- define은 변수 이름이 숫자 상수로 바꾸기 때문에 디버깅을 할 수 없다.  
- define 사용 횟수만큼 사본이 생기기 때문에 코드의 크기가 더 커질 수 있다.  
- define 된 함수가 error를 발생시킨다.  
  
**Example**  
▶ CONSTNUM을 모두 5로 변경하여 디버깅 할 수 없음.  
```c++
#define CONSTNUM = 5;
```    

▶정수 타입을 제외한 static 변수는 선언된 시점에 초기화를 지원하지 않는다.  
```c++ 
class Constants{
private:
    //static const int ConstNum = 5; //정수는 되지만
    //static const double ConstNum = 0.5; //error 발생!
    //아래 방법으로 사용
    static const double ConstNum; //헤더 파일에 선언
};

const double Constants::ConstNum = 0.5; //구현 파일에서 정의
```  

▶define 된 함수가 error를 발생시키는 경우.  
```c++
#define MAXVALUE(a, b) ((a) > (b) ? (a) : (b))
//a, b를 비교, return 할때 a에 대하여 각각 1씩 ++연산을 하여 7이 return 된다.

template<typename T>
inline T maxVal(const T& a,const T& b){
    return ((a > b) ? a : b);
}
//inline 함수 사용으로 6 return.

int main(int argc, const char * argv[]) {
    int a = 5, b = 1;
    
    cout << "Max: " << MAXVALUE(++a,b) << endl; // 7
    cout << "Max: " << maxVal(++a,b) << endl; // 6
    return 0;
}
```


## <span style="color:#8F7CEE"> 2. const를 애용하자  </span>
- const 키워드로 멤버 함수들을 오버로딩 할 수 있다.  
- 상수 멤버, 비상수 멤버 함수가 기능적으로 똑같이 구현되어 있다면 비상수 버전이 상수 버전을 호출하도록 하자.  
- 컴파일러는 비트수준 상수성, 프로그래머는 논리적 상수성을 사용.  

**Example**  
▶const 키워드를 사용하여 오버로딩.  
명시적으로 캐스팅을 이용하여 비상수 버전이 상수 버전 op[]를 호출할 수 있다.  
```c++
class Constants{
private:
    string text;
    
public:
    Constants(string a) : text(a)
    {}
    
    const char& operator[](int position) const // 상수 operator
    { return text[position]; }

//    char& operator[](int position) // 비상수 operator. 오버로딩 가능
//    { return text[position]; } 

//    char& operator[](int position)
//    {
//        return const_cast<char&>(static_cast<const Constants&>(*this)[position]);
//    } //명시적으로 캐스팅. 비상수 버전이 상수 버전을 호출하게함.
};

int main(int argc, const char * argv[]) {
    const Constants cct("Hello"); //상수 객체 생성
    //cct[0] = "J" //error
    
    Constants ct("Hello"); //비상수 객체 생성
    //ct[0] = 'J'; //error const 객체는 아니지만  const operator 호출한다.
                   //오버로딩시 비상수 operator 호출로 error 발생하지 않음.
    
    return 0;
}
```



▶비트 수준 상수성과 논리적 상수성.  
```c++
class Constants{
private:
    mutable string text; //비트 수준 상수성 해제
    int num;
public:
    Constants(string a) : text(a)
    {}
    
    void mutableTest() const{ //멤버 변수를 변경할 수 없지만
        text = "abc"; //mutable 사용으로 변경 가능.
        //num = 5;//error 변경 불가.
    }
};
```



## <span style="color:#8F7CEE"> 3. 객체를 반드시 초기화 하자</span>
- 기본 제공 타입 객체는 직접 초기화한다.  
- 생성자에서 대입 대신 멤버 초기화 리스트를 사용하자. (클래스에 멤버 변수가 선언된 순서와 똑같이 나열)  
- 비지역 정적 객체를 지역 정적 객체로 바꿔주자. (초기화 순서 문제)  

**Example**  
▶초기화되지 않은 값을 읽도록 하면 미정의 동작을 하게 된다. 초기화 보장에 대한 규칙은 명확히 있지만, 규칙이 복잡하여 모든 객체에 초기화 하는 것이 가장 좋은 방법이다.  
```c++
	int a = 0;
	const char* text = "text";
```



▶초기화 리스트를 사용하자!  
```c++
class Init {
private:
    string _text;
    char _ch;
    int _num;
    
public:
    Init() : _text(), _ch(), _num(0) //기본 생성자 에서도 빈 값으로 순서를 지켜 초기화를 해줍니다.
    {}

    Init(string text, char ch, int num) : _text(text), _ch(ch), _num(num) //초기화 리스트를 사용합니다.
    {
        //_text = text; //이건 대입입니다 초기화가 아님.
        //_ch = ch;
        //_num = num;
    }
};
```


▶초기화 순서 문제에 따른 비지역 정적 객체를 지역 정적 객체로 바꾸자. (Singleton pattern)  
  
정적 객체  
- 전역 객체  
- namespace 유효범위에서 선언된 객체  
- 클래스 안에서 static으로 선언된 객체  
- 함수 안에서 static으로 선언된 객체  
- 파일 유효 범위에서 static으로 선언된 객체  
  
지역 정적 객체   
- 함수 안에 있는 정적 객체  
  
비지역 정적 객체  
- 그 외  
  
비지역 정적 객체를 담당하는 함수를 만들고 그 함수 안에 정적 객체(지역 정적 객체)를 만들어 retrun 한다.  
아래처럼 정적 멤버로 넣어 사용해도 된다.  
```c++
class Singleton {
private:
    static Singleton* s_instance; //정적 멤버로 선언
    string _text;
public:
    Singleton() : _text()
    {}

    static Singleton* getInstance() {
        if (!s_instance) {
            s_instance = new Singleton;
        }
        return s_instance;
    }

    string getText() {
        return _text;
    }
};

Singleton* Singleton::s_instance = nullptr; //구현 파일에 정의

class SingletonCall {
private:
    string _text;
public:
    SingletonCall(){}

    void setText() {
        _text = Singleton::getInstance()->getText(); //초기화 순서 문제를 피해서 설계!
    }
};
```