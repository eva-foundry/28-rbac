# EVA RBAC - Troubleshooting Guide

**Last Updated**: February 4, 2026  
**Audience**: All Users, Administrators, DevOps  
**Purpose**: Comprehensive troubleshooting for common RBAC issues

---

## Table of Contents

1. [Quick Diagnostics](#quick-diagnostics)
2. [Authentication Issues](#authentication-issues)
3. [Authorization Issues](#authorization-issues)
4. [Document Upload Issues](#document-upload-issues)
5. [Search & Chat Issues](#search--chat-issues)
6. [Performance Issues](#performance-issues)
7. [Data Consistency Issues](#data-consistency-issues)
8. [Environment-Specific Issues](#environment-specific-issues)

---

## Quick Diagnostics

### 5-Minute Health Check

Run this checklist before diving into specific issues:

```powershell
# Health Check Script
# Run from PowerShell

# 1. Check backend health
Invoke-RestMethod -Uri "https://infoasst-web-dev2.azurewebsites.net/health"

# 2. Check your JWT token
$jwt = $env:X_MS_CLIENT_PRINCIPAL
if ($jwt) {
    Write-Host "[PASS] JWT token present" -ForegroundColor Green
} else {
    Write-Host "[FAIL] No JWT token - not authenticated" -ForegroundColor Red
}

# 3. Check group assignment
Invoke-RestMethod -Uri "https://infoasst-web-dev2.azurewebsites.net/getUsrGroupInfo" -Headers @{"X-MS-CLIENT-PRINCIPAL"=$jwt}

# 4. Check Cosmos DB connectivity
az cosmosdb check-name-exists --name "infoasst-cosmos-dev2"

# 5. Check Azure Search connectivity
az search service show --name "infoasst-search-dev2" --resource-group "rg-infoasst-dev2"
```

**Interpretation**:
- ✅ All checks pass → Proceed to specific issue
- ❌ Backend health fails → Check Azure Web App status
- ❌ JWT missing → Authentication issue (see [Authentication Issues](#authentication-issues))
- ❌ No groups returned → Authorization issue (see [Authorization Issues](#authorization-issues))

---

## Authentication Issues

### Issue 1: Cannot Log In

**Symptoms**:
- Redirect loop at login page
- "Authentication failed" message
- Blank screen after entering credentials

**Diagnostic Steps**:

1. **Verify Azure AD Configuration**:
```powershell
# Check app registration
az ad app show --id <app-id> --query "{DisplayName:displayName, SignInAudience:signInAudience}"
```

2. **Check Redirect URIs**:
   - Azure Portal → App Registrations → EVA → Authentication
   - Verify redirect URI matches your environment:
     - Dev2: `https://infoasst-web-dev2.azurewebsites.net/.auth/login/aad/callback`
     - HCCLD2: `https://infoasst-web-hccld2.azurewebsites.net/.auth/login/aad/callback`

3. **Verify Browser Configuration**:
   - Clear cookies and cache
   - Disable browser extensions (especially ad blockers)
   - Try incognito/private mode

**Solutions**:

**Solution A: Browser Cache**:
```javascript
// Open browser console (F12) and run:
localStorage.clear();
sessionStorage.clear();
location.reload();
```

**Solution B: Azure AD Consent**:
1. Navigate to Azure Portal → Azure AD → Enterprise Applications
2. Find EVA application
3. Click "Users and groups" → Add your account manually
4. Retry login

**Solution C: App Service Authentication**:
```powershell
# Verify authentication enabled
az webapp auth show --name infoasst-web-dev2 --resource-group rg-infoasst-dev2

# If disabled, enable it
az webapp auth update --name infoasst-web-dev2 --resource-group rg-infoasst-dev2 --enabled true
```

### Issue 2: JWT Token Expired

**Symptoms**:
- "Token expired" error in console
- 401 Unauthorized on API calls
- Automatic logout after short period

**Diagnostic**:
```javascript
// In browser console (F12)
const jwt = document.cookie.split('; ').find(row => row.startsWith('AppServiceAuthSession'))
if (jwt) {
    const token = atob(jwt.split('=')[1])
    const payload = JSON.parse(atob(token.split('.')[1]))
    const expiry = new Date(payload.exp * 1000)
    console.log(`Token expires: ${expiry}`)
} else {
    console.log('No JWT token found')
}
```

**Solutions**:

**Solution A: Refresh Session**:
1. Log out completely
2. Close all browser tabs
3. Log back in

**Solution B: Extend Token Lifetime** (Admin only):
```powershell
# Increase token lifetime to 8 hours
az webapp auth update `
    --name infoasst-web-dev2 `
    --resource-group rg-infoasst-dev2 `
    --token-refresh-extension-hours 8
```

### Issue 3: VPN/Private Endpoint Authentication Failure

**Symptoms** (HCCLD2 only):
- "Cannot reach server" when accessing private endpoint
- Authentication succeeds but API calls fail with connection timeout

**Diagnostic**:
```powershell
# Test connectivity to private endpoint
Test-NetConnection -ComputerName infoasst-web-hccld2.azurewebsites.net -Port 443

# Check DNS resolution
Resolve-DnsName infoasst-web-hccld2.azurewebsites.net
```

**Solutions**:

**Solution A: Connect to VPN**:
1. Open Cisco AnyConnect (or ESDC VPN client)
2. Connect to ESDC VPN
3. Wait 30 seconds for DNS propagation
4. Retry EVA access

**Solution B: Use DevBox**:
1. Connect to Microsoft DevBox
2. Access EVA from DevBox browser
3. DevBox is inside HCCLD2 VNet (no VPN needed)

**Solution C: Enable Fallback Mode** (Dev only):
```bash
# In backend.env
LOCAL_DEBUG=true
OPTIMIZED_KEYWORD_SEARCH_OPTIONAL=true
ENRICHMENT_OPTIONAL=true
```

---

## Authorization Issues

### Issue 4: "You are not assigned to any project"

**Symptoms**:
- Login succeeds but no groups shown
- Empty dropdown in group selector
- Error message: "No groups available for your account"

**Diagnostic Steps**:

1. **Check Azure AD Group Membership**:
```powershell
# Get user's groups
$userEmail = "user@hrsdc-rhdcc.gc.ca"
az ad user get-member-groups --id $userEmail --query "[].displayName" -o table
```

2. **Check JWT Claims**:
```javascript
// Browser console (F12)
fetch('/getUsrGroupInfo')
    .then(r => r.json())
    .then(data => console.log('Groups:', data))
```

3. **Check Cosmos DB Mapping**:
```powershell
# Query groupResourcesMapContainer
az cosmosdb sql container query `
    --account-name infoasst-cosmos-dev2 `
    --database-name groupsToResourcesMap `
    --name groupResourcesMapContainer `
    --query-string "SELECT * FROM c"
```

**Solutions**:

**Solution A: Admin Adds User to Group**:
```powershell
$groupId = "9f540c2e-e05c-4012-ba43-4846dabfaea6"  # Get from Azure AD
$userId = az ad user show --id $userEmail --query "id" -o tsv

az ad group member add --group $groupId --member-id $userId
```

**Solution B: Create Cosmos DB Mapping** (if missing):
```json
{
    "id": "9f540c2e-e05c-4012-ba43-4846dabfaea6",
    "group_id": "9f540c2e-e05c-4012-ba43-4846dabfaea6",
    "group_name": "AICoE_Admin_TestRBAC",
    "upload_storage": {
        "upload_container": "upload-testrbac",
        "role": "Storage Blob Data Owner"
    },
    "blob_access": {
        "blob_container": "content-testrbac",
        "role_blob": "Storage Blob Data Owner"
    },
    "vector_index_access": {
        "index": "index-testrbac",
        "role_index": "Search Index Data Contributor"
    }
}
```

**Solution C: Clear Backend Cache**:
```powershell
# Restart web app to reload groups from Cosmos DB
az webapp restart --name infoasst-web-dev2 --resource-group rg-infoasst-dev2
```

### Issue 5: 403 Forbidden on API Calls

**Symptoms**:
- Can see documents but can't perform actions
- Error: "403 Forbidden - Insufficient permissions"
- Specific operations fail (upload, delete, download)

**Diagnostic Steps**:

1. **Check User's Current Group**:
```javascript
// Browser console
fetch('/getUserProfile')
    .then(r => r.json())
    .then(data => console.log('Current group:', data.current_group))
```

2. **Verify Azure RBAC Roles**:
```powershell
$groupId = "9f540c2e-e05c-4012-ba43-4846dabfaea6"
$storageAccount = "/subscriptions/xxx/resourceGroups/rg-infoasst-dev2/providers/Microsoft.Storage/storageAccounts/infoasststoragedev2"

# List role assignments for group
az role assignment list --assignee $groupId --scope $storageAccount --query "[].{Role:roleDefinitionName, Scope:scope}" -o table
```

3. **Check Cosmos DB Mapping**:
```sql
SELECT c.group_name, c.upload_storage.role, c.blob_access.role_blob, c.vector_index_access.role_index
FROM c
WHERE c.group_id = '9f540c2e-e05c-4012-ba43-4846dabfaea6'
```

**Solutions**:

**Solution A: Assign Missing RBAC Role**:
```powershell
# Example: Assign Storage Blob Data Contributor
az role assignment create `
    --assignee $groupId `
    --role "Storage Blob Data Contributor" `
    --scope "$storageAccount/blobServices/default/containers/upload-testrbac"
```

**Solution B: Update Cosmos DB Mapping**:
Update the `upload_storage.role` field to match actual Azure RBAC assignment.

**Solution C: Check Role vs. Action**:
- **Reader trying to upload** → Contact admin for Contributor role
- **Contributor trying to delete others' docs** → Expected behavior (only Admin can)
- **Admin experiencing 403** → Azure RBAC misconfiguration

---

## Document Upload Issues

### Issue 6: Upload Fails with "Network Error"

**Symptoms**:
- File upload progress bar stalls at 0%
- Browser console shows ERR_CONNECTION_REFUSED
- No error message, just infinite loading

**Diagnostic Steps**:

1. **Check Blob Storage Connectivity**:
```powershell
# Test blob storage access
az storage blob list --account-name infoasststoragedev2 --container-name upload-testrbac --auth-mode login
```

2. **Check CORS Configuration**:
```powershell
az storage cors show --account-name infoasststoragedev2 --services b
```

3. **Check Upload Container Exists**:
```powershell
az storage container show --name upload-testrbac --account-name infoasststoragedev2 --auth-mode login
```

**Solutions**:

**Solution A: Enable CORS**:
```powershell
az storage cors add `
    --account-name infoasststoragedev2 `
    --services b `
    --methods POST PUT GET OPTIONS `
    --origins "https://infoasst-web-dev2.azurewebsites.net" `
    --allowed-headers "*" `
    --exposed-headers "*" `
    --max-age 86400
```

**Solution B: Create Upload Container** (if missing):
```powershell
az storage container create `
    --name upload-testrbac `
    --account-name infoasststoragedev2 `
    --auth-mode login
```

**Solution C: Check File Size**:
- Maximum file size: **50 MB**
- Split large files or compress before upload

### Issue 7: Document Stuck in "Processing" State

**Symptoms**:
- Upload succeeds but document never becomes searchable
- Status shows ⏳ "Processing" for hours
- No errors displayed

**Diagnostic Steps**:

1. **Check Azure Functions Status**:
```powershell
# List function apps
az functionapp list --resource-group rg-infoasst-dev2 --query "[].{Name:name, State:state}" -o table

# Check specific function
az functionapp show --name infoasst-func-dev2 --resource-group rg-infoasst-dev2
```

2. **Check Function Logs**:
```powershell
# Stream logs
az functionapp log tail --name infoasst-func-dev2 --resource-group rg-infoasst-dev2

# Or use Application Insights
az monitor app-insights query `
    --app <app-insights-id> `
    --analytics-query "traces | where timestamp > ago(1h) | where message contains 'FileUploadedEtrigger'"
```

3. **Check Queue Messages**:
```powershell
# Check text-enrichment-queue
az storage message peek --queue-name text-enrichment-queue --account-name infoasststoragedev2 --auth-mode login
```

**Solutions**:

**Solution A: Restart Function App**:
```powershell
az functionapp restart --name infoasst-func-dev2 --resource-group rg-infoasst-dev2
```

**Solution B: Manually Trigger Processing**:
1. Open Azure Portal → Storage Account → Queues
2. Find message in `text-enrichment-queue`
3. Delete stuck message
4. Re-upload document

**Solution C: Check Document Intelligence Quota**:
```powershell
# Check Document Intelligence limits
az cognitiveservices account show `
    --name infoasst-docintel-dev2 `
    --resource-group rg-infoasst-dev2 `
    --query "properties.quotaLimit"
```

If quota exceeded, wait for quota reset or upgrade SKU.

---

## Search & Chat Issues

### Issue 8: Chat Returns No Results

**Symptoms**:
- Question submitted but response is "No relevant documents found"
- Documents are indexed (show ✅ in dashboard)
- Other users can search successfully

**Diagnostic Steps**:

1. **Verify Index Contains Documents**:
```powershell
# Check index document count
az search index statistics --service-name infoasst-search-dev2 --index-name index-testrbac --resource-group rg-infoasst-dev2
```

2. **Test Raw Search Query**:
```powershell
# Direct search API call
$headers = @{
    "api-key" = $env:AZURE_SEARCH_ADMIN_KEY
    "Content-Type" = "application/json"
}

$body = @{
    search = "EI misconduct"
    top = 5
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://infoasst-search-dev2.search.windows.net/indexes/index-testrbac/docs/search?api-version=2023-11-01" -Method POST -Headers $headers -Body $body
```

3. **Check User's Search Scope**:
```javascript
// Browser console
fetch('/chat', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
        messages: [{role: 'user', content: 'test'}],
        session_state: {debug: true}
    })
})
.then(r => r.json())
.then(data => console.log('Search context:', data))
```

**Solutions**:

**Solution A: Rephrase Question**:
- Use more specific keywords
- Include document-specific terms
- Try broader search first, then narrow down

**Solution B: Verify Group Filter**:
```python
# In chatreadretrieveread.py, check filter logic
filter_expr = f"category eq '{user_group}'"  # Verify this matches document metadata
```

**Solution C: Rebuild Search Index**:
```powershell
# Reset indexer
az search indexer reset --service-name infoasst-search-dev2 --indexer-name text-enrichment-indexer --resource-group rg-infoasst-dev2

# Run indexer
az search indexer run --service-name infoasst-search-dev2 --indexer-name text-enrichment-indexer --resource-group rg-infoasst-dev2
```

### Issue 9: Citations Don't Match Answer

**Symptoms**:
- Answer seems correct but citations point to irrelevant documents
- Citation scores are low (<0.5)
- Multiple unrelated citations listed

**Diagnostic**:
This indicates vector search returning low-quality matches. Likely causes:
1. Insufficient documents in index
2. Embedding quality issues
3. Query optimization failure

**Solutions**:

**Solution A: Increase Top-K Results**:
```python
# In approaches config
overrides = {
    "top": 10,  # Increase from 5 to 10
    "retrieval_mode": "hybrid"  # Ensure hybrid search enabled
}
```

**Solution B: Enable Semantic Ranking**:
```bash
# In backend.env
USE_SEMANTIC_RERANKER=true
```

**Solution C: Check Embedding Quality**:
1. Verify text-embedding-ada-002 model deployed
2. Check embedding dimensions (should be 1536)
3. Verify chunks contain meaningful text (not just headers/footers)

---

## Performance Issues

### Issue 10: Slow API Response Times

**Symptoms**:
- Chat responses take >30 seconds
- Dashboard takes >10 seconds to load
- Timeout errors

**Diagnostic**:

```powershell
# Check Application Insights performance
az monitor app-insights metrics show `
    --app <app-insights-id> `
    --metric "requests/duration" `
    --aggregation avg `
    --start-time (Get-Date).AddHours(-1) `
    --end-time (Get-Date)
```

**Solutions**:

**Solution A: Scale Azure Search**:
```powershell
# Upgrade to Standard S1 or higher
az search service update `
    --name infoasst-search-dev2 `
    --resource-group rg-infoasst-dev2 `
    --sku Standard
```

**Solution B: Optimize Cosmos DB RU/s**:
```powershell
# Increase provisioned throughput
az cosmosdb sql container throughput update `
    --account-name infoasst-cosmos-dev2 `
    --database-name groupsToResourcesMap `
    --name groupResourcesMapContainer `
    --resource-group rg-infoasst-dev2 `
    --throughput 1000
```

**Solution C: Enable Application Caching**:
```python
# In app.py, increase cache TTL
CACHE_TTL = 600  # 10 minutes instead of 5
```

### Issue 11: High Cosmos DB Costs

**Symptoms**:
- Unexpected Azure bill increase
- Cosmos DB RU/s usage spikes
- 429 throttling errors

**Diagnostic**:
```powershell
# Check RU consumption
az monitor metrics list `
    --resource "/subscriptions/xxx/resourceGroups/rg-infoasst-dev2/providers/Microsoft.DocumentDB/databaseAccounts/infoasst-cosmos-dev2" `
    --metric "TotalRequestUnits" `
    --aggregation Total
```

**Solutions**:

**Solution A: Enable Autoscale**:
```powershell
az cosmosdb sql container throughput migrate `
    --account-name infoasst-cosmos-dev2 `
    --database-name groupsToResourcesMap `
    --name groupResourcesMapContainer `
    --resource-group rg-infoasst-dev2 `
    --throughput-type autoscale `
    --max-throughput 4000
```

**Solution B: Implement TTL**:
```python
# Set TTL on conversation history (delete after 90 days)
container_properties = {
    "id": "conversations",
    "defaultTtl": 7776000  # 90 days in seconds
}
```

**Solution C: Review Query Efficiency**:
- Ensure all queries use partition key
- Avoid cross-partition queries
- Use `SELECT` with specific fields (not `SELECT *`)

---

## Data Consistency Issues

### Issue 12: Group Changes Not Reflected

**Symptoms**:
- Admin adds user to group but user doesn't see it
- User switches group but old group still active
- Cosmos DB shows correct data but UI doesn't

**Root Cause**: Backend cache not refreshed

**Solutions**:

**Solution A: Restart Backend** (immediate fix):
```powershell
az webapp restart --name infoasst-web-dev2 --resource-group rg-infoasst-dev2
```

**Solution B: User Logout/Login**:
1. User logs out completely
2. Clears browser cache
3. Logs back in (fetches fresh JWT)

**Solution C: Reduce Cache TTL** (prevents future issues):
```python
# In app.py
CACHE_TTL = 300  # 5 minutes instead of default
```

### Issue 13: Document Count Mismatch

**Symptoms**:
- Dashboard shows 10 documents but search index has 8
- Upload succeeds but document never appears in search
- Deleted document still appears in chat results

**Diagnostic**:
```powershell
# Compare counts
# Blob storage
az storage blob list --container-name content-testrbac --account-name infoasststoragedev2 --auth-mode login | ConvertFrom-Json | Measure-Object

# Search index
az search index statistics --service-name infoasst-search-dev2 --index-name index-testrbac --resource-group rg-infoasst-dev2 --query "documentCount"

# Cosmos DB statuscontainer
az cosmosdb sql container query `
    --account-name infoasst-cosmos-dev2 `
    --database-name statusdb `
    --name statuscontainer `
    --query-string "SELECT VALUE COUNT(1) FROM c WHERE c.state = 'Indexed'"
```

**Solutions**:

**Solution A: Reindex Missing Documents**:
```powershell
# Reset indexer to reprocess all blobs
az search indexer reset --service-name infoasst-search-dev2 --indexer-name text-enrichment-indexer --resource-group rg-infoasst-dev2
az search indexer run --service-name infoasst-search-dev2 --indexer-name text-enrichment-indexer --resource-group rg-infoasst-dev2
```

**Solution B: Clean Up Orphaned Index Entries**:
```python
# Script to delete index entries without corresponding blob
from azure.search.documents import SearchClient

search_client = SearchClient(endpoint, index_name, credential)
results = search_client.search("*", select=["id", "sourcepage"])

for result in results:
    blob_path = result["sourcepage"]
    if not blob_exists(blob_path):
        search_client.delete_documents([{"id": result["id"]}])
```

---

## Environment-Specific Issues

### Dev2 Environment

**Issue 14: Mixed Public/Private Endpoints**

**Symptom**: Some services work, others return 403

**Cause**: Dev2 has public endpoints for development but some services configured for private access

**Solution**: Set fallback flags
```bash
# backend.env
LOCAL_DEBUG=true
OPTIMIZED_KEYWORD_SEARCH_OPTIONAL=true
ENRICHMENT_OPTIONAL=true
```

### HCCLD2 Environment

**Issue 15: All Services Return Connection Timeout**

**Symptom**: Cannot access any Azure services from local machine

**Cause**: HCCLD2 uses private endpoints only

**Solution**: Use VPN or DevBox
```powershell
# Test connectivity
Test-NetConnection -ComputerName infoasst-web-hccld2.azurewebsites.net -Port 443

# If fails, connect to VPN first
```

### Marco-Sandbox Environment

**Issue 16: Limited Azure Resources**

**Symptom**: 429 throttling or quota exceeded errors

**Cause**: Sandbox has lower SKUs/limits

**Solution**: Use dev2 for production-like testing

---

## Advanced Diagnostics

### Enable Debug Logging

**Backend**:
```python
# In app.py
import logging
logging.basicConfig(level=logging.DEBUG)

# Add to routes
@app.route("/chat")
async def chat():
    logger.debug(f"Request: {await request.get_json()}")
    # ... rest of code
```

**Frontend**:
```typescript
// In api.ts
const DEBUG = true;

export async function chatApi(request: ChatRequest): Promise<Response> {
    if (DEBUG) console.log('[DEBUG] Chat request:', request);
    const response = await fetch('/chat', {...});
    if (DEBUG) console.log('[DEBUG] Chat response:', response);
    return response;
}
```

### Capture Network Traces

**Browser DevTools**:
1. Open DevTools (F12)
2. Network tab
3. Reproduce issue
4. Right-click → Save all as HAR
5. Share HAR file with support

**PowerShell**:
```powershell
# Capture detailed HTTP trace
$ProgressPreference = 'SilentlyContinue'
$response = Invoke-WebRequest -Uri "https://infoasst-web-dev2.azurewebsites.net/chat" -Method POST -Body $body -ContentType "application/json" -Verbose
$response | ConvertTo-Json -Depth 10 | Out-File trace.json
```

---

## Getting Expert Help

### Before Contacting Support

Gather this information:
- [ ] Environment (dev2, hccld2, marco-sandbox)
- [ ] Your email address
- [ ] Group name and role
- [ ] Error message (exact text or screenshot)
- [ ] Steps to reproduce
- [ ] Timestamp of issue
- [ ] Browser and version
- [ ] Network trace (if API issue)

### Support Contacts

- **General Issues**: ServiceDesk@hrsdc-rhdcc.gc.ca
- **RBAC Configuration**: Marco Presta (marco.presta@hrsdc-rhdcc.gc.ca)
- **Infrastructure Issues**: Azure DevOps team
- **Emergency**: 24/7 ServiceDesk hotline

---

**Related Docs**: [README.md](README.md) | [ADMIN-GUIDE.md](ADMIN-GUIDE.md) | [USER-GUIDE.md](USER-GUIDE.md)
