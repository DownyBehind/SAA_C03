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
