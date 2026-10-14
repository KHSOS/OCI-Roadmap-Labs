# Alarms

Alarm은 metric query의 결과가 threshold 또는 absence 조건을 만족하는지 주기적으로 평가하고 상태 변화에 따른 메시지를 destination에 보내는 Monitoring 설정이다. 단순 chart와 달리 언제 알림을 보낼지와 어디로 보낼지를 정의한다.

## 역할과 구성

Query는 metric·interval·statistic·trigger rule을 포함한다. Trigger delay는 조건이 지속돼야 하는 기간이며 aggregation interval과 다르다. Monitoring은 alarm을 매분 평가하고 OK/FIRING 상태와 transition history를 제공한다. Repeat notification은 FIRING이 지속되는 동안 반복 메시지를 보내는 별도 설정이다.

## 사용 예시

특정 앱 stream의 오류 수가 기준을 넘을 때 topic으로 메시지를 보낼 수 있다. Alarm history에서 FIRING이 확인돼도 email subscription이 미확인 상태라면 수신자가 메시지를 받는 것은 별개의 문제다. 반대로 producer가 멈춰 데이터가 없어졌다고 오류 수가 0이 되거나 앱이 회복했다고 볼 수 없다.

## 선택 기준과 제약

Threshold와 absence alarm은 서로 다른 질문에 답한다. 데이터 부재로 내부 reset 후 OK가 되더라도 정상 metric이 돌아온 것과 다르므로 RESET 원인을 해석해야 한다. Suppression은 메시지 발행을 일정 기간 멈추는 설정이며 metric 수집 자체를 중단하는 기능이 아니다. Alarm 업데이트는 모든 곳에 반영되기까지 최대 5분 걸릴 수 있다. 다수 stream의 알림은 destination 한도·중복·지연을 고려해야 한다.

## 관련 개념

- [Monitoring](00_Monitoring.md)
- [Metrics](01_Metrics.md)
- [Notifications](06_Notifications.md)

## 더 읽기

- [Oracle — Overview of Monitoring](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Concepts/monitoringoverview.htm)

설정과 실습 절차는 [보존된 Alarms Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/02_Alarms.md)에서 읽을 수 있다.
