# 솔루션 설계자 관점의 서버리스 개요

데이터를 빠르게 검증해보고 싶을 때 주로 사용

 

다이나모 - 테이블 예제

 

Primary Key  -----------------------  Attributes

ㄴ Partition key + Sort Key             ㄴ Score Result

 

속성은 null로 설정하거나 추가할 수 있다.

 

Dynamo DB를 쓰려면 Read/Write Capacity Mode도 설정해야 한다.

 

- Provisioned Mode (default) : 초당 읽고/쓰는 횟수를 정해서, 용량을 미리 정한다.

그리고 프로비저닝된 RCU(Read Capacity Unit)와 WCU(Write Capacity Unit)만큼 비용을 지불한다.

ASG도 사용 가능하다.

 

- On-Demand Mode : 워크로드에 따라 자동으로 증감하므로, RCU, WCU 개념이 없다. 사전 계획이 필요없다.

사용한 만큼 돈을 지불한다. 워크로드를 예측할 수 없거나 급격히 증가하는 경우에 유용하다.

수천개의 트랜젝션이 수백만개로 1분 내로 확장해야 하는 앱에 한해서는 이 모드를 사용하는게 적합하다.

 

## Amazon DynamoDB 심화기능

 

DynamoDB Accelerator(DAX)

 

- 다이나모디비를 위한 고가용성의 완전 관리형 무결점 인메모리 캐시

- 읽기 작업이 많을때, DAX 클러스터를 생성하고 데이터를 캐싱하여 읽기 혼잡을 해결한다.

- 캐시데이터에 마이크로초 단위의 지연을 가진다.

- 기존 다이나모디비 API와 호환되므로 앱 로직을 변경할 필요가 없다.

- 캐시를 위한 TTL은 기본 5분이며 변경 가능하다.

 

ElastiCache 대신에 DAX를 쓰는 이유는?

 

- DAX는 다이나모디비 앞에 있고, 개별 객체 캐시와 쿼리와 스캔 캐시를 처리하는데 유용하다.

- 집계 결과를 저장할 때는 ElastiCache가 좋다. - 대용량의 연산을 저장할 때

 

DynamoDB - 스트리밍 프로세스

 

테이블의 모든 수정 사항(생성/업데이트/삭제 등)에 대한 스트림 생성 가능

 

사용자 테이블에 새로운 사용자가 등록되었을 때, 환영 이메일을 보내는 등 다이나모디비 테이블의 변경 사항에

실시간으로 반응하는데 활용할 수 있다.

 

- 실시간으로 응답하고

- 실시간으로 분석하고

- 파생 테이블에 삽입하고

- 다른 지역에 복제를 할 수도 있고

- 디비 변경에 따른 람다 함수 호출도 가능하다.

 

1. DynamoDB Streams

- 24시간 보존되며

- 소비자 수가 제한된다

- 람다 트리거랑 같이 쓰면 좋고, 자체적으로 읽으려면 DynomoDB Stream Kinesis 어댑터를 쓴다.

 

2. Kinesis Data Streams

- 변경 사항을 바로 보낼 수도 있다.

- 보존기간이 1년이고

- 더 많은 소비자를 갖고

- 데이터를 처리하는 방법이 더 많다.

 

글로벌 테이블 : 여러 리전 간에 복제가 가능한 테이블

서로 다른 리전에 있는 테이블끼리 양방향 복제가 가능하다.

 

여러 리전에서 낮은 지연시간으로 서로 엑세스하게 해준다.

서로 복제가 가능하므로, 앱이 모든 리전에서 테이블을 읽고 쓸 수 있다는 말이다.

 

TTL(Time To Live)

 

- 만료 타임스탬프가 지나면 자동으로 항목을 삭제하는 기능

TTL을 Attribute에 추가할 수 있고, 에포크 타음스템프가 TTL을 넘어가면 해당 항목을 만료처리하고

삭제를 하는 개념이다.

 

예를 들어 몇 년 뒤 삭제해야하는 규정이 있는 경우 적용할 수 있다.

 

자주 등장하는 예제는 웹 세션 저장입니다.

 

사용자가 웹에 접속하면 다이나모디비에 두 시간 정도 저장한다.

여기에 세션 데이터를 저장하면 모든 앱이 엑세스할 수 있고, 두 시간 뒤 ExpTime이 갱신되지 않으면 만료되어 삭제될 것이다.

 

재해복구에도 다이나모디비를 활용한다.

- 지정 시간 복구 PITR(Continuous backups using point-in-time recovery) : 지속적인 백업을 할 수 있다.

           ㄴ 활성화를 선택할 수 있고 35일 동안 지속된다.

           ㄴ 활성화하면 백업 기간 내는 언제든 지정 시간 복구를 실행할 수 있다.

           ㄴ 복구를 진행할 경우, 새로운 테이블을 생성한다.

          

- 온 디멘드 백업

           ㄴ 이 백업은 직접 삭제할 때까지 보존된다.

           ㄴ 성능이나 지연시간에 영향을 주지 않는다.

           ㄴ AWS 백업을 사용하면 수명 주기 정책을 활성화할 수 있고, 재해 복구 목적으로 리전 간 백업을 복사할 수도 있다.

           ㄴ 이 옵션 또한 백업으로 복구를 진행하면, 새로운 테이블이 생성된다.

          

- DynamoDB - Amazon S3간 통합

S3로 테이블을 보낼 수 있는데, PITR을 활성화해야 한다.

테이블을 S3로 보내고 쿼리를 사용하려면, Athena 엔진을 사용한다.

분석 목적이나 감사 목적으로 내보낼 수 있다.

 

S3에서 테이블을 가져올 수도 있다.

CSV, JSON, ION 형시으로 보낸 뒤, 새로운 DB 테이블을 생성하는 식이다.

