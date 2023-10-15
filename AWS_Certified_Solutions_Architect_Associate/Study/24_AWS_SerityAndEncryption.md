# AWS 보안 및 암호화 : KMS, SSM Parameter Store, CloudHSM, Sheild, WAF

# 암호화 101

암호화 매커니즘

Encryption in flight (SSL)

- 전송 중 암호화, 패킷을 암호화한다. 신용카드 정보 등
- 전송 전 암호화하고, 서버가 복호화해준다.
- 중간자 공격으로부터 안전하다.
- 인증서가 유효하고, HTTPS endpoint가 있다면 안전하다.

Server side encryption at rest

- 데이터가 서버에 수신된 후 암호화하는 것
- 서버에서 데이터를 저장할 때, 암호화해서 저장한다.
- 만약에 서버가 해킹당하더라도, 내부 내용이 안전하다.
- 클라이언트에 전송되기 전에는 복호화한다.
- 데이터 키라고하는 것 덕분에 데이터는 암호화된 형태로 저장된다.
- 암호화/복호화 키는 관리되고 서버가 접속할 수 있는 어딘가에 저장되어 있다. 보통 KMS에서 관리된다.

Client side encyption

- 클라이언트에서 데이터를 암호하하고 서버로 넘겨준다. 서버는 해당 데이터를 복호화할 수 없고, 클라이언트에 넘겨줘야 만 복호화할 수 있다.
- 데이터는 서버에 저장되지만 서버는 데이터를 알 수 없다.
- Envelop Ecryption

## KMS(Key Management Service) 개요

AWS에 암호화 이야기 나오면, KMS가 등장한다.

Fully integrated with IAM for authorization

데이터에 대한 엑세스를 쉽게 할 수 있다.
KMS를 사용할 경우, CloudTrail로 KMS key 사용을 쉽게 감사할 수 있다. API 이므로
KMS는 거의 모든 서비스에 편하게 사용할 수 있다.

절대로 secret를 평문에 저장하지 마세요, 특히 코드 안에 두면 안된다.
KMS는 API, SDK, CLI로도 사용이 가능하다.
암호화된 secret를 코드 상에 두고, key를 환경변수 등에 저장하는게 더 안전하다.

KMS Keys Types

예전에는 KMS Customer Master Key라고 불렀다.

Symmetric (AES-256 keys)

- Single encryption key that is used to Encrypt and Decrypt
- 키를 사용할 때, 우리는 키 자체에 접근할 수 없다.

Asymmetric (RSA & ECC key pairs)

- key가 2개 있다. private key(암호화), public key(복호화)
- 암호화/복호화 또는 서명/증명
- public key는 다운받을 수 있지만, private key는 접근할 수 없다.
- AWS 외부에서 KMS에 접근할 수 없는 사용자가 암호화하려는 경우에, 퍼블릭 키를 써서 데이터를 암호화하고 전송한다.

AWS KMS

- Types of KMS Keys

  - AWS Owned Keys (free): SSE-S3, SSE-SQS, SSE-DDB (default key)
    - S3 타입을 암호화하거나, SQS, DynamoDB를 암호화할 때 사용하는 키다.
  - AWS Managed Keys (free): aws/service-name, example: aws/rds or aws/ebs
    - 무료고 원하는대로 사용할 수 있지만, 키가 지정된 서비스 안에서만 사용할 수 있다.
  - Customer managed keys created in KMS $1 / month
  - Customer managed keys imported (must be symmetic key): $1 /month
  - pay for API call to KMS ($0.03 / 10000 calls)

Automatic Key rotation:

- AWS managed KMS Key : 자동으로 1년마다 순환된다.
- Customer managed key : 고객이 자동순환을 활성화해야만 한다.
- Imported KMS Key : 수작업으로만 순환을 시킬 수 있다.

Copying Snapshots across regions

- KMS key 범위는 리전이다.

특정 리전에 있는 키를 다른 리전으로 복사하려면 몇 가지 단계를 거쳐야 한다.

