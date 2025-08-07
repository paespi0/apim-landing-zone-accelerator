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

## **Security Features Included**

✅ **Subscription Guard**: The `artifacts/subscriptions/test.json` file demonstrates the security guard functionality  
✅ **Credential Detection**: Pipeline automatically scans for exposed secrets  
✅ **OIDC Authentication**: Zero-secret authentication to Azure  
✅ **Environment Gates**: Deploy job requires `sbx` environment approval  

## **Pipeline Testing**

1. **Security Guard Test**: The empty `test.json` file will trigger security violation (expected)
2. **Remove Test File**: Delete `artifacts/subscriptions/test.json` to pass security checks
3. **OIDC Test**: Verify authentication works with Devon's federated identity

---
*Repository handed off from Microsoft CSA to Devon Energy*
