# APIM Landing Zone Accelerator - Architecture Decision Log

*Key technical decisions made during APIOps migration and landing zone implementation*

## Decision Record Format

Each decision includes:
- **Context**: Why the decision was needed
- **Options**: Alternatives considered
- **Decision**: Chosen approach and rationale
- **Consequences**: Trade-offs and implications

---

## ADR-001: Devon vs Microsoft Tenant Selection

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: Platform Architecture Team

### Context
APIOps pipeline required selection between Devon corporate tenant and Microsoft tenant for Azure resource deployment and authentication.

### Options Considered

| Option | Pros | Cons |
|--------|------|------|
| **Devon Tenant** | Corporate compliance, existing governance, cost allocation | Limited scalability, internal dependencies |
| **Microsoft Tenant** | Azure native integration, better tooling support, scalability | External dependency, compliance complexity |

### Decision
**Selected: Devon Tenant**

**Rationale**:
- Corporate governance and compliance requirements mandate use of internal tenant
- Existing identity management and access controls already established
- Cost allocation and budgeting aligned with corporate processes
- Security policies and audit requirements already configured

### Consequences
- **Positive**: Simplified compliance, existing governance, cost transparency
- **Negative**: Potential scalability limitations, dependency on corporate IT
- **Mitigations**: Established escalation paths, documented service level agreements

---

## ADR-002: OIDC vs Service Principal Authentication

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: Security Team, DevOps Team

### Context
GitHub Actions workflow required authentication method for Azure resource access, choosing between traditional service principal secrets and modern OIDC federation.

### Options Considered

| Option | Security | Maintenance | Scalability | Audit |
|--------|----------|-------------|-------------|-------|
| **Service Principal + Secrets** | Medium | High maintenance | Good | Complex |
| **OIDC Federation** | High | Low maintenance | Excellent | Simple |

### Decision
**Selected: OIDC Federation**

**Technical Implementation**:
```json
{
  "name": "gh-paespi0-apim-landing-zone-accelerator",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:paespi0/apim-landing-zone-accelerator:ref:refs/heads/import",
  "audiences": ["api://AzureADTokenExchange"]
}
```

**Rationale**:
- **Security**: No secrets stored in GitHub, tokens short-lived
- **Maintenance**: No secret rotation required
- **Auditability**: Clear token lineage and usage tracking
- **Modern**: Industry best practice for CI/CD authentication

### Consequences
- **Positive**: Enhanced security, reduced operational overhead, better audit trail
- **Negative**: Newer technology, potential debugging complexity
- **Mitigations**: Comprehensive documentation, fallback procedures established

---

## ADR-003: East US 2 Region Selection

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: Platform Team, Cost Optimization Team

### Context
APIM service deployment required region selection, balancing cost, performance, compliance, and feature availability.

### Options Considered

| Region | Cost | Latency | Compliance | Feature Set |
|--------|------|---------|------------|-------------|
| **East US** | High | Medium | ✅ SOC 2 | Full |
| **East US 2** | Medium | Low | ✅ SOC 2 | Full |
| **Central US** | Low | High | ✅ SOC 2 | Limited |

### Decision
**Selected: East US 2**

**Service Configuration**:
```yaml
Service Name: apim-devon-dev-eastus2-uxs
Resource Group: rg-apim-devon-dev-eastus2-uxs
SKU: Developer Tier
Location: East US 2
```

**Rationale**:
- **Cost Optimization**: 15% lower than East US
- **Performance**: Low latency to primary user base
- **Compliance**: SOC 2 Type II certified
- **Feature Availability**: Full APIM feature set supported

### Consequences
- **Positive**: Cost savings, optimal performance, compliance alignment
- **Negative**: Single region deployment (no geo-redundancy initially)
- **Mitigations**: Disaster recovery plan documented, multi-region roadmap planned

---

## ADR-004: Subscription Guard Strategy

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: Security Team, Platform Team

### Context
APIOps pipeline needed protection against accidental production deployments while maintaining development agility.

### Options Considered

| Approach | Protection Level | Flexibility | Maintenance |
|----------|-----------------|-------------|-------------|
| **Hard-coded Guards** | High | Low | Low |
| **Environment Variables** | Medium | High | Medium |
| **Parameter-based** | High | High | High |

### Decision
**Selected: Parameter-based Guards with Environment Variables**

