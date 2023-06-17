---
layout: post
title: Code shape & Instruction selection
subtitle: code shape & instruction selection
published: true
categories: Compiler
tags: [Compiler]
---

## 컴파일러 프론트엔드와 백엔드의 명백한 차이
---

* 프론트엔드
  * 프로그래밍 언어에 의존적
  * 목표 기계에 비 의존적
* 백엔드
  * 언어에 다소 비 의존적
  * IR은 언어에 의존적
  * 목표 기계에 의존적

## 백엔드의 구성
---

* Instruction selection
  * IR을 어셈블리 코드로 변환
  * 고정된 스토리지 매핑 및 코드 모양을 추정
  * 주소 지정 모드를 사용하여 작업을 결합
* Instruction scheduling
  * 연산을 정렬하여 지연 시간을 줄임
  * 고정된 프로그램을 추정
  * 레지스터에 대한 요구를 변경
* Register allocation
  * 레지스터에 상주할 값을 결정
  * 스토리지 매핑을 변경
  * 데이터 및 메모리 작업 배치에 대한 우려

백엔드의 처리 로직은 NP-Complete 문제로, 최적의 로직이 아닐 수도 있다.

## Code shape
---

> 성능에 영향을 미치는 코드의 모든 모호한 속성

알고리즘 선택과 결과에 영향을 준다.

같은 결과를 만드는 것에 대해서는 다양한 구현이 가능하고, 최적의 형태는 코드의 문맥에 의존한다.

### Example
---

* case 문에 대한 처리를 하는 경우
  * if-then-else
    * 성능은 case의 개수에 의존적
    * O(number of cases)
  * jump table
    * 상수 시간에 처리
  * binary search
    * O(log n)

case 에 따른 분기 처리는 다양한 방법이 있고, 컴파일러는 최적의 방법을 선택해야 한다.

* Code shape에 신경을 써야하는 이유
  * 최적화 및 코드 생성을 돕기 위해
  * 컴파일러의 개별 패스가 빠르게 실행되어야 하므로
  * 유용한 정보를 IR에 인코딩하기 위해
    * 표현식 또는 제어 구조의 모양
    * 메모리가 아닌 레지스터에 보관되는 값
  * 가능한 경우 이러한 정보를 도출하는 데 비용이 많이 들 수 있음
  * IR에 명시적으로 기록하는 것이 더 쉽고 저렴할 때가 많음

* Code shape를 IR에 기록하는 방법
  * Instruction selection 수행 이전에 High-level IR을 Low-level IR로 변환한다.

### Low-level Representation
---

* Instrinsic operations
  * AST
* String
  * 끝에 null을 추가하여 구분
  * 길이를 명시하여 구분
* Structures
  * 속성의 선언 순으로 구분
  * 속성의 자료형 순서로 구분

A : [
  (1,1),(1,2),(1,3),(1,4),
  (2,1),(2,2),(2,3),(2,4),
]

* Arrays
  * Row major
    * 대부분의 언어에서 채택
    * 연속 행의 시퀀스로 배치
    * 가장 오른쪽 아래 첨자가 가장 빠르게 변함
    * A : [
      (1,1),(1,2),(1,3),(1,4),(2,1),(2,2),(2,3),(2,4),
    ]
  * Column major
    * Fortran에서 채택
    * 열 시퀀스로 배치
    * 가장 왼쪽 위 첨자가 가장 빠르게 변함
    * Cache miss가 많이 발생
    * A : [
      (1,1),(2,1),(1,2),(2,2),(1,3),(2,3),(1,4),(2,4),
    ]

* Loops
  * Loop는 IR에서 명시적으로 표현되지 않는다.
  * it-then-else 구조로 표현됨
* Switches
  * Linear search 등의 방법으로 재 구성됨


## Instruction selection
---

* 기본 접근법
  * 각 IR 튜플/하위 트리를 기계 명령어로 매크로 확장
  * 때때로 매핑은 N:1임
  * maximal munch
    * RISC와 합리적으로 잘 작동
* 다른 접근법
  * IR이 확장될 때 대상 머신 상태를 모델링 한다.
    * 해석 코드를 생성하는 것
  * Tuple을 읽는다
  * 해석기 상태를 갱신한다
  * 대상 코드를 만든다

