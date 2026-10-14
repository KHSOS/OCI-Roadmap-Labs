# Cloud-init / User Data

**User Data**는 인스턴스 시작 시 전달하는 초기화 입력이고, **cloud-init**는 지원 Linux image에서 그 입력을 해석해 초기 구성을 적용하는 소프트웨어다. 둘은 같은 자원이 아니다. OCI의 Initialization script는 custom script 또는 cloud-init configuration을 전달하는 창구이며 Windows에서는 cloudbase-init를 사용한다.

## 왜 사용하는가

같은 이미지에서 여러 서버를 시작할 때 필요한 초기 파일이나 설정을 사람이 각각 입력하면 차이가 생기기 쉽다. User Data로 서버의 역할별 초기 구성을 전달하면 동일한 시작 의도를 반복해서 표현할 수 있다. 예를 들어 웹 서버별 식별 파일이나 첫 시작 시각을 남기는 입력은 초기 설정이 실제 guest에 적용됐는지 이해하는 교육용 사례다.

## Image와 초기화의 관계

Image는 운영체제와 미리 포함된 소프트웨어의 출발점이고 User Data는 그 인스턴스에 전달하는 추가 입력이다. 이미지를 만들 때 포함한 애플리케이션과 시작 때 설치하는 애플리케이션은 복원 방식이 다르다. User Data가 외부 패키지를 가져오면 DNS·네트워크 경로·저장소 가용성에도 의존한다. Compute의 Running 상태는 초기화의 모든 작업이나 애플리케이션 준비가 끝났다는 뜻이 아니다.

## 선택할 때 생각할 것

|요구|판단 방향|
|---|---|
|소량의 인스턴스별 초기 설정|User Data로 입력과 이미지의 역할 분리|
|큰 소프트웨어 묶음과 반복 배포|Image에 포함할 부분과 외부 구성 관리가 맡을 부분 구분|
|민감한 인증 정보|초기화 파일에 비밀을 그대로 넣는 방식 대신 적절한 비밀 관리·인증 설계|
|Windows 초기화|Linux cloud-init 예제를 그대로 적용하지 않고 cloudbase-init 지원 확인|

## 한계와 주의점

Console은 업로드한 원문을 제출 시 Base64 인코딩한다. Base64는 암호화가 아니다. launch 문서는 User Data와 기타 metadata 총합을 32,000 bytes로 설명하지만 Compute overview는 user_data 자체를 최대 16 KB로 별도 설명한다. 따라서 총합 한도만으로 개별 user_data의 허용 크기를 판단하지 않으며 작은 입력을 사용하고 해당 인터페이스·image의 제한을 확인하는 것이 안전하다.

초기화 도중 reboot를 유발하면 launch가 실패할 수 있다. 이미 시작한 VM의 metadata를 바꿨다는 사실만으로 첫 boot 작업이 다시 실행됐다고 볼 수 없으며, 재실행 여부는 image와 cloud-init 모듈의 동작에 달려 있다.

## 관련 개념

- [Compute](00_Compute.md)
- [Capacity Types](08_Capacity_Types.md)

## Hands-on으로 이어가기

개념 설명과 자원 생성 절차는 구분된다. 기존 작업·검증·정리 내용은 [보존된 Hands-on](../../Hands-on/Original-Provisional/02_Compute/07_Cloud_init_User_Data.md)에서 읽을 수 있다.

## 공식문서

- [Creating an Instance](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm)
- [Overview of the Compute Service](https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm)
