# platform/

플랫폼 컴포넌트 Application들. **3개의 부모 Application** 이 각자 자기 서브폴더만
watch (recurse:false). 옛 단일 `platform-of-apps.yaml` 은 recurse:true 로 _postgres
등 raw CR 폴더까지 들어가서 CRD 등록 전에 Cluster CR 적용 시도 → 첫 부트스트랩
(2026-05-09) sync 실패. 3개로 분리 + recurse:false 로 fix.

| 부모 | 위치 | watch 대상 | wave |
|------|------|-----------|------|
| `bootstrap/platform-operators-app.yaml` | `platform/operators/` | 3 Application yamls | -50 |
| `bootstrap/platform-data-app.yaml` | `platform/data/` | 3 Application yamls (서브 _* 폴더 제외) | -40 |
| `bootstrap/platform-observability-app.yaml` | `platform/observability/` | 3 Application yamls | -30 |

## Sync Wave 순서 (전체)

```
projects-app                 wave -100  ← AppProject 가 가장 먼저
platform-operators-app       wave  -50  ← 부모 1: operators 부모
  cnpg/strimzi/redis-operator wave  -20 ← 자식: 실제 operator helm install
platform-data-app            wave  -40  ← 부모 2: data 부모
  postgres/kafka/redis-cluster wave  -10 ← 자식: Cluster CR 적용 (CRD 이미 등록됨)
platform-observability-app   wave  -30  ← 부모 3: 관측성 부모
  prometheus/loki/otel        wave    0 ← 자식
microservices ApplicationSet wave    0  ← 마이크로서비스 4 Application 자동 생성
  user-api-gateway 등         wave   10 ← 마지막 (DB/Kafka/Redis 다 뜬 후)
```

ArgoCD 가 부모 Application 의 wave 단위로 sync + Healthy 대기 → cross-Application
dependency 정확히 보장.

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
