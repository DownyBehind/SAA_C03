# Networking - VPC

## 세션 소개

지금까지 강의 기초가 되는 강의이다.

## CIDR, 비공개 밒 공개 IP

Understanding CIDR - IPv4

CIDR(Classless Inter-Domain Routing) - a method for allocating IP addresses

- IP 주소를 할당하는 방법
- 단순한 IP 범위를 정의하는데 도움을 준다.
  - WW.XX.YY.ZZ/32 : one IP
  - 0.0.0.0/0 : all IPs
  - 19.168.0.0/26 => 192.168.0.0 ~ 192.168.0.63 (64 IP addresses)

CIDR은 2개로 구성되어 있다.

Base IP

- Represents an IP contained in the range
  - Ex. 10.0.0.0, 192.168.0.0

Subnet Mask

- 얼마나 많은 bit가 IP내에서 변경될 수 있는지 정의
  - EX. /0. /24. /32
    - /8 == 255.0.0.0, /16 == 255.255.0.0, /24 == 255.255.255.0, /32 == 255.255.255.255

Understanding CIDR - Subnet Mask

ex. 192.168.0.0/32 : allows for 1 IP (2^0) -> 192.168.0.0
ex. 192.168.0.0/31 : allows for 2 IP (2^1)-> 192.168.0.0, 192.168.0.1
ex. 192.168.0.0/30 : allows for 2 IP (2^2)-> 192.168.0.0 ~ 192.168.0.3

마스킹되는 비트의 갯수를 의미한다.

Public vs Private IP (IPv4)

IANA에서 구축한 IPv4는 private(LAN) and public(Internet) 주소이다.

Private IP는 특정 값만 허용한다.

- 10.0.0.0 ~ 10.255.255.255 (10.0.0.0/8) -> big network
- 172.16.0.0 ~ 172.31.255.255 (172.16.0.0/12) -> AWS default VPC in that range
- 192.168.0.0 ~ 192.168.255.255 (192.168.0.0/16) -> home networks

All the rest of the IP addresses on the Internet are Public

## 기본 VPC 개요

모든 계정에는 기본 VPC가 있고, 바로 사용이 가능하다.
새로운 EC2는 서브넷을 지정하지 않으면 기본 VPC에서 실행된다.

기본 VPC는 처음부터 인터넷에 연결되어 있어서, 인스턴스가 인터넷에 엑세스하고, 내부 EC 인스턴스는 공용 IPv4 주소를 얻는다.

우리가 인스턴스를 생성하자 마자 인터넷에 연결할 수 있는 이유가 이것 때문이다.

또한 우리는 EC2 instance를 위한 public, private IP DNS 이름을 얻는다.

## VPC 개요

VPC: Virtual Private Cloud

AWS 리전 당 최대 5개까지 VPC 생성가능

VPC당 할당되 CIDR는 5개이다

- Min size is /28 (16개 IP)
- Max size is /16 (65536 IP)

VPC는 사설 네트워크이기 때문에 위에 정리한 사설 네트워크 범위만 허용된다.

## 서브넷 개요

Subnet : VPC 내부에 있는 IPv4 주소의 부분 범위이다.

AWS reserves 5 IP addresses (first 4 & last 1) in each subnet

- 이 IP는 사용도 안되고, EC2 instance IP로 할당되지도 못한다.
  - CIDR이 10.0.0.0/24라면,
    - 10.0.0.0 : Network Address
    - 10.0.0.1 : reserved by AWS for the VPC router
    - 10.0.0.2 : reserved by AWS for mapping to Amazon-provided DNS
    - 10.0.0.3 : reserved by AWS for future use
    - 10.0.0.255 : Network Broadcast Address. AWS does not support broadcast in a VPC, therefore the address is reserved

시험 팁, if you need 29 IP addresses for EC2 instances:
You can't choose a subnet size /27 (32 IP addresses 32 - 5 = 27 < 29)

- 항상 5개를 빼고 생각해야 한다.

