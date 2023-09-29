bb# EC2 Basic

## AWS 예산 설정

Billing & Cost Management에 접근하려면 루트 계정으로 접근해야 한다.
IAM 사용자가 접근하게 하려면 루트 계정으로 설정을 활성화해줘야 한다.

Billing Dashboard에서 청구 내역을 확인할 수 있다.
Bills 에 가면 총액과 서비스 별 금액을 확인할 수 있다.

Free Tier tab에 가면 무료로 사용하는 서비스의 한도가 얼마나 남았는지 알 수 있다.

Budgets 가면, 여기서 예산을 생성해서 한도 도달에 대한 알람을 받을 수 잇따.
Zero Spend Budget Alarm은 무료 서비스가 끝나는 걸 알려주는 알람이다.

이러한 예산은 여러 개를 생성할 수 있다.

## EC2 Basic

EC2 : Elasic Compute Cloud - Infrastructure as a Service

EC2, EBS, ELB, ASG -> 이 기능들이 포함되어 있다.

운영체제 : 리눅스, 윈도우, 맥 OS
컴퓨터 성능과 코어 양도 선택 가능하고 RAM 사이즈, 용량도 설정 가능하다.

EBS & EFS 선택 가능

어떤 네트워크를 연결할 지 설정 가능 - 속도, IP

방화벽 규칙

부트스트랩 스크립트

</br>

EC2 User Data
부트스크랩 : 인스턴스가 부팅할 때 한 번 실행되는 스크립트

업데이트, sw 설치, 인터넷에 다운로드 등등....

모든 명령은 sudo 명령으로 처리해야 한다. (루트 계정에서 실행해야 하므로)

instance type은 각 컴퓨터의 성능을 고르는 선택지이다.

## EC2 인스턴스 유형 기본 사항

다양한 사례에 적용 가능한 여러 타입이 있다.
총 7가지 유형이 있다.

General Purpose, Compute Optimized, Memory Optimiezed....

m5.2xlarge

m : instance class  
5 : generation
2xlarge : 인스턴스 클래스 내의 사이즈 / 크기가 클수록 더 많은 메모리와 CPU를 가질 수 있다.

General Purpose
: 범용 인스턴스, 밸런스가 잡혀있다.

Compute Optimized
: 고성능 프로세서 탑재, 미디어 변화, HPC 작업, 머신러닝, 전용 게임 서버 등등...
C로 시작

Memory Optimized
: 대규모 데이터셋을 빠르게 처리, 메모리는 RAM을 뜻한다. 비정형 데이터 처리 등등...
R로 시작 , X1, Z1

Storage Optimized
: 로컬 스토리지에서 대규모 데이터에 접근할 때 필요. NoSQL 시스템, Redis 문산 파일 시스템 등에 적용
I, G, H로 시작...

## 보안 그룹 및 클래식 포트 개요

Security Groups
: 네트워크 보안을 실행하는데 기반이다.
EC2 드나드는 트래픽 제어

allow rules. IP 참조

범용 인터넷을 통해 EC2 접근할 때, 보안 그룹이 그 주변에 방화벽을 세우는 것이다.

인바운드 트래픽 : 들어오는 트래픽 제어
아웃바운드 트래픽 : 나가는 트래픽 제어

Port 제어, IP 범위 확인. 아웃바운드, 인바운드 네트워크 확인

인바운드 규칙에 맞지 않으면 외부에서 접근 시 타임아웃이 걸린다.

아웃바운드 규칙에 맞지 않는 외부에 접근 하면 접근이 불가능하다.
하나의 인스턴스에 여러 개의 보안 그룹을 설정 할 수 있다.

EC2 밖에 존재하는 것이며 EC2인스턴스는 볼 수 없다.

SSH 접근에 대한 보안 그룹을 따로 구성하는 것은 좋다.

기본적으로 인바운드는 모두 차단되어있고, 아웃바운드는 허용되어 있다.

</br>
</br>

Referencing other security groups Diagrams

예를 들어 EC2 intance에 Inbound Security Group1,2를 적용한다면,
해당 Security Group을 가지고 있는 EC2 instance와는 서로 데이터를 주고 받을 수 있다.
하지만 그게 아닌 다른 Security Group이 할당된 instance의 경우에는 데이터를 주고 받을 수 없다.

알아야 할 포트

- SSH(Secure Shell) 22 = log into a Linux instance
- FTP(File Transfer Protocol) 21 = upload files into a file share
- SFTP(Secure File Transfer Protocol) 22 = upload files using SSH
  -> SSH를 사용하기 때문에 22번 포트를 사용한다.
- HTTP 80 = access unsecured websites
- HTTPS 443 = access secured websites
- RDP (Remote Desktop Protocol) 2289 = log into a Windows instance

## SSH 개요

SSH - cli : Mac, Linux, Windows 10 above

