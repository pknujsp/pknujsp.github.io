---
layout: post
title: Compiler Overview
subtitle: Compiler 학습 시작
published: true
categories: Compiler
tags: [Compiler]
---

# Compiler(컴파일러) 란?
>입력 : High-level 프로그래밍 언어 => **컴파일러** => 출력 : Low-level 어셈블리 언어
>컴파일러는 어떤 언어로 작성된 프로그램을 컴파일 시간과 런타임 시간에 다른 언어로 번역하는 역할을 한다.

* 컴파일러 동작 시점
  * 컴파일 시간(정적, 프로그램 실행 이전)
  * 런타임 시간(동적, 프로그램 실행 중)
  * 또는 컴파일과 런타임 시간 동시에

* 컴파일러 역할
  * 프로그램을 읽고 이해한다.
  * 필요한 작업을 정확하게 결정하고, 그 동작을 수행하는 방법을 알아낸다.
  * 컴퓨터가 그러한 작업을 하도록 한다.

## 컴파일러의 중요성
---
>아키텍처, 시스템, 프로그래밍 방법론, 언어 설계와 매우 밀접하게 상호 연결되어있다.

## 좋은 컴파일러의 특징
---
* 올바른 코드를 생성한다.
* 많은 작업을 빠르게 수행한다.
* 번역할 프로그래밍 언어의 모든 기능을 다룰 수 있다.
* 압축된 코드를 제공한다.
* 수정한 코드만 컴파일 할 수 있다.
  * 해당 부분만 컴파일 한 후 이전에 컴파일된 다른 코드들과 연결시켜 주면 된다.
* 프로그래머는 컴파일러가 복잡한 것을 모르게 해야 한다.
* 복잡한 상호작용을 완벽하게 처리한다.
  * 코드의 오류를 진단하는 것


## 컴파일러의 기본적인 구조
---
>Frontend - Optimizer - Backend, 크게 3개의 구성으로 이루어진다.

