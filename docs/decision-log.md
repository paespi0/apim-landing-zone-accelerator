# APIM Landing Zone Accelerator - Architecture Decision Log

*Key technical decisions made during APIOps migration and landing zone implementation*

> 📝 **Important Note for Devon Reviewers**
> This architecture decision log was created during the sandbox phase by the Microsoft CSA. These reflect technical implementations and recommendations that Devon's architecture, security, and operations teams should review, validate, or adapt before production rollout.

## Decision Record Format

Each decision includes:

- **Context**: Why the decision was needed
- **Options**: Alternatives considered
- **Recommended Decision**: Chosen approach and rationale
- **Implications**: Trade-offs and consequences

---

## ADR-001: Sandbox vs Production Tenant Strategy

**Date**: July 2025  
**Status**: ✅ Implementation Complete  
**Authors**: Microsoft CSA

### Context

The sandbox implementation required tenant selection for development and testing, while establishing clear guidance for Devon's production deployment approach.

### Options Considered

| Option                           | Use Case              | Benefits                          | Considerations              |
| -------------------------------- | --------------------- | --------------------------------- | --------------------------- |
| **Microsoft Tenant**       | Sandbox development   | Rapid prototyping, CSA access     | Not suitable for production |
| **Devon Corporate Tenant** | Production deployment | Enterprise governance, compliance | Requires internal setup     |

### Decision

**Sandbox**: Microsoft Tenant (implemented)
**Production**: Devon Corporate Tenant (recommended)

**Implementation Context**:

- Microsoft CSA used Microsoft tenant for sandbox to accelerate proof-of-concept
- Production deployment must use Devon's corporate tenant for governance and compliance
- No architectural decision needed for production - enterprise policy requirement

**Rationale**:

- **Sandbox**: Enabled rapid implementation and Microsoft CSA collaboration
- **Production**: Devon's corporate governance, existing identity management, cost allocation
- **Compliance**: Aligns with Devon's data residency and audit requirements

### Implications

- **Positive**: Clear separation of concerns, accelerated sandbox development
- **Production Requirements**: Devon must configure Azure AD app registrations and OIDC federation
- **Migration Path**: Artifacts and configurations portable between tenants

---

## ADR-002: OIDC Federation Implementation

**Date**: July 2025  
**Status**: ✅ Implementation Complete  
**Authors**: Microsoft CSA

### Context

GitHub Actions CI/CD pipeline required secure authentication to Azure resources, following Microsoft security best practices for DevOps workflows.

### Options Considered

| Approach                            | Security | Maintenance            | Implementation       |
| ----------------------------------- | -------- | ---------------------- | -------------------- |
| **Service Principal Secrets** | Medium   | High rotation overhead | Simple setup         |
| **OIDC Federation**           | High     | Zero secret management | Modern best practice |

### Decision

**Selected: OIDC Federation with Azure AD Federated Credentials**

**Implementation Details**:

```yaml
# Azure AD App Registration Configuration
federated_credential:
  issuer: "https://token.actions.githubusercontent.com"
  subject: "repo:devon-org/apim-accelerator:ref:refs/heads/main"
  audiences: ["api://AzureADTokenExchange"]
```

**GitHub Actions Configuration**:

