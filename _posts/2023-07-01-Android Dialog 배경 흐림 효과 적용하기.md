---
layout: post
title: Android Dialog Window 흐림 효과 적용하기
subtitle: How to blur the screen behind the given window in android
published: true
categories: Android
tags: [Android]
---

## Window 흐림 효과 적용 전에, 흐림 효과 적용 가능 대상 종류를 알아봅시다.
---
> 두 가지 대상에 대하여 흐림 효과 적용이 가능합니다.

1. Background blur
   - 배경을 흐리게 합니다.
2. Behind blur(Window blur)
   - Window를 흐리게 합니다.
   - Window(PhoneWindow)
     - 화면에 출력되고 있는 View가 보여지는 영역입니다.
     - Window는 DecorView를 가지고 있고, DecorView에 실제로 View가 그려지게 됩니다.
     - Activity는 Window를 관리, Window는 DecorView를 관리, DecorView는 그려질 View(Statusbar, content,Navigationbar)를 관리합니다.
     - ![](https://miro.medium.com/v2/resize:fit:292/1*pPA5fNL6-zcx-_X__nGuxQ.png)


**이 글에서는 Window 흐림 효과에 대해서 집중적으로 다룹니다. 한계점이 있기 때문입니다.**
  


|                                                      Background blur                                                       |                                                      Behind blur                                                       |                                                               모두 적용                                                               |
| :------------------------------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------: |
| <img src="https://source.android.com/static/docs/core/display/images/background-blur-only.png" width="auto" height="auto"> | <img src="https://source.android.com/static/docs/core/display/images/blur-behind-only.png" width="auto" height="auto"> | <img src="https://source.android.com/static/docs/core/display/images/blur-behind-and-background-blur.png" width="auto" height="auto"> |

## 흐림 효과 적용 방법
---

```xml
<style name="BlurDialog" parent="Theme.AppCompat.Dialog">

    // behind blur
    <item name="windowBlurBehindEnabled">true</item>
    // dp, px 등 단위를 사용하고, 흐림 강도를 설정합니다. 값이 클수록 흐림이 강해집니다.
    // 강도 조절은 Android 12 이상에서만 사용할 수 있습니다.
    <item name="windowBlurBehindRadius">10dp</item>

    // background blur
    // dp, px 등 단위를 사용하고, 흐림 강도를 설정합니다. 값이 클수록 흐림이 강해집니다.
    <item name="windowBackgroundBlurRadius">10dp</item>
    // 흐림 효과를 화면에서 보려면 true로 설정해야 합니다.
    <item name="windowIsTranslucent">true</item>

  </style>
```

**OR**

```kotlin
    window.apply{
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            // behind blur
            // <item name="windowBlurBehindEnabled">true</item>
            // API30에서 Undeprecated 되었는데, 공식 문서에는 Deprecated in API15로 표기되어 있습니다.
            addFlags(WindowManager.LayoutParams.FLAG_BLUR_BEHIND)

            // 아래와 같이 10만 입력하면 10px로 인식합니다.
            //item name="windowBlurBehindRadius">10dp</item>
            attributes.blurBehindRadius = 10

            // background blur
            // <item name="windowBackgroundBlurRadius">10dp</item>
            window.setBackgroundBlurRadius(10)

            // <item name="windowIsTranslucent">true</item> XML파일에 적용해야 합니다.
        }
    }
```

## Window Blur 한계점
---
> Window Blur의 강도 조절은 Android 12 이상에서만 사용할 수 있습니다.

* Android 11이하에서는 강도 조절 적용이 현재 API(Compat API도 없음)로는 불가능합니다.
* 기기에서 이 기능 사용이 내부적으로 비활성화 되어있으면, Window blur 적용 자체가 불가능합니다.

**이러한 한계점을 극복하기 위한 방법을 연구중입니다**

먼저 Window Blur가 적용되는 로직을 알아봅시다.

## Dialog에 Window blur가 적용 되는 로직

### 1. Dialog 객체의 create()호출
---

1. Dialog생성시 전달받은 context를 기반으로 PhoneWindow 인스턴스를 생성합니다.
2. 인스턴스를 만들면서, DecorView가 생성됩니다.

### 2. Dialog의 show()호출
---

1. 액티비티의 Window를 관리하는 WindowManager의 addView()를 호출합니다.
2. 애플리케이션의 Window를 관리하는 WindowManagerGlobal의 addView() 호출하여, Window를 본격적으로 추가하고 그림을 그립니다.

### 3. Dimmer class 로 블러 효과 적용
---

1. 액티비티 스레드에서 Window를 그려라는 명령을 받게 되면, WindowContainer객체에서 WindowLayoutParams의 값을 바탕으로 Window를 그립니다.
2. Dimmer 클래스를 사용해서 Blurring 작업을 시작합니다.
3. 안드로이드 native에서 OpenGL을 사용해서 Blurring 작업을 수행합니다.
4. 완료되면 WindowContainer의 Surface에 그립니다.


안드로이드 12미만에서는 말했듯이 Window Blur를 적용할 수 없습니다. 그렇다면 Window Blur를 적용할 수 있는 방법은 없을까요?

ViewRootImpl의 Surface를 받아서, Surface에 Blur를 적용하면 될 것 같습니다.

그러나 ViewRootImpl과 Surface를 직접받아서 로직을 제작하는 것은 불가능 한 것 같습니다.


### 구현 방법
---

1. Blur처리를 할 View의 ViewTreeObserver에 `OnPreDrawListener`를 등록합니다.
   - 해당 View가 그려지기 전에 위 Listener가 호출됩니다.
   - View가 다시 그려질 때 마다 호출됩니다.
2. Blur처리된 Bitmap을 그려줄 View를 Dialog가 그려지는 Window에 추가합니다.
   - GLSurfaceView를 사용합니다.
3. `OnPreDrawListener`가 호출될 때 마다, 해당 View를 Bitmap으로 변환합니다.
   - `View.drawToBitmap()`을 사용합니다.
4. Bitmap을 Blur처리 합니다.
   - RenderScript, Intrinsic Blur를 사용합니다.
5. Blur처리된 Bitmap을 GLSurfaceView에서 그려주도록 합니다.
   - GLSurfaceView의 Renderer를 구현합니다.
   - `onDrawFrame()`에서 Blur처리된 Bitmap을 그려줍니다.
6. 2~5번 과정이 계속 반복됩니다.

RenderScript가 Android 12부터 Deprecated되었고, 대체하기 위해 Google에서 Toolkit을 제공합니다.

해당 Toolkit을 적용해보았으나, RenderScript보다 Blur를 처리하는데에 약 1.5배의 시간이 더 소요 되는 것을 확인하였습니다.

다른 StackBlur라는 알고리즘을 적용하는 등, 여러 방법을 시도해보았지만, RenderScript를 대체할만한 Blur처리 방법을 찾지 못하였습니다.

Continue...