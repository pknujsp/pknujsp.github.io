---
layout: post
title: RecyclerView의 동작 로직에 대해 자세히 알아보기
subtitle: RecyclerView의 원리에 대해서 자세히 알아보자
published: true
categories: Android
tags: [Android]
---

>RecyclerView의 기본적인 사용법을 알고 있다는 가정하에, RecyclerView의 동작 원리에 대해서 알아보겠습니다.

## RecyclerView의 핵심 클래스
---

![recyclerview](https://github.com/pknujsp/android-blur/assets/48265129/07867150-5a29-403e-ac7a-d6cf39bc0cae)

* RecyclerView
  * 가장 중요한 클래스
  * ViewGroup을 상속받는다.
  * 총 감독 역할을 한다.
* LayoutManager
  * 레이아웃과 관련된 작업을 처리
* Adapter
  * 데이터를 관리하고 ViewHolder를 생성하고 Bind를 처리
* CachedViews
  * RecyclerView를 스크롤 할 때 가장 먼저 탐색하는 곳
  * `onBindViewHolder()` , `onCreateViewHolder()`를 사용할 필요없이 바로 보여줄 수 있는 View를 저장
  * Array로 관리되고, 크기를 개발자가 지정할 수 있다.
* RecycledViewPool
  * Cache에 View가 없으면 그 다음에 탐색하는 곳
  * ViewHolder를 저장하고 있다.
* Scrap
  * ItemView 레이아웃이 화면에 그려지는 동안 저장되는 목록


### LayoutManager
---

* 보여줄 ItemView의 위치, 배치, 레이아웃과 관련된 작업을 처리합니다.
* 이미 구현된 클래스로 `LinearLayoutManager`, `GridLayoutManager`, `StaggeredGridLayoutManager`가 있습니다.
* List를 스크롤 할 때, RecyclerView에서 스크롤을 감지하고, LayoutManager에게 스크롤을 처리하라고 요청합니다.
* 처리 요청을 받으면, 새로운 ItemView의 표시 위치를 계산하고, RecyclerView에게 ItemView를 달라고 요청합니다.
* RecyclerView로 부터 ItemView를 받으면, 해당 ItemView를 표시합니다.
  * `ChildHelper` 클래스를 사용하여 RecyclerView자체에 `addView()` , `removeView()`등의 작업을 처리합니다.

### Adapter
---

* ViewHolder, ItemView를 생성/관리, 이와 관련된 작업을 처리합니다.
* ViewHolder에 ItemView를 붙입니다.
* RecyclerView에 보여줄 데이터를 관리합니다.
* 사용자가 클릭하는 등의 이벤트를 처리합니다.

### CachedViews
---

* 재활용을 시도할때, 가장 먼저 탐색하는 곳입니다.
* ItemView가 저장되고, 이 곳의 View를 사용하면 `onBindViewHolder()` , `onCreateViewHolder()`를 사용할 필요없이 바로 보여줄 수 있습니다.
* Array로 관리되고, 크기를 개발자가 지정할 수 있습니다.

### RecycledViewPool
---

* CachedViews에 View가 없으면 그 다음에 탐색하는 곳입니다.
* ViewHolder가 저장되고, 이후 `onBindViewHolder()`를 사용하여 View를 생성합니다.

### Scrap
---

* ItemView 레이아웃이 화면에 그려지는 동안 저장되는 목록입니다.

## Dirty View
---

> RecycledViewPool에 존재하는 View를 Dirty View라고 합니다.

* View정보가 초기화되기 때문에, `onBindViewHolder()`를 사용하여 View를 생성해야 합니다.

## 기본 흐름
---

![recyclerview-logics](https://github.com/pknujsp/android-blur/assets/48265129/79d9e846-a226-4804-a226-bc199703202f)