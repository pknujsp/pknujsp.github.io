---
layout: post
title: Android DataBinding라이브러리 동작 원리
subtitle: DataBinding, ViewBinding
published: true
categories: [ Android]
tags: [Android]
---


## 주요 클래스

| 클래스명                      | 설명                                                                                                                                           |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| AndroidDataBinding            | 데이터바인딩의 핵심 클래스, 데이터 바인딩 작업을 총괄한다                                                                                      |
| LayoutFileParser              | XML 파일 목록을 읽고, 파싱하여 뷰바인딩과 데이터바인딩을 구분하여 처리하고 ResourceBundle 목록을 만든다                                        |
| LayoutXmlProcessor            | 개발자가 작성한 레이아웃 XML에서 바인딩 속성과 요소를 지우고, 어노테이션 프로세서가 처리하도록 어노테이션이 달린 클래스 파일에 정보를 작성한다 |
| BaseDataBinder                | 바인딩 클래스 파일을 생성/삭제한다                                                                                                             |
| ResourceBundle                | 레이아웃 파일 구문 분석 결과를 보관한다. 레이아웃 XML파일 내의 코드를 분석한다                                                                 |
| DataBindingGenBaseClassesTask | 데이터바인딩 클래스 생성을 위한 작업을 처리한다.(데이터바인딩 라이브러리가 아닌 Android Gradle Plugin에 포함되어있다)                          |
| LayoutInfoInput               | XML 파일에 대한 바인딩 정보를 담고 있다                                                                                                        |


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

ProcessFileCallback에서 응답을 받아 처리한다

### 4. LayoutXmlProcessor의 processAllInputFiles() 호출

XML파일이며 <layout>태그가 있는 파일에 대해서 processLayoutFile()를 호출

### 5. processSingleFile()

LayoutFileParser의 parseXml()을 사용해서 파싱

### 6. parseOriginalXml()

뷰바인딩, 데이터바인딩 인지에 따라 분리하여 처리

ResourceBundle.LayoutFileBundle를 만들어 파싱 결과를 저장

루트 뷰 타입을 분석하기도 한다

### 7. writeLayoutInfoFiles()

writeXmlFile()로 layout xml파일을 생성

불필요한 파일은 제거

### 8. java, kotlin 파일 생성

BaseDataBinder generateAll()을 사용하여 생성



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