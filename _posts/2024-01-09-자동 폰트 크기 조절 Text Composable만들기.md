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


## 구현 코드

```kotlin
private const val MIN_AUTO_SIZING_TEXT_SIZE = 12
private const val MAX_AUTO_SIZING_TEXT_SIZE = 30

/**
 * [minFontSize] ~ [maxFontSize] 범위 내에서, [step]만큼 fontSize를 동적으로
 * 조절하면서, [TextOverflow]가 발생하지 않도록 하는 Text Composable
 *
 * @param modifier
 * @param text
 * @param style
 * @param overflow [TextOverflow.Ellipsis]는 사용 불가(동적 크기 조절이 이루어지지 않는다)
 * @param minFontSize
 * @param maxFontSize
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
    maxFontSize: Int = MAX_AUTO_SIZING_TEXT_SIZE,
    step: Int = 1
) {
    BoxWithConstraints(modifier = modifier) {
        val textOverflow = remember(overflow) { if (overflow == TextOverflow.Ellipsis) TextOverflow.Clip else overflow }
        var textStyle by remember(text) { mutableStateOf(style.copy(fontSize = maxFontSize.sp)) }

        Text(text = text, maxLines = 1, style = textStyle, overflow = textOverflow, modifier = Modifier, onTextLayout = {
            if (it.didOverflowWidth || it.didOverflowHeight && textStyle.fontSize.value.toInt() > minFontSize) {
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

![](https://github.com/pknujsp/pknujsp/assets/48265129/54a14f2f-723a-4b9b-b310-c95973b3fd62)