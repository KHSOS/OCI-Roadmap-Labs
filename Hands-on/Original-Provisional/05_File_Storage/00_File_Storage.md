> **기존 실습 초안 — 미검증**: 원래 93개 문서의 절차를 보존한 자료입니다. 새 튜토리얼의 품질 승인이나 OCI 실행 검증을 뜻하지 않습니다.

# File Storage

> 통합 품질 gate: **BLOCKED / NOT_ACCEPTED**. 복사·구조 검사만으로 전체 실습 품질을 승인하지 않습니다. 실제 OCI 실행 없음; 모든 예상 출력은 패턴이며 실측값이 아닙니다. 검토 잔여사항은 [통합 상태](../../../Docs/README.md)를 확인하세요.

> **저술 완료 · OCI 실행 미검증** — Oracle 공식 작업 문서를 근거로 작성한 한국어 hands-on입니다. Oracle 공인 교육/실행 인증이 아니며 기대 결과와 실제 결과를 구분합니다. 비용 승인과 실제 선행 산출물 확인 후에만 실행하세요.

## 학습목표

- 승인된 전용 FSS subnet/security와 File System/표준 Mount Target/Export 한 세트를 생성한다.
- private-vm에서 exact MT:/lab NFSv3 mount를 확인한 뒤 작은 proof를 쓰고 unmount/remount 동일 bytes를 검증한다.

완료 기준은 아래 실제 생성·설정·요청 결과로 판정한다. 이 목표는 수행할 작업이며 OCI에서 관찰한 결과가 아니다.

## 대응 Roadmap 항목

- 항목명: **File Storage**
- Roadmap 경로: **File Storage**
- Node ID: `service-file-0`
- 상위 항목: 없음 — 최상위 학습 항목
- 문서 범위: 공식 개념 목표와 실제 구성·허용/불허 검증·정리 절차. 아래 실습은 저술되었으나 OCI에서 실행 검증되지 않았습니다.
- [전체 항목 목록](../../../Docs/README.md)

## 사전준비

22의 기존 B_PRIVATE_IP/AD/SSH·24 client 경로,19 VCN_ID,20 현재 CIDR 계획,25/26 보안 owner 반환을 받는다.30 설치 준비는 nfs-utils 설치가 필요할 때만 사용한다. MT_PRIVATE_IP/FS_ID/MT_ID/EXPORT_ID/EXPORT_SET_ID는 아래 생성 후 출력이며 미리 추측하지 않는다. exact B→MT NFSv3 mount/source/rw와 proof hash를44–47에 인계한다.

### 기준 계약과 실제 단계 선행조건

[공통 baseline-contract.json](../../../baseline-contract.json)의 OS/CIDR/소유권을 따른다. Oracle Linux 9 / `opc` / `dnf` 외의 숨은 Ubuntu 또는 probe VM은 만들지 않는다. OCID/IP의 null은 실행 전 반드시 실제 출력으로 채우는 **실행 차단 입력**이다. approval/capacity/quota/identity 미확인은 완료가 아니다.