**Implementation**:
```json
// infra/params/sandbox.json
{
  "subscriptionId": {
    "value": "5734f8d9-881b-4d80-96d3-abd833647290"
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
- **Flexibility**: Parameter files allow environment-specific configurations
- **Auditability**: Clear deployment target in version control
- **Scalability**: Easy to extend for multiple environments

### Consequences
- **Positive**: Strong protection, environment flexibility, clear audit trail
- **Negative**: Additional parameter file maintenance
- **Mitigations**: Automated validation, template standardization

---

## ADR-005: API Extraction Format (JSON vs YAML)

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: API Standards Team, DevOps Team

### Context
APIOps extractor tool supported multiple formats for OpenAPI specifications, requiring standardization for consistency and tooling compatibility.

### Options Considered

| Format | Readability | Tool Support | Version Control | File Size |
|--------|-------------|--------------|-----------------|-----------|
| **JSON** | Low | Excellent | Poor diffs | Compact |
| **YAML** | High | Good | Clean diffs | Verbose |

### Decision
**Selected: YAML for OpenAPI Specifications**

**Configuration**:
```yaml
# configuration.extractor.yaml
openApiSpecFormat: "yaml"
preserveComments: true
indentSize: 2
```

**Rationale**:
- **Human Readability**: YAML more accessible for API documentation review
- **Version Control**: Clean, meaningful diffs for change tracking
- **Industry Standard**: OpenAPI community preference for YAML
- **Tooling**: Spectral and other linting tools optimized for YAML

### Consequences
- **Positive**: Better readability, cleaner version control, industry alignment
- **Negative**: Larger file sizes, potential YAML formatting issues
- **Mitigations**: Automated validation, standardized formatting rules

---

## ADR-006: Test Strategy for APIOps Artifacts

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: Quality Engineering Team, Security Team

### Context
APIOps pipeline required comprehensive testing strategy for 11 artifact types while maintaining security boundaries.

### Analysis Results

| Category | Artifact Types | Test Coverage | Security Impact |
|----------|----------------|---------------|-----------------|
| **Fully Testable** | APIs, Policies, Products, Groups, Tags | 5/11 (45%) | Low risk |
| **Partially Testable** | Backends, Diagnostics, Certificates, Versions | 4/11 (36%) | Medium risk |
| **Security Excluded** | Named Values, Loggers | 2/11 (18%) | High risk |

### Decision
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

### Consequences
- **Positive**: Strong quality gates, security compliance, systematic improvement
- **Negative**: Initial coverage gaps, complex testing matrix
- **Mitigations**: Phased rollout, comprehensive documentation, regular reviews

---

## ADR-007: Artifact Storage and Version Control Strategy

**Date**: July 2025  
**Status**: ✅ Decided  
**Deciders**: DevOps Team, Security Team

### Context
APIOps artifacts required version control strategy balancing transparency, security, and operational efficiency.

### Artifact Classification

| Classification | Storage Strategy | Examples |
|----------------|------------------|----------|
| **Public** | Full version control | OpenAPI specs, group definitions |
| **Internal** | Version controlled with review | Product policies, API configurations |
| **Confidential** | Excluded via .gitignore | Named values, logger credentials |

### Decision
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
- **Compliance**: Meets corporate data classification requirements
- **Operational**: Automated extraction without manual filtering

### Consequences
- **Positive**: Strong security, full audit trail for non-sensitive changes
- **Negative**: Limited visibility into credential configurations
- **Mitigations**: Separate credential management process, security reviews

---

## Future Decisions

### Pending Architecture Reviews

| Decision | Timeline | Priority | Owner |
|----------|----------|----------|-------|
| **Multi-region Deployment** | Q3 2025 | Medium | Platform Team |
| **Production Environment Setup** | Q4 2025 | High | DevOps Team |
| **API Versioning Strategy** | Q2 2025 | Medium | API Standards Team |
| **Monitoring and Alerting** | Q2 2025 | High | Operations Team |

### Decision Review Schedule
- **Quarterly Reviews**: Every 3 months
- **Annual Assessment**: Full architecture review
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
- See `docs/step-4.1-artifact-testability-analysis.md` for technical analysis

---

*Architecture Decision Log for APIM Landing Zone Accelerator - documenting key technical choices and rationale*
