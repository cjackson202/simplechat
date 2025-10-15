# SimpleChat Azure OpenAI Load Test

## Overview

This JMeter load test is designed to test the SimpleChat application's Azure OpenAI integration with support for both Azure Commercial and Government clouds, focusing on the actual `gpt-4o` deployment used by the application. The test uses JMeter's built-in `${__GetSecret()}` function for secure Key Vault integration.

## Features

- ✅ **Azure Load Testing Integration** - Designed for Azure Load Testing with Application Settings
- ✅ **Secure Configuration** - No hardcoded secrets or sensitive data in the test plan
- ✅ **Azure Government Support** - Automatically configures endpoints for Azure Government
- ✅ **Key Vault Integration** - Uses JMeter's built-in `${__GetSecret()}` function for secure authentication
- ✅ **Model Deployment Testing** - Tests actual `gpt-4o` deployment used by the application
- ✅ **Environment-Based Configuration** - Easy to manage different environments (dev, staging, prod)
- ✅ **Realistic User Behavior** - Includes think time and multiple conversation patterns
- ✅ **Comprehensive Reporting** - Multiple result collectors for detailed analysis

## Prerequisites

### Azure Environment Setup
1. **Managed Identity** - JMeter must run on an Azure resource with managed identity enabled (VM, Container Instance, etc.)
2. **Key Vault Access** - The managed identity must have "Get" permissions on the Key Vault
3. **App Registration** - Must have client credentials configured in Key Vault

### Required Configuration

#### Application Settings Configuration
Configure these as **Application Settings** in your Azure Load Testing resource:

