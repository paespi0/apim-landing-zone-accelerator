# 🚀 Devon Energy Repository Fork Setup Guide

## **✅ GitHub Fork Instructions**

### **Step 1: Fork the Repository**
1. **Navigate to source repository:** https://github.com/paespi0/apim-landing-zone-accelerator
2. **Click "Fork" button** (top-right corner)
3. **Configure fork settings:**
   - **Owner:** Select Devon Energy organization
   - **Repository name:** `apim-landing-zone-accelerator-devon`
   - **Description:** `Devon Energy APIM Landing Zone Accelerator`
   - **Visibility:** Keep public initially (will change to private in Step 2)
4. **Click "Create fork"**

### **Step 1a: Switch to Import Branch (Critical)**
After forking, the Devon Energy-specific work is in the `import` branch:

```bash
# Clone your forked repository
git clone https://github.com/[DEVON-ORG]/apim-landing-zone-accelerator-devon.git
cd apim-landing-zone-accelerator-devon

# Switch to the import branch (contains all Devon-specific preparation)
git checkout import

# Optional: Make import your default branch
git branch -m import main
git push -u origin main
git push origin --delete import  # Clean up old import branch

# Update default branch in GitHub Settings → General → Default branch
```

> **⚠️ IMPORTANT**: All Devon Energy setup documentation and configurations are in the `import` branch, not `main`. The `main` branch contains only the generic Azure reference implementation.

### **Step 2: Make Repository Private**
1. **Go to forked repository:** `https://github.com/[DEVON-ORG]/apim-landing-zone-accelerator-devon`
2. **Navigate to Settings** → **General**
3. **Scroll to "Danger Zone"**
4. **Click "Change repository visibility"**
5. **Select "Make private"**
6. **Confirm by typing repository name**

### **Step 3: Configure Default Branch (Optional)**
```bash
# If you want to rename 'import' branch to 'main':
git clone https://github.com/[DEVON-ORG]/apim-landing-zone-accelerator-devon.git
cd apim-landing-zone-accelerator-devon
git branch -m import main
git push -u origin main

# Then in GitHub Settings → General → Default branch:
# Change default branch from 'import' to 'main'
# Delete 'import' branch after confirmation
```

### **Step 4: Set Up Repository Variables**
**Navigate to:** `Settings` → `Secrets and variables` → `Actions` → `Variables` tab

**Create these repository variables:**
| Variable Name | Description | Your Value |
|---------------|-------------|------------|
| `AZURE_CLIENT_ID` | Devon's App Registration Client ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `AZURE_TENANT_ID` | Devon's Azure AD Tenant ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `AZURE_SUBSCRIPTION_ID` | Devon's Target Subscription ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |

## **🛡️ Security Recommendations**

### **Enable Repository Security Features**
1. **Settings** → **Security & analysis**
   - ✅ **Dependency graph:** Enable
   - ✅ **Dependabot alerts:** Enable  
   - ✅ **Dependabot security updates:** Enable
   - ✅ **Secret scanning:** Enable
   - ✅ **Push protection:** Enable

### **Configure Branch Protection**
1. **Settings** → **Branches** → **Add rule**
2. **Branch name pattern:** `main` (or your default branch)
3. **Enable these protections:**
   - ✅ **Require a pull request before merging**
   - ✅ **Require approvals:** Set to `1` minimum
   - ✅ **Dismiss stale PR approvals when new commits are pushed**
   - ✅ **Require review from code owners**
   - ✅ **Require status checks to pass before merging**
   - ✅ **Require branches to be up to date before merging**
   - ✅ **Require conversation resolution before merging**
   - ✅ **Restrict pushes that create files to LFS**

### **Alternative: Repository Rulesets (Recommended)**
GitHub's newer ruleset feature provides more flexible protection:

1. **Settings** → **Rules** → **Rulesets** → **New ruleset**
2. **Target branches:** `main` (or pattern `main*`, `release/*`)
3. **Rules to enable:**
   - ✅ **Restrict deletions:** Prevent branch deletion
   - ✅ **Restrict force pushes:** Prevent history rewriting
   - ✅ **Require pull request:** Require PR workflow
   - ✅ **Required status checks:** Ensure CI passes
   - ✅ **Require review:** Minimum 1 approval
4. **Enforcement status:** Active
5. **Bypass permissions:** Allow repository administrators to bypass (recommended)

**Benefits of rulesets:**
- More granular control than branch protection rules
- Can target multiple branches with patterns
- Easier to manage across multiple repositories
- Better integration with GitHub Actions

### **Set Up Code Owners (Optional)**
Create `.github/CODEOWNERS` file:
```
# Global owners
* @devon-energy/api-platform-team

# Infrastructure files
/infra/ @devon-energy/infrastructure-team
/.github/ @devon-energy/devops-team

# Documentation
/docs/ @devon-energy/api-platform-team @devon-energy/architecture-team
```

## **📋 Post-Fork Checklist**

### **Immediate Actions (Required)**
- [ ] **Follow DEVON-SETUP.md completely**
- [ ] **Remove demo files:**
  ```powershell
  Remove-Item "artifacts\subscriptions\test.json"
  git add .
  git commit -m "cleanup: remove demo files for production use"
  git push
  ```
- [ ] **Update federated-credential.json:**
  ```powershell
  (Get-Content "federated-credential.json") -replace "repo:paespi0/apim-landing-zone-accelerator:ref:refs/heads/import", "repo:[DEVON-ORG]/apim-landing-zone-accelerator-devon:ref:refs/heads/main" | Set-Content "federated-credential.json"
  ```

### **Environment Configuration**
- [ ] **Configure Azure AD App Registration** (see DEVON-SETUP.md)
- [ ] **Set up federated identity credentials**
- [ ] **Test OIDC authentication** with GitHub Actions
- [ ] **Customize parameter files** in `/infra/params/`
- [ ] **Update naming conventions** to match Devon standards

### **Security Validation**
- [ ] **Review all GitHub repository variables**
- [ ] **Verify no Microsoft-specific references remain**
- [ ] **Test secret scanning** by attempting to commit a fake secret
- [ ] **Validate branch protection** by testing PR workflow
- [ ] **Confirm private repository access** is properly restricted

### **Customization Steps**
- [ ] **Update README.md** with Devon-specific information
- [ ] **Modify parameter files** for Devon's Azure environment
- [ ] **Customize CI/CD workflows** for Devon's approval processes
- [ ] **Add Devon-specific documentation** to `/docs/` folder
- [ ] **Configure monitoring and alerting** per Devon standards

## **📞 Support Resources**

**Primary Documentation:**
- **DEVON-SETUP.md** - Complete setup instructions
- **docs/README.devon.md** - Enterprise implementation guide  
- **docs/security-assessment.md** - Security configuration details

**Validation Commands:**
```powershell
# Verify clean demo file removal
git log --oneline | findstr "cleanup"

# Test Azure authentication
az account show

# Validate GitHub Actions
# Trigger a manual workflow run to test OIDC
```

**✅ Repository successfully forked and configured when all checklist items are complete!**