```yaml
permissions:
  id-token: write
  contents: read

- name: Azure Login
  uses: azure/login@v1
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

**Rationale**:

- **Security**: Short-lived tokens, no secret storage, built-in Azure AD integration
- **Microsoft Guidance**: Recommended approach per Azure DevOps security documentation
- **Industry Standard**: GitHub and Microsoft joint best practice
- **Auditability**: Full token lineage and Azure AD audit logs

### Implications

- **Security Enhancement**: Eliminates secret rotation and GitHub secret management
- **Setup Requirement**: Devon must configure Azure AD app registrations per environment
- **Operational Simplicity**: Zero ongoing secret management overhead

---

## ADR-003: Regional Deployment Strategy

**Date**: July 2025  
**Status**: ✅ Implementation Complete  
**Authors**: Microsoft CSA

### Context

APIM service deployment required region selection based on Devon's existing infrastructure footprint and operational requirements.

### Analysis of Devon's Current Deployment

**Discovered Configuration**:

- **Current Region**: North Central US (`northcentralus`)
- **APIM Service**: `apiservicejx6p2m5bi3lky`
- **DNS Resolution**: `*.northcentralus.cloudapp.azure.com`
- **Custom Domains**: `apim.pre.dvn.com`, `api.pre.dvn.com`

### Options Considered

| Region                     | Proximity to Devon | Cost     | Feature Availability | Current Usage |
| -------------------------- | ------------------ | -------- | -------------------- | ------------- |
| **North Central US** | Good (Chicago)     | Standard | Full APIM + OpenAI   | ✅ In use     |
| **South Central US** | Excellent (Dallas) | Standard | Full APIM + OpenAI   | Alternative   |
| **East US 2**        | Good (Virginia)    | Premium  | Full features        | Sandbox only  |

### Decision

**Validated: North Central US (existing deployment)**

**Implementation Evidence**:

```bash
# DNS resolution confirms regional deployment
nslookup apiservicejx6p2m5bi3lky.azure-api.net
# Returns: *.northcentralus.cloudapp.azure.com
```

**Rationale**:

- **Operational Reality**: Devon already deployed in North Central US
- **Geographic Fit**: Chicago region provides good coverage for Devon's Texas operations
- **Feature Support**: All required APIM and Azure OpenAI policies available
- **Performance**: Adequate latency for Devon's user base and backend services

### Implications

- **Architectural Validation**: Current deployment is well-architected for Devon's needs
- **Future Planning**: North Central US suitable for production scaling
- **Multi-region Consideration**: South Central US remains viable option for disaster recovery

---

## ADR-004: CI/CD Safety Gates and Subscription Guard

**Date**: July 2025  
**Status**: ✅ Implementation Complete  
**Authors**: Microsoft CSA

### Context

APIOps CI/CD pipeline required comprehensive safety controls to prevent security violations and accidental deployments. Two critical safety mechanisms needed implementation: subscription-level artifact protection and environment-based deployment controls.

### Options Considered

| Safety Control                                    | Security Level | Scope                | Implementation Complexity |
| ------------------------------------------------- | -------------- | -------------------- | ------------------------- |
| **Manual Code Reviews Only**                | Low            | Human-dependent      | Simple but unreliable     |
| **Basic Branch Protection**                 | Medium         | Repository-level     | Good but insufficient     |
| **CI-based Security Gates + Environment Controls** | High           | Multi-layered        | Best practice (selected)  |

### Decision

**Selected: Multi-layered CI/CD Safety Gates with Automated Subscription Guard**

**Implementation Overview**:

1. **Subscription Artifact Guard** (`.github/workflows/apim-ci.yml` - security job)
2. **GitHub Environment Protection** (branch-scoped deployments)
3. **Automated Security Scanning** (credential detection)

**Subscription Guard Implementation**:

```yaml
# .github/workflows/apim-ci.yml - Security Job
security:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    
    - name: Check for prohibited subscription artifacts
      run: |
        if [ -d "artifacts/subscriptions" ]; then
          echo "❌ SECURITY VIOLATION: Subscription artifacts detected!"
          echo "The artifacts/subscriptions/ folder contains sensitive subscription-level configurations."
          echo "This data should never be committed to version control for security reasons."
          find artifacts/subscriptions -type f
          echo "Please remove these files and add artifacts/subscriptions/ to .gitignore"
          exit 1
        else
          echo "✅ Security check passed: No prohibited subscription artifacts found"
        fi
```

**What the Subscription Guard Protects**:
- **Subscription-level configurations**: Network policies, access controls, billing information
- **Tenant-specific settings**: Identity providers, custom domains, security configurations  
- **Cross-tenant data**: Prevents accidental exposure of Devon's subscription details

**How It Works**:
1. **PR Check**: Runs on every pull request to `import` branch
2. **Automated Scanning**: Detects `artifacts/subscriptions/` folder presence
3. **Fail-Fast**: Blocks merge if subscription artifacts are detected
4. **Clear Guidance**: Provides specific remediation steps to developers

**GitHub Environment Controls**:

```yaml
# Environment-specific deployment scoping
on:
  push:
    branches: [import]  # Only deploys from import branch
  pull_request:
    branches: [import]  # Only validates against import branch
