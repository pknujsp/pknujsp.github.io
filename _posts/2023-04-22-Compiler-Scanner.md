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
  * RE → NFA → DFA 변환 과정이 중요하다.
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
  * 어떠한 상태에도 ε-전환은 없다.
    * NFA는 ε-전환을 가진다.
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

| state |    a     |   b   |
| :---: | :------: | :---: |
|  S0   | {S0, S1} | {S0}  |
|  S1   |          | {S2}  |
|  S2   |          | {S3}  |

S0는 a!에서 여러 전환을 한다.

## RE → DFA
---
1. RE → NFA w/ε-moves
   - 각각의 항을 NFA로 빌드한다.
   - 그것들을 ε-moves로 묶는다.
2. NFA w/ε-moves 를 DFA로 만든다.
    - NFA를 시뮬레이션 하기 위해 DFA를 만든다.
    - 하위 집합을 만드는 것.
3. DFA → Minimized DFA
    - 호환 가능한 상태를 병합한다.
4. DFA → RE
    - 모든 쌍과 모든 경로들의 문제
    - S0 부터 마지막 상태까지의 경로들을 합친다.

이 과정을 반복한다.  
**RE → NFA w/ε-moves → DFA → Minimized DFA → RE** ..

### 1. RE to NFA
---
>**Thompson's construction(정규 표현식을 NFA로 변환하는 알고리듬)**을 보통 사용한다.  
>NFA 패턴을 사용한다.  
>각 연산에 대해서, NFA하위 그래프를 생성하여 전체 NFA를 구성한다.  


