---
layout: post
title: 11. Data Link Control
subtitle: Data communication - 11. Data Link Control
published: true
categories: Data communication
tags: [Data communication]
---

# DLC Services

1. Framing
2. Flow Control
3. Error Control

## Framing
---

### Character-Oriented Framing
---
<img width="520" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/05274a1a-99a0-4f9a-9891-3647f79f49c3">

데이터를 구별하기 위해 flag를 앞 뒤에 추가한다.

데이터 안에 flag패턴이 포함되어 있으면, 혼동이 생기므로, 데이터 내에 추가적인 패턴을 삽입하여 구분하게 된다. 이를 byte stuffing이라고 한다.

Header에 검증을 위한 데이터를 추가한다.

아스키 코드 전달이 목적

최근에는 잘 사용되지 않는다.

### Bit-Oriented Framing
---
<img width="508" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/2709a90b-2438-40fb-ad67-fe3d7712d632">


비트 단위로 데이터를 구별하기 위해 flag를 사용한다.

Character 방식과 마찬가지로 flag와 같은 패턴이 데이터에 나타날 수 있으므로, Bit stuffing/unstuffing 기법을 사용한다.


## Flow Control
---

### Buffers
---
<img width="509" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/2bc79391-f7ae-4bcf-a24a-6e7e803a86ef">

Flow와 Error Control을 같이 진행하면 더 효율적이다.

프레임을 전송하기 위해 메모리 버퍼에 저장한다. 수신 측에서도 버퍼에 저장한다.

송수신 간의 통신 속도 차이 때문에 흐름 제어가 필요하다.

송신 측은 수신 측의 상태를 고려하여 데이터를 전송한다.

헤더에 ACK를 추가하여 흐름 제어를 한다.

헤더의 내용이 중요하다.

## Error Control
---

Error Detection

프레임이 손상된 경우, 그 프레임을 버린다.

손상되지 않은 경우, ACK를 보낸다.

### Connectionless & Connection-Oriented protocol
---

* Connectionless
  * 데이터를 전송하기 전에, 연결을 설정하지 않는다.
  * 신속한 데이터 처리에 목적
  * 데이터 순서 유지 보장이 없다.
  * UDP
* Connection-Oriented(연결 지향적)
  * 장거리 통신시 사용
  * 데이터 전송의 신뢰성을 보장.
  * TCP

### Data Link Layer Protocols
---

* 표현 방법
1. Finite state machine
2. Flow Diagram

* 종류(For Noisy Channel)
1. Stop-and-Wait
2. Slide Window
  1. Go-Back-N
  2. Selective Repeat


### Stop-and-Wait
---
<img width="536" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/157db311-483d-4e49-9743-56c2806cfe50">

* 송신 측
  * 네트워크 계층에서 패킷을 수신
  * 프레임 생성
  * 복사하고 저장
  * 프레임 전송
  * 타이머 시작하고 Blocking 모드로 전환
  * ACK를 기다린다.
    * 정상적인 ACK를 받으면 타이머를 멈추고, 저장한 프레임을 제거한 뒤 다음 프레임을 보낸다.
    * 오류있는 ACK를 받으면 버린다.
    * 타임 아웃될 때 까지 ACK를 받지 못하면 프레임을 재 전송한다.
* 수신 측
  * 프레임을 받으면 ACK를 보낸다.
  * 이미 받은 프레임이 중복되어 오면 그 프레임은 버린다.
    * 이 경우에도 ACK는 보낸다.

* 단점
  * 처리 속도가 느리다
  * 한 프레임씩 전송하기 때문에 효율이 떨어진다.
  * 전송의 성공 여부를 타이머에 의존한다.
  
개선 방법 : Pipelining

### Pipelining
---

보낼 프레임이 있으면 계속 보내는 방법.

송신 중에 오류가 발생하면 대처가 어렵다.

개선 방법 : Slide Window

### Slide Window
---

### Go-Back-N
---

* Window Size <= 2^m - 1
* Sequence number bits = 2

pipelining을 하고 오류 처리를 개선한다.

<img width="571" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/577e6fd5-ca05-4e41-bc78-eb10ba27812c">

* 송신 측
  * Window Size만큼 데이터를 전송한다.
  * ACK를 받으면 Window를 이동시킨다.
  * ACK를 받지 못하면 Window의 시작부터 다시 전송한다.
  * ACK를 받지 못하면 타이머가 만료되면 Window의 시작부터 다시 전송한다.


### Selective Repeat
---

