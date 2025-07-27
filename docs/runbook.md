# APIM Landing Zone Accelerator - Operations Runbook

*Standard Operating Procedures for APIOps CI/CD Pipeline Management*

## Quick Reference

| Component | Status Check | Emergency Contact |
|-----------|--------------|-------------------|
| GitHub Actions | [Workflow Status](.github/workflows/apim-ci.yml) | DevOps Team |
| APIM Service | `apim-devon-dev-eastus2-uxs` | Platform Team |
| Spectral Linting | PR Quality Gates | API Standards Team |
| Artifact Security | `.gitignore` exclusions | Security Team |

## Setup Instructions

### 1. Repository Configuration

#### Required GitHub Variables
```bash
AZURE_CLIENT_ID=21699d40-4802-42c5-abb0-bac88a3101e1
AZURE_TENANT_ID=<tenant-guid>
AZURE_SUBSCRIPTION_ID=5734f8d9-881b-4d80-96d3-abd833647290
```

#### Required GitHub Secrets
- None (using OIDC authentication)

#### Federated Identity Setup
1. **Service Principal**: `21699d40-4802-42c5-abb0-bac88a3101e1`
2. **Subject**: `repo:paespi0/apim-landing-zone-accelerator:ref:refs/heads/import`
3. **Issuer**: `https://token.actions.githubusercontent.com`

### 2. APIM Service Configuration

#### Target Environment
- **Service**: `apim-devon-dev-eastus2-uxs`
- **Resource Group**: `rg-apim-devon-dev-eastus2-uxs`
- **Region**: East US 2
- **Subscription**: Devon Dev (5734f8d9-881b-4d80-96d3-abd833647290)

#### Parameter File Location
```
infra/params/sandbox.json
```

### 3. APIOps Tools Version
- **Extractor**: v6.0.1.0
- **Publisher**: v6.0.1.0
- **Spectral**: Latest (GitHub Actions)

## Standard Operations

### Daily Operations

#### Health Check Procedure
1. **Verify Workflow Status**
   ```bash
   gh workflow list --repo paespi0/apim-landing-zone-accelerator
   ```

2. **Check APIM Service Health**
   ```bash
   az apim show --name apim-devon-dev-eastus2-uxs --resource-group rg-apim-devon-dev-eastus2-uxs
   ```

3. **Validate Security Exclusions**
   ```bash
   git status --ignored
   # Should show: artifacts/named\ values/, artifacts/loggers/
   ```

#### Artifact Extraction Process
1. **Manual Trigger Workflow**
   - Navigate to Actions tab
   - Select "APIOps CI/CD Pipeline"
   - Click "Run workflow" on `import` branch

2. **Review Extraction Results**
   - Check workflow logs for errors
   - Verify new artifacts in `artifacts/` folder
   - Ensure sensitive data excluded

### Weekly Operations

#### Security Audit
1. **Check for Credential Exposure**
   ```bash
   git log --all --grep="secret\|password\|key" --oneline
   ```

2. **Validate .gitignore Effectiveness**
   ```bash
   find artifacts/ -name "*secret*" -o -name "*credential*" -o -name "*key*"
   # Should return no results if properly excluded
   ```

3. **Review Access Permissions**
   - Verify service principal permissions
   - Check OIDC federation configuration
   - Validate repository access controls

## Toggle Flags & Guards

### Subscription Protection Guards

#### Production Subscription Lock
```json
// In infra/params/sandbox.json
{
  "subscriptionId": {
    "value": "5734f8d9-881b-4d80-96d3-abd833647290"  // Dev only
  },
  "deployToProduction": {
    "value": false  // Hard lock for production
  }
}
```

#### Environment-Specific Toggles
- **Dev Environment**: All operations enabled
- **Staging Environment**: Publisher disabled by default
- **Production Environment**: Extraction only (no publishing)

### Product Configuration Guards

#### Product Staging Controls
```yaml
# In .github/workflows/apim-ci.yml
env:
  EXTRACT_PRODUCTS: true      # Always extract for audit
  PUBLISH_PRODUCTS: false     # Manual approval required
  PUBLISH_POLICIES: false     # Security review required
```

#### API-Level Toggles
- **echo-api**: Fully automated (test API)
- **Production APIs**: Manual approval workflow
- **External APIs**: Extraction only (no modifications)

## GitHub Actions Management

### Variable Setup Procedure

#### Organization Variables (if applicable)
1. **Navigate**: GitHub.com → Settings → Actions → Variables
2. **Required Variables**:
   - `AZURE_CLIENT_ID`: Service principal client ID
   - `AZURE_TENANT_ID`: Azure AD tenant identifier
   - `AZURE_SUBSCRIPTION_ID`: Target subscription GUID

