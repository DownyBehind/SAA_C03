# AWS 기초 : RDS + Aurora + Elastic Cache

## Amazon RDS 개요

RDS 표준은 Relational Database Service이다.
SQL이라는 쿼리 언어를 사용해서 DB를 관리하는 서비스이다.

AWS가 DB를 관리한다.

- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Aurora (AWS Proprietray database)

그럼 왜 EC2 instance에 직접 DB를 설치하지 않고 RDS를 사용할까?

- 안 쓸 이유가 없기 때문에
- RDS는 AWS에서 관리하기 때문에 DB 뿐 아니라 다양한 서비스를 제공한다.
- 데이터베이스 프로비저닝과 OS 패치가 자동으로 된다.
- 연속적으로 백업이 되고, 특정 시점에 대해서 데이터를 저장할 수 있다.
- 모니터링 대시보드로 현황을 모니터링할 수 있다.
- 읽기 전용 복사본을 이용하여, 읽기 성능을 올릴 수도 있다.
- DR(Disaster Recovery)를 위한 다중 AZ 구축도 가능하다.
- Maintenance 기간동안 업그레이드 가능
- Scaling capacity (vertical and horizonral) 확장 가능
- 파일 스토리지는 EBS (sp2, io1)

- 단점은 RDS 인스턴스에 ssh 접근이 불가하는 점이다.

## RDS - storage Auto Scaling

RDS DB instance의 용량을 다이나믹하게 증가시켜준다.
따라서 DB를 끄고 용량을 늘리는 일을 할 필요가 없다.

- Maximum Storage Threshold를 지정해야 한다. (최대 임계값)
- 남은 공간이 10% 미만이 되면 ~
- Low-storage가 5분간 지속되면 ~
- 마지막으로 변경한 지 6시간이 지나면 ~
  자동으로 스토리지가 확장된다.

워크로드를 예측할 수 없는 어플리케이션에 유용하며 모든 DB에 적용 가능하다.

## RDS 읽기 전용 복제본과 다중 AZ

RDS Read Replica

- 15개까지 Read Replica 생성 가능
- AZ 내에, AZ 간에, Region 간에 생성 가능
- Replica는 Async로 원본과 데이터를 맞춘다.
- Replica는 정식 DB로 승격될 수 있다.
- App은 반드시 read replica와 연결을 업데이트해야 한다.

Use Cases

일반적인 앱을 운용하고 있는데, 다른 팀에서 우리 DB를 기반으로 분석을 하고 싶다고 한다.
이럴 때 원본 DB를 같이 사용하게 되면, 오버로드가 발생할 수 있다.

따라서 이럴 때는 읽기 전용 복제본을 생성하여 사용하면 유용하다.
이렇게 되면 복제본과 DB instance간에 비동기식 복제가 발생한다.
그 다음에 다른 팀에 해당 복제본을 기반으로 데이터를 읽어들인다.

복제본의 경우 SELECT 키워드만 사용할 수 있다.
데이터 자체를 바꾸는 키워드는 불가능하다.

Network Cost

- 보통 AWS에서는 하나의 AZ에서 다른 AZ로 데이터가 넘어갈 때 비용이 발생한다.
- 하지만 예외가 존재하는데 보통 관리형 서비스에서는 예외가 있다.
- RDS는 관리형 서비스이고 같은 리전에 있다면 비용이 발생하지 않는다.
- 하지만 서로 다른 리전에 복제된다면, 복제 비용이 발생하낟.

## RDS Multi SZ (Disaster Recovery)

RDS DB instance Stnadby에 원본의 데이터가 Sync 복제가 된다.
(즉, 원본에서 일어나는 변화가 그대로 반영된다.)

즉, 하나의 DNS 이름을 가지며, 원본이 문제가 생기면 바로 대체된다.
가용성을 증가시켜주면, AZ에 문제가 발생하거나, 네트워크가 손실되거나 인스턴스가 문제가 있을 때를 대비한다.

앱에서 수동으로 처리할 필요없고, 자동으로 처리된다.
스케일링 목적으로 사용할 수 없으며, 반드시 장애 처리를 위해서만 사용된다.

재해 복구를 목적으로 읽기전용 복제본을 사용할 수 있을까?
사용할 수 있다.

## RDS - From Single-AZ to Multi-AZ

- Zero DownTime Operation (no need to stop the DB)
- Just click on "modify" for the database
- The following happens internally
  - A snapshot is taken
  - A new DB is restored from the sanpshot in a new AZ
  - Sychronization is established between the two databases

## RDS Custom for Oracle과 Microsoft SQL Server

RDS에서는 운영체제나 사용자 지정 설정등에 접근이 불가능하다.
하지만 몇가지의 경우에는 Custom이 가능한데, Oracle과 Microsoft SQL Server DB이다.

RDS로 사용하면, 자동으로 셋업, 운영 그리고 DB scaling을 AWS에서 지원한다.
여기에 Custom 옵션을 사용하면, underlying DB와 OS에도 접근할 수 있게 된다.

- Confifure Settings
- Install patches
- Enable native features
- Acess the underlying EC2 instance using SSH or SSM Session Manager

