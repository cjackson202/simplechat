## ✅ Steps to Configure Azure AD App for Token Retrieval and Web App Access

> **Note**: Replace all placeholder values (in `{BRACKETS}`) with your actual Azure resource values.

### **Prerequisites**
- Azure AD App Registration created
- App Service deployed
- Appropriate permissions to modify Azure AD and App Service settings

### **Required Values**
Before starting, gather these values from your Azure environment:
- `{YOUR_APP_REGISTRATION_ID}`: Found in Azure Portal → Microsoft Entra ID → App registrations → [Your App] → Overview → Application (client) ID
- `{YOUR_TENANT_ID}`: Found in Azure Portal → Microsoft Entra ID → Overview → Tenant ID
- `{YOUR_CLIENT_SECRET}`: Created in Azure Portal → Microsoft Entra ID → App registrations → [Your App] → Certificates & secrets
- `{YOUR_APP_ROLE_ID}`: Retrieved from step 4 below using Azure CLI

### **1️⃣ Expose the API**

1. Go to **Microsoft Entra ID → App registrations →** your app (`{YOUR_APP_REGISTRATION_ID}`).
2. Under **Expose an API**, click **Set** next to “Application ID URI”.

   * Set it to:

     ```
     api://{YOUR_APP_REGISTRATION_ID}
     ```
3. This step allows the app to represent an API resource that other apps can call using tokens.

4. Click “Add a scope”

5. If prompted to set the Application ID URI, confirm it (should be `api://{YOUR_APP_REGISTRATION_ID}`)

6. Fill out:

    - **Scope name**: `access_as_app` (or anything descriptive)

    - **Who can consent**: Admins only (since this is client credentials)

    - **Admin consent display name**: `Access this API as an application`

    - **Admin consent description**: `Allows the calling app to access this API via client credentials`

    - **State**: Enabled

7. Click **Add scope**

✅ Now your app officially exposes an API scope:

`api://{YOUR_APP_REGISTRATION_ID}/access_as_app`

---

### **2️⃣ Configure Allowed Token Audiences in App Service**

1. Go to your **App Service** in the Azure portal.
2. Navigate to **Authentication → Allowed Token Audiences**.
3. Add the **App ID URI** you set in the previous step:

   ```
   api://{YOUR_APP_REGISTRATION_ID}
   ```
4. Save and **restart the app** to apply the changes.

> This ensures the web app only accepts tokens intended for its API audience (`aud` claim).

---

### **3️⃣ Create an App Role**

1. Still under your app registration, go to **App roles → + Create app role**.
2. Fill in the following details:

   * **Display name:** `App.Access`
   * **Allowed member types:** `Applications`
   * **Value:** `App.Access`
   * **Description:** `Allows the app to access this API as itself`
3. Save the new role.

This defines the app-only permission that can be assigned and granted later.

---

### **4️⃣ Get the App Role ID**

Retrieve the new role’s ID using the Azure CLI or Cloud Shell:

```bash
APP_REGISTRATION_ID="{YOUR_APP_REGISTRATION_ID}"

az ad app show --id $APP_REGISTRATION_ID --query "appRoles" -o json
```

From the JSON output, note the `"id"` of the `App.Access` role:

```bash
APP_ROLE_ID="{YOUR_APP_ROLE_ID}"
```

---

### **5️⃣ Assign and Grant the Role Permission**

Authorize the app to call itself:

```bash
az ad app permission add \
  --id $APP_REGISTRATION_ID \
  --api $APP_REGISTRATION_ID \
  --api-permissions $APP_ROLE_ID=Role
```

Grant the permission:

```bash
az ad app permission grant \
  --id $APP_REGISTRATION_ID \
  --api $APP_REGISTRATION_ID \
  --scope ".default"
```

---

### **6️⃣ Test Token Retrieval**

Choose the appropriate endpoint for your Azure environment:

**For Azure Government:**
```bash
curl -X POST "https://login.microsoftonline.us/{YOUR_TENANT_ID}/oauth2/v2.0/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "client_id={YOUR_APP_REGISTRATION_ID}&client_secret={YOUR_CLIENT_SECRET}&scope=api://{YOUR_APP_REGISTRATION_ID}/.default&grant_type=client_credentials"
```

**For Azure Commercial:**
```bash
curl -X POST "https://login.microsoftonline.com/{YOUR_TENANT_ID}/oauth2/v2.0/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "client_id={YOUR_APP_REGISTRATION_ID}&client_secret={YOUR_CLIENT_SECRET}&scope=api://{YOUR_APP_REGISTRATION_ID}/.default&grant_type=client_credentials"
```

This returns a valid `access_token`, confirming that the configuration worked.

---

### ✅ **Summary**

* **Expose API** → Created an API URI (`api://{YOUR_APP_REGISTRATION_ID}`)
* **Allowed Token Audiences** → Added the API URI to the App Service so it trusts tokens
* **Create App Role** → Defined an app-only permission (`App.Access`)
* **Get Role ID** → Used `az ad app show` to retrieve the role ID
* **Assign & Grant Role** → Gave the app permission to access its own API
* **Test** → Verified successful token retrieval using `curl` with appropriate Azure endpoint

### **Environment Endpoints**
- **Azure Government**: `https://login.microsoftonline.us/`
- **Azure Commercial**: `https://login.microsoftonline.com/`