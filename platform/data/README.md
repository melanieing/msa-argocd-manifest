# platform/data/

데이터 클러스터(PostgreSQL ×5 / Kafka KRaft / Redis Cluster) Application들이 들어갈 자리.
**Phase B에서 추가**한다.

## 추가 시 형태

```
platform/data/
├── postgres-clusters.yaml   # Application(wave=-10) → _postgres/ 디렉토리 watch
├── kafka-cluster.yaml       # Application(wave=-10) → _kafka/ 디렉토리 watch
├── redis-cluster.yaml       # Application(wave=-10) → _redis/ 디렉토리 watch
├── _postgres/
│   ├── user-db.yaml         # CNPG Cluster CRD
│   ├── product-db.yaml
│   ├── inventory-db.yaml
│   ├── order-db.yaml
│   └── notification-db.yaml
├── _kafka/
│   ├── kafka-cluster.yaml   # Strimzi Kafka KRaft 3-broker StatefulSet
│   └── topics/              # KafkaTopic CRDs (PDF 부록 A의 5개 토픽)
└── _redis/
    └── redis-cluster.yaml   # RedisCluster CRD (3 master + 3 replica)
```

## Sync Wave

모두 `-10` (operators의 `-20` 다음, observability의 `0` 이전).
operators/ 가 CRD를 등록한 뒤에야 Cluster CRD를 적용할 수 있기 때문.

## 의존성

- `cnpg-operator` 가동 후 `Cluster` (postgresql.cnpg.io) CRD 사용 가능
- `strimzi-operator` 가동 후 `Kafka`, `KafkaTopic` (kafka.strimzi.io) CRD 사용 가능
- `redis-operator` 가동 후 `RedisCluster` (redis.redis.opstreelabs.in) CRD 사용 가능
