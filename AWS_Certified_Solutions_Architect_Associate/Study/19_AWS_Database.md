# AWS의 데이터베이스

 

## 올바른 데이터베이스 선택

 

워크로드에 맞는 올바른 데이터베이스를 써야 한다 .

무엇을 쓸 지는 문제가 어떤 아키텍쳐를 요구하는지에 따라 다르다.

 

쓰기가 많은지, 읽기가 많은 지, 처리량은 얼마나 되는지, 처리량은 급변하는지, 데이터는 얼마나 저장되고 확장은 가능하지

 

평균 객체 크기는 어떻고 엑세스 빈도와 방법은 무엇인지?

 

데이터 내구성이나 신뢰성은 어떻게 보장하는지

감사가 필요한 지 검색이 필요한 지

 

 

Database Types

 

- RDBMS(=SQL/OLTP):RDS, Aurora - great for joins

- NoSQL database - no joins, no SQL: DynamoDB(~JSON), ElastiCache(key/value pair),

Neptune(grapths), DocumentDB(for MongoDB), Keyspaces(for Apache Cassandra)

 

- Object Store: S3 (for big object) / Glacier (for back ups/archives)

- Data Warehouse (=SQL Analytics / BI): Redshift (OLAP), Athena, EMR

- Search : OpenSearch(JSON) - free text, unstructured searches

- Graphs : Amazon Neptune - display relationships between data

- Ledger : Amazon Quantum Ledger Database

- Time series : Amazon Timestream

 

## RDS

 

관리형 PostreSQL / MySQL / Oracle / SQL Server / MariaDB / Custom

 

RDS instance Size, EBS 볼륨과 크기 / 타입을 프로비저닝해야 한다.

스토리지 계층에 AS 기능이 있어도 프로비저닝을 해야 한다.

 

읽기 용량 확장을 위해 읽기 복제본을 지원한다.

고가용성을 목적으로 다중 AZ 적용 가능 그리고 대기 RDS는 재난 상황 대비용으로 쿼리를 수행할 수는 없다.

RDS 보안은 IAM, 보안 그룹,KMS, SSL in transit

자동 백업 옵션이 있으며 최대 35일까지 지원됨

장기 백업이 필요한 경우, 수동 DB 스냅샷을 찍으면 된다.

유지보수를 스케줄링할 수 있는데, 이 때문에 다운타임이 발생할 수도 있다.

RDS 프록시에 강제하여 RDS에 IAM 인증을 추가하는 기능도 있다.

 

 

## Aurora

 

PostgreSQL / MySQL과 호환, 컴퓨팅과 스토리지가 분리되어 있다.

스토리지의 경우 세 가용영역에 걸쳐 6개의 복제본에 저장한다. - 고가용성, 자동복구, 오토스케일링

컴퓨팅의 경우, 클러스터화된 실제 데이터베이스 인스턴스를 여러 가용영역에 걸쳐 저장할 수 있다.

읽기 복제본이 있다면, 로드가 증가할 때 AS를 통해 용량을 늘릴 수 있다.

 

데이터 베이스 인스턴스가 있으므로, 읽기 / 쓰기를 위한 사용자 지정 엔드포인트, 즉 라이터 엔드 포인트,

리더 엔드 포인트가 필요하다.

 

Aurora는 RDS와 동일한 보안, 모니터링, 유지 관리 기능을 갖고 있다.

 

Aurora Serverless - 워크로드가 간헐적이거나 예측할 수 없는 경우 용량 게획을 하지 않아도 되므로 유용하다

Aurora Multi-Master - 연속 쓰기 실패 시 (고 쓰기 가용성)

Aurora Global - 글로벌 데이터베이스를 원할 때, 지역당 16개 읽기 전용 디비 인스턴스, 리전 간 스토리지 복제에 걸리는 시간은 1초 미만이다.

기본 리전이 문제가 생기면, 보조 리전을 승격시킬 수 있다.

Aurora Machine Learning : SageMAker & Comprehend on Aurora

Aurora Database Cloning : Test나 Prod 용 데이터 베이스 생성, 스냅샷 보다 빠르다.

 

## ElastiCache

 

ElastiCache는 완전 관리형 Redis / Memcached로 RDS와 유사하지만, 이 경우엔 캐싱 작업에 활용된다.

인 메모리 데이타 스토어로 1ms 미만의 지연시간을 제공

EC2 인스턴스 타입을 프로비저닝해야 한다.

클러스팅을 지원하고, 다중 AZ, 읽기 전용 복제본(샤딩)지원

