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

RDS Backups - Automated Backups : Daily full backup of the database (during the backup window)

5분 마다 트랜잭션 백업이 진행된다.
따라서 자동 백업으로 5분 전으로 복구될 수 있고 이 백업 보존 기간은 1 ~ 35일 중에 설정 가능하고, 0이면 사용안함

Manual DB Snapshot

- 수동으로 한 백업을 원하는 기간동안 보관할 수 있다.

한달에 2시간 만 사용하는 데이터베이스의 경우에 굳이 사용안하는 기간동안 RDS 데이터 유지비용을 낼 필요가 없다.
따라서 사용하지 않을 때는 수동 스냅샷을 찍고, 다시 사용할 때 해당 디비를 복구시키면 비용을 절감할 수 있다.

Aurora Backups

- 1 ~ 35일 자동 백업 - 비활성화 불가능
- 시점 복구 기능이 있다.

Manual DB snapshot

- 수동으로 트리거 할 수 있고, 원하는 만큼 유지할 수 있다.

RDS & Aurora Restore options

1. RDS/Aurora 모두 수동 백업을 복구할 때 새로운 디비로 복구 가능하다.

2. 또 다른 옵션으로는 S3에서 MySQL DB를 복구할 수 있다.

   - Create a backup of your on-premises database
   - Store it on Amazon S3 (object storage)
   - Restore the backup file onto a new RDS instance running MySQL

3. Restoring MySQL Aurora cluster from S3
   - Create a backup of your on-premises database using Percona XtraBackup
   - Store the backup file on Amazon S3
   - Restore the backup file onto a new Aurora cluster running MySQL

Aurora Database Cloning

기존 데이터베이스에서 새로운 Aurora DB를 만들 수 있다.
스냅샷을 찍고 복원하는 것보다 빠른데 이유는 copy-on-write 프로토콜을 사용하기 때문이다.

빠르고 비용 효율적며 기존 디비에 영향을 주지않는다.

## RDS Security

저장된 데이터를 암호화할 수 있다.
AWS KMS를 이용해 마스터와 복제본에 암호화할 수 있다.
만약에 마스터를 암호화하지 않으면 복제본도 암호화할 수 없다.

암호화되지 않은 디비를 암호화하려면 스냅샷을 생성해서 암호화하고 복원하는 과정을 거쳐야 한다.

- In-flight encryption : TLS-ready by default, use the AWSTLS root certificates client-side

- IAM Authentication : IAM roles to connect your database (instead of username/pw)
- Security Groups : Control Network access to your RDS / Aurora DB
- No SSH available : except on RDS custom
- Audit Logs can be enabled and sent to CloudWatch Logs for longer retention

## RDS Proxy

Fully managed database proxy for RDS
프록시를 사용하면 디비 연결풀을 형성하고 사용할 수 있다.

Allows apps to pool and share DB connections established with DB

Improving DB efficiency by reducing the stress on DB resources and minimize open connections

Serverless, autoscaling highly available (multi-AZ)
프록시로 장애시간을 66% 줄일 수 있다.
MySQL, PostgreSQL, MariaDB 그리고 Aurora에서 지원한다.

앱 코드 바꿀 필요없이 RDS로의 연결을 RDS 프록시로 변경하면 사용할 수 있다.

RDS는 퍼블릭 엑세스가 불가능하므로 VPC에서만 사용 가능하다.

## ElastiCache

Redis 또는 Memcached를 관리하는 것을 도와준다.
캐시는 인 메모리 DB이며 자주 사용하는 데이터를 저장하는데 사용한다.

디비 캐시 원리

앱이 쿼리를 보낸다. 이때 캐시에 해당 내용이 있으면 즉각적으로 응답을 하지만 없을 경우, 디비로 가서 해당 값을 읽어온다. 이때 읽은 내용을 캐시에 써놓으면 다음 번에 같은 쿼리를 보내면 즉각적으로 응답할 수 있게 된다.

즉각적으로 응답하는 것을 cache hit, 응답하지 못하는 것을 cache miss라고 한다.

- Cache must have an invalidation strategy to make sure only the most current data is used in there. -> 어려운 기술이다.

User Session Store

사용자가 앱에 로그인하면, 해당 세션 데이터를 캐시에 저장한ㄴ다.
사용자가 이후에 다른 인스턴스를 통해 로그인하더라도 해당 세션 상태가 캐시에 있으므로 재 로그인할 필요가 없다.

Redis : 자동 장애 조치 기능이 있는 Multi-AZ가 있고, 읽기 복제본이 있다. 백업과 복구가 있다.
Supports Sets and Stored Sets : 복제되는 캐시이다. 따라서 가용성과 내구성이 뛰어나다.

Mamcached : 데이터 분할을 위한 멀티 노드 (sharding), 저 가용성, 백업이나 복구가 없다. Multi-threaded architecture

## 솔루션 설계자를 위한 ElastiCache

- ElastiCache supports IAM Authentication for Redis

- IAM policies on ElastiCache are only used for AWS API-level security
- Redis AUTH

  - you can set a "password/token" when you create a Redis cluster
  - 보안 그룹 추가로 캐시에 추가적인 보안레벨 부여
  - SSL in flight encryption 제공

- Memcached
  - Supports SASL-based authentication(advanced)

## Pattern for ElastiCahce

- Lazy Loading : all the read data is cahced, data can become stale in cache
- Write Through : Adds or update data in the cache when written to a DB (no stale data)
- Session Store : store temporary session data in a cache(using TTL features)

Redis Use Case

- 게이밍 리더 보드 만들기

* Redis Sorted sets guarantee both uniqueness and element ordering
* Each time a new element added, it's ranked in real time, then added in correct order

## 친숙한 포트 목록

중요한 포트

FTP : 21
SSH : 22
SFTP : 22 (SSH와 같은)
HTTP : 80
HTTPS : 443

vs RDS 데이터베이스 포트 :

PostgreSQL : 5432
MySQL : 3306
Oracle RDS : 1521
MSSQL Server : 1433
MariaDB : 3306 (MySQL과 같음)
Aurora : 5432 (PostgreSQL와 호환 시), 3306(MySQL과 호환 시)
