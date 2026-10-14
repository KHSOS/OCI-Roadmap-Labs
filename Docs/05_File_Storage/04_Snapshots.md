# Snapshots

File Storage Snapshot은 File System 전체의 일관된 point-in-time view다. Copy-on-write 방식으로 보존되며 개별 client의 mount point나 특정 Export만의 Snapshot이 아니다. 원본 File System과 Snapshot 데이터는 at rest 암호화된다.

## 현재 데이터와 시점 데이터

Snapshot 이후 원본 파일이 바뀌면 현재 파일과 Snapshot이 가리키는 과거 파일은 다른 값을 가질 수 있다. Snapshot은 File System root의 숨겨진 `.snapshot/name` 경로에서 접근한다. 이 경로는 Export path와 별개인 File System 내부 경로다.

예를 들어 현재 보고서가 수정된 상태에서도 Snapshot에는 수정 전 보고서가 남을 수 있다. 과거 파일을 현재 영역의 별도 파일로 복사하는 복원과 Snapshot 자체를 바꾸는 것은 다르다. 전체 File System의 view라고 해서 application transaction의 일관성까지 자동으로 보장하는 것은 아니므로 애플리케이션의 저장·동기화 방식도 고려한다.

## 생성 방식의 선택

- **User-created Snapshot**: 변경 전이나 특정 작업 완료 시점처럼 사용자가 선택한 상태를 보존한다.
- **Policy-based Snapshot**: Snapshot Policy와 schedule에 따라 자동 생성하고 retention에 따라 만료·삭제한다.
- **Replication Snapshot**: Replication 자원이 데이터 전달에 사용하는 서비스 관리 Snapshot이다. 사용자가 수정하는 일반 Snapshot과 같지 않다.

수동 시점 보존은 단순하지만 반복적인 보호 주기를 대신하지 않는다. Policy는 자동화를 제공하지만 보존 기간과 생성·삭제 처리 지연을 고려해야 한다. 많은 동시 policy 작업은 throttling되어 뒤의 기회에 처리될 수 있다.

## 복원과 재해복구의 관계

Snapshot의 파일 또는 디렉터리 내용을 현재 영역이나 다른 저장 위치로 복사할 수 있다. Clone은 Snapshot에서 새 File System을 만드는 별도의 선택이며, Replication은 다른 File System으로 데이터를 전달하는 별도 기능이다. Snapshot 하나를 만드는 것만으로 다른 AD/Region의 독립적인 데이터 보호가 완성되지는 않는다.

## 보존량과 수명

과금 사용량은 Snapshot이 보존하는 차등 데이터에 따라 측정된다. 이전 Snapshot 이후 변경이 없으면 새 Snapshot이 추가 저장량을 사용하지 않지만, 변경·삭제된 데이터를 과거 view가 계속 보존하면 저장량이 남는다. 파일의 현재 크기만으로 전체 사용량을 판단할 수 없다.

Expiration Time은 만료 후 서비스가 삭제할 시점을 정하고, Exclusive Bytes는 해당 Snapshot만 사용하는 저장량으로 삭제 시 회수되는 양을 나타낸다. Clone과 Replication의 관계, 보존 목적과 필요한 복원 시점을 함께 고려해야 한다. 오래된 view를 보존하는 것과 비용 없이 영구 보존하는 것은 다르다.

## 함께 읽기

- [File Storage](00_File_Storage.md)
- [Exports](02_Exports.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/05_File_Storage/04_Snapshots.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Managing Snapshots](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/managingsnapshots.htm)
- [Restoring from a Snapshot on a Unix-style Instance](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/restore-from-snapshot.htm)
- [Paths in File Systems](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filesystempaths.htm)
