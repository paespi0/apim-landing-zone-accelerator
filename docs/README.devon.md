# APIM Landing Zone Accelerator - Devon Energy Implementation

*Sandbox-phase APIOps implementation for Devon Energy's API Management platform*

> 🏗️ **Sandbox Implementation Notice**  
> This is a **sandbox-phase implementation** created for Devon Energy's evaluation and testing. This implementation demonstrates APIOps principles and provides a foundation for production deployment, but requires additional hardening, customization, and Devon-specific configuration before production use.

---

## Executive Summary

This repository contains a working implementation of the Azure API Management Landing Zone Accelerator, specifically configured for Devon Energy's evaluation during the sandbox phase. The implementation includes automated CI/CD pipelines, infrastructure-as-code templates, and comprehensive documentation to support Devon's transition to modern API management practices.

### Key Capabilities Demonstrated
- **Automated API Lifecycle Management** via APIOps methodology
- **Infrastructure as Code** with parameterized deployment templates  
- **Quality Gates** through Spectral linting and validation
- **Security-First Approach** with OIDC authentication and secret exclusions
- **Multi-Environment Support** with parameter-driven deployments

---

## Repository Structure

```
├── .github/workflows/          # CI/CD automation
│   └── apim-ci.yml            # Main GitHub Actions workflow
├── .spectral.yml              # API linting rules and standards
├── artifacts/                 # APIM configuration artifacts
│   ├── apis/                  # OpenAPI specifications (YAML format)
│   ├── groups/                # User groups and permissions
│   ├── products/              # Product definitions and policies
│   └── policy.xml            # Global policy configurations
├── diagnostics/               # Monitoring and logging configuration
├── docs/                      # Comprehensive documentation
│   ├── decision-log.md        # Complete architectural decisions (7 ADRs)
│   ├── handoff-checklist.md   # Production readiness checklist
│   ├── key-decisions-summary.md # Executive summary of key decisions
│   ├── runbook.md            # Operations procedures
│   └── test-coverage-map.md   # Testing capabilities overview
├── infra/                     # Infrastructure as Code templates
│   └── params/               # Environment-specific parameters
├── loggers/                   # Application logging configuration
├── named values/              # Configuration values (excluded from git)
└── products/                  # Product-specific configurations
```

### 🔒 **Security Note**
Sensitive artifacts like `named values/` and `loggers/` are excluded from version control via `.gitignore` for security compliance. These contain credentials and sensitive configuration that must be managed separately.

---

## CI/CD Pipeline Overview

The automated pipeline implements a 4-stage APIOps workflow:

### Stage 1: Extract & Validation 🔍
```yaml
# Triggered on: Pull Request, Push to main/import
- APIOps artifact extraction from APIM service
- OpenAPI specification validation
- Configuration schema verification
```

### Stage 2: Quality Gates 🚦
```yaml
# Spectral linting for API standards
- OpenAPI 3.0 compliance checking
- API design standard enforcement  
- YAML syntax and structure validation
```

### Stage 3: Security Scanning 🔐
```yaml
# Security and compliance validation
- Subscription guard verification
- Sensitive data exclusion checks
- Parameter validation against approved values
```

### Stage 4: Deployment 🚀
```yaml
# Automated deployment to target environment
- OIDC-authenticated Azure deployment
- Parameter-driven environment targeting
- Rollback capability on failure
```

---

## Getting Started for Devon Engineers

### Prerequisites
- Access to Devon's GitHub organization
- Azure CLI installed and configured
- Basic understanding of OpenAPI specifications
- Familiarity with YAML configuration files

### 1. Repository Access
```bash
# Clone the repository
git clone https://github.com/paespi0/apim-landing-zone-accelerator.git
cd apim-landing-zone-accelerator

# Switch to the authoritative branch
git checkout import
```

### 2. Understanding Current Configuration
```bash
# Review the key architectural decisions
open docs/decision-log.md

# Check current artifact structure
ls -la artifacts/

# Examine CI/CD configuration
cat .github/workflows/apim-ci.yml
```

