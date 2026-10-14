# Base Database Service

Oracle Base Database Service는 OCI virtual machine의 DB System에서 Oracle Database를 운영하는 서비스다. Database edition/version·compute·storage·network를 선택하고 database를 관리한다. 서비스가 제공하는 provisioning·관리 기능과 애플리케이션의 접속·성능·데이터 보호 설계는 구분해야 한다.

## 역할과 구성

DB System은 VM node와 storage 관리의 서비스 경계다. 하나의 DB System에는 하나의 Database Home과 하나의 database가 있으며 선택 edition/version의 조건에 따라 여러 PDB를 지원할 수 있다. CDB와 PDB의 연결 대상, OCI lifecycle, 실제 database session의 가용성은 구분한다. Single-node와 multi-node RAC는 가용성·비용·지원 옵션이 다르다.

## 사용 예시

Private subnet의 단일 DB System에서 애플리케이션이 PDB service를 통해 접속하는 구성이 가능하다. OCI에서 AVAILABLE로 표시돼도 client route·보안·listener·PDB service가 맞지 않으면 앱 접속은 실패할 수 있다. Public web LB를 DB 접속의 필수 구성요소로 볼 필요는 없다.

## 선택 기준과 제약

Shape의 ECPU/OCPU, memory, edition/license, storage total, backup·network 요구를 함께 비교한다. VM.BaseDB.x86는 현재 공식 문서상 4~256 ECPU(4단위), single-node/LVM, 19c·26ai, SE/EE/EE-HP를 지원한다. VM.Standard.x86·EE-EP·OCPU-metered SKU retirement 공지는 2026-07-31을 명시하고 신규 provisioning과 renewal에 영향을 준다. 과거 shape 표만으로 현재 신규 제공을 보장할 수 없다. ASM과 backup은 지원 조합과 별도 비용을 고려하고 DB 종료가 모든 backup의 즉시 삭제를 뜻하지 않는다.

## 관련 개념

- [DB Systems](01_DB_Systems.md)
- [Database Storage](02_Database_Storage.md)
- [Backup / Restore](03_Backup_Restore.md)
- [ASM](04_ASM.md)
- [LVM](05_LVM.md)

## 더 읽기

- [Oracle — About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)

설정과 실습 절차는 [보존된 Base Database Service Hands-on 문서](../../Hands-on/Original-Provisional/13_Base_Database_Service/00_Base_Database_Service.md)에서 읽을 수 있다.
