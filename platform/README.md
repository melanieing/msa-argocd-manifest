# platform/

플랫폼 컴포넌트 Application들. `bootstrap/platform-of-apps.yaml`이 이 디렉토리를
recurse하여 모든 `*.yaml`을 Application 또는 ApplicationSet 정의로 적용한다.

## Sync Wave 순서

```
operators/      wave -20  ← Operator CRD가 다른 모든 Cluster CRD의 전제
data/           wave -10  ← Operator 설치 후 PostgreSQL/Kafka/Redis Cluster 적용
observability/  wave   0  ← Cluster 가동 후 모니터링
                wave   5  ← OpenTelemetry Collector (Prometheus/Loki 의존)
```

마이크로서비스(Sync Wave 10)는 `bootstrap/apps-appset.yaml`이 별도로 처리한다.

## 차트 출처 (예정)

| 컴포넌트 | Helm 차트 |
|---------|----------|
| CNPG (PostgreSQL Operator) | `cloudnative-pg/cloudnative-pg` |
| Strimzi (Kafka Operator) | `strimzi/strimzi-kafka-operator` |
| Redis Operator | `ot-helm/redis-operator` |
| kube-prometheus-stack | `prometheus-community/kube-prometheus-stack` |
| Loki | `grafana/loki-stack` (또는 `grafana/loki`) |
| OpenTelemetry Collector | `open-telemetry/opentelemetry-collector` |

각 Application의 `spec.source.targetRevision`(차트 버전)은 작업 시점에 채운다.
지금은 **골격만 배치**하고 실제 차트 등록은 Phase B에서 진행.
