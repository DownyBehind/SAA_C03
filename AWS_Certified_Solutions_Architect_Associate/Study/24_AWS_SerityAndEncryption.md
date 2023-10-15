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

## AWS Secrets Manager -개요
