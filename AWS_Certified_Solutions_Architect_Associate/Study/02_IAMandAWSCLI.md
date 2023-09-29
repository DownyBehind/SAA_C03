# IAM and AWS CLI

계정 생성 시, 루트 계정을 생성했는데 이게 기본 계정이다.

보통 이후에는 사용하지고 공유하지 않는다.
보통 IAM에서 계정을 생성하여 사용한다.
그룹을 생성해서 계정들을 관리할 수 있다.

그룹 내에는 사용자만 배치할 수 있다. 그룹을 배치할 수는 없다.
한 사용자가 여러 그룹에 동시에 속할 수 있다.

Permissions : policy이다. 특정 사용자, 특정 그룹에 있는 사람에게 리소스를 허용하거나 금지하도록 구현

AWS : least privilege principle : 최소 권한 원칙

## IAM policy

그룹 레벨에 정책을 연결할 수 있다.

개발자와 운영자로 나눈다면 둘은 서로 다른 정책을 연결할 수 있고, 그룹이 아닌 특정 인원에게 직접 정책을 연결 할 수도 있다.

서로 다른 그룹에 있는 사람에게 감사 정책을 연결하면 해당 정책도 상속받게 된다.

정책은 JSON 구조로 구성된다.

version, Id, Statement로 구성된다.

Statement내에는 sid, effect, pricipal, action, resource등이 들어간다.

- 각 정책 내에 들어가는 요소는 확실히 이해하자

## IAM MFA 개요

Password policy : 비밀번호가 강력할 수록 보안이 강해진다.
여러가지 정책이 있는데 길이, 특수문자, 대문자 등을 요구할 수 있다.

IAM 사용자들의 비밓번호 만료기간을 설정할 수도 있다.
또한 비밀번호 재사용을 막을 수도 있다.

MFA(Multi Facotr Authentication)
-> AWS에서는 필수적으로 사용하길 원한다. 관리자의 경우 리소스를 삭제하거나 하는 일을 하는데, IAM 계정을 보호해야 한다. 이때 MFA를 적용하여 보안을 강화한다.

MFA 생성 토큰을 로그인에 사용하는 식이다.
MFA의 장점은 비밀번호를 누출되도 유저의 물리 장치가 필요하기 때문에 침투하기 어렵다.

- Virtual MFA Device
  Google Authentication, Authy....
- Universal 2nd Factor Security Key
  물리적인 장치, USB같은 것, 가지고 다니면서 연결해서 사용하는 식이다.
- Hardware key Fob MFA device

- Hardware Key Fob MFA Device for AWS GovCloud(US)

## AWS 엑세스 키, CLI 및 SDK

ASW 콘솔 : 비밀번호 + MFA로 보호
CLI : 엑세스 키로 보호
SDK -> 어플리케이션 코드 내에서 AWS 리소스 동작 : 엑세스 키로 보호

엑세스 키는 관리 콘솔에서 생성할 수 있다.

Access Key ID : user name
Secret Access Key : password

- 공유해서는 안된다.

CLI를 통해 AWS 리소스에 직접접근할 수 있고, 스크립트로 리소스 관리를 할 수도 있다.

SDK는 개발 키트 즉 라이브러리이다.

## AWS CloudShell: Region 가용성

- 모든 리전에서 해당 서비스를 사용할 수 있는 것은 아니다.

## AWS 클라우드쉘

터미널에 명령을 내리는 것 외에 AWS 콘솔에서 CloudShell에 접속해서 유사하게 사용할 수 있다.

## AWS 서비스에 대한 IAM Role

AWS 서비스 중 몇 개는 우리의 계정에서 실행해야한다.
따라서 IAM 계정에 권한을 줘야 할 수도 있다.

Role은 유저를 위해서 만드는 것이 아니라 서비스에 적용하기 위한 것이다.
EC2 instance가 어떤 AWS 리소스에 접근하고자 할 때, 이런 권한을 주는 것을 Role이라고 하는 것이다.

AWS 개체가 잠깐 크리덴셜을 받고 작업을 할 수 있게 하는 것이다.
우리 시험 범위는 AWS 서비스에 대한 Role이다.

EC2 instance나 Lambda에 Role을 적용시킬 수 있다.

## IAM 보안도구

IAM Credentials Report : 계정에 있는 사용자의 다양한 자격 증명을 나타낸다. (acoount-level)
IAM Access Advidor : 사용자에게 부여된 권한과 해당 서비스에 마지막에 접근한 시간을 알려준다. (user-level)

## IAM 모범 사례

가이드라인
: 루트 계정은 IAM계정 설정 외에는 사용하지 맗자
하나의 AWS 계정은 한 명이 사용하자
사용자를 그룹에 넣어서 보안을 관리할 수 있다.
비밀번호 정책을 만들자
MFA를 생성하는 것이 보안에 좋다.
AWS 서비스를 생성할 때마다 Role을 생성해서 적용하는 것이 좋다.
CLI 사용 시에는 Access keyfㅡㄹ 반드시 만들자.
자격증명 보고서를 활용하자.
