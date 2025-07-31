# APIM Landing Zone Accelerator - Architecture Decision Log

*Key technical decisions made during APIOps migration and landing zone implementation*

> 📝 **Important Note for Devon Reviewers**  
> This architecture decision log was created during the sandbox phase by Microsoft CSA and engineering partners. These reflect early recommendations that Devon's architecture, security, and operations teams should review, ratify, or revise before production rollout.

## Decision Record Format

Each decision includes:
- **Context**: Why the decision was needed
- **Options**: Alternatives considered
- **Recommended Decision**: Chosen approach and rationale
- **Implications**: Trade-offs and consequences

---

## ADR-001: Devon vs Microsoft Tenant Selection

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
APIOps pipeline required selection between Devon corporate tenant and Microsoft tenant for Azure resource deployment and authentication.

### Options Considered

| Option | Pros | Cons |
|--------|------|------|
| **Devon Corporate Tenant** | Existing governance, cost allocation, compliance alignment | Potential scalability limits, internal dependencies |
| **Microsoft Tenant** | Azure native integration, tooling support, scalability | External dependency, compliance complexity |

### Recommended Decision (to be reviewed by Devon)
**Selected: Devon Corporate Tenant**

**Rationale**:
- Devon's corporate governance and compliance requirements
- Existing identity management and access controls
- Cost allocation aligned with Devon's corporate processes
- Security policies and audit requirements already in place

### Implications
- **Positive**: Simplified compliance, existing governance, cost transparency
- **Negative**: Potential scalability limitations, dependency on Devon corporate IT
- **Mitigations**: Established escalation paths, documented service level agreements

---

## ADR-002: OIDC vs Service Principal Authentication

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
GitHub Actions workflow required authentication method for Azure resource access, choosing between traditional service principal secrets and modern OIDC federation.

### Options Considered

| Option | Security | Maintenance | Scalability | Audit |
|--------|----------|-------------|-------------|-------|
| **Service Principal + Secrets** | Medium | High maintenance | Good | Complex |
| **OIDC Federation** | High | Low maintenance | Excellent | Simple |

### Recommended Decision (to be reviewed by Devon)
**Selected: OIDC Federation**

**Technical Implementation**:
```json
{
  "name": "gh-devon-apim-landing-zone-accelerator",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:<devon-github-org>/<devon-repo-name>:ref:refs/heads/<devon-branch>",
  "audiences": ["api://AzureADTokenExchange"]
}
```

**Rationale**:
- **Security**: No secrets stored in GitHub, tokens short-lived
- **Maintenance**: No secret rotation required
- **Auditability**: Clear token lineage and usage tracking
- **Modern**: Industry best practice for CI/CD authentication

### Implications
- **Positive**: Enhanced security, reduced operational overhead, better audit trail
- **Negative**: Newer technology, potential debugging complexity
- **Mitigations**: Comprehensive documentation, fallback procedures established

---

## ADR-003: Azure Region Selection

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
APIM service deployment required region selection, balancing cost, performance, compliance, and feature availability for Devon's workload.

### Options Considered

| Region | Cost Profile | Latency | Compliance | Feature Set |
|--------|-------------|---------|------------|-------------|
| **South Central US** | Standard pricing | Low to Devon users | SOC 2 Type II | Full APIM features |
| **East US 2** | Premium pricing | Medium latency | SOC 2 Type II | Full features |
| **Central US** | Budget pricing | Higher latency | SOC 2 Type II | Limited features |

