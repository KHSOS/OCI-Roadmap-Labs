# Backup / Restore

Backup은 장애 전 데이터를 복구할 수 있도록 별도 사본과 필요한 복구 정보를 보존하는 기능이고 Restore/Recovery는 그 자료로 database 상태를 되돌리는 작업이다. 백업 작업 완료, 복구 가능 시점, 복구 후 데이터의 정확성은 각각 다른 의미다.

## 역할과 구성

Managed automatic backup은 Recovery Service 또는 Object Storage를 사용한다. Object Storage의 자동 체인은 weekly level 0, daily level 1과 archived redo를 포함한다. On-demand backup은 Object Storage에서 full, Recovery Service에서 incremental로 생성된다. Latest, timestamp, SCN restore는 서로 다른 복구 목표를 선택하며 standalone backup으로 새 database를 만드는 방식은 기존 DB의 in-place 복구와 다르다.

## 사용 예시

잘못된 변경 직전 상태가 필요하면 단순 Latest가 아니라 해당 변경 전의 유효한 timestamp 또는 SCN이 필요하다. Backup 목록의 Active만으로 그 시점까지 필요한 redo가 확보됐다고 단정할 수 없다. 복구 상태가 완료돼도 원하는 PDB와 데이터가 실제 되돌아왔는지는 별도로 확인해야 한다.

## 선택 기준과 제약

Recovery Service는 전용 database VCN network path가 필요하고 Object Storage 접근은 service gateway/Swift endpoint 조건을 고려한다. Destination 제공은 tenancy 생성일·Region·DB version·capacity에 따라 달라지며 Object Storage가 항상 Console에 노출되는 것은 아니다. LTR은 장기 보존용이며 in-place restore를 지원하지 않는다. DB termination 후 managed backup은 선택한 retention 또는 72시간 후 삭제 조건을 따르고 Object Storage full standalone backup이 남을 수 있다. Backup 비용·retention·삭제 보호와 실제 recovery window를 함께 검토한다.

## 관련 개념

- [Base Database Service](00_Base_Database_Service.md)
- [Database Storage](02_Database_Storage.md)

## 더 읽기

- [Oracle — Back Up a Database Using the Console](https://docs.oracle.com/en/cloud/paas/base-database/backup-db/index.html)
- [Oracle — Recover a Database Using the Console](https://docs.oracle.com/en/cloud/paas/base-database/recover-db/index.html)
- [Oracle — Back Up and Recovery in Base Database Service](https://docs.oracle.com/en/cloud/paas/base-database/backup-recover/index.html)

설정과 실습 절차는 [보존된 Backup / Restore Hands-on 문서](../../Hands-on/Original-Provisional/13_Base_Database_Service/03_Backup_Restore.md)에서 읽을 수 있다.
