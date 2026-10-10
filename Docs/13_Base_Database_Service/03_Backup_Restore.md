# Backup / Restore

> Draft / 작성예정 — 문서 구조 준비용 초안입니다. 완성된 실습 가이드가 아닙니다.

## 학습목표

- Recovery Service와 Object Storage 백업 대상을 구분할 수 있다.[42]
- 백업·복구 구성과 복구 시점 관련 조건을 공식문서에서 확인할 수 있다.[42]


## 대응 Roadmap 항목

- 항목명: **Backup / Restore**
- Roadmap 경로: **Base Database Service → Backup / Restore**
- Node ID: `sequence-10-3`
- 상위 항목: **Base Database Service** (`sequence-10-0`)
- 문서 범위: 이 항목의 개념 학습목표와 공식 출처. 실습절차 및 실행 검증은 작성예정입니다.
- [전체 항목 목록](../README.md)


## 사전준비

작성예정 — 실습 내용은 아직 작성하지 않았습니다.

## 권한

작성예정 — 실습 내용은 아직 작성하지 않았습니다.

## 비용및주의사항

작성예정 — 해당 항목의 과금 조건과 안전 주의사항을 공식문서 기준으로 확인합니다. 이 초안만으로 리소스를 생성하지 마세요. 무료 사용을 보장하지 않습니다.

## 실습구성

> 아래는 Oracle 공식문서가 설명하는 기능을 기반으로 설계한 **권장 실습안**이며 Oracle이 공인·권장한 교육과정이라는 뜻은 아닙니다. 실제 OCI 실행·검증은 수행하지 않았습니다.

- **권장 실습:** 격리 DB Backup·Restore 계획
- **실습 방식:** 문서 기반 설계·비교를 기본으로 하고 Console에서 실제 옵션을 읽기 전용으로 관찰한다. 생성 화면 검토만 하는 경우 제출하지 않는다.
  1. 승인된 비운영 DB의 backup 설정·대상과 복원 방법을 검토하고 선택적으로 별도 DB 복원을 설계한다
  2. 공식문서의 해당 설정·제약과 관찰값을 대조한다.
  3. 아래 결과를 기록하고 선택 생성한 리소스의 정리 범위를 확인한다.
- **확인할 결과:** backup 상태·복원 대상·정합성 확인 계획
- **선행/연계 실습:** [Base Database Service](00_Base_Database_Service.md) — 문서 연계이며 모두 리소스 생성을 요구하는 필수 순서는 아니다.
- **주의/대체:** 로그인 가능한 OCI 계정과 해당 범위의 조회 권한이 필요하다. 생성·변경은 별도 승인된 전용 Compartment에서만 수행하며 권한·Region 지원·한도·현재 비용을 먼저 확인한다. 실습 후 생성 리소스의 삭제 대상과 보존 여부를 확인한다. 리소스가 없거나 생성 승인이 없으면 공식 예제 기반 구성표로 대체한다. 고비용·고급 선택 실습이다. 기본은 문서와 생성 화면 검토이며 DB 생성·RAC 구성·복원은 자동 수행하지 않는다. 유료 승인과 별도 격리 DB·backup·접속 준비가 있을 때만 확장한다. DB system의 ASM/LVM 디스크를 직접 재구성하지 않는다. 운영 DB에 덮어쓰기 복원하지 않으며 별도 대상과 지원 복원 방식·예상 비용을 검토한다.
- **공식 근거:** [Back Up and Recovery in Base Database Service — Backup / Restore](https://docs.oracle.com/en/cloud/paas/base-database/backup-recover/index.html)

### 학습용 구성도

![Backup / Restore — 학습용 제안 구성도](../diagrams/90-Backup-Restore.png)

> **학습용 제안 구성도**입니다. 실제 배포 현황(as-built)이나 OCI 실습 실행 결과가 아닙니다. 화살표는 그림에 표시된 요청·응답 또는 관리·개념 관계를 뜻합니다. 서비스 계열 아이콘의 정확한 Oracle Toolkit 명칭과 해석 범위는 구성 모델에 기록했습니다.

- [편집 가능한 PowerPoint](../diagrams/90-Backup-Restore.pptx) · [구성 모델(JSON)](../diagrams/90-Backup-Restore.json)
- 아이콘 출처: [Oracle OCI Architecture Diagram Toolkits](https://docs.oracle.com/en-us/iaas/Content/General/Reference/graphicsfordiagrams.htm). 실제 리소스를 생성하지 않은 학습 설계입니다.

## 실습절차

작성예정 — 실습 내용은 아직 작성하지 않았습니다.

## 결과검증

작성예정 — 기대 결과와 검증 방법을 정의합니다. 실제 OCI 실습 실행 및 검증은 수행하지 않았습니다.

## 문제해결

작성예정 — 실습 내용은 아직 작성하지 않았습니다.

## 리소스정리

작성예정 — 실습 내용은 아직 작성하지 않았습니다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [42] [Back Up and Recovery in Base Database Service — Backup / Restore](https://docs.oracle.com/en/cloud/paas/base-database/backup-recover/index.html)
  - 확인 범위: The Oracle-managed automatic backups.
