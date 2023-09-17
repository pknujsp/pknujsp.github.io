---
layout: post
title: Garbage collector에 대해서 알아보자
subtitle: 역할, 동작 방식
published: trueas
categories: GarbageCollector
tags: [GarbageCollector]
---
## Garbage collector(GC)란

C, C++과 같이 GC가 없는 언어에서는 동적으로 할당한 메모리를 모두 개발자가 직접 관리해야 한다.   

```c++
// int array를 동적 할당
int *arr = new int[5];

for(int i = 0; i < 5; ++i) {
    arr[i] = i;
}

// 해제
delete[] arr;
```

위와 같은 방식으로 할당시켜주는데, 사용 후 해제를 하지 않는다면 메모리 누수가 발생하게 되며 지속적으로 이러한 상황이 누적될 경우 프로그램에 치명적으로 작용하게 된다. Java/Kotlin(Jvm), C#(.Net)과 같은 고 수준 언어에서는 GC가 메모리 관리를 대신 해주기 때문에 상대적으로 편리하다.

### GC의 구조

GC는 크게 3가지 영역으로 나뉘어져 있다.

메모리 Heap영역에서 Young generation, Old generation으로 나뉘어진다.

- Young generation
  - Eden : 새로운 객체가 생성될 때 할당되는 영역
  - Survivor 0, 1 : Eden에서 GC가 발생하면 살아남은 객체가 이곳으로 이동한다.
  - Minor GC가 발생.
- Old generation : Young generation에서 살아남은 객체가 특정 조건을 넘어서면 이곳으로 이동한다.(Promotion)
  - Major GC가 발생.

