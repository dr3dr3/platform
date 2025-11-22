# ADR-001: Terraform Deployment Workflow - GitHub Actions vs Terraform Cloud VCS Integration

## Status
Proposed

## Date
2025-11-20

## Context

We are establishing a Terraform repository to manage AWS infrastructure foundations using an Infrastructure-as-Code (IaC) approach. The infrastructure will include:
- AWS EKS clusters
- IAM roles and permissions
- Supporting AWS services and resources

We will use Terraform Cloud for state management and remote execution. However, we need to decide on the orchestration layer that triggers and manages the deployment pipeline.

### Two Primary Options

1. **Terraform Cloud Native VCS Integration**: Direct repository connection with automatic plan/apply triggers
2. **GitHub Actions Orchestration**: Custom workflow that controls the pipeline and uses Terraform Cloud as a backend

### Current Platform Context

Our existing platform architecture (as documented in `platform-reference-architecture-v2.md` and `best-practices.md`) includes:
- **CI/CD**: GitHub Actions as the standard pipeline platform
- **Version Control**: GitHub for all repositories
- **Service Catalog**: Port.io for unified platform metadata and observability
- **Security**: Emphasis on scanning (Trivy, Snyk, SonarCloud, tfsec, checkov)
- **Developer Experience**: Unified developer portal with single pane of glass approach

### Best Practices Requirements

From `best-practices.md`, our desired Terraform workflow includes:
```
Developer → git push → GitHub Actions
  ├── terraform fmt -check (formatting)
  ├── tfsec (security scanning)
  ├── checkov (compliance checks)
  ├── terraform validate
  ├── terraform plan → Manual approval → terraform apply
  └── Update Port.io entities
```

## Decision

**We will use GitHub Actions to orchestrate the Terraform deployment workflow, with Terraform Cloud providing state management and remote execution.**

GitHub Actions will:
- Control the entire pipeline workflow
- Execute security scanning (tfsec, checkov)
- Perform Terraform validation and formatting checks
- Trigger Terraform Cloud operations via CLI/API
- Update Port.io service catalog entities post-deployment
- Manage notifications and integrations

Terraform Cloud will:
- Manage Terraform state
- Provide remote execution environment
- Handle state locking
- Store sensitive variables
- Provide run history and audit logs

## Rationale

### Alignment with Platform Principles

1. **Unified Developer Experience**
   - All CI/CD pipelines (application and infrastructure) visible in GitHub
   - Consistent workflow patterns across the platform
   - Single source of truth for pipeline execution history

2. **Service Catalog Integration**
   - Seamless updates to Port.io entities after infrastructure changes
   - Infrastructure components tracked alongside services and deployments
   - Better visibility into infrastructure state in the developer portal

3. **Security-First Approach**
   - Native integration with security scanning tools (tfsec, checkov)
   - Ability to fail pipelines on security violations
   - Consistent security posture across all pipelines
   - Custom policy enforcement before Terraform execution

4. **Flexibility and Extensibility**
   - Easy to add custom validation steps (terratest, policy checks)
   - Support for custom notifications (Slack, Teams, Port.io)
   - Ability to integrate with other platform tools
   - Multi-cloud ready if we expand beyond AWS

### Specific Advantages

**Control and Customization**
- Complete control over workflow steps and ordering
- Ability to add pre-flight and post-deployment actions
- Custom approval gates using GitHub Environments
- Integration with third-party tools and services

**Developer Experience**
- Developers already familiar with GitHub Actions
- PR comments with plan output for review
- Consistent interface across all repository types
- GitHub native approval workflows

**Observability and Debugging**
- Detailed logs in GitHub Actions UI
- Easy correlation between code changes and infrastructure deployments
- Integration with existing monitoring and alerting

**Cost Optimization**
- GitHub Actions minutes included in current plan
- Terraform Cloud free tier for state management
- No additional tooling costs

### Trade-offs Accepted

**Increased Complexity**
- More configuration required than native VCS integration
- Need to maintain GitHub Actions workflow definitions
- Two systems in the critical path (GitHub + Terraform Cloud)

**Additional Token Management**
- Terraform Cloud API tokens stored in GitHub Secrets
- Requires rotation and lifecycle management

**Potential Reliability**
- Dependency on both GitHub and Terraform Cloud availability
- Mitigation: Both are enterprise-grade SaaS platforms with high SLA

## Implementation

### Repository Structure
```
terraform-infrastructure/
├── .github/
│   └── workflows/
│       ├── terraform-plan.yml      # PR validation workflow
│       └── terraform-apply.yml     # Main branch deployment workflow
├── terraform/
│   ├── eks/                        # EKS cluster configuration
│   ├── iam/                        # IAM roles and policies
│   └── shared/                     # Shared resources
├── .tfsec/                         # tfsec configuration
├── .checkov.yml                    # Checkov configuration
└── README.md
```

### Workflow Pattern

**Pull Request Flow:**
1. Developer opens PR with infrastructure changes
2. GitHub Actions triggered automatically
3. Run formatting check (`terraform fmt -check`)
4. Run security scanning (tfsec, checkov)
5. Run Terraform validate
6. Run Terraform plan via Terraform Cloud
7. Post plan output as PR comment
8. Block merge if security issues found

**Deployment Flow (main branch):**
1. PR merged to main branch
2. GitHub Actions triggered automatically
3. Re-run all validation and security checks
4. Execute Terraform plan
5. Wait for manual approval (GitHub Environment protection)
6. Execute Terraform apply via Terraform Cloud
7. Update Port.io entities with new infrastructure state
8. Send notifications (Slack, Port.io)

