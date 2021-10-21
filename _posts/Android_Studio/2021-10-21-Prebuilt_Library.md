---
title:  "Prebuild Library"
excerpt: "미리 빌드된 라이브러리 사용"

toc: true
toc_sticky: true
toc_label: "Contents"

categories:
 - "Android_Studio"
tags:
 - [c++, Android_Studio]

date: 2021-10-21
last_modified_at: 2021-10-21

published: true
---

## <span style="color:#8F7CEE"> Android-studio prebuilt so file

안드로이드 스튜디오에서 미리 빌드된 라이브러리 사용하는 방법입니다.  

글 하단의 링크에 Android Studio에서 제공하는 가이드가 있지만 저는 처음 하려니 잘 안되더라구요.. 알고 보면 참 쉬운 건데..  

저와 같은 분은 아래와 같이 따라 하시면 쉽게 되실 겁니다.  

일단 CMakeList나 mk는 모두 cpp 폴더 밑 같은 경로에 있는 전제입니다.  

  
1. CMakeList.txt  

- 먼저 사용하고자 하는 라이브러리를 알맞은 시스템에 맞게 빌드되도록 jniLibs 아래에 만들어 넣어줍니다.  

<img src="/assets/images/Android_Studio/20211021/1.png" width="70%" height="50%">  

- CMakeList.txt file에서 so 파일이 있는 폴더를 포함해 줍니다.  

<img src="/assets/images/Android_Studio/20211021/2.png" width="70%" height="50%">  

- 그 후 target_link_libraries에 lib_Pre를 링크시켜주면 됩니다.   

<img src="/assets/images/Android_Studio/20211021/3.png" width="70%" height="50%">  


  
2. Android.mk  

- CMakeList와 마찬가지로 jniLibs 아래에 만드는 것까지 동일하게 한 후 mk 파일에서 위치를 선언해 줍니다.  

<img src="/assets/images/Android_Studio/20211021/4.png" width="70%" height="50%">  

그리고 참조한 프로젝트에 아래와 같이 연결해 주면 됩니다.  

<img src="/assets/images/Android_Studio/20211021/5.png" width="70%" height="50%">  

  
당연히 미리 빌드된 라이브러리 내의 함수를 사용하고자 하면 해당 헤더 파일을 타겟 프로젝트에 포함시켜 주면 됩니다.  

혹시 so 파일을 못 찾는다고 나오는 경우에는 build.gradle에서 android 밑에 아래와 같이 포함해 주면 됩니다.  

```
   sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jni/jnilibs']
        }
    }
```


Android studio -  "미리 빌드된 라이브러리 사용" 아래 링크 참조.  

참조: https://developer.android.com/ndk/guides/prebuilts?hl=ko

