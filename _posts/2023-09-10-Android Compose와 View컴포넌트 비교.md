---
layout: post
title: Android Compose와 View컴포넌트 비교
subtitle: Compose components matching with View components
published: true
categories: Android
tags: [Android]
---

## Android Compose와 View컴포넌트 비교

XML만으로 개발하다가, Compose를 처음 접하게 되면 처음 보는 수 많은 컴포넌트들을 만나게 된다. 기존 View에 있던 것들과는 다른 이름을 가지고 있어서 어떤 컴포넌트가 어떤 역할을 하는지 학습에 시간이 다소 걸린다. 이번 글에서는 기존 View와 비교하여 Compose의 컴포넌트들을 살펴보고자 한다.


#### Compose의 컴포넌트 목록

- Box
- Column
- Row
- LazyColumn
- LazyRow
- Text
- Image
- TextField
- Card
- Scaffold
- 등등


#### View의 컴포넌트 목록

- FrameLayout
- LinearLayout
- RelativeLayout
- ConstraintLayout
- RecyclerView
- TextView
- ImageView
- EditText
- CardView
- ScrollView
- CoordinatorLayout
- 등등


Column, Box, Row, Scaffold는 특히 View에는 같은 이름을 가진 클래스가 없기 때문에 처음에는 어떤 역할을 하는지 파악하기 어려울 수도 있다.

### Compose와 View의 컴포넌트들을 같은 역할을 하는 것끼리 모으면 다음과 같다.

- Box: FrameLayout
- Column, Row: LinearLayout
- LazyColumn, LazyRow: RecyclerView
- Text: TextView
- Image: ImageView
- TextField: EditText
- Card: CardView
- Scaffold: CoordinatorLayout

ConstraintLayout은 같은 이름으로 Compose에서도 사용이 가능하다. `androidx.constraintlayout:constraintlayout-compose` 라이브러리를 추가하면 된다. 다른 Composable들 간의 상대적인 관계를 이용해 레이아웃을 구성하는 방식 역시 그대로 동일하다.

> View를 Compose에서는 Composable이라고 부른다.


### Box와 FrameLayout

Box와 FrameLayout은 같은 기능을 한다. 

FrameLayout에 layout_width, layout_height가 MATCH_PARENT인 TextView를 배치한 것과 같은 코드이다.

```kotlin
Box(
    modifier = Modifier
        .fillMaxSize()
) {
    Text("Box")
}
```

### Text와 TextView

Text는 TextView와 같은 기능을 한다. 

간단하게 "text"를 출력하는 코드이다.

```kotlin
Text("Text")
```

### Image와 ImageView

Image는 ImageView와 같은 기능을 한다.

`ic_launcher_background` Drawable을 그리는 코드이다.

```kotlin
Image(
    painter = painterResource(id = R.drawable.ic_launcher_background),
    contentDescription = "Image"
)
```

### TextField와 EditText

TextField는 EditText와 같은 기능을 한다.

`onValueChange`를 사용하여 입력된 값을 저장하며, 기본값은 "text"로 설정한다.

```kotlin
TextField(
    value = "text",
    onValueChange = { text = it },
    label = { Text("TextField") }
)
```


### ScrollView와 Modifier

ScrollView는 Compose에서 사용하려면 좀 다른 방법인 **Modifier**를 사용하여 구현한다. 먼저 **Modifier**를 간단히 알아보자.

View에서는 다음과 같이 레이아웃 속성을 정의한다.

```xml
<ScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

</ScrollView>
```

Compose에서는 다음과 같이 **Modifier**를 사용하여 레이아웃 속성을 정의한다.

```kotlin
Box(
    modifier = Modifier
        .fillMaxSize()
        .scrollable(
            state = rememberScrollState(),
            orientation = Orientation.Vertical
    )
) {
    
}
```

LayoutParams, Padding, Width, Height 등의 속성은 **Modifier**를 사용하여 정의한다. 레이아웃 속성 뿐만 아니라 다양한 속성을 정의하는 데에 쓰인다. ScrollView를 `Modifier.scrollable`를 사용해서 구현할 수 있다.

### View의 OnClickListener와 Compose의 Modifier.clickable

`onClickListener`의 경우는 `Modifier.clickable`을 사용하여 정의한다.

```kotlin
Box(
    modifier = Modifier
        .fillMaxSize()
        .clickable {
           
        }
) {
    
}
```

### RecyclerView와 LazyList

RecyclerView와 같은 기능은 Compose에서 LazyList인 `LazyColumn`과 `LazyRow`를 사용하여 구현한다. `LazyColumn`은 세로 스크롤 리스트, `LazyRow`는 가로 스크롤 리스트를 구현할 때 사용한다.

LazyList의 기본 동작 원리는 RecyclerView와 유사하게 화면에 보여지고 있는 영역에 대해서만 Item을 그리는 방식을 사용한다.

#### Column, Row 뿐만 아니라 Grid도 제공한다.

RecyclerView의 LayoutManager와 대응되는 Compose의 LazyList는 다음과 같다.

| LayoutManager              | Composable                                              |
| -------------------------- | ------------------------------------------------------- |
| LinearLayoutManager        | LazyColumn, LazyRow                                     |
| GridLayoutManager          | LazyVerticalGrid, LazyHorizontalGrid                    |
| StaggeredGridLayoutManager | LazyVerticalStaggeredGrid , LazyHorizontalStaggeredGrid |


아래는 Text를 100개 만들어서 세로 스크롤 리스트를 구현하는 코드이다.


```kotlin
LazyColumn {
    items(100) {
        Text("Item #$it")
    }
}
```


`LazyColumn`을 `LazyRow`로 바꾸면 가로 스크롤 리스트가 된다.
