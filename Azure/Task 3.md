# Task 3: Create Azure Virtual Machine via CLI

##  Overview

The Nautilus DevOps team is in the process of migrating some of their workloads to Azure. One of the tasks involves creating a new Virtual Machine (VM) using the Azure CLI. The team does not have access to the Azure portal but can manage Azure resources via the `azure-client` host (the landing host for this lab).

1) Create a new Azure Virtual Machine named `datacenter-vm` using the Azure CLI.

2) Use the `Ubuntu2204` image and set the VM size to `Standard_B2s`.

3) Make sure the admin username is set to `azureuser` and SSH keys are generated for secure access.

4) Use `Standard_LRS` storage account, disk size must be `30GB` and ensure the VM `datacenter-vm` is in the `running` state after creation.
5) 
##  Task Requirements

Create an Azure Virtual Machine with the following specifications:

| Requirement | Value |
|-------------|-------|
| **VM Name** | `datacenter-vm` |
| **OS Image** | Ubuntu2204 (Ubuntu 22.04 LTS) |
| **VM Size** | `Standard_B2s` |
| **Admin Username** | `azureuser` |
| **Authentication** | SSH keys (auto-generated) |
| **Storage Type** | `Standard_LRS` (Standard HDD) |
| **Disk Size** | 30 GB |
| **VM State** | Running |
| **Management** | Azure CLI only |

##  Azure Credentials

> **Note:** Run `showcreds` command on the `azure-client` host to retrieve your lab credentials

Access credentials will be provided in the KodeKloud lab environment.

---

##  Execution Environment

All commands are executed in the **KodeKloud lab terminal** on the **azure-client host**.

> [!IMPORTANT]
> You do **NOT** have access to Azure Portal. All operations must be performed via Azure CLI.

---

##  Step-by-Step Solution

### Step 1: Login to Azure CLI

Authenticate to Azure using device login flow.

```bash
az login
```

**What happens:**
- Azure CLI displays a device login URL (e.g., `https://microsoft.com/devicelogin`)
- Provides a unique authentication code

**Authentication process:**
1. Open the displayed URL in your browser
2. Enter the authentication code
3. Sign in with your Azure credentials
4. Return to terminal - login completes automatically


---

### Step 2: Verify Login & Active Subscription

Confirm successful authentication and check active subscription.

```bash
az account show --output table
```

**Expected output:**

```
Name              CloudName    SubscriptionId                         State    IsDefault
----------------  -----------  -------------------------------------  -------  -----------
KodeKloud-Sub     AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   Enabled  True
```

**Success indicators:**
- `State` = `Enabled` ✅
- `IsDefault` = `True` ✅

**Alternative verification (JSON format):**

```bash
az account show
```



> [!TIP]
> If you see multiple subscriptions, ensure the correct one is set as default using `az account set --subscription <SUBSCRIPTION_ID>`

---

### Step 3: Identify Existing Resource Group

List all available resource groups to find the existing one.

```bash
az group list --output table
```

**Expected output:**

```
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-xxxxxxxxxxxxx     westus      Succeeded
```

![Resource Groups](images/task3-resource-groups.png)

**Important notes:**
-  **Note the resource group name** - you'll need it for the next steps
-  **Do NOT create a new resource group** - KodeKloud provides one
-  Use the existing resource group for all operations

---

### Step 4: Create the Virtual Machine 

Execute the main command to create the VM with all required specifications.

```bash
az vm create \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30
```

> [!NOTE]
> Replace `<RESOURCE_GROUP>` with the actual resource group name from Step 3

**Example with actual resource group:**

```bash
az vm create \
  --resource-group kml_rg_main-4cdca7496c4f4938 \
  --name datacenter-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30
```

**Command Parameters Breakdown:**

| Parameter | Value | Purpose | Requirement Met |
|-----------|-------|---------|-----------------|
| `--resource-group` | Existing RG | Uses pre-created resource group | ✅ |
| `--name` | `datacenter-vm` | VM name | ✅ Required name |
| `--image` | `Ubuntu2204` | Operating system | ✅ Ubuntu 22.04 LTS |
| `--size` | `Standard_B2s` | VM size (2 vCPUs, 4GB RAM) | ✅ Required size |
| `--admin-username` | `azureuser` | Admin user account | ✅ Required username |
| `--generate-ssh-keys` | Auto-generate | Creates SSH key pair | ✅ SSH authentication |
| `--storage-sku` | `Standard_LRS` | Storage type (Standard HDD) | ✅ Required storage |
| `--os-disk-size-gb` | `30` | Disk size in GB | ✅ Required size |

**Expected output:**

```json
{
  "fqdns": "",
  "id": "/subscriptions/.../resourceGroups/.../providers/Microsoft.Compute/virtualMachines/datacenter-vm",
  "location": "westus",
  "macAddress": "00-0D-3A-XX-XX-XX",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "20.123.45.67",
  "resourceGroup": "kml_rg_main-xxxxx",
  "zones": ""
}
```



