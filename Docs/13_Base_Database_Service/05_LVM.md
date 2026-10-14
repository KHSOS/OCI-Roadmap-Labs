# LVM

Logical Volume Manager (LVM)는 disk나 partition을 논리적인 storage로 묶는 OS 계층의 관리 방식이다. 물리 장치와 filesystem 사이에 추상화를 제공하여 공간을 Volume Group으로 모으고 Logical Volume으로 나눌 수 있다. Base Database에서는 single-node fast provisioning의 storage management 옵션으로 사용된다.

## 역할과 구성

Physical Volume(PV)은 LVM이 관리하는 disk/partition, Volume Group(VG)은 PV를 묶은 공간, Logical Volume(LV)은 VG에서 할당한 논리 device다. XFS/EXT4 같은 filesystem을 LV 위에 두고 그 안에 database datafile이 들어갈 수 있다. LV, mount, datafile, tablespace는 같은 계층의 이름이 아니다.

## 사용 예시

Datafile이 filesystem 안에 있을 때 파일 → mount의 device → LV → VG → PV라는 관계로 storage 구조를 이해할 수 있다. Filename이 일반 경로라고 그것만으로 LVM이라고 단정할 수 없고 해당 filesystem의 실제 device 관계가 중요하다. 일반 Linux LVM 관리 예와 Oracle가 관리하는 Base Database storage는 책임 경계가 다르다.

## 선택 기준과 제약

Base Database의 LVM fast provisioning은 single-node용이며 RAC에는 사용할 수 없다. Custom database software image로 LVM provisioning하는 것도 지원되지 않는다. 초기 data 256GB 또는 512GB의 공식 fast-provisioning table은 최대 2,560GB scale을 설명하므로 작은 초기 구성이 무제한 확장을 뜻하지 않는다. Generation-agnostic x86와 Ampere A1은 LVM만 지원한다. Managed DB에서 일반 OS의 format·PV/VG/LV 변경을 service scale 작업 대신 적용하지 않는다.

## 관련 개념

- [Base Database Service](00_Base_Database_Service.md)
- [DB Systems](01_DB_Systems.md)
- [Database Storage](02_Database_Storage.md)
- [ASM](04_ASM.md)

## 더 읽기

- [Oracle — Configure Logical Volumes on Oracle Linux](https://docs.oracle.com/en/learn/ol-lvm/)
- [Oracle — About DB Systems](https://docs.oracle.com/en/cloud/paas/base-database/about-dbs/index.html)

설정과 실습 절차는 [보존된 LVM Hands-on 문서](../../Hands-on/Original-Provisional/13_Base_Database_Service/05_LVM.md)에서 읽을 수 있다.
