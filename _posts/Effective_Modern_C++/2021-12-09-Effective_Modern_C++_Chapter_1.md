---
title:  "Effective_Modern_C++_Chapter 1"
excerpt: "형식 연역"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2021-12-09
last_modified_at: 2022-03-20

published: true
---

## <span style="color:#8F7CEE">  1. 템플릿 형식 연역 규칙을 숙지하라 </span>
 
**1-1. ParamType이 포인터 또는 참조 형식이지만 보편 참조는 아님**

```c++
template<typename T>
void f(T& param); //참조

int x = 27;
const int cx = x;
const int& rx = x;


f(x); // T - int        / param - int&
f(cx); // T - const int  / param - const int&
f(rx); // T - const int& / param - const int&
```

 예상대로 T 에는 매개변수의 자료형, ParamType 에는 선언된 형까지 더해진다.  

**1-2. ParamType이 보편 참조임**

```c++
template<typename T>
void f(T&& param); //보편참조
 
int x = 27;
const int cx = x;
const int& rx = x;


//왼값
f(x);  // T - int&        / param - int&
f(cx); // T - const int&  / param - const int&
f(rx); // T - const int&  / param - const int&

//오른값
f(27); // T - int         / param - int&&
```

왼값에 대해서는 T 에 &가 붙고, 오른값에 대해서는 T는 오른값에 대한 자료형, ParamType 은 보편참조로 받게된다.  

**1-3. ParamType이 포인터, 참조가 아님**

```c++
template<typename T>
void f(T param); // 포인터, 참조가 아님
 
int x = 27;
const int cx = x;
const int& rx = x;

f(x);  // T - int / param - int
f(cx); // T - int / param - int
f(rx); // T - int / param - int
```

**배열 인수**

우선 배열 형식의 함수 매개변수라는 것은 없다.  
```c++
void Func(int param[]);
```

위와 같은 선언 자체는 적법하다. 하지만 위의 구문은 포인터 선언으로 취급된다.  
```c++
void Func(int* param);
```

진짜 배열 선언은 없지만 배열에 대한 참조로 선언할 수 있다.  
```c++
template<typename T>
void Func(T& param); 
//& 사용으로 배열로 연역
const char name[] = "kimdoseong"

Func(name);
```

위와 같이 &로 선언 한다면 T에 배열로 형식이 연역된다.  
T -> const char [11]  / paramType -> const char (&)[11]  

배열에 대한 참조 선언을 사용하면 배열에 담긴 원소의 개수를 연역하는 템플릿을 만들 수 있다.  
```c++
template<typename T, size_t N>
constexpr std::size_t ArrSize(T (&)[N]){ 
//param 생략
    return N;
}
```

**함수 인수**

포인터로 붕괴되는 것은 함수 포인터도 포함된다.

```c++
void Func(int, double);

template<typename T>
void f1(T param);

template<typename T>
void f2(T& param);

f1(Func); //함수 포인터로 연역 - void (*)(int, double)

f2(Func); //함수 참조로 연역 - void (&)(int, double)
```

*기억해 둘 사항들*
- 템플릿 형식 연역 도중에 참조 형식의 인수들은 비참조로 취급된다. 즉, 참조성이 무시된다.  
- 보편 참조 매개변수에 대한 형식 연역 과정에서 왼값 인수들은 특별하게 취급된다.  
- 값 전달 방식의 매개변수에 대한 형식 연역 과정에서 const or volatile 인수는 비 const, 비 volatile 인수로 취급된다.  
- 템플릿 형식 연역 과정에서 배열이나 함수 이름에 해당하는 인수는 포인터로 붕괴한다. 단, 그런 인수가 참조를 초기화 하는데  
  쓰이는 경우에는 포인터로 붕괴하지 않는다.  


## <span style="color:#8F7CEE"> 2. auto의 형식 연역 규칙을 숙지하라 </span>

auto의 형식도 템플릿 형식 연역과 똑같이 작동한다.  

```c++
const char name[] = "kimdoseong";

auto arr1 = name; // const char*
auto arr2 = name; // const char &[11]

void Func(int, double);
auto func1 = Func // void(*)(int, double)
auto func2 = Func // void(&)(int, double)
```

그러나 다른점도 존재한다.  

C++98 두가지 구문  
```c++
int x = 27;
int x(27);
```

C++11 에 추가된 두가지 구문  
```c++
int x = {27};
int x{27};
```

