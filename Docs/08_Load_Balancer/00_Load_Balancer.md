# Load Balancer

OCI Load Balancer는 하나의 entry point로 들어오는 트래픽을 VCN에서 도달 가능한 backend server들에 분배하는 managed reverse proxy다. Public 또는 Private IP와 provisioned bandwidth를 선택하며 TCP와 application-level HTTP 계열 트래픽을 처리한다.

## 어디에 쓰는가

여러 web server 앞에 안정적인 접속 주소를 두고 backend health에 따라 정상 server로 요청을 전달하거나, application을 확장하면서 client가 개별 server 주소를 알지 않도록 할 때 사용한다. 하나의 backend만 있어도 frontend/backend 경로를 구성할 수 있지만 backend 장애 때 서비스를 이어받을 다른 server가 없으므로 application 이중화를 얻는 것은 아니다.

## 구성 요소와 요청의 관계

Client 요청은 LB IP의 **Listener** protocol·port로 들어온다. Listener는 기본 Backend Set과 선택적인 routing 설정을 통해 요청을 연결한다. **Backend Set**은 server 목록, 분산 policy, health check policy를 묶으며 각 **Backend Server**는 IP와 application port로 식별된다.

예를 들어 frontend HTTP port 80과 backend application port 8080은 서로 다를 수 있다. Health Check는 backend의 port 또는 URI 응답을 별도로 평가하며 실패한 server를 rotation에서 제외하고 회복하면 돌려놓는다. Listener 존재, backend health, client의 정상 응답은 서로 다른 관계다.

## Public과 Private 선택

Public LB는 인터넷에서 접근 가능한 public IP를 제공하며 public Subnet과 internet 경로가 필요하다. Private LB는 Subnet의 private IP를 entry point로 사용한다. Private 주소에는 연결된 private network 경로가 필요하며 Peering·FastConnect 등의 접근 방식도 고려할 수 있다. 어느 방식이든 NSG/Security List와 route, backend OS/application 조건이 맞아야 한다.

Frontend가 public이라고 backend도 public일 필요는 없다. 인터넷 공개가 필요 없는 내부 서비스에는 Private 선택이 노출 범위를 줄인다. IP의 공개 여부와 application 인증·TLS는 다른 보안 축이며 NSG의 CIDR 규칙은 HTTP path 접근 제한을 대신하지 않는다.

## 분산·가용성과 TLS의 선택

Round Robin·Least Connections·IP Hash는 요청 분산 방법의 선택이다. Session Persistence는 한 logical client의 요청을 같은 backend로 보내는 기능이며 모든 workload에 필수는 아니다. Pool과 연결한 자동 backend 등록 역시 application health와 traffic 분산이 준비되어야 의미가 있다.

HTTPS에서 LB가 TLS를 terminate하고 backend에는 평문을 보내는 방식, backend 연결도 암호화하는 방식, TCP Listener로 TLS를 backend까지 tunnel하는 방식은 다르다. Client→LB의 암호화만으로 LB→backend도 암호화된다고 보면 안 된다.

LB 서비스는 primary/standby 구성을 제공하지만 AD 장애 보호 범위는 regional/AD-specific Subnet과 Region의 AD 수에 따라 다르다. 여러 backend와 실패 도메인 분산을 함께 고려해야 application 가용성을 높일 수 있다.

## 용량과 제한

Flexible shape는 ingress와 egress를 합친 사전 provisioned bandwidth의 최소·최대 범위를 정한다. 이는 한 client의 throughput 보장치가 아니다. 요청량뿐 아니라 연결 수·timeout·보안 규칙의 state 추적과 backend 성능도 영향을 준다.

현재 공식 overview는 LB당 Backend Set 16개, Listener 16개, virtual hostname 16개, Backend Set당 server 512개와 LB 전체 server 512개 제한을 설명한다. Backend Set 수를 곱해서 전체 server 한도를 계산하면 안 된다. AD-specific LB와 regional LB는 서로 변환할 수 없다. Service Limit·quota와 실제 사용시간·bandwidth 및 관련 logging·VM·전송 비용도 별도 고려 대상이다.

## 함께 읽기

- [Listeners](01_Listeners.md)
- [Backend Sets](02_Backend_Sets.md)
- [Health Checks](03_Health_Checks.md)
- [SSL Certificates](04_SSL_Certificates.md)
- [보존된 Hands-on 초안](../../Hands-on/Original-Provisional/08_Load_Balancer/00_Load_Balancer.md) — 실습 절차는 별도 자료에서 다룬다.

## 공식 참고 문서

- [Overview of Load Balancer](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/balanceoverview.htm)
- [Load Balancer Types](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/load_balancer_types.htm)
- [Load Balancer Concepts](https://docs.oracle.com/en-us/iaas/Content/Balance/Concepts/balanceoverview_topic-Load_Balancing_Concepts.htm)
- [Listeners for Load Balancers](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners.htm)
- [Creating a Load Balancer Listener](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managinglisteners_topic-Creating_Listeners.htm)
- [Creating Instance Pools](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm)
