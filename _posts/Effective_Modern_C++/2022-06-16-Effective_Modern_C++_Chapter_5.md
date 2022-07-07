---
title:  "Effective_Modern_C++_Chapter 5"
excerpt: "오른값 참조, 이동 의미론, 완벽 전달"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2022-06-16
last_modified_at: 2022-07-26

published: true
---

## <span style="color:#8F7CEE">  1. std::move 와 std::forward 를 숙지하라 </span>

std::move 는 주어진 인수를 무조건 오른값으로 캐스팅하고, std::forward 는 특정 조건이 만족될 때에만 그런 캐스팅을 수행한다.

둘다 그냥 캐스팅하는 함수이다.

std::move 구현

```c++

    template<typename T>
    decltype(auto) move(T&& param){
        using ReturnType = remove_reference_t<T>&&;
        return static_cast<ReturnType>(param);
    }

```

T&& 가 왼값인 경우를 위해 std::remove_reference를 적용한다. 결과적으로 std::move 는 오른값 참조를 돌려준다.

- std::move 는 자신의 인수를 오른값으로 캐스팅하며 이것이 std::move 의 전부이다.

주의해야 할 점

1. 이동을 지원할 객체는 const 로 선언하지 말아야 한다. const 객체에 대한 이동 요청은 소리 없이 복사 연산으로 변환된다.
2. std::move 는 아무것도 실제로 이동하지 않을 뿐만 아니라, 캐스팅되는 객체가 이동 자격을 갖추게 된다는 보장도 제공하지 않는다.

  

std::forward 예제

```c++

    void process(const Widget& lval);
    void process(Widget&& rval);

    template<typename T>
    void CallProcess(T&& param){
        process(std::forward<T>(param));
    }

```

호출

```c++

    Widget w;
    CallProcess(w);
    CallProcess(std::move(w));

```

process 함수는 왼값과 오른값을 받을수 있게 overloading 되어 있고 param 에 따라 왼값 오른값 나뉘어 호출하는 것을 기대한다.

하지만 param 은 모든 함수 매개변수 처럼 하나의 왼값이며 process 중 왼값 버전만 호출된다. (위의 코드에서 std::forward<T> 가 없는 경우 위의 경우는 잘 된다!)

오른값 호출을 하기 위해서는 param 을 초기화 하는데 쓰인 인수가 오른값이면, 그리고 오직 그럴 때에만, param을 오른값으로 캐스팅하는 것이 std::forward 이다.  

**오른값으로 초기화**된 것 일 때에만 그것을 오른값으로 캐스팅 한다는 점에서 조건부 캐스팅이라고 부른다.

*기억해 둘 사항들*
 - std::move 는 오른값으로의 무조건 캐스팅을 수행한다. std::move 자체는 아무것도 이동하지 않는다.
 - std::forward 는 주어진 인수가 오른값에 묶인 경우에만 그것을 오른값으로 캐스팅한다.
 - std::move 와 std::forward 둘 다, 실행 시점에서는 아무 일도 하지 않는다.

## <span style="color:#8F7CEE">  2. 보편 참조와 오른값 참조를 구별하라 </span>

어떤 형식 T에 대한 오른값 참조를 선언할 때에는 T&&라는 표시를 사용한다.  
하지만 T&& 라고 모두 오른값 참조는 아니다.

```c++

    void f(Widget&& param); // 오른값 참조
    Widget&& var1 = Widget(); // 오른값 참조

    auto&& var2 = var1; // 오른값 참조 아님

    template<typename T>
    void f(std::vector<T>&& param) // 오른값 참조

    template<typename T>
    void T(T&& param) // 오른값 참조 아님
    
```

**T&& 는 두가지 의미가 있다. **
1. 오른값 참조
2. 왼값 참조

둘중 하나라는 것이다. 이를 보편 참조라고 한다.

```c++

// 1. template
    template<typename T>
    void f(T&& param); // param 은 보편 참조

// 2. auto
    auto&& var2 = var1; // var2는 보편 참조

```

둘의 공통점은 param의 형식이 연역되며, var2의 형식이 연역된다. 이처럼 형식 연역이 일어나지 않는 문맥에서 T&&를 발견했다면 그것은 오른값 참조이다.  

