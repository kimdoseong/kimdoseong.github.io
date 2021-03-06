---
title:  "Effective_C++_Chapter 2"
excerpt: "생성자, 소멸자 및 대입 연산자"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++, Effective_C++]


date: 2021-06-28
last_modified_at: 2021-06-28

published: true
---

## <span style="color:#8F7CEE">1. 컴파일러는 경우에 따라 클래스에 대해 기본 생성자, 복사 생성자, 복사 대입 연산자, 소멸자를 암시적으로 만들어 놓을 수 있다 </span>

## <span style="color:#8F7CEE">2. 컴파일러가 만들어낸 함수가 필요 없다면 사용하지 않게 만들자 </span>

- 대응되는 멤버 함수를 private로 선언한 후에 구현은 하지 않은 채로 두자.  
- Uncopyable 같은 기반 클래스를 쓰는 것도 가능.  

**Example**  
▶private에 선언 or Uncopyable 클래스 상속  

```c++
class Uncopyable {
protected:
    Uncopyable() {}
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&) {}
    const Uncopyable& operator=(const Uncopyable&) {}
};

class Derived : private Uncopyable {
private:
    //Derived(const Derived&){} /상속받기 싫다면 private 안에 가둬두면 됩니다.
    //const Derived& operator=(const Derived&){}
public:
    Derived(){}
};


int main(int argc, const char* argv[]) {
    Derived a;
    Derived b;
//    a = b;    //error 발생. 복사 함수들이 기본 클래스에서 공개되지 않음.

    return 0;
}
```

## <span style="color:#8F7CEE">3. 다형성을 가진 기반 클래스에서는 소멸자를 반드시 가상 소멸자로 선언하자  </span>

- 생성자 호출 순서 = 기반 클래스 -> 파생 클래스  
- 소멸자 호출 순서 = 파생 클래스 -> 기반 클래스  
- 다형성을 갖도록 설계된 클래스는 가상 소멸자를 선언해야 한다. (가상 함수를 하나라도 갖고 있으면)  

**Example**  
▶ 아래 처럼 기반 클래스로 파생 클래스를 할당받고 가상 함수를 선언하지 않은 경우   

```c++
class Base {
public:
    Base() { cout << "Base 생성자" << endl; }
    /*virtual*/ ~Base() { cout << "Base 소멸자" << endl; }
    //virtual 키워드를 꼭 붙여줘야 합니다.
};

class Derived : public Base {
public:
    Derived(){ cout << "Derived 생성자" << endl; }
    ~Derived(){  cout << "Derived 소멸자" << endl; }
};


int main(int argc, const char* argv[]) {
    Base *a = new Derived();
    delete a; //메모리 누수 발생.

    return 0;
}
```

**결과창**  
Base 생성자  
Derived 생성자  
Base 소멸자  

Derived 소멸자가 호출되지 않아 메모리 누수가 생깁니다..  

위의 코드에서 이렇게 기반 클래스에 virtual 키워드만 붙여주게 되면   
```c++
virtual ~Base() { cout << "Base 소멸자" << endl; }  
```


**결과창**  
Base 생성자  
Derived 생성자  
Derived 소멸자  
Base 소멸자      

Derived 소멸자까지 제대로 호출됩니다.   

## <span style="color:#8F7CEE">4. 예외가 소멸자를 떠나지 못하게 하자  </span>

- 소멸자에서 예외가 빠져나가면 안된다.   
- 소멸자에서 프로그램을 종료 or 예외 삼키기.  
- 예외가 발생할 수 있다면 함수로 미리 정의해두자.    

**Example**  
▶소멸자에서 예외 처리  
```c++
class Manager {
public:
    Manager(){}
    ~Manager(){ }
    void close() {} //사용자가 직접 호출하여 에러를 처리할 수 있게함.
};

class Exception {
private: Manager mg;
public:
    Exception() {}
    ~Exception() {
        try { mg.close(); }
        catch(...) {
            abort(); //프로그램 종료 혹은
            //cout << "mg Close Fail" << endl; //실패했다는 로그 둘중에 하나
            //소멸자에서 처리를 하여 미정의 동작으로 빠지지 않게 한다.
        }
    }
};
```


