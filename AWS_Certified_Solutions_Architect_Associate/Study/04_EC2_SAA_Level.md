# EC2 Solution Architect level 

## 프라이빗 vs 퍼블릭 vs 탄력적 IP

네트워크는 두 종류의 IP가 있다. 
- IP v4, IP v6

일반적으로 IP v4를 주로 사용하고, IP v6의 경우에는 IoT 기기에 주로 사용하곤 한다. 

IP v4는 Public Space에서 37억 개의 서로 다른 주소를 지원하는데, 이 주소가 점점 고갈되고 있다. 

Private IP의 개념은 예를 들어 특정 회사에서 사용하는 트래픽은 전부 하나의 Internet Gateway를 통해 외부의 서버와 통신하겠금 만드는 것이다. 사설 네트워크 내부 컴퓨터들은 외부와 모두 통신할 수 있고, 그것은 Internet Gateway를 통해서 접근하는 것임, 사설 네트워크 IP는 사설 네트워크 내부에서만 식별이 가능하다.

Elastic IP 

우리가 EC2 instance를 종료하고 시작할 때, public IP를 변경할 수 있게 하는 것이다. 만약에 public IP 고정이 필요하다면, Elastic IP를 사용하면 된다. 

Elastic IP는 종종 문제가 생길 경우, 해당 주소를 다른 instance로 재 맵핑할 수도 있다. 하지만 AWS에서는 한 계정 당 Elastic IP를 5개만 허용한다. (문의 시 늘릴 수 있음)

하지만 Elastic IP 사용은 피하는 것이 좋다. 
왜냐면 이것은 구조적으로 좋지 못한 설계의 예로 사용되곤 하기 때문이다. 
대신에 임의의 public IP를 사용하여 DNS 이름을 할당하는 것이 좋다. 

정리하면, 기본적으로 EC2는 AWS 내부에서는 사설 IP를 사용하고, 외부망과는 공용 IP를 사용한다. 

SSH로 머신에 접근할 때는 공용 IP로 접근해야 한다.

또한 머신을 중단하고 재 시작할 때, 공용 IP는 변경될 수 있다. 

## EC2 배치 그룹

우리는 EC2 instance를 어디에 배치할 지에 대한 전략을 세우고 싶을때가 있다. 

이러한 배치 전략은 placement groups을 사용하여 정의한다. 

- Cluster : clusters instances into a low-latency group in a single Availability Zone
: 모든 인스턴스가 같은 가용영역 내에 같은 렉에 있다. 
장점은 네트워크 속도인데, 10Gbps의 네트워크 속도를 낼 수 있다. 
단점은 렉에 문제가 발생하면, 모든 인스턴스가 동시에 문제가 발생할 수 있다.

이런 구조는 네트워크가 매우 빨라야 하는 빅데이터 작업이나 고속도 앱에 사용한다. 

- Spread : spreads instances across underlying hardware(max 7 instances per group per AZ) - critical applicatoins
: 실패의 위험을 최소화한다. 모든 인스턴스가 다른 하드웨어에 적용된다. 장점은 여러 가용영역에 걸쳐 있어 동시 실패 위험이 줄어든다. 단점은 배치 그룹 당 가용영역에 7개의 인스턴스 사용이라는 제한이 있다. 

- Partition : spreads instances across many different partitions (which rely on different sets of racks) within an AZ. Scales to 100s of EC2 instances per group(Hadoop, Cassandra, Kafka)
: 가용역역 당 7개의 파티션을 사용할 수 있다. 파티션은 하드웨어 렉을 의미하며 이 안에 인스턴스는 여러개를 사용할 수 있다. 
같은 지역의 여러 AZ에 걸쳐서 적용할 수 있으며, 파티션에는 100개까지의 인스턴스를 생성할 수 있다. 파티션이 다른 인스턴스끼리는 하드웨어 렉을 공유하지 않는다. 

사용 예는 HDFS, HBase, Cassandra, Kafka 등이 되겠다.

## ENI(Elastic Network Interface) - Basic

- VPC의 논리적 구성 요소이며, 가상 네트워크 카드를 나타낸다. 
ENI는 EC2 instance가 네트워크에 엑세스할 수 있게 해준다.

ENI는 인스턴스 외부에서도 사용된다.

AZ에 하나의 EC2가 있다고 하자, Primary ENI는 Eth0에 붙어서 EC2에 네트워크 연결을 제공한다. - private IP

ENI는 다음과 같은 속성을 가질 수 있다. 

1. Primary private IPv4, 하나 이상의 secondary IPv4를 가질 수 있다.

2. 하나의 ENI는 private IPv4 당 하나의 Elastic IP를 갖거나, 하나의 Public IPv4를 가질 수 있다.
-> 사설 및 공용 IP가 한 개씩 제공다.

3. ENI에 하나 이상의 보안 그룹을 연결할 수 있다. 

4. 맥 주소

ENI를 독립적으로 생성할 수 있고, 장애 조치를 위해서 EC2 instance에서 이동시킬 수도 있다.

ENI는 AZ에 바인딩 된다.

동일 AZ 내에서 인스턴스 1에 장애가 발생했을 때, 거기에 연결되어 있던 ENI를 같은 AZ 내에 다른 인스턴스로 옮겨올 수 있다. 즉, 장애 조치에 매우 유용하다. 

## EC2 Hibernate(절전모드)

EC2는 Stop, Terminate 할 수 있다.
- Stop : EBS는 다음 시작 때까지 데이터를 유지하고 있는다. 
- Teminate : EBS 설정을 삭제되게 있다면, 데이터를 잃게 된다. 하지만 그렇게 설정하지 않으면 살아있게 된다.

다시 Start하게 되면,
- OS 부트를 하고, EC2 User Data Script가 실행된다.
- OS 부트하면서, 앱도 실행되고 캐시도 구성된다.


Hibernate mode에서는
- in-memory(RAM)이 그대로 보존된다.
- 인스턴스 부팅이 빨라진다. (OS를 멈추거나 재시작하지 않았으므로)
- 절전모드 하에서는 RAM 상태가 루트 EBS 볼륨에 쓰여딘다. 
- 그리고 루트 EBS 볼륨은 암호화되어야 한다.

오랫동안 프로세스를 중지 하지 않을 때, 램 정보를 우지학 싶을 때, 서비스 시작 속도를 빠르게 하고 싶을 때

랩 사이즈는 150GB보다 작아야 한다. 

EBS 볼륨을 써야 하며 암호화되어야 하고, instance storage는 사용할 수 없으며 RAM 정보를 담을만큼 사이즈가 커야 한다. 

On-Demand, Reserved, Spot과 같은 인스턴스에 적용가능하다. 

절전모드는 최대 60일까지 사용할 수 있다. 

