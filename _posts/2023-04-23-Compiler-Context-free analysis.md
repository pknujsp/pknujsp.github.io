---
layout: post
title: Compiler Context-free analysis
subtitle: Syntax analyzer in Parser
published: true
categories: Compiler
tags: [Compiler]
---

# Syntax analyzer(Context-free analysis, 구문 분석기)
>Scanner에 의해 생성된 토큰을 기본적인 문법(Context-free)에 맞는지를 검사하고, Parse tree를 생성한다.  
>문법에 맞지 않으면 오류를 알린다.

## Context-free grammer(CGF)
---
>모든 생성 규칙이 **V $\rightarrow$ w** 를 따르는 형식적인 문법이다.  
>V : Non-terminal 기호, w : Non-terminal과 Terminal로 구성된 문자열  

* 구성
  * G = (S, NT, T, P)
    * S
      * 시작 기호(심볼)
      * L(G) 내의 문자열 집합
    * NT
      * Non-terminal 기호의 집합
      * syntatic 변수
    * T
      * Terminal 기호의 집합
      * words
    * P
      * 생성 규칙
      * P : NT $\rightarrow$(NT$\cup$T)+


### Terminal/Non-terminal Symbol
---

* Terminal Symbol
  * 언어의 최소 단위
  * 더 이상 분리가 불가능하다.
  * 키워드, 연산자, 숫자, 문자열, 변수명 등
* Non-terminal Symbol
  * 언어의 구조와 문법을 나타내는데 사용된다.
  * 함수 정의, 분기문, 반복문 등 추상적인 개념을 의미한다.


### Context-free grammar의 중요성
---
>프로그래밍 언어의 구문을 설명하기에 강력하다.  
>입력 문자열이 문법으로 만들어 질 수 있는지 여부와 방법을 결정하는 효율적인 파싱 알고리즘을 만들기에 충분히 간단하다.

### CGF를 이용한 예제, SheepNoise
---

S = {SheepNoise}  
NT = {SheepNoise}  
P = {SheepNoise $\rightarrow$ SheepNoise *baa*  , SheepNoise $\rightarrow$ *baa*}  
* 규칙
  1. 표현식은 양의 울음 소리 baa를 추가하여 확장가능 하다.  
  2. 표현식은 단일 양의 울음 소리로 구성가능 하다.  

T = {*baa*}

| 규칙 | Sentential form |
| :---: | :--- |
| - | SheepNoise |
| 1 | SheepNoise *baa* |

| 규칙 | Sentential form |
| :---: | :--- |
| - | SheepNoise |
| 2 | *baa* |

| 규칙 | Sentential form |
| :---: | :--- |
| - | SheepNoise |
| 1 | SheepNoise *baa*  |
| 1 | SheepNoise *baa*  *baa*  |
| ... | SheepNoise *baa*  *baa*  ... *baa*  |
| 2 | *baa* *baa* *baa* *baa* ... |

**Sentential form(문장 형태)**
Terminal form으로 만 이루어져 있다. Derivation 과정에서 생성되는 문자열이다.

### CFG를 나타내기 위한 더 좋은 표기법, BNF(Backus-Naur form)
---
>기호와 표현식의 조합을 사용해서 나타낸다.

```
0 <expr> → <expr><op><expr>
1 | number
2 | id
3 <op> → +
4 | -
5 | *
6 | /
```

### Derivation(유도), Parsing시 가장 중요한 것
---
>문법 규칙을 바탕으로 구체적인 구문이나 구조를 만들어 내는 과정을 의미한다.

Parser는 유도를 통해 올바른 문법으로 작성된 트리를 만드는데, 이 트리는 Semantic analysis와 Code generation 단계에서 사용된다.

* 유도 방법
  * 각 단계에서 규칙에 따라 바꿀 비단말 심볼을 선택한다.  
  * 선택에 따라 다양한 결과(Parse tree)가 나올 수 있다.
  * Leftmost derivation
    * 맨 좌측의 비단말 심볼을 교체한다.
  * Rightmost derivation
    * 맨 우측의 비단말 심볼을 교체한다.

* LHS(Left hand side)
  * 등호나 관계연산자 왼쪽에 위치한 표현식을 의미한다.
  * x = y + z
    * x가 LHS이다.

* RHS(Right hand side)
  * 등호나 관계연산자 오른쪽에 위치한 표현식을 의미한다.
  * x = y  + z
    * y + z가 RHS이다.


심볼은 매 단계마다 생성규칙에 따라 교체되고, 해당 과정을 문자열이 완성될 때까지(비단말을 포함하지 않을 때까지) 반복한다. 

**유도는 재 작성 단계로 구성된다.**

