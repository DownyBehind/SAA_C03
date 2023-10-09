# 아마존 S3 보안

Object Encryption

4가지 방법이 있다.

1.  Server Side Encryption (SSE)

    - SSE-S3는 Amazon S3에서 관리하는 키를 이용한 서버 측 암호화 - Default
    - SSE-KMS는 KMS 키를 이용해서 암호화 키를 관리함
    - SSE-C는 고객이 제공한 키를 사용하는 암호화 방법

2.  Client-Side Encryption

SSE-S3

- AWS가 관리하고 처리한 키로 암호화하고 우리는 키에 접근할 수 없다.
- Encryption type : AWS-256
- header는 "x-amz-server-side-encryption" : "AES256"으로 할당해야 한다.
- 기본값을 활성화되어 있다.
- 유저가 올릴 때 헤더에 저 값을 넣어서 올리면 AWS S3에서 암호화한다.

SSE-KMS

- S3 서비스가 보유한 키에 의존하지 않고 KMS 서비스를 이용하여 자신의 키를 관리한다.
- 이 경우 키를 우리가 통제할 수도 있고 CloudTrail로 키 사용을 검사할 수 있다.
- headers는 "x-amz-server-side-encryption" : "aws:kms"
- KMS는 KMS API를 써야 하기 때문에 버킷 처리량이 아주 많다면 문제가 될 수 있다.
- KMS quota per second 는 5500, 10000, 30000 req/s 이다, 이건 리전 별로 다를 수 있다.
- 업로드 시에는 GenerateDataKey KMS API, 다운로드 시에는 Decrypt KMS API를 호출해야 한다.

SSE-C

- 키가 외부에서 관리되지만 서버측 암호화이다.
- 키는 절대로 AWS S3 내에 저장되지 않으며 키를 전송해야 하기 때문에 HTTPS를 써야 한다.
- HTTPS 헤더의 일부로 키를 반드시 보내줘야 한다.
- 파일을 읽을 때도 키를 제공해야 한다.

Client-Side Encryption

- 라이브러리를 활용하면 쉽게 구현 가능, 암호화해서 보내는 개념이고 복호화는 받아서 푸는 식이다.
- 암호화 사이클과 키를 클라이언트가 관리하는 개념이다.

Encryption in Transit (SSL/TLS)

- S3에는 2개의 엔드포인트가 있다.
  - HTTP Endpoint - non encrypted
  - HTTPS Endpoint - encryption in flight

HTTPS를 추천한다.
SSE-C의 경우 반드시 HTTPS를 사용해야 한다.

버킷 정책에 추가하기

```Bash
...
"Effect": "Deny",
"Action": "s3:GetObject",
"Condition": {
    "Bool": {
        "aws:SecureTransport" : "false"
    }
}
```

위와 같은 S3 정책을 추가하면, SecureTransport가 아니라면 파일을 가져갈 수 없게 할 수 있다.

## S3 기본 암호화

Default Encryption vs. Bucket Policies

모든 버킷은 기본적으로 SSE-S3 암호화가 되어 있다.
다른 기본 암호화 방법으로 설정할 수 있고, 버킷 정책을 이용하여 해더에 적절한 암호화 헤더가 없는 PUT 요청은 거절하게 할 수도 있다. -> SSE-KMS, SSE-C의 경우 적용 가능
정책이 암호보다 우선이다.

## CORS-개요

CORS - Cross Origin Resource Sharing

Origin = scheme(protocol) + host(domain) + port
ex. https://www.example.com HTTPS의 포트는 443, 프로토콜은 HTTPS, 도메인은 www.example.com이다.

CORS는 웹 브라우저 기반 보안 매커니즘이다.

예를 들어

1. https://example.com/app1
2. https://example.com/app2

위 둘은 같은 오리진이다.

1. http://www.example.com
2. http://other.example.com

위 둘은 서로 다른 오리진이다.

웹 브라우저가 한 웹사이트를 방문하는 동안, 요청 체계의 일부로 다른 웹사이트에 요청을 보내야 할 때, 다른 오리진이 CORS 헤더를 사용해서 요청을 허용하지 않는 한 해당 요청은 이행되지 않는다.

- CORS Headers : Access-Control-Allow-Origin

(ex. 클라이언트 레벨에서 특정 웹서버에 접속한 상태에서 다른 API 서버에 데이터를 요청하는 경우, 이런 이슈가 종종 발생한다.)

S3에 대입해보자

if a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers

이 작업을 빠르게 하려면 all origins (\*)하여 모든 오리진을 허용할 수도 있다.

## S3 MFA Delete - 개요

MFA (Multi-Factor Authentication) : 사용자가 중요한 일을 하기 전에 특정 디바이스(보통은 스마트폰이나 기타 하드웨어)에서 생성된 코드를 입력하게 하는 기술

MFA는 S3에서 객체 버전을 영구적으로 삭제할 때 필요하다.
또는 버저닝을 종료할 때 필요하다.

버저닝을 삭제하거나 삭제된 버전을 나열하는 것에는 필요없다.

MFA delete를 사용하기 위해서는 반드시 버저닝이 먼저 활성화되어야 한다.
오로지 Bucket owner만이 MFA Delete를 활성화 혹으 비활성화할 수 있다.

현재로써 MFA Delete를 활성화하려면 CLI를 통해 명령어를 입력해야 한다.
콘솔로는 활성화가 불가능하다.

## S3 엑세스 로그 - 개요

- 감사 목적으로 S3 버킷에 대한 모든 엑세스를 기록할 수 있다.
- 어떤 계정에서 보낸 요청이든, 승인 혹은 거부된 것과 관계없이 다른 S3 버킷에 파일로 기록된다.
- 해당 데이터는 분석 가능하다.
- 로깅 버킷은 같은 리전에 있어야 한다.

