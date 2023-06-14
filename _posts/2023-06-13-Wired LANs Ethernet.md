---
layout: post
title: 13. Wired LANs Ethernet
subtitle: Data communication - 13. Wired LANs Ethernet
published: true
categories: Data communication
tags: [Data communication]
---

# Ethernet
---

* Wired LAN
  * Token Bus
  * Token Ring
  * FDDI
  * ATM LAN

<img width="696" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/a9c8694a-35c8-41e6-b6d2-87eb926ddbbe">

## Standard Ethernet
---

Connectionless and Unreliable Service

Frame format

<img width="843" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/5729efaa-fef3-4e8d-a653-0a6409af9860">

* preamble
  * 7 bytes 0101...
* start frame delimiter(sfd)
  * 1 byte 10101011
* destination address
  * 6 bytes
* source address
  * 6 bytes
* type
  * upper layer protocol
* data
  * 46 ~ 1500 bytes
* crc-32

<img width="510" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/1d492d43-a385-4e56-9d09-16a681a0fd97">

46~1500 bytes 전송 가능하다.

1 bytes만 전송하려면, 45 bytes의 fake data를 포함해야 한다.

## Addressing
---

* Network interface card(NIC) address
* Unicast, Multicast, Broadcast addresses
  * Broadcast : 48 1s

unicast이면 0을, multicast이면 1을 사용한다.

## Categories of Standard Ethernet
---

<img width="540" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/cc98e2ae-cecf-4ae6-ad07-4ca372ffb27e">

뒤에가 숫자이면, 동축 케이블이다.

## Changes in the Standard
---

* Bridged Ethernet
* Switched Ethernet
* Full-duplex Ethernet


### Bridged Ethernet
---

Separating collision domains

<img width="556" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/7ef7d26a-e06b-4f3f-a37a-44c766ca60df">

Bridge로 인해 Collision domain이 2개로 나눠진다.

### Switched Ethernet
---

Separating collision domains completely

<img width="748" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/05ce3c29-08b4-46a5-bcc5-11d07601d7f3">


### Full-duplex Ethernet
---

No need for CSMA/CD

<img width="556" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/eaf8bc5e-bbee-46b0-9ae1-0ef08cf07dc3">


## Fast Ethernet
---

* 100 Mbps 이더넷
  * 48비트 주소
  * 같은 프레임 포맷
  * Autonegotiation
    * 서로 동작 속도가 다른 연결이 있을 경우 맞춰서 하향 조정한다.


## Gigabit Ethernet
---

* 1 Gbps 이더넷
  * 48비트 주소
  * 같은 프레임 포맷
    * 최소/최대 프레임 길이 유지
  * Autonegotiation