### 3. Making API Changes
```bash
# 1. Create a feature branch
git checkout -b feature/api-updates

# 2. Modify OpenAPI specifications in artifacts/apis/
# Note: Use YAML format as per Devon's standards

# 3. Update policies if needed in artifacts/policies/
# Note: XML validation will run automatically

# 4. Commit and push changes
git add .
git commit -m "feat: update API specifications for new endpoints"
git push origin feature/api-updates

# 5. Create Pull Request
# CI/CD will automatically run validation and provide feedback
```

### 4. Environment-Specific Deployment
```bash
# Parameter files are located in infra/params/
# Create Devon-specific parameter files:

# infra/params/devon-dev.json     - Development environment
# infra/params/devon-test.json    - Testing environment  
# infra/params/devon-staging.json - Staging environment
# infra/params/devon-prod.json    - Production environment

# Each file should contain Devon's actual subscription IDs and resource names
```

---

## Key Documentation for Devon Teams

### 📋 **Architecture & Decisions**
- **[Architecture Decision Log](./decision-log.md)** - Complete record of 7 architectural decisions made during sandbox phase
- **[Key Decisions Summary](./key-decisions-summary.md)** - Executive summary of top 10 critical decisions requiring Devon review
- **[Handoff Checklist](./handoff-checklist.md)** - Production readiness checklist with 28 specific tasks for Devon teams

### 🔧 **Operations & Procedures**
- **[Operations Runbook](./runbook.md)** - Standard operating procedures for CI/CD pipeline management
- **[Test Coverage Map](./test-coverage-map.md)** - Overview of automated testing capabilities and gaps
- **[Test Gap Closure Plan](./test-gap-closure-plan.md)** - 4-phase plan to improve test coverage from 45% to 80%

### 🏗️ **Technical Implementation**
- **[Artifact Testability Analysis](./artifact-testability-analysis.md)** - Detailed analysis of testing capabilities
- **[Audit API Testability Analysis](./audit-api-testability-analysis.md)** - Devon's legacy APIM validation analysis
- **[Promotion Strategy](./promotion-strategy.md)** - Multi-environment CI/CD deployment strategy
- **[Final Evaluation](./final-evaluation.md)** - Sandbox phase completion assessment
- **[Security Assessment](./security-assessment.md)** - Comprehensive security validation
- **[Naming Conventions](./naming-conventions.md)** - Resource naming standards for Devon environments
- **[Network Integration](./README-network.md)** - Network topology and integration guidance

---

## Current Implementation Status

### ✅ **Implemented Features**
- **OIDC Authentication** - Secure, secret-free deployment authentication
- **Multi-Environment Support** - Parameter-driven deployments with subscription guards
- **Quality Gates** - Spectral linting with OpenAPI validation
- **Security Compliance** - Sensitive data exclusion and proper secret management
- **Documentation** - Comprehensive architecture and operational documentation

### ⚠️ **Sandbox Limitations**
- **Single Region** - Currently configured for South Central US only
- **Limited Test Coverage** - 45% artifact coverage (improvement plan available)
- **Basic Monitoring** - Enhanced observability needed for production
- **Template Values** - Contains placeholder values requiring Devon customization

### 🚀 **Production Readiness Requirements**
1. **Security Review** - Devon InfoSec team approval of OIDC and authentication strategy
2. **Parameter Customization** - Replace all template values with Devon-specific configuration
3. **Environment Setup** - Configure actual dev/test/staging/production environments
4. **Compliance Validation** - Ensure alignment with Devon's governance requirements
5. **Team Training** - APIOps methodology training for Devon development teams

---

## Devon-Specific Customization Guide

### 🏢 **Required Devon Configuration**

#### 1. Azure Environment Setup
```json
// infra/params/devon-prod.json
{
  "subscriptionId": "devon-production-subscription-id",
  "resourceGroupName": "rg-apim-devon-prod-scus-001", 
  "apimServiceName": "apim-devon-prod-scus-001",
  "region": "southcentralus"
}
```

#### 2. GitHub Secrets Configuration
```bash
# Required GitHub repository variables:
AZURE_CLIENT_ID=<devon-service-principal-id>
AZURE_TENANT_ID=<devon-azure-tenant-id>
AZURE_SUBSCRIPTION_ID=<devon-target-subscription>
```

