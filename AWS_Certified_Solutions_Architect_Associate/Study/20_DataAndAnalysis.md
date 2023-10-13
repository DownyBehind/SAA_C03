# 데이터 & 분석

 

## Athena

 

S3에 저장된 데이터를 분석하는 서버리스 쿼리 서비스이다.

데이터를 분석하려면 표준 SQL 언어로 파일을 쿼리해야한다.

 

가격은 스캔된 데이터의 TB 당 고정 가격 지불

전체 서비스가 서버리스여서 DB를 프로비저닝 할 필요가 없다.

 

Amazon Quicksight라는 서비스랑 주로 같이 사용한다.

 

주요 사용사례는 임시 쿼리 수행이나 분석, 보고, 로그 쿼리 분석, VPC 흐름, 로드밸런서 등등...

 

서버리스 SQL 분석 엔진에 대해 나오면 Athena를 떠올리면 된다.

 

성능향상

 

- 용량당 고정 비용이므로, 데이터를 적게 스캔할 유형의 데이터를 사용한다.

colum 형 데이터를 스캔하면 필요한 열만 스캔하므로 비용을 절감할 수 있다.

           - Apache Parquet이나 ORC를 추천한다.

           - 위 형식을 사용하면 성능이 많이 향상된다.

           - 하지만 위의 형식으로 변환하려면 Glue를 사용해야한다.

           - Glues는 적재(ETL)작업으로 CSV와 Parquet 간의 데이터 변환에 유용하다.

          

데이터를 압축해서 더 적게 검색한다.

 

특정 열을 항상 검색한다면, 해당 열을 데이터 분리한다.

           - S3 버킷에 있는 전체 경로를 슬래시로 분할한다는 의미이다.

          

마지막으로 큰 파일(128MB 이상)을 사용해서 오버헤드를 최소화하면 성능을 향상시킬 수 있다.

사이즈가 작은 파일이 많은 경우가 더 성능이 떨어진다.

 

Federated Query (연합 쿼리)

 

S3 뿐 아니라 어떤 곳의 데이터도 쿼리할 수 있다.

사용 방법은 Data Source Connector를 Lambda에서 실행하는 것이다.

그러면 Federated Query를 실행할 수 있다.

 

쿼리 결과는 사후 분석을 위해 S3 버킷에 저장할 수 있다.

 

## RedShift

 

PostgreSQL 기반이지만, OLTP에 사용되지는 않는다.

 

OLAP - Online analytical processing (analytics and data warehousing)

온라인 분석처리를 의미하는 OLAP 유형의 데이터 베이스이다.

 

다른 어떤 데이터 웨어하우징보다 성능이 10배 이상 좋다.

 

성능향상이 가능한데, column 기반 데이터 스토리지를 사용하며, 병렬 쿼리 엔진을 쓴다.

프로비저닝한 인스턴스 비용만 지불하면 된다.

쿼리를 수행할 때 SQL을 사용할 수 있다.

 

Athena랑 비교하면,

 

Redshift는 모든 데이터를 로드해야 한다.

데이터를 모두 로드하고 나면 더 빠르고, join과 aggregation을 더 빠르게 할 수 있다.

왜냐면 Athena에는 없는 인덱스가 있기 때문이다.

 

따라서 임시 쿼리 수행에는 Athena가 낫지만 분석을 할 목적의 집중적인 데이터 웨어하우스라면 Redshift가 더 낫다.

 

레드 쉬프트 클러스터에는 2개의 노드가 있는데

 

Leader Node : 쿼리 계획, 결과 병합

 

Compute Node : 쿼리 수행, 결과를 리더에게 송부

 

노드 크기를 미리 프로비저닝해야 한다.

비용 절감을 위해서는 예약 인스턴스를 사용한다.

 

 

Redshift은 일부 클러스터에 한해 다중 AZ모드 가능 - 재난 복구 전략

단일 AZ라면, 스냅샷을 복구 전략으로 사용해야 한다.

 

스냅샷은 지정 시간 백업이며, S3에 저장되고, 증분한다.

 

변경 사항만 저장되므로 효율적이다.

 

새 클러스터를 위해 스냅샷을 찍을 수도 있다.

스냅샷은 자동으로 하거나 수동으로 할 수 있다.

 

Redshift의 자동스냅샷은 다른 리전의 클러스터에 복사할 수 있는데 재해복구 전략으로 가능하다.

 

