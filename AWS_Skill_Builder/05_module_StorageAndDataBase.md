# Storage and DataBase

- 스토리지 및 데이터 베이스의 기본 개념을 요약할 수 있다.
- Amazon Elastic Block Store(Amazon EBS)의 이점을 설명할 수 있다.
- Amazon Simple Storage Solution(Amazon S3)의 이점을 설명할 수 있다.
- Amazon Elastic File System(Amazon EFS)의 이점을 설명할 수 있다.
- 다양한 스토리지 솔류션을 요약할 수 있다.
- Amazon Relational Database Service(RDS)의 이점을 설명할 수 있다.
- Amazon DynamoDB의 이점을 설명할 수 있다.
- 다양한 데이터베이스 서비스를 요약할 수 있다.

## 인스턴스 스토어 및 Amazon Elastic Block Store(Amazon EBS)

인스턴스 스토어 - 블록 수준 스토리지 볼륨은 물리적 하드 드라이브처럼 동작

- Amazon EC2 인스턴스에 임시 블록 수준 스토리지를 제공, 인스턴스 스토어는 물리적으로 EC2 instance의 호스트 컴퓨터에 연결되어 있고, instance와 수명이 동일한
  디스크 스토리지이다. 인스턴스 종료 시, 인스턴스 스토어의 데이터가 손실된다.

Amazon EBS(Elastic Block Store) - EC2에서 사용할 수 있는 블록 수준 스토리지 볼륨을 제공하는 서비스

- EC2 인스턴스를 중지, 종료하더라도 연결되 EBS 볼륨의 데이터를 사용할 수 있다.
- EBS 볼륨을 생성하려면, 구성(볼륨의 크기, 유형)을 정의하고 볼륨을 프로비저닝한다. EBS 볼륨을 생성한 다음, 볼륨을 EC2 인스턴스에 연결할 수 있다.
- EBS 볼륨은 보존해야 하는 데이터를 위한 것이므로, 백업이 중요하다. Amazon EBS 스냅샷을 생성하여 EBS 볼륨을 증분 백업할 수 있다.

EBS 스냅샷 - 증분 백업이다.

- 처음 볼륨을 백업하면 모든 데이터가 복사된다. 이후에 백업에서는 가장 최근의 스냅샷 이후 변경된 데이터 블록만 저장된다.
- 증분 백업은 백업이 실행될 때 마다 스토리지 볼륨의 모든 데이터가 복사되는 전체 백업과는 다르다. 전체 백업에는 가장 최근의 백업 이후 변경되지 않은 데이터도 포함된다.

## Amazon Simple Storage Service(Amazon S3)

Object Storage(객체 스토리지) : Data, MetaData, Key

Data : 이미지, 동영상, 텍스트 문서 또는 기타 유형의 파일
MetaData : 데이터의 내용, 사용 방법, 객체 크기
Key : 고유한 식별자

볼륨 스토리지에서 파일 수정 시에는 변경된 부분만 업데이트된다, 하지만 객체 스토리지에서는 파일을 수정하면 전체 개체가 업데이트 된다.

Amazon Simple Storage Service(Amazon S3)

- 객체 수준 스토리지를 제공하는 서비스, Amazon S3는 데이터를 버킷에 객체로 저장한다.

Amazon S3는 이미지, 동영상, 텍스트 파일 등, 모든 유형의 파일을 업로드할 수 있고, S3를 사용하여 백업 파일, 웹 사이트 용 미디어 파일을 저장할 수 있다. S3는 저장공간을 무제한으로 제공한다. 단 S3에 저장할 수 있는 객체의 최대 파일 크기는 5TB이다.

S3에 파일 업로드 시, 권한을 설정할 수 있으며 파일에 대한 표시 여부나 엑세스 제어도 가능하다.
S3의 버전관리 기능을 사용하여, 시간에 따른 객체 변경사항을 추적할 수도 있다.

Amazon S3 Storage Class

S3에서는 사용한 만큼만 비용을 지불하는데, 이때 비지니스 및 비용 요구 사항에 맞춰서 다양한 스토리지 클래스 중에서 선택이 가능하다. 고려할 점은 두 가지이다.

- 데이터를 검색할 빈도
- 필요한 데이터 가용성

1. S3 Standard

   - 자주 엑세스하는 데이터용으로 설계
   - 최소 3개의 가용 영역에 데이터를 저장

   S3 Standard는 객체에 대한 고가용성을 제공, 웹사이트, 콘텐츠 배포, 데이터 분석 등 광범위한 사례에 적합
   S3 Standard는 자주 엑세스하지 않는 데이터 및 보관 스토리지를 위한 다른 스토리지 클래스보다 비용이 비싸다.