```

**Additional Security Layers**:

```yaml
- name: Check for sensitive named values
  run: |
    if find artifacts -name "*credential*" -o -name "*secret*" -o -name "*key*" | grep -q .; then
      echo "❌ SECURITY VIOLATION: Potential credential files detected!"
      find artifacts -name "*credential*" -o -name "*secret*" -o -name "*key*"
      exit 1
```

**Rationale**:

- **Microsoft Security**: Aligns with Azure DevOps security best practices and GitOps principles
- **Defense in Depth**: Multiple validation layers prevent different types of security violations
- **APIOps Standard**: Follows Microsoft's official APIOps security guidance for artifact management
- **Compliance Ready**: Provides audit trail and prevents accidental credential exposure
- **Developer Experience**: Clear error messages guide developers to correct security violations

### Implications

- **Security Posture**: Prevents subscription-level data leakage and credential exposure
- **Compliance**: Automated gates support SOX and regulatory audit requirements  
- **Operational Safety**: Branch protection ensures only validated changes reach target environments
- **Scalability**: Pattern extends to Devon's full environment hierarchy (dev/test/staging/prod)
- **Audit Trail**: Complete security validation history preserved in GitHub Actions logs

---

## ADR-005: API Extraction Format (JSON vs YAML)

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Authors**: Microsoft CSA

### Context

APIOps extractor tool supported multiple formats for OpenAPI specifications, requiring standardization for consistency and tooling compatibility within Devon's development workflow.

### Options Considered

| Format         | Readability | Tool Support | Version Control | File Size |
| -------------- | ----------- | ------------ | --------------- | --------- |
| **JSON** | Low         | Excellent    | Poor diffs      | Compact   |
| **YAML** | High        | Good         | Clean diffs     | Verbose   |

### Recommended Decision (to be reviewed by Devon)

**Selected: YAML for OpenAPI Specifications**

**Configuration**:

```yaml
# configuration.extractor.yaml
openApiSpecFormat: "yaml"
preserveComments: true
indentSize: 2
```

**Rationale**:

- **Human Readability**: YAML more accessible for Devon's API documentation review
- **Version Control**: Clean, meaningful diffs for change tracking
- **Industry Standard**: OpenAPI community preference for YAML
- **Tooling**: Spectral and other linting tools optimized for YAML

### Implications

- **Positive**: Better readability, cleaner version control, industry alignment
- **Negative**: Larger file sizes, potential YAML formatting issues
- **Mitigations**: Automated validation, standardized formatting rules

---

## ADR-006: Test Strategy for APIOps Artifacts

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Authors**: Microsoft CSA

### Context

APIOps pipeline required comprehensive testing strategy for Devon's 24 APIs and 8 distinct artifact types while maintaining security boundaries for Devon's APIM deployment.

### Analysis Results

**Devon's APIM Deployment Analysis**: 24 APIs successfully extracted and analyzed across 8 artifact types

| Category                     | Artifact Types                                | Coverage by Type | Security Impact |
| ---------------------------- | --------------------------------------------- | ---------------- | --------------- |
| **Fully Testable**     | APIs, Global Policies, Groups, Tags           | 4/8 types (50%)  | Low risk        |
| **Partially Testable** | Products                                      | 1/8 types (12%)  | Medium risk     |
| **Security Excluded**  | Named Values, Loggers, Diagnostics           | 3/8 types (38%)  | High risk       |

**API-Level Coverage**: All 24 APIs from Devon's deployment have OpenAPI specifications that can be validated with Spectral linting and schema validation.

**Artifact Types Identified**:
- **APIs (24)**: OpenAPI specifications for business services
- **Named Values (29)**: API keys, credentials, configuration values  
- **Products (8)**: Subscription tiers and access control
- **Groups (3)**: User access categories (administrators, developers, guests)
- **Loggers (2)**: Application Insights and Azure Monitor configurations
- **Diagnostics (2)**: Monitoring and telemetry configurations  
- **Global Policies (1)**: CORS and authentication baseline
- **Tags (Various)**: API categorization and metadata

### Recommended Decision (to be reviewed by Devon)

**Selected: Multi-layered Testing with Security Exclusions**

**Testing Framework**:

```yaml
# .github/workflows/apim-ci.yml
testing:
  spectral_validation: true    # OpenAPI specs (24 APIs)
  xml_validation: true        # Global policy documents
  json_schema: true           # Groups, products, tags configuration
  security_scan: false        # Excluded: named values, loggers, diagnostics
