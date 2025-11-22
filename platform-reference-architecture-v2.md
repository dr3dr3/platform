# Platform Reference Architecture

## Overview

This reference architecture is based on [PlatformEngineering.Org](https://platformengineering.org/blog/create-your-own-platform-engineering-reference-architectures) and defines our Internal Developer Platform (IDP).

**Related Documentation:**
- [Best Practices Guide](./best-practices.md) - Guidelines and recommendations for using the platform
- [Solution Comparisons](./solution-comparisons.md) - Vendor comparisons and alternatives

## Terminology

### Internal Developer Platform (IDP)
The complete ecosystem including all planes, tools, workflows, infrastructure, and golden paths.

### Internal Developer Portal
The user interface (Port.io) providing access to the platform through service catalog, self-service actions, documentation, and dashboards.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   Internal Developer Portal                 │
│                        (Port.io UI)                         │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Service   │  │ Self-Service │  │  Documentation   │  │
│  │   Catalog   │  │   Actions    │  │   & Runbooks     │  │
│  └─────────────┘  └──────────────┘  └──────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│            Internal Developer Platform                       │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Developer  │  │ Integration  │  │  Observability   │  │
│  │Control Plane │  │& Delivery    │  │      Plane       │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────────────────────────┐   │
│  │   Security   │  │      Resource Plane              │   │
│  │    Plane     │  │  (Compute, Data, Network, etc.)  │   │
│  └──────────────┘  └──────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Developer Control Plane

### Integrated Development Environment

#### Local Development
- **Devcontainers**: All repositories configured for containerized development
- **DevPod**: Runs devcontainers on local PCs
- **Docker Desktop**: Container engine
- **VS Code**: Primary IDE
- **GitHub Copilot**: AI pair programming (Pro license)
- **Claude Code**: AI assistant in terminal (Pro license)

#### Cloud Development Environments
- **GitHub Codespaces**: Primary CDE solution
- **Configuration**: devcontainer.json in all repositories
- **Features**: Pre-configured environments, automatic resource connection, prebuilds

### Service Catalog

#### Port.io Implementation
- **Software catalog**: Auto-discovery and tracking of all platform entities
- **Single source of truth**: Centralized platform metadata
- **Integrations**: GitHub, AWS, Kubernetes for automatic data sync

#### Entity Model

**Core Entities:**
1. **Service**: Microservices/applications with tier, owner, lifecycle, relationships
2. **API**: REST/GraphQL/gRPC endpoints with versions and specifications
3. **Environment**: dev/staging/production with configuration and quotas
4. **Database**: PostgreSQL via Neon with branches
5. **Repository**: GitHub repositories with metadata
6. **Deployment**: Deployment instances with status and metrics
7. **Pipeline Run**: CI/CD executions with results
8. **Infrastructure Component**: Terraform-managed resources
9. **Team**: Service owners with contact information

**Data Sources:**
- GitHub integration (repositories, workflows, releases)
- Kubernetes integration (deployments, services, pods)
- AWS integration (EKS, S3, ECR)
- ArgoCD integration (application status, deployments)
- Terraform integration (infrastructure state)
- Neon integration (databases, branches)

### Internal Developer Portal

#### Port.io Portal Features

**Core Capabilities:**
1. **Discover**: Search services, APIs, infrastructure, documentation
2. **Understand**: Visualize dependencies, ownership, architecture
3. **Self-Serve**: Execute tasks without platform team intervention
4. **Monitor**: Track deployments, incidents, service health

**Self-Service Actions:**
- Scaffold new service from template
- Deploy service to environment
- Create database branch (Neon)
- Scale service replicas
- Request production access
- Run database migration

**Documentation Hub:**
- Getting Started guides
- Golden Paths for common tasks
- Runbooks for incident response
- Architecture Decision Records (ADRs)
- API documentation (OpenAPI/GraphQL)
- Platform changelog

**Dashboards:**
- Service health per environment
- Platform overview metrics
- Team dashboards
- Cost tracking

**Access:**
- Authentication: GitHub SSO
- Authorization: Role-based (developers, owners, platform team, read-only)
- Audit trail: All actions logged with retention

### Version Control

**GitHub Organization** (Team plan):
- Application repositories: One per service/component
- Platform repositories:
  - Terraform (IaC for environments, organized by layers)
  - Terraform Modules (versioned via tags)
  - Dotfiles (personal IDE setup)
  - CI/CD pipelines (centralized, reusable)
  - Kubernetes manifests (GitOps, used by ArgoCD)

**GitOps Repository Structure:**
- **Foundation Layer**: Helm charts (Istio, Kyverno, ESO) - not managed by ArgoCD, uses Kustomize
- **Application Layer**: Application manifests - managed by ArgoCD, minimal Kustomize variations

## Integration and Delivery Plane

### Continuous Integration

**GitHub Actions**:
- Application pipeline: Linting, security scanning, testing, container image build
- Infrastructure pipeline: Terraform validation and security scanning
- Static analysis: GitHub security services
- Registry: AWS ECR for container images

**Score.dev Integration**:
- Developers specify infrastructure needs via Score files
- Documentation links for golden paths
- Validation and conversion to Terraform/K8s manifests

### Platform Orchestrator

**Current**: Custom GitHub Actions workflows
**Future**: Humanitec evaluation

### Continuous Delivery

#### Applications (ArgoCD)
- Kubernetes manifest repository as source
- Automatic sync on repository changes
- Custom Resource Definitions via Crossplane
- Container images from AWS ECR

#### Infrastructure (Terraform)
- Foundation provisioning (AWS EKS cluster)
- Management layer (environment administration)
- Layered approach for organization

## Observability Plane

### Architecture

**OpenTelemetry (OTel)**:
- Unified framework for metrics, logs, traces
- Vendor-neutral standard
- OTel Collector for data processing
- OTel Operator for Kubernetes management
- Auto-instrumentation for supported languages

**Collection Pattern:**
```
Applications (OTel SDKs) → OTel Collector (OTLP)
  → Processors (transform, sample, enrich)
    → Backends (Prometheus, Loki, Tempo)
```

### Metrics

**Stack:**
- **OTel SDKs**: Application instrumentation following semantic conventions
- **OTel Collector**: OTLP receiver, processing, export
- **Prometheus**: Metrics storage and query (kube-prometheus-stack)
- **Storage**: Local (15-30 days), Grafana Mimir or AWS AMP for long-term

**Standards:**
- RED method for services: Rate, Errors, Duration
- USE method for resources: Utilization, Saturation, Errors
- OTel Semantic Conventions for attributes

### Logging

**Stack:**
- **OTel Collector**: Primary log collector (DaemonSet + Deployment)
- **Grafana Loki**: Log aggregation and storage
- **Storage**: S3-compatible backend for log chunks
- **Optional**: AWS CloudWatch for AWS-native services

**Format:**
- Structured logs (JSON)
- Automatic trace context injection (trace_id, span_id)
- OTel Semantic Conventions for attributes

### Tracing

**Stack:**
- **OTel SDKs**: Automatic and manual instrumentation
- **OTel Collector**: Trace processing (batching, sampling, tail-based)
- **Grafana Tempo**: OTLP-native trace storage
- **Storage**: S3-compatible backend
- **Optional**: AWS X-Ray for AWS service tracing

**Standards:**
- W3C Trace Context propagation
- OTel Semantic Conventions for spans
- Sampling strategies per environment

**Span Metrics:**
- OTel Collector spanmetrics processor generates RED metrics
- Exported to Prometheus for alerting

### Visualization

**Grafana**:
- Unified UI for metrics, logs, traces
- Data sources: Prometheus, Loki, Tempo
- Dashboards: Golden signals, infrastructure, applications, business metrics
- Correlations: Navigate between logs, metrics, traces
- Exemplars: Link metrics to example traces

### Alerting

**AlertManager** (Prometheus stack):
- Prometheus rules for metrics-based alerts
- Grafana alerts for multi-source alerting
- Routing, grouping, deduplication
- Notifications: Slack, email, PagerDuty, webhooks

**Alert Design:**
- Symptom-based, not cause-based
- SLO-based alerting on error budget burn
- Runbooks linked in annotations
- Severity levels: Critical, Warning, Info

### Service Level Objectives

**Implementation:**
- SLIs from OTel span metrics (availability, latency, throughput)
- SLOs tracked via Prometheus recording rules
- Error budget calculation and visualization
- Alerts on error budget burn rate
- Tools: Sloth or Pyrra for SLO management

## Security Plane

### Secrets Management

**Primary: AWS Secrets Manager**
- Application secrets storage
- Automatic encryption with KMS
- Rotation support for database credentials
- CloudTrail audit logging

**Alternative: HashiCorp Vault** (self-hosted)
- For 50+ secrets or advanced features
- Dynamic secrets, PKI, encryption as a service
- Multi-cloud support

**Kubernetes Integration:**
- External Secrets Operator (ESO)
- Syncs from AWS Secrets Manager or Vault
- Creates Kubernetes Secrets

### Identity Management

#### AWS Identity

**AWS Organizations Structure:**
```
Management Account (SSO, Billing)
├── Security Account (Audit, Logging)
├── Shared Services Account (DevOps tools)
├── Development Account (EKS dev)
├── Staging Account (EKS staging)
└── Production Account (EKS prod)
```

**AWS IAM Identity Center (SSO):**
- Permission sets for role-based access
- Cross-account access via SSO
- MFA enforced
- No long-term credentials

#### Kubernetes RBAC

**Integration:**
- aws-auth ConfigMap: Maps IAM roles to K8s groups
- IRSA (IAM Roles for Service Accounts): Pods assume IAM roles

**Roles:**
- ClusterRoles: platform-admin, platform-viewer, developer-base
- Namespace Roles: namespace-admin, namespace-developer, namespace-viewer
- ServiceAccounts: Per-application with IRSA binding

### Security Scanning

#### Infrastructure-as-Code
- **TFSec**: Terraform security checks
- **Checkov**: Comprehensive IaC scanning
- **tflint**: Terraform linting

#### Application Code
- **SonarCloud**: Code quality and security (SAST)
- **Semgrep**: Fast static analysis with custom rules
- **Snyk**: Dependency vulnerability scanning
- **Dependabot**: Automated dependency updates

#### Containers
- **Trivy**: Comprehensive scanning (images, IaC, filesystems)
- **AWS ECR**: Basic image scanning
- **Sigstore/Cosign**: Image signing and verification

#### Runtime
- **Falco**: Kubernetes runtime threat detection

### Policy-as-Code

**Kubernetes (Kyverno):**
- YAML-based policies
- Validate, mutate, generate resources
- Pod security, resource management, networking, RBAC

**Infrastructure (Checkov + TFSec):**
- Terraform plan validation
- Custom policies for organizational standards
- CI/CD integration

### Audit and Compliance

**Logging:**
- AWS CloudTrail: API call logging
- Kubernetes Audit Logs: Sent to CloudWatch Logs

**Scanning:**
- kube-bench: CIS Kubernetes benchmarks
- Continuous vulnerability tracking

## Resource Plane

### Compute
- **AWS EKS**: Kubernetes cluster for all workloads
- **Pods**: Primary compute unit
- No compute resources outside EKS cluster

### Data
- **Neon**: PostgreSQL database platform
- **Branching**: Environment-specific database branches
- **Connection**: Managed via ESO and Kubernetes secrets

### Networking
- **Service Mesh**: AWS EKS networking
- **Simplicity**: Minimal complexity approach
- **Exploration**: Tailscale for hybrid on-premise/cloud connectivity

### Services
- **IaC-First**: All SaaS provisioned via Infrastructure-as-Code
- **Terraform**: For foundation/environment-level services
- **Crossplane**: For cluster-managed services
