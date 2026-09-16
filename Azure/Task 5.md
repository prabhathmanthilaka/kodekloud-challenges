# Task 5: Create Azure VNet with Specific CIDR Block

##  Overview

The Nautilus DevOps team is strategically planning infrastructure migration to Azure in incremental phases. Virtual Networks (VNets) form the foundation, as various services will be provisioned under different VNets. This task focuses on creating a VNet with a specific CIDR block for controlled IP allocation.

##  Task Requirements

Create an Azure Virtual Network with the following specifications:

| Requirement | Value |
|-------------|-------|
| **VNet Name** | `devops-vnet` |
| **Region** | `East US` |
| **IP Address Space** | `192.168.0.0/24` (Specific CIDR) |
| **Resource Group** | Use existing (provided by lab) |

##  Azure Credentials

> **Note:** Run `showcreds` command on the `azure-client` host to retrieve credentials

| Field | Value |
|-------|-------|
| Portal URL | https://portal.azure.com |
| Username | `kk_lab_user_main-fb42f1b00d444b3b@azurefreekmlprod.onmicrosoft.com` |
| Password | `a7eh4v@j` |
| Start Time | Sun Jan 18 01:32:18 UTC 2026 |
| End Time | Sun Jan 18 02:32:18 UTC 2026 |

---

##  Execution Environment

All commands are executed in the **KodeKloud lab terminal** on the **azure-client host**.

---

##  Understanding the CIDR Block: 192.168.0.0/24

### Quick Reference

| Component | Value | Meaning |
|-----------|-------|---------|
| **Network Address** | `192.168.0.0` | Base IP address |
| **Prefix Length** | `/24` | First 24 bits are network portion |
| **Subnet Mask** | `255.255.255.0` | Dotted decimal notation |
| **Total IPs** | 256 | 2^(32-24) = 2^8 |
| **Usable IPs (Azure)** | 251 | 256 - 5 (Azure reserved) |
| **IP Range** | `192.168.0.0 - 192.168.0.255` | Complete address space |
| **First Usable** | `192.168.0.4` | After Azure reserved IPs |
| **Last Usable** | `192.168.0.254` | Before broadcast |

### Azure Reserved IPs in 192.168.0.0/24

```
192.168.0.0     ← Network address (reserved)
192.168.0.1     ← Azure default gateway (reserved)
192.168.0.2     ← Azure DNS mapping (reserved)
192.168.0.3     ← Azure DNS mapping (reserved)
192.168.0.4     ← First usable IP for resources
192.168.0.5
...
192.168.0.254   ← Last usable IP for resources
192.168.0.255   ← Broadcast address (reserved)
```

### Why 192.168.0.0/24?

**Advantages:**
- ✅ Standard private IP range (RFC 1918)
- ✅ 256 IPs suitable for small to medium deployments
- ✅ Easy to remember and manage
- ✅ Common in enterprise networks
- ✅ No subnet overlap with typical home networks (192.168.1.x)

**Limitations:**
- ⚠️ Only 251 usable IPs in Azure
- ⚠️ Cannot expand without changing CIDR
- ⚠️ No room for multiple large subnets

> [!NOTE]
> This CIDR block is perfect for controlled deployments where you want to limit the IP space intentionally.

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
Username: kk_lab_user_main-fb42f1b00d444b3b@azurefreekmlprod.onmicrosoft.com
Password: a7eh4v@j
Start Time: Sun Jan 18 01:32:18 UTC 2026
End Time: Sun Jan 18 02:32:18 UTC 2026
```



**Purpose:**
- Displays temporary Azure credentials
- Shows session validity (1-hour window)
- Required for Azure authentication

> [!WARNING]
> Credentials expire after the session ends. Complete the task within the time window.

---

### Step 2: Login to Azure CLI

Authenticate to Azure using device login flow.

```bash
az login
```

**What happens:**
- Azure CLI displays a device login URL: `https://microsoft.com/devicelogin`
- Provides a unique authentication code (e.g., `ABCD1234`)

**Authentication steps:**
1. Open the displayed URL in your browser
2. Enter the authentication code shown in terminal
3. Sign in with credentials:
   - **Username:** `kk_lab_user_main-fb42f1b00d444b3b@azurefreekmlprod.onmicrosoft.com`
   - **Password:** `a7eh4v@j`
4. Return to terminal - login completes automatically

**Expected output:**

```json
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "isDefault": true,
    "managedByTenants": [],
    "name": "KodeKloud Subscription",
    "state": "Enabled",
    "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "user": {
      "name": "kk_lab_user_main-fb42f1b00d444b3b@azurefreekmlprod.onmicrosoft.com",
      "type": "user"
    }
  }
]
```


