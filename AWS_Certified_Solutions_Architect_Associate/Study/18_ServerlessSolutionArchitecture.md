# 서버리스 솔루션 아키텍쳐 토론

 

## 모바일 어플리케이션 : To Do List

 

요구사항

           - HTTPS endpoint가 있는 REST API가 노출되야 한다.

           - 서버리스 아키텍쳐

           - 사용자가 원하면 스스로 데이터를 관리하도록 S3에 접근 가능하게 한다

           - 사용자는 관리형 서버리스 서비스에 인증해한다.

           - 사용자는 읽고 쓸 수 있지만 대부분 읽는다.

           - 데이터베이스는 읽는 성능이 좋아야 한다.

          

          

 

클라이언트 ---- api gateway ---- Lamda ----- DynamoDB

           ㄴ---- Cognito ---- STS

           ㄴ---- S3

          

STS는 사용자 자격증명을 관리한다.

 

따라서 모바일 사용자가 자신의 폰에서 자격증명을 관리한다고 하면 틀린 설명이다.

 

 

읽기 처리량을 늘리고, 비용을 줄이려면 DAX를 서야한다.

RCU는 비용이 든다.

 

API Gateway도 Reponse를 캐싱할 수 있다.

 

## 서버리스 웹사이트 : Myblog.com

 

요구사항

 

           - 글로벌하게 스케일되어야 한다.

           - 블로그는 쓰는건 가끔이고 읽는게 자주 일어난다.

           - 웹사이트는 대부분 정적 파일로 구성되고 일부는 dynamic REST API이다.

           - 캐싱을 적용해 비용을 절감하고 응답속도를 줄이고 싶다.

           - 블로그에 처음 방문하는 사람에게는 따뜻한 환영 이메일을 발송하고 싶다.

           - 썸네일을 서버리스로 구현하고 싶다.

          

 

콘텐츠는 정적이고 글로벌이어야 한다. - S3 + CloudFront + Edge location

 

안전하게 - origin access를 제한해서 S3에 CloudFront만 접근하게 버킷 정책 수정

 

서버리스 REST API - API Gateway + Lambda + DAX Cache + Dynamo DB

 

환영 이메일 - Dynomo DB stream + Lambda + IAM Role + SES

 

썸네일 - S3 + CloudFront OAI = S3 Transfer Acceleration + Lambda

 

## 마이크로서비스 아키텍쳐

 

- 마이크로 서비스로 전환하려고 한다.

- 많은 서비스가 REST API를 통해 상호작용할 것이다.

- 아키텍쳐는 모습과 형태가 다르다.

- 사용하는 이유는 개발 수명 주기를 줄이기 위함이다.

 

마이크로 서비스는 각자 서비스가 별도의 DNS를 가지고 있고,

서로 상호작용도 할 수 있다.

 

- 동기식 패턴은 다른 서비스를 명시적으로 호출한다.

 

- 비동기식 패턴은 S3처럼 SQS, Kinesis, SNS, Lambda에 트리거로 작동하는 경우이다.

           - 언제 응답할 지는 미지수이다.

          

문제는 마이크로서비스를 생성할 때마다 오버헤드가 발생할 수 있다는 점이다.

서버 밀도나 사용률을 최적화하는데도 쉽지 않다.

 

여러 버전을 동시에 돌리다가 문제가 생기기도 하고 요구사항이 급증하기도 한다.

 

이런 문제는 서버리스로 어느정도 해결되는데, API Gateway, Lambda는 사용한 만큼만 비용을 내면 되기 때문이다.

 

## 소프트웨어 업데이트 배포

 

업데이트 오프로딩 - EC2에 동작하는 앱이 잇는데 종종 업데이트 해야한다.

 

콘텐츠를 네트워크로 배포하면 돈이 많이든다.

 

비용없이 앱의 CPU 사용량을 최적화하고 싶은데 방법이 있을가?

User - ELB - EC2 - EFS

이 구조에서 User와 ELB 사이에 CloudFront를 두면 된다.

 

아키텍쳐에는 변화가 없다.

엣지에서 소프트웨어 업데이트 파일은 캐시에 저장된다.

소프트웨어 업데이트 파일은 정적이기 때문에 변하지 않는다.

EC2는 서버리스가 아니지만 CloudFront는 서버리스라서 확장 가능하다.

우리 아키텍쳐 내의 ASG는 많이 늘지 안아 EC2, EFS 비용을 절감할 수 있게 된다.

가용성을 확보 할 수 있다.

 

엣지 캐싱 서비스는 비용을 절감할 수 있다.