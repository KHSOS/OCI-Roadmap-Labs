# Database Storage

Database Storage는 Base Database의 파일을 저장하는 service allocation과 database 내부의 logical storage를 함께 이해하는 주제다. Tablespace의 free space, datafile 크기, DB System에 할당된 전체 storage는 서로 다른 계층의 숫자다.

## 역할과 구성

DB System은 local disk가 아니라 OCI block storage를 사용한다. Total storage는 available data storage, recovery area와 DB software용 reserve를 포함한다. Data 영역은 database 파일을, recovery 영역은 복구 관련 데이터를 담는다. ASM 또는 LVM 선택은 파일을 관리하는 하위 구조에 영향을 준다.

## 사용 예시

공식 storage table의 최소 예는 data 256GB, recovery 256GB, software reserve 200GB로 total 712GB다. 이 예는 PDB datafile의 사용량이거나 모든 shape에 대한 영구적인 주문값이 아니다. PDB datafile이 작아도 이미 할당한 service storage가 자동으로 줄거나 그 차이가 환불 가능 공간이 되는 것은 아니다.

## 선택 기준과 제약

Data/recovery storage는 별도로 scale할 수 있고 공식 문서는 recovery를 total의 20% 이상으로 유지하도록 권장한다. RAC에서는 두 node가 storage를 공유한다. Flexible와 standard shape의 allocation table이 다르며 LVM fast provisioning은 초기 크기에 따라 최대 scale 한도가 달라진다. 비용 비교에는 data만 아니라 total과 performance·backup을 고려한다. 용량 증가를 자유로운 축소나 일반 Linux disk 조작과 같은 것으로 보지 않는다.

## 관련 개념

- [Base Database Service](00_Base_Database_Service.md)
- [DB Systems](01_DB_Systems.md)
- [Backup / Restore](03_Backup_Restore.md)
- [ASM](04_ASM.md)
- [LVM](05_LVM.md)

## 더 읽기

- [Oracle — About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)

설정과 실습 절차는 [보존된 Database Storage Hands-on 문서](../../Hands-on/Original-Provisional/13_Base_Database_Service/02_Database_Storage.md)에서 읽을 수 있다.
