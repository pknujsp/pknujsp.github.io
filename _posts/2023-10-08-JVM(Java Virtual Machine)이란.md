---
layout: post
title: JVM (Java Virtual Machine)이란?
subtitle: JVM
published: true
categories: [JVM]
tags: [JVM]
---

## 가상 머신, JVM이란?

> Java코드를 구동하기 위한 프로그램(컴퓨터)이다.

C와 같은 Native 언어는 컴파일러를 통해 기계어로 변환되어 실행된다. 하지만 Java는 컴파일러를 통해 바로 기계어로 변환되는 것이 아니라, JVM이라는 가상머신을 통해 실행된다. JVM은 Java Virtual Machine의 약자로, Java를 실행하기 위한 가상의 컴퓨터라고 생각하면 된다. JVM은 OS에 종속적이지 않고, Java가 설치되어 있다면 어디서든지 실행할 수 있다.

- Native언어로 개발하면, CPU아키텍처나 운영체제 마다 다른 기계어를 사용하기 때문에, 특정 운영체제나 CPU에 종속적이며, 개발자는 각각의 CPU나 운영체제에 대해서 프로그램을 만들어야 한다.
- 가상 머신에서 동작하는 언어는 가상 머신만 있으면 어디서든지 실행할 수 있으며, 개발자는 CPU나 운영 체제 각각에 대해서 프로그램을 만들 필요가 없다.

## JVM의 동작 과정

1. .java 파일을 javac(컴파일러)를 통해 Bytecode인 .class 파일로 변환한다.
2. .class 파일을 JVM의 클래스 로더에게 전달한다.
3. 클래스 로더는 .class 파일을 JVM의 메모리에 로드한다.
4. 로드된 .class 파일은 Execution engine을 통해 해석된다.
5. 해석된 Bytecode는 Runtime Data Area에 배치되어 실질적인 수행이 이루어진다.

.java -> .class -> JVM -> 기계어 -> 실행

## JVM의 구성

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*slIuYO633BCuBh_gfYRmGg.png)


- Class Loader
  - .class 파일을 불러와서 JVM 메모리에 탑재하는 역할을 수행한다.
- Execution Engine
  - 내부에 Interpreter, JIT Compiler, Garbage Collector를 포함
  - class를 실행한다.
  - class loader에 의해 JVM 내의 Runtime Data Area에 배치된 바이트 코드를 실행한다.
- Runtime Data Area
  - 프로그램을 동작하기 위해 OS에서 할당받은 메모리 공간
  - Heap, Stack, Method, Native Method Stack, PC Register, Native Method Stack 등으로 구성
  - Heap
    - 실행 시 동적으로 할당된 객체를 저장하는 공간
  - Stack
    - 지역변수, 매개변수, 메서드, 임시 데이터 등을 저장하는 공간
    - 메서드 호출 시 내부에 공간이 생성되며, 메서드가 종료되면 그 공간은 사라진다.
  - Method
    - 클래스, 인터페이스, 메서드, 필드, static 변수 등의 바이트 코드를 보관하는 공간
  - PC Register
    - Thread가 시작될 때 생성되며, Thread가 어떤 부분을 어떤 명령으로 실행해야 할지에 대한 기록을 하는 공간
  - Native Method Stack
    - 실제로 구동되는 기계어로 작성된 프로그램을 실행하기 위한 공간
    - JNI(Java native interface)를 통해 호출되는 C코드를 처리하는 공간


### JIT Compiler, AOT Compiler

- JIT(Just-In-Time, Dynamic translation)
  - 런타임에 Bytecode를 기계어로 컴파일
  - 장점
    - 프로그램 설치 속도가 빠르다
  - 단점
    - 실행 속도가 느릴 수도 있다
    - 런타임 시 계속 컴파일을 하기 때문에 CPU사용량이 AOT 대비 높다
    - 프로그램 실행 시 많은 양의 코드를 한번에 메모리에 탑재하기 때문에 메모리 사용량이 높다
  - 종류
    - JVM(Java), CLR(C#), CPython(Python)
- AOT(Ahead-Of-Time)
  - 런타임 이전(설치 시)에 Bytecode를 미리 기계어로 컴파일
  - 장점
    - 더 빠른 실행 시간을 가져온다.
  - 단점
    - JIT사용 대비 프로그램 설치 속도가 느려진다


두 방식은 각각 장단점이 있으며, 최신 Android OS의 경우 JIT와 AOT를 혼합한 방식을 사용한다.

다음 포스팅에서는 JIT와 AOT를 중점으로 Android의 Runtime에 대해 알아보겠다.