---
layout: post
title: Domain, DroidKnights 2023 '빈혈(anemic) 도메인 모델과 쓸모없는 유스케이스 그리고 비대한(Bloated) 뷰모델에 대해 생각해보기'내용 정리
subtitle: 빈혈(anemic) 도메인 모델, 쓸모없는 유스케이스, 비대한(Bloated) 뷰모델에 대해 생각해보고 개선 방안을 탐색
published: true
categories: [Android]
tags: [Android]
---

## 개요

> 이번 글은 아래의 영상으로 부터 얻은 내용을 바탕으로 작성하였습니다. 영상을 먼저 시청 하시고 글을 읽으시면 이해가 쉬울거라고 생각합니다

**빈혈 도메인, 쓸모없는 유스케이스, 비대한 뷰모델에 대하여**를 주제로 DroidKnights 2023에서 발표하신 박종혁님(카카오스타일)의 영상입니다.

[![Architecture: the UI layer](https://img.youtube.com/vi/3mR8_vT7m1U/sddefault.jpg)](https://youtu.be/3mR8_vT7m1U?si=sIs_2rVc667qZUtC)
***Video : 빈혈(anemic) 도메인 모델과 쓸모없는 유스케이스 그리고 비대한(Bloated) 뷰모델에 대해 생각해보기***

## 영상의 핵심 : 비효율적인 코드를 도메인으로 

작업 중 만들어 질수 있는 비효율적인 경우로 **빈혈 도메인 모델**, **쓸모없는 유스케이스**, **비대한 뷰모델** 세 가지가 소개되었습니다.

#### 빈혈, 무기력한, 빈약한(Anemic) 도메인 모델

> 도메인 모델(클래스)이 데이터만 가지고 있고, 어떠한 로직이 없는 상태, [AnemicDomainModel](https://martinfowler.com/bliki/AnemicDomainModel.html)

- 안티 패턴 중 하나(특정 경우에는 적절할 수 있으므로 무조건 사용하지 말아야 하는 것은 아닙니다.)
- 해당 모델의 데이터를 사용하는 로직이 다른 곳에 존재하게 되므로 객체지향적인 설계(단일 책임 원칙)를 벗어납니다.
- 단순 구조체와 다를 바 없으므로, 모델을 생성하는 의미가 없다고 볼수 있습니다.
 

```kotlin
data class Developer(
    val name: String,
    val age: Int,
)

data class DeveloperList(
    val list: List<Developer>
)
```

#### 쓸모없는 유스케이스

> 유스케이스(클래스)가 단순히 레포지토리와 뷰모델을 연결만 해주는 상태, 특별한 기능이 없음

- 비즈니스 로직을 서버에 위임하는 아키텍처에서 주로 보임
- 안티 패턴으로 간주되기도 하지만, 코드 일관성 및 미래의 수정에 대비하는 등의 이유로 사용되기도 함


```kotlin
interface SpeakerRepository {
    suspend fun getSpeakerList(): List<Speaker>
}

class LoadSpeakerListUseCase(
    private val speakerRepository: SpeakerRepository
) {
    suspend operator fun invoke() = speakerRepository.getSpeakerList()
}
```

오직 `speakerRepository.getSpeakerList()`를 위해서 유스케이스가 쓰입니다.


#### 비대한(Bloated) 뷰모델

> 뷰모델이 과하게 크고 복잡한 상태

```kotlin
class SpeakerListViewModel(
    private val loadSpeakerList: LoadSpeakerListUseCase
) : ViewModel() {
    private val speakerList // ...

    fun load() {
        viewModelScope.launch {
            speakerList = loadSpeaker()
            // ...
        }
    }

    fun getYoungDevelopers() = speakerList.filter { 
        it.age < 30 
    }

    // ...
}
```