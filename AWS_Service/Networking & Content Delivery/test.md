# Public IP

- IPv4 기준 약 43억 개

# Private IP

- 한정된 IP 주소를 최대한 활용하기 위해 IP주소 분할하고자 만든 개념

- 사설망: 사설망 내부에는 외부 인테넛 망으로 통신이 불가능한 사설 IP로 구성

- 외부 통신 시 통신 가능한 공인 IP로 나누어 사용

# NAT | Network Address Translation

- 사설 IP가 공용 IP로 통신할 수 있도록 주소를 변환해 주는 방법

1. Dynamic NAT : 1개의 사설 IP를 가용 가능한 공용IP로 연결
    * 공인 IP 그룹(NAT Pool)에서 현재 사용 가능한 IP를 가져와서 연결
2. Static NAT : 하나의 사설 IP를 고정된 하나의 공인 IP로 연결
    * AWS Internet Gateway가 사용하는 방식


3. PAT(Port Address Translation): 많은 사설 IP를 하나의 공인 IP로 연결
    * NAT Gateway & NAT Instance가 사용하는 방식 
    * NAT Gateway : 
    * NAT Instance : 
> 포트 포워딩
# NAPT

# CIDR | Classless Inter Domain Routing

- IP는 주소의 영역을 여러 Network 영역으로 나누기 위해 IP를 묶는 방식
- 여러 개의 사설망을 구축하기 위해 망을 나누는 방법

CIDR Block : IP 주소의 집합
CIDR Notaion : CIDR Block을 표시하는 방법
    * 네트워크 주소와 호스트 주소로 구성
    * 각 호스트 주소 숫자 만큼의 IP를 가진 네트워크 망 형성 가능

> A.B.C.D/E 형식
A.B.C: 네트워크 주소 (불변), 각 ~8bit
D: 호스트 주소 (가변) ~8bit
E : 0~32 네트워크 주소가 몇 bit인지 표시

계산 법

32 - E = X
2^X = 호스트 주소 가능 범위


# Subnet

Network 안의 Network
큰 network를 잘게 쪼갠 단위
일정 IP 주소의 범위를 보유

    큰 Network에 부여된 IP 범위를 조금씩 잘라 작은 단위로 나눈 후 각 서브넷에 할당 (CIDR)

큰 Network (VPC) 192.168.0.0/16
    * 2^16 대역을 가짐
subnet_dev | 192.168.1.0/24
    * 2^8 대역 1
subnet_prod | 192.168.2.0/24
    * 2^8 대역 2
subnet_test | 192.168.2.0/24
    * 2^8 대역 3


# VPC | Virtual Private Cloud

사용자의 AWS 계정 전용 가상 Network
VPC는 AWS에서 다른 가상 Network와 논리적으로 분리되 있음.
Amazon EC2와 같은 AWS 리소스를 VPC에서 실행할 수 있음.
IP 주소 범위와 VPC 범위를 설정하고,
서브넷을 추가하고 보안 그룹을 연결한 다음 라우팅 테이블을 구성

- 가상으로 존재하는 데이터 센터
- 외부에 격리된 Netwrok 컨테이너 구성 가능
    * 원하는 대로 사설망 구축 가능
    * 부여된 IP 대역을 분할하여 사용 가능
- 리전 단위

![A](./img/VPC.JPG)

## VPC 사용 사례
- EC2,RDS, Lambda 등 AWS 컴퓨팅 서비스 실행
- 다양한 subnet 구성
- 보안 설정 (IP Block, 인터넷에 노출되지 않는 EC2 등 구성)

## VPC 구성 요소

- Subnet
    * VPC의 하위 단위로 VPC에 할당된 IP를 더 작은 단위로 분할한 개념
    * 하나의 서브넷은 하나의 가용영역(AZ) 안에 위치
    * CIDR Block range로 IP 주소 지정

    AWS 서브넷 IP 개수 --> 전체 - 5
    이유 
    - 10.30.40.0 : Network Address
    - 10.30.40.1 : VPC Router
    - 10.30.40.2 : DNS Server
    - 10.30.40.3 : AWS 지정 여분
    - 10.30.40.255 : Network 브로드캐스트 Address (단, 브로드캐스트는 지원하지 않음)

    * Public Subnet
        - IGW(Internet Gateway)을 통해 외부의 인터넷과 연결되어 있음
        - 안에 위치한 인스턴스에 퍼블릭 IP 부여 가능
        - 웹서버, WAS 등 유저에게 노출되어야 하는 인프라
    * Private Subnet
        - 외부 인터넷으로 연결된 경로가 없음
        - public ip 부여 불가
        - DB, 로직 서버 등 외부 노출이 필요없는 인프라

- Route Table
    * 트래픽이 어디로 가야할지 알려주는 이정표
    * 가장 구체적인 IP오 매칭 (subnet mask 높은 숫자)
    * VPC 생성 시 기본으로 1개 제공


- 인터넷 Gateway
    - VPC 가 외부의 인터넷과 통신할 수 있도록 경로를 만들어주는 리소스
    - 기본적으로 확장성과 고가용성이 확보되어 있음
    - IPv4(NAT 역할), IPv6 지원
    - Route Table에서 경로 설정 후에 접근 가능
    - 무료
    - Public Subnet에서만 연결되어 있음(Public Subnet->Route Table-> Internet Gateway)

- NACL / 보안그룹
- NAT Instance / NAT Gateway
- Bastion Host
- VPC Endpoint





## VPC 구축 이론

* Basic Architecture
![A](./img/VPC_Basic_Architecture.JPG)

best practice

- VPC는 Private Subnet과 Public Subnet으로 나누어 운영

- AZ 존을 나눈다. (고가용성 -> 3개 이상)


- IP를 고정하지 않는다.

- 각 AZ에 NAT Gateway를 1개 씩 사용

- Route Table 3개, Public 용 1개 

- Private Subnet은 용도별로 각 1개 씩 사용

