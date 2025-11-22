# Platform Reference Architecture

## Sources

This reference architecture is based on that from [PlatformEngineering.Org](https://platformengineering.org/blog/create-your-own-platform-engineering-reference-architectures).

## Terminology: Platform vs Portal

It's important to distinguish between two commonly confused terms in platform engineering:

### Internal Developer Platform (IDP)
The **Internal Developer Platform** refers to the **entire ecosystem** described in this document:
* **All the planes**: Developer Control Plane, Integration & Delivery Plane, Observability Plane, Security Plane, Resource Plane
* **All the tools and services**: GitHub, AWS, Kubernetes, Terraform, ArgoCD, monitoring tools, etc.
* **All the workflows and processes**: CI/CD pipelines, GitOps, deployment strategies, security policies
* **All the infrastructure**: Compute, networking, storage, services
* **The golden paths and standards**: How things should be done, paved paths for common tasks

**Think of it as**: The platform is the *product* that the Platform Engineering team builds and maintains for developers.

### Internal Developer Portal (also IDP - confusing!)
The **Internal Developer Portal** refers to the **user interface** that provides access to the platform:
* **The UI**: A web interface (in our case, Port.io) where developers interact with the platform
* **Service catalog**: Visibility into services, APIs, dependencies, and ownership
* **Self-service actions**: Buttons and workflows to perform common tasks (scaffold new service, create environment, etc.)
* **Documentation hub**: Centralized place for guides, runbooks, and golden paths
* **Metrics and dashboards**: Aggregated view of service health, deployments, incidents
* **Search and discovery**: Find services, APIs, documentation, and resources

**Think of it as**: The portal is the *interface* or *front door* to the platform - it's how developers discover, understand, and interact with the underlying platform capabilities.

### The Relationship

```
┌─────────────────────────────────────────────────────────────┐
│                   Internal Developer Portal                 │
│                        (Port.io UI)                         │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   Service   │  │ Self-Service │  │  Documentation   │  │
│  │   Catalog   │  │   Actions    │  │   & Runbooks     │  │
│  └─────────────┘  └──────────────┘  └──────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │ (Interface Layer)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│            Internal Developer Platform (The System)          │
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

**In this document**: 
* When we say "**platform**" → we mean the entire system
* When we say "**portal**" → we mean the Port.io UI specifically
* Both are abbreviated "IDP" in industry, so context matters!

## Developer Control Plane

### Integrated Developer Environment (IDE)

#### Local Development Environment

* All repositories are setup as devcontainers for local development
* We use DevPod on local development PC's to run these devcontainers
* We use Docker Desktop as the container engine for these devcontainers
* VS Code is the main editor used (loaded into the devcontainer)
* GitHub CoPilot chat is enabled in IDE and Pro licensing provided
* Claude Code is enabled in the terminal and Pro licensing provided

#### Cloud Development Environments (CDE)

**Overview**
* Ephemeral, on-demand development environments running in the cloud
* Developers can spin up standardized environments in seconds without local setup
* Ensures consistency across team ("works on my machine" becomes irrelevant)
* Provides access to production-like resources and data without local constraints

**Current Implementation**
* GitHub Codespaces as primary CDE solution
* All repositories configured with devcontainer.json for seamless CDE provisioning
* Pre-configured environments with all necessary tools, extensions, and dependencies
* Automatically connects to cloud resources and services

**Best Practices**

*Environment Lifecycle Management*
* Use ephemeral environments - spin up for specific tasks, tear down when complete
* Implement automatic timeout policies (e.g., 4-hour inactivity timeout)
* Set organizational policies for maximum environment retention (e.g., 7 days)
* Enable prebuilds to reduce environment startup time from minutes to seconds

*Configuration Management*
* Maintain devcontainer.json in every repository as single source of truth
* Use base images with common tooling, extend per-repository as needed
* Version control all IDE configurations, extensions, and settings
* Standardize across local (DevPod) and cloud (Codespaces) environments

*Security & Access*
* Environments inherit repository permissions - no additional access grants needed
* Use secrets management integration (GitHub Secrets, External Secrets Operator)
* Enable audit logging for environment creation and access
* Implement network policies to control outbound access from CDEs

*Resource Optimization*
* Start with 4-core/8GB machines as default, scale up for specific workloads
* Use prebuilds for frequently accessed repositories
* Implement cost tracking and alerts per team/project
* Set organizational spending limits to prevent overruns

*Developer Experience*
* Enable VS Code Settings Sync for personalized configurations
* Configure dotfiles repository for personal shell/terminal preferences
* Provide clear documentation on when to use CDE vs local development
* Set up one-click environment creation from Issues/PRs

**Vendor Solutions Comparison**

*GitHub Codespaces (Current Choice)*
* **Pros**: 
  - Native GitHub integration, seamless workflow from code to environment
  - Strong VS Code/devcontainer support
  - Generous free tier (60 hours/month for Pro, 120 hours/month for Teams)
  - Prebuilds reduce cold start time significantly
  - Built-in secrets management
* **Cons**: 
  - Limited to GitHub ecosystem
  - Can become expensive at scale without proper policies
  - Fewer machine type options compared to dedicated solutions
* **Best for**: Teams already on GitHub with devcontainer standardization
* **Pricing**: ~$0.18/hour for 4-core machines, ~$0.36/hour for 8-core

*Gitpod (Alternative)*
* **Pros**:
  - Multi-platform support (GitHub, GitLab, Bitbucket)
  - Superior prebuild system with incremental updates
  - Better resource optimization and cost controls
  - Dedicated support for JetBrains IDEs
  - Self-hosted option available
* **Cons**:
  - Another tool to manage if heavily invested in GitHub
  - Smaller ecosystem compared to GitHub Codespaces
* **Best for**: Multi-platform teams or those requiring self-hosted option
* **Pricing**: Free tier available, paid plans from $9/user/month

*DevPod with Cloud Providers (Hybrid Approach)*
* **Pros**:
  - Unified CLI/UX for local and cloud environments
  - Freedom to choose any cloud provider (AWS, GCP, Azure, DigitalOcean)
  - Cost-effective - direct cloud VM pricing without markup
  - Complete control over machine types and configurations
* **Cons**:
  - Requires cloud infrastructure management
  - No built-in prebuild system
  - Less integrated experience compared to Codespaces
* **Best for**: Platform teams wanting full control and cloud flexibility
* **Pricing**: Direct cloud provider costs (e.g., AWS EC2 pricing)

*Coder (Enterprise Option)*
* **Pros**:
  - Self-hosted with complete control
  - Terraform-based provisioning for any infrastructure
  - Strong enterprise features (SSO, RBAC, audit logging)
  - Support for any IDE (VS Code, JetBrains, vim, etc.)
  - Best cost control for large teams
* **Cons**:
  - Requires platform engineering effort to operate
  - Larger upfront investment
  - Needs dedicated infrastructure
* **Best for**: Large enterprises requiring self-hosted solution
* **Pricing**: Open source free, Enterprise from $35/user/month

**Recommended Approach**

*Current Phase (Startup/Small Team)*
* Primary: GitHub Codespaces for cloud development
* Secondary: DevPod for local development
* Rationale: Minimal operational overhead, native GitHub integration, good free tier

*Growth Phase (>20 developers)*
* Consider: Gitpod for better cost controls and prebuild optimization
* Or: DevPod + AWS EC2 for maximum cost efficiency
* Or: Evaluate Coder if self-hosting aligns with platform strategy

*Decision Criteria*
* **Stick with Codespaces if**: 
  - Team < 50 developers
  - Fully committed to GitHub ecosystem
  - Prefer managed solution over self-hosting
* **Switch to Gitpod if**: 
  - Using multiple Git providers
  - Need better cost optimization at scale
  - Want self-hosted option without heavy operational lift
* **Move to Coder if**:
  - Team > 100 developers
  - Require strict compliance/data residency
  - Have platform engineering team to operate infrastructure

**Integration Points**

* **IDP Integration**: Track environment usage and costs in Port.io
* **Cost Management**: Tag environments by team/project for chargeback
* **Security Scanning**: Include CDE images in vulnerability scanning pipeline
* **Metrics & Observability**: Monitor environment spin-up times, usage patterns, costs
* **Documentation**: Maintain runbooks for common CDE scenarios in IDP

### Service Catalog / API Catalog

**Purpose**: The service catalog provides a centralized, searchable registry of all services, APIs, infrastructure, and resources across the platform. It answers questions like: "What services exist?", "Who owns this?", "What dependencies does this have?", and "Where is this deployed?".

#### Port.io as the Catalog

* [Port](https://www.port.io) is used for the service and API catalog
* Provides a **software catalog** that auto-discovers and tracks all platform entities
* Acts as the **single source of truth** for platform metadata
* Integrates with existing tools (GitHub, AWS, Kubernetes) to sync data automatically

#### Entity Model in Port.io

Port.io uses a **blueprint-based** entity model. Here's our catalog structure:

**Core Blueprints (Entities)**

1. **Service** (Microservices/Applications)
   * Properties:
     * Name, description, tier (critical/high/medium/low)
     * Owner team, tech lead, on-call contact
     * Programming language, framework version
     * Repository URL (GitHub)
     * Lifecycle stage (experimental/active/deprecated)
   * Relations:
     * Consumes → API (many-to-many)
     * Runs in → Environment (many-to-many)
     * Uses → Database (many-to-many)
     * Depends on → Service (service mesh dependencies)
     * Has → Deployment (one-to-many)

2. **API**
   * Properties:
     * API name, version, protocol (REST/GraphQL/gRPC)
     * OpenAPI/GraphQL schema URL
     * Base URL per environment
     * Authentication method (OAuth2, API Key, mTLS)
     * SLA/SLO definitions
   * Relations:
     * Provided by → Service (many-to-one)
     * Consumed by → Service (many-to-many)

3. **Environment**
   * Properties:
     * Name (dev/staging/production)
     * AWS account ID
     * EKS cluster name
     * Region (e.g., us-east-1)
     * Namespace prefix
     * Resource limits and quotas
   * Relations:
     * Contains → Service instances
     * Contains → Database branches (Neon)

4. **Database**
   * Properties:
     * Database name, type (PostgreSQL via Neon)
     * Main branch (production)
     * Connection string template
     * Size, storage limits
   * Relations:
     * Has → Database Branch (one-to-many)
     * Used by → Service (many-to-many)

5. **Database Branch** (Neon-specific)
   * Properties:
     * Branch name (maps to environment)
     * Parent branch
     * Created date, created by
     * Auto-delete policy
   * Relations:
     * Branch of → Database
     * Used in → Environment

6. **Repository**
   * Properties:
     * Repository name, visibility (public/private)
     * Primary language, topics/tags
     * Default branch
     * Has devcontainer? Has Score file?
     * Last commit date, commit count
   * Relations:
     * Contains code for → Service/Infrastructure

7. **Deployment**
   * Properties:
     * Deployment ID, timestamp
     * Deployed by (user/automation)
     * Commit SHA, image tag
     * Status (success/failed/in-progress)
     * Deployment duration
   * Relations:
     * Deploys → Service
     * To → Environment
     * Triggered by → Pipeline Run

8. **Pipeline Run** (CI/CD)
   * Properties:
     * Run number, status, duration
     * Triggered by (user/webhook/schedule)
     * Branch/tag
     * Test results, coverage %
   * Relations:
     * For → Repository
     * May create → Deployment

9. **Infrastructure Component** (Terraform modules)
   * Properties:
     * Component name (e.g., EKS cluster, VPC, ALB)
     * Terraform module version
     * AWS resources created
     * Cost per month (estimate)
   * Relations:
     * Provisioned in → Environment
     * Managed by → Repository (Terraform repo)

10. **Team**
    * Properties:
      * Team name, slack channel
      * Manager, team members
      * On-call rotation link (PagerDuty)
    * Relations:
      * Owns → Service (one-to-many)
      * Owns → Repository (one-to-many)

#### Catalog Data Sources and Integrations

**Automated Discovery via Port.io Integrations**

1. **GitHub Integration**
   * Auto-discovers repositories
   * Syncs: repository metadata, topics, languages, contributors
   * Tracks: Pull requests, workflow runs, releases
   * Updates: Real-time via webhooks
   * **Entities created/updated**: Repository, Pipeline Run

2. **Kubernetes Integration** (via Port K8s exporter)
   * Auto-discovers: Deployments, Services, Pods, ConfigMaps, Namespaces
   * Syncs: Resource status, replicas, images, labels
   * Health checks: Pod status, restarts, resource usage
   * Updates: Every 5 minutes (configurable)
   * **Entities created/updated**: Service (runtime info), Deployment, Environment

3. **AWS Integration** (via Port AWS exporter)
   * Auto-discovers: EKS clusters, RDS instances, S3 buckets, ECR repositories
   * Syncs: Resource tags, configurations, costs
   * Updates: Every 15 minutes
   * **Entities created/updated**: Environment, Infrastructure Component, Database

4. **ArgoCD Integration**
   * Syncs: Application status, sync status, health
   * Tracks: Deployment history, rollback events
   * Updates: Real-time via webhooks
   * **Entities created/updated**: Deployment

5. **Terraform Integration** (via custom GitHub Action)
   * Parses: Terraform state files
   * Syncs: Infrastructure components, resource dependencies
   * Cost data: From terraform-cost-estimation
   * Updates: After each Terraform apply
   * **Entities created/updated**: Infrastructure Component

6. **Neon Integration** (custom webhook)
   * Syncs: Databases, branches, connection strings
   * Tracks: Branch creation/deletion tied to environments
   * Updates: Real-time via Neon webhooks
   * **Entities created/updated**: Database, Database Branch

**Manual/Semi-Automated Entry**
* **Service ownership**: Defined in service repository's `port.yml` or `catalog-info.yaml`
* **API specifications**: Linked OpenAPI/GraphQL schema URLs in repository
* **SLOs**: Defined per service, tracked against actual metrics
* **Team information**: Manually managed (could sync from GitHub teams in future)

#### Catalog Metadata Management

**Metadata Sources Priority** (in case of conflicts):
1. **Explicit declarations** in repository (`port.yml`, `catalog-info.yaml`) - highest priority
2. **Runtime state** from Kubernetes/AWS - current actual state
3. **Infrastructure-as-Code** from Terraform - intended state
4. **Automated discovery** - fallback for undeclared resources

**Metadata Enrichment Pipeline**
```
Repository (port.yml)
  → GitHub webhook → Port ingestion
    → Merge with K8s runtime data
      → Enrich with AWS resource tags
        → Calculate relationships
          → Update catalog entity
```

**Metadata Quality Scoring** (Port.io Scorecards - see Port.io Capabilities section)
* Repository has README: +10 points
* Service has owner team: +20 points
* API has OpenAPI spec: +15 points
* Has health check endpoint: +10 points
* Has SLO defined: +15 points
* Has runbook links: +10 points
* Passing security scans: +20 points
* **Goal**: Bronze (50), Silver (70), Gold (90+)

#### Search and Discovery

**Port.io Search Capabilities**
* **Free-text search**: Across all entity properties
* **Filtered search**: By entity type, owner, environment, tier
* **Relationship traversal**: "Show me all services that depend on this API"
* **Saved searches**: Common queries saved for quick access

**Common Search Queries**
* "Show all critical tier services in production"
* "Which services consume the payments API?"
* "What databases does the checkout service use?"
* "Show all services owned by team-platform"
* "Find all deprecated services"

#### Catalog Benefits

**For Developers**
* **Discoverability**: Easily find services and APIs to reuse
* **Understanding**: See dependencies and relationships at a glance
* **Ownership clarity**: Know who to contact for questions/incidents
* **Self-service**: Know what's available without asking around

**For Platform Team**
* **Inventory management**: Know what exists and where
* **Compliance tracking**: Identify services missing required metadata
* **Cost attribution**: Tag resources by team/service for chargeback
* **Decommissioning**: Find and sunset unused/deprecated services

**For Incident Response**
* **Blast radius**: Quickly identify what's affected by an outage
* **Escalation**: Find on-call contact for affected services
* **Dependencies**: Trace issue to upstream/downstream services

#### Cost Considerations
* **Port.io Startup Plan**: Currently subscribed
* Includes: Unlimited entities, integrations, users
* **Cost**: Check current pricing (was ~$200-300/month for small teams)
* **Value**: Saves hours of manual documentation and tribal knowledge gaps 

### Internal Developer Portal

**Purpose**: The Internal Developer Portal (the UI layer) provides developers with a **single pane of glass** to interact with the platform. It's the primary interface for discovery, self-service actions, documentation, and visibility into the entire platform ecosystem.

#### Port.io as the Portal UI

* [Port](https://www.port.io) is used as the Internal Developer Portal
* Provides the **web interface** where developers interact with the platform
* **Not the platform itself** - it's the interface to the platform (see Terminology section)
* Accessible at: `https://app.getport.io/[org-name]` (or custom domain)

#### Portal Capabilities Overview

The portal serves four primary functions:

1. **Discover**: Find services, APIs, infrastructure, and documentation
2. **Understand**: Visualize dependencies, ownership, and system architecture
3. **Self-Serve**: Execute common tasks without platform team intervention
4. **Monitor**: Track deployments, incidents, and service health

#### Portal Home Page Experience

**Developer Landing Page** (when logging into Port.io)

```
┌──────────────────────────────────────────────────────────────┐
│  Welcome back, [Developer Name]                   [Profile]  │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Quick Actions                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Create    │  │   Deploy    │  │    Create   │         │
│  │ New Service │  │  to Staging │  │  DB Branch  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                               │
│  My Services (Services you own or contribute to)            │
│  ┌────────────────────────────────────────────────────┐     │
│  │ payment-service        │ ✅ Healthy │ v2.3.1      │     │
│  │ checkout-api           │ ⚠️  Warn   │ v1.8.2      │     │
│  │ user-profile-service   │ ✅ Healthy │ v3.1.0      │     │
│  └────────────────────────────────────────────────────┘     │
│                                                               │
│  Recent Deployments                                          │
│  • payment-service v2.3.1 → production (5 min ago) ✅       │
│  • checkout-api v1.8.2 → staging (2 hours ago) ✅          │
│                                                               │
│  Active Incidents                                            │
│  • [INC-123] High latency in checkout-api (P2)              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

#### Portal Navigation Structure

**Main Navigation Sections**

1. **Catalog** (Service Discovery)
   * All Services
   * All APIs
   * All Databases
   * All Infrastructure
   * All Teams
   * Custom entity views

2. **Self-Service** (Actions Hub)
   * Create new service from template
   * Deploy service to environment
   * Create database branch
   * Scale service replicas
   * Request access to resource
   * Run database migration

3. **Documentation** (Knowledge Base)
   * Getting Started guides
   * Golden Paths (how to do common tasks)
   * Runbooks (incident response)
   * Architecture Decision Records (ADRs)
   * Platform changelog
   * Tool documentation

4. **Insights** (Dashboards & Metrics)
   * Platform health overview
   * Deployment frequency & success rate
   * Service scorecards (compliance)
   * Cost dashboards
   * Security posture

5. **Search** (Global search across all entities and docs)

#### Self-Service Actions

**What are Self-Service Actions?**
* **Buttons/forms** in the Portal that trigger automated workflows
* Abstract away complexity of underlying platform tools
* Provide **golden paths** for common tasks
* Reduce wait time and dependencies on platform team
* All actions are **audited** and **logged**

**Example Self-Service Actions in Port.io**

**1. Scaffold New Service**
* **Trigger**: Button on Portal home or "Create Entity" flow
* **Input form**: 
  * Service name
  * Service type (API/Worker/Frontend)
  * Programming language (Python/Node.js/Go/Java)
  * Owner team
  * Include database? (Y/N)
* **Backend workflow** (GitHub Actions or Port Workflow):
  1. Create new GitHub repository from template
  2. Replace template variables with service name
  3. Create Neon database and branches (if requested)
  4. Create Port.io entity for the service
  5. Setup CI/CD workflows
  6. Create Kubernetes manifests in GitOps repo
  7. Generate initial documentation
* **Output**: Link to new repository, next steps guide

**2. Deploy Service to Environment**
* **Trigger**: Button on Service entity page
* **Input form**:
  * Service (auto-filled if clicked from service page)
  * Target environment (dev/staging/production)
  * Version/tag to deploy
  * Deployment strategy (rolling/blue-green/canary)
* **Backend workflow**:
  1. Validate user has permission for target environment
  2. Update image tag in GitOps repository
  3. Trigger ArgoCD sync (or auto-sync picks it up)
  4. Wait for deployment to complete
  5. Run smoke tests
  6. Update Port.io deployment entity
  7. Send Slack notification
* **Output**: Deployment status, link to ArgoCD, link to logs

**3. Create Database Branch** (Neon)
* **Trigger**: Button on Database entity page
* **Input form**:
  * Database (auto-filled)
  * Branch name (typically environment name)
  * Parent branch (default: main)
  * Auto-delete after X days? (for ephemeral environments)
* **Backend workflow**:
  1. Call Neon API to create branch
  2. Wait for branch to be ready
  3. Update Port.io with new Database Branch entity
  4. Store connection string in AWS Secrets Manager
  5. Update ESO ExternalSecret in K8s namespace
* **Output**: Connection string, branch details

**4. Request Production Access**
* **Trigger**: Button on Environment entity page or manual form
* **Input form**:
  * Reason for access
  * Duration needed (1 hour / 4 hours / 8 hours)
  * Resources needed (read-only / kubectl exec / full admin)
* **Backend workflow**:
  1. Create approval request (could integrate with Slack/PagerDuty)
  2. Notify approvers (manager + platform team)
  3. Upon approval: Temporarily grant AWS IAM role via Identity Center
  4. Upon approval: Add user to K8s RBAC group (time-limited)
  5. Log access in audit system
  6. Auto-revoke after duration
* **Output**: Access credentials, expiration time, audit trail

**5. Scale Service Replicas**
* **Trigger**: Button on Service entity page
* **Input form**:
  * Service (auto-filled)
  * Environment
  * Desired replica count
  * Reason (optional, for audit)
* **Backend workflow**:
  1. Validate user permission (can only scale non-prod without approval)
  2. Update deployment replicas in GitOps repo
  3. ArgoCD syncs change
  4. Monitor rollout status
  5. Update Port.io entity
* **Output**: New replica count, pods status

**6. Run Database Migration**
* **Trigger**: Button on Service entity page or Database page
* **Input form**:
  * Database
  * Environment
  * Migration version/tag
  * Dry-run first? (Y/N)
* **Backend workflow**:
  1. Pull migration scripts from repository
  2. Run in dry-run mode if requested
  3. Execute migration job in K8s
  4. Monitor job completion
  5. Update schema version in Port.io
  6. Log migration in audit trail
* **Output**: Migration logs, new schema version

**Action Permissions**
* Actions can have **role-based access control**
* Examples:
  * All developers: Create service in dev environment
  * Service owners only: Deploy to staging
  * Platform team only: Deploy to production (or requires approval)
  * Platform team only: Delete production resources

#### Golden Paths Integration

**What are Golden Paths?**
* **Opinionated, pre-paved routes** to accomplish common tasks
* Best practices codified into workflows
* Reduces decision fatigue ("How should I do X?")
* Ensures consistency and compliance
* Makes the "right way" the "easy way"

**Golden Paths in the Portal**

**1. "Create a New Service" Golden Path**
* **Documentation page** in Portal: "How to Create a New Service"
* **Steps**:
  1. Click "Create New Service" in Portal
  2. Fill in service details (form guides you through required metadata)
  3. Portal scaffolds repository with:
     * devcontainer.json for local development
     * Dockerfile following security best practices
     * GitHub Actions CI/CD pipeline
     * score.yaml for infrastructure requirements
     * OpenAPI spec template (if API)
     * Basic tests and linting
  4. Portal creates database if needed
  5. Portal sets up monitoring dashboards (Grafana)
  6. Portal creates initial Port.io entity
* **Result**: Fully functional service in <5 minutes, ready to develop

**2. "Deploy to Production" Golden Path**
* **Documentation page**: "Production Deployment Checklist"
* **Steps**:
  1. Service must pass scorecard checks (Bronze minimum):
     * Has README
     * Has owner
     * Has health check endpoint
     * Has passing tests
     * Has security scan (no high/critical vulns)
  2. Create PR to update production image tag in GitOps repo
  3. Automated checks run:
     * Terraform plan (if infra changes)
     * Policy checks (Kyverno/OPA)
     * Approval required from service owner + platform team
  4. Merge PR
  5. ArgoCD auto-syncs (or use "Deploy" action in Portal)
  6. Monitor deployment in Portal (shows ArgoCD status, pod health)
  7. Smoke tests run automatically
  8. Rollback button available in Portal if issues
* **Result**: Safe, auditable production deployment

**3. "Onboard a New Team Member" Golden Path**
* **Documentation page**: "Developer Onboarding Guide"
* **Steps**:
  1. Add to GitHub organization
  2. Add to AWS IAM Identity Center group (auto-provisions access)
  3. Portal shows personalized onboarding checklist:
     * Install devpod locally
     * Clone sample repository
     * Setup GitHub Copilot
     * Setup Claude Code
     * Tour of Portal features
     * First deployment (to dev environment)
  4. Portal guides through first self-service action
* **Result**: Developer productive on day one

#### Documentation Integration

**Documentation in Port.io**

Port.io supports embedding documentation directly in the Portal:

**Documentation Types**

1. **Getting Started Guides**
   * Platform overview
   * How to use the Portal
   * How to develop locally (devcontainers)
   * How to deploy your first service

2. **Golden Path Guides** (see above)
   * Create new service
   * Deploy to production
   * Create API endpoint
   * Add database to service
   * Implement observability (OTel)
   * Secure your service

3. **Runbooks** (Incident Response)
   * Linked to service entities
   * Stored in service repositories (docs/runbooks/)
   * Searchable in Portal
   * Examples:
     * "Service is returning 5xx errors"
     * "Database connection pool exhausted"
     * "High memory usage"
     * "Traffic spike handling"

4. **Architecture Decision Records (ADRs)**
   * Why we chose Neon for databases
   * Why we chose OTel for observability
   * Why we use monorepo vs multirepo (answer: multirepo)
   * Service mesh decision (Istio vs Linkerd vs none)

5. **API Documentation**
   * OpenAPI specs rendered in Portal
   * GraphQL schema rendered
   * Try-it-out functionality
   * Example requests/responses

6. **Platform Changelog**
   * New features in the platform
   * Upcoming changes (breaking or otherwise)
   * Deprecation notices

**Documentation Sources**
* **Markdown files** in Git repositories
* **Port.io pages** (native documentation feature)
* **External links** (to Confluence, Notion, etc. if used)
* **Auto-generated** from code (OpenAPI from annotations)

**Documentation Search**
* Unified search across all documentation
* Indexed by Port.io
* Filter by: type, team, service, tag

#### Visibility and Dashboards

**Service Health Dashboard** (per service)
* Current deployment version per environment
* Health status (from K8s probes + OTel metrics)
* Recent deployments (last 10)
* Open incidents
* SLO compliance (error rate, latency)
* Dependencies (services, databases, APIs)
* Owner and on-call information

**Platform Overview Dashboard**
* Total services, APIs, databases
* Deployment frequency (last 7/30 days)
* Deployment success rate
* Service scorecard distribution (Bronze/Silver/Gold)
* Services by tier (critical/high/medium/low)
* Top services by incident count
* Orphaned services (no owner)

**Team Dashboard** (for each team)
* Services owned by team
* Deployment activity
* Incident ownership and MTTR
* Scorecard compliance
* On-call rotation

**Cost Dashboard** (if cost data integrated)
* Cost per service
* Cost per environment
* Top 10 most expensive services
* Cost trends over time

#### Developer Experience Goals

**Portal UX Principles**

1. **Discoverability**: Everything is searchable and linked
2. **Self-service**: Common tasks don't require tickets or waiting
3. **Transparency**: See status of all requests and workflows
4. **Feedback**: Actions provide clear success/failure messages
5. **Learning**: Documentation is contextual and just-in-time

**Metrics to Track** (Portal effectiveness)
* Time to first deployment (new developer)
* Self-service action usage vs. manual requests
* Portal daily active users
* Documentation search queries (identify gaps)
* Average time to complete common tasks

#### Portal Access and Permissions

**Authentication**
* SSO via GitHub (primary)
* Or: AWS IAM Identity Center SAML integration (future)

**Authorization**
* **All developers**: Read access to catalog, documentation
* **Service owners**: Execute actions on their services
* **Platform team**: Admin access, all actions
* **Read-only users**: Stakeholders, management (view only)

**Audit Trail**
* All self-service actions logged
* Shows: who, what, when, why (if reason provided)
* Filterable by user, action type, entity
* Retention: 1 year minimum

#### Cost Considerations

**Port.io Startup Plan**
* Current subscription level
* Includes: 
  * Unlimited entities and integrations
  * Self-service actions
  * Full API access
  * Reasonable user limits
* **Cost**: ~$200-300/month (verify current pricing)

**ROI Calculation**
* Platform team time saved: ~20 hours/month (manual tasks automated)
* Developer time saved: ~5 hours/dev/month (self-service vs. waiting)
* For team of 10 devs: 70 hours saved/month
* At $100/hour blended rate: **$7,000/month value**
* **ROI**: ~23x minimum

#### Future Portal Enhancements

**Phase 2 Enhancements** (next 6 months)
* Embed Grafana dashboards directly in service pages
* AI-powered search (semantic search across catalog and docs)
* Automated runbook suggestions based on alerts
* Service dependency graph visualization (interactive)
* Cost optimization recommendations

**Integration Wishlist**
* PagerDuty: Show on-call schedule and incidents
* Slack: Two-way integration (alerts in Slack, actions from Slack)
* Jira: Link incidents to tickets automatically
* Grafana: Embedded dashboards per service

### Port.io Capabilities

Have paid license (Startup Plan)

* We would like to leverage more of the capabilities of Port.io, including:
    * Context lake
    * Actions (example: scaffold a new service)
    * Scorecards
    * Workflow Orchestrator
    * Integrations (with the tools we have in this reference architecture)

### Version Control

* All repositories are in GitHub
* Organization setup and have GitHub Team plan subscription

#### Application Source Code

* Every application service/component has its own repository
* Repository for each micro-frontend component
* Repository for each back-end microservice

#### Platform Source Code

* Repository for Terraform (included IaC for each environment, organized by layers)
* Repository for own Terraform Modules that are versioned via tags and used in above Terraform repo
* Repository for dotfiles (personal setup of IDE per developer)
* Repository for CICD and automation pipelines (centralize, used by application repositories)
* Repository for Kubernetes manifests (used by ArgoCD / adopts GitOps)

##### K8s GitOps Repository

Much like for have layers in Terraform, we also have layers for K8s manifests

* **Foundation Layer** - Helm charts for things like Istio, Kyverno, ESO
    * Not managed by ArgoCD
    * Use Kustomize to create variations for environments
* **Application Layer** - Manifests for the "application stack"
    * Managed by ArgoCD
    * Use Kustomize for environment variations, but should be minimal

## Integration and Delivery Plane

### Continuous Integration (CI) Pipeline

* GitHub Actions used for CI pipeline (code in CICD repository)
* Separate pipeline for application services/components
* Separate pipeline for Terraform infrastructure-as-code provisioning
* For all the pipeline validates the relevant linting rules
* For all the pipeline runs static analysis and security scanning (using GitHub services)
* For all the pipeline runs the automated unit and integration tests
* For application services/components container images are created
* Container images are loaded to the Registry

#### Infrastructure Pipelines

* GitHub Actions in the Terraform repository
* Score.dev file used in application repositories
* Score.dev file is how Developers specify what infra is needed
* Link in Score.dev to documentation that guides Developers
* We define the supported golden paths / paved paths for Score.dev
* We build tool to validate score file and convert to Terraform / K8s manaifests

### Registry

* Use AWS ECR

### Platform Orchestrator

* Logic and workflows managed in custom GitHub Actions pipeline
* In future, will look to use Humanitec

### Continuous Delivery (CD) Pipeline

#### Applications

* ArgoCD will be used for deploying applications to environments
* Git repositority of K8s manifest used as basis for ArgoCD
* Changes to this repository will trigger ArgoCD to make changes to the K8s cluster
* This includes Custom Resource Definiions (CRDs) using Crossplane within the cluster
* The container images created by the CI pipelines (and in AWS ECR) are deployed to cluster

#### Infrastructure

* Terraform repository will be used for deploying changes to Infrastructure foundation
* Foundations cover provisioning the AWS EKS cluster (i.e. the K8s cluster)
* Foundations also cover "management" layer (Administering the "environments")

## Observability Plane

* **OpenTelemetry (OTel)** as the unified observability framework and vendor-neutral standard
* Focus on learning and certification preparation for Grafana and Prometheus
* Leverage free tiers and low-cost AWS services where appropriate
* OTel provides a single set of APIs, SDKs, and tools for all three pillars (metrics, logs, traces)

### OpenTelemetry Architecture

#### OpenTelemetry Collector
* **Central component** for receiving, processing, and exporting telemetry data
* Deployed as DaemonSet (for logs/host metrics) and Deployment (for traces/app metrics) in EKS
* Vendor-agnostic: can export to multiple backends simultaneously
* Supports processors for filtering, sampling, batching, and enrichment

#### Collection Pattern
```
Applications (OTel SDKs) 
  → OTel Collector (receives OTLP)
    → Processors (transform, sample, enrich)
      → Exporters (to Prometheus, Loki, Tempo, etc.)
```

#### OpenTelemetry Operator
* Deploy **OTel Operator** in EKS for simplified management
* Auto-instrumentation for supported languages (Java, .NET, Python, Node.js, Go)
* Manages OTel Collector instances via CRDs
* Simplifies instrumentation: add annotations to pods for auto-instrumentation

### Metrics

#### OpenTelemetry Metrics
* Applications instrumented with **OTel SDKs** to generate metrics
* OTel supports three metric types: Counter, Gauge, Histogram
* Metrics follow **OpenTelemetry Semantic Conventions** for consistency
* OTel Collector receives metrics via **OTLP** (OpenTelemetry Protocol)

#### Prometheus Integration
* **Prometheus** as the metrics storage and query engine
  * Deployed in EKS cluster using the kube-prometheus-stack Helm chart
  * **OTel Collector exports metrics to Prometheus** via Remote Write or prometheus exporter
  * Can also scrape Prometheus exposition format metrics directly
  * AlertManager for alert routing and notification management
  * Includes node-exporter for host-level metrics
  * kube-state-metrics for K8s object state metrics

#### OTel Collector Metrics Pipeline
```
App (OTel SDK) → OTLP → OTel Collector → Prometheus Remote Write → Prometheus
                                       ↘ Also exports to AWS AMP (optional)
```

#### Application Metrics Strategy
* **Primary**: Instrument applications with OTel SDKs (vendor-neutral)
  * Automatic instrumentation for frameworks (HTTP servers, databases, messaging)
  * Custom business metrics via OTel Meter API
* **Legacy**: Support for existing Prometheus client libraries
  * OTel Collector can scrape `/metrics` endpoints in Prometheus format
  * Gradually migrate to OTel SDKs for unified instrumentation

#### Metrics Standards
* Follow **RED method** for services: Rate, Errors, Duration
* Follow **USE method** for resources: Utilization, Saturation, Errors
* Apply **OTel Semantic Conventions** for attribute names:
  * `service.name`, `service.version`, `deployment.environment`
  * Standard HTTP, database, messaging attributes
  * Ensures consistency across services and languages

#### Metrics Storage and Retention
* Short-term: Prometheus local storage (15-30 days)
* Long-term options:
  * **Grafana Mimir** (OTel native, free tier available)
  * **AWS Managed Prometheus (AMP)** - supports OTLP and Prometheus Remote Write
  * **Thanos or Cortex** for self-hosted long-term storage
* OTel Collector can export to multiple backends simultaneously

#### Cost Considerations
* Start with self-hosted Prometheus in EKS (no additional cost beyond compute)
* Grafana Cloud free tier includes: 10k series, 14-day retention
* AWS Managed Prometheus: Pay per metric ingestion (~$0.30 per million samples)
  * Supports OTLP native ingestion (no conversion needed)
* OTel Collector batching and filtering reduces data volume and costs

### Logging

#### OpenTelemetry Logs
* Applications emit logs via **OTel Logs SDK** or OTel Logs Bridge API
* OTel Collector receives logs via **OTLP** protocol
* Supports automatic context injection (trace_id, span_id) for correlation
* Backwards compatible: can ingest logs from existing logging frameworks

#### Log Collection Architecture
* **OTel Collector** as primary log collector
  * DaemonSet deployment for collecting stdout/stderr from containers
  * Filelog receiver for reading Kubernetes container logs
  * OTLP receiver for applications using OTel SDK directly
  * Processors for parsing, filtering, and enriching logs
  
* **Fluent Bit** as complementary/alternative collector (optional)
  * Lightweight option if OTel Collector overhead is a concern
  * Can forward to OTel Collector or directly to backends
  * Native Kubernetes metadata enrichment

#### OTel Collector Logs Pipeline
```
Container Logs → Filelog Receiver → OTel Collector → Loki Exporter → Loki
App (OTel SDK) → OTLP Receiver → OTel Collector → Loki Exporter → Loki
                                                  ↘ CloudWatch (optional)
```

#### Log Aggregation and Storage
* **Grafana Loki** as primary log aggregation system
  * Deployed in EKS using Helm chart (loki-stack or loki-distributed)
  * Receives logs from OTel Collector via Loki exporter
  * Cost-effective: indexes only metadata (labels), not full text
  * S3-compatible storage backend (AWS S3) for log chunks
  * Integrated with Grafana for querying via LogQL
  * **Native OTel support**: Loki can receive OTLP directly (v2.8+)

#### AWS CloudWatch Integration (Optional)
* Use selectively for AWS-native services (ALB, RDS, Lambda if used)
* OTel Collector can export to CloudWatch via awscloudwatchlogs exporter
* Configure retention policies to minimize costs (7-14 days)
* CloudWatch Logs Insights for ad-hoc queries on AWS service logs

#### Structured Logging Best Practices
* All applications emit **structured logs** (JSON format preferred)
* OTel Semantic Conventions for log attributes:
  * `service.name`, `service.version`, `deployment.environment`
  * Automatic: `trace_id`, `span_id` (for correlation with traces)
* Consistent log levels: TRACE, DEBUG, INFO, WARN, ERROR, FATAL
* Key fields: `timestamp`, `severity`, `body`, `attributes`, `resource`

#### Log-Trace Correlation
* OTel automatically injects trace context into logs
* Grafana can navigate from logs to traces and vice versa
* Correlation via `trace_id` and `span_id` attributes
* Enables root cause analysis: log error → trace → affected services

#### Cost Considerations
* Self-hosted Loki in EKS (compute cost only)
* S3 storage for Loki chunks (very low cost, ~$0.023/GB/month)
* Grafana Cloud free tier: 50GB logs, 14-day retention (with OTLP support)
* Avoid CloudWatch Logs for high-volume application logs (expensive at scale)
* OTel Collector filtering reduces log volume and costs

### Traces

#### OpenTelemetry Distributed Tracing
* **OTel SDKs** as the instrumentation standard (vendor-neutral)
  * Language-specific SDKs: Java, .NET, Python, Node.js, Go, Ruby, PHP, etc.
  * Automatic instrumentation via OTel Operator in Kubernetes
  * Manual instrumentation for custom spans and business context
  * **OTLP (OpenTelemetry Protocol)** for data transmission (gRPC or HTTP)

#### Trace Collection and Processing
* **OpenTelemetry Collector** as the trace receiver and processor
  * Receives traces via OTLP from applications
  * Processors for:
    * **Batch processor**: Reduces export frequency, improves efficiency
    * **Sampling processor**: Tail-based sampling for cost control
    * **Attributes processor**: Add, update, or remove span attributes
    * **Resource processor**: Enrich with K8s metadata
  * Can export to multiple backends simultaneously

#### OTel Collector Traces Pipeline
```
App (OTel SDK) → OTLP → OTel Collector → Tempo Exporter → Tempo
                                       ↘ OTLP Exporter → AWS X-Ray (optional)
                                       ↘ Metrics generation (span metrics)
```

#### Trace Storage and Analysis
* **Grafana Tempo** as primary tracing backend
  * Fully OTel-native: accepts OTLP directly
  * Can receive from OTel Collector or directly from applications
  * Deployed in EKS (single binary or distributed mode)
  * S3-compatible storage (AWS S3) for trace data
  * Integrated with Grafana for visualization
  * **TraceQL**: Tempo's query language for trace search
  * Correlated with logs and metrics for complete observability

#### Alternative/Complementary Options
* **Jaeger** (CNCF project, OTel-native)
  * Alternative to Tempo, more mature UI
  * OTel Collector has native Jaeger exporter
  * Can run alongside Tempo for evaluation
  
* **AWS X-Ray** for AWS service tracing
  * OTel Collector supports X-Ray exporter (awsxray exporter)
  * Useful for tracing through AWS services (ALB, API Gateway, Lambda)
  * Shows AWS service latency and errors
  * Free tier: 100k traces recorded/month, 1M retrieved/month
  * Can combine: Tempo for app traces, X-Ray for AWS infrastructure

#### Context Propagation
* **W3C Trace Context** as the standard (default in OTel)
  * `traceparent` and `tracestate` HTTP headers
  * Ensures trace context propagation across service boundaries
  * Compatible with most modern frameworks and proxies
* Support for legacy propagators (B3, Jaeger) if needed

#### Span Semantics and Best Practices
* Follow **OTel Semantic Conventions** for span attributes:
  * HTTP: `http.method`, `http.status_code`, `http.route`
  * Database: `db.system`, `db.statement`, `db.name`
  * Messaging: `messaging.system`, `messaging.destination`
  * RPC: `rpc.system`, `rpc.service`, `rpc.method`
* Consistent naming: Use semantic convention names, not custom names
* Add business context as span attributes (user_id, tenant_id, etc.)

#### Sampling Strategy
* **Head-based sampling** in applications (configure via OTel SDK)
  * Always sample: errors, slow requests (> threshold)
  * Probabilistic sampling: percentage-based for normal traffic
  * Parent-based: inherit sampling decision from parent span
  
* **Tail-based sampling** in OTel Collector
  * Make sampling decision after seeing entire trace
  * Always keep: traces with errors, high latency, specific attributes
  * More intelligent but requires buffering and processing power
  
* Configuration per environment:
  * Development: 100% sampling (full visibility)
  * Staging: 50% sampling
  * Production: 10-20% sampling (adjust based on traffic)

#### Span Metrics Generation
* OTel Collector **spanmetrics processor** generates RED metrics from traces
  * Rate: request rate per service/endpoint
  * Errors: error rate per service/endpoint
  * Duration: latency distribution (histogram)
* Exported to Prometheus for alerting and dashboards
* Provides service-level metrics without additional instrumentation

#### Cost Considerations
* Self-hosted Tempo in EKS (compute cost only)
* S3 storage for traces (low cost, ~$0.023/GB/month)
  * Implement lifecycle policies: delete after retention period
* Grafana Cloud free tier: 50GB traces, 14-day retention (OTLP native)
* AWS X-Ray: Stay within free tier for supplementary AWS service traces
* Sampling dramatically reduces costs: 10% sampling = 90% cost reduction

### Visualization and Dashboards

#### Grafana
* **Grafana** as unified observability UI
  * Supports all OTel data natively (metrics, logs, traces)
  * Self-hosted in EKS or use Grafana Cloud (free tier: 3 users, 10k metrics, 50GB logs/traces)
  * Single pane of glass for correlated telemetry data
  * Pre-built dashboards from Grafana community
  * Custom dashboards for application-specific metrics

#### Data Source Configuration
* **Prometheus**: Metrics query via PromQL
* **Loki**: Logs query via LogQL
* **Tempo**: Traces query via TraceQL
* **Exemplars**: Link metrics to traces (Prometheus → Tempo)
* **Correlations**: Navigate between logs, metrics, and traces seamlessly

#### Dashboard Strategy
* **Golden Signals**: Latency, Traffic, Errors, Saturation
* **Infrastructure dashboards**: 
  * Cluster overview: Node health, pod status, resource utilization
  * Node details: CPU, memory, disk, network
  * Pod/Container views: Per-service resource consumption
* **Application dashboards**: 
  * Per-service RED metrics (from OTel span metrics)
  * SLI dashboards: Availability, latency percentiles, error rates
  * Dependency graphs: Service topology from trace data
* **Business metrics dashboards**: 
  * User flows, conversion rates, feature usage
  * Custom business KPIs from OTel instrumentation
* **Cost dashboards**: 
  * AWS cost explorer data
  * Resource utilization and efficiency
  * OTel Collector metrics (data volume, processing time)

#### Trace-Metrics-Logs Correlation
* **Exemplars**: Prometheus metrics link to example traces
  * Click on metric spike → see traces from that time period
* **Trace to Logs**: Click on span → see related logs (matched by trace_id)
* **Logs to Traces**: Click on log entry → see full trace context
* **Unified view**: Single Grafana dashboard showing all three signals

#### Learning Path Alignment
* **Prometheus Certified Associate (PCA)**: 
  * PromQL queries and functions
  * Recording rules for OTel-derived metrics
  * Alerting rules and AlertManager configuration
  * Prometheus architecture and scaling
  
* **Grafana Certified Associate (GCA)**: 
  * Dashboard creation and templating
  * Data source configuration (Prometheus, Loki, Tempo)
  * Transformations and calculations
  * Alerting and notification channels
  * User management and permissions
  
* **OTel Learning**:
  * OTel Collector configuration and pipelines
  * SDK instrumentation across multiple languages
  * Semantic conventions and best practices
  * Sampling strategies and optimization

### Alerting

#### Alert Management
* **AlertManager** (part of Prometheus stack)
  * Handles alerts fired by Prometheus rules
  * Grouping, throttling, and silencing capabilities
  * Routes alerts based on labels and severity
  * Supports inhibition rules (suppress related alerts)

#### Alert Sources
* **Prometheus Alerts**: Metrics-based alerting via PromQL
  * Infrastructure alerts: Node down, high CPU, memory pressure
  * Application alerts: High error rate, slow response time, low throughput
  * OTel-derived alerts: Span metrics RED signals
  
* **Grafana Alerts**: Multi-source alerting (metrics, logs, traces)
  * Can alert on Loki queries (log patterns)
  * Can alert on Tempo queries (trace anomalies)
  * Unified alerting across all data sources

#### Alert Design Best Practices
* **Symptom-based alerts**, not cause-based
  * Alert on: "API latency > 500ms" (symptom)
  * Not on: "CPU > 80%" (possible cause)
* **SLO-based alerting**: Alert when error budget is burning
* **Runbooks linked** in alert annotations
  * Include troubleshooting steps
  * Link to relevant dashboards
  * Include OTel trace query examples
* **Alert severity levels**: 
  * **Critical**: Page on-call, immediate action required
  * **Warning**: Investigate during business hours
  * **Info**: Logged for awareness, no action needed
* **Avoid alert fatigue**: 
  * Tune thresholds based on historical data
  * Use multi-window, multi-burn-rate alerting for SLOs
  * Regular alert review and pruning

#### Notification Channels
* Start with low-cost/free channels:
  * **Slack** (free tier): Team notifications, incident channels
  * **Email** (AWS SES free tier: 3000 emails/month via EC2)
  * **PagerDuty** (free tier: 1 user) for critical production alerts
  * **Webhook integrations**: Custom workflows, ticketing systems
* Routing strategy:
  * Critical → PagerDuty (page on-call)
  * Warning → Slack (team channel)
  * Info → Email (digest)

#### Alert Enrichment with OTel Context
* Include **trace_id** in alert annotations when available
  * Direct link to example trace showing the issue
* Include **service.name**, **deployment.environment** attributes
* Link to pre-filtered Grafana dashboards (by service, time range)
* Include PromQL/LogQL queries for investigation

### Service Level Objectives (SLOs)

* Define **SLIs** (Service Level Indicators) for each service using OTel data
  * Availability: Success rate from span metrics
  * Latency: P95/P99 from span duration histograms
  * Throughput: Request rate from span metrics
* Track **SLOs** using Prometheus recording rules
* Calculate **error budgets** and visualize in Grafana
* Alert when error budget burn rate is too high
* **Sloth** or **Pyrra** for SLO management (both open source, OTel-compatible)

### OpenTelemetry Implementation Strategy

#### Phase 1: Foundation (Month 1-2)
* Deploy OTel Operator in EKS cluster
* Deploy OTel Collector (DaemonSet + Deployment)
* Configure backends: Prometheus, Loki, Tempo
* Set up Grafana with all data sources
* Instrument one pilot service end-to-end

#### Phase 2: Application Rollout (Month 3-4)
* Auto-instrumentation via OTel Operator for all services
  * Add annotation to deployments: `instrumentation.opentelemetry.io/inject-java: "true"`
  * Supports Java, .NET, Python, Node.js, Go out of the box
* Custom instrumentation for business metrics
* Implement sampling strategies per environment
* Create standard dashboards for all services

#### Phase 3: Advanced Features (Month 5-6)
* Tail-based sampling in OTel Collector
* Span metrics generation (spanmetrics processor)
* Exemplars configuration (link metrics to traces)
* SLO implementation with alerting
* Service mesh integration (if using Istio/Linkerd)
  * OTel can collect metrics from service mesh
  * Correlate service mesh traces with app traces

#### Language-Specific Considerations
* **Java**: Excellent auto-instrumentation support, mature
* **Python**: Good auto-instrumentation, growing ecosystem
* **Node.js**: Good support, requires some manual spans for custom code
* **Go**: More manual instrumentation required, but lightweight
* **.NET**: Good auto-instrumentation support
* **Frontend (Browser)**: OTel JS for RUM (Real User Monitoring)

#### OTel Collector Configuration Management
* Store collector configs in Git (GitOps)
* Use ConfigMaps in Kubernetes
* Version and test configs before deployment
* Monitor collector health metrics:
  * `otelcol_receiver_accepted_spans`
  * `otelcol_exporter_sent_spans`
  * `otelcol_processor_dropped_spans`

#### Migration from Legacy Observability
* OTel Collector can bridge legacy and modern:
  * Receive Prometheus metrics while migrating to OTel metrics
  * Receive Zipkin/Jaeger traces while migrating to OTLP
  * Receive fluentd/fluentbit logs while migrating to OTel logs
* Gradual migration: Run both systems in parallel during transition
* Export to multiple backends during migration for validation

### Cost Optimization Strategy

#### Start Small (Free/Low-Cost)
1. Self-host Prometheus, Loki, Tempo in EKS
2. Use S3 for long-term storage (cheap)
3. Grafana Cloud free tier for visualization (optional)
4. Stay within AWS free tiers (X-Ray, CloudWatch minimal usage)

#### Scale Considerations
* Monitor monthly costs via AWS Cost Explorer
* Set up billing alerts (AWS Budgets free tier: 2 budgets)
* Optimize retention periods based on actual needs
* Use sampling and filtering to reduce data volume
* Consider managed services (AMP, Grafana Cloud paid) only when self-hosting becomes operational burden

#### Estimated Monthly Costs (assuming small-scale)
* EKS cluster compute: $30-50 (covered by existing infrastructure)
* S3 storage (logs/traces): $2-5
* AWS CloudWatch (minimal): $0-10
* Total additional cost: ~$10-15/month (mostly storage)

## Security Plane

### Secrets Management

#### Secret Storage Strategy

**Primary: AWS Secrets Manager**
* Store application secrets in AWS Secrets Manager
* **Cost optimization**:
  * $0.40 per secret per month
  * $0.05 per 10,000 API calls
  * Strategy: Minimize number of secrets, use secret rotation wisely
  * Cache secrets in applications (reduce API calls)
  * Use Parameter Store for non-sensitive config (free up to 10,000 parameters)

**Alternative: HashiCorp Vault**
* **When to consider Vault**:
  * If you have 50+ secrets: Vault becomes cheaper
  * Breakeven: ~$20/month (50 secrets in AWS SM vs. self-hosted Vault)
  * If you want dynamic secrets (database credentials, AWS credentials)
  * If you need advanced features: PKI, encryption as a service
  
* **Self-hosted Vault in EKS**:
  * Cost: Only compute resources (~$10-20/month for small deployment)
  * High availability: Requires 3+ replicas (increases cost)
  * Backend storage: Can use AWS S3 (cheap) or integrated Raft storage
  * Operational overhead: Unsealing, backups, upgrades
  
* **Vault HCP (Managed)**:
  * Development tier: ~$0.50/hour (~$360/month) - too expensive for personal use
  * Not recommended for cost-sensitive deployments

**Recommended Approach**:
* **Start with AWS Secrets Manager** (predictable, low maintenance)
* **Migrate to self-hosted Vault** if:
  * Secret count exceeds 40-50
  * Need dynamic secrets features
  * Want to learn Vault deeply (good for career development)

#### Secret Access Patterns

**Kubernetes Secret Management**
* **External Secrets Operator (ESO)** for syncing secrets into K8s
  * Fetches secrets from AWS Secrets Manager
  * Creates/updates Kubernetes Secrets automatically
  * Supports secret rotation and refresh
  * Can also support Vault as backend (migration path)

**Application Secret Access**
* **Option 1: Kubernetes Secrets** (via ESO)
  * Apps read from K8s secrets (env vars or mounted volumes)
  * Simple, no AWS SDK required in apps
  * ESO handles the AWS Secrets Manager integration
  
* **Option 2: Direct AWS API** (for AWS-native apps)
  * Use AWS SDK with IRSA (IAM Roles for Service Accounts)
  * Reduces K8s secret sprawl
  * Better audit trail in AWS CloudTrail
  * Client-side caching reduces API costs

**Secret Rotation**
* Enable automatic rotation for database credentials
* AWS Secrets Manager Lambda rotation (AWS provides templates)
* Rotation cost: Lambda executions (minimal, ~$0.01/month)
* Applications must handle secret refresh gracefully

#### Secret Best Practices
* **Never commit secrets to Git** (use git-secrets or gitleaks hooks)
* **Least privilege**: Scope secrets to specific services
* **Encryption at rest**: AWS Secrets Manager uses KMS (included in cost)
* **Audit logging**: CloudTrail logs all secret access
* **Secret scanning**: GitHub Advanced Security (free for public repos)

### Identity Management

#### AWS Identity Strategy

**AWS Organizations Structure**
```
Management Account (SSO, Billing)
├── Security Account (Audit, Logging)
├── Shared Services Account (DevOps tools)
├── Development Account (EKS dev cluster)
├── Staging Account (EKS staging cluster)
└── Production Account (EKS prod cluster)
```

**AWS IAM Identity Center (AWS SSO)**
* Enable in Management Account
* **Permission Sets** for role-based access:
  * **PlatformAdmin**: Full access to all accounts (emergency only)
  * **PlatformEngineer**: Admin access to dev/staging, read-only to prod
  * **Developer**: Access to dev environment, limited staging access
  * **ReadOnly**: View-only access for auditing
  * **SecurityAuditor**: Security-specific read access across all accounts

**Permission Set Design**
```
Permission Set: PlatformEngineer
├── Managed Policy: ViewOnlyAccess
├── Managed Policy: PowerUserAccess (in dev/staging)
├── Custom Policy: EKS Management
├── Custom Policy: Secrets Management (read/write)
└── Session Duration: 8 hours
```

**Cross-Account Access**
* SSO handles cross-account role assumption
* No need for IAM users or long-term credentials
* MFA enforced at SSO level
* Session tags for attribute-based access control (ABAC)

**Cost**: AWS IAM Identity Center is **FREE** ✅

#### Kubernetes Identity and Access Management

**RBAC (Role-Based Access Control) Strategy**

**Integration with AWS IAM**
* **aws-auth ConfigMap**: Maps IAM roles to K8s groups
* **IRSA (IAM Roles for Service Accounts)**: Pods assume IAM roles
  * No need to store AWS credentials in pods
  * Fine-grained IAM permissions per service
  * Automatic credential rotation

**K8s RBAC Hierarchy**
```
ClusterRole (cluster-wide)
├── cluster-admin: Platform team only
├── admin: Namespace admins
├── edit: Developers
└── view: Read-only access

Role (namespace-scoped)
├── developer: Deploy apps, view logs
├── operator: All of above + scale, restart
└── viewer: Read-only in namespace
```

**Recommended RBAC Structure**

**ClusterRoles (Cluster-wide permissions)**
* **platform-admin**: 
  * For platform team only
  * All permissions across all namespaces
  * Can manage CRDs, operators, cluster infrastructure
  
* **platform-viewer**:
  * Read-only access to cluster resources
  * Security audit, troubleshooting
  
* **developer-base**:
  * Base permissions for all developers
  * View namespaces, nodes (read-only)

**Namespace Roles (Per-environment/service)**
* **namespace-admin**:
  * Full control within namespace
  * Cannot modify namespace itself or resource quotas
  * For service owners
  
* **namespace-developer**:
  * Deploy applications, scale, view logs
  * Cannot modify RBAC, network policies, resource quotas
  * For development team members
  
* **namespace-viewer**:
  * Read-only access to namespace resources
  * For stakeholders, support team

**Service Accounts (For Applications)**
* Each application deployment gets dedicated ServiceAccount
* IRSA binding for AWS access
* Minimal permissions via RoleBindings
* Never use default ServiceAccount

**RBAC Best Practices**
* **Principle of least privilege**: Grant minimum required permissions
* **Use Groups, not individual users**: Bind roles to groups (mapped from AWS SSO)
* **Namespace isolation**: Separate namespaces per environment and service
* **Regular audits**: Review and remove unused permissions
* **Pod Security Standards**: Enforce baseline or restricted profiles

**Example RBAC Mapping**
```yaml
# AWS SSO Group → K8s Group → K8s Role
PlatformEngineers (AWS SSO)
  → platform-engineers (K8s Group via aws-auth)
    → ClusterRoleBinding: platform-admin

Developers (AWS SSO)
  → developers (K8s Group via aws-auth)
    → RoleBinding (per namespace): namespace-developer

Service: payment-service
  → ServiceAccount: payment-service-sa (in payment-ns)
    → RoleBinding: app-runtime (minimal permissions)
    → IRSA: IAM role for DynamoDB, S3 access
```

**Cost**: Native K8s RBAC is **FREE** ✅

### Static Analysis and Security Scanning

#### Infrastructure-as-Code Security

**Terraform Static Analysis**

* **TFSec** (free, open source) - **RECOMMENDED**
  * Checks Terraform code for security misconfigurations
  * 1000+ built-in rules for AWS, Azure, GCP
  * Can run locally and in CI/CD
  * JSON output for automation
  * **Cost**: FREE ✅
  * GitHub Action available: `aquasecurity/tfsec-action`

* **Checkov** (free, open source) - **RECOMMENDED**
  * More comprehensive than TFSec
  * Supports Terraform, CloudFormation, Kubernetes manifests, Dockerfiles
  * 1000+ policies across multiple frameworks
  * Can enforce custom policies
  * **Cost**: FREE ✅
  * Also available as GitHub Action: `bridgecrewio/checkov-action`

* **Terrascan** (free, open source)
  * Alternative to above, good for compliance checking
  * 500+ policies across multiple IaC types
  * **Cost**: FREE ✅

* **tflint** (free, open source)
  * Linter for Terraform
  * Finds errors and enforces best practices
  * Plugin system for cloud-specific rules
  * **Cost**: FREE ✅

**Terraform Workflow**
```
Developer → git push → GitHub Actions
  ├── terraform fmt -check (formatting)
  ├── terraform validate (syntax)
  ├── tflint (linting)
  ├── tfsec (security scanning)
  ├── checkov (compliance checking)
  └── terraform plan → Manual approval → terraform apply
```

**Recommended: Use both TFSec and Checkov** (complementary coverage)

#### Application Code Security

**GitHub Advanced Security (GHAS)**
* **For public repositories**: **FREE** ✅
* **For private repositories**: $49/committer/month (expensive for personal use)
* Features:
  * CodeQL: Semantic code analysis
  * Secret scanning: Detects committed credentials
  * Dependency scanning: Vulnerable dependencies (Dependabot)

**Cost-Effective Alternatives for Private Repos**

**SonarCloud** - **RECOMMENDED**
* **Free for public repositories**
* **Free for private projects** up to 100k lines of code
* Features:
  * Code quality and security issues
  * 30+ programming languages
  * Technical debt tracking
  * Pull request decoration
* **Cost**: FREE for your use case ✅
* GitHub App integration available

**Snyk** - **RECOMMENDED for Dependencies**
* **Free tier**: Unlimited tests for open source
* **Free tier private**: Up to 200 tests/month
* Features:
  * Dependency vulnerability scanning
  * Container image scanning
  * Infrastructure-as-Code scanning
  * Fix recommendations and PRs
* **Cost**: FREE tier sufficient ✅
* GitHub integration available

**Semgrep** (free, open source)
* Static analysis with custom rules
* Fast, language-agnostic
* 2000+ community rules
* **Cost**: FREE ✅
* Cloud version has free tier: 10 contributors

**Trivy** (free, open source) - **RECOMMENDED**
* Comprehensive security scanner:
  * Container images
  * Filesystem
  * Git repositories
  * Kubernetes manifests
  * IaC (Terraform, CloudFormation)
* **Cost**: FREE ✅
* Easy to integrate in CI/CD
* GitHub Action: `aquasecurity/trivy-action`

**Grype** (free, open source)
* Vulnerability scanner for container images
* Alternative to Trivy
* **Cost**: FREE ✅

**Recommended Application Security Stack** (All FREE)
```
Language-specific:
├── SonarCloud: Code quality & security (SAST)
├── Semgrep: Custom rules, fast scanning

Dependencies:
├── Snyk: Vulnerability scanning, fix PRs
├── Dependabot: Automated dependency updates (GitHub native, free)

Containers:
├── Trivy: Image scanning, CVE detection
└── Docker Scout: Image analysis (free tier)

Runtime:
└── Falco: Runtime security in K8s (free, CNCF)
```

#### CI/CD Security Integration

**GitHub Actions Security Workflow**
```yaml
on: [push, pull_request]
jobs:
  security:
    - name: Run TFSec
      uses: aquasecurity/tfsec-action@v1
    
    - name: Run Checkov
      uses: bridgecrewio/checkov-action@v12
    
    - name: Run Trivy
      uses: aquasecurity/trivy-action@master
    
    - name: SonarCloud Scan
      uses: SonarSource/sonarcloud-github-action@master
    
    - name: Snyk Test
      uses: snyk/actions@master
```

**Cost**: All tools have free GitHub Actions minutes (2000/month on free plan)

#### Container Security

**Image Scanning**
* **Trivy** in CI/CD (pre-deployment)
* **Scan on push to ECR**: AWS ECR has basic scanning (free tier)
* **Enhanced scanning**: ECR Inspector ($0.09/image scan) - use sparingly

**Runtime Security**
* **Falco** (free, CNCF project)
  * Runtime threat detection in K8s
  * Detects anomalous behavior
  * Rules for common attacks
  * **Cost**: FREE ✅

**Image Signing and Verification**
* **Sigstore/Cosign** (free, open source)
  * Sign container images
  * Verify signatures before deployment
  * **Cost**: FREE ✅
* **Kyverno** can enforce signature verification policies

### Policy-as-Code

#### Kubernetes Policy Enforcement

**Kyverno** (free, CNCF project) - **RECOMMENDED**
* Policy engine for Kubernetes
* Validate, mutate, generate resources
* Written in YAML (no new language to learn)
* Examples:
  * Require resource limits on all pods
  * Enforce image pull policies
  * Require labels for ownership tracking
  * Block privileged containers
  * Generate network policies automatically
* **Cost**: FREE ✅

**Kyverno vs. OPA/Gatekeeper**
* Kyverno: Easier to learn, YAML-based
* OPA: More powerful, requires Rego language
* Recommendation: Start with Kyverno

**Policy Categories**
* **Pod Security**: Restrict privileged containers, host access
* **Resource Management**: Require limits/requests
* **Networking**: Enforce network policies
* **RBAC**: Validate role bindings
* **Supply Chain**: Verify image signatures, allowed registries
* **Compliance**: Enforce organizational standards

#### Infrastructure Policy-as-Code

**OPA (Open Policy Agent)** for Terraform
* **Conftest** (free, uses OPA)
  * Write policies for Terraform plans
  * Validate before apply
  * **Cost**: FREE ✅

**Terraform Sentinel**
* HashiCorp's policy framework
* Requires Terraform Cloud/Enterprise
* **Cost**: Not free (not recommended for personal use)

**Recommended: Use Checkov + custom policies**
* Checkov supports custom policies
* Python-based, easier than Rego
* Good for personal use

#### Policy Best Practices
* **Start with audit mode**: Log violations before enforcing
* **Gradual rollout**: Enable policies per namespace
* **Document exceptions**: When and why policies are bypassed
* **Regular review**: Update policies as threats evolve
* **GitOps**: Store policies in Git, version controlled

### Security Monitoring and Compliance

#### Audit Logging
* **AWS CloudTrail**: API call logging (free tier: 90-day history)
* **K8s Audit Logs**: Enable in EKS (sent to CloudWatch Logs)
  * Configure retention to minimize costs (7-30 days)
  * Export to S3 for long-term storage (cheaper)

#### Compliance Scanning
* **AWS Security Hub** (not free, ~$0.001/check)
  * Use selectively or disable for personal accounts
* **kube-bench** (free, CNCF)
  * CIS Kubernetes Benchmark checks
  * Run periodically to validate cluster security
  * **Cost**: FREE ✅

#### Vulnerability Management
* Track vulnerabilities in dashboards
* Prioritize by CVSS score and exploitability
* Automated patching where possible (Dependabot PRs)

### Estimated Monthly Security Costs

**Minimal Configuration (Recommended Start)**
* AWS Secrets Manager: $2-8 (5-20 secrets)
* AWS IAM Identity Center: FREE
* Static analysis tools: FREE (all open source)
* Kyverno: FREE (runs in cluster)
* CloudTrail: FREE (basic tier)
* **Total: $2-8/month** ✅

**If Scaling to HashiCorp Vault**
* Self-hosted Vault: $10-20/month (compute only)
* Worth it if: 50+ secrets OR want dynamic secrets
* **Total: $10-20/month**

## Resource Plane

* AWS is the cloud provider of choice
* Opt to use the relevant AWS resources whereever possible

### Compute

* Compute leverages pods in the AWS EKS cluster
* Avoid any AWS compute resources outside of the EKS cluster

### Data

* [Neno](https://neon.com) used for any databases
* Want to leverage the branching capabilities to support each environment

### Networking

* In AWS EKS want to leverage service mesh for networking
* Take the simplest apporach as possible for networking
* Want to explore the possibility to use Tailscale
* Tailscale could combine on-premise homelab K8s with AWS EKS

### Services

* Want any software-as-a-service services use to be managed/provisioned via IaC approach
* This could utilise Terraform providers if part of the foundation/environments
* This could utilise Crossplane if best managed within the K8s cluster