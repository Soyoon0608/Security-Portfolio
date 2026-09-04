VLAN 실습

1. 실습 목적

하나의 스위치 네트워크를 VLAN을 이용하여 논리적으로 분리하고,
각 VLAN에 서로 다른 네트워크를 구성하는 과정을 실습했다.

VLAN을 통해 하나의 물리적인 스위치 환경을 여러 개의 논리적인 네트워크로 분리하고,
Access Port와 Trunk Port의 역할을 이해하는 것을 목표로 했다.

---

2. 학습 내용

* VLAN의 개념
* VLAN 생성
* Access Port 설정
* Trunk Port 설정
* VLAN별 네트워크 분리
* VLAN 설정 확인
* VLAN 인터페이스(SVI) 설정
* VLAN 간 통신을 위한 라우팅

---

3. 실습 환경

* Cisco Switch
* Cisco Router
* PC
* GNS3
  
---

4. VLAN 구성

이번 실습에서는 3개의 VLAN을 구성하였다.

| VLAN    | 네트워크        |
| ------- | --------------- |
| VLAN 11 | 192.168.11.0/24 |
| VLAN 12 | 192.168.12.0/24 |
| VLAN 13 | 192.168.13.0/24 |

각 VLAN은 서로 다른 네트워크로 분리하여 구성하였다.

---

5. VLAN 생성

스위치에서 VLAN 11, 12, 13을 생성하였다.

R1

```cisco
vlan database
vlan 11
vlan 12
vlan 13
apply
exit
```

R2

```cisco
vlan database
vlan 11
vlan 12
vlan 13
apply
exit
```

---

6. VLAN 생성 확인

VLAN이 정상적으로 생성되었는지 확인하였다.

```cisco
show vlan-switch brief
```

이 명령어를 통해 생성된 VLAN과 VLAN에 할당된 포트를 확인할 수 있다.

---

7. Access Port 설정

각 포트를 특정 VLAN에 할당하였다.

VLAN 11

```cisco
interface range fastEthernet 1/1 - 5
switchport mode access
switchport access vlan 11
exit
```

VLAN 12

```cisco
interface range fastEthernet 1/6 - 10
switchport mode access
switchport access vlan 12
end
```

VLAN 13

```cisco
interface range fastEthernet 1/11 - 15
switchport mode access
switchport access vlan 13
end
```

포트 구성

| 포트            | VLAN    |
| --------------- | ------- |
| Fa1/1 ~ Fa1/5   | VLAN 11 |
| Fa1/6 ~ Fa1/10  | VLAN 12 |
| Fa1/11 ~ Fa1/15 | VLAN 13 |

Access Port는 일반적으로 PC와 같은 단말을 연결할 때 사용하며, 해당 포트는 지정된 하나의 VLAN에 속하도록 구성하였다.

---

8. VLAN 인터페이스 설정

VLAN별로 논리적인 인터페이스를 구성하였다.

R1

```cisco
interface vlan 11
ip add 192.168.11.1 255.255.255.0

interface vlan 12
ip add 192.168.12.1 255.255.255.0

interface vlan 13
ip add 192.168.13.1 255.255.255.0
```

R2

```cisco
interface vlan 11
ip add 192.168.11.2 255.255.255.0

interface vlan 12
ip add 192.168.12.2 255.255.255.0

interface vlan 13
ip add 192.168.13.2 255.255.255.0
```

이를 통해 각 VLAN에 서로 다른 IP 네트워크를 연결하였다.

---

9. Native VLAN 참고

실습 과정에서 VLAN 1은 기본 VLAN으로 존재한다.

```cisco
interface vlan 1
ip add 192.168.11.1 255.255.255.0
```

VLAN 1은 스위치에서 기본적으로 존재하며, VLAN 설정과 별도로 관리 및 Native VLAN과 관련된 개념을 확인하였다.

---

10. Trunk Port

Trunk Port는 여러 VLAN의 트래픽을 하나의 링크를 통해 전달하기 위해 사용한다.

스위치 간 연결에서 VLAN 정보를 전달하기 위해 Trunk를 구성할 수 있다.

```cisco
interface fastEthernet 1/15
switchport trunk encapsulation dot1q
switchport mode trunk
no shutdown
exit
```

802.1Q 방식으로 VLAN 태그를 사용하여 여러 VLAN의 트래픽을 하나의 물리적인 링크에서 구분할 수 있다.

---

11. VLAN 설정 확인

VLAN 확인

```cisco
show vlan-switch brief
```

Trunk 확인

```cisco
show interfaces trunk
```

`show vlan-switch brief` 명령어를 통해 VLAN과 포트 할당 상태를 확인하고,
`show interfaces trunk` 명령어를 통해 Trunk Port의 동작 상태를 확인하였다.

---

12. 네트워크 구성 흐름

```text
PC
 │
 │ Access Port
 ▼
Switch
 │
 ├── VLAN 11 ── 192.168.11.0/24
 │
 ├── VLAN 12 ── 192.168.12.0/24
 │
 └── VLAN 13 ── 192.168.13.0/24
```

스위치에서 단말을 VLAN별로 분리하고, VLAN별로 서로 다른 IP 네트워크를 구성하였다.

---

13. 실습을 통해 이해한 내용

VLAN

VLAN은 하나의 물리적인 스위치 네트워크를 논리적으로 분리하는 기술이다.

예를 들어 하나의 스위치에 연결된 PC들을 VLAN 11, VLAN 12, VLAN 13으로 나누면 서로 다른 논리적인 네트워크로 분리할 수 있다.

Access Port

Access Port는 특정 하나의 VLAN에 단말을 연결할 때 사용한다.

```text
PC → Access Port → 특정 VLAN
```

Trunk Port

Trunk Port는 여러 VLAN의 트래픽을 하나의 링크를 통해 전달할 때 사용한다.

```text
VLAN 11 ─┐
VLAN 12 ─┼→ Trunk Link
VLAN 13 ─┘
```

---

14. 실습에서 확인한 핵심

이번 실습을 통해 다음과 같은 VLAN 구성 과정을 직접 수행하였다.

1. VLAN 생성
2. VLAN별 네트워크 구성
3. Access Port에 VLAN 할당
4. VLAN 인터페이스 설정
5. Trunk Port 구성
6. `show` 명령어를 이용한 설정 확인

단순히 VLAN의 개념을 학습하는 것에서 끝내지 않고 Cisco 장비에서 직접 VLAN을 생성하고 포트에 할당하면서 VLAN의 동작 방식을 확인하였다.
