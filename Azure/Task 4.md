# Task 4: Create Azure Virtual Network (VNet)

##  Overview

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations.

##  Task Requirements

Create an Azure Virtual Network with the following specifications:

| Requirement | Value |
|-------------|-------|
| **VNet Name** | `devops-vnet` |
| **Region** | `East US` |
| **IP Address Space** | Any IPv4 CIDR block |
| **Resource Group** | Use existing (provided by lab) |

##  Azure Credentials

> **Note:** Run `showcreds` command on the `azure-client` host to retrieve credentials

| Field | Value |
|-------|-------|
| Portal URL | https://portal.azure.com |
| Username | `kk_lab_user_main-56d7672fd1194c22@azurefreekmlprod.onmicrosoft.com` |
| Password | `&62VWS9$` |
| Start Time | Sat Jan 17 02:00:32 UTC 2026 |
| End Time | Sat Jan 17 03:00:32 UTC 2026 |

---

##  Execution Environment

All commands are executed in the **KodeKloud lab terminal** on the **azure-client host**.

---

##  Step-by-Step Solution

### Step 1: Retrieve Azure Credentials

Get the lab credentials using the KodeKloud helper command.

```bash
showcreds
```

**Expected output:**

```
Portal URL: https://portal.azure.com
Username: kk_lab_user_main-56d7672fd1194c22@azurefreekmlprod.onmicrosoft.com
Password: &62VWS9$
Start Time: Sat Jan 17 02:00:32 UTC 2026
End Time: Sat Jan 17 03:00:32 UTC 2026
```


**Purpose:**
- Displays temporary Azure credentials
- Shows session validity period
- Required for Azure authentication

> [!NOTE]
> Credentials are temporary and expire after the lab session ends.

---

### Step 2: Login to Azure CLI

Authenticate to Azure using device login flow.

```bash
az login
```

**What happens:**
- Azure CLI displays a device login URL (e.g., `https://microsoft.com/devicelogin`)
- Provides a unique authentication code

**Authentication steps:**
1. Open the displayed URL in your browser
2. Enter the authentication code
3. Sign in with credentials:
   - **Username:** `kk_lab_user_main-56d7672fd1194c22@azurefreekmlprod.onmicrosoft.com`
   - **Password:** `&62VWS9$`
4. Return to terminal - login completes automatically

**Expected output:**

```json
[
  {
    "cloudName": "AzureCloud",
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "isDefault": true,
    "name": "KodeKloud Subscription",
    "state": "Enabled",
    "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "user": {
      "name": "kk_lab_user_main-56d7672fd1194c22@azurefreekmlprod.onmicrosoft.com",
      "type": "user"
    }
  }
]
```



✅ **Authentication successful**

---

### Step 3: Verify Login and Active Subscription

Confirm successful authentication and check subscription details.

```bash
az account show
```

**Expected output:**

```json
{
  "environmentName": "AzureCloud",
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "isDefault": true,
  "name": "KodeKloud Subscription",
  "state": "Enabled",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "user": {
    "name": "kk_lab_user_main-56d7672fd1194c22@azurefreekmlprod.onmicrosoft.com",
    "type": "user"
  }
}
```

**Success indicators:**
- `"state": "Enabled"` ✅
- `"isDefault": true` ✅
- User matches your credentials ✅

**Alternative (table format):**

```bash
az account show --output table
```

**Expected output:**

```
Name                  CloudName    SubscriptionId                         State    IsDefault
--------------------  -----------  -------------------------------------  -------  -----------
KodeKloud Subscription  AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   Enabled  True
```



---

### Step 4: List Existing Resource Groups

Identify the pre-created resource group to use for the VNet.

```bash
az group list --output table
```

**Expected output:**

```
Name                              Location    Status
--------------------------------  ----------  ---------
kml_rg_main-56d7672fd1194c22      eastus      Succeeded
```


**Important notes:**
- 📌 **Copy the resource group name** - you'll need it for the next step
- ⚠️ **Do NOT create a new resource group** - KodeKloud provides one
- ✅ Always use the existing resource group

> [!IMPORTANT]
> KodeKloud labs do not allow creating new resource groups. You must use the pre-created one.

---

### Step 5: Create the Virtual Network ⭐

Create the VNet with the required specifications.

```bash
az network vnet create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --address-prefix 10.0.0.0/16 \
  --location eastus
```

> [!NOTE]
> Replace `<RESOURCE_GROUP>` with the actual resource group name from Step 4

**Example with actual resource group:**

```bash
az network vnet create \
  --resource-group kml_rg_main-56d7672fd1194c22 \
  --name devops-vnet \
  --address-prefix 10.0.0.0/16 \
  --location eastus
```

