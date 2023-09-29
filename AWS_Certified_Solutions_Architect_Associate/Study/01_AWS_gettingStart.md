# AWS Getting Start

AWS Global Infrastructure

- AWS Regions
  : 이름을 가지고 있다. 데이터 센터의 집합이다.
  대부분 서비스들은 특정 리전에 연결이 제한된다.

  어떤 리전을 선택할까?

  - 상황에 따라 다르다.

  1. Complience : 어떤 정부는 해당 앱의 정보가 자국에 있길 원한다.
  2. Proximity : 대부분의 사용자가 있는 곳에 구축하는 것이 당연히 지연시간을 줄이게 해줄 것이다.
  3. Available Services : 특정 리전은 특정 서비스가 지원이 안될 수 있다.
  4. Pricing : 리전 마다 요금이 차이가 있다.

- AWS Availability Zones
  : 각각 리전 내에 3 ~ 6개 존재한다.
  각각 가용영역은 별도의 전원, 네트워크를 가지고 있다.
  각각의 가용 영역이 재난 방지를 위해 서로 분리되어 있다.
  서로 영향을 미치지 않겠금 단절되어 설계되어 있다.
  높은 대역폭의 초저저 지연 네트워크를 통해 서로 통신한다.

- AWS Data Centers
- AWS Edge Locations / Points of Presence
  :216개 존재 / 205 Edge Locations

IAM, DNS, CloudFront, WAF 등은 모든 리전에 존재하지만 그렇지 않은 서비스도 많다.
