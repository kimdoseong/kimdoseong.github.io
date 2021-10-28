---
title:  "Strategy Pattern"
excerpt: "전략 패턴"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "Design Pattern"
tags:
 - [C++, Design_Pattern_행태]

date: 2021-07-23
last_modified_at: 2021-07-23

published: true
---

- 알고리즘과 상세 구현을 분리하여 구현한다. 

- 알고리즘을 정의하고 필요할 때 변경하여 사용할 수 있게 한다. 

- 동적형태와 정적형태가 있다. 


## <span style="color:#8F7CEE"> 동적 전략 </span>

▶런타임 타입에 변경 가능한 동적 전략 패턴

```c++
enum class StrategyFormat {
	StrategyA,
	StrategyB,
	StrategyC
}; //전략 패턴 클래스 Format

class Strategy {
public:
	virtual void start(ostringstream& oss) {};
	virtual void addListItem(ostringstream& oss, const std::string& item) {};
	virtual void end(ostringstream& oss) {};
};

class StrategyA :public Strategy {
public:
	void start(std::ostringstream& oss) override { oss << "StrategyA Start" << endl; }

	void end(std::ostringstream& oss) override { oss << "StrategyA End" << endl; }

	void addListItem(std::ostringstream& oss, const string& item) override { oss << "StrategyA Item: " << item << endl; }
};

//... B, C class도 override 해줍니다.

class Context {
private:
	unique_ptr<Strategy> _pStrategy; //unique 포인터
	ostringstream oss;
public:
	void appenListProcess(const vector<string> list) {
		_pStrategy->start(oss);
		for (auto& item : list)
			_pStrategy->addListItem(oss, item);
		_pStrategy->end(oss);
	}

	void setStrategyFormat(const StrategyFormat sf) {
		switch (sf) { //Strate Format 형식에 따라 구현을 나눕니다.
		case StrategyFormat::StrategyA:
			_pStrategy = make_unique<StrategyA>();
			break;
		case StrategyFormat::StrategyB:
			_pStrategy = make_unique<StrategyB>();
			break;
		case StrategyFormat::StrategyC:
			_pStrategy = make_unique<StrategyC>();
			break;
		}
	}

	string str() {
		return oss.str();
	}
    
    void clear() {
		oss.str("");
		oss.clear();
	}
};

//사용
Context ct;
ct.setStrategyFormat(StrategyFormat::StrategyA); 
ct.appenListProcess({ "a", "b", "c" });
cout << ct.str() << endl;

ct.clear(); //버퍼 비워줍니다.
ct.setStrategyFormat(StrategyFormat::StrategyB); //전략 B class로 변경
ct.appenListProcess({ "1", "2", "3" });
cout << ct.str() << endl;
```

```c++
StrategyA Start
StrategyA Item: a
StrategyA Item: b
StrategyA Item: c
StrategyA End

StrategyB Start
StrategyB Item: 1
StrategyB Item: 2
StrategyB Item: 3
StrategyB End
```

## <span style="color:#8F7CEE"> 정적 전략 </span>

▶컴파일 타임에 결정되는 정적 전략 패턴

조금 더 간단하긴 한데 Context 인스턴스가 목록 랜더링 만큼 개별적으로 가져야 한다.  

```c++
//enum class 삭제

//ConText class 구조만 조금 바뀝니다.
template<typename T> //tempate 사용
class Context {
private:
	T _pStrategy; //T로 변경
	ostringstream oss;
public:
	void appenListProcess(const vector<string> list) {
		_pStrategy.start(oss);
		for (auto& item : list)
			_pStrategy.addListItem(oss, item);
		_pStrategy.end(oss);
	}

	//void setStrategyFormat(const StrategyFormat sf) {...} //type T로 받습니다.

	string str() {
		return oss.str();
	}
};

Context<StrategyA> cta; //template 호출
//Format 따로 설정하지 않습니다.
ct.appenListProcess({ "a", "b", "c" });
cout << ct.str() << endl;

Context<StrategyB> ctb; //B로 변경
ctb.appenListProcess({ "1", "2", "3" });
cout << ctb.str() << endl;
```

결과창

```c++
StrategyA Start
StrategyA Item: a
StrategyA Item: b
StrategyA Item: c
StrategyA End

StrategyB Start
StrategyB Item: 1
StrategyB Item: 2
StrategyB Item: 3
StrategyB End
```

