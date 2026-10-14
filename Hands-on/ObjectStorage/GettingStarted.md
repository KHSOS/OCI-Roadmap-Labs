# Object Storage 시작하기: 내 파일 올리고 내려받기

Object Storage는 파일을 object로 저장하는 서비스입니다. Bucket은 object를 담는 공간이며 object name은 그 안에서 파일을 찾는 이름입니다. 개인 계정에 새 private Standard bucket 하나를 만들고 작은 파일을 올려 내려받습니다. 다운로드한 내용이 원본과 같으면 첫 파일 사용을 마친 것입니다. 삭제·재업로드는 선택 연습이고, 마지막 자원 정리는 반드시 진행합니다. VM·VCN·SSH·Cloud Shell은 필요하지 않습니다.

[실습 목록으로](../README.md)

**바로가기**: [1. 원본 준비](#1-작은-원본-만들기--pc) · [2. Bucket 생성](#2-새-private-standard-bucket-만들기--console) · [3. 업로드](#3-pc-원본-올리기) · [4. 다운로드·첫 성공](#4-내려받아-내용-비교하기) · [5. 선택 재업로드](#5-선택-삭제-후-재업로드하기) · [검증·문제해결](#결과검증과-문제해결) · [정리](#리소스정리)

## 파일이 object가 되면 무엇이 달라지나요

Object Storage는 사진·문서·백업 파일처럼 데이터를 저장하고 다시 읽을 때 사용하는 서비스입니다. OS를 부팅하는 Compute의 Boot Volume과 달리, 이번에는 VM에 디스크를 연결하지 않고 브라우저에서 파일을 업로드·다운로드합니다. 따라서 VCN이나 SSH가 필요하지 않습니다.

### Bucket과 Object name으로 파일 찾기

Bucket은 object를 담는 컨테이너이고, object는 파일 데이터와 그 데이터를 설명하는 metadata로 이루어집니다. 이번 파일의 데이터는 한 줄의 텍스트이고 Console에서는 object name, 크기, storage tier 같은 정보를 함께 볼 수 있습니다.

예를 들어 이번 bucket 안의 `lab/payload.txt`에서 `lab/`도 object name의 일부입니다. Console이 폴더처럼 보여 주더라도 PC의 폴더를 먼저 만들 필요는 없습니다. 업로드 화면의 prefix `lab/`와 PC 파일명 `payload.txt`가 합쳐져 이 이름이 됩니다. `payload.txt`만 올린 object와 `lab/payload.txt`는 다른 이름이므로, 다운로드할 때는 bucket과 전체 object name을 함께 확인합니다.

### Private과 Standard는 서로 다른 선택입니다

**Private**은 누가 접근할 수 있는지에 관한 설정이고, **Standard**는 데이터를 어떤 storage tier에 둘지에 관한 설정입니다. Private bucket도 권한이 있는 계정은 Console에서 내려받을 수 있습니다. 다운로드를 위해 public으로 바꾸지 않습니다. Standard는 바로 읽을 이번 텍스트 파일에 맞으며, Archive처럼 읽기 전 복원하는 경로는 이번에 사용하지 않습니다.

전체 흐름은 **PC 원본 → bucket에 Upload → object 선택 → PC의 다른 폴더로 Download → 내용 비교**입니다. 업로드 뒤 PC 원본과 object는 별개의 사본입니다. PC 파일을 수정하는 것만으로 저장된 object가 바뀌지는 않으며, versioning을 끈 이번 실습에서 object를 삭제하면 PC 원본으로 다시 올려야 합니다. 첫 다운로드를 마친 뒤에만 선택 재업로드 연습을 진행합니다. [공식 Object Storage 개요](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm)에서 bucket·object와 서비스 사용 방식을 더 볼 수 있습니다.

## 학습목표

private Standard bucket 하나에 `lab/payload.txt`를 올리고 Console로 내려받아 원본 내용을 비교합니다. 마지막에 이번 object와 bucket을 삭제합니다. 원한다면 정리 전에 원본으로 삭제·재업로드를 한 번 연습할 수 있습니다.

## 대응 Roadmap 항목

**Object Storage** · `sequence-6-0`. Bucket, Object, 이름의 prefix, storage tier를 실제 생성과 파일 작업에서 익힙니다. Versioning, Lifecycle, PAR, Archive는 이 시작 경로에 포함하지 않습니다.

## 사전준비

개인 OCI Console에 로그인하고 Region과 자원을 담을 Compartment 하나를 선택합니다. PC에서 텍스트 파일을 저장하고 브라우저 업로드·다운로드를 할 수 있으면 됩니다. CLI·Python·다른 튜토리얼 완료는 요구하지 않습니다. 기존 bucket은 사용하지 않고 이번 새 bucket만 대상으로 합니다.

## 권한

선택 Compartment에서 bucket 생성·조회·삭제, object 목록·업로드·다운로드·삭제가 가능해야 합니다. Console 로그인과 private bucket 접근 권한은 별개입니다. 작업이 거부되면 IAM을 확인하고 멈춥니다. public 전환이나 PAR 생성으로 우회하지 않습니다. [Object Storage IAM reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/objectstoragepolicyreference.htm)의 bucket/object 권한표를 참조하세요.

## 비용및주의사항

저장 용량·요청·조건별 전송의 요금을 확인합니다. 작은 텍스트 파일 하나만 사용합니다. 선택 연습을 할 때만 재업로드 한 번을 추가합니다. Standard는 바로 읽기 위한 선택이며 Archive의 복원 대기는 포함하지 않습니다. private도 저장 비용이 면제되는 것은 아닙니다.

Versioning을 켜지 않으므로 object 삭제는 영구적입니다. PC 원본을 끝까지 보관하고 민감한 파일은 사용하지 않습니다. 공개 접근이나 추가 연동은 설정하지 않습니다.

## 실습구성

이번 새 private Standard bucket 하나와 `lab/payload.txt` 하나를 사용합니다. PC 원본의 이름은 `payload.txt`입니다. `lab/`는 object 이름의 prefix이며 실제 디렉터리를 만드는 작업이 아닙니다. 다운로드 파일은 원본과 다른 폴더에 저장하여 원본을 덮어쓰지 않습니다.

## 실습절차

### 1. 작은 원본 만들기 — PC

이번 실습 폴더에 텍스트 편집기로 `payload.txt`를 만들고 다음 한 줄을 저장합니다. 일반 텍스트 UTF-8을 사용하고 원본 경로를 기억합니다. 편집기별 줄바꿈 차이가 있으므로 특정 byte 크기를 억지로 맞출 필요는 없습니다.

```text
OCI Object Storage getting started
```

파일을 다시 열어 위 한 줄을 확인합니다. 실습이 끝날 때까지 이 PC 원본은 남겨 둡니다.

### 2. 새 private Standard bucket 만들기 — Console

**Storage → Object Storage & Archive Storage → Buckets**에서 선택 Region·Compartment를 확인하고 **Create bucket**을 누릅니다.

| 필드 | 값 | 이유 |
|---|---|---|
| Bucket name | 자동 생성된 이름 그대로 또는 고유한 `personal-lab-...` | 이름 충돌 없이 이번 bucket 구분 |
| Default storage tier | `Standard` | 올린 파일을 즉시 읽기 |
| Enable auto-tiering | 해제 | 자동 tier 변경 없이 시작 |
| Enable object versioning | 해제 | 이번 파일 하나만 보관 |
| Encryption | `Encrypt using Oracle-managed keys` | 별도 Vault/key 생성 없음 |

나머지는 추가 기능 없이 진행합니다. Bucket Scope=`NAMESPACE`, Emit object events·Resource logging·Uncommitted multipart uploads cleanup은 해제 상태로 두고 lifecycle rule을 추가하지 않습니다. 이 옵션들의 설명은 첫 파일 사용 뒤의 [추가 옵션 알아보기](#추가-옵션-알아보기)에 있습니다.

Create bucket을 한 번 누릅니다. 방금 생성한 상세에서 **Visibility=`Private`, Storage tier=`Standard`, Versioning=`Disabled`, Auto-tiering=`Disabled`**, Objects 목록이 비어 있는지 확인합니다. 이름·Region·Compartment를 메모합니다. private이 아니면 업로드 전에 이번 bucket을 private로 고치고 다시 확인합니다. 생성 실패면 오류와 Buckets 목록을 읽고 중복 생성하지 않습니다.

### 3. PC 원본 올리기

같은 bucket의 **Objects → Upload**에서 Object name prefix=`lab/`, Storage tier=`Standard`, **Choose files from your computer**로 PC 원본 `payload.txt` 하나를 선택합니다. Upload를 누릅니다.

성공 상태와 목록의 정확한 object name=`lab/payload.txt`, Standard tier, 실제 크기를 확인합니다. 크기는 PC 원본의 실제 크기와 같아야 합니다. overwrite 경고가 나오면 취소합니다. 빈 새 bucket에 처음 올리는 흐름이므로 다른 object를 덮어쓰지 않습니다.

### 4. 내려받아 내용 비교하기

**Objects → lab/payload.txt → Actions → Download**를 선택합니다. 브라우저가 저장한 파일을 원본과 다른 다운로드 폴더에서 열고 `OCI Object Storage getting started` 한 줄이 같은지 비교합니다. private bucket의 파일을 로그인된 Console에서 받는 것이므로 public으로 바꿀 필요가 없습니다. 내용이 같으면 핵심 실습은 끝났습니다. 선택 연습을 하지 않으려면 [리소스정리](#리소스정리)로 이동합니다.

### 5. 선택: 삭제 후 재업로드하기

PC 원본이 남아 있고 정상으로 열리는 것을 확인한 뒤 **Objects → lab/payload.txt → Actions → Delete**에서 정확한 이름을 확인하고 삭제합니다. Objects 목록을 정상 새로고침하고 해당 이름이 없는지 확인합니다. 목록 조회가 실패하면 새로고침이 정상으로 끝난 뒤 삭제 여부를 확인합니다.

3단계와 같은 PC 원본·`lab/` prefix·Standard로 한 번 더 Upload합니다. 같은 이름과 원본 크기를 읽고 다시 Download하여 별도 파일에서 내용을 확인합니다. 원본이 없으면 삭제하지 말고 먼저 다운로드를 확보합니다. 이 재업로드는 PC 원본으로 새 object를 만드는 것이며 삭제된 version을 복원하는 기능이 아닙니다.

### 결과검증과 문제해결

핵심 결과는 private Standard bucket의 `lab/payload.txt`와 PC에 내려받은 파일의 한 줄이 같은 것입니다. 5단계도 선택했다면 삭제 후 이름이 사라지고, 재업로드 후 다시 내려받은 내용이 같은지 확인합니다.

- bucket이 안 보이면 Region과 Compartment를 확인합니다. 같은 이름의 다른 bucket을 임의로 사용하지 않습니다.
- object 이름이 다르면 prefix `lab/`와 파일명 `payload.txt`가 합쳐지는지 확인합니다. 이번 잘못 올린 object만 삭제하고 다시 올립니다.
- 다운로드 파일이 안 보이면 브라우저 다운로드 목록/폴더를 확인합니다. 원본을 덮어쓰지 않습니다.
- 내용이 다르면 선택한 PC 파일과 다운로드 대상을 확인합니다. 잘못 올린 이번 object만 삭제한 후 정확한 원본을 올립니다.
- 삭제가 거부되면 오류와 현재 설정을 읽습니다. retention을 임의로 해제하거나 권한을 무작정 넓히지 않습니다.

### 리소스정리

기본은 지금 object와 bucket을 삭제하는 것입니다. **선택: 같은 세션에서 [Service Gateway](../Networking/ServiceGateway.md#사전준비)로 이어가려면**, NAT의 private VM 준비와 HTTPS 시험까지 완료했고 bucket이 그 VCN과 같은 Region인지 먼저 확인합니다. 그때만 이번 bucket·`lab/payload.txt`와 PC 원본을 잠시 남기고 SGW로 이동합니다. 이 Object Storage 실습 자체에는 VM이 필요하지 않습니다. SGW를 마치면 PAR를 먼저 삭제한 뒤 이 목록으로 돌아와 object와 bucket을 삭제합니다. 준비가 없거나 다음 실습을 하지 않으면 아래에서 바로 정리합니다.

1. 필요한 PC 원본/다운로드 파일을 남긴 뒤 **Objects → 이번 lab/payload.txt → Actions → Delete**를 수행합니다. 목록을 새로고침하여 비어 있는지 확인합니다.
2. **Buckets → 이번 bucket의 Actions → Delete**에서 대상 이름을 다시 확인하고 삭제합니다. 이번 실습에서 만들지 않은 object/version/PAR/replication 항목이 나오면 취소하여 대상을 재확인합니다. bucket 삭제는 되돌릴 수 없습니다.
3. 같은 Region·Compartment에서 목록을 정상 새로고침하여 이번 bucket이 없는지 확인합니다. 목록 조회가 실패하면 정상으로 다시 열린 뒤 삭제 여부를 확인합니다. Compartment는 삭제하지 않습니다.
4. PC의 이번 파일만 필요 없을 때 정리합니다. 전체 Downloads 폴더를 지우지 않습니다.

### 추가 옵션 알아보기

첫 파일을 사용한 뒤 필요할 때만 읽습니다. 이번 실습에서는 설정을 바꾸지 않습니다.

- **Bucket Scope**: `NAMESPACE`는 tenancy·Region 안에서 이름이 고유합니다. `REGION`은 Region 전체에서 고유한 이름과 virtual-hosted 접근을 지원하며, 변경 후 `NAMESPACE`로 되돌릴 수 없습니다.
- **Versioning / Auto-tiering**: 이전 object version 보관이나 자동 tier 변경을 위한 기능입니다. 이번에는 둘 다 사용하지 않습니다.
- **Events / Logging / Multipart cleanup**: 이벤트 연동·로그 수집·완료되지 않은 분할 업로드의 자동 정리를 위한 옵션입니다. 이번 작은 파일의 첫 사용에는 필요하지 않습니다.
- **큰 파일 업로드 권한**: 공식 업로드 문서에 따르면 Console은 64 MiB보다 큰 object에 multipart를 사용하며 `OBJECT_CREATE`와 `OBJECT_OVERWRITE`가 필요합니다. 이번 한 줄 파일을 큰 파일로 바꾸지 않습니다.

### 공식문서

- [Creating a Bucket](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm)
- [Uploading an Object](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm)
- [Downloading an Object](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_download_an_object_from_a_bucket.htm)
- [Deleting an Object](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_delete_objects_from_a_bucket.htm)
- [Deleting a Bucket](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_delete_a_bucket.htm)