예를 들어 EBS 볼륨을 KMS key로 복사하고 있었다고하자.

1. EBS 볼륨의 스냅샷을 찍는다.
2. 암호화된 데이터를 스냅샷 찍으면, 그 스냅샷도 동일한 키로 암호화되어 있을 것이다.
3. 스냅샷을 다른 리전에 복사하기 위해 우리는 다른 KMS 키를 사용해서, 그 스냅샷을 다시 암호화 해야 한다. - 이 부분은 AWS에서 처리한다. (기존 키로 암호화된 데이터를 복호화하고 다른 키로 재암호화)
4. 다른 리전에 온 스냅샷에는 추가된 키로 암호화/복호화하게 된다.

KMS Key policies

- KMS key 접근 제어에 대한 키이고, S3 bucket policies와 유사하다.
- 키에 KMS 정책이 없다면, 아무도 접근할 수 없을 것이다.

1. Default KMS Key Policy: 아무런 특정한 커스텀 KMS 키 정책을 제공하지 않았을 때 생성된다.

   - 기본값은 계정에 있는 모든 사람이 이 키에 접근할 수 있도록 허용하는 것이다.
   - 만일 User or Role이 이 key에 엑세스하도록 허용하는 IAM 정책이 있다면 문제가 없게 된다.

2. Custom KMS Key Policy: KMS 키에 엑세스할 수 있는 user와 role을 정의하고, 누가 키를 관리할 수 있는지 정의한다. - KMS 키에 대한 교차 계정 엑세스를 하려는 경우에 유용하다.

Copying Snapshots across accounts

1. KMS Key로 암호화한 스냅샷을 찍는다.
2. KMS key policy에 cross-account access 권한을 붙인다.
3. 그리고 암호화된 스냅샷을 타겟 계정과 공유한다.
4. 타겟 계정 안에서 스냅샷의 사본을 생성하고, 다른 고객 관리형 키를 사용해서 그 타겟 계정 안에서 암호화한다.
5. 그러면 타겟 계정 안에서 스냅샷으로 볼륨을 생성할 수 있다.

## KMS - Multi-Region Keys

한 리전에 multi-region key를 생성하면, 다른 리전에 해당 키가 복제된다.
키 ID가 완전히 동일하다.

한 리전에 암호화하고 다른 리전에서 복호화할 수 있다.
자동 교체를 활성화하면 키 교체 시 다른 리전도 같이 교체된다.

- cross-Region API calls 시에 다른 키로 재 암호화 하지 않아도 된다.

KMS multi-region key가 글로벌이라는 것은 아니고(즉, 하나로 다같이 쓰는 개념이 아니고) 복제하는 것이다.
각 다중 리전 키는 자체 키 정책에 따라 독립적으로 관리된다.

따라서 특정 사례를 제외하고는 사용을 권하지 않는다.
KMS 키는 단일 리전에 제한되는 것이 더 좋기 때문이다.

use case : global client-side encryption, encryption on Global DyanmoDB, Global Auroa

DynamoDB Global Tables and KMS Multi Region Keys Client-side encryption

- Amazon DynamoDB Encryption Client를 사용해서 DynamoDB 테이블의 특정 속성을 암호화할 수 있다.
  즉, 데이터 자체는 암호하되어 있으므로, 테이블의 속성을 암호화해서 특정 클라이언트만 사용할 수 있게 한다. 이때 데이터베이스 관리자도 사용할 수 없게 된다.

Global Aurora and KMS Multi-Region Keys Client-Side enryption

- AWS Encryption SDK로 오로라 테이블의 특정 속성을 클라이언트 측에서 암호화할 수 있다.

## S3 암호화된 복제

한 리전에서 다른 리전으로 S3를 복사하게 되면, 기본적으로 암호화되지 않은 객체와 SSE-S3로 암호화된 객체가 복제된다.

SSE-C로 암호화하면 복제가 안되는데 이유는 키를 제공할 수 없기 때문이다.

