# ASM

Automatic Storage Management (ASM)은 Oracle Database 파일을 위한 volume manager와 file system이다. 여러 disk를 Disk Group으로 관리하고 파일 내용을 분산하여 storage 관리를 단순화한다. 일반 OS filesystem이나 database instance와 같은 객체는 아니다.

## 역할과 구성

ASM instance는 diskgroup metadata와 파일 배치 정보를 관리하고 database instance는 그 정보를 통해 파일을 사용한다. ASM은 Grid Infrastructure home에 설치되며 Database Home과 구분된다. ASM filename은 +와 diskgroup 이름으로 시작한다. Disk Group에는 파일·disk·allocation unit이 포함되고 한 ASM 파일은 하나의 diskgroup 안에 들어간다.

## 사용 예시

+DATA로 시작하는 datafile은 일반 filesystem 경로가 아니라 ASM diskgroup 파일 이름의 예다. 파일을 여러 disk에 분산하고 disk 구성 변경 시 rebalance하는 기능은 storage 배치의 관리이며 database backup을 대신하지 않는다. Diskgroup free 공간과 OCI total provisioned storage도 정의가 다르다.

## 선택 기준과 제약

일반 ASM은 Normal의 2-way, High의 3-way mirroring과 External redundancy를 제공하지만 이것만으로 Base Database의 실제 redundancy 설정을 추정하지 않는다. Base Database의 standard provisioning은 ASM이고 RAC는 ASM이 필요하다. Generation-agnostic x86와 Ampere A1은 LVM 전용이라 ASM 선택과 호환되지 않는다. 현재 신규 ASM 제공 shape/version은 retirement 공지와 서비스 지원을 함께 확인한다. Managed DB의 diskgroup을 일반 Compute disk처럼 임의 생성·삭제하는 학습 범위와 구분한다.

## 관련 개념

- [Base Database Service](00_Base_Database_Service.md)
- [DB Systems](01_DB_Systems.md)
- [Database Storage](02_Database_Storage.md)
- [LVM](05_LVM.md)

## 더 읽기

- [Oracle — Introducing Oracle Automatic Storage Management](https://docs.oracle.com/en/database/oracle/oracle-database/19/ostmg/asm-intro.html)
- [Oracle — About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)

설정과 실습 절차는 [보존된 ASM Hands-on 문서](../../Hands-on/Original-Provisional/13_Base_Database_Service/04_ASM.md)에서 읽을 수 있다.
