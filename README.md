# Internal Developer Platform

This repository contains the platform engineering documentation and reference architecture for our Internal Developer Platform (IDP).

## Documentation Structure

### 📐 [Platform Reference Architecture](./platform-reference-architecture.md)
**What:** Comprehensive technical architecture of the entire platform.

Describes the five planes of the platform:
- **Developer Control Plane**: IDEs, service catalog, portal, version control
- **Integration & Delivery Plane**: CI/CD pipelines, GitOps, orchestration
- **Observability Plane**: OpenTelemetry, metrics, logs, traces, dashboards
- **Security Plane**: Secrets, identity, scanning, policies
- **Resource Plane**: Compute, data, networking, services

**Use this when:** You need to understand the complete platform architecture and component relationships.

### ✨ [Best Practices Guide](./best-practices.md)
**What:** Guidelines, standards, and recommendations for using the platform.

Covers best practices for:
- Development environments (local and cloud)
- Service catalog metadata quality
- CI/CD workflows
- Observability implementation (OTel, dashboards, alerting)
- Security practices (secrets, RBAC, policies)
- Golden paths for common tasks
- Cost optimization

**Use this when:** You're implementing a feature or want to know the recommended way to do something.

### 🔍 [Solution Comparisons](./solution-comparisons.md)
**What:** Vendor comparisons and alternative solutions for each platform component.

Compares options for:
- Development environments (Codespaces, Gitpod, DevPod, Coder)
- Secrets management (AWS Secrets Manager, Vault, Parameter Store)
- Observability backends (Prometheus/Loki/Tempo, managed alternatives)
- Policy engines (Kyverno, OPA, Terraform scanning)
- Security scanning tools (SAST, SCA, container scanning)

Includes decision criteria and migration strategies.

**Use this when:** You're evaluating alternatives or need to justify technology choices.

### 📋 Architecture Decision Records (ADRs)
**What:** Documented architectural decisions with context, rationale, and consequences.

Current ADRs:
- [ADR-001: Terraform Deployment Workflow](./adr-001-terraform-deployment-workflow.md) - GitHub Actions vs Terraform Cloud VCS Integration

**Use this when:** You need to understand why specific architectural decisions were made or need a template for new decisions.

## Quick Links

- **New Service**: Follow the ["Create a New Service" Golden Path](./best-practices.md#create-a-new-service-golden-path)
- **Production Deployment**: Follow the ["Deploy to Production" Golden Path](./best-practices.md#deploy-to-production-golden-path)
- **Developer Onboarding**: Follow the ["Onboard a New Team Member" Golden Path](./best-practices.md#onboard-a-new-team-member-golden-path)

## Repository Structure

```
.
├── README.md                              # This file
├── platform-reference-architecture.md     # Complete platform architecture
├── platform-reference-architecture-v2.md  # Updated platform architecture
├── best-practices.md                      # Platform usage guidelines
├── solution-comparisons.md                # Technology comparison and alternatives
├── adr-001-terraform-deployment-workflow.md  # ADR: Terraform deployment approach
└── [Future: Kubernetes manifests for GitOps]
```

## Platform Overview

Our Internal Developer Platform provides:

- 🚀 **Self-Service**: Developers can create services, deploy, and manage infrastructure without platform team intervention
- 📊 **Observability**: OpenTelemetry-based unified observability across all services
- 🔒 **Security**: Built-in secrets management, RBAC, policy enforcement, and scanning
- 📚 **Documentation**: Centralized portal with service catalog, runbooks, and golden paths
- 🎯 **Golden Paths**: Opinionated, pre-paved routes for common tasks
- 💰 **Cost-Effective**: Designed with cost optimization using free tiers and open source

## Technology Stack

**Portal & Catalog:** Port.io  
**Version Control:** GitHub  
**Cloud Provider:** AWS (EKS, Secrets Manager, ECR)  
**Compute:** Kubernetes (AWS EKS)  
**Database:** Neon PostgreSQL  
**CI/CD:** GitHub Actions, ArgoCD  
**IaC:** Terraform, Crossplane  
**Observability:** OpenTelemetry, Prometheus, Loki, Tempo, Grafana  
**Security:** Kyverno, Trivy, SonarCloud, Snyk, Falco  

## Contributing

This documentation is living and evolves with the platform. See individual documents for specific contribution guidelines.

## Support

- **Portal Access**: [Port.io Portal](https://app.getport.io/[org-name])
- **Platform Team**: Contact via Slack #platform-engineering
