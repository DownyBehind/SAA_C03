# 고급 Amazon S3

## S3 수명 주기 규칙

- 다른 스토리지 클래스 간에 객체를 옮기는 방법
  각 클래스는 치환이 가능한 클래스가 존재한다.

예를 들어 자주 접근하지 않을 것 같은 Standard tier의 object는 Standard IA tier로 변경할 수 있고, 오래동안 아카이빙할 것 같은 object의 경우에는 Glacier 또는 Glacier Deep Archive로 변경할 수도 있다.

다만, 우리는 이걸 수작업으로 옮길 수도 있지만, Lifecycle Rule을 통해서 자동으로 옮길 수도 있다.

## Life Cycle Rules

- Trasition Actions

  - 생성 후 60일이 지나면 Standard IA로 옮긴다.
  - 생성 후 6개월이 지나면 Glacier로 옮긴다.

- Expiration Actions

  - 365일이 지나면 삭제한다.
  - 버저닝이 활성화되어 있다면, 이전 버전은 삭제할 수 있다.
  - 불완전한 멀티 파트 업로드는 삭제할 수 있다.

- 특정한 prefix로 룰을 정할 수도 있다.
- 특제 개체 테크에만 룰을 정할 수도 있다.

Amazon S3 Analytics

- 언제 트렌지션하고 어떤 저장 클래스로 변경할 지 분석해준다.
- Standard와 Standard IA에 대해 추천해준다. (One-Zone IA와 Glacier는 지원하지 않는다.)
- 결과를 볼 때까지는 24 ~ 48시간이 소요될 수 있다.

## S3 요청자 지불

Requester Pays

- 버킷 소유자가 버킷 관련 비용을 다 내는 것으로 배웠다.
- 수많은 대형 파일이 있고, 일부 고객이 이것을 다운로드 받을 때, 우리는 Requester Pay를 활성화 할 수 있다.

요청자가 네트워킹 비용을 부담하는 것이다.
아주 큰 데이터를 공유할 때, 유용하다.
그럴려면 요청자가 AWS에 인증을 받아야만 한다.

## S3 이벤트 알림

이벤트 : 객체가 생성/삭제/복구/복제 되었을 때

- SNS 토픽, SQS, Lambda 들을 통해 이벤트에 대한 응답을 보낼 수 있다.

IAM Permissions

- 이벤트를 받으면 SNS에 보내고 싶다면, SNS Resource policy를 첨부해야 한다.
- S3 버킷이 SNS 토픽에 직접 메시지를 보내는 것을 허용한다.

EventBridge : 모든 이벤트는 여기로 가고, 18개가 넘는 AWS 서비스가 목적지가 될 수 있다.

## S3 Performance

S3 기준 성능

- 요청이 아주 많을 때 자동으로 확장한다.
- 지연시간은 100 ~ 200ms
- 버킷 당 하나의 prefix에서 초당 3,500개의 PUT/COPY/POST/DELETE와 5,500개의 GET/HEAD 요청 수행 가능 [고성능]
- 접두사의 의미는 예를 들어 bukcet/folder1/sub1/file이면 /forder1/sub1/ 이 접두사이다.
- 접두사의 수는 제한이 없다.

Multi-Part Upload

- 100MB 이상 파일에 추천, 5GB 이상 시 무조건 사용
- 업로드를 병렬화하므로 효율이 높다.

S3 Transfer Acceleration

- 파일을 엣지 로케이션으로 전송하여, 전송속도를 높이고 데이터를 대상 리전에 있는 S3 버킷으로 전달한다.
- 엣지 로케이션은 리전보다 많고, 현재 200개 이상이다.
- multi-part upload와 호환된다.
- 우선 유저와 가까운 엣지 로케이션에 파일을 저장하고, 엣지로케이션과 S3 bucket간에는 AWS 프라이빗 경로로 빠르게 데이터를 전송시킨다.

S3 Byte-Range Fetches

- 바이트 범위 가져오기, 특정 바이트 범위를 가져와서 GET 명령을 병렬 수행한다.
- 특정 바이트 범위를 가져오는데 실패하더라도 더 작은 바이트 범위에서 시도하므로 복원력이 좋다.
- 다운로드 속도를 증가시킬 때 사용한다.

## S3 셀렉트 & Glacier Select

S3에서 파일을 검색할 때 서비 사이드 필터링을 사용하면 효율적이다.
SQL 문에서 행과 열에 간단한 필터를 사용하면 시간을 줄일 수 있다.

- S3 Select를 사용하면 속도는 4배 빠르고 비용은 80%이다.

## S3 Batch Operations

- 단일 요청으로 S3 객체에 대량 작업을 수행
- S3 객체의 메타데이터 프로퍼티 수정, 배치 작업, 암호화되지않은 버킷을 전부 암호화
- ACL 태그 수정 등등

- 작업은 객체의 목록, 수행할 작업, 옵션 매개변수로 구성된다.
- 왜 스크립팅 하지않고 S3 Batch Operation을 사용할까? 왜냐면 이렇게 해야 재시도를 관리할 수 있고 진행 상황을 추적하고 보고서도 생성할 수 있다.

S3 Inventory 기능으로 객체 목록을 가져오고 S3 Select로 필터링한 뒤 S3 Batch Operations로 임무를 수행시킨다.