![gc drawio](https://github.com/pknujsp/android-smartdeeplink/assets/48265129/3228d5ea-f91a-496b-b67c-96456e513127)  


#### 용어

- Minor GC : Young generation에서 발생하는 GC
- Major(Full) GC : Old generation에서 발생하는 GC
- Reacheable : 객체가 **참조되고 있는** 상태, 다른 영역으로 이동하게 된다.
- Unreacheable : 객체가  **참조되고 있지 않은** 상태, 회수 대상
- Age bit : 객체가 Young generation에서 살아남은 횟수를 기록한다.(보통 정수로 표현)
- GC Root : GC가 동작할 때 참조를 따라가며 참조 상태를 확인하는 시작점이 되는 객체들이다. GC Root로부터 참조되고 있는 객체들은 살아남고, 그렇지 않은 객체들은 회수된다.

## 동작 방식

Mark and Sweep 이라는 알고리즘으로 동작한다.

![gcactions drawio](https://github.com/pknujsp/android-smartdeeplink/assets/48265129/fa314b28-01e2-4342-9ff6-89690af8c366)  


1. Young(Eden, Survivor), Old에서 어떤 영역의 용량이 가득차게 된다면 GC는 해당 영역의 객체들을 검사한다.
2. **GC Root**로부터 참조되고 있는 객체를 따라가며 참조 상태를 확인한다.
3. 여전히 참조되고 있는(유효한, Reachable) 객체는 살아남고, 그렇지 않은(Unreacheable) 객체는 회수된다.
4. 이동할 때 **Age bit**를 1씩 증가시킨다.
5. 이때, Reacheable 객체가 존재하는 영역이 바뀐다.
   - Eden에 있다면 Survivor 영역으로 이동.(두 영역 중 하나로 이동)
   - Survivor 0/1 에 있다면 Survivor 1/0으로 이동.
   - Age bit가 특정 조건을 넘어서면 Old 영역으로 이동.

동작 시에 Survivor 두 영역 중 하나는 반드시 비어있는 상태이다.

GC가 동작할 때에는 GC가 동작하는 스레드를 제외한 나머지 모든 스레드가 일시 정지된다. 이를 **Stop the world**라고 한다. 만약 스레드가 정지되지 않는다면 GC가 동작하는 동안에도 객체가 생성되어 GC가 제대로 동작하지 않을 수 있는 등 여러 문제가 발생할 수 있다.

#### Unreachable 상태가 되는 경우

- 할당된 객체가 연결된 변수가 없을 때
  - 객체가 생성된 메서드가 종료될 때(Stack에서 pop) 객체와 연결된 변수가 사라짐.
  - 변수에 null이 할당될 때
  - 변수에 새로운 객체가 할당될 때

#### Unreachable은 아니나 회수가 되는 경우

- 객체의 참조 유형이 일반적인 유형이 아닌 경우
  - WeakReference, SoftReference의 참조 유형을 사용하는 경우

## 객체의 참조 유형

#### Strong Reference : 일반적인 참조 유형

흔히 객체를 생성할 때 사용되는 방식이다.

```kotlin
val obj = Object()
```

#### Weak Reference

GC가 발생하면 무조건 회수되는 참조 유형이다. WeakReference를 사용하면 GC가 발생하면 무조건 회수되기 때문에 메모리 누수를 방지할 수 있다.


```kotlin
val weakRef = WeakReference(Object())
// null 또는 not null, GC가 동작한 경우에는 null이 obj에 반환된다.
val obj = weakRef.get()
```


#### Soft Reference

GC가 발생하면 회수되지만, 메모리가 부족할 때에만 회수되는 참조 유형이다. WeakReference와 다르게 무조건 회수되지 않는다. 메모리가 부족할 때에만 회수되기 때문에 메모리 누수를 방지할 수 있다.

```kotlin
val softRef = SoftReference(Object())
// null 또는 not null, 메모리가 부족한 상황인 경우라면 null이 obj에 반환될 수 있다.
val obj = softRef.get()
```

![gc-weak-strong](https://github.com/pknujsp/android-smartdeeplink/assets/48265129/520a7fb3-3150-4841-89db-d42b8cc655e5)  


Weak Reference로 할당된 객체와 연결된 하위 객체는 모두 Weak로 참조되기 떄문에, 상위 Weak Reference가 회수되면  모두 따라서 회수가 된다.


## Stop the world

GC가 너무 자주 동작하는 것도, 적게 동작하는 것도 적합하지 않다. 적절한 빈도로 동작하는 것이 중요하다.

GC가 동작할 때에는, 다른 모든 스레드가 멈추기 때문에, GC 동작이 잦다면, 그 만큼 프로그램이 멈추는 시간이 늘어나게 되므로 성능에 치명적인 영향을 미친다.


## GC의 종류(Jvm 기준)

현재 기본 GC는 JDK 8까지는 Parallel GC, JDK 9부터는 G1 GC가 사용된다.

### Serial GC

GC가 동작할 때, GC를 담당하는 스레드가 하나이다. GC가 동작하는 동안에는 다른 모든 스레드가 멈춘다.

### Parallel GC

멀티 스레드로 Young generation GC를 처리, Old generation GC는 단일 Thread로 처리한다.

### Parallel Old GC

Old generation GC도 멀티 스레드로 처리한다.

### G1(Garbage First) GC

Heap을 Region이라는 영역으로 나눈다.

Eden, Survivor, Old를 여러 개의 Region으로 나누고, 아래 2개의 새로운 영역이 추가되었다.

- Humongous : Region크기의 절반을 초과하는 큰 객체를 저장하는 영역
- Available, Unused : 아직 사용되지 않은 영역

Mark and Sweep에 추가로 Compaction이라는 단계가 추가되었다.

- Compaction : Mark and Sweep 동작으로 인해 객체가 회수된 영역을 비워둔 채로 두지 않고, 다른 영역의 객체를 이동시켜서 메모리 조각모음을 하여 메모리를 효율적으로 사용할 수 있도록 한다.

이외에도 Epsilon, ZGC, Shenandoah GC 등 수 많은 GC알고리즘이 존재한다. 그러나 Stop the world를 완전히 피할 수 있는 GC는 아직 존재하지 않는다.


## GC설정 튜닝

JVM을 사용하는 프로그램에 대해서 GC설정을 직접 수정할 수 있다. 간단하게 Heap의 크기를 설정하는 방법이 있다.
Android studio의 경우, `Android/Android Studio/bin` 디렉토리 내에 studio64.vmoptions 파일을 수정하면 된다.  

다른 IDE도 마찬가지로 이러한 파일을 수정하면 된다.  

**-Xms7g**  
**-Xmx7g**  

vmoptions파일을 열면 위와 같은 설정이 있다.  
Xms는 JVM 시작 시 Heap의 크기, Xmx는 Heap의 최대 크기를 의미한다.
본인 PC의 메모리 용량에 맞게 적절히 값을 수정하면 되고, 크게 할 수록 GC의 동작 빈도가 줄어들기 때문에 어느 정도 성능 향상을 기대할 수 있다.  

보통 Xms와 Xmx의 크기를 같게 설정한다. 왜냐하면 Heap의 크기가 동적으로 변하는 것을 방지하기 위해서이다. 만약 Xms와 Xmx의 크기가 다르다면, Heap의 크기가 동적으로 변하게 되는데, 이는 GC의 동작 빈도를 높일 수 있기 때문이다.  

아래와 같이 추가적인 설정이 가능하다. 그러나 적절한 값을 찾는 과정은 많은 테스트를 통해 찾아야 한다.  

- -XX:NewRatio : Young, Old generation의 비율
- -XX:SurvivorRatio : Eden, Survivor의 비율
- -XX:MaxTenuringThreshold : Age bit의 임계값, 이 값을 넘어서면 Old generation으로 이동, 기본 값은 15(Jvm), [Do Not Set -XX:MaxTenuringThreshold to a Value Greater Than 15](https://support.oracle.com/knowledge/Middleware/1283267_1.html)
- -XX:ParallelGCThreads : Parallel GC의 스레드 개수