SSE-KMS로 암호화하면 특정한 옵션을 활성화해야 하는데

- specify which KMS key to encrypt the object within the target bucket
- Adapt the KMS key policy for the target key
- An IAM Role with kms:Decrypt for the source KMS Key and kms:Encrypt for the target KMS Key
- You might get KMS throttling errors, in which case you can ask for a Service Quotas increase

* You can use multi-region AWS KMS Keys, but they are currently treated as independent keys by Amazon S3 (the object will still be decrypted and then encrypted)

## 암호화된 AMI 공유 프로세스

AMI은 KMS 키로 암호화되어 있다.

1. Must modify the image attribute to add a Launch Permission which corresponds to the specified target AWS account.

2. Must share the KMS keys used to encypted the snapshot the AMI references with the target account / IAM role

3. The IAM Role/User in the target account must have the permissions to DecribeKey, ReEncrypted, CreateGrant, Decrypt

4. When Launching an EC2 instance from the AMI, optionally the target account can specify a new KMS key in its own account to re-encrypt the volume.

## SSM Parameter Store 개요

구성 및 암호를 위한 보안 스토리지

Optional Seamless Encryption using KMS
SSM Parameter Store는 서버리스이며, 확장성이 있고, 견고하며 SDK 사용도 용이하다.
Configuration, secrets을 추적할 수 있다.

보안은 IAM을 통해서, 알람은 이벤트 브릿지로부터 받을 수 있다.

CloudFormation과도 통합이 된다.

Parameters Policies(for advanced parameters)

- Allow to assign a TTL to a parameter (expiration data) to force updating or deleting sensitive data such as passwords

- Can assign multiple policies at a time

## AWS Secrets Manager - 개요

암호를 저장하는 최신 서비스

SSM Parameter store와는 다른 서비스이다.

Secrets Manager는 X일 마다 강제로 암호를 교체하는 기능이 있어, 암호관리를 더 효율적으로 할 수 있다.
교체할 암호를 강제 생성 및 자동화할 수 있다.

이를 위해서 새 암호를 생성할 Lambda함수를 정의해야 한다.

그리고 다른 AWS service와도 잘 결합되는데 Amazon RDS (MySQL, PostgreSQL, Aurora)등이 있다.
즉, 데이터베이스에 접근할 수 있는 암호들이 AWS Secrets Manager에 바로 저장되고 교체도 가능하다는 말이다.

Secrets는 KMS를 통해서 암호화된다.

RDS와 Aurora의 통합 혹은 Secrets에 대한 내용이 시험에 나오면 AWS Secret Manager를 떠올리자

AWS Secrets Manager - Multi-Region Secrets

복수 AWS 리전에 암호를 복제할 수 있고, 이 서비스가 기본 암호와 동기화된 읽기 전용 복제본을 유지한다는 개념이다.

이렇게 하는 이유는 한 리전에서 문제가 생기면 암호 복제본을 standalone secret으로 승격시킬 수 있다.
그리고 여러 리전에 키가 복제되니 다중 리전 앱을 구축하고 재해 복구 전략도 짤 수 있다.

한 리전에서 다음 리전으로 복제되는 RDS 데이터베이스가 있다면, 동일한 암호로 동일한 RDS DB에 엑세스할 수 있다.

## AWS Certificate Manager (ACM)

ACM은 AWS에서 TLS 인증서를 프로비저닝, 관리 그리고 배포하게 해준다.

TLS/SSL 인증서는 웹사이트 전송 중 암호화를 제공하는데 쓰인다. (HTTPS)

ALB와 ASC는 로컬 네트워크이기 때문에 HTTP로 서로 데이터를 주고 받지만, ALB와 유저 사이에는 암호화가 필요하다. 따라서 ALB는 ACM에게 인증설르 받아서 유저에게 HTTPS 프로토콜을 제공한다.

ACM은 public, private TLS인증서를 모두 지원한다.

public TLS 인증서는 무료이다.

Integrations with (load TLS certifactes on)

