---
layout: post
title: About the UI layer
subtitle: 안드로이드 공식 문서 정독
published: true
categories: [Android]
tags: [Android]
---

## UI layer

[](https://developer.android.com/topic/architecture/ui-layer#compose_2)

> UI : 앱의 데이터를 화면에 표시하고 사용자와 상호 작용하는 부분입니다, Data 계층으로 부터 변경되는 앱의 상태를 시각적으로 표현하는 역할

데이터가 변경될 때마다 UI는 그에 맞게 바뀌어야 합니다.(버튼 클릭, 네트워크 응답 등)  

보통 Data 계층에서 받은 데이터를 화면에 바로 보여주기에는 적절하지 않습니다. 예를 들어, 일부 데이터만 보여줘야 하거나, 여러 개의 데이터들을 합쳐서 보여줘야 하는 경우가 있습니다. UI 계층은 이러한 데이터를 UI로 표시할 수 있게 가공하여, 화면에 보여주는 역할을 중요한 과정을 담당합니다.  

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-overview.png" width="70%">

***Android 앱 아키텍처에서 UI 계층의 역할***  

> 참고 : 이 페이지에 제시된 권장 사항 및 모범 사례는 다양한 앱에 적용하여 앱의 확장성, 품질 및 견고성을 개선하고 테스트하기 쉽게 만들 수 있습니다. 하지만 이를 가이드라인으로 간주하고 필요에 따라 요구 사항에 맞게 조정해야 합니다.

[![Architecture: the UI layer](https://img.youtube.com/vi/p9VR8KbmzEE/0.jpg)](https://www.youtube.com/watch?v=p9VR8KbmzEE)
***Video : Architecture: The UI layer***

## 기본 사례

뉴스 기사를 가져오는 앱을 생각해봅시다.


기사 목록을 표시하는 기사 목록 화면이 있고, 로그인한 사용자는 기사를 즐겨찾기에 추가할 수 있습니다. 기사가 너무 많을 수 있으니 사용자는 카테고리별로 기사를 볼수도 있습니다.  

- 사용자가 할 수 있는 기능
  - 기사 목록을 볼 수 있습니다.
  - 카테고리별로 기사를 볼 수 있습니다.
  - 로그인하여 기사를 즐겨찾기에 추가할 수 있습니다.
  - 자격이 된다면 일부 프리미엄 기능을 사용할 수 있습니다.(유료 결제)

<img src="https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-basic-case-study.png" width="60%">

***샘플 뉴스 앱***  

다음 내용에서는 이 앱을 예시로 하여 단방향 데이터 흐름 원칙을 설명하고, 이러한 원칙이 UI 계층에 대하여 앱 아키텍처의 관점에서 도움이 되는 문제를 설명합니다.

## UI 계층 아키텍처

Android에서 UI는 데이터를 보여주는 액티비티 또는 프래그먼트와 같은 UI 요소를 의미합니다. 이를 구현하기 위해 쓰이는 뷰와 컴포즈 같은 API와는 무관합니다. Data 계층의 역할은 앱 데이터를 보유/관리하고 접근하는 권한을 제공하는 것이므로, UI 계층은 다음 과정을 따라야 합니다.

1. 앱 데이터를 사용하고 UI가 쉽게 표시할 수 있도록 데이터를 가공합니다.
2. UI가 표시가능한 데이터를 사용하고, 사용자에게 표시할 수 있도록 UI 요소로 가공합니다.
3. UI 요소에서 사용자의 입력을 받고, 필요에 따라 UI 데이터에 관련 효과를 적용합니다.

필요한 만큼 `1~3` 과정을 반복합니다.  

지금부터는 UI 계층에서 이런 과정을 수행하도록 구현하는 방법을 설명하겠습니다. 구체적으로 다음과 같은 내용을 다루겠습니다.  

- **UI 상태 정의**: 앱의 화면과 사용자간의 상호작용을 어떻게 데이터로 표현할지 정의하는 방법.
- **단방향 데이터 흐름(UDF)**: UI 상태를 만들고 관리하는 데 사용되는 방식. 데이터가 한 방향으로만 흐르며, 이로 인해 데이터 관리가 더 명확해집니다.
- **관찰 가능한 데이터 타입으로 UI 상태를 나타내기**: UDF 원칙을 따라 UI 상태를 어떻게 다른 부분들이 관찰하고 반응할 수 있는 형태로 만드는지 설명합니다.
- **관찰 가능한 UI 상태를 사용하는 UI 구현**: 실제 UI가 이러한 상태를 어떻게 받아들이고 화면에 표시하는지 보여줍니다.

이 중에서 가장 중요한 것은 **UI 상태의 정의** 입니다.

## UI 상태(State)의 정의

> UI State: 앱이 사용자에게 보여주는 데이터

UI와 UI State는 정의가 다릅니다.

- UI: 사용자가 화면으로 보고 있는 것
- UI 상태: 앱이 사용자에게 보여주는 것

### 동전을 예로 들어보겠습니다.

- UI: UI 상태의 시각적 표현(눈으로 보이는 동전의 모습), 앞면이나 뒷면이 보이는 것
- UI 상태: 동전의 양면 중 어느 쪽이 위로 향하고 있는지 '앞면' 또는 '뒷면'과 같이 현재 상태를 나타내는 정보


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

### 이름 짓기(Naming conventions)

현재 문서에서는 UI 상태 클래스의 이름을 화면의 기능 또는 데이터가 표현되는 화면의 일부를 기반으로 하여 짓습니다.  

**functionality + UiState.kt**  

- NewsUiState: 뉴스를 표시하는 화면의 상태
- NewsItemUiState: 기사 목록내 각 아이템의 상태

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

기사를 북마크하는 것은 비즈니스 로직의 예입니다. (이 부분에 대한 자세한 내용은 [Data 계층](https://developer.android.com/jetpack/guide/data-layer)페이지를 살펴보세요) 정의해야하는 주요 로직의 유형은 다음과 같습니다.  

- Business logic: 앱의 데이터에 대한 요구 사항을 구현하는 로직
  - 예시: 기사를 북마크하는 것
  - 보통 도메인 또는 Data 계층에서 구현합니다.
  - UI 계층에서는 구현하지 않도록 합니다.
- UI behavior logic, UI logic: 화면에 상태 변경을 보여주는 로직
  - 예시: `Resource`를 사용하여 표시할 문자열을 가져오는 것, 사용자가 버튼을 눌렀을때 다른 화면으로 이동하는 것, toast 또는 snackbar를 표시하는 것
  - `Context`와 같은 UI 유형과 관련된 로직의 경우, ViewModel이 아닌 UI에서 구현해야 합니다.
  - 해당 로직이 너무 복잡해진다면 `State holder`와 같이 간단한 클래스를 만들어서 위임하면 됩니다.
  - [Jetpack Compose State guide](https://developer.android.com/jetpack/compose/state#managing-state)에서 State holder와 UI 생성과 관련된 내용을 살펴보실수 있습니다.


### UDF를 사용해야 하는 이유

UDF로 상태를 생성하는 흐름을 모델링할 수 있습니다. 또한 상태 변화가 일어나는 곳/변환되는 곳/최종적으로 소비되는 곳 세 가지를 명확하게 분리할 수 있습니다. 이를 통해 UI는 상태를 관찰하면서 데이터를 표시하고 변경 사항을 ViewModel에 전달하여 사용자의 의도를 전달할 수 있습니다.  

- UDF 사용의 이점
  - **데이터 일관성 유지**: UI에 대한 데이터는 하나의 출처만 존재하게 됩니다.
  - **테스트 용이성**: 상태 소스가 분리되어 있어 UI와 독립적으로 테스트 가능합니다.
  - **유지보수성**: 상태의 변화는 잘 정의된 패턴을 따르며, 변화는 사용자의 이벤트와 이벤트에 의해 얻은 데이터 소스의 결과입니다.

## UI 상태 노출

UI 상태를 정의하고 해당 상태의 생성을 관리하는 방법을 결정한 후, 다음 단계는 생성된 상태를 UI에 표시하는 것입니다. UDF를 사용하여 상태 생성을 관리하기 때문에 생성된 상태를 스트림(흐름)으로 간주할 수 있습니다. 다시 말해, 시간이 지남에 따라 여러 버전의 상태가 생성될 것입니다. 따라서 UI 상태를 `LiveData` 또는 `StateFlow`와 같은 관찰 가능한 데이터 홀더에 노출해야 합니다. 그 이유는 ViewModel에서 직접 데이터를 수동으로 가져올 필요 없이 UI가 상태의 모든 변경 사항에 반응할 수 있도록 하기 위해서입니다. 또한 이러한 유형은 항상 최신의 UI 상태가 캐시되어 있어 구성 변경 후 상태를 빠르게 복원하는 데 유용합니다.  

```kotlin
class NewsViewModel(...) : ViewModel() {
    // Compose
    val uiState: NewsUiState = …

    // Views
    val uiState: StateFlow<NewsUiState> = …
}
```

> 참고: Compose에서는 Compose 전용인 관찰 가능한 `State APIs`를 사용할 수 있습니다. `mutableStateOf`, `snapshotFlow`등이 있으며 UI 상태를 노출하는데에 쓰입니다. 물론 `LiveData`와 `StateFlow`도 추가적인 [확장 함수](https://developer.android.com/jetpack/compose/libraries#streams)를 통해 쓸수 있습니다.


UI에 노출되는 데이터가 단순하다면, 데이터를 UI 상태로 감싸는 것이 좋습니다. 왜냐하면 UI 상태는 State holder의 방출과 관련된 화면 또는 UI 요소 사이의 관계를 전달하기 때문입니다. 또한 UI 요소가 더 복잡해진다면 UI 요소를 표시하는데 필요한 추가 정보를 수용하기 위해 UI 상태 정의에 추가하는게 더 낫습니다.  

`UiState` 스트림을 만들려면 ViewModel의 가변 스트림을 불변 스트림으로 노출시키면 됩니다.  

```kotlin
class NewsViewModel(...) : ViewModel() {
    // Compose
    var uiState by mutableStateOf(NewsUiState())
        private set

    // Views
    private val _uiState = MutableStateFlow(NewsUiState())
    val uiState: StateFlow<NewsUiState> = _uiState.asStateFlow()
}
```


ViewModel은 내부적으로 상태를 바꾸는 메서드를 노출시켜서 UI가 사용할 업데이트를 만들어 낼 수 있습니다. 예를 들어, 다음과 같은 비동기 작업을 가정해봅시다. `viewModelScope`에서 동작하는 코루틴이 있고 코루틴이 완료될 때 상태가 업데이트 되는 로직입니다.


```kotlin
class NewsViewModel(
    private val repository: NewsRepository,
    ...
) : ViewModel() {
    // Compose
    var uiState by mutableStateOf(NewsUiState())
        private set

    // Views
    private val _uiState = MutableStateFlow(NewsUiState())
    val uiState: StateFlow<NewsUiState> = _uiState.asStateFlow()

    private var fetchJob: Job? = null

    fun fetchArticles(category: String) {
        fetchJob?.cancel()
        fetchJob = viewModelScope.launch {
            try {
                val newsItems = repository.newsItemsForCategory(category)

                // Compose
                uiState = uiState.copy(newsItems = newsItems)

                // Views
                _uiState.update {
                    it.copy(newsItems = newsItems)
                }
            } catch (ioe: IOException) {
                // 오류를 처리하여 UI에 알립니다
                // Compose
                val messages = getMessagesFromThrowable(ioe)
                uiState = uiState.copy(userMessages = messages)

                // Views
                _uiState.update {
                    val messages = getMessagesFromThrowable(ioe)
                    it.copy(userMessages = messages)
                 }
            }
        }
    }
}
```

`NewsViewModel`은 특정 카테고리의 기사들을 가져와서 UI가 반응할 수 있도록 UI 상태에 성공 또는 실패의 결과를 반영하는 작업을 합니다.  

> 위의 예제에서 ViewModel의 메서드로 상태가 변경되는 것은 가장 널리쓰이는 UDF의 구현 방식입니다.

### 추가 고려 사항

UI 상태를 노출할 때에는 다음의 내용들을 고려해야 합니다.

#### UI 상태 객체는 서로 연관된 상태들을 다루어야 합니다.

이를 지키면 상태의 불일치가 줄어들고 코드를 이해하기 더 수월해집니다. 만약 뉴스 기사 목록과 북마크한 개수를 서로 다른 스트림에 노출시킨다면, 한 스트림이 제대로 업데이트 되지 않는 오류가 발생할 수 있습니다. 이를 방지하기 위해서 단일 스트림에 두 개를 노출시켜야 합니다. 또한 일부 비즈니스 로직에는 여러 개의 소스들의 조합이 필요할 수도 있습니다. 예를 들어, 사용자가 로그인한 상태이면서 프리미엄 뉴스 구독자인 경우에만 북마크 버튼을 보여줘야 하는 경우가 있습니다. 이때 다음과 같이 UI 상태를 정의할 수 있습니다.  

```kotlin
data class NewsUiState(
    val isSignedIn: Boolean = false,
    val isPremium: Boolean = false,
    val newsItems: List<NewsItemUiState> = listOf()
)

val NewsUiState.canBookmarkNews: Boolean get() = isSignedIn && isPremium
```

북마크 버튼의 표시 여부를 두 개의 속성을 조합하여 결정하고 있습니다. 비즈니스 로직이 복잡해짐에 따라 모든 속성을 즉시 사용할 수 있는 단일 `UiState`클래스가 점점 더 중요해지고 있습니다.  

#### UI 상태: 단일 스트림 또는 복수 스트림

UI 상태를 단일과 복수 스트림 중 무엇으로 노출시켜야 할지를 결정할 때 중요한 것은 앞서 방출되는 항목 간의 관계를 살피는 것 입니다. 단일 스트림 노출의 장점은 편의성과 데이터 일관성에 입니다. 상태를 소비하는 측에서는 항상 최신의 정보를 사용할 수 있습니다. 하지만 분리된 상태 스트림을 쓰는게 더 적절한 경우도 있습니다.

- 서로 관련이 없는 데이터 유형일 때
  - UI를 표시하기 위해 필요한 일부 상태는 서로 관련이 없을 수도 있습니다.
  - 이러한 상태 중 하나가 다른 것보다 훨씬 빈번하게 업데이트 된다면 스트림을 분리하는게 더 낫습니다. 서로 다른 상태를 묶는 비용이 이점보다 더 클 수 있기 때문입니다.
- UI 상태 차이
  - `UiState` 객체에 필드가 많을수록 필드 중 하나가 업데이트될 때 스트림이 생성될 가능성이 높아집니다. 뷰는 연속적인 방출이 같거나 다른지의 여부를 파악할 수 없기 때문에, 모든 방출은 뷰의 업데이트를 발생시킵니다. 이러한 부분은  `Flow` 또는 `LiveData`의 `distinctUntilChanged`같은 메서드를 사용하여 완화시킬 수 있습니다.


## UI 상태를 사용(소비)

UI에서 `UiState` 객체의 스트림을 사용하려면, 사용중인 관찰 가능한 데이터 유형에 대해 종단(terminal) 연산자를 사용하면 됩니다. 예를 들어, `LiveData`에는 `observe()`메서드를, `Flow`에는 `collect()`메서드, 또는 그 변형을 사용합니다.  

UI에서 관찰 가능한 Data holders를 사용할 때는, UI의 생명 주기를 따라야 합니다. 뷰가 화면에 보이지 않을 때는 UI가 UI 상태를 관찰하지 않아야 하기 때문입니다. 이와 관련된 내용은 [A safer way to collect flows from Android UIs](https://medium.com/androiddevelopers/a-safer-way-to-collect-flows-from-android-uis-23080b1f8bda) 페이지를 추천드립니다. `LiveData`를 쓰는 경우에는, `LifecyclerOwner`는 암시적으로 생명 주기 문제를 처리하며, `Flow`를 쓴다면 적절한 `Coroutine scope`와 `repeatOnLifecycle API`를 사용하여 생명 주기에 따른 로직을 처리해야 합니다.  

```kotlin
// Views
class NewsActivity : AppCompatActivity() {

    private val viewModel: NewsViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect {
                    // UI 요소를 업데이트
                }
            }
        }
    }
}

// Compose
@Composable
fun LatestNewsScreen(
    viewModel: NewsViewModel = viewModel()
) {
    // viewModel.uiState에 따라 UI 요소를 업데이트
}
```

> 예제의 `StateFlow` 객체는 활성화된 수집기가 없을 때 작업을 중단하지 않지만, flow로 작업할 때 그것들이 어떻게 구현되는지 모를 수 있습니다. 생명 주기를 인식하는 flow를 사용하면 나중에 ViewModel flow에 이러한 변경을 하더라도 하위 수집 코드를 다시 검토할 필요 없이 조정할 수 있습니다.

### 작업 중임을 나타내기

`UiState`에서 로딩중이라는 것을 나타내는 가장 쉬운 방법은 `boolean` 필드를 사용하는 것입니다.  

```kotlin
data class NewsUiState(
    val isFetchingArticles: Boolean = false,
    ...
)

이 flag의 값은 UI에서 progress bar의 표시 여부를 나타냅니다.  

```kotlin
// Views
class NewsActivity : AppCompatActivity() {

    private val viewModel: NewsViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        ...

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                // progressBar의 가시성을 isFetchingArticles의 상태와 결합
                viewModel.uiState
                    .map { it.isFetchingArticles }
                    .distinctUntilChanged()
                    .collect { progressBar.isVisible = it }
            }
        }
    }
}

// Compose
@Composable
fun LatestNewsScreen(
    modifier: Modifier = Modifier,
    viewModel: NewsViewModel = viewModel()
) {
    Box(modifier.fillMaxSize()) {

        if (viewModel.uiState.isFetchingArticles) {
            CircularProgressIndicator(Modifier.align(Alignment.Center))
        }

        // 리스트같은 다른 UI 요소를 업데이트
    }
}
```

### 화면에서 오류 발생을 나타내기

화면에서 오류 발생을 표시하는 것과 로딩 중임을 표시하는 것 모두 유사합니다. 왜냐하면 둘다 `boolean` 필드로 간단히 표현될 수 있기 때문입니다. 그러나 오류는 때에 따라서 사용자에게 관련한 메시지를 보여주거나 실패한 작업을 재시도하는 작업을 포함하기도 합니다. 따라서 진행 중인 작업을 로딩 중이거나 로딩 중이지 않은 두 가지로 분류하는 것처럼 오류 상태는 오류와 관련된 데이터를 포함하는 데이터 클래스로 모델링될 필요가 있습니다.  

예를 들어, 기사 목록을 가져오고 있을 때 progress bar를 보여주는 작업을 생각해보세요. 만약 작업이 실패한다면 사용자에게 오류 메시지를 보여줘야 할 수 있습니다.  

```kotlin
data class Message(val id: Long, val message: String)

data class NewsUiState(
    val userMessages: List<Message> = listOf(),
    ...
)
```

오류 메시지는 snackbar와 같은 UI 요소로 사용자에게 보여져야 합니다. 왜냐하면 이는 UI 요소가 어떻게 생성/소비되는지와 연관되어있기 때문입니다. 더 자세한 내용은 [UI events](https://developer.android.com/jetpack/guide/ui-layer/events) 페이지를 참고하세요.  

## 스레드와 동시성

ViewModel에서 수행되는 작업은 메인 스레드에서 호출되어도 안전해야합니다. 왜냐하면 Data/Domain 계층은 다른 스레드로 작업을 넘기는 것에 대한 책임이 있기 때문입니다.  

만약 ViewModel에서 오래 걸리는 작업을 수행한다면, 백그라운드 스레드에서 수행하는게 좋습니다. 코루틴은 동시 작업을 관리하는 데 좋은 방법입니다. 그리고 Jetpack 아키텍처 컴포넌트는 이를 위한 기능을 지원하고 있습니다. [Kotlin coroutines on Android](https://developer.android.com/kotlin/coroutines) 페이지를 살펴보세요.

## 네비게이션

앱 네비게이션의 변경은 종종 이벤트와 같은 방출로 인해 일어납니다. 예를 들어, `SignInViewModel`에서 로그인 작업을 수행한 후에 UI 상태 클래스는 `isSignedIn`의 값을 `true`로 설정되어야 합니다. 이러한 트리거는 [UI 상태를 사용(소비)]()에서 다룬 것과 같이 소비되어야 하지만, 소비 로직의 구현은 [Navigation component](https://developer.android.com/guide/navigation)에 의존해야 합니다.

## 페이징

[Paging library](https://developer.android.com/topic/libraries/architecture/paging/v3-overview)는 `PagingData`라는 타입으로 UI에서 소비됩니다. `PagingData`는 시간이 지남에 따라 바뀔 수 있는 데이터를 보여주고 표현한다는 점에서 불변 타입이 아니므로 불변 UI 상태로 표시되어선 안됩니다. 대신 독립적인 스트림으로 ViewModel에서 노출시켜야 합니다. 자세한 내용은 [Android Paging](https://developer.android.com/codelabs/android-paging)을 참고해보세요.

## 애니메이션

자연스럽고 부드러운 최상위 수준의 네비게이션 전환을 제공하려면, 애니메이션이 시작되기 전에 데이터를 불러고기 위해 잠시동안 화면을 대기 상태로 만들어야 합니다. Android View 프레임워크는 `postponeEnteTransition()`, `startPostponedEnterTransition()`으로 프래그먼트 목적지 간 전환에 지연을 시키는 기능을 제공합니다. 이런 API는 두 번째 화면(네트워크에서 가져온 사진)으로 전환하는 애니메이션을 시작하기 전에 화면의 UI 요소가 준비되었는지 확인하는 방법을 제공합니다. 자세한 내용은 [Android Motion sample](https://github.com/android/animation-samples/tree/main/Motion)을 참고해보세요.

## 샘플

|          [Sunflower with Compose](https://github.com/android/sunflower/tree/main)           |                      [Now in Android App](https://github.com/android/nowinandroid/tree/main)                      | [Architecture starter template(single module)](https://github.com/android/architecture-templates/tree/base) |
| :-----------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------: |
| ![](https://raw.github.com/android/sunflower//main//screenshots/SunflowerM3Screenshots.png) |                ![](https://raw.github.com/android/nowinandroid//main//docs/images/screenshots.png)                |               ![](https://github.com/android/architecture-templates/raw/main/screenshots.png)               |
|          [Architecture](https://github.com/android/architecture-samples/tree/main)          | [Architecture starter template(multi module)](https://github.com/android/architecture-templates/tree/multimodule) |             [Jetcaster sample](https://github.com/android/compose-samples/tree/main/Jetcaster)              |
| ![](https://raw.github.com/android/architecture-samples//main//screenshots/screenshots.png) |                  ![](https://github.com/android/architecture-templates/raw/main/screenshots.png)                  |          ![](https://raw.github.com/android/compose-samples//main/Jetcaster//docs/screenshots.png)          |