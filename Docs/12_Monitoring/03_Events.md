# Events

Events는 OCI resource의 상태 변화 메시지에 반응하여 자동화를 연결하는 서비스다. 시간마다 metric을 평가하는 Alarm과 달리 지원 서비스가 발생시킨 event를 filter로 선택하여 action을 수행한다.

## 역할과 구성

Producer가 CloudEvents 형식의 event를 내보내면 Event Rule은 event type과 attribute/tag 조건을 평가한다. 일치한 event는 rule에 지정한 Notifications, Streaming, Functions destination으로 전달된다. Rule, event 메시지, destination resource는 서로 다른 객체다.

## 사용 예시

Database backup 완료 event를 운영 topic에 전달하거나 Object Storage의 object 업로드 event로 변환 함수를 호출할 수 있다. Bucket 생성 event와 object 생성 event는 다른 event type이며, object 상태 변화는 bucket의 object event emission을 활성화해야 한다. Filter의 예시 JSON이 일치하는 것은 실제 자원 event가 발생해 전달되었다는 뜻이 아니다.

## 선택 기준과 제약

서비스마다 제공 event type과 payload attribute가 다르므로 지원 catalog의 범위를 기준으로 설계한다. 뒤늦게 filter를 수정해도 이미 발생한 변화가 자동 replay된다고 가정하지 않는다. Event 수신과 후속 function 성공·email 배달은 별도로 관찰한다. 공식 overview의 tenancy당 rule 한도는 50개이며 한도 증가는 서비스 정책을 확인한다. API caller 추적이 목적이면 Events가 아니라 Audit와 함께 해석한다.

## 관련 개념

- [Monitoring](00_Monitoring.md)
- [Alarms](02_Alarms.md)
- [Audit](05_Audit.md)
- [Notifications](06_Notifications.md)

## 더 읽기

- [Oracle — Overview of Events](https://docs.oracle.com/en-us/iaas/Content/Events/Concepts/eventsoverview.htm)

설정과 실습 절차는 [보존된 Events Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/03_Events.md)에서 읽을 수 있다.
