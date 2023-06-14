---
layout: post
title: 12. Media Access Control
subtitle: Data communication - 12. Media Access Control
published: true
categories: Data communication
tags: [Data communication]
---

# MAC
---

* Data link layer
  * Data link control
  * Multiple-access resolution


## Multiple Access Protocols
---

* Multiple Access Protocols
  * Channelization(Partitioning)
    * TDMA
    * FDMA
    * CDMA
  * Random access
    * ALOHA
    * CSMA
    * CSMA/CD
    * CSMA/CA
  * Controlled-access
    * Reservation
    * Polling
    * Token passing

## Random-access
---
>다른 스테이션보다 앞서는 스테이션이 없고, 모든 스테이션이 같다.

### ALOHA
---
>라디오에 적합하게 설계되었다.

* ALOHA
  * Pure ALOHA
  * Slotted ALOHA

### Pure ALOHA
---

<img width="762" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/38784bba-9f01-4867-91f2-d3feb6ce0e21">

프레임이 충돌하는 시점이 발생한다.

### Slotted ALOHA
---

<img width="871" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/2283977a-8cea-463f-af2b-44b449e28911">

### CSMA(Carrier Sense Multiple Access)
---

<img width="660" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/b6cb1801-8940-480e-9662-73a1859c887e">

지연이 발생한다.

통신하는 선의 길이가 제한적이다.

vulnerable time이 발생한다.

### Persistence Methods
---

* 1-persistent method
  * 지속적으로 감시
  * 이더넷에서 사용
* non-persistent method
  * 랜덤하게 감시
  * 감지하고 대기
  * 가장 뛰어난 성능
* p-persistent method
  * 확률적으로 감시

<img width="839" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/3c33965d-37c2-4bf7-93a7-7b14e76b7b87">

### CSMA/CD(Carrier Sense Multiple Access with Collision Detection)
---

최소/최대 길이가 제한적이다.

충돌 시 고전압을 발생시켜서 알린다.

<img width="929" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/7343f573-38e0-4aac-809b-29f2ebb1a612">

* 에너지 레벨 변화
  * 1-persistent
    * 50%
  * non-persistent
    * up to 90%

### CSMA/CA(Carrier Sense Multiple Access with Collision Avoidance)
---

무선에서 사용한다.

* IFS
  * 스테이션 또는 프레임의 우선순위를 결정
* RTS(Request to Send)
  * 수신 노드에게 데이터 전송 준비 완료를 알리는 신호
* CTS(Clear to Send)
  * 수신 노드가 RTS를 받고 데이터 전송을 수락하는 신호
  * 송신 노드에게 전달
* NAV(Network Allocation Vector)
  * 노드가 다음 프레임을 전송하기 까지 기다려야 하는 시간
  * RTS CTS에 포함되어 있다
* DIFS(Distributed Interframe Space)
  * 노드가 채널이 사용 가능한지 확인하기 위해 기다려야 하는 시간
* SIFS(Short Interframe Space)
  * 두 프레임 사이의 최소 대기 시간
  * 특정 프레임을 수신한 후 다음 프레임을 전송하기 전에 기다려야 하는 시간

위와 같은 개념들을 사용하여, 충돌을 방지한다.


<img width="553" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/4fe7c92e-6b21-4d29-ba6f-59f4480b27a0">

RTS가 성공해야 CTS가 성공한다.

SIFS, DIFS의 간격은 우선순위를 나타낸다.

## Controlled-access
---

### Reservation
---

<img width="791" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/51d82757-909d-47f6-b110-9f60202a8857">

데이터를 보내기 전에 노드는 먼저 슬롯을 예약한다.

각 간격에서 예약 프레임은 해당 간격에서 전송된 데이터 프레임보다 우선시 된다.

충돌 없음

### Polling
---

HDLC에서 사용

중앙 제어 장치에 의존적, 이 장치에 문제가 생기면 전체 네트워크가 중단된다.

충돌 없음

<img width="843" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/430d0284-e764-4fcc-babc-80a812dbbd69">

### Token passing
---

네트워크 상의 노드들은 논리적인 Ring을 형성한다.

Token을 돌리면서 통신을 한다.

Token이 손실되면, 네트워크가 중단될 수 있다.

충돌 없음

<img width="561" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/e157b731-c20d-4a89-be7f-2c0579f3d867">


## Channelization
---

여러 노드가 동시에 채널을 공유할 수 있다.

### FDMA(Frequency Division Multiple Access)
---

<img width="671" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/9ad25768-872e-4a73-b30f-25c3d462df08">

전체 주파수 대역을 노드 별로 나눠서 사용한다.

간단, 각 노드가 독립적으로 통신 가능

주파수 대역을 효율적으로 사용하기가 어렵다

### TDMA(Time Division Multiple Access)
---

<img width="759" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/aa99c43e-4ab7-49b4-a8d1-8de7309f9d7d">

각 노드에게 시간 슬롯을 할당한다.

할당받은 시간 슬롯만큼만 통신이 가능하다.

주파수 대역을 효율적으로 사용할 수 있다.

동기화가 필요하다.

### CDMA(Code Division Multiple Access)
---

각 노드에게 고유한 코드를 할당한다.

이 코드를 사용하여 모든 노드가 동시에 같은 주파수 대역을 공유할 수 있다.

주파수 대역을 효율적으로 사용할 수 있다.

복잡한 동기화가 필요

<img width="465" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/4a032f8e-c2dc-4d42-acb6-86e0283c311c">

<img width="1095" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/43d2aee3-39fd-408e-94ec-87105f9f3d70">

<img width="740" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/283513e4-9c4a-4e6c-ace0-38acc80d56c4">