- Elastic Load Balancers (CLB, ALB, NLB)
- CloudFront Distributions
- APIs on API Gateway

EC2에는 ACM을 사용할 수 없다.

ACM - Requesting Public Certificates

1. List domain names to be included in the certificate

   - Fully Qualified Domain Name (FQDN) corp.example.com
   - Wildcard Domain : \*.example.com

2. Select Validation Method: DNS Validation or Email validation

   - DNS validaton is preferred for automation purposes (자동화할 목적이라면...)
   - Email validation will send emails to contact addresses in the WHOIS database
   - DNS Validation will leverage a CNAME record to DNS config(ex. Route 53)

3. It will take a few hours to get verified
4. The Public Certificate will be enrolled for automatic renewal

ACM - Importing Public Certificates

- Option to generate the certificate outside of ACM and then import it
- 외부에서 가져온 인증서는 자동 갱신이 불가하다.
- ACM은 인증서 만료 45일 전부터 이벤트브릿지로 만료 이벤트를 전송해준다. -> 언제부터 알려줄지는 유저가 조정할 수 있다.

- AWS Config를 이용할 수 있는데 acm-cerficate-expiration-check를 사용해서 만료여부를 체크할 수 있다. AWS Config가 ACM을 감시

ACM - Integration with ALB

- ALB에는 HTTP를 HTTPS로 리다이렉트할 수 있는 규칙이 있다.
- 유저가 만약에 ALB에 HTTP로 접근해도 ALB가 유저에게 HTTPS 리다이렉트 요청을 보내는 것이다.
- 이후에는 유저는 HTTPS로 접근하여 ACM의 TLS 인증서를 사용한다.

API Gateway - Endpoint Types

Edge-Optimized (default) : For global clients

Regional : for clietns withih the same region

Private : Can only be accessed from your VPC using an interface VPC endpoint(ENI)

ACM은 엣지최적화와 리전 엔드포인트에 적합하다.

ACM - Integration with API Gateway

- create a Custom Domain Name in API Gateway
- Edge-Optimized (default) : For global clients

  - Requests are routed through the CloudFront Edge locations (improves latency)
  - The API Gateway still lives in only one region
  - The TLS Certificate must be in the same region as CloudFront, in us-east-1
  - Then setup CNAME or (better) A-Alias record in Route 53

- Regiona
  - For clients within the same region
  - The TLS Certificate must be imported on API Gateway, in the same region as the API Stage
  - Then Setup CNAME or (better) A-Alias record in Route 53

## 웹 어플리케이션 방화벽(Web Application FireWall - WAF)

WAF는 layer7에서 일어나는 공격으로부터 웹앱을 보호한다.

Layer 7 is HTTP (TLS/UDP is Layer 4)

WAF는 ALB, API Gateway, CloudFront, AppSync GraphQL API, Cognito User Pool에 배포할 수 있다. -> 기억할 것

이렇게 서비스에 WAF를 배포하고 나서 Web ACL(Web Access Control List)와 Rule을 정의해야 한다.

- 이 Rule이란 IP 주소를 기반으로 필터링하는 등의 규칙이다.
- IP set은 최대 10,000개의 IP 주소를 가질 수 있다.
- HTTP headers, HTTP body,or URI strings protects from common attack - SQL injection and Cross-Site Scripting(XSS)
- Size constraints, geo-match (block countries)
- Rate-based rules (to count occerences of events) - for DDoS protection

Web ACL은 리전 단위로 적용되는데 CloudFront만 글로벌로 정의된다.

Rule group이 있는데 여러 웹 ACL에 추가할 수 있는 재사용 가능한 규칙모음이다.

사례

WAF - Fixed IP while using WAP with a Load Balancer

- WAF는 NLB를 지원하지 않는다.
- ALB가 있어야 한다. 하지만 ALB는 고정 IP가 없다.
- 이런 경우, Global Accelerator로 고정 IP를 할당받은 다음 ALB에서 WAF를 활성화 시키면 해결할 수 있다.

## shield - DDos 보호

