# Identity and Access Management(IAM) - 고급

## AWS Organizations 개요

- 글로벌 서비스
- 다수의 AWS 계정을 관리
- 메인 계징이 관리 계정이 된다.
- 다른 계정은 맴버 계정이 된다.
- 멤버 계정은 한 조직에만 속한다.
- 모든 계정의 비용을 통합할 수 있다.
- 모든 계정에 집계된 사용량에 기반한 할인을 받을 수 있다.
- 계정간 예약 인스턴스와 Saving Plans 할인이 공유된다.
- 누군가 예약 인스턴스를 사용하지 않을 경우, 다른 계정에서 그걸 사용할 수 있다.
- 할인이 조직 전체에 걸쳐 적용되므로 비용절감에 도움을 줄 수 있다.
- Organization 내에 계정 생성을 자동화할 수 있는 API가 있다.

Root Organizational Unit(OU)

- 전체 계정 외곽에 있는 개념

- Management Account

- OU(Dev) : 개발 계정 맴버 모임
- OU(Prod) : 운영 계정 맴버 모임

원하는 만큼 추가 할 수 있다.
OU 안에 OU 추가 가능

마치 조직도와 같이 꾸밀 수도 있다.

장점

- 다수의 계정을 가지므로, 다수의 VPC를 가진 단일 계정보다 보안에 뛰어나다.
- 결제 관련해서 유리하다.
- CloudTrail을 사용해서 모든 로그를 중앙 S3 계정으로 전송 가능
- CloudWatch로 로그 중앙 관리 가능
- 관리 계정에서 모든 계정 규칙 관리

보안: Service Contorl Policies(SCP)

- 특정 OU 혹은 계정 별로 IAM 정책을 제한할 수 있다.
- 관리 계정에는 적용되지 않는다.
- SCP는 기본적으로 아무것도 허용하지 않는데, 구체적인 허용항목을 설정해야 작동한다. (IAM과 마찬가지)

## IAM - 고급

IAM Conditions

```Bash
"Effect":"Deny",
"Action":"*",
"Resource":"*",
"Condition": {
    "NotIpAddress": {
        "aws:SourceID" : ["192.0.2.0/24", "203.0.113.0/24"]
    }
}
```

위와 같이 조건이 설정되면, 두 개의 IP 주소 범위에 포함되지 않는 주소는 거부하게 된다.
Client가 해당 주소 안에서 API를 호출하지 않으면 거부하겠다는 말이다.

예를 들어 회사 네트워크에서만 AWS 접근을 허용한다던지 하는 것이다.

```Bash
"Effect":"Deny",
"Action": ["ec2:*", "rds:*", "dynamodb:*"],
"Resource":"*",
"Condition": {
    "StringEquals": {
        "aws:RequestedRegion" : ["eu-central-1", "eu-west-1"]
    }
}
```

해당 리전의 ec2, rds, dynamodb api 호출을 제한한다.

이 조건을 조직SCP에 적요하면 좀 더 글로벌하게 적용할 수 있다.

```c
"Statement": [
    {
        "Effect": "Allow",
        "Action": ["s3:ListBucket"],
        "Resource": "arn:aws:s3:::test"
    }, // bucket level permission
    {
        "Effect": "Allow",
        "Action": [
            "s3:PutObject",
            "s3:GetObject",
            "s3:DeleteObject"
        ],
        "Resource": "arn:aws:s3:::test/*"
    } // object level permission
]
```

권한 접근 수준이 다르기 때문에 arn이 바뀐다.

```bash
aws:PrincipalOrgID
```

위 조건은 AWS Organizaions member 게정에만 리소스 정책이 적요되도록 제한한다.

조직 외부의 사람은 허용되지 않는다

## IAM - Resource-based Policies vs IAM Roles

Cross account :

- S3 버킷에 S3 정책을 적용하는 것처럼, 리소스에 리소스 기반 정책을 붙인다.

  - User A --- S3 Bucket Policy --- S3

- 또는 리소스에 접근할 수 있는 Role을 가진 Proxy 계정을 이용한다.

  - User A --- User B(S3 Access Role) --- S3

두 방법 모두 유효하지만 차이가 있다.

Role은 기존의 권한을 포기하고 해당 Role을 상속 받는다.

리소스 기반 정책은 Role을 맡은게 아니므로 권한을 포기할 핋요가 없다.

예를 들어, Account A가 다이나모디비를 스캔애서 AccountB의 S3 버켓에 넣고 싶을 때는 굳이 Role을 바꾸지 않고 리소스 기반 정책으로 권한을 얻어 처리하는게 더 낫다.

리소스 기반 정책을 지원하는 리소스가 늘어나고 있다.

Amazon EventBridge - Security

EventBridge Rule을 돌리리면, 대상에 대한 권한을 요구하는 규칙들이 있다.

- 리소스 기반 정책 : Lambda, SNS, SQS, CloudWatch, API Gateway

- IAM role : Kinesis stream, Systems Manager Run Command, ECS task...

IAM role을 EventBridge Rule에 추가해야만 접근이 가능하다.

이 내용을 숙지하자.