#### 3. OIDC Federation Setup
```json
// Azure AD App Registration
{
  "name": "gh-devon-apim-landing-zone-accelerator",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:devon-github-org/apim-landing-zone-accelerator:ref:refs/heads/main"
}
```

### 📝 **Customization Checklist**
- [ ] Replace all `<your-*>` placeholders in documentation
- [ ] Configure Devon-specific subscription IDs in parameter files
- [ ] Update resource naming to match Devon's conventions
- [ ] Customize Spectral rules for Devon's API standards
- [ ] Configure monitoring integration with Devon's observability platform
- [ ] Set up proper branch protection rules in Devon's GitHub organization

---

## Support & Escalation

### 🆘 **Getting Help**

| Issue Type | Contact | Resources |
|------------|---------|-----------|
| **Technical Questions** | Microsoft CSA Team | [Runbook](./runbook.md) |
| **Architecture Decisions** | Devon Enterprise Architecture | [Decision Log](./decision-log.md) |
| **Production Readiness** | Devon Platform Team | [Handoff Checklist](./handoff-checklist.md) |
| **API Standards** | Devon API Center of Excellence | [Test Coverage Map](./test-coverage-map.md) |

### 📞 **Emergency Contacts**
- **Pipeline Failures**: Devon DevOps Team
- **Security Concerns**: Devon InfoSec Team  
- **Compliance Issues**: Devon Governance Team
- **Production Issues**: Devon Operations Center

---

## Common Workflows

### 🔄 **Adding a New API**
1. Create OpenAPI specification in `artifacts/apis/new-api/`
2. Define API policies in `artifacts/apis/new-api/policy.xml`
3. Update product configuration if needed
4. Create pull request - CI/CD will validate automatically
5. Review Spectral linting feedback and address issues
6. Merge after approval - deployment triggers automatically

### 🛠️ **Updating Global Policies**
1. Modify `artifacts/policy.xml` for global policy changes
2. Test XML syntax and policy logic
3. Create pull request with detailed change description
4. Review impact on existing APIs
5. Deploy through standard CI/CD process

### 🔧 **Environment Promotion**
1. Validate changes in development environment
2. Update environment-specific parameters
3. Run security and compliance checks
4. Deploy to staging for final validation
5. Promote to production with proper approvals

---

## Next Steps for Devon

### 📈 **Immediate Actions** (Week 1-2)
1. Review and approve architectural decisions in [decision log](./decision-log.md)
2. Complete [handoff checklist](./handoff-checklist.md) tasks for security and compliance
3. Customize parameter files with Devon-specific values
4. Set up OIDC federation and GitHub integration

### 🚀 **Phase 2 Enhancements** (Months 2-6)
1. Implement multi-region deployment strategy
2. Enhance test coverage from 45% to 80%
3. Integrate with Devon's monitoring and alerting systems
4. Establish comprehensive API governance processes
5. Implement advanced security and compliance automation

### 🎯 **Long-term Roadmap** (6+ Months)
1. Full API lifecycle management implementation
2. Advanced analytics and API usage monitoring  
3. Developer portal and self-service capabilities
4. Integration with Devon's existing enterprise systems
5. Continuous compliance and security automation

---

## Important Notes for Devon Teams

### ⚠️ **Critical Reminders**
- This is a **sandbox implementation** - not production-ready without Devon customization
- All template values (marked with `<your-*>`) must be replaced before production use
- Sensitive configuration is properly excluded from version control
- OIDC authentication requires proper Azure AD setup before deployment
- Testing coverage is currently at 45% - enhancement plan available

### 🔐 **Security Considerations**
- Never commit secrets or credentials to version control
- Use OIDC federation instead of service principal secrets
- Validate all parameter files contain only approved values
- Regularly audit `.gitignore` effectiveness for sensitive data exclusion
- Follow Devon's security policies for access control and authentication

---

*Devon Energy APIM Landing Zone Accelerator - Sandbox Implementation*  
*Document Version: 1.0 | Date: July 2025 | Implementation Phase: Sandbox*  
*Next Review: Production Readiness Assessment*
