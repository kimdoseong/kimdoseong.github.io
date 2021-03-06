---
title:  "Effective_C++_Chapter 6"
excerpt: "상속, 그리고 객체 지향 설계"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]


date: 2021-07-18 
last_modified_at: 2021-07-18

published: true
---

## <span style="color:#9282CD">1. public 상속 모형은 반드시 "is-a"를 따르도록 만들자</span>

- public 상속의 의미는 "is-a"의 일종이다. 기반 클래스에 적용되는 것들은 파생 클래스 에도 적용 되어야 한다.
- 파생 클래스는 기반 클래스 객체의 일종이다.

**Example**

```c++
class Bird {
public:
    virtual void fly() {
        cout << "Bird fly" << endl;
    }
};

class Penguin :public Bird {
public:
    //void fly() override{ //주석 제거하면 올바른 결과가 나오게 됩니다.
    //    cout << "Penguins can't fly." << endl; //fly를 재정의 하지 않는다면
    //}
};

Penguin P;
P.fly();
```

결과창

```c++
Bird fly //펭귄은 날지 못하는데.. 날려버리고 맙니다..

//주석 제거시 올바른 결과
Penguins can't fly. 
```

## <span style="color:#8F7CEE"> 2. 상속된 이름을 숨기는 일은 피하자 </span>

- 파생 클래스의 이름은 기본 클래스의 이름을 가린다. public 상속 에서의 이런 이름 가림 현상을 조심하자.
- 기본의 오버로딩과 달리 매개변수가 달라도 이름을 가리게 된다.
- 오버로딩 된 함수 중 함수 몇 개만 오버라이딩 하고 싶다면 using 선언을 붙여주면 된다.(혹은 전달 함수)

**Example**  

▶잘못된 예  

```c++
class Base {
public:    
    virtual void test1() = 0;
    virtual void test1(int) {
        cout << "Base int test1" << endl;
    }
    virtual void test2() {
        cout << "Base void test2 " << endl;
    }
    virtual void test3(double) {
        cout << "Base double test3 " << endl;
    }
};

class Derived :public Base {
public:
    void test1() override {
        cout << "Derived void test1" << endl;
    }
    void test3() {
        cout << "Derived void test3" << endl;
    }
};


Derived d;
d.test1();
d.test1(1); //error, Base::test(int)도 오버라이딩 해야합니다.
d.test2(); //Base::test2()를 호출 합니다. Derived에 test2 이름의 함수가 없습니다.
d.test3();
d.test3(1.0);//error 파생 클래스에 test3()가 있어서
//Base::test3(double)을 호출하지 못합니다.
//함수 이름을 통해서만 호출 합니다.
```

▶좋은 예

```c++
class Derived :public Base {
public:
    using Base::test1; //Base 클래스 test1(int) 함수도 접근이 가능합니다.
    using Base::test3; //Base 클래스 test3(double) 함수도 접근이 가능합니다.
    
    void test1() override {
        cout << "Derived void test1" << endl;
    }
    void test3() {
        cout << "Derived void test3" << endl;
    }
};
```



## <span style="color:#8F7CEE"> 3. 인터페이스 상속과 구현 상속의 차이를 제대로 파악하고 구별하자 </span>

- 인터페이스 상속은 구현 상속과 다르다. public 상속에서, 파생 클래스는 항상 기반 클래스의 인터페이스를 모두 물려받는다.
- 순수 가상 함수는 인터페이스 상속만 허용
- 단순 가상 함수는 인터페이스 상속과 더불어 기본 구현의 상속도 가능하도록 지정.

**Example**  

▶단순 가상 함수에서의 기본 구현 

```c++
class Base1 {
public:    
    virtual void print() {
        cout << "Base print()" << endl; //단순 가상 함수의 기본 구현
    }
};

class Derived1 :public Base1 {
public:
    void print() override{
        cout << "Derived1 print()" << endl;
    }
};

class Derived2 :public Base1 {
public:
};

Base1 b1;
Derived1 d1;
Derived2 d2;

b1.print();
d1.print(); //재정의로 Derived1::print() 호출.
d2.print(); //재정의 하지 않음으로 Base1::print() 호출.
```

