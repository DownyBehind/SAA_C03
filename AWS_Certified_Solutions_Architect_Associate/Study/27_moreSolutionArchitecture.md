# 더 많은 솔루션 아키텍쳐

## AWS의 이벤트 처리

Lambda, SNS & SQS

AWS에서 이벤트 처리에 대해서

SQS + Lambda

SQS에서 큐를 채우고 Lambda가 메시지를 polling하는데 만약에 문제가 발생하면,
해당 메시지를 다시 SQS 큐에 입력하고 polling을 다시 시도한다.

이러던 중 한 메시지에서 중대한 문제가 발생하여, 다섯 번의 재시도 후에는 DLQ(Dead Letter Queue)로 보내도록 SQS를 설정할 수 있다.

SQS FIFO + Lambda

순서대로 처리하기 때문에 하나라도 문제가 생기면, 차단이 발생하여 처리가 끝나지 않고 결국 전체 대기열 처리가 차단된다.

이 경우에도 DLQ로 해당 메시지를 빼고 함수가 동작하게 할 수 있다.

SNS + Lambda

메시지가 통과하고 메시지는 비동기식으로 Lambda에게 전달된다.
처리하지 못하는 메시지가 발생하더라도 내부적으로 재시도한다.

재시도는 3회까지만 하며 그래도 안되면 해당 메시지를 제거하거나 SQS의 DLQ로 보낸다.

Fan Out Pattern : deliver to multiple SQS

팬 아웃 패턴은 여러 SQS에 보내는 패턴이다.
3개의 SQS가 있으면 하나씩 동일한 메시지를 보낸다.
이 방식은 동작은 하지만 안정성이 높지는 않다.

SNS + SQS

동시에 같은 메시지를 큐에 채울 수 있다.
안정성이 높아진다.

S3 Event Notification

S3 버킷이 특정 이벤트에만 반응하도록 설정할 수 있어 객체 생성, 삭제 복원, 복제본 생성 시 알람을 보내도록 할 수 있고 객체 이름 별로 필터링도 가능하다.

사용 사례로는 S3에 업로드된 이미지의 썸네일을 생성하는 경우가 있다.

S3 이벤트를 SNS, SQS, Lambda 함수로 보내는데 이 때 S3 이벤트는 원하는 만큼 생성할 수 있다.

이벤트 알림은 보통 수 초내로 도착한다.

S3 Event Notifacations with Amazon EventBridge

S3에 일어난 모든 이벤트를 이벤트브릿지로 전송하는 방식
규칙을 설정하면 18개 이상의 서비스로 전달 가능

왜 이벤트브릿지를 쓸까?

Advanced filtering 옵션에 JSON rules를 사용해서
메타데이터, 객체 크기, 이름등을 필터링할 수 있기 때문이다.

Multiple Destinations : 여러 대상에 이벤트를 한번에 보낼 수 있다.

EventBridge Capabilities : Archive, Replay Evnets, Reliable delivery

Amazon EventBridge - Intercept API Calls

모든 API를 이벤트 브릿지에서 intercept하려면 CloudTrail을 활용하면 된다.

API Gateway - AWS Service Integration Kinesis Data Streams example

클라이언트가 요청을 보내면, 그 명령을 kinesis로 보내고 firehose로 간 다음 S3에 저장된다.

## AWS 캐싱 전략

Caching Strategies

클라우드프론트는 엣지에서 캐싱하는데, 사용자와 최대한 가까이에서 캐싱한다는 뜻이다.
캐싱을 허용하는 즉시, 캐시를 히트하는 모든 사용자는 즉시 데이터를 얻게 된다.

엣지에 있기 때문에 백엔드 변화를 바로 감지하지 못할 수도 있다.
TTL을 사용하여 이 부분을 업데이트할 수 있다.

API Gateway

이 서비스도 캐싱이 가능해서 클라우드 프론트와 함께 사용할 필요가 없다.
리전 서비스라서 캐싱을 사용 할 시 리전에 묶이게 된다.
그러니 클라이언트와 api gateway 사이에 네트워크 라인이 형성되어 캐시 히드된다.

EC2/Lambda

보통 캐싱을 하지 않는데 DynameDB, Redis, Memcached, DAX 등을 사용할 수 있는 캐시를 할 때 사용된다.

데이터베이스를 반복적으로 히트하길 원하지 않을 것이다.
디비는 캐싱을 하지 못하기 때문이다.

자주발생하는 쿼리나 결과는 공유 캐시에 저장되어 쉽게 엑세스 할 수 있게 하는 것이다.
따라서 이 캐싱을 통해 디비에 가해지는 압력을 줄이고 읽기 용량을 늘린다.

S3의 디비에는 캐싱능력이 없다.

## AWS에서 IP 주소 차단

이번에는 네트워크와 보안에 대한 내용이다.

클라이언트에게 요청받았을 때 앱까지 가는 과정을 제대로 이해해야 한다.

