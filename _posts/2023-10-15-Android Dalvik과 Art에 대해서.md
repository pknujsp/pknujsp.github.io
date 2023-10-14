---
layout: post
title: Android Dalvik과 ART에 대해서
subtitle: Dalvik, ART
published: true
categories: [Dalvik, ART, Android]
tags: [Dalvik, ART, Android]
---

## Dalvik 이란?

> Android에서 Java 애플리케이션을 구동하는 가상머신(Virtual Machine)이다.

**Java**의 라이센스 문제(오라클, 이 문제로 10년간 소송)와 최적화 등의 이유로 인해, 구글은 **JVM**을 사용하지 않고, 자체적으로 **DVM(Dalvik Virtual Machine)**을 개발하였다.


### 특징

- JIT(Just-In-Time)
  - **Android 2.2(2011)**이전에는 **JIT** 컴파일이 없어, 앱이 실행 될 때마다 컴파일을 하여 메모리 효율이 나빴고 앱 구동 성능이 좋지 않았다.
  - **Android 2.2(2011)**부터 **JIT** 컴파일이 도입되었다.
  - 자주 사용되는 코드는 캐싱하여 재 사용하기 때문에 전반적으로 성능이 향상되었다.
- .dex(Dalvik Executable, Dalvik Bytecode)
  - 앱 빌드 시 **.class** 파일을 **.dex** 파일로 변환한다.
  - `.dex` 파일을 **Dalvik**에서 실행한다.
- CMS(Concurrent Mark Sweep) Garbage Collector 사용
  - ![Java Garbage Collection](https://d2.naver.com/helloworld/1329)
  - GC동작 시 스레드가 멈추는 시간이 다른 알고리즘들 보다 짧다.
- 메모리 효율 향상
  - **레지스터 기반**이기 때문에 **스택 기반**인 **JVM**보다 메모리를 적게 사용한다.


Android 기기의 보급이 늘어나기 시작했던 2010년 즈음에는 메모리의 용량이 요즘 나오는 고성능 기기의 1/10 수준(512MB 이하)으로 매우 작았다.

이러한 상황에서 메모리 사용을 최적화하고자, **Dalvik**은 **JIT(Just-In-Time)** 컴파일 형식을 사용하여 앱이 실행되고 있을 때 필요한 부분만 컴파일한다.

필요한 것만 처리하기 때문에 메모리 절약이 가능한 것이다.

그러나, 실행 중에 컴파일을 하기 때문에 앱의 성능에 영향을 미칠 수 밖에 없는 단점이 있었다.

시간이 흐르면서 **애플리케이션의 규모가 커져감**과 동시에 **하드웨어의 성능이 향상**되어갔고 **Dalvik**을 대체할 **ART**가 도입되었다.

## ART(Android Runtime)의 도입

> Android 4.4에서 처음 도입된 Runtime

**Dalvik**과 달리 **AOT** 방식을 사용한다.

앱을 설치할 때 바이트 코드인 **.dex**파일들을 기계어로 컴파일하여 **.oat**파일들로 저장한 후 앱 실행 시 **.oat**파일을 불러와서 실행한다.


### 특징

- AOT(Ahead-Of-Time)
- .oat(Optimized Ahead-of-Time)
  - **ART**에서 사용하는 파일
  - dex2oat도구를 통해 **.dex**를 **.odex**로 변환한 후 다시 **.oat**로 변환한다.
- .odex(Optimized Dalvik Executable)
  - **Dalvik**에서 사용하는 파일
  - dexopt도구를 통해 **.dex** 파일을 **.odex** 파일로 변환한다.
  - 앱 실행 시 이 파일을 기계어로 변환한다.
- Garbage Collector에서 상당한 개선
  - **Dalvik**보다 처리가 2배 빠르다.

### Dalvik과 ART의 비교

간단히 각각의 장단점을 정리하면 다음과 같다.

|              | Dalvik |  ART  |
| :----------: | :----: | :---: |
| 앱 설치 속도 |  빠름  | 느림  |
| 앱 실행 속도 |  느림  | 빠름  |
| 앱 설치 용량 |  적음  | 많음  |

서로 장단점이 반대이다.

그래서 시간이 지나면서 런타임에 여러 번 변경이 있었다.

### Android에서 ART의 적용 역사

1. **Android 4.4(2013)**에서 처음 도입(기본 설정은 **Dalvik**), 호환성 문제
2. **Android 5.0(2014)**에서 기본 Runtime으로 적용
3. **Android 7.0(2016)**부터 **JIT**와 **AOT**를 혼합하여 사용한다

**Android 7.0(2016)**에서 다시 **JIT**를 사용하는 `Profile Guided Compilation` 방식이 도입되었다.

#### Profile Guided Compilation

> **JIT**와 **AOT**를 혼합하여 컴파일을 수행

- 앱 설치 시 **AOT**컴파일 없이 수행되도록 바뀌었고 이로인해 설치 속도가 빨라졌다.
- 앱이 실행되면 **JIT** 컴파일을 수행하여 자주 사용되는 코드는 캐싱된다.
- 기기가 유휴 상태 또는 충전 중일 때 자주 사용되는 코드룰 **AOT**컴파일 한다.
- 이후 앱이 실행될 때 **AOT**컴파일된 코드를 사용한다.