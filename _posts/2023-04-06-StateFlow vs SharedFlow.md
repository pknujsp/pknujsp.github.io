---
layout: post
title: StateFlow vs SharedFlow
subtitle: StateFlow, SharedFlow
published: true
categories: Android
tags: [Android, Flow]
---

StateFlow, SharedFlow
=============    


## 상속관계, Cold/Hot stream  

__Flow__ <- __SharedFlow__ <- __StateFlow__  

### Flow  
__Cold stream__, __collect__ 를 할 때마다 __emit__ 된 모든 값들을 받는다.  

ex) 어떤 __Flow__ 를 1부터 10사이의 모든 정수를 __emit(방출)__ 한 경우, __Collector__ 가 해당 __Flow__ 를 __collect__ 하면 모든 값(1~10사이의 정수, 이때 까지 emit된 모든 값)들을 전달받음

### SharedFlow, StateFlow  
__Hot stream__, __collect__ 이 시작된 시점 이후의 __emit__ 된 값을 전달받음  


## StateFlow  
  
> __StateFlow__ 는 가장 최신의 값만 내보낸다(emit)  


## MutableStateFlow  

값을 바꿀 수 있음

__MutableStateFlow__ 는 인스턴스 생성 시 __초기 값__ 을 필수로 설정해야 한다.  
__StateFlow__ 는 __State__ 를 표현하는 __Data Model__ 로 사용하기에 유용함  

__Flow__ 에서는 여러 연산자들을 사용해서 __다양한 값__ 로 바꾸고 정의할 수 있음  
특히 __Combine__ 연산자는 __임의의 함수__ 를 사용하여 __여러 StateFlow__ 의 값들을 __결합__ 하는 데 유용함  

```kotlin
class CounterModel {
    private val _counter = MutableStateFlow(0)
    val counter = _counter.asStateFlow()

    fun increment() {
        _counter.update {
            count → count + 1
        }
    }
}
```


```kotlin
val aModel = CounterModel()
val bModel = CounterModel()
val sumFlow : Flow<Int> = aModel.counter.combine(bModel.counter) {
    a, b → a + b
}
```  

**MutableStateFlow의 대안으로, stateIn 연산자를 통해서 ColdFlow를 StateFlow로 변환할 수 있음**  

## SharedFlow  

가장 최근의 값만 내보내지 않고, __collect__ 이전에 __emit__ 된 값들에 대해 내보낼 값의 개수를 지정하는 것이 가능  

__Buffer__ 가 가득 찼을 경우의 처리 동작을 설정 가능

## StateFlow 대신 SharedFlow를 사용하는 경우  

* 여러 개의 값들을 가지면서, 사용하고자 할 때 
* 초기 값을 생략하고자 할 때
* 추가 Buffering이 필요할 때  

## StateFlow는 SharedFlow이다  

__StateFlow__ 는 좁지만 널리 사용되는 상태 공유를 위한 __SharedFlow__ 의 특수 목적, 고성능, 효율적인 구현이다.  

__StateFlow__ 는 새로운 __Collector(Subscriber)__ 에게 가장 최근의 값만을 주고, 2개 이상의 값은 담아두지 않음  


아래와 같이 __MutableSharedFlow__ 에 매개변수를 지정하고, __distinctUntilChanged()__ 를 사용하면 __SharedFlow__ 를 __StateFlow__ 처럼 사용가능

```kotlin
val shared = MutableSharedFlow(
    replay = 1,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)

shared.tryEmit(initialValue)
val state = shared.distinctUntilChanged()
```  

## MutableSharedFlow 생성 매개변수  

### __replay__  
__collect__ 시 전달받을 __방출된(emitted)__ 데이터의 개수를 설정  




## 동시성(Concurrency)  

모든 __StateFlow__ 의 메서드는 __Thread-safe__  
__Coroutine__ 에서 외부 동기화 없이 안전하게 호출가능  

## 연산자 융합(Operator fusion)   

__StateFlow__ 은 __flowOn__, __conflate__, __CONFLATED or RENDEZVOUS capacity__, __distinctUntilChanged__, __cancellable__ 연산자를 사용하더라도 영향을 받지 않음  