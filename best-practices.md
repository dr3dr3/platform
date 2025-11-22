# Platform Engineering Best Practices

This document captures best practices, guidelines, and recommendations for using the platform described in the [Platform Reference Architecture](./platform-reference-architecture.md).

## Table of Contents
- [Development Environment Best Practices](#development-environment-best-practices)
- [Service Catalog Best Practices](#service-catalog-best-practices)
- [CI/CD Best Practices](#cicd-best-practices)
- [Observability Best Practices](#observability-best-practices)
- [Security Best Practices](#security-best-practices)

## Development Environment Best Practices

### Cloud Development Environments (CDE)

#### Environment Lifecycle Management
* Use ephemeral environments - spin up for specific tasks, tear down when complete
* Implement automatic timeout policies (e.g., 4-hour inactivity timeout)
* Set organizational policies for maximum environment retention (e.g., 7 days)
* Enable prebuilds to reduce environment startup time from minutes to seconds

#### Configuration Management
* Maintain `devcontainer.json` in every repository as single source of truth
* Use base images with common tooling, extend per-repository as needed
* Version control all IDE configurations, extensions, and settings
* Standardize across local (DevPod) and cloud (Codespaces) environments

#### Security & Access
* Environments inherit repository permissions - no additional access grants needed
* Use secrets management integration (GitHub Secrets, External Secrets Operator)
* Enable audit logging for environment creation and access
* Implement network policies to control outbound access from CDEs

#### Resource Optimization
* Start with 4-core/8GB machines as default, scale up for specific workloads
* Use prebuilds for frequently accessed repositories
* Implement cost tracking and alerts per team/project
* Set organizational spending limits to prevent overruns

#### Developer Experience
* Enable VS Code Settings Sync for personalized configurations
* Configure dotfiles repository for personal shell/terminal preferences
* Provide clear documentation on when to use CDE vs local development
* Set up one-click environment creation from Issues/PRs

## Service Catalog Best Practices

### Metadata Quality

#### Metadata Sources Priority
In case of conflicts, use this priority order:
1. **Explicit declarations** in repository (`port.yml`, `catalog-info.yaml`) - highest priority
2. **Runtime state** from Kubernetes/AWS - current actual state
3. **Infrastructure-as-Code** from Terraform - intended state
4. **Automated discovery** - fallback for undeclared resources

#### Metadata Quality Scoring
Track these metrics for each service (Port.io Scorecards):
* Repository has README: +10 points
* Service has owner team: +20 points
* API has OpenAPI spec: +15 points
* Has health check endpoint: +10 points
* Has SLO defined: +15 points
* Has runbook links: +10 points
* Passing security scans: +20 points
* **Goals**: Bronze (50), Silver (70), Gold (90+)

### Search and Discovery

#### Common Search Patterns
* "Show all critical tier services in production"
* "Which services consume the payments API?"
* "What databases does the checkout service use?"
* "Show all services owned by team-platform"
* "Find all deprecated services"

## CI/CD Best Practices

### Infrastructure Pipelines

#### Score.dev Usage
* Use Score.dev files to declare infrastructure needs
* Link to documentation explaining golden paths
* Validate Score files before conversion
* Support only approved golden paths initially

#### Terraform Workflow
```
Developer → git push → GitHub Actions
  ├── terraform fmt -check (formatting)
  ├── tfsec (security scanning)
  ├── checkov (compliance checks)
  ├── terraform validate
  ├── terraform plan → Manual approval → terraform apply
  └── Update Port.io entities
```

## Observability Best Practices

### OpenTelemetry Implementation

#### Instrumentation Standards
* Follow **OTel Semantic Conventions** for all attributes
* Standard resource attributes: `service.name`, `service.version`, `deployment.environment`
* Standard HTTP attributes: `http.method`, `http.status_code`, `http.route`
* Standard database attributes: `db.system`, `db.name`, `db.operation`

#### Metrics Best Practices
* Follow **RED method** for services: Rate, Errors, Duration
* Follow **USE method** for resources: Utilization, Saturation, Errors
* Use consistent naming conventions across all services
* Include business metrics alongside technical metrics

#### Logging Best Practices
* Emit **structured logs** (JSON format preferred)
* Include trace context automatically (`trace_id`, `span_id`)
* Consistent log levels: TRACE, DEBUG, INFO, WARN, ERROR, FATAL
* Key fields: `timestamp`, `severity`, `body`, `attributes`, `resource`

#### Tracing Best Practices
* Use W3C Trace Context for propagation
* Add business context as span attributes (user_id, tenant_id)
* Follow semantic conventions for span naming
* Implement appropriate sampling strategies per environment

#### Sampling Strategy
* **Development**: 100% sampling (full visibility)
* **Staging**: 50% sampling with always-sample for errors
* **Production**: 10-20% sampling, always-sample errors and slow requests
* Use tail-based sampling for intelligent decision-making

### Dashboard Strategy

#### Golden Signals Dashboards
All services should track:
* **Latency**: Request duration distribution
* **Traffic**: Request rate over time
* **Errors**: Error rate and types
* **Saturation**: Resource utilization

#### Service Dashboard Requirements
* Current deployment version per environment
* Health status (K8s probes + OTel metrics)
* Recent deployments (last 10)
* Open incidents
* SLO compliance
* Dependencies visualization
* Owner and on-call information

### Alerting Best Practices

#### Alert Design Principles
* **Symptom-based alerts**, not cause-based
  * Alert on: "API latency > 500ms" (symptom)
  * Not on: "CPU > 80%" (possible cause)
* **SLO-based alerting**: Alert when error budget is burning
* Link runbooks in alert annotations
* Include troubleshooting steps
* Provide example queries for investigation

#### Alert Severity Levels
* **Critical**: Page on-call, immediate action required
* **Warning**: Investigate during business hours
* **Info**: Logged for awareness, no action needed

#### Avoiding Alert Fatigue
* Tune thresholds based on historical data
* Implement proper alert grouping and deduplication
* Regular alert review and pruning
* Monitor alert signal-to-noise ratio

#### Alert Enrichment
* Include `trace_id` in annotations when available
* Include `service.name`, `deployment.environment` attributes
* Link to pre-filtered Grafana dashboards
* Include PromQL/LogQL queries for investigation

### Service Level Objectives (SLOs)

#### SLO Definition
* Define **SLIs** (Service Level Indicators) using OTel data:
  * Availability: Success rate from span metrics
  * Latency: P95/P99 response time from span metrics
  * Throughput: Request rate from span metrics
* Track SLOs using Prometheus recording rules
* Calculate error budgets and visualize in Grafana
* Alert when error budget burn rate is too high

## Security Best Practices

### Secrets Management

#### Secret Handling
* **Never commit secrets to Git** (use git-secrets or gitleaks hooks)
* **Least privilege**: Scope secrets to specific services
* **Encryption at rest**: Use AWS KMS for encryption
* **Audit logging**: Enable CloudTrail for all secret access
* **Secret scanning**: Enable in CI/CD pipelines

#### Secret Rotation
* Enable automatic rotation for database credentials
* Applications must handle secret refresh gracefully
* Test rotation procedures regularly
* Document rotation schedules

### Identity and Access Management

#### AWS IAM Best Practices
* Use IAM Identity Center (AWS SSO) for human access
* Enable MFA for all users
* Use IAM roles, not IAM users for applications
* Implement least privilege access
* Regular access reviews and cleanup

#### Kubernetes RBAC Best Practices
* **Principle of least privilege**: Grant minimum required permissions
* **Use Groups, not individual users**: Bind roles to groups
* **Namespace isolation**: Separate namespaces per environment and service
* **Regular audits**: Review and remove unused permissions
* **Pod Security Standards**: Enforce baseline or restricted profiles

### Policy-as-Code Best Practices

#### Policy Development
* **Start with audit mode**: Log violations before enforcing
* **Gradual rollout**: Enable policies per namespace
* **Document exceptions**: When and why policies are bypassed
* **Regular review**: Update policies as threats evolve
* **GitOps**: Store policies in Git, version controlled

#### Policy Categories to Implement
* **Pod Security**: Restrict privileged containers, host access
* **Resource Management**: Require limits/requests
* **Networking**: Enforce network policies
* **RBAC**: Validate role bindings
* **Supply Chain**: Verify image signatures, allowed registries
* **Compliance**: Enforce organizational standards

### Vulnerability Management

#### Scanning Practices
* Scan infrastructure-as-code before deployment
* Scan container images in CI/CD pipeline
* Scan dependencies for known vulnerabilities
* Regular runtime security monitoring

#### Vulnerability Response
* Prioritize by CVSS score and exploitability
* Track vulnerabilities in dashboards
* Automated patching where possible
* Document remediation timelines

## Golden Paths

### "Create a New Service" Golden Path

1. Click "Create New Service" in Port.io Portal
2. Fill in service details (form guides through required metadata)
3. Portal scaffolds repository with:
   * devcontainer.json for local/cloud development
   * CI/CD workflows (GitHub Actions)
   * Kubernetes manifests templates
   * Basic tests and linting configuration
   * Observability instrumentation (OTel)
4. Portal creates database if needed (Neon)
5. Portal sets up monitoring dashboards (Grafana)
6. Portal creates initial Port.io entity
7. **Result**: Fully functional service in <5 minutes, ready to develop

### "Deploy to Production" Golden Path

1. Service must pass scorecard checks (Bronze minimum):
   * Has README with proper documentation
   * Has owner team assigned
   * Has passing tests
   * Has OpenAPI spec (for APIs)
   * Has security scan (no high/critical vulnerabilities)
2. Create PR to update production image tag in GitOps repo
3. Automated checks run:
   * Terraform plan (if infra changes)
   * Security scans
   * Approval required from service owner + platform team
4. Merge PR
5. ArgoCD auto-syncs (or use "Deploy" action in Portal)
6. Monitor deployment in Portal (shows ArgoCD status, pod health)
7. Smoke tests run automatically
8. Rollback button available in Portal if issues
9. **Result**: Safe, auditable production deployment

### "Onboard a New Team Member" Golden Path

1. Add to GitHub organization
2. Add to AWS IAM Identity Center group
3. Portal shows personalized onboarding checklist:
   * Install DevPod locally or use GitHub Codespaces
   * Clone first repository
   * Complete "Getting Started" guide
   * Deploy to dev environment
   * First deployment (to dev environment)
4. Portal guides through first self-service action
5. **Result**: Developer productive on day one

## Cost Optimization

### Development Environments
* Monitor CDE usage and set spending limits
* Use prebuilds to reduce active environment time
* Implement auto-shutdown policies
* Track costs per team/project

### Observability
* Use sampling to reduce data volume
* Implement data retention policies
* Monitor collector resource usage
* Use free tiers effectively before scaling to paid

### Security
* Consolidate secrets to minimize per-secret costs
* Use free tier security scanning tools
* Self-host where operational overhead is manageable
* Regular cost reviews and optimization