✅ **Authentication successful - Ready to create resources**

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
  "homeTenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "isDefault": true,
  "managedByTenants": [],
  "name": "KodeKloud Subscription",
  "state": "Enabled",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "user": {
    "name": "kk_lab_user_main-fb42f1b00d444b3b@azurefreekmlprod.onmicrosoft.com",
    "type": "user"
  }
}
```

**Success indicators:**
- `"state": "Enabled"` ✅
- `"isDefault": true` ✅
- User matches your credentials ✅

**Alternative (table format for quick view):**

```bash
az account show --output table
```

**Expected output:**

```
Name                     CloudName    SubscriptionId                         State    IsDefault
-----------------------  -----------  -------------------------------------  -------  -----------
KodeKloud Subscription   AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   Enabled  True
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
kml_rg_main-fb42f1b00d444b3b      eastus      Succeeded
```


**Important notes:**
- 📌 **Copy the exact resource group name** - you'll need it for the next step
- ⚠️ **Do NOT create a new resource group** - KodeKloud provides one
- ✅ The resource group should already exist in `eastus` location

> [!IMPORTANT]
> KodeKloud labs do not permit creating new resource groups. You must use the pre-created one. Attempting to create a new resource group will result in an error.

**Alternative command (detailed JSON view):**

```bash
az group list
```

---

### Step 5: Create the Virtual Network ⭐

Create the VNet with the specific CIDR block requirement.

```bash
az network vnet create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --address-prefix 192.168.0.0/24 \
  --location eastus
```

> [!NOTE]
> Replace `<RESOURCE_GROUP>` with the actual resource group name from Step 4

**Example with actual resource group:**

```bash
az network vnet create \
  --resource-group kml_rg_main-fb42f1b00d444b3b \
  --name devops-vnet \
  --address-prefix 192.168.0.0/24 \
  --location eastus
```

**Command Parameters Explained:**

| Parameter | Value | Purpose | Requirement Met |
|-----------|-------|---------|-----------------|
| `--resource-group` | Existing RG | Uses pre-created resource group | ✅ |
| `--name` | `devops-vnet` | VNet name | ✅ Required name |
| `--address-prefix` | `192.168.0.0/24` | IPv4 CIDR block (256 IPs) | ✅ Specific CIDR |
| `--location` | `eastus` | Azure region | ✅ East US region |

**Expected output:**

```json
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "192.168.0.0/24"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
    "id": "/subscriptions/.../resourceGroups/kml_rg_main-fb42f1b00d444b3b/providers/Microsoft.Network/virtualNetworks/devops-vnet",
    "location": "eastus",
    "name": "devops-vnet",
    "provisioningState": "Succeeded",
    "resourceGroup": "kml_rg_main-fb42f1b00d444b3b",
    "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "subnets": [],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}
```


**What Azure does:**
- ✅ Creates virtual network in East US region
- ✅ Allocates the IP address space (192.168.0.0/24)
- ✅ Reserves 5 IPs for Azure services
- ✅ Makes 251 IPs available for resources
- ✅ Sets provisioning state to "Succeeded"
- ✅ Returns VNet resource ID and details

> [!TIP]
> VNet creation typically completes in 10-30 seconds. The command will wait until the operation finishes.

**What "default subnet automatically created" means:**

Azure may create a default subnet, but you can also create custom subnets later. Check with:

```bash
az network vnet subnet list \
  --resource-group <RESOURCE_GROUP> \
  --vnet-name devops-vnet \
  --output table
```

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
  --resource-group kml_rg_main-fb42f1b00d444b3b \
  --name devops-vnet \
  --output table
```

**Expected output:**

```
Name         ResourceGroup                     Location    NumSubnets    ProvisioningState    ResourceGuid
-----------  --------------------------------  ----------  ------------  -------------------  ------------------------------------
devops-vnet  kml_rg_main-fb42f1b00d444b3b      eastus      0             Succeeded            xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```



**Success indicators:**
- Name: `devops-vnet` ✅
- Location: `eastus` ✅
- ProvisioningState: `Succeeded` ✅

**Alternative (detailed JSON view):**

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet
```

---

### Step 7: Verify IP Address Space

Confirm the CIDR block is exactly `192.168.0.0/24`.

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
----------------
192.168.0.0/24
```

**Comprehensive verification:**

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet \
  --query "{Name:name, Location:location, CIDR:addressSpace.addressPrefixes[0], State:provisioningState}" \
  --output table
