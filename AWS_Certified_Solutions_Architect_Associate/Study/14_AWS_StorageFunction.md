# AWS 스토리지 추가 기능

## AWS Snow Fmaily 개요

Snow Family : 아주 안전한 휴대 기기

- Data migration : Snowcone, Snowball Edge, Snowmobile

- Edge computing : Snowcone, snowvall Edge

## Data migration

대량의 데이터를 네트워크 라인으로 전송하려면 상당한 시간이 걸린다.
우리는 종종 AWS에 빠르게 데이터를 보내고 싶다.

그러면 생기는 문제는

- Limited connectivity
- Limited bandwidth
- High netwotk cost
- Shared bandwidth
- Connection stability

AWS Snow Family: offline devices to perform data migrations
신청하면 AWS는 물리적인 기기를 배송해준다.
그러면 우리는 그 물리적 기기에 데이터를 저장해서 다시 AWS에 보낸다.

Snowball Edge

- 커다란 박스형태의 기기
- TB or PB 용량의 데이터를 옮길 때 사용
- 전송 작업 당 비용
- 타입

  - Snowball Edge Storage Optimized
  - 80TB HDD - S3 호환 Object storage

  - Snowball Edge Compute Optimized
  - 42TB HDD or 28TB NvMs - S3 호환 Object storage

Snowcone SSD

- 아주 작은 기기고 열약한 환경에도 견딜 수 있다.
- 2.1kg이며, 원하는 경우 드론에 실어 나를 수도 있다.
- 타입

  - Snowcone - 8TB HDD Storage
  - Snowcone SDD - 14TB SSD Storage

- 공간 제약이 있을 경우 사용하며, 베터리랑 케이블은 직접 준비해야 한다.
- 직접 AWS로 배송해도 되고, 인터넷 연결이 가능하면 AWS DataSync 서비스를 이용해 AWS에 전송해도 된다.

AWS Snowmobile

- 데이터를 옮기는 트럭이다
- 최대 1 EB 전송 가능하다. (1 EB = 1,000 PB = 1,000,000 TB)
- 각 Snowmobile은 100 PB이며, 10대를 주문할 수 있다.
- 온도 조절 및 GPS 24/7 감시, 비디오 감ㅅ가 붙는다.

## Edge Computing

- 에지 위치에서 데이터를 생성하면서 처리하는 것을 말한다.
- 도로에 있는 트럭이나 바다 위의 배 등이 있을 수 있다.
- 연결이 제한되거나 인터넷이 안되는 경우가 있을 수 있다.
- 제한된 장소에서 컴퓨팅을 하도록 지원한다.

- 엣지 위치에 기기를 설치하고
  - 데이터 전처리
  - 머신러닝
  - 미디어 스트림 전송

결국 해당 엣지 컴퓨터를 AWS로 보낸다.

## AWS OpsHub

- CLI 대신 설치해서 GUI로 스노우 기기에 접속하는 프로그램

## 아키텍쳐 : Snowball에서 Glacier까지

스노우볼은 Glacier로 직접 데이터를 넣을 수는 없다.
먼저 S3를 사용해서 수명 주기 정책을 생성하여 Glacier로 넣을 수 있다.

## Amazon FSx 개요

FSx는 완전 관리형 서비스로 타사 고성능 파일 시스템을 실행시킨다.
예를 들어 RDS에서 AWS에 MySQL이나 Postgres를 실행하는 것과 같은 개념이다.

FSx에서 Lustre를 실행하거나, Windows File Server를 실행할 수 있다.
혹은 FSx에서 NetApp ONTAP이나 OpenZFS가 될 수도 있다.

## Amazon FSx for Windows (File Server)

- 완전 관리형 윈도우 파일 서버이다.
- SMB 프로토콜 & Windows NTFS 지원
- Microsoft Active Directory integration, ACLs, user quotas

- 리눅스 EC2에도 마운트 할 수 있다.
- Microsoft's Distributed File System (DFS) Namespaces로 파일 시스템을 그룹화 할 수 있다.

- 10s of GB/s, millions of IOPS, 100s PB of data
- Storage Options
  - SSD
  - HDD
- 프리이빗 연결로 온프레미스 인프라에 엑세스 가능
- 고가용성 다중 AZ에 구성할 수 있다.
- 모든 데이터는 재해 복구 목적으로 S3에 매일 백업된다.

## Amazon FSx for Lustre

- 분산 파일 시스템, 대형 연산에 쓰였다.
- Lustre = Linux + cluster