De-activate Automation Mode to Perform your customization, better to take a DB snapshot before

RDS vs RDS Custom

- RDS : entire DB and the OS to be managed by AWS
- RDS Custom : full admin access to the underlying OS and the DB

## Amazon Aurora 개요

- Aurora is a proprietary technology from AWS (not open sourced)
- Prostgres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL DB)
- Aurora는 RDS의 MySQL보다 5배 성능이 좋고, RDS의 Postgres보다 3배 성능이 좋다.
- Aurora는 자동으로 용량을 증가시키는데, 10GB에서 시작해서 128TB까지 확장이 가능하다.
- 읽기전용 복제본은 15개까지 가능하다. (MySQL은 5개까지 가능하다.) - 또한 복제본의 속도도 빠르다.
- Aurora의 장애조치는 즉각적이다.
- 비용은 RDS보다 20%만큼 비싸다.

## Aurora High Avilabilty and Read Scaling

- Aurora는 3 AZ에 걸쳐서 데이터를 6개로 복사해놓는다. - 가용성이 높다.
- 쓰기에는 6개 중 4개만 있으면 되고, 읽기에는 6개 중 3개만 있으면 된다.
- 문제가 생길 경우에는 백엑드에서 P2P 복제를 통한 자가 복구가 진행된다.
- 단일 볼륨에 의존하지 않고 수백개의 볼륨에 동작한다.

Aurora도 master에서 쓰기 명령을 받는데, 30초 동안 동작하지 않으면 장애조치를 시작한다.
마스터 외에 읽기를 제공하는 읽기 전용 복제본을 15개까지 둘 수 있다.
마스터가 문제 생기면 읽기 전용 복제본 중 하나가 마스터 역할을 할 수 있다.

## Aurora DB Cluster

Write Endpoint - 스토리지에 쓰는 것은 마스터에만 가능하다. 라이트 엔드포인트로 DNS로 마스터를 가르킨다.

Reader Endpoint - 모든 읽기 전용 복제본과 자동으로 연결된다. - 읽기 전용 복제본은 오토 스케일링이 가능하다.
유저 입장에서 읽기 전용 복제본이 어디있는지 알기 어려울 수 있다.
이 때 리더 엔드포인트가 로드 벨런싱을 도와준다. - 문장 레벨이 아닌 연결 레벨에서 일어난다.

Features of Aurora

- Automatic fail-over
- Backup and Recovery
- Isolation and security
- Industry compliance
- Push-buttom scaling
- Automated Patching with Zero Downtime
- Advanced Monitoring
- Routine Maintenance
- Backtrack - 특정 어느시점으로 돌아가게 해주는 기능

## Amazon Aurora - 고급개념

Replica - Auto Scaling

총 3개의 Aurora instance가 있고, 1개는 Writer Endpoint는 2개는 Reader Endpoint라고 하자.
그리고 리더 엔드포인트에 많은 읽기 요청이 발생한다고 하자.

Aurora DB CPU 사용량이 증가되면, 복제본이 증가한다.
그리고 리더 엔드포인트가 이 인스턴스를 사용하기 위해 동작하고 로드밸런싱을 한다.

Custom Endpoints

설정은 동일하지만 다른 인스턴스가 있다고 가정하자.
리더 엔드포인트에 2개의 db.r3.large와 2개의 db.r5.2xlarge가 있다.

이렇게 하는 이유는 custom Endpoint를 지정하기 위해서인데, 예를 들어 종 더 성능이 좋은 두 인스턴스에게 Custom endpoint를 지정할 수 있다.

예를 들어 이 특정 복제본을 이용하여 분석 쿼리를 돌리는 것이 가능하다.
일반적으로 특정 인스턴스가 사용자 지정 엔드포인트로 지정되면 리더 엔드포인트는 해당 인스턴스를 일반적으로 사용하지 않는다.

serverless

- Automated DB instantiation and auto-scaling based on actual usage
- Good for infrequent, intermittent or unpredictable workloads
- No capacity planning needed
- Pay per second, can be more cost-effective

Mulit-Mater

- 쓰기 노드에 대한 지속적인 가용성을 원할 때이다.
- 모든 노드가 R/W가 가능하다. 물론 마스터는 하나이지만

Global Aurora

- 리전간 읽기 복제본
- 재해에 좋다.

Aurora Global DB

- 1 Primary Region (R/W)
- Up to 5 secondary (Read-only) flwjs, 복제 지연은 1초 미만
- 보조 리전 당 최대 16개의 읽기 복제본 설정 가능
- 전세계 읽기 워크로드 지연시간을 줄일 수 잇다.
- 재해로 인해 복구하는 시간 RTO는 1분 미만이다.

"Aurora 글로벌 데이터베이스의 데이터를 리전 간에 복제하는데 평균 1초 미만이 소요된다."
-> 글로벌 Aurora 이야기이다.

Aurora Machine Learing

- SQL을 통해서 앱을 통해 ML 기반 예측을 추가할 수 있다.
- Aurora와 다양한 머신러닝 서비스간의 간단하고 최적화되고 안전한 통합이다.

## RDS & Aurora - 백업과 모니터링