### Required Secrets

GitHub repository secrets:
- `TF_API_TOKEN`: Terraform Cloud API token
- `PORT_CLIENT_ID`: Port.io client ID
- `PORT_CLIENT_SECRET`: Port.io client secret
- `AWS_ACCESS_KEY_ID`: AWS credentials (if needed for validation)
- `AWS_SECRET_ACCESS_KEY`: AWS credentials (if needed for validation)

### Terraform Cloud Configuration

Workspace settings:
- **Execution Mode**: Remote
- **Auto Apply**: Disabled (controlled by GitHub Actions)
- **VCS Connection**: Optional (for visibility, not for triggers)
- **Terraform Version**: Locked to specific version

## Consequences

### Positive

✅ **Consistency**: Infrastructure pipelines follow same patterns as application pipelines  
✅ **Security**: Integrated security scanning before any infrastructure changes  
✅ **Visibility**: All deployments tracked in service catalog (Port.io)  
✅ **Flexibility**: Easy to add new tools, checks, or integrations  
✅ **Developer Experience**: Single platform (GitHub) for all workflows  
✅ **Auditability**: Complete deployment history in GitHub Actions logs  
✅ **Cost-Effective**: Uses existing GitHub Actions allocation  

### Negative

⚠️ **Maintenance Burden**: Need to maintain GitHub Actions workflow definitions  
⚠️ **Learning Curve**: Team needs GitHub Actions expertise for troubleshooting  
⚠️ **Complexity**: More moving parts than native Terraform Cloud VCS integration  
⚠️ **Potential Lock-in**: Heavily invested in GitHub Actions patterns  

### Neutral

ℹ️ **Migration Path**: Can switch to Terraform Cloud native VCS if requirements change  
ℹ️ **Hybrid Option**: Can use both approaches for different workspaces if needed  
ℹ️ **Future Flexibility**: Pattern supports multi-cloud (GCP, Azure) without changes  

## Alternatives Considered

### Alternative 1: Terraform Cloud Native VCS Integration

**Description**: Use Terraform Cloud's built-in VCS integration to automatically trigger runs on repository changes.

**Rejected Because**:
- Limited ability to integrate security scanning tools
- Difficult to update Port.io service catalog automatically
- Creates inconsistency with application deployment patterns
- Less flexibility for custom workflow steps
- Developers would need to context-switch between GitHub and Terraform Cloud UIs

**When to Reconsider**:
- If GitHub Actions becomes cost-prohibitive
- If Terraform Cloud adds native support for external security scanning
- If team size is very small (< 5 developers) and simplicity is paramount

### Alternative 2: Terraform Cloud with Sentinel Policies Only

**Description**: Use native VCS integration but rely on Sentinel policies for security and compliance.

**Rejected Because**:
- Sentinel is a paid Terraform Cloud feature
- Doesn't provide pre-commit feedback (runs after Terraform plan)
- Steeper learning curve (HCL-based policy language)
- Doesn't solve Port.io integration requirement
- Still creates UI fragmentation

### Alternative 3: Jenkins or GitLab CI

**Description**: Use a different CI/CD platform for Terraform workflows.

**Rejected Because**:
- Introduces additional platform to maintain
- Inconsistent with existing GitHub-centric workflow
- Additional cost and operational overhead
- Team already trained on GitHub Actions
- Would fragment developer experience

### Alternative 4: Atlantis (Self-Hosted)

**Description**: Deploy Atlantis for Terraform automation with PR-based workflows.

**Rejected Because**:
- Requires additional infrastructure to host and maintain
- Operational overhead for platform team
- Another service to monitor and secure
- GitHub Actions provides similar capabilities with less overhead
- Adds complexity without clear benefits for our scale

## Migration Strategy

### Phase 1: Initial Implementation (Week 1-2)
- Create GitHub Actions workflows for Terraform validation
- Configure Terraform Cloud workspaces with remote execution
- Set up security scanning (tfsec, checkov)
- Test with non-production workspaces

### Phase 2: Integration (Week 3-4)
- Add Port.io entity updates to workflows
- Configure GitHub Environments for production approvals
- Set up notifications (Slack, Port.io)
- Document workflow for development team

### Phase 3: Production Rollout (Week 5-6)
- Deploy EKS infrastructure using new workflow
- Monitor and refine approval processes
- Collect feedback from platform team
- Document lessons learned

### Rollback Plan
If GitHub Actions approach proves problematic:
1. Disable GitHub Actions workflows
2. Enable Terraform Cloud VCS integration
3. Manually update Port.io entities until automation is resolved
4. Reassess integration requirements

## References

- [Platform Reference Architecture v2](./platform-reference-architecture-v2.md)
- [Platform Best Practices - CI/CD](./best-practices.md#cicd-best-practices)
- [Terraform Cloud Documentation](https://developer.hashicorp.com/terraform/cloud-docs)
- [GitHub Actions for Terraform](https://github.com/hashicorp/setup-terraform)
- [tfsec Documentation](https://github.com/aquasecurity/tfsec)
- [Checkov Documentation](https://www.checkov.io/)

## Review and Update

This ADR should be reviewed:
- After 3 months of production usage
- If Terraform Cloud releases significant new features
- If GitHub Actions pricing or features change significantly
- If team size grows beyond 50 developers
- If multi-cloud requirements emerge

---

**Decision Makers**: Platform Engineering Team  
**Stakeholders**: DevOps Team, Infrastructure Team, Development Teams  
**Last Reviewed**: 2025-11-20  
**Next Review**: 2026-02-20