```

**Expected output:**

```
Name         Location    CIDR             State
-----------  ----------  ---------------  ---------
devops-vnet  eastus      192.168.0.0/24   Succeeded
```

✅ **All requirements verified!**

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
Name         ResourceGroup                     Location    NumSubnets    ProvisioningState
-----------  --------------------------------  ----------  ------------  -------------------
devops-vnet  kml_rg_main-fb42f1b00d444b3b      eastus      0             Succeeded
```

---

## ✅ Task Validation Checklist

Before submitting, verify all requirements are met:

- [ ] VNet name is exactly `devops-vnet`
- [ ] VNet is in `East US` region
- [ ] IPv4 CIDR block is exactly `192.168.0.0/24`
- [ ] VNet provisioning state is `Succeeded`
- [ ] Used existing resource group (not created new one)
- [ ] VNet is visible in `az network vnet list`
- [ ] Address space verified with query command

---

##  Final Verification Summary

| Requirement | Expected Value | Verification Command | Status |
|-------------|----------------|----------------------|--------|
| VNet Name | `devops-vnet` | `az network vnet show -n devops-vnet` | ✅ |
| Region | `East US` | Check `location` field | ✅ |
| CIDR Block | `192.168.0.0/24` | Check `addressSpace.addressPrefixes` | ✅ |
| Total IPs | 256 | Calculated from /24 | ✅ |
| Usable IPs (Azure) | 251 | 256 - 5 reserved | ✅ |
| Subnet Mask | `255.255.255.0` | CIDR /24 equivalent | ✅ |
| Provisioning State | `Succeeded` | Check `provisioningState` | ✅ |
| Resource Group | Existing (pre-created) | `az group list` | ✅ |

---

## 🔍 Deep Dive: 192.168.0.0/24 Analysis

### Binary Representation

```
IP Address: 192.168.0.0
Binary: 11000000.10101000.00000000.00000000

Subnet Mask: /24
Binary: 11111111.11111111.11111111.00000000
Decimal: 255.255.255.0

Network Portion: First 24 bits (192.168.0)
Host Portion: Last 8 bits (0-255)
```

### Complete IP Range Breakdown

| IP Address | Purpose | Available for VMs? |
|------------|---------|-------------------|
| `192.168.0.0` | Network address | ❌ Reserved |
| `192.168.0.1` | Azure default gateway | ❌ Reserved |
| `192.168.0.2` | Azure DNS | ❌ Reserved |
| `192.168.0.3` | Azure DNS | ❌ Reserved |
| `192.168.0.4` - `192.168.0.254` | **Usable IPs** | ✅ Available (251 IPs) |
| `192.168.0.255` | Broadcast address | ❌ Reserved |

### Capacity Planning

**What can you deploy in 192.168.0.0/24?**

| Scenario | Resources | IPs Needed | Fits? |
|----------|-----------|------------|-------|
| Small web app | 5 VMs + 1 Load Balancer | 6 | ✅ Yes |
| Medium deployment | 50 VMs + 5 services | 55 | ✅ Yes |
| Large deployment | 200 VMs + 10 services | 210 | ✅ Yes |
| Maximum capacity | 251 VMs (single NIC each) | 251 | ✅ Exactly fits |
| Over-provisioned | 300 VMs | 300 | ❌ Need larger CIDR |

**Subnetting options within 192.168.0.0/24:**

```
Option 1: Two /25 subnets
├─ Subnet A: 192.168.0.0/25   (128 IPs, 123 usable)
└─ Subnet B: 192.168.0.128/25 (128 IPs, 123 usable)

Option 2: Four /26 subnets
├─ Subnet A: 192.168.0.0/26   (64 IPs, 59 usable)
├─ Subnet B: 192.168.0.64/26  (64 IPs, 59 usable)
├─ Subnet C: 192.168.0.128/26 (64 IPs, 59 usable)
└─ Subnet D: 192.168.0.192/26 (64 IPs, 59 usable)

Option 3: Eight /27 subnets
├─ Subnet 1-8: Each has 32 IPs, 27 usable
```

---

## 🐛 Common Troubleshooting

### Error: "The specified address prefix is invalid"

**Cause:** Incorrect CIDR format

**Solutions:**

```bash
# ❌ Wrong formats:
192.168.0.0/24/     # Extra slash
192.168.0.1/24      # Not a network address
192.168.0.0/33      # Prefix too large
192.168.0.0 /24     # Space in CIDR

# ✅ Correct format:
192.168.0.0/24
```

**Verification:**