| Name | Value | Notes |
|------|-------|-------|
| `APP_URL` | `your-app.azurewebsites.net` | SimpleChat app domain (without https://) |
| `TENANT_ID` | `your-tenant-guid` | Azure AD tenant ID |
| `CLIENT_ID` | `your-client-guid` | App registration client ID |
| `CLIENT_SECRET` | `your-client-secret` | App registration client secret |
| `AZURE_ENVIRONMENT` | `public` or `usgovernment` | Optional, defaults to public |
| `MODEL_DEPLOYMENT_NAME` | `gpt-4o` | Optional, defaults to gpt-4o |
| `THREADS` | `25` | Optional, defaults to 10 |
| `DURATION` | `1800` | Optional, defaults to 300 seconds |
| `RAMP_UP` | `300` | Optional, defaults to 60 seconds |

### How to Configure:

#### Step 1: Create Key Vault Secrets
```bash
# Create the required secrets in your Key Vault
az keyvault secret set --vault-name "your-keyvault" --name "app-url" --value "your-app.azurewebsites.net"
az keyvault secret set --vault-name "your-keyvault" --name "tenant-id" --value "your-tenant-guid"
az keyvault secret set --vault-name "your-keyvault" --name "client-id" --value "your-client-guid"
az keyvault secret set --vault-name "your-keyvault" --name "client-secret" --value "your-client-secret"
```

#### Step 2: Configure Azure Load Testing Application Settings
1. **Navigate to your Azure Load Testing resource**
2. **Go to Settings → Application settings**
3. **Add only the KEY_VAULT_NAME and optional configuration:**

| Name | Value |
|------|-------|
| `KEY_VAULT_NAME` | `your-keyvault` |
| `AZURE_ENVIRONMENT` | `public` or `usgovernment` |
| `THREADS` | `25` (optional) |
| `DURATION` | `1800` (optional) |
## Azure Government Configuration

For Azure Government environments:
1. Set `AZURE_ENVIRONMENT=usgovernment`
2. The test automatically configures:
   - Login endpoint: `login.microsoftonline.us`
   - Key Vault suffix: `.vault.usgovcloudapi.net`
   - Graph scope: `https://graph.microsoft.us/.default`

## Test Scenarios

The load test includes three main scenarios:

1. **Basic Chat Request** - Random message with standard parameters
2. **Complex Analytical Query** - Long-form question with hybrid search enabled  
3. **Follow-up Question** - Conversation continuity test

## Authentication Flow

1. **Key Vault Setup** - Configures JMeter's `${__GetSecret()}` function
2. **Azure AD Token** - Obtains access token using client credentials
3. **Application Access** - Authenticates with SimpleChat application
4. **Chat API Testing** - Performs load testing on `/api/chat` endpoint

## Running the Test

### Azure Load Testing (Recommended)
1. **Upload `load.jmx`** to your Azure Load Testing resource
2. **Configure Application Settings** as described above
3. **Run the test** - All settings are automatically loaded from environment variables

### Local Development/Testing
If running locally, you can set environment variables:

#### Command Line (Windows)
```cmd
set APP_URL=your-app.azurewebsites.net
set TENANT_ID=your-tenant-id
set CLIENT_ID=your-client-id
set KEY_VAULT_NAME=your-keyvault-name
jmeter -n -t load.jmx -l results.jtl
```

#### Command Line (Linux/Mac)
```bash
export APP_URL=your-app.azurewebsites.net
export TENANT_ID=your-tenant-id
export CLIENT_ID=your-client-id
export KEY_VAULT_NAME=your-keyvault-name
jmeter -n -t load.jmx -l results.jtl
```

#### GUI Mode (Development)
```bash
jmeter -t load.jmx
```
*Note: For GUI mode, you may need to set environment variables in your system or use JMeter's System Properties.*

## Monitoring and Results

The test includes multiple result collectors:
- **View Results Tree** - Individual request/response details
- **Summary Report** - Aggregate statistics saved to `azure-openai-load-test-results.jtl`
- **Graph Results** - Real-time performance graphs
- **Response Times Over Time** - Trend analysis

## Load Characteristics

### Current Configuration (Development)
- **Users**: 10 concurrent
- **Duration**: 5 minutes
- **Expected Load**: ~60-120 requests per minute
- **Token Usage**: ~45,000-90,000 tokens
- **Estimated Cost**: $0.45-$0.90 per test

### Production Simulation Recommendations
- **Small Enterprise**: 25-50 users, 30 minutes
- **Medium Enterprise**: 75-150 users, 1 hour  
- **Large Enterprise**: 200+ users, 2+ hours

## Troubleshooting

### Key Vault Access Issues
- Verify managed identity has Key Vault "Get" permission
- Check Key Vault firewall settings
- Ensure secret name matches exactly: `MICROSOFT-PROVIDER-AUTHENTICATION-SECRET`

### Azure Government Issues
- Verify `AZURE_ENVIRONMENT=usgovernment` is set correctly
- Check that all Azure resources are in the government cloud
- Validate tenant ID is for the government cloud tenant

### Authentication Failures
- Verify app registration has correct permissions
- Check client ID and tenant ID are correct
- Ensure client secret in Key Vault is current and valid

### High Token Costs
- Consider using `gpt-35-turbo` for volume testing
- Reduce `THREADS` and `DURATION` for cost control
- Monitor Azure OpenAI usage quotas and rate limits

## Cost Management

⚠️ **Important**: This test can consume significant Azure OpenAI tokens and incur costs.

**Recommendations**:
- Start with default settings (10 users, 5 minutes)
- Monitor costs in Azure portal during testing
- Set spending alerts before large-scale tests
- Use staging/test environments when possible


- **JMeter Version**: Keep JMeter updated to latest stable version
- **Test Scenarios**: Update test scenarios as application features evolve
- **Performance Baselines**: Adjust expected performance metrics as needed
- **Authentication**: Update authentication flows as Azure AD configuration changes

### Monitoring Integration
- **Application Insights**: Monitor correlation with application telemetry
- **Azure Monitor**: Track infrastructure metrics during tests
- **Log Analytics**: Correlate test results with application logs

## Support and Documentation

### Related Documentation
- [SimpleChat Admin Configuration](../docs/reference/admin_configuration.md)
- [Azure OpenAI Integration Guide](../docs/setup_instructions_manual.md)
- [Authentication Setup](../docs/setup_instructions_manual.md#app-registration)

### Support Resources
- **JMeter Documentation**: https://jmeter.apache.org/usermanual/index.html
- **Azure Load Testing**: https://docs.microsoft.com/en-us/azure/load-testing/
- **Azure OpenAI Service**: https://docs.microsoft.com/en-us/azure/cognitive-services/openai/

## Version History

- **0.230.002**: Initial GPT-5 load testing implementation with Azure Key Vault integration