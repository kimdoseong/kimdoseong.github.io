---
title:  "Template Method Pattern"
excerpt: "템플릿 메서드 패턴"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "Design Pattern"
tags:
 - [c++, Design_Pattern, Design_Pattern_행태]

date: 2021-08-04
last_modified_at: 2021-08-04

published: true
---

- 전략 패턴과 매우 비슷하지만 전략 패턴은 컴포지션(정적, 동적)을 이용하는데 반해 템플릿 메서드 패턴은 **상속**을 이용한다.
- 한곳에 알고리즘을 정의하고 상세 구현은 다른 곳에 두는 점은 **동일**하다.

```c++
class Beverage {
private:
    virtual void make() = 0; //가상 함수는 private으로 둡니다.
    virtual void add() = 0;

    void boil() { cout << "물을 끓인다" << endl; }
    void cup() { cout << "컵에 넣는다" << endl << endl; }
public:
    void process() { //호출할 함수
        boil();
        add(); // 파생 클래스마다 구현이 달라 재정의 해줍니다.
        make(); // 마찬가지.
        cup();
    }
};

class Coffee : public Beverage {
private:
    void make() override { cout << "커피를 만든다." << endl; } //파생 클래스에서 재정의 해줍니다.
    void add() override { cout << "프림을 넣는다" << endl; }
public:
    Coffee(){}
};

class Ade : public Beverage {
private:
    void make() override { cout << "에이드를 만든다." << endl; }
    void add() override { cout << "시럽을 넣는다" << endl; }
public:
    Ade() {}
};

int main() {
	Beverage* coffee = new Coffee();
	Beverage* ade = new Ade();

	coffee->process(); 
	ade->process();
}
```



결과창

```c++
물을 끓인다
프림을 넣는다
커피를 만든다.
컵에 넣는다

물을 끓인다
시럽을 넣는다
에이드를 만든다.
컵에 넣는다
```



템플릿 메서드 패턴의 설계 선택 사항은 한가지 뿐이다.   

이용할 메서드를 순수 가상함수로 둘 것인지, 실 구현체(공백 함수일 수도 있음)로 할 것 인지 선택한다.  

모든 메서드를 구현할 필요가 없다면, 실 구현체를 선택하고 아무것도 하지 않는 공백 함수를 두는 것이 더 편리할 수 있다.  