# Export Options

Export Options는 하나의 Export를 사용하는 NFS client의 접근 수준을 정하는 ordered entry 목록이다. 각 entry는 source IP 또는 CIDR, Read Only/Read Write, source port 조건, 인증 방식, UID/GID 매핑 등을 담는다. Export당 최대 100개 entry를 지원한다.

## source와 첫 일치 규칙

File Storage는 client의 source IP와 처음 일치하는 entry 하나만 적용한다. 가장 좁은 CIDR을 자동으로 우선하는 방식이 아니다. 예를 들어 넓은 subnet의 Read Only entry가 앞에 있고 그 안의 host Read Write entry가 뒤에 있으면 host도 Read Only가 된다.

일치하는 entry가 없으면 해당 Export는 그 client에게 보이지 않는다. 그러나 같은 File System의 다른 Export 또는 다른 Mount Target에 허용 entry가 있으면 우회 접근이 남는다. 한 Export의 거부와 File System 전체의 거부는 다르다.

## 주요 선택의 의미

| 선택 | 역할과 주의점 |
|---|---|
| Source | 실제 요청 source IP에 대한 host 또는 network 범위다. 사용자 이름이나 Group을 뜻하지 않는다. |
| Access | Read Only는 서버 쪽 쓰기를 제한한다. Read Write도 파일의 POSIX 권한을 무시하지 않는다. |
| Ports | Privileged는 source port 1–1023만 허용한다. 서버 destination port 조건과 다르며 client도 reserved source port를 사용할 수 있어야 한다. |
| Squash | None은 매핑하지 않고, Root는 root UID/GID 0/0을, All은 모든 사용자와 group을 anonymous UID/GID로 매핑한다. |
| Squash UID/GID | 매핑 결과의 숫자 identity이며 기본값은 65534다. 해당 identity의 실제 파일 권한이 접근 결과를 좌우한다. |

AUTH_SYS에서는 client가 보내는 숫자 UID/GID가 중요하다. 서로 다른 client의 동일 사용자 이름이 동일 숫자 identity를 보장하지 않으며, Root squash는 모든 사용자를 차단하는 기능도 아니다. 신뢰할 수 없는 client를 network와 source 범위로 제한하는 이유다.

## 인증 방식과 anonymous 접근

Allowed Authentication Options의 SYS는 NFSv3 UNIX 인증이다. KRB5는 Kerberos 인증, KRB5I는 인증과 무결성, KRB5P는 인증·무결성·전송 중 privacy를 제공한다. Kerberos 사용에는 Mount Target과 LDAP 등 관련 구성이 필요하며 목록 선택만으로 준비가 끝나는 것은 아니다. 허용 목록이 비면 모든 인증 방식이 제거되어 접근을 잃을 수 있다.

Anonymous Access는 Kerberos의 LDAP 사용자 조회 실패 시 Squash UID/GID로 처리할지 정한다. AUTH_SYS의 Root squash를 켜거나 끄는 스위치가 아니다. Kerberos를 사용하지 않는 Export에서는 Not Allowed가 공식 권장 선택이다.

## 기본값과 보안 경계

Console의 secure option 선택, API/CLI에서 ClientOption 배열을 명시했는지에 따라 privileged port 기본값이 다르다. 기본 IPv4 Export 예시는 모든 source에 Read Write와 Squash None을 허용하므로 기본값을 제한된 설계로 착각하면 안 된다. 필요한 source·access·auth·squash를 명확히 선택하는 것이 중요하다.

Network 보안 규칙은 endpoint까지의 통신을, Export Options는 Export 접근을, POSIX 권한은 개별 파일 접근을 제어한다. client의 로컬 Read Only mount와 서버 Export의 Read Only도 서로 다른 제약이다. 이 층들을 혼동하면 접근 실패의 원인이나 실제 보호 범위를 잘못 이해하게 된다.

## 함께 읽기

- [File Storage](00_File_Storage.md)
- [Exports](02_Exports.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/05_File_Storage/03_Export_Options.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Working with NFS Exports and Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/exportoptions.htm)
- [Overview of File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