### Tree patterns
---
> 각 기계의 명령어를 IR tree의 조각으로 표현하는 것

`Instruction selection`은 tree patterns의 최소한의 집합을 가지는 `Tiling IR tree`를 의미한다.

### Tiling
---
> Target machine이 지원하는 명령어, tree를 생성하므로 해당 기계에 의존적이다.

* 목표
  * tiles 오버래핑이 없도록 IR tree를 보호하는 것

### Optimal tiling
---

* Greedy
  * Tree의 루트에서 시작
  * 가장 큰 tile을 찾는다
  * 서브 트리를 모두 순회한다
* Dynamic programming
  * 모든 트리 노드에 비용을 할당한다
  * 각 노드에 가장 적합한 tiling의 명령어 비용 집합을 계산한다

### Code generation
---

* 컨셉
  * Topological sort
    * 모든 피연산자가 수행 전에 준비되도록 한다.

* tree IRs 에 대해 코드를 생성하는 방법
  * Sethi-Ullman Numbering
    * 레지스터가 최소한으로 사용되도록 한다.
  * Code generation for DAGs

### Register and temporary management
---

임시적으로 현재 수행과 연관된 데이터를 저장한다. (레지스터, 스택 등)

* Register allocation
  * Hard registers(Physical registers) 일부는 스토리지에 할당되어야 한다.
  * 각 임시 레지스터에 대하여 Pseudo-register(Virtual registers)를 추정한다.
  * Register를 사용하는 것 자체가 최적화를 하는 것이다.

### Sethi-Ullman Numbering
---

- Numbering
  - Spiling 없는 서브 트리를 평가하기 위해 필요한 레지스터들을 계산한다.
  - 번호로 각 내부 노드에 레이블을 지정한다.
- Code generation
  - 트리를 순회하고 코드를 생성한다.
  - 라벨에 따라 순서를 평가함

* 특징
  * 간단한 기계 모델에 최적화 되어 있다.
  * 레지스터와 명령어를 줄인다.

* 실제 기계 모델에 최적화를 위해서는
  * 동작이 지연되는 아키텍처는 매우 복잡하며 아래의 문제를 고려해야 한다.
    * Issue LOAD, 결과가 나중에 지연 주기로 나타난다.
    * 결과가 참조되지 않는 한 계속 실행된다.
    * 너무 이른 참조로 인해 hw 정지가 발생하거나 컴파일러가 지연 슬롯을 NOPs로 채울 수 있다.


### Code generation for DAGs
---
> Tree보다 더 복잡하다.

* 사용할 모든 코드가 생성되기 전 까지 공유 값을 가지고 있는다.
* 레지스터 사용을 최소화하기 어렵게 만든다.
* 좌측 피연산자는 보통 제거되고, 공유는 사용하기 전에 왼쪽 피연산자를 강제로 복사한다.
* 최적화 코드 생성기는 복사를 최소화하기 위해 DAG 평가를 정렬해야 한다.
* 피연산자가 마지막으로 사용될 때 왼쪽 피연산자로 사용되도록 DAG 평가를 정렬한다.

* 과정
  1. Schedule
  2. Allocate virtual registers
  3. Map virtual registers to hard registers


### Local optimization
---
> Data-flow 분석없이, 해석을 기본 블럭으로 제한한다.

* 어떻게 컴파일러가 기본 블록에 대해서 최적화를 하는 가?
  * 불필요한 수행을 참조로 교체한다.
  * 일반 목적 코드를 구체화한다.
  * 사용되지 않는 코드를 제거한다.
  * 다른 기회를 노출시킴.

### Peephole optimization
---
> A small window (2-3 tuples/instruction)

* 특징
  * 코드에 대한 peephole을 분석한다.
  * 특별한 패턴을 찾아서 최적화를 한다.

* Examples
  * Constant folding
  * Strength reduction
  * Null sequence
  * Combine operations


## Data-flow analysis
---
> 정적 코드만 검사하여 프로그램의 동적 동작에 대한 정보를 도출한다.

