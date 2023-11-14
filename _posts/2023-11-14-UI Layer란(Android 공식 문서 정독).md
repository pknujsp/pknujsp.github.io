---
layout: post
title: About the UI layer
subtitle: 안드로이드 공식 문서 정독
published: true
categories: [Android]
tags: [Android]
---

## UI layer

> UI : 앱의 데이터를 화면에 표시하고 사용자와 상호 작용하는 부분입니다, Data 계층으로 부터 변경되는 앱의 상태를 시각적으로 표현하는 역할

데이터가 변경될 때마다 UI는 그에 맞게 바뀌어야 한다.(버튼 클릭, 네트워크 응답 등)

보통 Data 계층에서 받은 데이터를 화면에 바로 보여주기에는 적절하지 않습니다. 예를 들어, 일부 데이터만 보여줘야 하거나, 여러 개의 데이터들을 합쳐서 보여줘야 하는 경우가 있습니다. UI 계층은 이러한 데이터를 UI로 표시할 수 있게 가공하여, 화면에 보여주는 역할을 중요한 과정을 담당합니다.

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-overview.png" width="70%">

***Android 앱 아키텍처에서 UI 계층의 역할***

> 참고 : 이 페이지에 제시된 권장 사항 및 모범 사례는 다양한 앱에 적용하여 앱의 확장성, 품질 및 견고성을 개선하고 테스트하기 쉽게 만들 수 있습니다. 하지만 이를 가이드라인으로 간주하고 필요에 따라 요구 사항에 맞게 조정해야 합니다.

