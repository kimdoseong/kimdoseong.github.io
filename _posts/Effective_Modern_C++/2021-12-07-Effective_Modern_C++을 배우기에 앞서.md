---
title:  "Effective_Modern_C++을 공부하기 앞서.."
excerpt: "우측값과 우측값 참조"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_Modern_C++]

date: 2021-12-07
last_modified_at: 2021-12-07

published: true
---

## <span style="color:#8F7CEE">  1. 우측값 참조 </span>

Modern c++에서는 이 전에는 사용하지 않았던 개념들이 나오는데 그중의 하나가 바로 **우측값 참조**이다.  

도대체 뭐 때문에 저런 개념이 나왔을까 싶지만 앞으로의 내용에 꼭 필요하니 우측값 참조 정의를 잘 정리하고 넘어가야만 한다.  

우측값 참조는 주소를 취할 수 있는지 보는 것 이다.  

- 주소를 취할 수 있으면 좌측값, 취할 수 없다면 우측값이다.  

**Example**

```c++
void function(int&& rhs) {...}

int main() {
	int num;
	int& lvalref = num;  // Ok
	int& lvalref = 10; // error

	int&& rvalref = 10; // ok
	int&& rvalref = num; //error

	function(10);
    return 0;
}
```

기존에 사용하던 &가 하나 붙은 좌측값 참조는 알고 있던 대로 "10"은 넣을 수 없다.  

하지만 밑 &&참조 rvalref 변수는 "10"이라는 값을 참조로 받고 있으며, function 함수 또한 마찬가지이다. 오히려 "num"이라는 주소를 가진 변수는 error가 발생한다.  

## <span style="color:#8F7CEE">  2. 우측값 참조와 우측값 </span>

우측값 참조와 우측값은 다르다.  

우측값은 식이 끝나면 사라지는 값이고, 우측값 참조는 사라지지 않는다.  

또한 좌측값을 우측값으로 바꿨을 때,바뀐 좌측 값은 사용하지 말자.   

```c++
int main()
{
    int&& rvalref/*우측값 참조(사라지지 않음)*/ = 10; /*우측값(사라짐)*/ 
    
    std::vector<int> v1;
    v1.emplace_back(10);
    
    std::vector<int> v2 = std::move(v1); //move로 v1(좌측값)을 우측값으로 변환
    std::cout << v1.size() << std::endl; //우측값으로 변환된 v1을 사용
    std::cout << v2.size() << std::endl;
    return 0;
}
```



**Result**

```c++
0 //0이 출력..
1 //v2는 v1값을 받아 size "1"을 잘 출력한다.
```



또한 우측값 참조는 우측값이다 라는 생각이 들지만 전혀 아닙니다.  

```c++
void function(int&& rhs) {...}
```

rhs는 우측값 참조로 선언 됐지만, 우측값은 아닙니다. 이게 무슨 말인가 하니 우측값 참조는 좌측값 혹은 우측값이 될 수 있다.  
이를 판단하는 기준은, 이름이 있다면 죄측값, 없다면 우측값 입니다. 라고 정의되어 있습니다.  

위의 예시를 보면, 파라미터가 rhs라는 변수명이 있으니 좌측값 입니다. 만약 변수명이 없다면 우측값 이겠죠?  

**요약**

- 주소를 취할 수 있으면 좌측값, 취할 수 없다면 우측값이다.
- 우측값은 식이 끝나면 사라지는 값이고, 우측값 참조는 사라지지 않는다.
- 좌측값을 우측값으로 바꿨을 때, 바뀐 좌측 값은 사용하지 말자.
- 우측값 참조는 좌측값 혹은 우측값이 될 수 있다. 이를 판단하는 기준은, 이름이 있다면 죄측값, 없다면 우측값이다.

