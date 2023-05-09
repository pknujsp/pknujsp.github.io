---
layout: post
title: Android Compose 첫 도입 후기
subtitle: View 만 사용하다가 Compose 첫 사용
published: true
categories: Compose
tags: [Android]
---

<img src="https://github.com/pknujsp/Blogcomments/assets/48265129/60052a35-db1e-4404-b7c3-615d6d1eadd1" height="30%" width="auto">

### 먼저, 코드를 보면 전체적인 구조가 기존 View와는 상당히 다른 것을 알 수 있습니다.
---

### 대한민국 식약처가 제약사에 대해 회수/폐기와 행정 처분을 내린 목록을 표시하는 화면입니다.


**예제 코드 클래스 구성**
* NewsFragment
  * 뉴스 화면을 표시할 메인 Fragment
* NewsScreen
  * 위 Fragment에 나타나는 Compose 기반 뉴스 화면
* RecallSuspensionScreen
  * 회수 폐기 목록 화면
* RecallSuspensionViewModel
  * 위 화면에서 쓰이는 ViewModel


View위에서도 Compose를 사용할 수 있습니다!

[How to use Compose in XML layout in Android](https://pknujsp.github.io/android/2023/04/04/How-to-use-Compose-in-Fragment-in-Android.html)


### NewsScreen

```kotlin

/**
 * 뉴스 타입
 */
enum class ChipType {
    RECALLS_SUSPENSION, ADMIN_ACTION
}

/**
 * 뉴스 화면
 */
@Preview
@Composable
fun NewsScreen() {
    var selectedChip by remember { mutableStateOf(ChipType.RECALLS_SUSPENSION) }

    Column {
        Text(
            text = stringResource(id = com.android.mediproject.core.ui.R.string.news),
            style = MaterialTheme.typography.headlineMedium,
            modifier = Modifier.padding(top = 16.dp, start = 24.dp, end = 24.dp)
        )

        ChipGroup(selectedChip, onChipSelected = { chip ->
            selectedChip = chip
        })
        if (selectedChip == ChipType.RECALLS_SUSPENSION) RecallDisposalScreen()
        else Text(text = "AdminAction")

    }
}

/**
 * 뉴스 타입 선택
 *
 * @param selectedChip 선택된 뉴스 타입
 * @param onChipSelected 뉴스 타입 선택 시 호출되는 콜백
 */
@Composable
fun ChipGroup(selectedChip: ChipType, onChipSelected: (ChipType) -> Unit) {
    Row(
        verticalAlignment = Alignment.CenterVertically, modifier = Modifier.padding(top = 16.dp, bottom = 8.dp, start = 24.dp, end = 24.dp)
    ) {
        CustomFilterChip(
            title = stringResource(id = R.string.recallSuspension),
            isSelected = selectedChip == ChipType.RECALLS_SUSPENSION,
            type = ChipType.RECALLS_SUSPENSION
        ) {
            onChipSelected(if (selectedChip == ChipType.RECALLS_SUSPENSION) ChipType.RECALLS_SUSPENSION else ChipType.ADMIN_ACTION)
        }
        Spacer(Modifier.width(8.dp))
        CustomFilterChip(
            title = stringResource(id = R.string.adminAction),
            isSelected = selectedChip == ChipType.ADMIN_ACTION,
            type = ChipType.ADMIN_ACTION
        ) {
            onChipSelected(if (selectedChip == ChipType.RECALLS_SUSPENSION) ChipType.RECALLS_SUSPENSION else ChipType.ADMIN_ACTION)
        }
    }
}

/**
 * 뉴스 타입 Chip
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CustomFilterChip(type: ChipType, title: String, isSelected: Boolean, onClick: (ChipType) -> Unit) {
    FilterChip(
        selected = isSelected,
        onClick = { onClick.invoke(type) },
        label = { Text(title, fontSize = 13.sp) },
        shape = RoundedCornerShape(36.dp),
        colors = FilterChipDefaults.filterChipColors(
            selectedContainerColor = Color.Blue,
            selectedLabelColor = Color.White,
            disabledContainerColor = Color.White,
            disabledLabelColor = Color.Blue
        ),
    )
}
```

### NewsFragment

```kotlin
NewsFragment.kt

@AndroidEntryPoint
class NewsFragment : BaseFragment<FragmentNewsBinding, NewsViewModel>(FragmentNewsBinding::inflate) {

    override val fragmentViewModel: NewsViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.apply {
            lifecycleOwner = viewLifecycleOwner
            composeView.setContent {
                NewsScreen()
            }
        }
    }
}
```

### RecallSuspensionScreen

```kotlin
회수 폐기 chip클릭 시 나오는 화면
RecallSuspensionScreen.kt

/**
 * 회수 폐기 목록 표시
 */
@Preview
@Composable
fun RecallDisposalScreen(viewModel: RecallSuspensionViewModel = hiltViewModel()) {

    val list = viewModel.recallDisposalList.collectAsLazyPagingItems()

    LazyColumn(modifier = Modifier.fillMaxWidth()) {
        items(
            count = list.itemCount, key = list.itemKey(), contentType = list.itemContentType(
            )
        ) { index ->
            list[index]?.let { ListItem(it) }
            if (index < list.itemCount - 1) Divider(modifier = Modifier.padding(horizontal = 24.dp))
        }

        when (list.loadState.append) {
            is LoadState.NotLoading -> Unit
            is LoadState.Loading -> {
                item {
                    Column(horizontalAlignment = Alignment.CenterHorizontally) {
                        CircularProgressIndicator()
                    }
                }
            }

            is LoadState.Error -> TODO()
            else -> TODO()
        }
    }
}

/**
 * 회수 폐기 목록 아이템
 *
 * @param recallSuspensionListItemDto 회수 폐기 목록 아이템
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ListItem(recallSuspensionListItemDto: RecallSuspensionListItemDto) {
    Surface(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 24.dp, vertical = 9.dp),
        shape = RectangleShape,
        onClick = {
            recallSuspensionListItemDto.onClick?.invoke(recallSuspensionListItemDto)
        },
    ) {
        Column(
            modifier = Modifier.fillMaxWidth()
        ) {
            Row(
                horizontalArrangement = Arrangement.spacedBy(8.dp),
                verticalAlignment = CenterVertically,
            ) {
                Text(
                    text = recallSuspensionListItemDto.product,
                    style = MaterialTheme.typography.titleMedium,
                    fontSize = 14.sp,
                    color = Color.Black,
                    modifier = Modifier
                        .align(Alignment.CenterVertically)
                        .weight(1f),
                    overflow = TextOverflow.Ellipsis,
                    maxLines = 1
                )
                Text(
                    text = recallSuspensionListItemDto.let {
                        if (it.recallCommandDate != null) it.recallCommandDate
                        else it.rtrlCommandDt
                    }!!.toJavaLocalDate().format(dateFormat),
                    fontSize = 12.sp,
                    modifier = Modifier.align(Alignment.CenterVertically),
                    color = Color.Gray,
                    maxLines = 1,
                )
            }
            Spacer(modifier = Modifier.height(8.dp))
            Text(
                text = recallSuspensionListItemDto.rtrvlResn, fontSize = 12.sp, color = Color.Gray, maxLines = 1
            )
        }
    }
}

private val dateFormat = DateTimeFormatter.ofPattern("yyyy-MM-dd")
```

### 결과 화면
---
<img width="70%"  height="auto" src="https://github.com/pknujsp/Blogcomments/assets/48265129/9c02c47f-6567-4c79-958d-5391cfc1a077">


### View, Compose 비교
---

ViewModel은 View, Compose 모두 같습니다.

Compose를 사용하면서 알게된 View와 Compose의 차이점 및 장단점 입니다.

* View의 장점
  1. 안드로이드 개발의 기존 방식으로, 다양한 자료와 커뮤니티 지원이 있습니다.
  2. 현재까지의 모든 안드로이드 버전과 호환됩니다.

* View의 단점
  1. 상태 관리와 레이아웃 업데이트를 수동으로 처리해야 합니다.
  2. XML을 사용하기 때문에 코드와 레이아웃 파일 간에 이동이 필요합니다.
  3. 코드가 깁니다.

* Compose의 장점
  1. 선언적 UI로 인해 상태 관리가 쉽고, 코드 가독성이 높습니다.
  2. 코틀린 DSL을 사용하여 코드 작성과 동시에 UI를 프리뷰할 수 있습니다.
  3. 커스텀 뷰를 쉽게 만들 수 있으며, 재사용이 용이합니다.
  4. Compose를 사용하면 UI 작성 시 더 성능이 좋습니다.

* Compose의 단점
  1. 최소 API 레벨 21(Android 5.0 Lollipop) 이상만 지원합니다.
  2. 상대적으로 새로운 기술로서, 자료와 커뮤니티 지원이 View에 비해 상대적으로 적습니다.

### Compose의 주요 개념
---
* State
* @Composable 함수로 화면 레이아웃을 구성
* Modifier
* Row, Column

### State(상태)
---
Compose는 상태에 따라 UI가 자동으로 갱신 됩니다.

상태를 관리하기 위해 remember와 mutableStateOf와 같은 함수를 사용하여 상태를 저장하고 변경할 수 있습니다.

상태가 변경되면, 관련된 Composable 함수가 자동으로 재구성되어 UI를 갱신 합니다.


### State 와 ViewModel
---
State와 ViewModel은 서로 유사한 역할을 합니다.

ViewModel에서 LiveData, Flow등으로 관련 화면의 생명주기 동안 필요한 데이터를 저장하고 유지하는 것이 가능한 것 처럼,

Compose의 State도 이와 유사한 역할을 합니다.

ViewModel에서 데이터를 저장 유지 하는 기능에 특화되었다고 생각하면 됩니다.

```kotlin
var selectedChip by remember { mutableStateOf(ChipType.RECALLS_SUSPENSION) }
```

위 코드는 enum class 의 속성을 State로 관리하는 것을 의미합니다.

만약 selectedChip의 값이 바뀐다면 자동으로 selectedChip의 값을 사용하는 레이아웃을 갱신합니다.



### @Composable 함수로 화면 레이아웃을 구성
---
Compose에서 UI는 작은 조각으로 나누어진 Composable 함수로 구성됩니다.

이 함수를 통해 레이아웃을 구성합니다. 

Composable 함수는 다른 Composable 함수를 호출하여 계층적인 레이아웃을 구성하는 것이 가능합니다.

ViewGroup내에 ViewGroup을 선언하는 것과 같은 역할입니다.


```kotlin
@Composable
fun RecallDisposalScreen(viewModel: RecallSuspensionViewModel = hiltViewModel()) {
}
```

위와 같은 방식으로 코드를 작성합니다.

>Compose에서 ViewModel을 사용하려면 매개변수에 위와 같이 viewmodel()을 하면됩니다.


### Modifier
---
Composable 함수의 레이아웃 디자인에 관한 설정을 할 때 사용합니다.

여러 가지 Modifier를 사용하여 레이아웃, 패딩, 배경색 등의 속성을 설정할 수 있습니다.

```kotlin
      modifier = Modifier.padding(top = 16.dp, start = 24.dp, end = 24.dp)
```

위 내용은, View에서 setPadding() 또는 XML내에서 android:padding 속성으로 지정하는 것을 의미합니다.

### Row, Column
---

* Row
  * 수평 방향으로 레이아웃을 배치할 수 있습니다.
  * LinearLayout 속성을 Horizontal로 설정한 것과 유사합니다.
* Column
  * 수직 방향으로 레이아웃을 배치할 수 있습니다.
  * LinearLayout 속성을 Vertical로 설정한 것과 유사합니다.

위 두 개를 이용해서 List화면을 구성하는 것이 가능합니다.

그러나 모든 List의 Item을 로드하여 화면에 표시하기 때문에 오버헤드가 큰 **문제점**이 있습니다.

View에서 **ListView**를 사용하는 것과 같다고 생각하면 됩니다.

오버헤드를 줄이기 위해 **LazyColumn**, **LazyRow**를 사용하는 것이 좋습니다.

**LazyColumn**와 **LazyRow** 는 View의 **RecyclerView**와 같은 역할을 합니다.

### 정리
---

써보니 생각보다 어렵지 않고, 코드가 확실히 줄어서 편합니다.

## 하루라도 빨리 Compose로 개발을 시작해야 하는게 좋을 듯합니다!