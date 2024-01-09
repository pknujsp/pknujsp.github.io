---
layout: post
title: 자동으로 폰트 크기 조절되는 Text Composable만들기
subtitle: AutoAdjustingFontSizeText Composable
published: true
categories: [Android]
tags: [Android, Compose, Text, AutoAdjustingFontSize]
---

## 자동으로 폰트 크기 조절되는 Text Composable만들기

![](https://github.com/pknujsp/pknujsp/assets/48265129/54a14f2f-723a-4b9b-b310-c95973b3fd62)

## 필요성

|                                                                                                     |                                                                                                     |
| :-------------------------------------------------------------------------------------------------: | :-------------------------------------------------------------------------------------------------: |
| <img src="https://github.com/pknujsp/pknujsp/assets/48265129/154a7984-643f-4e67-9a5c-bc5d40514b60"> | <img src="https://github.com/pknujsp/pknujsp/assets/48265129/53127974-2c00-4f2a-b654-037b9fea7e46"> |

Text를 화면에 보여줄 때, 값의 길이가 길어지면 위와 같이 내용이 잘리게 된다.(`maxLines = 1`인 경우)

잘리지 않게하려면 길이에 따라 폰트 크기를 조절한 후 Text를 그리면 된다.

이를 위해 다음 단계를 따르면 된다.
- `BoxWithConstraints`Composable 내에 `Text` Composable을 배치한다.
  - `BoxWithConstraints`를 사용하면 화면상에 그려질 수 있는 크기를 알 수 있다.
- `TextMeasurer`를 사용해 Text가 화면에서 실제로 보여지는 크기를 구하고, 폰트 크기를 바꿔준다.

### 구현 1

```kotlin
@Composable
fun AutoText(
    modifier: Modifier = Modifier,
    text: String,
    style: TextStyle,
    overflow: TextOverflow = TextOverflow.Clip,
    minFontSize: Int = MIN_AUTO_SIZING_TEXT_SIZE,
    defaultFontSize: Int = DEFAULT_AUTO_SIZING_TEXT_SIZE,
    step: Int = 1
) {
    BoxWithConstraints(modifier = modifier) {
        val textMeasurer = rememberTextMeasurer()
        val textOverflow = remember(overflow) { if (overflow == TextOverflow.Ellipsis) TextOverflow.Clip else overflow }
        var textStyle by remember(text) { mutableStateOf(style.copy(fontSize = defaultFontSize.sp)) }

        LaunchedEffect(textStyle) {
            textMeasurer.measure(text, textStyle).run {
                if (textStyle.fontSize.value.toInt() > minFontSize && size.width >= constraints.maxWidth || size.height >= constraints.maxHeight) {
                    val newFontSize = (textStyle.fontSize.value.toInt() - step).coerceAtLeast(minFontSize)
                    textStyle = textStyle.copy(fontSize = newFontSize.sp)
                }
            }
        }

        Text(text = text, maxLines = 1, style = textStyle, overflow = textOverflow)
    }
}
```

- `val textMeasurer = rememberTextMeasurer()` : Text의 크기를 측정하는 데 쓰인다.
- `var textStyle by remember(text) { mutableStateOf(style.copy(fontSize = defaultFontSize.sp)) }` : `textStyle`을 조정해야 하므로 **State**로 생성한다.
  - `text`가 변경되면, `textStyle`는 초기화된다.
- `LaunchedEffect(textStyle) { ... }` : `textStyle`이 변경될 때 마다 실행된다.
  - `textStyle`이 변경되면, Text의 크기를 다시 측정하고, `constraints`와 비교하여 동적으로 폰트 크기를 조절한다.
  - `if (textStyle.fontSize.value.toInt() > minFontSize && size.width >= constraints.maxWidth || size.height >= constraints.maxHeight)` : `textStyle`의 폰트 크기가 `minFontSize`보다 크고, Text의 크기가 `constraints`를 초과하는지 검사


### 위 코드의 단점

- 코드가 길다

현재 `textMeasurer`으로 `text`의 크기를 측정하여 처리하는 로직을 직접 구현했는데, 이 부분은 감사하게도 `Text` Composable 파라미터로 제공되는 `onTextLayout`을 사용해서 코드를 간결하게 정리할 수 있다.

간편하게 `Text(..., onTextLayout = {})`의 `onTextLayout`람다를 사용하면 된다.
이 람다는 `text`의 레이아웃이 새롭게 계산될 때 실행되는 콜백이기 때문에, 람다 인자로 전달되는 `TextLayoutResult`의 `didOverflowWidth`와 `didOverflowHeight`를 통해 `Text`의 크기가 주어진 크기를 초과하는지 알수 있다.



## 구현 2, 최종

```kotlin
private const val MIN_AUTO_SIZING_TEXT_SIZE = 12
private const val MAX_AUTO_SIZING_TEXT_SIZE = 30

/**
 * [minFontSize] ~ [defaultFontSize] 범위 내에서, [step]만큼 fontSize를 동적으로
 * 조절하면서, [TextOverflow]가 발생하지 않도록 하는 Text Composable
 *
 * @param modifier
 * @param text
 * @param style
 * @param overflow [TextOverflow.Ellipsis]는 사용 불가(동적 크기 조절이 이루어지지 않는다)
 * @param minFontSize
 * @param defaultFontSize
 * @param step
 *
 */
@Composable
fun AutoAdjustingFontSizeText(
    modifier: Modifier = Modifier,
    text: String,
    style: TextStyle,
    overflow: TextOverflow = TextOverflow.Clip,
    minFontSize: Int = MIN_AUTO_SIZING_TEXT_SIZE,
    defaultFontSize: Int = DEFAULT_AUTO_SIZING_TEXT_SIZE,
    step: Int = 1
) {
    BoxWithConstraints(modifier = modifier) {
        val textOverflow = remember(overflow) { if (overflow == TextOverflow.Ellipsis) TextOverflow.Clip else overflow }
        var textStyle by remember(text) { mutableStateOf(style.copy(fontSize = defaultFontSize.sp)) }

        Text(text = text, maxLines = 1, style = textStyle, overflow = textOverflow, modifier = Modifier, onTextLayout = {
            if (textStyle.fontSize.value.toInt() > minFontSize && it.didOverflowWidth || it.didOverflowHeight) {
                val newFontSize = (textStyle.fontSize.value.toInt() - step).coerceAtLeast(minFontSize)
                textStyle = textStyle.copy(fontSize = newFontSize.sp)
            }
        })
    }
}
```

## 사용 예시

```kotlin
Box {
    AutoAdjustingFontSizeText(
        text = "value",
        modifier = Modifier.fillMaxWidth(),
        style = TextStyle(color = Color.Black, fontWeight = FontWeight.SemiBold),
    )
}
```

다음과 같이 자동으로 조절됨을 확인할 수 있다.

![](https://github.com/pknujsp/pknujsp/assets/48265129/54a14f2f-723a-4b9b-b310-c95973b3fd62)