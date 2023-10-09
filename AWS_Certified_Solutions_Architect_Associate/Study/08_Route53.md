# Route 53

## DNS란 무엇일까요

DNS (Domain Name Service) : IP 주소를 사람에게 익숙한 주소로 변환해준다.
DNS is the backbone of the internet
DNS는 계층 구조이다. ~.com, ~.kr 등등

DNS 용어

Domain Register (도메인 이름을 등록하는 곳) : Amazon Route 53, GoDaddy, ...
DNS Records : A, AAAA, CNAME, NS, ...
Zone File : contains DNS records
Name Server : resolves DNS queries (Authoritative or Non-Authoritative) : DNS 쿼리를 실제로 해결함
Top Level Domain (TLD) : .com, .us, .in, .gov, .org ...
Second Level Domain (SLD) : amazon.com, google.com

FQDN : Fully Qualified Domain Name -> 전체 주소

How DNS Works

IP 9.10.11.1로 운영되며, example.com의 DNS를 가지는 웹서버가 있다고 하자.

유저가 해당 주소를 입력해서 요청하면, 최초로 Local DNS Server에 물어보게 된다.

Local DNS Server는 Root DNS Server (managed by ICANN)에 물어보게 되는데,

Root DNS Server는 해당 주소는 모르지만 최상단 도메인인 .com은 알고 있으며, 해당 내용을 NS 1.2.3.4 에 문의하라고 알려준다.

다시 Local DNS Server는 해당 내용을 TLD DNS Server(.com managed by IANA)에 물어보고,

TLD DNS Server는 정확히는 모르지만 example.com은 NS 5.6.7.8에 문의하라고 한다.

다시 Local DNS Server는 SLD DNS Server (managed by Domain Register ex. Amazon Route 53...)에게 물어보고, 여기에는 해당 주소로 등록된 IP가 있으므로 해당 내용을 반환하게 된다.

그리고 Local DNS Server는 해당 주소를 캐시에 저장하며, 누가 물어보면 바로 답할 수 있게 구성한다.

## Route 53

A highly available, scalable, fully managed and Authoritative DNS

- Authoritative = 사용자가 DNS에 대해서 완전히 제어가능함

Records

How you want to route traffic for a domain

- Each record contains :

  - Domain/subdomain Name : ex. example.com
  - Record Type : ex. A or AAAA
  - Value : ex. 123.456.789.123
  - Routing Policy : how Route 53 responds to queries
  - TTL(Time to Leave) : amount of time the record cached at DNS Resolvers

- Route 53 supports the following DNS record types
  - (must know) A / AAAA / CNAME / NS
  - (advanced) CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

Record Types

- A - maps a hostname to IPV4
- AAAA - maps a hostname to IPV6
- CNAME - maps a hostname to another hostname
  - The target is a domain which must have an A or AAAA record
  - Can't create a CNAME record for the top node of a DNS namespace (Zone Apex)
- NS - Name Servers for the Hosted Zone
  - Control how traffic is routed for a domain

Hosted Zone

A container for records that define how to route traffic to a domain and its subdomains

- Public Hosted Zones : contains records that specify how to route traffic on the Internet (public domain names)

- Private Hosted Zones : contains records that specify how you route traffic within one or more VPCs (private domain names) : 사기엡에서 회사 내부망에서만 접근 가능한 URL 등등

You pay $0.50 per month per hosted zone

프라이빗 DNS는 VPC 내부에서만 Resolve 할 수 있다.

## Records TTL (Time To Live)

클라이언트가 Amazon Route53에게 DNS 요청을 보내면, Amazon Route 53은 IP와 함께 TTL을 보낸다.
TTL은 300초라고 하면, 300초동안 클라이언트로 하여금 결과를 캐시하게 한다. (해당 시간이 만료되어야 재 요청함)

이렇게 클라이언트가 해당 값을 캐시하면, 이후에 클라이언트가 DNS register에게 요청보낼 필요없이 해당 값을 보고 이동해도 된다는 말이다.

1. high TTL - 24hr

   - Less Traffic on Route 53
   - Possibly outdated records

2. Low TTL - 60s
   - More traffic on Route 53($$)
   - Records are outdated for less time
   - Easy to change records

