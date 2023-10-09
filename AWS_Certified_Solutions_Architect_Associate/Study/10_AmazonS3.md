# Amazon S3 소개

## S3 개요

S3는 AWS의 주요 구성 요소 중 하나이다.
무한하게 확장 가능한 스토리지라고 할 수 있다.

S3는 본질적으로 스토리지이다.

Use Cases

- Backup and Storage
- Disaster Recovery
- Archive
- Hybrid Cloud storage
- Application hosting
- Media hosting
- Data lakes & big data analytics
- Software delivery
- Static website

나스닥은 7년간의 데이터를 S3 Glacier에 저장해둔다.

Sysco는 자체 데이터에 대한 분석을 S3를 통해 수행한다.

## Buckets

Buckets은 객체라고 한다.
버킷은 계정 안에서 생성되며 전역적으로 유일해야한다. (unique)
이름은 계정에 있는 모든 리전과 AWS에 존재하는 모든 계정에서 고유해야 한다.
버킷은 리전 수준에서 정의된다.

S3는 전역서비스처럼 보이지만 리전 레벨 서비스이다.

네이밍 컨벤션이 있는데 대문자나 밑줄이 없어야 한다. 길이는 3자에서 63자까지이며, 소문자 혹은 숫자로 시작해야하고 IP는 안되며, xn- prefix 불가 -s3alias suffix 불가이다

## Object

객체는 키를 가진다.
키는 Full path형태이다.

- s3://my-bucket/myfile.txt
- s3://my-bucket/my_folder1/another_folder/my_file.txt

key는 prefix + object name으로 구성된다.

S3의 핵심은 키이다.

최대 객체 크기는 5TB이다.

- 만약에 업로드하는 파일이 5GB보다 크면, "multi-part upload"를 사용해야 한다.
- 메타데이터 : 키 - 값 쌍이다.
- 테그 : 유니코드 키 / 값 페어는 10개까지 가능한데, 보안과 수명 주기에 유용하다.

## S3 보안 및 버킷 정책

User-Based

- IAM Policies - IAM의 특정 유저로 부터 어떤 API가 허용되어야 하는지 지정

Resource-Based

- Bucket Policies - S3 콘솔에서 할당할 수 있는 버킷 룰 : 버킷을 공개할 수 있는 룰
- Object Acess Control List(ACL) : finer grain (can be disabled)
- Bucket Acess Control List(ACL) : less common (can be disabled)

an IAM principal can access an S3 object if

- The user IAM permissions Allow it OR the resource policy Allow it
- And there's no explicit Deny

Encryption : 암호키를 사용하여 객체를 암호화하는 것

S3 Bucket Policies

- JSON based policies
  - Resources: buckets and objects
  - Effect : Allow / Deny
  - Actions : Set of API to Allow or Deny
  - Principal : The account or user to apply the policy to

Ex.

사용자가 버킷에 접근하고자 한다. 정책이 허용하면 사용자는 접근이 가능하다.
다른 방법은 사용자에게 IAM 권한을 할당하는 것이다.

EC2 instance에서 S3 bucket에 접근할 때는 IAM에서 EC2 instance role을 생성하여 접근한다.

Cross-Account Access

IAM user other AWS account 인 경우에는 S3 Bucket policy에서 Cross-Account 정책을 허용하면 접근이 가능하다.

Block Public Access

기업의 보안 유출을 방지하기 위해 AWS가 별도로 개발함
S3 버킷 정책을 설정하여 공개로 만들더라도 이 설정이 활성화가 되어 있으면 버킷은 절대로 공개되지 않는다.

## S3 웹사이트

S3만으로 정적 웹사이트를 만들 수도 있다.

만약에 403 Forbidden error가 발생한다면, 버킷 정책이 public read로 설정되었는지 확인해야 한다.

## S3 Versioning

S3로 버전 관리를 할 수 있다.
동일 키로 업로드하면 버전2, 3.. 등을 생성할 수 있따.
버킷을 버전단위로 관리하면 이전 버전을 복구할 수 있고 쉽게 롤백할 수 있다.

