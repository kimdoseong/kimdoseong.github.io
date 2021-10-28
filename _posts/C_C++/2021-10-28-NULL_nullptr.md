---
title:  "NULL/nullptr 차이"
excerpt: "NULL/nullptr 차이"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "C/C++"
tags:
 - [C++]

date: 2021-10-28
last_modified_at: 2021-10-28

published: true
---
## <span style="color:#8F7CEE"> NULL / nullptr 차이 </span>

포인터를 초기화할 때 NULL 대신 nullptr을 사용하는 이유가 뭘까요?  

<img src="/assets/images/C_C++/20211028/0.png" width="50%" height="30%">  

먼저 NULL은 '0' or "(void*) 0"입니다. 그러니까 정수 0일 수도 있고, (void *) 형의 0 이 될 수도 있다는 말이죠.  

뭔가 다재다능한 친구인 것 같지만 그만큼 문제를 일으킬 소지가 있다는 말입니다.  

아래의 코드는 xcode clang 컴파일러에서 확인한 결과입니다.  
<img src="/assets/images/C_C++/20211028/1.png" width="100%" height="90%">  

**결과창**  
<img src="/assets/images/C_C++/20211028/2.png" width="50%" height="30%">  

코드의 12번째 라인을 보면 func 함수를 호출하는 데에 있어서 모호하다는 에러가 발생합니다.  
정수 0일 수도 있고, (void*) 형일 수도 있으니 당연한 말입니다.  

하지만 이와 같은 경우는 다행히 에러를 보여주어 문제를 사전에 방지할 수 있지만  
gcc에서 확인한 결과 에러를 띄우지 않고 매개변수 (int) 함수를 호출합니다.  

만약에 매개변수 (int *)의 함수를 호출하길 원했다면 예상치 못한 문제가 일어날 수 있게 됩니다.  

코드 18, 19, 20 라인은 예상한 결과입니다.  
NULL 과 nullptr 을 비교했을 때 같다는 결과입니다.  

하지만 23 라인은 ERROR가 나옵니다. 22 라인에서 a(0)는 NULL과 같다고 나오지만   
23라인 a(0)는 int형과 nullptr_t형을 비교를 할 수 없다고 합니다..  
 
앞서 말한 것과 같이 NULL은 굉장히 모호한 면이 있습니다. 따라서 NULL 대신 nullptr을 사용해야 합니다.