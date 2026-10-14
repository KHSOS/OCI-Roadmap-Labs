# DB Systems

DB System은 Base Database의 compute node, Database Home, database와 OCI block storage를 묶는 운영 단위다. Node는 VM이고 Database Home은 database software가 설치된 환경이며 PDB는 database 내부의 논리 container다. 각 이름을 서로 다른 VM이나 독립 DB System으로 해석하지 않는다.

## 역할과 구성

Single-node는 한 VM, Multi-node RAC는 두 VM 구성이다. 공식 Base Database 모델은 DB System당 Database Home 하나, 그 안의 database 하나를 지원한다. PDB 수는 edition/version 및 licensing 제약에 따른다. Shape가 compute·memory를, storage 선택이 database 파일의 관리 방식을 정한다.

## 사용 예시

단일 시스템 안의 CDB와 PDB를 조회하면 OCI resource hierarchy와 SQL session의 container가 다른 계층임을 이해할 수 있다. 두 RAC node는 storage capacity를 공유하므로 node가 둘이라는 이유로 사용 가능한 database storage를 두 배로 계산하지 않는다.

## 선택 기준과 제약

Single-node와 RAC 선택에는 장애 범위·edition·license·ASM 지원을 함께 고려한다. Generation-agnostic x86와 Ampere A1은 single-node/LVM 전용이며 x86 shapes는 fault domain 선택을 지원하지 않는다. Edition은 생성 후 변경할 수 없다는 공식 제약이 있다. OCPU와 ECPU를 같은 숫자 단위로 취급하지 않고, retired SKU와 현재 제공 shape를 구분한다. Node lifecycle 정상은 listener/PDB/client 접속의 정상과 별개다.

## 관련 개념

- [Base Database Service](00_Base_Database_Service.md)
- [Database Storage](02_Database_Storage.md)
- [ASM](04_ASM.md)
- [LVM](05_LVM.md)

## 더 읽기

- [Oracle — About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)

설정과 실습 절차는 [보존된 DB Systems Hands-on 문서](../../Hands-on/Original-Provisional/13_Base_Database_Service/01_DB_Systems.md)에서 읽을 수 있다.
