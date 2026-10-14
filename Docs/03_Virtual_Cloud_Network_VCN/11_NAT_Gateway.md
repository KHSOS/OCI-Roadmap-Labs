# NAT Gateway

NAT Gateway는 공인 IPv4 주소가 없는 VCN 리소스가 인터넷으로 연결을 시작하고 그 응답을 받을 수 있도록 하는 관리형 gateway입니다. 인터넷에서 시작한 신규 연결을 private 리소스로 전달하는 서비스는 아닙니다.

## 역할과 사용 사례

Private subnet의 서버가 소프트웨어 업데이트나 외부 HTTPS API에 접근해야 하지만 인터넷 inbound endpoint는 필요하지 않을 때 사용합니다. 외부 서비스는 NAT Gateway의 공인 IP를 송신 주소로 보게 됩니다. NAT Gateway 자체는 웹 서버나 port-forwarding endpoint가 아닙니다.

## 다른 구성 요소와의 관계

- **VCN**: gateway는 한 VCN에 속합니다. Peering 상대 VCN이나 FastConnect/VPN으로 연결된 온프레미스 리소스는 이 NAT Gateway를 사용할 수 없습니다.
- **Subnet과 Route Table**: gateway가 존재하는 것만으로 경로가 생기지는 않습니다. 인터넷 목적지에 대한 route가 NAT를 가리키는 subnet에서 사용됩니다.
- **NSG와 Security List**: route는 패킷의 다음 경로를 정하고 security rule은 허용할 트래픽을 정합니다. NAT는 보안 규칙을 대신하지 않습니다.
- **Internet Gateway**: 인터넷에서 시작하는 연결도 필요한 public 리소스의 경로입니다. Private outbound-only 목적의 NAT와 역할이 다릅니다.
- **Service Gateway**: 지원되는 Oracle 서비스에 대한 사설 경로입니다. 일반 인터넷 접근을 제공하는 NAT와 대체 범위가 다릅니다.

## 옵션과 제약

| 항목 | 의미와 선택 기준 |
|---|---|
| 공인 IP | 새 주소를 할당하거나 새 NAT Gateway에 기존 reserved public IP를 지정할 수 있습니다. 외부 허용 목록에 안정적인 송신 주소가 필요한지 검토합니다. |
| Block traffic | 기본 생성 상태는 허용입니다. Gateway 전체 차단은 기존 route/security rule과 관계없이 통과 트래픽을 막습니다. |
| Gateway Route Table Association | 응답 트래픽을 firewall private IP 같은 중간 대상으로 보내는 고급 ingress routing 용도입니다. Subnet의 outbound route table과 구분합니다. |
| 프로토콜 | TCP, UDP, ICMP ping을 지원합니다. |
| 연결 수 | 단일 목적지 주소와 port에 대한 동시 연결은 최대 20,000개입니다. 애플리케이션의 집중된 outbound 연결 규모를 검토합니다. |
| 서비스 한도 | VCN별 gateway 한도가 있으며 증설 요청이 필요할 수 있습니다. 계정의 실제 한도를 확인합니다. |
| 삭제 의존성 | Gateway를 route target으로 참조하는 route table이 있으면 삭제할 수 없습니다. |

Public subnet을 private subnet으로 바꾸는 것은 지원되지 않습니다. NAT 도입과 subnet 유형 변경은 같은 작업이 아닙니다. 리소스의 public IP 존재 여부와 실제 route/security 구성을 별도로 판단합니다.

## 관련 튜토리얼

[NATGateway 튜토리얼](../../Hands-on/Networking/NATGateway.md)에서 자원 구성, 동작 확인과 정리 순서를 따라갈 수 있다.

## Oracle 공식 문서

- [NAT Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/NATgateway.htm) — 역할, 연결 시작 방향, VCN 범위, 지원 프로토콜, 연결 수, 주소 선택, route 관계와 삭제 의존성.
- [VCN Route Tables](https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/managingroutetables.htm) — route table의 역할과 설정.
- [Networking IAM policy reference](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/corepolicyreference.htm) — gateway 및 route 관리 권한.

[VCN 주제 목록](../README.md)
