# EC2 Instance Storage

## EBS Volumn

EBS(Elastic Block Store) : 인스턴스가 실행 중인 동안 연결 가능한 네트워크 드라이브이다.

인스턴스가 종료되도 데이터를 유지할 수 있다.

한 번에 하나의 인스턴스에 연결할 수 있다. (CCP 레벨, 어소시에트 레벨에서는 일부 EBS가 다중 연결이 가능하다)
하나의 인스턴스에 여러개의 EBS 연결은 가능하다.

그리고 EBS는 특정 AZ에서만 연결이 가능하다. (예를 들어 us-east-1a에서 생성된 경우 us-east-1b에서는 연결이 불가능하다)
-> 단 스냅샷 기능을 이용하면 볼륨을 옮길 수 있다.

네트워크 ucb 스틱이라고 생각하면 편하다.
물리적인 연결은 없지만 네트워크로 연결이 가능하다.

인스턴스와 EBS가 통신하기 위해서는 네트워크가 필요하다.
약간의 지연이 있을 수도 있다.

볼륨이기 때문에 미리 용량을 정해야 한다. IOPS(단위 초당 전송 수)도 또한 미리 정의해야 한다.

이후에 더 용량이 필요하면 늘릴 수도 있다.

EBS 볼륨을 연결하지 않고 생성만하고 둘 수도 있다.

EBS는 종료 시 삭제라는 기능이 있다.
보통은 인스턴스 생성 시에 루트 EBS 볼륨은 해당 기능이 체크되어 있고, 보조 EBS는 체크되어 있지 않다.
즉 루트 EBS는 인스턴스 종료 시 같이 삭제되도록 설정되어 있는 것이다.

## EBS Snapshot 개요

EBS 볼륨의 특정 시점의 백업이다.
스내샷은 필수는 아니지만 권고한다.
이건 다른 AZ나 리전에 복사할 수 있다.

기능

- EBS Snapshot Archive : 스냅샷을 아카이브 티어로 옮기면 75% 저렴하다. 단 아카이브에서 복원 시에는 24 ~ 72시간이 소요된다.

- Recycle Bin for EBS Snapshots : 영구 삭제하는 대신에 휴지통에 넣을 수 있다. 실수로 삭제하는 경우, 복원 할 수 있고 보관 기간은 1일 ~ 1년이다.

- Fast Snapshot Restore(FSR) : 강제로 스냅샷을 초기화해서 첫 사용에 지연시간을 없애는 기능이다. 하지만 비용이 많이 든다.

## AMI 개요

AMI - Amazon Machine Image

- 운영체제를 설치할 수도 있고 모니터링 툴을 설치할 수도 있고 부팅 시간을 줄일 수도 있다. 모든 소프트웨어를 패키징할 수 있다.

우리는 보통 public AMI를 사용했다.

하지만 우리는 우리 스스로 우리만의 AMI를 만들 수 있다. 하지만 그럴 경우, 관리해야 한다.

그리고 AMI marketplace에 올려서 판매할 수도 있다.

AMI Process

1. EC2를 시작하고 커스텀한다.
2. Data integrity를 위해 EC2를 중단한다.
3. AMI를 생성한다. (EBS 스냅샷도 생성한다.)
4. 다른 AZ에서 AMI로 부터 인스턴스를 시작한다.

## EC2 Instance Store

EBS는 네트워크 드라이브로 좋지만 제한된 성능을 가지고 있다.
만약에 당신이 고성능의 HDD를 필요로 한다면, EC2 Instance Store를 사용해야 한다.

EC2는 가상서버지만 물리적인 하드웨어가 존재하고 그 하드웨어에는 물리적인 디스크 공간이 연결되어 있다.

I/O 성능때문에 사용할 수 있다. 대역폭이 크기 대문이다.
문제는 인스턴스 종료 시 해당 스토리지가 손상된다.

buffer/cache/scratch data/ temporary content를 위해서 사용할 때 적합하다.

장기 보관에는 적합하지 않다.
장기 보관에는 EBS가 적합하다.

필요에 따라서 데이터는 백업하는게 좋다.

## EBS Volume Types

- gp2/gp3 (General Purpose 2/3) (SSD) : 가격과 성능이 균형잡힌 다양한 용도로 사용 가능한 타입

짧은 지연시간, 비용 저렴
시스템 부트 볼륨, 가상 데스크탑, 개발 및 테스트 환경에 적합
1GB ~ 16TB

gp3 : 최신 세대 볼륨, 기본적으로 3000 iops, 125MB/s
독립적으로는 iops를 16,000까지 그리고 throughput을 1000MB/s까지 증가시킬 수 있다.

gp2 : 오래된 버전, IOPS와 볼륨이 서로 연결되어 있다. (gp3와 다르게), GB 당 3 iops, 즉 5,334 GB라면, 16,000을 초과하게 된다.

IOPS 성능이 유지되어야 하는 비지니스 모델
혹은 16,000 이상의 IOPS가 필요한 비지니스 모델인 경우에는 io1/io2를 적용하는 것이 적합하다.

- io1/io2 (Highest Performance SSD) : 임무 수행에 있어 저 지연과 고 대역폭을 제공

4 GB ~ 16 TB