결과창

```c++
Base print()
Derived1 print()
Base print()
```

▶순수 가상 함수의 구현

```c++
class Base1 {
public:
    virtual void print() = 0; //순수 가상 함수
};

void Base1::print() {
        cout << "Base1 print()" << endl; //순수 가상 함수의 구현
}

class Derived1 :public Base1 {
public:
    void print() override {
        Base1::print(); //재정의가 필요 없으면 명시적으로 인터페이스 함수 호출
    }
};

class Derived2 :public Base1 {
public:
    void print() override {
        cout << "Derived2 print()" << endl; //재정의가 필요한 파생 클래스의 경우.
    }
};
```

순수 가상 함수의 구현 이유  

기반 클래스에서 순수 가상 함수를 만들어 파생 클래스에서의 오버라이딩을 강제하고, 파생 클래스에서 특별한 기능을 구현할 필요가 없을 경우 단순 가상 함수처럼 기반 클래스의 함수를 사용하도록 한다. 오버라이딩의 강제로 상속받은 파생 클래스에서 순수 가상 함수를 한번 더 확인하게 하여 호출함에 있어서 프로그래머의 실수를 막도록 한다.  

## <span style="color:#8F7CEE">4. 가상 함수 대신 쓸 것들도 생각해 두는 자세를 시시때때로 길러 두자 </span>

-  가상 함수 대신 쓸수 있는 방법 - NVI(non-virtual-interface idiom) 관용구(템플릿 메소드 패턴), 전략패턴

**Example**  

▶템플릿 메소드 패턴 구현  

템플릿 메소드는 "가상 함수는 반드시 private 멤버로 두어야 한다"는 가상 함수 은폐론 이다.

```c++
class Beverage {
private:
    virtual void make() = 0; //가상 함수는 private으로 둡니다.
    virtual void add() = 0;

    void boil() { cout << "물을 끓인다" << endl; }
    void cup() { cout << "컵에 넣는다" << endl << endl; }
public:
    void process() {
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

	delete coffee;
	delete ade;
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



▶전략 패턴 구현    

동적으로 알고리즘을 교체할 수 있는 구조이다.  

```c++
class Strategy {
public:
    virtual void interface() = 0;
};

class StrategyA :public Strategy {
public:
    void interface() override { cout << "Call StrategyA" << endl; }
};

class StrategyB :public Strategy {
public:
    void interface() override { cout << "Call StrategyB" << endl; }
};

class StrategyC :public Strategy {
public:
    void interface() override { cout << "Call StrategyC" << endl; }
};

class Context {
private:
    Strategy* _pStrategy;
public:
    void changeStrategy(Strategy* strategy) {
        if (_pStrategy)
            delete _pStrategy;
        _pStrategy = strategy;
    }

    void contextInterface() {
        _pStrategy->interface();
    }
};



Context* pContext = new Context();
pContext->changeStrategy(new StrategyA);
pContext->contextInterface();

pContext->changeStrategy(new StrategyB);
pContext->contextInterface();

pContext->changeStrategy(new StrategyC);
pContext->contextInterface();
```

결과창

```c++
Call StrategyA
Call StrategyB
Call StrategyC
```



## <span style="color:#8F7CEE">5. 상속받은 비가상 함수를 파생 클래스에서 재정의 하지 말자 </span>

- 상속받은 비가상 함수를 재정의 하지 말자  

**Example**  

▶비가상 함수를 재정의 하면 안됩니다.  

원하지 않은 동작을 할 수 있다.  

```c++
class Base {
public:
    void printClass() {
    	cout << "Base Class" << endl;
    }
};

class Derived : public Base {
public:
    void printClass() {
        cout << "Derived Class" << endl; 
    }// 비가상 함수를 재정의 
};