## IAM - 정책 평가 로직

IAM Permission Boundaries

- User, Role만 지원하고 Group은 지원하지 않는다.
- IAM 개체의 최대 권한을 정의하는 고급기능이다.

```c
{
    "statement":
    [
        {
            "Effect":"Allow",
            "Action": [
                "s3":"*",
                "cloudwatch":"*",
                "ec2":"*"
            ],
            "Resource": "*"
        }
    ]
}
```

IAM policy와 비슷하게 생겼다.
s3, cloudwatch, ec2를 모두 허용하고 있다.
이걸 IAM User에게 연결하면 Permission Boundary가 생기는 것이다.

이건 기준이므로 여기에 IAM Policy를 더하면 이제 IAM permission이 정의가 되는 것이다.

예를 들어 Permission Boundary 외에 리소스에 대한 IAM policy를 정의한 유저는 어떤 권한을 갖게 될까
아무것도 갖지 못한다.

IAM Permission Boundary는 Organizaion SCP와 함께 사용할 수 있다.
즉, IAM permission Boundary, Organizaion SCP, IAM policy이 3개가 겹친 부분이 effective permission이 된다.

IAM Policy Evaluation Logic

중간 중간 어떤 평가 요소가 있는지 이해하면 된다.

## AWS IAM identity Center

- 한 번 로그인하면 되는 서비스 (AWS Single Sign-On)
- AWS accounts in AWS Organizaions

  - Business cloud applications (e.g., Salesforce, Box, Microsoft 365, ...)
  - SAML2.0-enabled application
  - EC2 Windows Instances

- Identity providers - 자격증명이 저장되는 위치
  - Built-in identity store in IAM Identity Center
  - 3rd Party: Active Directory(AD), OneLogin, Okta,...

IAM Identity Center Fine-grained Permissions and Assignments
-> 로그인했다고 해서 권한까지 가지는건 아니다. 권한을 주는 절차를 거쳐야 한다.

- Multi-Account Permissions

  - AWS OS 내에 여러 계정의 Access를 관리할 수 있다.
  - Permission Sets
    - 사용자를 Group에 할당하는 하나 이상의 IAM policy를 정의한다.
    - AWS가 사용자에게 어떤 서비스에 접근가능한지 정의해준다.
    - 결국 사용자가 이 방법으로 로그인하고 특정 서비스에 접근하는 순간 이 권한 셋이 붙어서 권한을 갖게 된다.

- Application Assignments

- Attribute-Based Access Control (ABAC)

## AWS 디렉토리 서비스 - 개요

Microsoft Active Directory

- 어떤 윈도우 서버에서도 볼 수 있다.
- Database of object : 사용자 계정, 컴퓨터, 프린터, 파일공유, 보안 그룹이 객체가 될 수 있다.
- 중앙집중식 보안관리 시스템이며, 계정 생성, 권한 할당 등의 작업이 가능하다.

모든 객체는 tree로 구성되며, tree의 그룹을 forest라고 한다.

AWS Directory Services

1. AWS Managed Microsoft AD

   - AWS 내에 나만의 AD 생성, 유저를 로컬에서 관리할 수 있고, MFA를 지원
   - 사용자가 있는 온프레미스 AD와 신뢰관계를 구축할 수 있다.
   - 온프레미스와 AWS AD간에 사용자를 공유할 수 있다.

2. AD Connector

   - Directory Gateway(proxy)로 온프레미스 AD에 리다이렉트하며 MFA를 지원한다.
   - 사용자는 온프레미스에서만 관리된다.

3. Simple AD
   - AD-compatible managed directory on AWS
   - Microsoft 디렉토리를 사용하지 않으며 온프레미스 AD와도 결합하지 않는다.

IAM Identity Center - Active Directory Setup

- Connect to an AWS Managed Microsoft AD (Directory Service)

  - integration is out of box : IAM Identity Center를 AWS 관리형 MS AD에 통합하고 연결하도록 설정하면 된다.

- Connect to a Self-Managed Directory(온프레미스)

  - Create Two-way Trust Relationship usin AWS Managed MS AD : AWS 관리형 MS AD를 사용해 양방향 신뢰관계를 구축하는 것이다.
  - Create an AD Connector : IAM Identity Center와 연결한 뒤, 온프레미스와는 Proxy한다.

## AWS Control Tower

Easy way to set up and govern a secure and complaint multi-account AWS environment based on best practives

- AWS Control Tower는 Organization을 이용하여 계정을 자동 생성한다.

장점 :

- 클릭 몇 번으로 환경을 세팅하고 미리 구성할 수 있다.
- 가드레일로 정책을 관리할 수 있다.
- 정책 위반을 감시할 수 있다.

Guardrails

- Control Tower 내에 모든 계정에 관한 거버넌스를 얻을 수 있다.
- Preventive Guardrail - using SCPs(Service Contol Policies) : 계정을 무언가로부터 보호하는 것 - 모든 계정에 적용

- Detective Guardrail - using AWS Config : 규정을 준수하지 않는 것을 탐지하는 것 - 계정에 테그가 지정되지 않은 리소스를 식별한다던지 등등