- 주의 점은 절대 로깅 버킷과 모니터링 버킷을 동일하게 구성하며 안된다. 그럴 경우, 로킹 루프가 생겨 비용이 천문학적으로 나올 수 있다.

## S3 사전 서명된 URL - 게요

- S3 콘솔, CLI, SDK를 사용하여 생성할 수 있는 URL이다.
- 만료기간이 있는데
  - S3 콘솔 사용 시 ~ 12시간 (720분)
  - AWS CLI 사용 시 ~ 168시간 까지 사용 가능하다.

미리 서명된 URL을 만들 때, URL을 받는 사람은 URL을 보낸 사람의 GET 또는 PUT에 대한 권한을 상속한다.

use csse : 프라이빗한 S3 버킷이 있다고 할 때, AWS 외부 사용자에게 한 파일에 대한 엑세스 권한을 부여해야 하는 상황이 생길 수 있다. 보안 때문에 퍼블릭 엑세스가 안된다고 할 때, 해당 파일을 기준으로 미리 서명된 URL을 생성하고 이를 제공한다.

아주 널리 사용되는 방식이다.
특정 사용자에게 특정 동영상을 내려 받게 한다던가 하는 식이다.

## S3 잠금 정책 및 Glacier Vault

Glacier Vault Lock

- WORM(Write Once Read Many) 모델을 채용하기 위해 Glacier 볼트를 잠그는 것이다.
- 객체를 가져와 볼트에 넣은 다음에 수정하거나 삭제할 수 없도록 잠구는 것이다.
- 먼저 Glacier 위에 볼트 잠근 정책을 생성한 뒤, 잠근다.
- 규정 준수에 도움이 된다.
- 관리자나 AWS 서비스도 삭제 하지 못한다.

S3 Object Lock (versioning must be enabled)

- 버저닝 활성화 필수
- WORM 모델 채택
- 객체 잠금은 버킷 수준의 잠금이 아니라 모든 객체에 각각 적용할 수 있는 잠금이다.
- 단일 객체를 객체 잠금 할 수 있따.
- 적용 시 특정 객체 버전이 특정 시간동안 삭제되는 것을 차단할 수 있다.
- 보존 모드
  - 규정 준수 모드 : 사용자를 포함한 누구도 덮어쓰거나 삭제할 수 없다. 보존 모드도 바꿀 수 없고, 시간도 변경 불가
  - 거버넌스 보존 모드 : 대부분 사용자는 덮어쓰거나 삭제할 수 없지만, IAM으로 권한을 받은 일부 사용자나 관리자는 보존기간을 변경하거나 객체를 삭제할 수 있다.
- 기간을 설정해야 한다.

- 객체에 법적 보전 모드를 설정할 수 있다. 이 경우 버킷 내의 모든 객체를 무기한으로 보호한다. 대개 재판에 사용할 자료를 설정한다. 객체가 영구적으로 보존된다.
- s3:PutObjectLegalHold IAM Permission을 가진 사용자는 어떤 객체에든 법적보존을 설정하거나 제거할 수 있다.

## S3 엑세스 포인트

- 데이터와 사용자가 많이질 수록 관리가 어려워진다.
- 우리는 S3 엑세스 포인트를 만들 수 있는데, 예를 들어 Finance 엑세스 포인트를 만들었다고 하자.
- 이건 재무 데이터랑 연결될 것인데, 우리는 포인트 정책을 생성하고, /finance에 읽기, 쓰기 권한을 할당한다.

- 이런식으로 특정 prefix에 대한 읽기 쓰기 정책을 만들 수 있다.
- 보안관리를 S3 버킷에서 꺼내서 각각 엑세스 포인트에 넣었다.
- 사용자 중에 적절한 권한을 가진 자들은 이 엑세스 포인트를 통해 버킷의 특정 영역에 접근할 수 있다.
- 이렇게 하면 엑세스 포인트에 정책을 나눠주고 버킷 정책 자체는 심플하게 가져갈 수 있다.

- 각 엑세스 포인트는 개별적으로 DNS 이름을 갖게된다. 해당 엑세스 포인트가 Internet Origin이나 VPC Origin에 연결되도록 할 수 있다.

## S3 - Access Points - VPC Origin

- 엑세스 포인트를 VPC 내부에서만 접근 가능하게 할 수 있다.
- 우리는 Access pointdㅘ 연결하기 위해 반드시 VPC Endpoint를 생성해야 한다. (Gateway or Inteface Endpoint)
- 이 엔드포인트에는 정책이 있고, 타겟 버킷과 엑세스포인트에 접근 허용을 해줘야 한다.

## S3 오브젝트 람다

- 여러분에게 S3 버킷이 있는데 호출자 앱이 객체를 받기 직전에 그 객체를 수정하려는 경우에, 또는 우리의 버킷을 복제해서 버킷에 각 개체의 다른 버전을 갖는 대신에 S3 객체 람다를 사용할 수 있다.

- 이를 사용하기 위해 엑세스 포인트가 필요하다.

- 예를 들어 버킷에 앱이 데이터를 요청하고 받는 도중에, 분석을 위해서 데이터를 받아야 하는 경우가 있다. 다만 이떄 이 분석 앱은 데이터가 일부 삭제된 객체만 받고 싶다. 우리는 이를 위해 새로운 버킷을 만드는게 아니라 버킷에 엑세스 포인트를 생성한다. 그리고 해당 엑세스 포인트는 람다 함수와 연결된다. 이 람다함수는 객체를 받는 중에 그 객체에서 데이터를 삭제한다. 그리고 S3 객체 람다 엑세스 포인트를 만들어서 람다 함수에서 분석기 앱으로 연결한다.
