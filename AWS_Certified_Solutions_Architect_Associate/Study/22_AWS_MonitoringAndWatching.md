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

## CloudWatch 에이전트 및 CloudWatch Logs 에이전트

기본적으로 EC2에서는 어떤 로그도 CloudWatch로 넘어가지 않는다.
로그를 넘기려면 EC2에서 어떤 프로그램을 실행시켜서 원하는 로그 파일을 넘겨줘야 한다.

그 프로그램이 CloudWatch Logs Agent이고, 온프레미스 서버에도 동일하게 설치해서 사용할 수 있다.

CloudWatch에는 2가지 agent가 있는데 둘 다 가상서버를 위한 것이다. (EC2 instance, on-premise severs)

1. Logs Agent

   - CloudWatch log만 전송 가능

2. Unified Agent
   - 시스템 레벨 메트릭, 예를 들어 RAM, process 등을 추가로 보낼 수 있다.
   - 지표와 메트릭을 모두 보낼 수 있어 unified이다.
   - SSM Paramter Store를 사용해서 설정할 수 있다.

CloudWatch Unified Agent - Metrics

Linux 서버 / EC2에서 곧장 데이터를 수집한다.

- CPU metric
- Disk metric
- RAM
- Netstat
- Processes
- Swap Space

## CloudWatch 경보 개요

알람은 지표랑 설정된 알람을 표출

알람 상태

- OS
- INSUFFICENT_DATA
- ALARM

Period : 지표를 평가할 시간 단위

Target

- EC2 instance
- ASG
- SNS

Cloud Alarm은 하나의 지표에 대해서 트리거 한다.

Composite Alarms : 복합 지표 알람
-> 알람 노이즈를 줄이는데 좋다.

EC2 상태검사를 할 수 있다.

EC2 Instance Recovery : Same Private, Public, Elastic IP, metadata, placement group

Alarms can be created based on CloudWatch Logs Metrics Filters

## EventBridge 개요 (구 CloudWatch Evetns)

- Schedule : Cron jobs (scheduled scripts)

  - 한 시간마다 이벤트가 생성되어 람다함수를 실행시칸다던가 하는 것

- Event Pattern : 특정 이벤트에 반응하도록 구성

Ex. intance가 켜거나 꺼질 떄, 빌드 실패할 떄 등등..

Filter events로 이벤트를 필터링 할 수도 있다.
이 input을 받아서 JSON filer을 만든다. [이벤트 관련 내용]
그리고 대상 작업을 수행한다.

이벤트 버스 같은 느낌이다.

이벤트를 아카이빙할 수도 있다.

- 보존 기간은 영구적으로 하거나 기간을 정할 수 있다.
- 이렇게 replay archived events는 디버깅에 효과적이다.

EventBridge는 JSON형태로된 스키마에 이벤트를 분석한다.

스키마 레지스트리는 앱에 대한 코드를 생성하고 데이터가 이벤트 버스에서 어떤 구조로 있을 지 미리 볼 수 있다.

스키마는 버저닝이 가능하다.

리소스 기반 정책

- 특정 이벤트 버스의 권한을 정할 수 있다.

## CloudWatch Insights and Operaional Visibility

Container insight

- 컨테이너의 지표나 로그를 수집 결합 요약 함

EKS, Kubernetes에서 CloudWatch는 CloudWatch Agent의 컨테이너화된 버전을 사용해야 한다.

Lambda insight

서버리스 앱을 위한 지표 로그 수집

람다 계층에서 생산되어야 한다.

Contributor insights

Contributor (시스템에 영향을 미치는 상위자) 지표 로그
불향 호스트를 구분 할 수 있따.
네트워크를 많이 쓰는 사용자를 식별할 수 있따.

Application insight

앱에 잠재적인 문제를 모니터링한다.

SageMaker에 의해 대시보드가 생성된다.

발견된 알람은 이벤트브릿지에 전달된다.

## CloudTrail 개요 강의

거버넌스, 감사 등을 할 수 있다.

모든 이벤트와 API 호툴이력을 알 수 있다.
모든 로그가 기록된다.

A trail can be applied to All Regions (default) or a single Region

Event