![image](https://user-images.githubusercontent.com/48265129/233826506-1f49341e-38a0-4003-ab95-2c503f763370.png)
![image](https://user-images.githubusercontent.com/48265129/233826511-37b247c0-7bc1-4a11-b9e3-4502e9ed7639.png)
![image](https://user-images.githubusercontent.com/48265129/233826519-98176b09-2a56-456c-b92d-f0878f0b5a23.png)
![image](https://user-images.githubusercontent.com/48265129/233827104-45379a02-80f0-45c2-8ef6-536ef49e55f2.png)
![image](https://user-images.githubusercontent.com/48265129/233827112-87bb5677-6f59-40cd-88f5-a297fac21abf.png)
N(A+)를 만드는 방법은 N(A*)에서 S0에서 마지막 상태로 가는 ε을 제거하면 된다.

![image](https://user-images.githubusercontent.com/48265129/233832439-8fadc01d-e2d9-4573-afa4-fff11376ad6b.png)

### 2. NFA → DFA(Subset construction)
---
>NFA를 시뮬레이션 해야한다.  

* 두 가지 핵심 함수가 있다.
  * move(s<sub>i</sub>, a)
    * 상태 s<sub>i</sub>에서 입력 기호(심볼) a에 대해 가능한 모든 다음 상태들의 집합을 반환한다.
    * NFA에서 전환 규칙에 따라 각 상태와 입력 기호 쌍에 대해 일어날 수 있는 상태 전환을 결정한다.
  * ε-closure(s<sub>i</sub>)
    * 상태 s<sub>i</sub>에서 ε-전환만을 사용하여 도달 가능한 모든 상태들의 집합을 반환한다.
    * move(s<sub>i</sub>, ε)처럼 추가 입력이 없을 경우에 바로 다음 상태로 전환이 가능할 때 사용한다.


* 알고리즘
  1. 시작 상태 설정
     * NFA의 시작 상태 s<sub>0</sub>에서 ε-전환에 의해 도달 가능한 모든 상태들의 집합을 DFA의 시작 상태로 설정한다.
     * ε-closure 함수를 사용해서 상태를 찾는 것이다.
  2. 새로운 상태 집합 생성
     * DFA의 상태에서 입력 기호에 대해 move 함수를 통해 가능한 상태 집합을 찾는다.
     * 찾은 상태 집합에 대해서 ε-closure 함수를 통해, 가능한 상태 집합을 찾는다.
     * S<sub>0</sub> = ε-closure({s<sub>0</sub>})
  3. 최종적으로 찾은 상태 집합을 DFA에 추가한다.
     * DFA에 대한 전환 테이블에 집합을 추가한다.
  4. 추가적인 상태가 나오지 않을 때 까지 이 과정을 반복한다.

#### 예제
---
![image](https://user-images.githubusercontent.com/48265129/233835795-1f28cab8-f186-43a0-b1f6-5fce4514079c.png)

| states | | ε-closure(move(s, *)) | | |
| :---: | :---: | :---: | :---: | :---: |
| DFA | NFA | a | b | c |
| s<sub>0</sub> | q<sub>0</sub> | q<sub>1</sub>, q<sub>2</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>6</sub>, q<sub>9</sub> | none | none |
| s<sub>1</sub> | q<sub>1</sub>, q<sub>2</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>6</sub>, q<sub>9</sub> | none | q<sub>5</sub>, q<sub>8</sub>, q<sub>9</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>6</sub> | q<sub>7</sub>, q<sub>8</sub>, q<sub>9</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>6</sub> |
| s<sub>2</sub> | q<sub>5</sub>, q<sub>8</sub>, q<sub>9</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>6</sub> | none | s<sub>2</sub> | s<sub>3</sub> |
| s<sub>3</sub> | q<sub>7</sub>, q<sub>8</sub>, q<sub>9</sub>, q<sub>3</sub>, q<sub>4</sub>, q<sub>6</sub> | none | s<sub>2</sub> | s<sub>3</sub> |

s<sub>1</sub>, s<sub>2</sub>, s<sub>3</sub>는 종료 상태이다.

**NFA를 DFA로 바꾼 결과**
>ε-전환이 없기 때문에, NFA보다 작다.  
>모든 전환은 결정론적이다.  
>Use same code skeleton as before
![image](https://user-images.githubusercontent.com/48265129/233835970-c1fcb6f3-42cf-44a0-a80b-1771270ded2d.png)

 ||a | b | c | 
| :---: | :---: | :---: | :---: |
| s<sub>0</sub> |  s<sub>1</sub> | none | none |
| s<sub>1</sub> | none | s<sub>2</sub> | s<sub>3</sub> |
| s<sub>2</sub> | none | s<sub>2</sub> | s<sub>3</sub> |
| s<sub>3</sub> | none | s<sub>2</sub> | s<sub>3</sub> |

## 3. DFA → Minimized DFA
---
>DFA에서 동일한 상태 집합을 찾아서 각 집합을 단일 상태로 나타낸다.

* 알고리즘
 1. 서로 동일하지 않은 모든 상태를 찾는다.
     - 상태 X와 Y는 다음과 같은 경우에 서로 다른 상태이다.
       - X가 종료 상태, Y는 종료 상태가 아닐 때
       - 서로 같은 입력 기호를 가지더라도 그 이전 상태가 다를 때
 2. 서로 동등하지 않은 쌍은 제거한다.


**축소 결과**
| | 동일한 상태 | a | b | c | 
| :---: | :---: | :---: | :---: | :---: |
 s<sub>0</sub> | {s<sub>0</sub>} |  s<sub>1</sub> |  s<sub>1</sub> | none | none |
 s<sub>1</sub> | {s<sub>1</sub>,  s<sub>2</sub>,  s<sub>3</sub>} | none | s<sub>2</sub> | s<sub>3</sub> |

  s<sub>1</sub>과  s<sub>2</sub>,  s<sub>2</sub>과  s<sub>3</sub>는 동일하다.

 ![image](https://user-images.githubusercontent.com/48265129/233837364-e2310227-a866-4eab-9652-a88bc50603f0.png)


## 정규 표현식의 장점
---
>패턴을 지정하는 단순하고 강력한 표기법이다.  
>많은 종류의 구문을 정규 표현식으로 지정할 수 있다.  


## Regular language의 한계점
---

* 정규 표현식은 균형있고 중첩된 구성을 기술하는데 사용할 수 없다.
  * 균형잡힌 괄호 문자열의 집합은 정규 표현식으로 기술할 수 없다.
  * 이러한 집합은 Context-free 문법으로 처리할 수 있다.
* L = {p<sup>k</sup> q<sup>k</sup>} (k는 임의의 수이다.)
  * 정규 표현식으로 이 내용을 처리할 수 없다.
  * Parser로 처리한다.
* 정규 표현식은 오직 정해진 반복 횟수와 지정되지 않은 반복 횟수만 나타내는데 사용될 수 있다.
  * (ε|1)(01)*(ε|0)
  * (01|10)+


## Lex and Yacc
---
>Scanner와 Parser의 소스 코드를 만드는 데 사용된다.  
>또 다른 Flex(Lex와 같은 역할), Bison(Yacc과 같은 역할)도 같은 역할을 한다.