* Window Size <= 2^m - 1
* NAK를 사용한다.
  * NAK 1 : 1번 프레임을 재전송하라


<img width="708" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/ac471b10-8707-4e9a-86f7-30645c3a678c">


### HDLC(High-Level Data Link Control)
---

 * Support Connection
   * 데이터를 전송하기 전에 전처리 과정을 거친다.

* NRM
  * 일반적인 전송 모드
  * 데이터 순서가 유지된다.
  * 하나의 주요 스테이션과 여러 개의 보조 스테이션으로 구성된다.
* ABM
  * 비동기 전송 모드
  * 데이터를 동시에 전송 가능
  * 데이터 순서가 보장되지 않음.
  * P2P

* HDLC파생 프로토콜
  * SDLC
  * PPP


### HDLC Frame
---

<img width="618" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/b08f6a7e-91f8-4068-b4a3-87b95194cbca">

* flag
  * 01111110
* address
  * byte or multi-byte
* control
* information
  * 서로 간에 프로토콜의 정하기 위해 사용
  * User information
  * Management information
* FCS(Frame Check Sequence)
  * CRC

bit stuffing 방식을 사용한다.

* Frame 종류
  * I : 정보 프레임
    * 실제 데이터를 전송
    * 데이터 순서를 유지하기 위해 관리
  * S : 감독 프레임
    * 흐름 제어, 오류 복구
    * I-Frame에 대한 응답으로 사용
    * 정보 전송은 하지 않음
  * U : 비정보 프레임
    * 다양한 제어, 관리 기능

* 각 프레임 별 Control field
  * I
  * S
    * ready, reject, etc...
  * U
    * 세션 관리, 제어 정보
    * Unnumbered frames


### PPP(Point-to-Point Protocol)
---

다중 네트워크 계층 프로토콜이다.

Internet, OSI, Xerox, AppleTalk, DECnet, IPX/SPX 등을 지원한다.

* Service
  * Format of the frame
  * 링크를 수립하고 데이터를 교환한다
  * 몇몇 네트워크 계층으로 부터의 데이터를 허용
  * 인증
  * Multilink PPP


* 제공 되지 않음
  * Flow control
  * Error control에 대한 아주 간단한 방법만 제공


### PPP frame format
---

<img width="648" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/4b679c6d-86df-42ac-9e1a-81b656838c43">

* flag
* protocol
* payload
  * 데이터가 포함된다
* fcs

byte stuffing을 사용한다.

escape byte는 01111101 이다.

address, control field는 실제로 사용되지 않는다.

address의 값이 1이면, 브로드 캐스트 전송을 한다.


### Multiplexing in PPP
---

<img width="602" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/0e6d3462-cd99-40d9-9c90-60095cd8eb9f">

PPP는 여러 개의 네트워크 계층 프로토콜을 지원한다.

LCP 패킷은 프레임의 payload에 포함된다. 프로토콜 값은 0xC021이다.

IPCP역시 마찬가지이다. IPCP는 NCP에 종속되므로 프로토콜 값이 0x8021이다.

### 인증 프로토콜
---

* PAP(Password Authentication Protocol)
  * ID, Password를 전송한다.
  * 보안에 취약하다.

<img width="486" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/9f1f005b-f180-4255-be10-be14e8d9f46f">

* CHAP(Challenge Handshake Authentication Protocol)
  * Three way handshake
  * Challenge & Response

<img width="540" alt="image" src="https://github.com/pknujsp/android-smartdeeplink/assets/48265129/58663ad1-fe48-484e-9247-e16b21f66f0a">



## 정리
---

### DataLink Layers Protocols
---

|                    | Stop-And-Wait                                     | Go-Back-N                                                                                            | Selective Repeat                                                                                       |
| ------------------ | ------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| 통신 속도          | 느림                                              | 빠름                                                                                                 | 빠름                                                                                                   |
| 재전송 프레임 개수 | 1                                                 | N                                                                                                    | 1, 손상된 프레임만 재전송                                                                              |
| 동작 방식          | 프레임 하나씩 처리, ACK를 받아야 다음 프레임 전송 | Window Size만큼 프레임을 보내고, 특정 프레임이 손상되어 ACK를 받지 못하면, 그 프레임부터 다시 재전송 | Window Size만큼 프레임을 보내고, 손상된 프레임에 대해서만 재전송, ACK를 받지 못한 프레임만 재전송 한다 |


### HDLC, PPP 프레임 비교
---

| | HDLC | PPP |
| --- | --- | --- |
| 프레임 종류 | I, S, U | 