```

**Rationale**:

- **Security First**: Named values, loggers, and diagnostics excluded from version control
- **Quality Gates**: Spectral linting validates all 24 API specifications from Devon's deployment  
- **Focused Coverage**: 50% of artifact types fully testable, with clear security boundaries
- **Risk Mitigation**: 3-phase improvement plan for enhanced validation

### Test Gap Closure Plan

1. **Phase 1**: Implement JSON schema validation for products and groups (Month 1)
2. **Phase 2**: Add enhanced policy validation and security scanning (Month 2)
3. **Phase 3**: Establish secure configuration management for excluded artifacts (Month 3)

### Implications

- **Positive**: Strong quality gates, security compliance, systematic improvement
- **Negative**: Initial coverage gaps, complex testing matrix
- **Mitigations**: Phased rollout, comprehensive documentation, regular reviews

---

## ADR-007: Artifact Storage and Version Control Strategy

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Authors**: Microsoft CSA

### Context

APIOps artifacts required version control strategy balancing transparency, security, and operational efficiency for Devon's APIM deployment.

### Artifact Classification

| Classification         | Storage Strategy               | Examples                             |
| ---------------------- | ------------------------------ | ------------------------------------ |
| **Public**       | Full version control           | OpenAPI specs, group definitions     |
| **Internal**     | Version controlled with review | Product policies, API configurations |
| **Confidential** | Excluded via .gitignore        | Named values, logger credentials     |

### Recommended Decision (to be reviewed by Devon)

**Selected: Selective Version Control with Security Exclusions**

**Implementation**:

```gitignore
# Security exclusions
artifacts/named\ values/
artifacts/loggers/
artifacts/**/*credential*
artifacts/**/*secret*
```

**Rationale**:

- **Security**: Sensitive data never enters version control
- **Transparency**: Configuration changes fully auditable
- **Compliance**: Meets Devon's corporate data classification requirements
- **Operational**: Automated extraction without manual filtering

### Implications

- **Positive**: Strong security, full audit trail for non-sensitive changes
- **Negative**: Limited visibility into credential configurations
- **Mitigations**: Separate credential management process, security reviews

---

## Future Decisions for Devon Review

### Pending Architecture Reviews

| Decision                               | Timeline | Priority | Devon Owner              |
| -------------------------------------- | -------- | -------- | ------------------------ |
| **Multi-region Deployment**      | Q3 2025  | Medium   | Devon Platform Team      |
| **Production Environment Setup** | Q4 2025  | High     | Devon DevOps Team        |
| **API Versioning Strategy**      | Q2 2025  | Medium   | Devon API Standards Team |
| **Monitoring and Alerting**      | Q2 2025  | High     | Devon Operations Team    |

### Decision Review Schedule for Devon

- **Initial Review**: Before production deployment
- **Quarterly Reviews**: Every 3 months after production
- **Annual Assessment**: Full architecture review with Microsoft CSA
- **Change Triggers**: Major feature releases, security incidents, compliance updates

---

## References

### Documentation

- [APIOps Best Practices](https://azure.github.io/apiops/)
- [Azure APIM Documentation](https://docs.microsoft.com/en-us/azure/api-management/)
- [OIDC Federation Guide](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

### Related Decisions

- See `docs/test-gap-closure-plan.md` for testing strategy details
- See `docs/runbook.md` for operational procedures
- See `docs/artifact-testability-analysis.md` for technical analysis

---

## ADR-008: Environment-Specific Parameter Files Strategy

**Date**: August 2025  
**Status**: 🔄 Implementation Required  
**Authors**: Microsoft CSA

### Context

Devon Energy's APIM Landing Zone Accelerator requires a scalable approach to manage infrastructure parameters across multiple environments (development, test, staging, production). The current sandbox implementation uses a single `sandbox.json` parameter file, which is insufficient for enterprise multi-environment deployment patterns.

**Key Requirements**:
- Separate Azure subscriptions for environment isolation
- Environment-specific resource naming and configuration
- Prevention of accidental cross-environment deployments
- Support for GitOps promotion workflows
- Alignment with Microsoft Well-Architected Framework principles

### Options Considered

| Approach | Benefits | Considerations |
|----------|----------|----------------|
| **Single Parameter File** | Simple, current state | No environment isolation, high risk |
| **Environment Variables** | Flexible, CI/CD friendly | Secrets exposure risk, difficult audit |
| **Environment-Specific Files** | Clear separation, auditable | Multiple files to maintain |
| **Dynamic Parameter Generation** | Flexible, template-based | Complex, difficult to validate |

### Recommended Decision

**Selected: Environment-Specific Parameter Files with Structured Naming Convention**

**Implementation Pattern**:

```
infra/params/
├── devon-dev.json      # Development environment
├── devon-test.json     # Testing environment  
├── devon-staging.json  # Staging environment
└── devon-prod.json     # Production environment
```

**File Structure Template**:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": { "value": "southcentralus" },
    "workloadName": { "value": "devon" },
    "environment": { "value": "dev|test|staging|prod" },
    "identifier": { "value": "scus" },
    
    "subscriptionId": { "value": "environment-specific-subscription-guid" },
    "appGatewayFqdn": { "value": "api-{env}.devon.energy" },
    "appGatewayCertType": { "value": "KeyVault|SelfSigned" },
    
    "monitoring": {
      "value": {
        "level": "basic|standard|enhanced",
        "retentionDays": 7|14|30|90
      }
    },
    "security": {
      "value": {
        "requireHttps": true,
        "allowedOrigins": ["https://devon-{env}.local"]
      }
    },
    "backup": {
      "value": {
        "enabled": false|true,
        "frequency": "none|daily|weekly",
        "retentionDays": 0|30|90
      }
    }
  }
}
```