데이터 입력 방법

 

1. Kinesis Data Firehose

           - S3에 먼저 쓰고 복사

          

2. 수동으로도 가능

 

3. JDBC driver (from EC2)

- 큰 배치로 데이터를 쓰는 것이 좋다.

 

Spectrum

 

S3에 있는 데이터를 분석하지만 로드하지 않는다.

이 기능을 쓰려면 쿼리를 시작할 수 있는 Redshift cluster가 있어야 한다.

 

이 기능을 쓰려면 SQL에 데이터가 S3에 있음을 명시해야 하고, 수행하며 Redshift Spectrum 수천개가 노드로 형성되어 S3 데이터를 분석하고,

결과를 쿼리를 시작한 곳으로 전송한다.

 

## 오픈 서치 Open Search - Elastic Search

 

Dynamo DB에서는 Primary key 혹은 index로 쿼리가 가능하다.

오픈서치는 모든 필드에 대해서 검색이 가능한데, 부분 매칭이어도 된다.

 

관리 모드 - 직접 관리

서버리스 모드 - 자동 관리

 

OpenSearch에는 자체 쿼리 언어가 있다.

SQL을 호환하지 않지만 플러그인을 설치하면 가능하다.

 

DynamoDB - DynamoDB Stream - Lambda function - OpenSearch

 

여기서 EC2같은 곳에서 오픈 서치에 API를 보낸다. 뭔가 찾기 위해서

 

특정 항목의 ID를 획득하면, 해당 ID 기반으로 디비에서 그 항목을 찾는 식이다.

 

## EMR (Elastic MapReduce)

 

- 빅 데이터 작업을 위한 하둡 클러스터 생성에 사용된다.

 

방대한 양의 데이터를 분석하고 처리할 수 있다.

 

하둡, 빅데이터 이야기 나오면 EMR을 고려하자.

프로비저닝을 해야하며 수백개의 ec2가 사용된다.

 

EMR은 빅데이터 전문가가 사용하는데 사용이 편리하고, 전체 클러스터가 AS되고,

스팟 인스턴스와 통합되므로 가격 할인을 받을 수 있다.

 

마스터 노드 : 클러스터 관리, 다른 모드 노드 상태 조정 - 장기 실행

 

코어 노드 : 테스크 실행, 데이터 저장 - 장기 실행

 

테스크 노드 : 테스크 실행 - 대게 일시적

 

## QuickSight

 

서버리스 머시러닝 기반 비지니스 인텔리전스 서비스이다.

 

대화형 인터페이스있다.

 

대시보드를 생성하고, 시각화 할 수 있다 .

 

웹사이트에 임베드할 수 있고, 세션당 비용을 지불해야 한다.

 

다양한 데이터 소스와 연결할 수 있다.

 

SPICE 엔진 - 데이터를 가져 올 때 사용한다.

 

열(column) 수준 보안을 제고안다.

 

## Glue

 

추출과 변환 로드 서비스를 관리한다.

 

ETL(extract, transform, and load) 서비스

분석을 위해 데이터를 변환하고 준비하는데 유용하다.

 

S3, RDS에서 데이터를 추출하여 원하는 형태로 변형한 뒤, Redshift로 보낼 수 있다.

 

Convert data into Parquet format

 

Athena의 성능 향상을 위해 열 형식인 Parquet으로 변형

 

Glue Data Catalog : catalog of datasets

 

S3, RDS, DynamoDB 등을 Glue Data Crawler로 읽어서

Glue Data Catalog에 보낸다.

 

Glue Data Catalog는 이 데이터, 메타데이터, 스키마들을 ETL 하는데 참고한다.

 

Athena의 경우에도 분석 시 이 카타로그를 활용한다.

 

Redshift Spectrum, EMR도 활용한다. 여러 AWS 서비스의 중심이다.

 

Glue Job Bookmarks : 새 ETL 작업을 실행할 때, 이전 데이터의 전처리를 방지한다.

 

Glue Elastic Views : SQL을 사용해 여러 데이터 스토어의 데이터를 결합하고 복제한다.

 

Glue DataBrew : 사전 빌드된 변환을 활용해 데이터를 정규화하고 정리한다.

 

Glue Studio : 글루에서 ETL job을 생성, 실행 모니터링하는 새로운 GUI이다.

 

