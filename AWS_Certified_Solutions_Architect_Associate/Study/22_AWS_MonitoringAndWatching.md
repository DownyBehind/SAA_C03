# AWS 모니터링 및 감사: CloudWatch, CloudTrail 및 Config

 

## AWS 모니터링 - 섹션 소개

 

모니터링은 굉장히 중요하다.

 

## CloudWatch

 

- 모든 서비스에 대한 지표를 제공한다.

- metric is a variable to monitor

- Metric belong to namespace

- 지표 당 최대 측정 수는 10개

- 메트릭은 타임스템프를 가진다.

- cloudWatch custom metric도 가능하다.

 

cloudWatch Metric Streams

- 원하는 대상으로 지속적으로 스트리밍 가능, 거의 실시간으로 전송되고 지연시간도 짧아진다.

 

cloudwatch metric --- kinesis data firehose --- S3 --- Athena

                                                     ㄴ 거의 실시간

                                                    

필터로 메트릭을 적용할 수도 있다.

 

## 로그 개요

 

CloudWatch Log - Log를 저장하기 가장 좋은 곳

 

Log groups : arbitrary name, usually representing an application

Log stream : instances within application / log files / containers

- 로그 만료 정책을 정한다. (never expire / day ~ 10 years)

 

- 다양한 곳에 로그를 보낼 수 있다. : kinesis Data stream, kinesis data firehose, s3, lambda, opensearch

 

- 기본적으로 로그는 암호화되고, 원하면 자신의 키를 사용해서 KMS 기반 암호화도 가능하다.

 

Source

 

- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent

- Elastic Beanstalk : collection of logs from application

- ECS : 컨테이너에서 모아서 보냄

- Lambda : 함수로그를 모아서 보냄

- VPC Flow Logs : VPC metadata network traffic의 특정 로그를 전송한다.

- API Gateway : 모든 요청을 보낸다.

 

CloudWatch Logs Insight

 

cloudwatch log안의 쿼리 기능이다.

그에 대한 결과를 시각화해서 받을 수 있다.

대시보드에 추가할 수 있다.

 

쿼리 언어를 제공한다.

실시간 엔진이 아니라 쿼리 엔진이다.

 

로그 데이터를 내보내는데 최장 12시간이 걸릴 수도 있다.

api call은 CreateExportTask

- Batch를 내보내는 것이기 때문에 실시간이 아니다.

 

cloudWatch Logs Subscriptions를 사용하면, 로그 이벤트들의 실시간 스트림을 얻게 된다.

 

Subscription Filter : 로그 이벤트의 종류를 필터링 할 수 있다.