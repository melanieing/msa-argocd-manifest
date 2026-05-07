# msa-argocd-manifest

Argo CD가 클러스터 상태의 **단일 소스 오브 트루스**(SSOT)로 동기화하는 매니페스트 리포.
Market Service MSA 프로젝트의 **클러스터 부트스트랩 + 5개 마이크로서비스 배포**를
GitOps로 일괄 관리한다.

> 📋 **기술 스택 + 버전**: [`msa-provisioning/STACK.md`](https://github.com/melanieing/msa-provisioning/blob/main/STACK.md)
> 📌 **백로그 / 진행 상황**: [`msa-provisioning/BACKLOG.md`](https://github.com/melanieing/msa-provisioning/blob/main/BACKLOG.md)

## 배경

본 리포는 `msa-provisioning/ansible/argocd-setup.yaml`의 root Application이 가리키는
대상이다. Argo CD root Application이 한 번 생성되면, 이 리포의 `bootstrap/` 디렉토리를
스캔하여 다음 두 가지를 자동 적용한다:

1. **Platform Layer** (App-of-Apps + Sync Wave) — CNPG, Strimzi, Redis Operator 등
   플랫폼 컴포넌트를 의존성 순서대로 설치
2. **App Layer** (ApplicationSet + Git Generator) — `msa-spring-boot` 레포의
   `charts/services/*` 디렉토리를 스캔하여 5개 마이크로서비스를 자동 등록

PDF 5.5.3절 "두 가지 용도로 동시 사용" 권장 패턴을 따른다.

## 디렉토리 구조

```
.
├── bootstrap/                # Argo CD root Application의 진입점
│   ├── platform-of-apps.yaml # platform/ 디렉토리를 watch하는 App
│   └── apps-appset.yaml      # 마이크로서비스용 ApplicationSet (Git Generator)
├── projects/                 # AppProject 정의 (권한 분리)
│   ├── platform-project.yaml
│   └── apps-project.yaml
├── platform/                 # 플랫폼 컴포넌트 Application들
│   ├── operators/            # Sync Wave -20: 오퍼레이터 먼저
│   ├── data/                 # Sync Wave -10: 데이터 클러스터 (CRD 의존)
│   └── observability/        # Sync Wave 0: 관측성 스택
└── applications/             # (앱 차트는 msa-spring-boot 레포에 거주)
```

## Sync Wave 정책

| Wave | 무엇 | 이유 |
|------|------|------|
| -20  | CNPG / Strimzi / Redis Operator | CRD 등록이 다른 모든 것의 전제 |
| -10  | PostgreSQL Cluster / Kafka Cluster / Redis Cluster | Operator 설치 후 Cluster CRD 적용 가능 |
| 0    | Prometheus, Loki, Grafana | 관측성은 Cluster 가동 후 |
| 5    | OpenTelemetry Collector | Prometheus/Loki 엔드포인트 의존 |
| 10   | 마이크로서비스 5개 (ApplicationSet 자동) | DB/Kafka/Redis 클러스터 가동 후 |

## Sync Policy (모든 Application 공통)

```yaml
syncPolicy:
  automated:
    prune: true       # Git에서 사라진 리소스는 클러스터에서도 제거
    selfHeal: true    # 누군가 수동 변경해도 Git 상태로 자동 복원
  syncOptions:
    - CreateNamespace=true
    - ServerSideApply=true
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

PDF 5.5.3절 표준을 따른다.

## 사용

이 리포는 사람이 `kubectl apply`하지 않는다. 모든 변경은 PR → merge → Argo CD가 자동 sync.
클러스터에 직접 손대는 명령은 `msa-provisioning/ansible/argocd-setup.yaml`의 root
Application 생성 **딱 한 번**.

## 관련 레포

- `msa-provisioning` — Terraform + Ansible (인프라 + K8s 부트스트랩)
- `msa-spring-boot` — 마이크로서비스 소스 + Helm 차트 (`charts/services/*`)