## 인터넷 Gateway 및 라우팅

IGW(Internet Gateway)

VPC 내의 모든 리소스에 대해 인터넷 연결을 해줌
수평으로 확장되고 가용성이 좋다.
VPC와는 별개로 생성해야 한다.

1 개의 VPC는 1개의 IGW와 연결되며, 반대도 동일하다.

IGW는 혼자서는 인터넷 연결을 허용하지 않는다.
Route table이 반드시 수정되어야 한다.

EC2를 라우터에 연결하고 라우터를 IGW에 연결한다.

## Bastion 호스트

사용자가 프라이빗 서브넷에 있는 EC2에 엑세스하고자 한다.
Bastion host는 EC2 instance인데 이 인스턴스는 public subnet에 있다는게 특징이다.

이 인스턴스는 bastion host security group이 있고 물론 private subnet 내 instance도 serurity group이 있다.

이 baston host로 private subnet instance로 엑세스할 수 있다.

결국 외부의 유저가 이 private subnet instance에 접근하려면, ssh로 bastion host에 접근하고, 이 bastion host가 ssh로 private subnet instance에 접근한다.

시험에서는 보안규직 관련한 내요이 나오는데 무엇을 정의할 수 있는지 잘 이해해야한다.

bastion host를 위해서는 보안그룹이 반드시 인터넷 엑세스를 허용해야한다.
하지만 모든 인터넷을 허용하면 위험하므로, 기업의 퍼블릭 CIDR 엑세스만 허용하거나, 사용자의 인터넷 엑세스만 허용하는 등 이를 제한하는 것이 좋다.

프라이빗 서브넷의 인스턴스 보안그룹에서는 반드시 ssh 입력을 허용해야 한다. 따라서 포트 22번이 bastion host의 private IP가 되거나 bastion의 보안그룹이 되면 된다.

## NAT 인스턴스 (outdated)

NAT = Network Address Translation

NAT 인스턴스는 사실 subnet ec2 instance가 인터넷에 연결되도록 허용한다.
그리고 이 인스턴스는 반드시 퍼블릿 서브넷에서 실행되어야 한다.

EC2 setting에서 Source/destination Check는 꺼야 한다.

NAT instance에는 Elastic IP가 붙어야 한다.

Route Talbe을 수정해서 private subnet의 트레픽을 NAT instance로 전송하게 한다.

그리고 private 인스턴스 IP는 공용서버로 엑세스하는데 NAT instance를 거쳐야 한다.

이때 원래는 보내는 쪽이 private instance이고, 받는 쪽이 외부의 공용 서버인데 NAT istance를 거치면서
보내는 쪽이 NAT 받는 쪽은 외부 공용 서버가 된다.

이렇게 네트워크 패킷을 재정리하는 걸 NAT가 한다.

이렇게 트래픽을 보내면, 다시 공용서버는 source는 자신이고 destination은 NAT 인스턴스가 될 것이다.

NAT는 다시 이 네트워크 패킷을 수정해서 사설 서브넷 인스턴스로 보낸다.

어떤 인스턴스로 갈 지는 알고 있다.

이런 이유로 IP 소스/데스티네이션이 변경되기 때문에 NAT instance에서 source/destination check는 꺼야 한다.

NAT Instance - Comments

- 사전에 정의된 Linux AMI가 사용가능하다.

  - 사용할 수 있었지만 20.12.31에 표준 지원이 마감되어서 이제는 이 기술 말고 NAT Gateway 사용을 권장

- NAT instance는 가용성이 낮고 초기화 설정으로 복원할 수 없어서 AZ마다 ASG를 생성해야 하고 복원되는 사용자 데이터 스크립트가 필요하기 때문에 복잡하다

- 작은 인스턴스는 큰 인스턴스에 비해 대역폭이 작다

- 보안 그룹과 규칙을 관리해야 한다.

- Inbound

  - Allow HTTP/HTTPS traffic coming from Private Subnets
  - Allow SSH from your home network