**What Azure does:**
- ✅ Creates the virtual machine with specified configuration
- ✅ Generates SSH key pair (stored in `~/.ssh/id_rsa`)
- ✅ Allocates public and private IP addresses
- ✅ Creates and attaches 30GB Standard HDD disk
- ✅ Starts the VM automatically
- ✅ Returns VM details including IP addresses

> [!TIP]
> VM creation takes approximately 2-3 minutes. Wait for the command to complete before proceeding.

---

### Step 5: Verify VM is in Running State

Confirm the VM is running using instance view query.

```bash
az vm get-instance-view \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --query "instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus" \
  --output table
```

**Expected output:**

```
Result
-----------
VM running
```

**Alternative method (full details):**

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --show-details \
  --query "powerState" \
  --output tsv
```


---

### Step 6: Retrieve Public IP Address

Get the public IP address for SSH connectivity.

```bash
az vm list-ip-addresses \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --output table
```

**Expected output:**

```
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
datacenter-vm     20.123.45.67         10.0.0.4
```

 **Copy the Public IP Address** - you'll need it for SSH access

**Alternative methods:**

**Method 1: Query specific field**

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --show-details \
  --query "publicIps" \
  --output tsv
```

**Method 2: List all network interfaces**

```bash
az vm list-ip-addresses \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm
```

---

### Step 7: Verify SSH Key Generation

Confirm that SSH keys were generated and stored locally.

```bash
ls -l ~/.ssh/id_rsa
```

**Expected output:**

```
-rw------- 1 user user 2602 Jan 16 10:30 /home/user/.ssh/id_rsa
```

**View public key:**

```bash
cat ~/.ssh/id_rsa.pub
```

> [!WARNING]
> **Never share your private key** (`~/.ssh/id_rsa`). Only the public key (`.pub`) should be distributed.

**Verify key permissions:**

```bash
chmod 600 ~/.ssh/id_rsa
```

---

### Step 8: SSH into the Virtual Machine

Connect to the VM to verify SSH access and VM functionality.

```bash
ssh azureuser@<PUBLIC_IP>
```

Replace `<PUBLIC_IP>` with the IP address from Step 6.

**Example:**

```bash
ssh azureuser@20.123.45.67
```

**First-time connection prompt:**

