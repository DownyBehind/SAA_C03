# 기타 서비스

## CloudFormation 소개

CloudFormation is a declarative way of outlining your AWS Infrastructure, for any resources (most of them are supported).

예를 들어서 CloudFormation Template 안에다가

- 보안 그룹이 필요해
- 이 보안 그룹을 사용하는 EC2 instance 2개가 필요해
- S3 버킷이 필요해
- ELB가 내 머신 앞에 있으면 좋겠어

CloudFormation은 이 모든 걸 여러분이 정한 순서와 구성 그래도 생성한다.

Benefits of AWS CloudFormation

- Infastructure as code

  - 매뉴얼로 조작할 필요없음
  - 인프라 변경 시 코드 리뷰

- Cost

  - 아마도 각 리소스들은 비슷하게 만들어질테니 비용을 확인하기 쉬울 것
  - 템플릿을 사용하므로 비용 예측이 쉽
  - 탬플릿으로 절약 계획을 세울 수도 있는데, 어떤 환경에서는 오후 5시에 자동으로 모든 탬플릿을 삭제하게 할 수도 있다.

- Productivity

  - 재생산이 쉽다.
  - 자동으로 다이어그램을 그릴 수 있다.
  - Declarative programming

- Don't re-invent the wheel
  - 웹 상에 이미 존재하는 탬플릿 확용
  - 문서 활용

CloudFormation Stack Designer

- 어떤 리소스가 구성되어 있는지 연결되어 있는지 보기 쉽다.

## Amazon SES (Simple Email Service)

- 완전 관리형 서비스이며, 이메일을 보내준다.

- SES API 혹은 SMTP를 사용하면 사용자들에게 대량으로 이메일을 보낸다.
- 아웃바운드/인바운드 허용이라 이메일을 받을 수도 있다.

## Amazon Pinpoint

확장 가능한 양방향 인바운드/아웃마운드 마케팅 서비스

Pinpoint를 통해 이메일, SMS, 푸시, 음성, 인앱 메시지를 보내고 싶다는 것이다.

usecase 중 하나가 SMS이다.

## SSM(Systems Manager) Session Manager

Allows you to start a secure shell on your EC2 and on-premises servers

- No SSH access, bastion hosts or SSH keys needed
- No port 22 needed (better security)

포트가 아니라 사용자는 Session Manager를 통해서 EC2에 접근할 수 있다.

Linux, Mx, Window에서 지원 된다.

## SSM 기타 서비스

Run Command

Script나 command를 실행한다
리스소 그룹에 있는 여러개의 인스턴스에 실행된다.
SSH가 필요하지 않다.
실행 명령과 결과는 S3 또는 CloudWatch로 보내진다.
실패 시, SNS로 전송된다.
자동화할 수 있다.
EC2와 온프레미스 지원

Patch Manager

인스턴스를 관리하는데 있어 패칭을 자동화하는데 사용
운영체제 및 앱 업데이트
EC2와 온프레미스 지원
즉시 패치와 스케줄 패치를 "Maintenance Windows"를 통해 설정한다.

Maintenance Windows

- 인스턴스에 수행할 작업을 언제 할 지 정의한다.
- OS 패치, 드라이버 업데이트, 소프트웨어 설치...
- Maintenance Window contain
  - 스케쥴
  - 기간
  - 어느 인스턴스
  - 실행될 작업

Automation

Simplifies common maintenance and deployment tasks of EC2 instances and other AWS resources

자동화를 사용하면, 인스턴스들은 한 번에 다시 시작이 가능하다

Automation Runbook

- SSM Documents to define actions preformed on your EC2 instnaces or AWS resources(pre-defined or custom)

## AWS 비용 탐색기

시간에 따른 사용량을 시각화

사용자 정의 보고서 생성

대시보드 그리고 도면

전체 계정 간 총비용 등등...

리소스 별 금액

청구서 요금을 낮출 수 있는 Saving Plan을 얻을 수 있다.
향후 12개월까지의 사용량을 예측할 수 있다.

## Elastic Transcoder

Elastic Transcoder is used to convert media files stored in S3 into media files in the formats required by consumer playback devices(phone etc...)

## AWS Batch

어떤 규모의 배치라도 실행 가능

수십만개의 컴퓨팅 배치작업도 쉽게 효율적으로 할 수 있다.

배치는 시작과 끝이 있는 작업이다. (반대는 스트리밍 작업)

배치 서비스는 EC2 instance 또는 spot instance를 동적으로 시작한다.

Batch jobs are defined as Docker images and run on ECS

## Amazon AppFlow

Fully managed integration service that enables you to securely transfer data between Software-as-a-Service (SaaS) applications ans AWS

Source : Salesforce, SAP, Zendesk, Slack, ServiceNow

Destination : S3, Redshift

## AWS Amlify

웹 및 모바일 어플리케이션 개발 도구이다.

- A set of tools and services that helps you develop and deploy scalable full stack web and mobile app

다양한 AWS 서비스에 대해서 Amplify로 한 곳에 인증하고 각종 API를 설정할 수 있다.

Amplify API로 백엔드 설정을 수정할 수 있다.

Amplify는 웹 모바일을 위한 Beamstalk이라고 보자