- Outbound
  - Allow HTTP / HTTPS traffic to the Internet

## NAT Gateway

- AWS 관리형 NAT 인스턴스이며 높은 대역폭을 가지고 있다.
- 가용성이 높고 관리할 필요가 없다.
- 사용량 및 NAT Gateway 대역폭에 따라 청구된다.

- NAT Gateway는 특정 AZ에서 생성되고 Elastic IP를 사용한다.
- EC2 instance와 같은 subnet에서 사용할 수 없어서 다른 subnet에서 엑세스할 때만 NAT Gateway가 도움이 된다.

경로는 Private Subnet ----> NATGW ----> IGW

대역폭은 5GB/s 최대 45GB/s까지 확장 가능

따로 보안그룹은 관리할 필요없다 -> 연결을 위해 어떤 포트를 열 지 고민할 필요 없다.

NAT Gateway with High Availability

NAT Gateway는 단일 AZ에서 복원이 가능하지만, 해당 AZ가 고장날 상황을 대비하여 여러 AZ에 multiple NAT Gateway를 생성해서 재난 대비를 할 수 있다.

routing table을 통해서 다른 AZ를 서로 연결할 필요는 없다.
어차피 AZ가 먹통이 되면 그 안에 instance도 접근 불가 상태가 된다.

## NACL 및 보안 그룹

NACL : Network ACL

특징 : Stateless 이다.

NACL은 Subnet 밖에 있는 계층인데, 가령 예를 들어 외부에서 요청이 오면 우선 NACL의 inbound rule에 검증을 받는다. 그리고 이 검증 후에 subnet 안에 들어가서 Security group의 incbound 규칙에 검증을 받는다. 이렇게 검증을 받은 요청에 대한 답에 대해서는 SG의 경우 Stateful가 있어서 나가는 요청에 대해서는 outbound 검증을 하지 않는다. 하지만 NACL은 stateless 이기 때문에 그것과 상관없이 outbound 검증을 거쳐서 응답을 내보낸다.

Network Access Control List

- subnet을 오가는 트래픽을 제어하는 방화벽과 유사하다
- 한 subnet마다 하나의 NACL이 있고 새로운 서브넷에는 기본 NACL이 할당된다.
- NACL rule을 정의할 수 있는데

  - 1 ~ 32766, 숫자가 낮을수록 우선순위가 높다
  - First rule match will drive the decision
  - 예를 들어 100번이 해당 IP deny이고, 200번이 Accept라면 100번에 의해 거부된다.
  - (\*)rule - 가장 마지막 규칙이며, 모든걸 거부하는 내용이 있다. 즉 이 전까지 일치하는 규칙이 없으면 모든 요청을 거부한다.

- 새로 만든 NACL은 기본적으로 다 거부한다.

- NACL은 서브넷 수준에서 특정한 IP를 차단하는데 적합하다

Default NACL - 시험에서 정말 중요한 부분

- 기본 NACL은 서브넷과 연결되어 있고 inbound/outbound를 모두 허용하는 특수성을 가진다.
- 기본 NACL은 수정하지 말고, 필요하면 사용자 정의 NACL을 만들어라

* 시험에서 기본 NACL이 인스턴스와 연결되어 있다고 하면 모든 것이 허용된다고 하는 것이다

Emhemeral Ports(임시 포트)

클라이언트와 서버가 연결되면 포트를 사용해야 한다

클라이언트는 규정된 포트의 서버에 연결한다 (ssh - 22, http - 80 https 443)

근데 서버도 클라이언트에 정보를 보내려면 포트가 연결되어야 한다.
하지만 클라이언튼 기본적으로 개방된 포트가 없어서 클라이언트가 서버로 접속할 때 자체적으로 특정한 포트를 열게 된다.

이 포트는 임시라서 클라이언트와 서버간 연결이 유지되는동안만 열려있는다.

임시 포트는: OS에 따라 달라진다.

- IANA & MS Windows 10 : 49152 ~ 65535
- Many Linux Kernels : 32768 ~ 60999

