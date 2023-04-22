---
layout: post
title: Compiler Scanner
subtitle: Scanner
published: true
categories: Compiler
tags: [Compiler]
---

# Scanner

## Word 인식 방법
---
>상태 머신을 사용하고, 정규 표현식으로 구문을 분석한다.

* 정규 표현식
  * Formal language(형식적인 언어)이다.
    * 규칙에 의해 명시된 Symbol의 집합이다.
    * 애매한 부분이 있으면 안된다.

복잡한 Scanner개발 과정을 간소화 하기 위해, Scanner Generator를 사용하여 개발할 수도 있다.

### 예제
---
### while
![image](https://user-images.githubusercontent.com/48265129/233789669-1b5a6c18-58de-4a4c-98fc-94d15aea511d.png)

### fee | fie
![image](https://user-images.githubusercontent.com/48265129/233789675-d99bd608-3f8a-450b-ad5d-93eea46a61fd.png)

### while | fee | fie
![image](https://user-images.githubusercontent.com/48265129/233789682-6233ca6f-aad3-4aae-a634-046d8f298306.png)

### Transition table
---
>레지스터 : r0, r1, ..., r31

![image](https://user-images.githubusercontent.com/48265129/233789820-1ab00e0b-809a-4343-aa13-9b6645983bf8.png)

## Recognizer(인식기)
---
>언어 인식기는 문자열 x를 입력으로 하였을때, x가 언어의 문장인지 여부에 따라 yes 또는 no를 출력으로 하는 프로그램이다.

우리는 전환 다이어그램(DFA)을 구성하여 RE(정규 표현식)를 인식기로 컴파일 할 수 있다.

Scanner는 정규 표현식과 유사한 것을 통해서 자동으로 생성될 수 있다.

* DFA를 구성한다.
  * RE => NFA => DFA 변환 과정이 중요하다.
* 상태 최소화 기법을 사용한다.
* Scanner에 대한 코드를 내보낸다.

### 예제
---

![image](https://user-images.githubusercontent.com/48265129/233790794-5c5bb65d-feab-4547-8c2b-d7d419b520a6.png)

현재 상태에 따라 다음이 정해진다.

* Identifier
  * letter
    * (a|b|...|z|A|B|...|Z|)
  * digit
    * (0|1|2|...|9)
  * id
    * letter(letter|digit)*

## NFA(비결정론적 유한 오토마타), DFA(결정론적 유한 오토마타)
---
>DFA는 NFA의 특별한 경우이다.

* DFA 특징
  * 어떠한 상태에도 $\epsilon$-전환은 없다.
    * NFA는 $\epsilon$-전환을 가진다.
    * DFA와 NFA의 대표적인 차이점이다.
  * 각 상태 s와 입력 기호 a에 대해서 a로 표시된 edge는 최대 하나이다.
  * 시작 상태인 S0에서 x로 표시된 edge를 따라 표시된  허용된 상태까지에 대해서 전환 그래프내에 고유한 경로가 존재하면 x를 허용한다.
  
* DFA와 NFA의 공통점
  * DFA는 NFA로 시뮬레이션 가능하다.
  * NFA는 DFA로 변환이 가능하다.

* 비결정론적이란
  * 하나의 상태에서 값이 들어왔을때, 어떤 상태로 갈지 모르는 것을 의미한다.

### NFA 예제
---

__(a|b)*abb__
![image](https://user-images.githubusercontent.com/48265129/233790959-2c34bc0d-8c7c-4ee3-9699-b13f87f2fec4.png)

| state | a | b |
| :---: | :---: | :---: |
| S0 | {S0, S1} | {S0} |
| S1 |  | {S2} |
| S2 |  | {S3} |

S0는 a!에서 여러 전환을 한다.

## RE => DFA
---
1. RE => NFA w/$\epsilon$-moves
   - sds