때로는 IP주소를 클라이언트로부터 막고 싶은데 말썽이 나기도 하고 따라서 방어선에 대해서 알아야 한다.

간단한 instance + public ip가 VPC 내부에 있고, NACL이 있다.

1. NACL이 첫번째 방어선

   - 차단 규칙을 만들 수 잇다.

2. instance 보안규칙
   - 차단 규친은 만들 수 없고, 허용 규칙만 가능하다.

선택적으로 Firewall을 EC2에 실행해 클라이언트 요청을 거절할 수도 있다.

Client ---- NACL ---- ALB ---- EC2

이제는 보안그룹이 2개이다
ALB, EC2

ALB는 connection termination 기능이 있는데 클라이언트에게 요청을 받고, 연결은 끊은 뒤 EC2와 연결을 하는 식이다.

EC2 보안그룹은 ALB를 허용해야 한다.
ALB 보안그룹은 클라이언트를 허용해야 한다.
NACL은 같다.

Client ---- NACL ---- NLB ---- EC2

NLB는 연결종료를 하지 않는다.
NLB는 보안그룹이 없기 때문에 사실상 트래픽이 지나간다.

그렇기 때문에 결국 방어선은 NACL이 되게 된다.

Client ---- NACL ---- ALB ---- EC2
ㄴ WAF

WAF는 조금 비쌀 수 있지만 복잡한 필터링이 가능하고 클라이언트로 동시에 많은 요청을 받지 않도록 할 수 있고 보안이 강해진다.

ALB 앞에 CloudFront를 사용하면 Cloud Front는 VPC 밖에 위치한다.
ALB는 엣지 로케이션으로 오는 CloudFront의 public IP를 허용해야 한다.
이때는 NACL은 아무런 도움이 되지 않는다 왜냐면 클라이언트 IP가 오지 않기 때무이다.

CloudFront가 막으려면 두 가지 방법이 있다.

1. 지리적 제한 기능 사용 - 국가 레벨
2. WAF를 사용해서 IP 필터링

## AWS의 고성능 컴퓨팅(HPC)

점점 많이 나온다.

클라우드는 고성능 컴퓨터를 실행하기 좋다
왜냐면 리소스를 즉각적으로 늘릴 수 있기 때문이고 시간을 단축하며 효율적이기 때문이다.

유체역학, 금융모델, 머신러닝 등ㄷ으...

Data Management & Transfer

- AWS Direct Connect

  - 안전한 망으로 GB/s 속도로 데이터 전송

- Snowball & Snowmobile

  - PB data 옯길 때 사용

- AWS DataSync
  - 대용량의 데이터를 전송한다.

Compute and Networking

- EC2 instance

  - CPU optimized, GPU Optimized
  - Spot instance / spot Fleets for cost saving + auto scaling

- EC2 Placement Groups: Cluster for good network performance

  - Same Rack, Same AZ
  - 같은 렉에 있을 때 레이턴시가 낮고 10Gbps 네트워크 사용가능

- EC2 Enhanced Networking(SR-IOV)

  - 더 넓은 대역폭
  - 낮은 지연시간
  - 1. ENA(Elastic Network Adaptor)로 100Gbps까지 속도 올리기
  - 2. Intel 82599 VF라는 걸 사용해 최대 10Gbps까지 빨라질 수 있다.

- Elastic Fabric Adaptor (EFA)
  - 고성능 컴퓨팅을 위한 개선된 ENA
  - 리눅스에서만 가능
  - 분산 계산에 적합
  - 안정적이고 지연시간이 짧다

Storage

- Instance-attached storage

  - EBS
  - Instance Store

- Network Storage
  - S3
  - EFS
  - FSx

Automation and Orchestration

- AWS Batch

  - 다중 노드 병렬 작업
  - 작업 예약으로 실행이 간편

- AWS Parallel Cluster
  - 오픈소스
  - 텍스트 파일을 구성해서 배포
  - 자동으로 VPC, subnet, cluster type, instance type 생성
  - 이 기능은 EFA와 함께 사용한다. (클러스터 상에서 EFA를 활성화하는 매개변수가 텍스트 파일에 있기 때문이다.)

## EC2 인스턴스 고가용성

가용성을 높이자

EC2는 AZ 하나에서 실행된다.

모니터링이 필수 있다.

CloudWatch Event로 알람을 만든다.

알람에 대한 응답으로 람다함수를 호출 가능하다.

람다함수로 대기 인스턴스를 실행시키고 elastic ip를 먹이게 할 수 있다.

ASG 사용하는 방법

최소값 최대값 희망값을 1로 2 AZ에 구성하게 한다.

하나가 사라지면 하나가 생길 것이다.

EC2 + ASG + EBS

EBS는 AZ에 고정되어 있다.

EBS snapshot을 생성하고 이 스냅샷을 이용해 새 AZ에 EBS를 재생성한다.
