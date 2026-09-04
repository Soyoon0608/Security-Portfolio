NAT/PAT 구성 및 주소 변환 실습

1. 실습 개요

사설 IP를 사용하는 내부 네트워크가 외부 네트워크와 통신할 수 있도록 **NAT(Network Address Translation)**과 **PAT(Port Address Translation)**을 구성하였다.

하나의 라우터를 기준으로 내부 네트워크와 외부 네트워크를 구분하고, 사설 IP가 공인 IP로 변환되는 과정을 직접 확인하였다.

주요 실습 내용은 다음과 같다.

* NAT Inside / Outside 인터페이스 지정
* Access List를 이용한 변환 대상 지정
* Static/Dynamic NAT 방식 이해
* PAT Overload 구성
* NAT 변환 테이블 확인
* NAT 통계 확인

---

2. 실습 환경

* Cisco Router
* PC 2대
* Server
* Packet Tracer

---

3. 네트워크 구성

```text
PC0 ─ Router0 ─ Router1 ─ Server
        │
      (PAT)
PC1 ────┘
```

내부 네트워크의 PC 2대가 하나의 라우터를 통해 외부 네트워크의 Server와 통신하도록 구성하였다.

IP 주소

| 장비     | 인터페이스  | IP 주소         |
| ------ | ------ | ------------- |
| PC0    | -      | 192.168.10.10 |
| PC1    | -      | 192.168.10.20 |
| R1     | G0/0   | 192.168.10.1  |
| R1     | S0/0/0 | 1.1.12.1      |
| R2     | S0/0/0 | 1.1.12.2      |
| Server | -      | 2.2.2.2       |

---

4. NAT 변환 대상 지정

먼저 Access List를 이용하여 NAT 변환 대상이 되는 내부 네트워크를 지정하였다.

```cisco
access-list 10 permit 192.168.10.0 0.0.0.255
```

이를 통해 `192.168.10.0/24` 네트워크에 포함된 내부 주소를 NAT 대상으로 지정하였다.

---

5. NAT 구성

NAT Pool을 생성하고 내부 사설 IP를 외부 IP로 변환하도록 구성하였다.

```cisco
ip nat pool DNAT 1.1.12.100 1.1.12.100 netmask 255.255.255.0

ip nat inside source list 10 pool DNAT
```

이후 라우터의 인터페이스에 내부(Inside)와 외부(Outside)를 지정하였다.

Inside 인터페이스

```cisco
interface g0/0
 ip nat inside
```

Outside 인터페이스

```cisco
interface s0/0/0
 ip nat outside
```

이를 통해 라우터가 어떤 방향의 트래픽을 내부/외부로 처리해야 하는지 구분하도록 구성하였다.

---

6. PAT 구성

여러 개의 내부 사설 IP가 하나의 외부 IP를 공유할 수 있도록 PAT를 구성하였다.

```cisco
ip nat pool PAT 1.1.12.100 1.1.12.100 netmask 255.255.255.0

ip nat inside source list 10 pool PAT overload
```

여기서 `overload` 옵션을 사용하여 여러 내부 단말이 하나의 공인 IP를 공유하도록 구성하였다.

즉,

```text
PC0 192.168.10.10 ─┐
                   ├─→ 1.1.12.100
PC1 192.168.10.20 ─┘
```

와 같이 여러 사설 IP가 하나의 외부 IP를 사용할 수 있다.

PAT에서는 각 통신을 구분하기 위해 포트 번호 등의 정보를 함께 사용한다.

---

7. NAT 변환 확인

구성 후 NAT가 실제로 동작하는지 변환 테이블을 확인하였다.

```cisco
show ip nat translations
```

NAT 변환 상태에 대한 통계 정보는 다음 명령어로 확인하였다.

```cisco
show ip nat statistics
```

이를 통해 내부 사설 IP가 외부 주소로 변환되었는지 확인할 수 있었다.

---

8. NAT와 PAT의 차이

| 구분        | NAT               | PAT                         |
| ----------- | ----------------  | --------------------------- |
| 변환 방식    | 사설 IP ↔ 공인 IP | 여러 사설 IP → 하나의 공인 IP |
| 공인 IP 사용 | 1:1 변환 가능     | 하나의 IP를 여러 단말이 공유  |
| 구분 기준    | IP 주소           | IP + Port 등의 정보      |
| 주요 목적    | 주소 변환         | 공인 IP 효율적 사용         |

---

9. 실습 결과

내부 네트워크의 사설 IP를 NAT 대상으로 지정하고 라우터의 Inside/Outside 인터페이스를 구분하였다.

이후 PAT Overload를 적용하여 여러 내부 단말이 하나의 외부 IP를 공유할 수 있도록 구성하였다.

최종적으로 `show ip nat translations`와 `show ip nat statistics` 명령어를 통해 NAT 동작 상태를 확인하였다.

---

10. 핵심 학습 내용

이번 실습을 통해 다음 내용을 직접 구성하고 확인하였다.

* NAT의 주소 변환 과정
* NAT Inside / Outside의 역할
* Access List를 이용한 NAT 대상 지정
* NAT Pool 구성
* PAT Overload
* 여러 사설 IP의 공인 IP 공유
* NAT Translation Table 확인
* NAT Statistics 확인

특히 **NAT는 주소를 변환하는 기능이고, PAT는 포트 등의 정보를 활용하여 여러 내부 단말이 하나의 공인 IP를 공유할 수 있도록 한다는 차이**를 실습을 통해 확인하였다.