int main() {
	Base* pb = new Derived();
	pb->printClass();
}
```

결과창

```c++
Base Class
//Derived Class가 출력되길 원했는데 말이죠..
```



## <span style="color:#8F7CEE">6. 상속받은 함수에 대해 기본 매개변수 값은 절대로 재정의 하지 말자</span>

- 기본 매개변수는 정적으로 바인딩 되지만, 가상 함수는 동적으로 바인딩 된다.
- 비가상 함수는 (5 참조) 언제라도 재정의 하면 안되는 함수이다.

**Example**  

▶디폴트 매개 변수를 설정하여 원하지 않은 동작이 실행된 예

```c++
class Shape {
public:
    enum class DrawColor{Red, Green, Blue};
    virtual void draw(DrawColor dc = DrawColor::Red) = 0; //디폴트 매개 변수(정적 타입) Red 설정
};

class Rectangle :public Shape{
public:
    virtual void draw(DrawColor dc = DrawColor::Green) { //디폴트 매개 변수(정적 타입) Green 으로 변경
        switch (dc) {
        case DrawColor::Red:
            cout << "Rectangle Draw Color is Red" << endl;
            break;
        case DrawColor::Blue:
            cout << "Rectangle Draw Color is Blue" << endl;
            break;
        case DrawColor::Green:
            cout << "Rectangle Draw Color is Green" << endl;
            break;
        }
    }
};


int main() {
	Shape* ps = new Rectangle(); //Shape* 정적 타입 입니다. 근데 동적 타입으로 Rectangle을 받으니
	ps->draw(); //draw 함수에서 섞여버립니다.
}
```

결과창

```c++
Rectangle Draw Color is Red //Rectangle 객체인데 Red가 출력됐습니다..
```



▶올바른 예로 NVI를 사용할 수 있습니다.

```c++
class Shape {
public:
    enum class DrawColor { Red, Green, Blue };
    void draw(DrawColor dc = DrawColor::Red) {
        doDraw(dc);
    }// 호출 함수 
    //기본 매개변수 Red설정, 코드 중복과 의존성을 없애준다.
private:
    virtual void doDraw(DrawColor dc) = 0; //private에 선언, 실제 작업이 이루어지는 함수
};

class Rectangle :public Shape{
public:
    virtual void doDraw(DrawColor dc) {
        switch (dc) {
        case DrawColor::Red:
            cout << "Rectangle Draw Color is Red" << endl;
            break;
        case DrawColor::Blue:
            cout << "Rectangle Draw Color is Blue" << endl;
            break;
        case DrawColor::Green:
            cout << "Rectangle Draw Color is Green" << endl;
            break;
        }
    }
};

int main() {
	Shape* ps = new Rectangle();
	ps->draw(); 
	ps->draw(Shape::DrawColor::Green); //원하는 매개변수로 호출 가능
}
```

결과창

```c++
Rectangle Draw Color is Red
Rectangle Draw Color is Green 
```



## <span style="color:#8F7CEE">7. "has-a" 개념 or "is-implement-in-terms-of"을 모형화할 때는 객체 합성을 사용</span>

- 객체 합성은 public(is-a) 상속이 가진 의미와 완전히 다르다.
- 객체 합성은 "has-a"이고, 구현은 "is-implemented-in-terms-of"이다.

**Example**  

▶객체 합성

is-a와 has-a 관계  

- Date is a Month(날짜는 월의 일종) 보다 Date has a Month(날짜는 월을 가진다)의 관계가 자연스럽다.  

```c++
struct Day {...};

struct Month {...};

struct Year {...};

class Date {
private:
    Month _month; //Date 클래스을 이루는 객체 중 하나
    Day _day; //has - a 관계
    Year _year;
public:
    Date(const Month& month, const Day& day, const Year& year){...}
};
```



## <span style="color:#8F7CEE">8. private 상속은 심사숙고해서 구사하자</span>

- private 상속의 의미는 is-implement-in-terms-of(~는~를 써서 구현됨)이다.
- 파생 클래스에서 기본 클래스 protected 멤버에 접근해야 할 경우 혹은 상속받은 가상 함수를 재정의해야 할 경우에 나름 의미를 갖는다.
- 기본 클래스 최적화(EBO)를 활성화시킬 수 있다. 

**Example**  

▶private 상속  

public "is-a" 상속이 틀린 선택일 경우 private 상속을 하게 되는데, 웬만하면 객체 합성을 사용한다.

```c++
class Timer {
public:
	virtual void onTick(int num) {
		val = num;
		cout << "Timer onTick val" << endl;
	}
protected:
	int val = 1;
};

