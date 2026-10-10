# DNS Zones / Records

> Draft / 작성예정 — 문서 구조 준비용 초안입니다. 완성된 실습 가이드가 아닙니다.

## 학습목표

- SOA로 정의되는 Zone과 그 안의 Resource Record를 구분할 수 있다.[17]
- A·AAAA·MX Record의 RDATA 의미를 설명할 수 있다.[17]


## 대응 Roadmap 항목

- 항목명: **DNS Zones / Records**
- Roadmap 경로: **DNS → DNS Zones / Records**
- Node ID: `sequence-8-1`
- 상위 항목: **DNS** (`sequence-8-0`)
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

- **권장 실습:** 격리 DNS Zone 레코드 구성
- **실습 방식:** Console에서 전용 실습 리소스를 관찰·구성하는 방식. 아래 활동은 구성 제안이며 상세 실행절차는 별도 작성 대상이다.
  1. private DNS 실습 zone에 테스트 A record를 구성하고 VCN 내 조회를 확인한다
  2. 공식문서의 해당 설정·제약과 관찰값을 대조한다.
  3. 아래 결과를 기록하고 선택 생성한 리소스의 정리 범위를 확인한다.
- **확인할 결과:** 레코드 값·scope·resolver 조회 결과
- **선행/연계 실습:** [DNS](00_DNS.md) — 문서 연계이며 모두 리소스 생성을 요구하는 필수 순서는 아니다.
- **주의/대체:** 로그인 가능한 OCI 계정과 해당 범위의 조회 권한이 필요하다. 생성·변경은 별도 승인된 전용 Compartment에서만 수행하며 권한·Region 지원·한도·현재 비용을 먼저 확인한다. 실습 후 생성 리소스의 삭제 대상과 보존 여부를 확인한다. 리소스가 없거나 생성 승인이 없으면 공식 예제 기반 구성표로 대체한다. 운영 zone·registrar 위임은 변경하지 않는다. Public DNS 실습은 소유한 별도 테스트 도메인이 있을 때만 선택하고 없으면 문서 설계로 대체한다.
- **공식 근거:** [Overview of DNS — DNS Zones / Records](https://docs.oracle.com/en-us/iaas/Content/DNS/Concepts/dnszonemanagement.htm)

- **확장 준비:** private zone·resolver view 연결과 VCN 내 DNS 조회 가능한 실습 클라이언트가 필요하다. 없으면 zone·record 설계표로 대체한다.

- **추가 공식 근거:** [Private DNS — zone·view·resolver 구성](https://docs.oracle.com/en-us/iaas/Content/DNS/Tasks/privatedns.htm)

### 학습용 구성도

![DNS Zones / Records — 학습용 제안 구성도](../diagrams/77-DNS-Zones-Records.png)

> **학습용 제안 구성도**입니다. 실제 배포 현황(as-built)이나 OCI 실습 실행 결과가 아닙니다. 화살표는 그림에 표시된 요청·응답 또는 관리·개념 관계를 뜻합니다. 서비스 계열 아이콘의 정확한 Oracle Toolkit 명칭과 해석 범위는 구성 모델에 기록했습니다.

- [편집 가능한 PowerPoint](../diagrams/77-DNS-Zones-Records.pptx) · [구성 모델(JSON)](../diagrams/77-DNS-Zones-Records.json)
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

- [17] [Overview of DNS — DNS Zones / Records](https://docs.oracle.com/en-us/iaas/Content/DNS/Concepts/dnszonemanagement.htm)
  - 확인 범위: zone A zone is / resource records.