최대 PIOPS : 64,000 for Nitro EC2 instances & 32,000 for other

gp3처럼 스토리지 크기와 독자적으로 IOPS를 늘릴 수 있다.

io2의 장점은 동일한 비용 대비해서 io1보다 많은 GB 당 IOPS를 사용할 수 있다. 현재는 io2를 사용하는 것이 더 합리적이다.

io2 Block Express(4GB ~ 64TB) : 좀 더 고사양의 불륨이다. 수 밀리초 이하의 지연을 가지고 있으며, IOPS 대 GB 비율이 1000: 1일 때, 최대 PIOPS가 256,000이다.

아래 두 개의 HDD는 부트 볼륨으로 사용할 수 없다.
125MB ~ 16TB 사용 가능

- St1 (HDD) : 저비용 HDD로 빈번하게 접근하고 대역폭이 강화된 타입

ThroughPut 최적화 HDD이며, Big Data, Data WareHouse, Log Processing등에 적합

500MB/s의 최대 Throughput과 최대 500 IOPS를 가진다.

- sc1 (HDD) : 최저비용 HDD로 저빈도 접근에 적합

자주 데이터에 접근하지 않는 어플리케이션이 적합하다.
250MB/s의 최대 Throughput과 최대 250 IOPS를 가진다.

Size, Throughput, IOPS를 고려하여 타입을 정하면 된다..

오직 gp2,3와 io1/2만이 부트 볼륨에 사용이 가능하다.

## EBS 다중 연결

같은 가용 영역에 있는 여러 인스턴스에 해당 볼륨을 동시에 연결할 수 있다.
단, io1, io2 타입에만 가능하고, 읽고, 쓰는 권한을 가질 수 잇다.

높은 앱 가용성을 위해서 사용하고, 한 번에 16개의 인스턴스만 연결 할 수 있다.

그리고 반드시 cluster-aware 파일 시스템을 사용해야 한다. (XFS, EX4는 안된다.)

## EBS 암호화

EBS 볼륨을 복사하면, 볼륨 내부가 암호화되고, 인스턴스와 볼륨 간의 데이터 전송도 암호화 된다.

암호화와 복호화는 백그라운드에서 수행되어 따로 내가 할 게 없다.
그리고 암호화는 Latency에 최소한의 영향을 준다.
EBS는 KMS(AES-256)의 키를 사용한다.

1. EBS 스냅샷을 생성
2. EBS 스냅샷을 암호화 (복사를 사용)
3. 스냅샷을 통해 새로운 EBS 볼륨 생성 (이 볼륨은 암호화 됨)
4. 이제 암호화된 볼륨을 인스턴스에 연결

## Amazon EFS - Elastic File System 강의

Managed NFS (network file system) 여러 EC2에 마운트될 수 있다.
여러 AZ 내의 EC2 인스턴스와 함께 작업할 수 있다.
고사용성, 확장성이 좋다.
단 가격이 비싼데 gp2 EBS의 3배이다.

사용량에 따라 비용을 낸다.

- use case : Content management, web serving, data sharing, wordpress
- NFSv4.1 프로토콜 사용
- EFS에 제어 접근을 위해서는 보안그룹을 사용해야 한다.
- 리눅스 기반 AMI에만 호환이 된다. (윈도우 사용불가)
- KMS로 암호화 함

용량을 미리 정할 필요 없다.

- 수천 개의 NFS 클라이언트과 10GB/s 처리량 확보 가능
- PB 규모의 네트워크 파일 시스템으로 확장 가능
- 성능 모드 설정 가능 (범용, 최대 I/O)
- 처리량 모드

  - Bursting : 1 TB = 50 MB/s to 100MB/s
  - Provisioned : set your throughput regardless of storage size ex: 1 GB/s for 1TB storage
  - Elastic : automatically scales throughput up or down based on your workloads
    - Up to 3GB/s for reads and 1GB/s for writes
    - Used for unpredictable workloads

- Storage Classes
  : Storage Tiers (lifecycle management feature - move file after N days) - Standard : for frequently accessed files - Infrequent access (EFS-IA) : cost to retrieve files, lower price to store. Enable EFS-IA with a Lifecycle policy

EFS-IA를 사용하면, 저장 하는데 비용은 저렴하지만, 읽기 시 비용이 더 비싸다.

- Availability and durability
  - Standard : Multi-AZ. great for production
  - One Zone : One AZ. great for develop. backup enabled by default. compatible with IA (EFS One Zone-IA) - 할인이 크다 최대 90%

## EFS vs EBS

EBS
EBS는 한 번에 하나의 인스턴스에 연결된다. (io1, io2 다중 연결 제외)
AZ에 종속적이다.
gp2 : 디스크 사이즈가 커질 수록 io도 커진다.
io1 : io를 독립적으로 증가시킬 수 있다.

EBS를 다른 AZ로 옮기려면, 스냅샷을 이용해야 한다.

EC2 instance는 인스턴스 종료 시 사라질 수 있다. (기능을 끌 수 있음)

EFS
여러 AZ에서 수백개의 인스턴스에 연결 가능
리눅스 인스터스에서 사용 가능 (POSIX)

EFS는 EBS에 비해 비용이 비싸지만, EFS-IA를 사용하면 비용을 줄일 수 있다.