2. S3 Standard-Infrequent Access(S3 Standard-IA)

   - 자주 엑세스하지 않는 데이터에 이상적
   - S3 Standard와 비슷하지만 스토리지 가격은 더 저렴하고 검색 가격은 더 높음

   S3 Standard-IA는 자주 접근하지는 않지만 필요에 따라 고가용성이 요구되는 데이터에 적합
   마찬가지로 최소 3개의 가용영역에 데이터를 저장한다.
   S3 Standard와 동일한 수준의 가용성을 제공하지만 스토리지 가격은 더 저렴하고, 검색 가격이 더 높다.

3. S3 One Zone-Infrequent Access(S3 One Zone-IA)

   - 단일 가용 영역에 데이터를 저장
   - S3 Standard-IA보다 낮은 스토리지 가격

   단일 가용 영역에 데이터를 저장한다. 따라서 스토리지 비용을 절감하고 가용 영역에 장애가 발생하더라도 데이터를 손쉽게 재현할 수 있는 경우에 적합하다.

4. S3 Intelligent-Tiering

   - 엑세스 패턴을 알 수 없거나 자주 변화하는 데이터에 이상적
   - 객체 당 소량의 월별 모니터링 및 자동화 요금을 부과

   S3 Intelligent-Tiering에서는 객체 엑세스 패턴을 모니터링한다.
   30일 연속 객체에 엑세스 하지 않으면 S3 Standard-IA로 이동한다.
   이 객체에 사용자가 접근하면 해당 객체를 S3 Standard로 이동한다.

5. S3 Glacier

   - 데이터 보관용으로 설계된 저비용 스토리지
   - 객체를 몇 분에서 몇 시간 이내에 검색

   데이터 보관에 이상적인 저비용 스토리지 클래스이다. 고객 레코드나 이전 사진 및 비디오를 저장할 수 있다.

6. S3 Glacier Deep Archive

   - 보관에 이상적인 가장 저렴한 객체 스토리지 클래스
   - 객체를 12시간 이내에 검색

   S3 Glacier와 이 클래스 중에 결정할 때 고려해야 할 사항은 보관된 객체를 얾마나 빨리 검색해야 하냐에 달려있다. S3 Glacier에 저장된 객체는 몇 분에서 몇 시간 이내에 검색할 수 있다. 하지만 이 클래스는 12시간 이내에 검색할 수 있다.

## Amazon EBS와 Amazon S3 비교

EBS

- 최대 16TB
- EC2 intance 종료 후에도 생존
- 기본적으로 SSD
- HDD 옵션

S3

- 무제한 스토리지
- 최대 5TB의 개별 객체
- 한 번 쓰기 / 여러 번 읽기 (WORM)
- 99.999999999% 내구성

ex. 사진 어플리케이션의 경우 : 수백개의 동물 사진을 업로드 및 인덱싱하고 수천명의 유저가 동시 접근이 되어야 하는 경우
-> S3 사용 : 웹 지원, 리전 별 분산, 비용 절감 효과 제공, 서버리스

ex. 80GB의 오류를 수정해야할 동영상 파일이 있는 경우
-> 객체의 경우, 파일 중 수정이 필요할 때, 전체 업데이트가 필요하고 블록 스토리지는 변경된 부분만 업데이트하는 증분 업데이트가 가능하다. 따라서 이 경우에는 EBS를 적용하는 것이 유리하다.

## Amazon Elastic File System(Amazon EFS)

파일 스토리지에서는 여러 클라이언트(예: 사용자, 어플리케이션, 서버 등)가 공유 파일 폴더에 저장된 데이터를 엑세스할 수 있다. 이 접근 방식에는 스토리지 서버가 블록 스토리지를 로컬 파일 시스템과 함께 사용하여 파일을 구성한다. 클라이언트는 파일 경로를 통해 데이터에 접근한다.

블록 스토리지와 객체 스토리지에 대해 비교하면, 파일 스토리지는 많은 수의 서비스 및 리소스가 동시에 데이터에 엑세스해야 하는 사례에 이상적이다.

Amazon Elastic File System(EFS)는 AWS 클라우드 서비스 및 온프레미스 리소스와 함께 사용되는 확장 가능한 파일 시스템이다. 파일을 추가 혹은 제거하면 EFS가 자동으로 확장하거나 축소된다. 어플리케이션을 중단하지 않고 온디맨드로 페타바이트 규모로 확장도 가능하다.

EBS vs EFS

EBS

    - 단일 가용 영역에 데이터를 저장한다.
    - EC2 intance에 EBS 볼륨을 연결하려면, 동일한 가용 영역에 상주해야 한다.

EFS

    - EFS는 리전 별 서비스이고, 여러 가용 영역에 걸쳐 데이터를 저장한다.
    - 중복 스토리지를 사용하면, 파일 시스템이 위치한 리전의 모든 가용 여역에서 동시에 데이터에 엑세스할 수 있다.
    - 온프레미스 서버는 AWS Direct Connect를 사용하여 EFS에 엑세스할 수 있다.

## Amazon Relational Database Service(Amazon RDS)