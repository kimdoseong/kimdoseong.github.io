---
title:  "Effective_Modern_C++_Chapter 2"
excerpt: "auto"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2022-03-20
last_modified_at: 2022-03-27

published: true
---

## <span style="color:#8F7CEE">  1. 명시적 형식 선언보다는 auto를 선호하라 </span>
 
 초기화

 ```c++
int x1; // 문맥에 따라서 초기화 되지 않을 수 있음
auto x2; // error 초기화 꼭 필요
auto x3 = 0; // x3의 값이 잘 정의됨.
 ```

**std::fuction**

c++11 표준 라이브러리의 한 템플릿으로 함수 포인터 개념을 일반화한 것이다.  

함수 포인터와 달리 호출 가능한 객체라면 그 어떤 것도 가리킬 수 있다.  
(함수처럼 호출할 수 있는 것)

```c++
// std::function
    std::function<bool(const int &, const int &)> fp = [](
            const int &p1, const int &p2) -> bool {
        return p1 < p2;
    };

// auto
    auto aFp = [](
            const auto &p1, const auto &p2) -> bool {
        return p1 < p2;
    };
...
    auto b1 = fp(a, b);
    auto b2 = aFp(a, b);
```

auto는 클로저에 요구되는 만큼의 메모리만 사용한다.

std::function으로 선언된 변수의 형식은 std::function 템플릿의 한 인스턴스 이며  
그 크기는 임의의 주어진 서명에 대해 고정되어 있다. 그런데 그 크기가 요구된 클로저를 저장하기에  
부족할 수도 있으며, 힙 메모리를 할당하여 클로저를 저장한다.  

std::function은 대체로 auto로 선언된 객체보다 메모리를 더 많이 소비한다.  
그리고 인라인화를 제한하고, 구현 세부사항 때문에, 거의 항상 auto로 선언된  
객체를 통해 호출하는 것보다 느리다. 때에 따라서는 out of memory를 유발한다.  

auto의 장점은 변수 초기화 누락을 방지하고 장황한 변수 선언을 피하는 것, 클로저를 직접 담는 것,  
또 type shortcut 이라고 부르는 것과 관련한 문제를 피할 수 있다.  

```c++
std::vector<int> v;
int i = v.size();
```
v.size()의 공식적인 반환 형식은 std::vector<int>::size_type인데, 정확한 자료형으로 받지 않는다.  
또한 std::vector<int>::size_type 은 시스템에 따라서 다를 수 있기 때문에 32비트 시스템에서 잘 작독 하다가도,  
64비트 시스템에서 오작동할 수 있다.

```c++
std::unordered_map<std::string, int> m  
for(const std::pair<std::string, int>&p : m){
    ...
}
```

std::unordered_map 의 키 부분은 const 이다. 하지만 그 부분을 잊은채 위와 같은 코드를 작성한다면  
컴파일러는 std::<string, int>로 변환하려 든다. 루프문에서 생각지도 못한 복잡한 작업이 일어나기 때문에  
아래와 같이 작성하는 것이 낫다.

```c++
for(const auto& p: m){
    ...
}
```

*기억해 둘 사항들*
- auto 변수는 반드시 초기화해야 하며, 이식성 또는 효율성 문제를 유발할 수 있는 형식 불일치가  
발생하는 경우가 거의 없으며, 대체로 변수의 형식을 명시적으로 지정할 때보다 타자량도 더 적다.

## <span style="color:#8F7CEE">  2. auto가 원치 않은 형식으로 연역될 떄에는 명식적 형식의 초기치를 사용하라 </span>

- bool 

bool은 bool당 1비트의 압축된 형태로 표현하도록 되어있다.  

하지만 c++ 에서는 비트에 대한 참조는 금지되어 있으므로 bool&를 직접 돌려줄 수 없다.  

-> std::vector<bool>::reference (bool& 이 아닌 bool 형)

따라서 auto로 vector<bool> 형을 받으면 원하던 bool을 받을 수 없게된다.  

(_Bit_reference)


그런 경우에 아래와 같이 강제한다.  

```c++
auto b = static_cast<bool>(features(w)[5]);
```

*기억해 둘 사항들*
- auto가 초기화 표현식의 형식을 잘못 연역 할 수 있다.
- auto가 원하는 형식을 연역하도록 강제한다.