- 버전 관리를 활성화하기 전에 모든 파일은 null 버전을 갖게 된다.

## S3 복제

Replication (CRR & SRR)

소스와 데스티네이션 버킷이 모두 버저닝이 활성화되어야 한다.

CRR - Cross Region Replication

- : 리전이 서로 달라야 한다.

SRR - Same Region Replication

- : 리전이 같아야 한다.

다른 AWS 계정에도 복사가 가능하다.
복사는 비동기로 백그라운드에서 수행된다.
정상적으로 수행하려면 S3에 읽기, 쓰기 권한을 IAM을 통해 부여해야 한다.

Use case

- CRR : 컴플라이언스, 낮은 지여시간 등에 의해 사용, 계정 간 복사
- SRR : 버킷 간의 로그를 통합하거나 개발 환경에 테스트할 때 사용한다.

복제를 활성화 한 후에는 새로운 객체만 복제 대상이 된다.
기존 객체를 복제하려면 S3 Batch Replication을 사용해야 한다.

삭제를 하려면, 소스 버킷에서 대상 버킷으로 삭제 마커를 복제하면 된다.
-> 삭제 마커는 설정으로 지정할 수 있는데, 이걸 설정하면 오리진 버킷에서 삭제마커로 삭제 시에 해당 내용도 복제한다.

버전 ID로 삭제는 복제할 수 없다.

Chaining 복제는 불가하다.

1번 버킷을 복제한 2번 버킷을 복제한 3번 버킷이 있다고 해서, 1번 객체가 3번으로 복제되지는 않는다.

## S3 Storage Class

- Standard - General Purpose

  - 가용성 99.99%
  - 2개의 기능장애를 버틸 수 있다.

- Standard Infrequent Access (IA)
  - 자주 엑세스하지 않지만 빠르게 접근해야 할 경우
  - 보관 비용은 Standard보다 저렴하지만 검색비용은 비싸ㅓ다
  - 재해 복구나 백업 시 사용
  - 가용성 99.9%
- One Zone-Infrequent Acess
  - 단일 AZ 내에서는 높은 내구성을 가지지만 AZ 파괴 시 데이터 손실
  - 가용성 99.5%
  - 온프레미스 데이터를 2차 백업하거나 재생성이 가능한 데이터 저장용

Glacier Class의 경우에 저비용이며 아카이빙 및 백업용
가격은 저장 비용 + 검색 비용

- Glacier Instant Retrieval
  - 밀리초 단위로 검색이 가능, 분기 당 한 번 접근 시 적합
  - 최소 보관 기간 90일
- Glacier Flexible Retrieval
  - 3가지 옵션이 있다.
    - Expeditied (1 to 5 m)
    - Standard (3 to 5 hours)
    - Bulk (5 to 12 hours)
    - 데이터 받는 시간이다.
    - 무료이다.
    - 최소 보관기간은 90일이다
- Glacier Deep Archive
  - 2가지 검색 티어가 있다.
    - Standard (12 시간)
    - Bulk (48 시간)
    - 오래걸리긴 하지만 비용이 가장 저렴하다
    - 최소 보관 기간은 180일이다
- Intelligent Tiering
  - 사용 패턴에 따라 엑세스된 티어에 객체를 이동시킨다.
  - 모니터링 비용과 티어 변경 비용이 발생한다.
  - 티어
    - Frequent Access tier (automatic): default tier
    - Infrequent Access tier (automatic) : objects not accessed for 30 days
    - Archive Instant Access tier (automatic) : objects not accessed for 90 days
    - Archive Access tier (optional) : configurable from 90 days to 700+ days
    - Deep Archive Access tier (optional) : configurable from 180 days to 700+ days

Durability

- 99.999999999% 내구성 보장
- 모든 스토리지 클래스에서 동일

Avalibility

- standard의 경우 99.99% 가용성 / 1년에 53분은 사용할 수 없다.