#### Repository Variables
1. **Navigate**: Repository → Settings → Secrets and variables → Actions
2. **Variables Tab**: Add required Azure configuration
3. **Validation**: Test workflow trigger after setup

### Workflow Troubleshooting

#### Common Issues & Solutions

| Issue | Symptom | Resolution |
|-------|---------|------------|
| OIDC Authentication Failure | `Error: AADSTS70021` | Verify federated credential subject matches branch |
| Spectral Lint Failures | PR checks failing | Review OpenAPI spec against `.spectral.yml` rules |
| Artifact Extraction Empty | No files in `artifacts/` | Check APIM service connectivity and permissions |
| Policy Validation Errors | XML parsing failures | Validate policy syntax against APIM schema |

## Spectral Linting Management

### Configuration Location
```
.spectral.yml
```

### Rule Categories
- **OpenAPI 3.0 Compliance**: Schema validation
- **API Design Standards**: Naming conventions, structure
- **Security Requirements**: Authentication, HTTPS enforcement
- **Documentation Quality**: Description completeness

### Handling Spectral Failures

#### Investigation Steps
1. **Review Failure Details**
   ```bash
   gh pr checks <pr-number>
   ```

2. **Local Validation**
   ```bash
   npx @stoplight/spectral-cli lint artifacts/apis/*/specification.yaml
   ```

3. **Rule Override (if justified)**
   ```yaml
   # In specification.yaml (temporary)
   # spectral:disable-next-line rule-name
   ```

#### Exception Process
1. **Document Justification**: Add comment explaining override
2. **Security Review**: For authentication-related overrides
3. **Time-bound**: Set removal date for temporary overrides

## Artifact Security Management

### Sensitive Data Exclusions

#### Automated Exclusions (via .gitignore)
```gitignore
# APIOps - Exclude sensitive artifacts
artifacts/named\ values/
artifacts/loggers/
artifacts/**/*credential*
artifacts/**/*secret*
artifacts/**/*key*
```

#### Manual Review Required
- **Policy Files**: May contain backend URLs
- **Product Configurations**: May reference internal systems
- **Diagnostic Settings**: May expose resource identifiers

### Data Classification

| Classification | Examples | Handling |
|----------------|----------|----------|
| **Public** | OpenAPI specs, group definitions | Version controlled |
| **Internal** | Product policies, diagnostic configs | Version controlled with review |
| **Confidential** | Named values, logger credentials | Excluded from Git |
| **Restricted** | Production secrets, certificates | Never in repository |

## Rollback Procedures

### Emergency Rollback (< 5 minutes)

#### Immediate Actions
1. **Disable Workflow**
   ```bash
   gh workflow disable apim-ci.yml --repo paespi0/apim-landing-zone-accelerator
   ```

2. **Revert Last Deployment**
   ```bash
   git log --oneline -5  # Find last good commit
   git revert <commit-hash>
   git push origin import
   ```

3. **Verify Service Health**
   - Check APIM service availability
   - Validate API endpoints responding
   - Monitor error rates

### Planned Rollback (maintenance window)

#### Pre-rollback Checklist
- [ ] Notify stakeholders of maintenance window
- [ ] Backup current APIM configuration
- [ ] Prepare rollback commit hash
- [ ] Have escalation contacts available

#### Rollback Execution
1. **Create Rollback Branch**
   ```bash
   git checkout -b rollback/emergency-fix
   git revert <problematic-commit>
   ```

2. **Test Rollback Changes**
   - Validate in sandbox environment
   - Run Spectral linting
   - Verify artifact extraction

3. **Deploy Rollback**
   ```bash
   git push origin rollback/emergency-fix
   gh pr create --title="Emergency Rollback" --body="Rolling back problematic changes"
   ```

## Monitoring & Alerting

### Key Metrics
- **Workflow Success Rate**: >95%
- **Spectral Lint Pass Rate**: >98%
- **Artifact Extraction Completeness**: 100%
- **Security Exclusion Effectiveness**: 100%

### Alert Thresholds
- **Workflow Failure**: Immediate notification
- **Repeated Spectral Failures**: 3 consecutive failures
- **Credential Exposure**: Immediate security alert
- **Unauthorized Artifact Changes**: Real-time notification

## Contact Information

### Escalation Matrix
1. **L1 - DevOps Team**: Workflow and deployment issues
2. **L2 - Platform Team**: APIM service and infrastructure
3. **L3 - Security Team**: Credential exposure and compliance
4. **L4 - Architecture Team**: Design decisions and major changes

### Communication Channels
- **Incidents**: #apim-ops-alerts
- **Changes**: #apim-deployments  
- **Security**: #security-incidents
- **Architecture**: #platform-architecture

---

*Operations runbook for APIM Landing Zone Accelerator - ensuring reliable and secure APIOps pipeline management*