client가 요청을 보낼 때, 위의 내용을 함께 실어서 보내게 되는데

Src IP, Src Port, Dest IP, Dest Port, Payload, ...

여기서 Src Port가 Ephemeral Ports이다
즉 응답을 위한 임시 포트이다.

NACL with Ephemeral Ports

자 VPC 내부에 2개의 subnet이 있는데, public subnet에 있는 instance에서 private subnet에 있는 DB instance로 데이터를 주고 받는다고 생각해보자.

요청을 할 떄 NACL을 거쳐야 하는데 클라이언트 쪽에서는 DB 쪽 port를 알고 있기 때문에 outbound에 해당 포트로 데이터를 보낼 수 있다. 그리고 DB 사이드도 응답을 받을 수 있다. 문제는 DB쪽에서 응답을 보낼 때 인데 클라이언트 쪽은 임시포트를 쓰기 때문에 어떤 포트가 열릴지 미리 알 수 없다. 따라서 DB 쪽 outbound rule에 port 범위 전체를 등록하는게 중요하다. 그리고 클라이언트 쪽도 inbound rule을 전체 port 범위로 잡아야지 응답을 제대로 받을 수 있다.

Create NACL rules for each target subnets CIDR

다중 NACL, subnet이 있다면 각 NACL 조합이 NACL 내에서 허용되어야 한다
CIDR이 서브넷마다 고유하기 때문이다

NACL에 subnet을 추가하면 NACL 규칙도 업데이트해서 연결조합이 가능하지 확인해야 한다.

## VPC peering

2개의 VPC간의 private connect를 할 때, AWS network를 상요한다.
왜그러냐면 VPC를 모두 같은 네트워크에서 동작하게 만들기 위해서 이다.

VPC가 다른 리전 혹은 다른 계정에 있어도 이를 연결하거나 연결하기 싫다면 VPC 네트워크 CIDR가 서로 멀리 떨어져야 한다. 연결할 때 CIDR가 겹치면 통신을 할 수 없기 때문이다.

VPC Peering connection is Not transitive
-> 즉 서로 다른 VPC끼리 통신하려면 VPC Peering을 활성화해야 한다.

당연히 각 VPC subnet의 route table을 업데이트하여 EC instance가 서로 통신할 수 있게 한다.

VPC Peering - Good to know

단일 계정에서 할 수 있짐나, 다른 계정 간에도 가능하다.
또는 리전 간에 연결도 가능하다

peeredVPC끼리 보안그룹을 참조할 수 있다. (단, cross-accounts - 같은 리전에서)

CIDR이나 IP를 source로 가질 필요없이 보안그룹을 참조할 수 있다.

## VPC Endpoints

DynamoDB같은 서비스를 이용하면, 퍼블릭 엑세스가 가능한데 바로 IGW나 NAT를 통해 엑세스 할 수 있다는 말이다.

하지만 이 모든 트래픽은 퍼블릭 인터넷에서 온다.

CloudWatch, S3는 인터넷을 거치지 않고, private access를 원할 수도 있다.

VPC Endpoint를 구성하면 public internet을 거치지 않고도 instance에 엑세스할 수 있다.

private AWS network만 거쳐서 해당 서비스로 엑세스할 수 있다.

퍼블릭 엑세스는 비용이 많이드는데 이유는 NAT를 거칠 때 비용이 발생하고, IGW는 비용이 들지는 않지만 허브가 여러개 있으니 효율적이지 않다.

VPC endpoint는 VPC 내 배포되고 private instance에서 VPC endpoint를 직접 거쳐 서비스에 연결된다.
네트워크가 AWS 내부에서만 이루어진다는 장점이 있다.

모든 AWS 서비스는 public에 노출되어 있고, public url을 가진다.

VPC endpoint를 사용하면 AWS PrivateLink를 통해 유용하다.

VPC endpoint는 중복과 수평 확장이 가능하다.

