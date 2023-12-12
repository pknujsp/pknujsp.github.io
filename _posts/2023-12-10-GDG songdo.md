---
layout: post
title: GOG songdo
subtitle: ss
published: true
categories: [Conference]
tags: [Conference]
---

## 1. KMP

- KMP
- Compose Multiplatform
- Fleet


### KMP

- 각 플랫폼(IOS, ANDROID) 별로 UI구현을 할 줄 알아야 함
- shared(common)모듈이 바이너리로 변환되는 경우 불편, 디버깅 불편, 디버깅 시 타입 값이 메모리 주소? 로 표현됨
  - 라이브러리를 사용하여 개선가능
- 코루틴 취소가 ios에서 불가
  - 라이브러리, 직접 기능을 구현해야
- 윈도우, 웹은 코틀린 자바 스크립트/네이티브로 구축가능
- 안드로이드 친화적


? 앱 실행 성능으로 질문 플러터/리액트 네이티브는 네이티브와 성능 차이가 좀 있는 편인데 네이티브와 KMP 둘의 앱 실행 성능 차이

## 2. KMP로 번역기 제작

- 공통 기능은 common모듈에 구현
- KMP모듈과 각 플랫폼 간 브릿지?로 연결되어 동작
- SqlDelight로 DB 구현 가능
- 안드로이드 의존성이 있는 모듈을 타 플랫폼에서 사용하는 경우 따로 구현해야
- 뷰모델 대신 Decompose, moko-mvvm 과 같은 라이브러리 사용 가능
- 각 플랫폼의 지식이 있어야 함

CMP는 플러터, 리액트 네이티브와 유사
KMP는 데이터 레이어만 하나로 개발하고 UI는 각 플랫폼 별로 개발


## 3. KMP DI, 유용한 라이브러리 목록

- Koin
  - 코틀린 DI 프레임워크
  - 컴파일 검증 없음
  - 런타임에 작동하므로 빌드 속도가 dagger사용 시 보다 나음
- Ktorfit
  - Retrofit과 유사
- 권한
  - moko-permissions 사용하여 편하게 가능
- Decompose, Orbit
  - 아키텍처 라이브러리
- Crash reporting
  - SENTRY, CrashKiOS, moko-crash-reporting
- 보안
  - Cryptography, libsodium
- File
  - okio
- Serializer
  - kotlinx-serialization
- Date, time
  - kotlinx.datetime
- Firebase는 공식적인 SDK없음 Firebase Kotlin SDK사용

## 4. Compose 팁

### 1. 애니메이션 개발

- 카카오뱅크 앱 메인 우측 상단 프로필 이미지 애니메이션
  - UI구현
    - 커스텀 뷰, 컴포즈는 Modifier 세부 조작
  - 업데이트, 뷰 클릭
    - 어느 뷰를 기준으로 바인딩할지
    - 클릭 시 어느 프로필이 메인인가
- 이미지 화질 저하
  - 계좌 정보 화면에서 금액 표시 하단 이미지 화질이 저하되는 문제
  - Glide를 사용한다면 뷰 크기와 circleCrop API건드려서 개선
- 이미지 테스트
  - UI Test시 mocking해서 테스트
- Placeholder를 사용해서 기본 이미지라도 보여주자
- 테스트 시 기기 조건을 가혹하게 설정한 상태에서도 테스트해보기
- 터치 영역 확보: 패딩, 터치위임, 터치 영역 확장을 스펙아웃


앱 알림기능 구현 시 알림 유형 별로 WorkManager 작업을 만드는지 : 각각 만들기
안드 신입 뽑을때 상당히 서류 통과 벽이 높은데 개인적으로 서류 통과 기준이 궁금 : ??
다중 언어 지원 시 언어별로 특정 값이 들어가는 위치가 달라지는데 이때 각 언어별로 분기처리를 하는게 맞는지
실시간 위치, 백그라운드에서 현재 위치 가져올때 안드 버전 올라가면서 계속 제한되어 왔는데 휴대폰 화면이 꺼져있을때도 깔끔하게 현재 위치를 가져오는 방법
실험 API기능 사용 여부


## 5. 컴포즈 애니메이션

다양한 컴포넌트 제공



레이아웃이 통째로 애니메이션 -> 등장 -> 애니메이트비지빌리티


날씨 효과 애니메이션 구현, SurfaceView or Canvas로 구현 질문, SurfaceView를 효과 구현에 주로 사용하는지 : 잘 쓰이지 않고 lottie같은 벡터애니메이션
chatgpt처럼 텍스트가 스트림 형식으로 나올때 구현, 텍스트를 바꿔가는지 아니면 문자단위로 받으면서 업데이트해가는지 : 문자 단위로 표시?
비밀번호 같은 경우 lock을 걸어서 접근을 못하게?


## 6. Compose UI Test

- UI test?
- XML ui test
- Compose ui test

어떤게 좋은 테스트인가 : 훼손을 잘 감지, 세부 구현에 독립적, 실패를 잘 설명, 이해쉽고 빠른 실행

resource string
날씨 시간별 예보 enum으로 날씨 상태(맑음, 흐림 등) 리소스 id일때 뷰모델에서 처리하여 보내줘도 괜찮은지?
lazyrow로 수 십개의 열들을 보여줄때 텍스트를 ui에서 context로 가져오지 않고 백그라운드 스레드에서 문자열로 다 바꿔서 전달하는데 안티패턴?
언어 별로 문자열 처리는 annotatedString사용? 이것도 뷰모델에서 처리해서 ui로 넘기는데 안티?


## 7. 안드로이드 모듈화

