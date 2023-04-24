---
layout: post
title: Compiler Context-sensitive analysis
subtitle: Semantic analyzer in Parser
published: true
categories: Compiler
tags: [Compiler]
---

# Semantic analyzer(Context-sensitive analysis, 문법 분석기)
>이전 두 개의 분석기(Lexical, Syntex analyzer) 보다 더 상세한 부분을 분석한다.  
>변수 선언, 자료형 등의 깊은 정보를 분석한다.

이 분석기에서는 **types**, **scopes**가 중요하다!

## Type systems
---

* 공통 타입은 integer, list, character를 포함한다.

* Type system의 구성
  * 기본(Primitive) 타입
    * boolean, char, integer, real, etc..
  * 구조화된 타입
    * Arrays, Strings, Records and structures, Pointers

### Type systems의 목적
---
>1. 런타임 안전성을 보장

일부 언어에서 컴파일러는 모든 표현에 대한 타입을 추론하지 못한다.  

- Strongly typed language
  - Statically typed
    - 컴파일 시간에 모든 표현의 자료형이 정해진다.
    - C, C++ 등
  - Dynamically typed
    - 런타임 시간에 일부 표현의 자료형이 정해질 수 있다.
      - Python, Js 등
- An untyped language
  - 자료형 지원이 약한 언어
  - 어셈블리


>2. 표현력 향상

- 잘 구성된 자료형은 언어 개발자가 Context-free 규칙으로 가능한 것 보다 더 정확하게 행위를 지정할 수 있게 한다. 
- Context-free 문법으로 지정할 수 없는 기능을 만들 수 있다.
  - 연산자 오버로딩

>3. 더 나은 코드 생성

컴파일 시간에 결정될 수 없는 자료형을 가지는 언어에서는, 런타임까지 자료형 확인 일부가 연기 될 수 있다.

>4. 자료형 확인

런타임 시 자료형 확인에 대한 오버헤드를 피하기 위해, 컴파일러는 프로그램을 분석하고 각 이름과 표현에 대해서 자료형을 할당해야 한다.


## 자료형 호환성
---
>자료형 확인은 자료형이 동등한지 판별해야 한다.

* 두 가지 접근법이 있다.
  * 이름 동등성
    * 각 자료형의 이름이 고유하다.
    * 예제
      * next, last는 같은 자료형이다.
      * p, next는 다른 자료형이다.
  * 구조적 동등성
    * 서로 같은 구조를 가질 때, 두 자료형은 동등하다.
    * 예제

  **구조적 동등성 예제**

  ```
  type link = cell;
  var next : link;
  last : link;
  p : cell; // last, p는 구조적 동등성으로 판단 가능하다.
  q, r : cell; // q, r은 이름 동등성으로 판단 가능하다
  ```

## Scope Information
---
>Scope information은 식별자 선언과 허용된 각 식별자가 프로그램에서 사용되는 부분에 대해서 **문자화**한다.

* Lexical scope는 프로그램 내에서 텍스트 형식의 영역이다.
  * Statement block
  * Formal argument list
  * Object body
  * Function or method body
  * Module body
  * Whole program(Multiple modules)

* 식별자의 scope
  * 식별자가 유효한 Lexical scope


## Symbol tables
---

* Semantic checks 는 프로그램 내 식별자의 속성을 따른다.
  * 식별자의 Scope와 Type
* 식별자 정보를 저장할 환경이 필요하다.
  * **Symbol table**을 사용한다.
  * Hashmap을 사용한다.

* 구성
  * 식별자 명
  * 추가 정보
    * 종류 : fun, var, parameter ~
    * 자료형 : int, bool ~
    * 상수 여부

### Scope 정보를 나타내는 방법
---

* 프로그램 내 Scope의 계층이 있다.
* 유사한 계층 구조를 Symbol tables로 사용한다.
* Symbol table은 Scope마다 연결된다.
* 각 table은 Lexical scope에서 선언된 기호들을 포함한다.

### Table 내 데이터 처리 방법
---

1. Create
   - 주어진 부모 테이블에 따라 빈 Symbol table을 만든다.
2. Insert
   - 새로운 식별자를 추가한다.
3. Lookup
   - 식별자를 조회한다.

Lexical 분석 중에는 Symbol table을 만들 수 없다.
- Scope 계층이 구문 내에 인코딩되기 때문이다.

테이블을 만드는 시점
- Semantic actions로 파싱을 하고 있을 때
- AST가 구성된 이후에


## Knuth's attribute grammers
---
>트리 속성을 기반으로 하는 문법이다.

- 값 할당은 production과 연관된다.
- 각 속성은 고유하고, 지역적으로 정의된다.
- 라벨 식별 용어도 고유하다.

* Syntethic attributes
  * 자식들로 부터 수행된 값
  * val, neg
* Inherited attributes
  * 형제와 부모로 부터 수행된 값
  * pos


## Attribute dependence graph
---
>사이클이 없는 그래프여야 한다!

속성을 정렬하기 위해 그래프를 위상 정렬로 처리한다.