네트워크 인프라를 간단하게 만들어 줄 수 있다.

Types of Endpoints

1. Interface Endpoints (powered by PrivateLink)

   - ENI(VPC의 private IP Address)를 프로비저닝한다.
   - ENI가 있으므로 보안그룹을 연결해야 한다.
   - 대부분의 AWS 서비스 커버
   - 시간당 GB당 과금

2. Gateway Endpoints
   - 게이트웨이를 프로비저닝하고, 이 게이트위이는 반드시 라우팅 테이블의 대상이 되어야 한다.
   - 게이트웨이 엔트포인트 대상은 S3, DynamoDB 두 가지
   - 장점은 무료이고, 자동으로 확장된다.

그렇다면 S3에는 어떤 Endpoint를 써야 할 까

시험에서는 게이트웨이 엔드포인트를 고르는게 유리하다
왜냐면 라우팅 테이블만 수정하면 되기 때문이다.
무료기도 하다

인터페이스가 권장되는 경우는 온프레미스에서 엑세스가 필요한 경우이다.
혹은 다른 VPC에 연결하는 경우가 있다.

## VPC Flow 로그

- interface로 들어오는 IP traffic에서 정보를 포착할 수 있다.
- VPC, subnet, ENI(Elastic Network Interface) 수준에서 포착할 수 있다.

이 로그를 여러 로그 모니터링 서비스에 연결할 수 있다.

얻을 수 있는 정보

- srcaddr & dstaddr - 문제있는 IP를 구별하게 도와줌
- srcport & dstport - 마찬가지
- Action - accpet or deny - SG / NACL
- VPC flow log를 쿼리하는 방법은 S3에 Athena를 사용하거나 CloudWatch Logs Insight를 사용할 수 있다.

VPC Flow logs - Troubleshoot SG & NACL issues

Action 필드를 살펴 본다.

## 사이트 간 VPN, 가상사설 Gateway 및 고객 Gateway

AWS Site to Site VPN

AWS와 특정 외부 회사 데이터 센터와 비공개로 연결하는 법을 알아보자

고객은 Gateway를 AWS는 VPN Gateway를 갖춰야 한다.
공용 인터넷을 이용해 사설 VPN을 연결해야 한다.
VPN 연결이기 때문에 암호화 되어 있다.

1. VGW (Virtual Private Gateway)

   - VPN Connection에서 AWS 측에 있는 VPN Concentrator이다.
   - VGW는 생성되면 VPC에 연결된다.
   - ASN(Autonomous System Number)도 지정할 수 있다.

2. Customter Gateway (CGW)
   - 소프트웨어, 하드웨어 장비로 VPN 연결을 한다

Customer Gateway Device (On-premises)

- what IP address to use
- 고객이 사용 가능한 public IP가 있다면 그걸 사용하면 도니다.
- 고객이 private IP를 사용할 경우, NAT-T를 사용하여 NAT의 public IP를 연결하면 된다.

다음으로 시험에 잘나오는게 서브넷의 VPC에서 route propagation을 활성화해야 site-to-site VPN이 실제로 작동한다는 점이다.

온프레미스에서 AWS로 EC2 인스턴스 상태를 진단할 때, 보안 그룹 인바운드 ICMP 프로토콜이 활성화되었느닞 확인해야 한다. 그렇지 않으면 연결이 안된다.

AWSVPN CloudHub

클라우드 허브는 여러 VPN과 모든 사이트 간의 안전한 소통을 보장한다.

비용이 적게 드는 허브 엔 스포크 모델로 VPN만을 활용해 서로 다른 지역 사이 기본 및 보조 네트워크 연결성에 사용한다.

VPC 내 CGW와 VGW하나 사이에 site-to-site VPN을 생성하게 되는 것이다.
이렇게 연결되면 고객 네트워크끼리 VPN 연결을 통해 서로 소통할 수 있게 된다.

VPN 연결이므로 모든 트래픽이 공용 인터넷을 통한다.

