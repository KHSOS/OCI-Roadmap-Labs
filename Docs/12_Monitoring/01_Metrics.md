# Metrics

Metric은 resource 또는 애플리케이션의 건강·용량·성능을 숫자와 시간으로 표현하는 측정값이다. Namespace는 발행 주체의 구분, dimension은 resourceId나 availabilityDomain 같은 필터·grouping 조건, metadata는 unit 같은 해석 정보를 제공한다.

## 역할과 구성

Raw datapoint는 timestamp/value이고 집계 datapoint는 일정 interval의 raw data에 statistic을 적용한 결과다. Max는 가장 큰 값, Sum은 합, Mean은 Sum/Count를 나타낸다. Interval은 집계 window의 길이이고 resolution은 연속 window가 이동하는 간격이다. Producer의 발행 frequency와 query의 interval/resolution은 별개다.

## 사용 예시

같은 stream의 두 관측값이 한 집계 window에 들어오면 Max는 더 큰 값을, Sum은 두 값을 합친 결과를 표현한다. 관측 시각이 window 경계 양쪽에 있으면 서로 다른 집계값으로 나뉠 수 있다. 같은 metric 이름이어도 resource dimension이 다르면 서로 다른 stream이므로 비교나 합산 범위를 의식해야 한다.

## 선택 기준과 제약

Utilization에는 평균·최대·percentile 등 목적에 맞는 통계가 필요하며 숫자가 크다는 이유로 Sum이 항상 적절하지 않다. 동일 시간 범위·dimension·unit 없이 두 chart를 비교하면 잘못 해석할 수 있다. Aggregated timestamp는 평가 window의 끝을 나타낸다. Alarm query의 resolution은 1분만 유효하며 metric query의 지원 시간 범위는 resolution과 연관된다. Metric은 개별 OCID resource가 아니므로 metric 이름을 서버 식별자로 취급하지 않는다.

## 관련 개념

- [Monitoring](00_Monitoring.md)
- [Alarms](02_Alarms.md)

## 더 읽기

- [Oracle — Overview of Monitoring](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Concepts/monitoringoverview.htm)
- [Oracle — Selecting the Statistic for a Query](https://docs.oracle.com/en-us/iaas/Content/Monitoring/Tasks/query-metric-statistic.htm)

설정과 실습 절차는 [보존된 Metrics Hands-on 문서](../../Hands-on/Original-Provisional/12_Monitoring/01_Metrics.md)에서 읽을 수 있다.
