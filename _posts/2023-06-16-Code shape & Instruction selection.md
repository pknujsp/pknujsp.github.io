---
layout: post
title: Compiler backend
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
  (2,1),(2,2),(2,3),(2,4)
]

* Arrays
  * Row major
    * 대부분의 언어에서 채택
    * 연속 행의 시퀀스로 배치
    * 가장 오른쪽 아래 첨자가 가장 빠르게 변함
    * A : [
      (1,1),(1,2),(1,3),(1,4),(2,1),(2,2),(2,3),(2,4)
    ]
  * Column major
    * Fortran에서 채택
    * 열 시퀀스로 배치
    * 가장 왼쪽 위 첨자가 가장 빠르게 변함
    * Cache miss가 많이 발생
    * A : [
      (1,1),(2,1),(1,2),(2,2),(1,3),(2,3),(1,4),(2,4)
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

1. Numbering
  - 레지스터를 최소한으로 사용하기 위한 과정
  - Spiling 없는 서브 트리를 평가하기 위해 필요한 레지스터들을 계산한다.
  - 번호로 각 내부 노드에 레이블을 지정한다.
2. Code generation
  - 트리를 순회하고 코드를 생성한다.
  - 라벨에 따라 순서를 평가함

* 특징
  * 간단한 기계 모델에 최적화 되어 있다.
  * 레지스터와 명령어를 줄인다.

* 실제 기계 모델 최적화를 위해서는
  * 동작이 지연되는 아키텍처는 매우 복잡하며 아래의 문제를 고려해야 한다.
    * Issue LOAD, 결과가 나중에 지연 주기로 나타난다.
    * 결과가 참조되지 않는 한 계속 실행된다.
    * 너무 이른 참조로 인해 HW정지가 발생하거나 컴파일러가 지연 슬롯을 NOPs로 채울 수 있다.


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

불필요한 계산을 제거하거나, 코드의 실행 로직을 단순화하는 등 최적화 수행에 필요한 정보를 분석한다.

### Liveness analysis
---
> 컴파일러는 임시 레지스터에 대해 Liveness analysis 를 수행해야 한다.

어떤 변수가 프로그램에서 사용되지 않는다면, 메모리를 회수하여 최적화 한다.

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
  * single predecessor/successor를 가지는 노드를 합쳐서 Basic block으로 만들어 CFG의 크기를 줄인다.
* One variables at a time
  * 모든 변수에 대해서 data flow analysis를 수행하는 것보다, 하나의 변수에 대해서 단순화한 분석을 하는 것이 더 낫다.
* Representation of sets
  * For dense sets
    * 정수형 배열에서 1인 비트를 원소로 간주하는 경우에 사용한다.
    * 쓰이지 않는 0이 저장되므로 메모리 사용에서 비효율적이다.
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

<img src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/8c95a61e-ef9f-49ea-994d-ceb51e0751ff">

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

## Register allocation
---

* N개의 레지스터에 대한 정확한 코드를 생성한다.
* live 레지스터의 재사용을 최대화한다.
  * load, store 를 최소화한다.
* Spill 을 최소화한다,
  * save a stack space


### Register allocation 과정
---

N register code -> `Register allocation` -> K register code

1. liveness analysis로 부터 interference graph(간섭 그래프, IG)를 생성한다.
2. IG 에 대한 k-coloring을 분석한다. 또는 코드를 k-color가 될수 있는 문제에 근접하도록 바꾼다.
3. 각 k-colored 변수(임시 레지스터)를 k registers로 매핑한다.

### Graph coloring problem
---
> 그래프는 동일한 에지를 공유하는 두 노드가 동일한 색상을 갖지 않는 색상으로 그래프의 노드에 레이블을 지정하는 경우에만 k-colorable이 가능하다.(한 에지에 연결된 노드는 서로 다른 색상을 가져야 한다)

각 색상은 고유한 논리 레지스터에 매핑된다.

### IG
---

* 노드는 변수를 나타낸다.
* 에지는 변수가 동일한 레지스터에 할당될 수 없음을 나타낸다.

**IG의 k-coloring은 k 레지스터 할당으로 매핑될 수 있다.**

이웃이 k개 미만인 정점 v는 항상 색상이 지정될 수 있다.

### 레지스터 할당 문제를 해결하는 방법
---

1. Chaitin's algorithm
2. Chaitin-Briggs algorithm

### Chaitin's algorithm
---

1. IG 생성
2. 노드 제거
   * 이웃이 k개 미만인 노드를 제거하고, 스택에 추가한다.
   * IG에서 이러한 노드가 완전히 사라질때 까지 작업을 반복한다.
3. 스킬링
   * k보다 많은 이웃을 가진 노드를 제거한다.
4. 레지스터 할당
   * 스택에서 노드를 꺼내면서 레지스터를 할당(색을 입힘)한다.
   * 색은 다른 노드에 할당되지 않은 색이어야 한다.

### Improvement of Color scheme
---

모든 정점이 최소 k개의 이웃을 가질 때 멈추는 것 보다, 우선 순위에 따라 스택에 추가하는 것이 더 낫다.


### 간단한 레지스터 할당 방식
---

1. Build
2. Simplify
3. Spill
4. Select
5. Finish


## Instruction selection
---

* 고려 사항
 * 실행 시간을 줄이기 위해 명령어 순서 재조정
 * 정확성을 유지하면서 성능 향상
 * 최적화

* 스케줄링 방식
  * 동적
  * 정적

* Hazard in pipeline
  * Structural hazard
    * 하드웨어 자원이 충분하지 않은 경우
  * Data hazard
    * 피연산자는 이전 명령어에 의존함
  * Control hazard
    * 조건 분기

### 프로세서 수행 과정
---

1. Fetch
2. Decode
3. Fetch
4. Execute
5. Store
6. Next

### Multi-cycle design 장단점
---

* 장점
  * 사이클 시간을 줄인다
  * 프로세스가 파이프라이닝하는 것이 가능함
* 단점
  * 단계 사이에서 데이터를 저장할 때 추가적인 레지스터가 필요하다
  * HW 설계가 복잡해진다

### Pipelining
---
> 수행 시 여러 명령어가 동시에 실행되도록 해주는 기술

* 장점
  * 기능 단위의 사용을 더욱 향상시킨다.
* 단점
  * HW 성능 요구가 더 높다.
  * hazard를 발생시킨다.


### Hazards
---

* Structural hazard
  * HW자원의 충돌
  * 두 명령어가 동시에 같은 자원을 사용하려고 할 때 발생
* Data hazard
  * 데이터 의존성
  * 한 명령어가 다른 명령어의 결과에 의존하는 경우
* Control hazard
  * 분기 명령어와 같은 제어 명령어로 인해 파이프라인의 흐름이 변경되는 경우에 발생
  * 분기 명령어가 실행되어야 다음에 어떤 명령어가 실행될 지 결정되기 때문이다.




### Instruction scheduling
---
> slow code -> `Scheduler` -> fast code

* 특징
  * 올바른 코드를 만든다.
  * 불필요한 사이클을 줄인다.
  * Avoid spill registers
    * Spill register : 변수의 값을 메모리로 옮기는 작업이 일어나는 레지스터, 레지스터 공간이 부족한 경우에 Spilling이 발생한다.
  * 작동 효율성 향상
  
To capture properties of the code, build a precedence graph G.

instruction -> node

dependency -> edge

### Data dependencies
---

- 의존성은 아래의 두 조건을 만족할 때에 나타난다.
  - 두 메모리 참조가 같은 위치에 접근할 때
  - 최소한 하나의 메모리 참조가 쓰기를 수행할 때

- 종류
   - RAW : read after write
     - True dependence
     - 지울수 없다
   - WAR : write after read
     - Anti dependence
     - 변수명 변경으로 제거 가능
   - WAW : write after write
     - Output dependence
     - 변수명 변경으로 제거 가능
   - RAR : read after read
     - Input dependence
     - 의존성이 없다

- 이러한 위험 중 일부를 제거하기 위해 컴파일러를 제한할 수 있지만 이러한 위험은 자주 발생하며 하드웨어에서 문제를 해결하는 것이 좋다.
  - `NOP` : 목적지에 의존성이 없는 것을 load delay slot에 채운다.
- 여전히 미세한 최적화를 위해서는 컴파일러가 지연 동작을 고려해야 한다.
  - 파이프라인 스케줄링 또는 명령어 스케줄링은 컴파일러가 stall을 피하기 위해 명령어를 재 배치하는 방법이다.

### Instruction scheduling(Scope)
---

- Basic blocks
  - `List scheduling`
  - trace로 바꾼다
- Branches
  - `Trace scheduling`
  - control hazard를 줄이기 위해, superblock인 trace scheduling을 이용한다.
- Loops
  - Unrolling
  - Software pipelining


### List scheduling
---
1. RAW/WAW를 없애기 위해 이름을 바꾼다.
2. 우선 Graph를 만든다.
3. 명령어에 우선순위를 부여한다.
4. select와 명령어 스케줄링을 반복한다.
   - Candidates : Roots of graph, 실행 가능한 명령어들
   - Candidates가 남아있을때
     1. 최고 우선순위 후보를 선택한다.
     2. 명령어를 스케줄링 한다.
     3. 노출된 명령어를 후보에 추가한다.

  
* Two flavors of list scheduling
  * Forward
  * Backward

### 스케줄링 휴리스틱
---

- 고려사항
  - 준비된 명령어들 중 얼마나 선택해야 하는가?
  - NP-hard for straight-line code
- 후보의 우선 순위를 설정하는 방법
  - pipeline stall을 발생시키지 않는다.
  - 루트까지의 가장 큰 가중치인 경로(critical path)
  - 가장 긴 지연시간(more overlap)
  - Most immediate successors(후보를 만듦)
  - Most descendants(후보를 더 만듦)

## Trace scheduling
---
- Parallelism across if branches
  1. trace selection
  2. trace compaction

<img src="https://github.com/skydoves/chatgpt-android/assets/48265129/2a95550a-d5c7-4c8c-a5f3-2f1bf39aaafd">


## Optimization
---

### 잠재적으로 컴파일 시 최적화 가능한 영역
---

* Source language(AST)
  * Constant bounds in loops and arrays
  * Loop unrolling
  * Suppressing runtime check
  * Enable later optimizations
* IR : Local and Global
  * CSE
  * Liveness analysis
  * Code hoisting
  * Enable later optimizations
* Code generation(machine code)
  * Register allocation
  * Instruction scheduling
  * Peephole optimization

### Classical Distinction
---

* 머신 '비의존적' 변환 
  * 광범위한 머신에 적용 가능 
  * 중복 계산 제거 
  * 쓸모없는 코드 찾아 제거 
  * 실행 빈도가 낮은 곳으로 더 많은 평가 수행
  * 일부 범용 코드 전문화
  * 다른 최적화를 위한 기회 노출
* 머신 '의존적' 변환
  * 머신별 속성 활용
  * IR에서 머신으로의 매핑 개선
  * 비용이 많이 드는 작업을 더 저렴한 작업으로 대체
  * 명령어 시퀀스를 더 강력한 것으로 대체

### Scope of Optimization
---

* Local
* Intraprocedural(global)
* Interprocedural(whole program)


### 최적화 시 핵심 목표
---

* Safety
* Profitability
* Opportunity

### Safety
---
> 최적화 시에 발생한 변화가 프로그램의 실행 결과를 바꾸지 않는다는 것을 보장하는 것이 중요

* 컴파일 시 분석
  * Loop unrolling
    * 대부분의 경우에 안전함
  * DAGs and CSEs
    * 간단한 분석이다
  * Dataflow analysis
    * 복잡한 추론이 필요할 수 있다

### Profitability
---
> 최적화를 통해 프로그램의 실행 속도가 빨라지는 것을 보장하는 것이 중요

* 컴파일 시 추정
  * Always profitable
  * Heuristic rules
  * Compute benefit

### Opportunity
---
> 최적화를 해야 하는 적절한 지점을 찾는 것이 중요

* Issues
  * 변환을 적용하는 프레임워크를 제공
  * 체계적으로 모든 지점을 찾기
  * 이전 변경 사항을 반영하도록 안전 정보를 업데이트 하기