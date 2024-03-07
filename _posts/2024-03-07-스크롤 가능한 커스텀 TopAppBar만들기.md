---
layout: post
title: 스크롤 가능한 커스텀 TopAppBar만들기
subtitle: Scrollable custom TopAppBar
published: true
categories: [Android]
tags: [Android, Compose]
---

## 다음과 같이 스크롤 할 수 있는 TopAppBar를 만들어 보고자 합니다.

![](https://github.com/pknujsp/pknujsp/assets/48265129/19c9e7b8-1c58-4886-9291-dad3e2104703)

## 만들게된 계기

- 자체적으로 LargeTopAppBar를 제공하지만 가장 큰 Title 컴포저블의 높이가 최대 `152dp`로 제한되어 있다.
- 아래 코드는 Compose material3의 TopAppBar를 구현한 클래스의 코드 중 일부이다. 보다시피 최대 높이인 `ContainerHeight`가 `152.0.dp`로 고정되어 있다. 이로 인해서 더 큰 높이의 Title 컴포저블을 사용하면 아래 일부가 잘리게 되는 문제가 생긴다.
- 또한 기본적으로 이 컴포저블은 material3의 스타일을 따른 것이므로 커스텀이 제한된다.
- 아래처럼 좀 더 높이가 큰 TopAppBar가 필요하고, 제한을 벗어나 커스텀이 가능한게 필요하여 구현하게 되었다.

|                                           기본제공                                           |                                      구현하고자 하는 것                                      |
| :------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------: |
| ![](https://developer.android.com/static/images/jetpack/compose/components/appbar-large.svg) | ![](https://github.com/pknujsp/pknujsp/assets/48265129/fa263fe9-cfa4-4ba5-b463-70c74b4008bf) |


```kotlin
// package androidx.compose.material3.tokens.TopAppBarLargeTokens.kt
internal object TopAppBarLargeTokens {
    ...
    val ContainerHeight = 152.0.dp
    ...
}

// package androidx.compose.material3.AppBar.kt
@OptIn(ExperimentalMaterial3Api::class)
@Composable
private fun TwoRowsTopAppBar(
    ...
    maxHeight: Dp,
    pinnedHeight: Dp,
    ...
) {
    val pinnedHeightPx: Float // 작은 Title 컴포저블의 높이
    val maxHeightPx: Float // 허용하는 최대 Title 컴포저블의 높이
    val titleBottomPaddingPx: Int
    LocalDensity.current.run {
        pinnedHeightPx = pinnedHeight.toPx()
        maxHeightPx = maxHeight.toPx()
        titleBottomPaddingPx = titleBottomPadding.roundToPx()
    }
```

## 구현

### 먼저 기본 TopAppBar의 구현 코드를 살펴보자.

#### `LargeTopAppBar` 컴포저블의 구현은 다음과 같다.

- Scaffold에서 `topBar`를 사용하여 TopAppBar를 배치할 때 이 함수를 사용한다.
- 실제 구현하는 컴포저블 함수를 따로 `TwoRowsTopAppBar`로 구현하고 있다.

```kotlin
@ExperimentalMaterial3Api
@Composable
fun LargeTopAppBar(
    title: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    navigationIcon: @Composable () -> Unit = {},
    actions: @Composable RowScope.() -> Unit = {},
    windowInsets: WindowInsets = TopAppBarDefaults.windowInsets,
    colors: TopAppBarColors = TopAppBarDefaults.largeTopAppBarColors(),
    scrollBehavior: TopAppBarScrollBehavior? = null
) {
    TwoRowsTopAppBar(
        title = title,
        titleTextStyle = MaterialTheme.typography.fromToken(TopAppBarLargeTokens.HeadlineFont),
        smallTitleTextStyle = MaterialTheme.typography.fromToken(TopAppBarSmallTokens.HeadlineFont),
        titleBottomPadding = LargeTitleBottomPadding,
        smallTitle = title,
        modifier = modifier,
        navigationIcon = navigationIcon,
        actions = actions,
        colors = colors,
        windowInsets = windowInsets,
        maxHeight = TopAppBarLargeTokens.ContainerHeight,
        pinnedHeight = TopAppBarSmallTokens.ContainerHeight,
        scrollBehavior = scrollBehavior
    )
}
```

#### `TwoRowsTopAppBar`은 다음과 같이 구현되어 있다.

- 먼저 높이를 체크하고
- dp인 값을 px로 변환한다.
- 하단 타이틀의 접히고 펼치는 동작이 자연스럽게 이루어지도록 드래그 시 y축의 드래그 양의 상한을 조정한다.
- 앱 바가 접힌 상태를 `0.0 - 1.0` 사이의 값으로 결정한다.
- 컴포저블의 경우 Surface에 Column을 배치하고 순서대로 상단 타이틀과 하단 타이틀을 배치한다.


```kotlin
@Composable
private fun TwoRowsTopAppBar(
    modifier: Modifier = Modifier,
    title: @Composable () -> Unit,
    titleTextStyle: TextStyle,
    titleBottomPadding: Dp,
    smallTitle: @Composable () -> Unit,
    smallTitleTextStyle: TextStyle,
    navigationIcon: @Composable () -> Unit,
    actions: @Composable RowScope.() -> Unit,
    windowInsets: WindowInsets,
    colors: TopAppBarColors,
    maxHeight: Dp,
    pinnedHeight: Dp,
    scrollBehavior: TopAppBarScrollBehavior?
) { 
    // maxHeight가 pinnedHeight 이하일 경우 예외를 던진다.
    // 이 부분은 private 내부 함수이므로, 개발자가 직접 API에서 접근이 불가능하다.
    if (maxHeight <= pinnedHeight) { ... }

    // 높이를 픽셀로 변환
    val pinnedHeightPx: Float
    val maxHeightPx: Float
    val titleBottomPaddingPx: Int
    LocalDensity.current.run {
        pinnedHeightPx = pinnedHeight.toPx()
        maxHeightPx = maxHeight.toPx()
        titleBottomPaddingPx = titleBottomPadding.roundToPx()
    }

    // 앱 바가 드래그 될 때 접히기 시작해야 하는 y축 드래그 양의 상한을 조정해서 상단, 하단 타이틀 영역이 올바르게 보이도록 한다.
    SideEffect {
        if (scrollBehavior?.state?.heightOffsetLimit != pinnedHeightPx - maxHeightPx) {
            scrollBehavior?.state?.heightOffsetLimit = pinnedHeightPx - maxHeightPx
        }
    }

    // 스크롤 동작이 발생할 때, 상하단 타이틀의 가시성을 조정하기 위한 값으로 사용된다.
    // 이 값이 0.5f 라면 반 정도 접힌 상태, 1.0f 라면 완전히 펼쳐진 상태를 의미한다.
    val colorTransitionFraction = scrollBehavior?.state?.collapsedFraction ?: 0f
    val appBarContainerColor = colors.containerColor(colorTransitionFraction)

    // 액션 아이콘들을 Row로 묶어서 표시한다.
    val actionsRow = @Composable {
        Row(
            horizontalArrangement = Arrangement.End,
            verticalAlignment = Alignment.CenterVertically,
            content = actions
        )
    }
    val topTitleAlpha = TopTitleAlphaEasing.transform(colorTransitionFraction)
    val bottomTitleAlpha = 1f - colorTransitionFraction

    // 하단 타이틀을 표시할지 여부를 결정하는 기준 값
    // 0.5f 이하일 경우 상단 타이틀을 표시하고, 그렇지 않을 경우 하단 타이틀을 표시한다.
    val hideTopRowSemantics = colorTransitionFraction < 0.5f
    val hideBottomRowSemantics = !hideTopRowSemantics

    // 앱 바에 드래그 동작을 가능하게 한다.
    val appBarDragModifier = if (scrollBehavior != null && !scrollBehavior.isPinned) {
        Modifier.draggable(
            orientation = Orientation.Vertical,
            state = rememberDraggableState { delta ->
                scrollBehavior.state.heightOffset = scrollBehavior.state.heightOffset + delta
            },
            onDragStopped = { velocity ->
                // 드래그가 멈췄을 때 앱 바의 확장 또는 접힘 동작을 결정한다.
                // colorTransitionFraction 값이 0.5f를 기준으로 접힐지 펼쳐질지 결정한다.
                // 이 때 애니메이션이 발생하게 된다.
                settleAppBar(
                    scrollBehavior.state,
                    velocity,
                    scrollBehavior.flingAnimationSpec,
                    scrollBehavior.snapAnimationSpec
                )
            }
        )
    } else {
        Modifier
    }

    Surface(modifier = modifier.then(appBarDragModifier), color = appBarContainerColor) {
        Column {
            TopAppBarLayout(
                modifier = Modifier
                    .windowInsetsPadding(windowInsets)
                    .clipToBounds(),
                heightPx = pinnedHeightPx,
                navigationIconContentColor =
                colors.navigationIconContentColor,
                titleContentColor = colors.titleContentColor,
                actionIconContentColor =
                colors.actionIconContentColor,
                title = smallTitle,
                titleTextStyle = smallTitleTextStyle,
                titleAlpha = topTitleAlpha,
                titleVerticalArrangement = Arrangement.Center,
                titleHorizontalArrangement = Arrangement.Start,
                titleBottomPadding = 0,
                hideTitleSemantics = hideTopRowSemantics,
                navigationIcon = navigationIcon,
                actions = actionsRow,
            )
            TopAppBarLayout(
                modifier = Modifier
                    .windowInsetsPadding(windowInsets.only(WindowInsetsSides.Horizontal))
                    .clipToBounds(),
                heightPx = maxHeightPx - pinnedHeightPx + (scrollBehavior?.state?.heightOffset
                    ?: 0f),
                navigationIconContentColor =
                colors.navigationIconContentColor,
                titleContentColor = colors.titleContentColor,
                actionIconContentColor =
                colors.actionIconContentColor,
                title = title,
                titleTextStyle = titleTextStyle,
                titleAlpha = bottomTitleAlpha,
                titleVerticalArrangement = Arrangement.Bottom,
                titleHorizontalArrangement = Arrangement.Start,
                titleBottomPadding = titleBottomPaddingPx,
                hideTitleSemantics = hideBottomRowSemantics,
                navigationIcon = {},
                actions = {}
            )
        }
    }
 }
```

#### TopAppBarLayout은 다음과 같이 구현되어 있다.

- `Layout`을 사용하여 `TopAppBar`의 레이아웃을 구성한다.
- 네비게이션 아이콘, 액션, 타이틀을 모두 따로 `Box`로 감싼다.
- 각각의 `Box`에 `layoutId`를 부여하여 각각의 `Box`를 구분한다.
- `layout` 함수를 사용하여 각각의 컴포넌트의 크기에 따라 `Box`의 위치를 결정한다.
- 하단 타이틀이 접히고 펼쳐지는 동작을 자연스럽게 하기 위해 `alpha`값을 가지고 투명도를 조절하는 방식을 사용한다.

```kotlin
@Composable
private fun TopAppBarLayout(
    modifier: Modifier,
    heightPx: Float,
    navigationIconContentColor: Color,
    titleContentColor: Color,
    actionIconContentColor: Color,
    title: @Composable () -> Unit,
    titleTextStyle: TextStyle,
    titleAlpha: Float,
    titleVerticalArrangement: Arrangement.Vertical,
    titleHorizontalArrangement: Arrangement.Horizontal,
    titleBottomPadding: Int,
    hideTitleSemantics: Boolean,
    navigationIcon: @Composable () -> Unit,
    actions: @Composable () -> Unit,
) {
    Layout(
        {
            Box(
                Modifier
                    .layoutId("navigationIcon")
                    .padding(start = TopAppBarHorizontalPadding)
            ) {
                CompositionLocalProvider(
                    LocalContentColor provides navigationIconContentColor,
                    content = navigationIcon
                )
            }
            Box(
                Modifier
                    .layoutId("title")
                    .padding(horizontal = TopAppBarHorizontalPadding)
                    .then(if (hideTitleSemantics) Modifier.clearAndSetSemantics { } else Modifier)
                    .graphicsLayer(alpha = titleAlpha)
            ) {
                ProvideContentColorTextStyle(
                    contentColor = titleContentColor,
                    textStyle = titleTextStyle,
                    content = title)
            }
            Box(
                Modifier
                    .layoutId("actionIcons")
                    .padding(end = TopAppBarHorizontalPadding)
            ) {
                CompositionLocalProvider(
                    LocalContentColor provides actionIconContentColor,
                    content = actions
                )
            }
        },
        modifier = modifier
    ) { measurables, constraints ->
        val navigationIconPlaceable =
            measurables.fastFirst { it.layoutId == "navigationIcon" }
                .measure(constraints.copy(minWidth = 0))
        val actionIconsPlaceable =
            measurables.fastFirst { it.layoutId == "actionIcons" }
                .measure(constraints.copy(minWidth = 0))

        val maxTitleWidth = if (constraints.maxWidth == Constraints.Infinity) {
            constraints.maxWidth
        } else {
            (constraints.maxWidth - navigationIconPlaceable.width - actionIconsPlaceable.width)
                .coerceAtLeast(0)
        }
        val titlePlaceable =
            measurables.fastFirst { it.layoutId == "title" }
                .measure(constraints.copy(minWidth = 0, maxWidth = maxTitleWidth))

        val titleBaseline =
            if (titlePlaceable[LastBaseline] != AlignmentLine.Unspecified) {
                titlePlaceable[LastBaseline]
            } else {
                0
            }

        val layoutHeight = if (heightPx.isNaN()) 0 else heightPx.roundToInt()

        layout(constraints.maxWidth, layoutHeight) {
            navigationIconPlaceable.placeRelative(
                x = 0,
                y = (layoutHeight - navigationIconPlaceable.height) / 2
            )

            titlePlaceable.placeRelative(
                x = when (titleHorizontalArrangement) {
                    Arrangement.Center -> {
                        var baseX = (constraints.maxWidth - titlePlaceable.width) / 2
                        if (baseX < navigationIconPlaceable.width) {
                            baseX += (navigationIconPlaceable.width - baseX)
                        } else if (baseX + titlePlaceable.width >
                            constraints.maxWidth - actionIconsPlaceable.width
                        ) {
                            baseX += ((constraints.maxWidth - actionIconsPlaceable.width) -
                                (baseX + titlePlaceable.width))
                        }
                        baseX
                    }

                    Arrangement.End ->
                        constraints.maxWidth - titlePlaceable.width - actionIconsPlaceable.width
                    else -> max(TopAppBarTitleInset.roundToPx(), navigationIconPlaceable.width)
                },
                y = when (titleVerticalArrangement) {
                    Arrangement.Center -> (layoutHeight - titlePlaceable.height) / 2
                    Arrangement.Bottom ->
                        if (titleBottomPadding == 0) layoutHeight - titlePlaceable.height
                        else layoutHeight - titlePlaceable.height - max(
                            0,
                            titleBottomPadding - titlePlaceable.height + titleBaseline
                        )
                    else -> 0
                }
            )

            actionIconsPlaceable.placeRelative(
                x = constraints.maxWidth - actionIconsPlaceable.width,
                y = (layoutHeight - actionIconsPlaceable.height) / 2
            )
        }
    }
}
```


### 기본적인 레이아웃은 TopAppBar의 형식을 그대로 따르므로, material3에 구현된 코드를 일부 가져왔다.

- 그대로 따오지는 않았으며, 일부 수정하여 사용하였다.

#### - 상단, 하단 타이블의 배치 방식을 수정하였다.

Column에 상단, 하단 타이틀이 배치된 부분을 수정하여 단일 컴포저블 함수로 만들었다.
왜냐하면, 상단, 하단 타이틀의 배치 방식이 다르지 않고, 상단 타이틀이 접히고 펼쳐지는 동작을 자연스럽게 하기 위해 `alpha`값을 가지고 투명도를 조절하는 방식을 사용하기 때문이다. 그래서 단일 함수로 만들어서 코드 길이를 줄이는 방향이 좋다고 판단하였다.


```kotlin
private val topAppBarHorizontalPadding = 4.dp
private val topAppBarTitleInset = 16.dp

@Composable
fun CustomTopAppBar(
    modifier: Modifier = Modifier,
    windowInsets: WindowInsets,
    colors: CustomTopAppBarColors,
    scrollState: ScrollState,
    bigTitle: @Composable () -> Unit,
    smallTitle: @Composable () -> Unit,
    navigationIcon: @Composable (() -> Unit)? = null,
    actions: @Composable (RowScope.() -> Unit)? = null,
) {
    val coroutineScope = rememberCoroutineScope()
    var bigTitleHeight by remember { mutableIntStateOf(0) }
    val collapsedFraction by remember {
        derivedStateOf { if (scrollState.value < bigTitleHeight) scrollState.value / bigTitleHeight.toFloat() else 1f }
    }
    val nestedScrollConnection = remember {
        object : NestedScrollConnection {
            override suspend fun onPostFling(consumed: Velocity, available: Velocity): Velocity {
                coroutineScope.onScroll(scrollState, bigTitleHeight, collapsedFraction)
                return super.onPostFling(consumed, available)
            }
        }
    }

    if (scrollState.isScrollInProgress && scrollState.value != 0) {
        DisposableEffect(scrollState.isScrollInProgress) {
            onDispose {
                coroutineScope.onScroll(scrollState, bigTitleHeight, collapsedFraction)
            }
        }
    }

    val actionsRow: @Composable (() -> Unit)? = actions?.run {
        @Composable { Row(verticalAlignment = Alignment.CenterVertically, content = this) }
    }

    val bigTitleBox: @Composable () -> Unit = {
        Box(modifier = Modifier.onGloballyPositioned {
            if (bigTitleHeight == 0) {
                bigTitleHeight = it.size.height
            }
        }) {
            bigTitle()
        }
    }

    Box(modifier = modifier.nestedScroll(nestedScrollConnection)) {
        TopAppBarLayout(
            modifier = Modifier
                .windowInsetsPadding(windowInsets),
            navigationIconContentColor = colors.navigationIconContentColor,
            actionIconContentColor = colors.actionIconContentColor,
            smallTitleAlpha = collapsedFraction,
            smallTitle = smallTitle,
            bigTitle = bigTitleBox,
            navigationIcon = navigationIcon,
            actions = actionsRow,
        )
    }
}
```


#### - 드래그 동작 부분이 가장 큰 문제였다. 이 부분은 직접 구현하였다.

기본 API 함수의 경우 드래그를 처리하는 코드가 internal 접근자로 되어 있는 클래스들과 복잡하게 연결되어 있어서 그대로 복사하는 것은 너무 많은 내용을 복사해야 해서 엄청 비효율적이라서 직접 구현하였다.

구현에 많은 시간이 소요되었다.

- 이 앱 바를 배치한 상위 컴포저블에서 scrollState를 함수 파라미터로 받아온다.
- 앱 바 영역이 펼치고 접힐 때 하단 타이틀 영역의 높이를 가지고 계산되어야 하므로 `onGloballyPositioned`를 사용하여 하단 타이틀 영역의 높이를 따로 저장한다.
- `nestedScrollConnection` 객체를 생성하여 앱 바 전체 영역을 가지는 루트 컴포저블에 `nestedScroll`을 적용한다.
- nestedScroll만 그대로 적용하면 날리는 식으로 스크롤을 할 때 제대로 동작이 발생하지 않고 끊기는 현상이 발생한다. 이를 해결하기 위해 `onPostFling`을 오버라이드하여 스크롤이 멈추면 앱 바의 확장 또는 접힘 동작을 결정한다. 이 때 애니메이션이 발생하게 된다.
- API의 `settleAppBar` 함수가 쓰이는 목적을 구현하기 위해 `DisposableEffect`를 사용하여 앱바가 중간에 펼쳐진 상태에서 스크롤이 멈췄을 때 앱바의 확장 또는 접힘 동작을 결정한다. 이 때도 애니메이션이 발생하게 된다.
- 최종 구현은 기본 API를 사용하였을 때와 차이가 없음을 확인하였다.


`CoroutineScope.onScroll` 함수에서 `scrollState`의 상태를 직접 갱신하여 앱 바를 드래그하여도 Column을 드래그하는 것과 똑같은 동작을 하도록 구현하였다. 실제 사용에서 화면 어느 영역을 드래그 하더라도 완전히 동일한 터치감이 나타난다.

`scrollState의 animateScrollTo` 메서드를 사용하면 `nestedScrollConnection`에서 따로 드래그 상태에 따른 로직을 구현하지 않더라도 자연스러운 드래그 동작을 만들 수 있다.


```kotlin
private const val BIG_TITLE = "bigTitle"
private const val SMALL_TITLE = "smallTitle"
private const val NAVIGATION_ICON = "navigationIcon"
private const val ACTION_ROW = "actionRow"

@Composable
private fun TopAppBarLayout(
    modifier: Modifier,
    navigationIconContentColor: Color,
    actionIconContentColor: Color,
    smallTitleAlpha: Float,
    bigTitle: @Composable () -> Unit,
    smallTitle: @Composable () -> Unit,
    navigationIcon: @Composable (() -> Unit)? = null,
    actions: @Composable (() -> Unit)? = null,
) {
    Layout({
        Box(Modifier
            .layoutId(NAVIGATION_ICON)
            .padding(start = topAppBarHorizontalPadding)) {
            navigationIcon?.run {
                CompositionLocalProvider(LocalContentColor provides navigationIconContentColor, content = this)
            }
        }
        Box(Modifier
            .layoutId(BIG_TITLE)
            .padding(horizontal = topAppBarHorizontalPadding)
            .graphicsLayer(alpha = 1f - smallTitleAlpha)) {
            bigTitle()
        }
        Box(Modifier
            .layoutId(SMALL_TITLE)
            .padding(horizontal = topAppBarHorizontalPadding)
            .graphicsLayer(alpha = smallTitleAlpha)) {
            smallTitle()
        }
        Box(Modifier
            .layoutId(ACTION_ROW)
            .padding(end = topAppBarHorizontalPadding)) {
            actions?.run {
                CompositionLocalProvider(LocalContentColor provides actionIconContentColor, content = this)
            }
        }
    }, modifier = modifier) { measurables, constraints ->
        val navigationIconPlaceable = measurables.first { it.layoutId == NAVIGATION_ICON }.measure(constraints.copy(minWidth = 0))
        val actionIconsPlaceable = measurables.first { it.layoutId == ACTION_ROW }.measure(constraints.copy(minWidth = 0))
        val bigTitlePlaceable = measurables.first { it.layoutId == BIG_TITLE }.measure(constraints.copy(minWidth = 0))
        val smallTitlePlaceable = measurables.first { it.layoutId == SMALL_TITLE }.measure(constraints.copy(minWidth = 0))

        val expandedRatio = 1f - (1f - smallTitleAlpha)
        val layoutHeight = navigationIconPlaceable.height + (bigTitlePlaceable.height * (1f - expandedRatio)).toInt()
        val titleInset = topAppBarTitleInset.roundToPx()

        layout(constraints.maxWidth, layoutHeight) {
            navigationIconPlaceable.place(x = 0, y = 0)
            actionIconsPlaceable.place(x = constraints.maxWidth - actionIconsPlaceable.width, y = 0)

            bigTitlePlaceable.place(x = (titleInset + navigationIconPlaceable.width * expandedRatio).toInt(),
                y = navigationIconPlaceable.height - (bigTitlePlaceable.height * expandedRatio).toInt())
            smallTitlePlaceable.place(x = navigationIconPlaceable.width + titleInset, y = (layoutHeight - smallTitlePlaceable.height) / 2)
        }
    }
}


private val animationSpec: AnimationSpec<Float> =
    SpringSpec(dampingRatio = Spring.DampingRatioLowBouncy, stiffness = Spring.StiffnessMediumLow)
private const val COLLAPSE_THRESHOLD = 0.5f

private fun CoroutineScope.onScroll(scrollState: ScrollState, shiftY: Int, collapsedFraction: Float) {
    if (scrollState.value < shiftY) {
        launch {
            scrollState.animateScrollTo(if (collapsedFraction < COLLAPSE_THRESHOLD) 0 else shiftY, animationSpec)
        }
    }
}
```


## 최종 구현

![](https://github.com/pknujsp/pknujsp/assets/48265129/19c9e7b8-1c58-4886-9291-dad3e2104703)