### Liveness analysis
---
> 컴파일러는 임시 레지스터에 대해 Liveness analysis 를 수행해야 한다.

나중에 사용되는 레지스터가 값을 가지고 있으면, Live 상태이다.

* 특징
  * Register allocation
    * Virtual to Physical
  * Problem
    * IR는 무제한으로 임시 레지스터를 가진다.
    * 기계는 제한적인 수의 레지스터를 가진다
  * Approach
    * 분리된 live 범위가 있는 임시는 동일한 레지스터에 매핑될 수 있다.
    * 충분한 레지스터가 없으면, 임시로 저장된다.
  * 컴파일러는 이 프로세스를 자동화해야 한다.

### Example
---

```c
int func(int b, int c) {
  int a = 0;

  do {
    b = a + 1;
    c = c + b;
    a = b * 2;
  } while(a < 9);

  return c;
}
```

1. a = 0
2. L1 : b = a + 1
3. c = c + b
4. a = b * 2
5. if a < 9 goto L1
6. return c

* Live range of a
  * 1 -> 2
  * again
    * 4 -> 5 -> 2
  * dead
    * 2 -> 3 -> 4
* Live range of b
  * 2 -> 3 -> 4

### Control flow analysis
---

Liveness analysis 수행 전에, CFG를 만드는 Control flow를 이해해야 한다.

* 노드는 독립적인 프로그램 문장이나 기본 블록이다.
* 에지는 잠재적인 Control flow이다.

### Example of CFG
---

1. a = 0
2. L1 : b = a + 1
3. c = c + b
4. a = b * 2
5. if a < 9 goto L1
6. return c

* 노드 n으로 부터 나오는 에지는 후속 노드인 succ[n]로 이어진다.
* 노드 n에 대한 in 에지는 이전 노드인 pred[n]에서 나온다. 
  
* Out-edges of node 5
  * 5 -> 2
  * 5 -> 6
* succ[5]
  * {2, 6}
* pred[5]
  * {4}
* pred[2]
  * {1, 5}


## Dominance
---

CFG의 형태와 구조를 추론하는 핵심 도구는 지배자(Dominanace) 개념이다.

노드 b0이 있는 흐름 그래프에서, 노드 bi는 b0에서 bj까지의 모든 경로에 있는 경우 노드 bj를 지배한다.

### Liveness analysis
---

Liveness information을 모으는 것은 CFG에 대한 data flow analysis를 수행하는 것과 같다.

* def
  * def(v)
    * v를 정의하는 노드
  * def(n)
    * n으로 정의되는 변수
* use
  * use(v)
    * v를 사용하는 노드
  * use(n)
    * n에서 사용하는 변수

### Performance consideration
---

* Basic blocks
  * single predecessor/successor를 가지는 노드를 합쳐서 CFG의 크기를 줄인다.
* One variables at a time
  * 모든 변수에 대해서 data flow analysis를 수행하는 것보다, 하나의 변수에 대해서 단순화한 분석을 하는 것이 더 낫다.
* Representation of sets
  * For dense sets
    * 정수형 배열에서 1인 비트를 원소로 간주하는 경우에 사용한다.
    * 0또한 저장되므로 메모리 사용이 비 효율적이다.
  * For sparse sets
    * 원소가 존재하는 메모리 위치를 저장하므로, 메모리 사용량이 적다.

### Basic blocks
---
> 입구와 출구 외에는 분기가 없는 코드 시퀀스이다.

* Basic blocks를 나누는 과정
  1. 코드에서 핵심이 되는 리드 요소를 파악한다.
      * 첫번째 명령어
      * 분기의 대상
      * 분기를 즉시 따르는 명령어
  2. 다음 리드 요소를 제외하고 현재 리드를 따르는 모든 명령어를 포함하는 block을 분리한다.

### Example
---

1. a = 0
2. L1 : b = a + 1
3. c = c + b
4. a = b * 2
5. if a < 9 goto L1
6. return c

* leader(case)
  * 1, 2, 6

* Splitted result(Basic blocks)
  1. a = 0
  2. L1 : b = a + 1, c = c + b, a = b * 2, if a < 9 goto L1
  3. return c