>S $\rightarrow$ $\gamma$<sub>0</sub>$\rightarrow$ $\gamma$<sub>1</sub>$\rightarrow$ $\gamma$<sub>2</sub>$\rightarrow$ ... $\rightarrow$ $\gamma$<sub>n-1</sub> $\rightarrow$ $\gamma$<sub>n</sub> $\rightarrow$ sentence

* $\gamma$<sub>i</sub>는 문장 형태이다.
  * $\gamma$가 단말 심볼만 가진다면, 그 $\gamma$는 L(G)의 문장이다.
  * $\gamma$가 비단말 심볼을 가진다면, 그 $\gamma$는 문장 형태이다.
* $\gamma$<sub>i‒1</sub> 에서 $\gamma$<sub>i</sub>를 얻으려면 A $\rightarrow$ β 를 사용하여 일부 NT A ∈ $\gamma$<sub>i-1</sub> 을 확장합니다.
  * A ∈ $\gamma$<sub>i‒1</sub> 발생을 $\beta$로 교체하여 $\gamma$<sub>i</sub>를 얻습니다.
  * Leftmost derivation에서 첫 번째 NT A ∈ $\gamma$<sub>i‒1</sub>

* 왼쪽 문장 형태는 Leftmost derivation에서 생긴다.
* 오른쪽 문장 형태는 Rightmost derivation에서 생긴다.


### x-2*y 에 대한 두 가지 유도
---