```bash
# Test if IP is a valid network address
# For /24, last octet must be 0
192.168.0.0/24  ✅
192.168.0.128/24 ❌ (should be /25)
```

---

### Error: "AddressPrefix overlaps with existing VNet"

**Cause:** CIDR block conflicts with another VNet

**Check for conflicts:**

```bash
az network vnet list \
  --resource-group <RESOURCE_GROUP> \
  --query "[].{Name:name, CIDR:addressSpace.addressPrefixes}" \
  --output table
```

**Solution:**

```bash
# If 192.168.0.0/24 conflicts, use:
192.168.1.0/24  # Different third octet
192.168.2.0/24
# Or completely different range:
10.0.0.0/24
172.16.0.0/24
```

---

### Error: "Location 'EastUS' is not valid"

**Cause:** Incorrect location format

**Solution:**

```bash
# ❌ Wrong:
--location EastUS
--location "East US"
--location EASTUS

# ✅ Correct:
--location eastus
```

**Verify available locations:**

```bash
az account list-locations \
  --query "[?displayName=='East US'].name" \
  --output table
```

---

### Warning: "VNet has no subnets"

**Not an error** - This is expected. VNet created without explicit subnets.

**To add a subnet:**

```bash
az network vnet subnet create \
  --resource-group <RESOURCE_GROUP> \
  --vnet-name devops-vnet \
  --name default-subnet \
  --address-prefix 192.168.0.0/25
```

**This divides the /24 into usable subnets:**

```
VNet: 192.168.0.0/24 (256 IPs)
└─ Subnet: 192.168.0.0/25 (128 IPs, 123 usable)
   └─ Reserved: 192.168.0.128/25 (128 IPs for future)
```

---

### Issue: Cannot SSH to VMs in this VNet

**Cause:** Network Security Group (NSG) blocking SSH

**Solution:**

```bash
# Create NSG allowing SSH
az network nsg create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-nsg

# Add SSH rule
az network nsg rule create \
  --resource-group <RESOURCE_GROUP> \
  --nsg-name devops-nsg \
  --name AllowSSH \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 22
```

---

##  Comparing CIDR Sizes

### Why 192.168.0.0/24 vs Other Options?

| CIDR | Total IPs | Usable (Azure) | Use Case | Pros | Cons |
|------|-----------|----------------|----------|------|------|
| `/16` | 65,536 | 65,531 | Large VNets | Scalable, flexible | Wastes IPs for small deployments |
| `/20` | 4,096 | 4,091 | Medium VNets | Good balance | May need expansion |
| `/24` | 256 | 251 | **Small-Medium** | **Controlled, manageable** | **Limited expansion** |
| `/27` | 32 | 27 | Tiny subnets | Minimal waste | Too small for most uses |
| `/30` | 4 | 2 | Point-to-point | Efficient for links | Cannot host VMs |

**Task's choice (`/24`) is ideal for:**
- ✅ Development environments
- ✅ Small production workloads
- ✅ Controlled deployments
- ✅ Learning/testing Azure networking
- ✅ Environments with known, limited size

---

##  Security Best Practices

### Network Isolation

**Use this VNet for:**
```
Development Environment:
└─ 192.168.0.0/24
   ├─ Web Servers: 192.168.0.0/26
   ├─ App Servers: 192.168.0.64/26
   └─ DB Servers: 192.168.0.128/26
```

**Keep separate VNets for:**
- Production (different CIDR)
- Staging (different CIDR)
- Management (different CIDR)

### Network Security Groups

**Recommended NSG structure:**

```bash
# Create NSG for this VNet
az network nsg create \
  --resource-group <RESOURCE_GROUP> \
  --name devops-vnet-nsg \
  --location eastus

# Allow only necessary ports
# Example: Web tier (HTTP/HTTPS only)
az network nsg rule create \
  --resource-group <RESOURCE_GROUP> \
  --nsg-name devops-vnet-nsg \
  --name AllowHTTP \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80 443
```

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
> - Deleting a VNet will remove all associated resources (subnets, peerings)
> - Ensure no VMs or services are using this VNet before deletion
> - Deletion is **irreversible**

---

## 📚 Additional Resources

- [Azure Virtual Network Documentation](https://docs.microsoft.com/azure/virtual-network/)
- [Azure CIDR Best Practices](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq)
- [Plan Virtual Networks](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)
- [CIDR Calculator Tool](https://www.ipaddressguide.com/cidr)
- [RFC 1918 - Private Address Space](https://tools.ietf.org/html/rfc1918)
- [Complete CIDR Guide](../docs/CIDR-Complete-Guide.md) - Internal reference