가져올 때 발생하는 오류는 모두 CloudWatch Log에 기록된다.

 

## API GateWay 개요

 

람다함수에서 API의 데이터베이스로 다이나모디비를 사용할 수 있으며, 테이블 CRUD를 할 수 있다.

클랑언트도 이 람다 함수를 지연 호출하게 하고 싶다.

 

 

클라이언트 ---- API GateWay ---- 람다 ---- 다이나모디비

 

클라이언트는 API Gateway에 REST API로 요청하고, API GateWay는 람다에 프록시 요청을 보낸다.

그리고 람다는 다시 다이나모 디비에 CRUD 요청을 보내는 식이다.

 

API Gateway를 사용하는 이유는 HTTP Endpoint 뿐 아니라 다양한 기능들 에를 들어 인증부터, 사용량 계획, 개발 단계 등의 기능을 제공하기 때문이다.

 

AWS Lamda + API Gateway : 인프라 구축이 필요없다.

 

WebSocket 프로토콜을 지원한다.

버저닝도 지원하므로, 버전이 올라가도 클라이언트 연결이 끊기지 않는다.

여러 환경을 다룰 수 있다. (dev, test, prod, ...)

인증, 권한 부여 등 수많은 보안 기능을 활성화 할 수 있다.

API key 생성할 수 있고, 클라언트 요청이 과도할 때, 요청을 스로틀링할 수 있다.

Swagger / 오픈 API를 통해 신속하게 API 정의를 가져올 수 있다.

 

API Gateway - Integrations High Level

 

- Lambda Function

           - 람다함수 지연 실행

           - 람다함수를 REST API로 노출시키는 일반적이고 간단한 방법

          

- HTTP

           - HTTP endpoint를 노출시킬 수 있다.

          

- AWS Service

           - 어떤 AWS API도 노출시킬 수 있다.

           - AWS Step Function workflow를 시작하거나, SQS에 메시지를 보낼 수도 있다.

          

예시

 

클라이언트 ---- API GATEWAY ---- Kinesis Data Streams ---- Kinesis Data firehose ---- Amazon S3

 

API gateway의 장점은 AWS 서비스를 외부에 노출시킨다는 점이다.

 

API Gateway를 노출시키는 방법을 Endpoint Type이라고 하는데

 

1. Edge-Optimized (default): For global clients

- 전세계 누구나 엑세스할 수 있다.

- 모든 요청이 클라우드프론트 엣지 로케이션으로 라우팅되므로 지연시간이 개선된다.

- API Gateway는 생성된 리전에 있지만, 모든 클라우드프론트 엣지 로케이션에 의해 엑세스될 수 있다.

 

2. Reginal : 같은 리전 사용자

- 자체 클라우드프로트 배포를 할 수 있다. 캐싱과 배포에 더 많은 권한을 가질 수 있다.

 

3. private : VPC 내에서만 접근 가능

- ENI를 통해 접근

- 엑세스 정의는 리소스 정책을 사용한다.

 

 

보안

 

IAM role : 내부 앱에 유용

Cognito : 외부 사용자

Custom Auhorizer : 내 로직

 

Custom Domain Name HTTPS security through integraton with AWS Certificate Manager(ACM)

 

## Amazon Step Functions

 

람다함수를 이용하여 시각적인 워크프로를 만드는 것

 

각 단계의 성공, 실패에 따라 어떤 걸 수행하는지 정의합니다.

복잡한 워크플로를 만들어 AWS에 실행시킬 수 있는 편리한 도구이다.

EC2, ECS, 온프레미스 서버, API GateWay, SQS 큐등을 넣을 수 있다.

 

중간에 사람이 승인해야 넘어갈 수 있는 단계를 넣을 수도 있다.

 

## Amazon Cognito 개요

 

사용자에게 웹이나 앱에 상호작용할 수 있는 자격증명을 준다.

 

- Cognito User Pools:

           - Sign in functionality for app users

           - Integrate with API Gateway & Applicaton Load Balancer

          

           - 서버리스 데이터 베이스이다.

           - 간단하게 사용자 이름, 이메일, 비밀번호 조합으로 로그인 절차를 정의할 수 있다.

           - 비밀번호 재설정 기능이 있고, 이메일 및 번호 검증, MFA도 있다.

           - Facebook, Google과 통합할 수 있다.

          

           사용자가 Cognito를 통해 토큰을 받으면, 해당 토큰으로 API Gateway에 REST API로 토큰을 보내고,

           Congito를 통해 인증받으면, 사용 자격을 얻고 람다 함수로 가게 된다.

          

           다른 방법으로는 Cognito 앞에 ALB을 놓고 먼저 인증을 받고 나면 ALB가 타겟 그룹으로 리다이렉트하는 식으로 쓸 수도 있다.

          

- Cognito Identity Pools (Federated Identity):

           - 앱에 등록된 사용자에게 임시 AWS 자격 증명을 제공하여 리소스에 직접 엑세스하게 해준다.

           - Cognito User Pool과 원활히 통합된다.

          

           - 사용자에게 자격증명을 주지만, 임시 자격증명으로 AWS 계정에 직접 접근한다.

           - 사용자는 사용자 풀 내 사용자일수도 있지만 타사 사용자일 수도 있다.

           - 자격증명에 쓰인 IAM 정책은 Cognito에 정의되어 있다.

           - 인증된 사용자나 게스트 유저에게 Default IAM role을 부여할 수도 있다.

          

IAM에도 사용자를 다루는 기능이 있지만, Cognito는 외부의 웹가 앱 사용자를 대상으로 한다.

 

수백명의 사용자, 모바일 사용자, SAML을 통한 인증 같은 내용이 나오면 Cognito를 찾아보자