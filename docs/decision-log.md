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

## ADR-004: Deployment Safety Controls

**Date**: July 2025  
**Status**: ✅ Implementation Complete  
**Authors**: Microsoft CSA

### Context

CI/CD pipeline required safeguards to prevent accidental cross-environment deployments while maintaining deployment flexibility for different environments.

### Options Considered

| Approach                                        | Safety Level | Flexibility | Implementation   |
| ----------------------------------------------- | ------------ | ----------- | ---------------- |
| **Hard-coded Subscription IDs**           | High         | Low         | Simple but rigid |
| **Environment Variables**                 | Medium       | High        | Good balance     |
| **GitHub Actions + Parameter Validation** | High         | High        | Best practice    |

### Decision

**Selected: GitHub Actions Environment Protection with Parameter Validation**

**Implementation**:

```yaml
# .github/workflows/apim-ci.yml
jobs:
  validate-target:
    runs-on: ubuntu-latest
    steps:
      - name: Validate Deployment Target
        run: |
          EXPECTED_SUB=$(jq -r '.parameters.subscriptionId.value' infra/params/${{ github.ref_name }}.json)
          if [ "$EXPECTED_SUB" != "${{ secrets.AZURE_SUBSCRIPTION_ID }}" ]; then
            echo "❌ Subscription mismatch detected"
            exit 1
          fi
```

**Parameter File Structure**:

```json
// infra/params/devon-prod.json
{
  "subscriptionId": {"value": "devon-production-subscription-id"},
  "environment": {"value": "production"},
  "resourceGroup": {"value": "rg-apim-devon-prod-ncus-001"}
}
```

**Rationale**:

- **GitHub Best Practice**: Leverages GitHub Environments for approval workflows
- **Parameter Validation**: Cross-checks target subscription with environment configuration
- **Microsoft Guidance**: Aligns with Azure DevOps deployment safety recommendations
- **Audit Trail**: Complete deployment history in GitHub Actions logs

### Implications

- **Safety**: Prevents accidental production deployments through multiple validation layers
- **Governance**: Supports approval gates for sensitive environments
- **Scalability**: Easy to extend for Devon's multiple environments (dev/test/staging/prod)

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

*Architecture Decision Log for APIM Landing Zone Accelerator - sandbox phase recommendations for Devon Energy review*
