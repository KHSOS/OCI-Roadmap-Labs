# Notifications

Notifications는 Topic과 Subscription으로 메시지 발행자와 수신 endpoint를 연결하는 OCI 서비스다. Alarm과 Event Rule이 보낸 메시지뿐 아니라 사용자·앱의 직접 발행도 처리한다. Notifications 자체가 어떤 metric을 평가하거나 어떤 resource 변화를 선택하는 것은 아니다.

## 역할과 구성

Topic은 tenancy에서 고유한 이름을 갖는 통신 채널이고 Subscription은 email, HTTPS, Functions, Slack, PagerDuty, SMS 같은 수신 endpoint를 정의한다. 발행된 메시지는 topic의 active subscription으로 전달된다. Pending subscription은 confirmation을 마치지 않은 상태이며 active와 같지 않다.

## 사용 예시

하나의 운영 topic에 email과 HTTPS endpoint를 두면 같은 alarm 메시지를 사람이 읽고 자동 처리할 수 있다. Publish 응답의 message ID는 접수 식별자이며 mailbox의 실제 수신을 뜻하지 않는다. Endpoint가 응답하지 않으면 retry하며 기본 retry duration은 2시간이다.

## 선택 기준과 제약

전달은 subscription당 at-least-once여서 중복 처리 가능성을 고려한다. 공식 한도는 메시지당 64KB, topic당 분당 발행 60건이며 email 전달은 분당 10건이다. SMS는 모든 producer에 지원되지 않고 Announcements·Monitoring·Connector Hub 메시지에 한정된다. 대형 첨부나 raw email을 보내는 Email Delivery와 용도가 다르며 email 처리·전달은 미국 Region의 Oracle resource를 이용한다.

## 관련 개념

- [Alarms](02_Alarms.md)
- [Events](03_Events.md)
- [Logging](04_Logging.md)

## 더 읽기

- [Oracle — Overview of Notifications](https://docs.oracle.com/en-us/iaas/Content/Notification/Concepts/notificationoverview.htm)

설정과 실습 절차는 [보존된 Notifications Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/06_Notifications.md)에서 읽을 수 있다.