## Route 53 CNAME vs Alias

AWS 리소스(로드 밸런서, 클라우드프론트...)는 AWS hostname을 노출한다.
그리고 보유한 도메인에 호스트 이름을 매핑하고 싶을 수 있다.

CNAME : 호스트 이름을 다른 호스트 네임으로 향하게 할 수 잇다.
이건 루트 도메인이 아닌 경우에만 가능해서, (mydomain.com은 안되고, something.mydomain.com은 가능하다.)

Alias : Route 53에 한정되지만 hostname이 특정 AWS 리소스로 향하도록 할 수 있다.
이건 루트 도메인이든 아니든 다 작동한다.

- 무료이다.
- 상태확인이 가능하다.

Alias Records

- Maps a hostname to an AWS resource
- An extension to DNS functionality
- Automatically recognizes changes in the resource's IP addresses
- Unlike CNAME, it can be used for the top node of a DNS namespace (Zone Apex) ex. example.com
- Alias Record is always of type A / AAAA for AWS recource (IPv4 / IPv6)
- Alias를 설정하면, TTL을 설정할 수 없다. 자동으로 설정된다.

Alias Records Targets

- ELB
- CloudFront Distributions
- API Gateway
- Elastic Beanstalk environments
- S3 Websites
- VPC Interface Endpoints
- Global Accelerator
- Route 53 record in the same hosted zone

* EC2 DNS 이름에 대한 별칭은 설정할 수 없다.

## Route 53 - Routing Policies

라우팅 정책은 라우트가 DNS 쿼리에 응답하는 것을 돕니다.
DNS는 트래픽을 라우팅하는게 아니라 DNS 쿼리에 응답한다.

- Simple

  - 일반적으로 트래픽을 단일 리소스로 보내는 방식
  - 클라이언트가 foo.example.com으로 가고자 하면 11.22.33.44로 응답해주는 식이다.
  - 같은 레코드에 여러 값을 할당하는 것도 가능한데, 이럴 경우 클라이언트는 여러 답을 받게 되고 이 중 하나를 고르게 된다.
  - 별칭을 활성화 하면 오로지 하나의 AWS 리소스만 연결할 수 있다.

- Weighted

  - 이 정책을 사용하면, 요청의 %를 제어하여 특정 리소스로 보내는 식이 가능하다.
  - 각 인스턴스에 가중치를 지정하면, 인스턴스 별로 가중치에 따라 라우팅된다.
  - 해당 레코드 가중치 / 전체 레코드 가중치 = 트레픽
  - 합은 100이 아니어도 된다.
  - 이렇게 하려면 DNS record들은 전부 같은 이름에 같은 타입이어야 한다.
  - 상태확인과 관련 될 수 있다.
  - 사용되는 사례는 명확한데 서로 다른 지역에 걸쳐 로드 밸런싱을 할 때나, 작은 양의 트래픽을 보내서 테스트 하는 경우이다.
  - 가중치가 0이면 해당 리소스로 트래픽을 안 보낼 수도 있고, 전부 0이면 다시 동일한 가중치를 가지게 된다.

- Failover

  - Route 53과 두 개의 instance가 있는데, 하나는 Primary instance이고, 다른 하나는 Secondary - Disaster Recovery instnace이다.
  - 이 구성에서 Health Check는 필수이며, 만약에 상태 확인 결과가 비정상이면, 보조 인스턴스로 요청을 변경한다.

- Latency based

  - 지연시간이 가장 짧은 가장 가까운 리소스로 리다이렉팅하는 정책
  - 지연시간에 민감한 앱의 경우 유용
  - 지연시간은 유저와 AWS 리전간의 트래픽을 기반으로 측정된다.

- Geolocation

  - Different from Latency-based
  - This routing is based on user location
  - 잂치하는 지역이 없으면 "Default"로 설정해야 한다.
  - 특정 지역에 대해서 특정 인스턴스로 접근하게 구성

- Multi-Value Answer

  - 트래픽을 다중 리소스로 응답할 때 사용한다.
  - 정상 상태를 확인하고 반환합ㄴ다.
  - 각 멀티 값 쿼리에 대해서 8개까지의 건강한 레코드를 반환할 수 있다.
  - 멀티 값 응답은 ELB를 대체할 수 없다. 어디까지나 클라이언트 레벨의 동작이기 때문이다.