Glue Streaming ETL : Apache Spark Structured Streaming 위에서 수행되며,

배치 형태로 데이터를 가져다가 작업하는게 아니라 스트리밍 작업을 실행할 수 있다.

 

## Lake formation

 

데이터 레이크 생성을 돕는다.

 

데이터를 한 곳으로 모아주는 중앙 집중식 데이터 저장소이다.

 

보통 몇 달씩 걸리는 작업을 이 서비스는 며칠만에 완료할 수 있다.

 

Out-of-the-box source blueprints : S3, RDS, Relational & NoSQL DB

Fine-Grained Access Control for your applications (row and column-level)

 

Glue 위에서 동작하지만, 상호작용하지는 않는다.

 

Lake formation을 사용하는 이유는 Centralized Permission이다.

 

데이터 분석 시, 사용자는 허용된 데이터만 보고 읽기 권한이 있어야 한다.

여러 서비스에 보안 설정을 따로 하면 관리가 어렵다.

 

따라서 Lake Formation에 넣고 엑세스 제어를 이 안에서 한다.

 

## Kinesis Data Analytics for SQL applications

 

데이터 소스를 Data stream 또는 Data Firehose로 부터 읽어온다.

그리고 SQL문에 적용하여 실시간 분석을 처리할 수 있다.

 

S3 버킷 데이터를 참조할 수도 있다.

 

그리고나서 여러 대상에 전달한다.

 

Data stream 또는 Data Firehose로 보낼 수도 있고, 다른 분석 툴에 보낼 수도 있다.


전송된 데이터만큼 과금

 

## Kinesis Data Analytics for Apache Flink

 

Apache Flink를 사용하면, Java, Scala, SQL로 앱을 작성하고 스트리밍 데이터를 처리, 분석할 수 있다.

Apache Flink는 SQL보다 강력하기 깨문에 고급 쿼리 능력이 필요하거나 다른 서비스에서 데이터를 받을 때 사용한다.

 

## MSK - Managed Streaming for Apache Kafka

 

Kafka는 Kinesis의 대안이다.

MSK는 AWS의 완전관리형 kafka cluster 서비스로 클러스터를 생성, 업데이트, 삭제 합니다.

MSK는 카프카 브로커 노드와 주키퍼 노드를 생성하고 관리한다.

고가용성을 위해 VPC의 MSK cluster를 최대 세 개의 다중 AZ에 배포한다.

일반 카프카는 장애복구 기능이 있으며, EBS 볼륨에 데이터를 저장할 수도 있다.

 

MSK Serverless

 

- 서버 프로비저닝이 필요없다.

 

Apache Kafka

 

- 데이터를 스트리밍하는 방식이다.

- 카프카 클러스터는 여러 브로커로 구성되고 데이터를 생산하는 생산자는 여러 소스로부터 데이터를 주입한다.

- 카프카에 topic으로 데이터를 전송하면, 해당 데이터는 다른 브로커로 완전 복제된다.

- 카프카 topic은 실시간으로 데이터를 스트리밍하고

- 소비자는 데이터를 소비하기 위해 topic을 polling한다.

- 소비자는 데이터로 원하는대로 처리하거나 특정 서비스를 대상으로 보낸다.

 

Kinesis와 Kafka의 차이점

 

Kinesis

 

- 1MB 사이즈 제한

- 샤드로 데이터를 스트리밍

- 용량을 확장하려면 샤드를 분할한다. 또한 병합할 수도 있다.

- in flight TLS 암호화 존재

 

MSK

 

- 1MB 기본, 설정 시 더 높게 가능 (10MB)

- 파티션을 이용한 Kafka topic을 사용

- MSK는 파티션 추가로 topic 확장만 할 수 있다.

- PLAINTEXT or TLS In-flight Encryption

 

## 빅데이터 수집 파이프라인

 

- 빅데이터 수집 파이프라인이 서버리스로 있으면 좋을 것이다.

- 실시간으로 데이터를 수집하고 싶다.

- 데이터를 변형하고 싶다.

- SQL을 사용해서 변형된 데이터를 쿼리하고

- 쿼리로 리포트를 생성하여 S3에 저장하고

- 데이터를 웨어하우스에 넣고 대시보드를 생성하고 싶다.

 

IoT core IoT 장치 관리를 돕는다.