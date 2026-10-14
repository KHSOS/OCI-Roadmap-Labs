# Long-running Connections

Long-running Connections는 하나의 연결을 오래 유지하는 애플리케이션의 특성이다. 매 요청마다 연결을 새로 여는 방식과 달리 database session, streaming, 지속적인 TCP 통신에서는 flow가 같은 backend로 유지되는 것이 중요하다. NLB는 수일·수개월 수준의 장기 연결에 최적화된 서비스로 설명되지만 무조건적인 연결 생존 보장은 아니다.

## 역할과 구성

연결 수명과 idle timeout을 구분한다. 연결이 오래 살아 있어도 계속 traffic이 흐르면 idle하지 않다. NLB는 TCP/UDP flow의 양방향 traffic을 추적하며 idle timeout을 넘긴 flow를 제거할 수 있다. 공식 기본 idle timeout은 TCP 6분, UDP 2분이고 listener에서 조정할 수 있다.

## 사용 예시

지속적으로 메시지를 교환하는 TCP session은 장기 연결이면서 active flow다. 잠시 아무 데이터도 보내지 않는 client는 별도의 idle 조건을 만나며, client 자체 read timeout이나 OS keepalive 설정도 독립적으로 작용한다. 서버가 재시작되어 연결이 끊겼다면 애플리케이션은 새 연결과 필요한 session 상태 재구성을 고려해야 한다.

## 선택 기준과 제약

Flow affinity는 같은 연결의 backend 배치를 뜻하며 장애 후 기존 socket이 새 서버에 자동 복원된다는 뜻이 아니다. Idle timeout 후 도착한 TCP packet은 버려지고 UDP의 후속 packet은 새 flow로 취급될 수 있다. 주기적 application message를 주고받은 사례는 무송신 idle 내구성을 설명하지 못한다. HA 요구에는 backend 수, 상태 보관, client retry와 중복 처리까지 함께 고려한다.

## 관련 개념

- [Network Load Balancer](00_Network_Load_Balancer.md)
- [Layer 3 / Layer 4](01_Layer_3_Layer_4.md)
- [Health Check Policies](03_Health_Check_Policies.md)

## 더 읽기

- [Oracle — Overview of Flexible Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
- [Oracle — Introduction to Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/introduction.htm)

설정과 실습 절차는 [보존된 Long-running Connections Hands-on 문서](../../Hands-on/Networking/Original-Provisional/09_Network_Load_Balancer/04_Long_running_Connections.md)에서 읽을 수 있다.