[![Architecture: the UI layer](https://img.youtube.com/vi/p9VR8KbmzEE/0.jpg)](https://www.youtube.com/watch?v=p9VR8KbmzEE)
***Video : Architecture: The UI layer***

## 기본 사례

뉴스 기사를 가져오는 앱을 생각해봅시다.

기사 목록을 표시하는 기사 목록 화면이 있고, 로그인한 사용자는 기사를 즐겨찾기에 추가할 수 있습니다. 기사가 너무 많을 수 있으니 사용자는 카테고리별로 기사를 볼수도 있습니다.

- 사용자가 할 수 있는 기능 정리
  - 기사 목록을 볼 수 있습니다.
  - 카테고리별로 기사를 볼 수 있습니다.
  - 로그인하여 기사를 즐겨찾기에 추가할 수 있습니다.
  - 자격이 된다면 일부 프리미엄 기능을 사용할 수 있습니다.(유료 결제)

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-basic-case-study.png" width="70%">

***샘플 뉴스 앱***

다음 내용에서는 이 앱을 예시로 하여 단방향 데이터 흐름 원칙을 설명하고, 이러한 원칙이 UI 계층에 대하여 앱 아키텍처의 관점에서 도움이 되는 문제를 설명합니다.

## UI 계층 아키텍처

Android에서 UI는 데이터를 보여주는 액티비티 또는 프래그먼트와 같은 UI요소를 의미합니다. 이를 구현하기 위해 쓰이는 뷰와 컴포즈 같은 API는 UI라고 하지 않습니다. 그 이유는 Data 계층의 역할은 앱 데이터를 보유/관리하고 접근하는 권한을 제공하는 것이기 때문입니다.

그러므로 UI 계층은 다음 과정을 따라야 합니다.

1. 앱 데이터를 사용하고 UI가 쉽게 표시할 수 있도록 데이터를 가공합니다.
2. UI가 표시가능한 데이터를 사용하고, 사용자에게 표시할 수 있도록 UI 요소로 가공합니다.
3. UI 요소에서 사용자의 입력을 받고, 필요에 따라 UI 데이터에 관련 효과를 적용합니다.

필요한 만큼 `1~3` 과정을 반복합니다.

지금부터는 UI 계층에서 이런 과정을 수행하도록 구현하는 방법을 설명하겠습니다. 구체적으로 다음과 같은 내용을 다루겠습니다.

- **UI 상태 정의**: 앱의 화면과 사용자간의 상호작용을 어떻게 데이터로 표현할지 정의하는 방법.
- **단방향 데이터 흐름(UDF)**: UI 상태를 만들고 관리하는 데 사용되는 방식. 데이터가 한 방향으로만 흐르며, 이로 인해 데이터 관리가 더 명확해집니다.
- **관찰 가능한 데이터 타입으로 UI 상태를 나타내기**: UDF 원칙을 따라 UI 상태를 어떻게 다른 부분들이 관찰하고 반응할 수 있는 형태로 만드는지 설명합니다.
- **관찰 가능한 UI 상태를 사용하는 UI 구현**: 실제 UI가 이러한 상태를 어떻게 받아들이고 화면에 표시하는지 보여줍니다.

이 중에서 가장 중요한 것은 'UI 상태의 정의' 입니다.

## UI 상태(State)의 정의

> UI State : 앱이 사용자에게 보여주는 데이터

UI와 UI State는 정의가 다릅니다.

- UI: 사용자가 화면으로 보고 있는 것
- UI 상태: 앱이 사용자에게 보여주는 것

### 동전을 예로 들어보겠습니다.

- UI 상태: 동전의 양면 중 어느 쪽이 위로 향하고 있는지 '앞면' 또는 '뒷면'과 같이 현재 상태를 나타내는 정보
- UI: UI 상태의 시각적 표현(눈으로 보이는 동전의 모습), 앞면이나 뒷면이 보이는 것

UI 상태가 바뀌면 그에 따라서 UI도 바뀝니다.

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-elements-state.png" width="70%">

***UI는 화면 상의 UI 요소와 UI 상태가 결합한 것입니다***

뉴스 앱의 요구 사항을 따라 UI를 만들기 위해서 `NewsUiState` 라는 클래스에 데이터를 담을 수 있습니다.

```kotlin
data class NewsUiState(
    val isSignedIn: Boolean = false,
    val isPremium: Boolean = false,
    val newsItems: List<NewsItemUiState> = listOf(),
    val userMessages: List<Message> = listOf()
)

data class NewsItemUiState(
    val title: String,
    val body: String,
    val bookmarked: Boolean = false,
    ...
)
```

### 불변성(Immutability)

위 예제 코드의 UI 상태는 불변합니다. 불변 객체의 장점은 앱의 상태가 특정 시점에서 어떤 상태인지 보장해준다는 것입니다. 이로 인해 UI가 하나의 역할(상태를 읽고 적절하게 UI 요소를 업데이트)만 집중할 수 있게 됩니다. 그래서 UI자체가 데이터의 유일한 출처가 아니라면 개발자는 UI 상태를 직접 변경하면 안됩니다. 직접 변경한다면 같은 데이터에 대해 여러개의 출처가 생기면서 데이터의 일관성이 깨지며 버그가 발생할 수 있습니다.

예를 들어, `NewsUiState`내 `bookmarked`의 값이 `Activity`에서 바뀐다면 `bookmarked`는 실제 북마크 상태를 두고 Data 계층과 충돌하게 됩니다. 왜냐하면 북마크 상태를 실제로 변경하는 작업은 Data 계층에서 이루어져야 하기 때문입니다. 이런 문제를 방지하기 위해서 불변 클래스를 사용하는 것이 좋습니다.

> 데이터를 소유하거나 직접적으로 변경하는 객체는 스스로 노출시키는 데이터를 업데이트 할 책임이 있습니다.

### 이름 짓기 가이드(Naming conventions)

현재 문서에서는 UI 상태 클래스의 이름을 화면의 기능 또는 데이터가 표현되는 화면의 일부를 기반으로 하여 짓습니다.

**functionality + UiState.kt**

`NewsUiState`: 뉴스를 표시하는 화면의 상태
`NewsItemUiState`: 기사 목록내 각 아이템의 상태

## 단방향 데이터 흐름(UDF, Unidirectional Data Flow)으로 UI 상태를 관리하기

위에서 UI 상태가 UI를 표시하기 위해 필요한 세부 사항의 불변 스냅샷이라고 설명했습니다. 그러나 보통 데이터는 동적으로 바뀌고 따라서 상태도 계속 변경됩니다. 사용자가 앱을 쓰기 때문입니다. 

이러한 상호작용은 중개자(이벤트에 적용할 로직을 정의하고 UI 상태를 만들기 위한 데이터를 가공하는 작업을 수행)의 도움을 받아야 합니다. 이런 상호작용과 로직은 UI 자체에 포함될 수는 있지만, UI가 이를 처리하는 것은 좋지 않습니다. 왜냐하면 UI는 UI 상태를 표시하는 것에만 집중해야 하기 때문입니다. 데이터 소유, 생산, 변환과 같은 역할까지 해야 한다면 UI는 너무 많은 역할을 하게 됩니다. 이는 코드를 이해하기 어렵게 만들고, 테스트하기 어렵게 만들며, 유지보수하기 어렵게 만듭니다.

UI는 오직 UI 상태를 가지고 화면에 보여주는 역할만을 하여야 합니다.

### 상태 홀더(State holders)

> 상태 홀더 : UI 상태를 만들고 관련 로직을 가지고 있는 클래스

상태 홀더 클래스의 규모는 관리하는 UI 요소(App bar와 같은 작은 요소에서 부터 화면 전체 또는 화면 이동 대상)에 따라 다양합니다.

> Data 계층에 접근하는 화면 수준의 UI 상태를 관리하는 경우에는 `ViewModel`을 사용하는 것을 권장합니다. `ViewModel`은 구성이 변경되더라도 데이터를 유지할 수 있습니다.`ViewModel`은 앱의 이벤트를 처리하는 로직을 정의하고 처리 결과를 상태로 만들어 내는데 사용합니다.

UI와 상태 생성간의 상호 의존성을 만드는 방법은 다양합니다. 그러나 UI와 ViewModel간의 상호작용은 크게 이벤트의 입력과 그에 따른 상태를 출력하는 것으로 이해할 수 있는데, 다이어그램으로 표현하면 다음과 같습니다.

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-udf.png" width="70%">

***앱 아키텍처에서 UDF의 흐름***

상태가 아래로 흐르고 이벤트가 위로 흐르는 이러한 패턴이 바로 `단방향 데이터 흐름(UDF)`입니다. UDF를 통해 앱 아키텍처는 다음과 같이 변화합니다.

- ViewModel은 UI가 사용할 상태를 보유하고 노출시킵니다. UI 상태는 ViewModel에서 변환되는 앱 데이터입니다.
- UI는 ViewModel에 사용자 이벤트를 알립니다.
- ViewModel은 사용자 이벤트를 처리하고 상태를 업데이트합니다.
- 업데이트된 상태는 UI에 전달됩니다.
- 위 과정은 상태를 바꾸는 이벤트에 의해서 반복됩니다.

네비게이션 목적지나 화면에 대해서 ViewModel은 Repository 또는 UseCase에 접근하여 데이터를 가져오고, 이를 UI 상태로 변환합니다. 이때 상태를 변화시킬수 있는 이벤트(사용자 입력, 데이터 변경 등)의 영향을 고려하여 처리합니다. 현재 문서의 예제에서 기사 목록의 경우는 제목, 내용, 출처, 기자, 작성 날짜, 북마크 여부를 포함하고 있고 목록 내 각 아이템에 대한 UI는 다음과 같습니다.

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-basic-case-study-item.png" width="70%">

***기사 하나를 표현하는 아이템의 UI***

기사를 북마크에 추가하는 사용자는 상태를 변화시키는 이벤트입니다. ViewModel은 상태를 생산하는 역할을 맡고 있으므로 UI 상태 내의 모든 필드를 다루고 UI가 표시되기 위해 필요한 이벤트를 처리하여야 합니다.

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-udf-in-action.png" width="70%">

***UDF에서 이벤트와 데이터가 처리되는 흐름***

아래에서는 상태 변경을 일으키는 이벤트와 이를 UDF를 사용하여 처리하는 방법을 자세히 설명합니다.

### 로직의 유형

기사를 북마크하는 것은 비즈니스 로직의 예입니다. (이 부분에 대한 자세한 내용은 [Data 계층](https://developer.android.com/jetpack/guide/data-layer)페이지를 살펴보세요.) 정의해야하는 주요 로직의 유형은 다음과 같습니다.

- Business logic: 앱의 데이터에 대한 요구 사항을 구현하는 로직
  - 예시: 기사를 북마크하는 것
  - 보통 도메인 또는 Data 계층에서 구현합니다.
  - UI 계층에서는 구현하지 않도록 합니다.
- UI behavior logic, UI logic: 화면에 상태 변경을 보여주는 로직
  - 예시: `Resource`를 사용하여 표시할 문자열을 가져오는 것, 사용자가 버튼을 눌렀을때 다른 화면으로 이동하는 것, toast 또는 snackbar를 표시하는 것
  - `Context`와 같은 UI 유형과 관련된 로직의 경우, ViewModel이 아닌 UI에서 구현해야 합니다.
  - 해당 로직이 너무 복잡해진다면 `State holder`와 같이 간단한 클래스를 만들어서 위임하면 됩니다.
  - [Jetpack Compose State guide](https://developer.android.com/jetpack/compose/state#managing-state)에서 State holder와 UI 생성과 관련된 내용을 살펴보실수 있습니다. 