- [24.ready Route Tables — FUNCTIONAL_COMPLETION](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md#baseline-24-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [22.ready Private Subnets — FUNCTIONAL_COMPLETION](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md#baseline-22-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [25.ready Network Security Groups — FUNCTIONAL_COMPLETION](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md#baseline-25-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [26.ready Security Lists — FUNCTIONAL_COMPLETION](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/07_Security_Lists.md#baseline-26-ready): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.
- [30.prepare NAT Gateway — PRE_CREATE_OR_INSTALL_PREPARATION](../../Networking/Original-Provisional/03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md#nat-installation-preparation): 표시된 FUNCTIONAL_COMPLETION은 해당 linked ready 단계의 실제 검증/반환이 필요하다. PRE_CREATE_OR_INSTALL_PREPARATION으로 표시한 링크만 준비 출력까지 요구한다. 필요한 OCID/주소만 인계한다.

공유 A/B consumer는 INSTANCE_A_ID에 owner10 vm-a의 실제 OCID, INSTANCE_B_ID에 owner22 private-vm의 실제 OCID, `A_PUBLIC_IP`, `A_PRIVATE_IP`, `B_PRIVATE_IP`를 실제 생성 응답에서 전달받는다. 해당 주제에 필요하지 않은 VM/변수는 생성·요구하지 않는다.

- [Route Tables](../../../Docs/03_Virtual_Cloud_Network_VCN/05_Route_Tables.md) **RT-ready**의 SSH/Linux VM 보존, [Private Subnets](../../../Docs/03_Virtual_Cloud_Network_VCN/03_Private_Subnets.md) **private Subnet/Public IP 금지 완료**. mount client는 `private-vm`를 재사용하고 NFS 패키지를 설치할 Internet/NAT 경로가 필요하면 [NAT Gateway](../../../Docs/03_Virtual_Cloud_Network_VCN/11_NAT_Gateway.md)의 **nat-installation-preparation** 설치 경로를 일시 사용한다.
- [Network Security Groups](../../../Docs/03_Virtual_Cloud_Network_VCN/06_Network_Security_Groups.md)/[Security Lists](../../../Docs/03_Virtual_Cloud_Network_VCN/07_Security_Lists.md)의 NSG/SL 이해 완료. 아래 전용 mount-target Subnet·NSG 생성은 Network 관리자 승인 작업이다. File Storage 생성 비용 승인·Region/AD/한도 확인 전에는 진행하지 않는다.
- Linux client와 File System/Mount Target은 동일 AD를 선택해 cross-AD 비용/지연을 최소화한다.44~47까지 선택된 consumer와 승인 expiry를 기록하고 같은 File System/Mount Target/Export/VM을 보존한다. 선택47 Snapshot 삭제/반환 전 owner43 최종 삭제 금지.

## 권한

- `GetVcn`: `VCN_READ`; `ListSubnets`: `SUBNET_READ` — [공식 core API 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm). CIDR gate의 network 관리자는 VCN 및 모든 inventory 대상 compartment 읽기 scope를 승인받는다. 목록 권한 누락이면 전체 inventory라고 취급하지 않는다.

### 실제 task/API별 공식 권한 본문 대조

단계6 Export Options 편집은 `UpdateExport` 작업이다. live 공식 operation 표에는 그 행이 없으므로 `UpdateExportSet`의 권한을 대신 주장하지 않는다. exact UpdateExport 권한 mapping은 미확정이며 실행 전 관리자 확인이 필요하다. 단계6 이전 [46 실행 승인 계약](03_Export_Options.md#권한)에 actual principal/EXPORT_ID/Compartment/UpdateExport 편집·복구 범위/관리자·approval UTC·expiry/독립 복구담당자를 기록하고 해당 작업 권한을 명시 승인받는다. 미승인 mutation은 실행 금지다.

아래 공식 operation 표에서 이 실습의 읽기·변경·정리 권한을 대조한다. family grant/일부 dependency 인용은 검증된 minimum policy가 아니며 실제 principal과 resource scope를 관리자가 승인해야 한다.

- `AddNetworkSecurityGroupSecurityRules`: NETWORK_SECURITY_GROUP _UPDATE_SECURITY_RULES and NETWORK_SECURITY_GROUP_INSPECT if writing a rule that specifies a network security group as the source (for ingress rules) or destination (for egress rules) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateExport`: EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_EXPORT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `CreateFileSystem`: FILE_SYSTEM_CREATE Cloning a file system also requires FILE_SYSTEM_CLONE Associating the file system with a snapshot policy also requires FILE_SYSTEM_UPDATE_FILESYSTEM​_SNAPSHOT_POLICY — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `CreateMountTarget`: MOUNT_TARGET_CREATE + VNIC_CREATE(vnicCompartment) + SUBNET_ATTACH(subnetCompartment) + VNIC_ATTACH(vnicCompartment) + PRIVATE _IP_CREATE(subnetCompartment) + PRIVATE_IP_ASSIGN(subnetCompartment) + VNIC_ASSIGN(subnetCompartment) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `CreateNetworkSecurityGroup`: NETWORK_SECURITY_GROUP_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateRouteTable`: ROUTE_TABLE_CREATE and VCN_ATTACH and INTERNET_GATEWAY_ATTACH (if creating a route rule that uses an internet gateway as a target) and DRG_ATTACH (if creating a route rule that uses a DRG as a target) and PRIVATE_IP _ROUTE_TABLE_ATTACH (if creating a route rule that uses a private IP as a target) and LOCAL_PEERING _GATEWAY_ATTACH (if creating a route rule that uses an LPG as a target) and NAT_GATEWAY_ATTACH (if creating a route rule that uses a NAT gateway as a target) and SERVICE_GATEWAY_ATTACH (if creating a route rule that uses a service gateway as a target) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSecurityList`: SECURITY_LIST_CREATE and VCN_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `CreateSubnet`: SUBNET_CREATE and VCN_ATTACH and ROUTE_TABLE_ATTACH and SECURITY_LIST_ATTACH and DHCP_ATTACH — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).
- `DeleteExport`: EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_UNEXPORT — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `DeleteFileSystem`: FILE_SYSTEM_DELETE — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `DeleteMountTarget`: MOUNT_TARGET_DELETE + VNIC_DELETE(vnicCompartment) + SUBNET_DETACH(subnetCompartment) + VNIC_DETACH(vnicCompartment) + PRIVATE_IP_DELETE(subnetCompartment) + PRIVATE_IP_UNASSIGN(subnetCompartment) + VNIC_UNASSIGN(vnicCompartment) — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `GetExport`: EXPORT_SET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `GetFileSystem`: FILE_SYSTEM_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `GetMountTarget`: MOUNT_TARGET_READ — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm).
- `UpdateVnic`: VNIC_UPDATE If adding or removing the VNIC from a network security group, also need NETWORK_SECURITY_GROUP_UPDATE_MEMBERS and VNIC_ASSOCIATE_NETWORK_SECURITY_GROUP — [공식 해당 표](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm).

관리자가 실제 작업자 Group OCID와 File Storage/Network Compartment OCID를 넣어 다음 정책을 승인한다. 같은 Compartment를 쓰더라도 두 권한군은 별개다.

```text
Allow group id <LAB_OPERATOR_GROUP_OCID> to manage file-family in compartment id <FSS_COMPARTMENT_OCID>
Allow group id <LAB_OPERATOR_GROUP_OCID> to use virtual-network-family in compartment id <NETWORK_COMPARTMENT_OCID>
```

첫 문장은 [Common Policies — Let users create, manage, and delete file systems](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/commonpolicies.htm)의 Compartment 제한 예제다. Mount Target 생성/삭제에는 File Storage 권한만으로 부족하다. [File Storage policy reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm)의 CreateMountTarget는 `MOUNT_TARGET_CREATE`, `VNIC_CREATE`, `SUBNET_ATTACH`, `VNIC_ATTACH`, `PRIVATE_IP_CREATE`, `PRIVATE_IP_ASSIGN`, `VNIC_ASSIGN`를 요구한다. CreateExport는 `EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_EXPORT`, DeleteExport는 `EXPORT_SET_UPDATE + FILE_SYSTEM_NFSv3_UNEXPORT`다. NSG/전용 Subnet 생성은 Networking 관리자가 수행한다. Linux의 `sudo mount`는 호스트 권한이고 IAM 정책으로 NFS 파일의 POSIX 권한을 주는 것이 아니다. NFS 데이터 접근은 네트워크·Export Options·UID/GID/POSIX 조건을 따로 만족해야 한다.

## 비용및주의사항

**선택 유료 생성 트랙**이다. File Storage 생성 전 Region 가격표와 사용량/한도를 확인하고 비용 승인자가 허용한 시간 안에서 실행한다. 승인 없이 실행하지 않는다. File System 하나, 표준 Mount Target 하나, 1 KiB 미만 파일만 만든다. High Performance Mount Target·Snapshot Policy·Replication·두 번째 Mount Target·고객관리 키는 만들지 않는다. 43~46은 같은 자원을 이어 쓰며 관찰만으로 실제 mount/RW 테스트를 대체했다고 보고하지 않는다. 승인 거절이면 이 트랙은 미실행으로 표시한다. File Storage는 실제 사용량/기능별 과금 조건을 확인해야 하며 무료라고 단정하지 않는다.

## 실습구성

### 이번 workshop 시나리오와 입력 계약

전용 private Subnet과 File System/MT/Export 한 이벤트를 만들고 NFS write/remount한다.

|입력|값/출처|시점|실행 위치|선택 이유|
|---|---|---|---|---|
|`VCN_ID`|owner19의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`B_PRIVATE_IP`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`B_AD`|owner22의 actual output; `<...>` 대체 필수|실행 전 선행 완료값|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`MT_PRIVATE_IP`|owner43의 actual output; `<...>` 대체 필수|생성 후 readback|B guest|정확한 대상/소유관계 고정|
|`FS_ID`|owner43의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`MT_ID`|owner43의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|
|`EXPORT_ID`|owner43의 actual output; `<...>` 대체 필수|생성 후 readback|Console/사용자 전용 shell|정확한 대상/소유관계 고정|

|Console field|실제 입력/선택|이유|
|---|---|---|
|FS AD / MT subnet / Export source / ports|same AD as B; lab-fss10.77.30.0/24; /lab B/32 RW Privileged SYS squashNone|이번 기능/실패 원인을 한 변수로 분리|

**생성·변경 범위:** lab-fss:Subnet,rt-fss:RouteTable,sl-fss:SecurityList,nsg-fss-mt:NetworkSecurityGroup,nsg-fss-client:NetworkSecurityGroup,fs-lab:FileSystem,mt-lab:MountTarget,lab-export:Export. **완료 비교:** mount actual MT:/lab NFSv3; write/remount same hash. 기대 패턴이며 OCI 실측값이 아니다. **정리:** 본 주제의 변경만 원복하고 전용 신규 resource만 dependency 역순으로 정리한다; 공유 자원은 creator와 consumer registry/expiry 기준으로 반환한다.

File System `fs-lab`, 표준 Mount Target `mt-lab`, Export `/lab` 하나를 **한 생성 이벤트**로 구성한다. 전용 regional private Subnet `lab-fss`, CIDR `10.77.30.0/24`, public IP 금지, blank external routes, VCN DNS 사용. NFS client B actual IP를 Export source `/32`로 제한한다. AUTH_SYS/Oracle-managed encryption, secure export, no snapshot/replication/LDAP/Kerberos.

## 실습절차

### 작업 순서 체크포인트
network 준비→FS/MT/export 생성→checked mount→파일 쓰기/remount comparison→consumer return→unmount/export/MT/FS 순서로 수행한다. existing VM/network는 소비자 실습에서 삭제하지 않는다. 단계 실패는 다음 생성으로 넘기지 않고 같은 target의 복구/반환으로 처리한다.

### 개별 완주 보정 — 생성·검증·소유권

MT_PRIVATE_IP는 Mount Target Details의 실제 private IPv4를 B shell에서 export한다. mount 전에 /mnt/oci-lab이 다른 mount인지, 비어 있지 않은 local directory인지 확인하고 기존 local 파일이 있으면 중단한다. findmnt 출력의 SOURCE가 actual MT:/lab이고 FSTYPE=nfs인 경우에만 쓰기한다. 무조건 mkdir 성공을 NFS 성공으로 보지 않는다.

### A. NFS endpoint용 Subnet와 Security

**첫 생성 전 owner43 abort/복구 station:** exact 신규 OCID/created-by43 flag를 제출마다 registry에 저장한다(기존 같은 이름은 소유하지 않음). 독립 Console 담당자와 최대120초 mount 요청 복구 창/전체 비용 expiry를 승인한다.8 실패/세션 손실에도 담당자는 원래 B/32 RW options·MT ACTIVE·통신을 복원/readback한다. deadline은 kernel 종료와 독립이다. 부분 생성 실패는 재Create하지 않고 승인 expiry 안에 실제 생성 목록을 읽어: owned test file(실제 NFS mount/rw 확인시에만) → 정상 client unmount/잔존 request 종료 → 생성된 Export → FS → 다른 Export 없는 MT → B의43 membership →43 NSG →43 subnet → 미참조 rt-fss/sl-fss 순으로 존재하는 owned 대상만 정리/readback한다. mount가 성공한 적 없어도 이 역순 abort를 따른다. D-state/미확인 mount면 endpoint 삭제 금지 및 아래 residual station으로 인계한다. 공유 A/B·기존 subnet/NSG는 삭제하지 않는다.

**A.1 제출 전 추가 CIDR 승인 gate (host: 관리자 Cloud Shell):** owner20의 현재 CONNECTED_CIDRS_JSON 전체 계획(없으면 조직 확인 []), VCN_ID/REGION과 신규 FSS_SUBNET_CIDR을 지정한다. 기존 subnet을 pagination 포함해 모두 조회한다. 실패/IPv6/overlap이면 rt-fss/sl-fss/subnet 생성 전 중단하고 새 계획을 승인받는다. [Subnet containment and non-overlap](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/create_subnet.htm).

```bash
for p in fss43-vcn-plan.json fss43-existing-subnets.json; do
  test ! -e "$p" && test ! -L "$p" || exit 1
done
set -C
export FSS_SUBNET_CIDR='10.77.30.0/24' 
export CONNECTED_CIDRS_JSON='<owner20_current_approved_JSON_array>'
oci network vcn get --vcn-id "$VCN_ID" --region "$REGION" > fss43-vcn-plan.json || exit 1
# 관리자가 VCN 전체 subnet inventory(다른 compartment 포함)를 Console에서 확인하고
# 전부 LAB_COMPARTMENT_ID에 속함을 승인한 경우에만 아래 list가 전체 inventory다.
# 다른 compartment subnet이 하나라도 있거나 조회권한 누락이면 이 gate에서 STOP.
oci network subnet list --all --compartment-id "$LAB_COMPARTMENT_ID" --vcn-id "$VCN_ID" --region "$REGION" > fss43-existing-subnets.json || exit 1
python3 - <<'PY'
import json,os,ipaddress
n=ipaddress.ip_network(os.environ['FSS_SUBNET_CIDR'])
v=json.load(open('fss43-vcn-plan.json'))['data']
s=json.load(open('fss43-existing-subnets.json'))['data']
assert n.version==4 and any(n.subnet_of(ipaddress.ip_network(x)) for x in v['cidr-blocks'])
assert all(x['vcn-id']==os.environ['VCN_ID'] for x in s)
for x in s:
 assert not n.overlaps(ipaddress.ip_network(x['cidr-block'])), 'STOP: existing subnet overlap'
connected=json.loads(os.environ['CONNECTED_CIDRS_JSON']); assert isinstance(connected,list)
for x in connected:
 other=ipaddress.ip_network(x)
 assert other.version==4 and not n.overlaps(other), 'STOP: connected plan overlap'
print('FSS candidate checked against current VCN/all existing subnets/approved connected plan')
PY
if [ "$?" -ne 0 ]; then exit 1; fi
```

후속 재진입 때 이미 lab-fss가 생성되어 있으면 이 신규 gate를 다시 생성 허가로 쓰지 않는다. 기존 owner43 exact OCID/CIDR을 readback해 재사용한다.

1. 관리자 **VCN → Subnets → Create Subnet**: `lab-fss`, regional, `10.77.30.0/24`, Private, VCN DNS/default DHCP, 외부 route 없는 전용 `rt-fss`, 명시적 ingress/egress 빈 `sl-fss`만 연결. 먼저 VCN → Route Tables → Create rt-fss(rule 없음), Security Lists → Create sl-fss(두 배열 비어 있음)를 생성하고 OCID를 기록한다. default SL을 연결하지 않는다. `/30`처럼 IP가 모자라는 CIDR은 사용하지 않는다. Mount Target은 내부 IP 세 개 여유가 필요하다.
2. **NSG → Create** `nsg-fss-mt`와 `nsg-fss-client`. MT ingress source `<B_PRIVATE_IP>/32`, **stateful TCP** destination **111**, **2048-2050**; **stateful UDP** destination **111**, **2048**. NFS 관련 source port는 전체 범위로 두고 Export secure option의 privileged source port를 통해 제한한다. 공식 문서는 source1-1023도 권장하므로 적용하려면 mount/RPC source 관찰 후 일치 여부를 확인한다.
3. B client NSG의 새 rule ID/기존 B VNIC NSG membership을 먼저 저장한다. **Compute → private-vm → Attached VNICs → primary VNIC → Network security groups → Edit**에서 기존 SSH용 NSG를 유지하고 `nsg-fss-client`를 추가/Save한다. exact B VNIC의 membership readback 전에는 NFS 시험을 시작하지 않는다. B client NSG stateful egress destination `<MT_PRIVATE_IP>/32`로 같은 TCP/UDP destination port를 넣는다. MT IP는 생성 후 실제 값으로 대체한다. B SSH 관리 rule은 보존한다. broad SL/NSG가 NFS를 허용하지 않도록 확인한다. NFS를 `0.0.0.0/0` source로 열지 않는다.

### B. File System/Mount Target/Export 생성
4. **Storage → File Storage → File Systems → Create file system**에서 **File system for NFS** 선택. Name `fs-lab`, 교육 Compartment, AD는 B와 동일, Encryption Oracle-managed, Attach Snapshot Policy off.
5. **Export information**: path `/lab`, **Use secure export options on**, LDAP off. **Mount Target**: Create new `mt-lab`, 교육 Compartment, `shared-vcn`, `lab-fss`, 표준/default shape, `nsg-fss-mt`, hostname `mtlab`(VCN DNS 사용), unused private IP 자동. High Performance upgrade를 하지 않는다. 생성/ACTIVE를 기다린다.
6. Export details **NFS Export Options → Edit**: 기존 broad source를 제거하고 source `<B_PRIVATE_IP>/32`, Ports **Privileged**, Access **Read/Write**, Squash **None**, allowed authentication **SYS**, Anonymous access **Not Allowed**를 명시한다. UID/GID 기본65534는 squash none에서 적용되지 않지만 기록한다. 이 선택은 root 권한 POSIX 학습을 단순화하는 격리 실습용이며46에서 RO/RW를 구분한다.
7. MT private IP로 step3 egress를 완성한다. File System/MT/Export OCID·AD·Export Set OCID·path를 저장한다.

### C-0. Oracle Linux9 NFS client / guest firewall gate

B guest는 **NFS client**이며 Managed MT가 server다. B에 nfs-server service를 시작하거나 guest ingress2049을 여는 `--add-service=nfs`를 수행하지 않는다. 이 AUTH_SYS/NFSv3 시험의 필요한 새 연결은 B→MT TCP111/2048/2049/2050 및 UDP111/2048이며 NSG egress와 MT ingress는 A.2/A.3의 정확한 표를 따른다. TCP/UDP return은 stateful이며 B의 SSH22는 유지한다. [OCI File Storage security rules — different subnets](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/securitylistsfilestorage.htm).

```bash
# B guest; NAT 설치 path/expiry는 owner30 승인 단계만 사용
rpm -q nfs-utils || { sudo dnf install -y nfs-utils || exit 1; }
command -v mount.nfs >/dev/null || exit 1
command -v rpcinfo >/dev/null || exit 1
export MT_PRIVATE_IP='<owner43_actual_private_IP_after_MT_Create>'
ip route get "$MT_PRIVATE_IP" || exit 1
if systemctl is-active --quiet firewalld; then
  sudo firewall-cmd --get-active-zones || exit 1
  # 실제 primary zone은 위 interface와 연결된 값으로 지정
  export FSS_CLIENT_ZONE='<actual_primary_interface_zone>'
  sudo firewall-cmd --zone="$FSS_CLIENT_ZONE" --list-all || exit 1
  sudo firewall-cmd --get-policies || exit 1
fi
# 실제 OUTPUT/policy가 새 TCP/UDP와 ESTABLISHED return을 차단하는지 관리자와 읽는다.
sudo nft list ruleset || exit 1
# 보안 owner가 OUTPUT 제한 없음/위 exact peer-port allow 확인한 뒤만 진행
if timeout -k 5s 15s rpcinfo -p "$MT_PRIVATE_IP"; then RC=0; else RC=$?; printf 'RPC preparation RC=%s; STOP before mount\n' "$RC"; exit 1; fi
```

기대 rpcinfo 결과의 NFS/mountd 프로그램·TCP/UDP port를 기록한다. firewall OUTPUT 제한이 있으면 임의 zone rich-rule이 outbound를 해결한다고 가정하지 말고 **host security owner가 해당 HOST-output policy의 MT/32·위 TCP/UDP ports만 별도 승인**한 exact rule ID/before/after/expiry/remove recipe를 전달할 때까지 BLOCKED_HOST_FIREWALL로 중단한다. 기본 unrestricted-output에서 guest rule 추가는0건이며 삭제도0건이다. 모든 query 오류는 mutation 허가가 아니다. SELinux/firewalld 전체 disable·permanent reload·전체 ruleset flush를 하지 않는다. NFS client mount 뒤 `systemctl is-active rpc-statd`로 NLM client 상태를 확인한다; `nfs-server` 활성 여부는 client 준비 기준이 아니다. [Oracle Linux9 NFS client](https://docs.oracle.com/en/operating-systems/oracle-linux/9/nfs/mounting_task.html).

**단계8 전 capture station:** **capture terminal을 먼저 시작하고 다른 B request terminal에서8의 mount/read를 실행한다.** server MT에는 SSH하지 않는다. B에서 `sudo timeout 25s tcpdump -ni any 'host <MT_PRIVATE_IP> and (port 111 or port 2048 or port 2049 or port 2050)'`로 RPC/NFS packet을 관찰한다. 데이터는 1KiB 미만.

### C. 실제 Linux mount와 공유 데이터
8. B의 Oracle Linux 9에서 NFS client를 준비하고 실제 mount를 수행한다. 공식 mount samples를 Console에서 복사해 IP/path/options와 대조한다.

```bash
set -euo pipefail
export MT_PRIVATE_IP='<owner43-actual-MT-private-IPv4>'
rpm -q nfs-utils || { printf "STOP: C-0 package readiness missing\n"; exit 1; }
MOUNTPOINT="/mnt/oci-lab"
# Exact mount query: rc 1 is no match; all other errors stop.
if test -L "$MOUNTPOINT"; then printf 'STOP: symlink target\n'; exit 1; fi
if test -e "$MOUNTPOINT" && ! test -d "$MOUNTPOINT"; then printf 'STOP: non-directory target\n'; exit 1; fi
if mounted=$(findmnt -rn -M "$MOUNTPOINT" -o TARGET); then
  printf 'STOP: occupied mountpoint %s\n' "$mounted"; exit 1
else
  rc=$?; test "$rc" -eq 1 || { printf 'STOP: mount inspection error %s\n' "$rc"; exit 1; }
fi
if test -d "$MOUNTPOINT"; then
  entries=$(sudo find "$MOUNTPOINT" -mindepth 1 -maxdepth 1 -print -quit) || { printf 'STOP: directory inspection failed\n'; exit 1; }
  test -z "$entries" || { printf 'STOP: nonempty local directory\n'; exit 1; }
fi
FSS43_MOUNT_DIR_CREATED=0
if ! test -d "$MOUNTPOINT"; then sudo mkdir "$MOUNTPOINT" || exit 1; FSS43_MOUNT_DIR_CREATED=1; fi
# 이 생성 flag와 exact path를 owner43 기록에 저장; 기존 빈 directory 소유권은 취득하지 않음
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,rw,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || { printf 'BLOCKED_MOUNT: independent owner recovery/residual station now\n'; exit 1; }
findmnt -rn -M /mnt/oci-lab -o SOURCE,FSTYPE
[ "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" ] || exit 1
[ "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs ] || exit 1
sudo test ! -L /mnt/oci-lab/proof.txt && sudo test ! -e /mnt/oci-lab/proof.txt || { printf 'STOP: proof fixture already exists\n'; exit 1; }
sudo timeout -k 5s 10s sh -c "printf 'oci-fss-proof\n' > /mnt/oci-lab/proof.txt"
sudo timeout -k 5s 10s sha256sum /mnt/oci-lab/proof.txt
sudo timeout -k 5s 20s umount /mnt/oci-lab
sudo timeout -k 5s 20s mount -t nfs -o vers=3,proto=tcp,nosuid,resvport,sec=sys,rw,retry=0 "$MT_PRIVATE_IP:/lab" /mnt/oci-lab || { printf 'BLOCKED_MOUNT: independent owner recovery/residual station now\n'; exit 1; }
test "$(findmnt -rn -M /mnt/oci-lab -o SOURCE)" = "$MT_PRIVATE_IP:/lab" || exit 1
test "$(findmnt -rn -M /mnt/oci-lab -o FSTYPE)" = nfs || exit 1
printf 'oci-fss-proof\n' | sudo timeout -k 5s 10s cmp - /mnt/oci-lab/proof.txt
sudo timeout -k 5s 10s sha256sum /mnt/oci-lab/proof.txt
```

**residual-kernel station (모든 timeout/실패 직후):** timeout20초+kill유예5초는 user request 상한이지 hard NFS kernel I/O 종료 보장이 아니다. request terminal 손실에도 담당자가120초 안에 options/통신을 먼저 복원한다. B 관리 terminal에서 `ps -eo pid,ppid,stat,args`와 exact `findmnt -rn -M /mnt/oci-lab`을 읽고 mount/helper/NFS request의 D-state/잔존 여부를 확인한다. 요청 종료 확인 뒤만 `sudo timeout -k 5s 20s umount /mnt/oci-lab`을 정상 수행/readback한다. D-state·timeout·query 오류는 BLOCKED_RECOVERY로 owner43/host 관리자에게 인계하며 새 mount/write/endpoint delete/강제 lazy unmount/reboot를 하지 않는다. 정상 요청 종료/정상 unmount의 exact 부재 확인 후에만 abort cleanup을 진행한다. timeout을 mount 성공/deny 증거로 쓰지 않는다.

공식 Linux task body의 `sudo yum install nfs-utils`는 Oracle Linux 9에서 dnf 호환 경로다. `soft,rw`는 사용하지 않는다. 패키지 설치 실패와 NFS mount 실패를 섞지 않도록 설치를 먼저 끝낸다.
9. 단계8 전에 시작한 bounded capture 결과를 확인한다. 추가 mount/write fence를 반복하지 않는다.

## 결과검증


### baseline-43-ready


완료 단계 **FSS-mounted**: File System/MT/Export ACTIVE, same AD, restricted source/security, findmnt source `MT:/lab`/NFSv3, write hash와 unmount/remount read 동일. mount point에 local file을 쓰는 false success를 방지하기 위해 write 전에 findmnt를 반드시 확인한다. deny 검증은46에서 같은 mount·POSIX 조건을 고정해 수행하며44/45에서도 경로별 negative를 확인한다.

**실행 기록(아직 미작성):** 실행자/Region/시각, exact resource OCID, 설정 전·후, 성공 출력, deny 오류·exit code/request-id, positive control, 정리 결과. 문서 검증은 실제 OCI 실행 성공을 뜻하지 않는다.

## 문제해결

timeout: MT ACTIVE/IP/Subnet route/MT ingress/client egress/host firewall/UDP RPC 확인. access denied: Export source IP·privileged port/resvport·SYS auth·path·options 순서. Permission denied: 실제 mount 후 UID/GID와 POSIX 권한 확인; IAM을 넓혀 NFS 파일 권한을 해결하지 않는다. mount command가 멈추면 network 복구 후 정상 unmount하고 `soft,rw`로 억지 변경하지 않는다.

## 리소스정리

owner43은 44–46 및 선택 [47 Snapshots](../../../Docs/05_File_Storage/04_Snapshots.md)의 실제 예약 consumer/owner/EXPIRY_UTC/비용 상한을 registry에 기록한다. 47이 선택되면 46은 B/32 RW와 mounted `/lab`을 복구해 File System/MT/Export OCID·mount source·실제 RW 증거를 47에 인계하며 공유 자원을 삭제하지 않는다. 47이 NOT_SELECTED이거나 예약 소비자가 없으면 즉시 정리한다. 승인 expiry 이후 idle 상태로 무기한 보존하지 않는다; 미완료 소비자는 중단/반환하고 새 승인을 받기 전 재개하지 않는다. 47의 snapshot/파일 복구·삭제 완료 또는 NOT_SELECTED 인계 후 **owner43만** test files 삭제 → 모든 client unmount → 모든 해당 Export 삭제 → File System 삭제 → MT 삭제 → B VNIC에서 이번 nsg-fss-client membership만 제거/readback(기존 SSH NSG 보존) → 이번 NSG/전용 Subnet/table 삭제를 수행한다. File System 삭제는 영구 데이터 손실이므로 test OCID 확인. Mount Target에 다른 File System export가 남으면 공유 MT를 삭제하지 않는다.

**owner43 삭제 readback/로컬 종료:**삭제 전에 fs/mt/export/NSG/subnet/rt-fss/sl-fss exact OCID 목록을 저장한다. 각 Delete 뒤 성공 응답만 믿지 말고 Console의 동일 Compartment/AD 목록을 새로 열어 exact OCID 부재/종료 상태를 확인한다. Export 전부 부재 → FS 부재 → MT 부재 → B 기존 NSG 배열 유지/이번 membership 부재 → 전용 NSG 부재 → lab-fss subnet 부재 → rt-fss와 **sl-fss 둘 다 부재**를 각각 기록한다. 권한/필터 오류는 부재 증거가 아니다. 공유 NSG/table/SL은 삭제하지 않는다. B의 mount dir은 exact mount 부재와 잔존 kernel 요청 없음 확인 후 **FSS43_MOUNT_DIR_CREATED=1 기록인 경우에만** `sudo rmdir /mnt/oci-lab`한다. 관리자 PC의 이번 fss43-vcn-plan.json/fss43-existing-subnets.json 및 실행 중 등록된 own snapshot/capture 파일만 제거한다. 기존 파일/symlink는 보존하며 다음 실행의 gate 출력은 noclobber로 생성한다.

## 공식문서

아래 Oracle 공식문서의 해당 항목 본문을 학습 근거로 사용합니다. 목표는 학습 후 설명·구분·확인할 내용이며 실제 OCI 실행 결과를 뜻하지 않습니다.

- [19] [Overview of File Storage — File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm)
  - 확인 범위: File Storage.

### 실습 작업을 지원하는 추가 공식 본문

- [실습근거 1] [Creating a File System](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/create-file-system.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 2] [Configuring VCN Security Rules for File Storage](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/securitylistsfilestorage.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 3] [Mounting File Systems From UNIX-Style Instances](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/mountingunixstyleos.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 4] [Working with NFS Exports and Export Options](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/exportoptions.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
- [실습근거 5] [Details for the File Storage Service](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/filestoragepolicyreference.htm) — 생성/수정·접근 제어·검증 관련 본문. 공식 본문을 수집해 저술 근거로 사용했으며 Console 메뉴·현재 CLI/이미지 호환성은 실행 시 재확인한다.
