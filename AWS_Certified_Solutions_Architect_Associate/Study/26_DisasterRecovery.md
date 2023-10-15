# 재해 복구 및 마이그레이션

## AWS의 재해 복구

재해복구는 중요하다.

Any event that has a negative impact on a company's business continuity or finances is a disaster

Disaster Recovery(DR)은 이러한 재해를 준비하고 복구하는 작업을 의미한다.

What kind of disaster recovery

- On-premise -> On-premise : traditional DR, and very expensive
- On-premise -> AWS Cloud : hybrid recovery
- AWS Cloud Region A -> AWS Cloud Region B

RPO : Recovery Point Objective - 복구 시점 목표
RTO : Recovery Time Objective - 복구 시간 목표

RPO

- 얼마나 자주 백업을 할지?
- 시간상 어느 정도 과거로 되돌릴 수 있는지를 결정한다.
- Disaster - PRO = Data loss
- 재해가 발생했을 때 데이터 손실을 얾마나 감수할 수 있는지를 정한다.

RTO

- 재해 발생 후 복구할 때 사용
- RTO - Disaster = Downtime

RPO, RTO 최적화는 중요하며, 시간이 짧을 수록 비싸다

Disaster Recovery Strategies

1. Backup and Restore

   - High RPO
   - 저장하는데 시간이 오래걸린다.
   - 데이터 복구도 시간이 오래걸린다.
   - 값이 저렴하다

2. Pliot Light

   - 앱의 축소 버전이 클라우드에서 항상 실행되고,
   - 보통 크리티걸 코어가 있는데 이걸 플라이트 라이트라고 부른다.
   - 백업 엔 리스토어랑 유사하지만 크리티걸 시스템이 작동하기 때문에 복구할 때 여타 시스템만 더해 주면 된다.
   - 주요 데이터는 클라우드에 계속 복사해둔다.
   - Route 53으로 온프레미스와 클라우드 양 쪽을 라우팅 할 수 있다.

3. Warm Standby

   - 시스템 전체를 실행하되 최소한 규모로 가동해서 대기하는 방법이다.
   - 재해가 발생하면 프로덕션 Load로 활장할 수 있다.
   - 즉, 위기 상황용 시스템을 아주 저전력으로 동작시키고 있다가 위기 발생 시, 부하를 늘리면서 메인 역할을 하게 한다

4. Hot site / Multi Site Approach
   - RTO, RPO가 몇 초정도로 엄청 짧다.
   - On premise와 AWS 양 쪽을 Full Production Scale로 돌린다.
   - 비용이 정말 많이 든다.

어떤 전략을 쓸 지는 고객이 결정해야 한다.

Disaster Recoery Tips

슬라이드 한 번 읽어보기

## 데이터베이스 마이그레이션 서비스(DMS)

온프레미스에서 클라우드로 옮길 때 사용하는 서비스
마이그레이션을 해도 Source 데이터베이스를 계속 사용할 수 있다.

동종 마이그레이션도 되고 이종 마이그레이션도 된다.

CDC(Continuous Data Replication): 옮기고 나서도 지속적인 데이터 복제가 가능하다.
DMS를 수행하려면 그 일을 할 instance를 생성해야하고, 그 인스턴스가 DMS를 수행할 것이다.

DMS Source and Targets

슬라이드를 한 번 볼 것

AWS Schema Conversion Tool (SCT)

소스 디비와 타겟 디비의 엔진이 다른 경우
AWS SCT를 사용하여 데이터베이스 스키마를 변경해야한다.

만약에 같은 엔진이라면 SCT를 사용할 필요가 없다.

DMS - Continuous Replication

외부에는 오라클, 내부에는 RDS라고 하면 SCT를 사용해야 한다.
AST를 외부 서버에 설치하고 스키마 변환을 한다.

DMS 인스턴스를 VPC 내부에 생성하고 마이그레이션을 수행한다.

AWS DMS - Multi-AZ Deployment

다중 AZ를 활성화하면, AZ에 DMS 복제 인스턴스가 있고 그 인스턴스를 다른 AZ에 동기화 복제하게 된다.

## RDS와 Aurora Migrations

RDS -> 오로라

