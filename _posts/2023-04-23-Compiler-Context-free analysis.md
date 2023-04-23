---
layout: post
title: Compiler Context-free analysis
subtitle: Syntax analyzer in Parser
published: true
categories: Compiler
tags: [Compiler]
---

# Syntax analyzer(Context-free analysis, 구문 분석기)

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

## Top-down vs Bottom-up parsing
---

### Top-down parser
---
* 유도 트리의 루트(최상단)에서 시작하면서 채운다.
* production을 선택하고 입력과 일치시키도록 한다.
* 나쁜 선택을 한 경우에 Backtracking(역추적)을 할 수도 있다.
* 일부 문법은 역추적이 안될 수도 있다.
  * predictive parsing

### Bottom-up parser
---
* 