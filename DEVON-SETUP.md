# 🚀 **DEVON ENERGY SETUP GUIDE**

## **Required GitHub Repository Variables**

After importing this repository, Devon Energy must configure the following repository variables:

### **Step 1: Navigate to Repository Settings**
```
https://github.com/[DEVON-ORG]/apim-landing-zone-accelerator-devon/settings/variables/actions
```

### **Step 2: Create Repository Variables**
| Variable Name | Description | Example Value |
|---------------|-------------|---------------|
| `AZURE_CLIENT_ID` | Devon's App Registration Client ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `AZURE_TENANT_ID` | Devon's Azure AD Tenant ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `AZURE_SUBSCRIPTION_ID` | Devon's Target Subscription ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |

### **Step 3: Configure Azure AD Federated Identity**
Create federated identity credential in Devon's App Registration:
```json
{
  "name": "github-actions-devon-apim",
  "issuer": "https://token.actions.githubusercontent.com", 
  "subject": "repo:devon-org/apim-landing-zone-accelerator-devon:environment:sbx",
  "audience": ["api://AzureADTokenExchange"]
}
```

### **Step 4: Update Parameter Files**
Update the following files with Devon-specific values:
- `infra/params/sandbox.json`
- `scenarios/apim-baseline/bicep/parameters.json`

## **⚠️ IMPORTANT: Demo File Cleanup Required**

### **`artifacts/subscriptions/test.json` - REMOVE THIS FILE**

**Purpose**: This file exists solely to demonstrate the security guard functionality during Microsoft's handoff demo.

**Action Required**: Devon Energy must delete this file before using the repository:

```powershell
# Remove the demo file and folder
Remove-Item "artifacts\subscriptions\test.json" -Force
Remove-Item "artifacts\subscriptions" -Force -Recurse

# Commit the cleanup
git add -A
git commit -m "cleanup: remove demo security test file"
git push
```

**Why it exists**: 
- 🎯 **Demo Purpose**: Shows Microsoft CSA that security guards work correctly
- 🚨 **Intentional Failure**: Makes pipeline fail to demonstrate quality gates
- 🛡️ **Security Validation**: Proves subscription artifacts are properly blocked

**After removal**: Pipeline security job will pass ✅

## **Security Features Included**

✅ **Subscription Guard**: Automatically blocks sensitive subscription-level data  
✅ **Credential Detection**: Pipeline scans for exposed secrets and keys  
✅ **OIDC Authentication**: Zero-secret authentication to Azure  
✅ **Environment Gates**: Deploy job requires `sbx` environment approval  
✅ **Automated Scanning**: Spectral linting for API specifications

## **Pipeline Testing Workflow**

1. **Initial State**: Pipeline will fail due to demo `test.json` file (expected)
2. **Remove Demo File**: Follow cleanup steps above
3. **Configure Variables**: Set up Devon's Azure credentials in repository variables
4. **Test OIDC**: Verify authentication works with Devon's federated identity
5. **Deploy**: Pipeline should now pass all security checks ✅

---
*Repository handed off from Microsoft CSA to Devon Energy*