총 네가지 구문이 존재한다.  

```c++
auto x = 27; // int
auto x(27);  // int
auto x = {27}; //std::initializer_list<int>
auto x{27}; //std::initializer_list<int>
```

std::initializer_list<T> 에 연역되지 않으면 컴파일이 거부된다.
```c++
auto x = {1, 2, 3.0}; // 자료형이 다르므로 std::initalizer_list<T>의 T를 연역할 수 없다.
```

```c++
template<typename T>
void f(T param);

f({1, 2, 3}) // Error -> T로 연역되지 않음
```

정상 연역
```c++
template<typename T>
void f(std::initializer_list<T> param);

f({1, 2 ,3}) // 제대로 연역됨.
```

템플릿과 auto의 차이점  
- auto 는 중괄호가 std::initializer_list<T> 를 나타낸다고 가정
- template 는 그렇지 않다.

-> 의도하지 않았지만 std::initializer_list 로 선언하는 것을 주의해야 한다.


C++14 에서는 auto 도 반환 형식으로 지정할 수 있다. 하지만 그러한 구문은  
auto 형식 연역이 아니라 template 형식 연역 규칙이 적용된다.

```c++
auto f(){
    return ({1, 2, 3}) // Error 위의 template과 같은 사유
}

std::vector<int> v;

// value type에 auto도 std::initializer_list 로 연역되지 않는다.
auto V = [&](const auto& value){ v = value };

// template 과 같은 형식 연역.
V({1, 2, 3})
```

## <span style="color:#8F7CEE"> 3. decltype의 작동 방식을 숙지하라 </span>

decltype - 주어진 이름이나 표현식의 형식을 알려준다.
(declared type)
```c++
//decltype
const int i = 0; // decltype(i) - const int
bool f(const Widget& w) //decltype(w) - const Widget&
                        //decltype(f) - bool(cont Widget&)

template<typename T>
class vector{
  public:
  T& operator[](std::size_t index);
  ...
}               

vector<int> v;
if(v[0] == 0)..
//decltype(v[0]) 은 int&
```

후행 반환 형식
- 반환 형식을 매개변수 목록 다음에 선언.

```c++
//c++11
template<typename T, typename Index>
auto f(T &t, Index i)
-> decltype(t[i]) {
    return t[i];
}

//return type int&
```

```c++
//c++14 후행반환 없음
template<typename T, typename Index>
auto f(T &t, Index i)
{
    return t[i];
}

...
std::deque<int> d;
f(d, 5) = 10;
// d[5] 는 int& 이지만, return은 auto로 int(t[i]의 값) 를 return 한다.
// -> compile error

---
template<typename T, typename Index>
decltype(auto) f(T &t, Index i)
{
    return t[i];
}
// return type int&

std::deque<int> d;
f(d, 5) = 10;
// compile success
``` 

위의 예제 에서는 오른값은 전달할 수 없다. 그렇다고 오른값 참조 매개변수를 받는 함수를 만들면 관리해야 할 함수가  
두개가 되므로 왼값과 오른값 모두 받을 수 있는 참조 매개변수를 넣어준다.

```c++
// c++14 보편 참조 사용
template<typename T, typename Index>
decltype(auto) f(T &&t, Index i)
{
    return std::forward<T>(t)[i];
}

// c++11
template<typename T, typename Index>
auto f(T &&t, Index i)
->decltype(std::forward<T>(t)[i])
{
    return std::forward<T>(t)[i];
}
```
*기억해 둘 사항들*
- decltype은 항상 변수나 표현식의 형식을 아무 수정 없이 보고한다.
- decltype은 형식이 T이고 이름이 아닌 왼값 표현식에 대해서는 항상 T& 형식을 보고한다. (첫번째 c++11 예제)
- c++14는 decltype(auto)를 지원한다.

## <span style="color:#8F7CEE"> 4. 연역된 형식을 파악하는 방법을 알아두라 </span>

*기억해 둘 사항들*
- 컴파일러가 연역하는 형식을 IDE 편집기나 컴파일러 오류 메시지, Boost TypeIndex 라이브러리를 이용해서 파악할 수 있는 경우가 많다.
- 일부 도구의 결과는 유용하지도 않고 정확하지도 않을 수 있으므로, c++ 의 형식 연역 규칙들을 제대로 이해하는 것은 여전히 필요한 일이다.





