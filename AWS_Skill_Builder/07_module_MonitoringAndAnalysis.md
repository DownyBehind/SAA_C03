# Module 7 Monitoring and Analysis

- AWS 환경의 모니터링 방법을 요약할 수 있다.
- Amazon CloudWatch의 이점을 설명할 수 있다.
- AWS CloudTrail의 이점을 설명할 수 있다.
- AWS Trusted Advisor의 이점을 설명할 수 있다.

## Amazon CloudWatch

Amazon CloudWatch - 다양한 지표를 모니터링 및 관리하고 해당 지표의 데이터를 기반으로 경보 작업을 구성할 수 있는 웹 서비스이다.

지표를 사용하여 리소스의 데이터 포인트를 나타낸다. AWS 서비스는 지표를 CloudWatch로 전송한다. 그러면 CloudWatch가 이러한 지표를 사용하여 시간 경과에 따라 성능이 어떻게 변화하는지 보여주는 그래프를 자동으로 생성한다.

CloudWatch 경보 - 지표의 값이 미리 정의된 임계값을 상회 또는 하회할 경우 자동으로 특정 작업을 수행하는 경보를 생성할 수 있다.

Ex. 개발자들이 앱 개발 또는 테스트를 위해 EC2 인스턴스를 사용한다고 할 때, 개발자가 가끔 인스턴스를 중지하는 것을 잊는 경우에 인스턴스가 실행되어 요금이 발생한다.

이 시나리오에서는 CPU 사용률이 지정된 기간동안 특정 값 미만으로 유지된면 EC2 인스턴스를 자동으로 중지하는 CloudWatch 경보를 생성할 수 있다. 경보를 구성 시, 경보가 트리거 될 때마다 알람을 받도록 지정할 수 있다.

이 기능을 사용하면 해당 대시보드에서 여러 리소스 들의 지표를 볼 수 있다.

## AWS CloudTrail

AWS CloudTrail - 계정에 대한 API 호출을 기록, 기록되는 정보에는 API 호출자 ID, API 호출 시간, API 호출자의 소스 IP 주소등이 포함된다. CloudTrail은 누군가 남긴 이동 경로(또는 작업 로그)의 추적으로 생각할 수 있다.

API 호출을 사용하여, AWS 리소스를 프로비저닝, 관리 및 구성할 수 있다. 또한 앱 및 리소스에 대한 사용자 활동 및 API 호출 전체 내역을 볼 수 있다.

일반적으로 API 호출 후 15분 이내에 CloudTrail에 업데이트된다. API 호출이 발생한 날짜 및 시간, 작업을 요청한 사용자, API 호출에 포함된 리소스 유형을 지정하여 이벤트를 필터링 할 수 있다.

ex. 루트 사용자가 IAM에 들어가서 처음보는 IAM 계정을 보았다고 해보자. 언제 생성된 지 기억이 나지는 않는다. 따라서 이런 것을 확인하기 위해서는 CloudTrail에 들어가보면 된다.
필터에 IAM의 CreateUser만 검색하도록 하면, 해당 사용자가 언제 생성되었는지 확인할 수 있다.

CloudTrail Insights - CloudTrail에서 해당 기능을 활성화할 수 있다. 이 옵션 기능을 사용하면 CloudTrail이 AWS 계정에서 비정상적인 API 활동을 자동으로 감지할 수 있다.

예를 들어 해당 기능은 최근에 계정 내에 평소보다 많은 수의 EC2 인스턴스가 시작될 경우 감지할 수 있다.

## AWS Trusted Advisor

AWS Trusted Advisor - AWS 환경을 검사하고 AWS 모범 사례에 따라 실시간 권장사항을 제시하는 웹 서비스이다.

- 비용 최적화
- 성능
- 보안
- 내결함성
- 서비스 한도

위의 5개 범주 안에서 결과를 AWS 모범사례와 비교한다.

AWS Management Console에서 Trusted Advisor 대시보드에 엑세스하면 위의 5가지 항목에 대한 완료된 검사를 검토할 수 있다.

- 녹색: 문제가 감지되지 않은 항목의 수
- 주황색: 권장 조사 항목의 수
- 빨간색: 권장 조치의 수

## 모듈 7 요약

- Amazon CloudWatch
- AWS CloudTrail
- AWS Trusted Advisor