Windows 10 below : putty 사용 , 10 이상도 사용 가능

EC2 instance connect : 모든 OS에서 사용 가능하다. 단, instance가 Amazon Linux2여야만 한다.
SSH troubleshoooting : 강의를 다시 보자~ 아니면 EC2 instance connect를 사용하자 ~

## Linux 또는 Mac을 사용하여 SSH 실행하기

SSH - 터미널이나 cmd로 클라우드 리눅스를 제어한다.

이미 생성해놓은 instance의 보안 그룹에는 22 port가 TCP로 허용되어 있어야한다.
그 다음에 호스트 PC에서 터미널을 열고

```bash
ssh ec2-user@{intance IP address}
```

ssh ec2-user라고 하는 이유는 Amazon Linux2 AMI에는 이미 사용자 하나가 설정되어 있다.
그 사용자 이름이 ec2-user이기 때문이다.

이렇게 입력하면,

```bash
Too many authentication failures
```

위와 같이 나오는데 그 이유는 우리가 ec2-user에 인증하지 않았기 때문이다.

전에 다운로드한 키를 아직 올리지 않았기 때문이다.
pem file을 우리 명령 안으로 참조해야 한다.
이때 파일 이름에 공백이 있어서는 안된다.

```bash
ssh -i {pem file name.pem} ec2-user@{intance IP address}
```

위와 같이 pem file을 넣어준다.

위와 같이 입력하면 다른 종류의 오류가 나온다.

```bash
Permissions 0644 for '{pem file name.pem}' are too open.
```

보호되지 않은 키 파일이 있다고 한다.
권한을 변경해줘야 한다.

```bash
chmod 0400 {pem file name.pem}
ssh -i {pem file name.pem} ec2-user@{intance IP address}
```

이렇게 하면 이제 instance에 로그인하게 된다.
이제 터미널 앞에는 instance의 ip가 표기된다.

## EC2 instance 연결

EC2 instance connect - 더 쉬운 연결 방법, 브라우저 기반으로 SSH 연결을 할 수 있다.
이 방법은 SSH 키를 관리할 필요도 없다.

이 경우에도 22 포트가 열려있는지 확인을 해야 한다.

## EC2 instance Role

IAM access key를 넣지 맗자.
예를 들어 instance에 iam 명령을 수행하면, aws configure로 access key를 입력하겠냐고 물어볼 수 있다.
이걸 넣으면 IAM에 대한 모든 서비스를 접근할 수도 있다. 따라서 매우 위험하다.

대신에 instance에 IAM Role을 넣고 Read 권한만 넣어주면, IAM에 대한 응답을 받을 수 있다.

## EC2 instance start

- On-Demand Instances - short workload, predictable pricing, pay by second
- Reserved (1 & 3 years)
  - Reserved Instances - long workloads
  - Convertible Reserved Instances - long workloads with flexible instances
- Savings plans(1 & 3 years) - commitment to an amount of usage, long workload
- Spot Instances - short workloads, cheap, can lose instances(less reliable)
- Dedicated Hosts - book an entire physical server, control instance placement
- Dedicated Instances - no other customers will share your hardware
- Capacity Reservations - reserve capacity in a specific AZ for any duration

## 스팟 인스턴스 및 스팟 집합

온디맨드에 비해 최대 90% 절감할 수 있다.

스팟 요금이 최대 요금을 넘어가면, 인스턴스를 중단할 수도 있고 아니면 종료할 수도 있다.
이 옵션을 선택할 수 있는 시간은 2분이다.

스팟 인스턴스가 차단하는걸 스팟 블록이라고 한다. 보통은 1~6시간 내에 블록된다. 이를 해제하면 블록되지 않는다.

스팟 인스턴스 요청은 1회성, 영구 요청이 있다.
영구 요청의 경우에는 어떤 경우에 스팟 인스턴스가 종료 혹은 중단되었다면 유효성이 확보되었을 때 다시 시작할 수 있음을 의미한다.

만약에 이제 멈추고 싶다면, 반드시 스팟 요청부터 종료하고, 수행 중인 인스턴스를 종료해야한다.
그렇지 않고 인스턴스부터 종료한다면, 스팟 요청으로 인해 곧바로 다른 스팟 인스턴스가 수행될 수 있다.

Spot Fleets = set of Spot Instances + (optional) On-Demand Instances

The Spot Fleet will try to meet the target capacity with price constraints

- Define possilbe launch pools: instance type, OS, AZ
- Can have multiple launach pools, so that the fleet can choose
- Spot Fleet stops launching instances when reaching capacity or max cost

Strategies to allocate Spot Instances:

- lowest price : from the pool with the lowest price (cost optimization, short workload)
- diversified : distributed across all pools (great for availability, long workloads)
- capacityOptimizaed : pool with the optimal capacity for the number of instances
- price Capacity Optimized : pools with highest capacity available, then select the pool with the lowest price