사설 네트워크로는 연결되지 않는다.

VPC 하나에 site-to-site VPN 연결을 여러 개 만들어 Dynamic Routing을 활성화하고 Routing Table을 구성하면 된다.

## Direct Connect & Direct Connect Gateway

Driect Connect(DX)

원격 네트워크에서 VPC로 전용 private 연결을 제공한다.

이걸 사용할 때는 전용 연결을 생성해야 하고 AWS Direct Connect location을 사용한다.
VPC에는 VGW를 설치해야 온프레미스 데이터 센터와 AWS 간 연결이 가능하다.

use case : 큰 대역폭이 필요하거나 큰 데이터 세트를 처리할 때 속도가 빨라진다.
비용도 절약할 수 있다. 인터넷 문제에 구애받지 않는다.

하이브리드 환경 구축에 적합하다

구축을 하면, Direct Connection locaction에서 AWS로 VIF(Virtual interface)로 VGW로 연결되는데 S3, Glacier와 같은 서비스를 연결할 때는 public virtual interface를 생성하여 VGW를 거치지 않고 직접 연결된다.

다른 리전에 있는 하나 이상의 VPC와 온프레미스 서버와 연결하고 싶다면 Direct Connect Gateway를 써야 한다.

Direct Connect - Connection Type

1. Dedicated Connection : 1Gbps, 10 Gbps and 100 Gbps capacity

   - 고객은 물리적 전용 이더넷 포트를 할당받는다.

2. Hosted Connections : 50Mbps, 500 Mpbs, to 10 Gbps
   - 파트터를 통해 연결한다.
   - 용량은 추가하거나 제거하거나 요청할 수 있으니 Dedicated보다는 유연하다

새 연결을 만들려면 한 달 정도 걸릴 수 있다.

Direct Connect - Encryption

암호화 기능이 없다.
프라이빗 연결이므로 보안이 유지 된다.

AWS Driect Connect + VPN : IPsec으로 암호화된 private 연결이 가능하다

보안을 추가하면 좋지만 구현이 복잡하다

Direct Connect - Resiliency (복원력)

최대 복원력을 위해서는 각각 2개의 Direct Connect를 구현하면 된다

## Direct Connect + site to site VPN

시험에 나오는 아키텍쳐

## 환승 Gateway

network topologies는 복잡해보일 수 있다.

예를 들어 VPC 여러개를 peering하고 VPN을 구축하고 Direct Connect Gateway로 여러 VPC를 한 번에 연결하는 식 말이다.

이런 문제를 해결하기 위해 Transit Gateway를 만들었다.

For having transitive peering between thousands of VPC and on-premises, hub-and-spoke (star) connection

각종 다양한 주체들을 Transit Gateway를 중심으로 별 모양의 아키텍쳐를 구성한다.

VPC들이 서로 peering할 필요 없이 Transit Gateway를 통해 연결된다.

모든 VPC가 통신할 수 있다.

Direct Connect Gateway가 transit Gateway에 연결되면 각 VPC와 연결된다.

리전 리소스이며 리전 간에 작동한다.

cross-account는 Resource Access Manager를 통해 할 수 있다.

Transit Gateway에 Route Table을 생성해서 누가 통신할 지 정리할 수 있다.

모든 트레픽 경로를 제어하여 네트워크 보안을 유지한다.

IP Multicast를 지원한다.

Transit Gateway: Site-to-Site VPN ECMP

연결 대역폭을 ECMP를 사용해 늘릴 수 있다.

ECMP = Equal-cost multi-path routing

여러 최적 경로를 통해 패킷을 전달하는 라우팅 전략이다.

## VPC 트래픽 미러링

보안 기능

VPC에서 네트워크 트래픽을 수집하고 검사하되 방해되지 않는 방식으로 실행하는 기능

Route the traffic to securiry appliance that you manage

Capture the traffic

- From(source) - ENIs
- To(Targets) - an ENI or a Network Load Balancer

Capture all packets or capture the packets of your interest (optionally, tuncate packtes)