**Environment-Specific Examples**:

**Development Environment** (`devon-dev.json`):
```json
{
  "parameters": {
    "environment": { "value": "dev" },
    "subscriptionId": { "value": "11111111-1111-1111-1111-111111111111" },
    "tier": { "value": "Developer" },
    "monitoring": { 
      "value": { "level": "basic", "retentionDays": 7 }
    },
    "backup": { 
      "value": { "enabled": false }
    }
  }
}
```

**Production Environment** (`devon-prod.json`):
```json
{
  "parameters": {
    "environment": { "value": "prod" },
    "subscriptionId": { "value": "44444444-4444-4444-4444-444444444444" },
    "tier": { "value": "Premium" },
    "monitoring": { 
      "value": { "level": "enhanced", "retentionDays": 90 }
    },
    "backup": { 
      "value": { "enabled": true, "frequency": "daily", "retentionDays": 30 }
    }
  }
}
```

### GitHub Actions Integration

**Workflow Enhancement** (reference: `.github/workflows/apim-ci.yml`):

```yaml
- name: Load Environment Parameters
  id: params
  run: |
    ENV_FILE="infra/params/devon-${{ inputs.environment }}.json"
    if [ ! -f "$ENV_FILE" ]; then
      echo "❌ Parameter file not found: $ENV_FILE"
      exit 1
    fi
    
    echo "apim_name=apim-$(jq -r '.parameters.workloadName.value + "-" + .parameters.environment.value + "-" + .parameters.location.value + "-" + .parameters.identifier.value' $ENV_FILE)" >> $GITHUB_OUTPUT
    echo "resource_group=rg-apim-$(jq -r '.parameters.workloadName.value + "-" + .parameters.environment.value + "-" + .parameters.location.value + "-" + .parameters.identifier.value' $ENV_FILE)" >> $GITHUB_OUTPUT
    echo "subscription_id=$(jq -r '.parameters.subscriptionId.value' $ENV_FILE)" >> $GITHUB_OUTPUT

- name: Validate Subscription Guard
  run: |
    EXPECTED_SUB="${{ steps.params.outputs.subscription_id }}"
    CURRENT_SUB=$(az account show --query id -o tsv)
    if [ "$EXPECTED_SUB" != "$CURRENT_SUB" ]; then
      echo "❌ SECURITY VIOLATION: Subscription mismatch!"
      echo "Expected: $EXPECTED_SUB"
      echo "Current:  $CURRENT_SUB"
      exit 1
    fi
```

