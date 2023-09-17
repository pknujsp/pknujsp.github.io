---
layout: post
title: Garbage collector에 대해서
subtitle: 역할, 동작 방식
published: trueas
categories: GarbageCollector
tags: [GarbageCollector]
---
asd as
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

## 동작 방식

Mark and Sweep 이라는 알고리즘으로 동작한다.

![gcactions drawio](https://github.com/pknujsp/android-smartdeeplink/assets/48265129/fa314b28-01e2-4342-9ff6-89690af8c366)

1. Young(Eden, Survivor), Old에서 어떤 영역의 용량이 가득차게 된다면 GC는 해당 영역의 객체들을 검사한다.
2. GC Root로부터 참조되고 있는 객체를 따라가며 참조 상태를 확인한다.
3. 여전히 참조되고 있는(유효한, Reachable) 객체는 살아남고, 그렇지 않은(Unreacheable) 객체는 회수된다.
4. 이동할 때 Age bit를 1씩 증가시킨다.
5. 이때, Reacheable 객체가 존재하는 영역이 바뀐다.
   - Eden에 있다면 Survivor 영역으로 이동.
   - Survivor 0/1 에 있다면 Survivor 1/0으로 이동.
   - Age bit가 특정 조건을 넘어서면 Old 영역으로 이동.


GC가 동작할 때에는 GC가 동작하는 스레드를 제외한 나머지 모든 스레드가 일시 정지된다. 이를 **Stop the world**라고 한다. 만약 스레드가 정지되지 않는다면 GC가 동작하는 동안에도 객체가 생성되어 GC가 제대로 동작하지 않을 수 있는 등 여러 문제가 발생할 수 있다.

#### Unreacheble 상태가 되는 경우

- 할당된 객체가 연결된 변수가 없을 때
  - 객체가 생성된 메서드가 종료될 때(Stack에서 pop) 객체와 연결된 변수가 사라짐.
  - 변수에 null이 할당될 때
  - 변수에 새로운 객체가 할당될 때

#### Unreachable은 아니나 회수가 되는 경우

- 객체의 참조 유형이 일반적인 유형이 아닌 경우
  - WeakReference, SoftReference, PhantomReference 등의 참조 유형을 사용하는 경우

## 객체의 참조 유형

#### Strong Reference : 일반적인 참조 유형

흔히 객체를 생성할 때 사용되는 방식이다.

```kotlin
val obj = Object()
```

#### Weak Reference : 약한 참조

GC가 발생하면 무조건 회수되는 참조 유형이다. WeakReference를 사용하면 GC가 발생하면 무조건 회수되기 때문에 메모리 누수를 방지할 수 있다.


```kotlin
val weakRef = WeakReference(Object())
// null 또는 not null, GC가 동작한 경우에는 null이 obj에 반환된다.
val obj = weakRef.get()
```


#### Soft Reference : 약한 참조

GC가 발생하면 회수되지만, 메모리가 부족할 때에만 회수되는 참조 유형이다. WeakReference와 다르게 GC가 발생하면 무조건 회수되지 않는다. 메모리가 부족할 때에만 회수되기 때문에 메모리 누수를 방지할 수 있다.

```kotlin
val softRef = SoftReference(Object())
// null 또는 not null, 메모리가 부족한 상황인 경우라면 null이 obj에 반환될 수 있다.
val obj = softRef.get()
```


![gc-weak-strong](https://github.com/pknujsp/android-smartdeeplink/assets/48265129/520a7fb3-3150-4841-89db-d42b8cc655e5)

Weak Reference로 할당된 객체와 연결된 하위 객체는 모두 Weak로 참조되기 떄문에, 최상위 Weak Reference가 회수되면 같이 모두 회수가 된다.

