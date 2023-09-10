# Cloud Computing

## EC2

뛰어난 유연성

비용 효율성

빠른 속도

EC2는 가상화 기술을 이용하여 물리적인 서버에서 실행된다.

멀티 테넌시 - 하이퍼 바이저 기술

EC2에는 다양한 성능과 용량, OS들을 선택할 수 있다.

수직 확장이라고 한다.

## Amazon EC2 작동 방식

instance를 시작하는데, instance Template를 선택 [운영체제, Application Server 또는 Application], instance로 드나드는 네트워크 트레픽을 제어할 보안사양 지정

instance에 연결하는데, 연결 방법은 여러가지가 있고 [ssh 등...], 연결 시 바로 사용 가능

## Amazon EC2 instance

### General Purpose

- Application Server
- Game Server
- Enterprise Application Back-end Server
- Middle Size Database

### Computing Optimizaiton

- 고성능 프로세서를 활용하는 Computing 집약적인 Application에 적합
- General Purpose instance와 마찬가지로 Web Server, Application Server, Game Server 등에 사용이 가능하다.
- 하지만 좀 더 고사양의 웹서버나 컴퓨팅 파워가 많이 필요한 Application Server, Game 전용 Server에 더 적합하다.

### Memory Optimization

- 대규모 데이터 센터를 처리하는 WorkLoad를 위해 빠른 성능을 제공
- Memory는 임시 Storage 영역이라고 볼 수 있는데, 여기에는 CPU가 작업을 완료하는데 필요한 모든 Data의 명령이 들어있다.
- 이걸 Load Process라고 하며, 사전 Load Process덕분에 CPU가 직접 Program에 Access 할 수 있다.
- 예를 들어 Application 실행 전에 대규모의 데이터를 미리 Load해야 한다고 가정하자. 고성능 Data Base일 수도 있고, 비정형 데이터를 실시간으로 처리해야 할 수도 있다. 이럴때, 메모리 최적화 인스턴스 사용을 고려해야 한다.

### Accelerated Computer

- 하드웨어 가속 또는 Co-Processor를 사용하며, 일부 특수한 기능을 CPU보다 효율적으로 처리한다.
- 예를 들어 부동 소수점 계산, 그래픽 처리, 데이터 패턴 일치 등이 있다.

### Storage Optimization

- Local Storage의 대규모 데이터 세트에 대한 순차 읽기 및 쓰기 엑세스가 많이 필요한 워크로드를 위해 설계됨
- Storage 최적화 instance가 필요한 예로는 분산 파일 시스템, 데이터 웨어 하우징 어플리케이션, 고빈도 온라인 트렌젝션 처리(OLPT) 시스템 등이 있다.
- IOPS(초당 입출력 작업 수)라는 용어는 Storage Device의 성능을 측정하는 지표인데, 스토리지 최적화 인스턴스는 지연 시간이 짧은, 임의의 IOPS를 어플리케이션에 제공하도록 설계됨

## Amazon EC2 요금

1. 온디맨드 - 실행한 기간만 비용을 낸다. 인스턴스 유형과 운영체제에 따라 다르고, 선불제도 있다. 다만 이 방법은 1년 이상, 사용 패턴이 있는 워크로드에는 권장하지 않는다. 다른 방식이 비용절감 효가가 더 크기 때문이다.
2. Savings Plans - 일정 사용량을 약정하여, 1년에서 3년 동안 계약한다. 요금을 최대 72%까지 절약이 가능하다.
3. 예약 인스턴스 - 온디맨드보다 75% 저렴하다. 약정 시 일부 금액을 낸다. 온디맨드 인스턴스 사용 시, 결제 할인 옵션이다.
4. 스팟 인스턴스 - 온디맨드보다 90% 저렴하다. 워크로드 중단이 가능한 인스턴스 전용이다. 비용이 저렴한 대신에 예고없이 중단이 될 수 있다. 예를 들어 스팟 인스턴스의 수요가 늘어날 경우, 중단이 가능하다.
5. 전용 호스트 - 이 호스트의 태넌시는 다른 사람과 공유하지 않는다. 대신에 가장 비싼 옵션이다.