```
The authenticity of host '20.123.45.67 (20.123.45.67)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter.

**Expected result:**

```
Welcome to Ubuntu 22.04.x LTS (GNU/Linux 5.15.x-xxx-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

azureuser@datacenter-vm:~$
```



---

### Step 9: Verify Ubuntu Version

Inside the VM, confirm the operating system version.

```bash
lsb_release -a
```

**Expected output:**

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.x LTS
Release:        22.04
Codename:       jammy
```

**Alternative verification:**

```bash
cat /etc/os-release
```

**Expected output:**

```
NAME="Ubuntu"
VERSION="22.04.x LTS (Jammy Jellyfish)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 22.04.x LTS"
VERSION_ID="22.04"
```

✅ **Confirms Ubuntu 22.04 LTS is installed**

---

### Step 10: Verify VM Specifications (Optional)

Check VM hardware specifications to confirm size.

**Check CPU cores:**

```bash
nproc
```

**Expected:** `2` (Standard_B2s has 2 vCPUs)

**Check memory:**

```bash
free -h
```

**Expected:** Approximately 4GB total memory

**Check disk size:**

```bash
df -h /
```

**Expected:** Approximately 30GB total

**Check OS disk configuration:**

```bash
lsblk
```

**Expected output:**

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   30G  0 disk
├─sda1   8:1    0 29.9G  0 part /
└─sda15  8:15   0  100M  0 part /boot/efi
```

---

### Step 11: Exit VM

Return to the azure-client host.

```bash
exit
```

---

## ✅ Task Validation Checklist

Before submitting, verify all requirements are met:

- [ ] Used Azure CLI only (no portal access)
- [ ] VM name is exactly `datacenter-vm`
- [ ] Ubuntu2204 image used
- [ ] VM size is `Standard_B2s`
- [ ] Admin username is `azureuser`
- [ ] SSH keys generated automatically
- [ ] Storage type is `Standard_LRS`
- [ ] Disk size is 30 GB
- [ ] VM is in running state
- [ ] Successfully SSH'd into the VM
- [ ] Verified Ubuntu 22.04 LTS inside VM

---

##  Final Verification Summary

| Requirement | Expected Value | Verification Command | Status |
|-------------|----------------|----------------------|--------|
| Azure CLI only | CLI operations | All commands via `az` | ✅ |
| VM name | `datacenter-vm` | `az vm show -n datacenter-vm` | ✅ |
| OS Image | Ubuntu 22.04 | `lsb_release -a` (inside VM) | ✅ |
| VM Size | `Standard_B2s` | `az vm show --query "hardwareProfile.vmSize"` | ✅ |
| Admin Username | `azureuser` | `whoami` (inside VM) | ✅ |
| SSH Keys | Auto-generated | `ls ~/.ssh/id_rsa` | ✅ |
| Disk Size | 30 GB | `df -h` (inside VM) | ✅ |
| Storage Type | `Standard_LRS` | `az vm show --query "storageProfile.osDisk.managedDisk.storageAccountType"` | ✅ |
| VM State | Running | `az vm get-instance-view` | ✅ |

---

## 🐛 Common Troubleshooting

### Error: "InvalidTemplateDeployment"

**Cause:** Invalid VM size or image name

**Solution:** Verify exact names:
- Image: `Ubuntu2204` (case-sensitive)
- Size: `Standard_B2s` (case-sensitive)

```bash
# List available Ubuntu images
az vm image list --offer Ubuntu --all --output table

# List available VM sizes
az vm list-sizes --location westus --output table | grep B2s
```

---

### Error: "QuotaExceeded"

**Cause:** Subscription quota limits reached

**Solution:** Check your quota:

```bash
az vm list-usage --location westus --output table
```

Or use a different VM size if allowed.

---

### Error: SSH connection timeout

**Possible causes:**
1. VM still starting up
2. Network security rules blocking SSH
3. Wrong public IP

**Solution:**

**Wait for VM to fully start:**

```bash
az vm get-instance-view \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --query "instanceView.statuses[1].displayStatus"
```

**Check network security group:**

```bash
az network nsg list --resource-group <RESOURCE_GROUP> --output table
```

**Verify correct IP:**

```bash
az vm list-ip-addresses \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --output table
```

---

### Error: "Permission denied (publickey)"

**Solution:**

Ensure SSH key has correct permissions:

```bash
chmod 600 ~/.ssh/id_rsa
```

Manually specify key path:

```bash
ssh -i ~/.ssh/id_rsa azureuser@<PUBLIC_IP>
```

Check if key exists:

```bash
ls -la ~/.ssh/
```

---

### Error: Resource group not found

**Solution:**

List all resource groups:

```bash
az group list --output table
```

Ensure you're using the correct subscription:

```bash
az account show
az account set --subscription <SUBSCRIPTION_ID>
```

---

##  Advanced Verification Commands

### Check VM Size

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --query "hardwareProfile.vmSize" \
  --output tsv
```

**Expected:** `Standard_B2s`

---

### Check Storage Account Type

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --query "storageProfile.osDisk.managedDisk.storageAccountType" \
  --output tsv
```

**Expected:** `Standard_LRS`

---

### Check Disk Size

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --query "storageProfile.osDisk.diskSizeGb" \
  --output tsv
```

**Expected:** `30`

---

### Check OS Details

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --query "storageProfile.imageReference" \
  --output table
```

**Expected:** Ubuntu 22.04 image reference

---

### Get Complete VM Details

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --show-details
```

---

## Security Best Practices

> [!WARNING]
> **Critical Security Considerations**

**SSH Key Security:**
- ✅ Keep private key (`~/.ssh/id_rsa`) secure - **never share it**
- ✅ Set proper permissions: `chmod 600 ~/.ssh/id_rsa`
- ✅ Use passphrase-protected keys in production
- ✅ Regularly rotate SSH keys

**Network Security:**
- Consider restricting SSH access to specific IP ranges
- Enable Azure Security Center for threat detection
- Use Azure Bastion for enhanced security in production
- Regularly update and patch the VM

**Access Management:**
- Use Azure AD integration for user management
- Implement just-in-time (JIT) access
- Enable multi-factor authentication
- Audit access logs regularly

---

## 🧹 Cleanup (Optional)

To delete the VM and associated resources after completing the task:

### Delete VM Only (Keeps Network Resources)

```bash
az vm delete \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --yes
```

### Delete VM and All Associated Resources

```bash
# Delete VM
az vm delete \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --yes

# Delete network interface
az network nic delete \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vmVMNic

# Delete public IP
az network public-ip delete \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vmPublicIP

# Delete network security group (if created)
az network nsg delete \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vmNSG
```

> [!CAUTION]
> Only run cleanup commands after task completion and verification. Deletion is **irreversible**.

---

## 📚 Additional Resources

- [Azure Virtual Machines Documentation](https://docs.microsoft.com/azure/virtual-machines/)
- [Azure CLI VM Commands Reference](https://docs.microsoft.com/cli/azure/vm)
- [Ubuntu on Azure](https://ubuntu.com/azure)
- [Azure VM Sizes - B-series](https://docs.microsoft.com/azure/virtual-machines/sizes-b-series-burstable)
- [SSH Key Management in Azure](https://docs.microsoft.com/azure/virtual-machines/linux/create-ssh-keys-detailed)

---

