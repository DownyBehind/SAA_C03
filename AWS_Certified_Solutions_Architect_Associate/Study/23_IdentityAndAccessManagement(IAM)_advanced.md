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