**Command Parameters Explained:**

| Parameter | Value | Purpose | Requirement Met |
|-----------|-------|---------|-----------------|
| `--resource-group` | Existing RG | Uses pre-created resource group | ✅ |
| `--name` | `devops-vnet` | VNet name | ✅ Required name |
| `--address-prefix` | `10.0.0.0/16` | IPv4 CIDR block (65,536 IPs) | ✅ IPv4 CIDR |
| `--location` | `eastus` | Azure region | ✅ East US region |

**Expected output:**

```json
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
    "id": "/subscriptions/.../resourceGroups/.../providers/Microsoft.Network/virtualNetworks/devops-vnet",
    "location": "eastus",
    "name": "devops-vnet",
    "provisioningState": "Succeeded",
    "resourceGroup": "kml_rg_main-56d7672fd1194c22",
    "subnets": [],
    "type": "Microsoft.Network/virtualNetworks"
  }
}
```



**What Azure does:**
- ✅ Creates the virtual network in East US region
- ✅ Allocates the IP address space (10.0.0.0/16)
- ✅ Sets up network infrastructure
- ✅ Returns VNet details and resource ID

> [!TIP]
> VNet creation typically completes in 10-30 seconds.

---

### Step 6: Verify VNet Creation

Confirm the VNet was created successfully with correct specifications.

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --output table
```

**Example:**

```bash
az network vnet show \
  --resource-group kml_rg_main-56d7672fd1194c22 \
  --name devops-vnet \
  --output table
```

**Expected output:**

```
Name         ResourceGroup                    Location    NumSubnets    ProvisioningState
-----------  -------------------------------  ----------  ------------  -------------------
devops-vnet  kml_rg_main-56d7672fd1194c22     eastus      0             Succeeded
```

**Alternative (detailed JSON view):**

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet
```



**Success indicators:**
- Name: `devops-vnet` ✅
- Location: `eastus` ✅
- ProvisioningState: `Succeeded` ✅
- Address space: `10.0.0.0/16` ✅

---

### Step 7: Verify IP Address Space

Check the configured IP address range.

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --query "addressSpace.addressPrefixes" \
  --output table
```

**Expected output:**

```
Result
-------------
10.0.0.0/16
```

**Verify address space details:**

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --query "{Name:name, Location:location, AddressSpace:addressSpace.addressPrefixes[0]}" \
  --output table
```

**Expected output:**

```
Name         Location    AddressSpace
-----------  ----------  --------------
devops-vnet  eastus      10.0.0.0/16
```

---

### Step 8: List All VNets (Optional Verification)

View all virtual networks in the resource group.

```bash
az network vnet list \
  --resource-group <RESOURCE_GROUP> \
  --output table
```

**Expected output:**

```
Name         ResourceGroup                    Location    NumSubnets    ProvisioningState
-----------  -------------------------------  ----------  ------------  -------------------
devops-vnet  kml_rg_main-56d7672fd1194c22     eastus      0             Succeeded
```

---

## ✅ Task Validation Checklist

Before submitting, verify all requirements are met:

- [ ] VNet name is exactly `devops-vnet`
- [ ] VNet is in `East US` region
- [ ] IPv4 CIDR block is configured (10.0.0.0/16)
- [ ] VNet is in `Succeeded` state
- [ ] Used existing resource group (not created new one)
- [ ] VNet is visible in `az network vnet list`

---

## 🎯 Final Verification Summary

| Requirement | Expected Value | Verification Command | Status |
|-------------|----------------|----------------------|--------|
| VNet Name | `devops-vnet` | `az network vnet show -n devops-vnet` | ✅ |
| Region | `East US` | Check `location` field | ✅ |
| IP Address Space | IPv4 CIDR (10.0.0.0/16) | Check `addressSpace.addressPrefixes` | ✅ |
| Provisioning State | `Succeeded` | Check `provisioningState` | ✅ |
| Resource Group | Existing (pre-created) | `az group list` | ✅ |

---

## 🔍 Understanding CIDR Blocks

### What is 10.0.0.0/16?

| Component | Meaning |
|-----------|---------|
| `10.0.0.0` | Network address (starting IP) |
| `/16` | Subnet mask (first 16 bits reserved) |
| **Address Range** | 10.0.0.0 - 10.0.255.255 |
| **Usable IPs** | 65,536 addresses |

### Why This CIDR Block?

- **Scalability**: Large enough for incremental migration
- **Subnetting**: Can be divided into multiple subnets
- **Private Range**: RFC 1918 private address space
- **Common Standard**: Widely used in Azure environments

