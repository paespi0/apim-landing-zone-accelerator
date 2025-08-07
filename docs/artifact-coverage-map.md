# APIOps Artifact Coverage Map

This document provides a comprehensive classification of all APIM artifacts extracted by the APIOps pipeline. Each artifact is categorized based on version control, security, and testing considerations for CI/CD operations.

## Classification Legend

- **✅ Versioned**: Safe to commit to Git and track changes
- **🔒 Secure**: Contains sensitive data, requires special handling
- **🛑 Excluded from Git**: Should not be committed to version control
- **🧪 Testable**: Can be validated through automated testing

## Artifact Coverage

| Artifact | Classification | Reason |
|----------|---------------|--------|
| **APIs** | | |
| `apis/echo-api/apiInformation.json` | ✅ Versioned, 🧪 Testable | API metadata and configuration - safe to version |
| `apis/echo-api/specification.yaml` | ✅ Versioned, 🧪 Testable | OpenAPI specification - core API contract |
| `apis/echo-api/operations/*/policy.xml` | ✅ Versioned, 🧪 Testable | Operation-level policies - business logic |
| **Global Configuration** | | |
| `policy.xml` | ✅ Versioned, 🧪 Testable | Global APIM policies - infrastructure as code |
| **Products** | | |
| `products/starter/` | ✅ Versioned, 🧪 Testable | Product configuration - access control definitions |
| `products/unlimited/` | ✅ Versioned, 🧪 Testable | Product configuration - access control definitions |
| **Groups** | | |
| `groups/administrators/` | ✅ Versioned | User group definitions - organizational structure |
| `groups/developers/` | ✅ Versioned | User group definitions - organizational structure |
| `groups/guests/` | ✅ Versioned | User group definitions - organizational structure |
| **Diagnostics** | | |
| `diagnostics/applicationinsights/` | ✅ Versioned | Diagnostic configuration - monitoring setup |
| **Loggers** | | |
| `loggers/appi-devon-dev-eastus2-uxs/loggerInformation.json` | 🔒 Secure, 🛑 Excluded from Git | Contains resource IDs and references to secrets |
| **Named Values** | | |
| `named values/6872a2c44634610e28a21629/namedValueInformation.json` | 🔒 Secure, 🛑 Excluded from Git | Secret named value - contains sensitive credentials |

## Security Considerations

### 🔒 Secure Artifacts
These artifacts contain sensitive information and require special handling:

1. **Named Values with `"secret": true`**: All secret named values should be excluded from Git and managed through secure configuration management
2. **Logger Information**: Contains resource IDs and subscription details that could expose infrastructure topology
3. **Connection Strings**: Any artifacts containing connection strings or access keys

### 🛑 Git Exclusion Rules
The following patterns should be added to `.gitignore`:

```gitignore
# APIOps - Exclude sensitive artifacts
artifacts/named\ values/
artifacts/loggers/
artifacts/**/*credential*
artifacts/**/*secret*
artifacts/**/*key*
```

## Testing Strategy

### 🧪 Testable Artifacts
These artifacts can be validated through automated testing:

- **API Specifications**: OpenAPI schema validation, contract testing
- **Policies**: Policy syntax validation, security rule compliance
- **Product Configurations**: Access control validation
- **Global Policies**: Security policy compliance checks

### Recommended Tests
1. **Schema Validation**: Ensure all JSON/XML artifacts are well-formed
2. **Policy Linting**: Validate policy syntax and security rules
3. **OpenAPI Compliance**: Verify API specifications meet standards
4. **Security Scanning**: Check for hardcoded secrets or insecure configurations

## APIOps Pipeline Integration

### Extract Phase
- ✅ All versioned artifacts are extracted and committed
- 🔒 Secure artifacts are extracted but handled separately
- 🛑 Excluded artifacts are documented but not committed

### Deploy Phase
- ✅ Versioned artifacts are deployed directly from Git
- 🔒 Secure artifacts are injected from secure stores (Key Vault, environment variables)
- 🧪 All artifacts pass validation tests before deployment

## Best Practices

1. **Environment-Specific Values**: Use named values for environment-specific configuration
2. **Secret Management**: Store all secrets in Azure Key Vault, not in Git
3. **Policy Templating**: Use parameterized policies for multi-environment deployment
4. **Automated Testing**: Implement pre-deployment validation for all testable artifacts
5. **Security Scanning**: Regular security audits of policy configurations

---

*Generated from APIOps extraction on July 24, 2025*
*Repository: paespi0/apim-landing-zone-accelerator (import branch)*