디도스 공격을 막기 위한 서비스

인프라로 동시에 많은 트래픽을 보내 서비스를 이용할 수 없게 만드는 공격

AWS Shield Standard Service : 이 서비스는 무료이다.

- SYN/UDP Floods 방어

AWS Shield Advanced : organization 당 한 달에 3,000달러 과금

- DDoS 완화 서비스
- 더 정교한 공격 방어
- 24/7 도움 받을 수 있음

## Firewall Manager

- Organization의 모든 계정에 대한 방화벽 규칙을 관리

- Security policy: common set of security rules
  - WAF rules (ALB, API Gateway, CloudFront)
  - AWS Shield Advanced Rule (ALB, CLB, NLB, Elastic IP, CloudFront)
  - Security Groups for EC2, ALB and ENI resource in VPC
  - AWS Network Firewall (VPC Level)
  - Amazon Route 53 Resolver DNS Firewall

즉, 방화벽과 관련된 규칙을 전부 한 곳에서 관리하는 것이다.

정책은 리전 레벨로 정해지고, 조직에 등록된 모든 계정에 반영된다.

WAF vs Firewall Manager vs Shield

3 서비스는 모두 포괄적인 보호를 위한 서비스이다.

- WAF에서는 Web ACL을 정의
- 리소스 별 보호를 구성하는데는 WAF가 적합하다.
- 하지만 여러 서비스에서 WAF를 사용하고 WAF 설정을 빠르게 하고 새 리소스 구성 시 자동으로 적용하고자 하면 Firewall Manager가 적합하다.
- 쉴드 어드벤스드는 DDos로부터 보호해준다.

## DDos Protection Best Practices

<슬라이드 참조>

## Amazon GuardDuty

지능형 위험탐지를 이용하여 AWS 계정을 보호할 수 있다.

- 이 서비스 내에는 머신러닝 알고리즘이 있어서 이상 탐지를 수행하고, 서드 파티 데이터를 사용하여 위험을 탐지한다.
- 원 클릭으로 실행이 가능하고 (30일 무료 체험), 뭔가 설치할 필요가 없다.

- 이 서비스는 CloudTrail Event log와 같은 입력 데이터를 확인해서 비정상동작을 검색
- VPC Flow log를 읽어서 비정상적인 IP 주소 검색
- DNS Logs - DNS 쿼리 내에 EC2 intance에 보내는 데이터 확인
- 옵션 기능이 있다.
- 이 서비스는 암호화폐 공격을 방어하기 위한 좋은 도구이다.

## Amazon Inspector

자동으로 감사를 하는 시스템

EC2 instance에 SSM(AWS System Manager) agnet를 사용하면 이 서비스가 EC2의 보안을 평가하기 시작한다.

연속적으로 수행된다.

또한 Container image를 ECR로 푸시할 때, 실행된다.
함다 함수가 배포될 때, 함수 코드 및 패키지 종속성을 확인한다.

검사가 끝나면 보안 허브에 보고 한다.
이 결과 이벤트를 이벤트 브릿지로 보낸다.

인프라에 있는 취약점을 보아서 볼 수 있다.

뭘 평가하는 걸까?

실행 중인 EC2, ECR 이미지, Lambda함수에만 사용되며, 필요할 때 인프라만 지속적으로 스캔한다.
패키지 취약성과 네트워크 도달성을 살펴본다.

실행될 때마다, 우선순위를 정하기 위해 위험 점수가 모든 취약성과 다시 연관된다.

## Amazon Macie

완벽히 관리되는 데이터 보안 및 데이터 프라이버시 서비스

머신러닝과 패턴 매칭을 이용해서 AWS에 있는 민감한 데이터를 발경하고 보호한다.

개인식별정보, PII같은 민감정보에 경보를 제공한다.

S3 내에 PII가 있으면 이 서비스에 의해 분석될 것이다.

이벤트브릿지를 통해 발결 결과를 알려주고 그걸 SNS 토픽이나 람다함수와 결함한다.