- 머신러닝과 HPC에 쓰인다.
- 비디오 프로세싱, 금융 모델링, 전자 설계 자동화 등에 쓰인다.
- 100s GB/s, millions of IOPS, sub-ms latencies
- Storage Options

  - SSD
  - HDD

- Seamless integration with S3 : FSx로 S3 파일 시스템처럼 읽어드릴 수 있다.
- FSx 출력값을 S3에 쓸 수 있다.
- VPN or Direct Connect로 온프레미스 서버와 연결이 가능하다.

## FSx File System Deployment Options

- Scratch File System

  - 일시적인 스토리지
  - 데이터는 복제가 안되어 있다. 즉 기저 서버가 오작동하면 파일이 모두 유실된다.
  - 고성능 (영구 시스템보다 6배 성능이 좋고, 200MBps per TiB)
  - 단기 처리용, 비용 최적화 시 사용

- Persistent File System
  - 장기 스토리지
  - 같은 AZ에 데이터 복제해서 관리
  - 기저 서버가 오작동해도 몇 분만에 데이터 대체

## Amazon FSx for NetApp ONTAP

- 관리형 파일 시스템
- NFS, SMB, iSCSI 프로토콜과 호환이 가능하다.
- NAS에서 실행 중인 워크로드를 AWS로 옮길 수 있다.
- 사용 가능한 운영체제

  - Linux
  - Windows
  - MacOS
  - VMware Cloud on AWS
  - Amazon Workspace & AppStream 2.0
  - Amazon EC2, ECS and EKS

- 스토리지는 오토 스케일링 기능이 있고, 복제와 스냅샷도 지원한다.
- 지정 시간 복제 기능이 있다. - 테스트에 좋다

## Amazon FSx for OpenZFS

- 관리형 파일 시스템
- 여러버전의 NFS 프로토콜과 호환가능
- ZFS에 실행되는 워크로드를 AWS에 옮길 때 사용한다.
- 사용 가능한 운영체제

  - Linux
  - Windows
  - MacOS
  - VMware Cloud on AWS
  - Amazon Workspace & AppStream 2.0
  - Amazon EC2, ECS and EKS

- 1,000,000 IOPS with < 0.5 ms latency
- 스냅샷, 압축을 지원하고 비용은 적지만 데이터 중복제거 기능은 없다.
- 지정 시간 복제 기능이 있다. - 테스트에 좋다

## 스토리지 Gateway 개요

- AWS는 hybrid cloud 사용을 권장한다.

  - 일부는 AWS 클라우드에 있고, 나머지는 온프레미스에 두는 방식을 의미한다.

- 이렇게 권장하는 이유는 여러개가 있는데

  - 클라우드 마이그레이션이 오랙 걸리거나
  - 보안 요구사항이 있거나
  - 규정 준수 요건이 있을 수 있기 때문이다.
  - IT 전략

  S3는 자사의 저장 기술을 이용하는데 (EFS / NFS와는 다른), S3 데이터를 온프레미스에 두려면 어떻게 해야 할까?

AWS Storage Gateway가 그 역할을 해준다.

AWS Storage Cloud Native Options

Block

- Amazon EBS
- EC2 Instance Store

File

- Amazon EFS
- Amazon FSx

Object

- Amazon S3
- Amazon Glacier

## AWS Storage Gateway

- 온프레미스 데이터와 클라우드 데이터 간의 Bridge 역할을 한다.
- 이를 이용해서 온프레미스 데이터를 클라우드로 이동시킬 수 있다.

Usa Case:

- 재해복구
- 백업 & 저장
- 스토리지 확장
- 데이터 지연을 방지하기 위해 AWS Storage Gateway를 온프레미스 캐시로 사용한다.

Types

- S3 File Gateway
- FSx File Gateway
- Volumn Gateway
- Tape Gateway

## Amazon S3 File Gateway

- S3 버킷에서는 원하는 스토리지 타입으로 데이터를 넣어을 수 있다. (Glacier는 안됨)
- 온프레미스와 데이터를 주고 받을 건데 이때 표준 네트워크 파일 시스템을 활용하려고 한다.
- S3 File Gateway를 생성하여, 앱서버는 NFS나 SMB 프로토콜을 사용하도록 한다.
- S3 File Gateway는 HTTPS로 S3에 데이터를 전송한다.
- 해당 데이터를 아카이빙하고자 할 때는 버킷 life cycle 정책을 설정하여 S3 Glacier로 넘어가게 하면 된다.
- 사용한 데이터는 파일 게이트웨이에 캐시로 저장된다.
- SMB 프로토콜을 사용할 경우에는 AD(Active Directory)를 사용자 인증을 위해 거쳐야 한다.

