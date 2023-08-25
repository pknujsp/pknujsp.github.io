---
layout: post
title: 그래프 그리는 View만들기
subtitle: Canvas를 이용하여 View에서 그래프를 그리는 방법
published: true
categories: Android
tags: [Android]
---

## 그래프 그리는 View만들기
---

> 아래의 사진 처럼 그래프를 그려주는 View를 만드는 방법을 이번 글에서 다루어 보겠습니다.

<img src="https://github.com/pknujsp/BestWeather/assets/48265129/41af7a3c-b456-4ce1-8e80-f43213a04db1" width="60%">

- 사용하는 주요 클래스
  - View를 상속하여 CustomView를 만듭니다.
  - Canvas
    - 화면에 실제로 보여줄 View를 그리는 클래스입니다.
    - 그림을 그리는 종이라고 생각하면 이해하기 쉬울 것 같습니다.
    - 참고 : [Android의 Canvas에 그려보자 : 선, 도형 그리고 그림까지!?](https://www.charlezz.com/?p=1433)
  - Paint
    - Canvas에 그릴 객체(선, 도형, 글자 등)의 속성(색상, 스타일 등)을 정의하는 클래스입니다.
  - Path
    - 그래프를 그릴 때 사용되는 핵심 클래스입니다.
    - 그릴 선의 정보를 담고 있습니다.


## 그래프를 그릴 View 클래스 정의
---

> View를 상속하여 CustomView를 만듭니다.

예제에서는 **GraphView**라는 이름으로 클래스를 정의합니다.

```kotlin
class GraphView(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : View(context, attrs, defStyleAttr) {

    constructor(context: Context, attrs: AttributeSet?) : this(context, attrs, 0)

    constructor(context: Context) : this(context, null, 0)
}
```

### 그래프에 그릴 객체의 속성을 정의하기
---

```kotlin
    // 그릴 선, 도형의 속성 목록
    private val _mPaints: MutableList<Paints> = mutableListOf()
    private val mPaints: List<Paints> get() = _mPaints

    // 그래프 상단/하단의 여백 크기, 12dp
    private val mVerticalSpace = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 12f, Resources.getSystem().displayMetrics)

    // 그래프 x축 값의 간격, 48dp
    private val mXAxisValueIntervalSpace: Float =
        TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 48f, Resources.getSystem().displayMetrics)

    // 그릴 선, 도형의 Paint객체를 담을 클래스
    // 여러 개의 선을 그릴 때 각각의 선의 속성을 다르게 설정 하기 위해 사용합니다.
    private data class Paints(
        val linePaint: Paint,
        val pointPaint: Paint,
    )
```

### 표시할 데이터 관련 로직 정의하기
---

> 그래프로 표시할 값을 처리하기 위한 로직을 만들어야 합니다.

```kotlin
    // 각각의 선으로 그리기 위한 값 목록
    private val _mDataList: MutableList<List<Float>> = mutableListOf()
    private val mDataList: List<List<Float>> = _mDataList

    fun setDataList(dataList: List<List<Float>>) {
        _mDataList.clear()
        _mDataList.addAll(dataList)
    }
```

### 그래프를 그리는 로직 정의하기
---

그래프를 그리기 위한 사전 정보를 정의하는 코드입니다.


그래프의 그리기 정보를 담고 있는 `DrawInfo` 클래스를 정의합니다.

`Fragment`, `Activity`에서 `GraphView`에 그릴 객체의 정보를 설정하기 위해서 `setDrawInfo()`를 사용합니다.

```
data class DrawInfo(
    @Dimension(Dimension.DP) val lineThickness: Int = 1,
    @ColorInt val lineColor: Int = Color.WHITE,
    @ColorRes val lineColorResId: Int = NONE_ID,
    @Dimension(Dimension.DP) val pointRadius: Int = 2,
    @ColorInt val pointColor: Int = Color.WHITE,
    @ColorRes val pointColorResId: Int = NONE_ID,
) {

    companion object {
        const val NONE_ID = -1
    }
}
```

```kotlin
    fun setDrawInfo(drawInfoList: List<DrawInfo>) {
        _mPaints.clear()

        // 그릴 객체의 크기를 dp로 설정하기 위해서 기기 화면의 크기 정보를 가져옵니다.
        val displayMetrics = Resources.getSystem().displayMetrics

        for (drawInfo in drawInfoList) {
            val line = Paints(
                // 선의 속성
                linePaint = Paint().apply {
                    isAntiAlias = true
                    style = Paint.Style.STROKE
                    strokeWidth = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, drawInfo.lineThickness.toFloat(), displayMetrics)
                    color = if (drawInfo.lineColorResId == NONE_ID) drawInfo.lineColor else context.getColor(drawInfo.lineColorResId)
                },
                // 선 위에 그릴 도형(점)의 속성
                pointPaint = Paint().apply {
                    isAntiAlias = true
                    style = Paint.Style.FILL
                    strokeWidth = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, drawInfo.pointRadius.toFloat(), displayMetrics)
                    color = if (drawInfo.pointColorResId == NONE_ID) drawInfo.pointColor else context.getColor(drawInfo.pointColorResId)
                },
            )
            _mPaints.add(line)
        }
    }
```

**GraphView**의 너비를 계산하여 지정하는 코드입니다.

* 데이터가 있으면 데이터의 개수만큼 x축의 간격을 계산하여 너비를 지정합니다.
  * 데이터의 개수만큼 너비가 설정됩니다.
* 데이터가 없다면 View정의 시 지정한 너비를 그대로 사용합니다.


```kotlin
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        val width: Int =
            if (mDataList.isEmpty()) widthMeasureSpec else mDataList.first().size * mXAxisValueIntervalSpace.toInt()
        setMeasuredDimension(width,
            heightMeasureSpec)
    }

```

그래프를 실제로 그리는 코드입니다.

View의 레이아웃이 계산된 후 `invalidate()`가 호출될 때 `onDraw()`가 호출됩니다.



```kotlin

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        drawLinePaths(canvas)
    }

    private fun drawLinePaths(canvas: Canvas) {
        // 모든 데이터에서 최소값, 최대값을 구합니다.
        val (minValue, maxValue) = calculateMinMaxValue()

        // 그래프 y축이 그려질 최상단 y좌표값
        val graphYAxisTop = mVerticalSpace

        // 그래프 y축의 길이, View의 높이 - 위아래 여백
        val graphYAxisHeight = height - 2 * mVerticalSpace

        // 데이터 값의 범위
        // 최소값: -10, 최대값: 200 -> 210
        val valueLength = maxValue.toFloat() - minValue.toFloat()

        // 그릴 선의 정보
        val linePathList = mutableListOf<Path>()

        // 첫 값이 그려질 x좌표
        val startPointX = mXAxisValueIntervalSpace / 2f

        // 각 선이 그려질 점의 좌표 목록
        val linePoints = mutableListOf<List<PointF>>()

        mDataList.forEach { dataList ->
            val path = Path()
            linePathList.add(path)

            // 첫 값의 좌표이자 이전 값의 좌표
            var lastPoint =
                PointF(startPointX, graphYAxisTop + calculateYPosition(dataList.first().toFloat(), minValue.toFloat
                    (), valueLength) *
                        graphYAxisHeight)

            // 선의 시작 좌표를 첫 값의 좌표로 지정합니다.
            path.moveTo(lastPoint.x, lastPoint.y)

            val points = mutableListOf<PointF>()
            linePoints.add(points)

            dataList.forEachIndexed { i, value ->
                if (i > 0) {
                    val newPoint = PointF(lastPoint.x + xAxisValueInterval,
                        graphYAxisTop + calculateYPosition(value.toFloat(), minValue.toFloat
                            (), valueLength) * graphYAxisHeight)
                    val point1 = PointF(lastPoint.x + xAxisValueInterval / 2, lastPoint.y)
                    val point2 = PointF(point1.x, newPoint.y)

                    // 곡선을 그리기 위해서 좌표값을 설정합니다.
                    path.cubicTo(point1.x, point1.y, point2.x, point2.y, newPoint.x, newPoint.y)
                    lastPoint = newPoint
                }
                points.add(lastPoint)
            }
        }

        // 선과 점을 그립니다.
        mPaints.zip(linePathList).forEachIndexed { i, pair ->
            canvas.drawPath(pair.second, mPaints[i].linePaint)
            drawPoints(linePoints[i], canvas, pair.first.pointPaint)
        }
    }

    // 선 위의 점을 그립니다.
    private fun drawPoints(points: List<PointF>, canvas: Canvas, paint: Paint) {
        points.forEach { pointF ->
            canvas.drawCircle(pointF.x, pointF.y, paint.strokeWidth, paint)
        }
    }

    // 데이터 값에 따라서 그래프에서 그려질 선의 y좌표 위치를 계산합니다.
    // 최소값: 5, 최대값: 10, 값: 7 -> 0.5
    // y축에서 중간에 위치하게 됩니다.
    private fun calculateYPosition(value: Float, minValue: Float, valueLength: Float) =
        (1f - ((value - minValue) / valueLength))

    // 모든 데이터에서 최소값, 최대값을 구합니다.
    private fun calculateMinMaxValue(): Pair<Number, Number> {
        val min: Number = mDataList.minBy { numbers -> numbers.minOf { it } }.minOf { it }
        val max: Number = mDataList.maxBy { numbers -> numbers.maxOf { it } }.maxOf { it }
        return min to max
    }
```

#### cubicTo()


`fun cubicTo(x1: Float, y1: Float, x2: Float, y2: Float, x3: Float, y3: Float): Unit`

- bezier 곡선을 그립니다.
  - x1 : 첫 번째 제어점의 x좌표
  - y1 : 첫 번째 제어점의 y좌표
  - x2 : 두 번째 제어점의 x좌표
  - y2 : 두 번째 제어점의 y좌표
  - x3 : 끝 점의 x좌표
  - y3 : 끝 점의 y좌표

<img src="https://github.com/pknu-wap/2023_1_WAP_APP_TEAM_MEDI/assets/48265129/c970224e-f948-4fd8-ae74-93f27964bcdb" width="60%">

그림에서 **2, 3, 4** 점이 **(x1, y1), (x2, y2), (x3, y3)** 에 해당합니다.


#### calculateYPosition()

`1f - (value - minValue) / (maxValue - minValue)`의 계산식을 통해서 데이터 값에 따라서 선의 y좌표 위치를 계산합니다.

<img src="https://github.com/pknujsp/BestWeather/assets/48265129/e936c43f-4f82-49ea-a34b-aad282c92b74" width="60%">

### 사용
---

#### ScrollView로 감싸서 사용하도록 합니다.

`GraphView`의 높이는 200dp로 고정하였습니다.

```xml
    <HorizontalScrollView
        android:layout_width="match_parent"
        android:nestedScrollingEnabled="true"
        android:layout_height="wrap_content">

        <FrameLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">

            <toss.next.naversovc.ui.common.view.GraphView
                android:id="@+id/graphView"
                android:layout_width="wrap_content"
                android:layout_height="200dp"
                android:background="#B8B8B8" />

        </FrameLayout>

    </HorizontalScrollView>
```

```kotlin
    binding.run {
        // 그릴 객체 정보
        graphView.setDrawInfo(listOf(
            DrawInfo(
                lineThickness = 4,
                lineColor = Color.WHITE,
                pointRadius = 8,
                pointColor = Color.WHITE,
            ),
            DrawInfo(
                lineThickness = 4,
                lineColor = Color.YELLOW,
                pointRadius = 8,
                pointColor = Color.YELLOW,
            ),
            DrawInfo(
                lineThickness = 4,
                lineColor = Color.RED,
                pointRadius = 8,
                pointColor = Color.RED,
            ),
        ))

        // 데이터 값
        graphView.setDataList(
            listOf(
                (-50..50 step 10).toList(),
                (50 downTo -50 step 10).toList(),
                listOf(10, 20, 30, 20, 10, 20, 30, 20, 10, 20, 30)
            )
        )
    }
```


<img width="60%" alt="image" src="https://github.com/pknujsp/BestWeather/assets/48265129/114f47d8-7316-4eeb-bb3f-d4f0bc14f450">


## 활용

> 시간 별 날씨예보의 기온 그래프와 같은 화면을 그리는 경우 등에 사용할 수 있습니다.

그 외에도 수학적인 그래프나, 값들 간에 비교를 위한 경우에도 사용할 수 있습니다.

<img src="https://github.com/pknujsp/BestWeather/assets/48265129/24e6f7d1-57ba-408b-b203-0831574f8eac" width="60%">
