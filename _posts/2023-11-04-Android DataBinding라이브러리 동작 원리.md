---
layout: post
title: Android DataBinding라이브러리 동작 원리
subtitle: DataBinding, ViewBinding
published: true
categories: [ Android]
tags: [Android]
---


## 주요 클래스

| 클래스명                      | 설명                                                                                                                  |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| AndroidDataBinding            | 데이터바인딩의 핵심 클래스, 데이터 바인딩 작업을 총괄한다                                                             |
| LayoutFileParser              | XML을 파싱하여 ResourceBundle 목록을 만든다                                                                           |
| LayoutXmlProcessor            | 레이아웃 XML파일을 읽고 처리한다. 내부적으로 사용할 XML파일도 만든다                                                  |
| BaseDataBinder                | 바인딩 클래스 파일을 생성/삭제한다                                                                                    |
| ResourceBundle                | 레이아웃 파일 구문 분석 결과를 보관한다. 레이아웃 XML파일 내의 코드를 분석한다                                        |
| DataBindingGenBaseClassesTask | 데이터바인딩 클래스 생성을 위한 작업을 처리한다.(데이터바인딩 라이브러리가 아닌 Android Gradle Plugin에 포함되어있다) |
| LayoutInfoInput               | XML 파일에 대한 바인딩 정보를 담고 있다                                                                               |
| LayoutFileBundle              | XML 파일에서 데이터 바인딩 관련 정보를 파싱한 결과를 가진다                                                           |


## DataBinding 파일의 생성 과정

### 1. 프로젝트 빌드 시, 바인딩 처리할 파일 등에 대한 정보를 AndroidDataBinding에 전달한다.

- 전달 인자
  - appId : 애플리케이션 패키지 명
  - resInput : 리소스(layout, drawable 등)를 담고 있는 폴더
  - resOutput : 처리한 리소스를 담을 폴더
  - layoutInfoOutput : 레이아웃 파일에 대한 데이터 바인딩 정보를 가지고 있는 XML파일을 저장할 폴더

### 2. AndroidDataBinding의 `processResources()` 호출

LayoutXmlProcessor를 초기화한다.

### 3. LayoutXmlProcessor의 `processResources()` 호출

### 4. LayoutXmlProcessor의 `processAllInputFiles()` 호출

- 입력으로 받은 폴더의 모든 파일을 하나씩 확인한다.
- layout 폴더 내에 있는 파일에 대해서 `processLayoutFile()`을 호출한다.
- `processLayoutFile()`에서는 `processSingleFile()`을 호출한다.

### 5. processSingleFile()

LayoutFileParser의 `parseXml()`을 호출하여 XML을 파싱한다.

`parseOriginalXml()`를 호출하여 실제로 파싱을 한다.

- RootView 분석
- data태그 내의 import, variable, class 정보를 분석하고, LayoutFileBundle에 저장한다.
- 배치된 View에 대한 정보(id, tag, class)와 바인딩 정보(@{}과 같은 표현식)를 분석하여 LayoutFileBundle에 저장한다.

분석한 데이터 바인딩 정보는 추가적으로 XML파일을 만들어 저장해둔다.

원본 파일이 `activity_main.xml` 이라면 `activity_main-layout.xml`이름으로 생성한다.

#### 원본 레이아웃 파일

```xml
R.layout.activity_main

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="viewModel"
            type="com.example.main.MainViewModel" />

    </data>

    <TextView android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="@{viewModel.text}" />

</layout>

```

#### 바인딩 정보만을 담고있는 추가로 생성된 파일

```xml
activity_main-layout.xml

<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<Layout layout="activity_main" absoluteFilePath="/home/framgia/Projects/harpa-crista/harpacrista/android/app/src/main/res/layout/activity_main.xml"
    directory="layout"
    isMerge="false" modulePackage="com.harpacrista">
    <Variables declared="true" name="viewModel" type="com.example.main.MainViewModel">
        <location endLine="8" endOffset="51" startLine="6" startOffset="8" />
    </Variables>
    <Imports name="View" type="android.view.View">
        <location endLine="10" endOffset="42" startLine="10" startOffset="8" />
    </Imports>
    <Targets>
        <Target tag="layout/activity_main_0" view="TextView">
            <Expressions>
                <Expression attribute="android:text" text=" viewModel.text ">
                    <Location endLine="16" endOffset="41" startLine="16" startOffset="8" />
                    <TwoWay>false</TwoWay>
                    <ValueLocation endLine="16" endOffset="39" startLine="16" startOffset="24" />
                </Expression>
            </Expressions>
            <location endLine="16" endOffset="44" startLine="14" startOffset="4" />
        </Target>
    </Targets>
</Layout>
```

