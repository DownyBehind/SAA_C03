# 고가용성 및 스케일링성 : ELB and ASG

## 고가용성 및 스케일링성

Vertical Scalability : incresing the size of the instance
ex. t2.micro -> t2.large

Horizontal Scalability : the number of instances / systems for your application
ex. ditributed systems

High Availability : 데이터 센터의 손실에서 살아남는 것. AZ를 분산시키는 것

## Elastic Load Balancing(ELB) 개요

Load Balancer : 트래픽을 여러 서버로 내리는 것을 의미한다.

부하를 다수의 다운스트림의 인스턴스로 보내기 위해서 사용한다.

- Spread load across multiple downstream instances
- Expose a single point of access(DNS) to your application
- Seamlessly handle failures of downstream instances
- Do regular health checks to your instances
- Provide SSL termination (HTTPS) for your websites
- Enforce stickiness with cookies
- High availability across zones
- Separate public traffic from private traffic

* managed Load balancer

- AWS가 관리하며 동작을 보장한다. 이걸 사용하는 것을 추천한다.

Health Checks

- ELB가 EC2의 상태를 체크한다. 이 체크는 포트와 라우트에서 체크한다.
- 만약 EC2가 200 응답을 하지 않으면 상태가 안좋다고 판단하고 트레픽을 보내주지 않는다.

CLB(Classic Load Balancer) - AWS에서는 사용을 권장하지 않는다. 구형이다.
: HTTP, HTTPS, TCP, SSL (secure TCP)
ALB(Application Load Balancer) - 2016년 춢시
: HTTP, HTTPS, WebSocket
NLB(Network Load Balancer) - 2017년 출시
: TCP, TLS (secure TCP), UDP
GWLB(Gateway Load Balancer) - 2020년 출시
: Operations at layer 3(Network layer) - IP Protocol

-> 더 많은 기능을 가지고 있는 최신 밸런서를 사용하는 것을 추천한다.
-> 몇몇 로드 벨런서는 internal(private) 또는 external(pubulic) ELB로 설정이 가능하다.

유저는 HTTP, HTTPS로 ELB에 접근이 가능한데, EC2 입장에서는 ELB의 트래픽만 허용해야하므로 소스에 IP 범위가 아니라 ELB의 보안그룹이 설정되어야 한다.

## CLB

Classic Load Balancer는 이제 AWS에서 지원되지 않는다.

## ALB(Application Load Balancer) 개요

- Application Load Balancer is Layer 7 (HTTP)
- Load balancing to multiple HTTP applications across machines (target groups)
- Support for HTTP/2 and WebSocket
- Support redirects (from HTTP to HTTPS for example)

- Routing tables to different target groups:

  - Routing based on path in URL
  - Routing based on hostname in URL
  - Routing based on Query String, Headers

- ALB are a great fit for micro services & container-based application
- Has a port mapping feature to redirect to dynamic port in ECS

Target Group이란

- EC2 instances (can be managed by an Auto Scaling Group) - HTTP
- ECS tasks
- Lambda functions
- IP Address

어플리케이션 로드 벨런서를 사용하는 경우에도 고정된 호스트 이름을 갖게된다.
(XXX.region.elb.amazonaws.com)

그리고 어플리케이션 서버는 클라이언트의 IP를 직접 보지 못한다.
클라이언트의 실제 IP는 X-Forwarded-For라는 헤더에 삽입된다.
따라서 Port(X-Forwarded-Port)와 Proto(X-Forwarded-Proto)도 갖게 된다.

Client IP를 통해 Client는 로드 밸런서와 통신을 하고, Connection Termination 후에
로드 벨런서는 Private IP를 통해 EC2 Instance와 통신을 한다.
만약에 어플리케이션 서버에서 클라이언트의 IP를 확인하고 싶다면, HTTP 헤더 내의 Port와 Proto를 확인해야 한다.

## NLB(Network Load Balancer)

Layer 4 Load Balancer로 Layer 7을 다루는 ALB보다 하위계층을 다룬다.
TCP & UDP 트래픽을 인스턴스로 보낸다.

성능이 매우 높다. 초당 수백만 건의 요청을 처리할 수 있고 ALB보다 지연시간도 짧다.
ALB 400ms
NLB 100ms

NLB는 AZ 별로 1개의 고정 IP를 갖고 Elastic IP 할당을 지원할 수 있다.
여러 개의 고정 IP를 가진 앱을 지원할 때 좋다.

고성능, TCP, UDP, static ip가 나오면 NLB를 생각하자.
작동 방식은 ALB와 유사하다.

외부로부터는 TCP, UDP를 받고, 백엔드 앱 instance에게는 TCP 혹은 HTTP로 통신

대상 그룹

- EC2 instance
- IP Addresses - must be private IPs (온프레미스 서버 포함 할 때)
- ALB 앞에 사용 가능 - NLB 덕에 고정 IP를 얻고, ALB를 통해 HTTP를 처리할 수 있다 .
- Health check는 TCP, HTTP, HTTPS를 지원한다.

## GWLB(Gateway Load Balancer)

Deploy, scale and manage a fleet of 3rd party network virtual appliances in AWS

모든 트래픽이 방화벽이나 침입 방지 시스템을 통과하게 한다.
트래픽이 어플리케이션에 도달하기 전에 Route Table의 값을 통해 GWLB로 가고,
여기서 3rd Party Security Virtual Appliances로 구성된 Target Group을 통해 트래픽을 분석하고 처리한다. [방화벽, 침투감지]

이상이 없으면 GWLB로 다시 보내고, 해당 트래픽을 앱으로 보낸다.

Layer 3(Network Layer) 즉, IP Packets 레벨에서 동작한다.

- Transparent Network Gateway - single entry/exit for all traffic
- Load Balanser - distributes traffic to your virtual appliances

Uses the GENEVE Protocol on port 6081

대상 그룹

- EC2 instance
- IP address

## Elasitc Load Balancer - Sticky Sessions