```c++

    void f(Widget&& param); // 형식 연역 없음 
                            // param 은 오른값 참조

    Widget&& var1 = Widget() // 형식 연역 없음
                             // var1은 오른값 참조
    
```

보편 참조는 참조이므로 반드시 초기화해야 한다.  

초기치가 오른값인지 왼값인지에 따라 보편 참조의 형식을 결정하게 된다.  

```c++

    template<typename T>
    void f(T&& param);  //param은 보편 참조

    Widget w;
    f(w);   // f에 왼값이 전달됨 param 형식은 
            // Widget& (왼값 참조)

    f(std::move(w)); // f에 오른값이 전달됨 param 형식은
                     //  Widget&& (오른값 참조))

```

이처럼 보편 참조가 되려면 형식 연역에 관여를 해야하지만 충분 조건은 아니다.
구체적으로 딱 T&& 형태이어야 한다.

```c++

    template<typename T>
    void f(std::vector<T>&& param); // param은 오른값 참조

```

f 호출시 T가 연역된다. 하지만 앞서 말한것과 같이 T&& 형이 아닌 std::vector<T>&& 이다.  

이 때문에 param은 보편 참조가 될 수 없다. (오른값 참조임)

const 도 마찬가지이다. 

```c++

    std::vector<int> v;
    f(v); // 왼값으로 error 발생.

```


**T&& 가 보편 참조가 아닌 경우**

------

1. **std::vector 의 push_back 함수**

```c++

    template<class T, class Allocator = allocator<T>>
    class vector {
        public:
        void push_back(T&& x);
    }

```

push_back 은 T&& 로 보편참조의 형식이지만 형식 연역이 전혀 일어나지 않는다.  

push_back 은 반드시 구체적으로 인스턴스화 된 vector의 일부이어야 하며, 그 인스턴스의 형식은 push_back 의 선언을 완전하게 결정하기 때문이다.

```c++

    std::vector<Widget> v;
    
    //

    class vector<Widget, allocator<Widget>> {
        public:
        void push_back(Widget&& x); // 오른값 참조
    }

```

std::vector 라는 선언에 의해 std::vector 템플릿은 위와 같이 인스턴스화 되며 구체화 된다.  

반대로 std::vector 의 emplace_back 멤버 함수는 실제로 형식 연역을 사용한다.  

```c++

    template<class T, class Allocator = allocator<T>>
    class vector {
        public:
        template<class T, Args>
        void emplace_back(Arg&&... args);
        ...
    }
```

Args 는 T 와 독립적이며, emplace_back 이 호출될 때마다 연역 되어야 한다.  

따라서 Args 는 형식연역이 일어나는 T&& 형 으로 보편 참조이다.  


2. **auto&&**

auto&&의 경우도 형식 연역이 일어나며 T&& 형태이므로 보편 참조가 된다.

```c++

    // c++14
    auto timeFuncInvocation =
    [](auto&& func, auto&&... params)
    {
        std::forward<decltype(func)>(func)( // params로
        std::forward<decltype(params)>(params)...   // func 호출
        );
    // 타이머를 정지하고 경과 시간을 기록한다.
    }

```

func는 어떤 호출 가능 객체와도 묶일 수 있는 보편 참조이다.

params는 임의의 형식, 임의의 개수의 객체들과 묶일 수 있는 0개 이상의 보편 참조들이다.

auto 보편 참조 덕분에 timeFuncInvocation함수는 거의 모든 함수의 실행 시간을 측정할 수 있다.


*기억해 둘 사항들*
- 함수 템플릿 매개변수의 형식이 T&& 형태이고 T가 연역된다면, 또는 객체를 auto&&로 선언해야 한다면, 그 매개변수나 객체는 보편 참조이다.
- 형식 선언의 형태가 정확히 형식&&가 아니면, 또는 형식 연역이 일어나지 않으면, 형식&&는 오른값 참조를 뜻한다.
- 오른값으로 초기화되는 보편 참조는 오른값 참조에 해당한다. 왼값으로 초기화되는 보편 참조는 왼값 참조에 해당한다.

## <span style="color:#8F7CEE">  3. 오른값 참조에는 std::move를, 보편 참조에는 std::forward를 사용하라 </span>


*기억해 둘 사항들*