# APIOps Test Coverage Map

*Comprehensive overview of automated testing capabilities for all APIM artifacts*

## Artifact Testing Overview

| Artifact Type | Testability | Test Strategy |
|---------------|-------------|---------------|
| APIs (OpenAPI) | ✅ Fully Testable | Automated validation via Spectral linting in GitHub Actions with comprehensive OpenAPI 3.0 schema checks |
| API Information | ✅ Fully Testable | JSON schema validation against APIM service configuration requirements for paths, protocols, and subscription settings |
| API Operations | ⚠️ Partial | XML syntax validation and policy structure checks but runtime behavior testing requires backend service mocking |
| API Policies (XML) | ⚠️ Partial | Automated XML syntax validation and APIM policy element verification but functional testing needs live environment |
| Global Policies | ⚠️ Partial | XML structure validation and policy syntax checking but cross-cutting behavior testing requires full APIM deployment |
| Products | ✅ Fully Testable | Complete JSON schema validation for product configuration including state, limits, and approval workflow settings |
| Product Policies | ⚠️ Partial | XML syntax and rate limiting configuration validation but actual quota enforcement testing needs traffic simulation |
| Groups | ✅ Fully Testable | Full JSON schema validation for group metadata including system vs custom group type verification |
| Diagnostics | ✅ Fully Testable | Complete configuration validation for Application Insights integration including logger references and sampling settings |
| Loggers | ❌ Excluded | Contains sensitive credential references and resource IDs that are properly excluded from version control for security |
| Named Values | ❌ Excluded | Contains secrets and sensitive configuration values that are correctly excluded from automated testing for security compliance |

## Coverage Summary

**5 fully testable** artifact types with complete automated validation capabilities  
**4 partially testable** types requiring manual verification for runtime behavior  
**2 excluded** types properly secured from testing due to sensitive content

Current automated testing is implemented via [Spectral OpenAPI linting in GitHub Actions](.github/workflows/apim-ci.yml) with comprehensive schema validation and style guide enforcement.

---

*Test coverage analysis for APIOps CI/CD pipeline - ensuring quality gates for all deployable artifacts*
