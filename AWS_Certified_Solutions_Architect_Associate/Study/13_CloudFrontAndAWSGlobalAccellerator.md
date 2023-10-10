# Cloud Front 및 AWS 글로벌 엑셀러레이터

## CloudFront 개요

CDN(Content Delivery Network)을 보면 CloudFront를 떠올리자

- 서로 다른 엣지 로케이션에 미리 컨텐츠를 케싱하여 읽기 성능을 높이는 기술이다.
- 우리의 컨텐츠가 네트워크 전체에 캐싱되므로 전세계 사용자들이 낮은 latency로 접근할 수 있다.
- Cloud Front는 216개의 엣지 로케이션을 통해 구성되어 있다.
- 또한 컨텐츠가 분산되어 있어서 DDOS 공격에도 보호를 받을 수 있다.

## CloudFront - Origins

S3 bucket

- S3 버킷으로, CloudFront를 통해 파일을 분산하고 캐싱할 수 있게 한다.
- 버킷에는 OAC(Origin Access Control)을 통해 CloudFront가 접근할 수 있게 한다.
- OAI(Origin Access Identity)를 OAC가 대체한다.
- CloudFront를 통해서 버킷에 데이터를 보내는 것도 가능하다. 이를 Ingress라고 한다.

Custom Origin (HTTP)

- ALB, EC2instance, S3 website, HTTP Backend

CloudFront 동작 방법

- 클라이언트가 S3나 기타 HTTP 백엔드에 특정 파일을 요청하면 엣지 로케이션은 먼저 해당 데이터가 자신에게 있는지 확인 후, 없으면 Origin에 가서 해당 데이터를 가져와서 응답한다. 그리고 해당 데이터를 자신의 로컬 캐시에 저장하는데, 이후에 다른 클라이언트로부터 같은 컨테츠를 요청받으면 즉시 응답한다.

S3 as an Origin

- LA 엣지에 접근하는 사용자는 특정 컨텐츠를 요청하는데 LA 엣지 로케이션은 해당 데이터가 자신에게 없다면, Origin에 가서 데이터를 요청한다. 이 떄 OAC와 버킷 정책에 의거해서 데이터를 받게 되고 해당 데이터는 로컬 캐시에 저장한다. 다른 지역도 마찬가지이다. 서로 데이터는 내부망으로 이동한다.

- 이 기술로 특정 리전에 있는 컨텐츠를 전세계에 보낼 수 있다.

## CloudFront vs S3 Cross Region Replication(CRR)의 차이점은?

CloudFront

- 글로벌 엣지 네트워크 사용
- 파일은 TTL 안에 캐시됨 (하루 정도)
- 전세계 대상의 정적 컨텐츠에 적합하다.

Cross Region Replicaiton

- 복제본이 필요한 리전에 각각 설정해야함
- 파일은 실시간으로 업데이트된다.
- 캐싱되지 않고 읽기 전용이다.
- 이 기술은 일부 리전을 대상으로 동적 컨텐츠를 지연시간 최소화하여 제공할 때 사용

## Cloud Front - ALB of EC2 as an origin

EC2에 Cloud Front를 사용하려면, 우선 EC2를 Public으로 만들어야 하고,
Edge Location의 Public IP가 EC2에 접근할 수 있도록 보안그룹을 설정해야 한다.

ALB의 경우에도 공용 접근이 가능하게 설정해야 하고, 대신에 EC2는 프라이빗으로 구성해도 된다.
ALB와 EC2 사이에는 보안그룹으로 접근 가능하게 할 수 있기 때문이다.

## CloudFront - 지리적 제한 설정

- 접근 가능한 국가 목록, 불가능한 목록을 지정할 수 있다.
- 컨첸츠 저작권법으로 제한되는 국가를 지정할 수도 있다.

## CloudFront - Price Classes

전역에 퍼진 서비스이므로, 엣지 로케이션 마다 데이터 비용이 다르다.
더 많은 데이터가 보내질 수록 가격이 내려간다.

가격 등급

- Price Class All : all regions - best performance, high price
- Price Class 200 : most regions, but excludes the most expensive regions
- Price Class 100 : only the least expensive regions

## CloudFront - Cache Invalidation

CloudFront는 항상 백엔드 Origin이 있는데, Origin이 업데이트해도 업데이트 상황을 모를 수 있다.
TTL이 만료되면 업데이트된 콘테츠를 받게 된다.

하지만 최대한 빠르게 새 콘텐츠를 받고 싶을 수 있다.

이럴 경우, 강제로 전체 혹은 일부 캐시를 리프레쉬할 수 있는데 이게 Cache Invalidation이다.
Cache Invalidation을 할 때, 전체 파일 (_) 혹은 특정 경로 (/image//_)로 지정할 수 있다.

## AWS Global Accelerator

문제: 우리는 글로벌 서비스를 운영하고자 하는데 운영 서버 (Public ALB)는 현재 인도 리전에 존재한다. 그런데 유저는 미국, 유럽, 호주 등에 걸쳐 있고 이들이 우리 서비스에 도달하기 까지 공용 인터넷망의 수많은 라우터를 거쳐서 와야 한다. 이런 과정은 불안정하기도 하고 지연시간이 크기도 하다. 따라서 최대한 유저들이 빠르게 AWS network를 통하길 원한다.

- Unitcast IP : one server holds one IP address

- Anycast IP : all servers hold the same IP address and the client is routed to the nearest one

AWS Global Accelerator는 Anycast IP 방식을 이용하는데, 미국 유저는 통신하기 위해 공용 인터넷망을 사용하는 대신 미국의 엣지 로케이션과 통신하여 인도 리전에 있는 ALB에 연결한다.

엣지 로케이션과 ALB 사이는 프라이빗 네트워크이다.

사용자 앱에 2개의 Anycast IP가 생성된다.

Anycast IP는 엣지 로케이션으로 트래픽을 직접 전송한다.

Global Accelerator는 Elastic IP, EC2 instance, ALB, NLB, public or private과 함께 동작한다.

안정적인 성능을 보여준다.
아무것도 캐시 하지 않기 때문에 캐시 문제도 없다.

Health Check

- Global Accelerator는 앱의 health check를 한다.
- 한 리전에 있는 ALB가 고장이면 자동화된 장애조치가 1분 내에 정상 엔드포인트로 실행된다.
- 재해 복구에 뛰어나다.

- 보안측면에서
  - 2개의 external IP가 있기 때문에 뛰어나다
  - AWS Shield를 통해서 DDoS 공격도 방어가능하다.