- Management Event:

  - AWS 계정 안에서 리소스에 대해 수행한 작업
  - 기본적으로 Trail은 모든 관리 이벤트를 로깅하도록 설정되어 있다.
  - Read Event(리소스를 수정하지 않음), Write Event(리소스를 수정함)으로 구분할 수 있다.

- Data Event:

  - 기본적으로 이벤트를 로깅하지 않는다. (왜냐면 고용량작업이기에)
  - S3 관련 이벤트,
  - 읽기 쓰기 이벤트 분리

- CloudTrail insight Event

  - 비정상적인 활동이 감지되면 반응한다.
  - 자동화가 필요하면, 비정상적인 패턴을 감지하는 이벤트를 생성한다.
  - 그 이벤트는 S3로 전달되고, EventBridge에 이벤트를 생성한다.

- CloudTrail Events Retention
  - 이벤트는 90일간 보관이 가능하다.
  - 이 기간 이후에도 가령 감사 목적으로 보관하고자하면 보관할 수 있는데, S3나 Athena에 보관할 수 있다.
  - 즉, Management Events, Data Events, insights Events는 전부 CloudTrail로 가서 90일 동안 보관되고 장기간 보관하려면 S3 bucket에 로깅하고 그걸 분석할 준비가 되면, S3 데이터를 쿼리하는 Athena 서비스를 사용하면 된다.

## CloudTrail - EventBridge 통합

Example

우리가 IAM 규칙이 변경되는 걸 모니터링하고 싶다면,

IAM이 변경될 때, api를 호출하므로 이 기록은 CloudTrail에 기록되고 이 건 이벤트이기 때문에
EventBridge에 기록이 된다. 이 기록은 다시 SNS를 트리거 할 수 있다.

## AWS Config 개요

- AWS 리소스 감사와 규정준수를 기록
- 설정을 기록하여 시간에 따라 어떻게 변했는지 확인하는데 도움
- 예를 들어, 보안 그룹에 제한되지 않은 SSH 접근이 있는지?, 버킷에 공용 엑세스가 있는지, 시간에 따라 변한 ALB 규칙이 있는지?
- 어떤 변화가 있을 때마다 우리는 SNS 알람을 받을 수 있다.
- Config는 리전별 서비스이기 때문에 모든 리전별로 구성해야 한다.
- 데이터를 중앙화하기 위해 리전과 계정간에 통합할 수 있다.
- S3에 저장해서 Athena로 분석 가능

Config Rule

- AWS 관리 룰 - 75개
- 사용자 커스텀 룰

- 룰은 평가되고 호출된다.

  - 어떤 설정 변화에 따라
  - 정기적인 시간 텀에 따라

- Config는 규칙 준수를 위한 것이지 예방하기 위함은 아니다.

Config는 리전 당 Config 기록에 따라 3센트 지불
리전 당 규칙 평가 별로 1센트

AWS Config Resource

- 리소스 별로 규칙 준수 여부를 확인 가능
- 시간 대 별로 누가 설정을 변경했는지 모니터링 가능

Config Rules - Remediations

- 예방할 수는 없지만 SSM Automation을 통해 규정 준수하지 않는 리소스는 수정할 수 있다.

예를 들어 IAM Access Key가 90일 이상 지나서 만료되었다고 하자, 이런 경우는 규정을 준수하지 않은 상태라고 볼 수 있다. 예방은 못하지만 규정 위반할 때마다 수정은 할 수 있다.

UserCredentials라는 이름의 SSM 문서가 있고, 이걸 이용한다고 해보자. SSM문서가 IAM 엑세스키를 비활성화 할 것이다.

수정 작업은 재시도 할 수 있는데 5번까지 재시도 가능하다.

Config Rules - Notifications

AWS Config - EventBridge - SNS, SQS

AWS Config - SNS - Admin

## CloudTrail vs CloudWatch vs Conifg

CloudWatch

- 지표 분석
- 알람 경고
- 로그 집계 및 분석

CloudTrail

- API 호출 기록 모니터링
- 특정 리소스 기준으로 추적 정의 가능
- 글로벌 서비스

Config

- 구성 변경 기록
- 규정준수 모니터링
