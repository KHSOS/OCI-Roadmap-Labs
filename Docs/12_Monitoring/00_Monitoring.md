# Monitoring

Monitoring은 cloud resource와 애플리케이션의 상태·용량·성능을 metric으로 관찰하고, 정한 조건을 만족하면 alarm으로 알리는 서비스다. Resource가 내보내는 service metric과 애플리케이션이 발행하는 custom metric을 함께 사용할 수 있다.

## 역할과 구성

Metric producer가 timestamp/value와 dimension·metadata를 발행하면 Monitoring은 query의 interval과 statistic에 따라 집계값을 반환한다. Alarm은 이 집계 결과를 trigger rule로 평가하고 Notifications topic 또는 Streaming 같은 destination에 메시지를 보낸다. Metric 발행, 조회, alarm 평가, 최종 수신은 서로 다른 단계다.

## 사용 예시

Compute CPU utilization은 자원 부하를, 앱의 실패 transaction 수는 업무 기능의 이상을 보여줄 수 있다. 둘을 함께 보면 CPU가 낮아도 앱 오류가 증가하는 상황을 설명할 수 있다. Custom heartbeat의 수집 성공은 producer가 동작한다는 단서이지만 모든 사용자 요청이 성공한다는 뜻은 아니다.

## 선택 기준과 제약

Region·Compartment·namespace·dimension·시간 범위를 맞춰야 의미 있는 데이터를 비교할 수 있다. 빈 chart는 데이터가 없는 경우 외에도 필터·권한·수집 지연의 결과일 수 있다. 발행용 ingestion endpoint와 query endpoint의 역할을 구분한다. Monitoring metric 자체에는 OCID가 없으며 metric definition과 alarm history의 공식 보존 기간은 90일이다. 집계 선택과 alarm 설계는 각각 Metrics와 Alarms에서 다룬다.

## 관련 개념

- [Metrics](01_Metrics.md)
- [Alarms](02_Alarms.md)
- [Events](03_Events.md)
- [Logging](04_Logging.md)
- [Audit](05_Audit.md)
- [Notifications](06_Notifications.md)

## 더 읽기

- [Oracle — Overview of Monitoring](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Concepts/monitoringoverview.htm)

설정과 실습 절차는 [보존된 Monitoring Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/00_Monitoring.md)에서 읽을 수 있다.
