# AWS의 컨테이너: ECS, Fargate, ECR 및 EKS

## Docker 소개

도커 란 - 앱 ㅍ배포를 위한 소프트웨어 개발 플랫폼이다.

컨테이너는 표준화되어 있어서, 어느 운영체제에든 똑같이 돌아간다.

- 어떤 머신이든
- 호환성 문제가 없고
- 예측 가능하고
- 할 게 적고
- 운용하기 쉽고
- 어느 언어, 어느 운영체제, 어느 기술에도 돌아간다.

도커는 마이크로 서비스 아키텍쳐, lift and shift apps from on-premises to the AWS Cloud

도커 리포지토리 - 도커 이미지 저장

- Docker Hub : Public repo

- Amazon ECR (Amazon Elastic Container Register) : Private repo

Docker vs VM

- 도커도 가상화 기술이지만, 리소스가 호스트와 공유되어 한 서버에서 다수의 컨테이너를 공유할 수 있다.

Dockerfile - 도커를 시작할 때 필요한 파일

## Amazon ECS

- EC2 Launch Type

AWS에 도커 컨테이너를 실행시킨다. = ECS 클러스터에 ECS Task를 실행시킨다.

EC2 Launch Type의 경우, EC2를 미리 프로비저닝하고 인프라를 유지해야 한다.

EC2 instance들은 각자 ECS Agent를 실행시켜서 ECS Cluster에 등록해야 한다.
그러면 이제 ECU task에 따라서 컨테이너를 각각 EC2에 실행시키거나 종료한다.

- Fargate Launch Type

미리 프로비저닝하지 않는다. 그리고 서버리스이다. EC2를 관리하지 않는다.
ECS 클러스터가 있을 때, ECS Task만 정의하는 Task Definition만 생성하면, 필요한 리소스는 ECS Task가 대신 실행한다.

새로운 도커 컨테이너를 실행하면, 어디에 실행되는지 알려주지 않고 실행도니다.
확장하려면 Task 수만 늘리면 된다.

EC2 타입보다 사용하기 쉽다.

IAM Roles for ECS

EC2 Launch Type only :

- EC2 instance profile used by the ECS agent
- EC2 instnace profile을 이용해 API 호출을 한다.
- 그럼 인스턴스가 ECS 서비스가 CloudWatch 로그에 API 호출을 해서
- 컨테이너 로그를 보내고 ECR로부터 도커 이미지를 가져온다.

ECS Task Role :

- 각자 특정 역할을 만들 수 있다.
- 각자 다른 서비스와 연결할 수도 있기 때문에 롤을 다르게 가져갈 수도 있다.
- Task Definition에서 Task Role을 정의한다.

ECS - Load Balancer Integrations

ALB를 ECS에 연결하면, 사용자가 ECS Task에 직접 연결될 수 잇다.

NLB는 처리랴량이 많거나 복잡할 때 추천한다.

AWS Private Link

ECS - Data Volumes (EFS)
EFS는 ECS Task에 직접 연결될 수 있어서 자주 사용된다.

Fargate + EFS = Serverless

EFS + ECS : 영구적으로 다중 AZ인 저장소를 컨테이너에 쓸 수 있다.

S3는 파일 시스템으로 마운트될 수 없다.

## Amazon ECS - 오토 스케일링

ECS Task를 자동으로 늘릴 수 있다.

세 개 지표에 대해 확장 가능하다.

    - CPU 사용량
    - 메모리 사용률 - RAM
    - ALB Request Count Per Target

Target Tracking - 대상 추적 스케일링
Step Scaling
Scheduled Scaling

ECS Scaling과 EC2 Scaling은 다르다.

Fargate를 사용하는게 오토 스케일링에 편한다. (서버리스니까)

EC2 intance type에서는
EC2를 스케일링하기 위해서 ASG를 사용한다.

- ASG Scaling
- ECS Cluster Capacity Provider : 새 테스크로 용량이 부족하면 자동으로 AG한다 -> 좋다.

