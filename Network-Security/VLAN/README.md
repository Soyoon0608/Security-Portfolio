VLAN 구성 및 네트워크 분리 실습

1. 실습 개요

하나의 스위치 네트워크를 VLAN으로 논리적으로 분리하고, VLAN별로 서로 다른 IP 네트워크를 구성하는 실습을 진행했다.

단순히 VLAN을 생성하는 것에서 끝내지 않고,

* VLAN 생성
* 단말 포트의 VLAN 할당
* VLAN 간 네트워크 분리
* Trunk를 통한 VLAN 전달
* VLAN 인터페이스 구성
* 설정 및 동작 확인

까지 단계별로 구성하였다.

---

2. 실습 환경

* Cisco Switch
* Cisco Router / Layer 3 장비
* PC
* GNS3 / Packet Tracer

---

3. 네트워크 설계

하나의 스위치에 연결된 단말을 세 개의 VLAN으로 분리하였다.

| VLAN    | 네트워크            | 용도     |
| ------- | --------------- | ------ |
| VLAN 11 | 192.168.11.0/24 | 네트워크 1 |
| VLAN 12 | 192.168.12.0/24 | 네트워크 2 |
| VLAN 13 | 192.168.13.0/24 | 네트워크 3 |

VLAN을 사용하여 물리적으로 하나의 스위치에 연결된 단말들을 논리적으로 서로 다른 네트워크로 분리하였다.

---

4. VLAN 생성

먼저 스위치에 VLAN 11, 12, 13을 생성하였다.

```cisco
vlan database
vlan 11
vlan 12
vlan 13
apply
exit
```

생성된 VLAN은 다음 명령어로 확인하였다.

```cisco
show vlan-switch brief
```

확인 결과

VLAN 11, VLAN 12, VLAN 13이 스위치에 생성되고 활성화된 것을 확인하였다.

---

5. Access Port 구성

각 단말이 연결되는 포트를 VLAN별로 할당하였다.

VLAN 11

```cisco
interface range fastEthernet 1/1 - 5
switchport mode access
switchport access vlan 11
exit
```

### VLAN 12

```cisco
interface range fastEthernet 1/6 - 10
switchport mode access
switchport access vlan 12
exit
```

VLAN 13

```cisco
interface range fastEthernet 1/11 - 15
switchport mode access
switchport access vlan 13
exit
```

이를 통해 단말이 연결된 포트에 VLAN을 지정하였다.

---

6. Trunk 구성

스위치 간 연결에서는 여러 VLAN의 트래픽을 하나의 링크를 통해 전달하기 위해 Trunk를 구성하였다.

```cisco
interface fastEthernet 1/0
switchport mode trunk
```

Trunk 포트에서는 VLAN 정보를 함께 전달할 수 있도록 구성하였다.

설정 상태는 다음 명령어로 확인하였다.

```cisco
show interfaces trunk
```

Cisco 장비에서는 `show interfaces <interface> trunk` 명령어를 통해 Trunk 상태와 전달 가능한 VLAN 정보를 확인할 수 있다.

---

7. VLAN 인터페이스 구성

VLAN별로 IP 네트워크를 구성하여 각 VLAN의 게이트웨이 역할을 할 인터페이스를 설정하였다.

```cisco
interface vlan 11
ip address 192.168.11.1 255.255.255.0

interface vlan 12
ip address 192.168.12.1 255.255.255.0

interface vlan 13
ip address 192.168.13.1 255.255.255.0
```

각 VLAN에 서로 다른 네트워크 대역을 할당하여 VLAN별 네트워크를 구분하였다.

---

8. Native VLAN 확인

실습 과정에서 Native VLAN의 개념도 확인하였다.

기본 VLAN인 VLAN 1이 존재하며, VLAN 인터페이스를 이용하여 스위치 관리 목적의 IP를 설정할 수도 있음을 확인하였다.

예시:

```cisco
interface vlan 1
ip address 192.168.11.1 255.255.255.0
```

다만 VLAN 1을 실제 사용자 네트워크의 게이트웨이로 사용할 것인지, 관리 목적으로 사용할 것인지는 네트워크 설계에 따라 구분해야 한다.

---

9. 검증

구성이 완료된 후 다음 명령어를 이용하여 VLAN과 Trunk 상태를 확인하였다.

### VLAN 확인

```cisco
show vlan brief
```

확인을 통해 VLAN이 생성되어 있는지, 각 포트가 올바른 VLAN에 할당되었는지 확인하였다.

Trunk 확인

```cisco
show interfaces trunk
```

Trunk 포트가 정상적으로 동작하고 VLAN 트래픽을 전달할 수 있는지 확인하였다.

인터페이스 확인

```cisco
show ip interface brief
```

VLAN 인터페이스의 IP 주소와 상태를 확인하였다.

---

10. 문제 상황 및 해결

실습 과정에서 VLAN 설정과 Layer 2/Layer 3 인터페이스의 차이를 확인하였다.

특히 Layer 2 스위치의 물리 포트는 일반적인 Access/Trunk 포트로 동작하므로 해당 포트에 라우터처럼 IP 주소를 직접 설정하는 방식이 아니라 VLAN 인터페이스(SVI)를 이용하여 IP를 설정해야 한다는 점을 확인하였다.

또한 VLAN 간 통신을 위해서는 단순히 VLAN을 생성하는 것만으로는 부족하며, 서로 다른 네트워크 사이의 라우팅 기능이 필요하다는 것을 확인하였다.

---

11. 결과

VLAN 11, 12, 13을 생성하고 각 포트를 VLAN별로 분리하였다.

또한 스위치 간 연결을 Trunk로 구성하여 하나의 물리적인 링크에서 여러 VLAN의 트래픽을 전달할 수 있도록 구성하였다.

최종적으로 다음과 같은 구조를 구성하였다.

```text
                 ┌───────────────┐
                 │     Switch    │
                 └───────┬───────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       VLAN 11         VLAN 12        VLAN 13
   192.168.11.0/24  192.168.12.0/24  192.168.13.0/24
          │              │              │
        PC들            PC들           PC들
```

VLAN을 통해 하나의 물리적 스위치 환경을 논리적으로 여러 네트워크로 분리할 수 있음을 확인하였다.

---

12. 핵심 학습 내용

이번 실습을 통해 다음 내용을 직접 구성하고 확인하였다.

* VLAN을 이용한 논리적 네트워크 분리
* Access Port와 VLAN의 관계
* Trunk Port를 이용한 VLAN 전달
* VLAN별 IP 네트워크 구성
* SVI(VLAN Interface)의 역할
* VLAN과 라우팅의 관계
* `show vlan brief`를 이용한 VLAN 상태 확인
* `show interfaces trunk`를 이용한 Trunk 상태 확인

특히 **"VLAN을 생성하는 것"과 "서로 다른 VLAN끼리 통신할 수 있도록 라우팅하는 것"은 별개의 과정**이라는 점을 이해하는 것을 핵심 목표로 하였다.