### Alternative CIDR Blocks

You can use any valid IPv4 CIDR block:

| CIDR Block | Address Range | Total IPs | Use Case |
|------------|---------------|-----------|----------|
| `10.0.0.0/16` | 10.0.0.0 - 10.0.255.255 | 65,536 | **Recommended** - Large deployments |
| `10.0.0.0/24` | 10.0.0.0 - 10.0.0.255 | 256 | Small deployments |
| `172.16.0.0/16` | 172.16.0.0 - 172.16.255.255 | 65,536 | Alternative private range |
| `192.168.0.0/16` | 192.168.0.0 - 192.168.255.255 | 65,536 | Traditional private range |

---

## 🐛 Common Troubleshooting

### Error: "The resource group does not exist"

**Cause:** Wrong resource group name or not created

**Solution:**

```bash
# List all resource groups
az group list --output table

# Use the exact name shown in the list
```

---

### Error: "Location 'EastUS' is not valid"

**Cause:** Incorrect location format

**Solution:** Use lowercase: `eastus` not `EastUS` or `East US`

**Verify available locations:**

```bash
az account list-locations --output table | grep -i east
```

---

### Error: "AddressPrefixesOverlap"

**Cause:** CIDR block conflicts with existing VNet

**Solution:**

```bash
# List existing VNets and their address spaces
az network vnet list \
  --resource-group <RESOURCE_GROUP> \
  --query "[].{Name:name, AddressSpace:addressSpace.addressPrefixes}" \
  --output table

# Use a different CIDR block
```

---

### Error: "InvalidAddressPrefix"

**Cause:** Invalid CIDR notation

**Solution:** Ensure proper CIDR format:
- ✅ Valid: `10.0.0.0/16`, `172.16.0.0/16`, `192.168.0.0/24`
- ❌ Invalid: `10.0.0.0`, `10.0.0.0/33`, `256.0.0.0/16`

---

### VNet Created but Not Showing

**Cause:** Wrong resource group or subscription

**Solution:**

```bash
# Verify correct subscription is active
az account show

# Search across all resource groups
az network vnet list --output table
```

---

## 📊 Advanced VNet Operations

### Add Subnet to VNet

Create a subnet within the VNet for future VMs:

```bash
az network vnet subnet create \
  --resource-group <RESOURCE_GROUP> \
  --vnet-name devops-vnet \
  --name default-subnet \
  --address-prefix 10.0.1.0/24
```

---

### View VNet Details in JSON

Get complete VNet configuration:

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --output json
```

---

### Update VNet DNS Servers

Configure custom DNS servers:

```bash
az network vnet update \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --dns-servers 8.8.8.8 8.8.4.4
```

---

### Check VNet Peering Options

View peering configuration (for connecting VNets):

```bash
az network vnet peering list \
  --resource-group <RESOURCE_GROUP> \
  --vnet-name devops-vnet \
  --output table
```

---

## 🔐 Network Security Considerations

> [!WARNING]
> **Important Security Practices**

### VNet Security Best Practices:

**Network Segmentation:**
- ✅ Use subnets to isolate workloads
- ✅ Implement Network Security Groups (NSGs)
- ✅ Apply least privilege access principles

**Address Planning:**
- ✅ Plan CIDR blocks to avoid overlap
- ✅ Reserve address space for future growth
- ✅ Document IP allocation strategy

**Monitoring:**
- ✅ Enable Network Watcher
- ✅ Configure flow logs
- ✅ Set up alerts for unusual activity

**Connectivity:**
- ✅ Use VNet peering for cross-VNet communication
- ✅ Implement VPN or ExpressRoute for hybrid scenarios
- ✅ Configure service endpoints for Azure services

---

## 🧹 Cleanup (Optional)

To delete the VNet after completing the task:

### Delete VNet

```bash
az network vnet delete \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet
```

### Verify Deletion

```bash
az network vnet list \
  --resource-group <RESOURCE_GROUP> \
  --output table
```

> [!CAUTION]
> Deleting a VNet will also delete all associated resources (subnets, peerings). Ensure no resources are dependent on this VNet before deletion.

---

## 📚 Additional Resources

- [Azure Virtual Network Documentation](https://docs.microsoft.com/azure/virtual-network/)
- [Azure CLI VNet Commands](https://docs.microsoft.com/cli/azure/network/vnet)
- [Plan Virtual Networks](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)
- [CIDR Calculator](https://www.ipaddressguide.com/cidr)
- [RFC 1918 - Private Address Space](https://tools.ietf.org/html/rfc1918)