예외가 발생한다면 프로그램을 종료하던 로그를 남기던 소멸자에서 처리를 하여 미정의 동작으로 빠지지 않게 한다.  
또한 함수를 미리 정의해두어 사용자가 호출하여 예외를 처리할 수 있게 해 준다.  

## <span style="color:#8F7CEE">5. 객체 생성, 소멸 과정 중에는 절대로 가상 함수를 호출하지 말자  </span>

- 생성자 혹은 소멸자 안에서 가상 함수를 호출하면 안된다.  
- 기반 클래스 생성자는 파생 클래스 생성자보다 앞서 실행되기 때문에 기반 클래스에서 어쩌다 파생 클래스 함수를 호출하게 되면 초기화 되지 않은 데이터 멤버를 건드리게 되어 미정의 동작으로 빠지게 된다.  
- 위와 같은 이유로 가상 함수가 생성과 소멸이 진행되는 동안 파생 클래스 쪽으로 내려가지 않기에 그런 설계를 하지말자.  

**Example**  
▶아래 처럼 가상 함수를 생성자 혹은 소멸자에서 호출하지 말자  
```c++
class Base {
public:
    Base() {
        logTransaction(); //생성자 or 소멸자에서 가상 함수를 호출하면 안됩니다.
    }
    virtual ~Base() {}
    
    virtual void logTransaction() = 0;
};

class Derived : public Base {
public:
    Derived() {}
    ~Derived() {}

    void logTransaction() {}
};
```

## <span style="color:#8F7CEE">6. 대입 연산자는 *this의 참조자를 반환하자  </span>

- 연산자에서 *this를 반환하자.  

**Example**   
▶*this 반환   

```c++
class Base {
private:
    string _text;
public:
    Base() : _text() {}
    Base& operator=(const Base& rhs) {
        this->_text = rhs._text;
        return *this; // 참조자를 반환.
    }
};
```

## <span style="color:#8F7CEE">7. operator=을 구현할 때, 객체가 자기 자신에 대입되는 경우를 제대로 처리할 수 있도록 만들자  </span> 
아래 두가지 방법으로 구현할 수 있다.

- 원본과 복사대상 객체의 주소를 비교    
- 복사 후 swap   

**Example**  
▶복사 후 swap하는 예제     
```c++
    Base& operator=(const Base& rhs) {
        Base temp(rhs); //사본을 만들어서
        swap(temp); //*this의 데이터와 바꾸기
        return *this; // 참조자를 반환.
    }
```

## <span style="color:#8F7CEE">8. 객체의 모든 부분을 빠짐없이 복사하자  </span>

- 객체 복사 함수는 해당 객체의 데이터 멤버 및 기반 클래스 부분도 모두 복사하자  

**Example**  
파생 클래스에서 기반 클래스의 데이터를 가지고 있으나, 복사 함수에서 기반 클래스 데이터 멤버를 복사 시켜주지 않으면 문제가 생기게 된다.  

복사 생성자의 경우 기반 클래스의 생성자를 호출하여 기본적인 **초기화** 라도 시켜주지만, 복사 대입 연산자의 경우 **초기화** 조차 해주지 않는다.  

▶복사 생성자와 복사 대입 연산자에서 기반 클래스 멤버까지 복사  
```c++
class Derived : public Base {
private:
    int _num;
public:
    Derived(): _num() {}
    Derived(const Derived& rhs) : Base(rhs), _num(rhs._num){} //기반 클래스 복사 생성자 호출, 파생 클래스 멤버 복사.
    Derived& operator=(const Derived& rhs) {
        Base::operator=(rhs); //기반 클래스 복사 대입 연산자 호출
        this->_num = rhs._num;
        return *this;
    }
};
```

