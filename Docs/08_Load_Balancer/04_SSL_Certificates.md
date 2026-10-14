# SSL Certificates

## 인증서와 TLS 구간

SSL Certificates는 Load Balancer의 TLS 연결에서 상대의 신원을 확인하고 암호화된 통신을 구성하는 데 사용한다. OCI 문서의 SSL이라는 용어를 읽을 때 인증서 관리와 TLS 종료 위치를 함께 봐야 한다. 인증서를 등록했다는 사실과 클라이언트가 인증서 chain 및 접속 hostname을 신뢰한다는 사실은 다르다. [Oracle Load Balancer-Managed SSL Certificates](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingcertificates.htm)

## Frontend와 Backend의 보호 범위

Frontend SSL은 Listener에서 클라이언트 TLS를 종료하는 구성이다. 이 구성만으로 Load Balancer와 backend 사이의 트래픽이 암호화되지는 않는다. Backend SSL은 Backend Set에 인증서를 연결하여 backend 구간을 암호화한다. Oracle이 point-to-point SSL이라고 부르는 구성은 Listener와 Backend Set 양쪽에 인증서 bundle을 연결하여 두 구간을 암호화한다. 두 별도 구간의 TLS 처리를 하나의 끊김 없는 TLS session이라고 해석하면 안 된다.

인터넷 클라이언트의 HTTPS 접속이 목적이라면 frontend TLS가 필요한 위치이고, 내부 backend 구간에도 암호화 요구가 있다면 backend TLS까지 설계 대상이다. backend 서비스도 SSL 연결을 받아 종료할 수 있어야 한다. [Backend Sets](02_Backend_Sets.md), [Listeners](01_Listeners.md), [Oracle SSL Handling](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingcertificates.htm)

## 관리 방식과 인증서 종류의 선택

Load Balancer service 안의 인증서 관리는 이미 소유한 인증서를 import하는 방식이며 이 서비스가 인증서를 발급하지는 않는다. Oracle은 인증서 생성과 관리에 별도의 Certificates service 사용을 권장한다. 두 방식은 관리 주체가 다르므로 Load Balancer 내부의 import 항목을 Certificates service의 신규 인증서와 동일 리소스로 취급하면 안 된다.

단일 domain, 여러 이름을 포함하는 SAN, wildcard 인증서를 지원한다. 여러 hostname을 라우팅하는 Listener라면 접속 이름을 포함하는 인증서 범위를 선택해야 한다. OCI가 수용하는 인증서는 PEM 형식의 X.509이며 certificate chain과 대응하는 private key는 서로 다른 역할을 한다. chain은 신뢰 연결을 전달하고 private key는 해당 신원의 비밀 재료다. [Oracle Certificates 관리 설명](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingcertificates.htm)

## 신뢰와 운영의 한계

일반 TLS와 mTLS는 인증 대상이 다르다. mTLS에서는 상대 인증을 위한 CA 또는 CA bundle을 추가로 고려한다. CA bundle은 CA 공개 인증서 모음이며 private key나 leaf certificate를 포함하는 묶음이 아니다. 만료, 이름 불일치, 누락된 intermediate, 잘못된 peer verification depth는 TCP 접속 성공과 별개로 TLS handshake 실패를 일으킬 수 있다.

Frontend HTTPS의 성공만으로 backend 암호화나 HTTP 애플리케이션 정상 동작을 확인할 수는 없다. 인증서 교체와 연결된 Listener/Backend Set의 수명주기를 함께 관리해야 하며 개인키를 문서나 로그에 노출해서는 안 된다. [Oracle 인증서 검증 설명](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingcertificates.htm)

## 관련 실습

아래 보존 초안은 기존 PEM import와 frontend HTTPS 신뢰 검사를 다룬다. Certificates service 생성 실습이나 backend TLS 구성으로 범위를 확대하지 않으며 기존 초안의 실행 검증은 완료되지 않았다.

[전체 보존 실습](../../Hands-on/Original-Provisional/08_Load_Balancer/04_SSL_Certificates.md)

