# 🏗️ Kubernetes Platform Engineering Blueprint

![Kubernetes](https://img.shields.io/badge/Kubernetes-1.30+-blue?logo=kubernetes)
![GitOps](https://img.shields.io/badge/GitOps-ArgoCD%20%7C%20Flux-orange?logo=git)
![Platform Engineering](https://img.shields.io/badge/Platform%20Engineering-IDP-green)
![License](https://img.shields.io/badge/License-MIT-yellow)
![Status](https://img.shields.io/badge/Status-Production--Ready-brightgreen)

> **Эталонная архитектура (Blueprint)** для построения внутренней платформы разработчиков (Internal Developer Platform, IDP) на базе Kubernetes с использованием лучших практик Platform Engineering 2026 года.

##  Что это и зачем?

**Это не просто набор Ansible-ролей** — это полноценный Blueprint Platform Engineering, демонстрирующий как построить production-ready Kubernetes-платформу с нуля.

### Почему этот проект?

Большинство Kubernetes-примеров просто устанавливают софт. Этот репозиторий показывает **как построить Internal Developer Platform** следуя принципам Platform Engineering:

- ✅ **Developer Experience First** — разработчики работают с абстракциями, а не с Kubernetes-примитивами
- ✅ **GitOps-native** — всё декларативно, всё в Git
- ✅ **Security-by-design** — Supply Chain Security, SBOM, Policy as Code
- ✅ **Multi-cluster ready** — архитектура масштабируется на десятки кластеров
- ✅ **Extensible** — драйверная модель для GitOps-движков и Platform API

## Возможности

| Категория | Возможности |
|-----------|-------------|
| **GitOps** | ArgoCD/Flux, ApplicationSet, Multi-cluster sync |
| **Networking** | Cilium (eBPF), Gateway API, L2 announcements |
| **Observability** | OpenTelemetry, Prometheus, Loki, Tempo, Pyroscope |
| **Security** | Kyverno, OPA/Conftest, Syft/Grype, Trivy, Sigstore |
| **Platform API** | Crossplane Composition Functions, XRD |
| **Supply Chain** | SBOM generation, vulnerability scanning, policy gates |

## 🏛️ Архитектура

Платформа разделена на три независимых слоя:

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Experience                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  03-api: Platform API (Crossplane XRD/Composition)  │   │
│  │  • XWebApplication                                  │   │
│  │  • DatabaseClaim                                    │   │
│  │  • StorageClaim                                     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓
─────────────────────────────────────────────────────────────┐
│                   GitOps Control Plane                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  02-gitops: Declarative Platform State              │   │
│  │  • Components: Cilium, Observability, Kyverno       │   │
│  │  • Clusters: prod-eu, prod-us, staging              │   │
│  │  • Supply Chain Gate: SBOM, Trivy, Conftest         │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                   Bootstrap Layer                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  01-bootstrap: Ansible GitOps Engine Installation   │   │
│  │  • ArgoCD / Flux / Fleet (driver model)             │   │
│  │  • OCI Helm charts                                  │   │
│  │  • Idempotent & production-ready                    │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Подробная документация:**
- [Архитектура платформы](docs/architecture.md)
- [Bootstrap процесс](docs/bootstrap.md)
- [GitOps структура](docs/gitops.md)
- [Security & Compliance](docs/security.md)

## 📂 Структура репозитория

```
k8s-platform/
├── 01-bootstrap/          # Шаг 1: Ansible для установки GitOps-движка
├── 02-gitops/             # Шаг 2: Декларативное состояние платформы
├── 03-api/                # Шаг 3: Crossplane XRD/Composition для разработчиков
├── docs/                  # Документация и ADR
├── examples/              # Примеры использования Platform API
└── playbooks/             # Ansible playbooks
```

## 🚀 Quick Start

### 1. Подготовка

```bash
cd 01-bootstrap
pip install ansible kubernetes
ansible-galaxy collection install -r requirements.yml
```

### 2. Настройка

Отредактируйте `inventory/production/hosts.ini` и `inventory/production/group_vars/all.yml`:

```yaml
kubernetes:
  kubeconfig: ~/.kube/config

gitops:
  engine: argocd  # или 'flux'
  version: "7.5.0"
  namespace: gitops-system
  repo_url: "https://github.com/your-org/k8s-platform.git"
```

### 3. Запуск

```bash
ansible-playbook playbooks/bootstrap.yml
```

**Готово!** ArgoCD установлен и автоматически синхронизирует компоненты из `02-gitops/`.

## 🎬 Demo

```
Developer Workflow:
┌─────────────┐
│ Git Push    │ → Developer pushes app claim (XWebApplication)
└──────┬──────┘
       ↓
┌─────────────┐
│ ArgoCD Sync │ → Detects changes in Git
└──────┬──────┘
       ↓
┌─────────────┐
│ Crossplane  │ → Composition Functions generate resources
└──────┬──────┘
       ↓
┌─────────────┐
│ Kubernetes  │ → Creates Deployment, Service, HTTPRoute
└──────┬──────┘
       ↓
─────────────┐
│ Gateway API │ → Routes traffic to application
└──────┬──────┘
       ↓
┌─────────────┐
│ Accessible  │ → App available at myapp.dev.company.com
│ Application │
└─────────────┘
```

**Пример использования Platform API:**

```yaml
apiVersion: platform.company.io/v1alpha1
kind: XWebApplication
metadata:
  name: my-awesome-app
spec:
  image: ghcr.io/myorg/myapp:v1.2.3
  domain: myapp.dev.company.com
  replicas: 3
```

Разработчик получает работающее приложение **без знания Kubernetes-примитивов**.

## 🛠️ Технологический стек

| Слой | Технологии |
|------|------------|
| **Infrastructure** | Ansible, Helm, OCI Registry |
| **GitOps Engine** | ArgoCD (или Flux) |
| **Networking** | Cilium (CNI + eBPF LB + Gateway API + Hubble) |
| **Observability** | OpenTelemetry, Prometheus, Loki, Tempo, Pyroscope |
| **Policy** | Kyverno, OPA/Conftest |
| **Platform API** | Crossplane (Composition Functions) |
| **Supply Chain** | Syft, Grype, Trivy, Sigstore/Cosign |

## 📚 Документация

| Документ | Описание |
|----------|----------|
| [Архитектура](docs/architecture.md) | Полная схема платформы и компоненты |
| [Bootstrap](docs/bootstrap.md) | Детали Ansible-ролей и драйверная модель |
| [GitOps](docs/gitops.md) | Структура компонентов и multi-cluster |
| [Security](docs/security.md) | Supply Chain Security и Policy as Code |
| [ADR](docs/adr/) | Architectural Decision Records |

##  Architectural Decision Records (ADR)

Все ключевые архитектурные решения задокументированы:

- **ADR-0001**: Monorepo Structure — почему monorepo с нумерацией
- **ADR-0002**: GitOps Driver Model — поддержка ArgoCD/Flux/Fleet
- **ADR-0003**: Cilium Preference — eBPF over traditional CNI
- **ADR-0004**: Grafana Separation — observability isolation
- **ADR-0005**: Crossplane Functions over Go Controllers — declarative compositions

## ️ Roadmap

- [x] GitOps bootstrap (ArgoCD/Flux)
- [x] Platform Components (Cilium, Observability, Kyverno)
- [x] Multi-cluster support
- [x] Crossplane Platform API
- [x] Supply Chain Security (SBOM, Trivy, Conftest)
- [ ] Cluster API integration
- [ ] Backstage Developer Portal
- [ ] Secrets Management (External Secrets Operator)
- [ ] Cluster Autoscaler
- [ ] Cost Management (OpenCost)

##  Contributing

1. Форкните репозиторий
2. Создайте ветку фичи (`git checkout -b feature/amazing-feature`)
3. Убедитесь, что `ansible-playbook --syntax-check` и `conftest test` проходят
4. Добавьте тесты для новой функциональности
5. Создайте Pull Request с описанием изменений

## 👥 Для кого этот проект

- **Platform Engineers** — построение современной Kubernetes-платформы
- **DevOps Engineers** — изучение GitOps и Platform Engineering
- **Архитекторы** — reference architecture для enterprise-сред
- **Разработчики** — понимание внутренней платформы

##  Лицензия

MIT License — см. [LICENSE](LICENSE) файл.

---

**Примечание:** Этот проект является живым документом. Архитектура будет эволюционировать вместе с экосистемой Kubernetes и лучшими практиками Platform Engineering.