## Amazon FSx File Gateway

- FSx File System을 사용하는 상태에서 클라이언트에서 SMB를 사용한다면, 별 다르게 작업할게 없다.
- 이미 온프레미스 시스템에서 엑세스가 가능하기 때문이다.
- 그럼 게이트웨이를 생성하는 이유는 뭘까? 바로 로컬캐시 때문이다.
- 로컬캐시를 사용하여 지연시간을 줄일 수 있기 때문이다.

## Amazon Volumn Gateway

- 블록 스토리지로 S3가 백업하는 iSCSI 프로토콜을 사용한다.
- 볼륨이 EBS 스냅샷으로 저장되어 필요에 따라 온프레미스 볼륨을 복구할 수 있다.
- 볼륨 게이트웨이는 두 가지 유형이 있는데
  - 하나는 캐시 볼륨으로 최근 데이터 엑세스 시 지연시간이 낮고
  - 다른 하나는 저장 볼륨으로 전체 데이터 세트가 온프레미스에 있으며 주기적 Amazon S3 백업이 따른다.

## Amazon Tape Gateway

- 물리적으로 테이프 백업 시스템이 있는 회사가 백업에 테이프 대신에 클라우드를 활용해 데이터를 백업할 수 있게 해준다.
- 가상 테이프 라이브러리(VTL)은 S3와 Glacier를 이용한다.
- 테이프 기반 프로세스의 기존 백업 데이터를 iSCSI 인터페이스를 사용하여 백업한다.
- 업계를 선도하는 백업 SW 벤터가 사용하는 서비스이다.

- 게이트웨이는 각자 회사의 데이터 센터에서 운영해야 한다.
- 하지만 종종 게이트웨이를 실행할 가상 서버가 없을 수 있다.

- Storage Gateway - Hardware appliance
  온프레미스에 서버가 없는 경우, Storage Gateway 하드웨어 어플라이언스를 사용할 수 있다.
  아마존에 구입이 가능하다.

## AWS Transfer Family

S3나 EFS의 안팍으로 데이터를 전송하는데 S3 api나 EFS를 사용하고 싶지 않고, FTP 프로토콜만 사용하고 싶을 수 있다.

이런 경우에 이 제품을 사용한다.

지원 프로토콜

- FTP(File Transfer Protocol) - 암호화 x
- FTPS(File Transfer Protocol over SSL)
- SFTP(Secure File Transfer Protocol)

- 완전 관리형 인프라이며, 확장성, 안정성이 높고 가용성이 높은게 특징이다.
- 비용은 시간당 프로비저닝된 엔드포인트 비용 + 안팍으로 전송된 데이터 GB당 요금
- 유저 크리덴셜을 저장하거나 관리할 수 있다.
- 다른 기존 인증 시스템과도 통합가능하다.
- Microsoft Active Directory, LDAP, Okta, Coginto, cutom이나 sharing file와 연동가능

## DataSync - 개요

- 서비스는 데이터를 동기화하여 대용량을 데이터를 다른곳으로 옮길 수 있따.
- 온프레미스나 다른 클라우드로 옮길 때, NFS, SMB, SHFS, S3 같은 것과 옮길 위치의 온프레미나 연결할 클라우드에 에이전트가 있어야 한다.
- 한 AWS 서비스에서 다른 AWS 서비스로 데이터를 옮실 수도 있다.

Can Synchronize to

- Amazon S3 (any storage classes - including Glacier)
- Amazon EFS
- Amazon FSx (Windows, Lustre, NetApp, OpenZFS,...)

복제 작업은 계속 이루어지는게 아니라 일정을 정해서, 하루 단위 혹은 시간 단위, 주 단위로 실행한다.
지연이 발생해도 동기화 된다.

파일 권한가 메타데이터 저장 기능이 있다. (NFS, POSIX, SMB...)

- 파일을 옮기더라도 파일의 메타데이터를 보존할 수 있다.
- 한 에이전트 테스크가 10Gbps를 사용가능하다. 또한 대역폭 제한도 걸 수 있다.

- 데이터 싱크를 사용하고 싶으나 네트워크가 따라주지 못하는 경우에 Snowcone을 사용할 수 있는데, 데이터 싱크가 안에 내장되어 있기 때문이다.

다른 AWS 서비스 끼리 동기화도 할 수 있다.

- 서로 다른 서비스의 메타데이터와 파일권한 동기화된다.

## 모든 AWS 스토리지 옵션 비교