### Security Implications

**Cross-Environment Protection**:
- **Subscription isolation**: Each environment targets different Azure subscription
- **Parameter validation**: CI/CD validates correct parameter file usage
- **Subscription guards**: Automated checks prevent cross-environment deployment
- **OIDC scoping**: Environment-specific Azure AD app registrations

**Audit and Compliance**:
- **Version controlled**: All parameter changes tracked in Git history
- **Code review required**: Parameter modifications require PR approval
- **Environment-specific approval**: Production parameters require security team review
- **Immutable deployments**: Parameters locked during deployment process

### Implementation Requirements

**Prerequisites for Devon**:
1. **Azure Subscriptions**: Separate subscriptions for dev/test/staging/prod
2. **Naming Convention**: Standardized resource naming across environments
3. **OIDC Federation**: Environment-specific Azure AD app registrations
4. **GitHub Environments**: Environment protection rules and approval gates

**Migration from Sandbox**:
1. Create environment-specific parameter files based on `sandbox.json` template
2. Update CI/CD workflow to support environment detection and parameter selection
3. Configure environment-specific secrets and variables in GitHub
4. Test deployment and rollback procedures for each environment

### Consequences

**Positive Outcomes**:
- **Environment isolation**: Prevents accidental cross-environment deployments
- **Configuration clarity**: Explicit parameters for each environment
- **Audit trail**: All environment changes tracked in version control
- **Scalability**: Easy to add new environments or modify existing configurations
- **GitOps alignment**: Supports promotion-based deployment workflows

**Trade-offs**:
- **Maintenance overhead**: Multiple parameter files to maintain
- **Initial setup complexity**: Requires environment-specific configuration
- **File synchronization**: Common parameters need updates across multiple files

**Risk Mitigation**:
- **Template validation**: JSON schema validation for parameter files
- **Automated testing**: Parameter file validation in CI/CD pipeline
- **Documentation**: Clear examples and naming conventions
- **Code review**: All parameter changes require approval

### Implementation Notes

**File Location**: `infra/params/devon-{environment}.json`  
**Schema Validation**: Azure Resource Manager parameter schema  
**Integration Points**: 
- GitHub Actions workflow (`.github/workflows/apim-ci.yml`)
- APIOps publisher configuration
- Azure Bicep deployment templates
- Environment promotion strategy (`docs/promotion-strategy.md`)

**Related Configurations**:
- GitHub Environments with protection rules
- Azure AD federated identity credentials per environment
- Environment-specific monitoring and alerting configurations

### References

**Technical Documentation**:
- [Promotion Strategy](./promotion-strategy.md) - Multi-environment deployment approach
- [GitHub Actions Workflow](../.github/workflows/apim-ci.yml) - CI/CD implementation
- [Azure Resource Manager Parameters](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/parameters)

**Microsoft Guidance**:
- [Azure Well-Architected Framework - Operational Excellence](https://docs.microsoft.com/en-us/azure/architecture/framework/devops/)
- [GitOps with Azure](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/gitops-aks/gitops-blueprint-aks)
- [Infrastructure as Code Best Practices](https://docs.microsoft.com/en-us/azure/architecture/framework/devops/iac)

---

### Related Decisions

- See `docs/test-gap-closure-plan.md` for testing strategy details
- See `docs/runbook.md` for operational procedures
- See `docs/artifact-testability-analysis.md` for technical analysis

---

*Architecture Decision Log for APIM Landing Zone Accelerator - sandbox phase recommendations for Devon Energy review*