`android:text="@{ viewModel.text }"`은 아래와 같이 변환된다.

```xml
<Expression attribute="android:text" text=" viewModel.text ">
    <Location endLine="23" endOffset="45" startLine="23" startOffset="12" />
    <TwoWay>false</TwoWay>
    <ValueLocation endLine="23" endOffset="43" startLine="23" startOffset="28" />
</Expression>
```

- `<Expression attribute="android:text" text="viewModel.text">` : `android:text="@{ viewModel.text }"`
- `<location endLine="16" endOffset="44" startLine="14" startOffset="4" />` : `android:text="@{viewModel.text}"`가 위치한 라인과 오프셋
- `<TwoWay>false</TwoWay>` : 양방향 바인딩의 여부
- `<ValueLocation endLine="16" endOffset="39" startLine="16" startOffset="24" />` : `viewModel.text`가 위치한 라인과 오프셋


```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:tag="layout/activity_main_0" />
```

루트 뷰 타입을 분석하기도 한다

### 6. writeLayoutInfoFiles()

`writeXmlFile()`을 호출하여 layout xml파일을 생성

불필요한 파일은 제거

### 7. java, kotlin 파일 생성

BaseDataBinder `generateAll()`을 호출하여 생성

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="@{ viewModel.text }"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="@{ viewModel.isVisible ? View.VISIBLE : View.INVISIBLE }"
        />
</LinearLayout>

```

```kotlin
public class ActivityMainBinding extends android.databinding.ViewDataBinding  {

    private static final android.databinding.ViewDataBinding.IncludedLayouts sIncludes;
    private static final android.util.SparseIntArray sViewsWithIds;

    static {
        sIncludes = null;
        sViewsWithIds = null;
    }
    
    private final android.widget.LinearLayout mboundView0;
    private final android.widget.TextView mboundView1;
    private final android.widget.Button mboundView2;
    ...
}
```

View 변수명은 XML에 id가 지정되어있으면 `id`를 사용하고, 지정되어있지 않다면 `mBoundView + index`로 만들어진다.

## View의 Binding 클래스 파일이 자동으로 생성되는 과정

1. Gradle Sync
2. buildFeatures의 viewBinding, dataBinding 둘 중 최소 하나가 활성화되어있는지 확인
3. 활성화되어 있다면 아래의 메서드를 호출하고, Gradle Task에 DataBindingGenBaseClassesTask를 추가한다.
4. XML파일이 추가/수정/삭제 될때마다 DataBindingGenBaseClassesTask가 동작한다.
5. BaseDataBinder를 사용하여 바인딩 파일을 처리한다.

- TaskManager : Android Gradle Plugin의 클래스, Gradle 작업을 관리한다.
- DataBindingGenBaseClassesTask : 데이터바인딩 처리를 위한 작업

두 클래스는 Jetpack DataBinding 라이브러리에 종속되어있지 않다.

```
in TaskManager.kt

protected fun createDataBindingTasksIfNecessary(creationConfig: ComponentCreationConfig) {
        val dataBindingEnabled = creationConfig.buildFeatures.dataBinding
        val viewBindingEnabled = creationConfig.buildFeatures.viewBinding
        if (!dataBindingEnabled && !viewBindingEnabled) {
            return
        }
        taskFactory.register(
                DataBindingMergeDependencyArtifactsTask.CreationAction(creationConfig))
        DataBindingBuilder.setDebugLogEnabled(logger.isDebugEnabled)
        taskFactory.register(DataBindingGenBaseClassesTask.CreationAction(creationConfig))
        ...

in DataBindingGenBaseClassesTask.kt

@TaskAction
    fun writeBaseClasses(inputChanges: InputChanges) {
        // TODO extend NewIncrementalTask when moved to new API so that we can remove the manual call to recordTaskAction

        recordTaskAction(analyticsService.get()) {
            // TODO figure out why worker execution makes the task flake.
            // Some files cannot be accessed even though they show up when directory listing is
            // invoked.
            // b/69652332
            val args = buildInputArgs(inputChanges)
            CodeGenerator(
                args,
                sourceOutFolder.get().asFile,
                Logger.getLogger(DataBindingGenBaseClassesTask::class.java),
                encodeErrors,
                getRPackageProvider()).run()
        }
    }
}
```