- Geoprocimity (using Route 53 Traffic Flow feature) 지리적 근접성

  - 사용자와 리소스의 지리적 위치를 기반으로 트래픽을 리소스로 라우팅하도록 한다.
  - 정의된 바이어스값을 기반으로 리소스로 향하는 트레픽을 조정할 수 있다.
  - 특정 리소스에 트래픽을 더 보내려면 편향값을 올리고(1 ~ 99), 줄이고 싶으면 음수를 입력하면 된다. (-1 ~ -99)

  - AWS 리소스, Non-AWS 리소스 (위도와 경도 지정 필요)
  - Route 53 Traffic Flow를 사용해야한다.

  - 지리적 편향은 한 리전에서 다른 리전으로 트래픽을 보내는데 유용하다.

- Health Checks

  - 공용 리소스에 대한 상태를 체크 하는 것이다.
  - 예를 들어 고가용성을 위해 각 리전에 ALB를 이용한 앱을 구축한다고 하자. 만약 특정 리전이 고장 상황이라면 해당 리전으로 DNS 라우트를 하면 안될 것이다. 이를 위해 상태 체크를 하면, DNS의 장애 조치를 자동화하기 위한 작업이다.
    - 공용 엔드포인트 체크
    - 사전에 연산된 부분 체크 (Health checks that monitor other health checks)
    - CloudWatch 경보의 상태를 모니터링

  Monitor an Endpoint

  - 15개의 글로벌 헬스 체커(다른 리전에서부터 온)가 엔드 포인트 상태 확인
    - 상태 양호/불량에 대한 임계값은 3(Default)
    - 30초 간격으로 체크 (10초 간격으로 할 수 있으나 비싸다)
    - 지원 프로토콜 : HTTP, HTTPs, TCP
    - 18% 이상의 헬스체커가 정상이라고 판단하면 Route 53도 정상으로 판단한다.
    - 네트워크 관점에서 이 기능을 사용하려면 ALB나 방화벽이 Route 53 헬스체커에서 해당 엔드포인트에 접근할 수 있게 해야한다.

Calculated Health Checks

- 계산된 상태 환익으로 여러 상태 결과를 하나로 합쳐주는 기능이다.
- 각각 인스턴스의 상태를 기반으로 상위 상태를 정의한다.
- 상태를 결합할 때, OR, AND, NOT으로 결합할 수 있다.
- 최대 256개의 상태를 기반으로 할 수 있다.
- 몇 개 이상의 자식 상태 체크가 통과해야 부모 상태 체크를 생성할 수 있는지를 정할 수 있다.

Private Hosted Zones

- 개인 리소스를 체크하는 것은 어려울 수 있는데, 왜냐면 Route 53의 상태확인이 VPC 외부에 있기 때문이다.
- 따라서 private endpoints에 접근할 수 없다.
- 따라서 CloudWatch 지표를 만들어서 CloudWatch Alarm을 할당하는 식으로 문제를 해결할 수 있다.

IP Based Routing

- You provide a list of CIDRs for your clients
  and the correspondin endpoints/locations (user-IP-to-endpoint mappings)
- Use cases : Optimize performanca, reduce network costs

예를 들어 인터넷공급업체가 IP 주소셋을 알고 있다면 특정 엔드포인트로 라우팅하도록 할 수 있다.
즉, 사용자의 IP 주소 범위에 따라서 라우팅 될 백엔드 서버가 결정되는 것이다.

## 도메인 레지스터 vs DNS 서비스

- 도메인을 등록하면 비용을 내야 한다.
- 도메인을 구매하면 보통 관리 서비스도 지원해준다.
- 하지만 DNS 서비스로 다른 DNS 레코드를 관리할 수도 있다.
- 가령 타사에서 도매인을 구입하고, 그 도메인 레코드를 Route 53으로 관리할 수도 있다.

방법은 타사에서 도메인 레코드르 생성할 때 네임서버를 Amazon Route 53의 Public Hosted Zone의 서버주소로 변경해줘야 한다.