### Recommended Decision (to be reviewed by Devon)
**Selected: South Central US (Devon's primary region)**

**Service Configuration Template**:
```yaml
Service Name: apim-devon-prod-scus-001
Resource Group: rg-apim-devon-prod-scus-001
SKU: Developer Tier (for development) / Standard (for production)
Location: southcentralus
```

**Rationale**:
- **Cost Optimization**: Competitive pricing for selected tier
- **Performance**: Optimal latency to Devon's primary user base
- **Compliance**: Meets Devon's organizational compliance requirements
- **Feature Availability**: All required APIM features supported

### Implications
- **Positive**: Cost optimization, adequate performance, compliance alignment
- **Negative**: Single region deployment (no geo-redundancy initially)
- **Mitigations**: Disaster recovery plan documented, multi-region roadmap planned

---

## ADR-004: Subscription Guard Strategy

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
APIOps pipeline needed protection against accidental production deployments while maintaining development agility for Devon's environments.

### Options Considered

| Approach | Protection Level | Flexibility | Maintenance |
|----------|-----------------|-------------|-------------|
| **Hard-coded Guards** | High | Low | Low |
| **Environment Variables** | Medium | High | Medium |
| **Parameter-based** | High | High | High |

### Recommended Decision (to be reviewed by Devon)
**Selected: Parameter-based Guards with Environment Variables**

**Implementation**:
```json
// infra/params/devon-dev.json
{
  "subscriptionId": {
    "value": "devon-dev-subscription-id"
  },
  "environmentType": {
    "value": "development"
  },
  "deploymentMode": {
    "value": "sandbox"
  }
}
```

**Rationale**:
- **Safety**: Explicit subscription targeting prevents cross-environment errors
- **Flexibility**: Parameter files allow Devon's environment-specific configurations
- **Auditability**: Clear deployment target in version control
- **Scalability**: Easy to extend for Devon's multiple environments

### Implications
- **Positive**: Strong protection, environment flexibility, clear audit trail
- **Negative**: Additional parameter file maintenance
- **Mitigations**: Automated validation, template standardization

---

## ADR-005: API Extraction Format (JSON vs YAML)

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
APIOps extractor tool supported multiple formats for OpenAPI specifications, requiring standardization for consistency and tooling compatibility within Devon's development workflow.

### Options Considered

| Format | Readability | Tool Support | Version Control | File Size |
|--------|-------------|--------------|-----------------|-----------|
| **JSON** | Low | Excellent | Poor diffs | Compact |
| **YAML** | High | Good | Clean diffs | Verbose |

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
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
APIOps pipeline required comprehensive testing strategy for 11 artifact types while maintaining security boundaries for Devon's APIM deployment.

### Analysis Results

| Category | Artifact Types | Test Coverage | Security Impact |
|----------|----------------|---------------|-----------------|
| **Fully Testable** | APIs, Policies, Products, Groups, Tags | 5/11 (45%) | Low risk |
| **Partially Testable** | Backends, Diagnostics, Certificates, Versions | 4/11 (36%) | Medium risk |
| **Security Excluded** | Named Values, Loggers | 2/11 (18%) | High risk |

### Recommended Decision (to be reviewed by Devon)
**Selected: Multi-layered Testing with Security Exclusions**

**Testing Framework**:
```yaml
# .github/workflows/apim-ci.yml
testing:
  spectral_validation: true    # OpenAPI specs
  json_schema: true           # Configuration files
  xml_validation: true        # Policy documents
  security_scan: false        # Excluded artifacts
```

**Rationale**:
- **Security First**: Named values and credentials excluded from version control
- **Quality Gates**: Spectral linting for API specifications
- **Comprehensive Coverage**: Multiple validation tools for different artifact types
- **Risk Mitigation**: 4-phase improvement plan for gap closure

### Test Gap Closure Plan
1. **Phase 1**: Implement missing schema validators (Month 1)
2. **Phase 2**: Add policy-specific testing (Month 2)
3. **Phase 3**: Backend connectivity validation (Month 3)
4. **Phase 4**: Diagnostic configuration testing (Month 4)

### Implications
- **Positive**: Strong quality gates, security compliance, systematic improvement
- **Negative**: Initial coverage gaps, complex testing matrix
- **Mitigations**: Phased rollout, comprehensive documentation, regular reviews

---

## ADR-007: Artifact Storage and Version Control Strategy

**Date**: July 2025  
**Status**: ✅ Sandbox Recommendation  
**Sandbox Phase Authors**: Microsoft CSA and DevOps Contributor

### Context
APIOps artifacts required version control strategy balancing transparency, security, and operational efficiency for Devon's APIM deployment.

### Artifact Classification

| Classification | Storage Strategy | Examples |
|----------------|------------------|----------|
| **Public** | Full version control | OpenAPI specs, group definitions |
| **Internal** | Version controlled with review | Product policies, API configurations |
| **Confidential** | Excluded via .gitignore | Named values, logger credentials |

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

| Decision | Timeline | Priority | Devon Owner |
|----------|----------|----------|-------------|
| **Multi-region Deployment** | Q3 2025 | Medium | Devon Platform Team |
| **Production Environment Setup** | Q4 2025 | High | Devon DevOps Team |
| **API Versioning Strategy** | Q2 2025 | Medium | Devon API Standards Team |
| **Monitoring and Alerting** | Q2 2025 | High | Devon Operations Team |

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