보안을 위해 IAM, 보안 그룹, KSM, Redis Auth

백업, 스냅샷

유지보수를 예약할 수 있다.

RDS를 쓰는 앱에서 캐싱 솔류션을 도입하려면 앱 내에 코드를 수정해야 한다.

키 벨류 저장, 자주 읽을 경우 데이터베이스 쿼리를 캐싱하는게 좋다.

 

## DynamoDB

 

NoSQL - 완전 관리형

프로비저닝, 온디맨드 용량 모드

수 밀리 지연시간

ElastiCache를 키 / 값 저장에 쓸 수 있고,

고가용성, 다중 AZ, 읽기 쓰기 분리

DAX 클러스터

Dynamo stream - 람다 호출 가능

Kinesis Data Stream을 적용할 수도 있다.

Kinesis Data Stream은 최장 1년까지 데이터 보관 가능

DynamoDB 35일까지 저장 가능, PITR 내에 s3에 보낼 수도 있다.

시험에서 빠른 스키마나 유연한 타입의 스키마가 필요하다면 DynamoDB이다

혹은 분산형 서버리스 캐시가 필요한 경우

 

## S3

 

키 - 밸류지만 큰 사이즈 용

서버리스이므로 확장성이 무제한이다.

객체당 5TB 제한

수명 주기를 통한 타입 변경

버저닝, 암호하, 복제, MFA-delete

IAM, ACL, Bucket Policis, Access Point, Vault Lock

Encryption: SSE-S3, SSE-KMS, SSE-C

Batch : 한 번에 객체들에 뭔가 할 때

inventory : 파일 리스트를 조회할 때 사용

성능 : Multi-part upload, S3 Transfer Acceleration(리전 간 전송), S3 Select

자동화 : S3 Event Notificatoin (SNS, SQS, Lambda, EventBridge)

 

 

## DocumentDB

 

mongoDB용 Aurora이다.

그리고 NoSQL DB이다.

 

몽고DB는 JSON 데이터를 저장, 쿼리, 인덱스 하는데 사용된다.

완전 관리형 데이터베이스이며 가용성이 높다.

 

데이터는 3개의 가용영역에 복제되고 스토리지는 자동을 10GB 단위로 64TB까지 증가할 수 있다.

초당 수백만건의 요청이들어올 수 있는 데이터베이스로 설계되었다.

 

## Neptune

 

그레프 데이터 베이스이다.

소셜네트워크가 그 예이다.

 

3개 가용영역에 걸쳐 15개의 읽기 복제본 구축

복잡하고 어려운 쿼리를 실행하기에 최적화되어 있다.

 

최대 수십억 개의 관계를 저장할 수 있고, 그래프를 쿼리할 때 지연시간은 밀리초이다.

워키피디아 같은 앱에 적합학, 사기 탐지, 추천 엔진, 소셜 네트워킹에도 적합하다.

 

## keyspaces

 

아파치 카산드라를 보조한다.

 

카산드라는 오픈소스 NoSQL 분산 데이터베이스이고 이걸 사용하면 AWS가 카산드라를 직접 관리한다.

테이블 데이터는 여러 AZ에 걸쳐 3번 복제된다.

 

서버리스, 확장성, 고가용성 완전관리형 서비스이다.

 

CQL(Cassandra Query Language)를 사용하여 쿼리를 수행

어떤 규모에서도 지연시간이 10ms 미만으로 짧고 초당 수천 건의 요청을 처리한다.

 

사용 사례는 IoT 장치 정보와 시계열 데이터 저장등이 있다.

 

## QLDB(Quantum Ledger Database)

 

금융 원장 데이터 베이스이다.

 

앱 데이터의 변경을 검토하는데 시간을 보낸다.

그래서 장부라고 한다.

 

불변 시스템이다. 뭔가 쓰면 수정할 수 없다.

암호화 서명을 하기도 한다.

 

내부적으로 저널이 있다.

수정이 일어날 때마다 암호화 해시가 된다.

데이터베이스 사용하는 사람들은 다 볼 수 있다.

 

일반 원장 블록체인 프레임워크보다 2,3배 더 나은 성능을 얻을 수 있다.

SQL을 이용해 관리할 수 있다.

 

QLDB는 탈중앙화 개념이 없다.

중앙 제어 요소가 있다.

 

## Timestream

 

시계열 데이터베이스이다.

빠르고 확장성이 있다.

시계열 데이터에는 더 싸고 적합하다.

SQL과도 호환된다.

분석 기능도 있다.