class Widget :private Timer{
private:
	void doCall(int num) {
		onTick(num);
		cout << "Timer::val: " << Timer::val << endl;;
		//protected 멤버 접근 가능
	}

	void onTick(int num) override { //가상 함수도 재정의 가능 
		cout << "Widget onTick" << endl;
		Timer::onTick(num); //상속받은 함수 호출
	}  
public:
	void call(int num) {
		doCall(num);
	}
};

//private 상속 대신 사용하는 방법

class Timer {...};

class Widget{
private: //private 내에 클래스를 선언하여 public으로 Timer를 상속 받습니다.
	class WidgetTimer : public Timer { 
	public:
		void onTick() override {
			Timer::onTick();
		}
	};
	WidgetTimer wt; //객체 생성
public:
	void call() {
		wt.onTick();
	}
};

// 아래 방법의 좋은점
1. Widget 클래스 파생은 가능하나, 파생 클래스에서 onTick 함수를 재정의 할수 없게 한다.
- private 가상 함수는 파생 클래스에서 재정의 가능합니다.
- WidgetTimer 클래스는 파생 클래스에서 건드릴 수 없습니다..
    
2. Widget의 컴파일 의존성을 최소화 하고 싶을 때.
- WidgetTimer의 정의를 widget에서 빼고 Widget이 WidgetTimer 포인터를 갖도록 하면 된다.  
```

아래 방법 결과창  

```c++
Timer onTick
```



## <span style="color:#8F7CEE">9. 다중 상속은 심사숙고해서 사용하자</span>

- 다중 상속(MI: mulitple inheritance)은 복잡하며, 모호성 문제를 일으킨다. 또한 가상 상속이 필요해질 수 있다.
- 가상 상속은 크기, 속도 비용이 늘어나며, 초기화 및 대입 연산의 복잡도가 커진다. 
- 가상 기본 클래스는 데이터를 두지 않는것이 실용적이다.
- 다중 상속을 적법하게 쓰는 경우 -> 인터페이스 public 상속, 구현 private 상속

**Example**  

▶ 다중 상속의 모호성

```c++
class A {
private: //private 멤버 함수여도 상관 없이 모호성 발생합니다.
	bool print() {
		cout << "class A print" << endl;
		return true;
	}
};

class B {
public:
    void print() {
		cout << "class B print" << endl;
	}
};

class MI : public A, public B {};

int main() {
	MI mi;
	mi.print(); //class A print 함수인지 B print 함수인지모호성 발생.
	mi.B::print(); //명시적으로 지정해 주어야 한다.
}
```



▶ 가상 상속

```c++
class A {...}; //가상 기본 클래스

class B :virtual public A {...}; //가상 상속 B, C

class C :virtual public A {...}; //중복 데이터가 파생 클래스에 상속 되는것을 막아줍니다.

class MI :public B, public C {...}; //MI 
```



▶ 올바른 다중 상속 - 인터페이스 public 상속, 구현 private 상속

```c++
class IBird { //인터페이스
public:
	virtual ~IBird() {}
	virtual void fly() = 0;

	static std::tr1::shared_ptr<IBird> create(std::string name);
};

class BirdInfo { //구현
public:
	virtual ~BirdInfo() {}
	virtual void Infofly(std::string name) {
		std::cout << name << " flying" << std::endl;
	}
};

class Bird :public IBird, private BirdInfo {
private:
	std::string _name;
public:
	Bird(std::string name) : _name(name) {}
	void fly() override { //인터페이스 함수
		BirdInfo::Infofly(_name); //구현부 호출
	}
};

std::tr1::shared_ptr<IBird> ibird = IBird::create("Penguin");
ibird->fly();
```



결과창

```c++
Penguin flying
```