![](https://user-images.githubusercontent.com/48265129/233845715-b9776e57-066b-4e36-9287-5e7d9f31f7d6.png)

두 경우 모두 Expr $\rightarrow$ * id - num * id 로 유도해낸다.

그러나 문장 형태가 다르다.

**Leftmost derivation 결과**
![](https://user-images.githubusercontent.com/48265129/233845855-2c70a937-50df-432a-b100-aac8de56a4f5.png)

**Rightmost derivation 결과**
![](https://user-images.githubusercontent.com/48265129/233845875-d65d8e21-ce41-491f-b8ec-afa1a1e11cfc.png)

## Ambiguity, 모호성
---
>만약 문법이 단일 문장 형태로 부터 여러 개의 유도를 만들어 낸다면, 그 유도 결과들은 애매모호 하다.

**예제**
```
<stmt> ::= if <expr> then <stmt>
                | if <expr> then <stmt> else <stmt>
                | other stmt

if E1 then if E2 then S1 else S2 의 경우
if E1 then (if E2 then S1 else S2)
if E1 then (if E2 then S1) else S2
```

모호성을 제거하기 위해서는 문법을 고쳐야 한다!

**모호성을 제거한 예제**
```
<stmt> ::= <matched>
 | <unmatched>
<matched> ::= if <expr> then <matched> else <matched>
 | other stmt
<unmatched> ::= if <expr> then <stmt>
 | if <expr> then <matched> else <unmatched>
 ```

위의 경우는 Context-free에서의 모호성에 대한 내용이다.  
Context-sensitive에서 모호성을 처리하려면 다음과 같은 부분이 필요하다.  
* 선언, 자료형에 대한 지식
* L(G)의 상위 집합을 허용하고 다른 방법으로 확인하는 것
* 언어 설계가 잘못되었는지 확인하는 것

## Top-down parser
---
* 문장의 구조를 분석하기 위해 구문 요소를 높은 수준에서 낮은 수준으로 차례대로 분해하는 방법이다.
* 일련의 문자열을 의미있는 토큰으로 분해하고 이들로 이루어진 Parse tree를 만든다.
* 트리의 루트(최상단)에서 시작하면서 단말 노드를 만들어 간다.
* 시작 기호 S로 부터 문법 규칙을 바탕으로 좌단 유도에 의해 주어진 문장 W를 찾아간다.
* 좋지 않은 선택을 한 경우에 Backtracking(역추적)을 할 수도 있다.
  * 일부 문법은 역추적이 안될 수도 있다.
* LL parser
  * 일반적인 방법
  * 문법의 규칙을 입력과 일치시키기 위해 왼쪽에서부터 파싱을 해서 좌측유도(Leftmost Derivation) 방식으로 동작한다.


**Backtracking**  
* Top-down parsing 기법
* 역추적, 다시 되돌아가는 방법이다.
* 가능한 모든 문법 규칙에 따라 유도를 시도하며, 매칭되는 규칙을 찾지 못할 때 이전 단계로 돌아가 다른 규칙을 시도하는 과정을 반복한다.
* 단점
  * 한 기호만 확인하면서 진행하기 때문에 오버헤드가 크다.

**Predictive parsing**
* Top-down parsing 기법
* 다음 입력 기호와 파싱 테이블을 기반으로 사용되는 문법 규칙을 예측한다.
* 이러한 예측을 하기 위해서, 파서는 입력 스트림에서 현재 토큰을 전방(look-ahead)으로 검토한다.
* 이 때 전방을 검토하는 기호는 일반적으로 한 개의 토큰까지만 미리보기가 가능한 LL(1) 파서가 사용된다.
* 왼쪽 재귀(Left recursion)가 없고, 각각의 입력 기호에 대해 단 하나의 파싱 규칙만 충족한다.


### Top-down parsing 과정
---

* Parse tree의 생성된 문자열과 입력 문자열이 일치할 때까지 아래의 과정을 반복한다.
  1. 시작 기호에 대해서 생성 규칙을 적용한다.
      * 생성 규칙이 여러 개인 경우, 첫번째 규칙부터 적용한다.
      * 생성 규칙을 적용할 때마다 부분 Parse tree가 구성된다.
  2. 생성된 문장 형태의 문자열과 입력 기호를 차례로 비교한다.
      * 비교 결과 같지 않으면, Backtracking을 한다.
  3. 비교 결과 같으면, 계속 비교한다.

결과가 서로 일치하지 않고, 파싱 중 적용가능한 생성 규칙이 없으면 오류를 알린다.

### 예제 : x-2*y로 파싱
---

| 규칙 | 기호 |  | 내용 |
| :---: | :---: | :---: | --- |
| 0 | Goal | $\rightarrow$ | Expr |
| 1 | Expr | $\rightarrow$ | Expr + Term |
| 2 |  | \|  | Expr - Term |
| 3 |  | \| | Term |
| 4 | Term | $\rightarrow$ | Term * Factor |
| 5 |  | \| | Term / Factor |
| 6 |  | \| | Factor |
| 7 | Factor | $\rightarrow$ | (Expr) |
| 8 |  | \| | number |
| 9 |  | \| | id |


1. E → E - T | T(2*y)
2. T → T * F | F(y)
3. F → ( E ) | id(y)
4. 파싱 성공


### Left recursion
---
>문법 규칙에 따라 유도되는 과정에서, 자기 자신을 가리키는 규칙이 왼쪽에 있는 것을 말한다.

Top-down parsing에서 무한 루프를 일으킬 수 있기 때문에 없애야 한다.


|  |  |  |
| :---: | :---: | --- |
 | Expr | $\rightarrow$ | Expr + Term |
| | \| | Expr - Term |
 | | \| | Term |

이 문법은 Left recursion문제가 발생한다.  
Left recursion을 없애기 위해 비단말 기호를 추가한다.

|  |  |  |
| :---: | :---: | --- |
 | Expr | $\rightarrow$ | Term Expr' |
 | Expr | $\rightarrow$ | + Term Expr' |
| | \| | - Term Expr' |
 | | \| | $\epsilon$ |

### 필요한 look-ahead의 횟수
---

* LL(1)
  * 왼쪽에서 오른쪽으로 검사한다.
  * Leftmost derivation
  * 1-token look-ahead
    * 미리보는 것(토큰)의 개수가 1개
* LR(1)
  * 오른쪽에서 왼쪽으로 검사한다.
  * Rightmost derivation
  * 1-token look-ahead
    * 미리보는 것(토큰)의 개수가 1개


### Predictive parsing
---
>A $\rightarrow$ $\alpha$ | $\beta$ 를 만들때, 우리는 $\alpha$ 또는 $\beta$ 로 확장하는 정확한 production을 선택하는 확실한 방법을 원한다.

$\alpha$ $\in$ G인 일부 RHS에 대해서, $\alpha$ 로 부터 유도된 일부 문자열의 첫번째 토큰 집합을 **FIRST($\alpha$)** 로 정의한다.

일부 $\gamma$ 에 대해 $\alpha$ $\rightarrow$ $\ast$ x $\gamma$ 일때, x $\in$ FIRST($\alpha$) 이다.

A $\rightarrow$ $\alpha$ 와 A $\rightarrow$ $\beta$ 이 두 production이 문법에서 같이 존재할때, FIRST($\alpha$) $\cap$ FIRST($\beta$) = $\emptyset$ 이어야 한다.

이는 파서가 하나의 기호를 미리보면서 올바른 선택을 하도록 한다.

A $\rightarrow$ $\alpha$ 와 A $\rightarrow$ $\beta$ 이고, $\epsilon$ $\in$ FIRST($\alpha$) 일때, FIRST($\beta$)는 FOLLOW(A)와 분리되어야 한다.

**FOLLOW(A)**  
한 문장 형태에서 A(A $\in$ NT)를 바로 따를 수 있는 단말 기호 집합이다.  
시작 기호 S에 대해서 FOLLOW(S) = {EOF} 이다.  

**FOLLOW 집합을 만들기 위해서 FIRST 집합을 사용한다.**

**FIRST+(A $\rightarrow$ $\alpha$)** 를 정의한다.  
$\epsilon$ $\in$ FIRST($\alpha$)이라면, FIRST($\alpha$) $\cup$ FOLLOW(A) 이다.  
그렇지 않으면, FIRST($\alpha$)이다.


A $\rightarrow$ $\alpha$ 와 A $\rightarrow$ $\beta$ 를 처리한다면 그 때 문법은 LL(1)이다.  
FIRST+(A $\rightarrow$ $\alpha$) $\cap$ FIRST+(A $\rightarrow$ $\beta$) = $\emptyset$

### FIRST 집합 예제
---

| 규칙 | 기호 |  | 내용 |
| :---: | :---: | :---: | --- |
| 0 | Goal | $\rightarrow$ | Expr |
| 1 | Expr | $\rightarrow$ | Term Expr' |
| 2 | Expr' | $\rightarrow$ | + Term Expr' |
| 3 |  | \| | - Term Expr' |
| 4 |  | $\rightarrow$ | $\epsilon$ |
| 5 | Term | $\rightarrow$ | Factor Term' |
| 6 | Term' | $\rightarrow$ | * Factor Term' |
| 7 |  | \| | / Factor Term' |
| 8 |  | \| | $\epsilon$ |
| 9 | Factor | $\rightarrow$ | ( Expr ) |
| 10 |  | \| | number |
| 11 |  | \| | id |

| 기호 $\alpha$ | FIRST($\alpha$) |
| ---| --- |
number, id, +, -, *, /, $\epsilon$ | number, id, +, -, *, /, $\epsilon$
Expr | (, number, id
Expr' | +, -, $\epsilon$
Term | (, number, id
Term' | *, /, $\epsilon$
Factor | (, number, id


### 언어 문법이 LL(1)이 아니라면?
---
>Left factoring을 하면, 일부 문법을 LL(1) 문법으로 만들 수 있다.

* 알고리즘
  1.  비단말 노드 A에 대해서 반복
       * 공통 Prefix가 있는 대체 RHS'를 가지는 비단말 기호가 없을 때까지 반복합니다.
  2. A에 대한 2개 이상의 대안에 공통되는 가장 긴 Prefix a를 찾는다.
  3. 만약 $\alpha$ != $\epsilon$ 이라면
     1. 모든 유도 결과를 바꾼다.
     2. A $\rightarrow$ $\alpha$$\beta$<sub>1</sub> | $\beta$<sub>2</sub> | $\beta$<sub>3</sub> | ... | $\alpha$$\beta$<sub>n</sub> | $\gamma$ 를 아래와 같이 바꾼다.
        1. A $\rightarrow$ $\alpha$A' | $\gamma$
        2. A' $\rightarrow$ $\beta$<sub>1</sub> | $\beta$<sub>2</sub> | $\beta$<sub>3</sub> | ... | $\beta$<sub>n</sub>

### Left factoring 예제
---

**아래의 규칙에 대해 Left factoring 수행**
![](https://user-images.githubusercontent.com/48265129/233979679-d5dbca20-69be-4e93-8ae5-31cca9c9a836.png)

**Left factoring 결과**
![](https://user-images.githubusercontent.com/48265129/233979733-a91cdffb-ba43-4e44-9798-722b084d6b90.png)



## Classic expression grammer
---

![](https://user-images.githubusercontent.com/48265129/233982669-5534181a-b6aa-41cc-ac79-6ffe15ccbd18.png)

![](https://user-images.githubusercontent.com/48265129/233982703-47b1f032-a14f-4571-992b-e40cecb910a0.png)

![](https://user-images.githubusercontent.com/48265129/233982736-fb99617b-fb02-4ef7-aa09-215a9116422f.png)


## Bottom-up parser
---
* 트리의 맨 끝(단말 노드)에서 시작하여 루트 노드로 만들어 간다.
* 입력된 문장 W에서 시작하여 Reduce에 의해 시작 기호 S를 찾아간다.
  * 우단 유도의 역순
* 상태와 문장 형태를 저장하기 위해서 **스택**을 사용한다.
* LR parser
  * 일반적인 방법
  * 왼쪽에서 파싱을 해서 우측유도(Rightmost Derivation) 방식으로 동작한다.

**Reduce**  
어떤 문장 형태에서 특정한 생성 규칙의 RHS(Right hand side)를 찾아 LHS(Left hand side)로 바꾸는 것이다.

### abbcde 예제
---

| 규칙 | 기호 |  | 내용 |
| :---: | :---: | :---: | --- |
| 1 | S | $\rightarrow$ | aABe |
| 2 | A | \|  | Abc |
| 3 |  | \| | b |
| 4 | B | $\rightarrow$ | d |

**stack**

| Production | 문장 형태 |
:---: | :---:
3 | abbcde
2 | aAbcde
4 | aAde
1 | aABe
\- | S

