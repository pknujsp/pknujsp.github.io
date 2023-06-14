---
layout: post
title: 15. Wireless LANs
subtitle: Data communication - 15. Wireless LANs
published: true
categories: Data communication
tags: [Data communication]
---

# Wireless LANs
---

* Wireless Ethernet
  * CSMA/CA
* Personal area network(PAN)
  * Bluetooth

## Architecture Comparison
---

* Medium
* Hosts
* Isolated LANs
* Connection to Other Networks

## 특징
---

* Attenuation
* Interference
* Multipath Propagation
  * 전파 반사로 인해 매우 다양한 경로가 만들어 진다.
* Error

## Access Control
---

* CSMA/CD는 무선랜으로 동작하지 않는다.
  * 충돌 탐지 문제
  * Hidden station 문제
  * 신호 약화 문제
* 이러한 문제를 극복하기 위해, CSMA/CA를 사용한다.

## 802.11
---

<img width="683" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/9ca9581e-fe62-4f87-9e30-bb7aef49b7af">

MAC Layers 802.11 standard

<img width="823" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/1bd722bd-7829-4e60-952d-9442c257983a">

* MAC Sublayer
  * Distributed Coordination Function(DCF)
    * CSMA/CA
  * Point Coordination Function(PCF)
    * in an infrastructure network
    * not in an ad hoc network

## CSMA/CA and NAV
---

<img width="407" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/a00681e4-4902-47c9-b118-831672babcc3">

<img width="413" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/09cb2ba7-3d45-41c3-bac4-d4dc2891ff06">

hidden station이 있는 경우, 충돌 감지가 불가능하다.

## Point Coordination Function
---

<img width="503" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/a1ec0378-b7ff-4ad3-8386-c9d0b946736b">

Frame format

<img width="926" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/375a2e36-cb0c-45b4-a37e-438d25a683a9">

### Hidden station problem
---

<img width="666" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/14f5af27-0691-4bd4-8c06-f5f8d10f3e1b">

<img width="993" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/f0d38d54-8b64-4c2d-a2e2-a9f7c9c5ae40">


## 802.11 Series
---

## Bluetooth
---

Piconet 또는 Small net이라고 불린다.

## Scatternet
---

하나의 Piconet 내의 보조 스테이션이 다른 Piconet의 주요 스테이션이 될 수 있다.

## Bluetooth layers
---

<img width="984" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/f0719631-c676-4292-935b-b77dcf36818c">

## Baseband Layer
---

* Baseband layer는 LAN의 MAC sublayer와 유사하다.
* 접근 메서드는 TDMA이다.
  * 625us의 슬롯으로 나누어진다.
  * 스테이션들은 서로 타임 슬롯을 통해서 통신한다.

<img width="726" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/4da576e7-1368-4eeb-8d8b-4abd111866e6">