![컴파일러 기본 구조](https://user-images.githubusercontent.com/48265129/233780957-b2de271c-ceba-46d9-90cb-3afe5b0165d3.png)

* Frontend
  * Scanner
    * Lexical analyzer
  * Parser
    * Syntax analyzer
    * Semantic analyzer
* Optimizer
  * Code optimizer
* Backend
  * Code generator

## Frontend
---
>소스 코드를 분석하여, 올바른 문법의 코드를 IR로 바꾼다.

* 특징
  * 코드의 문법이 올바른지 확인한다.
  * 문법 오류를 알려준다.
  * IR과 예비 스토리지 Map을 생성한다.
  * 백엔드를 위한 코드를 구성한다.
  * 대부분의 프론트엔드 설계는 자동화 될 수 있다.

### Scanner
---
>소스 코드를 token 단위로 분리 한다. 코드 character stream을 words로 매핑하는 것이다.

* 특징
  * 토큰
    * 의미있는 부분
    * 괄호, 숫자, 식별자, +, -, new, while, if, comma(,) 와 같은 것
  * 공백(주석 포함)을 제거한다.

- 예제
  - 234 * (56 + -79)
    - 234 : 숫자
    - \* : 곱셈
    - ( :  왼쪽 괄호
    - 56 : 숫자
    - \+ : 덧셈
    - -79 : 숫자
    - ) : 오른쪽 괄호
  - 12..34
    - 오류


## Parser
---
>코드 문맥과 무관한(Context-free) 문법과 문맥에 따른(Context-sensitive) 문법을 분석하고, 오류를 보고한다.  
>IR을 생성한다.


### Syntax analyzer(Context-free)
---
>코드의 문맥과 무관한 문법을 분석한다. 데이터 유형 분석없이 **형태만** 분석하는 것이다.

```kotlin
//  ) 가 하나 더 있다
test(start : Int, end :Int, v : Int)): Int {
    var x : Int = 0

    // for가 fr로 되어있다
    // ..가 아닌 ...로 되어있다
    fr(idx in start ... end) {
        // 연산자가 빠져있다
        x  idxx
    }
    
    // return이 아닌 retur으로 되어있다
    retur x
}
```

### Semantic analyzer(Context-sensitive)
---
>코드의 문맥과 관련된 문법을 분석한다. 데이터 유형과 변수 선언과 같은 문법을 분석하는 것이다.

```kotlin
// v의 유형이 정의되어 있지 않다
fun test(start : Int, end :Int, v)): Int {
    // x의 값이 초기화 되어 있지 않다
    var x : Double

    for(idx in start .. end) {
        // idxs는 선언되지 않은 변수이다
        x += idxs
    }
    
    // 반환 데이터 유형이 다르다, Double -> Int
    return x
}
```

## Optimizer
---
>IR을 최적화한다.

### 최적화 예시
```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    var y : Int

    x = 0
    y = 0

    while (i < N) {
        x = x + (4*a/b)*i+(i+1)*(i+1)
        x = x+b*y
        i++
    }
    return x
}
```

**상수를 분석하고, 교체**

```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    var y : Int

    x = 0
    y = 0

    while (i < N) {
        x = x + (4*a/b)*i+(i+1)*(i+1)
        x = x * b*0 // y는 0에서 값이 변하지 않기 때문에 상수이다
        i++
    }
    return x
}
```

**최적화**

```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    var y : Int

    x = 0
    y = 0

    while (i < N) {
        x = x + (4*a/b)*i+(i+1)*(i+1)
        // x = x * b*0는 x = x와 같으므로 교체
        x = x
        i++
    }
    return x
}
```

**쓰이지 않는 코드 삭제**

```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    // 사용되지 않는 y를 삭제

    x = 0
    // 불필요한 y = 0를 삭제

    while (i < N) {
        x = x + (4*a/b)*i+(i+1)*(i+1)
        // x = x는 불필요하므로 삭제
        i++
    }
    return x
}
```

**공통되는 하위 수식 삭제**

```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    var t : Int

    x = 0

    while (i < N) {
        t = i + 1
        // (i+1)*(i+1)를 t*t로 교체
        // 변수 t를 선언
        x = x + (4*a/b)*i+t*t
        i++
    }
    return x
}
```

**불변 반복 코드 교체**

```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    var t : Int
    var u : Int

    x = 0
    u = (4*a/b)

    while (i < N) {
        t = i + 1
        x = x + u*i+t*t
        i++
    }
    return x
}
```

**최적화, 강도를 줄인다**

```kotlin
fun calcSum(a : Int, b : Int, N : Int) : Int {
    var i = 0
    var x : Int
    var t : Int
    //  u = (4*a/b)를 좀더 부하가 적은 비트 마스크 연산으로 교체한다
    var u : Int
    var v : Int

    x = 0
    u = (a<<2/b)
    v = 0

    while (i < N) {
        t = i + 1
        // u*i를 v로 교체
        x = x + v+t*t
        // i를 곱한 값과 같다
        v = v + u
        i++
    }
    return x
}
```

### 최적화 결과 비교

| 처리 | 최적화 전 | 최적화 이후 |
| :---: | :---: | :---: |
| 할당 | 2+9*N  | 3+5*N |
| 곱셈 | 4*N  | N |
| 덧셈 | 4*N  | 4*N |
| 나누기 | N  | 1 |
| 왼쪽 시프트 연산 | 없음  | 1 |

## Backend
---
>IR로 Target code(Machine code)를 생성한다.

* 과정(모든 과정은 IR을 입력으로 받는다.)
  1. 명령어 선택
  2. 레지스터 할당
  3. 명령어 스케줄링

* 특징
  * 각 IR 동작을 구현하기 위한 **명령어**를 선택한다.
  * 레지스터에서 유지할 값을 결정한다.
  * 시스템 인터페이스를 준수하는지 확인한다.
  * 백엔드는 자동화 처리가 매우 어렵다.

### 명령어 선택
---
>IR을 Target 명령어로 매핑한다.

* 특징
  * 주소 모드와 같은 Target 기능을 활용한다.
  * 패턴 일치 문제로 간주된다.
    * ad hoc methods, pattern matching, dynamic programming


### 레지스터 할당
---
>변수를 유한한 레지스터 개수로 매핑한다.

* 특징
  * 레지스터는 사용될 때 값을 가진다.
  * 한정된 자원들을 관리한다.
  * 명령어 선택을 변경한다.
  * 읽기와 쓰기가 가능하다.
  * 최적의 할당 방법은 NP-완전 문제 이다.
  * 컴파일러는 NP-완전 문제에 대한 대략적인 방법(휴리스틱, 경험적인 최적의 방법)이다.


### 명령어 스케줄링
---
* 특징
  * 하드웨어에 과부하가 발생하지 않도록 명령어를 지정한다.
  * 모든 기능 단위를 생산적으로 활용한다.
  * 할당을 변경하면서 변수의 수명을 늘린다.
  * 최적의 스케줄링은 거의 NP-완전 문제 이다.
  * 휴리스틱 기술로 개발된다.