## Amazon ECS 솔루션 아키텍트

ECS tasks invoked by Event Bridge

클라이언트에서 S3로 객체를 업로드하고, 이걸 Amazon Event Bridge가 이벤트로 받아서, ECS Cluster 내에 Fargate ECS에 Task를 생성하라고 보낸다. ECS Task는 생성되고, S3 bucket에서 객체를 가져와서 처리 후에 ECS Task Role에 따라서 Dynamo DB에 결과를 저장한다.

-> 이미지나 객체 처리용 서버리스 아키텍쳐

ECS tasks invoked by Event Bridge Schedule

Event Bridge에서 1시간 마다 ECS Task run 신호를 보낸다.
그러면 ECS Cluster 내에 Fargate형 ECS는 Task를 생성해낸다.
그 두에 S3에 배치 데이터를 보낸 수도 있다.

ECS - SQS Queue Example

SQS에서 큐가 쌍이고, 이 데이터가 오면, ECS Autosaling이 보면서 리소스가 부족하면 채워준다.

ECS - Intercpet Stopped Tasks using Event Bridge

ECS Task에서 Task가 사라질 때, Event Bridge에 이벤트 트리거를 보내고, SNS를 거쳐 유저에게 이메일로 확인이 갈 수 있다.

## Amazon ECR

도커 이미지를 저장하고 관리

퍼블릭, 프라이빗 옵션이 있다.

ECS는 ECR과 S3를 뒤에 두고 완전히 합쳐져 있다.
ECS Cluster에서 ECR를 가져오고 싶으면, EC2 istance에 IAM role을 지정하면 된다.
ECR에 대한 모든 접근은 IAM이 보호하고 있다.

ECR은 이미지의 취약점, 스캐닝, 버저닝, 테그 및 수명 주기 확인을 지원한다.

## EKS 개요

관리형 쿠버네티스 클러스터를 실행할 수 있다.

쿠버네티스는 오픈소스로 도커로 컨테이너화한 앱의 자동 배포, 확장, 관리를 지원한다.
ECS와 유사하지만 API가 다르다.

Launch mode

- EC2 : worker node
- Fargate : serverless container

use case : 온프레미스에서 쿠버네티스를 사용 중일 때

쿠버네티스는 Cloud-agnostic으로 에져나 GCP에서도 지원된다.

쿨라우드 혹은 컨테이너간 마이그레이션 시 좋은 옵션이 될 수 있다.

Pods를 발견하면 쿠버네티스와 관련된 것 이다.

Node Type

- Managed Node Groups : Node(EC2)를 생성하고 관리한다, 노드는 EKS 서비스로 관리되는 ASG의 일부이다. 온디맨드, 스팟인스턴스를 지원한다.

- Self-Managed Nodes : 노드를 직접 생성하고, EKS 클러스터에 등록하고 ASG로 관리해야 한다. 기 제작된 AMI를 사용하면 시간 단축 가능, 온디맨드, 스팟인스턴스를 지원한다.

- AWS Fargate : 관리 필요 없음

Data Volumns

- EKS 클러스터에 Storage Class manifest를 지정해야한다.
- CSI (Container Storage Interface)를 활용한다.

## AWS App Runner - 개요

완전 관리형 서비스며 웹앱 배포를 쉽게 해준다.

어떤 인프라 경험도 필요없다.

소스코드나 도커 컨데이너 이미지로 원하는 구성을 만든다.
웹앱이나 API가 들어갈 곳을 설정하는 것이다.
그 다음에는 자동으로 된다.

컨테이너가 생성되고 배포된다.
이후에는 URL로 바로 엑세스할 수 있다.

장점 : ASG, 고가용성, LB, 암호화 , VPC에도 엑세스 할 수 있어서 DB, cache, queue 서비스에도 연결 가능하다.

빨리 배포해야 하는 상황에 적합하다.
