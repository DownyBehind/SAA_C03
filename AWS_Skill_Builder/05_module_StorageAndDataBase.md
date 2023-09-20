# Storage and DataBase

* 스토리지 및 데이터 베이스의 기본 개념을 요약할 수 있다. 
* Amazon Elastic Block Store(Amazon EBS)의 이점을 설명할 수 있다. 
* Amazon Simple Storage Solution(Amazon S3)의 이점을 설명할 수 있다. 
* Amazon Elastic File System(Amazon EFS)의 이점을 설명할 수 있다. 
* 다양한 스토리지 솔류션을 요약할 수 있다. 
* Amazon Relational Database Service(RDS)의 이점을 설명할 수 있다. 
* Amazon DynamoDB의 이점을 설명할 수 있다. 
* 다양한 데이터베이스 서비스를 요약할 수 있다. 

## 인스턴스 스토어 및 Amazon Elastic Block Store(Amazon EBS)

인스턴스 스토어 - 블록 수준 스토리지 볼륨은 물리적 하드 드라이브처럼 동작

* Amazon EC2 인스턴스에 임시 블록 수준 스토리지를 제공, 인스턴스 스토어는 물리적으로 EC2 instance의 호스트 컴퓨터에 연결되어 있고, instance와 수명이 동일한
디스크 스토리지이다. 인스턴스 종료 시, 인스턴스 스토어의 데이터가 손실된다. 

Amazon EBS(Elastic Block Store) - EC2에서 사용할 수 있는 블록 수준 스토리지 볼륨을 제공하는 서비스

* EC2 인스턴스를 중지, 종료하더라도 연결되 EBS 볼륨의 데이터를 사용할 수 있다. 
* EBS 볼륨을 생성하려면, 구성(볼륨의 크기, 유형)을 정의하고 볼륨을 프로비저닝한다. EBS 볼륨을 생성한 다음, 볼륨을 EC2 인스턴스에 연결할 수 있다. 
* EBS 볼륨은 보존해야 하는 데이터를 위한 것이므로, 백업이 중요하다. Amazon EBS 스냅샷을 생성하여 EBS 볼륨을 증분 백업할 수 있다. 

EBS 스냅샷 - 증분 백업이다. 

* 처음 볼륨을 백업하면 모든 데이터가 복사된다. 이후에 백업에서는 가장 최근의 스냅샷 이후 변경된 데이터 블록만 저장된다. 
* 증분 백업은 백업이 실행될 때 마다 스토리지 볼륨의 모든 데이터가 복사되는 전체 백업과는 다르다. 전체 백업에는 가장 최근의 백업 이후 변경되지 않은 데이터도 포함된다. 