1. RDS 스냅샷 생성 후, 스냅샷을 오로라 디비에서 복원

   - 다운타임 발생

2. 오로라 읽기 복제본을 RDS에서 생성하고, 완료되면 해당 복제본을 승격시킨다.
   - 시간이 많이걸리다.
   - 비싸다

DMS 사용하는 방법도 있다.

## AWS를 통한 온프레미스 전략

- 시험에 자주나온다.

AWS AMI 특히 리눅스의 경우에는 ISO로 다운받아서 VM으로 돌릴 수 있다.

따라서 VM Import/Export도 하나의 전략이 될 수 있다.

AWS Application Discovery Service

- 온프레미스의 정보를 모아주고 마이그레이션을 계획할 수 있게 해주는 서비스이다.
- 서버의 사용량 정보와 종속성 매핑에 대한 정보 제공
- 대량의 마이그레이션을 할 때 유용

AWS Migration Hub

AWS Database Migration Service(DMS)

AWS Server Migration Service

## AWS 백업 - 개요

완전관리형 서비스
섭스 간의 백업을 중점적을 관리 자동화

user data script나 메뉴얼 만들 필요 없이 백업전략을 구축 가능

Supported services

- EC2 / EBS
- S3
- RDS / Aurora / DynamoDB
- DocumentDB / Neptune
- EFS / FSx
- AWS Storage Gateway

리전 간 백업을 지원하므로 재해 복구 전략을 다른 리전에 푸시핧 수 있다.

계정 간 백업도 지원한다.

오로라와 같은 PITR(지정 시간 복구)를 지원한다.

온디맨드 혹은 예약된 백업 지원

Tag-based backup policies : tag가 지정된 리스소만 배업

Backup Plan : 백업 정책

- 매 12시간 마다... 백업 기간 ... 백업을 콜드 스토리지로 보낼지 .. 보유기간...

S3로 백업된다.

vault lock

WORM 정책 시행 시, 볼트 저장 데이터를 삭제할 수 없다.
루트 사용자도 삭제 할 수 없다.

## Application Migration Service(MGN)

마이그레이션 전략

온프로미스 -> 클라우드

서버 스캔, 종속성 검토, 마이그레이션 전략 수립

Agentless Discovery (AWS Agentless Discovery Connector)

- 가상 CPU, 메모리 디스크 성능 기록에 대한 정보 제공

Agent-based Discovery (AWS Application Discovery Agent)

- 가상 머신 내에서 더 많은 업데이트와 정보를 얻을 수 있다.
- 시스템 구성, 성능, 실행 중인 프로세스, 시스템 사이의 네트워크 연결에 대한 세부 정보
- 종속성 맵핑을 얻는데 좋다.

이 모든 결과는 AWS Migration Hub라는 서비스에서 볼 수 있다.

이 서비스는 이동해야 할 항목과 연결성을 확인할 수 있다.

AWS Application Migration Service(MSN)

Lift-and-shift, 물리적, 가상 또는 클라우드에 있는 다른 서버를 AWS 클라우드로 실행하는 것이다.

## 대규모 데이터 세트를 AWS로 전송

대규모 데이터를 전송하는 것 중 여러 제약에 따른 최적의 방식이 무엇인지 요약해보자

200TB 데이터를 클라우드로 옮기고 싶다.
현재 인터넷 속도는 100Mbps이다

1. Over the internet / Site to Site VPN

   - 설치가 빠르고 바로 연결 가능
   - 185일이 걸린다.

2. Over direct connect 1Gbps

   - 초기 설치 시간이 오래걸린다. - 한달 정도
   - 18.5일이 걸린다.

3. Over Snowball
   - 스노우볼 2,3개 주문
   - 일주일 정도 소요된다.
   - DMS로 나머지 데이터 이동

## VMware Cloud on AWS

온프레미스에 데이터 센터가 있을 때 VMware cloud로 데이터 센터를 관리하는 경우가 있다.

vSphere 기반 환경과 가상 머신을 VMWare Cloud로 관리하는 것이다.

데이터 센터와 클라우드를 관리하는데에 VMware Cloud를 사용하고 싶을 수 있다.

VMware Cloud on AWS

위 서비스를 사용하면 기존의 VMWare에 클라우드를 확장해서 사용할 수 있다.
