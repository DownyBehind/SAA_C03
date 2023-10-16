# 백서 및 아키텍쳐 - AWS 공인 솔루션 아키텍트 어소시에이트

## 백서 섹션 소개

## AWS Well-Architected Framework 와 Well-Architected Tool

도구이자 프레임워크

아키텍쳐를 모범 사례에 맞게 설계 할 수 있다.

- Stop guessing your capacity needs
- Test systems at production scale
- Automate to make architectural experimentation easier
- Allow for evolutionary architectures
- Drive architectures using data
- Improve through game day

5 pillars

1. Operational Excellence
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization
6. Sustainability

AWS Well-Arch. Tool

위의 6가지 정의를 기준으로 아키텍쳐에 대해 조언해준다.

## AWS Trusted Advisor 개요

AWS 게정에 대한 전체적인 평가를 제공한다.

많은 검사를 실행하고, 합격했는지 알려준다.

5가지 범주에 대해서 추천을 제공한다.

1. Cost optimzation
2. Performance
3. Security
4. Fault tolerance
5. Service limits

Support Plans

7 Core Checkes - Basic & Developer Support plan

1. S3 Bucket Permissions
2. Security Groups - Specific Ports Unrestricted
3. IAM Use (on IAM user minimum)
4. MFA on Root Account
5. EBS Public Snapshots
6. RDS Public Snapshots
7. Service Limits

Full Checks - Business & Enterprise Support plan

1. Full Checks available on the 5 categories
2. Ability to set CloudWatch alarms when reaching limits
3. Progammatic Access using AWS Support API

## 아키텍쳐의 예 - AWS SAA
