---
layout: post
title: Android 모듈화 with 'Now in Android'
subtitle: Android modularization with 'Now in Android'
published: true
---

Now in Android, Modularization  
=============  
[Now in Android repository](https://github.com/android/nowinandroid)<br><br>

Google에서 제작 중인 안드로이드 개발용 샘플 App으로,  
최신 앱 아키텍처에 따라 개발되고 있어 학습에 유용하다.
​
## Now in Android를 통해 알수 있는 것
- Kotlin, Coroutines, Flow, Compose, Material 3
- Official App Architeture [https://developer.android.com/topic/architecture](https://developer.android.com/topic/architecture)
- App Modularization [https://developer.android.com/topic/modularization](https://developer.android.com/topic/modularization)
- Hilt
- Unit Test
- gradle version.controller of Dependency
- build-logic instead of buildSrc

<br>

## Android의 권장 아키텍처

<br>  

<img src="https://user-images.githubusercontent.com/48265129/229345633-4fc8d412-6b1a-4600-ae10-1d27ac8823ea.png" width="50%" height="auto">
<img src="https://user-images.githubusercontent.com/48265129/229345655-f97c89d3-c5ad-4dac-9995-4ffd6f3b1ea1.png" width="50%" height="auto">

<br>

- UI Layer : View, ViewModel로 이루어짐  
* Data Layer : Repository, DataSource로 이루어짐  
+ Domain Layer : Clean architecture를 따라서 UseCase를 구성하기도 함  

## Modularization 이유  
**개발 성능 향상(빌드 속도), 코드 분할(의존성 최소화)이 큰 목적**
- App의 크기가 커지면서, 개발자들이 각자의 모듈을 만들어서 개발을 진행하게 됨
- 이러한 모듈들을 하나의 App으로 묶어서 배포하기 위해 필요함
- 기존은 Package단위로 분할 -> Module단위로 분할
- 수정하지 않은 모듈은 re-build가 불필요  

## Now in Android의 아키텍처

<br>


<center><img src="https://user-images.githubusercontent.com/48265129/229345715-6c3b92e9-9b60-4266-848c-98dba85ce880.png" width="80%" height="auto"></center>
<center><img src="https://user-images.githubusercontent.com/48265129/229345718-cd72f34e-f9e3-4de8-b2d4-0d29c57ce3e2.png" width="80%" height="auto"></center>

<br>

## Now in Android의 모듈 구조

<br>


<center><img src="https://user-images.githubusercontent.com/48265129/229345719-34cf9922-94cd-46ce-9f24-3e038b3a3e24.png" width="40%" height="auto"></center>

<br>

## 크게 2개의 앱으로 분리됨  
각각 필요한 모든 모듈을 조합하는 형태  
* app : Main 앱
* app-nia-catalog : UI 확인용 앱  

* core- : 베이스 코드를 가짐
   * common
   * network
   * model : 전체에서 공통으로 사용됨
   * ui
   * testing
  
* feature- : 화면, 기능별로 모듈을 나눔
   * author
   * bookmarks
   * foryou
   * etc..

* etc : 기타 모듈
   * lint
   * sync : CoroutineWorker를 사용한 Sync 모듈
   * build-logic : buildSrc 대체  

## 모듈 별로 가진 파일  
* core - common
  - DispatcherModule.kt : CoroutineDispatcher를 제공하는 모듈
  - Api result 에 대한 파일
    - sealed interface
    - flow 확장 함수

* core - network
  - network
    - retrofit
    - Api 정의
  - 응답 데이터 클래스

* core - model : 사용되는 모든 데이터 model 집합,  
view-viewmodel 사이의 데이터도 포함

* core - database : room 사용,  
  내부의 entity는 room에만 사용됨,  
  외부에서 사용되는 데이터는 core-model에 정의(core-data에서 entity복사를 수행)

- core - datastore
    - Datastore-protobuf 사용(Datastore preference로 대체가능)

- core - data
    - 응답 model을 entity로 변환
    - Repository 정의
    - Repository 에서 Remote(network, 온라인), Local(database, datastore, 내부)을 사용  
  
- core - design system
    - component, icon, theme
    - Compose를 사용한 디자인 시스템
    - Animation, 내부 UI 활용을 위한 Mapping
    - Compose 사용 시에는 Mapping 함수를 미리 만들어 두는 것이 좋음

* core - ui
    - ui 모듈에서 공통으로 활용하는 ui mapping을 추가
    - core-designsystem, core-model을 활용

* core - navigation
    - navigation router 처리를 위한 interface

* feature
    - Compose 기반의 ui 모듈
    - feature 에서 navigation 기반의 작업
    - main의 navhost에서 feature 모듈을 사용

**core모듈 간의 의존성 : 모든 모듈은 core-data에서 활용됨**  
**UI 모듈 간의 의존성 : core-model을 활용**  

## One Activity 구성은 필수인가?  
>One Activity 구성은 메모리 관리, Lifecycle 등에 대해서 많은 고민이 필요하다.  
>좋은 설계가 매우 중요

* 모듈은 단독으로 동작 가능한 화면이 존재 가능
    - Activity 모듈로 만들면, 바로 실행가능하게 코드 분리가 가능
* Hilt와 같은 라이브러리로 분리된 모듈들을 App으로 조합가능
* One Activity의 Lifecycle은 App의 Lifecycle과 같을 수 있음
* Compose navigation Lifecycle과 Activity Lifecycle은 다름

## 모듈 간 화면 전환, 통신 방법
Navigation을 사용, Arguments or Repository 를 통해 데이터 전달  

## 모듈이 많아지는 경우 전략  
* 화면마다 feature를 나눌 것 인가?
* 공통 코드는 core에 담을 것 인가?
* 폴더 구조를 잘 구성해서 Project형태로 펼쳐서 볼 것 인가?

## 폴더 구조를 구성해서 Project 형태로 나누기
### 기존 구조
core-common, core-data, core-data-test, core-database 등 모듈 폴더 들이 nowinandroid 폴더 안에 모두 존재  

* nowinandroid
  * core-common
  * core-data
  * core-data-test
  * core-database
  * core-designsystem
  * etc..

__개수가 많아 가독성이 낮고, 복잡__  

### Project 형태로 나눈 새로운 구조 

core, feature 폴더를 구분  
**단순히 폴더 구조를 나누기만 하면 빌드 불가!, 추가적으로 설정이 필요**  

* nowinandroid
  * core
    * core-common
    * core-testing
    * model
      * core-data
      * core-database
      * core-datastore
      * core-model
      * etc..
    * ui
      * core-designsystem
      * core-ui
      * etc..
  * feature
    * feature-author
    * feature-bookmarks
    * feature-foryou
    * etc..

### settings.gradle.kts에 코드 추가한 뒤 빌드 가능  

```kotlin
// Create a map to store the name of the module and its path
val modules = hashMapOf<String, String>()

// Recursively search for all build.gradle.kts files
rootProject.projectDir.listFiles()?.forEach {
    findSubProjects(it)
}

// Function to find subprojects
fun findSubProjects(file: File) {
    // Skip if the file is a hidden file
    if(file.name.startsWith(".")) {
        return
    }

    // If the file is a build.gradle.kts file, store the module name and path
    if(file.name == "build.gradle.kts") {
        modules[name] = file.parentFile.path
        return
    }

    // If the file is a directory, recursively search for build.gradle.kts files
    if(file.isDirectory) {
        file.listFiles()?.forEach {
            findSubProjects(it)
        }
    }

}

for(project in rootProject.children) {
    // Get the project name from the root project
    if(modules.containsKey(project.name)) {
        // If the project name is in the list of modules
        val directory = modules[project.name] ?: continue
        // Get the directory from the list of modules
        project.projectDir = File(directory)
        // Set the project directory to the module directory
    }
}
```  

**코드 작성하고 저장 -> Sync gradle**  

## build.gradle 파일 관리가 중요  

**모듈을 생성하면서 build.gradle의 중복 코드가 증가!**  

### gradle plugins로 관리  
  
plugins에 id("xx") 로 추가해 관리하자

```kotlin
plugins {
    id("nowinandroid.android.library")
    id("nowinandroid.android.feature")
    id("dagger.hilt.android.plugin")
}
```

### buildSrc vs build-logic  
**둘의 차이는 거의없다**  
**익숙하고 편한거 사용하면 됨**

build-logic은 kotlin으로 작성이 가능하다는 차이만 있는 정도  

[Gradle - build-logic vs buildSrc 알아보기]()  

**[본 내용은 이 슬라이드의 내용을 바탕으로 작성되었습니다](https://speakerdeck.com/taehwandev/android-module-gaebal-now-in-android-camgo)**