## VPC용 IPv6

주소가 부족해서 만들어 졌다.

IPv6는 모두 공용이고 사설 용은 없다.
형식은 x.x.x.x.x.x.x.x이다.

IPv6 in VPC

VPC와 subnet에서 IPv4 비활성은 불가능하지만 IPv6를 활성화할 수는 있다.

듀얼 스택 모드로 동작한다.

IPv6 Troubleshooting

if you cannot launch an EC2 instance in your subnet

: create a new IPv4 CIDR in your subnet

## 이그레스 전용 인터넷 Gateway

Egress-only Internet Gateway (송신 전용 인터넷 게이트웨이)

used for IPv6 only

VPC instance에서 IPv6상 아웃바운드 연결을 허용하고 동시에 인터넷이 인스턴스로 IPv6연결을 시작하지 못하게 막는다.

route table을 업데이트해야 한다. -> 게이트웨이를 다르게 설정해서 구분

## 세션 정리

## VPC 세션 요약

## AWS 네트워킹 비용

리전 하나에 가용영역이 2개 있는데 EC2로 들어오는 트래픽은 무료다.

같은 AZ내 인스턴스간의 통신은 무료이다.

같은 리전 다른 AZ에 있는 인스턴스끼리 통신을 위해서는 Public IP/Elastic IP가 필요하고 GB 2센트이다.
대신 Private IP 사용시 GB 당 1센트이다.
핵심은 공용 IP보다는 사설 IP를 사용하라는 것이다.

트래픽이 다른 리전으로 이동할 때, GB 당 0.02달러가 청구된다.

Egress Traffic은 송신 트래픽이다.

Ingress Traffic은 수신 트래픽이다. (일반적으로 무료)

인터넷 트래픽을 최대한 AWS 내부에 두어서 비용을 최소화할 수 있다.
최대한 출력되는 트래픽은 최소화하도록 아키텍쳐를 구성한다.

데이터 베이스 인스턴스와 연산 인스턴스를 같은 가용영역에 두는 식으로 처리

Direct Connect를 사용하면 비용을 줄일 수도 있다.

S3 ingress: free
S3 to internet : GB 당 0.09달러
S3 Transfer Accleration

- 전송속도가 50 ~ 500% 더 빨라진다.
- Data Transfer fee + GB 당 0.04 ~ 0.08 달러 추가

S3 to CloudFront : 무료

CloudFront to Internet : GB 당 0.085달러 (S3보다 살짝 저렴)

- 캐싱 기능 추가
- 7배나 저렴해진다.

S3 Cross Region Replication : GB 당 0.02달러

NAT Gateway vs Gateway VPC Endpoint

Private instance ---- NAT Gateway ---- IGW ---- Internet ---- S3 bucket

- NAT price : 시간 당 0.045달러 / GB 당 0.045달러 / S3로 리전간 데이터 송신 비용 시간당 0.09달러 (동일 리전이면 비용발생 X)

Private instance ---- VPC endpoint ---- S3

- VPC Endpoint price : 무료 / GB당 0.01달러 Data transfer in/out (same region)

## AWS Network Firewall

우리가 본 네트워크 보호 방법

- NACLs
- VPC security groups
- AWS WAF (protect against malicious requests)
- AWS Shield & AWS Shield Advanced
- AWS Firewall Manager

전체 VPC를 보호할 수 있는 수준 높은 방법은?

AWS Network FireWall

전체 VPC를 방화벽으로 보호하는 서비스
계층 3 ~ 7까지 보호
모든 방향에서 들어오는 모든 트래픽을 감시

- VPC to VPC
- Outbound / inbound internet
- To / From Direct Connect & Site to Site VPN

내부적으로 Network Firewall은 AWS Gateway Load Balancer를 사용한다.

VPC 레벨에서 수천개의 룰을 지원한다

Traffic filtering: Allow, drop or alert for the traffic that matched the rules

Active flow inspection : 침입 방지 능력
